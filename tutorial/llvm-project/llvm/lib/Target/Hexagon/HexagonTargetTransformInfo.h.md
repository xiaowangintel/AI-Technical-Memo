# HexagonTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonTargetTransformInfo.h`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file implements a TargetTransformInfo analysis pass specific to the
- Purpose / 作用 (CN): 该文件声明 Hexagon 后端组件的接口、类型或辅助入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //==- HexagonTargetTransformInfo.cpp - Hexagon specific TTI pass -*- C++ -*-==//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: /// \file
     8: /// This file implements a TargetTransformInfo analysis pass specific to the
     9: /// Hexagon target machine. It uses the target's detailed information to provide
    10: /// more precise answers to certain TTI queries, while letting the target
    11: /// independent and default TTI implementations handle the rest.
    12: ///
    13: //===----------------------------------------------------------------------===//
    14: 
    15: #ifndef LLVM_LIB_TARGET_HEXAGON_HEXAGONTARGETTRANSFORMINFO_H
    16: #define LLVM_LIB_TARGET_HEXAGON_HEXAGONTARGETTRANSFORMINFO_H
    17: 
    18: #include "Hexagon.h"
    19: #include "HexagonSubtarget.h"
    20: #include "HexagonTargetMachine.h"
    21: #include "llvm/ADT/ArrayRef.h"
    22: #include "llvm/Analysis/TargetTransformInfo.h"
    23: #include "llvm/CodeGen/BasicTTIImpl.h"
    24: #include "llvm/IR/Function.h"
    25: 
```
- EN: It imports headers such as Hexagon.h, HexagonSubtarget.h, HexagonTargetMachine.h, llvm/ADT/ArrayRef.h, ... (7 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetTransformInfo, HexagonSubtarget, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里引入了 Hexagon.h, HexagonSubtarget.h, HexagonTargetMachine.h, llvm/ADT/ArrayRef.h, ... (7 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetTransformInfo, HexagonSubtarget, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: namespace llvm {
    27: 
    28: class Loop;
    29: class ScalarEvolution;
    30: class User;
    31: class Value;
    32: 
    33: class HexagonTTIImpl final : public BasicTTIImplBase<HexagonTTIImpl> {
    34:   using BaseT = BasicTTIImplBase<HexagonTTIImpl>;
    35:   using TTI = TargetTransformInfo;
    36: 
    37:   friend BaseT;
    38: 
    39:   const HexagonSubtarget &ST;
    40:   const HexagonTargetLowering &TLI;
    41: 
    42:   const HexagonSubtarget *getST() const { return &ST; }
    43:   const HexagonTargetLowering *getTLI() const { return &TLI; }
    44: 
    45:   bool useHVX() const;
    46:   bool isHVXVectorType(Type *Ty) const;
    47: 
    48:   // Returns the number of vector elements of Ty, if Ty is a vector type,
    49:   // or 1 if Ty is a scalar type. It is incorrect to call this function
    50:   // with any other type.
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as Loop, ScalarEvolution, User, Value, ... (5 total), which carry the state or API of this component. It defines declarative TableGen records like Loop, ScalarEvolution, User, Value, HexagonTTIImpl; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as getST, getTLI, useHVX, isHVXVectorType, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 Loop, ScalarEvolution, User, Value, ... (5 total) 等类型，用来承载该组件的状态或接口。 这里定义了 Loop, ScalarEvolution, User, Value, HexagonTTIImpl 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 getST, getTLI, useHVX, isHVXVectorType 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-75 / 第 51-75 行

```cpp
    51:   unsigned getTypeNumElements(Type *Ty) const;
    52: 
    53: public:
    54:   explicit HexagonTTIImpl(const HexagonTargetMachine *TM, const Function &F)
    55:       : BaseT(TM, F.getDataLayout()),
    56:         ST(*TM->getSubtargetImpl(F)), TLI(*ST.getTargetLowering()) {}
    57: 
    58:   /// \name Scalar TTI Implementations
    59:   /// @{
    60: 
    61:   TTI::PopcntSupportKind
    62:   getPopcntSupport(unsigned IntTyWidthInBit) const override;
    63: 
    64:   // The Hexagon target can unroll loops with run-time trip counts.
    65:   void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
    66:                                TTI::UnrollingPreferences &UP,
    67:                                OptimizationRemarkEmitter *ORE) const override;
    68: 
    69:   void getPeelingPreferences(Loop *L, ScalarEvolution &SE,
    70:                              TTI::PeelingPreferences &PP) const override;
    71: 
    72:   /// Bias LSR towards creating post-increment opportunities.
    73:   TTI::AddressingModeKind
    74:   getPreferredAddressingMode(const Loop *L, ScalarEvolution *SE) const override;
    75: 
