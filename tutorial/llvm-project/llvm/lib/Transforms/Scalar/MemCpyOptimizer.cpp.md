# MemCpyOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/MemCpyOptimizer.cpp` | `llvm/lib/Transforms/Scalar/MemCpyOptimizer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements optimize use of memcpy and friends within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 MemCpyOptimizer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-149

```cpp
//===- MemCpyOptimizer.cpp - Optimize use of memcpy and friends -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass performs various transformations related to eliminating memcpy
// calls, or transforming sets of stores into memset's.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/MemCpyOptimizer.h"
#include "llvm/ADT/DenseSet.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/ScopeExit.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/ADT/iterator_range.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // However, merging 2 32-bit stores isn't useful on a 32-bit architecture (the
  // memset will be split into 2 32-bit stores anyway) and doing so can
  // pessimize the llvm optimizer.
  //
  // Since we don't have perfect knowledge here, make some assumptions: assume
  // the maximum GPR width is the same size as the largest legal integer
  // size. If so, check to see whether we will end up actually reducing the
  // number of stores used.
  unsigned Bytes = unsigned(End - Start);
  unsigned MaxIntSize = DL.getLargestLegalIntTypeSizeInBits() / 8;
  if (MaxIntSize == 0)
    MaxIntSize = 1;
  unsigned NumPointerStores = Bytes / MaxIntSize;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 150-294

```cpp
  // Assume the remaining bytes if any are done a byte at a time.
  unsigned NumByteStores = Bytes % MaxIntSize;

  // If we will reduce the # stores (according to this heuristic), do the
  // transformation.  This encourages merging 4 x i8 -> i32 and 2 x i16 -> i32
  // etc.
  return TheStores.size() > NumPointerStores + NumByteStores;
}

namespace {

class MemsetRanges {
  using range_iterator = SmallVectorImpl<MemsetRange>::iterator;

  /// A sorted list of the memset ranges.
  SmallVector<MemsetRange, 8> Ranges;

  const DataLayout &DL;

public:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      !RequiresNoCaptureBeforeUnwind)
    return false;

  // Check whether there are any unwinding instructions in the range.
  return any_of(make_range(Start->getIterator(), End->getIterator()),
                [](const Instruction &I) { return I.mayThrow(); });
}

