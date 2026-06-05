# ARMUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Utility/ARMUtils.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares the LLDB interfaces, data structures, and helper APIs associated with `ARMUtils`.
  - **CN**: 声明与 `ARMUtils` 相关的 LLDB 接口、数据结构以及辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMUtils.h ----------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLDB_SOURCE_PLUGINS_PROCESS_UTILITY_ARMUTILS_H
#define LLDB_SOURCE_PLUGINS_PROCESS_UTILITY_ARMUTILS_H

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 12-24
```cpp
#include "ARMDefines.h"
#include "InstructionUtils.h"
#include "llvm/ADT/bit.h"
#include "llvm/Support/MathExtras.h"

// Common utilities for the ARM/Thumb Instruction Set Architecture.

namespace lldb_private {

static inline uint32_t Align(uint32_t val, uint32_t alignment) {
  return alignment * (val / alignment);
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `ARMDefines.h`, `InstructionUtils.h`, `llvm/ADT/bit.h`, `llvm/Support/MathExtras.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `ARMDefines.h`, `InstructionUtils.h`, `llvm/ADT/bit.h`, `llvm/Support/MathExtras.h`。

### Lines 25-44
```cpp
static inline uint32_t DecodeImmShift(const uint32_t type, const uint32_t imm5,
                                      ARM_ShifterType &shift_t) {
  switch (type) {
  default:
    assert(0 && "Invalid shift type");
    break;
  case 0:
    shift_t = SRType_LSL;
    return imm5;
  case 1:
    shift_t = SRType_LSR;
    return (imm5 == 0 ? 32 : imm5);
  case 2:
    shift_t = SRType_ASR;
    return (imm5 == 0 ? 32 : imm5);
  case 3:
    if (imm5 == 0) {
      shift_t = SRType_RRX;
      return 1;
    } else {
```
- **EN**: Implements logic around `DecodeImmShift`, `assert`.
- **CN**: 围绕 `DecodeImmShift`, `assert` 实现具体逻辑。

### Lines 45-61
```cpp
      shift_t = SRType_ROR;
      return imm5;
    }
  }
  shift_t = SRType_Invalid;
  return UINT32_MAX;
}

// A8.6.35 CMP (register) -- Encoding T3
// Convenience function.
static inline uint32_t DecodeImmShiftThumb(const uint32_t opcode,
                                           ARM_ShifterType &shift_t) {
  return DecodeImmShift(Bits32(opcode, 5, 4),
                        Bits32(opcode, 14, 12) << 2 | Bits32(opcode, 7, 6),
                        shift_t);
}

```
- **EN**: Implements logic around `DecodeImmShiftThumb`, `DecodeImmShift`, `Bits32`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `DecodeImmShiftThumb`, `DecodeImmShift`, `Bits32` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 62-74
```cpp
// A8.6.35 CMP (register) -- Encoding A1
// Convenience function.
static inline uint32_t DecodeImmShiftARM(const uint32_t opcode,
                                         ARM_ShifterType &shift_t) {
  return DecodeImmShift(Bits32(opcode, 6, 5), Bits32(opcode, 11, 7), shift_t);
}

static inline uint32_t DecodeImmShift(const ARM_ShifterType shift_t,
                                      const uint32_t imm5) {
  ARM_ShifterType dont_care;
  return DecodeImmShift(shift_t, imm5, dont_care);
}

```
- **EN**: Implements logic around `DecodeImmShiftARM`, `DecodeImmShift`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `DecodeImmShiftARM`, `DecodeImmShift` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 75-90
```cpp
static inline ARM_ShifterType DecodeRegShift(const uint32_t type) {
  switch (type) {
  default:
    // assert(0 && "Invalid shift type");
    return SRType_Invalid;
  case 0:
    return SRType_LSL;
  case 1:
    return SRType_LSR;
  case 2:
    return SRType_ASR;
  case 3:
    return SRType_ROR;
  }
}

```
- **EN**: Implements logic around `DecodeRegShift`.
- **CN**: 围绕 `DecodeRegShift` 实现具体逻辑。

### Lines 91-101
```cpp
static inline uint32_t LSL_C(const uint32_t value, const uint32_t amount,
                             uint32_t &carry_out, bool *success) {
  if (amount == 0) {
    *success = false;
    return 0;
  }
  *success = true;
  carry_out = amount <= 32 ? Bit32(value, 32 - amount) : 0;
  return value << amount;
}

```
- **EN**: Implements logic around `LSL_C`, `Bit32`.
- **CN**: 围绕 `LSL_C`, `Bit32` 实现具体逻辑。

