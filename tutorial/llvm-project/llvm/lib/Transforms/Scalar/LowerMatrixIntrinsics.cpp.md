# LowerMatrixIntrinsics.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/LowerMatrixIntrinsics.cpp` | `llvm/lib/Transforms/Scalar/LowerMatrixIntrinsics.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements lower matrix intrinsics within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 LowerMatrixIntrinsics 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-187

```cpp
//===- LowerMatrixIntrinsics.cpp -  Lower matrix intrinsics -----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Lower matrix intrinsics to vector operations.
//
// TODO:
//  * Improve fusion:
//   * Support more cases, e.g. multiply-add, multiply-sub, operands/results
//     transposed.
//   * Improve cost-modeling, e.g. choose different number of rows/columns
//     columns for tiles, consider cost of copies on alias.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Scalar/LowerMatrixIntrinsics.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
//                Base   Col 1  Col 2
//                  |     |      |
//         v_1_0 |v_1_1 |v_1_2 |v_1_3
//         v_2_0 |v_2_1 |v_2_2 |v_2_3
//         v_3_0 {v_3_1 {v_3_2  v_3_3
//
static Value *computeVectorAddr(Value *BasePtr, Value *VecIdx, Value *Stride,
                                unsigned NumElements, Type *EltType,
                                IRBuilder<> &Builder) {

  assert((!isa<ConstantInt>(Stride) ||
          cast<ConstantInt>(Stride)->getZExtValue() >= NumElements) &&
         "Stride must be >= the number of elements in the result vector.");

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 188-397

```cpp
  // Compute the start of the vector with index VecIdx as VecIdx * Stride.
  Value *VecStart = Builder.CreateMul(VecIdx, Stride, "vec.start");

  // Get pointer to the start of the selected vector. Skip GEP creation,
  // if we select vector 0.
  if (isa<ConstantInt>(VecStart) && cast<ConstantInt>(VecStart)->isZero())
    VecStart = BasePtr;
  else
    VecStart = Builder.CreateInBoundsGEP(EltType, BasePtr, VecStart, "vec.gep");

  return VecStart;
}

namespace {
struct ShapeInfo {
  unsigned NumRows;
  unsigned NumColumns;

