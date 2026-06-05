# ARMTargetTransformInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMTargetTransformInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMTargetTransformInfo` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `ARMTargetTransformInfo`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ARMTargetTransformInfo.cpp - ARM specific TTI ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-23
```cpp
#include "ARMTargetTransformInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/Analysis/LoopInfo.h"
#include "llvm/CodeGen/CostTable.h"
#include "llvm/CodeGen/ISDOpcodes.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-38
```cpp
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsARM.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/IR/Type.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/KnownBits.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/SubtargetFeature.h"
#include "llvm/Transforms/InstCombine/InstCombiner.h"
#include "llvm/Transforms/Utils/Local.h"
#include "llvm/Transforms/Utils/LoopUtils.h"
#include "llvm/Transforms/Vectorize/LoopVectorizationLegality.h"
#include <algorithm>
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 39-41
```cpp
#include <cstdint>
#include <optional>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-43
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 45-45
```cpp
#define DEBUG_TYPE "armtti"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 47-49
```cpp
static cl::opt<bool> EnableMaskedLoadStores(
  "enable-arm-maskedldst", cl::Hidden, cl::init(true),
  cl::desc("Enable the generation of masked loads and stores"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 51-53
```cpp
static cl::opt<bool> DisableLowOverheadLoops(
  "disable-arm-loloops", cl::Hidden, cl::init(false),
  cl::desc("Disable the generation of low-overhead loops"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-57
```cpp
static cl::opt<bool>
    AllowWLSLoops("allow-arm-wlsloops", cl::Hidden, cl::init(true),
                  cl::desc("Enable the generation of WLS loops"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-61
```cpp
static cl::opt<bool> UseWidenGlobalArrays(
    "widen-global-strings", cl::Hidden, cl::init(true),
    cl::desc("Enable the widening of global strings to alignment boundaries"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-63
```cpp
extern cl::opt<TailPredication::Mode> EnableTailPredication;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 65-65
```cpp
extern cl::opt<bool> EnableMaskedGatherScatters;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 67-67
```cpp
extern cl::opt<unsigned> MVEMaxSupportedInterleaveFactor;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 69-72
```cpp
static cl::opt<int> ArmForceUnrollThreshold(
    "arm-force-unroll-threshold", cl::init(12), cl::Hidden,
    cl::desc(
        "Threshold for forced unrolling of small loops in Arm architecture"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 74-79
```cpp
/// Convert a vector load intrinsic into a simple llvm load instruction.
/// This is beneficial when the underlying object being addressed comes
/// from a constant, since we get constant-folding for free.
static Value *simplifyNeonVld1(const IntrinsicInst &II, unsigned MemAlign,
                               InstCombiner::BuilderTy &Builder) {
  auto *IntrAlign = dyn_cast<ConstantInt>(II.getArgOperand(1));
```
- EN: Implements `simplifyNeonVld1`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `simplifyNeonVld1`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 81-82
```cpp
  if (!IntrAlign)
    return nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 84-86
```cpp
  unsigned Alignment = IntrAlign->getLimitedValue() < MemAlign
                           ? MemAlign
                           : IntrAlign->getLimitedValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 88-89
```cpp
  if (!isPowerOf2_32(Alignment))
    return nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 91-93
```cpp
  return Builder.CreateAlignedLoad(II.getType(), II.getArgOperand(0),
                                   Align(Alignment));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-101
```cpp
bool ARMTTIImpl::areInlineCompatible(const Function *Caller,
                                     const Function *Callee) const {
  const TargetMachine &TM = getTLI()->getTargetMachine();
  const FeatureBitset &CallerBits =
      TM.getSubtargetImpl(*Caller)->getFeatureBits();
  const FeatureBitset &CalleeBits =
      TM.getSubtargetImpl(*Callee)->getFeatureBits();
```
- EN: Implements `ARMTTIImpl::areInlineCompatible`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::areInlineCompatible`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 103-109
```cpp
  // To inline a callee, all features not in the allowed list must match exactly.
  bool MatchExact = (CallerBits & ~InlineFeaturesAllowed) ==
                    (CalleeBits & ~InlineFeaturesAllowed);
  // For features in the allowed list, the callee's features must be a subset of
  // the callers'.
  bool MatchSubset = ((CallerBits & CalleeBits) & InlineFeaturesAllowed) ==
                     (CalleeBits & InlineFeaturesAllowed);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 111-115
```cpp
  LLVM_DEBUG({
    if (!MatchExact || !MatchSubset) {
      dbgs() << "=== Inline compatibility debug ===\n";
      dbgs() << "Caller: " << Caller->getName() << "\n";
      dbgs() << "Callee: " << Callee->getName() << "\n";
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 117-119
```cpp
      // Bit diffs
      FeatureBitset MissingInCaller = CalleeBits & ~CallerBits; // callee-only
      FeatureBitset ExtraInCaller = CallerBits & ~CalleeBits;   // caller-only
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-123
```cpp
      // Counts
      dbgs() << "Only-in-caller bit count: " << ExtraInCaller.count() << "\n";
      dbgs() << "Only-in-callee bit count: " << MissingInCaller.count() << "\n";
```
- EN: Declares `dbgs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `dbgs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 125-137
```cpp
      dbgs() << "Only-in-caller feature indices [";
      {
        bool First = true;
        for (size_t I = 0, E = ExtraInCaller.size(); I < E; ++I) {
          if (ExtraInCaller.test(I)) {
            if (!First)
              dbgs() << ", ";
            dbgs() << I;
            First = false;
          }
        }
      }
      dbgs() << "]\n";
```
- EN: Implements `dbgs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `dbgs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 139-151
```cpp
      dbgs() << "Only-in-callee feature indices [";
      {
        bool First = true;
        for (size_t I = 0, E = MissingInCaller.size(); I < E; ++I) {
          if (MissingInCaller.test(I)) {
            if (!First)
              dbgs() << ", ";
            dbgs() << I;
            First = false;
          }
        }
      }
      dbgs() << "]\n";
```
- EN: Implements `dbgs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `dbgs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 153-160
```cpp
      // Indices map to features as found in
      // llvm-project/(your_build)/lib/Target/ARM/ARMGenSubtargetInfo.inc
      dbgs() << "MatchExact=" << (MatchExact ? "true" : "false")
             << " MatchSubset=" << (MatchSubset ? "true" : "false") << "\n";
    }
  });
  return MatchExact && MatchSubset;
}
```
- EN: Declares `dbgs`, a target-specific routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `dbgs`，它是一个围绕子目标特性处理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 162-166
```cpp
TTI::AddressingModeKind
ARMTTIImpl::getPreferredAddressingMode(const Loop *L,
                                       ScalarEvolution *SE) const {
  if (ST->hasMVEIntegerOps())
    return TTI::AMK_PostIndexed;
```
- EN: Implements `ARMTTIImpl::getPreferredAddressingMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getPreferredAddressingMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 168-169
```cpp
  if (L->getHeader()->getParent()->hasOptSize())
    return TTI::AMK_None;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 171-173
```cpp
  if (ST->isMClass() && ST->isThumb2() &&
      L->getNumBlocks() == 1)
    return TTI::AMK_PreIndexed;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 175-176
```cpp
  return TTI::AMK_None;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 178-192
```cpp
std::optional<Instruction *>
ARMTTIImpl::instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const {
  using namespace PatternMatch;
  Intrinsic::ID IID = II.getIntrinsicID();
  switch (IID) {
  default:
    break;
  case Intrinsic::arm_neon_vld1: {
    Align MemAlign =
        getKnownAlignment(II.getArgOperand(0), IC.getDataLayout(), &II,
                          &IC.getAssumptionCache(), &IC.getDominatorTree());
    if (Value *V = simplifyNeonVld1(II, MemAlign.value(), IC.Builder)) {
      return IC.replaceInstUsesWith(II, V);
    }
    break;
```
- EN: Implements `ARMTTIImpl::instCombineIntrinsic`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::instCombineIntrinsic`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-193
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 195-209
```cpp
  case Intrinsic::arm_neon_vld2:
  case Intrinsic::arm_neon_vld3:
  case Intrinsic::arm_neon_vld4:
  case Intrinsic::arm_neon_vld2lane:
  case Intrinsic::arm_neon_vld3lane:
  case Intrinsic::arm_neon_vld4lane:
  case Intrinsic::arm_neon_vst1:
  case Intrinsic::arm_neon_vst2:
  case Intrinsic::arm_neon_vst3:
  case Intrinsic::arm_neon_vst4:
  case Intrinsic::arm_neon_vst2lane:
  case Intrinsic::arm_neon_vst3lane:
  case Intrinsic::arm_neon_vst4lane: {
    Align MemAlign =
        getKnownAlignment(II.getArgOperand(0), IC.getDataLayout(), &II,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 210-221
```cpp
                          &IC.getAssumptionCache(), &IC.getDominatorTree());
    unsigned AlignArg = II.arg_size() - 1;
    Value *AlignArgOp = II.getArgOperand(AlignArg);
    MaybeAlign Align = cast<ConstantInt>(AlignArgOp)->getMaybeAlignValue();
    if (Align && *Align < MemAlign) {
      return IC.replaceOperand(
          II, AlignArg,
          ConstantInt::get(Type::getInt32Ty(II.getContext()), MemAlign.value(),
                           false));
    }
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 223-237
```cpp
  case Intrinsic::arm_neon_vld1x2:
  case Intrinsic::arm_neon_vld1x3:
  case Intrinsic::arm_neon_vld1x4:
  case Intrinsic::arm_neon_vst1x2:
  case Intrinsic::arm_neon_vst1x3:
  case Intrinsic::arm_neon_vst1x4: {
    Align NewAlign =
        getKnownAlignment(II.getArgOperand(0), IC.getDataLayout(), &II,
                          &IC.getAssumptionCache(), &IC.getDominatorTree());
    Align OldAlign = II.getParamAlign(0).valueOrOne();
    if (NewAlign > OldAlign)
      II.addParamAttr(0,
                      Attribute::getWithAlignment(II.getContext(), NewAlign));
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 239-253
```cpp
  case Intrinsic::arm_mve_pred_i2v: {
    Value *Arg = II.getArgOperand(0);
    Value *ArgArg;
    if (match(Arg, PatternMatch::m_Intrinsic<Intrinsic::arm_mve_pred_v2i>(
                       PatternMatch::m_Value(ArgArg))) &&
        II.getType() == ArgArg->getType()) {
      return IC.replaceInstUsesWith(II, ArgArg);
    }
    Constant *XorMask;
    if (match(Arg, m_Xor(PatternMatch::m_Intrinsic<Intrinsic::arm_mve_pred_v2i>(
                             PatternMatch::m_Value(ArgArg)),
                         PatternMatch::m_Constant(XorMask))) &&
        II.getType() == ArgArg->getType()) {
      if (auto *CI = dyn_cast<ConstantInt>(XorMask)) {
        if (CI->getValue().trunc(16).isAllOnes()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 254-268
```cpp
          auto TrueVector = IC.Builder.CreateVectorSplat(
              cast<FixedVectorType>(II.getType())->getNumElements(),
              IC.Builder.getTrue());
          return BinaryOperator::Create(Instruction::Xor, ArgArg, TrueVector);
        }
      }
    }
    KnownBits ScalarKnown(32);
    if (IC.SimplifyDemandedBits(&II, 0, APInt::getLowBitsSet(32, 16),
                                ScalarKnown)) {
      return &II;
    }
    break;
  }
  case Intrinsic::arm_mve_pred_v2i: {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 269-274
```cpp
    Value *Arg = II.getArgOperand(0);
    Value *ArgArg;
    if (match(Arg, PatternMatch::m_Intrinsic<Intrinsic::arm_mve_pred_i2v>(
                       PatternMatch::m_Value(ArgArg)))) {
      return IC.replaceInstUsesWith(II, ArgArg);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 276-277
```cpp
    if (II.getMetadata(LLVMContext::MD_range))
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-279
```cpp
    ConstantRange Range(APInt(32, 0), APInt(32, 0x10000));
```
- EN: Declares `Range`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Range`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 281-285
```cpp
    if (auto CurrentRange = II.getRange()) {
      Range = Range.intersectWith(*CurrentRange);
      if (Range == CurrentRange)
        break;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 287-296
```cpp
    II.addRangeRetAttr(Range);
    II.addRetAttr(Attribute::NoUndef);
    return &II;
  }
  case Intrinsic::arm_mve_vadc:
  case Intrinsic::arm_mve_vadc_predicated: {
    unsigned CarryOp =
        (II.getIntrinsicID() == Intrinsic::arm_mve_vadc_predicated) ? 3 : 2;
    assert(II.getArgOperand(CarryOp)->getType()->getScalarSizeInBits() == 32 &&
           "Bad type for intrinsic!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 298-312
```cpp
    KnownBits CarryKnown(32);
    if (IC.SimplifyDemandedBits(&II, CarryOp, APInt::getOneBitSet(32, 29),
                                CarryKnown)) {
      return &II;
    }
    break;
  }
  case Intrinsic::arm_mve_vmldava: {
    Instruction *I = cast<Instruction>(&II);
    if (I->hasOneUse()) {
      auto *User = cast<Instruction>(*I->user_begin());
      Value *OpZ;
      if (match(User, m_c_Add(m_Specific(I), m_Value(OpZ))) &&
          match(I->getOperand(3), m_Zero())) {
        Value *OpX = I->getOperand(4);
```
- EN: Implements `CarryKnown`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `CarryKnown`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-314
```cpp
        Value *OpY = I->getOperand(5);
        Type *OpTy = OpX->getType();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 316-320
```cpp
        IC.Builder.SetInsertPoint(User);
        Value *V =
            IC.Builder.CreateIntrinsic(Intrinsic::arm_mve_vmldava, {OpTy},
                                       {I->getOperand(0), I->getOperand(1),
                                        I->getOperand(2), OpZ, OpX, OpY});
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 322-330
```cpp
        IC.replaceInstUsesWith(*User, V);
        return IC.eraseInstFromFunction(*User);
      }
    }
    return std::nullopt;
  }
  }
  return std::nullopt;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 332-336
```cpp
std::optional<Value *> ARMTTIImpl::simplifyDemandedVectorEltsIntrinsic(
    InstCombiner &IC, IntrinsicInst &II, APInt OrigDemandedElts,
    APInt &UndefElts, APInt &UndefElts2, APInt &UndefElts3,
    std::function<void(Instruction *, unsigned, APInt, APInt &)>
        SimplifyAndSetOp) const {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 338-343
```cpp
  // Compute the demanded bits for a narrowing MVE intrinsic. The TopOpc is the
  // opcode specifying a Top/Bottom instruction, which can change between
  // instructions.
  auto SimplifyNarrowInstrTopBottom =[&](unsigned TopOpc) {
    unsigned NumElts = cast<FixedVectorType>(II.getType())->getNumElements();
    unsigned IsTop = cast<ConstantInt>(II.getOperand(TopOpc))->getZExtValue();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 345-355
```cpp
    // The only odd/even lanes of operand 0 will only be demanded depending
    // on whether this is a top/bottom instruction.
    APInt DemandedElts =
        APInt::getSplat(NumElts, IsTop ? APInt::getLowBitsSet(2, 1)
                                       : APInt::getHighBitsSet(2, 1));
    SimplifyAndSetOp(&II, 0, OrigDemandedElts & DemandedElts, UndefElts);
    // The other lanes will be defined from the inserted elements.
    UndefElts &= APInt::getSplat(NumElts, IsTop ? APInt::getLowBitsSet(2, 1)
                                                : APInt::getHighBitsSet(2, 1));
    return std::nullopt;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 357-369
```cpp
  switch (II.getIntrinsicID()) {
  default:
    break;
  case Intrinsic::arm_mve_vcvt_narrow:
    SimplifyNarrowInstrTopBottom(2);
    break;
  case Intrinsic::arm_mve_vqmovn:
    SimplifyNarrowInstrTopBottom(4);
    break;
  case Intrinsic::arm_mve_vshrn:
    SimplifyNarrowInstrTopBottom(7);
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 371-372
```cpp
  return std::nullopt;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 374-376
```cpp
InstructionCost ARMTTIImpl::getIntImmCost(const APInt &Imm, Type *Ty,
                                          TTI::TargetCostKind CostKind) const {
  assert(Ty->isIntegerTy());
```
- EN: Implements `ARMTTIImpl::getIntImmCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getIntImmCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 378-380
```cpp
 unsigned Bits = Ty->getPrimitiveSizeInBits();
 if (Bits == 0 || Imm.getActiveBits() >= 64)
   return 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 382-396
```cpp
  int64_t SImmVal = Imm.getSExtValue();
  uint64_t ZImmVal = Imm.getZExtValue();
  if (!ST->isThumb()) {
    if ((SImmVal >= 0 && SImmVal < 65536) ||
        (ARM_AM::getSOImmVal(ZImmVal) != -1) ||
        (ARM_AM::getSOImmVal(~ZImmVal) != -1))
      return 1;
    return ST->hasV6T2Ops() ? 2 : 3;
  }
  if (ST->isThumb2()) {
    if ((SImmVal >= 0 && SImmVal < 65536) ||
        (ARM_AM::getT2SOImmVal(ZImmVal) != -1) ||
        (ARM_AM::getT2SOImmVal(~ZImmVal) != -1))
      return 1;
    return ST->hasV6T2Ops() ? 2 : 3;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 397-405
```cpp
  }
  // Thumb1, any i8 imm cost 1.
  if (Bits == 8 || (SImmVal >= 0 && SImmVal < 256))
    return 1;
  if ((~SImmVal < 256) || ARM_AM::isThumbImmShiftedVal(ZImmVal))
    return 2;
  // Load from constantpool.
  return 3;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 407-413
```cpp
// Constants smaller than 256 fit in the immediate field of
// Thumb1 instructions so we return a zero cost and 1 otherwise.
InstructionCost ARMTTIImpl::getIntImmCodeSizeCost(unsigned Opcode, unsigned Idx,
                                                  const APInt &Imm,
                                                  Type *Ty) const {
  if (Imm.isNonNegative() && Imm.getLimitedValue() < 256)
    return 0;
```
- EN: Implements `ARMTTIImpl::getIntImmCodeSizeCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getIntImmCodeSizeCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 415-416
```cpp
  return 1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 418-424
```cpp
// Checks whether Inst is part of a min(max()) or max(min()) pattern
// that will match to an SSAT instruction. Returns the instruction being
// saturated, or null if no saturation pattern was found.
static Value *isSSATMinMaxPattern(Instruction *Inst, const APInt &Imm) {
  Value *LHS, *RHS;
  ConstantInt *C;
  SelectPatternFlavor InstSPF = matchSelectPattern(Inst, LHS, RHS).Flavor;
```
- EN: Implements `isSSATMinMaxPattern`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSSATMinMaxPattern`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 426-428
```cpp
  if (InstSPF == SPF_SMAX &&
      PatternMatch::match(RHS, PatternMatch::m_ConstantInt(C)) &&
      C->getValue() == Imm && Imm.isNegative() && Imm.isNegatedPowerOf2()) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 430-442
```cpp
    auto isSSatMin = [&](Value *MinInst) {
      if (isa<SelectInst>(MinInst)) {
        Value *MinLHS, *MinRHS;
        ConstantInt *MinC;
        SelectPatternFlavor MinSPF =
            matchSelectPattern(MinInst, MinLHS, MinRHS).Flavor;
        if (MinSPF == SPF_SMIN &&
            PatternMatch::match(MinRHS, PatternMatch::m_ConstantInt(MinC)) &&
            MinC->getValue() == ((-Imm) - 1))
          return true;
      }
      return false;
    };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 444-451
```cpp
    if (isSSatMin(Inst->getOperand(1)))
      return cast<Instruction>(Inst->getOperand(1))->getOperand(1);
    if (Inst->hasNUses(2) &&
        (isSSatMin(*Inst->user_begin()) || isSSatMin(*(++Inst->user_begin()))))
      return Inst->getOperand(1);
  }
  return nullptr;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 453-465
```cpp
// Look for a FP Saturation pattern, where the instruction can be simplified to
// a fptosi.sat. max(min(fptosi)). The constant in this case is always free.
static bool isFPSatMinMaxPattern(Instruction *Inst, const APInt &Imm) {
  if (Imm.getBitWidth() != 64 ||
      Imm != APInt::getHighBitsSet(64, 33)) // -2147483648
    return false;
  Value *FP = isSSATMinMaxPattern(Inst, Imm);
  if (!FP && isa<ICmpInst>(Inst) && Inst->hasOneUse())
    FP = isSSATMinMaxPattern(cast<Instruction>(*Inst->user_begin()), Imm);
  if (!FP)
    return false;
  return isa<FPToSIInst>(FP);
}
```
- EN: Implements `isFPSatMinMaxPattern`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isFPSatMinMaxPattern`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 467-478
```cpp
InstructionCost ARMTTIImpl::getIntImmCostInst(unsigned Opcode, unsigned Idx,
                                              const APInt &Imm, Type *Ty,
                                              TTI::TargetCostKind CostKind,
                                              Instruction *Inst) const {
  // Division by a constant can be turned into multiplication, but only if we
  // know it's constant. So it's not so much that the immediate is cheap (it's
  // not), but that the alternative is worse.
  // FIXME: this is probably unneeded with GlobalISel.
  if ((Opcode == Instruction::SDiv || Opcode == Instruction::UDiv ||
       Opcode == Instruction::SRem || Opcode == Instruction::URem) &&
      Idx == 1)
    return 0;
```
- EN: Implements `ARMTTIImpl::getIntImmCostInst`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getIntImmCostInst`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 480-483
```cpp
  // Leave any gep offsets for the CodeGenPrepare, which will do a better job at
  // splitting any large offsets.
  if (Opcode == Instruction::GetElementPtr && Idx != 0)
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 485-492
```cpp
  if (Opcode == Instruction::And) {
    // UXTB/UXTH
    if (Imm == 255 || Imm == 65535)
      return 0;
    // Conversion to BIC is free, and means we can use ~Imm instead.
    return std::min(getIntImmCost(Imm, Ty, CostKind),
                    getIntImmCost(~Imm, Ty, CostKind));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 494-497
```cpp
  if (Opcode == Instruction::Add)
    // Conversion to SUB is free, and means we can use -Imm instead.
    return std::min(getIntImmCost(Imm, Ty, CostKind),
                    getIntImmCost(-Imm, Ty, CostKind));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 499-508
```cpp
  if (Opcode == Instruction::ICmp && Imm.isNegative() &&
      Ty->getIntegerBitWidth() == 32) {
    int64_t NegImm = -Imm.getSExtValue();
    if (ST->isThumb2() && NegImm < 1<<12)
      // icmp X, #-C -> cmn X, #C
      return 0;
    if (ST->isThumb() && NegImm < 1<<8)
      // icmp X, #-C -> adds X, #C
      return 0;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 510-512
```cpp
  // xor a, -1 can always be folded to MVN
  if (Opcode == Instruction::Xor && Imm.isAllOnes())
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 514-522
```cpp
  // Ensures negative constant of min(max()) or max(min()) patterns that
  // match to SSAT instructions don't get hoisted
  if (Inst && ((ST->hasV6Ops() && !ST->isThumb()) || ST->isThumb2()) &&
      Ty->getIntegerBitWidth() <= 32) {
    if (isSSATMinMaxPattern(Inst, Imm) ||
        (isa<ICmpInst>(Inst) && Inst->hasOneUse() &&
         isSSATMinMaxPattern(cast<Instruction>(*Inst->user_begin()), Imm)))
      return 0;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 524-525
```cpp
  if (Inst && ST->hasVFP2Base() && isFPSatMinMaxPattern(Inst, Imm))
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 527-533
```cpp
  // We can convert <= -1 to < 0, which is generally quite cheap.
  if (Inst && Opcode == Instruction::ICmp && Idx == 1 && Imm.isAllOnes()) {
    ICmpInst::Predicate Pred = cast<ICmpInst>(Inst)->getPredicate();
    if (Pred == ICmpInst::ICMP_SGT || Pred == ICmpInst::ICMP_SLE)
      return std::min(getIntImmCost(Imm, Ty, CostKind),
                      getIntImmCost(Imm + 1, Ty, CostKind));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 535-536
```cpp
  return getIntImmCost(Imm, Ty, CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 538-550
```cpp
InstructionCost ARMTTIImpl::getCFInstrCost(unsigned Opcode,
                                           TTI::TargetCostKind CostKind,
                                           const Instruction *I) const {
  if (CostKind == TTI::TCK_RecipThroughput &&
      (ST->hasNEON() || ST->hasMVEIntegerOps())) {
    // FIXME: The vectorizer is highly sensitive to the cost of these
    // instructions, which suggests that it may be using the costs incorrectly.
    // But, for now, just make them free to avoid performance regressions for
    // vector targets.
    return 0;
  }
  return BaseT::getCFInstrCost(Opcode, CostKind, I);
}
```
- EN: Implements `ARMTTIImpl::getCFInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getCFInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 552-558
```cpp
InstructionCost ARMTTIImpl::getCastInstrCost(unsigned Opcode, Type *Dst,
                                             Type *Src,
                                             TTI::CastContextHint CCH,
                                             TTI::TargetCostKind CostKind,
                                             const Instruction *I) const {
  int ISD = TLI->InstructionOpcodeToISD(Opcode);
  assert(ISD && "Invalid opcode");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 560-571
```cpp
  // TODO: Allow non-throughput costs that aren't binary.
  auto AdjustCost = [&CostKind](InstructionCost Cost) -> InstructionCost {
    if (CostKind != TTI::TCK_RecipThroughput)
      return Cost == 0 ? 0 : 1;
    return Cost;
  };
  auto IsLegalFPType = [this](EVT VT) {
    EVT EltVT = VT.getScalarType();
    return (EltVT == MVT::f32 && ST->hasVFP2Base()) ||
            (EltVT == MVT::f64 && ST->hasFP64()) ||
            (EltVT == MVT::f16 && ST->hasFullFP16());
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 573-574
```cpp
  EVT SrcTy = TLI->getValueType(DL, Src);
  EVT DstTy = TLI->getValueType(DL, Dst);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 576-578
```cpp
  if (!SrcTy.isSimple() || !DstTy.isSimple())
    return AdjustCost(
        BaseT::getCastInstrCost(Opcode, Dst, Src, CCH, CostKind, I));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 580-591
```cpp
  // Extending masked load/Truncating masked stores is expensive because we
  // currently don't split them. This means that we'll likely end up
  // loading/storing each element individually (hence the high cost).
  if ((ST->hasMVEIntegerOps() &&
       (Opcode == Instruction::Trunc || Opcode == Instruction::ZExt ||
        Opcode == Instruction::SExt)) ||
      (ST->hasMVEFloatOps() &&
       (Opcode == Instruction::FPExt || Opcode == Instruction::FPTrunc) &&
       IsLegalFPType(SrcTy) && IsLegalFPType(DstTy)))
    if (CCH == TTI::CastContextHint::Masked && DstTy.getSizeInBits() > 128)
      return 2 * DstTy.getVectorNumElements() *
             ST->getMVEVectorCostFactor(CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 593-607
```cpp
  // The extend of other kinds of load is free
  if (CCH == TTI::CastContextHint::Normal ||
      CCH == TTI::CastContextHint::Masked) {
    static const TypeConversionCostTblEntry LoadConversionTbl[] = {
        {ISD::SIGN_EXTEND, MVT::i32, MVT::i16, 0},
        {ISD::ZERO_EXTEND, MVT::i32, MVT::i16, 0},
        {ISD::SIGN_EXTEND, MVT::i32, MVT::i8, 0},
        {ISD::ZERO_EXTEND, MVT::i32, MVT::i8, 0},
        {ISD::SIGN_EXTEND, MVT::i16, MVT::i8, 0},
        {ISD::ZERO_EXTEND, MVT::i16, MVT::i8, 0},
        {ISD::SIGN_EXTEND, MVT::i64, MVT::i32, 1},
        {ISD::ZERO_EXTEND, MVT::i64, MVT::i32, 1},
        {ISD::SIGN_EXTEND, MVT::i64, MVT::i16, 1},
        {ISD::ZERO_EXTEND, MVT::i64, MVT::i16, 1},
        {ISD::SIGN_EXTEND, MVT::i64, MVT::i8, 1},
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 608-612
```cpp
        {ISD::ZERO_EXTEND, MVT::i64, MVT::i8, 1},
    };
    if (const auto *Entry = ConvertCostTableLookup(
            LoadConversionTbl, ISD, DstTy.getSimpleVT(), SrcTy.getSimpleVT()))
      return AdjustCost(Entry->Cost);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 614-628
```cpp
    static const TypeConversionCostTblEntry MVELoadConversionTbl[] = {
        {ISD::SIGN_EXTEND, MVT::v4i32, MVT::v4i16, 0},
        {ISD::ZERO_EXTEND, MVT::v4i32, MVT::v4i16, 0},
        {ISD::SIGN_EXTEND, MVT::v4i32, MVT::v4i8, 0},
        {ISD::ZERO_EXTEND, MVT::v4i32, MVT::v4i8, 0},
        {ISD::SIGN_EXTEND, MVT::v8i16, MVT::v8i8, 0},
        {ISD::ZERO_EXTEND, MVT::v8i16, MVT::v8i8, 0},
        // The following extend from a legal type to an illegal type, so need to
        // split the load. This introduced an extra load operation, but the
        // extend is still "free".
        {ISD::SIGN_EXTEND, MVT::v8i32, MVT::v8i16, 1},
        {ISD::ZERO_EXTEND, MVT::v8i32, MVT::v8i16, 1},
        {ISD::SIGN_EXTEND, MVT::v16i32, MVT::v16i8, 3},
        {ISD::ZERO_EXTEND, MVT::v16i32, MVT::v16i8, 3},
        {ISD::SIGN_EXTEND, MVT::v16i16, MVT::v16i8, 1},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 629-636
```cpp
        {ISD::ZERO_EXTEND, MVT::v16i16, MVT::v16i8, 1},
    };
    if (SrcTy.isVector() && ST->hasMVEIntegerOps()) {
      if (const auto *Entry =
              ConvertCostTableLookup(MVELoadConversionTbl, ISD,
                                     DstTy.getSimpleVT(), SrcTy.getSimpleVT()))
        return Entry->Cost * ST->getMVEVectorCostFactor(CostKind);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 638-648
```cpp
    static const TypeConversionCostTblEntry MVEFLoadConversionTbl[] = {
        // FPExtends are similar but also require the VCVT instructions.
        {ISD::FP_EXTEND, MVT::v4f32, MVT::v4f16, 1},
        {ISD::FP_EXTEND, MVT::v8f32, MVT::v8f16, 3},
    };
    if (SrcTy.isVector() && ST->hasMVEFloatOps()) {
      if (const auto *Entry =
              ConvertCostTableLookup(MVEFLoadConversionTbl, ISD,
                                     DstTy.getSimpleVT(), SrcTy.getSimpleVT()))
        return Entry->Cost * ST->getMVEVectorCostFactor(CostKind);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 650-664
```cpp
    // The truncate of a store is free. This is the mirror of extends above.
    static const TypeConversionCostTblEntry MVEStoreConversionTbl[] = {
        {ISD::TRUNCATE, MVT::v4i32, MVT::v4i16, 0},
        {ISD::TRUNCATE, MVT::v4i32, MVT::v4i8, 0},
        {ISD::TRUNCATE, MVT::v8i16, MVT::v8i8, 0},
        {ISD::TRUNCATE, MVT::v8i32, MVT::v8i16, 1},
        {ISD::TRUNCATE, MVT::v8i32, MVT::v8i8, 1},
        {ISD::TRUNCATE, MVT::v16i32, MVT::v16i8, 3},
        {ISD::TRUNCATE, MVT::v16i16, MVT::v16i8, 1},
    };
    if (SrcTy.isVector() && ST->hasMVEIntegerOps()) {
      if (const auto *Entry =
              ConvertCostTableLookup(MVEStoreConversionTbl, ISD,
                                     SrcTy.getSimpleVT(), DstTy.getSimpleVT()))
        return Entry->Cost * ST->getMVEVectorCostFactor(CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 665-665
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 667-677
```cpp
    static const TypeConversionCostTblEntry MVEFStoreConversionTbl[] = {
        {ISD::FP_ROUND, MVT::v4f32, MVT::v4f16, 1},
        {ISD::FP_ROUND, MVT::v8f32, MVT::v8f16, 3},
    };
    if (SrcTy.isVector() && ST->hasMVEFloatOps()) {
      if (const auto *Entry =
              ConvertCostTableLookup(MVEFStoreConversionTbl, ISD,
                                     SrcTy.getSimpleVT(), DstTy.getSimpleVT()))
        return Entry->Cost * ST->getMVEVectorCostFactor(CostKind);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 679-693
```cpp
  // NEON vector operations that can extend their inputs.
  if ((ISD == ISD::SIGN_EXTEND || ISD == ISD::ZERO_EXTEND) &&
      I && I->hasOneUse() && ST->hasNEON() && SrcTy.isVector()) {
    static const TypeConversionCostTblEntry NEONDoubleWidthTbl[] = {
      // vaddl
      { ISD::ADD, MVT::v4i32, MVT::v4i16, 0 },
      { ISD::ADD, MVT::v8i16, MVT::v8i8,  0 },
      // vsubl
      { ISD::SUB, MVT::v4i32, MVT::v4i16, 0 },
      { ISD::SUB, MVT::v8i16, MVT::v8i8,  0 },
      // vmull
      { ISD::MUL, MVT::v4i32, MVT::v4i16, 0 },
      { ISD::MUL, MVT::v8i16, MVT::v8i8,  0 },
      // vshll
      { ISD::SHL, MVT::v4i32, MVT::v4i16, 0 },
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 694-695
```cpp
      { ISD::SHL, MVT::v8i16, MVT::v8i8,  0 },
    };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 697-704
```cpp
    auto *User = cast<Instruction>(*I->user_begin());
    int UserISD = TLI->InstructionOpcodeToISD(User->getOpcode());
    if (auto *Entry = ConvertCostTableLookup(NEONDoubleWidthTbl, UserISD,
                                             DstTy.getSimpleVT(),
                                             SrcTy.getSimpleVT())) {
      return AdjustCost(Entry->Cost);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 706-716
```cpp
  // Single to/from double precision conversions.
  if (Src->isVectorTy() && ST->hasNEON() &&
      ((ISD == ISD::FP_ROUND && SrcTy.getScalarType() == MVT::f64 &&
        DstTy.getScalarType() == MVT::f32) ||
       (ISD == ISD::FP_EXTEND && SrcTy.getScalarType() == MVT::f32 &&
        DstTy.getScalarType() == MVT::f64))) {
    static const CostTblEntry NEONFltDblTbl[] = {
        // Vector fptrunc/fpext conversions.
        {ISD::FP_ROUND, MVT::v2f64, 2},
        {ISD::FP_EXTEND, MVT::v2f32, 2},
        {ISD::FP_EXTEND, MVT::v4f32, 4}};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 718-721
```cpp
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(Src);
    if (const auto *Entry = CostTableLookup(NEONFltDblTbl, ISD, LT.second))
      return AdjustCost(LT.first * Entry->Cost);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 723-732
```cpp
  // Some arithmetic, load and store operations have specific instructions
  // to cast up/down their types automatically at no extra cost.
  // TODO: Get these tables to know at least what the related operations are.
  static const TypeConversionCostTblEntry NEONVectorConversionTbl[] = {
    { ISD::SIGN_EXTEND, MVT::v4i32, MVT::v4i16, 1 },
    { ISD::ZERO_EXTEND, MVT::v4i32, MVT::v4i16, 1 },
    { ISD::SIGN_EXTEND, MVT::v2i64, MVT::v2i32, 1 },
    { ISD::ZERO_EXTEND, MVT::v2i64, MVT::v2i32, 1 },
    { ISD::TRUNCATE,    MVT::v4i32, MVT::v4i64, 0 },
    { ISD::TRUNCATE,    MVT::v4i16, MVT::v4i32, 1 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 734-748
```cpp
    // The number of vmovl instructions for the extension.
    { ISD::SIGN_EXTEND, MVT::v8i16, MVT::v8i8,  1 },
    { ISD::ZERO_EXTEND, MVT::v8i16, MVT::v8i8,  1 },
    { ISD::SIGN_EXTEND, MVT::v4i32, MVT::v4i8,  2 },
    { ISD::ZERO_EXTEND, MVT::v4i32, MVT::v4i8,  2 },
    { ISD::SIGN_EXTEND, MVT::v2i64, MVT::v2i8,  3 },
    { ISD::ZERO_EXTEND, MVT::v2i64, MVT::v2i8,  3 },
    { ISD::SIGN_EXTEND, MVT::v2i64, MVT::v2i16, 2 },
    { ISD::ZERO_EXTEND, MVT::v2i64, MVT::v2i16, 2 },
    { ISD::SIGN_EXTEND, MVT::v4i64, MVT::v4i16, 3 },
    { ISD::ZERO_EXTEND, MVT::v4i64, MVT::v4i16, 3 },
    { ISD::SIGN_EXTEND, MVT::v8i32, MVT::v8i8, 3 },
    { ISD::ZERO_EXTEND, MVT::v8i32, MVT::v8i8, 3 },
    { ISD::SIGN_EXTEND, MVT::v8i64, MVT::v8i8, 7 },
    { ISD::ZERO_EXTEND, MVT::v8i64, MVT::v8i8, 7 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 749-752
```cpp
    { ISD::SIGN_EXTEND, MVT::v8i64, MVT::v8i16, 6 },
    { ISD::ZERO_EXTEND, MVT::v8i64, MVT::v8i16, 6 },
    { ISD::SIGN_EXTEND, MVT::v16i32, MVT::v16i8, 6 },
    { ISD::ZERO_EXTEND, MVT::v16i32, MVT::v16i8, 6 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 754-756
```cpp
    // Operations that we legalize using splitting.
    { ISD::TRUNCATE,    MVT::v16i8, MVT::v16i32, 6 },
    { ISD::TRUNCATE,    MVT::v8i8, MVT::v8i32, 3 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 758-760
```cpp
    // Vector float <-> i32 conversions.
    { ISD::SINT_TO_FP,  MVT::v4f32, MVT::v4i32, 1 },
    { ISD::UINT_TO_FP,  MVT::v4f32, MVT::v4i32, 1 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 762-776
```cpp
    { ISD::SINT_TO_FP,  MVT::v2f32, MVT::v2i8, 3 },
    { ISD::UINT_TO_FP,  MVT::v2f32, MVT::v2i8, 3 },
    { ISD::SINT_TO_FP,  MVT::v2f32, MVT::v2i16, 2 },
    { ISD::UINT_TO_FP,  MVT::v2f32, MVT::v2i16, 2 },
    { ISD::SINT_TO_FP,  MVT::v2f32, MVT::v2i32, 1 },
    { ISD::UINT_TO_FP,  MVT::v2f32, MVT::v2i32, 1 },
    { ISD::SINT_TO_FP,  MVT::v4f32, MVT::v4i1, 3 },
    { ISD::UINT_TO_FP,  MVT::v4f32, MVT::v4i1, 3 },
    { ISD::SINT_TO_FP,  MVT::v4f32, MVT::v4i8, 3 },
    { ISD::UINT_TO_FP,  MVT::v4f32, MVT::v4i8, 3 },
    { ISD::SINT_TO_FP,  MVT::v4f32, MVT::v4i16, 2 },
    { ISD::UINT_TO_FP,  MVT::v4f32, MVT::v4i16, 2 },
    { ISD::SINT_TO_FP,  MVT::v8f32, MVT::v8i16, 4 },
    { ISD::UINT_TO_FP,  MVT::v8f32, MVT::v8i16, 4 },
    { ISD::SINT_TO_FP,  MVT::v8f32, MVT::v8i32, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 777-781
```cpp
    { ISD::UINT_TO_FP,  MVT::v8f32, MVT::v8i32, 2 },
    { ISD::SINT_TO_FP,  MVT::v16f32, MVT::v16i16, 8 },
    { ISD::UINT_TO_FP,  MVT::v16f32, MVT::v16i16, 8 },
    { ISD::SINT_TO_FP,  MVT::v16f32, MVT::v16i32, 4 },
    { ISD::UINT_TO_FP,  MVT::v16f32, MVT::v16i32, 4 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 783-788
```cpp
    { ISD::FP_TO_SINT,  MVT::v4i32, MVT::v4f32, 1 },
    { ISD::FP_TO_UINT,  MVT::v4i32, MVT::v4f32, 1 },
    { ISD::FP_TO_SINT,  MVT::v4i8, MVT::v4f32, 3 },
    { ISD::FP_TO_UINT,  MVT::v4i8, MVT::v4f32, 3 },
    { ISD::FP_TO_SINT,  MVT::v4i16, MVT::v4f32, 2 },
    { ISD::FP_TO_UINT,  MVT::v4i16, MVT::v4f32, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 790-792
```cpp
    // Vector double <-> i32 conversions.
    { ISD::SINT_TO_FP,  MVT::v2f64, MVT::v2i32, 2 },
    { ISD::UINT_TO_FP,  MVT::v2f64, MVT::v2i32, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 794-799
```cpp
    { ISD::SINT_TO_FP,  MVT::v2f64, MVT::v2i8, 4 },
    { ISD::UINT_TO_FP,  MVT::v2f64, MVT::v2i8, 4 },
    { ISD::SINT_TO_FP,  MVT::v2f64, MVT::v2i16, 3 },
    { ISD::UINT_TO_FP,  MVT::v2f64, MVT::v2i16, 3 },
    { ISD::SINT_TO_FP,  MVT::v2f64, MVT::v2i32, 2 },
    { ISD::UINT_TO_FP,  MVT::v2f64, MVT::v2i32, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 801-807
```cpp
    { ISD::FP_TO_SINT,  MVT::v2i32, MVT::v2f64, 2 },
    { ISD::FP_TO_UINT,  MVT::v2i32, MVT::v2f64, 2 },
    { ISD::FP_TO_SINT,  MVT::v8i16, MVT::v8f32, 4 },
    { ISD::FP_TO_UINT,  MVT::v8i16, MVT::v8f32, 4 },
    { ISD::FP_TO_SINT,  MVT::v16i16, MVT::v16f32, 8 },
    { ISD::FP_TO_UINT,  MVT::v16i16, MVT::v16f32, 8 }
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 809-814
```cpp
  if (SrcTy.isVector() && ST->hasNEON()) {
    if (const auto *Entry = ConvertCostTableLookup(NEONVectorConversionTbl, ISD,
                                                   DstTy.getSimpleVT(),
                                                   SrcTy.getSimpleVT()))
      return AdjustCost(Entry->Cost);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 816-830
```cpp
  // Scalar float to integer conversions.
  static const TypeConversionCostTblEntry NEONFloatConversionTbl[] = {
    { ISD::FP_TO_SINT,  MVT::i1, MVT::f32, 2 },
    { ISD::FP_TO_UINT,  MVT::i1, MVT::f32, 2 },
    { ISD::FP_TO_SINT,  MVT::i1, MVT::f64, 2 },
    { ISD::FP_TO_UINT,  MVT::i1, MVT::f64, 2 },
    { ISD::FP_TO_SINT,  MVT::i8, MVT::f32, 2 },
    { ISD::FP_TO_UINT,  MVT::i8, MVT::f32, 2 },
    { ISD::FP_TO_SINT,  MVT::i8, MVT::f64, 2 },
    { ISD::FP_TO_UINT,  MVT::i8, MVT::f64, 2 },
    { ISD::FP_TO_SINT,  MVT::i16, MVT::f32, 2 },
    { ISD::FP_TO_UINT,  MVT::i16, MVT::f32, 2 },
    { ISD::FP_TO_SINT,  MVT::i16, MVT::f64, 2 },
    { ISD::FP_TO_UINT,  MVT::i16, MVT::f64, 2 },
    { ISD::FP_TO_SINT,  MVT::i32, MVT::f32, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 831-844
```cpp
    { ISD::FP_TO_UINT,  MVT::i32, MVT::f32, 2 },
    { ISD::FP_TO_SINT,  MVT::i32, MVT::f64, 2 },
    { ISD::FP_TO_UINT,  MVT::i32, MVT::f64, 2 },
    { ISD::FP_TO_SINT,  MVT::i64, MVT::f32, 10 },
    { ISD::FP_TO_UINT,  MVT::i64, MVT::f32, 10 },
    { ISD::FP_TO_SINT,  MVT::i64, MVT::f64, 10 },
    { ISD::FP_TO_UINT,  MVT::i64, MVT::f64, 10 }
  };
  if (SrcTy.isFloatingPoint() && ST->hasNEON()) {
    if (const auto *Entry = ConvertCostTableLookup(NEONFloatConversionTbl, ISD,
                                                   DstTy.getSimpleVT(),
                                                   SrcTy.getSimpleVT()))
      return AdjustCost(Entry->Cost);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 846-860
```cpp
  // Scalar integer to float conversions.
  static const TypeConversionCostTblEntry NEONIntegerConversionTbl[] = {
    { ISD::SINT_TO_FP,  MVT::f32, MVT::i1, 2 },
    { ISD::UINT_TO_FP,  MVT::f32, MVT::i1, 2 },
    { ISD::SINT_TO_FP,  MVT::f64, MVT::i1, 2 },
    { ISD::UINT_TO_FP,  MVT::f64, MVT::i1, 2 },
    { ISD::SINT_TO_FP,  MVT::f32, MVT::i8, 2 },
    { ISD::UINT_TO_FP,  MVT::f32, MVT::i8, 2 },
    { ISD::SINT_TO_FP,  MVT::f64, MVT::i8, 2 },
    { ISD::UINT_TO_FP,  MVT::f64, MVT::i8, 2 },
    { ISD::SINT_TO_FP,  MVT::f32, MVT::i16, 2 },
    { ISD::UINT_TO_FP,  MVT::f32, MVT::i16, 2 },
    { ISD::SINT_TO_FP,  MVT::f64, MVT::i16, 2 },
    { ISD::UINT_TO_FP,  MVT::f64, MVT::i16, 2 },
    { ISD::SINT_TO_FP,  MVT::f32, MVT::i32, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 861-868
```cpp
    { ISD::UINT_TO_FP,  MVT::f32, MVT::i32, 2 },
    { ISD::SINT_TO_FP,  MVT::f64, MVT::i32, 2 },
    { ISD::UINT_TO_FP,  MVT::f64, MVT::i32, 2 },
    { ISD::SINT_TO_FP,  MVT::f32, MVT::i64, 10 },
    { ISD::UINT_TO_FP,  MVT::f32, MVT::i64, 10 },
    { ISD::SINT_TO_FP,  MVT::f64, MVT::i64, 10 },
    { ISD::UINT_TO_FP,  MVT::f64, MVT::i64, 10 }
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 870-875
```cpp
  if (SrcTy.isInteger() && ST->hasNEON()) {
    if (const auto *Entry = ConvertCostTableLookup(NEONIntegerConversionTbl,
                                                   ISD, DstTy.getSimpleVT(),
                                                   SrcTy.getSimpleVT()))
      return AdjustCost(Entry->Cost);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 877-891
```cpp
  // MVE extend costs, taken from codegen tests. i8->i16 or i16->i32 is one
  // instruction, i8->i32 is two. i64 zexts are an VAND with a constant, sext
  // are linearised so take more.
  static const TypeConversionCostTblEntry MVEVectorConversionTbl[] = {
    { ISD::SIGN_EXTEND, MVT::v8i16, MVT::v8i8, 1 },
    { ISD::ZERO_EXTEND, MVT::v8i16, MVT::v8i8, 1 },
    { ISD::SIGN_EXTEND, MVT::v4i32, MVT::v4i8, 2 },
    { ISD::ZERO_EXTEND, MVT::v4i32, MVT::v4i8, 2 },
    { ISD::SIGN_EXTEND, MVT::v2i64, MVT::v2i8, 10 },
    { ISD::ZERO_EXTEND, MVT::v2i64, MVT::v2i8, 2 },
    { ISD::SIGN_EXTEND, MVT::v4i32, MVT::v4i16, 1 },
    { ISD::ZERO_EXTEND, MVT::v4i32, MVT::v4i16, 1 },
    { ISD::SIGN_EXTEND, MVT::v2i64, MVT::v2i16, 10 },
    { ISD::ZERO_EXTEND, MVT::v2i64, MVT::v2i16, 2 },
    { ISD::SIGN_EXTEND, MVT::v2i64, MVT::v2i32, 8 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 892-893
```cpp
    { ISD::ZERO_EXTEND, MVT::v2i64, MVT::v2i32, 2 },
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 895-900
```cpp
  if (SrcTy.isVector() && ST->hasMVEIntegerOps()) {
    if (const auto *Entry = ConvertCostTableLookup(MVEVectorConversionTbl,
                                                   ISD, DstTy.getSimpleVT(),
                                                   SrcTy.getSimpleVT()))
      return Entry->Cost * ST->getMVEVectorCostFactor(CostKind);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 902-910
```cpp
  if (ISD == ISD::FP_ROUND || ISD == ISD::FP_EXTEND) {
    // As general rule, fp converts that were not matched above are scalarized
    // and cost 1 vcvt for each lane, so long as the instruction is available.
    // If not it will become a series of function calls.
    const InstructionCost CallCost =
        getCallInstrCost(nullptr, Dst, {Src}, CostKind);
    int Lanes = 1;
    if (SrcTy.isFixedLengthVector())
      Lanes = SrcTy.getVectorNumElements();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 912-916
```cpp
    if (IsLegalFPType(SrcTy) && IsLegalFPType(DstTy))
      return Lanes;
    else
      return Lanes * CallCost;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 918-928
```cpp
  if (ISD == ISD::TRUNCATE && ST->hasMVEIntegerOps() &&
      SrcTy.isFixedLengthVector()) {
    // Treat a truncate with larger than legal source (128bits for MVE) as
    // expensive, 2 instructions per lane.
    if ((SrcTy.getScalarType() == MVT::i8 ||
         SrcTy.getScalarType() == MVT::i16 ||
         SrcTy.getScalarType() == MVT::i32) &&
        SrcTy.getSizeInBits() > 128 &&
        SrcTy.getSizeInBits() > DstTy.getSizeInBits())
      return SrcTy.getVectorNumElements() * 2;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 930-933
```cpp
  // Scalar integer conversion costs.
  static const TypeConversionCostTblEntry ARMIntegerConversionTbl[] = {
    // i16 -> i64 requires two dependent operations.
    { ISD::SIGN_EXTEND, MVT::i64, MVT::i16, 2 },
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 935-940
```cpp
    // Truncates on i64 are assumed to be free.
    { ISD::TRUNCATE,    MVT::i32, MVT::i64, 0 },
    { ISD::TRUNCATE,    MVT::i16, MVT::i64, 0 },
    { ISD::TRUNCATE,    MVT::i8,  MVT::i64, 0 },
    { ISD::TRUNCATE,    MVT::i1,  MVT::i64, 0 }
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 942-947
```cpp
  if (SrcTy.isInteger()) {
    if (const auto *Entry = ConvertCostTableLookup(ARMIntegerConversionTbl, ISD,
                                                   DstTy.getSimpleVT(),
                                                   SrcTy.getSimpleVT()))
      return AdjustCost(Entry->Cost);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 949-954
```cpp
  int BaseCost = ST->hasMVEIntegerOps() && Src->isVectorTy()
                     ? ST->getMVEVectorCostFactor(CostKind)
                     : 1;
  return AdjustCost(
      BaseCost * BaseT::getCastInstrCost(Opcode, Dst, Src, CCH, CostKind, I));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 956-963
```cpp
InstructionCost ARMTTIImpl::getVectorInstrCost(
    unsigned Opcode, Type *ValTy, TTI::TargetCostKind CostKind, unsigned Index,
    const Value *Op0, const Value *Op1, TTI::VectorInstrContext VIC) const {
  // Penalize inserting into an D-subregister. We end up with a three times
  // lower estimated throughput on swift.
  if (ST->hasSlowLoadDSubregister() && Opcode == Instruction::InsertElement &&
      ValTy->isVectorTy() && ValTy->getScalarSizeInBits() <= 32)
    return 3;
```
- EN: Implements `ARMTTIImpl::getVectorInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getVectorInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 965-970
```cpp
  if (ST->hasNEON() && (Opcode == Instruction::InsertElement ||
                        Opcode == Instruction::ExtractElement)) {
    // Cross-class copies are expensive on many microarchitectures,
    // so assume they are expensive by default.
    if (cast<VectorType>(ValTy)->getElementType()->isIntegerTy())
      return 3;
```
- EN: Declares `copies`, packaging target-specific state and APIs around `ARMTargetTransformInfo`.
- CN: 这里声明 `copies`，把与 `ARMTargetTransformInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 972-980
```cpp
    // Even if it's not a cross class copy, this likely leads to mixing
    // of NEON and VFP code and should be therefore penalized.
    if (ValTy->isVectorTy() &&
        ValTy->getScalarSizeInBits() <= 32)
      return std::max<InstructionCost>(
          BaseT::getVectorInstrCost(Opcode, ValTy, CostKind, Index, Op0, Op1,
                                    VIC),
          2U);
  }
```
- EN: Declares `copy`, packaging target-specific state and APIs around `ARMTargetTransformInfo`.
- CN: 这里声明 `copy`，把与 `ARMTargetTransformInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 982-990
```cpp
  if (ST->hasMVEIntegerOps() && (Opcode == Instruction::InsertElement ||
                                 Opcode == Instruction::ExtractElement)) {
    // Integer cross-lane moves are more expensive than float, which can
    // sometimes just be vmovs. Integer involve being passes to GPR registers,
    // causing more of a delay.
    std::pair<InstructionCost, MVT> LT =
        getTypeLegalizationCost(ValTy->getScalarType());
    return LT.first * (ValTy->getScalarType()->isIntegerTy() ? 4 : 1);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 992-994
```cpp
  return BaseT::getVectorInstrCost(Opcode, ValTy, CostKind, Index, Op0, Op1,
                                   VIC);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 996-1000
```cpp
InstructionCost ARMTTIImpl::getCmpSelInstrCost(
    unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
    TTI::TargetCostKind CostKind, TTI::OperandValueInfo Op1Info,
    TTI::OperandValueInfo Op2Info, const Instruction *I) const {
  int ISD = TLI->InstructionOpcodeToISD(Opcode);
```
- EN: Implements `ARMTTIImpl::getCmpSelInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getCmpSelInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1002-1007
```cpp
  // Thumb scalar code size cost for select.
  if (CostKind == TTI::TCK_CodeSize && ISD == ISD::SELECT &&
      ST->isThumb() && !ValTy->isVectorTy()) {
    // Assume expensive structs.
    if (TLI->getValueType(DL, ValTy, true) == MVT::Other)
      return TTI::TCC_Expensive;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1009-1013
```cpp
    // Select costs can vary because they:
    // - may require one or more conditional mov (including an IT),
    // - can't operate directly on immediates,
    // - require live flags, which we can't copy around easily.
    InstructionCost Cost = getTypeLegalizationCost(ValTy).first;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1015-1016
```cpp
    // Possible IT instruction for Thumb2, or more for Thumb1.
    ++Cost;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1018-1021
```cpp
    // i1 values may need rematerialising by using mov immediates and/or
    // flag setting instructions.
    if (ValTy->isIntegerTy(1))
      ++Cost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1023-1024
```cpp
    return Cost;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1026-1040
```cpp
  // If this is a vector min/max/abs, use the cost of that intrinsic directly
  // instead. Hopefully when min/max intrinsics are more prevalent this code
  // will not be needed.
  const Instruction *Sel = I;
  if ((Opcode == Instruction::ICmp || Opcode == Instruction::FCmp) && Sel &&
      Sel->hasOneUse())
    Sel = cast<Instruction>(Sel->user_back());
  if (Sel && ValTy->isVectorTy() &&
      (ValTy->isIntOrIntVectorTy() || ValTy->isFPOrFPVectorTy())) {
    const Value *LHS, *RHS;
    SelectPatternFlavor SPF = matchSelectPattern(Sel, LHS, RHS).Flavor;
    unsigned IID = 0;
    switch (SPF) {
    case SPF_ABS:
      IID = Intrinsic::abs;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1041-1055
```cpp
      break;
    case SPF_SMIN:
      IID = Intrinsic::smin;
      break;
    case SPF_SMAX:
      IID = Intrinsic::smax;
      break;
    case SPF_UMIN:
      IID = Intrinsic::umin;
      break;
    case SPF_UMAX:
      IID = Intrinsic::umax;
      break;
    case SPF_FMINNUM:
      IID = Intrinsic::minnum;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1056-1070
```cpp
      break;
    case SPF_FMAXNUM:
      IID = Intrinsic::maxnum;
      break;
    default:
      break;
    }
    if (IID) {
      // The ICmp is free, the select gets the cost of the min/max/etc
      if (Sel != I)
        return 0;
      IntrinsicCostAttributes CostAttrs(IID, ValTy, {ValTy, ValTy});
      return getIntrinsicInstrCost(CostAttrs, CostKind);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1072-1079
```cpp
  // On NEON a vector select gets lowered to vbsl.
  if (ST->hasNEON() && ValTy->isVectorTy() && ISD == ISD::SELECT && CondTy) {
    // Lowering of some vector selects is currently far from perfect.
    static const TypeConversionCostTblEntry NEONVectorSelectTbl[] = {
      { ISD::SELECT, MVT::v4i1, MVT::v4i64, 4*4 + 1*2 + 1 },
      { ISD::SELECT, MVT::v8i1, MVT::v8i64, 50 },
      { ISD::SELECT, MVT::v16i1, MVT::v16i64, 100 }
    };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1081-1088
```cpp
    EVT SelCondTy = TLI->getValueType(DL, CondTy);
    EVT SelValTy = TLI->getValueType(DL, ValTy);
    if (SelCondTy.isSimple() && SelValTy.isSimple()) {
      if (const auto *Entry = ConvertCostTableLookup(NEONVectorSelectTbl, ISD,
                                                     SelCondTy.getSimpleVT(),
                                                     SelValTy.getSimpleVT()))
        return Entry->Cost;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1090-1092
```cpp
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(ValTy);
    return LT.first;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1094-1100
```cpp
  if (ST->hasMVEIntegerOps() && ValTy->isVectorTy() &&
      (Opcode == Instruction::ICmp || Opcode == Instruction::FCmp) &&
      cast<FixedVectorType>(ValTy)->getNumElements() > 1) {
    FixedVectorType *VecValTy = cast<FixedVectorType>(ValTy);
    FixedVectorType *VecCondTy = dyn_cast_or_null<FixedVectorType>(CondTy);
    if (!VecCondTy)
      VecCondTy = cast<FixedVectorType>(CmpInst::makeCmpResultType(VecValTy));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1102-1114
```cpp
    // If we don't have mve.fp any fp operations will need to be scalarized.
    if (Opcode == Instruction::FCmp && !ST->hasMVEFloatOps()) {
      // One scalaization insert, one scalarization extract and the cost of the
      // fcmps.
      return BaseT::getScalarizationOverhead(VecValTy, /*Insert*/ false,
                                             /*Extract*/ true, CostKind) +
             BaseT::getScalarizationOverhead(VecCondTy, /*Insert*/ true,
                                             /*Extract*/ false, CostKind) +
             VecValTy->getNumElements() *
                 getCmpSelInstrCost(Opcode, ValTy->getScalarType(),
                                    VecCondTy->getScalarType(), VecPred,
                                    CostKind, Op1Info, Op2Info, I);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1116-1130
```cpp
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(ValTy);
    int BaseCost = ST->getMVEVectorCostFactor(CostKind);
    // There are two types - the input that specifies the type of the compare
    // and the output vXi1 type. Because we don't know how the output will be
    // split, we may need an expensive shuffle to get two in sync. This has the
    // effect of making larger than legal compares (v8i32 for example)
    // expensive.
    if (LT.second.isVector() && LT.second.getVectorNumElements() > 2) {
      if (LT.first > 1)
        return LT.first * BaseCost +
               BaseT::getScalarizationOverhead(VecCondTy, /*Insert*/ true,
                                               /*Extract*/ false, CostKind);
      return BaseCost;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1132-1136
```cpp
  // Default to cheap (throughput/size of 1 instruction) but adjust throughput
  // for "multiple beats" potentially needed by MVE instructions.
  int BaseCost = 1;
  if (ST->hasMVEIntegerOps() && ValTy->isVectorTy())
    BaseCost = ST->getMVEVectorCostFactor(CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1138-1140
```cpp
  return BaseCost * BaseT::getCmpSelInstrCost(Opcode, ValTy, CondTy, VecPred,
                                              CostKind, Op1Info, Op2Info, I);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1142-1151
```cpp
InstructionCost
ARMTTIImpl::getAddressComputationCost(Type *PtrTy, ScalarEvolution *SE,
                                      const SCEV *Ptr,
                                      TTI::TargetCostKind CostKind) const {
  // Address computations in vectorized code with non-consecutive addresses will
  // likely result in more instructions compared to scalar code where the
  // computation can more often be merged into the index mode. The resulting
  // extra micro-ops can significantly decrease throughput.
  unsigned NumVectorInstToHideOverhead = 10;
  int MaxMergeDistance = 64;
```
- EN: Implements `ARMTTIImpl::getAddressComputationCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getAddressComputationCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1153-1156
```cpp
  if (ST->hasNEON()) {
    if (PtrTy->isVectorTy() && SE &&
        !BaseT::isConstantStridedAccessLessThan(SE, Ptr, MaxMergeDistance + 1))
      return NumVectorInstToHideOverhead;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1158-1163
```cpp
    // In many cases the address computation is not merged into the instruction
    // addressing mode.
    return 1;
  }
  return BaseT::getAddressComputationCost(PtrTy, SE, Ptr, CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1165-1179
```cpp
bool ARMTTIImpl::isProfitableLSRChainElement(Instruction *I) const {
  if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(I)) {
    // If a VCTP is part of a chain, it's already profitable and shouldn't be
    // optimized, else LSR may block tail-predication.
    switch (II->getIntrinsicID()) {
    case Intrinsic::arm_mve_vctp8:
    case Intrinsic::arm_mve_vctp16:
    case Intrinsic::arm_mve_vctp32:
    case Intrinsic::arm_mve_vctp64:
      return true;
    default:
      break;
    }
  }
  return false;
```
- EN: Implements `ARMTTIImpl::isProfitableLSRChainElement`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::isProfitableLSRChainElement`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1180-1180
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1182-1186
```cpp
bool ARMTTIImpl::isLegalMaskedLoad(Type *DataTy, Align Alignment,
                                   unsigned /*AddressSpace*/,
                                   TTI::MaskKind /*MaskKind*/) const {
  if (!EnableMaskedLoadStores || !ST->hasMVEIntegerOps())
    return false;
```
- EN: Implements `ARMTTIImpl::isLegalMaskedLoad`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::isLegalMaskedLoad`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1188-1191
```cpp
  if (auto *VecTy = dyn_cast<FixedVectorType>(DataTy)) {
    // Don't support v2i1 yet.
    if (VecTy->getNumElements() == 2)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1193-1197
```cpp
    // We don't support extending fp types.
     unsigned VecWidth = DataTy->getPrimitiveSizeInBits();
    if (VecWidth != 128 && VecTy->getElementType()->isFloatingPointTy())
      return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1199-1202
```cpp
  unsigned EltWidth = DataTy->getScalarSizeInBits();
  return (EltWidth == 32 && Alignment >= 4) ||
         (EltWidth == 16 && Alignment >= 2) || (EltWidth == 8);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1204-1206
```cpp
bool ARMTTIImpl::isLegalMaskedGather(Type *Ty, Align Alignment) const {
  if (!EnableMaskedGatherScatters || !ST->hasMVEIntegerOps())
    return false;
```
- EN: Implements `ARMTTIImpl::isLegalMaskedGather`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::isLegalMaskedGather`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1208-1211
```cpp
  unsigned EltWidth = Ty->getScalarSizeInBits();
  return ((EltWidth == 32 && Alignment >= 4) ||
          (EltWidth == 16 && Alignment >= 2) || EltWidth == 8);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1213-1220
```cpp
/// Given a memcpy/memset/memmove instruction, return the number of memory
/// operations performed, via querying findOptimalMemOpLowering. Returns -1 if a
/// call is used.
int ARMTTIImpl::getNumMemOps(const IntrinsicInst *I) const {
  MemOp MOp;
  unsigned DstAddrSpace = ~0u;
  unsigned SrcAddrSpace = ~0u;
  const Function *F = I->getParent()->getParent();
```
- EN: Implements `ARMTTIImpl::getNumMemOps`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getNumMemOps`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1222-1226
```cpp
  if (const auto *MC = dyn_cast<MemTransferInst>(I)) {
    ConstantInt *C = dyn_cast<ConstantInt>(MC->getLength());
    // If 'size' is not a constant, a library call will be generated.
    if (!C)
      return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1228-1230
```cpp
    const unsigned Size = C->getValue().getZExtValue();
    const Align DstAlign = MC->getDestAlign().valueOrOne();
    const Align SrcAlign = MC->getSourceAlign().valueOrOne();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1232-1241
```cpp
    MOp = MemOp::Copy(Size, /*DstAlignCanChange*/ false, DstAlign, SrcAlign,
                      /*IsVolatile*/ false);
    DstAddrSpace = MC->getDestAddressSpace();
    SrcAddrSpace = MC->getSourceAddressSpace();
  }
  else if (const auto *MS = dyn_cast<MemSetInst>(I)) {
    ConstantInt *C = dyn_cast<ConstantInt>(MS->getLength());
    // If 'size' is not a constant, a library call will be generated.
    if (!C)
      return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1243-1244
```cpp
    const unsigned Size = C->getValue().getZExtValue();
    const Align DstAlign = MS->getDestAlign().valueOrOne();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1246-1251
```cpp
    MOp = MemOp::Set(Size, /*DstAlignCanChange*/ false, DstAlign,
                     /*IsZeroMemset*/ false, /*IsVolatile*/ false);
    DstAddrSpace = MS->getDestAddressSpace();
  }
  else
    llvm_unreachable("Expected a memcpy/move or memset!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1253-1267
```cpp
  unsigned Limit, Factor = 2;
  switch(I->getIntrinsicID()) {
    case Intrinsic::memcpy:
      Limit = TLI->getMaxStoresPerMemcpy(F->hasMinSize());
      break;
    case Intrinsic::memmove:
      Limit = TLI->getMaxStoresPerMemmove(F->hasMinSize());
      break;
    case Intrinsic::memset:
      Limit = TLI->getMaxStoresPerMemset(F->hasMinSize());
      Factor = 1;
      break;
    default:
      llvm_unreachable("Expected a memcpy/move or memset!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1269-1277
```cpp
  // MemOps will be poplulated with a list of data types that needs to be
  // loaded and stored. That's why we multiply the number of elements by 2 to
  // get the cost for this memcpy.
  std::vector<EVT> MemOps;
  LLVMContext &C = F->getContext();
  if (getTLI()->findOptimalMemOpLowering(C, MemOps, Limit, MOp, DstAddrSpace,
                                         SrcAddrSpace, F->getAttributes(),
                                         nullptr))
    return MemOps.size() * Factor;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1279-1281
```cpp
  // If we can't find an optimal memop lowering, return the default cost
  return -1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1283-1284
```cpp
InstructionCost ARMTTIImpl::getMemcpyCost(const Instruction *I) const {
  int NumOps = getNumMemOps(cast<IntrinsicInst>(I));
```
- EN: Implements `ARMTTIImpl::getMemcpyCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getMemcpyCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1286-1291
```cpp
  // To model the cost of a library call, we assume 1 for the call, and
  // 3 for the argument setup.
  if (NumOps == -1)
    return 4;
  return NumOps;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1293-1304
```cpp
InstructionCost ARMTTIImpl::getShuffleCost(TTI::ShuffleKind Kind,
                                           VectorType *DstTy, VectorType *SrcTy,
                                           ArrayRef<int> Mask,
                                           TTI::TargetCostKind CostKind,
                                           int Index, VectorType *SubTp,
                                           ArrayRef<const Value *> Args,
                                           const Instruction *CxtI) const {
  assert((Mask.empty() || DstTy->isScalableTy() ||
          Mask.size() == DstTy->getElementCount().getKnownMinValue()) &&
         "Expected the Mask to match the return size if given");
  assert(SrcTy->getScalarType() == DstTy->getScalarType() &&
         "Expected the same scalar types");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1306-1320
```cpp
  Kind = improveShuffleKindFromMask(Kind, Mask, SrcTy, Index, SubTp);
  // Treat extractsubvector as single op permutation.
  bool IsExtractSubvector = Kind == TTI::SK_ExtractSubvector;
  if (IsExtractSubvector)
    Kind = TTI::SK_PermuteSingleSrc;
  if (ST->hasNEON()) {
    if (Kind == TTI::SK_Broadcast) {
      static const CostTblEntry NEONDupTbl[] = {
          // VDUP handles these cases.
          {ISD::VECTOR_SHUFFLE, MVT::v2i32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2f32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2i64, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2f64, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v4i16, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v8i8, 1},
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1322-1325
```cpp
          {ISD::VECTOR_SHUFFLE, MVT::v4i32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v4f32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v8i16, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v16i8, 1}};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1327-1341
```cpp
      std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(SrcTy);
      if (const auto *Entry =
              CostTableLookup(NEONDupTbl, ISD::VECTOR_SHUFFLE, LT.second))
        return LT.first * Entry->Cost;
    }
    if (Kind == TTI::SK_Reverse) {
      static const CostTblEntry NEONShuffleTbl[] = {
          // Reverse shuffle cost one instruction if we are shuffling within a
          // double word (vrev) or two if we shuffle a quad word (vrev, vext).
          {ISD::VECTOR_SHUFFLE, MVT::v2i32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2f32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2i64, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2f64, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v4i16, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v8i8, 1},
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1343-1346
```cpp
          {ISD::VECTOR_SHUFFLE, MVT::v4i32, 2},
          {ISD::VECTOR_SHUFFLE, MVT::v4f32, 2},
          {ISD::VECTOR_SHUFFLE, MVT::v8i16, 2},
          {ISD::VECTOR_SHUFFLE, MVT::v16i8, 2}};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1348-1357
```cpp
      std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(SrcTy);
      if (const auto *Entry =
              CostTableLookup(NEONShuffleTbl, ISD::VECTOR_SHUFFLE, LT.second))
        return LT.first * Entry->Cost;
    }
    if (Kind == TTI::SK_Select) {
      static const CostTblEntry NEONSelShuffleTbl[] = {
          // Select shuffle cost table for ARM. Cost is the number of
          // instructions
          // required to create the shuffled vector.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1359-1362
```cpp
          {ISD::VECTOR_SHUFFLE, MVT::v2f32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2i64, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2f64, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v2i32, 1},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1364-1366
```cpp
          {ISD::VECTOR_SHUFFLE, MVT::v4i32, 2},
          {ISD::VECTOR_SHUFFLE, MVT::v4f32, 2},
          {ISD::VECTOR_SHUFFLE, MVT::v4i16, 2},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1368-1368
```cpp
          {ISD::VECTOR_SHUFFLE, MVT::v8i16, 16},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1370-1370
```cpp
          {ISD::VECTOR_SHUFFLE, MVT::v16i8, 32}};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1372-1386
```cpp
      std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(SrcTy);
      if (const auto *Entry = CostTableLookup(NEONSelShuffleTbl,
                                              ISD::VECTOR_SHUFFLE, LT.second))
        return LT.first * Entry->Cost;
    }
  }
  if (ST->hasMVEIntegerOps()) {
    if (Kind == TTI::SK_Broadcast) {
      static const CostTblEntry MVEDupTbl[] = {
          // VDUP handles these cases.
          {ISD::VECTOR_SHUFFLE, MVT::v4i32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v8i16, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v16i8, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v4f32, 1},
          {ISD::VECTOR_SHUFFLE, MVT::v8f16, 1}};
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1388-1392
```cpp
      std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(SrcTy);
      if (const auto *Entry = CostTableLookup(MVEDupTbl, ISD::VECTOR_SHUFFLE,
                                              LT.second))
        return LT.first * Entry->Cost * ST->getMVEVectorCostFactor(CostKind);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1394-1408
```cpp
    if (!Mask.empty()) {
      std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(SrcTy);
      // Check for LD2/LD4 instructions, which are represented in llvm IR as
      // deinterleaving-shuffle(load). The shuffle cost could potentially be
      // free, but we model it with a cost of LT.first so that LD2/LD4 have a
      // higher cost than just the load.
      if (Args.size() >= 1 && isa<LoadInst>(Args[0]) &&
          (LT.second.getScalarSizeInBits() == 8 ||
           LT.second.getScalarSizeInBits() == 16 ||
           LT.second.getScalarSizeInBits() == 32) &&
          LT.second.getSizeInBits() == 128 &&
          ((TLI->getMaxSupportedInterleaveFactor() >= 2 &&
            ShuffleVectorInst::isDeInterleaveMaskOfFactor(Mask, 2)) ||
           (TLI->getMaxSupportedInterleaveFactor() == 4 &&
            ShuffleVectorInst::isDeInterleaveMaskOfFactor(Mask, 4))))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1409-1410
```cpp
        return ST->getMVEVectorCostFactor(CostKind) *
               std::max<InstructionCost>(1, LT.first / 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1412-1426
```cpp
      // Check for ST2/ST4 instructions, which are represented in llvm IR as
      // store(interleaving-shuffle). The shuffle cost could potentially be
      // free, but we model it with a cost of LT.first so that ST2/ST4 have a
      // higher cost than just the store.
      if (CxtI && CxtI->hasOneUse() && isa<StoreInst>(*CxtI->user_begin()) &&
          (LT.second.getScalarSizeInBits() == 8 ||
           LT.second.getScalarSizeInBits() == 16 ||
           LT.second.getScalarSizeInBits() == 32) &&
          LT.second.getSizeInBits() == 128 &&
          ((TLI->getMaxSupportedInterleaveFactor() >= 2 &&
            ShuffleVectorInst::isInterleaveMask(
                Mask, 2, SrcTy->getElementCount().getKnownMinValue() * 2)) ||
           (TLI->getMaxSupportedInterleaveFactor() == 4 &&
            ShuffleVectorInst::isInterleaveMask(
                Mask, 4, SrcTy->getElementCount().getKnownMinValue() * 2))))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1427-1427
```cpp
        return ST->getMVEVectorCostFactor(CostKind) * LT.first;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1429-1435
```cpp
      if (LT.second.isVector() &&
          Mask.size() <= LT.second.getVectorNumElements() &&
          (isVREVMask(Mask, LT.second, 16) || isVREVMask(Mask, LT.second, 32) ||
           isVREVMask(Mask, LT.second, 64)))
        return ST->getMVEVectorCostFactor(CostKind) * LT.first;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1437-1445
```cpp
  // Restore optimal kind.
  if (IsExtractSubvector)
    Kind = TTI::SK_ExtractSubvector;
  int BaseCost = ST->hasMVEIntegerOps() && SrcTy->isVectorTy()
                     ? ST->getMVEVectorCostFactor(CostKind)
                     : 1;
  return BaseCost * BaseT::getShuffleCost(Kind, DstTy, SrcTy, Mask, CostKind,
                                          Index, SubTp);
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1447-1461
```cpp
InstructionCost ARMTTIImpl::getArithmeticInstrCost(
    unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
    TTI::OperandValueInfo Op1Info, TTI::OperandValueInfo Op2Info,
    ArrayRef<const Value *> Args, const Instruction *CxtI) const {
  int ISDOpcode = TLI->InstructionOpcodeToISD(Opcode);
  if (ST->isThumb() && CostKind == TTI::TCK_CodeSize && Ty->isIntegerTy(1)) {
    // Make operations on i1 relatively expensive as this often involves
    // combining predicates. AND and XOR should be easier to handle with IT
    // blocks.
    switch (ISDOpcode) {
    default:
      break;
    case ISD::AND:
    case ISD::XOR:
      return 2;
```
- EN: Implements `ARMTTIImpl::getArithmeticInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getArithmeticInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1462-1465
```cpp
    case ISD::OR:
      return 3;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1467-1467
```cpp
  std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(Ty);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1469-1483
```cpp
  if (ST->hasNEON()) {
    const unsigned FunctionCallDivCost = 20;
    const unsigned ReciprocalDivCost = 10;
    static const CostTblEntry CostTbl[] = {
      // Division.
      // These costs are somewhat random. Choose a cost of 20 to indicate that
      // vectorizing devision (added function call) is going to be very expensive.
      // Double registers types.
      { ISD::SDIV, MVT::v1i64, 1 * FunctionCallDivCost},
      { ISD::UDIV, MVT::v1i64, 1 * FunctionCallDivCost},
      { ISD::SREM, MVT::v1i64, 1 * FunctionCallDivCost},
      { ISD::UREM, MVT::v1i64, 1 * FunctionCallDivCost},
      { ISD::SDIV, MVT::v2i32, 2 * FunctionCallDivCost},
      { ISD::UDIV, MVT::v2i32, 2 * FunctionCallDivCost},
      { ISD::SREM, MVT::v2i32, 2 * FunctionCallDivCost},
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1484-1498
```cpp
      { ISD::UREM, MVT::v2i32, 2 * FunctionCallDivCost},
      { ISD::SDIV, MVT::v4i16,     ReciprocalDivCost},
      { ISD::UDIV, MVT::v4i16,     ReciprocalDivCost},
      { ISD::SREM, MVT::v4i16, 4 * FunctionCallDivCost},
      { ISD::UREM, MVT::v4i16, 4 * FunctionCallDivCost},
      { ISD::SDIV, MVT::v8i8,      ReciprocalDivCost},
      { ISD::UDIV, MVT::v8i8,      ReciprocalDivCost},
      { ISD::SREM, MVT::v8i8,  8 * FunctionCallDivCost},
      { ISD::UREM, MVT::v8i8,  8 * FunctionCallDivCost},
      // Quad register types.
      { ISD::SDIV, MVT::v2i64, 2 * FunctionCallDivCost},
      { ISD::UDIV, MVT::v2i64, 2 * FunctionCallDivCost},
      { ISD::SREM, MVT::v2i64, 2 * FunctionCallDivCost},
      { ISD::UREM, MVT::v2i64, 2 * FunctionCallDivCost},
      { ISD::SDIV, MVT::v4i32, 4 * FunctionCallDivCost},
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1499-1511
```cpp
      { ISD::UDIV, MVT::v4i32, 4 * FunctionCallDivCost},
      { ISD::SREM, MVT::v4i32, 4 * FunctionCallDivCost},
      { ISD::UREM, MVT::v4i32, 4 * FunctionCallDivCost},
      { ISD::SDIV, MVT::v8i16, 8 * FunctionCallDivCost},
      { ISD::UDIV, MVT::v8i16, 8 * FunctionCallDivCost},
      { ISD::SREM, MVT::v8i16, 8 * FunctionCallDivCost},
      { ISD::UREM, MVT::v8i16, 8 * FunctionCallDivCost},
      { ISD::SDIV, MVT::v16i8, 16 * FunctionCallDivCost},
      { ISD::UDIV, MVT::v16i8, 16 * FunctionCallDivCost},
      { ISD::SREM, MVT::v16i8, 16 * FunctionCallDivCost},
      { ISD::UREM, MVT::v16i8, 16 * FunctionCallDivCost},
      // Multiplication.
    };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1513-1514
```cpp
    if (const auto *Entry = CostTableLookup(CostTbl, ISDOpcode, LT.second))
      return LT.first * Entry->Cost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1516-1517
```cpp
    InstructionCost Cost = BaseT::getArithmeticInstrCost(
        Opcode, Ty, CostKind, Op1Info, Op2Info);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1519-1527
```cpp
    // This is somewhat of a hack. The problem that we are facing is that SROA
    // creates a sequence of shift, and, or instructions to construct values.
    // These sequences are recognized by the ISel and have zero-cost. Not so for
    // the vectorized code. Because we have support for v2i64 but not i64 those
    // sequences look particularly beneficial to vectorize.
    // To work around this we increase the cost of v2i64 operations to make them
    // seem less beneficial.
    if (LT.second == MVT::v2i64 && Op2Info.isUniform() && Op2Info.isConstant())
      Cost += 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1529-1530
```cpp
    return Cost;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1532-1536
```cpp
  // If this operation is a shift on arm/thumb2, it might well be folded into
  // the following instruction, hence having a cost of 0.
  auto LooksLikeAFreeShift = [&]() {
    if (ST->isThumb1Only() || Ty->isVectorTy())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1538-1541
```cpp
    if (!CxtI || !CxtI->hasOneUse() || !CxtI->isShift())
      return false;
    if (!Op2Info.isUniform() || !Op2Info.isConstant())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1543-1557
```cpp
    // Folded into a ADC/ADD/AND/BIC/CMP/EOR/MVN/ORR/ORN/RSB/SBC/SUB
    switch (cast<Instruction>(CxtI->user_back())->getOpcode()) {
    case Instruction::Add:
    case Instruction::Sub:
    case Instruction::And:
    case Instruction::Xor:
    case Instruction::Or:
    case Instruction::ICmp:
      return true;
    default:
      return false;
    }
  };
  if (LooksLikeAFreeShift())
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1559-1569
```cpp
  // When targets have both DSP and MVE we find that the
  // the compiler will attempt to vectorize as well as using
  // scalar (S/U)MLAL operations. This is in cases where we have
  // the pattern ext(mul(ext(i16), ext(i16))) we find
  // that codegen performs better when only using (S/U)MLAL scalar
  // ops instead of trying to mix vector ops with (S/U)MLAL ops. We therefore
  // check if a mul instruction is used in a (U/S)MLAL pattern.
  auto MulInDSPMLALPattern = [&](const Instruction *I, unsigned Opcode,
                                 Type *Ty) -> bool {
    if (!ST->hasDSP())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1571-1572
```cpp
    if (!I)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1574-1575
```cpp
    if (Opcode != Instruction::Mul)
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1577-1578
```cpp
    if (Ty->isVectorTy())
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1580-1589
```cpp
    auto ValueOpcodesEqual = [](const Value *LHS, const Value *RHS) -> bool {
      return cast<Instruction>(LHS)->getOpcode() ==
             cast<Instruction>(RHS)->getOpcode();
    };
    auto IsExtInst = [](const Value *V) -> bool {
      return isa<ZExtInst>(V) || isa<SExtInst>(V);
    };
    auto IsExtensionFromHalf = [](const Value *V) -> bool {
      return cast<Instruction>(V)->getOperand(0)->getType()->isIntegerTy(16);
    };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1591-1605
```cpp
    // We check the arguments of the instruction to see if they're extends
    auto *BinOp = dyn_cast<BinaryOperator>(I);
    if (!BinOp)
      return false;
    Value *Op0 = BinOp->getOperand(0);
    Value *Op1 = BinOp->getOperand(1);
    if (IsExtInst(Op0) && IsExtInst(Op1) && ValueOpcodesEqual(Op0, Op1)) {
      // We're interested in an ext of an i16
      if (!I->getType()->isIntegerTy(32) || !IsExtensionFromHalf(Op0) ||
          !IsExtensionFromHalf(Op1))
        return false;
      // We need to check if this result will be further extended to i64
      // and that all these uses are SExt
      for (auto *U : I->users())
        if (!IsExtInst(U))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1606-1608
```cpp
          return false;
      return true;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1610-1611
```cpp
    return false;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1613-1614
```cpp
  if (MulInDSPMLALPattern(CxtI, Opcode, Ty))
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1616-1620
```cpp
  // Default to cheap (throughput/size of 1 instruction) but adjust throughput
  // for "multiple beats" potentially needed by MVE instructions.
  int BaseCost = 1;
  if (ST->hasMVEIntegerOps() && Ty->isVectorTy())
    BaseCost = ST->getMVEVectorCostFactor(CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1622-1627
```cpp
  // The rest of this mostly follows what is done in
  // BaseT::getArithmeticInstrCost, without treating floats as more expensive
  // that scalars or increasing the costs for custom operations. The results is
  // also multiplied by the MVEVectorCostFactor where appropriate.
  if (TLI->isOperationLegalOrCustomOrPromote(ISDOpcode, LT.second))
    return LT.first * BaseCost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1629-1639
```cpp
  // Else this is expand, assume that we need to scalarize this op.
  if (auto *VTy = dyn_cast<FixedVectorType>(Ty)) {
    unsigned Num = VTy->getNumElements();
    InstructionCost Cost =
        getArithmeticInstrCost(Opcode, Ty->getScalarType(), CostKind);
    // Return the cost of multiple scalar invocation plus the cost of
    // inserting and extracting the values.
    SmallVector<Type *> Tys(Args.size(), Ty);
    return BaseT::getScalarizationOverhead(VTy, Args, Tys, CostKind) +
           Num * Cost;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1641-1642
```cpp
  return BaseCost;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1644-1652
```cpp
InstructionCost ARMTTIImpl::getMemoryOpCost(unsigned Opcode, Type *Src,
                                            Align Alignment,
                                            unsigned AddressSpace,
                                            TTI::TargetCostKind CostKind,
                                            TTI::OperandValueInfo OpInfo,
                                            const Instruction *I) const {
  // TODO: Handle other cost kinds.
  if (CostKind != TTI::TCK_RecipThroughput)
    return 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1654-1657
```cpp
  // Type legalization can't handle structs
  if (TLI->getValueType(DL, Src, true) == MVT::Other)
    return BaseT::getMemoryOpCost(Opcode, Src, Alignment, AddressSpace,
                                  CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1659-1665
```cpp
  if (ST->hasNEON() && Src->isVectorTy() && Alignment != Align(16) &&
      cast<VectorType>(Src)->getElementType()->isDoubleTy()) {
    // Unaligned loads/stores are extremely inefficient.
    // We need 4 uops for vst.1/vld.1 vs 1uop for vldr/vstr.
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(Src);
    return LT.first * 4;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1667-1681
```cpp
  // MVE can optimize a fpext(load(4xhalf)) using an extending integer load.
  // Same for stores.
  if (ST->hasMVEFloatOps() && isa<FixedVectorType>(Src) && I &&
      ((Opcode == Instruction::Load && I->hasOneUse() &&
        isa<FPExtInst>(*I->user_begin())) ||
       (Opcode == Instruction::Store && isa<FPTruncInst>(I->getOperand(0))))) {
    FixedVectorType *SrcVTy = cast<FixedVectorType>(Src);
    Type *DstTy =
        Opcode == Instruction::Load
            ? (*I->user_begin())->getType()
            : cast<Instruction>(I->getOperand(0))->getOperand(0)->getType();
    if (SrcVTy->getNumElements() == 4 && SrcVTy->getScalarType()->isHalfTy() &&
        DstTy->getScalarType()->isFloatTy())
      return ST->getMVEVectorCostFactor(CostKind);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1683-1688
```cpp
  int BaseCost = ST->hasMVEIntegerOps() && Src->isVectorTy()
                     ? ST->getMVEVectorCostFactor(CostKind)
                     : 1;
  return BaseCost * BaseT::getMemoryOpCost(Opcode, Src, Alignment, AddressSpace,
                                           CostKind, OpInfo, I);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1690-1702
```cpp
InstructionCost
ARMTTIImpl::getMemIntrinsicInstrCost(const MemIntrinsicCostAttributes &MICA,
                                     TTI::TargetCostKind CostKind) const {
  switch (MICA.getID()) {
  case Intrinsic::masked_scatter:
  case Intrinsic::masked_gather:
    return getGatherScatterOpCost(MICA, CostKind);
  case Intrinsic::masked_load:
  case Intrinsic::masked_store:
    return getMaskedMemoryOpCost(MICA, CostKind);
  }
  return BaseT::getMemIntrinsicInstrCost(MICA, CostKind);
}
```
- EN: Implements `ARMTTIImpl::getMemIntrinsicInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getMemIntrinsicInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1704-1718
```cpp
InstructionCost
ARMTTIImpl::getMaskedMemoryOpCost(const MemIntrinsicCostAttributes &MICA,
                                  TTI::TargetCostKind CostKind) const {
  unsigned IID = MICA.getID();
  Type *Src = MICA.getDataType();
  Align Alignment = MICA.getAlignment();
  unsigned AddressSpace = MICA.getAddressSpace();
  if (ST->hasMVEIntegerOps()) {
    if (IID == Intrinsic::masked_load &&
        isLegalMaskedLoad(Src, Alignment, AddressSpace))
      return ST->getMVEVectorCostFactor(CostKind);
    if (IID == Intrinsic::masked_store &&
        isLegalMaskedStore(Src, Alignment, AddressSpace))
      return ST->getMVEVectorCostFactor(CostKind);
  }
```
- EN: Implements `ARMTTIImpl::getMaskedMemoryOpCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getMaskedMemoryOpCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1719-1724
```cpp
  if (!isa<FixedVectorType>(Src))
    return BaseT::getMemIntrinsicInstrCost(MICA, CostKind);
  // Scalar cost, which is currently very high due to the efficiency of the
  // generated code.
  return cast<FixedVectorType>(Src)->getNumElements() * 8;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1726-1731
```cpp
InstructionCost ARMTTIImpl::getInterleavedMemoryOpCost(
    unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
    Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
    bool UseMaskForCond, bool UseMaskForGaps) const {
  assert(Factor >= 2 && "Invalid interleave factor");
  assert(isa<VectorType>(VecTy) && "Expect a vector type");
```
- EN: Implements `ARMTTIImpl::getInterleavedMemoryOpCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getInterleavedMemoryOpCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1733-1734
```cpp
  // vldN/vstN doesn't support vector types of i64/f64 element.
  bool EltIs64Bits = DL.getTypeSizeInBits(VecTy->getScalarType()) == 64;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1736-1740
```cpp
  if (Factor <= TLI->getMaxSupportedInterleaveFactor() && !EltIs64Bits &&
      !UseMaskForCond && !UseMaskForGaps) {
    unsigned NumElts = cast<FixedVectorType>(VecTy)->getNumElements();
    auto *SubVecTy =
        FixedVectorType::get(VecTy->getScalarType(), NumElts / Factor);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1742-1749
```cpp
    // vldN/vstN only support legal vector types of size 64 or 128 in bits.
    // Accesses having vector types that are a multiple of 128 bits can be
    // matched to more than one vldN/vstN instruction.
    int BaseCost =
        ST->hasMVEIntegerOps() ? ST->getMVEVectorCostFactor(CostKind) : 1;
    if (NumElts % Factor == 0 &&
        TLI->isLegalInterleavedAccessType(Factor, SubVecTy, Alignment, DL))
      return Factor * BaseCost * TLI->getNumInterleavedAccesses(SubVecTy, DL);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1751-1760
```cpp
    // Some smaller than legal interleaved patterns are cheap as we can make
    // use of the vmovn or vrev patterns to interleave a standard load. This is
    // true for v4i8, v8i8 and v4i16 at least (but not for v4f16 as it is
    // promoted differently). The cost of 2 here is then a load and vrev or
    // vmovn.
    if (ST->hasMVEIntegerOps() && Factor == 2 && NumElts / Factor > 2 &&
        VecTy->isIntOrIntVectorTy() &&
        DL.getTypeSizeInBits(SubVecTy).getFixedValue() <= 64)
      return 2 * BaseCost;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1762-1765
```cpp
  return BaseT::getInterleavedMemoryOpCost(Opcode, VecTy, Factor, Indices,
                                           Alignment, AddressSpace, CostKind,
                                           UseMaskForCond, UseMaskForGaps);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1767-1769
```cpp
InstructionCost
ARMTTIImpl::getGatherScatterOpCost(const MemIntrinsicCostAttributes &MICA,
                                   TTI::TargetCostKind CostKind) const {
```
- EN: Implements `ARMTTIImpl::getGatherScatterOpCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getGatherScatterOpCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1771-1775
```cpp
  Type *DataTy = MICA.getDataType();
  const Value *Ptr = MICA.getPointer();
  bool VariableMask = MICA.getVariableMask();
  Align Alignment = MICA.getAlignment();
  const Instruction *I = MICA.getInst();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1777-1779
```cpp
  using namespace PatternMatch;
  if (!ST->hasMVEIntegerOps() || !EnableMaskedGatherScatters)
    return BaseT::getMemIntrinsicInstrCost(MICA, CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1781-1782
```cpp
  assert(DataTy->isVectorTy() && "Can't do gather/scatters on scalar!");
  auto *VTy = cast<FixedVectorType>(DataTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1784-1784
```cpp
  // TODO: Splitting, once we do that.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1786-1788
```cpp
  unsigned NumElems = VTy->getNumElements();
  unsigned EltSize = VTy->getScalarSizeInBits();
  std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(DataTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1790-1804
```cpp
  // For now, it is assumed that for the MVE gather instructions the loads are
  // all effectively serialised. This means the cost is the scalar cost
  // multiplied by the number of elements being loaded. This is possibly very
  // conservative, but even so we still end up vectorising loops because the
  // cost per iteration for many loops is lower than for scalar loops.
  InstructionCost VectorCost =
      NumElems * LT.first * ST->getMVEVectorCostFactor(CostKind);
  // The scalarization cost should be a lot higher. We use the number of vector
  // elements plus the scalarization overhead. If masking is required then a lot
  // of little blocks will be needed and potentially a scalarized p0 mask,
  // greatly increasing the cost.
  InstructionCost ScalarCost =
      NumElems * LT.first + (VariableMask ? NumElems * 5 : 0) +
      BaseT::getScalarizationOverhead(VTy, /*Insert*/ true, /*Extract*/ false,
                                      CostKind) +
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1805-1806
```cpp
      BaseT::getScalarizationOverhead(VTy, /*Insert*/ false, /*Extract*/ true,
                                      CostKind);
```
- EN: Declares `BaseT::getScalarizationOverhead`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `BaseT::getScalarizationOverhead`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1808-1809
```cpp
  if (EltSize < 8 || Alignment < EltSize / 8)
    return ScalarCost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1811-1825
```cpp
  unsigned ExtSize = EltSize;
  // Check whether there's a single user that asks for an extended type
  if (I != nullptr) {
    // Dependent of the caller of this function, a gather instruction will
    // either have opcode Instruction::Load or be a call to the masked_gather
    // intrinsic
    if ((I->getOpcode() == Instruction::Load ||
         match(I, m_Intrinsic<Intrinsic::masked_gather>())) &&
        I->hasOneUse()) {
      const User *Us = *I->users().begin();
      if (isa<ZExtInst>(Us) || isa<SExtInst>(Us)) {
        // only allow valid type combinations
        unsigned TypeSize =
            cast<Instruction>(Us)->getType()->getScalarSizeInBits();
        if (((TypeSize == 32 && (EltSize == 8 || EltSize == 16)) ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1826-1840
```cpp
             (TypeSize == 16 && EltSize == 8)) &&
            TypeSize * NumElems == 128) {
          ExtSize = TypeSize;
        }
      }
    }
    // Check whether the input data needs to be truncated
    TruncInst *T;
    if ((I->getOpcode() == Instruction::Store ||
         match(I, m_Intrinsic<Intrinsic::masked_scatter>())) &&
        (T = dyn_cast<TruncInst>(I->getOperand(0)))) {
      // Only allow valid type combinations
      unsigned TypeSize = T->getOperand(0)->getType()->getScalarSizeInBits();
      if (((EltSize == 16 && TypeSize == 32) ||
           (EltSize == 8 && (TypeSize == 32 || TypeSize == 16))) &&
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1841-1844
```cpp
          TypeSize * NumElems == 128)
        ExtSize = TypeSize;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1846-1847
```cpp
  if (ExtSize * NumElems != 128 || NumElems < 4)
    return ScalarCost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1849-1856
```cpp
  // Any (aligned) i32 gather will not need to be scalarised.
  if (ExtSize == 32)
    return VectorCost;
  // For smaller types, we need to ensure that the gep's inputs are correctly
  // extended from a small enough value. Other sizes (including i64) are
  // scalarized for now.
  if (ExtSize != 8 && ExtSize != 16)
    return ScalarCost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1858-1872
```cpp
  if (const auto *BC = dyn_cast<BitCastInst>(Ptr))
    Ptr = BC->getOperand(0);
  if (const auto *GEP = dyn_cast<GetElementPtrInst>(Ptr)) {
    if (GEP->getNumOperands() != 2)
      return ScalarCost;
    unsigned Scale = DL.getTypeAllocSize(GEP->getResultElementType());
    // Scale needs to be correct (which is only relevant for i16s).
    if (Scale != 1 && Scale * 8 != ExtSize)
      return ScalarCost;
    // And we need to zext (not sext) the indexes from a small enough type.
    if (const auto *ZExt = dyn_cast<ZExtInst>(GEP->getOperand(1))) {
      if (ZExt->getOperand(0)->getType()->getScalarSizeInBits() <= ExtSize)
        return VectorCost;
    }
    return ScalarCost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1873-1875
```cpp
  }
  return ScalarCost;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1877-1880
```cpp
InstructionCost
ARMTTIImpl::getArithmeticReductionCost(unsigned Opcode, VectorType *ValTy,
                                       std::optional<FastMathFlags> FMF,
                                       TTI::TargetCostKind CostKind) const {
```
- EN: Implements `ARMTTIImpl::getArithmeticReductionCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getArithmeticReductionCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1882-1884
```cpp
  EVT ValVT = TLI->getValueType(DL, ValTy);
  int ISD = TLI->InstructionOpcodeToISD(Opcode);
  unsigned EltSize = ValVT.getScalarSizeInBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1886-1900
```cpp
  // In general floating point reductions are a series of elementwise
  // operations, with free extracts on each step. These are either in-order or
  // treewise depending on whether that is allowed by the fast math flags.
  if ((ISD == ISD::FADD || ISD == ISD::FMUL) &&
      ((EltSize == 32 && ST->hasVFP2Base()) ||
       (EltSize == 64 && ST->hasFP64()) ||
       (EltSize == 16 && ST->hasFullFP16()))) {
    unsigned NumElts = cast<FixedVectorType>(ValTy)->getNumElements();
    unsigned VecLimit = ST->hasMVEFloatOps() ? 128 : (ST->hasNEON() ? 64 : -1);
    InstructionCost VecCost = 0;
    while (!TTI::requiresOrderedReduction(FMF) && isPowerOf2_32(NumElts) &&
           NumElts * EltSize > VecLimit) {
      Type *VecTy = FixedVectorType::get(ValTy->getElementType(), NumElts / 2);
      VecCost += getArithmeticInstrCost(Opcode, VecTy, CostKind);
      NumElts /= 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1901-1901
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1903-1911
```cpp
    // For fp16 we need to extract the upper lane elements. MVE can add a
    // VREV+FMIN/MAX to perform another vector step instead.
    InstructionCost ExtractCost = 0;
    if (!TTI::requiresOrderedReduction(FMF) && ST->hasMVEFloatOps() &&
        ValVT.getVectorElementType() == MVT::f16 && NumElts == 8) {
      VecCost += ST->getMVEVectorCostFactor(CostKind) * 2;
      NumElts /= 2;
    } else if (ValVT.getVectorElementType() == MVT::f16)
      ExtractCost = NumElts / 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1913-1916
```cpp
    return VecCost + ExtractCost +
           NumElts *
               getArithmeticInstrCost(Opcode, ValTy->getElementType(), CostKind);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1918-1932
```cpp
  if ((ISD == ISD::AND || ISD == ISD::OR || ISD == ISD::XOR) &&
      (EltSize == 64 || EltSize == 32 || EltSize == 16 || EltSize == 8)) {
    unsigned NumElts = cast<FixedVectorType>(ValTy)->getNumElements();
    unsigned VecLimit =
        ST->hasMVEIntegerOps() ? 128 : (ST->hasNEON() ? 64 : -1);
    InstructionCost VecCost = 0;
    while (isPowerOf2_32(NumElts) && NumElts * EltSize > VecLimit) {
      Type *VecTy = FixedVectorType::get(ValTy->getElementType(), NumElts / 2);
      VecCost += getArithmeticInstrCost(Opcode, VecTy, CostKind);
      NumElts /= 2;
    }
    // For i16/i8, MVE will perform a VREV + VORR/VAND/VEOR for the 64bit vector
    // step.
    if (ST->hasMVEIntegerOps() && ValVT.getScalarSizeInBits() <= 16 &&
        NumElts * EltSize == 64) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1933-1937
```cpp
      Type *VecTy = FixedVectorType::get(ValTy->getElementType(), NumElts);
      VecCost += ST->getMVEVectorCostFactor(CostKind) +
                 getArithmeticInstrCost(Opcode, VecTy, CostKind);
      NumElts /= 2;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1939-1944
```cpp
    // From here we extract the elements and perform the and/or/xor.
    InstructionCost ExtractCost = NumElts;
    return VecCost + ExtractCost +
           (NumElts - 1) * getArithmeticInstrCost(
                               Opcode, ValTy->getElementType(), CostKind);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1946-1948
```cpp
  if (!ST->hasMVEIntegerOps() || !ValVT.isSimple() || ISD != ISD::ADD ||
      TTI::requiresOrderedReduction(FMF))
    return BaseT::getArithmeticReductionCost(Opcode, ValTy, FMF, CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1950-1950
```cpp
  std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(ValTy);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1952-1958
```cpp
  static const CostTblEntry CostTblAdd[]{
      {ISD::ADD, MVT::v16i8, 1},
      {ISD::ADD, MVT::v8i16, 1},
      {ISD::ADD, MVT::v4i32, 1},
  };
  if (const auto *Entry = CostTableLookup(CostTblAdd, ISD, LT.second))
    return Entry->Cost * ST->getMVEVectorCostFactor(CostKind) * LT.first;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1960-1961
```cpp
  return BaseT::getArithmeticReductionCost(Opcode, ValTy, FMF, CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1963-1967
```cpp
InstructionCost ARMTTIImpl::getExtendedReductionCost(
    unsigned Opcode, bool IsUnsigned, Type *ResTy, VectorType *ValTy,
    std::optional<FastMathFlags> FMF, TTI::TargetCostKind CostKind) const {
  EVT ValVT = TLI->getValueType(DL, ValTy);
  EVT ResVT = TLI->getValueType(DL, ResTy);
```
- EN: Implements `ARMTTIImpl::getExtendedReductionCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getExtendedReductionCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1969-1969
```cpp
  int ISD = TLI->InstructionOpcodeToISD(Opcode);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1971-1974
```cpp
  switch (ISD) {
  case ISD::ADD:
    if (ST->hasMVEIntegerOps() && ValVT.isSimple() && ResVT.isSimple()) {
      std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(ValTy);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1976-1990
```cpp
      // The legal cases are:
      //   VADDV u/s 8/16/32
      //   VADDLV u/s 32
      // Codegen currently cannot always handle larger than legal vectors very
      // well, especially for predicated reductions where the mask needs to be
      // split, so restrict to 128bit or smaller input types.
      unsigned RevVTSize = ResVT.getSizeInBits();
      if (ValVT.getSizeInBits() <= 128 &&
          ((LT.second == MVT::v16i8 && RevVTSize <= 32) ||
           (LT.second == MVT::v8i16 && RevVTSize <= 32) ||
           (LT.second == MVT::v4i32 && RevVTSize <= 64)))
        return ST->getMVEVectorCostFactor(CostKind) * LT.first;
    }
    break;
  default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1991-1995
```cpp
    break;
  }
  return BaseT::getExtendedReductionCost(Opcode, IsUnsigned, ResTy, ValTy, FMF,
                                         CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1997-2004
```cpp
InstructionCost
ARMTTIImpl::getMulAccReductionCost(bool IsUnsigned, unsigned RedOpcode,
                                   Type *ResTy, VectorType *ValTy,
                                   TTI::TargetCostKind CostKind) const {
  if (RedOpcode != Instruction::Add)
    return InstructionCost::getInvalid(CostKind);
  EVT ValVT = TLI->getValueType(DL, ValTy);
  EVT ResVT = TLI->getValueType(DL, ResTy);
```
- EN: Implements `ARMTTIImpl::getMulAccReductionCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getMulAccReductionCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2006-2007
```cpp
  if (ST->hasMVEIntegerOps() && ValVT.isSimple() && ResVT.isSimple()) {
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(ValTy);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2009-2021
```cpp
    // The legal cases are:
    //   VMLAV u/s 8/16/32
    //   VMLALV u/s 16/32
    // Codegen currently cannot always handle larger than legal vectors very
    // well, especially for predicated reductions where the mask needs to be
    // split, so restrict to 128bit or smaller input types.
    unsigned RevVTSize = ResVT.getSizeInBits();
    if (ValVT.getSizeInBits() <= 128 &&
        ((LT.second == MVT::v16i8 && RevVTSize <= 32) ||
         (LT.second == MVT::v8i16 && RevVTSize <= 64) ||
         (LT.second == MVT::v4i32 && RevVTSize <= 64)))
      return ST->getMVEVectorCostFactor(CostKind) * LT.first;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2023-2025
```cpp
  return BaseT::getMulAccReductionCost(IsUnsigned, RedOpcode, ResTy, ValTy,
                                       CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2027-2031
```cpp
InstructionCost
ARMTTIImpl::getMinMaxReductionCost(Intrinsic::ID IID, VectorType *Ty,
                                   FastMathFlags FMF,
                                   TTI::TargetCostKind CostKind) const {
  EVT ValVT = TLI->getValueType(DL, Ty);
```
- EN: Implements `ARMTTIImpl::getMinMaxReductionCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getMinMaxReductionCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2033-2047
```cpp
  // In general floating point reductions are a series of elementwise
  // operations, with free extracts on each step. These are either in-order or
  // treewise depending on whether that is allowed by the fast math flags.
  if ((IID == Intrinsic::minnum || IID == Intrinsic::maxnum) &&
      ((ValVT.getVectorElementType() == MVT::f32 && ST->hasVFP2Base()) ||
       (ValVT.getVectorElementType() == MVT::f64 && ST->hasFP64()) ||
       (ValVT.getVectorElementType() == MVT::f16 && ST->hasFullFP16()))) {
    unsigned NumElts = cast<FixedVectorType>(Ty)->getNumElements();
    unsigned EltSize = ValVT.getScalarSizeInBits();
    unsigned VecLimit = ST->hasMVEFloatOps() ? 128 : (ST->hasNEON() ? 64 : -1);
    InstructionCost VecCost;
    while (isPowerOf2_32(NumElts) && NumElts * EltSize > VecLimit) {
      Type *VecTy = FixedVectorType::get(Ty->getElementType(), NumElts/2);
      IntrinsicCostAttributes ICA(IID, VecTy, {VecTy, VecTy}, FMF);
      VecCost += getIntrinsicInstrCost(ICA, CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2048-2049
```cpp
      NumElts /= 2;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2051-2059
```cpp
    // For fp16 we need to extract the upper lane elements. MVE can add a
    // VREV+FMIN/MAX to perform another vector step instead.
    InstructionCost ExtractCost = 0;
    if (ST->hasMVEFloatOps() && ValVT.getVectorElementType() == MVT::f16 &&
        NumElts == 8) {
      VecCost += ST->getMVEVectorCostFactor(CostKind) * 2;
      NumElts /= 2;
    } else if (ValVT.getVectorElementType() == MVT::f16)
      ExtractCost = cast<FixedVectorType>(Ty)->getNumElements() / 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2061-2066
```cpp
    IntrinsicCostAttributes ICA(IID, Ty->getElementType(),
                                {Ty->getElementType(), Ty->getElementType()},
                                FMF);
    return VecCost + ExtractCost +
           (NumElts - 1) * getIntrinsicInstrCost(ICA, CostKind);
  }
```
- EN: Implements `ICA`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ICA`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2068-2070
```cpp
  if (IID == Intrinsic::smin || IID == Intrinsic::smax ||
      IID == Intrinsic::umin || IID == Intrinsic::umax) {
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(Ty);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2072-2082
```cpp
    // All costs are the same for u/s min/max.  These lower to vminv, which are
    // given a slightly higher cost as they tend to take multiple cycles for
    // smaller type sizes.
    static const CostTblEntry CostTblAdd[]{
        {ISD::SMIN, MVT::v16i8, 4},
        {ISD::SMIN, MVT::v8i16, 3},
        {ISD::SMIN, MVT::v4i32, 2},
    };
    if (const auto *Entry = CostTableLookup(CostTblAdd, ISD::SMIN, LT.second))
      return Entry->Cost * ST->getMVEVectorCostFactor(CostKind) * LT.first;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2084-2085
```cpp
  return BaseT::getMinMaxReductionCost(IID, Ty, FMF, CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2087-2101
```cpp
InstructionCost
ARMTTIImpl::getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
                                  TTI::TargetCostKind CostKind) const {
  unsigned Opc = ICA.getID();
  switch (Opc) {
  case Intrinsic::get_active_lane_mask:
    // Currently we make a somewhat optimistic assumption that
    // active_lane_mask's are always free. In reality it may be freely folded
    // into a tail predicated loop, expanded into a VCPT or expanded into a lot
    // of add/icmp code. We may need to improve this in the future, but being
    // able to detect if it is free or not involves looking at a lot of other
    // code. We currently assume that the vectorizer inserted these, and knew
    // what it was doing in adding one.
    if (ST->hasMVEIntegerOps())
      return 0;
```
- EN: Implements `ARMTTIImpl::getIntrinsicInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getIntrinsicInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2102-2109
```cpp
    break;
  case Intrinsic::sadd_sat:
  case Intrinsic::ssub_sat:
  case Intrinsic::uadd_sat:
  case Intrinsic::usub_sat: {
    bool IsAdd = (Opc == Intrinsic::sadd_sat || Opc == Intrinsic::ssub_sat);
    bool IsSigned = (Opc == Intrinsic::sadd_sat || Opc == Intrinsic::ssub_sat);
    Type *RetTy = ICA.getReturnType();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2111-2125
```cpp
    if (auto *ITy = dyn_cast<IntegerType>(RetTy)) {
      if (IsSigned && ST->hasDSP() && ITy->getBitWidth() == 32)
        return 1; // qadd / qsub
      if (ST->hasDSP() && (ITy->getBitWidth() == 8 || ITy->getBitWidth() == 16))
        return 2; // uqadd16 / qadd16 / uqsub16 / qsub16 + possible extend.
      // Otherwise return the cost of expanding the node. Generally an add +
      // icmp + sel.
      CmpInst::Predicate Pred = CmpInst::ICMP_SGT;
      Type *CondTy = RetTy->getWithNewBitWidth(1);
      return getArithmeticInstrCost(IsAdd ? Instruction::Add : Instruction::Sub,
                                    RetTy, CostKind) +
             2 * getCmpSelInstrCost(BinaryOperator::ICmp, RetTy, CondTy, Pred,
                                    CostKind) +
             2 * getCmpSelInstrCost(BinaryOperator::Select, RetTy, CondTy, Pred,
                                    CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2126-2126
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2128-2129
```cpp
    if (!ST->hasMVEIntegerOps())
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2131-2145
```cpp
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(RetTy);
    if (LT.second == MVT::v4i32 || LT.second == MVT::v8i16 ||
        LT.second == MVT::v16i8) {
      // This is a base cost of 1 for the vqadd, plus 3 extract shifts if we
      // need to extend the type, as it uses shr(qadd(shl, shl)).
      unsigned Instrs =
          LT.second.getScalarSizeInBits() == RetTy->getScalarSizeInBits() ? 1
                                                                          : 4;
      return LT.first * ST->getMVEVectorCostFactor(CostKind) * Instrs;
    }
    break;
  }
  case Intrinsic::abs:
  case Intrinsic::smin:
  case Intrinsic::smax:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2146-2150
```cpp
  case Intrinsic::umin:
  case Intrinsic::umax: {
    if (!ST->hasMVEIntegerOps())
      break;
    Type *VT = ICA.getReturnType();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2152-2166
```cpp
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(VT);
    if (LT.second == MVT::v4i32 || LT.second == MVT::v8i16 ||
        LT.second == MVT::v16i8)
      return LT.first * ST->getMVEVectorCostFactor(CostKind);
    break;
  }
  case Intrinsic::minnum:
  case Intrinsic::maxnum: {
    if (!ST->hasMVEFloatOps())
      break;
    Type *VT = ICA.getReturnType();
    std::pair<InstructionCost, MVT> LT = getTypeLegalizationCost(VT);
    if (LT.second == MVT::v4f32 || LT.second == MVT::v8f16)
      return LT.first * ST->getMVEVectorCostFactor(CostKind);
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2167-2179
```cpp
  }
  case Intrinsic::fptosi_sat:
  case Intrinsic::fptoui_sat: {
    if (ICA.getArgTypes().empty())
      break;
    bool IsSigned = Opc == Intrinsic::fptosi_sat;
    auto LT = getTypeLegalizationCost(ICA.getArgTypes()[0]);
    EVT MTy = TLI->getValueType(DL, ICA.getReturnType());
    // Check for the legal types, with the correct subtarget features.
    if ((ST->hasVFP2Base() && LT.second == MVT::f32 && MTy == MVT::i32) ||
        (ST->hasFP64() && LT.second == MVT::f64 && MTy == MVT::i32) ||
        (ST->hasFullFP16() && LT.second == MVT::f16 && MTy == MVT::i32))
      return LT.first;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2181-2185
```cpp
    // Equally for MVE vector types
    if (ST->hasMVEFloatOps() &&
        (LT.second == MVT::v4f32 || LT.second == MVT::v8f16) &&
        LT.second.getScalarSizeInBits() == MTy.getScalarSizeInBits())
      return LT.first * ST->getMVEVectorCostFactor(CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2187-2201
```cpp
    // If we can we use a legal convert followed by a min+max
    if (((ST->hasVFP2Base() && LT.second == MVT::f32) ||
         (ST->hasFP64() && LT.second == MVT::f64) ||
         (ST->hasFullFP16() && LT.second == MVT::f16) ||
         (ST->hasMVEFloatOps() &&
          (LT.second == MVT::v4f32 || LT.second == MVT::v8f16))) &&
        LT.second.getScalarSizeInBits() >= MTy.getScalarSizeInBits()) {
      Type *LegalTy = Type::getIntNTy(ICA.getReturnType()->getContext(),
                                      LT.second.getScalarSizeInBits());
      InstructionCost Cost =
          LT.second.isVector() ? ST->getMVEVectorCostFactor(CostKind) : 1;
      IntrinsicCostAttributes Attrs1(IsSigned ? Intrinsic::smin
                                              : Intrinsic::umin,
                                     LegalTy, {LegalTy, LegalTy});
      Cost += getIntrinsicInstrCost(Attrs1, CostKind);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2202-2216
```cpp
      IntrinsicCostAttributes Attrs2(IsSigned ? Intrinsic::smax
                                              : Intrinsic::umax,
                                     LegalTy, {LegalTy, LegalTy});
      Cost += getIntrinsicInstrCost(Attrs2, CostKind);
      return LT.first * Cost;
    }
    // Otherwise we need to follow the default expansion that clamps the value
    // using a float min/max with a fcmp+sel for nan handling when signed.
    Type *FPTy = ICA.getArgTypes()[0];
    Type *RetTy = ICA.getReturnType();
    IntrinsicCostAttributes Attrs1(Intrinsic::minnum, FPTy, {FPTy, FPTy});
    InstructionCost Cost = getIntrinsicInstrCost(Attrs1, CostKind);
    IntrinsicCostAttributes Attrs2(Intrinsic::maxnum, FPTy, {FPTy, FPTy});
    Cost += getIntrinsicInstrCost(Attrs2, CostKind);
    Cost +=
```
- EN: Implements `Attrs2`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Attrs2`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2217-2228
```cpp
        getCastInstrCost(IsSigned ? Instruction::FPToSI : Instruction::FPToUI,
                         RetTy, FPTy, TTI::CastContextHint::None, CostKind);
    if (IsSigned) {
      Type *CondTy = RetTy->getWithNewBitWidth(1);
      Cost += getCmpSelInstrCost(BinaryOperator::FCmp, FPTy, CondTy,
                                 CmpInst::FCMP_UNO, CostKind);
      Cost += getCmpSelInstrCost(BinaryOperator::Select, RetTy, CondTy,
                                 CmpInst::FCMP_UNO, CostKind);
    }
    return Cost;
  }
  }
```
- EN: Implements `getCastInstrCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCastInstrCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2230-2231
```cpp
  return BaseT::getIntrinsicInstrCost(ICA, CostKind);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2233-2235
```cpp
bool ARMTTIImpl::isLoweredToCall(const Function *F) const {
  if (!F->isIntrinsic())
    return BaseT::isLoweredToCall(F);
```
- EN: Implements `ARMTTIImpl::isLoweredToCall`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::isLoweredToCall`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2237-2239
```cpp
  // Assume all Arm-specific intrinsics map to an instruction.
  if (F->getName().starts_with("llvm.arm"))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2241-2255
```cpp
  switch (F->getIntrinsicID()) {
  default: break;
  case Intrinsic::powi:
  case Intrinsic::sin:
  case Intrinsic::cos:
  case Intrinsic::sincos:
  case Intrinsic::pow:
  case Intrinsic::log:
  case Intrinsic::log10:
  case Intrinsic::log2:
  case Intrinsic::exp:
  case Intrinsic::exp2:
    return true;
  case Intrinsic::sqrt:
  case Intrinsic::fabs:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2256-2270
```cpp
  case Intrinsic::copysign:
  case Intrinsic::floor:
  case Intrinsic::ceil:
  case Intrinsic::trunc:
  case Intrinsic::rint:
  case Intrinsic::nearbyint:
  case Intrinsic::round:
  case Intrinsic::canonicalize:
  case Intrinsic::lround:
  case Intrinsic::llround:
  case Intrinsic::lrint:
  case Intrinsic::llrint:
    if (F->getReturnType()->isDoubleTy() && !ST->hasFP64())
      return true;
    if (F->getReturnType()->isHalfTy() && !ST->hasFullFP16())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2271-2285
```cpp
      return true;
    // Some operations can be handled by vector instructions and assume
    // unsupported vectors will be expanded into supported scalar ones.
    // TODO Handle scalar operations properly.
    return !ST->hasFPARMv8Base() && !ST->hasVFP2Base();
  case Intrinsic::masked_store:
  case Intrinsic::masked_load:
  case Intrinsic::masked_gather:
  case Intrinsic::masked_scatter:
    return !ST->hasMVEIntegerOps();
  case Intrinsic::sadd_with_overflow:
  case Intrinsic::uadd_with_overflow:
  case Intrinsic::ssub_with_overflow:
  case Intrinsic::usub_with_overflow:
  case Intrinsic::sadd_sat:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2286-2290
```cpp
  case Intrinsic::uadd_sat:
  case Intrinsic::ssub_sat:
  case Intrinsic::usub_sat:
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2292-2293
```cpp
  return BaseT::isLoweredToCall(F);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2295-2299
```cpp
bool ARMTTIImpl::maybeLoweredToCall(Instruction &I) const {
  unsigned ISD = TLI->InstructionOpcodeToISD(I.getOpcode());
  EVT VT = TLI->getValueType(DL, I.getType(), true);
  if (TLI->getOperationAction(ISD, VT) == TargetLowering::LibCall)
    return true;
```
- EN: Implements `ARMTTIImpl::maybeLoweredToCall`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::maybeLoweredToCall`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2301-2315
```cpp
  // Check if an intrinsic will be lowered to a call and assume that any
  // other CallInst will generate a bl.
  if (auto *Call = dyn_cast<CallInst>(&I)) {
    if (auto *II = dyn_cast<IntrinsicInst>(Call)) {
      switch(II->getIntrinsicID()) {
        case Intrinsic::memcpy:
        case Intrinsic::memset:
        case Intrinsic::memmove:
          return getNumMemOps(II) == -1;
        default:
          if (const Function *F = Call->getCalledFunction())
            return isLoweredToCall(F);
      }
    }
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2316-2316
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2318-2330
```cpp
  // FPv5 provides conversions between integer, double-precision,
  // single-precision, and half-precision formats.
  switch (I.getOpcode()) {
  default:
    break;
  case Instruction::FPToSI:
  case Instruction::FPToUI:
  case Instruction::SIToFP:
  case Instruction::UIToFP:
  case Instruction::FPTrunc:
  case Instruction::FPExt:
    return !ST->hasFPARMv8Base();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2332-2346
```cpp
  // FIXME: Unfortunately the approach of checking the Operation Action does
  // not catch all cases of Legalization that use library calls. Our
  // Legalization step categorizes some transformations into library calls as
  // Custom, Expand or even Legal when doing type legalization. So for now
  // we have to special case for instance the SDIV of 64bit integers and the
  // use of floating point emulation.
  if (VT.isInteger() && VT.getSizeInBits() >= 64) {
    switch (ISD) {
    default:
      break;
    case ISD::SDIV:
    case ISD::UDIV:
    case ISD::SREM:
    case ISD::UREM:
    case ISD::SDIVREM:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2347-2350
```cpp
    case ISD::UDIVREM:
      return true;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2352-2354
```cpp
  // Assume all other non-float operations are supported.
  if (!VT.isFloatingPoint())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2356-2368
```cpp
  // We'll need a library call to handle most floats when using soft.
  if (TLI->useSoftFloat()) {
    switch (I.getOpcode()) {
    default:
      return true;
    case Instruction::Alloca:
    case Instruction::Load:
    case Instruction::Store:
    case Instruction::Select:
    case Instruction::PHI:
      return false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2370-2373
```cpp
  // We'll need a libcall to perform double precision operations on a single
  // precision only FPU.
  if (I.getType()->isDoubleTy() && !ST->hasFP64())
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2375-2377
```cpp
  // Likewise for half precision arithmetic.
  if (I.getType()->isHalfTy() && !ST->hasFullFP16())
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2379-2380
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2382-2391
```cpp
bool ARMTTIImpl::isHardwareLoopProfitable(Loop *L, ScalarEvolution &SE,
                                          AssumptionCache &AC,
                                          TargetLibraryInfo *LibInfo,
                                          HardwareLoopInfo &HWLoopInfo) const {
  // Low-overhead branches are only supported in the 'low-overhead branch'
  // extension of v8.1-m.
  if (!ST->hasLOB() || DisableLowOverheadLoops) {
    LLVM_DEBUG(dbgs() << "ARMHWLoops: Disabled\n");
    return false;
  }
```
- EN: Implements `ARMTTIImpl::isHardwareLoopProfitable`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::isHardwareLoopProfitable`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2393-2396
```cpp
  if (!SE.hasLoopInvariantBackedgeTakenCount(L)) {
    LLVM_DEBUG(dbgs() << "ARMHWLoops: No BETC\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2398-2402
```cpp
  const SCEV *BackedgeTakenCount = SE.getBackedgeTakenCount(L);
  if (isa<SCEVCouldNotCompute>(BackedgeTakenCount)) {
    LLVM_DEBUG(dbgs() << "ARMHWLoops: Uncomputable BETC\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2404-2406
```cpp
  const SCEV *TripCountSCEV =
    SE.getAddExpr(BackedgeTakenCount,
                  SE.getOne(BackedgeTakenCount->getType()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2408-2412
```cpp
  // We need to store the trip count in LR, a 32-bit register.
  if (SE.getUnsignedRangeMax(TripCountSCEV).getBitWidth() > 32) {
    LLVM_DEBUG(dbgs() << "ARMHWLoops: Trip count does not fit into 32bits\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2414-2415
```cpp
  // Making a call will trash LR and clear LO_BRANCH_INFO, so there's little
  // point in generating a hardware loop if that's going to happen.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2417-2430
```cpp
  auto IsHardwareLoopIntrinsic = [](Instruction &I) {
    if (auto *Call = dyn_cast<IntrinsicInst>(&I)) {
      switch (Call->getIntrinsicID()) {
      default:
        break;
      case Intrinsic::start_loop_iterations:
      case Intrinsic::test_start_loop_iterations:
      case Intrinsic::loop_decrement:
      case Intrinsic::loop_decrement_reg:
        return true;
      }
    }
    return false;
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2432-2446
```cpp
  // Scan the instructions to see if there's any that we know will turn into a
  // call or if this loop is already a low-overhead loop or will become a tail
  // predicated loop.
  bool IsTailPredLoop = false;
  auto ScanLoop = [&](Loop *L) {
    for (auto *BB : L->getBlocks()) {
      for (auto &I : *BB) {
        if (maybeLoweredToCall(I) || IsHardwareLoopIntrinsic(I) ||
            isa<InlineAsm>(I)) {
          LLVM_DEBUG(dbgs() << "ARMHWLoops: Bad instruction: " << I << "\n");
          return false;
        }
        if (auto *II = dyn_cast<IntrinsicInst>(&I))
          IsTailPredLoop |=
              II->getIntrinsicID() == Intrinsic::get_active_lane_mask ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2447-2454
```cpp
              II->getIntrinsicID() == Intrinsic::arm_mve_vctp8 ||
              II->getIntrinsicID() == Intrinsic::arm_mve_vctp16 ||
              II->getIntrinsicID() == Intrinsic::arm_mve_vctp32 ||
              II->getIntrinsicID() == Intrinsic::arm_mve_vctp64;
      }
    }
    return true;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2456-2459
```cpp
  // Visit inner loops.
  for (auto *Inner : *L)
    if (!ScanLoop(Inner))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2461-2462
```cpp
  if (!ScanLoop(L))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2464-2466
```cpp
  // TODO: Check whether the trip count calculation is expensive. If L is the
  // inner loop but we know it has a low trip count, calculating that trip
  // count (in the parent loop) may be detrimental.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 2468-2475
```cpp
  LLVMContext &C = L->getHeader()->getContext();
  HWLoopInfo.CounterInReg = true;
  HWLoopInfo.IsNestingLegal = false;
  HWLoopInfo.PerformEntryTest = AllowWLSLoops && !IsTailPredLoop;
  HWLoopInfo.CountType = Type::getInt32Ty(C);
  HWLoopInfo.LoopDecrement = ConstantInt::get(HWLoopInfo.CountType, 1);
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2477-2491
```cpp
static bool canTailPredicateInstruction(Instruction &I, int &ICmpCount) {
  // We don't allow icmp's, and because we only look at single block loops,
  // we simply count the icmps, i.e. there should only be 1 for the backedge.
  if (isa<ICmpInst>(&I) && ++ICmpCount > 1)
    return false;
  // FIXME: This is a workaround for poor cost modelling. Min/Max intrinsics are
  // not currently canonical, but soon will be. Code without them uses icmp, and
  // so is not tail predicated as per the condition above. In order to get the
  // same performance we treat min and max the same as an icmp for tailpred
  // purposes for the moment (we often rely on non-tailpred and higher VF's to
  // pick more optimal instructions like VQDMULH. They need to be recognized
  // directly by the vectorizer).
  if (auto *II = dyn_cast<IntrinsicInst>(&I))
    if ((II->getIntrinsicID() == Intrinsic::smin ||
         II->getIntrinsicID() == Intrinsic::smax ||
```
- EN: Implements `canTailPredicateInstruction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `canTailPredicateInstruction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2492-2495
```cpp
         II->getIntrinsicID() == Intrinsic::umin ||
         II->getIntrinsicID() == Intrinsic::umax) &&
        ++ICmpCount > 1)
      return false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2497-2498
```cpp
  if (isa<FCmpInst>(&I))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2500-2503
```cpp
  // We could allow extending/narrowing FP loads/stores, but codegen is
  // too inefficient so reject this for now.
  if (isa<FPExtInst>(&I) || isa<FPTruncInst>(&I))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2505-2508
```cpp
  // Extends have to be extending-loads
  if (isa<SExtInst>(&I) || isa<ZExtInst>(&I) )
    if (!I.getOperand(0)->hasOneUse() || !isa<LoadInst>(I.getOperand(0)))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2510-2513
```cpp
  // Truncs have to be narrowing-stores
  if (isa<TruncInst>(&I) )
    if (!I.hasOneUse() || !isa<StoreInst>(*I.user_begin()))
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2515-2516
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2518-2532
```cpp
// To set up a tail-predicated loop, we need to know the total number of
// elements processed by that loop. Thus, we need to determine the element
// size and:
// 1) it should be uniform for all operations in the vector loop, so we
//    e.g. don't want any widening/narrowing operations.
// 2) it should be smaller than i64s because we don't have vector operations
//    that work on i64s.
// 3) we don't want elements to be reversed or shuffled, to make sure the
//    tail-predication masks/predicates the right lanes.
//
static bool canTailPredicateLoop(Loop *L, LoopInfo *LI, ScalarEvolution &SE,
                                 const DataLayout &DL,
                                 const LoopAccessInfo *LAI,
                                 const DominatorTree &DT) {
  LLVM_DEBUG(dbgs() << "Tail-predication: checking allowed instructions\n");
```
- EN: Implements `canTailPredicateLoop`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `canTailPredicateLoop`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2534-2545
```cpp
  // If there are live-out values, it is probably a reduction. We can predicate
  // most reduction operations freely under MVE using a combination of
  // prefer-predicated-reduction-select and inloop reductions. We limit this to
  // floating point and integer reductions, but don't check for operators
  // specifically here. If the value ends up not being a reduction (and so the
  // vectorizer cannot tailfold the loop), we should fall back to standard
  // vectorization automatically.
  SmallVector< Instruction *, 8 > LiveOuts;
  LiveOuts = llvm::findDefsUsedOutsideOfLoop(L);
  bool ReductionsDisabled =
      EnableTailPredication == TailPredication::EnabledNoReductions ||
      EnableTailPredication == TailPredication::ForceEnabledNoReductions;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2547-2558
```cpp
  for (auto *I : LiveOuts) {
    if (!I->getType()->isIntegerTy() && !I->getType()->isFloatTy() &&
        !I->getType()->isHalfTy()) {
      LLVM_DEBUG(dbgs() << "Don't tail-predicate loop with non-integer/float "
                           "live-out value\n");
      return false;
    }
    if (ReductionsDisabled) {
      LLVM_DEBUG(dbgs() << "Reductions not enabled\n");
      return false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2560-2562
```cpp
  // Next, check that all instructions can be tail-predicated.
  PredicatedScalarEvolution PSE = LAI->getPSE();
  int ICmpCount = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2564-2571
```cpp
  for (BasicBlock *BB : L->blocks()) {
    for (Instruction &I : *BB) {
      if (isa<PHINode>(I) || isa<PseudoProbeInst>(I))
        continue;
      if (!canTailPredicateInstruction(I, ICmpCount)) {
        LLVM_DEBUG(dbgs() << "Instruction not allowed: "; I.dump());
        return false;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2573-2587
```cpp
      Type *T  = I.getType();
      if (T->getScalarSizeInBits() > 32) {
        LLVM_DEBUG(dbgs() << "Unsupported Type: "; T->dump());
        return false;
      }
      if (isa<StoreInst>(I) || isa<LoadInst>(I)) {
        Value *Ptr = getLoadStorePointerOperand(&I);
        Type *AccessTy = getLoadStoreType(&I);
        int64_t NextStride =
            getPtrStride(PSE, AccessTy, Ptr, L, DT).value_or(0);
        if (NextStride == 1) {
          // TODO: for now only allow consecutive strides of 1. We could support
          // other strides as long as it is uniform, but let's keep it simple
          // for now.
          continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2588-2602
```cpp
        } else if (NextStride == -1 ||
                   (NextStride == 2 && MVEMaxSupportedInterleaveFactor >= 2) ||
                   (NextStride == 4 && MVEMaxSupportedInterleaveFactor >= 4)) {
          LLVM_DEBUG(dbgs()
                     << "Consecutive strides of 2 found, vld2/vstr2 can't "
                        "be tail-predicated\n.");
          return false;
          // TODO: don't tail predicate if there is a reversed load?
        } else if (EnableMaskedGatherScatters) {
          // Gather/scatters do allow loading from arbitrary strides, at
          // least if they are loop invariant.
          // TODO: Loop variant strides should in theory work, too, but
          // this requires further testing.
          const SCEV *PtrScev = PSE.getSE()->getSCEV(Ptr);
          if (auto AR = dyn_cast<SCEVAddRecExpr>(PtrScev)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2603-2613
```cpp
            const SCEV *Step = AR->getStepRecurrence(*PSE.getSE());
            if (PSE.getSE()->isLoopInvariant(Step, L))
              continue;
          }
        }
        LLVM_DEBUG(dbgs() << "Bad stride found, can't "
                             "tail-predicate\n.");
        return false;
      }
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2615-2617
```cpp
  LLVM_DEBUG(dbgs() << "tail-predication: all instructions allowed!\n");
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2619-2623
```cpp
bool ARMTTIImpl::preferTailFoldingOverEpilogue(TailFoldingInfo *TFI) const {
  if (!EnableTailPredication) {
    LLVM_DEBUG(dbgs() << "Tail-folding not enabled.\n");
    return false;
  }
```
- EN: Implements `ARMTTIImpl::preferTailFoldingOverEpilogue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::preferTailFoldingOverEpilogue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2625-2629
```cpp
  // Creating a tail-folded vector loop is the first step for generating a
  // tail-folded hardware loop, for which we need the MVE masked
  // load/stores instructions:
  if (!ST->hasMVEIntegerOps())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2631-2632
```cpp
  LoopVectorizationLegality *LVL = TFI->LVL;
  Loop *L = LVL->getLoop();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2634-2639
```cpp
  // For now, restrict this to single block loops.
  if (L->getNumBlocks() > 1) {
    LLVM_DEBUG(dbgs() << "preferTailFoldingOverEpilogue: not a single block "
                         "loop.\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2641-2642
```cpp
  assert(L->isInnermost() &&
         "preferTailFoldingOverEpilogue: inner-loop expected");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2644-2650
```cpp
  LoopInfo *LI = LVL->getLoopInfo();
  HardwareLoopInfo HWLoopInfo(L);
  if (!HWLoopInfo.canAnalyze(*LI)) {
    LLVM_DEBUG(dbgs() << "preferTailFoldingOverEpilogue: hardware-loop is not "
                         "analyzable.\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2652-2653
```cpp
  AssumptionCache *AC = LVL->getAssumptionCache();
  ScalarEvolution *SE = LVL->getScalarEvolution();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2655-2661
```cpp
  // This checks if we have the low-overhead branch architecture
  // extension, and if we will create a hardware-loop:
  if (!isHardwareLoopProfitable(L, *SE, *AC, TFI->TLI, HWLoopInfo)) {
    LLVM_DEBUG(dbgs() << "preferTailFoldingOverEpilogue: hardware-loop is not "
                         "profitable.\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2663-2668
```cpp
  DominatorTree *DT = LVL->getDominatorTree();
  if (!HWLoopInfo.isHardwareLoopCandidate(*SE, *LI, *DT)) {
    LLVM_DEBUG(dbgs() << "preferTailFoldingOverEpilogue: hardware-loop is not "
                         "a candidate.\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2670-2672
```cpp
  return canTailPredicateLoop(L, LI, *SE, DL, LVL->getLAI(),
                              *LVL->getDominatorTree());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2674-2676
```cpp
TailFoldingStyle ARMTTIImpl::getPreferredTailFoldingStyle() const {
  if (!ST->hasMVEIntegerOps() || !EnableTailPredication)
    return TailFoldingStyle::DataWithoutLaneMask;
```
- EN: Implements `ARMTTIImpl::getPreferredTailFoldingStyle`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getPreferredTailFoldingStyle`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2678-2692
```cpp
  // Intrinsic @llvm.get.active.lane.mask is supported.
  // It is used in the MVETailPredication pass, which requires the number of
  // elements processed by this vector loop to setup the tail-predicated
  // loop.
  return TailFoldingStyle::Data;
}
void ARMTTIImpl::getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
                                         TTI::UnrollingPreferences &UP,
                                         OptimizationRemarkEmitter *ORE) const {
  // Enable Upper bound unrolling universally, providing that we do not see an
  // active lane mask, which will be better kept as a loop to become tail
  // predicated than to be conditionally unrolled.
  UP.UpperBound =
      !ST->hasMVEIntegerOps() || !any_of(*L->getHeader(), [](Instruction &I) {
        return isa<IntrinsicInst>(I) &&
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2693-2695
```cpp
               cast<IntrinsicInst>(I).getIntrinsicID() ==
                   Intrinsic::get_active_lane_mask;
      });
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2697-2699
```cpp
  // Only currently enable these preferences for M-Class cores.
  if (!ST->isMClass())
    return BasicTTIImplBase::getUnrollingPreferences(L, SE, UP, ORE);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2701-2705
```cpp
  // Disable loop unrolling for Oz and Os.
  UP.OptSizeThreshold = 0;
  UP.PartialOptSizeThreshold = 0;
  if (L->getHeader()->getParent()->hasOptSize())
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2707-2711
```cpp
  SmallVector<BasicBlock*, 4> ExitingBlocks;
  L->getExitingBlocks(ExitingBlocks);
  LLVM_DEBUG(dbgs() << "Loop has:\n"
                    << "Blocks: " << L->getNumBlocks() << "\n"
                    << "Exit blocks: " << ExitingBlocks.size() << "\n");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2713-2716
```cpp
  // Only allow another exit other than the latch. This acts as an early exit
  // as it mirrors the profitability calculation of the runtime unroller.
  if (ExitingBlocks.size() > 2)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2718-2721
```cpp
  // Limit the CFG of the loop body for targets with a branch predictor.
  // Allowing 4 blocks permits if-then-else diamonds in the body.
  if (ST->hasBranchPredictor() && L->getNumBlocks() > 4)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2723-2725
```cpp
  // Don't unroll vectorized loops, including the remainder loop
  if (getBooleanLoopAttribute(L, "llvm.loop.isvectorized"))
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2727-2735
```cpp
  // Scan the loop: don't unroll loops with calls as this could prevent
  // inlining.
  InstructionCost Cost = 0;
  for (auto *BB : L->getBlocks()) {
    for (auto &I : *BB) {
      // Don't unroll vectorised loop. MVE does not benefit from it as much as
      // scalar code.
      if (I.getType()->isVectorTy())
        return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2737-2743
```cpp
      if (isa<CallInst>(I) || isa<InvokeInst>(I)) {
        if (const Function *F = cast<CallBase>(I).getCalledFunction()) {
          if (!isLoweredToCall(F))
            continue;
        }
        return;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2745-2749
```cpp
      SmallVector<const Value*, 4> Operands(I.operand_values());
      Cost += getInstructionCost(&I, Operands,
                                 TargetTransformInfo::TCK_SizeAndLatency);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2751-2765
```cpp
  // On v6m cores, there are very few registers available. We can easily end up
  // spilling and reloading more registers in an unrolled loop. Look at the
  // number of LCSSA phis as a rough measure of how many registers will need to
  // be live out of the loop, reducing the default unroll count if more than 1
  // value is needed.  In the long run, all of this should be being learnt by a
  // machine.
  unsigned UnrollCount = 4;
  if (ST->isThumb1Only()) {
    unsigned ExitingValues = 0;
    SmallVector<BasicBlock *, 4> ExitBlocks;
    L->getExitBlocks(ExitBlocks);
    for (auto *Exit : ExitBlocks) {
      // Count the number of LCSSA phis. Exclude values coming from GEP's as
      // only the last is expected to be needed for address operands.
      unsigned LiveOuts = count_if(Exit->phis(), [](auto &PH) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2766-2775
```cpp
        return PH.getNumOperands() != 1 ||
               !isa<GetElementPtrInst>(PH.getOperand(0));
      });
      ExitingValues = ExitingValues < LiveOuts ? LiveOuts : ExitingValues;
    }
    if (ExitingValues)
      UnrollCount /= ExitingValues;
    if (UnrollCount <= 1)
      return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2777-2791
```cpp
  // For processors with low overhead branching (LOB), runtime unrolling the
  // innermost loop is often detrimental to performance. In these cases the loop
  // remainder gets unrolled into a series of compare-and-jump blocks, which in
  // deeply nested loops get executed multiple times, negating the benefits of
  // LOB. This is particularly noticeable when the loop trip count of the
  // innermost loop varies within the outer loop, such as in the case of
  // triangular matrix decompositions. In these cases we will prefer to not
  // unroll the innermost loop, with the intention for it to be executed as a
  // low overhead loop.
  bool Runtime = true;
  if (ST->hasLOB()) {
    if (SE.hasLoopInvariantBackedgeTakenCount(L)) {
      const SCEV *BETC = SE.getBackedgeTakenCount(L);
      auto *Outer = L->getOutermostLoop();
      if ((L != Outer && Outer != L->getParentLoop()) ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2792-2796
```cpp
          (L != Outer && BETC && !SE.isLoopInvariant(BETC, Outer))) {
        Runtime = false;
      }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2798-2799
```cpp
  LLVM_DEBUG(dbgs() << "Cost of loop: " << Cost << "\n");
  LLVM_DEBUG(dbgs() << "Default Runtime Unroll Count: " << UnrollCount << "\n");
```
- EN: Declares `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2801-2806
```cpp
  UP.Partial = true;
  UP.Runtime = Runtime;
  UP.UnrollRemainder = true;
  UP.DefaultUnrollRuntimeCount = UnrollCount;
  UP.UnrollAndJam = true;
  UP.UnrollAndJamInnerLoopThreshold = 60;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2808-2812
```cpp
  // Force unrolling small loops can be very useful because of the branch
  // taken cost of the backedge.
  if (Cost < ArmForceUnrollThreshold)
    UP.Force = true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2814-2817
```cpp
void ARMTTIImpl::getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                                       TTI::PeelingPreferences &PP) const {
  BaseT::getPeelingPreferences(L, SE, PP);
}
```
- EN: Implements `ARMTTIImpl::getPeelingPreferences`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getPeelingPreferences`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2819-2821
```cpp
bool ARMTTIImpl::preferInLoopReduction(RecurKind Kind, Type *Ty) const {
  if (!ST->hasMVEIntegerOps())
    return false;
```
- EN: Implements `ARMTTIImpl::preferInLoopReduction`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::preferInLoopReduction`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2823-2830
```cpp
  unsigned ScalarBits = Ty->getScalarSizeInBits();
  switch (Kind) {
  case RecurKind::Add:
    return ScalarBits <= 64;
  default:
    return false;
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2832-2836
```cpp
bool ARMTTIImpl::preferPredicatedReductionSelect() const {
  if (!ST->hasMVEIntegerOps())
    return false;
  return true;
}
```
- EN: Implements `ARMTTIImpl::preferPredicatedReductionSelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::preferPredicatedReductionSelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2838-2852
```cpp
InstructionCost ARMTTIImpl::getScalingFactorCost(Type *Ty, GlobalValue *BaseGV,
                                                 StackOffset BaseOffset,
                                                 bool HasBaseReg, int64_t Scale,
                                                 unsigned AddrSpace) const {
  TargetLoweringBase::AddrMode AM;
  AM.BaseGV = BaseGV;
  AM.BaseOffs = BaseOffset.getFixed();
  AM.HasBaseReg = HasBaseReg;
  AM.Scale = Scale;
  AM.ScalableOffset = BaseOffset.getScalable();
  if (getTLI()->isLegalAddressingMode(DL, AM, Ty, AddrSpace)) {
    if (ST->hasFPAO())
      return AM.Scale < 0 ? 1 : 0; // positive offsets execute faster
    return 0;
  }
```
- EN: Implements `ARMTTIImpl::getScalingFactorCost`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getScalingFactorCost`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2853-2854
```cpp
  return InstructionCost::getInvalid();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2856-2861
```cpp
bool ARMTTIImpl::shouldConsiderVectorizationRegPressure() const {
  // MVE only has 8 vector registers, so we should consider register pressure to
  // avoid vectorizing when the cost of spills exceeds the gains from
  // vectorization.
  return ST->hasMVEIntegerOps();
}
```
- EN: Implements `ARMTTIImpl::shouldConsiderVectorizationRegPressure`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::shouldConsiderVectorizationRegPressure`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2863-2874
```cpp
bool ARMTTIImpl::hasArmWideBranch(bool Thumb) const {
  if (Thumb) {
    // B.W is available in any Thumb2-supporting target, and also in every
    // version of Armv8-M, even Baseline which does not include the rest of
    // Thumb2.
    return ST->isThumb2() || ST->hasV8MBaselineOps();
  } else {
    // B is available in all versions of the Arm ISA, so the only question is
    // whether that ISA is available at all.
    return ST->hasARMOps();
  }
}
```
- EN: Implements `ARMTTIImpl::hasArmWideBranch`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::hasArmWideBranch`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2876-2879
```cpp
/// Check if Ext1 and Ext2 are extends of the same type, doubling the bitwidth
/// of the vector elements.
static bool areExtractExts(Value *Ext1, Value *Ext2) {
  using namespace PatternMatch;
```
- EN: Implements `areExtractExts`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `areExtractExts`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2881-2884
```cpp
  auto areExtDoubled = [](Instruction *Ext) {
    return Ext->getType()->getScalarSizeInBits() ==
           2 * Ext->getOperand(0)->getType()->getScalarSizeInBits();
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2886-2890
```cpp
  if (!match(Ext1, m_ZExtOrSExt(m_Value())) ||
      !match(Ext2, m_ZExtOrSExt(m_Value())) ||
      !areExtDoubled(cast<Instruction>(Ext1)) ||
      !areExtDoubled(cast<Instruction>(Ext2)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2892-2893
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2895-2900
```cpp
/// Check if sinking \p I's operands to I's basic block is profitable, because
/// the operands can be folded into a target instruction, e.g.
/// sext/zext can be folded into vsubl.
bool ARMTTIImpl::isProfitableToSinkOperands(Instruction *I,
                                            SmallVectorImpl<Use *> &Ops) const {
  using namespace PatternMatch;
```
- EN: Implements `ARMTTIImpl::isProfitableToSinkOperands`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::isProfitableToSinkOperands`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2902-2903
```cpp
  if (!I->getType()->isVectorTy())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2905-2918
```cpp
  if (ST->hasNEON()) {
    switch (I->getOpcode()) {
    case Instruction::Sub:
    case Instruction::Add: {
      if (!areExtractExts(I->getOperand(0), I->getOperand(1)))
        return false;
      Ops.push_back(&I->getOperandUse(0));
      Ops.push_back(&I->getOperandUse(1));
      return true;
    }
    default:
      return false;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2920-2921
```cpp
  if (!ST->hasMVEIntegerOps())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2923-2934
```cpp
  auto IsFMSMul = [&](Instruction *I) {
    if (!I->hasOneUse())
      return false;
    auto *Sub = cast<Instruction>(*I->users().begin());
    return Sub->getOpcode() == Instruction::FSub && Sub->getOperand(1) == I;
  };
  auto IsFMS = [&](Instruction *I) {
    if (match(I->getOperand(0), m_FNeg(m_Value())) ||
        match(I->getOperand(1), m_FNeg(m_Value())))
      return true;
    return false;
  };
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2936-2950
```cpp
  auto IsSinker = [&](Instruction *I, int Operand) {
    switch (I->getOpcode()) {
    case Instruction::Add:
    case Instruction::Mul:
    case Instruction::FAdd:
    case Instruction::ICmp:
    case Instruction::FCmp:
      return true;
    case Instruction::FMul:
      return !IsFMSMul(I);
    case Instruction::Sub:
    case Instruction::FSub:
    case Instruction::Shl:
    case Instruction::LShr:
    case Instruction::AShr:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2951-2965
```cpp
      return Operand == 1;
    case Instruction::Call:
      if (auto *II = dyn_cast<IntrinsicInst>(I)) {
        switch (II->getIntrinsicID()) {
        case Intrinsic::fma:
          return !IsFMS(I);
        case Intrinsic::sadd_sat:
        case Intrinsic::uadd_sat:
        case Intrinsic::arm_mve_add_predicated:
        case Intrinsic::arm_mve_mul_predicated:
        case Intrinsic::arm_mve_qadd_predicated:
        case Intrinsic::arm_mve_vhadd:
        case Intrinsic::arm_mve_hadd_predicated:
        case Intrinsic::arm_mve_vqdmull:
        case Intrinsic::arm_mve_vqdmull_predicated:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2966-2980
```cpp
        case Intrinsic::arm_mve_vqdmulh:
        case Intrinsic::arm_mve_qdmulh_predicated:
        case Intrinsic::arm_mve_vqrdmulh:
        case Intrinsic::arm_mve_qrdmulh_predicated:
        case Intrinsic::arm_mve_fma_predicated:
          return true;
        case Intrinsic::ssub_sat:
        case Intrinsic::usub_sat:
        case Intrinsic::arm_mve_sub_predicated:
        case Intrinsic::arm_mve_qsub_predicated:
        case Intrinsic::arm_mve_hsub_predicated:
        case Intrinsic::arm_mve_vhsub:
          return Operand == 1;
        default:
          return false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2981-2987
```cpp
        }
      }
      return false;
    default:
      return false;
    }
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2989-2993
```cpp
  for (auto OpIdx : enumerate(I->operands())) {
    Instruction *Op = dyn_cast<Instruction>(OpIdx.value().get());
    // Make sure we are not already sinking this operand
    if (!Op || any_of(Ops, [&](Use *U) { return U->get() == Op; }))
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2995-3004
```cpp
    Instruction *Shuffle = Op;
    if (Shuffle->getOpcode() == Instruction::BitCast)
      Shuffle = dyn_cast<Instruction>(Shuffle->getOperand(0));
    // We are looking for a splat that can be sunk.
    if (!Shuffle || !match(Shuffle, m_Shuffle(m_InsertElt(m_Undef(), m_Value(),
                                                          m_ZeroInt()),
                                              m_Undef(), m_ZeroMask())))
      continue;
    if (!IsSinker(I, OpIdx.index()))
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3006-3012
```cpp
    // All uses of the shuffle should be sunk to avoid duplicating it across gpr
    // and vector registers
    for (Use &U : Op->uses()) {
      Instruction *Insn = cast<Instruction>(U.getUser());
      if (!IsSinker(Insn, U.getOperandNo()))
        return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3014-3020
```cpp
    Ops.push_back(&Shuffle->getOperandUse(0));
    if (Shuffle != Op)
      Ops.push_back(&Op->getOperandUse(0));
    Ops.push_back(&OpIdx.value());
  }
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3022-3027
```cpp
unsigned ARMTTIImpl::getNumBytesToPadGlobalArray(unsigned Size,
                                                 Type *ArrayType) const {
  if (!UseWidenGlobalArrays) {
    LLVM_DEBUG(dbgs() << "Padding global arrays disabled\n");
    return false;
  }
```
- EN: Implements `ARMTTIImpl::getNumBytesToPadGlobalArray`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMTTIImpl::getNumBytesToPadGlobalArray`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3029-3032
```cpp
  // Don't modify none integer array types
  if (!ArrayType || !ArrayType->isArrayTy() ||
      !ArrayType->getArrayElementType()->isIntegerTy())
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3034-3036
```cpp
  // We pad to 4 byte boundaries
  if (Size % 4 == 0)
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3038-3039
```cpp
  unsigned NumBytesToPad = 4 - (Size % 4);
  unsigned NewSize = Size + NumBytesToPad;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3041-3043
```cpp
  // Max number of bytes that memcpy allows for lowering to load/stores before
  // it uses library function (__aeabi_memcpy).
  unsigned MaxMemIntrinsicSize = getMaxMemIntrinsicInlineSizeThreshold();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3045-3046
```cpp
  if (NewSize > MaxMemIntrinsicSize)
    return 0;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3048-3049
```cpp
  return NumBytesToPad;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMTargetTransformInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`.
  - CN: 后端本地头文件：`ARMTargetTransformInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/APInt.h`, `llvm/ADT/SmallVector.h`, `llvm/Analysis/LoopInfo.h`, `llvm/CodeGen/CostTable.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/ValueTypes.h`, `llvm/CodeGenTypes/MachineValueType.h`, `llvm/IR/BasicBlock.h` ... (+17 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APInt.h`, `llvm/ADT/SmallVector.h`, `llvm/Analysis/LoopInfo.h`, `llvm/CodeGen/CostTable.h`, `llvm/CodeGen/ISDOpcodes.h`, `llvm/CodeGen/ValueTypes.h`, `llvm/CodeGenTypes/MachineValueType.h`, `llvm/IR/BasicBlock.h` ... (+17 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cstdint`, `optional`, `utility`.
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cstdint`, `optional`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