```
- EN: It declares or implements routines such as getTypeNumElements, HexagonTTIImpl, getPopcntSupport, getUnrollingPreferences, ... (6 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonTTIImpl, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getTypeNumElements, HexagonTTIImpl, getPopcntSupport, getUnrollingPreferences, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonTTIImpl, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   // L1 cache prefetch.
    77:   unsigned getPrefetchDistance() const override;
    78:   unsigned getCacheLineSize() const override;
    79: 
    80:   /// @}
    81: 
    82:   /// \name Vector TTI Implementations
    83:   /// @{
    84: 
    85:   unsigned getNumberOfRegisters(unsigned ClassID) const override;
    86:   unsigned getMaxInterleaveFactor(ElementCount VF) const override;
    87:   TypeSize
    88:   getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override;
    89:   unsigned getMinVectorRegisterBitWidth() const override;
    90:   ElementCount getMinimumVF(unsigned ElemWidth, bool IsScalable) const override;
    91: 
    92:   bool shouldMaximizeVectorBandwidth(
    93:       TargetTransformInfo::RegisterKind K) const override {
    94:     return true;
    95:   }
    96:   bool supportsEfficientVectorElementLoadStore() const override {
    97:     return false;
    98:   }
    99:   bool hasBranchDivergence(const Function *F = nullptr) const override {
   100:     return false;
```
- EN: It declares or implements routines such as getPrefetchDistance, getCacheLineSize, getNumberOfRegisters, getMaxInterleaveFactor, ... (10 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 getPrefetchDistance, getCacheLineSize, getNumberOfRegisters, getMaxInterleaveFactor, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   }
   102:   bool enableAggressiveInterleaving(bool LoopHasReductions) const override {
   103:     return false;
   104:   }
   105:   bool prefersVectorizedAddressing() const override { return false; }
   106:   bool enableInterleavedAccessVectorization() const override { return true; }
   107: 
   108:   InstructionCost getCallInstrCost(Function *F, Type *RetTy,
   109:                                    ArrayRef<Type *> Tys,
   110:                                    TTI::TargetCostKind CostKind) const override;
   111:   InstructionCost
   112:   getIntrinsicInstrCost(const IntrinsicCostAttributes &ICA,
   113:                         TTI::TargetCostKind CostKind) const override;
   114:   InstructionCost
   115:   getAddressComputationCost(Type *PtrTy, ScalarEvolution *SE, const SCEV *S,
   116:                             TTI::TargetCostKind CostKind) const override;
   117:   InstructionCost getMemoryOpCost(
   118:       unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
   119:       TTI::TargetCostKind CostKind,
   120:       TTI::OperandValueInfo OpInfo = {TTI::OK_AnyValue, TTI::OP_None},
   121:       const Instruction *I = nullptr) const override;
   122:   InstructionCost
   123:   getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy, VectorType *SrcTy,
   124:                  ArrayRef<int> Mask, TTI::TargetCostKind CostKind, int Index,
   125:                  VectorType *SubTp, ArrayRef<const Value *> Args = {},