  bool IsColumnMajor;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
///  2.4. Cache the result column matrix for the instruction we lowered
/// 3. After we lowered all instructions in a function, remove the now
///    obsolete instructions.
///
class LowerMatrixIntrinsics {
  Function &Func;
  const DataLayout &DL;
  const TargetTransformInfo &TTI;
  FunctionAnalysisManager *AM;
  AliasAnalysis *AA = nullptr;
  DominatorTree *DT = nullptr;
  LoopInfo *LI = nullptr;
  OptimizationRemarkEmitter *ORE = nullptr;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ShapeInfo, bool, getStride, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, DataLayout, DominatorTree, LoopInfo.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ShapeInfo, bool, getStride，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, DataLayout, DominatorTree, LoopInfo 等分析结果。

### Lines 398-587

```cpp
  /// Contains estimates of the number of operations (loads, stores, compute)
  /// required to lower a matrix operation.
  struct OpInfoTy {
    /// Number of stores emitted to generate this matrix.
    unsigned NumStores = 0;
    /// Number of loads emitted to generate this matrix.
    unsigned NumLoads = 0;
    /// Number of compute operations emitted to generate this matrix.
    unsigned NumComputeOps = 0;
    /// Most of the time transposes can be fused with matrix multiplies or can
    /// be folded away via algebraic simplifications.  This is the number of
    /// transposes that we failed to make "free" via such optimizations.
    unsigned NumExposedTransposes = 0;

    OpInfoTy &operator+=(const OpInfoTy &RHS) {
      NumStores += RHS.NumStores;
      NumLoads += RHS.NumLoads;
      NumComputeOps += RHS.NumComputeOps;
      NumExposedTransposes += RHS.NumExposedTransposes;
      return *this;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  DenseMap<Value *, ShapeInfo> ShapeMap;

  /// List of instructions to remove. While lowering, we are not replacing all
  /// users of a lowered instruction, if shape information is available and
  /// those need to be removed after we finished lowering.
  SmallVector<Instruction *, 16> ToRemove;

  /// Map from instructions to their produced column matrix.
  MapVector<Value *, MatrixTy> Inst2ColumnMatrix;

private:
  static FastMathFlags getFastMathFlags(Instruction *Inst) {
    FastMathFlags FMF;

```
- EN: Core entities appearing here include OpInfoTy, MatrixTy, IsColumnMajor, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 OpInfoTy, MatrixTy, IsColumnMajor，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 588-780

```cpp
    if (isa<FPMathOperator>(*Inst))
      FMF = Inst->getFastMathFlags();

    FMF.setAllowContract(AllowContractEnabled || FMF.allowContract());

    return FMF;
  }

public:
  LowerMatrixIntrinsics(Function &F, TargetTransformInfo &TTI,
                        FunctionAnalysisManager *AM)
      : Func(F), DL(F.getDataLayout()), TTI(TTI), AM(AM) {}

  unsigned getNumOps(Type *VT) {
    assert(isa<FixedVectorType>(VT) && "Expected vector type");
    return getNumOps(VT->getScalarType(),
                     cast<FixedVectorType>(VT)->getNumElements());
  }

  /// Is this the minimal version executed in the backend pipelines.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

      // New entry, set the value and insert operands
      bool Propagate = false;
      if (auto SI = computeShapeInfoForInst(Inst, ShapeMap))
        Propagate = setShapeInfo(Inst, *SI);

      if (Propagate) {
        NewWorkList.push_back(Inst);
        for (auto *User : Inst->users())
          if (ShapeMap.count(User) == 0)
            WorkList.push_back(cast<Instruction>(User));
      }
    }

```
- EN: Core entities appearing here include getNumOps, isMinimal, setShapeInfo, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getNumOps, isMinimal, setShapeInfo，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 781-991

```cpp
    return NewWorkList;
  }

  /// Propagate the shape to operands of instructions with shape information.
  /// \p Worklist contains the instruction for which we already know the shape.
  SmallVector<Instruction *, 32>
  propagateShapeBackward(SmallVectorImpl<Instruction *> &WorkList) {
    SmallVector<Instruction *, 32> NewWorkList;

    auto pushInstruction = [](Value *V,
                              SmallVectorImpl<Instruction *> &WorkList) {
      Instruction *I = dyn_cast<Instruction>(V);
      if (I)
        WorkList.push_back(I);
    };
    // Pop an element with known shape.  Traverse the operands, if their shape
    // derives from the result shape and is unknown, add it and add them to the
    // worklist.
    LLVM_DEBUG(dbgs() << "Backward-propagate shapes:\n");
    while (!WorkList.empty()) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
            bool IsFP = I.getType()->isFPOrFPVectorTy();
            auto *Mul = IsFP ? LocalBuilder.CreateFMul(T0, T1, "mmul")
                             : LocalBuilder.CreateMul(T0, T1, "mmul");
            auto *Result = cast<Instruction>(Mul);
            setShapeInfo(Result, Shape0);
            return Result;
          });
      updateShapeAndReplaceAllUsesWith(I, NewInst);
      eraseFromParentAndMove(&I, II, BB);
      eraseFromParentAndMove(TA, II, BB);
      Changed = true;
      return NewInst;
    }

```
- EN: Core entities appearing here include propagateShapeBackward, m_Value, match, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 propagateShapeBackward, m_Value, match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 992-1178

```cpp
    // (A + B)^t -> A^t + B^t
    // RxC RxC      CxR   CxR
    if (match(TA, m_AnyAdd(m_Value(TAMA), m_Value(TAMB)))) {
      IRBuilder<> LocalBuilder(&I);
      auto NewInst = distributeTransposes(
          TAMA, {R, C}, TAMB, {R, C}, Builder,
          [&](Value *T0, ShapeInfo Shape0, Value *T1, ShapeInfo Shape1) {
            bool IsFP = I.getType()->isFPOrFPVectorTy();
            auto *Add = IsFP ? LocalBuilder.CreateFAdd(T0, T1, "madd")
                             : LocalBuilder.CreateAdd(T0, T1, "madd");

            auto *Result = cast<Instruction>(Add);
            setShapeInfo(Result, Shape0);
            return Result;
          });
      updateShapeAndReplaceAllUsesWith(I, NewInst);
      eraseFromParentAndMove(&I, II, BB);
      eraseFromParentAndMove(TA, II, BB);
      Changed = true;
      return NewInst;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          MaybeFusableInsts.push_back(cast<CallInst>(&I));
        MatrixInsts.push_back(&I);
      }

    // Second, try to lower any dot products
    SmallPtrSet<Instruction *, 16> FusedInsts;
    for (CallInst *CI : MaybeFusableInsts)
      lowerDotProduct(CI, FusedInsts, getFastMathFlags(CI));

    // Third, try to fuse candidates.
    for (CallInst *CI : MaybeFusableInsts)
      if (!FusedInsts.contains(CI))
        LowerMatrixMultiplyFused(CI, FusedInsts, LifetimeEnds);

```
- EN: Core entities appearing here include liftTranspose, match, m_Value, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, DominatorTree, OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 liftTranspose, match, m_Value，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAManager, DominatorTree, OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1179-1374

```cpp
    Changed |= !FusedInsts.empty();

    // Fourth, pre-process all the PHINode's. The incoming values will be
    // assigned later in VisitPHI.
    for (Instruction *Inst : MatrixInsts) {
      if (FusedInsts.count(Inst))
        continue;

      auto *PHI = dyn_cast<PHINode>(Inst);
      if (!PHI)
        continue;

      const ShapeInfo &SI = ShapeMap.at(Inst);
      auto *EltTy = cast<FixedVectorType>(PHI->getType())->getElementType();
      MatrixTy PhiM(SI.NumRows, SI.NumColumns, EltTy);

      IRBuilder<> Builder(Inst);
      for (unsigned VI = 0, VE = PhiM.getNumVectors(); VI != VE; ++VI)
        PhiM.setVector(VI, Builder.CreatePHI(PhiM.getVectorTy(),
                                             PHI->getNumIncomingValues(),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    auto *VType = cast<FixedVectorType>(Ty);
    Type *EltTy = VType->getElementType();
    Type *VecTy = FixedVectorType::get(EltTy, Shape.getStride());
    Value *EltPtr = Ptr;
    MatrixTy Result;
    Stride = castToIndexType(Ptr, Stride, Builder);
    for (unsigned I = 0, E = Shape.getNumVectors(); I < E; ++I) {
      Value *GEP = computeVectorAddr(
          EltPtr, Builder.getIntN(Stride->getType()->getScalarSizeInBits(), I),
          Stride, Shape.getStride(), EltTy, Builder);
      Value *Vector = Builder.CreateAlignedLoad(
          VecTy, GEP, getAlignForIndex(I, Stride, EltTy, MAlign),
          IsVolatile, "col.load");

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1375-1572

```cpp
      Result.addVector(Vector);
    }
    return Result.addNumLoads(getNumOps(Result.getVectorTy()) *
                              Result.getNumVectors());
  }

