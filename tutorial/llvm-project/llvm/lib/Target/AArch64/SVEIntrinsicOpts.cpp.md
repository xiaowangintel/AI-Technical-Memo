# SVEIntrinsicOpts.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/SVEIntrinsicOpts.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements SVE vector support for the AArch64 backend. / 该文件实现 AArch64 后端中的SVE 向量支持。
## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Documented code section
```cpp
//===----- SVEIntrinsicOpts - SVE ACLE Intrinsics Opts --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Performs general IR level optimizations on SVE intrinsics.
//
// This pass performs the following optimizations:
//
// - removes unnecessary ptrue intrinsics (llvm.aarch64.sve.ptrue), e.g:
//     %1 = @llvm.aarch64.sve.ptrue.nxv4i1(i32 31)
//     %2 = @llvm.aarch64.sve.ptrue.nxv8i1(i32 31)
//     ; (%1 can be replaced with a reinterpret of %2)
//
// - optimizes ptest intrinsics where the operands are being needlessly
//   converted to and from svbool_t.
//
//===----------------------------------------------------------------------===//
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 22-50: Includes and compile-time dependencies
```cpp

#include "AArch64.h"
#include "Utils/AArch64BaseInfo.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Dominators.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/IntrinsicInst.h"
#include "llvm/IR/IntrinsicsAArch64.h"
#include "llvm/IR/LLVMContext.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PatternMatch.h"
#include "llvm/InitializePasses.h"
#include <optional>

using namespace llvm;
using namespace llvm::PatternMatch;

#define DEBUG_TYPE "aarch64-sve-intrinsic-opts"

namespace {
struct SVEIntrinsicOpts : public ModulePass {
  static char ID; // Pass identification, replacement for typeid
  SVEIntrinsicOpts() : ModulePass(ID) {}

  bool runOnModule(Module &M) override;
  void getAnalysisUsage(AnalysisUsage &AU) const override;
```
**EN:** This block pulls in the target-specific headers and LLVM infrastructure needed by the rest of the file.  
**CN:** 该代码块引入后续实现所需的目标专用头文件与 LLVM 基础设施。
### Lines 51-80: Core AArch64 backend logic
```cpp

private:
  bool coalescePTrueIntrinsicCalls(BasicBlock &BB,
                                   SmallSetVector<IntrinsicInst *, 4> &PTrues);
  bool optimizePTrueIntrinsicCalls(SmallSetVector<Function *, 4> &Functions);
  bool optimizePredicateStore(Instruction *I);
  bool optimizePredicateLoad(Instruction *I);

  bool optimizeInstructions(SmallSetVector<Function *, 4> &Functions);

  /// Operates at the function-scope. I.e., optimizations are applied local to
  /// the functions themselves.
  bool optimizeFunctions(SmallSetVector<Function *, 4> &Functions);
};
} // end anonymous namespace

void SVEIntrinsicOpts::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<DominatorTreeWrapperPass>();
  AU.setPreservesCFG();
}

char SVEIntrinsicOpts::ID = 0;
static const char *name = "SVE intrinsics optimizations";
INITIALIZE_PASS_BEGIN(SVEIntrinsicOpts, DEBUG_TYPE, name, false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass);
INITIALIZE_PASS_END(SVEIntrinsicOpts, DEBUG_TYPE, name, false, false)

ModulePass *llvm::createSVEIntrinsicOptsPass() {
  return new SVEIntrinsicOpts();
}
```
**EN:** This block continues the file's main SVE vector support logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的SVE 向量支持主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 81-106: Documented code section
```cpp

