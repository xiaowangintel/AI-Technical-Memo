# R600TargetTransformInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600TargetTransformInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600TargetTransformInfo for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600TargetTransformInfo 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===- R600TargetTransformInfo.cpp - AMDGPU specific TTI pass -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// \file
// This file implements a TargetTransformInfo analysis pass specific to the
// R600 target machine. It uses the target's detailed information to provide
// more precise answers to certain TTI queries, while letting the target
// independent and default TTI implementations handle the rest.
//
//===----------------------------------------------------------------------===//

#include "R600TargetTransformInfo.h"
#include "AMDGPU.h"
#include "AMDGPUTargetMachine.h"
#include "R600Subtarget.h"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-46: Preprocessor guards and macros
```cpp
#define DEBUG_TYPE "R600tti"

R600TTIImpl::R600TTIImpl(const AMDGPUTargetMachine *TM, const Function &F)
    : BaseT(TM, F.getDataLayout()),
      ST(static_cast<const R600Subtarget *>(TM->getSubtargetImpl(F))),
      TLI(ST->getTargetLowering()), CommonTTI(TM, F) {}

unsigned R600TTIImpl::getHardwareNumberOfRegisters(bool Vec) const {
  return 4 * 128; // XXX - 4 channels. Should these count as vector instead?
}

unsigned R600TTIImpl::getNumberOfRegisters(unsigned ClassID) const {
  bool Vec = ClassID == 1;
  return getHardwareNumberOfRegisters(Vec);
}

TypeSize
R600TTIImpl::getRegisterBitWidth(TargetTransformInfo::RegisterKind K) const {
  return TypeSize::getFixed(32);
}

unsigned R600TTIImpl::getMinVectorRegisterBitWidth() const { return 32; }

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `R600TTIImpl::R600TTIImpl`, `R600TTIImpl::getHardwareNumberOfRegisters`, `R600TTIImpl::getNumberOfRegisters`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`R600TTIImpl::R600TTIImpl`, `R600TTIImpl::getHardwareNumberOfRegisters`, `R600TTIImpl::getNumberOfRegisters`。

### Lines 47-64: Implements R600TTIImpl::getLoadStoreVecRegBitWidth
```cpp
unsigned R600TTIImpl::getLoadStoreVecRegBitWidth(unsigned AddrSpace) const {
  if (AddrSpace == AMDGPUAS::GLOBAL_ADDRESS ||
      AddrSpace == AMDGPUAS::CONSTANT_ADDRESS)
    return 128;
  if (AddrSpace == AMDGPUAS::LOCAL_ADDRESS ||
      AddrSpace == AMDGPUAS::REGION_ADDRESS)
    return 64;
  if (AddrSpace == AMDGPUAS::PRIVATE_ADDRESS)
    return 32;

  if ((AddrSpace == AMDGPUAS::PARAM_D_ADDRESS ||
       AddrSpace == AMDGPUAS::PARAM_I_ADDRESS ||
       (AddrSpace >= AMDGPUAS::CONSTANT_BUFFER_0 &&
        AddrSpace <= AMDGPUAS::CONSTANT_BUFFER_15)))
    return 128;
  llvm_unreachable("unhandled address space");
}

```
**EN:** This section contains concrete logic for R600TTIImpl::getLoadStoreVecRegBitWidth. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600TTIImpl::getLoadStoreVecRegBitWidth`.
**CN:** 本节包含与 R600TTIImpl::getLoadStoreVecRegBitWidth 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600TTIImpl::getLoadStoreVecRegBitWidth`。

### Lines 65-85: Implements R600TTIImpl::isLegalToVectorizeMemChain
```cpp
bool R600TTIImpl::isLegalToVectorizeMemChain(unsigned ChainSizeInBytes,
                                             Align Alignment,
                                             unsigned AddrSpace) const {
  // We allow vectorization of flat stores, even though we may need to decompose
  // them later if they may access private memory. We don't have enough context
  // here, and legalization can handle it.
  return (AddrSpace != AMDGPUAS::PRIVATE_ADDRESS);
}

bool R600TTIImpl::isLegalToVectorizeLoadChain(unsigned ChainSizeInBytes,
                                              Align Alignment,
                                              unsigned AddrSpace) const {
  return isLegalToVectorizeMemChain(ChainSizeInBytes, Alignment, AddrSpace);
}

bool R600TTIImpl::isLegalToVectorizeStoreChain(unsigned ChainSizeInBytes,
                                               Align Alignment,
                                               unsigned AddrSpace) const {
  return isLegalToVectorizeMemChain(ChainSizeInBytes, Alignment, AddrSpace);
}