  /// Loads a sub-matrix with shape \p ResultShape from a \p R x \p C matrix,
  /// starting at \p MatrixPtr[I][J].
  MatrixTy loadMatrix(Value *MatrixPtr, MaybeAlign Align, bool IsVolatile,
                      ShapeInfo MatrixShape, Value *I, Value *J,
                      ShapeInfo ResultShape, Type *EltTy,
                      IRBuilder<> &Builder) {
    Value *Offset = Builder.CreateAdd(
        Builder.CreateMul(J, getIndex(MatrixPtr, MatrixShape.getStride())), I);

    Value *TileStart = Builder.CreateInBoundsGEP(EltTy, MatrixPtr, Offset);
    auto *TileTy = FixedVectorType::get(EltTy, ResultShape.NumRows *
                                                   ResultShape.NumColumns);

    return loadMatrix(TileTy, TileStart, Align,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        Flattened = Matrix.embedInVector(Builder);
        LLVM_DEBUG(
            if (Instruction *User = dyn_cast<Instruction>(U.getUser())) dbgs()
                << "flattening a " << Matrix.shape() << " matrix:\n"
                << *Inst
                << "\nbecause we do not have a shape-aware lowering for its "
                   "user:\n"
                << *User << '\n';);
        FlattenedMatrices++;
      }
      U.set(Flattened);
    }
  }