void MemCpyOptPass::eraseInstruction(Instruction *I) {
  MSSAU->removeMemoryAccess(I);
  EEA->removeInstruction(I);
  I->eraseFromParent();
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include MemsetRanges, addInst, addStore, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 MemsetRanges, addInst, addStore，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 295-440

```cpp
// Check for mod or ref of Loc between Start and End, excluding both boundaries.
// Start and End must be in the same block.
// If SkippedLifetimeStart is provided, skip over one clobbering lifetime.start
// intrinsic and store it inside SkippedLifetimeStart.
static bool accessedBetween(BatchAAResults &AA, MemoryLocation Loc,
                            const MemoryUseOrDef *Start,
                            const MemoryUseOrDef *End,
                            Instruction **SkippedLifetimeStart = nullptr) {
  assert(Start->getBlock() == End->getBlock() && "Only local supported");
  for (const MemoryAccess &MA :
       make_range(++Start->getIterator(), End->getIterator())) {
    Instruction *I = cast<MemoryUseOrDef>(MA).getMemoryInst();
    if (isModOrRefSet(AA.getModRefInfo(I, Loc))) {
      auto *II = dyn_cast<IntrinsicInst>(I);
      if (II && II->getIntrinsicID() == Intrinsic::lifetime_start &&
          SkippedLifetimeStart && !*SkippedLifetimeStart) {
        *SkippedLifetimeStart = I;
        continue;
      }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      Ranges.addStore(*Offset, NextStore);
    } else {
      auto *MSI = cast<MemSetInst>(BI);

      if (MSI->isVolatile() || ByteVal != MSI->getValue() ||
          !isa<ConstantInt>(MSI->getLength()))
        break;

      // Check to see if this store is to a constant offset from the start ptr.
      std::optional<int64_t> Offset =
          MSI->getDest()->getPointerOffsetFrom(StartPtr, DL);
      if (!Offset)
        break;

```
- EN: Core entities appearing here include make_range, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout, MemorySSA.
- CN: 此处出现的核心实体包括 make_range，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout, MemorySSA 等分析结果。

### Lines 441-583

```cpp
      Ranges.addMemSet(*Offset, MSI);
    }
  }

  // If we have no ranges, then we just had a single store with nothing that
  // could be merged in.  This is a very common case of course.
  if (Ranges.empty())
    return nullptr;

  // If we had at least one store that could be merged in, add the starting
  // store as well.  We try to avoid this unless there is at least something
  // interesting as a small compile-time optimization.
  Ranges.addInst(0, StartInst);

  // If we create any memsets, we put it right before the first instruction that
  // isn't part of the memset block.  This ensure that the memset is dominated
  // by any addressing instruction needed by the start of the block.
  IRBuilder<> Builder(&*BI);

  // Now that we have full information about ranges, loop over the ranges and
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (isModSet(AA->getModRefInfo(C, LoadLoc)))
        return false;
      else if (const auto *Call = dyn_cast<CallBase>(C)) {
        // If we can't lift this before P, it's game over.
        if (isModOrRefSet(AA->getModRefInfo(P, Call)))
          return false;

        Calls.push_back(Call);
      } else if (isa<LoadInst>(C) || isa<StoreInst>(C) || isa<VAArgInst>(C)) {
        // If we can't lift this before P, it's game over.
        auto ML = MemoryLocation::get(C);
        if (isModOrRefSet(AA->getModRefInfo(P, ML)))
          return false;

```
- EN: Core entities appearing here include moveUp, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 moveUp，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 584-727

```cpp
        MemLocs.push_back(ML);
      } else
        // We don't know how to lift this instruction.
        return false;
    }

    ToLift.push_back(C);
    for (Value *Op : C->operands())
      if (!AddArg(Op))
        return false;
  }

  // Find MSSA insertion point. Normally P will always have a corresponding
  // memory access before which we can insert. However, with non-standard AA
  // pipelines, there may be a mismatch between AA and MSSA, in which case we
  // will scan for a memory access before P. In either case, we know for sure
  // that at least the load will have a memory access.
  // TODO: Simplify this once P will be determined by MSSA, in which case the
  // discrepancy can no longer occur.
  MemoryUseOrDef *MemInsertPoint = nullptr;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  };

  bool Changed = performCallSlotOptzn(
      LI, SI, SI->getPointerOperand()->stripPointerCasts(),
      LI->getPointerOperand()->stripPointerCasts(),
      DL.getTypeStoreSize(SI->getOperand(0)->getType()),
      std::min(SI->getAlign(), LI->getAlign()), BAA, GetCall);
  if (Changed) {
    eraseInstruction(SI);
    eraseInstruction(LI);
    ++NumMemCpyInstr;
    return true;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 728-870

```cpp
  // If this is a load-store pair from a stack slot to a stack slot, we
  // might be able to perform the stack-move optimization just as we do for
  // memcpys from an alloca to an alloca.
  if (performStackMoveOptzn(LI, SI, SI->getPointerOperand(),
                            LI->getPointerOperand(), DL.getTypeStoreSize(T),
                            BAA)) {
    // Avoid invalidating the iterator.
    BBI = SI->getNextNode()->getIterator();
    eraseInstruction(SI);
    eraseInstruction(LI);
    ++NumMemCpyInstr;
    return true;
  }

  return false;
}

