# CallPromotionUtils.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CallPromotionUtils.cpp` | `llvm/lib/Transforms/Utils/CallPromotionUtils.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements utilities for call promotion within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CallPromotionUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-65

```cpp
//===- CallPromotionUtils.cpp - Utilities for call promotion ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements utilities useful for promoting indirect call sites to
// direct call sites.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/CallPromotionUtils.h"
#include "llvm/Analysis/CtxProfAnalysis.h"
#include "llvm/Analysis/Loads.h"
#include "llvm/Analysis/TypeMetadataUtils.h"
#include "llvm/IR/AttributeMask.h"
#include "llvm/IR/Constant.h"
#include "llvm/IR/IRBuilder.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
///
///    normal_dst:
///      %t3 = phi i32 [ %x, %merge_bb ], ...
///
static void fixupPHINodeForNormalDest(InvokeInst *Invoke, BasicBlock *OrigBlock,
                                      BasicBlock *MergeBlock) {
  for (PHINode &Phi : Invoke->getNormalDest()->phis()) {
    int Idx = Phi.getBasicBlockIndex(OrigBlock);
    if (Idx == -1)
      continue;
    Phi.setIncomingBlock(Idx, MergeBlock);
  }
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 66-127

```cpp
/// Fix-up phi nodes in an invoke instruction's unwind destination.
///
/// After versioning an invoke instruction, values coming from the original
/// block will now be coming from either the "then" block or the "else" block.
/// For example, in the code below:
///
///   then_bb:
///     %t0 = invoke i32 %ptr() to label %merge_bb unwind label %unwind_dst
///
///   else_bb:
///     %t1 = invoke i32 %ptr() to label %merge_bb unwind label %unwind_dst
///
///   unwind_dst:
///     %t3 = phi i32 [ %x, %orig_bb ], ...
///
/// "orig_bb" is no longer a predecessor of "unwind_dst", so the phi nodes in
/// "unwind_dst" must be fixed to refer to "then_bb" and "else_bb":
///
///   unwind_dst:
///     %t3 = phi i32 [ %x, %then_bb ], [ %x, %else_bb ], ...
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                             BasicBlock *MergeBlock, IRBuilder<> &Builder) {

  if (OrigInst->getType()->isVoidTy() || OrigInst->use_empty())
    return;

  Builder.SetInsertPoint(MergeBlock, MergeBlock->begin());
  PHINode *Phi = Builder.CreatePHI(OrigInst->getType(), 0);
  SmallVector<User *, 16> UsersToUpdate(OrigInst->users());
  for (User *U : UsersToUpdate)
    U->replaceUsesOfWith(OrigInst, Phi);
  Phi->addIncoming(OrigInst, OrigInst->getParent());
  Phi->addIncoming(NewInst, NewInst->getParent());
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 128-192

```cpp
/// Cast a call or invoke instruction to the given type.
///
/// When promoting a call site, the return type of the call site might not match
/// that of the callee. If this is the case, we have to cast the returned value
/// to the correct type. The location of the cast depends on if we have a call
/// or invoke instruction.
///
/// For example, if the call instruction below requires a bitcast after
/// promotion:
///
///   orig_bb:
///     %t0 = call i32 @func()
///     ...
///
/// The bitcast is placed after the call instruction:
///
///   orig_bb:
///     ; Uses of the original return value are replaced by uses of the bitcast.
///     %t0 = call i32 @func()
///     %t1 = bitcast i32 %t0 to ...
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        SplitEdge(Invoke->getParent(), Invoke->getNormalDest())->begin();
  else
    InsertBefore = std::next(CB.getIterator());

  // Bitcast the return value to the correct type.
  auto *Cast = CastInst::CreateBitOrPointerCast(&CB, RetTy, "", InsertBefore);
  if (RetBitCast)
    *RetBitCast = Cast;

  // Replace all the original uses of the calling instruction with the bitcast.
  for (User *U : UsersToUpdate)
    U->replaceUsesOfWith(&CB, Cast);
}

```
- EN: Core entities appearing here include createRetBitCast, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createRetBitCast，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 193-256

```cpp
/// Predicate and clone the given call site.
///
/// This function creates an if-then-else structure at the location of the call
/// site. The "if" condition is specified by `Cond`.
/// The original call site is moved into the "else" block, and a clone of the
/// call site is placed in the "then" block. The cloned instruction is returned.
///
/// For example, the call instruction below:
///
///   orig_bb:
///     %t0 = call i32 %ptr()
///     ...
///
/// Is replace by the following:
///
///   orig_bb:
///     %cond = Cond
///     br i1 %cond, %then_bb, %else_bb
///
///   then_bb:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
///     ; The clone of the original invoke instruction is placed in the "then"
///     ; block, and its normal destination is set to the "merge" block. It is
///     ; not yet promoted.
///     %t1 = invoke i32 %ptr() to label %merge_bb unwind label %unwind_dst
///
///   else_bb:
///     ; The original invoke instruction is moved into the "else" block, and
///     ; its normal destination is set to the "merge" block.
///     %t0 = invoke i32 %ptr() to label %merge_bb unwind label %unwind_dst
///
///   merge_bb:
///     ; Uses of the original invoke instruction are replaced by uses of the
///     ; phi node, and the merge block branches to the normal destination.
///     %t2 = phi i32 [ %t0, %else_bb ], [ %t1, %then_bb ]
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 257-324

```cpp
///     br %normal_dst
///
/// An indirect musttail call is processed slightly differently in that:
/// 1. No merge block needed for the orginal and the cloned callsite, since
///    either one ends the flow. No phi node is needed either.
/// 2. The return statement following the original call site is duplicated too
///    and placed immediately after the cloned call site per the IR convention.
///
/// For example, the musttail call instruction below:
///
///   orig_bb:
///     %t0 = musttail call i32 %ptr()
///     ...
///
/// Is replaced by the following:
///
///   cond_bb:
///     %cond = Cond
///     br i1 %cond, %then_bb, %orig_bb
///
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      NewBitCast->replaceUsesOfWith(OrigInst, NewInst);
      NewBitCast->insertBefore(ThenTerm->getIterator());
      NewRetVal = NewBitCast;
      Next = BitCast->getNextNode();
    }

    // Place a clone of the return instruction after the new call site.
    ReturnInst *Ret = dyn_cast_or_null<ReturnInst>(Next);
    assert(Ret && "musttail call must precede a ret with an optional bitcast");
    auto NewRet = Ret->clone();
    if (Ret->getReturnValue())
      NewRet->replaceUsesOfWith(Ret->getReturnValue(), NewRetVal);
    NewRet->insertBefore(ThenTerm->getIterator());

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 325-393