```
- EN: Core entities appearing here include LowerColumnMajorLoad, LowerColumnMajorStore, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 LowerColumnMajorLoad, LowerColumnMajorStore，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1573-1762

```cpp
  /// Special case for MatMul lowering. Prevents scalar loads of row-major
  /// vectors Lowers to vector reduction add instead of sequential add if
  /// reassocation is enabled.
  void lowerDotProduct(CallInst *MatMul,
                       SmallPtrSet<Instruction *, 16> &FusedInsts,
                       FastMathFlags FMF) {
    if (FusedInsts.contains(MatMul) ||
        MatrixLayout != MatrixLayoutTy::ColumnMajor)
      return;
    ShapeInfo LShape(MatMul->getArgOperand(2), MatMul->getArgOperand(3));
    ShapeInfo RShape(MatMul->getArgOperand(3), MatMul->getArgOperand(4));

    if (LShape.NumRows != 1 || RShape.NumColumns != 1) // not a dot product
      return;

    Value *LHS = MatMul->getArgOperand(0);
    Value *RHS = MatMul->getArgOperand(1);

    Type *ElementType = cast<FixedVectorType>(LHS->getType())->getElementType();
    bool IsIntVec = ElementType->isIntegerTy();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          ConstantFP::get(
              cast<FixedVectorType>(LHS->getType())->getElementType(), 0.0),
          Mul);
      cast<Instruction>(Result)->setFastMathFlags(FMF);
    }

    // pack scalar back into a matrix and then replace matmul inst
    Result = Builder.CreateInsertElement(PoisonValue::get(MatMul->getType()),
                                         Result, uint64_t(0));
    MatMul->replaceAllUsesWith(Result);
    FusedInsts.insert(MatMul);
    ToRemove.push_back(MatMul);
  }

