# MipsMCCodeEmitter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MCTargetDesc/MipsMCCodeEmitter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the MipsMCCodeEmitter class.
- 用途 (CN): 实现 Mips 后端中的 `MipsMCCodeEmitter`，重点处理机器码编码输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsMCCodeEmitter.cpp - Convert Mips Code to Machine Code ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the MipsMCCodeEmitter class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MipsMCCodeEmitter.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsFixupKinds.h"
#include "MCTargetDesc/MipsMCAsmInfo.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APInt.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCFixup.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-33
```cpp
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/EndianStream.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 35-35
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 37-37
```cpp
#define DEBUG_TYPE "mccodeemitter"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 39-41
```cpp
#define GET_INSTRMAP_INFO
#include "MipsGenInstrInfo.inc"
#undef GET_INSTRMAP_INFO
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 43-43
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 45-48
```cpp
MCCodeEmitter *createMipsMCCodeEmitterEB(const MCInstrInfo &MCII,
                                         MCContext &Ctx) {
  return new MipsMCCodeEmitter(MCII, Ctx, false);
}
```
- EN: Implements `createMipsMCCodeEmitterEB`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createMipsMCCodeEmitterEB`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-53
```cpp
MCCodeEmitter *createMipsMCCodeEmitterEL(const MCInstrInfo &MCII,
                                         MCContext &Ctx) {
  return new MipsMCCodeEmitter(MCII, Ctx, true);
}
```
- EN: Implements `createMipsMCCodeEmitterEL`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createMipsMCCodeEmitterEL`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-55
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 57-71
```cpp
static void addFixup(SmallVectorImpl<MCFixup> &Fixups, uint32_t Offset,
                     const MCExpr *Value, uint16_t Kind) {
  bool PCRel = false;
  switch (Kind) {
  case Mips::fixup_Mips_PC16:
  case Mips::fixup_Mips_Branch_PCRel:
  case Mips::fixup_MIPS_PC18_S3:
  case Mips::fixup_MIPS_PC19_S2:
  case Mips::fixup_MIPS_PC21_S2:
  case Mips::fixup_MIPS_PC26_S2:
  case Mips::fixup_MIPS_PCHI16:
  case Mips::fixup_MIPS_PCLO16:
  case Mips::fixup_MICROMIPS_PC7_S1:
  case Mips::fixup_MICROMIPS_PC10_S1:
  case Mips::fixup_MICROMIPS_PC16_S1:
```
- EN: Implements `addFixup`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addFixup`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 72-79
```cpp
  case Mips::fixup_MICROMIPS_PC26_S1:
  case Mips::fixup_MICROMIPS_PC19_S2:
  case Mips::fixup_MICROMIPS_PC18_S3:
  case Mips::fixup_MICROMIPS_PC21_S1:
    PCRel = true;
  }
  Fixups.push_back(MCFixup::create(Offset, Value, Kind, PCRel));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 81-85
```cpp
// If the D<shift> instruction has a shift amount that is greater
// than 31 (checked in calling routine), lower it to a D<shift>32 instruction
static void LowerLargeShift(MCInst& Inst) {
  assert(Inst.getNumOperands() == 3 && "Invalid no. of operands for shift!");
  assert(Inst.getOperand(2).isImm());
```
- EN: Implements `LowerLargeShift`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LowerLargeShift`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-90
```cpp
  int64_t Shift = Inst.getOperand(2).getImm();
  if (Shift <= 31)
    return; // Do nothing
  Shift -= 32;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 92-93
```cpp
  // saminus32
  Inst.getOperand(2).setImm(Shift);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-109
```cpp
  switch (Inst.getOpcode()) {
  default:
    // Calling function is not synchronized
    llvm_unreachable("Unexpected shift instruction");
  case Mips::DSLL:
    Inst.setOpcode(Mips::DSLL32);
    return;
  case Mips::DSRL:
    Inst.setOpcode(Mips::DSRL32);
    return;
  case Mips::DSRA:
    Inst.setOpcode(Mips::DSRA32);
    return;
  case Mips::DROTR:
    Inst.setOpcode(Mips::DROTR32);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 110-112
```cpp
    return;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 114-119
```cpp
// Fix a bad compact branch encoding for beqc/bnec.
void MipsMCCodeEmitter::LowerCompactBranch(MCInst& Inst) const {
  // Encoding may be illegal !(rs < rt), but this situation is
  // easily fixed.
  MCRegister RegOp0 = Inst.getOperand(0).getReg();
  MCRegister RegOp1 = Inst.getOperand(1).getReg();
```
- EN: Implements `MipsMCCodeEmitter::LowerCompactBranch`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::LowerCompactBranch`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-122
```cpp
  unsigned Reg0 =  Ctx.getRegisterInfo()->getEncodingValue(RegOp0);
  unsigned Reg1 =  Ctx.getRegisterInfo()->getEncodingValue(RegOp1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 124-137
```cpp
  if (Inst.getOpcode() == Mips::BNEC || Inst.getOpcode() == Mips::BEQC ||
      Inst.getOpcode() == Mips::BNEC64 || Inst.getOpcode() == Mips::BEQC64) {
    assert(Reg0 != Reg1 && "Instruction has bad operands ($rs == $rt)!");
    if (Reg0 < Reg1)
      return;
  } else if (Inst.getOpcode() == Mips::BNVC || Inst.getOpcode() == Mips::BOVC) {
    if (Reg0 >= Reg1)
      return;
  } else if (Inst.getOpcode() == Mips::BNVC_MMR6 ||
             Inst.getOpcode() == Mips::BOVC_MMR6) {
    if (Reg1 >= Reg0)
      return;
  } else
    llvm_unreachable("Cannot rewrite unknown branch!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 139-141
```cpp
  Inst.getOperand(0).setReg(RegOp1);
  Inst.getOperand(1).setReg(RegOp0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-145
```cpp
bool MipsMCCodeEmitter::isMicroMips(const MCSubtargetInfo &STI) const {
  return STI.hasFeature(Mips::FeatureMicroMips);
}
```
- EN: Implements `MipsMCCodeEmitter::isMicroMips`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::isMicroMips`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 147-149
```cpp
bool MipsMCCodeEmitter::isMips32r6(const MCSubtargetInfo &STI) const {
  return STI.hasFeature(Mips::FeatureMips32r6);
}
```
- EN: Implements `MipsMCCodeEmitter::isMips32r6`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::isMips32r6`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 151-153
```cpp
void MipsMCCodeEmitter::EmitByte(unsigned char C, raw_ostream &OS) const {
  OS << (char)C;
}
```
- EN: Implements `MipsMCCodeEmitter::EmitByte`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::EmitByte`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 155-169
```cpp
/// encodeInstruction - Emit the instruction.
/// Size the instruction with Desc.getSize().
void MipsMCCodeEmitter::encodeInstruction(const MCInst &MI,
                                          SmallVectorImpl<char> &CB,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  // Non-pseudo instructions that get changed for direct object
  // only based on operand values.
  // If this list of instructions get much longer we will move
  // the check to a function call. Until then, this is more efficient.
  MCInst TmpInst = MI;
  switch (MI.getOpcode()) {
  // If shift amount is >= 32 it the inst needs to be lowered further
  case Mips::DSLL:
  case Mips::DSRL:
```
- EN: Implements `MipsMCCodeEmitter::encodeInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::encodeInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 170-184
```cpp
  case Mips::DSRA:
  case Mips::DROTR:
    LowerLargeShift(TmpInst);
    break;
  // Compact branches, enforce encoding restrictions.
  case Mips::BEQC:
  case Mips::BNEC:
  case Mips::BEQC64:
  case Mips::BNEC64:
  case Mips::BOVC:
  case Mips::BOVC_MMR6:
  case Mips::BNVC:
  case Mips::BNVC_MMR6:
    LowerCompactBranch(TmpInst);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-187
```cpp
  size_t N = Fixups.size();
  uint32_t Binary = getBinaryCodeForInstr(TmpInst, Fixups, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 189-195
```cpp
  // Check for unimplemented opcodes.
  // Unfortunately in MIPS both NOP and SLL will come in with Binary == 0
  // so we have to special check for them.
  const unsigned Opcode = TmpInst.getOpcode();
  if ((Opcode != Mips::NOP) && (Opcode != Mips::SLL) &&
      (Opcode != Mips::SLL_MM) && (Opcode != Mips::SLL_MMR6) && !Binary)
    llvm_unreachable("unimplemented opcode in encodeInstruction()");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 197-205
```cpp
  int NewOpcode = -1;
  if (isMicroMips(STI)) {
    if (isMips32r6(STI)) {
      NewOpcode = Mips::MipsR62MicroMipsR6(Opcode, Mips::Arch_micromipsr6);
      if (NewOpcode == -1)
        NewOpcode = Mips::Std2MicroMipsR6(Opcode, Mips::Arch_micromipsr6);
    }
    else
      NewOpcode = Mips::Std2MicroMips(Opcode, Mips::Arch_micromips);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 207-209
```cpp
    // Check whether it is Dsp instruction.
    if (NewOpcode == -1)
      NewOpcode = Mips::Dsp2MicroMips(Opcode, Mips::Arch_mmdsp);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 211-213
```cpp
    if (NewOpcode != -1) {
      if (Fixups.size() > N)
        Fixups.pop_back();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 215-217
```cpp
      TmpInst.setOpcode (NewOpcode);
      Binary = getBinaryCodeForInstr(TmpInst, Fixups, STI);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-224
```cpp
    if (((MI.getOpcode() == Mips::MOVEP_MM) ||
         (MI.getOpcode() == Mips::MOVEP_MMR6))) {
      unsigned RegPair = getMovePRegPairOpValue(MI, 0, Fixups, STI);
      Binary = (Binary & 0xFFFFFC7F) | (RegPair << 7);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 226-226
```cpp
  const MCInstrDesc &Desc = MCII.get(TmpInst.getOpcode());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 228-231
```cpp
  // Get byte count of instruction
  unsigned Size = Desc.getSize();
  if (!Size)
    llvm_unreachable("Desc.getSize() returns 0");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 233-243
```cpp
  auto Endian =
      IsLittleEndian ? llvm::endianness::little : llvm::endianness::big;
  if (Size == 2) {
    support::endian::write<uint16_t>(CB, Binary, Endian);
  } else if (IsLittleEndian && isMicroMips(STI)) {
    support::endian::write<uint16_t>(CB, Binary >> 16, Endian);
    support::endian::write<uint16_t>(CB, Binary & 0xffff, Endian);
  } else {
    support::endian::write<uint32_t>(CB, Binary, Endian);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 245-252
```cpp
/// getBranchTargetOpValue - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTargetOpValue(const MCInst &MI, unsigned OpNo,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 254-255
```cpp
  // If the destination is an immediate, divide by 4.
  if (MO.isImm()) return MO.getImm() >> 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 257-258
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValue expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 260-264
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_Mips_PC16);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 266-273
```cpp
/// getBranchTargetOpValue1SImm16 - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTargetOpValue1SImm16(const MCInst &MI, unsigned OpNo,
                              SmallVectorImpl<MCFixup> &Fixups,
                              const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTargetOpValue1SImm16`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValue1SImm16`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 275-276
```cpp
  // If the destination is an immediate, divide by 2.
  if (MO.isImm()) return MO.getImm() >> 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 278-279
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValue expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 281-285
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_Mips_PC16);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 287-294
```cpp
/// getBranchTargetOpValueMMR6 - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTargetOpValueMMR6(const MCInst &MI, unsigned OpNo,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTargetOpValueMMR6`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValueMMR6`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 296-298
```cpp
  // If the destination is an immediate, divide by 2.
  if (MO.isImm())
    return MO.getImm() >> 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 300-301
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValueMMR6 expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 303-307
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-2, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_Mips_PC16);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 309-316
```cpp
/// getBranchTargetOpValueLsl2MMR6 - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTargetOpValueLsl2MMR6(const MCInst &MI, unsigned OpNo,
                               SmallVectorImpl<MCFixup> &Fixups,
                               const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTargetOpValueLsl2MMR6`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValueLsl2MMR6`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 318-320
```cpp
  // If the destination is an immediate, divide by 4.
  if (MO.isImm())
    return MO.getImm() >> 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 322-323
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValueLsl2MMR6 expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 325-329
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_Mips_PC16);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 331-338
```cpp
/// getBranchTarget7OpValueMM - Return binary encoding of the microMIPS branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTarget7OpValueMM(const MCInst &MI, unsigned OpNo,
                          SmallVectorImpl<MCFixup> &Fixups,
                          const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTarget7OpValueMM`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTarget7OpValueMM`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 340-341
```cpp
  // If the destination is an immediate, divide by 2.
  if (MO.isImm()) return MO.getImm() >> 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 343-344
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValueMM expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 346-349
```cpp
  const MCExpr *Expr = MO.getExpr();
  addFixup(Fixups, 0, Expr, Mips::fixup_MICROMIPS_PC7_S1);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 351-358
```cpp
/// getBranchTargetOpValueMMPC10 - Return binary encoding of the microMIPS
/// 10-bit branch target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTargetOpValueMMPC10(const MCInst &MI, unsigned OpNo,
                             SmallVectorImpl<MCFixup> &Fixups,
                             const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTargetOpValueMMPC10`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValueMMPC10`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 360-361
```cpp
  // If the destination is an immediate, divide by 2.
  if (MO.isImm()) return MO.getImm() >> 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 363-364
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValuePC10 expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 366-369
```cpp
  const MCExpr *Expr = MO.getExpr();
  addFixup(Fixups, 0, Expr, Mips::fixup_MICROMIPS_PC10_S1);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 371-378
```cpp
/// getBranchTargetOpValue - Return binary encoding of the microMIPS branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTargetOpValueMM(const MCInst &MI, unsigned OpNo,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTargetOpValueMM`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTargetOpValueMM`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 380-381
```cpp
  // If the destination is an immediate, divide by 2.
  if (MO.isImm()) return MO.getImm() >> 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 383-384
```cpp
  assert(MO.isExpr() &&
         "getBranchTargetOpValueMM expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 386-389
```cpp
  const MCExpr *Expr = MO.getExpr();
  addFixup(Fixups, 0, Expr, Mips::fixup_MICROMIPS_PC16_S1);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 391-398
```cpp
/// getBranchTarget21OpValue - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTarget21OpValue(const MCInst &MI, unsigned OpNo,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTarget21OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTarget21OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 400-401
```cpp
  // If the destination is an immediate, divide by 4.
  if (MO.isImm()) return MO.getImm() >> 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 403-404
```cpp
  assert(MO.isExpr() &&
         "getBranchTarget21OpValue expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-410
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_MIPS_PC21_S2);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 412-419
```cpp
/// getBranchTarget21OpValueMM - Return binary encoding of the branch
/// target operand for microMIPS. If the machine operand requires
/// relocation, record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTarget21OpValueMM(const MCInst &MI, unsigned OpNo,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTarget21OpValueMM`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTarget21OpValueMM`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 421-422
```cpp
  // If the destination is an immediate, divide by 4.
  if (MO.isImm()) return MO.getImm() >> 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 424-425
```cpp
  assert(MO.isExpr() &&
    "getBranchTarget21OpValueMM expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 427-431
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_MICROMIPS_PC21_S1);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 433-440
```cpp
/// getBranchTarget26OpValue - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getBranchTarget26OpValue(const MCInst &MI, unsigned OpNo,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getBranchTarget26OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getBranchTarget26OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 442-443
```cpp
  // If the destination is an immediate, divide by 4.
  if (MO.isImm()) return MO.getImm() >> 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 445-446
```cpp
  assert(MO.isExpr() &&
         "getBranchTarget26OpValue expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 448-452
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_MIPS_PC26_S2);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 454-460
```cpp
/// getBranchTarget26OpValueMM - Return binary encoding of the branch
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::getBranchTarget26OpValueMM(
    const MCInst &MI, unsigned OpNo, SmallVectorImpl<MCFixup> &Fixups,
    const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `MipsMCCodeEmitter::getBranchTarget26OpValueMM`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getBranchTarget26OpValueMM`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 462-464
```cpp
  // If the destination is an immediate, divide by 2.
  if (MO.isImm())
    return MO.getImm() >> 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 466-467
```cpp
  assert(MO.isExpr() &&
         "getBranchTarget26OpValueMM expects only expressions or immediates");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 469-473
```cpp
  const MCExpr *FixupExpression = MCBinaryExpr::createAdd(
      MO.getExpr(), MCConstantExpr::create(-4, Ctx), Ctx);
  addFixup(Fixups, 0, FixupExpression, Mips::fixup_MICROMIPS_PC26_S1);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 475-482
```cpp
/// getJumpOffset16OpValue - Return binary encoding of the jump
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getJumpOffset16OpValue(const MCInst &MI, unsigned OpNo,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
```
- EN: Implements `getJumpOffset16OpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getJumpOffset16OpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 484-484
```cpp
  if (MO.isImm()) return MO.getImm();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 486-487
```cpp
  assert(MO.isExpr() &&
         "getJumpOffset16OpValue expects only expressions or an immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 489-494
```cpp
  const MCExpr *Expr = MO.getExpr();
  Mips::Fixups FixupKind =
      isMicroMips(STI) ? Mips::fixup_MICROMIPS_LO16 : Mips::fixup_Mips_LO16;
  addFixup(Fixups, 0, Expr, MCFixupKind(FixupKind));
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 496-505
```cpp
/// getJumpTargetOpValue - Return binary encoding of the jump
/// target operand. If the machine operand requires relocation,
/// record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getJumpTargetOpValue(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  // If the destination is an immediate, divide by 4.
  if (MO.isImm()) return MO.getImm()>>2;
```
- EN: Implements `getJumpTargetOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getJumpTargetOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 507-508
```cpp
  assert(MO.isExpr() &&
         "getJumpTargetOpValue expects only expressions or an immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 510-513
```cpp
  const MCExpr *Expr = MO.getExpr();
  addFixup(Fixups, 0, Expr, Mips::fixup_Mips_26);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 515-521
```cpp
unsigned MipsMCCodeEmitter::
getJumpTargetOpValueMM(const MCInst &MI, unsigned OpNo,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  // If the destination is an immediate, divide by 2.
  if (MO.isImm()) return MO.getImm() >> 1;
```
- EN: Implements `getJumpTargetOpValueMM`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getJumpTargetOpValueMM`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 523-524
```cpp
  assert(MO.isExpr() &&
         "getJumpTargetOpValueMM expects only expressions or an immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 526-529
```cpp
  const MCExpr *Expr = MO.getExpr();
  addFixup(Fixups, 0, Expr, Mips::fixup_MICROMIPS_26_S1);
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 531-541
```cpp
unsigned MipsMCCodeEmitter::
getUImm5Lsl2Encoding(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    // The immediate is encoded as 'immediate << 2'.
    unsigned Res = getMachineOpValue(MI, MO, Fixups, STI);
    assert((Res & 3) == 0);
    return Res >> 2;
  }
```
- EN: Implements `getUImm5Lsl2Encoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getUImm5Lsl2Encoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 543-544
```cpp
  assert(MO.isExpr() &&
         "getUImm5Lsl2Encoding expects only expressions or an immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 546-547
```cpp
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 549-557
```cpp
unsigned MipsMCCodeEmitter::
getSImm3Lsa2Value(const MCInst &MI, unsigned OpNo,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    int Value = MO.getImm();
    return Value >> 2;
  }
```
- EN: Implements `getSImm3Lsa2Value`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSImm3Lsa2Value`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 559-560
```cpp
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 562-570
```cpp
unsigned MipsMCCodeEmitter::
getUImm6Lsl2Encoding(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    unsigned Value = MO.getImm();
    return Value >> 2;
  }
```
- EN: Implements `getUImm6Lsl2Encoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getUImm6Lsl2Encoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 572-573
```cpp
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 575-583
```cpp
unsigned MipsMCCodeEmitter::
getSImm9AddiuspValue(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    unsigned Binary = (MO.getImm() >> 2) & 0x0000ffff;
    return (((Binary & 0x8000) >> 7) | (Binary & 0x00ff));
  }
```
- EN: Implements `getSImm9AddiuspValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSImm9AddiuspValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 585-586
```cpp
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 588-593
```cpp
unsigned MipsMCCodeEmitter::
getExprOpValue(const MCExpr *Expr, SmallVectorImpl<MCFixup> &Fixups,
               const MCSubtargetInfo &STI) const {
  MCExpr::ExprKind Kind = Expr->getKind();
  if (Kind == MCExpr::Specifier) {
    const auto *MipsExpr = cast<MCSpecifierExpr>(Expr);
```
- EN: Implements `getExprOpValue`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getExprOpValue`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 595-609
```cpp
    Mips::Fixups FixupKind = Mips::Fixups(0);
    switch (MipsExpr->getSpecifier()) {
    case Mips::S_None:
    case Mips::S_Special:
      llvm_unreachable("Unhandled fixup kind!");
      break;
    case Mips::S_DTPREL:
      // MEK_DTPREL is used for marking TLS DIEExpr only
      // and contains a regular sub-expression.
      return getExprOpValue(MipsExpr->getSubExpr(), Fixups, STI);
    case Mips::S_CALL_HI16:
      FixupKind = Mips::fixup_Mips_CALL_HI16;
      break;
    case Mips::S_CALL_LO16:
      FixupKind = Mips::fixup_Mips_CALL_LO16;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 610-624
```cpp
      break;
    case Mips::S_DTPREL_HI:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_TLS_DTPREL_HI16
                                   : Mips::fixup_Mips_DTPREL_HI;
      break;
    case Mips::S_DTPREL_LO:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_TLS_DTPREL_LO16
                                   : Mips::fixup_Mips_DTPREL_LO;
      break;
    case Mips::S_GOTTPREL:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GOTTPREL
                                   : Mips::fixup_Mips_GOTTPREL;
      break;
    case Mips::S_GOT:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GOT16
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 625-639
```cpp
                                   : Mips::fixup_Mips_GOT;
      break;
    case Mips::S_GOT_CALL:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_CALL16
                                   : Mips::fixup_Mips_CALL16;
      break;
    case Mips::S_GOT_DISP:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GOT_DISP
                                   : Mips::fixup_Mips_GOT_DISP;
      break;
    case Mips::S_GOT_HI16:
      FixupKind = Mips::fixup_Mips_GOT_HI16;
      break;
    case Mips::S_GOT_LO16:
      FixupKind = Mips::fixup_Mips_GOT_LO16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 640-654
```cpp
      break;
    case Mips::S_GOT_PAGE:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GOT_PAGE
                                   : Mips::fixup_Mips_GOT_PAGE;
      break;
    case Mips::S_GOT_OFST:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GOT_OFST
                                   : Mips::fixup_Mips_GOT_OFST;
      break;
    case Mips::S_GPREL:
      FixupKind = Mips::fixup_Mips_GPREL16;
      break;
    case Mips::S_LO:
      // Check for %lo(%neg(%gp_rel(X)))
      if (Mips::isGpOff(*MipsExpr))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 655-669
```cpp
        FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GPOFF_LO
                                     : Mips::fixup_Mips_GPOFF_LO;
      else
        FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_LO16
                                     : Mips::fixup_Mips_LO16;
      break;
    case Mips::S_HIGHEST:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_HIGHEST
                                   : Mips::fixup_Mips_HIGHEST;
      break;
    case Mips::S_HIGHER:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_HIGHER
                                   : Mips::fixup_Mips_HIGHER;
      break;
    case Mips::S_HI:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 670-684
```cpp
      // Check for %hi(%neg(%gp_rel(X)))
      if (Mips::isGpOff(*MipsExpr))
        FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_GPOFF_HI
                                     : Mips::fixup_Mips_GPOFF_HI;
      else
        FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_HI16
                                     : Mips::fixup_Mips_HI16;
      break;
    case Mips::S_PCREL_HI16:
      FixupKind = Mips::fixup_MIPS_PCHI16;
      break;
    case Mips::S_PCREL_LO16:
      FixupKind = Mips::fixup_MIPS_PCLO16;
      break;
    case Mips::S_TLSGD:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 685-699
```cpp
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_TLS_GD
                                   : Mips::fixup_Mips_TLSGD;
      break;
    case Mips::S_TLSLDM:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_TLS_LDM
                                   : Mips::fixup_Mips_TLSLDM;
      break;
    case Mips::S_TPREL_HI:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_TLS_TPREL_HI16
                                   : Mips::fixup_Mips_TPREL_HI;
      break;
    case Mips::S_TPREL_LO:
      FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_TLS_TPREL_LO16
                                   : Mips::fixup_Mips_TPREL_LO;
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 700-707
```cpp
    case Mips::S_NEG:
      FixupKind =
          isMicroMips(STI) ? Mips::fixup_MICROMIPS_SUB : Mips::fixup_Mips_SUB;
      break;
    }
    addFixup(Fixups, 0, MipsExpr, MCFixupKind(FixupKind));
    return 0;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 709-711
```cpp
  Ctx.reportError(Expr->getLoc(), "expected an immediate");
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 713-727
```cpp
/// getMachineOpValue - Return binary encoding of operand. If the machine
/// operand requires relocation, record the relocation and return zero.
unsigned MipsMCCodeEmitter::
getMachineOpValue(const MCInst &MI, const MCOperand &MO,
                  SmallVectorImpl<MCFixup> &Fixups,
                  const MCSubtargetInfo &STI) const {
  if (MO.isReg()) {
    MCRegister Reg = MO.getReg();
    unsigned RegNo = Ctx.getRegisterInfo()->getEncodingValue(Reg);
    return RegNo;
  } else if (MO.isImm()) {
    return static_cast<unsigned>(MO.getImm());
  } else if (MO.isDFPImm()) {
    return static_cast<unsigned>(bit_cast<double>(MO.getDFPImm()));
  }
```
- EN: Implements `getMachineOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMachineOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 728-732
```cpp
  // TODO: Set EncoderMethod to "getImmOpValue" for imm Operand so that
  // getMachineOpValue will not be called for isExpr code paths.
  assert(MO.isExpr());
  return getImmOpValue(MI, MO, Fixups, STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 734-748
```cpp
unsigned MipsMCCodeEmitter::getImmOpValue(const MCInst &MI, const MCOperand &MO,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  if (MO.isImm())
    return MO.getImm();
  assert(MO.isExpr() && "getImmOpValue expects only expressions or immediates");
  const MCExpr *Expr = MO.getExpr();
  int64_t Res;
  if (Expr->evaluateAsAbsolute(Res))
    return Res;
  unsigned MIFrm = MipsII::getFormat(MCII.get(MI.getOpcode()).TSFlags);
  if (!isa<MCSpecifierExpr>(Expr) && MIFrm == MipsII::FrmI) {
    addFixup(Fixups, 0, Expr, Mips::fixup_Mips_AnyImm16);
    return 0;
  }
```
- EN: Implements `MipsMCCodeEmitter::getImmOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getImmOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 749-750
```cpp
  return getExprOpValue(Expr, Fixups, STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 752-762
```cpp
/// Return binary encoding of memory related operand.
/// If the offset operand requires relocation, record the relocation.
template <unsigned ShiftAmount>
unsigned MipsMCCodeEmitter::getMemEncoding(const MCInst &MI, unsigned OpNo,
                                           SmallVectorImpl<MCFixup> &Fixups,
                                           const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 20-16, offset is encoded in bits 15-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups, STI)
                     << 16;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1), Fixups, STI);
```
- EN: Implements `MipsMCCodeEmitter::getMemEncoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getMemEncoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 764-765
```cpp
  // Apply the scale factor if there is one.
  OffBits >>= ShiftAmount;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 767-768
```cpp
  return (OffBits & 0xFFFF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 770-779
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm4(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 6-4, offset is encoded in bits 3-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo),
                                       Fixups, STI) << 4;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1),
                                       Fixups, STI);
```
- EN: Implements `getMemEncodingMMImm4`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm4`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 781-782
```cpp
  return (OffBits & 0xF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 784-793
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm4Lsl1(const MCInst &MI, unsigned OpNo,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 6-4, offset is encoded in bits 3-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo),
                                       Fixups, STI) << 4;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1),
                                       Fixups, STI) >> 1;
```
- EN: Implements `getMemEncodingMMImm4Lsl1`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm4Lsl1`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 795-796
```cpp
  return (OffBits & 0xF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 798-807
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm4Lsl2(const MCInst &MI, unsigned OpNo,
                         SmallVectorImpl<MCFixup> &Fixups,
                         const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 6-4, offset is encoded in bits 3-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo),
                                       Fixups, STI) << 4;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1),
                                       Fixups, STI) >> 2;
```
- EN: Implements `getMemEncodingMMImm4Lsl2`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm4Lsl2`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 809-810
```cpp
  return (OffBits & 0xF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 812-822
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMSPImm5Lsl2(const MCInst &MI, unsigned OpNo,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const {
  // Register is encoded in bits 9-5, offset is encoded in bits 4-0.
  assert(MI.getOperand(OpNo).isReg() &&
         (MI.getOperand(OpNo).getReg() == Mips::SP ||
         MI.getOperand(OpNo).getReg() == Mips::SP_64) &&
         "Unexpected base register!");
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1),
                                       Fixups, STI) >> 2;
```
- EN: Implements `getMemEncodingMMSPImm5Lsl2`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMSPImm5Lsl2`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 824-825
```cpp
  return OffBits & 0x1F;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 827-834
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMGPImm7Lsl2(const MCInst &MI, unsigned OpNo,
                           SmallVectorImpl<MCFixup> &Fixups,
                           const MCSubtargetInfo &STI) const {
  // Register is encoded in bits 9-7, offset is encoded in bits 6-0.
  assert(MI.getOperand(OpNo).isReg() &&
         MI.getOperand(OpNo).getReg() == Mips::GP &&
         "Unexpected base register!");
```
- EN: Implements `getMemEncodingMMGPImm7Lsl2`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMGPImm7Lsl2`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 836-837
```cpp
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1),
                                       Fixups, STI) >> 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 839-840
```cpp
  return OffBits & 0x7F;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 842-851
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm9(const MCInst &MI, unsigned OpNo,
                     SmallVectorImpl<MCFixup> &Fixups,
                     const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 20-16, offset is encoded in bits 8-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups,
                                       STI) << 16;
  unsigned OffBits =
      getMachineOpValue(MI, MI.getOperand(OpNo + 1), Fixups, STI);
```
- EN: Implements `getMemEncodingMMImm9`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm9`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 853-854
```cpp
  return (OffBits & 0x1FF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 856-864
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm11(const MCInst &MI, unsigned OpNo,
                      SmallVectorImpl<MCFixup> &Fixups,
                      const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 20-16, offset is encoded in bits 10-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups,
                                       STI) << 16;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1), Fixups, STI);
```
- EN: Implements `getMemEncodingMMImm11`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm11`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 866-867
```cpp
  return (OffBits & 0x07FF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 869-882
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm12(const MCInst &MI, unsigned OpNo,
                      SmallVectorImpl<MCFixup> &Fixups,
                      const MCSubtargetInfo &STI) const {
  // opNum can be invalid if instruction had reglist as operand.
  // MemOperand is always last operand of instruction (base + offset).
  switch (MI.getOpcode()) {
  default:
    break;
  case Mips::SWM32_MM:
  case Mips::LWM32_MM:
    OpNo = MI.getNumOperands() - 2;
    break;
  }
```
- EN: Implements `getMemEncodingMMImm12`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm12`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 884-888
```cpp
  // Base register is encoded in bits 20-16, offset is encoded in bits 11-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups, STI)
                     << 16;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1), Fixups, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 890-891
```cpp
  return (OffBits & 0x0FFF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 893-901
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm16(const MCInst &MI, unsigned OpNo,
                      SmallVectorImpl<MCFixup> &Fixups,
                      const MCSubtargetInfo &STI) const {
  // Base register is encoded in bits 20-16, offset is encoded in bits 15-0.
  assert(MI.getOperand(OpNo).isReg());
  unsigned RegBits = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups,
                                       STI) << 16;
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1), Fixups, STI);
```
- EN: Implements `getMemEncodingMMImm16`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm16`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 903-904
```cpp
  return (OffBits & 0xFFFF) | RegBits;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 906-920
```cpp
unsigned MipsMCCodeEmitter::
getMemEncodingMMImm4sp(const MCInst &MI, unsigned OpNo,
                       SmallVectorImpl<MCFixup> &Fixups,
                       const MCSubtargetInfo &STI) const {
  // opNum can be invalid if instruction had reglist as operand
  // MemOperand is always last operand of instruction (base + offset)
  switch (MI.getOpcode()) {
  default:
    break;
  case Mips::SWM16_MM:
  case Mips::SWM16_MMR6:
  case Mips::LWM16_MM:
  case Mips::LWM16_MMR6:
    OpNo = MI.getNumOperands() - 2;
    break;
```
- EN: Implements `getMemEncodingMMImm4sp`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMemEncodingMMImm4sp`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 921-921
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 923-927
```cpp
  // Offset is encoded in bits 4-0.
  assert(MI.getOperand(OpNo).isReg());
  // Base register is always SP - thus it is not encoded.
  assert(MI.getOperand(OpNo+1).isImm());
  unsigned OffBits = getMachineOpValue(MI, MI.getOperand(OpNo+1), Fixups, STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 929-930
```cpp
  return ((OffBits >> 2) & 0x0F);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 932-941
```cpp
// FIXME: should be called getMSBEncoding
//
unsigned
MipsMCCodeEmitter::getSizeInsEncoding(const MCInst &MI, unsigned OpNo,
                                      SmallVectorImpl<MCFixup> &Fixups,
                                      const MCSubtargetInfo &STI) const {
  assert(MI.getOperand(OpNo-1).isImm());
  assert(MI.getOperand(OpNo).isImm());
  unsigned Position = getMachineOpValue(MI, MI.getOperand(OpNo-1), Fixups, STI);
  unsigned Size = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups, STI);
```
- EN: Implements `MipsMCCodeEmitter::getSizeInsEncoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getSizeInsEncoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 943-944
```cpp
  return Position + Size - 1;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 946-955
```cpp
template <unsigned Bits, int Offset>
unsigned
MipsMCCodeEmitter::getUImmWithOffsetEncoding(const MCInst &MI, unsigned OpNo,
                                             SmallVectorImpl<MCFixup> &Fixups,
                                             const MCSubtargetInfo &STI) const {
  assert(MI.getOperand(OpNo).isImm());
  unsigned Value = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups, STI);
  Value -= Offset;
  return Value;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 957-967
```cpp
unsigned
MipsMCCodeEmitter::getSimm19Lsl2Encoding(const MCInst &MI, unsigned OpNo,
                                         SmallVectorImpl<MCFixup> &Fixups,
                                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    // The immediate is encoded as 'immediate << 2'.
    unsigned Res = getMachineOpValue(MI, MO, Fixups, STI);
    assert((Res & 3) == 0);
    return Res >> 2;
  }
```
- EN: Implements `MipsMCCodeEmitter::getSimm19Lsl2Encoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getSimm19Lsl2Encoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 969-970
```cpp
  assert(MO.isExpr() &&
         "getSimm19Lsl2Encoding expects only expressions or an immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 972-977
```cpp
  const MCExpr *Expr = MO.getExpr();
  Mips::Fixups FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_PC19_S2
                                            : Mips::fixup_MIPS_PC19_S2;
  addFixup(Fixups, 0, Expr, MCFixupKind(FixupKind));
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 979-989
```cpp
unsigned
MipsMCCodeEmitter::getSimm18Lsl3Encoding(const MCInst &MI, unsigned OpNo,
                                         SmallVectorImpl<MCFixup> &Fixups,
                                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  if (MO.isImm()) {
    // The immediate is encoded as 'immediate << 3'.
    unsigned Res = getMachineOpValue(MI, MI.getOperand(OpNo), Fixups, STI);
    assert((Res & 7) == 0);
    return Res >> 3;
  }
```
- EN: Implements `MipsMCCodeEmitter::getSimm18Lsl3Encoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getSimm18Lsl3Encoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 991-992
```cpp
  assert(MO.isExpr() &&
         "getSimm18Lsl2Encoding expects only expressions or an immediate");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 994-999
```cpp
  const MCExpr *Expr = MO.getExpr();
  Mips::Fixups FixupKind = isMicroMips(STI) ? Mips::fixup_MICROMIPS_PC18_S3
                                            : Mips::fixup_MIPS_PC18_S3;
  addFixup(Fixups, 0, Expr, MCFixupKind(FixupKind));
  return 0;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1001-1008
```cpp
unsigned
MipsMCCodeEmitter::getUImm3Mod8Encoding(const MCInst &MI, unsigned OpNo,
                                        SmallVectorImpl<MCFixup> &Fixups,
                                        const MCSubtargetInfo &STI) const {
  assert(MI.getOperand(OpNo).isImm());
  const MCOperand &MO = MI.getOperand(OpNo);
  return MO.getImm() % 8;
}
```
- EN: Implements `MipsMCCodeEmitter::getUImm3Mod8Encoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getUImm3Mod8Encoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1010-1024
```cpp
unsigned
MipsMCCodeEmitter::getUImm4AndValue(const MCInst &MI, unsigned OpNo,
                                    SmallVectorImpl<MCFixup> &Fixups,
                                    const MCSubtargetInfo &STI) const {
  assert(MI.getOperand(OpNo).isImm());
  const MCOperand &MO = MI.getOperand(OpNo);
  unsigned Value = MO.getImm();
  switch (Value) {
    case 128:   return 0x0;
    case 1:     return 0x1;
    case 2:     return 0x2;
    case 3:     return 0x3;
    case 4:     return 0x4;
    case 7:     return 0x5;
    case 8:     return 0x6;
```
- EN: Implements `MipsMCCodeEmitter::getUImm4AndValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getUImm4AndValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1025-1036
```cpp
    case 15:    return 0x7;
    case 16:    return 0x8;
    case 31:    return 0x9;
    case 32:    return 0xa;
    case 63:    return 0xb;
    case 64:    return 0xc;
    case 255:   return 0xd;
    case 32768: return 0xe;
    case 65535: return 0xf;
  }
  llvm_unreachable("Unexpected value");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1038-1042
```cpp
unsigned
MipsMCCodeEmitter::getRegisterListOpValue(const MCInst &MI, unsigned OpNo,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  unsigned res = 0;
```
- EN: Implements `MipsMCCodeEmitter::getRegisterListOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getRegisterListOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1044-1045
```cpp
  // Register list operand is always first operand of instruction and it is
  // placed before memory operand (register + imm).
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1047-1056
```cpp
  for (unsigned I = OpNo, E = MI.getNumOperands() - 2; I < E; ++I) {
    MCRegister Reg = MI.getOperand(I).getReg();
    unsigned RegNo = Ctx.getRegisterInfo()->getEncodingValue(Reg);
    if (RegNo != 31)
      res++;
    else
      res |= 0x10;
  }
  return res;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1058-1063
```cpp
unsigned
MipsMCCodeEmitter::getRegisterListOpValue16(const MCInst &MI, unsigned OpNo,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  return (MI.getNumOperands() - 4);
}
```
- EN: Implements `MipsMCCodeEmitter::getRegisterListOpValue16`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getRegisterListOpValue16`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1065-1069
```cpp
unsigned
MipsMCCodeEmitter::getMovePRegPairOpValue(const MCInst &MI, unsigned OpNo,
                                          SmallVectorImpl<MCFixup> &Fixups,
                                          const MCSubtargetInfo &STI) const {
  unsigned res = 0;
```
- EN: Implements `MipsMCCodeEmitter::getMovePRegPairOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getMovePRegPairOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1071-1085
```cpp
  if (MI.getOperand(0).getReg() == Mips::A1 &&
      MI.getOperand(1).getReg() == Mips::A2)
    res = 0;
  else if (MI.getOperand(0).getReg() == Mips::A1 &&
           MI.getOperand(1).getReg() == Mips::A3)
    res = 1;
  else if (MI.getOperand(0).getReg() == Mips::A2 &&
           MI.getOperand(1).getReg() == Mips::A3)
    res = 2;
  else if (MI.getOperand(0).getReg() == Mips::A0 &&
           MI.getOperand(1).getReg() == Mips::S5)
    res = 3;
  else if (MI.getOperand(0).getReg() == Mips::A0 &&
           MI.getOperand(1).getReg() == Mips::S6)
    res = 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1086-1094
```cpp
  else if (MI.getOperand(0).getReg() == Mips::A0 &&
           MI.getOperand(1).getReg() == Mips::A1)
    res = 5;
  else if (MI.getOperand(0).getReg() == Mips::A0 &&
           MI.getOperand(1).getReg() == Mips::A2)
    res = 6;
  else if (MI.getOperand(0).getReg() == Mips::A0 &&
           MI.getOperand(1).getReg() == Mips::A3)
    res = 7;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1096-1097
```cpp
  return res;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1099-1104
```cpp
unsigned
MipsMCCodeEmitter::getMovePRegSingleOpValue(const MCInst &MI, unsigned OpNo,
                                            SmallVectorImpl<MCFixup> &Fixups,
                                            const MCSubtargetInfo &STI) const {
  assert(((OpNo == 2) || (OpNo == 3)) &&
         "Unexpected OpNo for movep operand encoding!");
```
- EN: Implements `MipsMCCodeEmitter::getMovePRegSingleOpValue`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getMovePRegSingleOpValue`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1106-1120
```cpp
  MCOperand Op = MI.getOperand(OpNo);
  assert(Op.isReg() && "Operand of movep is not a register!");
  switch (Op.getReg().id()) {
  default:
    llvm_unreachable("Unknown register for movep!");
  case Mips::ZERO:  return 0;
  case Mips::S1:    return 1;
  case Mips::V0:    return 2;
  case Mips::V1:    return 3;
  case Mips::S0:    return 4;
  case Mips::S2:    return 5;
  case Mips::S3:    return 6;
  case Mips::S4:    return 7;
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1122-1132
```cpp
unsigned
MipsMCCodeEmitter::getSimm23Lsl2Encoding(const MCInst &MI, unsigned OpNo,
                                         SmallVectorImpl<MCFixup> &Fixups,
                                         const MCSubtargetInfo &STI) const {
  const MCOperand &MO = MI.getOperand(OpNo);
  assert(MO.isImm() && "getSimm23Lsl2Encoding expects only an immediate");
  // The immediate is encoded as 'immediate >> 2'.
  unsigned Res = static_cast<unsigned>(MO.getImm());
  assert((Res & 3) == 0);
  return Res >> 2;
}
```
- EN: Implements `MipsMCCodeEmitter::getSimm23Lsl2Encoding`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCCodeEmitter::getSimm23Lsl2Encoding`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1134-1134
```cpp
#include "MipsGenMCCodeEmitter.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

## Key Concepts / 关键概念

- EN: Primary role: machine-code encoding emission.
  - CN: 核心职责：机器码编码输出。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsMCCodeEmitter.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsFixupKinds.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MipsGenInstrInfo.inc`, `MipsGenMCCodeEmitter.inc`.
  - CN: 后端本地头文件：`MipsMCCodeEmitter.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsFixupKinds.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MipsGenInstrInfo.inc`, `MipsGenMCCodeEmitter.inc`。
- EN: LLVM infrastructure headers: `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrDesc.h` ... (+6 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/SmallVector.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCFixup.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrDesc.h` ... (+6 more)。
- EN: Standard/system headers: `cassert`, `cstdint`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
