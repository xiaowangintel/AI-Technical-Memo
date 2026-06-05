# Scalarizer.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/Scalarizer.cpp` | `llvm/lib/Transforms/Scalar/Scalarizer.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements scalarize vector operations within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 Scalarizer 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-89

```cpp
//===- Scalarizer.cpp - Scalarize vector operations -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass converts vector operations into scalar operations (or, optionally,
// operations on smaller vector widths), in order to expose optimization
// opportunities on the individual scalar operations.
// It is mainly intended for targets that do not have vector units, but it
// may also be useful for revectorizing code to different vector widths.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/Scalarizer.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Twine.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

namespace {

struct VectorSplit {
  // The type of the vector.
  FixedVectorType *VecTy = nullptr;

  // The number of elements packed in a fragment (other than the remainder).
  unsigned NumPacked = 0;

  // The number of fragments (scalars or smaller vectors) into which the vector
  // shall be split.
  unsigned NumFragments = 0;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 90-177

```cpp
  // The type of each complete fragment.
  Type *SplitTy = nullptr;

  // The type of the remainder (last) fragment; null if all fragments are
  // complete.
  Type *RemainderTy = nullptr;

  Type *getFragmentType(unsigned I) const {
    return RemainderTy && I == NumFragments - 1 ? RemainderTy : SplitTy;
  }
};

// Provides a very limited vector-like interface for lazily accessing one
// component of a scattered vector or vector pointer.
class Scatterer {
public:
  Scatterer() = default;

  // Scatter V into Size components.  If new instructions are needed,
  // insert them before BBI in BB.  If Cache is nonnull, use it to cache
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  UnaryOperator &UO;
};

// BinarySplitter(BO)(Builder, X, Y, Name) uses Builder to create
// a binary operator like BO called Name with operands X and Y.
struct BinarySplitter {
  BinarySplitter(BinaryOperator &bo) : BO(bo) {}

  Value *operator()(IRBuilder<> &Builder, Value *Op0, Value *Op1,
                    const Twine &Name) const {
    return Builder.CreateBinOp(BO.getOpcode(), Op0, Op1, Name);
  }

```
- EN: Core entities appearing here include Scatterer, FCmpSplitter, ICmpSplitter, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 Scatterer, FCmpSplitter, ICmpSplitter，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 178-275

```cpp
  BinaryOperator &BO;
};

// Information about a load or store that we're scalarizing.
struct VectorLayout {
  VectorLayout() = default;

  // Return the alignment of fragment Frag.
  Align getFragmentAlign(unsigned Frag) {
    return commonAlignment(VecAlign, Frag * SplitSize);
  }

  // The split of the underlying vector type.
  VectorSplit VS;

  // The alignment of the vector.
  Align VecAlign;

  // The size of each (non-remainder) fragment in bytes.
  uint64_t SplitSize = 0;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          Fragment, PoisonValue::get(Fragment->getType()), ExtendMask);
      if (I == 0) {
        Res = Fragment;
      } else {
        for (unsigned J = 0; J < NumPacked; ++J)
          InsertMask[I * VS.NumPacked + J] = NumElements + J;
        Res = Builder.CreateShuffleVector(Res, Fragment, InsertMask,
                                          Name + ".upto" + Twine(I));
        for (unsigned J = 0; J < NumPacked; ++J)
          InsertMask[I * VS.NumPacked + J] = I * VS.NumPacked + J;
      }
    }
  }

```
- EN: Core entities appearing here include VectorLayout, getFragmentAlign, isStructOfMatchingFixedVectors, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 VectorLayout, getFragmentAlign, isStructOfMatchingFixedVectors，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 276-362

```cpp
  return Res;
}

namespace {
class ScalarizerVisitor : public InstVisitor<ScalarizerVisitor, bool> {
public:
  ScalarizerVisitor(DominatorTree *DT, const TargetTransformInfo *TTI,
                    ScalarizerPassOptions Options)
      : DT(DT), TTI(TTI),
        ScalarizeVariableInsertExtract(Options.ScalarizeVariableInsertExtract),
        ScalarizeLoadStore(Options.ScalarizeLoadStore),
        ScalarizeMinBits(Options.ScalarizeMinBits) {}