```cpp
    // A return instructions is terminating, so we don't need the terminator
    // instruction just created.
    ThenTerm->eraseFromParent();

    return *NewInst;
  }

  // Create an if-then-else structure. The original instruction is moved into
  // the "else" block, and a clone of the original instruction is placed in the
  // "then" block.
  Instruction *ThenTerm = nullptr;
  Instruction *ElseTerm = nullptr;
  SplitBlockAndInsertIfThenElse(Cond, &CB, &ThenTerm, &ElseTerm, BranchWeights);
  BasicBlock *ThenBlock = ThenTerm->getParent();
  BasicBlock *ElseBlock = ElseTerm->getParent();
  BasicBlock *MergeBlock = OrigInst->getParent();

  ThenBlock->setName("if.true.direct_targ");
  ElseBlock->setName("if.false.orig_indirect");
  MergeBlock->setName("if.end.icp");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

// Predicate and clone the given call site using condition `CB.callee ==
// Callee`. See the comment `versionCallSiteWithCond` for the transformation.
CallBase &llvm::versionCallSite(CallBase &CB, Value *Callee,
                                MDNode *BranchWeights) {

  IRBuilder<> Builder(&CB);

  // Create the compare. The called value and callee must have the same type to
  // be compared.
  if (CB.getCalledOperand()->getType() != Callee->getType())
    Callee = Builder.CreateBitCast(Callee, CB.getCalledOperand()->getType());
  auto *Cond = Builder.CreateICmpEQ(CB.getCalledOperand(), Callee);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 394-457

```cpp
  return versionCallSiteWithCond(CB, Cond, BranchWeights);
}

bool llvm::isLegalToPromote(const CallBase &CB, Function *Callee,
                            const char **FailureReason) {
  assert(!CB.getCalledFunction() && "Only indirect call sites can be promoted");

  auto &DL = Callee->getDataLayout();

  // Check the return type. The callee's return value type must be bitcast
  // compatible with the call site's type.
  Type *CallRetTy = CB.getType();
  Type *FuncRetTy = Callee->getReturnType();
  if (CallRetTy != FuncRetTy)
    if (!CastInst::isBitOrNoopPointerCastable(FuncRetTy, CallRetTy, DL)) {
      if (FailureReason)
        *FailureReason = "Return type mismatch";
      return false;
    }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        *FailureReason = "inalloca mismatch";
      return false;
    }

    Type *FormalTy = Callee->getFunctionType()->getFunctionParamType(I);
    Type *ActualTy = CB.getArgOperand(I)->getType();
    if (FormalTy == ActualTy)
      continue;
    if (!CastInst::isBitOrNoopPointerCastable(ActualTy, FormalTy, DL)) {
      if (FailureReason)
        *FailureReason = "Argument type mismatch";
      return false;
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 458-522

```cpp
    // MustTail call needs stricter type match. See
    // Verifier::verifyMustTailCall().
    if (CB.isMustTailCall()) {
      PointerType *PF = dyn_cast<PointerType>(FormalTy);
      PointerType *PA = dyn_cast<PointerType>(ActualTy);
      if (!PF || !PA || PF->getAddressSpace() != PA->getAddressSpace()) {
        if (FailureReason)
          *FailureReason = "Musttail call Argument type mismatch";
        return false;
      }
    }
  }
  for (; I < NumArgs; I++) {
    // Vararg functions can have more arguments than parameters.
    assert(Callee->isVarArg());
    if (CB.paramHasAttr(I, Attribute::StructRet)) {
      if (FailureReason)
        *FailureReason = "SRet arg to vararg function";
      return false;
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // change the type).
  CB.setCalledOperand(Callee);

  // Since the call site will no longer be direct, we must clear metadata that
  // is only appropriate for indirect calls. This includes !prof and !callees
  // metadata.
  CB.setMetadata(LLVMContext::MD_prof, nullptr);
  CB.setMetadata(LLVMContext::MD_callees, nullptr);

  // If the function type of the call site matches that of the callee, no
  // additional work is required.
  if (CB.getFunctionType() == Callee->getFunctionType())
    return CB;

```
- EN: This region continues the CallPromotionUtils implementation with local helper logic centered on MustTail, See, Verifier, PointerType.
- CN: 这一段延续了 CallPromotionUtils 的主体实现，围绕 MustTail, See, Verifier, PointerType 等局部辅助逻辑展开。

### Lines 523-587

```cpp
  // Save the return types of the call site and callee.
  Type *CallSiteRetTy = CB.getType();
  Type *CalleeRetTy = Callee->getReturnType();

  // Change the function type of the call site the match that of the callee.
  CB.mutateFunctionType(Callee->getFunctionType());

  // Inspect the arguments of the call site. If an argument's type doesn't
  // match the corresponding formal argument's type in the callee, bitcast it
  // to the correct type.
  auto CalleeType = Callee->getFunctionType();
  auto CalleeParamNum = CalleeType->getNumParams();

  LLVMContext &Ctx = Callee->getContext();
  const AttributeList &CallerPAL = CB.getAttributes();
  // The new list of argument attributes.
  SmallVector<AttributeSet, 4> NewArgAttrs;
  bool AttributeChanged = false;

  for (unsigned ArgNo = 0; ArgNo < CalleeParamNum; ++ArgNo) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    RAttrs.remove(
        AttributeFuncs::typeIncompatible(CalleeRetTy, CallerPAL.getRetAttrs()));
    AttributeChanged = true;
  }

  // Set the new callsite attribute.
  if (AttributeChanged)
    CB.setAttributes(AttributeList::get(Ctx, CallerPAL.getFnAttrs(),
                                        AttributeSet::get(Ctx, RAttrs),
                                        NewArgAttrs));

  return CB;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 588-655

```cpp
CallBase &llvm::promoteCallWithIfThenElse(CallBase &CB, Function *Callee,
                                          MDNode *BranchWeights) {

  // Version the indirect call site. If the called value is equal to the given
  // callee, 'NewInst' will be executed, otherwise the original call site will
  // be executed.
  CallBase &NewInst = versionCallSite(CB, Callee, BranchWeights);

  // Promote 'NewInst' so that it directly calls the desired function.
  return promoteCall(NewInst, Callee);
}

CallBase *llvm::promoteCallWithIfThenElse(CallBase &CB, Function &Callee,
                                          PGOContextualProfile &CtxProf) {
  assert(CB.isIndirectCall());
  if (!CtxProf.isFunctionKnown(Callee))
    return nullptr;
  auto &Caller = *CB.getFunction();
  auto *CSInstr = CtxProfAnalysis::getCallsiteInstrumentation(CB);
  if (!CSInstr)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  auto ProfileUpdater = [&](PGOCtxProfContext &Ctx) {
    assert(Ctx.guid() == AssignGUIDPass::getGUID(Caller));
    assert(NewCountersSize - 2 == Ctx.counters().size());
    // All the ctx-es belonging to a function must have the same size counters.
    Ctx.resizeCounters(NewCountersSize);

    // Maybe in this context, the indirect callsite wasn't observed at all. That
    // would make both direct and indirect BBs cold - which is what we already
    // have from resising the counters.
    if (!Ctx.hasCallsite(CSIndex))
      return;
    auto &CSData = Ctx.callsite(CSIndex);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 656-729

```cpp
    uint64_t TotalCount = 0;
    for (const auto &[_, V] : CSData)
      TotalCount += V.getEntrycount();
    uint64_t DirectCount = 0;
    // If we called the direct target, update the DirectCount. If we didn't, we
    // still want to update the indirect BB (to which the TotalCount goes, in
    // that case).
    if (auto It = CSData.find(CalleeGUID); It != CSData.end()) {
      assert(CalleeGUID == It->second.guid());
      DirectCount = It->second.getEntrycount();
      // This direct target needs to be moved to this caller under the
      // newly-allocated callsite index.
      assert(Ctx.callsites().count(NewCSID) == 0);
      Ctx.ingestContext(NewCSID, std::move(It->second));
      CSData.erase(CalleeGUID);
    }

    assert(TotalCount >= DirectCount);
    uint64_t IndirectCount = TotalCount - DirectCount;
    // The ICP's effect is as-if the direct BB would have been taken DirectCount
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  APInt VTableOffset(DL.getIndexTypeSizeInBits(VTableEntryPtr->getType()), 0);
  Value *VTableBasePtr = VTableEntryPtr->stripAndAccumulateConstantOffsets(
      DL, VTableOffset, /* AllowNonInbounds */ true);
  LoadInst *VTablePtrLoad = dyn_cast<LoadInst>(VTableBasePtr);
  if (!VTablePtrLoad)
    return false; // Not a vtable load.
  Value *Object = VTablePtrLoad->getPointerOperand();
  APInt ObjectOffset(DL.getIndexTypeSizeInBits(Object->getType()), 0);
  Value *ObjectBase = Object->stripAndAccumulateConstantOffsets(
      DL, ObjectOffset, /* AllowNonInbounds */ true);
  if (!(isa<AllocaInst>(ObjectBase) && ObjectOffset == 0))
    // Not an Alloca or the offset isn't zero.
    return false;

```
- EN: Core entities appearing here include tryPromoteCall, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 tryPromoteCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 730-762

```cpp
  // Look for the vtable pointer store into the object by the ctor.
  BasicBlock::iterator BBI(VTablePtrLoad);
  Value *VTablePtr = FindAvailableLoadedValue(
      VTablePtrLoad, VTablePtrLoad->getParent(), BBI, 0, nullptr, nullptr);
  if (!VTablePtr || !VTablePtr->getType()->isPointerTy())
    return false; // No vtable found.
  APInt VTableOffsetGVBase(DL.getIndexTypeSizeInBits(VTablePtr->getType()), 0);
  Value *VTableGVBase = VTablePtr->stripAndAccumulateConstantOffsets(
      DL, VTableOffsetGVBase, /* AllowNonInbounds */ true);
  GlobalVariable *GV = dyn_cast<GlobalVariable>(VTableGVBase);
  if (!(GV && GV->isConstant() && GV->hasDefinitiveInitializer()))
    // Not in the form of a global constant variable with an initializer.
    return false;

  APInt VTableGVOffset = VTableOffsetGVBase + VTableOffset;
  if (!(VTableGVOffset.getActiveBits() <= 64))
    return false; // Out of range.

  Function *DirectCallee = nullptr;
  std::tie(DirectCallee, std::ignore) =
      getFunctionAtVTableOffset(GV, VTableGVOffset.getZExtValue(), *M);
  if (!DirectCallee)
    return false; // No function pointer found.

  if (!isLegalToPromote(CB, DirectCallee))
    return false;

  // Success.
  promoteCall(CB, DirectCallee);
  return true;
}

#undef DEBUG_TYPE
```
- EN: This region continues the CallPromotionUtils implementation with local helper logic centered on Look, BasicBlock, BBI, VTablePtrLoad.
- CN: 这一段延续了 CallPromotionUtils 的主体实现，围绕 Look, BasicBlock, BBI, VTablePtrLoad 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `createRetBitCast, tryPromoteCall` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`createRetBitCast, tryPromoteCall` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/CtxProfAnalysis.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/TypeMetadataUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/CtxProfAnalysis.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/TypeMetadataUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/AttributeMask.h`, `llvm/IR/Constant.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/CallPromotionUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/AttributeMask.h`, `llvm/IR/Constant.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/CallPromotionUtils.h`, `llvm/Transforms/Utils/BasicBlockUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ProfileData/PGOCtxProfReader.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ProfileData/PGOCtxProfReader.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