/// Checks if a ptrue intrinsic call is promoted. The act of promoting a
/// ptrue will introduce zeroing. For example:
///
///     %1 = <vscale x 4 x i1> call @llvm.aarch64.sve.ptrue.nxv4i1(i32 31)
///     %2 = <vscale x 16 x i1> call @llvm.aarch64.sve.convert.to.svbool.nxv4i1(<vscale x 4 x i1> %1)
///     %3 = <vscale x 8 x i1> call @llvm.aarch64.sve.convert.from.svbool.nxv8i1(<vscale x 16 x i1> %2)
///
/// %1 is promoted, because it is converted:
///
///     <vscale x 4 x i1> => <vscale x 16 x i1> => <vscale x 8 x i1>
///
/// via a sequence of the SVE reinterpret intrinsics convert.{to,from}.svbool.
static bool isPTruePromoted(IntrinsicInst *PTrue) {
  // Find all users of this intrinsic that are calls to convert-to-svbool
  // reinterpret intrinsics.
  SmallVector<IntrinsicInst *, 4> ConvertToUses;
  for (User *User : PTrue->users()) {
    if (match(User, m_Intrinsic<Intrinsic::aarch64_sve_convert_to_svbool>())) {
      ConvertToUses.push_back(cast<IntrinsicInst>(User));
    }
  }

  // If no such calls were found, this is ptrue is not promoted.
  if (ConvertToUses.empty())
    return false;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 107-136: Documented code section
```cpp

  // Otherwise, try to find users of the convert-to-svbool intrinsics that are
  // calls to the convert-from-svbool intrinsic, and would result in some lanes
  // being zeroed.
  const auto *PTrueVTy = cast<ScalableVectorType>(PTrue->getType());
  for (IntrinsicInst *ConvertToUse : ConvertToUses) {
    for (User *User : ConvertToUse->users()) {
      auto *IntrUser = dyn_cast<IntrinsicInst>(User);
      if (IntrUser && IntrUser->getIntrinsicID() ==
                          Intrinsic::aarch64_sve_convert_from_svbool) {
        const auto *IntrUserVTy = cast<ScalableVectorType>(IntrUser->getType());

        // Would some lanes become zeroed by the conversion?
        if (IntrUserVTy->getElementCount().getKnownMinValue() >
            PTrueVTy->getElementCount().getKnownMinValue())
          // This is a promoted ptrue.
          return true;
      }
    }
  }

  // If no matching calls were found, this is not a promoted ptrue.
  return false;
}

/// Attempts to coalesce ptrues in a basic block.
bool SVEIntrinsicOpts::coalescePTrueIntrinsicCalls(
    BasicBlock &BB, SmallSetVector<IntrinsicInst *, 4> &PTrues) {
  if (PTrues.size() <= 1)
    return false;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 137-169: Function llvm::max_element
```cpp

  // Find the ptrue with the most lanes.
  auto *MostEncompassingPTrue =
      *llvm::max_element(PTrues, [](auto *PTrue1, auto *PTrue2) {
        auto *PTrue1VTy = cast<ScalableVectorType>(PTrue1->getType());
        auto *PTrue2VTy = cast<ScalableVectorType>(PTrue2->getType());
        return PTrue1VTy->getElementCount().getKnownMinValue() <
               PTrue2VTy->getElementCount().getKnownMinValue();
      });

  // Remove the most encompassing ptrue, as well as any promoted ptrues, leaving
  // behind only the ptrues to be coalesced.
  PTrues.remove(MostEncompassingPTrue);
  PTrues.remove_if(isPTruePromoted);

  // Hoist MostEncompassingPTrue to the start of the basic block. It is always
  // safe to do this, since ptrue intrinsic calls are guaranteed to have no
  // predecessors.
  MostEncompassingPTrue->moveBefore(BB, BB.getFirstInsertionPt());

  LLVMContext &Ctx = BB.getContext();
  IRBuilder<> Builder(Ctx);
  Builder.SetInsertPoint(&BB, ++MostEncompassingPTrue->getIterator());

  auto *MostEncompassingPTrueVTy =
      cast<VectorType>(MostEncompassingPTrue->getType());
  auto *ConvertToSVBool = Builder.CreateIntrinsic(
      Intrinsic::aarch64_sve_convert_to_svbool, {MostEncompassingPTrueVTy},
      {MostEncompassingPTrue});

  bool ConvertFromCreated = false;
  for (auto *PTrue : PTrues) {
    auto *PTrueVTy = cast<VectorType>(PTrue->getType());
```
**EN:** This block implements llvm::max_element, advancing the file's SVE vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 llvm::max_element，通过 AArch64 专用的决策与数据处理推进本文件的SVE 向量支持流程。
### Lines 170-192: Documented code section
```cpp

    // Only create the converts if the types are not already the same, otherwise
    // just use the most encompassing ptrue.
    if (MostEncompassingPTrueVTy != PTrueVTy) {
      ConvertFromCreated = true;

      Builder.SetInsertPoint(&BB, ++ConvertToSVBool->getIterator());
      auto *ConvertFromSVBool =
          Builder.CreateIntrinsic(Intrinsic::aarch64_sve_convert_from_svbool,
                                  {PTrueVTy}, {ConvertToSVBool});
      PTrue->replaceAllUsesWith(ConvertFromSVBool);
    } else
      PTrue->replaceAllUsesWith(MostEncompassingPTrue);

    PTrue->eraseFromParent();
  }

  // We never used the ConvertTo so remove it
  if (!ConvertFromCreated)
    ConvertToSVBool->eraseFromParent();

  return true;
}
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 193-228: Documented code section
```cpp

/// The goal of this function is to remove redundant calls to the SVE ptrue
/// intrinsic in each basic block within the given functions.
///
/// SVE ptrues have two representations in LLVM IR:
/// - a logical representation -- an arbitrary-width scalable vector of i1s,
///   i.e. <vscale x N x i1>.
/// - a physical representation (svbool, <vscale x 16 x i1>) -- a 16-element
///   scalable vector of i1s, i.e. <vscale x 16 x i1>.
///
/// The SVE ptrue intrinsic is used to create a logical representation of an SVE
/// predicate. Suppose that we have two SVE ptrue intrinsic calls: P1 and P2. If
/// P1 creates a logical SVE predicate that is at least as wide as the logical
/// SVE predicate created by P2, then all of the bits that are true in the
/// physical representation of P2 are necessarily also true in the physical
/// representation of P1. P1 'encompasses' P2, therefore, the intrinsic call to
/// P2 is redundant and can be replaced by an SVE reinterpret of P1 via
/// convert.{to,from}.svbool.
///
/// Currently, this pass only coalesces calls to SVE ptrue intrinsics
/// if they match the following conditions:
///
/// - the call to the intrinsic uses either the SV_ALL or SV_POW2 patterns.
///   SV_ALL indicates that all bits of the predicate vector are to be set to
///   true. SV_POW2 indicates that all bits of the predicate vector up to the
///   largest power-of-two are to be set to true.
/// - the result of the call to the intrinsic is not promoted to a wider
///   predicate. In this case, keeping the extra ptrue leads to better codegen
///   -- coalescing here would create an irreducible chain of SVE reinterprets
///   via convert.{to,from}.svbool.
///
/// EXAMPLE:
///
///     %1 = <vscale x 8 x i1> ptrue(i32 SV_ALL)
///     ; Logical:  <1, 1, 1, 1, 1, 1, 1, 1>
///     ; Physical: <1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0>
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 229-261: Documented code section
```cpp
///     ...
///
///     %2 = <vscale x 4 x i1> ptrue(i32 SV_ALL)
///     ; Logical:  <1, 1, 1, 1>
///     ; Physical: <1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0>
///     ...
///
/// Here, %2 can be replaced by an SVE reinterpret of %1, giving, for instance:
///
///     %1 = <vscale x 8 x i1> ptrue(i32 i31)
///     %2 = <vscale x 16 x i1> convert.to.svbool(<vscale x 8 x i1> %1)
///     %3 = <vscale x 4 x i1> convert.from.svbool(<vscale x 16 x i1> %2)
///
bool SVEIntrinsicOpts::optimizePTrueIntrinsicCalls(
    SmallSetVector<Function *, 4> &Functions) {
  bool Changed = false;

  for (auto *F : Functions) {
    for (auto &BB : *F) {
      SmallSetVector<IntrinsicInst *, 4> SVAllPTrues;
      SmallSetVector<IntrinsicInst *, 4> SVPow2PTrues;

      // For each basic block, collect the used ptrues and try to coalesce them.
      for (Instruction &I : BB) {
        if (I.use_empty())
          continue;

        auto *IntrI = dyn_cast<IntrinsicInst>(&I);
        if (!IntrI || IntrI->getIntrinsicID() != Intrinsic::aarch64_sve_ptrue)
          continue;

        const auto PTruePattern =
            cast<ConstantInt>(IntrI->getOperand(0))->getZExtValue();
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 262-294: Core AArch64 backend logic
```cpp

        if (PTruePattern == AArch64SVEPredPattern::all)
          SVAllPTrues.insert(IntrI);
        if (PTruePattern == AArch64SVEPredPattern::pow2)
          SVPow2PTrues.insert(IntrI);
      }

      Changed |= coalescePTrueIntrinsicCalls(BB, SVAllPTrues);
      Changed |= coalescePTrueIntrinsicCalls(BB, SVPow2PTrues);
    }
  }

  return Changed;
}

// This is done in SVEIntrinsicOpts rather than InstCombine so that we introduce
// scalable stores as late as possible
bool SVEIntrinsicOpts::optimizePredicateStore(Instruction *I) {
  auto *F = I->getFunction();
  auto Attr = F->getFnAttribute(Attribute::VScaleRange);
  if (!Attr.isValid())
    return false;

  unsigned MinVScale = Attr.getVScaleRangeMin();
  std::optional<unsigned> MaxVScale = Attr.getVScaleRangeMax();
  // The transform needs to know the exact runtime length of scalable vectors
  if (!MaxVScale || MinVScale != MaxVScale)
    return false;

  auto *PredType =
      ScalableVectorType::get(Type::getInt1Ty(I->getContext()), 16);
  auto *FixedPredType =
      FixedVectorType::get(Type::getInt8Ty(I->getContext()), MinVScale * 2);
```
**EN:** This block continues the file's main SVE vector support logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的SVE 向量支持主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 295-326: Documented code section
```cpp

  // If we have a store..
  auto *Store = dyn_cast<StoreInst>(I);
  if (!Store || !Store->isSimple())
    return false;

  // ..that is storing a predicate vector sized worth of bits..
  if (Store->getOperand(0)->getType() != FixedPredType)
    return false;

  // ..where the value stored comes from a vector extract..
  auto *IntrI = dyn_cast<IntrinsicInst>(Store->getOperand(0));
  if (!IntrI || IntrI->getIntrinsicID() != Intrinsic::vector_extract)
    return false;

  // ..that is extracting from index 0..
  if (!cast<ConstantInt>(IntrI->getOperand(1))->isZero())
    return false;

  // ..where the value being extract from comes from a bitcast
  auto *BitCast = dyn_cast<BitCastInst>(IntrI->getOperand(0));
  if (!BitCast)
    return false;

  // ..and the bitcast is casting from predicate type
  if (BitCast->getOperand(0)->getType() != PredType)
    return false;

  IRBuilder<> Builder(I->getContext());
  Builder.SetInsertPoint(I);

  Builder.CreateStore(BitCast->getOperand(0), Store->getPointerOperand());
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 327-359: Core AArch64 backend logic
```cpp

  Store->eraseFromParent();
  if (IntrI->use_empty())
    IntrI->eraseFromParent();
  if (BitCast->use_empty())
    BitCast->eraseFromParent();

  return true;
}

// This is done in SVEIntrinsicOpts rather than InstCombine so that we introduce
// scalable loads as late as possible
bool SVEIntrinsicOpts::optimizePredicateLoad(Instruction *I) {
  auto *F = I->getFunction();
  auto Attr = F->getFnAttribute(Attribute::VScaleRange);
  if (!Attr.isValid())
    return false;

  unsigned MinVScale = Attr.getVScaleRangeMin();
  std::optional<unsigned> MaxVScale = Attr.getVScaleRangeMax();
  // The transform needs to know the exact runtime length of scalable vectors
  if (!MaxVScale || MinVScale != MaxVScale)
    return false;

  auto *PredType =
      ScalableVectorType::get(Type::getInt1Ty(I->getContext()), 16);
  auto *FixedPredType =
      FixedVectorType::get(Type::getInt8Ty(I->getContext()), MinVScale * 2);

  // If we have a bitcast..
  auto *BitCast = dyn_cast<BitCastInst>(I);
  if (!BitCast || BitCast->getType() != PredType)
    return false;
```
**EN:** This block continues the file's main SVE vector support logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的SVE 向量支持主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 360-393: Documented code section
```cpp

  // ..whose operand is a vector_insert..
  auto *IntrI = dyn_cast<IntrinsicInst>(BitCast->getOperand(0));
  if (!IntrI || IntrI->getIntrinsicID() != Intrinsic::vector_insert)
    return false;

  // ..that is inserting into index zero of an undef vector..
  if (!isa<UndefValue>(IntrI->getOperand(0)) ||
      !cast<ConstantInt>(IntrI->getOperand(2))->isZero())
    return false;

  // ..where the value inserted comes from a load..
  auto *Load = dyn_cast<LoadInst>(IntrI->getOperand(1));
  if (!Load || !Load->isSimple())
    return false;

  // ..that is loading a predicate vector sized worth of bits..
  if (Load->getType() != FixedPredType)
    return false;

  IRBuilder<> Builder(I->getContext());
  Builder.SetInsertPoint(Load);

  auto *LoadPred = Builder.CreateLoad(PredType, Load->getPointerOperand());

  BitCast->replaceAllUsesWith(LoadPred);
  BitCast->eraseFromParent();
  if (IntrI->use_empty())
    IntrI->eraseFromParent();
  if (Load->use_empty())
    Load->eraseFromParent();

  return true;
}
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 394-428: Function SVEIntrinsicOpts::optimizeInstructions
```cpp

bool SVEIntrinsicOpts::optimizeInstructions(
    SmallSetVector<Function *, 4> &Functions) {
  bool Changed = false;

  for (auto *F : Functions) {
    DominatorTree *DT = &getAnalysis<DominatorTreeWrapperPass>(*F).getDomTree();

    // Traverse the DT with an rpo walk so we see defs before uses, allowing
    // simplification to be done incrementally.
    BasicBlock *Root = DT->getRoot();
    ReversePostOrderTraversal<BasicBlock *> RPOT(Root);
    for (auto *BB : RPOT) {
      for (Instruction &I : make_early_inc_range(*BB)) {
        switch (I.getOpcode()) {
        case Instruction::Store:
          Changed |= optimizePredicateStore(&I);
          break;
        case Instruction::BitCast:
          Changed |= optimizePredicateLoad(&I);
          break;
        }
      }
    }
  }

  return Changed;
}

bool SVEIntrinsicOpts::optimizeFunctions(
    SmallSetVector<Function *, 4> &Functions) {
  bool Changed = false;

  Changed |= optimizePTrueIntrinsicCalls(Functions);
  Changed |= optimizeInstructions(Functions);
```
**EN:** This block implements SVEIntrinsicOpts::optimizeInstructions, advancing the file's SVE vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SVEIntrinsicOpts::optimizeInstructions，通过 AArch64 专用的决策与数据处理推进本文件的SVE 向量支持流程。
### Lines 429-460: Function SVEIntrinsicOpts::runOnModule
```cpp

  return Changed;
}

bool SVEIntrinsicOpts::runOnModule(Module &M) {
  bool Changed = false;
  SmallSetVector<Function *, 4> Functions;

  // Check for SVE intrinsic declarations first so that we only iterate over
  // relevant functions. Where an appropriate declaration is found, store the
  // function(s) where it is used so we can target these only.
  for (auto &F : M.getFunctionList()) {
    if (!F.isDeclaration())
      continue;

    switch (F.getIntrinsicID()) {
    case Intrinsic::vector_extract:
    case Intrinsic::vector_insert:
    case Intrinsic::aarch64_sve_ptrue:
      for (User *U : F.users())
        Functions.insert(cast<Instruction>(U)->getFunction());
      break;
    default:
      break;
    }
  }

  if (!Functions.empty())
    Changed |= optimizeFunctions(Functions);

  return Changed;
}
```
**EN:** This block implements SVEIntrinsicOpts::runOnModule, advancing the file's SVE vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SVEIntrinsicOpts::runOnModule，通过 AArch64 专用的决策与数据处理推进本文件的SVE 向量支持流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Scalable Vector Extension support **CN:** 可伸缩向量扩展 SVE 支持
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64.h, Utils/AArch64BaseInfo.h **CN:** 目标本地依赖：AArch64.h, Utils/AArch64BaseInfo.h
- **EN:** Core LLVM interfaces: llvm/ADT/PostOrderIterator.h, llvm/ADT/SetVector.h, llvm/IR/Constants.h, llvm/IR/Dominators.h, llvm/IR/IRBuilder.h, llvm/IR/Instructions.h, llvm/IR/IntrinsicInst.h, llvm/IR/IntrinsicsAArch64.h, llvm/IR/LLVMContext.h, llvm/IR/Module.h ... **CN:** 核心 LLVM 接口：llvm/ADT/PostOrderIterator.h, llvm/ADT/SetVector.h, llvm/IR/Constants.h, llvm/IR/Dominators.h, llvm/IR/IRBuilder.h, llvm/IR/Instructions.h, llvm/IR/IntrinsicInst.h, llvm/IR/IntrinsicsAArch64.h, llvm/IR/LLVMContext.h, llvm/IR/Module.h ...
- **EN:** Standard-library support: optional **CN:** 标准库支持：optional
- **EN:** Closely connected with neighboring AArch64 backend components responsible for SVE vector support. **CN:** 与周边负责SVE 向量支持的 AArch64 后端组件紧密协作。
