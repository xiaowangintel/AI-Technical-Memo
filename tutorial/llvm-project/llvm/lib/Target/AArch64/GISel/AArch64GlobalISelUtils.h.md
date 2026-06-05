# AArch64GlobalISelUtils.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/GISel/AArch64GlobalISelUtils.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements AArch64 backend logic for the AArch64 backend. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Documented code section
```cpp
//===- AArch64GlobalISelUtils.h ----------------------------------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file APIs for AArch64-specific helper functions used in the GlobalISel
/// pipeline.
//===----------------------------------------------------------------------===//
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 11-24: Preprocessor configuration
```cpp

#ifndef LLVM_LIB_TARGET_AARCH64_GISEL_AARCH64GLOBALISELUTILS_H
#define LLVM_LIB_TARGET_AARCH64_GISEL_AARCH64GLOBALISELUTILS_H
#include "MCTargetDesc/AArch64AddressingModes.h"
#include "Utils/AArch64BaseInfo.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/GlobalISel/Utils.h"
#include "llvm/CodeGen/Register.h"
#include "llvm/IR/InstrTypes.h"
#include <cstdint>

namespace llvm {

namespace AArch64GISelUtils {
```
**EN:** This block defines compile-time guards or macros that shape the remainder of the translation unit.  
**CN:** 该代码块定义编译期开关或宏，影响后续整个翻译单元的行为。
### Lines 25-41: Function isLegalArithImmed
```cpp

/// \returns true if \p C is a legal immediate operand for an arithmetic
/// instruction.
constexpr bool isLegalArithImmed(const uint64_t C) {
  return (C >> 12 == 0) || ((C & 0xFFFULL) == 0 && C >> 24 == 0);
}

/// \returns A value when \p MI is a vector splat of a Register or constant.
/// Checks for generic opcodes and AArch64-specific generic opcodes.
std::optional<RegOrConstant>
getAArch64VectorSplat(const MachineInstr &MI, const MachineRegisterInfo &MRI);

/// \returns A value when \p MI is a constant vector splat.
/// Checks for generic opcodes and AArch64-specific generic opcodes.
std::optional<int64_t>
getAArch64VectorSplatScalar(const MachineInstr &MI,
                            const MachineRegisterInfo &MRI);
```
**EN:** This block implements isLegalArithImmed, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isLegalArithImmed，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 42-55: Documented code section
```cpp

/// \returns true if \p MaybeSub and \p Pred are part of a CMN tree for an
/// integer compare.
bool isCMN(const MachineInstr *MaybeSub, const CmpInst::Predicate &Pred,
           const MachineRegisterInfo &MRI);

/// Replace a G_MEMSET with a value of 0 with a G_BZERO instruction if it is
/// supported and beneficial to do so.
///
/// \note This only applies on Darwin.
///
/// \returns true if \p MI was replaced with a G_BZERO.
bool tryEmitBZero(MachineInstr &MI, MachineIRBuilder &MIRBuilder,
                  const LibcallLoweringInfo &Libcalls, bool MinSize);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 56-71: Documented code section
```cpp

/// Analyze a ptrauth discriminator value to try to find the constant integer
/// and address parts, cracking a ptrauth_blend intrinsic if there is one.
/// \returns integer/address disc. parts, with NoRegister if no address disc.
std::tuple<uint16_t, Register>
extractPtrauthBlendDiscriminators(Register Disc, MachineRegisterInfo &MRI);

/// Find the AArch64 condition codes necessary to represent \p P for a scalar
/// floating point comparison.
///
/// \param [out] CondCode is the first condition code.
/// \param [out] CondCode2 is the second condition code if necessary.
/// AArch64CC::AL otherwise.
void changeFCMPPredToAArch64CC(const CmpInst::Predicate P,
                               AArch64CC::CondCode &CondCode,
                               AArch64CC::CondCode &CondCode2);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 72-88: Documented code section
```cpp

/// Find the AArch64 condition codes necessary to represent \p P for a vector
/// floating point comparison.
///
/// \param [out] CondCode - The first condition code.
/// \param [out] CondCode2 - The second condition code if necessary.
/// AArch64CC::AL otherwise.
/// \param [out] Invert - True if the comparison must be inverted with a NOT.
void changeVectorFCMPPredToAArch64CC(const CmpInst::Predicate P,
                                     AArch64CC::CondCode &CondCode,
                                     AArch64CC::CondCode &CondCode2,
                                     bool &Invert);

} // namespace AArch64GISelUtils
} // namespace llvm

#endif
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: MCTargetDesc/AArch64AddressingModes.h, Utils/AArch64BaseInfo.h **CN:** 目标本地依赖：MCTargetDesc/AArch64AddressingModes.h, Utils/AArch64BaseInfo.h
- **EN:** Core LLVM interfaces: llvm/CodeGen/GlobalISel/MachineIRBuilder.h, llvm/CodeGen/GlobalISel/Utils.h, llvm/CodeGen/Register.h, llvm/IR/InstrTypes.h **CN:** 核心 LLVM 接口：llvm/CodeGen/GlobalISel/MachineIRBuilder.h, llvm/CodeGen/GlobalISel/Utils.h, llvm/CodeGen/Register.h, llvm/IR/InstrTypes.h
- **EN:** Standard-library support: cstdint **CN:** 标准库支持：cstdint
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