```
- EN: It declares or implements routines such as enableAggressiveInterleaving, prefersVectorizedAddressing, enableInterleavedAccessVectorization, getCallInstrCost, ... (6 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 enableAggressiveInterleaving, prefersVectorizedAddressing, enableInterleavedAccessVectorization, getCallInstrCost, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 126-150 / 第 126-150 行

```cpp
   126:                  const Instruction *CxtI = nullptr) const override;
   127:   InstructionCost getInterleavedMemoryOpCost(
   128:       unsigned Opcode, Type *VecTy, unsigned Factor, ArrayRef<unsigned> Indices,
   129:       Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
   130:       bool UseMaskForCond = false, bool UseMaskForGaps = false) const override;
   131:   InstructionCost getCmpSelInstrCost(
   132:       unsigned Opcode, Type *ValTy, Type *CondTy, CmpInst::Predicate VecPred,
   133:       TTI::TargetCostKind CostKind,
   134:       TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
   135:       TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
   136:       const Instruction *I = nullptr) const override;
   137:   InstructionCost getArithmeticInstrCost(
   138:       unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
   139:       TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
   140:       TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
   141:       ArrayRef<const Value *> Args = {},
   142:       const Instruction *CxtI = nullptr) const override;
   143:   InstructionCost
   144:   getCastInstrCost(unsigned Opcode, Type *Dst, Type *Src,
   145:                    TTI::CastContextHint CCH, TTI::TargetCostKind CostKind,
   146:                    const Instruction *I = nullptr) const override;
   147:   using BaseT::getVectorInstrCost;
   148:   InstructionCost
   149:   getVectorInstrCost(unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind,
   150:                      unsigned Index, const Value *Op0, const Value *Op1,
```
- EN: It declares or implements routines such as getInterleavedMemoryOpCost, getCastInstrCost, translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 getInterleavedMemoryOpCost, getCastInstrCost 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 151-175 / 第 151-175 行

```cpp
   151:                      TTI::VectorInstrContext VIC =
   152:                          TTI::VectorInstrContext::None) const override;
   153: 
   154:   InstructionCost
   155:   getCFInstrCost(unsigned Opcode, TTI::TargetCostKind CostKind,
   156:                  const Instruction *I = nullptr) const override {
   157:     return 1;
   158:   }
   159:   bool shouldExpandReduction(const IntrinsicInst *II) const override;
   160:   bool isLegalMaskedStore(Type *DataType, Align Alignment,
   161:                           unsigned AddressSpace,
   162:                           TTI::MaskKind MaskKind) const override;
   163:   bool isLegalMaskedLoad(Type *DataType, Align Alignment, unsigned AddressSpace,
   164:                          TTI::MaskKind MaskKind) const override;
   165:   bool isLegalMaskedGather(Type *Ty, Align Alignment) const override;
   166:   bool isLegalMaskedScatter(Type *Ty, Align Alignment) const override;
   167:   bool forceScalarizeMaskedGather(VectorType *VTy,
   168:                                   Align Alignment) const override;
   169:   bool forceScalarizeMaskedScatter(VectorType *VTy,
   170:                                    Align Alignment) const override;
   171: 
   172:   InstructionCost getPartialReductionCost(
   173:       unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
   174:       ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
   175:       TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
```
- EN: It declares or implements routines such as getCFInstrCost, shouldExpandReduction, isLegalMaskedStore, isLegalMaskedLoad, ... (8 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 getCFInstrCost, shouldExpandReduction, isLegalMaskedStore, isLegalMaskedLoad, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 176-192 / 第 176-192 行

```cpp
   176:       TTI::TargetCostKind CostKind,
   177:       std::optional<FastMathFlags> FMF) const override {
   178:     return InstructionCost::getInvalid();
   179:   }
   180: 
   181:   /// @}
   182: 
   183:   InstructionCost
   184:   getInstructionCost(const User *U, ArrayRef<const Value *> Operands,
   185:                      TTI::TargetCostKind CostKind) const override;
   186: 
   187:   // Hexagon specific decision to generate a lookup table.
   188:   bool shouldBuildLookupTables() const override;
   189: };
   190: 
   191: } // end namespace llvm
   192: #endif // LLVM_LIB_TARGET_HEXAGON_HEXAGONTARGETTRANSFORMINFO_H
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as InstructionCost::getInvalid, getInstructionCost, shouldBuildLookupTables, translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 InstructionCost::getInvalid, getInstructionCost, shouldBuildLookupTables 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonSubtarget.h, HexagonTargetMachine.h, llvm/ADT/ArrayRef.h, llvm/Analysis/TargetTransformInfo.h, llvm/CodeGen/BasicTTIImpl.h, llvm/IR/Function.h`
- Hexagon symbols / Hexagon 符号: `HexagonTargetTransformInfo, HexagonSubtarget, HexagonTargetMachine, HexagonTTIImpl, HexagonTargetLowering`
- Interface role / 接口角色: included by sibling Hexagon implementation files to share declarations safely. / 由同级 Hexagon 实现文件包含，以共享声明。
