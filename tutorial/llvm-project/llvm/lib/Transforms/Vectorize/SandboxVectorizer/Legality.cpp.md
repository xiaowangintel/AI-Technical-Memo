# Legality.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Legality.cpp` | `llvm/lib/Transforms/Vectorize/SandboxVectorizer/Legality.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Legality within LLVM's vectorization and VPlan infrastructure layer. | 该源文件实现了 LLVM 向量化与 VPlan 基础设施层中的 Legality 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-31

```cpp
//===- Legality.cpp -------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Vectorize/SandboxVectorizer/Legality.h"
#include "llvm/SandboxIR/Instruction.h"
#include "llvm/SandboxIR/Operator.h"
#include "llvm/SandboxIR/Utils.h"
#include "llvm/SandboxIR/Value.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/InstrMaps.h"
#include "llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h"

namespace llvm::sandboxir {

#ifndef NDEBUG
void ShuffleMask::dump() const {
  print(dbgs());
  dbgs() << "\n";
}

void LegalityResult::dump() const {
  print(dbgs());
  dbgs() << "\n";
}
#endif // NDEBUG

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include dump, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 dump，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 32-61

```cpp
std::optional<ResultReason>
LegalityAnalysis::notVectorizableBasedOnOpcodesAndTypes(
    ArrayRef<Value *> Bndl) {
  auto *I0 = cast<Instruction>(Bndl[0]);
  auto Opcode = I0->getOpcode();
  // If they have different opcodes, then we cannot form a vector (for now).
  if (any_of(drop_begin(Bndl), [Opcode](Value *V) {
        return cast<Instruction>(V)->getOpcode() != Opcode;
      }))
    return ResultReason::DiffOpcodes;

  // If not the same scalar type, Pack. This will accept scalars and vectors as
  // long as the element type is the same.
  Type *ElmTy0 = VecUtils::getElementType(Utils::getExpectedType(I0));
  if (any_of(drop_begin(Bndl), [ElmTy0](Value *V) {
        return VecUtils::getElementType(Utils::getExpectedType(V)) != ElmTy0;
      }))
    return ResultReason::DiffTypes;

  // TODO: Allow vectorization of instrs with different flags as long as we
  // change them to the least common one.
  // For now pack if differnt FastMathFlags.
  if (isa<FPMathOperator>(I0)) {
    FastMathFlags FMF0 = cast<Instruction>(Bndl[0])->getFastMathFlags();
    if (any_of(drop_begin(Bndl), [FMF0](auto *V) {
          return cast<Instruction>(V)->getFastMathFlags() != FMF0;
        }))
      return ResultReason::DiffMathFlags;
  }

```
- EN: This region continues the Legality implementation with local helper logic centered on ResultReason, LegalityAnalysis, ArrayRef, Value.
- CN: 这一段延续了 Legality 的主体实现，围绕 ResultReason, LegalityAnalysis, ArrayRef, Value 等局部辅助逻辑展开。

### Lines 62-88

```cpp
  // TODO: Allow vectorization by using common flags.
  // For now Pack if they don't have the same wrap flags.
  bool CanHaveWrapFlags =
      isa<OverflowingBinaryOperator>(I0) || isa<TruncInst>(I0);
  if (CanHaveWrapFlags) {
    bool NUW0 = I0->hasNoUnsignedWrap();
    bool NSW0 = I0->hasNoSignedWrap();
    if (any_of(drop_begin(Bndl), [NUW0, NSW0](auto *V) {
          return cast<Instruction>(V)->hasNoUnsignedWrap() != NUW0 ||
                 cast<Instruction>(V)->hasNoSignedWrap() != NSW0;
        })) {
      return ResultReason::DiffWrapFlags;
    }
  }

  // Now we need to do further checks for specific opcodes.
  switch (Opcode) {
  case Instruction::Opcode::ZExt:
  case Instruction::Opcode::SExt:
  case Instruction::Opcode::FPToUI:
  case Instruction::Opcode::FPToSI:
  case Instruction::Opcode::FPExt:
  case Instruction::Opcode::PtrToAddr:
  case Instruction::Opcode::PtrToInt:
  case Instruction::Opcode::IntToPtr:
  case Instruction::Opcode::SIToFP:
  case Instruction::Opcode::UIToFP:
```
- EN: This region continues the Legality implementation with local helper logic centered on TODO, Allow, For, Pack.
- CN: 这一段延续了 Legality 的主体实现，围绕 TODO, Allow, For, Pack 等局部辅助逻辑展开。

### Lines 89-115

```cpp
  case Instruction::Opcode::Trunc:
  case Instruction::Opcode::FPTrunc:
  case Instruction::Opcode::BitCast: {
    // We have already checked that they are of the same opcode.
    assert(all_of(Bndl,
                  [Opcode](Value *V) {
                    return cast<Instruction>(V)->getOpcode() == Opcode;
                  }) &&
           "Different opcodes, should have early returned!");
    // But for these opcodes we should also check the operand type.
    Type *FromTy0 = Utils::getExpectedType(I0->getOperand(0));
    if (any_of(drop_begin(Bndl), [FromTy0](Value *V) {
          return Utils::getExpectedType(cast<User>(V)->getOperand(0)) !=
                 FromTy0;
        }))
      return ResultReason::DiffTypes;
    return std::nullopt;
  }
  case Instruction::Opcode::FCmp:
  case Instruction::Opcode::ICmp: {
    // We need the same predicate and the same operand type.
    auto Pred0 = cast<CmpInst>(I0)->getPredicate();
    Type *Ty0 = cast<CmpInst>(I0)->getOperand(0)->getType();
    bool Same = all_of(Bndl, [Pred0, Ty0](Value *V) {
      auto *CmpI = cast<CmpInst>(V);
      return CmpI->getPredicate() == Pred0 &&
             CmpI->getOperand(0)->getType() == Ty0;
```
- EN: This region continues the Legality implementation with local helper logic centered on Instruction, Opcode, Trunc, FPTrunc.
- CN: 这一段延续了 Legality 的主体实现，围绕 Instruction, Opcode, Trunc, FPTrunc 等局部辅助逻辑展开。

### Lines 116-142

```cpp
    });
    if (Same)
      return std::nullopt;
    return ResultReason::DiffOpcodes;
  }
  case Instruction::Opcode::Select: {
    auto *Sel0 = cast<SelectInst>(Bndl[0]);
    auto *Cond0 = Sel0->getCondition();
    if (VecUtils::getNumLanes(Cond0) != VecUtils::getNumLanes(Sel0))
      // TODO: For now we don't vectorize if the lanes in the condition don't
      // match those of the select instruction.
      return ResultReason::Unimplemented;
    return std::nullopt;
  }
  case Instruction::Opcode::FNeg:
  case Instruction::Opcode::Add:
  case Instruction::Opcode::FAdd:
  case Instruction::Opcode::Sub:
  case Instruction::Opcode::FSub:
  case Instruction::Opcode::Mul:
  case Instruction::Opcode::FMul:
  case Instruction::Opcode::FRem:
  case Instruction::Opcode::UDiv:
  case Instruction::Opcode::SDiv:
  case Instruction::Opcode::FDiv:
  case Instruction::Opcode::URem:
  case Instruction::Opcode::SRem:
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 143-169

```cpp
  case Instruction::Opcode::Shl:
  case Instruction::Opcode::LShr:
  case Instruction::Opcode::AShr:
  case Instruction::Opcode::And:
  case Instruction::Opcode::Or:
  case Instruction::Opcode::Xor:
    return std::nullopt;
  case Instruction::Opcode::Load:
    if (VecUtils::areConsecutive<LoadInst>(Bndl, SE, DL))
      return std::nullopt;
    return ResultReason::NotConsecutive;
  case Instruction::Opcode::Store:
    if (VecUtils::areConsecutive<StoreInst>(Bndl, SE, DL))
      return std::nullopt;
    return ResultReason::NotConsecutive;
  case Instruction::Opcode::PHI:
    return ResultReason::Unimplemented;
  case Instruction::Opcode::Opaque:
    return ResultReason::Unimplemented;
  case Instruction::Opcode::UncondBr:
  case Instruction::Opcode::CondBr:
  case Instruction::Opcode::Ret:
  case Instruction::Opcode::AddrSpaceCast:
  case Instruction::Opcode::InsertElement:
  case Instruction::Opcode::InsertValue:
  case Instruction::Opcode::ExtractElement:
  case Instruction::Opcode::ExtractValue:
```
- EN: This region continues the Legality implementation with local helper logic centered on Instruction, Opcode, Shl, LShr.
- CN: 这一段延续了 Legality 的主体实现，围绕 Instruction, Opcode, Shl, LShr 等局部辅助逻辑展开。

### Lines 170-197

```cpp
  case Instruction::Opcode::ShuffleVector:
  case Instruction::Opcode::Call:
  case Instruction::Opcode::GetElementPtr:
  case Instruction::Opcode::Switch:
  case Instruction::Opcode::Pack:
    return ResultReason::Unimplemented;
  case Instruction::Opcode::VAArg:
  case Instruction::Opcode::Freeze:
  case Instruction::Opcode::Fence:
  case Instruction::Opcode::Invoke:
  case Instruction::Opcode::CallBr:
  case Instruction::Opcode::LandingPad:
  case Instruction::Opcode::CatchPad:
  case Instruction::Opcode::CleanupPad:
  case Instruction::Opcode::CatchRet:
  case Instruction::Opcode::CleanupRet:
  case Instruction::Opcode::Resume:
  case Instruction::Opcode::CatchSwitch:
  case Instruction::Opcode::AtomicRMW:
  case Instruction::Opcode::AtomicCmpXchg:
  case Instruction::Opcode::Alloca:
  case Instruction::Opcode::Unreachable:
    return ResultReason::Infeasible;
  }

  return std::nullopt;
}

```
- EN: This region continues the Legality implementation with local helper logic centered on Instruction, Opcode, ShuffleVector, Call.
- CN: 这一段延续了 Legality 的主体实现，围绕 Instruction, Opcode, ShuffleVector, Call 等局部辅助逻辑展开。

### Lines 198-228

```cpp
CollectDescr
LegalityAnalysis::getHowToCollectValues(ArrayRef<Value *> Bndl) const {
  SmallVector<CollectDescr::ExtractElementDescr, 4> Vec;
  Vec.reserve(Bndl.size());
  for (auto [Elm, V] : enumerate(Bndl)) {
    if (auto *VecOp = IMaps.getVectorForOrig(V)) {
      // If there is a vector containing `V`, then get the lane it came from.
      std::optional<int> ExtractIdxOpt = IMaps.getOrigLane(VecOp, V);
      // This could be a vector, like <2 x float> in which case the mask needs
      // to enumerate all lanes.
      for (unsigned Ln = 0, Lanes = VecUtils::getNumLanes(V); Ln != Lanes; ++Ln)
        Vec.emplace_back(VecOp, ExtractIdxOpt ? *ExtractIdxOpt + Ln : -1);
    } else {
      Vec.emplace_back(V);
    }
  }
  return CollectDescr(std::move(Vec));
}

const LegalityResult &LegalityAnalysis::canVectorize(ArrayRef<Value *> Bndl,
                                                     bool SkipScheduling) {
  // If Bndl contains values other than instructions, we need to Pack.
  if (any_of(Bndl, [](auto *V) { return !isa<Instruction>(V); }))
    return createLegalityResult<Pack>(ResultReason::NotInstructions);
  // Pack if not in the same BB.
  if (LegalityAnalysis::differentBlock(Bndl))
    return createLegalityResult<Pack>(ResultReason::DiffBBs);
  // Pack if instructions repeat, i.e., require some sort of broadcast.
  if (!LegalityAnalysis::areUnique(Bndl))
    return createLegalityResult<Pack>(ResultReason::RepeatedInstrs);

```
- EN: Core entities appearing here include getHowToCollectValues, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getHowToCollectValues，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 229-256

```cpp
  auto CollectDescrs = getHowToCollectValues(Bndl);
  if (CollectDescrs.hasVectorInputs()) {
    if (auto ValueShuffleOpt = CollectDescrs.getSingleInput()) {
      auto [Vec, Mask] = *ValueShuffleOpt;
      if (Mask.isIdentity())
        return createLegalityResult<DiamondReuse>(Vec);
      return createLegalityResult<DiamondReuseWithShuffle>(Vec, Mask);
    }
    return createLegalityResult<DiamondReuseMultiInput>(
        std::move(CollectDescrs));
  }

  if (auto ReasonOpt = notVectorizableBasedOnOpcodesAndTypes(Bndl))
    return createLegalityResult<Pack>(*ReasonOpt);

  if (!SkipScheduling) {
    // TODO: Try to remove the IBndl vector.
    SmallVector<Instruction *, 8> IBndl;
    IBndl.reserve(Bndl.size());
    for (auto *V : Bndl)
      IBndl.push_back(cast<Instruction>(V));
    if (!Sched.trySchedule(IBndl))
      return createLegalityResult<Pack>(ResultReason::CantSchedule);
  }

  return createLegalityResult<Widen>();
}

```
- EN: This region continues the Legality implementation with local helper logic centered on CollectDescrs, Bndl, ValueShuffleOpt, Vec.
- CN: 这一段延续了 Legality 的主体实现，围绕 CollectDescrs, Bndl, ValueShuffleOpt, Vec 等局部辅助逻辑展开。

### Lines 257-261

```cpp
void LegalityAnalysis::clear() {
  Sched.clear();
  IMaps.clear();
}
} // namespace llvm::sandboxir
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include clear, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 clear，它们承载了该区间涉及的局部状态、声明或变换入口。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `dump, getHowToCollectValues, clear` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`dump, getHowToCollectValues, clear` 构成该文件对外 API 与主要实现挂钩。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/Transforms/Vectorize/SandboxVectorizer/Legality.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/InstrMaps.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/Transforms/Vectorize/SandboxVectorizer/Legality.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/InstrMaps.h`, `llvm/Transforms/Vectorize/SandboxVectorizer/VecUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Operator.h`, `llvm/SandboxIR/Utils.h`, `llvm/SandboxIR/Value.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/SandboxIR/Instruction.h`, `llvm/SandboxIR/Operator.h`, `llvm/SandboxIR/Utils.h`, `llvm/SandboxIR/Value.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
