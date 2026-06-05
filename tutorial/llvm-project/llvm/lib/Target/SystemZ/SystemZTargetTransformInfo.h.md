# SystemZTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZTargetTransformInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides target-specific optimization cost modelling for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责提供目标相关的优化开销建模。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZTargetTransformInfo.h - SystemZ-specific TTI ---------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETTRANSFORMINFO_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETTRANSFORMINFO_H
  11: 
  12: #include "SystemZTargetMachine.h"
  13: #include "llvm/Analysis/TargetTransformInfo.h"
  14: #include "llvm/CodeGen/BasicTTIImpl.h"
  15: 
  16: namespace llvm {
  17: 
  18: class SystemZTTIImpl final : public BasicTTIImplBase<SystemZTTIImpl> {
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZTargetMachine.h`, `TargetTransformInfo.h`, `BasicTTIImpl.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZTTIImpl`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZTargetMachine.h`, `TargetTransformInfo.h`, `BasicTTIImpl.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZTTIImpl` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```cpp
  19:   typedef BasicTTIImplBase<SystemZTTIImpl> BaseT;
  20:   typedef TargetTransformInfo TTI;
  21:   friend BaseT;
  22: 
  23:   const SystemZSubtarget *ST;
  24:   const SystemZTargetLowering *TLI;
  25: 
  26:   const SystemZSubtarget *getST() const { return ST; }
  27:   const SystemZTargetLowering *getTLI() const { return TLI; }
  28: 
  29:   unsigned const LIBCALL_COST = 30;
  30: 
  31:   bool isInt128InVR(Type *Ty) const {
  32:     return Ty->isIntegerTy(128) && ST->hasVector();
  33:   }
  34: 
  35: public:
  36:   explicit SystemZTTIImpl(const SystemZTargetMachine *TM, const Function &F)
```
- **EN**: The range implements or declares functions including `isInt128InVR`, `SystemZTTIImpl`.
- **CN**: 这一段实现或声明了 `isInt128InVR`, `SystemZTTIImpl` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:       : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
  38:         TLI(ST->getTargetLowering()) {}
  39: 
  40:   /// \name Scalar TTI Implementations
  41:   /// @{
  42: 
  43:   unsigned adjustInliningThreshold(const CallBase *CB) const override;
  44: 
  45:   InstructionCost getIntImmCost(const APInt &Imm, Type *Ty,
  46:                                 TTI::TargetCostKind CostKind) const override;
  47: 
  48:   InstructionCost getIntImmCostInst(unsigned Opcode, unsigned Idx,
  49:                                     const APInt &Imm, Type *Ty,
  50:                                     TTI::TargetCostKind CostKind,
  51:                                     Instruction *Inst = nullptr) const override;
  52:   InstructionCost
  53:   getIntImmCostIntrin(Intrinsic::ID IID, unsigned Idx, const APInt &Imm,
  54:                       Type *Ty, TTI::TargetCostKind CostKind) const override;
```
- **EN**: The range implements or declares functions including `BaseT`.
- **CN**: 这一段实现或声明了 `BaseT` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56:   TTI::PopcntSupportKind getPopcntSupport(unsigned TyWidth) const override;
  57: 
  58:   void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
  59:                                TTI::UnrollingPreferences &UP,
  60:                                OptimizationRemarkEmitter *ORE) const override;
  61: 
  62:   void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
  63:                              TTI::PeelingPreferences &PP) const override;
  64: 
  65:   bool isLSRCostLess(const TargetTransformInfo::LSRCost &C1,
  66:                      const TargetTransformInfo::LSRCost &C2) const override;
  67: 
  68:   /// @}
  69: 
  70:   /// \name Vector TTI Implementations
  71:   /// @{
  72: 
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   unsigned getNumberOfRegisters(unsigned ClassID) const override;
  74:   TypeSize
  75:   getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override;
  76: 
  77:   unsigned getCacheLineSize() const override { return 256; }
  78:   unsigned getPrefetchDistance() const override { return 4500; }
  79:   unsigned getMinPrefetchStride(unsigned NumMemAccesses,
  80:                                 unsigned NumStridedMemAccesses,
  81:                                 unsigned NumPrefetches,
  82:                                 bool HasCall) const override;
  83:   bool enableWritePrefetching() const override { return true; }
  84: 
  85:   unsigned getMaxInterleaveFactor(ElementCount VF) const override;
  86: 
  87:   bool hasDivRemOp(Type *DataType, bool IsSigned) const override;
  88:   bool prefersVectorizedAddressing() const override { return false; }
  89:   bool LSRWithInstrQueries() const override { return true; }
  90:   InstructionCost
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   getScalarizationOverhead(VectorType *Ty, const APInt &DemandedElts,
  92:                            bool Insert, bool Extract,
  93:                            TTI::TargetCostKind CostKind,
  94:                            bool ForPoisonSrc = true, ArrayRef<Value *> VL = {},
  95:                            TTI::VectorInstrContext VIC =
  96:                                TTI::VectorInstrContext::None) const override;
  97:   bool supportsEfficientVectorElementLoadStore() const override { return true; }
  98:   bool enableInterleavedAccessVectorization() const override { return true; }
  99: 
 100:   InstructionCost getArithmeticInstrCost(
 101:       unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
 102:       TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
 103:       TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
 104:       ArrayRef<const Value *> Args = {},
 105:       const Instruction *CxtI = nullptr) const override;
 106: 
 107:   InstructionCost getPartialReductionCost(
 108:       unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:       ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
 110:       TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
 111:       TTI::TargetCostKind CostKind,
 112:       std::optional<FastMathFlags> FMF) const override {
 113:     return InstructionCost::getInvalid();
 114:   }
 115: 
 116:   InstructionCost
 117:   getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy, VectorType *SrcTy,
 118:                  ArrayRef<int> Mask, TTI::TargetCostKind CostKind, int Index,
 119:                  VectorType *SubTp, ArrayRef<const Value *> Args = {},
 120:                  const Instruction *CxtI = nullptr) const override;
 121:   unsigned getVectorTruncCost(Type *SrcTy, Type *DstTy) const;
 122:   unsigned getVectorBitmaskConversionCost(Type *SrcTy, Type *DstTy) const;
 123:   unsigned getBoolVecToIntConversionCost(unsigned Opcode, Type *Dst,
 124:                                          const Instruction *I) const;
 125:   InstructionCost
 126:   getCastInstrCost(unsigned Opcode, Type *Dst, Type *Src,
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127:                    TTI::CastContextHint CCH, TTI::TargetCostKind CostKind,
 128:                    const Instruction *I = nullptr) const override;
 129:   InstructionCost getCFInstrCost(unsigned Opcode, TTI::TargetCostKind CostKind,
 130:                                  const Instruction *I = nullptr) const override;
 131:   InstructionCost getCmpSelInstrCost(
 132:       unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
 133:       TTI::TargetCostKind CostKind,
 134:       TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
 135:       TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
 136:       const Instruction *I = nullptr) const override;
 137:   using BaseT::getVectorInstrCost;
 138:   InstructionCost
 139:   getVectorInstrCost(unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind,
 140:                      unsigned Index, const Value *Op0, const Value *Op1,
 141:                      TTI::VectorInstrContext VIC =
 142:                          TTI::VectorInstrContext::None) const override;
 143:   bool isFoldableLoad(const LoadInst *Ld,
 144:                       const Instruction *&FoldedValue) const;
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   InstructionCost getMemoryOpCost(
 146:       unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
 147:       TTI::TargetCostKind CostKind,
 148:       TTI::OperandValueInfo OpInfo = {TTI::OK_AnyValue, TTI::OP_None},
 149:       const Instruction *I = nullptr) const override;
 150: 
 151:   InstructionCost getInterleavedMemoryOpCost(
 152:       unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
 153:       Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
 154:       bool UseMaskForCond = false, bool UseMaskForGaps = false) const override;
 155: 
 156:   InstructionCost
 157:   getArithmeticReductionCost(unsigned Opcode, VectorType *Ty,
 158:                              std::optional<FastMathFlags> FMF,
 159:                              TTI::TargetCostKind CostKind) const override;
 160:   InstructionCost
 161:   getMinMaxReductionCost(Intrinsic::ID IID, VectorType *Ty, FastMathFlags FMF,
 162:                          TTI::TargetCostKind CostKind) const override;
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-178 / 第 163-178 行
```cpp
 163: 
 164:   InstructionCost
 165:   getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
 166:                         TTI::TargetCostKind CostKind) const override;
 167: 
 168:   bool preferEpilogueVectorization(ElementCount Iters) const override {
 169:     return true;
 170:   }
 171: 
 172:   bool shouldExpandReduction(const IntrinsicInst *II) const override;
 173:   /// @}
 174: };
 175: 
 176: } // end namespace llvm
 177: 
 178: #endif
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZTargetMachine.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/BasicTTIImpl.h`