```
- EN: Core entities appearing here include m_Value, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 m_Value，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1763-1954

```cpp
  /// Given \p Remainder iterations of the the matmul inner loop,
  /// potentially lower \p Blocksize that is used for the underlying
  /// vector.
  unsigned capBlockSize(unsigned BlockSize, unsigned Remainder, Type *EltType) {
    if (BlockSize <= Remainder)
      return BlockSize;

    // If the remainder is also a legal type just use it.
    auto *VecTy = FixedVectorType::get(EltType, Remainder);
    if (TTI.isTypeLegal(VecTy))
      return Remainder;

    // Similarly, if the vector is small enough that we don't want
    // to split further.
    if (VecTy->getPrimitiveSizeInBits() <= SplitMatmulRemainderOverThreshold)
      return Remainder;

    // Gradually lower the vectorization factor to cover the
    // remainder.
    do {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Check if the store begins before the end of the load location. If the
    // condition holds, they alias, otherwise they are guaranteed to not
    // overlap.
    Check1->getTerminator()->eraseFromParent();
    Builder.SetInsertPoint(Check1, Check1->begin());

    auto *VT = cast<FixedVectorType>(Load->getType());
    // Use an array type for the alloca, to avoid potentially huge alignment
    // requirements for large vector types.
    auto *ArrayTy = ArrayType::get(VT->getElementType(), VT->getNumElements());
    AllocaInst *Alloca =
        AllocaBuilder.CreateAlloca(ArrayTy, Load->getPointerAddressSpace());
    Builder.CreateLifetimeStart(Alloca);

```
- EN: Core entities appearing here include capBlockSize, getNonAliasingPointer, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, DomTreeUpdater, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 capBlockSize, getNonAliasingPointer，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, DomTreeUpdater, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1955-2146

```cpp
    Value *LoadEnd = Builder.CreatePtrAdd(
        LoadBegin, ConstantInt::get(AddrTy, LoadLoc.Size.getValue()),
        "load.end",
        GEPNoWrapFlags::inBounds() | GEPNoWrapFlags::noUnsignedWrap());
    CondBrInst *BR2 = Builder.CreateCondBr(
        Builder.CreateICmpULT(StoreBegin, LoadEnd), Copy, Fusion);
    setExplicitlyUnknownBranchWeightsIfProfiled(*BR2, DEBUG_TYPE);

    // Copy load operand to new alloca.
    Builder.SetInsertPoint(Copy, Copy->begin());
    Builder.CreateMemCpy(Alloca, Alloca->getAlign(), Load->getPointerOperand(),
                         Load->getAlign(), LoadLoc.Size.getValue());
    Builder.SetInsertPoint(Fusion, Fusion->begin());
    PHINode *PHI = Builder.CreatePHI(Load->getPointerOperandType(), 3);
    PHI->addIncoming(Load->getPointerOperand(), Check0);
    PHI->addIncoming(Load->getPointerOperand(), Check1);
    PHI->addIncoming(Alloca, Copy);

    // Adjust DT.
    DTUpdates.push_back({DT->Insert, Check0, Check1});
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          storeMatrix(Res, CPtr, Store->getAlign(), Store->isVolatile(), {R, M},
                      getIndex(CPtr, I), getIndex(CPtr, J), EltType, Builder);
        }
    }

    // End the lifetime of the allocas used for alias-safe copies.
    {
      IRBuilder<> Builder(Store);
      if (AAlloca)
        Builder.CreateLifetimeEnd(AAlloca);
      if (BAlloca)
        Builder.CreateLifetimeEnd(BAlloca);
    }

```
- EN: Core entities appearing here include isFusionProfitable, getZeroMatrix, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DomTreeUpdater, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isFusionProfitable, getZeroMatrix，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DomTreeUpdater, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2147-2337

```cpp
    // Mark eliminated instructions as fused and remove them.
    FusedInsts.insert(Store);
    FusedInsts.insert(MatMul);
    eraseFromParentAndRemoveFromShapeMap(Store);
    eraseFromParentAndRemoveFromShapeMap(MatMul);
    if (LoadOp0->use_empty()) {
      FusedInsts.insert(LoadOp0);
      eraseFromParentAndRemoveFromShapeMap(LoadOp0);
    }
    if (LoadOp1 != LoadOp0 && LoadOp1->use_empty()) {
      FusedInsts.insert(LoadOp1);
      eraseFromParentAndRemoveFromShapeMap(LoadOp1);
    }
  }

  /// Try to lower matrix multiply chains by fusing operations.
  ///
  /// Call finalizeLowering on lowered instructions.  Instructions that are
  /// completely eliminated by fusion are added to \p FusedInsts.
  void
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    const unsigned R = LShape.NumRows;
    const unsigned C = RShape.NumColumns;
    assert(LShape.NumColumns == RShape.NumRows);

    // Initialize the output
    MatrixTy Result(R, C, EltType);
    assert(Lhs.getElementType() == Result.getElementType() &&
           "Matrix multiply result element type does not match arguments.");

    emitMatrixMultiply(Result, Lhs, Rhs, Builder, false, false,
                       getFastMathFlags(MatMul));
    return Result;
  }

