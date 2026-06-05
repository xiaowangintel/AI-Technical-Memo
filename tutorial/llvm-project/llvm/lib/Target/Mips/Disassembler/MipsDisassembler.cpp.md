# MipsDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Disassembler/MipsDisassembler.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file is part of the Mips Disassembler.
- 用途 (CN): 实现 Mips 后端中的 `MipsDisassembler`，重点处理指令解码与反汇编。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===- MipsDisassembler.cpp - Disassembler for Mips -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file is part of the Mips Disassembler.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "TargetInfo/MipsTargetInfo.h"
#include "llvm/ADT/ArrayRef.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDecoder.h"
#include "llvm/MC/MCDecoderOps.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-30
```cpp
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 32-33
```cpp
using namespace llvm;
using namespace llvm::MCD;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 35-35
```cpp
#define DEBUG_TYPE "mips-disassembler"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 37-37
```cpp
using DecodeStatus = MCDisassembler::DecodeStatus;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 39-39
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 41-43
```cpp
class MipsDisassembler : public MCDisassembler {
  bool IsMicroMips;
  bool IsBigEndian;
```
- EN: Declares `MipsDisassembler`, packaging target-specific state and APIs around `MipsDisassembler`.
- CN: 这里声明 `MipsDisassembler`，把与 `MipsDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 45-49
```cpp
public:
  MipsDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx, bool IsBigEndian)
      : MCDisassembler(STI, Ctx),
        IsMicroMips(STI.hasFeature(Mips::FeatureMicroMips)),
        IsBigEndian(IsBigEndian) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 51-53
```cpp
  bool hasMips2() const { return STI.hasFeature(Mips::FeatureMips2); }
  bool hasMips3() const { return STI.hasFeature(Mips::FeatureMips3); }
  bool hasMips32() const { return STI.hasFeature(Mips::FeatureMips32); }
```
- EN: Implements `hasMips2`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips2`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-57
```cpp
  bool hasMips32r6() const {
    return STI.hasFeature(Mips::FeatureMips32r6);
  }
```
- EN: Implements `hasMips32r6`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasMips32r6`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 59-59
```cpp
  bool isFP64() const { return STI.hasFeature(Mips::FeatureFP64Bit); }
```
- EN: Implements `isFP64`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isFP64`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 61-61
```cpp
  bool isGP64() const { return STI.hasFeature(Mips::FeatureGP64Bit); }
```
- EN: Implements `isGP64`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isGP64`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-63
```cpp
  bool isPTR64() const { return STI.hasFeature(Mips::FeaturePTR64Bit); }
```
- EN: Implements `isPTR64`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isPTR64`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 65-65
```cpp
  bool hasCnMips() const { return STI.hasFeature(Mips::FeatureCnMips); }
```
- EN: Implements `hasCnMips`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCnMips`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 67-67
```cpp
  bool hasCnMipsP() const { return STI.hasFeature(Mips::FeatureCnMipsP); }
```
- EN: Implements `hasCnMipsP`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCnMipsP`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 69-72
```cpp
  bool hasCOP3() const {
    // Only present in MIPS-I and MIPS-II
    return !hasMips32() && !hasMips3();
  }
