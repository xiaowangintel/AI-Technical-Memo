# MVELaneInterleavingPass.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MVELaneInterleavingPass.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MVELaneInterleavingPass` for the ARM backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 ARM 后端中的 `MVELaneInterleavingPass`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15
```cpp
//===- MVELaneInterleaving.cpp - Inverleave for MVE instructions ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass interleaves around sext/zext/trunc instructions. MVE does not have
// a single sext/zext or trunc instruction that takes the bottom half of a
// vector and extends to a full width, like NEON has with MOVL. Instead it is
// expected that this happens through top/bottom instructions. So the MVE
// equivalent VMOVLT/B instructions take either the even or odd elements of the
// input and extend them to the larger type, producing a vector with half the
// number of elements each of double the bitwidth. As there is no simple
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-30
```cpp
// instruction, we often have to turn sext/zext/trunc into a series of lane
// moves (or stack loads/stores, which we do not do yet).
//
// This pass takes vector code that starts at truncs, looks for interconnected
// blobs of operations that end with sext/zext (or constants/splats) of the
// form:
//   %sa = sext v8i16 %a to v8i32
//   %sb = sext v8i16 %b to v8i32
//   %add = add v8i32 %sa, %sb
//   %r = trunc %add to v8i16
// And adds shuffles to allow the use of VMOVL/VMOVN instructions:
//   %sha = shuffle v8i16 %a, undef, <0, 2, 4, 6, 1, 3, 5, 7>
//   %sa = sext v8i16 %sha to v8i32
//   %shb = shuffle v8i16 %b, undef, <0, 2, 4, 6, 1, 3, 5, 7>
//   %sb = sext v8i16 %shb to v8i32
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 31-43
```cpp
//   %add = add v8i32 %sa, %sb
//   %r = trunc %add to v8i16
//   %shr = shuffle v8i16 %r, undef, <0, 4, 1, 5, 2, 6, 3, 7>
// Which can then be split and lowered to MVE instructions efficiently:
//   %sa_b = VMOVLB.s16 %a
//   %sa_t = VMOVLT.s16 %a
//   %sb_b = VMOVLB.s16 %b
//   %sb_t = VMOVLT.s16 %b
//   %add_b = VADD.i32 %sa_b, %sb_b
//   %add_t = VADD.i32 %sa_t, %sb_t
//   %r = VMOVNT.i16 %add_b, %add_t
//
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 45-59
```cpp
#include "ARM.h"
#include "ARMBaseInstrInfo.h"
#include "ARMSubtarget.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/Instruction.h"
#include "llvm/IR/Instructions.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 60-67
```cpp
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/Type.h"
#include "llvm/IR/Value.h"
#include "llvm/InitializePasses.h"
#include "llvm/Pass.h"
#include "llvm/Support/Casting.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 69-69
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 71-71
```cpp
#define DEBUG_TYPE "mve-laneinterleave"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 73-75
```cpp
static cl::opt<bool> EnableInterleave(
    "enable-mve-interleave", cl::Hidden, cl::init(true),
    cl::desc("Enable interleave MVE vector operation lowering"));
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 77-77
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 79-81
```cpp
class MVELaneInterleaving : public FunctionPass {
public:
  static char ID; // Pass identification, replacement for typeid
```
- EN: Declares `MVELaneInterleaving`, packaging target-specific state and APIs around `MVELaneInterleavingPass`.
- CN: 这里声明 `MVELaneInterleaving`，把与 `MVELaneInterleavingPass` 相关的目标特定状态和 API 组织在一起。

### Lines 83-85
```cpp
  explicit MVELaneInterleaving() : FunctionPass(ID) {
    initializeMVELaneInterleavingPass(*PassRegistry::getPassRegistry());
  }
```
- EN: Implements `PassRegistry::getPassRegistry`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `PassRegistry::getPassRegistry`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-87
```cpp
  bool runOnFunction(Function &F) override;
```
- EN: Declares `runOnFunction`, a pass-entry routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnFunction`，它是一个围绕目标相关状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 89-89
```cpp
  StringRef getPassName() const override { return "MVE lane interleaving"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 91-96
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addRequired<TargetPassConfig>();
    FunctionPass::getAnalysisUsage(AU);
  }
};
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 98-98
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 100-100
```cpp
char MVELaneInterleaving::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 102-103
```cpp
INITIALIZE_PASS(MVELaneInterleaving, DEBUG_TYPE, "MVE lane interleaving", false,
                false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 105-107
```cpp
Pass *llvm::createMVELaneInterleavingPass() {
  return new MVELaneInterleaving();
}
```
- EN: Implements `llvm::createMVELaneInterleavingPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMVELaneInterleavingPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 109-119
```cpp
static bool isProfitableToInterleave(SmallSetVector<Instruction *, 4> &Exts,
                                     SmallSetVector<Instruction *, 4> &Truncs) {
  // This is not always beneficial to transform. Exts can be incorporated into
  // loads, Truncs can be folded into stores.
  // Truncs are usually the same number of instructions,
  //  VSTRH.32(A);VSTRH.32(B) vs VSTRH.16(VMOVNT A, B) with interleaving
  // Exts are unfortunately more instructions in the general case:
  //  A=VLDRH.32; B=VLDRH.32;
  // vs with interleaving:
  //  T=VLDRH.16; A=VMOVNB T; B=VMOVNT T
  // But those VMOVL may be folded into a VMULL.
```
- EN: Implements `isProfitableToInterleave`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isProfitableToInterleave`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-134
```cpp
  // But expensive extends/truncs are always good to remove. FPExts always
  // involve extra VCVT's so are always considered to be beneficial to convert.
  for (auto *E : Exts) {
    if (isa<FPExtInst>(E) || !isa<LoadInst>(E->getOperand(0))) {
      LLVM_DEBUG(dbgs() << "Beneficial due to " << *E << "\n");
      return true;
    }
  }
  for (auto *T : Truncs) {
    if (T->hasOneUse() && !isa<StoreInst>(*T->user_begin())) {
      LLVM_DEBUG(dbgs() << "Beneficial due to " << *T << "\n");
      return true;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 136-146
```cpp
  // Otherwise, we know we have a load(ext), see if any of the Extends are a
  // vmull. This is a simple heuristic and certainly not perfect.
  for (auto *E : Exts) {
    if (!E->hasOneUse() ||
        cast<Instruction>(*E->user_begin())->getOpcode() != Instruction::Mul) {
      LLVM_DEBUG(dbgs() << "Not beneficial due to " << *E << "\n");
      return false;
    }
  }
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 148-150
```cpp
static bool tryInterleave(Instruction *Start,
                          SmallPtrSetImpl<Instruction *> &Visited) {
  LLVM_DEBUG(dbgs() << "tryInterleave from " << *Start << "\n");
```
- EN: Implements `tryInterleave`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `tryInterleave`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-153
```cpp
  if (!isa<Instruction>(Start->getOperand(0)))
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 155-158
```cpp
  // Look for connected operations starting from Ext's, terminating at Truncs.
  std::vector<Instruction *> Worklist;
  Worklist.push_back(Start);
  Worklist.push_back(cast<Instruction>(Start->getOperand(0)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 160-164
```cpp
  SmallSetVector<Instruction *, 4> Truncs;
  SmallSetVector<Instruction *, 4> Reducts;
  SmallSetVector<Instruction *, 4> Exts;
  SmallSetVector<Use *, 4> OtherLeafs;
  SmallSetVector<Instruction *, 4> Ops;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 166-168
```cpp
  while (!Worklist.empty()) {
    Instruction *I = Worklist.back();
    Worklist.pop_back();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 170-177
```cpp
    switch (I->getOpcode()) {
    // Truncs
    case Instruction::Trunc:
    case Instruction::FPTrunc:
      if (!Truncs.insert(I))
        continue;
      Visited.insert(I);
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 179-188
```cpp
    // Extend leaves
    case Instruction::SExt:
    case Instruction::ZExt:
    case Instruction::FPExt:
      if (Exts.count(I))
        continue;
      for (auto *Use : I->users())
        Worklist.push_back(cast<Instruction>(Use));
      Exts.insert(I);
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 190-193
```cpp
    case Instruction::Call: {
      IntrinsicInst *II = dyn_cast<IntrinsicInst>(I);
      if (!II)
        return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 195-200
```cpp
      if (II->getIntrinsicID() == Intrinsic::vector_reduce_add) {
        if (!Reducts.insert(I))
          continue;
        Visited.insert(I);
        break;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 202-216
```cpp
      switch (II->getIntrinsicID()) {
      case Intrinsic::abs:
      case Intrinsic::smin:
      case Intrinsic::smax:
      case Intrinsic::umin:
      case Intrinsic::umax:
      case Intrinsic::sadd_sat:
      case Intrinsic::ssub_sat:
      case Intrinsic::uadd_sat:
      case Intrinsic::usub_sat:
      case Intrinsic::minnum:
      case Intrinsic::maxnum:
      case Intrinsic::fabs:
      case Intrinsic::fma:
      case Intrinsic::ceil:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 217-231
```cpp
      case Intrinsic::floor:
      case Intrinsic::rint:
      case Intrinsic::round:
      case Intrinsic::trunc:
        break;
      default:
        return false;
      }
      [[fallthrough]]; // Fall through to treating these like an operator below.
    }
    // Binary/tertiary ops
    case Instruction::Add:
    case Instruction::Sub:
    case Instruction::Mul:
    case Instruction::AShr:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-240
```cpp
    case Instruction::LShr:
    case Instruction::Shl:
    case Instruction::ICmp:
    case Instruction::FCmp:
    case Instruction::FAdd:
    case Instruction::FMul:
    case Instruction::Select:
      if (!Ops.insert(I))
        continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 242-249
```cpp
      for (Use &Op : I->operands()) {
        if (!isa<FixedVectorType>(Op->getType()))
          continue;
        if (isa<Instruction>(Op))
          Worklist.push_back(cast<Instruction>(&Op));
        else
          OtherLeafs.insert(&Op);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 251-253
```cpp
      for (auto *Use : I->users())
        Worklist.push_back(cast<Instruction>(Use));
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 255-259
```cpp
    case Instruction::ShuffleVector:
      // A shuffle of a splat is a splat.
      if (cast<ShuffleVectorInst>(I)->isZeroEltSplat())
        continue;
      [[fallthrough]];
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 261-265
```cpp
    default:
      LLVM_DEBUG(dbgs() << "  Unhandled instruction: " << *I << "\n");
      return false;
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 267-268
```cpp
  if (Exts.empty() && OtherLeafs.empty())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 270-284
```cpp
  LLVM_DEBUG({
    dbgs() << "Found group:\n  Exts:\n";
    for (auto *I : Exts)
      dbgs() << "  " << *I << "\n";
    dbgs() << "  Ops:\n";
    for (auto *I : Ops)
      dbgs() << "  " << *I << "\n";
    dbgs() << "  OtherLeafs:\n";
    for (auto *I : OtherLeafs)
      dbgs() << "  " << *I->get() << " of " << *I->getUser() << "\n";
    dbgs() << "  Truncs:\n";
    for (auto *I : Truncs)
      dbgs() << "  " << *I << "\n";
    dbgs() << "  Reducts:\n";
    for (auto *I : Reducts)
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 285-286
```cpp
      dbgs() << "  " << *I << "\n";
  });
```
- EN: Declares `dbgs`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `dbgs`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 288-291
```cpp
  assert((!Truncs.empty() || !Reducts.empty()) &&
         "Expected some truncs or reductions");
  if (Truncs.empty() && Exts.empty())
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 293-296
```cpp
  auto *VT = !Truncs.empty()
                 ? cast<FixedVectorType>(Truncs[0]->getType())
                 : cast<FixedVectorType>(Exts[0]->getOperand(0)->getType());
  LLVM_DEBUG(dbgs() << "Using VT:" << *VT << "\n");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 298-312
```cpp
  // Check types
  unsigned NumElts = VT->getNumElements();
  unsigned BaseElts = VT->getScalarSizeInBits() == 16
                          ? 8
                          : (VT->getScalarSizeInBits() == 8 ? 16 : 0);
  if (BaseElts == 0 || NumElts % BaseElts != 0) {
    LLVM_DEBUG(dbgs() << "  Type is unsupported\n");
    return false;
  }
  if (Start->getOperand(0)->getType()->getScalarSizeInBits() !=
      VT->getScalarSizeInBits() * 2) {
    LLVM_DEBUG(dbgs() << "  Type not double sized\n");
    return false;
  }
  for (Instruction *I : Exts)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 313-321
```cpp
    if (I->getOperand(0)->getType() != VT) {
      LLVM_DEBUG(dbgs() << "  Wrong type on " << *I << "\n");
      return false;
    }
  for (Instruction *I : Truncs)
    if (I->getType() != VT) {
      LLVM_DEBUG(dbgs() << "  Wrong type on " << *I << "\n");
      return false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 323-333
```cpp
  // Check that it looks beneficial
  if (!isProfitableToInterleave(Exts, Truncs))
    return false;
  if (!Reducts.empty() && (Ops.empty() || all_of(Ops, [](Instruction *I) {
                             return I->getOpcode() == Instruction::Mul ||
                                    I->getOpcode() == Instruction::Select ||
                                    I->getOpcode() == Instruction::ICmp;
                           }))) {
    LLVM_DEBUG(dbgs() << "Reduction does not look profitable\n");
    return false;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 335-336
```cpp
  // Create new shuffles around the extends / truncs / other leaves.
  IRBuilder<> Builder(Start);
```
- EN: Declares `Builder`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Builder`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 338-352
```cpp
  SmallVector<int, 16> LeafMask;
  SmallVector<int, 16> TruncMask;
  // LeafMask : 0, 2, 4, 6, 1, 3, 5, 7   8, 10, 12, 14,  9, 11, 13, 15
  // TruncMask: 0, 4, 1, 5, 2, 6, 3, 7   8, 12,  9, 13, 10, 14, 11, 15
  for (unsigned Base = 0; Base < NumElts; Base += BaseElts) {
    for (unsigned i = 0; i < BaseElts / 2; i++)
      LeafMask.push_back(Base + i * 2);
    for (unsigned i = 0; i < BaseElts / 2; i++)
      LeafMask.push_back(Base + i * 2 + 1);
  }
  for (unsigned Base = 0; Base < NumElts; Base += BaseElts) {
    for (unsigned i = 0; i < BaseElts / 2; i++) {
      TruncMask.push_back(Base + i);
      TruncMask.push_back(Base + i + BaseElts / 2);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 353-353
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 355-366
```cpp
  for (Instruction *I : Exts) {
    LLVM_DEBUG(dbgs() << "Replacing ext " << *I << "\n");
    Builder.SetInsertPoint(I);
    Value *Shuffle = Builder.CreateShuffleVector(I->getOperand(0), LeafMask);
    bool FPext = isa<FPExtInst>(I);
    bool Sext = isa<SExtInst>(I);
    Value *Ext = FPext ? Builder.CreateFPExt(Shuffle, I->getType())
                       : Sext ? Builder.CreateSExt(Shuffle, I->getType())
                              : Builder.CreateZExt(Shuffle, I->getType());
    I->replaceAllUsesWith(Ext);
    LLVM_DEBUG(dbgs() << "  with " << *Shuffle << "\n");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 368-374
```cpp
  for (Use *I : OtherLeafs) {
    LLVM_DEBUG(dbgs() << "Replacing leaf " << *I << "\n");
    Builder.SetInsertPoint(cast<Instruction>(I->getUser()));
    Value *Shuffle = Builder.CreateShuffleVector(I->get(), LeafMask);
    I->getUser()->setOperand(I->getOperandNo(), Shuffle);
    LLVM_DEBUG(dbgs() << "  with " << *Shuffle << "\n");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 376-377
```cpp
  for (Instruction *I : Truncs) {
    LLVM_DEBUG(dbgs() << "Replacing trunc " << *I << "\n");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 379-382
```cpp
    Builder.SetInsertPoint(I->getParent(), ++I->getIterator());
    Value *Shuf = Builder.CreateShuffleVector(I, TruncMask);
    I->replaceAllUsesWith(Shuf);
    cast<Instruction>(Shuf)->setOperand(0, I);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 384-385
```cpp
    LLVM_DEBUG(dbgs() << "  with " << *Shuf << "\n");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 387-388
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 390-396
```cpp
// Add reductions are fairly common and associative, meaning we can start the
// interleaving from them and don't need to emit a shuffle.
static bool isAddReduction(Instruction &I) {
  if (auto *II = dyn_cast<IntrinsicInst>(&I))
    return II->getIntrinsicID() == Intrinsic::vector_reduce_add;
  return false;
}
```
- EN: Implements `isAddReduction`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isAddReduction`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 398-405
```cpp
bool MVELaneInterleaving::runOnFunction(Function &F) {
  if (!EnableInterleave)
    return false;
  auto &TPC = getAnalysis<TargetPassConfig>();
  auto &TM = TPC.getTM<TargetMachine>();
  auto *ST = &TM.getSubtarget<ARMSubtarget>(F);
  if (!ST->hasMVEIntegerOps())
    return false;
```
- EN: Implements `MVELaneInterleaving::runOnFunction`, a pass-entry routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MVELaneInterleaving::runOnFunction`，它是一个围绕目标机器策略展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 407-407
```cpp
  bool Changed = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 409-416
```cpp
  SmallPtrSet<Instruction *, 16> Visited;
  for (Instruction &I : reverse(instructions(F))) {
    if (((I.getType()->isVectorTy() &&
          (isa<TruncInst>(I) || isa<FPTruncInst>(I))) ||
         isAddReduction(I)) &&
        !Visited.count(&I))
      Changed |= tryInterleave(&I, Visited);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 418-419
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMBaseInstrInfo.h`, `ARMSubtarget.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMBaseInstrInfo.h`, `ARMSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/SetVector.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h` ... (+11 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SetVector.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/TargetLowering.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h` ... (+11 more)。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
