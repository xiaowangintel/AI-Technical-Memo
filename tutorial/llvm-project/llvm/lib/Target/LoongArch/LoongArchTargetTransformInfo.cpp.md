# LoongArchTargetTransformInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchTargetTransformInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides target-specific optimization cost modelling for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责提供目标相关的优化开销建模。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- LoongArchTargetTransformInfo.cpp - LoongArch specific TTI ---------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: /// \file
   9: /// This file implements a TargetTransformInfo analysis pass specific to the
  10: /// LoongArch target machine. It uses the target's detailed information to
  11: /// provide more precise answers to certain TTI queries, while letting the
  12: /// target independent and default TTI implementations handle the rest.
  13: ///
  14: //===----------------------------------------------------------------------===//
  15: 
  16: #include "LoongArchTargetTransformInfo.h"
  17: 
  18: using namespace llvm;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchTargetTransformInfo.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchTargetTransformInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: #define DEBUG_TYPE "loongarchtti"
  21: 
  22: TypeSize LoongArchTTIImpl::getRegisterBitWidth(
  23:     TargetTransformInfo::RegisterKind K) const {
  24:   TypeSize DefSize = TargetTransformInfoImplBase::getRegisterBitWidth(K);
  25:   switch (K) {
  26:   case TargetTransformInfo::RGK_Scalar:
  27:     return TypeSize::getFixed(ST->is64Bit() ? 64 : 32);
  28:   case TargetTransformInfo::RGK_FixedWidthVector:
  29:     if (ST->hasExtLASX())
  30:       return TypeSize::getFixed(256);
  31:     if (ST->hasExtLSX())
  32:       return TypeSize::getFixed(128);
  33:     [[fallthrough]];
  34:   case TargetTransformInfo::RGK_ScalableVector:
  35:     return DefSize;
  36:   }
```
- **EN**: The range implements or declares functions including `LoongArchTTIImpl::getRegisterBitWidth`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTTIImpl::getRegisterBitWidth` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38:   llvm_unreachable("Unsupported register kind");
  39: }
  40: 
  41: unsigned LoongArchTTIImpl::getNumberOfRegisters(unsigned ClassID) const {
  42:   switch (ClassID) {
  43:   case LoongArchRegisterClass::GPRRC:
  44:     // 30 = 32 GPRs - r0 (zero register) - r21 (non-allocatable)
  45:     return 30;
  46:   case LoongArchRegisterClass::FPRRC:
  47:     return ST->hasBasicF() ? 32 : 0;
  48:   case LoongArchRegisterClass::VRRC:
  49:     return ST->hasExtLSX() ? 32 : 0;
  50:   }
  51:   llvm_unreachable("unknown register class");
  52: }
  53: 
  54: unsigned LoongArchTTIImpl::getRegisterClassForType(bool Vector,
```
- **EN**: The range implements or declares functions including `LoongArchTTIImpl::getNumberOfRegisters`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTTIImpl::getNumberOfRegisters` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                                                    Type *Ty) const {
  56:   if (Vector)
  57:     return LoongArchRegisterClass::VRRC;
  58:   if (!Ty)
  59:     return LoongArchRegisterClass::GPRRC;
  60: 
  61:   Type *ScalarTy = Ty->getScalarType();
  62:   if ((ScalarTy->isFloatTy() && ST->hasBasicF()) ||
  63:       (ScalarTy->isDoubleTy() && ST->hasBasicD())) {
  64:     return LoongArchRegisterClass::FPRRC;
  65:   }
  66: 
  67:   return LoongArchRegisterClass::GPRRC;
  68: }
  69: 
  70: unsigned LoongArchTTIImpl::getMaxInterleaveFactor(ElementCount VF) const {
  71:   return ST->getMaxInterleaveFactor();
  72: }