```
- EN: Implements `hasCOP3`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasCOP3`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 74-77
```cpp
  DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CStream) const override;
};
```
- EN: Declares `getInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-79
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 81-85
```cpp
static MCDisassembler *createMipsDisassembler(const Target &T,
                                              const MCSubtargetInfo &STI,
                                              MCContext &Ctx) {
  return new MipsDisassembler(STI, Ctx, true);
}
```
- EN: Implements `createMipsDisassembler`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createMipsDisassembler`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 87-91
```cpp
static MCDisassembler *createMipselDisassembler(const Target &T,
                                                const MCSubtargetInfo &STI,
                                                MCContext &Ctx) {
  return new MipsDisassembler(STI, Ctx, false);
}
```
- EN: Implements `createMipselDisassembler`, a query/helper routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createMipselDisassembler`，它是一个围绕子目标特性处理展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 93-104
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeMipsDisassembler() {
  // Register the disassembler.
  TargetRegistry::RegisterMCDisassembler(getTheMipsTarget(),
                                         createMipsDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheMipselTarget(),
                                         createMipselDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheMips64Target(),
                                         createMipsDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheMips64elTarget(),
                                         createMipselDisassembler);
}
```
- EN: Implements `TargetRegistry::RegisterMCDisassembler`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `TargetRegistry::RegisterMCDisassembler`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-118
```cpp
static MCRegister getReg(const MCDisassembler *D, unsigned RC, unsigned RegNo) {
  const MCRegisterInfo *RegInfo = D->getContext().getRegisterInfo();
  return RegInfo->getRegClass(RC).getRegister(RegNo);
}
static DecodeStatus DecodeHWRegsRegisterClass(MCInst &Inst, unsigned RegNo,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  // Currently only hardware register 29 is supported.
  if (RegNo != 29)
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createReg(Mips::HWR29));
  return MCDisassembler::Success;
}
```
- EN: Implements `getReg`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getReg`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 120-124
```cpp
static DecodeStatus DecodeAFGR64RegisterClass(MCInst &Inst, unsigned RegNo,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  if (RegNo > 30 || RegNo % 2)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeAFGR64RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAFGR64RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 126-129
```cpp
  MCRegister Reg = getReg(Decoder, Mips::AFGR64RegClassID, RegNo / 2);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 131-135
```cpp
static DecodeStatus DecodeACC64DSPRegisterClass(MCInst &Inst, unsigned RegNo,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if (RegNo >= 4)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeACC64DSPRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeACC64DSPRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 137-140
```cpp
  MCRegister Reg = getReg(Decoder, Mips::ACC64DSPRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 142-146
```cpp
static DecodeStatus DecodeHI32DSPRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo >= 4)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeHI32DSPRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeHI32DSPRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 148-151
```cpp
  MCRegister Reg = getReg(Decoder, Mips::HI32DSPRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 153-157
```cpp
static DecodeStatus DecodeLO32DSPRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo >= 4)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeLO32DSPRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLO32DSPRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 159-162
```cpp
  MCRegister Reg = getReg(Decoder, Mips::LO32DSPRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 164-168
```cpp
static DecodeStatus DecodeMSA128BRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMSA128BRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSA128BRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 170-173
```cpp
  MCRegister Reg = getReg(Decoder, Mips::MSA128BRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 175-179
```cpp
static DecodeStatus DecodeMSA128HRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMSA128HRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSA128HRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-184
```cpp
  MCRegister Reg = getReg(Decoder, Mips::MSA128HRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-190
```cpp
static DecodeStatus DecodeMSA128WRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMSA128WRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSA128WRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 192-195
```cpp
  MCRegister Reg = getReg(Decoder, Mips::MSA128WRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 197-201
```cpp
static DecodeStatus DecodeMSA128DRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMSA128DRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSA128DRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 203-206
```cpp
  MCRegister Reg = getReg(Decoder, Mips::MSA128DRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 208-212
```cpp
static DecodeStatus DecodeMSACtrlRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMSACtrlRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSACtrlRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 214-217
```cpp
  MCRegister Reg = getReg(Decoder, Mips::MSACtrlRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-223
```cpp
static DecodeStatus DecodeCOP0RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeCOP0RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCOP0RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 225-228
```cpp
  MCRegister Reg = getReg(Decoder, Mips::COP0RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 230-234
```cpp
static DecodeStatus DecodeCOP2RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeCOP2RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCOP2RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 236-239
```cpp
  MCRegister Reg = getReg(Decoder, Mips::COP2RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 241-246
```cpp
static DecodeStatus DecodeRegListOperand(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  unsigned Regs[] = {Mips::S0, Mips::S1, Mips::S2, Mips::S3, Mips::S4,
                     Mips::S5, Mips::S6, Mips::S7, Mips::FP};
  unsigned RegNum;
```
- EN: Implements `DecodeRegListOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRegListOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 248-248
```cpp
  unsigned RegLst = fieldFromInstruction(Insn, 21, 5);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 250-252
```cpp
  // Empty register lists are not allowed.
  if (RegLst == 0)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 254-254
```cpp
  RegNum = RegLst & 0xf;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 256-258
```cpp
  // RegLst values 10-15, and 26-31 are reserved.
  if (RegNum > 9)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 260-261
```cpp
  for (unsigned i = 0; i < RegNum; i++)
    Inst.addOperand(MCOperand::createReg(Regs[i]));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 263-264
```cpp
  if (RegLst & 0x10)
    Inst.addOperand(MCOperand::createReg(Mips::RA));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 266-267
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 269-283
```cpp
static DecodeStatus DecodeRegListOperand16(MCInst &Inst, unsigned Insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  unsigned Regs[] = {Mips::S0, Mips::S1, Mips::S2, Mips::S3};
  unsigned RegLst;
  switch (Inst.getOpcode()) {
  default:
    RegLst = fieldFromInstruction(Insn, 4, 2);
    break;
  case Mips::LWM16_MMR6:
  case Mips::SWM16_MMR6:
    RegLst = fieldFromInstruction(Insn, 8, 2);
    break;
  }
  unsigned RegNum = RegLst & 0x3;
```
- EN: Implements `DecodeRegListOperand16`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRegListOperand16`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 285-286
```cpp
  for (unsigned i = 0; i <= RegNum; i++)
    Inst.addOperand(MCOperand::createReg(Regs[i]));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 288-288
```cpp
  Inst.addOperand(MCOperand::createReg(Mips::RA));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 290-291
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 293-297
```cpp
template <typename InsnType>
static DecodeStatus DecodeINSVE_DF(MCInst &MI, InsnType insn, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  using DecodeFN =
      DecodeStatus (*)(MCInst &, unsigned, uint64_t, const MCDisassembler *);
```
- EN: Implements `DecodeINSVE_DF`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeINSVE_DF`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 299-313
```cpp
  // The size of the n field depends on the element size
  // The register class also depends on this.
  InsnType tmp = fieldFromInstruction(insn, 17, 5);
  unsigned NSize = 0;
  DecodeFN RegDecoder = nullptr;
  if ((tmp & 0x18) == 0x00) { // INSVE_B
    NSize = 4;
    RegDecoder = DecodeMSA128BRegisterClass;
  } else if ((tmp & 0x1c) == 0x10) { // INSVE_H
    NSize = 3;
    RegDecoder = DecodeMSA128HRegisterClass;
  } else if ((tmp & 0x1e) == 0x18) { // INSVE_W
    NSize = 2;
    RegDecoder = DecodeMSA128WRegisterClass;
  } else if ((tmp & 0x1f) == 0x1c) { // INSVE_D
```
- EN: Declares `also`, packaging target-specific state and APIs around `MipsDisassembler`.
- CN: 这里声明 `also`，把与 `MipsDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 314-317
```cpp
    NSize = 1;
    RegDecoder = DecodeMSA128DRegisterClass;
  } else
    llvm_unreachable("Invalid encoding");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 319-319
```cpp
  assert(NSize != 0 && RegDecoder != nullptr);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 321-335
```cpp
  // $wd
  tmp = fieldFromInstruction(insn, 6, 5);
  if (RegDecoder(MI, tmp, Address, Decoder) == MCDisassembler::Fail)
    return MCDisassembler::Fail;
  // $wd_in
  if (RegDecoder(MI, tmp, Address, Decoder) == MCDisassembler::Fail)
    return MCDisassembler::Fail;
  // $n
  tmp = fieldFromInstruction(insn, 16, NSize);
  MI.addOperand(MCOperand::createImm(tmp));
  // $ws
  tmp = fieldFromInstruction(insn, 11, 5);
  if (RegDecoder(MI, tmp, Address, Decoder) == MCDisassembler::Fail)
    return MCDisassembler::Fail;
  // $n2
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 336-336
```cpp
  MI.addOperand(MCOperand::createImm(0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 338-339
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 341-351
```cpp
template <typename InsnType>
static DecodeStatus DecodeDAHIDATIMMR6(MCInst &MI, InsnType insn,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  InsnType Imm = fieldFromInstruction(insn, 0, 16);
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID,
                                       Rs)));
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID,
                                       Rs)));
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Implements `DecodeDAHIDATIMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDAHIDATIMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 353-354
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 356-365
```cpp
template <typename InsnType>
static DecodeStatus DecodeDAHIDATI(MCInst &MI, InsnType insn, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Imm = fieldFromInstruction(insn, 0, 16);
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID,
                                       Rs)));
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID,
                                       Rs)));
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Implements `DecodeDAHIDATI`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDAHIDATI`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 367-368
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 370-382
```cpp
template <typename InsnType>
static DecodeStatus DecodeAddiGroupBranch(MCInst &MI, InsnType insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  // If we are called then we can assume that MIPS32r6/MIPS64r6 is enabled
  // (otherwise we would have matched the ADDI instruction from the earlier
  // ISA's instead).
  //
  // We have:
  //    0b001000 sssss ttttt iiiiiiiiiiiiiiii
  //      BOVC if rs >= rt
  //      BEQZALC if rs == 0 && rt != 0
  //      BEQC if rs < rt && rs != 0
```
- EN: Implements `DecodeAddiGroupBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddiGroupBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 384-387
```cpp
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Rt = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 389-396
```cpp
  if (Rs >= Rt) {
    MI.setOpcode(Mips::BOVC);
    HasRs = true;
  } else if (Rs != 0 && Rs < Rt) {
    MI.setOpcode(Mips::BEQC);
    HasRs = true;
  } else
    MI.setOpcode(Mips::BEQZALC);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 398-400
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 402-404
```cpp
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 406-407
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 409-415
```cpp
template <typename InsnType>
static DecodeStatus DecodePOP35GroupBranchMMR6(MCInst &MI, InsnType insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  InsnType Rt = fieldFromInstruction(insn, 21, 5);
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = 0;
```
- EN: Implements `DecodePOP35GroupBranchMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePOP35GroupBranchMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 417-431
```cpp
  if (Rs >= Rt) {
    MI.setOpcode(Mips::BOVC_MMR6);
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  } else if (Rs != 0 && Rs < Rt) {
    MI.setOpcode(Mips::BEQC_MMR6);
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 432-436
```cpp
    MI.setOpcode(Mips::BEQZALC_MMR6);
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 438-438
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 440-441
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 443-455
```cpp
template <typename InsnType>
static DecodeStatus DecodeDaddiGroupBranch(MCInst &MI, InsnType insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  // If we are called then we can assume that MIPS32r6/MIPS64r6 is enabled
  // (otherwise we would have matched the ADDI instruction from the earlier
  // ISA's instead).
  //
  // We have:
  //    0b011000 sssss ttttt iiiiiiiiiiiiiiii
  //      BNVC if rs >= rt
  //      BNEZALC if rs == 0 && rt != 0
  //      BNEC if rs < rt && rs != 0
```
- EN: Implements `DecodeDaddiGroupBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDaddiGroupBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 457-460
```cpp
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Rt = fieldFromInstruction(insn, 16, 5);
  int64_t  Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 462-469
```cpp
  if (Rs >= Rt) {
    MI.setOpcode(Mips::BNVC);
    HasRs = true;
  } else if (Rs != 0 && Rs < Rt) {
    MI.setOpcode(Mips::BNEC);
    HasRs = true;
  } else
    MI.setOpcode(Mips::BNEZALC);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 471-473
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 475-477
```cpp
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 479-480
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 482-488
```cpp
template <typename InsnType>
static DecodeStatus DecodePOP37GroupBranchMMR6(MCInst &MI, InsnType insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  InsnType Rt = fieldFromInstruction(insn, 21, 5);
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = 0;
```
- EN: Implements `DecodePOP37GroupBranchMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePOP37GroupBranchMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 490-504
```cpp
  if (Rs >= Rt) {
    MI.setOpcode(Mips::BNVC_MMR6);
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  } else if (Rs != 0 && Rs < Rt) {
    MI.setOpcode(Mips::BNEC_MMR6);
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 505-509
```cpp
    MI.setOpcode(Mips::BNEZALC_MMR6);
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 511-511
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 513-514
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 516-525
```cpp
template <typename InsnType>
static DecodeStatus DecodePOP65GroupBranchMMR6(MCInst &MI, InsnType insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  // We have:
  //    0b110101 ttttt sssss iiiiiiiiiiiiiiii
  //      Invalid if rt == 0
  //      BGTZC_MMR6   if rs == 0  && rt != 0
  //      BLTZC_MMR6   if rs == rt && rt != 0
  //      BLTC_MMR6    if rs != rt && rs != 0  && rt != 0
```
- EN: Implements `DecodePOP65GroupBranchMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePOP65GroupBranchMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 527-530
```cpp
  InsnType Rt = fieldFromInstruction(insn, 21, 5);
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 532-541
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0)
    MI.setOpcode(Mips::BGTZC_MMR6);
  else if (Rs == Rt)
    MI.setOpcode(Mips::BLTZC_MMR6);
  else {
    MI.setOpcode(Mips::BLTC_MMR6);
    HasRs = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 543-545
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                              Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 547-548
```cpp
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 550-550
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 552-553
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 555-564
```cpp
template <typename InsnType>
static DecodeStatus DecodePOP75GroupBranchMMR6(MCInst &MI, InsnType insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  // We have:
  //    0b111101 ttttt sssss iiiiiiiiiiiiiiii
  //      Invalid if rt == 0
  //      BLEZC_MMR6   if rs == 0  && rt != 0
  //      BGEZC_MMR6   if rs == rt && rt != 0
  //      BGEC_MMR6    if rs != rt && rs != 0  && rt != 0
```
- EN: Implements `DecodePOP75GroupBranchMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePOP75GroupBranchMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 566-569
```cpp
  InsnType Rt = fieldFromInstruction(insn, 21, 5);
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 571-580
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0)
    MI.setOpcode(Mips::BLEZC_MMR6);
  else if (Rs == Rt)
    MI.setOpcode(Mips::BGEZC_MMR6);
  else {
    HasRs = true;
    MI.setOpcode(Mips::BGEC_MMR6);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 582-584
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 586-587
```cpp
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 589-589
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 591-592
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 594-607
```cpp
template <typename InsnType>
static DecodeStatus DecodeBlezlGroupBranch(MCInst &MI, InsnType insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  // If we are called then we can assume that MIPS32r6/MIPS64r6 is enabled
  // (otherwise we would have matched the BLEZL instruction from the earlier
  // ISA's instead).
  //
  // We have:
  //    0b010110 sssss ttttt iiiiiiiiiiiiiiii
  //      Invalid if rs == 0
  //      BLEZC   if rs == 0  && rt != 0
  //      BGEZC   if rs == rt && rt != 0
  //      BGEC    if rs != rt && rs != 0  && rt != 0
```
- EN: Implements `DecodeBlezlGroupBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBlezlGroupBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 609-612
```cpp
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Rt = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 614-623
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0)
    MI.setOpcode(Mips::BLEZC);
  else if (Rs == Rt)
    MI.setOpcode(Mips::BGEZC);
  else {
    HasRs = true;
    MI.setOpcode(Mips::BGEC);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 625-627
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 629-630
```cpp
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 632-632
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 634-635
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 637-650
```cpp
template <typename InsnType>
static DecodeStatus DecodeBgtzlGroupBranch(MCInst &MI, InsnType insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  // If we are called then we can assume that MIPS32r6/MIPS64r6 is enabled
  // (otherwise we would have matched the BGTZL instruction from the earlier
  // ISA's instead).
  //
  // We have:
  //    0b010111 sssss ttttt iiiiiiiiiiiiiiii
  //      Invalid if rs == 0
  //      BGTZC   if rs == 0  && rt != 0
  //      BLTZC   if rs == rt && rt != 0
  //      BLTC    if rs != rt && rs != 0  && rt != 0
```
- EN: Implements `DecodeBgtzlGroupBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBgtzlGroupBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 652-652
```cpp
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 654-656
```cpp
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Rt = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 658-667
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0)
    MI.setOpcode(Mips::BGTZC);
  else if (Rs == Rt)
    MI.setOpcode(Mips::BLTZC);
  else {
    MI.setOpcode(Mips::BLTC);
    HasRs = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 669-671
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                              Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 673-674
```cpp
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 676-676
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 678-679
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 681-694
```cpp
template <typename InsnType>
static DecodeStatus DecodeBgtzGroupBranch(MCInst &MI, InsnType insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  // If we are called then we can assume that MIPS32r6/MIPS64r6 is enabled
  // (otherwise we would have matched the BGTZ instruction from the earlier
  // ISA's instead).
  //
  // We have:
  //    0b000111 sssss ttttt iiiiiiiiiiiiiiii
  //      BGTZ    if rt == 0
  //      BGTZALC if rs == 0 && rt != 0
  //      BLTZALC if rs != 0 && rs == rt
  //      BLTUC   if rs != 0 && rs != rt
```
- EN: Implements `DecodeBgtzGroupBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBgtzGroupBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 696-700
```cpp
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Rt = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
  bool HasRt = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 702-715
```cpp
  if (Rt == 0) {
    MI.setOpcode(Mips::BGTZ);
    HasRs = true;
  } else if (Rs == 0) {
    MI.setOpcode(Mips::BGTZALC);
    HasRt = true;
  } else if (Rs == Rt) {
    MI.setOpcode(Mips::BLTZALC);
    HasRs = true;
  } else {
    MI.setOpcode(Mips::BLTUC);
    HasRs = true;
    HasRt = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 717-719
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 721-723
```cpp
  if (HasRt)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rt)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 725-725
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 727-728
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 730-743
```cpp
template <typename InsnType>
static DecodeStatus DecodeBlezGroupBranch(MCInst &MI, InsnType insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  // If we are called then we can assume that MIPS32r6/MIPS64r6 is enabled
  // (otherwise we would have matched the BLEZL instruction from the earlier
  // ISA's instead).
  //
  // We have:
  //    0b000110 sssss ttttt iiiiiiiiiiiiiiii
  //      Invalid   if rs == 0
  //      BLEZALC   if rs == 0  && rt != 0
  //      BGEZALC   if rs == rt && rt != 0
  //      BGEUC     if rs != rt && rs != 0  && rt != 0
```
- EN: Implements `DecodeBlezGroupBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBlezGroupBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 745-748
```cpp
  InsnType Rs = fieldFromInstruction(insn, 21, 5);
  InsnType Rt = fieldFromInstruction(insn, 16, 5);
  int64_t Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 750-759
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0)
    MI.setOpcode(Mips::BLEZALC);
  else if (Rs == Rt)
    MI.setOpcode(Mips::BGEZALC);
  else {
    HasRs = true;
    MI.setOpcode(Mips::BGEUC);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 761-765
```cpp
  if (HasRs)
    MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                       Rs)));
  MI.addOperand(MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID,
                                     Rt)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 767-767
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 769-770
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 772-780
```cpp
// Override the generated disassembler to produce DEXT all the time. This is
// for feature / behaviour parity with  binutils.
template <typename InsnType>
static DecodeStatus DecodeDEXT(MCInst &MI, InsnType Insn, uint64_t Address,
                               const MCDisassembler *Decoder) {
  unsigned Msbd = fieldFromInstruction(Insn, 11, 5);
  unsigned Lsb = fieldFromInstruction(Insn, 6, 5);
  unsigned Size = 0;
  unsigned Pos = 0;
```
- EN: Implements `DecodeDEXT`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDEXT`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 782-796
```cpp
  switch (MI.getOpcode()) {
    case Mips::DEXT:
      Pos = Lsb;
      Size = Msbd + 1;
      break;
    case Mips::DEXTM:
      Pos = Lsb;
      Size = Msbd + 1 + 32;
      break;
    case Mips::DEXTU:
      Pos = Lsb + 32;
      Size = Msbd + 1;
      break;
    default:
      llvm_unreachable("Unknown DEXT instruction!");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 797-797
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 799-799
```cpp
  MI.setOpcode(Mips::DEXT);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 801-802
```cpp
  InsnType Rs = fieldFromInstruction(Insn, 21, 5);
  InsnType Rt = fieldFromInstruction(Insn, 16, 5);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 804-809
```cpp
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID, Rt)));
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID, Rs)));
  MI.addOperand(MCOperand::createImm(Pos));
  MI.addOperand(MCOperand::createImm(Size));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 811-812
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 814-822
```cpp
// Override the generated disassembler to produce DINS all the time. This is
// for feature / behaviour parity with binutils.
template <typename InsnType>
static DecodeStatus DecodeDINS(MCInst &MI, InsnType Insn, uint64_t Address,
                               const MCDisassembler *Decoder) {
  unsigned Msbd = fieldFromInstruction(Insn, 11, 5);
  unsigned Lsb = fieldFromInstruction(Insn, 6, 5);
  unsigned Size = 0;
  unsigned Pos = 0;
```
- EN: Implements `DecodeDINS`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDINS`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 824-838
```cpp
  switch (MI.getOpcode()) {
    case Mips::DINS:
      Pos = Lsb;
      Size = Msbd + 1 - Pos;
      break;
    case Mips::DINSM:
      Pos = Lsb;
      Size = Msbd + 33 - Pos;
      break;
    case Mips::DINSU:
      Pos = Lsb + 32;
      // mbsd = pos + size - 33
      // mbsd - pos + 33 = size
      Size = Msbd + 33 - Pos;
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 839-841
```cpp
    default:
      llvm_unreachable("Unknown DINS instruction!");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 843-844
```cpp
  InsnType Rs = fieldFromInstruction(Insn, 21, 5);
  InsnType Rt = fieldFromInstruction(Insn, 16, 5);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 846-852
```cpp
  MI.setOpcode(Mips::DINS);
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID, Rt)));
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR64RegClassID, Rs)));
  MI.addOperand(MCOperand::createImm(Pos));
  MI.addOperand(MCOperand::createImm(Size));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 854-855
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 857-870
```cpp
// Auto-generated decoder wouldn't add the third operand for CRC32*.
template <typename InsnType>
static DecodeStatus DecodeCRC(MCInst &MI, InsnType Insn, uint64_t Address,
                              const MCDisassembler *Decoder) {
  InsnType Rs = fieldFromInstruction(Insn, 21, 5);
  InsnType Rt = fieldFromInstruction(Insn, 16, 5);
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rt)));
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rs)));
  MI.addOperand(
      MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rt)));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeCRC`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCRC`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 872-876
