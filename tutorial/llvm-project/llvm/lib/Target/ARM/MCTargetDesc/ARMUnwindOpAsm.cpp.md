# ARMUnwindOpAsm.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMUnwindOpAsm.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the unwind opcode assembler for ARM exception handling table.
- 用途 (CN): 实现 ARM 后端中的 `ARMUnwindOpAsm`，重点处理MC 层目标描述与编码支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- ARMUnwindOpAsm.cpp - ARM Unwind Opcodes Assembler -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the unwind opcode assembler for ARM exception handling
// table.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-18
```cpp
#include "ARMUnwindOpAsm.h"
#include "llvm/ADT/bit.h"
#include "llvm/Support/ARMEHABI.h"
#include "llvm/Support/LEB128.h"
#include <cassert>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 20-20
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 22-22
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 24-31
```cpp
  /// UnwindOpcodeStreamer - The simple wrapper over SmallVector to emit bytes
  /// with MSB to LSB per uint32_t ordering.  For example, the first byte will
  /// be placed in Vec[3], and the following bytes will be placed in 2, 1, 0,
  /// 7, 6, 5, 4, 11, 10, 9, 8, and so on.
  class UnwindOpcodeStreamer {
  private:
    SmallVectorImpl<uint8_t> &Vec;
    size_t Pos = 3;
```
- EN: Declares `UnwindOpcodeStreamer`, packaging target-specific state and APIs around `ARMUnwindOpAsm`.
- CN: 这里声明 `UnwindOpcodeStreamer`，把与 `ARMUnwindOpAsm` 相关的目标特定状态和 API 组织在一起。

### Lines 33-34
```cpp
  public:
    UnwindOpcodeStreamer(SmallVectorImpl<uint8_t> &V) : Vec(V) {}
```
- EN: Implements `UnwindOpcodeStreamer`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnwindOpcodeStreamer`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 36-40
```cpp
    /// Emit the byte in MSB to LSB per uint32_t order.
    void EmitByte(uint8_t elem) {
      Vec[Pos] = elem;
      Pos = (((Pos ^ 0x3u) + 1) ^ 0x3u);
    }
```
- EN: Implements `EmitByte`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `EmitByte`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-48
```cpp
    /// Emit the size prefix.
    void EmitSize(size_t Size) {
      size_t SizeInWords = (Size + 3) / 4;
      assert(SizeInWords <= 0x100u &&
             "Only 256 additional words are allowed for unwind opcodes");
      EmitByte(static_cast<uint8_t>(SizeInWords - 1));
    }
```
- EN: Implements `EmitSize`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `EmitSize`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-55
```cpp
    /// Emit the personality index prefix.
    void EmitPersonalityIndex(unsigned PI) {
      assert(PI < ARM::EHABI::NUM_PERSONALITY_INDEX &&
             "Invalid personality prefix");
      EmitByte(ARM::EHABI::EHT_COMPACT | PI);
    }
```
- EN: Implements `EmitPersonalityIndex`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `EmitPersonalityIndex`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-62
```cpp
    /// Fill the rest of bytes with FINISH opcode.
    void FillFinishOpcode() {
      while (Pos < Vec.size())
        EmitByte(ARM::EHABI::UNWIND_OPCODE_FINISH);
    }
  };
```
- EN: Implements `FillFinishOpcode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `FillFinishOpcode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 64-64
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 66-71
```cpp
void UnwindOpcodeAssembler::EmitRegSave(uint32_t RegSave) {
  if (RegSave == 0u) {
    // That's the special case for RA PAC.
    EmitInt8(ARM::EHABI::UNWIND_OPCODE_POP_RA_AUTH_CODE);
    return;
  }
```
- EN: Implements `UnwindOpcodeAssembler::EmitRegSave`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnwindOpcodeAssembler::EmitRegSave`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 73-76
```cpp
  // One byte opcode to save register r14 and r11-r4
  if (RegSave & (1u << 4)) {
    // The one byte opcode will always save r4, thus we can't use the one byte
    // opcode when r4 is not in .save directive.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 78-82
```cpp
    // Compute the consecutive registers from r4 to r11.
    uint32_t Mask = RegSave & 0xff0u;
    uint32_t Range = llvm::countr_one(Mask >> 5); // Exclude r4.
    // Mask off non-consecutive registers. Keep r4.
    Mask &= ~(0xffffffe0u << Range);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-95
```cpp
    // Emit this opcode when the mask covers every registers.
    uint32_t UnmaskedReg = RegSave & 0xfff0u & (~Mask);
    if (UnmaskedReg == 0u) {
      // Pop r[4 : (4 + n)]
      EmitInt8(ARM::EHABI::UNWIND_OPCODE_POP_REG_RANGE_R4 | Range);
      RegSave &= 0x000fu;
    } else if (UnmaskedReg == (1u << 14)) {
      // Pop r[14] + r[4 : (4 + n)]
      EmitInt8(ARM::EHABI::UNWIND_OPCODE_POP_REG_RANGE_R4_R14 | Range);
      RegSave &= 0x000fu;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 97-99
```cpp
  // Two bytes opcode to save register r15-r4
  if ((RegSave & 0xfff0u) != 0)
    EmitInt16(ARM::EHABI::UNWIND_OPCODE_POP_REG_MASK_R4 | (RegSave >> 4));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 101-104
```cpp
  // Opcode to save register r3-r0
  if ((RegSave & 0x000fu) != 0)
    EmitInt16(ARM::EHABI::UNWIND_OPCODE_POP_REG_MASK | (RegSave & 0x000fu));
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 106-115
```cpp
/// Emit unwind opcodes for .vsave directives
void UnwindOpcodeAssembler::EmitVFPRegSave(uint32_t VFPRegSave) {
  // We only have 4 bits to save the offset in the opcode so look at the lower
  // and upper 16 bits separately.
  for (uint32_t Regs : {VFPRegSave & 0xffff0000u, VFPRegSave & 0x0000ffffu}) {
    while (Regs) {
      // Now look for a run of set bits. Remember the MSB and LSB of the run.
      auto RangeMSB = llvm::bit_width(Regs);
      auto RangeLen = llvm::countl_one(Regs << (32 - RangeMSB));
      auto RangeLSB = RangeMSB - RangeLen;
```
- EN: Implements `UnwindOpcodeAssembler::EmitVFPRegSave`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnwindOpcodeAssembler::EmitVFPRegSave`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 117-119
```cpp
      int Opcode = RangeLSB >= 16
                       ? ARM::EHABI::UNWIND_OPCODE_POP_VFP_REG_RANGE_FSTMFDD_D16
                       : ARM::EHABI::UNWIND_OPCODE_POP_VFP_REG_RANGE_FSTMFDD;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-121
```cpp
      EmitInt16(Opcode | ((RangeLSB % 16) << 4) | (RangeLen - 1));
```
- EN: Declares `EmitInt16`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInt16`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 123-127
```cpp
      // Zero out bits we're done with.
      Regs &= ~(-1u << RangeLSB);
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 129-132
```cpp
/// Emit unwind opcodes to copy address from source register to $sp.
void UnwindOpcodeAssembler::EmitSetSP(uint16_t Reg) {
  EmitInt8(ARM::EHABI::UNWIND_OPCODE_SET_VSP | Reg);
}
```
- EN: Implements `UnwindOpcodeAssembler::EmitSetSP`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnwindOpcodeAssembler::EmitSetSP`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 134-148
```cpp
/// Emit unwind opcodes to add $sp with an offset.
void UnwindOpcodeAssembler::EmitSPOffset(int64_t Offset) {
  if (Offset > 0x200) {
    uint8_t Buff[16];
    Buff[0] = ARM::EHABI::UNWIND_OPCODE_INC_VSP_ULEB128;
    size_t ULEBSize = encodeULEB128((Offset - 0x204) >> 2, Buff + 1);
    emitBytes(Buff, ULEBSize + 1);
  } else if (Offset > 0) {
    if (Offset > 0x100) {
      EmitInt8(ARM::EHABI::UNWIND_OPCODE_INC_VSP | 0x3fu);
      Offset -= 0x100;
    }
    EmitInt8(ARM::EHABI::UNWIND_OPCODE_INC_VSP |
             static_cast<uint8_t>((Offset - 4) >> 2));
  } else if (Offset < 0) {
```
- EN: Implements `UnwindOpcodeAssembler::EmitSPOffset`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnwindOpcodeAssembler::EmitSPOffset`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 149-156
```cpp
    while (Offset < -0x100) {
      EmitInt8(ARM::EHABI::UNWIND_OPCODE_DEC_VSP | 0x3fu);
      Offset += 0x100;
    }
    EmitInt8(ARM::EHABI::UNWIND_OPCODE_DEC_VSP |
             static_cast<uint8_t>(((-Offset) - 4) >> 2));
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 158-160
```cpp
void UnwindOpcodeAssembler::Finalize(unsigned &PersonalityIndex,
                                     SmallVectorImpl<uint8_t> &Result) {
  UnwindOpcodeStreamer OpStreamer(Result);
```
- EN: Implements `UnwindOpcodeAssembler::Finalize`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `UnwindOpcodeAssembler::Finalize`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 162-176
```cpp
  if (HasPersonality) {
    // User-specified personality routine: [ SIZE , OP1 , OP2 , ... ]
    PersonalityIndex = ARM::EHABI::NUM_PERSONALITY_INDEX;
    size_t TotalSize = Ops.size() + 1;
    size_t RoundUpSize = (TotalSize + 3) / 4 * 4;
    Result.resize(RoundUpSize);
    OpStreamer.EmitSize(RoundUpSize);
  } else {
    // If no personalityindex is specified, select ane
    if (PersonalityIndex == ARM::EHABI::NUM_PERSONALITY_INDEX)
      PersonalityIndex = (Ops.size() <= 3) ? ARM::EHABI::AEABI_UNWIND_CPP_PR0
                                           : ARM::EHABI::AEABI_UNWIND_CPP_PR1;
    if (PersonalityIndex == ARM::EHABI::AEABI_UNWIND_CPP_PR0) {
      // __aeabi_unwind_cpp_pr0: [ 0x80 , OP1 , OP2 , OP3 ]
      assert(Ops.size() <= 3 && "too many opcodes for __aeabi_unwind_cpp_pr0");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 177-187
```cpp
      Result.resize(4);
      OpStreamer.EmitPersonalityIndex(PersonalityIndex);
    } else {
      // __aeabi_unwind_cpp_pr{1,2}: [ {0x81,0x82} , SIZE , OP1 , OP2 , ... ]
      size_t TotalSize = Ops.size() + 2;
      size_t RoundUpSize = (TotalSize + 3) / 4 * 4;
      Result.resize(RoundUpSize);
      OpStreamer.EmitPersonalityIndex(PersonalityIndex);
      OpStreamer.EmitSize(RoundUpSize);
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 189-192
```cpp
  // Copy the unwind opcodes
  for (size_t i = OpBegins.size() - 1; i > 0; --i)
    for (size_t j = OpBegins[i - 1], end = OpBegins[i]; j < end; ++j)
      OpStreamer.EmitByte(Ops[j]);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 194-195
```cpp
  // Emit the padding finish opcodes if the size is not multiple of 4.
  OpStreamer.FillFinishOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 197-199
```cpp
  // Reset the assembler state
  Reset();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMUnwindOpAsm.h`.
  - CN: 后端本地头文件：`ARMUnwindOpAsm.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/bit.h`, `llvm/Support/ARMEHABI.h`, `llvm/Support/LEB128.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/bit.h`, `llvm/Support/ARMEHABI.h`, `llvm/Support/LEB128.h`。
- EN: Standard/system headers: `cassert`.
  - CN: 标准库/系统头文件：`cassert`。
