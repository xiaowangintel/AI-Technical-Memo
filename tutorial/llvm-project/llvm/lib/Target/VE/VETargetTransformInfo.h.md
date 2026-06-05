# VETargetTransformInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VETargetTransformInfo.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Provides target-specific cost-model hooks used by optimization passes through TargetTransformInfo.
  - **CN**: 通过 TargetTransformInfo 提供目标专用的代价模型钩子，供优化 Pass 使用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- VETargetTransformInfo.h - VE specific TTI ------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-14
```cpp
/// \file
/// This file a TargetTransformInfoImplBase conforming object specific to the
/// VE target machine. It uses the target's detailed information to
/// provide more precise answers to certain TTI queries, while letting the
/// target independent and default TTI implementations handle the rest.
///
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 15-23
```cpp

#ifndef LLVM_LIB_TARGET_VE_VETARGETTRANSFORMINFO_H
#define LLVM_LIB_TARGET_VE_VETARGETTRANSFORMINFO_H

#include "VE.h"
#include "VETargetMachine.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/BasicTTIImpl.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `VE.h`, `VETargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/BasicTTIImpl.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VE.h`, `VETargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/BasicTTIImpl.h`。

### Lines 24-34
```cpp
static llvm::Type *getVectorElementType(llvm::Type *Ty) {
  return llvm::cast<llvm::FixedVectorType>(Ty)->getElementType();
}

static llvm::Type *getLaneType(llvm::Type *Ty) {
  using namespace llvm;
  if (!isa<VectorType>(Ty))
    return Ty;
  return getVectorElementType(Ty);
}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 35-48
```cpp
static bool isVectorLaneType(llvm::Type &ElemTy) {
  // check element sizes for vregs
  if (ElemTy.isIntegerTy()) {
    unsigned ScaBits = ElemTy.getScalarSizeInBits();
    return ScaBits == 1 || ScaBits == 32 || ScaBits == 64;
  }
  if (ElemTy.isPointerTy()) {
    return true;
  }
  if (ElemTy.isFloatTy() || ElemTy.isDoubleTy()) {
    return true;
  }
  return false;
}
```
- **EN**: Implements logic around `isVectorLaneType`, `getScalarSizeInBits`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isVectorLaneType`, `getScalarSizeInBits` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 49-55
```cpp

namespace llvm {

class VETTIImpl final : public BasicTTIImplBase<VETTIImpl> {
  using BaseT = BasicTTIImplBase<VETTIImpl>;
  friend BaseT;

```
- **EN**: Introduces declarations for `llvm`, `VETTIImpl`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `VETTIImpl` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 56-63
```cpp
  const VESubtarget *ST;
  const VETargetLowering *TLI;

  const VESubtarget *getST() const { return ST; }
  const VETargetLowering *getTLI() const { return TLI; }

  bool enableVPU() const { return getST()->enableVPU(); }

```
- **EN**: Implements logic around `getST`, `getTLI`, `enableVPU`; this block returns target-specific results.
- **CN**: 围绕 `getST`, `getTLI`, `enableVPU` 实现具体逻辑；这一段返回目标相关结果。

### Lines 64-76
```cpp
  static bool isSupportedReduction(Intrinsic::ID ReductionID) {
#define VEC_VP_CASE(SUFFIX)                                                    \
  case Intrinsic::vp_reduce_##SUFFIX:                                          \
  case Intrinsic::vector_reduce_##SUFFIX:

    switch (ReductionID) {
      VEC_VP_CASE(add)
      VEC_VP_CASE(and)
      VEC_VP_CASE(or)
      VEC_VP_CASE(xor)
      VEC_VP_CASE(smax)
      return true;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 77-87
```cpp
    default:
      return false;
    }
#undef VEC_VP_CASE
  }

public:
  explicit VETTIImpl(const VETargetMachine *TM, const Function &F)
      : BaseT(TM, F.getDataLayout()), ST(TM->getSubtargetImpl(F)),
        TLI(ST->getTargetLowering()) {}

```
- **EN**: Implements logic around `VETTIImpl`, `BaseT`, `TLI`; this block returns target-specific results.
- **CN**: 围绕 `VETTIImpl`, `BaseT`, `TLI` 实现具体逻辑；这一段返回目标相关结果。

### Lines 88-94
```cpp
  unsigned getNumberOfRegisters(unsigned ClassID) const override {
    bool VectorRegs = (ClassID == 1);
    if (VectorRegs) {
      // TODO report vregs once vector isel is stable.
      return 0;
    }

```
- **EN**: Implements logic around `getNumberOfRegisters`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getNumberOfRegisters` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 95-108
```cpp
    return 64;
  }

  TypeSize
  getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const override {
    switch (K) {
    case TargetTransformInfo::RGK_Scalar:
      return TypeSize::getFixed(64);
    case TargetTransformInfo::RGK_FixedWidthVector:
      // TODO report vregs once vector isel is stable.
      return TypeSize::getFixed(0);
    case TargetTransformInfo::RGK_ScalableVector:
      return TypeSize::getScalable(0);
    }
```
- **EN**: Implements logic around `getRegisterBitWidth`, `getFixed`, `getScalable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getRegisterBitWidth`, `getFixed`, `getScalable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 109-120
```cpp

    llvm_unreachable("Unsupported register kind");
  }

  /// \returns How the target needs this vector-predicated operation to be
  /// transformed.
  TargetTransformInfo::VPLegalization
  getVPLegalizationStrategy(const VPIntrinsic &PI) const override {
    using VPLegalization = TargetTransformInfo::VPLegalization;
    return VPLegalization(VPLegalization::Legal, VPLegalization::Legal);
  }