  bool visit(Function &F);

  // InstVisitor methods.  They return true if the instruction was scalarized,
  // false if nothing changed.
  bool visitInstruction(Instruction &I) { return false; }
  bool visitSelectInst(SelectInst &SI);
  bool visitICmpInst(ICmpInst &ICI);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  void getAnalysisUsage(AnalysisUsage &AU) const override;
};

} // end anonymous namespace

ScalarizerLegacyPass::ScalarizerLegacyPass(const ScalarizerPassOptions &Options)
    : FunctionPass(ID), Options(Options) {}

void ScalarizerLegacyPass::getAnalysisUsage(AnalysisUsage &AU) const {
  AU.addRequired<DominatorTreeWrapperPass>();
  AU.addRequired<TargetTransformInfoWrapperPass>();
  AU.addPreserved<DominatorTreeWrapperPass>();
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ScalarizerVisitor, ScalarizerLegacyPass, getAnalysisUsage, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, TargetTransformInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ScalarizerVisitor, ScalarizerLegacyPass, getAnalysisUsage，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, TargetTransformInfo 等分析结果。

### Lines 363-452

```cpp
char ScalarizerLegacyPass::ID = 0;
INITIALIZE_PASS_BEGIN(ScalarizerLegacyPass, "scalarizer",
                      "Scalarize vector operations", false, false)
INITIALIZE_PASS_DEPENDENCY(DominatorTreeWrapperPass)
INITIALIZE_PASS_END(ScalarizerLegacyPass, "scalarizer",
                    "Scalarize vector operations", false, false)

Scatterer::Scatterer(BasicBlock *bb, BasicBlock::iterator bbi, Value *v,
                     const VectorSplit &VS, ValueVector *cachePtr)
    : BB(bb), BBI(bbi), V(v), VS(VS), CachePtr(cachePtr) {
  IsPointer = V->getType()->isPointerTy();
  if (!CachePtr) {
    Tmp.resize(VS.NumFragments, nullptr);
  } else {
    assert((CachePtr->empty() || VS.NumFragments == CachePtr->size() ||
            IsPointer) &&
           "Inconsistent vector sizes");
    if (VS.NumFragments > CachePtr->size())
      CachePtr->resize(VS.NumFragments, nullptr);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  return CV[Frag];
}

bool ScalarizerLegacyPass::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;

  DominatorTree *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  const TargetTransformInfo *TTI =
      &getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
  ScalarizerVisitor Impl(DT, TTI, Options);
  return Impl.visit(F);
}

```
- EN: Core entities appearing here include BB, runOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 BB, runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 453-539

```cpp
FunctionPass *llvm::createScalarizerPass(const ScalarizerPassOptions &Options) {
  return new ScalarizerLegacyPass(Options);
}

bool ScalarizerVisitor::visit(Function &F) {
  assert(Gathered.empty() && Scattered.empty());

  Scalarized = false;

  // To ensure we replace gathered components correctly we need to do an ordered
  // traversal of the basic blocks in the function.
  ReversePostOrderTraversal<BasicBlock *> RPOT(&F.getEntryBlock());
  for (BasicBlock *BB : RPOT) {
    for (BasicBlock::iterator II = BB->begin(), IE = BB->end(); II != IE;) {
      Instruction *I = &*II;
      bool Done = InstVisitor::visit(I);
      ++II;
      if (Done && I->getType()->isVoidTy()) {
        I->eraseFromParent();
        Scalarized = true;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      if (V == nullptr || SV[I] == CV[I])
        continue;

      Instruction *Old = cast<Instruction>(V);
      if (isa<Instruction>(CV[I]))
        CV[I]->takeName(Old);
      Old->replaceAllUsesWith(CV[I]);
      PotentiallyDeadInstrs.emplace_back(Old);
    }
  }
  SV = CV;
  Gathered.push_back(GatherList::value_type(Op, &SV));
}

```
- EN: Core entities appearing here include visit, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visit，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 540-633

```cpp
// Replace Op with CV and collect Op has a potentially dead instruction.
void ScalarizerVisitor::replaceUses(Instruction *Op, Value *CV) {
  if (CV != Op) {
    Op->replaceAllUsesWith(CV);
    PotentiallyDeadInstrs.emplace_back(Op);
    Scalarized = true;
  }
}

// Return true if it is safe to transfer the given metadata tag from
// vector to scalar instructions.
bool ScalarizerVisitor::canTransferMetadata(unsigned Tag) {
  return (Tag == LLVMContext::MD_tbaa
          || Tag == LLVMContext::MD_fpmath
          || Tag == LLVMContext::MD_tbaa_struct
          || Tag == LLVMContext::MD_invariant_load
          || Tag == LLVMContext::MD_alias_scope
          || Tag == LLVMContext::MD_noalias
          || Tag == LLVMContext::MD_mem_parallel_loop_access
          || Tag == LLVMContext::MD_access_group);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (!VS)
    return {};

  VectorLayout Layout;
  Layout.VS = *VS;
  // Check that we're dealing with full-byte fragments.
  if (!DL.typeSizeEqualsStoreSize(VS->SplitTy) ||
      (VS->RemainderTy && !DL.typeSizeEqualsStoreSize(VS->RemainderTy)))
    return {};
  Layout.VecAlign = Alignment;
  Layout.SplitSize = DL.getTypeStoreSize(VS->SplitTy);
  return Layout;
}

```
- EN: Core entities appearing here include replaceUses, canTransferMetadata, getVectorSplit, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 replaceUses, canTransferMetadata, getVectorSplit，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 634-723

```cpp
// Scalarize one-operand instruction I, using Split(Builder, X, Name)
// to create an instruction like I with operand X and name Name.
template<typename Splitter>
bool ScalarizerVisitor::splitUnary(Instruction &I, const Splitter &Split) {
  std::optional<VectorSplit> VS = getVectorSplit(I.getType());
  if (!VS)
    return false;

  std::optional<VectorSplit> OpVS;
  if (I.getOperand(0)->getType() == I.getType()) {
    OpVS = VS;
  } else {
    OpVS = getVectorSplit(I.getOperand(0)->getType());
    if (!OpVS || VS->NumPacked != OpVS->NumPacked)
      return false;
  }

  IRBuilder<> Builder(&I);
  Scatterer Op = scatter(&I, I.getOperand(0), *OpVS);
  assert(Op.size() == VS->NumFragments && "Mismatched unary operation");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return false;

  Intrinsic::ID ID = F->getIntrinsicID();

  if (ID == Intrinsic::not_intrinsic || !isTriviallyScalarizable(ID))
    return false;

  // unsigned NumElems = VT->getNumElements();
  unsigned NumArgs = CI.arg_size();

  ValueVector ScalarOperands(NumArgs);
  SmallVector<Scatterer, 8> Scattered(NumArgs);
  SmallVector<int> OverloadIdx(NumArgs, -1);

```
- EN: Core entities appearing here include splitUnary, splitBinary, splitCall, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 splitUnary, splitBinary, splitCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 724-811

```cpp
  SmallVector<llvm::Type *, 3> Tys;
  // Add return type if intrinsic is overloaded on it.
  if (isVectorIntrinsicWithOverloadTypeAtArg(ID, -1, TTI))
    Tys.push_back(VS->SplitTy);

  if (AreAllVectorsOfMatchingSize) {
    for (unsigned I = 1; I < CallType->getNumContainedTypes(); I++) {
      std::optional<VectorSplit> CurrVS =
          getVectorSplit(cast<FixedVectorType>(CallType->getContainedType(I)));
      // It is possible for VectorSplit.NumPacked >= NumElems. If that happens a
      // VectorSplit is not returned and we will bailout of handling this call.
      // The secondary bailout case is if NumPacked does not match. This can
      // happen if ScalarizeMinBits is not set to the default. This means with
      // certain ScalarizeMinBits intrinsics like frexp will only scalarize when
      // the struct elements have the same bitness.
      if (!CurrVS || CurrVS->NumPacked != VS->NumPacked)
        return false;
      if (isVectorIntrinsicWithStructReturnOverloadAtField(ID, I, TTI))
        Tys.push_back(CurrVS->SplitTy);
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      }
    }

    if (IsRemainder)
      NewIntrin = Intrinsic::getOrInsertDeclaration(F->getParent(), ID, Tys);

    Res[I] = Builder.CreateCall(NewIntrin, ScalarCallOps,
                                CI.getName() + ".i" + Twine(I));
  }

  gather(&CI, Res, *VS);
  return true;
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 812-898

```cpp
bool ScalarizerVisitor::visitSelectInst(SelectInst &SI) {
  std::optional<VectorSplit> VS = getVectorSplit(SI.getType());
  if (!VS)
    return false;

  std::optional<VectorSplit> CondVS;
  if (isa<FixedVectorType>(SI.getCondition()->getType())) {
    CondVS = getVectorSplit(SI.getCondition()->getType());
    if (!CondVS || CondVS->NumPacked != VS->NumPacked) {
      // This happens when ScalarizeMinBits is used.
      return false;
    }
  }

  IRBuilder<> Builder(&SI);
  Scatterer VOp1 = scatter(&SI, SI.getOperand(1), *VS);
  Scatterer VOp2 = scatter(&SI, SI.getOperand(2), *VS);
  assert(VOp1.size() == VS->NumFragments && "Mismatched select");
  assert(VOp2.size() == VS->NumFragments && "Mismatched select");
  ValueVector Res;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  for (unsigned I = 0; I < 1 + NumIndices; ++I) {
    if (auto *VecTy =
            dyn_cast<FixedVectorType>(GEPI.getOperand(I)->getType())) {
      std::optional<VectorSplit> OpVS = getVectorSplit(VecTy);
      if (!OpVS || OpVS->NumPacked != VS->NumPacked) {
        // This can happen when ScalarizeMinBits is used.
        return false;
      }
      ScatterOps[I] = scatter(&GEPI, GEPI.getOperand(I), *OpVS);
    } else {
      ScalarOps[I] = GEPI.getOperand(I);
    }
  }

```
- EN: Core entities appearing here include visitSelectInst, visitICmpInst, visitFCmpInst, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitSelectInst, visitICmpInst, visitFCmpInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 899-988

```cpp
  ValueVector Res;
  Res.resize(VS->NumFragments);
  for (unsigned I = 0; I < VS->NumFragments; ++I) {
    SmallVector<Value *, 8> SplitOps;
    SplitOps.resize(1 + NumIndices);
    for (unsigned J = 0; J < 1 + NumIndices; ++J) {
      if (ScalarOps[J])
        SplitOps[J] = ScalarOps[J];
      else
        SplitOps[J] = ScatterOps[J][I];
    }
    Res[I] = Builder.CreateGEP(GEPI.getSourceElementType(), SplitOps[0],
                               ArrayRef(SplitOps).drop_front(),
                               GEPI.getName() + ".i" + Twine(I));
    if (GEPI.isInBounds())
      if (GetElementPtrInst *NewGEPI = dyn_cast<GetElementPtrInst>(Res[I]))
        NewGEPI->setIsInBounds();
  }
  gather(&GEPI, Res, *VS);
  return true;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                       MidVS.NumPacked * MidVS.NumFragments);
    MidVS.SplitTy = DstVS->SplitTy;

    unsigned ResI = 0;
    for (unsigned I = 0; I < SrcVS->NumFragments; ++I) {
      Value *V = Op0[I];

      // Look through any existing bitcasts before converting to <N x t2>.
      // In the best case, the resulting conversion might be a no-op.
      Instruction *VI;
      while ((VI = dyn_cast<Instruction>(V)) &&
             VI->getOpcode() == Instruction::BitCast)
        V = VI->getOperand(0);

```
- EN: Core entities appearing here include visitCastInst, visitBitCastInst, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitCastInst, visitBitCastInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 989-1074

```cpp
      V = Builder.CreateBitCast(V, MidVS.VecTy, V->getName() + ".cast");

      Scatterer Mid = scatter(&BCI, V, MidVS);
      for (unsigned J = 0; J < MidVS.NumFragments; ++J)
        Res[ResI++] = Mid[J];
    }
  } else if (DstSplitBits % SrcSplitBits == 0) {
    // Gather enough source fragments to make up a destination fragment and
    // then convert to the destination type.
    VectorSplit MidVS;
    MidVS.NumFragments = DstSplitBits / SrcSplitBits;
    MidVS.NumPacked = SrcVS->NumPacked;
    MidVS.VecTy = FixedVectorType::get(SrcVS->VecTy->getElementType(),
                                       MidVS.NumPacked * MidVS.NumFragments);
    MidVS.SplitTy = SrcVS->SplitTy;

    unsigned SrcI = 0;
    SmallVector<Value *, 8> ConcatOps;
    ConcatOps.resize(MidVS.NumFragments);
    for (unsigned I = 0; I < DstVS->NumFragments; ++I) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    for (unsigned I = 0; I < VS->NumFragments; ++I) {
      Value *ShouldReplace =
          Builder.CreateICmpEQ(InsIdx, ConstantInt::get(InsIdx->getType(), I),
                               InsIdx->getName() + ".is." + Twine(I));
      Value *OldElt = Op0[I];
      Res[I] = Builder.CreateSelect(ShouldReplace, NewElt, OldElt,
                                    IEI.getName() + ".i" + Twine(I));
    }
  }

  gather(&IEI, Res, *VS);
  return true;
}