```cpp
static DecodeStatus
DecodeCPU16RegsRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                             const MCDisassembler *Decoder) {
  return MCDisassembler::Fail;
}
```
- EN: Implements `DecodeCPU16RegsRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCPU16RegsRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 878-882
```cpp
static DecodeStatus DecodeGPR64RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeGPR64RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPR64RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 884-887
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR64RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 889-897
```cpp
static DecodeStatus DecodeGPRMM16RegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
  MCRegister Reg = getReg(Decoder, Mips::GPRMM16RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeGPRMM16RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRMM16RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 899-907
```cpp
static DecodeStatus
DecodeGPRMM16ZeroRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                               const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
  MCRegister Reg = getReg(Decoder, Mips::GPRMM16ZeroRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeGPRMM16ZeroRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRMM16ZeroRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 909-917
```cpp
static DecodeStatus
DecodeGPRMM16MovePRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                                const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
  MCRegister Reg = getReg(Decoder, Mips::GPRMM16MovePRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeGPRMM16MovePRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRMM16MovePRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 919-926
```cpp
// Tablegen emits references to these unimplemented functions due to usage of
// RegClassByHwMode - it does not detect that the RegClassByHwMode decoders are
// unused, which in turn use these register class decoders.
static DecodeStatus DecodeGP32RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  llvm_unreachable("this is unused");
}
```
- EN: Declares `decoders`, packaging target-specific state and APIs around `MipsDisassembler`.
- CN: 这里声明 `decoders`，把与 `MipsDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 928-932
```cpp
static DecodeStatus DecodeGP64RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  llvm_unreachable("this is unused");
}
```
- EN: Implements `DecodeGP64RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGP64RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 934-938
```cpp
static DecodeStatus DecodeSP32RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  llvm_unreachable("this is unused");
}
```
- EN: Implements `DecodeSP32RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSP32RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 940-944
```cpp
static DecodeStatus DecodeSP64RegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  llvm_unreachable("this is unused");
}
```
- EN: Implements `DecodeSP64RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSP64RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 946-954
```cpp
static DecodeStatus DecodeGPR32RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeGPR32RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPR32RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 956-960
```cpp
static DecodeStatus DecodePtrRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (static_cast<const MipsDisassembler *>(Decoder)->isGP64())
    return DecodeGPR64RegisterClass(Inst, RegNo, Address, Decoder);
```
- EN: Implements `DecodePtrRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePtrRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 962-963
```cpp
  return DecodeGPR32RegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 965-969