```
- **EN**: Implements logic around `llvm_unreachable`, `getVPLegalizationStrategy`, `VPLegalization`; this block returns target-specific results.
- **CN**: 围绕 `llvm_unreachable`, `getVPLegalizationStrategy`, `VPLegalization` 实现具体逻辑；这一段返回目标相关结果。

### Lines 121-134
```cpp
  unsigned getMinVectorRegisterBitWidth() const override {
    // TODO report vregs once vector isel is stable.
    return 0;
  }

  bool shouldBuildRelLookupTables() const override {
    // NEC nld doesn't support relative lookup tables.  It shows following
    // errors.  So, we disable it at the moment.
    //   /opt/nec/ve/bin/nld: src/CMakeFiles/cxxabi_shared.dir/cxa_demangle.cpp
    //   .o(.rodata+0x17b4): reloc against `.L.str.376': error 2
    //   /opt/nec/ve/bin/nld: final link failed: Nonrepresentable section on
    //   output
    return false;
  }
```
- **EN**: Implements logic around `getMinVectorRegisterBitWidth`, `shouldBuildRelLookupTables`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getMinVectorRegisterBitWidth`, `shouldBuildRelLookupTables` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 135-148
```cpp

  // Load & Store {
  bool
  isLegalMaskedLoad(Type *DataType, Align Alignment, unsigned /*AddressSpace*/,
                    TargetTransformInfo::MaskKind /*MaskKind*/) const override {
    return isVectorLaneType(*getLaneType(DataType));
  }
  bool isLegalMaskedStore(
      Type *DataType, Align Alignment, unsigned /*AddressSpace*/,
      TargetTransformInfo::MaskKind /*MaskKind*/) const override {
    return isVectorLaneType(*getLaneType(DataType));
  }
  bool isLegalMaskedGather(Type *DataType, Align Alignment) const override {
    return isVectorLaneType(*getLaneType(DataType));
```
- **EN**: Implements logic around `isLegalMaskedLoad`, `isVectorLaneType`, `isLegalMaskedStore`, `isLegalMaskedGather`; this block returns target-specific results.
- **CN**: 围绕 `isLegalMaskedLoad`, `isVectorLaneType`, `isLegalMaskedStore`, `isLegalMaskedGather` 实现具体逻辑；这一段返回目标相关结果。

### Lines 149-160
```cpp
  };
  bool isLegalMaskedScatter(Type *DataType, Align Alignment) const override {
    return isVectorLaneType(*getLaneType(DataType));
  }
  // } Load & Store

  bool shouldExpandReduction(const IntrinsicInst *II) const override {
    if (!enableVPU())
      return true;
    return !isSupportedReduction(II->getIntrinsicID());
  }

```
- **EN**: Implements logic around `isLegalMaskedScatter`, `isVectorLaneType`, `shouldExpandReduction`, `isSupportedReduction`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isLegalMaskedScatter`, `isVectorLaneType`, `shouldExpandReduction`, `isSupportedReduction` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 161-172
```cpp
  InstructionCost getPartialReductionCost(
      unsigned Opcode, Type *InputTypeA, Type *InputTypeB, Type *AccumType,
      ElementCount VF,
      TargetTransformInfo::PartialReductionExtendKind OpAExtend,
      TargetTransformInfo::PartialReductionExtendKind OpBExtend,
      std::optional<unsigned> BinOp,
      TargetTransformInfo::TargetCostKind CostKind,
      std::optional<FastMathFlags> FMF) const override {
    return InstructionCost::getInvalid();
  }
};

```
- **EN**: Implements logic around `getPartialReductionCost`, `getInvalid`; this block returns target-specific results.
- **CN**: 围绕 `getPartialReductionCost`, `getInvalid` 实现具体逻辑；这一段返回目标相关结果。

### Lines 173-175
```cpp
} // namespace llvm

#endif // LLVM_LIB_TARGET_VE_VETARGETTRANSFORMINFO_H
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **Cost modeling / 代价建模**:
  - **EN**: Feeds optimization passes with target-specific profitability estimates
  - **CN**: 向优化 Pass 提供目标专用收益估计
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VE.h`, `VETargetMachine.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/BasicTTIImpl.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_VE_VETARGETTRANSFORMINFO_H`
