# ARMAddressingModes.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMAddressingModes.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains the ARM addressing mode implementation stuff.
- 用途 (CN): 声明 ARM 后端中的 `ARMAddressingModes`，并提供与MC 层目标描述与编码支持相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMAddressingModes.h - ARM Addressing Modes -------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the ARM addressing mode implementation stuff.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_MCTARGETDESC_ARMADDRESSINGMODES_H
#define LLVM_LIB_TARGET_ARM_MCTARGETDESC_ARMADDRESSINGMODES_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-21
```cpp
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/bit.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 23-23
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 25-35
```cpp
/// ARM_AM - ARM Addressing Mode Stuff
namespace ARM_AM {
  enum ShiftOpc {
    no_shift = 0,
    asr,
    lsl,
    lsr,
    ror,
    rrx,
    uxtw
  };
```
- EN: Defines enumeration `ShiftOpc` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `ShiftOpc`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 37-40
```cpp
  enum AddrOpc {
    sub = 0,
    add
  };
```
- EN: Defines enumeration `AddrOpc` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `AddrOpc`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 42-42
```cpp
  inline const char *getAddrOpcStr(AddrOpc Op) { return Op == sub ? "-" : ""; }
```
- EN: Implements `getAddrOpcStr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAddrOpcStr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 44-54
```cpp
  inline StringRef getShiftOpcStr(ShiftOpc Op) {
    switch (Op) {
    default: llvm_unreachable("Unknown shift opc!");
    case ARM_AM::asr: return "asr";
    case ARM_AM::lsl: return "lsl";
    case ARM_AM::lsr: return "lsr";
    case ARM_AM::ror: return "ror";
    case ARM_AM::rrx: return "rrx";
    case ARM_AM::uxtw: return "uxtw";
    }
  }
```
- EN: Implements `getShiftOpcStr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftOpcStr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-64
```cpp
  inline unsigned getShiftOpcEncoding(ShiftOpc Op) {
    switch (Op) {
    default: llvm_unreachable("Unknown shift opc!");
    case ARM_AM::asr: return 2;
    case ARM_AM::lsl: return 0;
    case ARM_AM::lsr: return 1;
    case ARM_AM::ror: return 3;
    }
  }
```
- EN: Implements `getShiftOpcEncoding`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getShiftOpcEncoding`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 66-72
```cpp
  enum AMSubMode {
    bad_am_submode = 0,
    ia,
    ib,
    da,
    db
  };
```
- EN: Defines enumeration `AMSubMode` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `AMSubMode`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 74-82
```cpp
  inline const char *getAMSubModeStr(AMSubMode Mode) {
    switch (Mode) {
    default: llvm_unreachable("Unknown addressing sub-mode!");
    case ARM_AM::ia: return "ia";
    case ARM_AM::ib: return "ib";
    case ARM_AM::da: return "da";
    case ARM_AM::db: return "db";
    }
  }
```
- EN: Implements `getAMSubModeStr`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAMSubModeStr`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 84-98
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #1: shift_operand with registers
  //===--------------------------------------------------------------------===//
  //
  // This 'addressing mode' is used for arithmetic instructions.  It can
  // represent things like:
  //   reg
  //   reg [asr|lsl|lsr|ror|rrx] reg
  //   reg [asr|lsl|lsr|ror|rrx] imm
  //
  // This is stored three operands [rega, regb, opc].  The first is the base
  // reg, the second is the shift amount (or reg0 if not present or imm).  The
  // third operand encodes the shift opcode and the imm if a reg isn't present.
  //
  inline unsigned getSORegOpc(ShiftOpc ShOp, unsigned Imm) {
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 99-102
```cpp
    return ShOp | (Imm << 3);
  }
  inline unsigned getSORegOffset(unsigned Op) { return Op >> 3; }
  inline ShiftOpc getSORegShOp(unsigned Op) { return (ShiftOpc)(Op & 7); }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 104-109
