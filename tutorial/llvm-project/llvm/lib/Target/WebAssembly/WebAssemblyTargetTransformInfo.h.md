# WebAssemblyTargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyTargetTransformInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file a TargetTransformInfoImplBase conforming object specific to the WebAssembly target machine.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyTargetTransformInfo.h`，主要负责 WebAssembly 后端的目标变换代价建模。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//==- WebAssemblyTargetTransformInfo.h - WebAssembly-specific TTI -*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-16

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file a TargetTransformInfoImplBase conforming object specific
/// to the WebAssembly target machine.
///
/// It uses the target's detailed information to provide more precise answers to
/// certain TTI queries, while letting the target independent and default TTI
/// implementations handle the rest.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 17-24

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYTARGETTRANSFORMINFO_H
#define LLVM_LIB_TARGET_WEBASSEMBLY_WEBASSEMBLYTARGETTRANSFORMINFO_H

#include "WebAssemblyTargetMachine.h"
#include "llvm/CodeGen/BasicTTIImpl.h"
```
- **EN**: Pulls in direct dependencies required by this target transform cost modeling, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标变换代价建模所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 25-50

```cpp
namespace llvm {

class WebAssemblyTTIImpl final : public BasicTTIImplBase<WebAssemblyTTIImpl> {
  typedef BasicTTIImplBase<WebAssemblyTTIImpl> BaseT;
  typedef TargetTransformInfo TTI;
  friend BaseT;

  const WebAssemblySubtarget *ST;
  const WebAssemblyTargetLowering *TLI;

  const WebAssemblySubtarget *getST() const { return ST; }
  const WebAssemblyTargetLowering *getTLI() const { return TLI; }

public:
  WebAssemblyTTIImpl(const WebAssemblyTargetMachine *TM, const Function &F)
      : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
        TLI(ST->getTargetLowering()) {}

  /// \name Scalar TTI Implementations
  /// @{

  // TODO: Implement more Scalar TTI for WebAssembly

  TTI::PopcntSupportKind getPopcntSupport(unsigned TyWidth) const override;

  void getUnrollingPreferences(Loop *L, ScalarEvolution &SE,
```
- **EN**: Opens or re-enters namespace scope so backend symbols stay grouped under LLVM naming conventions. Subtarget feature gating influences the behavior here.
- **CN**: 这里进入或重新进入命名空间作用域，使后端符号按照 LLVM 的命名约定组织在一起。 子目标特性裁剪会影响这里的行为。

### Lines 51-76

```cpp
                               TTI::UnrollingPreferences &UP,
                               OptimizationRemarkEmitter *ORE) const override;

  /// @}

  /// \name Vector TTI Implementations
  /// @{

  bool enableInterleavedAccessVectorization() const override { return true; }

  unsigned getNumberOfRegisters(unsigned ClassID) const override;
  unsigned getMinVectorRegisterBitWidth() const override { return 32; }
  TypeSize
  getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override;
  InstructionCost getArithmeticInstrCost(
      unsigned Opcode, Type *Ty, TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo Op1Info = {TTI::OK_AnyValue, TTI::OP_None},
      TTI::OperandValueInfo Op2Info = {TTI::OK_AnyValue, TTI::OP_None},
      ArrayRef<const Value *> Args = {},
      const Instruction *CxtI = nullptr) const override;

  InstructionCost
  getCastInstrCost(unsigned Opcode, Type *Dst, Type *Src,
                   TTI::CastContextHint CCH, TTI::TargetCostKind CostKind,
                   const Instruction *I = nullptr) const override;
```
- **EN**: Implements helper routine(s) `enableInterleavedAccessVectorization`, `getNumberOfRegisters`, `getMinVectorRegisterBitWidth` for this portion of the WebAssembly backend target transform cost modeling.
- **CN**: 这里实现了 WebAssembly 后端该部分目标变换代价建模所需的辅助例程 `enableInterleavedAccessVectorization`, `getNumberOfRegisters`, `getMinVectorRegisterBitWidth`。

### Lines 77-102

```cpp
  TTI::MemCmpExpansionOptions
  enableMemCmpExpansion(bool OptSize, bool IsZeroCmp) const override;

  InstructionCost getMemoryOpCost(
      unsigned Opcode, Type *Src, Align Alignment, unsigned AddressSpace,
      TTI::TargetCostKind CostKind,
      TTI::OperandValueInfo OpInfo = {TTI::OK_AnyValue, TTI::OP_None},
      const Instruction *I = nullptr) const override;
  InstructionCost getInterleavedMemoryOpCost(
      unsigned Opcode, Type *Ty, unsigned Factor, ArrayRef<unsigned> Indices,
      Align Alignment, unsigned AddressSpace, TTI::TargetCostKind CostKind,
      bool UseMaskForCond, bool UseMaskForGaps) const override;
  InstructionCost
  getShuffleCost(TTI::ShuffleKind Kind, VectorType *DstTy, VectorType *SrcTy,
                 ArrayRef<int> Mask, TTI::TargetCostKind CostKind, int Index,
                 VectorType *SubTp, ArrayRef<const Value *> Args = {},
                 const Instruction *CxtI = nullptr) const override;

  using BaseT::getVectorInstrCost;
  InstructionCost
  getVectorInstrCost(unsigned Opcode, Type *Val, TTI::TargetCostKind CostKind,
                     unsigned Index, const Value *Op0, const Value *Op1,
                     TTI::VectorInstrContext VIC =
                         TTI::VectorInstrContext::None) const override;
  InstructionCost getPartialReductionCost(
      unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
```
- **EN**: Implements helper routine(s) `enableMemCmpExpansion`, `getMemoryOpCost`, `getInterleavedMemoryOpCost` for this portion of the WebAssembly backend target transform cost modeling.
- **CN**: 这里实现了 WebAssembly 后端该部分目标变换代价建模所需的辅助例程 `enableMemCmpExpansion`, `getMemoryOpCost`, `getInterleavedMemoryOpCost`。

### Lines 103-123

```cpp
      ElementCount VF, TTI::PartialReductionExtendKind OpAExtend,
      TTI::PartialReductionExtendKind OpBExtend, std::optional<unsigned> BinOp,
      TTI::TargetCostKind CostKind,
      std::optional<FastMathFlags> FMF) const override;

  TTI::ReductionShuffle
  getPreferredExpandedReductionShuffle(const IntrinsicInst *II) const override;

  bool supportsTailCalls() const override;

  bool isProfitableToSinkOperands(Instruction *I,
                                  SmallVectorImpl<Use *> &Ops) const override;

  std::optional<Instruction *>
  instCombineIntrinsic(InstCombiner &IC, IntrinsicInst &II) const override;

  /// @}
};

} // end namespace llvm
```
- **EN**: Declares function entry points including `getPreferredExpandedReductionShuffle`, `supportsTailCalls`, `isProfitableToSinkOperands` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getPreferredExpandedReductionShuffle`, `supportsTailCalls`, `isProfitableToSinkOperands`。

### Lines 124-124

```cpp
#endif
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Target transform cost modeling / 目标变换代价建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyTargetMachine.h`
- `llvm/CodeGen/BasicTTIImpl.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