```
- EN: Core entities appearing here include match, sort, LowerMultiply, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match, sort, LowerMultiply，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2338-2528

```cpp
  /// Lowers llvm.matrix.transpose.
  MatrixTy LowerTranspose(CallInst *Inst, IRBuilder<> &Builder) {
    MatrixTy Result;
    Value *InputVal = Inst->getArgOperand(0);
    FixedVectorType *VectorTy = cast<FixedVectorType>(InputVal->getType());
    ShapeInfo ArgShape(Inst->getArgOperand(1), Inst->getArgOperand(2));
    MatrixTy InputMatrix = getMatrix(InputVal, ArgShape, Builder);

    const unsigned NewNumVecs =
        InputMatrix.isColumnMajor() ? ArgShape.NumRows : ArgShape.NumColumns;
    const unsigned NewNumElts =
        InputMatrix.isColumnMajor() ? ArgShape.NumColumns : ArgShape.NumRows;

    for (unsigned I = 0; I < NewNumVecs; ++I) {
      // Build a single result vector. First initialize it.
      Value *ResultVector = PoisonValue::get(
          FixedVectorType::get(VectorTy->getElementType(), NewNumElts));
      // Go through the old elements and insert it into the resulting vector.
      for (auto J : enumerate(InputMatrix.vectors())) {
        Value *Elt = Builder.CreateExtractElement(J.value(), I);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return Result.addNumComputeOps(getNumOps(Result.getVectorTy()) *
                                   Result.getNumVectors());
  }

  /// Helper to linearize a matrix expression tree into a string. Currently
  /// matrix expressions are linarized by starting at an expression leaf and
  /// linearizing bottom up.
  struct ExprLinearizer {
    unsigned LengthToBreak = 100;
    std::string Str;
    raw_string_ostream Stream;
    unsigned LineLength = 0;
    const DataLayout &DL;

```
- EN: Core entities appearing here include LowerTranspose, VisitPHI, zip_equal, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 LowerTranspose, VisitPHI, zip_equal，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2529-2719

```cpp
    /// Mapping from instructions to matrixes. It is used to identify
    /// matrix instructions.
    const MapVector<Value *, MatrixTy> &Inst2Matrix;

    /// Mapping from values to the leaves of all expressions that the value is
    /// part of.
    const DenseMap<Value *, SmallPtrSet<Value *, 2>> &Shared;

    /// Set of matrix expressions in the scope of a given DISubprogram.
    const SmallSetVector<Value *, 32> &ExprsInSubprogram;

    /// Leaf node of the expression to linearize.
    Value *Leaf;

    /// Used to keep track of sub-expressions that get reused while linearizing
    /// the expression. Re-used sub-expressions are marked as (reused).
    SmallPtrSet<Value *, 8> ReusedExprs;

    ExprLinearizer(const DataLayout &DL,
                   const MapVector<Value *, MatrixTy> &Inst2Matrix,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    void linearizeExpr(Value *Expr, unsigned Indent, bool ParentReused,
                       bool ParentShared) {
      auto *I = cast<Instruction>(Expr);
      maybeIndent(Indent);
      SmallVector<Value *, 8> Ops;

      // Is Expr shared with other expression leaves?
      bool ExprShared = false;

      // Deal with shared subtrees. Mark them as shared, if required.
      if (!ParentShared) {
        auto SI = Shared.find(Expr);
        assert(SI != Shared.end() && SI->second.count(Leaf));

```
- EN: Core entities appearing here include indent, lineBreak, maybeIndent, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 indent, lineBreak, maybeIndent，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 2720-2909

```cpp
        for (Value *S : SI->second) {
          if (S == Leaf)
            continue;
          DebugLoc DL = cast<Instruction>(S)->getDebugLoc();
          write("shared with remark at line " + std::to_string(DL.getLine()) +
                " column " + std::to_string(DL.getCol()) + " (");
        }
        ExprShared = SI->second.size() > 1;
      }

      bool Reused = !ReusedExprs.insert(Expr).second;
      if (Reused && !ParentReused)
        write("(reused) ");

      if (auto *CI = dyn_cast<CallInst>(I)) {
        writeFnName(CI);

        Ops.append(CI->arg_begin(), CI->arg_end() - getNumShapeArgs(CI));
      } else if (isa<BitCastInst>(Expr)) {
        // Special case bitcasts, which are used to materialize matrixes from
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          DILocation *Context = cast<Instruction>(L)->getDebugLoc();
          while (Context) {
            if (getSubprogram(Context->getScope()) == KV.first) {
              Loc = Context;
              break;
            }
            Context = DebugLoc(Context).getInlinedAt();
          }

          SmallPtrSet<Value *, 8> ReusedExprs;
          OpInfoTy Counts, SharedCounts;
          std::tie(Counts, SharedCounts) =
              sumOpInfos(L, ReusedExprs, ExprsInSubprogram, Shared);

```
- EN: Core entities appearing here include RemarkGenerator, getExpressionLeaves, emitRemarks, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, OptimizationRemarkEmitter. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 RemarkGenerator, getExpressionLeaves, emitRemarks，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, OptimizationRemarkEmitter 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2910-2974

```cpp
          OptimizationRemark Rem(DEBUG_TYPE, "matrix-lowered", Loc,
                                 cast<Instruction>(L)->getParent());

          Rem << "Lowered with ";
          Rem << ore::NV("NumStores", Counts.NumStores) << " stores, "
              << ore::NV("NumLoads", Counts.NumLoads) << " loads, "
              << ore::NV("NumComputeOps", Counts.NumComputeOps)
              << " compute ops, "
              << ore::NV("NumExposedTransposes", Counts.NumExposedTransposes)
              << " exposed transposes";

          if (SharedCounts.NumStores > 0 || SharedCounts.NumLoads > 0 ||
              SharedCounts.NumComputeOps > 0) {
            Rem << ",\nadditionally "
                << ore::NV("NumStores", SharedCounts.NumStores) << " stores, "
                << ore::NV("NumLoads", SharedCounts.NumLoads) << " loads, "
                << ore::NV("NumFPOps", SharedCounts.NumComputeOps)
                << " compute ops"
                << " are shared with other expressions";
          }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return PA;
  }
  return PreservedAnalyses::all();
}

void LowerMatrixIntrinsicsPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<LowerMatrixIntrinsicsPass> *>(this)->printPipeline(
      OS, MapClassName2PassName);
  OS << '<';
  if (Minimal)
    OS << "minimal";
  OS << '>';
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `MatrixLayoutTy, ShapeInfo, LowerMatrixIntrinsics, OpInfoTy, MatrixTy, ExprLinearizer, RemarkGenerator, isSplat` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`MatrixLayoutTy, ShapeInfo, LowerMatrixIntrinsics, OpInfoTy, MatrixTy, ExprLinearizer, RemarkGenerator, isSplat` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAManager, AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, OptimizationRemarkEmitter, TargetTransformInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAManager, AliasAnalysis, DataLayout, DominatorTree, DomTreeUpdater, LoopInfo, OptimizationRemarkEmitter, TargetTransformInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `FuseMatrix, TileSize, TileLoopsThreshold, ForceFusion, AllowContractEnabled` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `FuseMatrix, TileSize, TileLoopsThreshold, ForceFusion, AllowContractEnabled` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/DomTreeUpdater.h`, `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/CFG.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MatrixBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/CFG.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/MatrixBuilder.h`, `llvm/IR/PatternMatch.h`, `llvm/IR/ProfDataUtils.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Alignment.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Alignment.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Compiler.h`, `llvm/Support/Debug.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/ScopeExit.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAManager`, `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `OptimizationRemarkEmitter`, `TargetTransformInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAManager`, `AliasAnalysis`, `DataLayout`, `DominatorTree`, `DomTreeUpdater`, `LoopInfo`, `OptimizationRemarkEmitter`, `TargetTransformInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