```cpp
static DecodeStatus DecodeDSPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  return DecodeGPR32RegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Implements `DecodeDSPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDSPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 971-975
```cpp
static DecodeStatus DecodeFGR64RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeFGR64RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFGR64RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 977-980
```cpp
  MCRegister Reg = getReg(Decoder, Mips::FGR64RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 982-986
```cpp
static DecodeStatus DecodeFGR32RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeFGR32RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFGR32RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 988-991
```cpp
  MCRegister Reg = getReg(Decoder, Mips::FGR32RegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 993-1001
```cpp
static DecodeStatus DecodeCCRRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
  MCRegister Reg = getReg(Decoder, Mips::CCRRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeCCRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCCRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1003-1011
```cpp
static DecodeStatus DecodeFCCRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
  MCRegister Reg = getReg(Decoder, Mips::FCCRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeFCCRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFCCRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1013-1017
```cpp
static DecodeStatus DecodeFGR32CCRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeFGR32CCRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFGR32CCRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1019-1022
```cpp
  MCRegister Reg = getReg(Decoder, Mips::FGR32CCRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1024-1028
```cpp
static DecodeStatus DecodeFGR64CCRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeFGR64CCRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFGR64CCRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1030-1033
```cpp
  MCRegister Reg = getReg(Decoder, Mips::FGR64CCRegClassID, RegNo);
  Inst.addOperand(MCOperand::createReg(Reg));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1035-1039
```cpp
static DecodeStatus DecodeMem(MCInst &Inst, unsigned Insn, uint64_t Address,
                              const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned RegNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeMem`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMem`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1041-1042
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1044-1046
```cpp
  if (Inst.getOpcode() == Mips::SC || Inst.getOpcode() == Mips::SC64 ||
      Inst.getOpcode() == Mips::SCD)
    Inst.addOperand(MCOperand::createReg(Reg));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1048-1050
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1052-1053
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1055-1059
```cpp
static DecodeStatus DecodeMemEVA(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  int Offset = SignExtend32<9>(Insn >> 7);
  unsigned RegNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeMemEVA`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemEVA`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1061-1062
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1064-1065
```cpp
  if (Inst.getOpcode() == Mips::SCE)
    Inst.addOperand(MCOperand::createReg(Reg));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1067-1069
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1071-1072
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1074-1079
```cpp
static DecodeStatus DecodeLoadByte15(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
  unsigned RegNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeLoadByte15`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLoadByte15`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1081-1082
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1084-1086
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1088-1089
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1091-1095
```cpp
static DecodeStatus DecodeCacheOp(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned Hint = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeCacheOp`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCacheOp`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1097-1097
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1099-1101
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
  Inst.addOperand(MCOperand::createImm(Hint));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1103-1104
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1106-1111
```cpp
static DecodeStatus DecodeCacheOpMM(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  int Offset = SignExtend32<12>(Insn & 0xfff);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
  unsigned Hint = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeCacheOpMM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCacheOpMM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1113-1113
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1115-1117
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
  Inst.addOperand(MCOperand::createImm(Hint));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1119-1120
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1122-1127
```cpp
static DecodeStatus DecodePrefeOpMM(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  int Offset = SignExtend32<9>(Insn & 0x1ff);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
  unsigned Hint = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodePrefeOpMM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePrefeOpMM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1129-1129
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1131-1133
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
  Inst.addOperand(MCOperand::createImm(Hint));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1135-1136
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1138-1143
```cpp
static DecodeStatus DecodeCacheeOp_CacheOpR6(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  int Offset = SignExtend32<9>(Insn >> 7);
  unsigned Hint = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeCacheeOp_CacheOpR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCacheeOp_CacheOpR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1145-1145
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1147-1149
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
  Inst.addOperand(MCOperand::createImm(Hint));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1151-1152
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1154-1157
```cpp
static DecodeStatus DecodeSyncI(MCInst &Inst, unsigned Insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeSyncI`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSyncI`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1159-1159
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1161-1162
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1164-1165
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1167-1171
```cpp
static DecodeStatus DecodeSyncI_MM(MCInst &Inst, unsigned Insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
```
- EN: Implements `DecodeSyncI_MM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSyncI_MM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1173-1173
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1175-1176
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1178-1179
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1181-1184
```cpp
static DecodeStatus DecodeSynciR6(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  int Immediate = SignExtend32<16>(Insn & 0xffff);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
```
- EN: Implements `DecodeSynciR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSynciR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1186-1186
```cpp
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1188-1189
```cpp
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Immediate));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1191-1192
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1194-1199
```cpp
static DecodeStatus DecodeMSA128Mem(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  int Offset = SignExtend32<10>(fieldFromInstruction(Insn, 16, 10));
  unsigned RegNo = fieldFromInstruction(Insn, 6, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 11, 5);
```
- EN: Implements `DecodeMSA128Mem`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSA128Mem`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1201-1202
```cpp
  MCRegister Reg = getReg(Decoder, Mips::MSA128BRegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1204-1205
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1207-1221
```cpp
  // The immediate field of an LD/ST instruction is scaled which means it must
  // be multiplied (when decoding) by the size (in bytes) of the instructions'
  // data format.
  // .b - 1 byte
  // .h - 2 bytes
  // .w - 4 bytes
  // .d - 8 bytes
  switch(Inst.getOpcode())
  {
  default:
    assert(false && "Unexpected instruction");
    return MCDisassembler::Fail;
    break;
  case Mips::LD_B:
  case Mips::ST_B:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1222-1236
```cpp
    Inst.addOperand(MCOperand::createImm(Offset));
    break;
  case Mips::LD_H:
  case Mips::ST_H:
    Inst.addOperand(MCOperand::createImm(Offset * 2));
    break;
  case Mips::LD_W:
  case Mips::ST_W:
    Inst.addOperand(MCOperand::createImm(Offset * 4));
    break;
  case Mips::LD_D:
  case Mips::ST_D:
    Inst.addOperand(MCOperand::createImm(Offset * 8));
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1238-1239
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1241-1246
```cpp
static DecodeStatus DecodeMemMMImm4(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  unsigned Offset = Insn & 0xf;
  unsigned Reg = fieldFromInstruction(Insn, 7, 3);
  unsigned Base = fieldFromInstruction(Insn, 4, 3);
```
- EN: Implements `DecodeMemMMImm4`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMImm4`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1248-1262
```cpp
  switch (Inst.getOpcode()) {
    case Mips::LBU16_MM:
    case Mips::LHU16_MM:
    case Mips::LW16_MM:
      if (DecodeGPRMM16RegisterClass(Inst, Reg, Address, Decoder)
            == MCDisassembler::Fail)
        return MCDisassembler::Fail;
      break;
    case Mips::SB16_MM:
    case Mips::SB16_MMR6:
    case Mips::SH16_MM:
    case Mips::SH16_MMR6:
    case Mips::SW16_MM:
    case Mips::SW16_MMR6:
      if (DecodeGPRMM16ZeroRegisterClass(Inst, Reg, Address, Decoder)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1263-1266
```cpp
            == MCDisassembler::Fail)
        return MCDisassembler::Fail;
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1268-1270
```cpp
  if (DecodeGPRMM16RegisterClass(Inst, Base, Address, Decoder)
        == MCDisassembler::Fail)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1272-1286
```cpp
  switch (Inst.getOpcode()) {
    case Mips::LBU16_MM:
      if (Offset == 0xf)
        Inst.addOperand(MCOperand::createImm(-1));
      else
        Inst.addOperand(MCOperand::createImm(Offset));
      break;
    case Mips::SB16_MM:
    case Mips::SB16_MMR6:
      Inst.addOperand(MCOperand::createImm(Offset));
      break;
    case Mips::LHU16_MM:
    case Mips::SH16_MM:
    case Mips::SH16_MMR6:
      Inst.addOperand(MCOperand::createImm(Offset << 1));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1287-1293
```cpp
      break;
    case Mips::LW16_MM:
    case Mips::SW16_MM:
    case Mips::SW16_MMR6:
      Inst.addOperand(MCOperand::createImm(Offset << 2));
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1295-1296
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1298-1302
```cpp
static DecodeStatus DecodeMemMMSPImm5Lsl2(MCInst &Inst, unsigned Insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  unsigned Offset = Insn & 0x1F;
  unsigned RegNo = fieldFromInstruction(Insn, 5, 5);
```
- EN: Implements `DecodeMemMMSPImm5Lsl2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMSPImm5Lsl2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1304-1304
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1306-1308
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Mips::SP));
  Inst.addOperand(MCOperand::createImm(Offset << 2));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1310-1311
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1313-1317
```cpp
static DecodeStatus DecodeMemMMGPImm7Lsl2(MCInst &Inst, unsigned Insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  unsigned Offset = Insn & 0x7F;
  unsigned RegNo = fieldFromInstruction(Insn, 7, 3);
```
- EN: Implements `DecodeMemMMGPImm7Lsl2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMGPImm7Lsl2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1319-1319
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1321-1323
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Mips::GP));
  Inst.addOperand(MCOperand::createImm(Offset << 2));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1325-1326
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1328-1340
```cpp
static DecodeStatus DecodeMemMMReglistImm4Lsl2(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  int Offset;
  switch (Inst.getOpcode()) {
  case Mips::LWM16_MMR6:
  case Mips::SWM16_MMR6:
    Offset = fieldFromInstruction(Insn, 4, 4);
    break;
  default:
    Offset = SignExtend32<4>(Insn & 0xf);
    break;
  }
```
- EN: Implements `DecodeMemMMReglistImm4Lsl2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMReglistImm4Lsl2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1342-1344
```cpp
  if (DecodeRegListOperand16(Inst, Insn, Address, Decoder)
      == MCDisassembler::Fail)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1346-1347
```cpp
  Inst.addOperand(MCOperand::createReg(Mips::SP));
  Inst.addOperand(MCOperand::createImm(Offset << 2));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1349-1350
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1352-1357
```cpp
static DecodeStatus DecodeMemMMImm9(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  int Offset = SignExtend32<9>(Insn & 0x1ff);
  unsigned RegNo = fieldFromInstruction(Insn, 21, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
```
- EN: Implements `DecodeMemMMImm9`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMImm9`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1359-1360
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1362-1363
```cpp
  if (Inst.getOpcode() == Mips::SCE_MM || Inst.getOpcode() == Mips::SC_MMR6)
    Inst.addOperand(MCOperand::createReg(Reg));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1365-1367
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1369-1370
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1372-1377
```cpp
static DecodeStatus DecodeMemMMImm12(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  int Offset = SignExtend32<12>(Insn & 0x0fff);
  unsigned RegNo = fieldFromInstruction(Insn, 21, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
```
- EN: Implements `DecodeMemMMImm12`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMImm12`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1379-1380
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1382-1396
```cpp
  switch (Inst.getOpcode()) {
  case Mips::SWM32_MM:
  case Mips::LWM32_MM:
    if (DecodeRegListOperand(Inst, Insn, Address, Decoder)
        == MCDisassembler::Fail)
      return MCDisassembler::Fail;
    Inst.addOperand(MCOperand::createReg(Base));
    Inst.addOperand(MCOperand::createImm(Offset));
    break;
  case Mips::SC_MM:
    Inst.addOperand(MCOperand::createReg(Reg));
    [[fallthrough]];
  default:
    Inst.addOperand(MCOperand::createReg(Reg));
    if (Inst.getOpcode() == Mips::LWP_MM || Inst.getOpcode() == Mips::SWP_MM)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1397-1397
```cpp
      Inst.addOperand(MCOperand::createReg(Reg+1));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1399-1401
```cpp
    Inst.addOperand(MCOperand::createReg(Base));
    Inst.addOperand(MCOperand::createImm(Offset));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1403-1404
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1406-1411
```cpp
static DecodeStatus DecodeMemMMImm16(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned RegNo = fieldFromInstruction(Insn, 21, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
```
- EN: Implements `DecodeMemMMImm16`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMMImm16`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1413-1414
```cpp
  MCRegister Reg = getReg(Decoder, Mips::GPR32RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1416-1418
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1420-1421
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1423-1427
```cpp
static DecodeStatus DecodeFMem(MCInst &Inst, unsigned Insn, uint64_t Address,
                               const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned RegNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeFMem`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFMem`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1429-1430
```cpp
  MCRegister Reg = getReg(Decoder, Mips::FGR64RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1432-1434
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1436-1437
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1439-1446
```cpp
static DecodeStatus DecodeFMemMMR2(MCInst &Inst, unsigned Insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  // This function is the same as DecodeFMem but with the Reg and Base fields
  // swapped according to microMIPS spec.
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
  unsigned RegNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeFMemMMR2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFMemMMR2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1448-1449
```cpp
  MCRegister Reg = getReg(Decoder, Mips::FGR64RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1451-1453
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1455-1456
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1458-1462
```cpp
static DecodeStatus DecodeFMem2(MCInst &Inst, unsigned Insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned RegNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeFMem2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFMem2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1464-1465
```cpp
  MCRegister Reg = getReg(Decoder, Mips::COP2RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1467-1469
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1471-1472
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1474-1478
```cpp
static DecodeStatus DecodeFMem3(MCInst &Inst, unsigned Insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  int Offset = SignExtend32<16>(Insn & 0xffff);
  unsigned RegNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeFMem3`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFMem3`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1480-1481
```cpp
  MCRegister Reg = getReg(Decoder, Mips::COP3RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1483-1485
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1487-1488
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1490-1495
```cpp
static DecodeStatus DecodeFMemCop2R6(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  int Offset = SignExtend32<11>(Insn & 0x07ff);
  unsigned RegNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 11, 5);
```
- EN: Implements `DecodeFMemCop2R6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFMemCop2R6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1497-1498
```cpp
  MCRegister Reg = getReg(Decoder, Mips::COP2RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1500-1502
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1504-1505
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1507-1512
```cpp
static DecodeStatus DecodeFMemCop2MMR6(MCInst &Inst, unsigned Insn,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  int Offset = SignExtend32<11>(Insn & 0x07ff);
  unsigned RegNo = fieldFromInstruction(Insn, 21, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 16, 5);
```
- EN: Implements `DecodeFMemCop2MMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFMemCop2MMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1514-1515
```cpp
  MCRegister Reg = getReg(Decoder, Mips::COP2RegClassID, RegNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1517-1519
```cpp
  Inst.addOperand(MCOperand::createReg(Reg));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1521-1522
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1524-1529
```cpp
static DecodeStatus DecodeSpecial3LlSc(MCInst &Inst, unsigned Insn,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  int64_t Offset = SignExtend64<9>((Insn >> 7) & 0x1ff);
  unsigned RtNo = fieldFromInstruction(Insn, 16, 5);
  unsigned BaseNo = fieldFromInstruction(Insn, 21, 5);
```
- EN: Implements `DecodeSpecial3LlSc`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSpecial3LlSc`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1531-1532
```cpp
  MCRegister Rt = getReg(Decoder, Mips::GPR32RegClassID, RtNo);
  MCRegister Base = getReg(Decoder, Mips::GPR32RegClassID, BaseNo);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1534-1536
```cpp
  if(Inst.getOpcode() == Mips::SC_R6 || Inst.getOpcode() == Mips::SCD_R6){
    Inst.addOperand(MCOperand::createReg(Rt));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1538-1540
```cpp
  Inst.addOperand(MCOperand::createReg(Rt));
  Inst.addOperand(MCOperand::createReg(Base));
  Inst.addOperand(MCOperand::createImm(Offset));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1542-1543
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1545-1551
```cpp
static DecodeStatus DecodeBranchTarget(MCInst &Inst, unsigned Offset,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  int32_t BranchOffset = (SignExtend32<16>(Offset) * 4) + 4;
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeBranchTarget`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1553-1559
```cpp
static DecodeStatus DecodeBranchTarget1SImm16(MCInst &Inst, unsigned Offset,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  int32_t BranchOffset = (SignExtend32<16>(Offset) * 2);
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeBranchTarget1SImm16`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget1SImm16`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1561-1567
```cpp
static DecodeStatus DecodeJumpTarget(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  unsigned JumpOffset = fieldFromInstruction(Insn, 0, 26) << 2;
  Inst.addOperand(MCOperand::createImm(JumpOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeJumpTarget`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeJumpTarget`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1569-1572
```cpp
static DecodeStatus DecodeBranchTarget21(MCInst &Inst, unsigned Offset,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<21>(Offset) * 4 + 4;
```
- EN: Implements `DecodeBranchTarget21`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget21`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1574-1576
```cpp
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1578-1581
```cpp
static DecodeStatus DecodeBranchTarget21MM(MCInst &Inst, unsigned Offset,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<21>(Offset) * 4 + 4;
```
- EN: Implements `DecodeBranchTarget21MM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget21MM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1583-1585
```cpp
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1587-1590
```cpp
static DecodeStatus DecodeBranchTarget26(MCInst &Inst, unsigned Offset,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<26>(Offset) * 4 + 4;
```
- EN: Implements `DecodeBranchTarget26`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget26`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1592-1594
```cpp
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1596-1602
```cpp
static DecodeStatus DecodeBranchTarget7MM(MCInst &Inst, unsigned Offset,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<8>(Offset << 1);
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeBranchTarget7MM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget7MM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1604-1610
```cpp
static DecodeStatus DecodeBranchTarget10MM(MCInst &Inst, unsigned Offset,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<11>(Offset << 1);
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeBranchTarget10MM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget10MM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1612-1618
```cpp
static DecodeStatus DecodeBranchTargetMM(MCInst &Inst, unsigned Offset,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<16>(Offset) * 2 + 4;
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeBranchTargetMM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTargetMM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1620-1623
```cpp
static DecodeStatus DecodeBranchTarget26MM(MCInst &Inst, unsigned Offset,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  int32_t BranchOffset = SignExtend32<27>(Offset << 1);
```
- EN: Implements `DecodeBranchTarget26MM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchTarget26MM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1625-1627
```cpp
  Inst.addOperand(MCOperand::createImm(BranchOffset));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1629-1635
```cpp
static DecodeStatus DecodeJumpTargetMM(MCInst &Inst, unsigned Insn,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  unsigned JumpOffset = fieldFromInstruction(Insn, 0, 26) << 1;
  Inst.addOperand(MCOperand::createImm(JumpOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeJumpTargetMM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeJumpTargetMM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1637-1643
```cpp
static DecodeStatus DecodeJumpTargetXMM(MCInst &Inst, unsigned Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  unsigned JumpOffset = fieldFromInstruction(Insn, 0, 26) << 2;
  Inst.addOperand(MCOperand::createImm(JumpOffset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeJumpTargetXMM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeJumpTargetXMM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1645-1655
```cpp
static DecodeStatus DecodeAddiur2Simm7(MCInst &Inst, unsigned Value,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  if (Value == 0)
    Inst.addOperand(MCOperand::createImm(1));
  else if (Value == 0x7)
    Inst.addOperand(MCOperand::createImm(-1));
  else
    Inst.addOperand(MCOperand::createImm(Value << 2));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeAddiur2Simm7`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddiur2Simm7`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1657-1665
```cpp
static DecodeStatus DecodeLi16Imm(MCInst &Inst, unsigned Value,
                                  uint64_t Address,
                                  const MCDisassembler *Decoder) {
  if (Value == 0x7F)
    Inst.addOperand(MCOperand::createImm(-1));
  else
    Inst.addOperand(MCOperand::createImm(Value));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeLi16Imm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLi16Imm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1667-1672
```cpp
static DecodeStatus DecodePOOL16BEncodedField(MCInst &Inst, unsigned Value,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(Value == 0x0 ? 8 : Value));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodePOOL16BEncodedField`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePOOL16BEncodedField`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1674-1682
```cpp
template <unsigned Bits, int Offset, int Scale>
static DecodeStatus
DecodeUImmWithOffsetAndScale(MCInst &Inst, unsigned Value, uint64_t Address,
                             const MCDisassembler *Decoder) {
  Value &= ((1 << Bits) - 1);
  Value *= Scale;
  Inst.addOperand(MCOperand::createImm(Value + Offset));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeUImmWithOffsetAndScale`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeUImmWithOffsetAndScale`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1684-1691
```cpp
template <unsigned Bits, int Offset = 0, int ScaleBy = 1>
static DecodeStatus
DecodeSImmWithOffsetAndScale(MCInst &Inst, unsigned Value, uint64_t Address,
                             const MCDisassembler *Decoder) {
  int32_t Imm = SignExtend32<Bits>(Value) * ScaleBy;
  Inst.addOperand(MCOperand::createImm(Imm + Offset));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1693-1699
```cpp
template <unsigned Bits, int Offset>
static DecodeStatus DecodeUImmWithOffset(MCInst &Inst, unsigned Value,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  return DecodeUImmWithOffsetAndScale<Bits, Offset, 1>(Inst, Value, Address,
                                                       Decoder);
}
```
- EN: Implements `DecodeUImmWithOffset`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeUImmWithOffset`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1701-1710
```cpp
static DecodeStatus DecodeInsSize(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  // First we need to grab the pos(lsb) from MCInst.
  // This function only handles the 32 bit variants of ins, as dins
  // variants are handled differently.
  int Pos = Inst.getOperand(2).getImm();
  int Size = (int) Insn - Pos + 1;
  Inst.addOperand(MCOperand::createImm(SignExtend32<16>(Size)));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeInsSize`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeInsSize`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1712-1717
```cpp
static DecodeStatus DecodeSimm19Lsl2(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(SignExtend32<19>(Insn) * 4));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeSimm19Lsl2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSimm19Lsl2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1719-1724
```cpp
static DecodeStatus DecodeSimm18Lsl3(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(SignExtend32<18>(Insn) * 8));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeSimm18Lsl3`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSimm18Lsl3`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1726-1738
```cpp
static DecodeStatus DecodeSimm9SP(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  int32_t DecodedValue;
  switch (Insn) {
  case 0: DecodedValue = 256; break;
  case 1: DecodedValue = 257; break;
  case 510: DecodedValue = -258; break;
  case 511: DecodedValue = -257; break;
  default: DecodedValue = SignExtend32<9>(Insn); break;
  }
  Inst.addOperand(MCOperand::createImm(DecodedValue * 4));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeSimm9SP`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSimm9SP`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1740-1749
```cpp
static DecodeStatus DecodeANDI16Imm(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  // Insn must be >= 0, since it is unsigned that condition is always true.
  assert(Insn < 16);
  int32_t DecodedValues[] = {128, 1,  2,  3,  4,  7,   8,     15,
                             16,  31, 32, 63, 64, 255, 32768, 65535};
  Inst.addOperand(MCOperand::createImm(DecodedValues[Insn]));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeANDI16Imm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeANDI16Imm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1751-1765
```cpp
static DecodeStatus DecodeMovePRegPair(MCInst &Inst, unsigned RegPair,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  switch (RegPair) {
  default:
    return MCDisassembler::Fail;
  case 0:
    Inst.addOperand(MCOperand::createReg(Mips::A1));
    Inst.addOperand(MCOperand::createReg(Mips::A2));
    break;
  case 1:
    Inst.addOperand(MCOperand::createReg(Mips::A1));
    Inst.addOperand(MCOperand::createReg(Mips::A3));
    break;
  case 2:
```
- EN: Implements `DecodeMovePRegPair`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMovePRegPair`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1766-1780
```cpp
    Inst.addOperand(MCOperand::createReg(Mips::A2));
    Inst.addOperand(MCOperand::createReg(Mips::A3));
    break;
  case 3:
    Inst.addOperand(MCOperand::createReg(Mips::A0));
    Inst.addOperand(MCOperand::createReg(Mips::S5));
    break;
  case 4:
    Inst.addOperand(MCOperand::createReg(Mips::A0));
    Inst.addOperand(MCOperand::createReg(Mips::S6));
    break;
  case 5:
    Inst.addOperand(MCOperand::createReg(Mips::A0));
    Inst.addOperand(MCOperand::createReg(Mips::A1));
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1781-1789
```cpp
  case 6:
    Inst.addOperand(MCOperand::createReg(Mips::A0));
    Inst.addOperand(MCOperand::createReg(Mips::A2));
    break;
  case 7:
    Inst.addOperand(MCOperand::createReg(Mips::A0));
    Inst.addOperand(MCOperand::createReg(Mips::A3));
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1791-1792
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1794-1800
```cpp
static DecodeStatus DecodeMovePOperands(MCInst &Inst, unsigned Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  unsigned RegPair = fieldFromInstruction(Insn, 7, 3);
  if (DecodeMovePRegPair(Inst, RegPair, Address, Decoder) ==
      MCDisassembler::Fail)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMovePOperands`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMovePOperands`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1802-1810
```cpp
  unsigned RegRs;
  if (static_cast<const MipsDisassembler *>(Decoder)->hasMips32r6())
    RegRs = fieldFromInstruction(Insn, 0, 2) |
            (fieldFromInstruction(Insn, 3, 1) << 2);
  else
    RegRs = fieldFromInstruction(Insn, 1, 3);
  if (DecodeGPRMM16MovePRegisterClass(Inst, RegRs, Address, Decoder) ==
      MCDisassembler::Fail)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1812-1815
```cpp
  unsigned RegRt = fieldFromInstruction(Insn, 4, 3);
  if (DecodeGPRMM16MovePRegisterClass(Inst, RegRt, Address, Decoder) ==
      MCDisassembler::Fail)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1817-1818
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1820-1825
```cpp
static DecodeStatus DecodeSimm23Lsl2(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(SignExtend32<25>(Insn << 2)));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeSimm23Lsl2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSimm23Lsl2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1827-1836
```cpp
template <typename InsnType>
static DecodeStatus DecodeBgtzGroupBranchMMR6(MCInst &MI, InsnType insn,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  // We have:
  //    0b000111 ttttt sssss iiiiiiiiiiiiiiii
  //      Invalid      if rt == 0
  //      BGTZALC_MMR6 if rs == 0 && rt != 0
  //      BLTZALC_MMR6 if rs != 0 && rs == rt
  //      BLTUC_MMR6   if rs != 0 && rs != rt
```
- EN: Implements `DecodeBgtzGroupBranchMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBgtzGroupBranchMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1838-1842
```cpp
  InsnType Rt = fieldFromInstruction(insn, 21, 5);
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  InsnType Imm = 0;
  bool HasRs = false;
  bool HasRt = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1844-1858
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0) {
    MI.setOpcode(Mips::BGTZALC_MMR6);
    HasRt = true;
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  }
  else if (Rs == Rt) {
    MI.setOpcode(Mips::BLTZALC_MMR6);
    HasRs = true;
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  }
  else {
    MI.setOpcode(Mips::BLTUC_MMR6);
    HasRs = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1859-1861
```cpp
    HasRt = true;
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1863-1865
```cpp
  if (HasRs)
    MI.addOperand(
    MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rs)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1867-1869
```cpp
  if (HasRt)
    MI.addOperand(
    MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rt)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1871-1871
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1873-1874
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1876-1885
```cpp
template <typename InsnType>
static DecodeStatus DecodeBlezGroupBranchMMR6(MCInst &MI, InsnType insn,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  // We have:
  //    0b000110 ttttt sssss iiiiiiiiiiiiiiii
  //      Invalid        if rt == 0
  //      BLEZALC_MMR6   if rs == 0  && rt != 0
  //      BGEZALC_MMR6   if rs == rt && rt != 0
  //      BGEUC_MMR6     if rs != rt && rs != 0  && rt != 0
```
- EN: Implements `DecodeBlezGroupBranchMMR6`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBlezGroupBranchMMR6`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1887-1890
```cpp
  InsnType Rt = fieldFromInstruction(insn, 21, 5);
  InsnType Rs = fieldFromInstruction(insn, 16, 5);
  InsnType Imm = 0;
  bool HasRs = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1892-1906
```cpp
  if (Rt == 0)
    return MCDisassembler::Fail;
  else if (Rs == 0) {
    MI.setOpcode(Mips::BLEZALC_MMR6);
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  }
  else if (Rs == Rt) {
    MI.setOpcode(Mips::BGEZALC_MMR6);
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 2 + 4;
  }
  else {
    HasRs = true;
    MI.setOpcode(Mips::BGEUC_MMR6);
    Imm = SignExtend64(fieldFromInstruction(insn, 0, 16), 16) * 4 + 4;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1908-1912
```cpp
  if (HasRs)
    MI.addOperand(
    MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rs)));
  MI.addOperand(
    MCOperand::createReg(getReg(Decoder, Mips::GPR32RegClassID, Rt)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1914-1914
```cpp
  MI.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1916-1917
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1919-1926
```cpp
// This instruction does not have a working decoder, and needs to be
// fixed. This "fixme" function was introduced to keep the backend compiling,
// while making changes to tablegen code.
static DecodeStatus DecodeFIXMEInstruction(MCInst &Inst, unsigned Insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  return MCDisassembler::Fail;
}
```
- EN: Implements `DecodeFIXMEInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeFIXMEInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1928-1928
```cpp
#include "MipsGenDisassemblerTables.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 1930-1939
```cpp
/// Read two bytes from the ArrayRef and return 16 bit halfword sorted
/// according to the given endianness.
static DecodeStatus readInstruction16(ArrayRef<uint8_t> Bytes, uint64_t Address,
                                      uint64_t &Size, uint32_t &Insn,
                                      bool IsBigEndian) {
  // We want to read exactly 2 Bytes of data.
  if (Bytes.size() < 2) {
    Size = 0;
    return MCDisassembler::Fail;
  }
```
- EN: Implements `readInstruction16`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `readInstruction16`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1941-1945
```cpp
  if (IsBigEndian) {
    Insn = (Bytes[0] << 8) | Bytes[1];
  } else {
    Insn = (Bytes[1] << 8) | Bytes[0];
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1947-1948
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1950-1959
```cpp
/// Read four bytes from the ArrayRef and return 32 bit word sorted
/// according to the given endianness.
static DecodeStatus readInstruction32(ArrayRef<uint8_t> Bytes, uint64_t Address,
                                      uint64_t &Size, uint32_t &Insn,
                                      bool IsBigEndian, bool IsMicroMips) {
  // We want to read exactly 4 Bytes of data.
  if (Bytes.size() < 4) {
    Size = 0;
    return MCDisassembler::Fail;
  }
```
- EN: Implements `readInstruction32`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `readInstruction32`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1961-1967
```cpp
  // High 16 bits of a 32-bit microMIPS instruction (where the opcode is)
  // always precede the low 16 bits in the instruction stream (that is, they
  // are placed at lower addresses in the instruction stream).
  //
  // microMIPS byte ordering:
  //   Big-endian:    0 | 1 | 2 | 3
  //   Little-endian: 1 | 0 | 3 | 2
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1969-1981
```cpp
  if (IsBigEndian) {
    // Encoded as a big-endian 32-bit word in the stream.
    Insn =
        (Bytes[3] << 0) | (Bytes[2] << 8) | (Bytes[1] << 16) | (Bytes[0] << 24);
  } else {
    if (IsMicroMips) {
      Insn = (Bytes[2] << 0) | (Bytes[3] << 8) | (Bytes[0] << 16) |
             (Bytes[1] << 24);
    } else {
      Insn = (Bytes[0] << 0) | (Bytes[1] << 8) | (Bytes[2] << 16) |
             (Bytes[3] << 24);
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1983-1984
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1986-1992
```cpp
DecodeStatus MipsDisassembler::getInstruction(MCInst &Instr, uint64_t &Size,
                                              ArrayRef<uint8_t> Bytes,
                                              uint64_t Address,
                                              raw_ostream &CStream) const {
  uint32_t Insn;
  DecodeStatus Result;
  Size = 0;
```
- EN: Implements `MipsDisassembler::getInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsDisassembler::getInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1994-1997
```cpp
  if (IsMicroMips) {
    Result = readInstruction16(Bytes, Address, Size, Insn, IsBigEndian);
    if (Result == MCDisassembler::Fail)
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1999-2010
```cpp
    if (hasMips32r6()) {
      LLVM_DEBUG(
          dbgs() << "Trying MicroMipsR616 table (16-bit instructions):\n");
      // Calling the auto-generated decoder function for microMIPS32R6
      // 16-bit instructions.
      Result = decodeInstruction(DecoderTableMicroMipsR616, Instr, Insn,
                                 Address, this, STI);
      if (Result != MCDisassembler::Fail) {
        Size = 2;
        return Result;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2012-2020
```cpp
    LLVM_DEBUG(dbgs() << "Trying MicroMips16 table (16-bit instructions):\n");
    // Calling the auto-generated decoder function for microMIPS 16-bit
    // instructions.
    Result = decodeInstruction(DecoderTableMicroMips16, Instr, Insn, Address,
                               this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 2;
      return Result;
    }
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2022-2024
```cpp
    Result = readInstruction32(Bytes, Address, Size, Insn, IsBigEndian, true);
    if (Result == MCDisassembler::Fail)
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2026-2036
```cpp
    if (hasMips32r6()) {
      LLVM_DEBUG(
          dbgs() << "Trying MicroMips32r632 table (32-bit instructions):\n");
      // Calling the auto-generated decoder function.
      Result = decodeInstruction(DecoderTableMicroMipsR632, Instr, Insn,
                                 Address, this, STI);
      if (Result != MCDisassembler::Fail) {
        Size = 4;
        return Result;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2038-2045
```cpp
    LLVM_DEBUG(dbgs() << "Trying MicroMips32 table (32-bit instructions):\n");
    // Calling the auto-generated decoder function.
    Result = decodeInstruction(DecoderTableMicroMips32, Instr, Insn, Address,
                               this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      return Result;
    }
```
- EN: Implements `LLVM_DEBUG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVM_DEBUG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2047-2055
```cpp
    if (isFP64()) {
      LLVM_DEBUG(dbgs() << "Trying MicroMipsFP64 table (32-bit opcodes):\n");
      Result = decodeInstruction(DecoderTableMicroMipsFP6432, Instr, Insn,
                                 Address, this, STI);
      if (Result != MCDisassembler::Fail) {
        Size = 4;
        return Result;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2057-2064
```cpp
    // This is an invalid instruction. Claim that the Size is 2 bytes. Since
    // microMIPS instructions have a minimum alignment of 2, the next 2 bytes
    // could form a valid instruction. The two bytes we rejected as an
    // instruction could have actually beeen an inline constant pool that is
    // unconditionally branched over.
    Size = 2;
    return MCDisassembler::Fail;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2066-2071
```cpp
  // Attempt to read the instruction so that we can attempt to decode it. If
  // the buffer is not 4 bytes long, let the higher level logic figure out
  // what to do with a size of zero and MCDisassembler::Fail.
  Result = readInstruction32(Bytes, Address, Size, Insn, IsBigEndian, false);
  if (Result == MCDisassembler::Fail)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2073-2074
```cpp
  // The only instruction size for standard encoded MIPS.
  Size = 4;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2076-2082
```cpp
  if (hasCOP3()) {
    LLVM_DEBUG(dbgs() << "Trying COP3_ table (32-bit opcodes):\n");
    Result =
        decodeInstruction(DecoderTableCOP3_32, Instr, Insn, Address, this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2084-2091
```cpp
  if (hasMips32r6() && isGP64()) {
    LLVM_DEBUG(
        dbgs() << "Trying Mips32r6_64r6 (GPR64) table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableMips32r6_64r6_GP6432, Instr, Insn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2093-2100
```cpp
  if (hasMips32r6() && isPTR64()) {
    LLVM_DEBUG(
        dbgs() << "Trying Mips32r6_64r6 (PTR64) table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableMips32r6_64r6_PTR6432, Instr, Insn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2102-2108
```cpp
  if (hasMips32r6()) {
    LLVM_DEBUG(dbgs() << "Trying Mips32r6_64r6 table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableMips32r6_64r632, Instr, Insn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2110-2117
```cpp
  if (hasMips2() && isPTR64()) {
    LLVM_DEBUG(
        dbgs() << "Trying Mips32r6_64r6 (PTR64) table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableMips32_64_PTR6432, Instr, Insn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2119-2125
```cpp
  if (hasCnMips()) {
    LLVM_DEBUG(dbgs() << "Trying CnMips table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableCnMips32, Instr, Insn, Address, this,
                               STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2127-2133
```cpp
  if (hasCnMipsP()) {
    LLVM_DEBUG(dbgs() << "Trying CnMipsP table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableCnMipsP32, Instr, Insn, Address,
                               this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2135-2141
```cpp
  if (isGP64()) {
    LLVM_DEBUG(dbgs() << "Trying Mips64 (GPR64) table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableMips6432, Instr, Insn, Address, this,
                               STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2143-2150
```cpp
  if (isFP64()) {
    LLVM_DEBUG(
        dbgs() << "Trying MipsFP64 (64 bit FPU) table (32-bit opcodes):\n");
    Result = decodeInstruction(DecoderTableMipsFP6432, Instr, Insn, Address,
                               this, STI);
    if (Result != MCDisassembler::Fail)
      return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2152-2157
```cpp
  LLVM_DEBUG(dbgs() << "Trying Mips table (32-bit opcodes):\n");
  // Calling the auto-generated decoder function.
  Result =
      decodeInstruction(DecoderTableMips32, Instr, Insn, Address, this, STI);
  if (Result != MCDisassembler::Fail)
    return Result;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2159-2160
```cpp
  return MCDisassembler::Fail;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: instruction decoding and disassembly.
  - CN: 核心职责：指令解码与反汇编。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/MipsMCTargetDesc.h`, `TargetInfo/MipsTargetInfo.h`, `MipsGenDisassemblerTables.inc`.
  - CN: 后端本地头文件：`MCTargetDesc/MipsMCTargetDesc.h`, `TargetInfo/MipsTargetInfo.h`, `MipsGenDisassemblerTables.inc`。
- EN: LLVM infrastructure headers: `llvm/ADT/ArrayRef.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h` ... (+6 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/ArrayRef.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h` ... (+6 more)。
- EN: Standard/system headers: `cassert`, `cstdint`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