```cpp
  /// getSOImmValImm - Given an encoded imm field for the reg/imm form, return
  /// the 8-bit imm value.
  inline unsigned getSOImmValImm(unsigned Imm) { return Imm & 0xFF; }
  /// getSOImmValRot - Given an encoded imm field for the reg/imm form, return
  /// the rotate amount.
  inline unsigned getSOImmValRot(unsigned Imm) { return (Imm >> 8) * 2; }
```
- EN: Implements `getSOImmValImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSOImmValImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 111-118
```cpp
  /// getSOImmValRotate - Try to handle Imm with an immediate shifter operand,
  /// computing the rotate amount to use.  If this immediate value cannot be
  /// handled with a single shifter-op, determine a good rotate amount that will
  /// take a maximal chunk of bits out of the immediate.
  inline unsigned getSOImmValRotate(unsigned Imm) {
    // 8-bit (or less) immediates are trivially shifter_operands with a rotate
    // of zero.
    if ((Imm & ~255U) == 0) return 0;
```
- EN: Implements `getSOImmValRotate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSOImmValRotate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 120-121
```cpp
    // Use CTZ to compute the rotate amount.
    unsigned TZ = llvm::countr_zero(Imm);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 123-125
```cpp
    // Rotate amount must be even.  Something like 0x200 must be rotated 8 bits,
    // not 9.
    unsigned RotAmt = TZ & ~1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 127-129
```cpp
    // If we can handle this spread, return it.
    if ((llvm::rotr<uint32_t>(Imm, RotAmt) & ~255U) == 0)
      return (32-RotAmt)&31;  // HW rotates right, not left.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 131-138
```cpp
    // For values like 0xF000000F, we should ignore the low 6 bits, then
    // retry the hunt.
    if (Imm & 63U) {
      unsigned TZ2 = llvm::countr_zero(Imm & ~63U);
      unsigned RotAmt2 = TZ2 & ~1;
      if ((llvm::rotr<uint32_t>(Imm, RotAmt2) & ~255U) == 0)
        return (32-RotAmt2)&31;  // HW rotates right, not left.
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 140-144
```cpp
    // Otherwise, we have no way to cover this span of bits with a single
    // shifter_op immediate.  Return a chunk of bits that will be useful to
    // handle.
    return (32-RotAmt)&31;  // HW rotates right, not left.
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 146-152
```cpp
  /// getSOImmVal - Given a 32-bit immediate, if it is something that can fit
  /// into an shifter_operand immediate operand, return the 12-bit encoding for
  /// it.  If not, return -1.
  inline int getSOImmVal(unsigned Arg) {
    // 8-bit (or less) immediates are trivially shifter_operands with a rotate
    // of zero.
    if ((Arg & ~255U) == 0) return Arg;
```
- EN: Implements `getSOImmVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSOImmVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 154-154
```cpp
    unsigned RotAmt = getSOImmValRotate(Arg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 156-158
```cpp
    // If this cannot be handled with a single shifter_op, bail out.
    if (llvm::rotr<uint32_t>(~255U, RotAmt) & Arg)
      return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 160-162
```cpp
    // Encode this correctly.
    return llvm::rotl<uint32_t>(Arg, RotAmt) | ((RotAmt >> 1) << 8);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-170
```cpp
  /// isSOImmTwoPartVal - Return true if the specified value can be obtained by
  /// or'ing together two SOImmVal's.
  inline bool isSOImmTwoPartVal(unsigned V) {
    // If this can be handled with a single shifter_op, bail out.
    V = llvm::rotr<uint32_t>(~255U, getSOImmValRotate(V)) & V;
    if (V == 0)
      return false;
```
- EN: Implements `isSOImmTwoPartVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSOImmTwoPartVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 172-175
```cpp
    // If this can be handled with two shifter_op's, accept.
    V = llvm::rotr<uint32_t>(~255U, getSOImmValRotate(V)) & V;
    return V == 0;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 177-181
```cpp
  /// getSOImmTwoPartFirst - If V is a value that satisfies isSOImmTwoPartVal,
  /// return the first chunk of it.
  inline unsigned getSOImmTwoPartFirst(unsigned V) {
    return llvm::rotr<uint32_t>(255U, getSOImmValRotate(V)) & V;
  }
```
- EN: Implements `getSOImmTwoPartFirst`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSOImmTwoPartFirst`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 183-187
```cpp
  /// getSOImmTwoPartSecond - If V is a value that satisfies isSOImmTwoPartVal,
  /// return the second chunk of it.
  inline unsigned getSOImmTwoPartSecond(unsigned V) {
    // Mask out the first hunk.
    V = llvm::rotr<uint32_t>(~255U, getSOImmValRotate(V)) & V;
```
- EN: Implements `getSOImmTwoPartSecond`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSOImmTwoPartSecond`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 189-192
```cpp
    // Take what's left.
    assert(V == (llvm::rotr<uint32_t>(255U, getSOImmValRotate(V)) & V));
    return V;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 194-206
```cpp
  /// isSOImmTwoPartValNeg - Return true if the specified value can be obtained
  /// by two SOImmVal, that -V = First + Second.
  /// "R+V" can be optimized to (sub (sub R, First), Second).
  /// "R=V" can be optimized to (sub (mvn R, ~(-First)), Second).
  inline bool isSOImmTwoPartValNeg(unsigned V) {
    unsigned First;
    if (!isSOImmTwoPartVal(-V))
      return false;
    // Return false if ~(-First) is not a SoImmval.
    First = getSOImmTwoPartFirst(-V);
    First = ~(-First);
    return !(llvm::rotr<uint32_t>(~255U, getSOImmValRotate(First)) & First);
  }
```
- EN: Implements `isSOImmTwoPartValNeg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isSOImmTwoPartValNeg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 208-213
```cpp
  /// getThumbImmValShift - Try to handle Imm with a 8-bit immediate followed
  /// by a left shift. Returns the shift amount to use.
  inline unsigned getThumbImmValShift(unsigned Imm) {
    // 8-bit (or less) immediates are trivially immediate operand with a shift
    // of zero.
    if ((Imm & ~255U) == 0) return 0;
```
- EN: Implements `getThumbImmValShift`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbImmValShift`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 215-217
```cpp
    // Use CTZ to compute the shift amount.
    return llvm::countr_zero(Imm);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-225
```cpp
  /// isThumbImmShiftedVal - Return true if the specified value can be obtained
  /// by left shifting a 8-bit immediate.
  inline bool isThumbImmShiftedVal(unsigned V) {
    // If this can be handled with
    V = (~255U << getThumbImmValShift(V)) & V;
    return V == 0;
  }
```
- EN: Implements `isThumbImmShiftedVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isThumbImmShiftedVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 227-232
```cpp
  /// getThumbImm16ValShift - Try to handle Imm with a 16-bit immediate followed
  /// by a left shift. Returns the shift amount to use.
  inline unsigned getThumbImm16ValShift(unsigned Imm) {
    // 16-bit (or less) immediates are trivially immediate operand with a shift
    // of zero.
    if ((Imm & ~65535U) == 0) return 0;
```
- EN: Implements `getThumbImm16ValShift`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbImm16ValShift`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 234-236
```cpp
    // Use CTZ to compute the shift amount.
    return llvm::countr_zero(Imm);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 238-244
```cpp
  /// isThumbImm16ShiftedVal - Return true if the specified value can be
  /// obtained by left shifting a 16-bit immediate.
  inline bool isThumbImm16ShiftedVal(unsigned V) {
    // If this can be handled with
    V = (~65535U << getThumbImm16ValShift(V)) & V;
    return V == 0;
  }
```
- EN: Implements `isThumbImm16ShiftedVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isThumbImm16ShiftedVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-250
```cpp
  /// getThumbImmNonShiftedVal - If V is a value that satisfies
  /// isThumbImmShiftedVal, return the non-shiftd value.
  inline unsigned getThumbImmNonShiftedVal(unsigned V) {
    return V >> getThumbImmValShift(V);
  }
```
- EN: Implements `getThumbImmNonShiftedVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getThumbImmNonShiftedVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 253-266
```cpp
  /// getT2SOImmValSplat - Return the 12-bit encoded representation
  /// if the specified value can be obtained by splatting the low 8 bits
  /// into every other byte or every byte of a 32-bit value. i.e.,
  ///     00000000 00000000 00000000 abcdefgh    control = 0
  ///     00000000 abcdefgh 00000000 abcdefgh    control = 1
  ///     abcdefgh 00000000 abcdefgh 00000000    control = 2
  ///     abcdefgh abcdefgh abcdefgh abcdefgh    control = 3
  /// Return -1 if none of the above apply.
  /// See ARM Reference Manual A6.3.2.
  inline int getT2SOImmValSplatVal(unsigned V) {
    unsigned u, Vs, Imm;
    // control = 0
    if ((V & 0xffffff00) == 0)
      return V;
```
- EN: Implements `getT2SOImmValSplatVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SOImmValSplatVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 268-273
```cpp
    // If the value is zeroes in the first byte, just shift those off
    Vs = ((V & 0xff) == 0) ? V >> 8 : V;
    // Any passing value only has 8 bits of payload, splatted across the word
    Imm = Vs & 0xff;
    // Likewise, any passing values have the payload splatted into the 3rd byte
    u = Imm | (Imm << 16);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 275-277
```cpp
    // control = 1 or 2
    if (Vs == u)
      return (((Vs == V) ? 1 : 2) << 8) | Imm;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-281
```cpp
    // control = 3
    if (Vs == (u | (u << 8)))
      return (3 << 8) | Imm;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 283-284
```cpp
    return -1;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 286-293
```cpp
  /// getT2SOImmValRotateVal - Return the 12-bit encoded representation if the
  /// specified value is a rotated 8-bit value. Return -1 if no rotation
  /// encoding is possible.
  /// See ARM Reference Manual A6.3.2.
  inline int getT2SOImmValRotateVal(unsigned V) {
    unsigned RotAmt = llvm::countl_zero(V);
    if (RotAmt >= 24)
      return -1;
```
- EN: Implements `getT2SOImmValRotateVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SOImmValRotateVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 295-298
```cpp
    // If 'Arg' can be handled with a single shifter_op return the value.
    if ((llvm::rotr<uint32_t>(0xff000000U, RotAmt) & V) == V)
      return (llvm::rotr<uint32_t>(V, 24 - RotAmt) & 0x7f) |
             ((RotAmt + 8) << 7);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 300-301
```cpp
    return -1;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 303-311
```cpp
  /// getT2SOImmVal - Given a 32-bit immediate, if it is something that can fit
  /// into a Thumb-2 shifter_operand immediate operand, return the 12-bit
  /// encoding for it.  If not, return -1.
  /// See ARM Reference Manual A6.3.2.
  inline int getT2SOImmVal(unsigned Arg) {
    // If 'Arg' is an 8-bit splat, then get the encoded value.
    int Splat = getT2SOImmValSplatVal(Arg);
    if (Splat != -1)
      return Splat;
```
- EN: Implements `getT2SOImmVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SOImmVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 313-316
```cpp
    // If 'Arg' can be handled with a single shifter_op return the value.
    int Rot = getT2SOImmValRotateVal(Arg);
    if (Rot != -1)
      return Rot;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 318-319
```cpp
    return -1;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 321-326
```cpp
  inline unsigned getT2SOImmValRotate(unsigned V) {
    if ((V & ~255U) == 0) return 0;
    // Use CTZ to compute the rotate amount.
    unsigned RotAmt = llvm::countr_zero(V);
    return (32 - RotAmt) & 31;
  }
```
- EN: Implements `getT2SOImmValRotate`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SOImmValRotate`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 328-337
```cpp
  inline bool isT2SOImmTwoPartVal(unsigned Imm) {
    unsigned V = Imm;
    // Passing values can be any combination of splat values and shifter
    // values. If this can be handled with a single shifter or splat, bail
    // out. Those should be handled directly, not with a two-part val.
    if (getT2SOImmValSplatVal(V) != -1)
      return false;
    V = llvm::rotr<uint32_t>(~255U, getT2SOImmValRotate(V)) & V;
    if (V == 0)
      return false;
```
- EN: Implements `isT2SOImmTwoPartVal`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isT2SOImmTwoPartVal`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 339-340
```cpp
    // If this can be handled as an immediate, accept.
    if (getT2SOImmVal(V) != -1) return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 342-349
```cpp
    // Likewise, try masking out a splat value first.
    V = Imm;
    if (getT2SOImmValSplatVal(V & 0xff00ff00U) != -1)
      V &= ~0xff00ff00U;
    else if (getT2SOImmValSplatVal(V & 0x00ff00ffU) != -1)
      V &= ~0x00ff00ffU;
    // If what's left can be handled as an immediate, accept.
    if (getT2SOImmVal(V) != -1) return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 351-353
```cpp
    // Otherwise, do not accept.
    return false;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 355-361
```cpp
  inline unsigned getT2SOImmTwoPartFirst(unsigned Imm) {
    assert (isT2SOImmTwoPartVal(Imm) &&
            "Immedate cannot be encoded as two part immediate!");
    // Try a shifter operand as one part
    unsigned V = llvm::rotr<uint32_t>(~255, getT2SOImmValRotate(Imm)) & Imm;
    // If the rest is encodable as an immediate, then return it.
    if (getT2SOImmVal(V) != -1) return V;
```
- EN: Implements `getT2SOImmTwoPartFirst`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SOImmTwoPartFirst`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 363-365
```cpp
    // Try masking out a splat value first.
    if (getT2SOImmValSplatVal(Imm & 0xff00ff00U) != -1)
      return Imm & 0xff00ff00U;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 367-370
```cpp
    // The other splat is all that's left as an option.
    assert (getT2SOImmValSplatVal(Imm & 0x00ff00ffU) != -1);
    return Imm & 0x00ff00ffU;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 372-379
```cpp
  inline unsigned getT2SOImmTwoPartSecond(unsigned Imm) {
    // Mask out the first hunk
    Imm ^= getT2SOImmTwoPartFirst(Imm);
    // Return what's left
    assert (getT2SOImmVal(Imm) != -1 &&
            "Unable to encode second part of T2 two part SO immediate");
    return Imm;
  }
```
- EN: Implements `getT2SOImmTwoPartSecond`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getT2SOImmTwoPartSecond`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 382-396
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #2
  //===--------------------------------------------------------------------===//
  //
  // This is used for most simple load/store instructions.
  //
  // addrmode2 := reg +/- reg shop imm
  // addrmode2 := reg +/- imm12
  //
  // The first operand is always a Reg.  The second operand is a reg if in
  // reg/reg form, otherwise it's reg#0.  The third field encodes the operation
  // in bit 12, the immediate in bits 0-11, and the shift op in 13-15. The
  // fourth operand 16-17 encodes the index mode.
  //
  // If this addressing mode is a frame index (before prolog/epilog insertion
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 397-411
```cpp
  // and code rewriting), this operand will have the form:  FI#, reg0, <offs>
  // with no shift amount for the frame offset.
  //
  inline unsigned getAM2Opc(AddrOpc Opc, unsigned Imm12, ShiftOpc SO,
                            unsigned IdxMode = 0) {
    assert(Imm12 < (1 << 12) && "Imm too large!");
    bool isSub = Opc == sub;
    return Imm12 | ((int)isSub << 12) | (SO << 13) | (IdxMode << 16) ;
  }
  inline unsigned getAM2Offset(unsigned AM2Opc) {
    return AM2Opc & ((1 << 12)-1);
  }
  inline AddrOpc getAM2Op(unsigned AM2Opc) {
    return ((AM2Opc >> 12) & 1) ? sub : add;
  }
```
- EN: Implements `getAM2Opc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM2Opc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 412-415
```cpp
  inline ShiftOpc getAM2ShiftOpc(unsigned AM2Opc) {
    return (ShiftOpc)((AM2Opc >> 13) & 7);
  }
  inline unsigned getAM2IdxMode(unsigned AM2Opc) { return (AM2Opc >> 16); }
```
- EN: Implements `getAM2ShiftOpc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM2ShiftOpc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-429
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #3
  //===--------------------------------------------------------------------===//
  //
  // This is used for sign-extending loads, and load/store-pair instructions.
  //
  // addrmode3 := reg +/- reg
  // addrmode3 := reg +/- imm8
  //
  // The first operand is always a Reg.  The second operand is a reg if in
  // reg/reg form, otherwise it's reg#0.  The third field encodes the operation
  // in bit 8, the immediate in bits 0-7. The fourth operand 9-10 encodes the
  // index mode.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 431-441
```cpp
  /// getAM3Opc - This function encodes the addrmode3 opc field.
  inline unsigned getAM3Opc(AddrOpc Opc, unsigned char Offset,
                            unsigned IdxMode = 0) {
    bool isSub = Opc == sub;
    return ((int)isSub << 8) | Offset | (IdxMode << 9);
  }
  inline unsigned char getAM3Offset(unsigned AM3Opc) { return AM3Opc & 0xFF; }
  inline AddrOpc getAM3Op(unsigned AM3Opc) {
    return ((AM3Opc >> 8) & 1) ? sub : add;
  }
  inline unsigned getAM3IdxMode(unsigned AM3Opc) { return (AM3Opc >> 9); }
```
- EN: Implements `getAM3Opc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM3Opc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 443-456
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #4
  //===--------------------------------------------------------------------===//
  //
  // This is used for load / store multiple instructions.
  //
  // addrmode4 := reg, <mode>
  //
  // The four modes are:
  //    IA - Increment after
  //    IB - Increment before
  //    DA - Decrement after
  //    DB - Decrement before
  // For VFP instructions, only the IA and DB modes are valid.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 458-460
```cpp
  inline AMSubMode getAM4SubMode(unsigned Mode) {
    return (AMSubMode)(Mode & 0x7);
  }
```
- EN: Implements `getAM4SubMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM4SubMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 462-462
```cpp
  inline unsigned getAM4ModeImm(AMSubMode SubMode) { return (int)SubMode; }
```
- EN: Implements `getAM4ModeImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM4ModeImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 464-473
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #5
  //===--------------------------------------------------------------------===//
  //
  // This is used for coprocessor instructions, such as FP load/stores.
  //
  // addrmode5 := reg +/- imm8*4
  //
  // The first operand is always a Reg.  The second operand encodes the
  // operation (add or subtract) in bit 8 and the immediate in bits 0-7.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 475-483
```cpp
  /// getAM5Opc - This function encodes the addrmode5 opc field.
  inline unsigned getAM5Opc(AddrOpc Opc, unsigned char Offset) {
    bool isSub = Opc == sub;
    return ((int)isSub << 8) | Offset;
  }
  inline unsigned char getAM5Offset(unsigned AM5Opc) { return AM5Opc & 0xFF; }
  inline AddrOpc getAM5Op(unsigned AM5Opc) {
    return ((AM5Opc >> 8) & 1) ? sub : add;
  }
```
- EN: Implements `getAM5Opc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM5Opc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 485-494
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #5 FP16
  //===--------------------------------------------------------------------===//
  //
  // This is used for coprocessor instructions, such as 16-bit FP load/stores.
  //
  // addrmode5fp16 := reg +/- imm8*2
  //
  // The first operand is always a Reg.  The second operand encodes the
  // operation (add or subtract) in bit 8 and the immediate in bits 0-7.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 496-506
```cpp
  /// getAM5FP16Opc - This function encodes the addrmode5fp16 opc field.
  inline unsigned getAM5FP16Opc(AddrOpc Opc, unsigned char Offset) {
    bool isSub = Opc == sub;
    return ((int)isSub << 8) | Offset;
  }
  inline unsigned char getAM5FP16Offset(unsigned AM5Opc) {
    return AM5Opc & 0xFF;
  }
  inline AddrOpc getAM5FP16Op(unsigned AM5Opc) {
    return ((AM5Opc >> 8) & 1) ? sub : add;
  }
```
- EN: Implements `getAM5FP16Opc`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAM5FP16Opc`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 508-519
```cpp
  //===--------------------------------------------------------------------===//
  // Addressing Mode #6
  //===--------------------------------------------------------------------===//
  //
  // This is used for NEON load / store instructions.
  //
  // addrmode6 := reg with optional alignment
  //
  // This is stored in two operands [regaddr, align].  The first is the
  // address register.  The second operand is the value of the alignment
  // specifier in bytes or zero if no explicit alignment.
  // Valid alignments depend on the specific instruction.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 521-531
```cpp
  //===--------------------------------------------------------------------===//
  // NEON/MVE Modified Immediates
  //===--------------------------------------------------------------------===//
  //
  // Several NEON and MVE instructions (e.g., VMOV) take a "modified immediate"
  // vector operand, where a small immediate encoded in the instruction
  // specifies a full NEON vector value.  These modified immediates are
  // represented here as encoded integers.  The low 8 bits hold the immediate
  // value; bit 12 holds the "Op" field of the instruction, and bits 11-8 hold
  // the "Cmode" field of the instruction.  The interfaces below treat the
  // Op and Cmode values as a single 5-bit value.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 533-539
```cpp
  inline unsigned createVMOVModImm(unsigned OpCmode, unsigned Val) {
    return (OpCmode << 8) | Val;
  }
  inline unsigned getVMOVModImmOpCmode(unsigned ModImm) {
    return (ModImm >> 8) & 0x1f;
  }
  inline unsigned getVMOVModImmVal(unsigned ModImm) { return ModImm & 0xff; }
```
- EN: Implements `createVMOVModImm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createVMOVModImm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 541-547
```cpp
  /// decodeVMOVModImm - Decode a NEON/MVE modified immediate value into the
  /// element value and the element size in bits.  (If the element size is
  /// smaller than the vector, it is splatted into all the elements.)
  inline uint64_t decodeVMOVModImm(unsigned ModImm, unsigned &EltBits) {
    unsigned OpCmode = getVMOVModImmOpCmode(ModImm);
    unsigned Imm8 = getVMOVModImmVal(ModImm);
    uint64_t Val = 0;
```
- EN: Implements `decodeVMOVModImm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `decodeVMOVModImm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 549-563
```cpp
    if (OpCmode == 0xe) {
      // 8-bit vector elements
      Val = Imm8;
      EltBits = 8;
    } else if ((OpCmode & 0xc) == 0x8) {
      // 16-bit vector elements
      unsigned ByteNum = (OpCmode & 0x6) >> 1;
      Val = Imm8 << (8 * ByteNum);
      EltBits = 16;
    } else if ((OpCmode & 0x8) == 0) {
      // 32-bit vector elements, zero with one byte set
      unsigned ByteNum = (OpCmode & 0x6) >> 1;
      Val = Imm8 << (8 * ByteNum);
      EltBits = 32;
    } else if ((OpCmode & 0xe) == 0xc) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 564-578
```cpp
      // 32-bit vector elements, one byte with low bits set
      unsigned ByteNum = 1 + (OpCmode & 0x1);
      Val = (Imm8 << (8 * ByteNum)) | (0xffff >> (8 * (2 - ByteNum)));
      EltBits = 32;
    } else if (OpCmode == 0x1e) {
      // 64-bit vector elements
      for (unsigned ByteNum = 0; ByteNum < 8; ++ByteNum) {
        if ((ModImm >> ByteNum) & 1)
          Val |= (uint64_t)0xff << (8 * ByteNum);
      }
      EltBits = 64;
    } else {
      llvm_unreachable("Unsupported VMOV immediate");
    }
    return Val;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 579-579
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 581-590
```cpp
  // Generic validation for single-byte immediate (0X00, 00X0, etc).
  inline bool isNEONBytesplat(unsigned Value, unsigned Size) {
    assert(Size >= 1 && Size <= 4 && "Invalid size");
    unsigned count = 0;
    for (unsigned i = 0; i < Size; ++i) {
      if (Value & 0xff) count++;
      Value >>= 8;
    }
    return count == 1;
  }
```
- EN: Implements `isNEONBytesplat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isNEONBytesplat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 592-598
```cpp
  /// Checks if Value is a correct immediate for instructions like VBIC/VORR.
  inline bool isNEONi16splat(unsigned Value) {
    if (Value > 0xffff)
      return false;
    // i16 value with set bits only in one byte X0 or 0X.
    return Value == 0 || isNEONBytesplat(Value, 2);
  }
```
- EN: Implements `isNEONi16splat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isNEONi16splat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 600-608
```cpp
  // Encode NEON 16 bits Splat immediate for instructions like VBIC/VORR
  inline unsigned encodeNEONi16splat(unsigned Value) {
    assert(isNEONi16splat(Value) && "Invalid NEON splat value");
    if (Value >= 0x100)
      Value = (Value >> 8) | 0xa00;
    else
      Value |= 0x800;
    return Value;
  }
```
- EN: Implements `encodeNEONi16splat`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `encodeNEONi16splat`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 610-614
```cpp
  /// Checks if Value is a correct immediate for instructions like VBIC/VORR.
  inline bool isNEONi32splat(unsigned Value) {
    // i32 value with set bits only in one byte X000, 0X00, 00X0, or 000X.
    return Value == 0 || isNEONBytesplat(Value, 4);
  }
```
- EN: Implements `isNEONi32splat`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isNEONi32splat`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 616-626
```cpp
  /// Encode NEON 32 bits Splat immediate for instructions like VBIC/VORR.
  inline unsigned encodeNEONi32splat(unsigned Value) {
    assert(isNEONi32splat(Value) && "Invalid NEON splat value");
    if (Value >= 0x100 && Value <= 0xff00)
      Value = (Value >> 8) | 0x200;
    else if (Value > 0xffff && Value <= 0xff0000)
      Value = (Value >> 16) | 0x400;
    else if (Value > 0xffffff)
      Value = (Value >> 24) | 0x600;
    return Value;
  }
```
- EN: Implements `encodeNEONi32splat`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `encodeNEONi32splat`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 628-632
```cpp
  //===--------------------------------------------------------------------===//
  // Floating-point Immediates
  //
  inline float getFPImmFloat(unsigned Imm) {
    // We expect an 8-bit binary encoding of a floating-point number here.
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 634-636
```cpp
    uint8_t Sign = (Imm >> 7) & 0x1;
    uint8_t Exp = (Imm >> 4) & 0x7;
    uint8_t Mantissa = Imm & 0xf;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 638-649
```cpp
    //   8-bit FP    IEEE Float Encoding
    //   abcd efgh   aBbbbbbc defgh000 00000000 00000000
    //
    // where B = NOT(b);
    uint32_t I = 0;
    I |= Sign << 31;
    I |= ((Exp & 0x4) != 0 ? 0 : 1) << 30;
    I |= ((Exp & 0x4) != 0 ? 0x1f : 0) << 25;
    I |= (Exp & 0x3) << 23;
    I |= Mantissa << 19;
    return bit_cast<float>(I);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 651-657
```cpp
  /// getFP16Imm - Return an 8-bit floating-point version of the 16-bit
  /// floating-point value. If the value cannot be represented as an 8-bit
  /// floating-point value, then return -1.
  inline int getFP16Imm(const APInt &Imm) {
    uint32_t Sign = Imm.lshr(15).getZExtValue() & 1;
    int32_t Exp = (Imm.lshr(10).getSExtValue() & 0x1f) - 15;  // -14 to 15
    int64_t Mantissa = Imm.getZExtValue() & 0x3ff;  // 10 bits
```
- EN: Implements `getFP16Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP16Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 659-663
```cpp
    // We can handle 4 bits of mantissa.
    // mantissa = (16+UInt(e:f:g:h))/16.
    if (Mantissa & 0x3f)
      return -1;
    Mantissa >>= 6;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 665-668
```cpp
    // We can handle 3 bits of exponent: exp == UInt(NOT(b):c:d)-3
    if (Exp < -3 || Exp > 4)
      return -1;
    Exp = ((Exp+3) & 0x7) ^ 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 670-671
```cpp
    return ((int)Sign << 7) | (Exp << 4) | Mantissa;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 673-675
```cpp
  inline int getFP16Imm(const APFloat &FPImm) {
    return getFP16Imm(FPImm.bitcastToAPInt());
  }
```
- EN: Implements `getFP16Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP16Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 677-683
```cpp
  /// If this is a FP16Imm encoded as a fp32 value, return the 8-bit encoding
  /// for it. Otherwise return -1 like getFP16Imm.
  inline int getFP32FP16Imm(const APInt &Imm) {
    if (Imm.getActiveBits() > 16)
      return -1;
    return ARM_AM::getFP16Imm(Imm.trunc(16));
  }
```
- EN: Implements `getFP32FP16Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP32FP16Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 685-687
```cpp
  inline int getFP32FP16Imm(const APFloat &FPImm) {
    return getFP32FP16Imm(FPImm.bitcastToAPInt());
  }
```
- EN: Implements `getFP32FP16Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP32FP16Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 689-695
```cpp
  /// getFP32Imm - Return an 8-bit floating-point version of the 32-bit
  /// floating-point value. If the value cannot be represented as an 8-bit
  /// floating-point value, then return -1.
  inline int getFP32Imm(const APInt &Imm) {
    uint32_t Sign = Imm.lshr(31).getZExtValue() & 1;
    int32_t Exp = (Imm.lshr(23).getSExtValue() & 0xff) - 127;  // -126 to 127
    int64_t Mantissa = Imm.getZExtValue() & 0x7fffff;  // 23 bits
```
- EN: Implements `getFP32Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP32Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 697-703
```cpp
    // We can handle 4 bits of mantissa.
    // mantissa = (16+UInt(e:f:g:h))/16.
    if (Mantissa & 0x7ffff)
      return -1;
    Mantissa >>= 19;
    if ((Mantissa & 0xf) != Mantissa)
      return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 705-708
```cpp
    // We can handle 3 bits of exponent: exp == UInt(NOT(b):c:d)-3
    if (Exp < -3 || Exp > 4)
      return -1;
    Exp = ((Exp+3) & 0x7) ^ 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 710-711
```cpp
    return ((int)Sign << 7) | (Exp << 4) | Mantissa;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-715
```cpp
  inline int getFP32Imm(const APFloat &FPImm) {
    return getFP32Imm(FPImm.bitcastToAPInt());
  }
```
- EN: Implements `getFP32Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP32Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 717-723
```cpp
  /// getFP64Imm - Return an 8-bit floating-point version of the 64-bit
  /// floating-point value. If the value cannot be represented as an 8-bit
  /// floating-point value, then return -1.
  inline int getFP64Imm(const APInt &Imm) {
    uint64_t Sign = Imm.lshr(63).getZExtValue() & 1;
    int64_t Exp = (Imm.lshr(52).getSExtValue() & 0x7ff) - 1023; // -1022 to 1023
    uint64_t Mantissa = Imm.getZExtValue() & 0xfffffffffffffULL;
```
- EN: Implements `getFP64Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP64Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 725-731
```cpp
    // We can handle 4 bits of mantissa.
    // mantissa = (16+UInt(e:f:g:h))/16.
    if (Mantissa & 0xffffffffffffULL)
      return -1;
    Mantissa >>= 48;
    if ((Mantissa & 0xf) != Mantissa)
      return -1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 733-736
```cpp
    // We can handle 3 bits of exponent: exp == UInt(NOT(b):c:d)-3
    if (Exp < -3 || Exp > 4)
      return -1;
    Exp = ((Exp+3) & 0x7) ^ 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 738-739
```cpp
    return ((int)Sign << 7) | (Exp << 4) | Mantissa;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 741-743
```cpp
  inline int getFP64Imm(const APFloat &FPImm) {
    return getFP64Imm(FPImm.bitcastToAPInt());
  }
```
- EN: Implements `getFP64Imm`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getFP64Imm`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 745-746
```cpp
} // end namespace ARM_AM
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 748-748
```cpp
#endif
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。

## Dependencies / 依赖关系

- EN: Backend-local headers: `(none)`.
  - CN: 后端本地头文件：`(none)`。
- EN: LLVM infrastructure headers: `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/bit.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/MathExtras.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/bit.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/MathExtras.h`。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