```
- **EN**: The range implements or declares functions including `LoongArchTTIImpl::getMaxInterleaveFactor`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTTIImpl::getMaxInterleaveFactor` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74: const char *LoongArchTTIImpl::getRegisterClassName(unsigned ClassID) const {
  75:   switch (ClassID) {
  76:   case LoongArchRegisterClass::GPRRC:
  77:     return "LoongArch::GPRRC";
  78:   case LoongArchRegisterClass::FPRRC:
  79:     return "LoongArch::FPRRC";
  80:   case LoongArchRegisterClass::VRRC:
  81:     return "LoongArch::VRRC";
  82:   }
  83:   llvm_unreachable("unknown register class");
  84: }
  85: 
  86: TargetTransformInfo::PopcntSupportKind
  87: LoongArchTTIImpl::getPopcntSupport(unsigned TyWidth) const {
  88:   assert(isPowerOf2_32(TyWidth) && "Ty width must be power of 2");
  89:   return ST->hasExtLSX() ? TTI::PSK_FastHardware : TTI::PSK_Software;
  90: }
```
- **EN**: The range implements or declares functions including `LoongArchTTIImpl::getPopcntSupport`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTTIImpl::getPopcntSupport` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92: unsigned LoongArchTTIImpl::getCacheLineSize() const { return 64; }
  93: 
  94: unsigned LoongArchTTIImpl::getPrefetchDistance() const { return 200; }
  95: 
  96: bool LoongArchTTIImpl::enableWritePrefetching() const { return true; }
  97: 
  98: bool LoongArchTTIImpl::shouldExpandReduction(const IntrinsicInst *II) const {
  99:   switch (II->getIntrinsicID()) {
 100:   default:
 101:     return true;
 102:   case Intrinsic::vector_reduce_add:
 103:   case Intrinsic::vector_reduce_and:
 104:   case Intrinsic::vector_reduce_or:
 105:   case Intrinsic::vector_reduce_smax:
 106:   case Intrinsic::vector_reduce_smin:
 107:   case Intrinsic::vector_reduce_umax:
 108:   case Intrinsic::vector_reduce_umin:
```
- **EN**: The range implements or declares functions including `LoongArchTTIImpl::getCacheLineSize`, `LoongArchTTIImpl::getPrefetchDistance`, `LoongArchTTIImpl::enableWritePrefetching`, `LoongArchTTIImpl::shouldExpandReduction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `LoongArchTTIImpl::getCacheLineSize`, `LoongArchTTIImpl::getPrefetchDistance`, `LoongArchTTIImpl::enableWritePrefetching`, `LoongArchTTIImpl::shouldExpandReduction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   case Intrinsic::vector_reduce_xor:
 110:     return false;
 111:   }
 112: }
 113: 
 114: LoongArchTTIImpl::TTI::MemCmpExpansionOptions
 115: LoongArchTTIImpl::enableMemCmpExpansion(bool OptSize, bool IsZeroCmp) const {
 116:   TTI::MemCmpExpansionOptions Options;
 117: 
 118:   if (!ST->hasUAL())
 119:     return Options;
 120: 
 121:   Options.MaxNumLoads = TLI->getMaxExpandSizeMemcmp(OptSize);
 122:   Options.NumLoadsPerBlock = Options.MaxNumLoads;
 123:   Options.AllowOverlappingLoads = true;
 124: 
 125:   // TODO: Support for vectors.
 126:   if (ST->is64Bit()) {
```
- **EN**: The range implements or declares functions including `LoongArchTTIImpl::enableMemCmpExpansion`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTTIImpl::enableMemCmpExpansion` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-135 / 第 127-135 行
```cpp
 127:     Options.LoadSizes = {8, 4, 2, 1};
 128:     Options.AllowedTailExpansions = {3, 5, 6};
 129:   } else {
 130:     Options.LoadSizes = {4, 2, 1};
 131:     Options.AllowedTailExpansions = {3};
 132:   }
 133: 
 134:   return Options;
 135: }
```
- **EN**: This span continues the file's main responsibility: this file provides target-specific optimization cost modelling for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `LoongArchTargetTransformInfo.h`
