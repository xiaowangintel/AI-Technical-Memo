# CloneFunction.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CloneFunction.cpp` | `llvm/lib/Transforms/Utils/CloneFunction.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements clone a function into another function within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CloneFunction 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-83

```cpp
//===- CloneFunction.cpp - Clone a function into another function ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the CloneFunctionInto interface, which is used as the
// low-level function cloner.  This is used by the CloneFunction and function
// inliner to do the dirty work of copying the body of a function around.
//
//===----------------------------------------------------------------------===//

#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/SmallVectorExtras.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/ConstantFolding.h"
#include "llvm/Analysis/DomTreeUpdater.h"
#include "llvm/Analysis/InstructionSimplify.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (!M)
    return;
  // Inspect instructions to process e.g. DILexicalBlocks of inlined functions
  for (const Instruction &I : instructions(F))
    DIFinder.processInstruction(*M, I);
}

// Create a predicate that matches the metadata that should be identity mapped
// during function cloning.
static MetadataPredicate
createIdentityMDPredicate(const Function &F, CloneFunctionChangeType Changes) {
  if (Changes >= CloneFunctionChangeType::DifferentModule)
    return [](const Metadata *MD) { return false; };

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 84-172

```cpp
  DISubprogram *SPClonedWithinModule = F.getSubprogram();

  // Don't clone inlined subprograms.
  auto ShouldKeep = [SPClonedWithinModule](const DISubprogram *SP) -> bool {
    return SP != SPClonedWithinModule;
  };

  return [=](const Metadata *MD) {
    // Avoid cloning compile units.
    if (isa<DICompileUnit>(MD))
      return true;

    if (auto *SP = dyn_cast<DISubprogram>(MD))
      return ShouldKeep(SP);

    // If a subprogram isn't going to be cloned skip its lexical blocks as well.
    if (auto *LScope = dyn_cast<DILocalScope>(MD))
      return ShouldKeep(LScope->getSubprogram());

    // Avoid cloning local variables of subprograms that won't be cloned.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    }

    if (isa<CallInst>(I) && !I.isDebugOrPseudoInst()) {
      hasCalls = true;
      hasMemProfMetadata |= I.hasMetadata(LLVMContext::MD_memprof);
      hasMemProfMetadata |= I.hasMetadata(LLVMContext::MD_callsite);
    }
    if (const AllocaInst *AI = dyn_cast<AllocaInst>(&I)) {
      if (!AI->isStaticAlloca()) {
        hasDynamicAllocas = true;
      }
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 173-255

```cpp
  if (CodeInfo) {
    CodeInfo->ContainsCalls |= hasCalls;
    CodeInfo->ContainsMemProfMetadata |= hasMemProfMetadata;
    CodeInfo->ContainsDynamicAllocas |= hasDynamicAllocas;
  }
  return NewBB;
}

void llvm::CloneFunctionAttributesInto(Function *NewFunc,
                                       const Function *OldFunc,
                                       ValueToValueMapTy &VMap,
                                       bool ModuleLevelChanges,
                                       ValueMapTypeRemapper *TypeMapper,
                                       ValueMaterializer *Materializer) {
  // Copy all attributes other than those stored in Function's AttributeList
  // which holds e.g. parameters and return value attributes.
  AttributeList NewAttrs = NewFunc->getAttributes();
  NewFunc->copyAttributesFrom(OldFunc);
  NewFunc->setAttributes(NewAttrs);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }
}

void llvm::CloneFunctionBodyInto(Function &NewFunc, const Function &OldFunc,
                                 ValueToValueMapTy &VMap, RemapFlags RemapFlag,
                                 SmallVectorImpl<ReturnInst *> &Returns,
                                 const char *NameSuffix,
                                 ClonedCodeInfo *CodeInfo,
                                 ValueMapTypeRemapper *TypeMapper,
                                 ValueMaterializer *Materializer,
                                 const MetadataPredicate *IdentityMD) {
  if (OldFunc.isDeclaration())
    return;

```
- EN: This region continues the CloneFunction implementation with local helper logic centered on CodeInfo, ContainsCalls, ContainsMemProfMetadata, ContainsDynamicAllocas.
- CN: 这一段延续了 CloneFunction 的主体实现，围绕 CodeInfo, ContainsCalls, ContainsMemProfMetadata, ContainsDynamicAllocas 等局部辅助逻辑展开。

### Lines 256-340

```cpp
  // Loop over all of the basic blocks in the function, cloning them as
  // appropriate.  Note that we save BE this way in order to handle cloning of
  // recursive functions into themselves.
  for (const BasicBlock &BB : OldFunc) {
    // Create a new basic block and copy instructions into it!
    BasicBlock *CBB =
        CloneBasicBlock(&BB, VMap, NameSuffix, &NewFunc, CodeInfo);

    // Add basic block mapping.
    VMap[&BB] = CBB;

    // It is only legal to clone a function if a block address within that
    // function is never referenced outside of the function.  Given that, we
    // want to map block addresses from the old function to block addresses in
    // the clone. (This is different from the generic ValueMapper
    // implementation, which generates an invalid blockaddress when
    // cloning a function.)
    if (BB.hasAddressTaken()) {
      Constant *OldBBAddr = BlockAddress::get(const_cast<Function *>(&OldFunc),
                                              const_cast<BasicBlock *>(&BB));
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    assert((NewFunc->getParent() == nullptr ||
            NewFunc->getParent() == OldFunc->getParent()) &&
           "Expected NewFunc to have the same parent, or no parent");
  } else {
    assert((NewFunc->getParent() == nullptr ||
            NewFunc->getParent() != OldFunc->getParent()) &&
           "Expected NewFunc to have different parents, or no parent");

    if (Changes == CloneFunctionChangeType::DifferentModule) {
      assert(NewFunc->getParent() &&
             "Need parent of new function to maintain debug info invariants");
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 341-421

```cpp
  MetadataPredicate IdentityMD = createIdentityMDPredicate(*OldFunc, Changes);

  // Cloning is always a Module level operation, since Metadata needs to be
  // cloned.
  const RemapFlags RemapFlag = RF_None;

  CloneFunctionMetadataInto(*NewFunc, *OldFunc, VMap, RemapFlag, TypeMapper,
                            Materializer, &IdentityMD);

  CloneFunctionBodyInto(*NewFunc, *OldFunc, VMap, RemapFlag, Returns,
                        NameSuffix, CodeInfo, TypeMapper, Materializer,
                        &IdentityMD);

  // Only update !llvm.dbg.cu for DifferentModule (not CloneModule). In the
  // same module, the compile unit will already be listed (or not). When
  // cloning a module, CloneModule() will handle creating the named metadata.
  if (Changes != CloneFunctionChangeType::DifferentModule)
    return;

  // Update !llvm.dbg.cu with compile units added to the new module if this
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                        F->getFunctionType()->isVarArg());

  // Create the new function...
  Function *NewF = Function::Create(FTy, F->getLinkage(), F->getAddressSpace(),
                                    F->getName(), F->getParent());

  // Loop over the arguments, copying the names of the mapped arguments over...
  Function::arg_iterator DestI = NewF->arg_begin();
  for (const Argument &I : F->args())
    if (VMap.count(&I) == 0) {     // Is this argument preserved?
      DestI->setName(I.getName()); // Copy the name over...
      VMap[&I] = &*DestI++;        // Add mapping to VMap
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 422-503

```cpp
  SmallVector<ReturnInst *, 8> Returns; // Ignore returns cloned.
  CloneFunctionInto(NewF, F, VMap, CloneFunctionChangeType::LocalChangesOnly,
                    Returns, "", CodeInfo);

  return NewF;
}

namespace {
/// This is a private class used to implement CloneAndPruneFunctionInto.
struct PruningFunctionCloner {
  Function *NewFunc;
  const Function *OldFunc;
  ValueToValueMapTy &VMap;
  bool ModuleLevelChanges;
  const char *NameSuffix;
  ClonedCodeInfo &CodeInfo;
  bool HostFuncIsStrictFP;

  Instruction *cloneInstruction(BasicBlock::const_iterator II);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // rounding mode (absent in some intrinsics) and exception behavior. The
  // inlined function uses default settings.
  if (Intrinsic::hasConstrainedFPRoundingModeOperand(CIID))
    Args.push_back(
        MetadataAsValue::get(Ctx, MDString::get(Ctx, "round.tonearest")));
  Args.push_back(
      MetadataAsValue::get(Ctx, MDString::get(Ctx, "fpexcept.ignore")));

  SmallVector<Type *> ArgTys = llvm::map_to_vector(Args, &Value::getType);
  Function *IFn = Intrinsic::getOrInsertDeclaration(NewFunc->getParent(), CIID,
                                                    OldInst.getType(), ArgTys);
  return CallInst::Create(IFn, Args, OldInst.getName() + ".strict");
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include PruningFunctionCloner, CodeInfo, cloneInstruction, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 PruningFunctionCloner, CodeInfo, cloneInstruction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 504-591

```cpp
/// The specified block is found to be reachable, clone it and
/// anything that it can reach.
void PruningFunctionCloner::CloneBlock(
    const BasicBlock *BB, BasicBlock::const_iterator StartingInst,
    std::vector<const BasicBlock *> &ToClone) {
  WeakTrackingVH &BBEntry = VMap[BB];

  // Have we already cloned this block?
  if (BBEntry)
    return;

  // Nope, clone it now.
  BasicBlock *NewBB;
  Twine NewName(BB->hasName() ? Twine(BB->getName()) + NameSuffix : "");
  BBEntry = NewBB = BasicBlock::Create(BB->getContext(), NewName, NewFunc);

  // It is only legal to clone a function if a block address within that
  // function is never referenced outside of the function.  Given that, we
  // want to map block addresses from the old function to block addresses in
  // the clone. (This is different from the generic ValueMapper
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // Eagerly constant fold the newly cloned instruction. If successful, add
      // a mapping to the new value. Non-constant operands may be incomplete at
      // this stage, thus instruction simplification is performed after
      // processing phi-nodes.
      if (Value *V = ConstantFoldInstruction(
              NewInst, BB->getDataLayout())) {
        if (isInstructionTriviallyDead(NewInst)) {
          VMap[&*II] = V;
          NewInst->eraseFromParent();
          continue;
        }
      }
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 592-672

```cpp
    if (auto *CB = dyn_cast<CallBase>(II); CB && CB->isIndirectCall())
      CodeInfo.OriginallyIndirectCalls.insert(NewInst);

    if (II->hasName())
      NewInst->setName(II->getName() + NameSuffix);
    VMap[&*II] = NewInst; // Add instruction map to value.
    if (isa<CallInst>(II) && !II->isDebugOrPseudoInst()) {
      hasCalls = true;
      hasMemProfMetadata |= II->hasMetadata(LLVMContext::MD_memprof);
      hasMemProfMetadata |= II->hasMetadata(LLVMContext::MD_callsite);
    }

    CloneDbgRecordsToHere(NewInst, II);

    CodeInfo.OrigVMap[&*II] = NewInst;
    if (auto *CB = dyn_cast<CallBase>(&*II))
      if (CB->hasOperandBundles())
        CodeInfo.OperandBundleCallSites.push_back(NewInst);

    if (const AllocaInst *AI = dyn_cast<AllocaInst>(II)) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Instruction *NewInst = OldTI->clone();
    if (OldTI->hasName())
      NewInst->setName(OldTI->getName() + NameSuffix);
    NewInst->insertInto(NewBB, NewBB->end());

    CloneDbgRecordsToHere(NewInst, OldTI->getIterator());

    VMap[OldTI] = NewInst; // Add instruction map to value.

    CodeInfo.OrigVMap[OldTI] = NewInst;
    if (auto *CB = dyn_cast<CallBase>(OldTI))
      if (CB->hasOperandBundles())
        CodeInfo.OperandBundleCallSites.push_back(NewInst);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 673-758

```cpp
    // Recursively clone any reachable successor blocks.
    append_range(ToClone, successors(BB->getTerminator()));
  } else {
    // If we didn't create a new terminator, clone DbgVariableRecords from the
    // old terminator onto the new terminator.
    Instruction *NewInst = NewBB->getTerminator();
    assert(NewInst);

    CloneDbgRecordsToHere(NewInst, OldTI->getIterator());
  }

  CodeInfo.ContainsCalls |= hasCalls;
  CodeInfo.ContainsMemProfMetadata |= hasMemProfMetadata;
  CodeInfo.ContainsDynamicAllocas |= hasDynamicAllocas;
  CodeInfo.ContainsDynamicAllocas |=
      hasStaticAllocas && BB != &BB->getParent()->front();
}

/// This works like CloneAndPruneFunctionInto, except that it does not clone the
/// entire function. Instead it starts at an instruction provided by the caller
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Move the new block to preserve the order in the original function.
    NewBB->moveBefore(NewFunc->end());

    // Handle PHI nodes specially, as we have to remove references to dead
    // blocks.
    for (const PHINode &PN : BI.phis()) {
      // PHI nodes may have been remapped to non-PHI nodes by the caller or
      // during the cloning process.
      if (isa<PHINode>(VMap[&PN]))
        PHIToResolve.push_back(&PN);
      else
        break;
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 759-849

```cpp
    // Finally, remap the terminator instructions, as those can't be remapped
    // until all BBs are mapped.
    RemapInstruction(NewBB->getTerminator(), VMap,
                     ModuleLevelChanges ? RF_None : RF_NoModuleLevelChanges,
                     TypeMapper, Materializer);
  }

  // Defer PHI resolution until rest of function is resolved, PHI resolution
  // requires the CFG to be up-to-date.
  for (unsigned phino = 0, e = PHIToResolve.size(); phino != e;) {
    const PHINode *OPN = PHIToResolve[phino];
    unsigned NumPreds = OPN->getNumIncomingValues();
    const BasicBlock *OldBB = OPN->getParent();
    BasicBlock *NewBB = cast<BasicBlock>(VMap[OldBB]);

    // Map operands for blocks that are live and remove operands for blocks
    // that are dead.
    for (; phino != PHIToResolve.size() &&
           PHIToResolve[phino]->getParent() == OldBB;
         ++phino) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (PN->getNumIncomingValues() == 0) {
      BasicBlock::iterator I = NewBB->begin();
      BasicBlock::const_iterator OldI = OldBB->begin();
      while ((PN = dyn_cast<PHINode>(I++))) {
        Value *NV = PoisonValue::get(PN->getType());
        PN->replaceAllUsesWith(NV);
        assert(VMap[&*OldI] == PN && "VMap mismatch");
        VMap[&*OldI] = NV;
        PN->eraseFromParent();
        ++OldI;
      }
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 850-936

```cpp
  // Drop all incompatible return attributes that cannot be applied to NewFunc
  // during cloning, so as to allow instruction simplification to reason on the
  // old state of the function. The original attributes are restored later.
  AttributeList Attrs = NewFunc->getAttributes();
  AttributeMask IncompatibleAttrs = AttributeFuncs::typeIncompatible(
      OldFunc->getReturnType(), Attrs.getRetAttrs());
  NewFunc->removeRetAttrs(IncompatibleAttrs);

  // As phi-nodes have been now remapped, allow incremental simplification of
  // newly-cloned instructions.
  const DataLayout &DL = NewFunc->getDataLayout();
  for (const BasicBlock &BB : *OldFunc) {
    for (const Instruction &I : BB) {
      auto *NewI = dyn_cast_or_null<Instruction>(VMap.lookup(&I));
      if (!NewI)
        continue;

      if (Value *V = simplifyInstruction(NewI, DL)) {
        NewI->replaceAllUsesWith(V);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  // Now that the inlined function body has been fully constructed, go through
  // and zap unconditional fall-through branches. This happens all the time when
  // specializing code: code specialization turns conditional branches into
  // uncond branches, and this code folds them.
  Function::iterator I = Begin;
  while (I != NewFunc->end()) {
    UncondBrInst *BI = dyn_cast<UncondBrInst>(I->getTerminator());
    if (!BI) {
      ++I;
      continue;
    }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 937-1017

```cpp
    BasicBlock *Dest = BI->getSuccessor();
    if (!Dest->getSinglePredecessor() || Dest->hasAddressTaken()) {
      ++I;
      continue;
    }

    // We shouldn't be able to get single-entry PHI nodes here, as instsimplify
    // above should have zapped all of them..
    assert(!isa<PHINode>(Dest->begin()));

    // We know all single-entry PHI nodes in the inlined function have been
    // removed, so we just need to splice the blocks.
    BI->eraseFromParent();

    // Make all PHI nodes that referred to Dest now refer to I as their source.
    Dest->replaceAllUsesWith(&*I);

    // Move all the instructions in the succ to the pred.
    I->splice(I->end(), Dest);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// Clones a loop \p OrigLoop.  Returns the loop and the blocks in \p
/// Blocks.
///
/// Updates LoopInfo and DominatorTree assuming the loop is dominated by block
/// \p LoopDomBB.  Insert the new blocks before block specified in \p Before.
Loop *llvm::cloneLoopWithPreheader(BasicBlock *Before, BasicBlock *LoopDomBB,
                                   Loop *OrigLoop, ValueToValueMapTy &VMap,
                                   const Twine &NameSuffix, LoopInfo *LI,
                                   DominatorTree *DT,
                                   SmallVectorImpl<BasicBlock *> &Blocks) {
  Function *F = OrigLoop->getHeader()->getParent();
  Loop *ParentLoop = OrigLoop->getParentLoop();
  DenseMap<Loop *, Loop *> LMap;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1018-1097

```cpp
  Loop *NewLoop = LI->AllocateLoop();
  LMap[OrigLoop] = NewLoop;
  if (ParentLoop)
    ParentLoop->addChildLoop(NewLoop);
  else
    LI->addTopLevelLoop(NewLoop);

  BasicBlock *OrigPH = OrigLoop->getLoopPreheader();
  assert(OrigPH && "No preheader");
  BasicBlock *NewPH = CloneBasicBlock(OrigPH, VMap, NameSuffix, F);
  // To rename the loop PHIs.
  VMap[OrigPH] = NewPH;
  Blocks.push_back(NewPH);

  // Update LoopInfo.
  if (ParentLoop)
    ParentLoop->addBasicBlockToLoop(NewPH, *LI);

  // Update DominatorTree.
  DT->addNewBlock(NewPH, LoopDomBB);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Move them physically from the end of the block list.
  F->splice(Before->getIterator(), F, NewPH->getIterator());
  F->splice(Before->getIterator(), F, NewLoop->getHeader()->getIterator(),
            F->end());

  return NewLoop;
}

/// Duplicate non-Phi instructions from the beginning of block up to
/// StopAt instruction into a split block between BB and its predecessor.
BasicBlock *llvm::DuplicateInstructionsInSplitBetween(
    BasicBlock *BB, BasicBlock *PredBB, Instruction *StopAt,
    ValueToValueMapTy &ValueMapping, DomTreeUpdater &DTU) {

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, LoopInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, LoopInfo 等分析结果。

### Lines 1098-1188

```cpp
  assert(count(successors(PredBB), BB) == 1 &&
         "There must be a single edge between PredBB and BB!");
  // We are going to have to map operands from the original BB block to the new
  // copy of the block 'NewBB'.  If there are PHI nodes in BB, evaluate them to
  // account for entry from PredBB.
  BasicBlock::iterator BI = BB->begin();
  for (; PHINode *PN = dyn_cast<PHINode>(BI); ++BI)
    ValueMapping[PN] = PN->getIncomingValueForBlock(PredBB);

  BasicBlock *NewBB = SplitEdge(PredBB, BB);
  NewBB->setName(PredBB->getName() + ".split");
  Instruction *NewTerm = NewBB->getTerminator();

  // FIXME: SplitEdge does not yet take a DTU, so we include the split edge
  //        in the update set here.
  DTU.applyUpdates({{DominatorTree::Delete, PredBB, BB},
                    {DominatorTree::Insert, PredBB, NewBB},
                    {DominatorTree::Insert, NewBB, BB}});

  // Clone the non-phi instructions of BB into NewBB, keeping track of the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (MDNode *MD = dyn_cast<MDNode>(MDOp)) {
        if (auto *NewMD = ClonedScopes.lookup(MD)) {
          NewScopeList.push_back(NewMD);
          NeedsReplacement = true;
          continue;
        }
        NewScopeList.push_back(MD);
      }
    }
    if (NeedsReplacement)
      return MDNode::get(Context, NewScopeList);
    return nullptr;
  };

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1189-1253

```cpp
  if (auto *Decl = dyn_cast<NoAliasScopeDeclInst>(I))
    if (MDNode *NewScopeList = CloneScopeList(Decl->getScopeList()))
      Decl->setScopeList(NewScopeList);

  auto replaceWhenNeeded = [&](unsigned MD_ID) {
    if (const MDNode *CSNoAlias = I->getMetadata(MD_ID))
      if (MDNode *NewScopeList = CloneScopeList(CSNoAlias))
        I->setMetadata(MD_ID, NewScopeList);
  };
  replaceWhenNeeded(LLVMContext::MD_noalias);
  replaceWhenNeeded(LLVMContext::MD_alias_scope);
}

void llvm::cloneAndAdaptNoAliasScopes(ArrayRef<MDNode *> NoAliasDeclScopes,
                                      ArrayRef<BasicBlock *> NewBlocks,
                                      LLVMContext &Context, StringRef Ext) {
  if (NoAliasDeclScopes.empty())
    return;

  DenseMap<MDNode *, MDNode *> ClonedScopes;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    ArrayRef<BasicBlock *> BBs, SmallVectorImpl<MDNode *> &NoAliasDeclScopes) {
  for (BasicBlock *BB : BBs)
    for (Instruction &I : *BB)
      if (auto *Decl = dyn_cast<NoAliasScopeDeclInst>(&I))
        NoAliasDeclScopes.push_back(Decl->getScopeList());
}

void llvm::identifyNoAliasScopesToClone(
    BasicBlock::iterator Start, BasicBlock::iterator End,
    SmallVectorImpl<MDNode *> &NoAliasDeclScopes) {
  for (Instruction &I : make_range(Start, End))
    if (auto *Decl = dyn_cast<NoAliasScopeDeclInst>(&I))
      NoAliasDeclScopes.push_back(Decl->getScopeList());
}
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `PruningFunctionCloner, mapAtomInstance, createIdentityMDPredicate, CodeInfo, cloneInstruction` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`PruningFunctionCloner, mapAtomInstance, createIdentityMDPredicate, CodeInfo, cloneInstruction` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, DomTreeUpdater, LoopInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, DomTreeUpdater, LoopInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/LoopInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/AttributeMask.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Metadata.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/AttributeMask.h`, `llvm/IR/CFG.h`, `llvm/IR/Constants.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/LLVMContext.h`, `llvm/IR/MDBuilder.h`, `llvm/IR/Metadata.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/ADT/SmallVector.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/ADT/SmallVector.h`, `llvm/ADT/SmallVectorExtras.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cstdint`, `map`, `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cstdint`, `map`, `optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