### Lines 102-114
```cpp
static inline uint32_t LSL(const uint32_t value, const uint32_t amount,
                           bool *success) {
  *success = true;
  if (amount == 0)
    return value;
  uint32_t dont_care;
  uint32_t result = LSL_C(value, amount, dont_care, success);
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Implements logic around `LSL`, `LSL_C`.
- **CN**: 围绕 `LSL`, `LSL_C` 实现具体逻辑。

### Lines 115-125
```cpp
static inline uint32_t LSR_C(const uint32_t value, const uint32_t amount,
                             uint32_t &carry_out, bool *success) {
  if (amount == 0) {
    *success = false;
    return 0;
  }
  *success = true;
  carry_out = amount <= 32 ? Bit32(value, amount - 1) : 0;
  return value >> amount;
}

```
- **EN**: Implements logic around `LSR_C`, `Bit32`.
- **CN**: 围绕 `LSR_C`, `Bit32` 实现具体逻辑。

### Lines 126-138
```cpp
static inline uint32_t LSR(const uint32_t value, const uint32_t amount,
                           bool *success) {
  *success = true;
  if (amount == 0)
    return value;
  uint32_t dont_care;
  uint32_t result = LSR_C(value, amount, dont_care, success);
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Implements logic around `LSR`, `LSR_C`.
- **CN**: 围绕 `LSR`, `LSR_C` 实现具体逻辑。

### Lines 139-156
```cpp
static inline uint32_t ASR_C(const uint32_t value, const uint32_t amount,
                             uint32_t &carry_out, bool *success) {
  if (amount == 0 || amount > 32) {
    *success = false;
    return 0;
  }
  *success = true;
  bool negative = BitIsSet(value, 31);
  if (amount <= 32) {
    carry_out = Bit32(value, amount - 1);
    int64_t extended = llvm::SignExtend64<32>(value);
    return UnsignedBits(extended, amount + 31, amount);
  } else {
    carry_out = (negative ? 1 : 0);
    return (negative ? 0xffffffff : 0);
  }
}

```
- **EN**: Implements logic around `ASR_C`, `BitIsSet`, `Bit32`, `SignExtend64`, and 1 more symbols.
- **CN**: 围绕 `ASR_C`, `BitIsSet`, `Bit32`, `SignExtend64`, and 1 more symbols 实现具体逻辑。

### Lines 157-169
```cpp
static inline uint32_t ASR(const uint32_t value, const uint32_t amount,
                           bool *success) {
  *success = true;
  if (amount == 0)
    return value;
  uint32_t dont_care;
  uint32_t result = ASR_C(value, amount, dont_care, success);
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Implements logic around `ASR`, `ASR_C`.
- **CN**: 围绕 `ASR`, `ASR_C` 实现具体逻辑。

### Lines 170-181
```cpp
static inline uint32_t ROR_C(const uint32_t value, const uint32_t amount,
                             uint32_t &carry_out, bool *success) {
  if (amount == 0) {
    *success = false;
    return 0;
  }
  *success = true;
  uint32_t result = llvm::rotr<uint32_t>(value, amount);
  carry_out = Bit32(value, 31);
  return result;
}

```
- **EN**: Implements logic around `ROR_C`, `rotr`, `Bit32`.
- **CN**: 围绕 `ROR_C`, `rotr`, `Bit32` 实现具体逻辑。

### Lines 182-194
```cpp
static inline uint32_t ROR(const uint32_t value, const uint32_t amount,
                           bool *success) {
  *success = true;
  if (amount == 0)
    return value;
  uint32_t dont_care;
  uint32_t result = ROR_C(value, amount, dont_care, success);
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Implements logic around `ROR`, `ROR_C`.
- **CN**: 围绕 `ROR`, `ROR_C` 实现具体逻辑。

### Lines 195-212
```cpp
static inline uint32_t RRX_C(const uint32_t value, const uint32_t carry_in,
                             uint32_t &carry_out, bool *success) {
  *success = true;
  carry_out = Bit32(value, 0);
  return Bit32(carry_in, 0) << 31 | Bits32(value, 31, 1);
}

static inline uint32_t RRX(const uint32_t value, const uint32_t carry_in,
                           bool *success) {
  *success = true;
  uint32_t dont_care;
  uint32_t result = RRX_C(value, carry_in, dont_care, success);
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Implements logic around `RRX_C`, `Bit32`, `RRX`.
- **CN**: 围绕 `RRX_C`, `Bit32`, `RRX` 实现具体逻辑。

### Lines 213-232
```cpp
static inline uint32_t Shift_C(const uint32_t value, ARM_ShifterType type,
                               const uint32_t amount, const uint32_t carry_in,
                               uint32_t &carry_out, bool *success) {
  if (type == SRType_RRX && amount != 1) {
    *success = false;
    return 0;
  }
  *success = true;

  if (amount == 0) {
    carry_out = carry_in;
    return value;
  }
  uint32_t result;
  switch (type) {
  case SRType_LSL:
    result = LSL_C(value, amount, carry_out, success);
    break;
  case SRType_LSR:
    result = LSR_C(value, amount, carry_out, success);
```
- **EN**: Implements logic around `Shift_C`, `LSL_C`, `LSR_C`.
- **CN**: 围绕 `Shift_C`, `LSL_C`, `LSR_C` 实现具体逻辑。

### Lines 233-252
```cpp
    break;
  case SRType_ASR:
    result = ASR_C(value, amount, carry_out, success);
    break;
  case SRType_ROR:
    result = ROR_C(value, amount, carry_out, success);
    break;
  case SRType_RRX:
    result = RRX_C(value, carry_in, carry_out, success);
    break;
  default:
    *success = false;
    break;
  }
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Declares APIs around `ASR_C`, `ROR_C`, `RRX_C`.
- **CN**: 声明与 `ASR_C`, `ROR_C`, `RRX_C` 相关的 API。

### Lines 253-264
```cpp
static inline uint32_t Shift(const uint32_t value, ARM_ShifterType type,
                             const uint32_t amount, const uint32_t carry_in,
                             bool *success) {
  // Don't care about carry out in this case.
  uint32_t dont_care;
  uint32_t result = Shift_C(value, type, amount, carry_in, dont_care, success);
  if (*success)
    return result;
  else
    return 0;
}

```
- **EN**: Implements logic around `Shift`, `Shift_C`.
- **CN**: 围绕 `Shift`, `Shift_C` 实现具体逻辑。

### Lines 265-278
```cpp
static inline uint32_t bits(const uint32_t val, const uint32_t msbit,
                            const uint32_t lsbit) {
  return Bits32(val, msbit, lsbit);
}

static inline uint32_t bit(const uint32_t val, const uint32_t msbit) {
  return bits(val, msbit, msbit);
}

static uint32_t ror(uint32_t val, uint32_t N, uint32_t shift) {
  uint32_t m = shift % N;
  return (val >> m) | (val << (N - m));
}

```
- **EN**: Implements logic around `bits`, `Bits32`, `bit`, `ror`.
- **CN**: 围绕 `bits`, `Bits32`, `bit`, `ror` 实现具体逻辑。

### Lines 279-294
```cpp
// (imm32, carry_out) = ARMExpandImm_C(imm12, carry_in)
static inline uint32_t ARMExpandImm_C(uint32_t opcode, uint32_t carry_in,
                                      uint32_t &carry_out) {
  uint32_t imm32;                         // the expanded result
  uint32_t imm = bits(opcode, 7, 0);      // immediate value
  uint32_t amt = 2 * bits(opcode, 11, 8); // rotate amount
  if (amt == 0) {
    imm32 = imm;
    carry_out = carry_in;
  } else {
    imm32 = ror(imm, 32, amt);
    carry_out = Bit32(imm32, 31);
  }
  return imm32;
}

```
- **EN**: Implements logic around `ARMExpandImm_C`, `bits`, `ror`, `Bit32`.
- **CN**: 围绕 `ARMExpandImm_C`, `bits`, `ror`, `Bit32` 实现具体逻辑。

### Lines 295-311
```cpp
static inline uint32_t ARMExpandImm(uint32_t opcode) {
  // 'carry_in' argument to following function call does not affect the imm32
  // result.
  uint32_t carry_in = 0;
  uint32_t carry_out;
  return ARMExpandImm_C(opcode, carry_in, carry_out);
}

// (imm32, carry_out) = ThumbExpandImm_C(imm12, carry_in)
static inline uint32_t ThumbExpandImm_C(uint32_t opcode, uint32_t carry_in,
                                        uint32_t &carry_out) {
  uint32_t imm32 = 0; // the expanded result
  const uint32_t i = bit(opcode, 26);
  const uint32_t imm3 = bits(opcode, 14, 12);
  const uint32_t abcdefgh = bits(opcode, 7, 0);
  const uint32_t imm12 = i << 11 | imm3 << 8 | abcdefgh;

```
- **EN**: Implements logic around `ARMExpandImm`, `ARMExpandImm_C`, `ThumbExpandImm_C`, `bit`, and 1 more symbols.
- **CN**: 围绕 `ARMExpandImm`, `ARMExpandImm_C`, `ThumbExpandImm_C`, `bit`, and 1 more symbols 实现具体逻辑。

### Lines 312-324
```cpp
  if (bits(imm12, 11, 10) == 0) {
    switch (bits(imm12, 9, 8)) {
    default: // Keep static analyzer happy with a default case
      break;

    case 0:
      imm32 = abcdefgh;
      break;

    case 1:
      imm32 = abcdefgh << 16 | abcdefgh;
      break;

```
- **EN**: Implements logic around `bits`.
- **CN**: 围绕 `bits` 实现具体逻辑。

### Lines 325-341
```cpp
    case 2:
      imm32 = abcdefgh << 24 | abcdefgh << 8;
      break;

    case 3:
      imm32 = abcdefgh << 24 | abcdefgh << 16 | abcdefgh << 8 | abcdefgh;
      break;
    }
    carry_out = carry_in;
  } else {
    const uint32_t unrotated_value = 0x80 | bits(imm12, 6, 0);
    imm32 = ror(unrotated_value, 32, bits(imm12, 11, 7));
    carry_out = Bit32(imm32, 31);
  }
  return imm32;
}

```
- **EN**: Implements logic around `bits`, `ror`, `Bit32`.
- **CN**: 围绕 `bits`, `ror`, `Bit32` 实现具体逻辑。

### Lines 342-358
```cpp
static inline uint32_t ThumbExpandImm(uint32_t opcode) {
  // 'carry_in' argument to following function call does not affect the imm32
  // result.
  uint32_t carry_in = 0;
  uint32_t carry_out;
  return ThumbExpandImm_C(opcode, carry_in, carry_out);
}

// imm32 = ZeroExtend(i:imm3:imm8, 32)
static inline uint32_t ThumbImm12(uint32_t opcode) {
  const uint32_t i = bit(opcode, 26);
  const uint32_t imm3 = bits(opcode, 14, 12);
  const uint32_t imm8 = bits(opcode, 7, 0);
  const uint32_t imm12 = i << 11 | imm3 << 8 | imm8;
  return imm12;
}

```
- **EN**: Implements logic around `ThumbExpandImm`, `ThumbExpandImm_C`, `ThumbImm12`, `bit`, and 1 more symbols.
- **CN**: 围绕 `ThumbExpandImm`, `ThumbExpandImm_C`, `ThumbImm12`, `bit`, and 1 more symbols 实现具体逻辑。

### Lines 359-370
```cpp
// imm32 = ZeroExtend(imm7:'00', 32)
static inline uint32_t ThumbImm7Scaled(uint32_t opcode) {
  const uint32_t imm7 = bits(opcode, 6, 0);
  return imm7 * 4;
}

// imm32 = ZeroExtend(imm8:'00', 32)
static inline uint32_t ThumbImm8Scaled(uint32_t opcode) {
  const uint32_t imm8 = bits(opcode, 7, 0);
  return imm8 * 4;
}

```
- **EN**: Implements logic around `ThumbImm7Scaled`, `bits`, `ThumbImm8Scaled`.
- **CN**: 围绕 `ThumbImm7Scaled`, `bits`, `ThumbImm8Scaled` 实现具体逻辑。

### Lines 371-377
```cpp
// This function performs the check for the register numbers 13 and 15 that are
// not permitted for many Thumb register specifiers.
static inline bool BadReg(uint32_t n) { return n == 13 || n == 15; }

} // namespace lldb_private

#endif // LLDB_SOURCE_PLUGINS_PROCESS_UTILITY_ARMUTILS_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `ARMDefines.h`, `InstructionUtils.h`, `llvm/ADT/bit.h`, `llvm/Support/MathExtras.h`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1), LLVM support-library helpers / LLVM Support 库辅助组件 (1)