```
- EN: Core entities appearing here include visitInsertElementInst, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitInsertElementInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1075-1166

```cpp
bool ScalarizerVisitor::visitExtractValueInst(ExtractValueInst &EVI) {
  Value *Op = EVI.getOperand(0);
  Type *OpTy = Op->getType();
  ValueVector Res;
  if (!isStructOfMatchingFixedVectors(OpTy))
    return false;
  if (CallInst *CI = dyn_cast<CallInst>(Op)) {
    Function *F = CI->getCalledFunction();
    if (!F)
      return false;
    Intrinsic::ID ID = F->getIntrinsicID();
    if (ID == Intrinsic::not_intrinsic || !isTriviallyScalarizable(ID))
      return false;
    // Note: Fall through means Operand is a`CallInst` and it is defined in
    // `isTriviallyScalarizable`.
  } else
    return false;
  Type *VecType = cast<FixedVectorType>(OpTy->getContainedType(0));
  std::optional<VectorSplit> VS = getVectorSplit(VecType);
  if (!VS)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  Value *Res = PoisonValue::get(VS->VecTy->getElementType());
  for (unsigned I = 0; I < VS->NumFragments; ++I) {
    Value *ShouldExtract =
        Builder.CreateICmpEQ(ExtIdx, ConstantInt::get(ExtIdx->getType(), I),
                             ExtIdx->getName() + ".is." + Twine(I));
    Value *Elt = Op0[I];
    Res = Builder.CreateSelect(ShouldExtract, Elt, Res,
                               EEI.getName() + ".upto" + Twine(I));
  }
  replaceUses(&EEI, Res);
  return true;
}