```
**EN:** This section contains concrete logic for R600TTIImpl::isLegalToVectorizeMemChain. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600TTIImpl::isLegalToVectorizeMemChain`, `R600TTIImpl::isLegalToVectorizeLoadChain`, `R600TTIImpl::isLegalToVectorizeStoreChain`.
**CN:** 本节包含与 R600TTIImpl::isLegalToVectorizeMemChain 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600TTIImpl::isLegalToVectorizeMemChain`, `R600TTIImpl::isLegalToVectorizeLoadChain`, `R600TTIImpl::isLegalToVectorizeStoreChain`。

### Lines 86-100: Implements R600TTIImpl::getMaxInterleaveFactor
```cpp
unsigned R600TTIImpl::getMaxInterleaveFactor(ElementCount VF) const {
  // Disable unrolling if the loop is not vectorized.
  // TODO: Enable this again.
  if (VF.isScalar())
    return 1;

  return 8;
}

InstructionCost R600TTIImpl::getCFInstrCost(unsigned Opcode,
                                            TTI::TargetCostKind CostKind,
                                            const Instruction *I) const {
  if (CostKind == TTI::TCK_CodeSize || CostKind == TTI::TCK_SizeAndLatency)
    return Opcode == Instruction::PHI ? 0 : 1;

```
**EN:** This section contains concrete logic for R600TTIImpl::getMaxInterleaveFactor. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600TTIImpl::getMaxInterleaveFactor`, `R600TTIImpl::getCFInstrCost`.
**CN:** 本节包含与 R600TTIImpl::getMaxInterleaveFactor 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600TTIImpl::getMaxInterleaveFactor`, `R600TTIImpl::getCFInstrCost`。

### Lines 101-124: Implements R600TTIImpl::getVectorInstrCost
```cpp
  // XXX - For some reason this isn't called for switch.
  switch (Opcode) {
  case Instruction::UncondBr:
  case Instruction::CondBr:
  case Instruction::Ret:
    return 10;
  default:
    return BaseT::getCFInstrCost(Opcode, CostKind, I);
  }
}

InstructionCost R600TTIImpl::getVectorInstrCost(
    unsigned Opcode, Type *ValTy, TTI::TargetCostKind CostKind, unsigned Index,
    const Value *Op0, const Value *Op1, TTI::VectorInstrContext VIC) const {
  switch (Opcode) {
  case Instruction::ExtractElement:
  case Instruction::InsertElement: {
    unsigned EltSize =
        DL.getTypeSizeInBits(cast<VectorType>(ValTy)->getElementType());
    if (EltSize < 32) {
      return BaseT::getVectorInstrCost(Opcode, ValTy, CostKind, Index, Op0, Op1,
                                       VIC);
    }

```
**EN:** This section contains concrete logic for R600TTIImpl::getVectorInstrCost. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `BaseT::getCFInstrCost`, `R600TTIImpl::getVectorInstrCost`, `BaseT::getVectorInstrCost`.
**CN:** 本节包含与 R600TTIImpl::getVectorInstrCost 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`BaseT::getCFInstrCost`, `R600TTIImpl::getVectorInstrCost`, `BaseT::getVectorInstrCost`。

### Lines 125-147: Result computation and returns
```cpp
    // Extracts are just reads of a subregister, so are free. Inserts are
    // considered free because we don't want to have any cost for scalarizing
    // operations, and we don't have to copy into a different register class.

    // Dynamic indexing isn't free and is best avoided.
    return Index == ~0u ? 2 : 0;
  }
  default:
    return BaseT::getVectorInstrCost(Opcode, ValTy, CostKind, Index, Op0, Op1,
                                     VIC);
  }
}

void R600TTIImpl::getUnrollingPreferences(
    Loop *L, ScalarEvolution &SE, TTI::UnrollingPreferences &UP,
    OptimizationRemarkEmitter *ORE) const {
  CommonTTI.getUnrollingPreferences(L, SE, UP, ORE);
}

void R600TTIImpl::getPeelingPreferences(Loop *L, ScalarEvolution &SE,
                                        TTI::PeelingPreferences &PP) const {
  CommonTTI.getPeelingPreferences(L, SE, PP);
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `BaseT::getVectorInstrCost`, `R600TTIImpl::getUnrollingPreferences`, `R600TTIImpl::getPeelingPreferences`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`BaseT::getVectorInstrCost`, `R600TTIImpl::getUnrollingPreferences`, `R600TTIImpl::getPeelingPreferences`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600TTIImpl::R600TTIImpl`, `R600TTIImpl::getHardwareNumberOfRegisters`, `R600TTIImpl::getNumberOfRegisters`, `R600TTIImpl::getRegisterBitWidth`, `TypeSize::getFixed`, `R600TTIImpl::getMinVectorRegisterBitWidth`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600TargetTransformInfo.h"`
- `"AMDGPU.h"`
- `"AMDGPUTargetMachine.h"`
- `"R600Subtarget.h"`