bool MemCpyOptPass::processStore(StoreInst *SI, BasicBlock::iterator &BBI) {
  if (!SI->isSimple())
    return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  //
  // Since moving the memcpy is technically awkward, we additionally check that
  // src only holds uninitialized values at the moment of the call, meaning that
  // the memcpy can be discarded rather than moved.

  // We can't optimize scalable types.
  if (cpySize.isScalable())
    return false;

  // Require that src be an alloca.  This simplifies the reasoning considerably.
  auto *srcAlloca = dyn_cast<AllocaInst>(cpySrc);
  if (!srcAlloca)
    return false;

```
- EN: Core entities appearing here include processStore, tryMergingIntoMemset, processMemSet, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 processStore, tryMergingIntoMemset, processMemSet，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 871-1016

```cpp
  const DataLayout &DL = cpyLoad->getDataLayout();
  // We can't optimize scalable types or variable-length allocas.
  std::optional<TypeSize> SrcAllocaSize = srcAlloca->getAllocationSize(DL);
  if (!SrcAllocaSize || SrcAllocaSize->isScalable())
    return false;
  uint64_t srcSize = SrcAllocaSize->getFixedValue();

  if (cpySize < srcSize)
    return false;

  CallInst *C = GetC();
  if (!C)
    return false;

  // Lifetime marks shouldn't be operated on.
  if (Function *F = C->getCalledFunction())
    if (F->isIntrinsic() && F->getIntrinsicID() == Intrinsic::lifetime_start)
      return false;

  if (C->getParent() != cpyStore->getParent()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        MemoryLocation(srcAlloca, LocationSize::precise(srcSize));
    for (Instruction &I :
         make_range(++C->getIterator(), C->getParent()->end())) {
      // Lifetime of srcAlloca ends at lifetime.end.
      if (auto *II = dyn_cast<IntrinsicInst>(&I)) {
        if (II->getIntrinsicID() == Intrinsic::lifetime_end &&
            II->getArgOperand(0) == srcAlloca)
          break;
      }

      // Lifetime of srcAlloca ends at return.
      if (isa<ReturnInst>(&I))
        break;

```
- EN: Core entities appearing here include make_range, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 make_range，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 1017-1175

```cpp
      // Ignore the direct read of src in the load.
      if (&I == cpyLoad)
        continue;

      // Check whether this instruction may mod/ref src through the captured
      // pointer (we have already any direct mod/refs in the loop above).
      // Also bail if we hit a terminator, as we don't want to scan into other
      // blocks.
      if (isModOrRefSet(BAA.getModRefInfo(&I, SrcLoc)) || I.isTerminator())
        return false;
    }
  }

  // Since we're changing the parameter to the callsite, we need to make sure
  // that what would be the new parameter dominates the callsite.
  bool NeedMoveGEP = false;
  if (!DT->dominates(cpyDest, C)) {
    // Support moving a constant index GEP before the call.
    auto *GEP = dyn_cast<GetElementPtrInst>(cpyDest);
    if (GEP && GEP->hasAllConstantIndices() &&
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // want to do something like this in another place. Then we'd probably
      // have to delay instruction removal until all transforms on an
      // instruction finished.
      eraseInstruction(NewCopySource);
  });
  MaybeAlign CopySourceAlign = MDep->getSourceAlign();
  auto MCopyLoc = MemoryLocation::getForSource(MDep);
  // Truncate the size of the MDep access to just the bytes read
  if (MDep->getLength() != CopyLength) {
    auto *ConstLength = cast<ConstantInt>(CopyLength);
    MCopyLoc = MCopyLoc.getWithNewSize(
        LocationSize::precise(ConstLength->getZExtValue()));
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1176-1320

```cpp
  // When the forwarding offset is greater than 0, we transform
  //    memcpy(d1 <- s1)
  //    memcpy(d2 <- d1+o)
  // to
  //    memcpy(d2 <- s1+o)
  if (MForwardOffset > 0) {
    // The copy destination of `M` maybe can serve as the source of copying.
    std::optional<int64_t> MDestOffset =
        M->getRawDest()->getPointerOffsetFrom(MDep->getRawSource(), DL);
    if (MDestOffset == MForwardOffset)
      CopySource = M->getDest();
    else {
      CopySource = Builder.CreateInBoundsPtrAdd(
          CopySource, Builder.getInt64(MForwardOffset));
      NewCopySource = dyn_cast<Instruction>(CopySource);
    }
    // We need to update `MCopyLoc` if an offset exists.
    MCopyLoc = MCopyLoc.getWithNewPtr(CopySource);
    if (CopySourceAlign)
      CopySourceAlign = commonAlignment(*CopySourceAlign, MForwardOffset);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return false;

  // We know that dst up to src_size is not written. We now need to make sure
  // that dst up to dst_size is not accessed. (If we did not move the memset,
  // checking for reads would be sufficient.)
  if (accessedBetween(BAA, MemoryLocation::getForDest(MemSet),
                      MSSA->getMemoryAccess(MemSet),
                      MSSA->getMemoryAccess(MemCpy)))
    return false;

  // Use the same i8* dest as the memcpy, killing the memset dest if different.
  Value *Dest = MemCpy->getRawDest();
  Value *DestSize = MemSet->getLength();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1321-1467

```cpp
  if (mayBeVisibleThroughUnwinding(Dest, MemSet, MemCpy))
    return false;

  // If the sizes are the same, simply drop the memset instead of generating
  // a replacement with zero size.
  if (DestSize == SrcSize) {
    eraseInstruction(MemSet);
    return true;
  }

  // By default, create an unaligned memset.
  Align Alignment = Align(1);
  // If Dest is aligned, and SrcSize is constant, use the minimum alignment
  // of the sum.
  const Align DestAlign = std::max(MemSet->getDestAlign().valueOrOne(),
                                   MemCpy->getDestAlign().valueOrOne());
  if (DestAlign > 1)
    if (auto *SrcSizeC = dyn_cast<ConstantInt>(SrcSize))
      Alignment = commonAlignment(DestAlign, SrcSizeC->getZExtValue());

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // covered by the memset unless undef bytes. Don't worry about sizes larger
    // than i64.
    auto *CMemSetSize = dyn_cast<ConstantInt>(MemSetSize);
    auto *CCopySize = dyn_cast<ConstantInt>(CopySize);
    if (!CMemSetSize || !CCopySize || MOffset < 0 ||
        CCopySize->getZExtValue() + MOffset > CMemSetSize->getZExtValue()) {
      if (!overreadUndefContents(MSSA, MemCpy, MemSet, BAA))
        return false;

      if (CMemSetSize && CCopySize) {
        uint64_t MemSetSizeVal = CMemSetSize->getZExtValue();
        uint64_t MemCpySizeVal = CCopySize->getZExtValue();
        uint64_t NewSize;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout, MemorySSA. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout, MemorySSA 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1468-1611

```cpp
        if (MOffset < 0) {
          // Offset from beginning of the initialized region.
          uint64_t Offset = -MOffset;
          NewSize = MemCpySizeVal <= Offset ? 0 : MemCpySizeVal - Offset;
        } else if (MOffset == 0) {
          NewSize = MemSetSizeVal;
        } else {
          NewSize =
              MemSetSizeVal <= (uint64_t)MOffset ? 0 : MemSetSizeVal - MOffset;
        }
        CopySize = ConstantInt::get(CopySize->getType(), NewSize);
      } else {
        if (MOffset < 0)
          return false;
      }
    }
  }

  IRBuilder<> Builder(MemCpy);
  Value *DestPtr = MemCpy->getRawDest();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

        if (Visited.size() >= MaxUsesToExplore) {
          LLVM_DEBUG(
              dbgs()
              << "Stack Move: Exceeded max uses to see ModRef, bailing\n");
          return false;
        }
        if (!Visited.insert(&U).second)
          continue;
        UseCaptureInfo CI = DetermineUseCaptureKind(U, AI);
        if (capturesAnyProvenance(CI.UseCC))
          return false;
        AddressCaptured |= capturesAddress(CI.UseCC);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1612-1761

```cpp
        if (UI->mayReadOrWriteMemory()) {
          if (UI->isLifetimeStartOrEnd()) {
            // We note the locations of these intrinsic calls so that we can
            // delete them later if the optimization succeeds, this is safe
            // since both llvm.lifetime.start and llvm.lifetime.end intrinsics
            // practically fill all the bytes of the alloca with an undefined
            // value, although conceptually marked as alive/dead.
            LifetimeMarkers.push_back(UI);
            continue;
          }
          AAMetadataInstrs.insert(UI);

          if (!ModRefCallback(UI))
            return false;
        }

        if (capturesAnything(CI.ResultCC)) {
          Worklist.push_back(UI);
          continue;
        }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  DestAlloca->replaceAllUsesWith(NewDestPtr);
  eraseInstruction(DestAlloca);

  // Drop metadata on the source alloca.
  SrcAlloca->dropUnknownNonDebugMetadata();

  // TODO: Reconstruct merged lifetime markers.
  // Remove all other lifetime markers. if the original lifetime intrinsics
  // exists.
  if (!LifetimeMarkers.empty()) {
    for (Instruction *I : LifetimeMarkers)
      eraseInstruction(I);
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1762-1916

```cpp
  // As this transformation can cause memory accesses that didn't previously
  // alias to begin to alias one another, we remove !alias.scope, !noalias,
  // !tbaa and !tbaa_struct metadata from any uses of either alloca.
  // This is conservative, but more precision doesn't seem worthwhile
  // right now.
  for (Instruction *I : AAMetadataInstrs) {
    I->setMetadata(LLVMContext::MD_alias_scope, nullptr);
    I->setMetadata(LLVMContext::MD_noalias, nullptr);
    I->setMetadata(LLVMContext::MD_tbaa, nullptr);
    I->setMetadata(LLVMContext::MD_tbaa_struct, nullptr);
  }

  LLVM_DEBUG(dbgs() << "Stack Move: Performed stack-move optimization\n");
  NumStackMove++;
  return true;
}

static bool isZeroSize(Value *Size) {
  if (auto *I = dyn_cast<Instruction>(Size))
    if (auto *Res = simplifyInstruction(I, I->getDataLayout()))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // to perform the stack-move optimization. See the comments in
  // performStackMoveOptzn() for more details.
  ConstantInt *Len = dyn_cast<ConstantInt>(M->getLength());
  if (Len == nullptr)
    return false;
  if (performStackMoveOptzn(M, M, M->getDest(), M->getSource(),
                            TypeSize::getFixed(Len->getZExtValue()), BAA)) {
    // Avoid invalidating the iterator.
    BBI = M->getNextNode()->getIterator();
    eraseInstruction(M);
    ++NumMemCpyInstr;
    return true;
  }

```
- EN: Core entities appearing here include isZeroSize, processMemCpy, getFixed, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isZeroSize, processMemCpy, getFixed，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1917-2064

```cpp
  return false;
}

/// Memmove calls with overlapping src/dest buffers that come after a memset may
/// be removed.
bool MemCpyOptPass::isMemMoveMemSetDependency(MemMoveInst *M) {
  const auto &DL = M->getDataLayout();
  MemoryUseOrDef *MemMoveAccess = MSSA->getMemoryAccess(M);
  if (!MemMoveAccess)
    return false;

  // The memmove is of form memmove(x, x + A, B).
  MemoryLocation SourceLoc = MemoryLocation::getForSource(M);
  auto *MemMoveSourceOp = M->getSource();
  auto *Source = dyn_cast<GEPOperator>(MemMoveSourceOp);
  if (!Source)
    return false;

  APInt Offset(DL.getIndexTypeSizeInBits(Source->getType()), 0);
  LocationSize MemMoveLocSize = SourceLoc.Size;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // The type of the memcpy source must match the byval argument
  if (MDep->getSource()->getType() != ByValArg->getType())
    return false;

  // Verify that the copied-from memory doesn't change in between the memcpy and
  // the byval call.
  //    memcpy(a <- b)
  //    *b = 42;
  //    foo(*a)
  // It would be invalid to transform the second memcpy into foo(*b).
  if (writtenBetween(MSSA, BAA, MemoryLocation::getForSource(MDep),
                     MSSA->getMemoryAccess(MDep), CallAccess))
    return false;

```
- EN: Core entities appearing here include isMemMoveMemSetDependency, processMemMove, processByValArgument, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout, MemorySSA.
- CN: 此处出现的核心实体包括 isMemMoveMemSetDependency, processMemMove, processByValArgument，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout, MemorySSA 等分析结果。

### Lines 2065-2215

```cpp
  LLVM_DEBUG(dbgs() << "MemCpyOptPass: Forwarding memcpy to byval:\n"
                    << "  " << *MDep << "\n"
                    << "  " << CB << "\n");

  // Otherwise we're good!  Update the byval argument.
  combineAAMetadata(&CB, MDep);
  CB.setArgOperand(ArgNo, MDep->getSource());
  ++NumMemCpyInstr;
  return true;
}

/// This is called on memcpy dest pointer arguments attributed as immutable
/// during call. Try to use memcpy source directly if all of the following
/// conditions are satisfied.
/// 1. The memcpy dst is neither modified during the call nor captured by the
/// call.
/// 2. The memcpy dst is an alloca with known alignment & size.
///     2-1. The memcpy length == the alloca size which ensures that the new
///     pointer is dereferenceable for the required range
///     2-2. The src pointer has alignment >= the alloca alignment or can be
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        RepeatInstruction = processMemSet(M, BI);
      else if (auto *M = dyn_cast<MemCpyInst>(I))
        RepeatInstruction = processMemCpy(M, BI);
      else if (auto *M = dyn_cast<MemMoveInst>(I))
        RepeatInstruction = processMemMove(M, BI);
      else if (auto *CB = dyn_cast<CallBase>(I)) {
        for (unsigned i = 0, e = CB->arg_size(); i != e; ++i) {
          if (CB->isByValArgument(i))
            MadeChange |= processByValArgument(*CB, i);
          else if (CB->onlyReadsMemory(i))
            MadeChange |= processImmutArgument(*CB, i);
        }
      }

```
- EN: Core entities appearing here include processImmutArgument, iterateOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults, DataLayout.
- CN: 此处出现的核心实体包括 processImmutArgument, iterateOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults, DataLayout 等分析结果。

### Lines 2216-2272

```cpp
      // Reprocess the instruction if desired.
      if (RepeatInstruction) {
        if (BI != BB.begin())
          --BI;
        MadeChange = true;
      }
    }
  }

  return MadeChange;
}

PreservedAnalyses MemCpyOptPass::run(Function &F, FunctionAnalysisManager &AM) {
  auto &TLI = AM.getResult<TargetLibraryAnalysis>(F);
  auto *AA = &AM.getResult<AAManager>(F);
  auto *AC = &AM.getResult<AssumptionAnalysis>(F);
  auto *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  auto *PDT = &AM.getResult<PostDominatorTreeAnalysis>(F);
  auto *MSSA = &AM.getResult<MemorySSAAnalysis>(F);

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  EarliestEscapeAnalysis EEA_(*DT);
  EEA = &EEA_;

  while (true) {
    if (!iterateOnFunction(F))
      break;
    MadeChange = true;
  }

  if (VerifyMemorySSA)
    MSSA_->verifyMemorySSA();

  return MadeChange;
}
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, AliasAnalysis, AssumptionCache, DominatorTree.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAManager, AliasAnalysis, AssumptionCache, DominatorTree 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `MemsetRange, MemsetRanges, isProfitableToUseMemset, addInst, addStore, addMemSet, eraseInstruction, make_range` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`MemsetRange, MemsetRanges, isProfitableToUseMemset, addInst, addStore, addMemSet, eraseInstruction, make_range` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AAManager, AliasAnalysis, AssumptionCache, DataLayout, DominatorTree, MemorySSA, MemorySSAUpdater` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AAManager, AliasAnalysis, AssumptionCache, DataLayout, DominatorTree, MemorySSA, MemorySSAUpdater` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnableMemCpyOptWithoutLibcalls` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnableMemCpyOptWithoutLibcalls` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CFG.h`, `llvm/Analysis/CaptureTracking.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CFG.h`, `llvm/Analysis/CaptureTracking.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/MemorySSAUpdater.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/iterator_range.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/DenseSet.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/iterator_range.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DominatorTree`, `MemorySSA`, `MemorySSAUpdater` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DominatorTree`, `MemorySSA`, `MemorySSAUpdater` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