```
- EN: Core entities appearing here include visitExtractValueInst, visitExtractElementInst, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitExtractValueInst, visitExtractElementInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1167-1253

```cpp
bool ScalarizerVisitor::visitShuffleVectorInst(ShuffleVectorInst &SVI) {
  std::optional<VectorSplit> VS = getVectorSplit(SVI.getType());
  std::optional<VectorSplit> VSOp =
      getVectorSplit(SVI.getOperand(0)->getType());
  if (!VS || !VSOp || VS->NumPacked > 1 || VSOp->NumPacked > 1)
    return false;

  Scatterer Op0 = scatter(&SVI, SVI.getOperand(0), *VSOp);
  Scatterer Op1 = scatter(&SVI, SVI.getOperand(1), *VSOp);
  ValueVector Res;
  Res.resize(VS->NumFragments);

  for (unsigned I = 0; I < VS->NumFragments; ++I) {
    int Selector = SVI.getMaskValue(I);
    if (Selector < 0)
      Res[I] = PoisonValue::get(VS->VecTy->getElementType());
    else if (unsigned(Selector) < Op0.size())
      Res[I] = Op0[Selector];
    else
      Res[I] = Op1[Selector - Op0.size()];
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

bool ScalarizerVisitor::visitStoreInst(StoreInst &SI) {
  if (!ScalarizeLoadStore)
    return false;
  if (!SI.isSimple())
    return false;

  Value *FullValue = SI.getValueOperand();
  std::optional<VectorLayout> Layout = getVectorLayout(
      FullValue->getType(), SI.getAlign(), SI.getDataLayout());
  if (!Layout)
    return false;

```
- EN: Core entities appearing here include visitShuffleVectorInst, visitPHINode, visitLoadInst, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitShuffleVectorInst, visitPHINode, visitLoadInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1254-1348

```cpp
  IRBuilder<> Builder(&SI);
  Scatterer VPtr = scatter(&SI, SI.getPointerOperand(), Layout->VS);
  Scatterer VVal = scatter(&SI, FullValue, Layout->VS);

  ValueVector Stores;
  Stores.resize(Layout->VS.NumFragments);
  for (unsigned I = 0; I < Layout->VS.NumFragments; ++I) {
    Value *Val = VVal[I];
    Value *Ptr = VPtr[I];
    Stores[I] =
        Builder.CreateAlignedStore(Val, Ptr, Layout->getFragmentAlign(I));
  }
  transferMetadataAndIRFlags(&SI, Stores);
  return true;
}

bool ScalarizerVisitor::visitCallInst(CallInst &CI) {
  return splitCall(CI);
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      } else {
        assert(CV.size() == 1 && Op->getType() == CV[0]->getType());
        Res = CV[0];
        if (Op == Res)
          continue;
      }
      Op->replaceAllUsesWith(Res);
    }
    PotentiallyDeadInstrs.emplace_back(Op);
  }
  Gathered.clear();
  Scattered.clear();
  Scalarized = false;

```
- EN: Core entities appearing here include visitCallInst, visitFreezeInst, finish, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 visitCallInst, visitFreezeInst, finish，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1349-1362

```cpp
  RecursivelyDeleteTriviallyDeadInstructionsPermissive(PotentiallyDeadInstrs);

  return true;
}

PreservedAnalyses ScalarizerPass::run(Function &F, FunctionAnalysisManager &AM) {
  DominatorTree *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  const TargetTransformInfo *TTI = &AM.getResult<TargetIRAnalysis>(F);
  ScalarizerVisitor Impl(DT, TTI, Options);
  bool Changed = Impl.visit(F);
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  return Changed ? PA : PreservedAnalyses::all();
}
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `VectorSplit, Scatterer, FCmpSplitter, ICmpSplitter, UnarySplitter, BinarySplitter, VectorLayout, ScalarizerVisitor` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`VectorSplit, Scatterer, FCmpSplitter, ICmpSplitter, UnarySplitter, BinarySplitter, VectorLayout, ScalarizerVisitor` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstVisitor.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstVisitor.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h`, `llvm/InitializePasses.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h`, `llvm/InitializePasses.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstdint`, `iterator`, `map`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstdint`, `iterator`, `map`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
