# ARMDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/Disassembler/ARMDisassembler.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMDisassembler` for the ARM backend, focusing on instruction decoding and disassembly.
- 用途 (CN): 实现 ARM 后端中的 `ARMDisassembler`，重点处理指令解码与反汇编。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- ARMDisassembler.cpp - Disassembler for ARM/Thumb ISA ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-23
```cpp
#include "ARMBaseInstrInfo.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "MCTargetDesc/ARMMCTargetDesc.h"
#include "TargetInfo/ARMTargetInfo.h"
#include "Utils/ARMBaseInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDecoder.h"
#include "llvm/MC/MCDecoderOps.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-32
```cpp
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/SubtargetFeature.h"
#include <algorithm>
#include <cassert>
#include <cstdint>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 34-35
```cpp
using namespace llvm;
using namespace llvm::MCD;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 37-37
```cpp
#define DEBUG_TYPE "arm-disassembler"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 39-39
```cpp
using DecodeStatus = MCDisassembler::DecodeStatus;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 41-41
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 43-52
```cpp
// Handles the condition code status of instructions in IT blocks
class ITStatus {
public:
  // Returns the condition code for instruction in IT block
  unsigned getITCC() {
    unsigned CC = ARMCC::AL;
    if (instrInITBlock())
      CC = ITStates.back();
    return CC;
  }
```
- EN: Declares `ITStatus`, packaging target-specific state and APIs around `ARMDisassembler`.
- CN: 这里声明 `ITStatus`，把与 `ARMDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 54-55
```cpp
  // Advances the IT block state to the next T or E
  void advanceITState() { ITStates.pop_back(); }
```
- EN: Implements `advanceITState`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `advanceITState`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-58
```cpp
  // Returns true if the current instruction is in an IT block
  bool instrInITBlock() { return !ITStates.empty(); }
```
- EN: Implements `instrInITBlock`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `instrInITBlock`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 60-61
```cpp
  // Returns true if current instruction is the last instruction in an IT block
  bool instrLastInITBlock() { return ITStates.size() == 1; }
```
- EN: Implements `instrLastInITBlock`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `instrLastInITBlock`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-77
```cpp
  // Called when decoding an IT instruction. Sets the IT state for
  // the following instructions that for the IT block. Firstcond
  // corresponds to the field in the IT instruction encoding; Mask
  // is in the MCOperand format in which 1 means 'else' and 0 'then'.
  void setITState(char Firstcond, char Mask) {
    // (3 - the number of trailing zeros) is the number of then / else.
    unsigned NumTZ = llvm::countr_zero<uint8_t>(Mask);
    unsigned char CCBits = static_cast<unsigned char>(Firstcond & 0xf);
    assert(NumTZ <= 3 && "Invalid IT mask!");
    // push condition codes onto the stack the correct order for the pops
    for (unsigned Pos = NumTZ + 1; Pos <= 3; ++Pos) {
      unsigned Else = (Mask >> Pos) & 1;
      ITStates.push_back(CCBits ^ Else);
    }
    ITStates.push_back(CCBits);
```
- EN: Implements `setITState`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setITState`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 78-78
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 80-82
```cpp
private:
  std::vector<unsigned char> ITStates;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 84-91
```cpp
class VPTStatus {
public:
  unsigned getVPTPred() {
    unsigned Pred = ARMVCC::None;
    if (instrInVPTBlock())
      Pred = VPTStates.back();
    return Pred;
  }
```
- EN: Declares `VPTStatus`, packaging target-specific state and APIs around `ARMDisassembler`.
- CN: 这里声明 `VPTStatus`，把与 `ARMDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 93-93
```cpp
  void advanceVPTState() { VPTStates.pop_back(); }
```
- EN: Implements `advanceVPTState`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `advanceVPTState`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 95-95
```cpp
  bool instrInVPTBlock() { return !VPTStates.empty(); }
```
- EN: Implements `instrInVPTBlock`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `instrInVPTBlock`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 97-97
```cpp
  bool instrLastInVPTBlock() { return VPTStates.size() == 1; }
```
- EN: Implements `instrLastInVPTBlock`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `instrLastInVPTBlock`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 99-112
```cpp
  void setVPTState(char Mask) {
    // (3 - the number of trailing zeros) is the number of then / else.
    unsigned NumTZ = llvm::countr_zero<uint8_t>(Mask);
    assert(NumTZ <= 3 && "Invalid VPT mask!");
    // push predicates onto the stack the correct order for the pops
    for (unsigned Pos = NumTZ + 1; Pos <= 3; ++Pos) {
      bool T = ((Mask >> Pos) & 1) == 0;
      if (T)
        VPTStates.push_back(ARMVCC::Then);
      else
        VPTStates.push_back(ARMVCC::Else);
    }
    VPTStates.push_back(ARMVCC::Then);
  }
```
- EN: Implements `setVPTState`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setVPTState`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 114-116
```cpp
private:
  SmallVector<unsigned char, 4> VPTStates;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 118-123
```cpp
/// ARM disassembler for all ARM platforms.
class ARMDisassembler : public MCDisassembler {
public:
  std::unique_ptr<const MCInstrInfo> MCII;
  mutable ITStatus ITBlock;
  mutable VPTStatus VPTBlock;
```
- EN: Declares `ARMDisassembler`, packaging target-specific state and APIs around `ARMDisassembler`.
- CN: 这里声明 `ARMDisassembler`，把与 `ARMDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 125-131
```cpp
  ARMDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
                  const MCInstrInfo *MCII)
      : MCDisassembler(STI, Ctx), MCII(MCII) {
        InstructionEndianness = STI.hasFeature(ARM::ModeBigEndianInstructions)
                                    ? llvm::endianness::big
                                    : llvm::endianness::little;
  }
```
- EN: Implements `ARMDisassembler`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 133-133
```cpp
  ~ARMDisassembler() override = default;
```
- EN: Declares `~ARMDisassembler`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `~ARMDisassembler`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 135-137
```cpp
  DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
                              ArrayRef<uint8_t> Bytes, uint64_t Address,
                              raw_ostream &CStream) const override;
```
- EN: Declares `getInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 139-140
```cpp
  uint64_t suggestBytesToSkip(ArrayRef<uint8_t> Bytes,
                              uint64_t Address) const override;
```
- EN: Declares `suggestBytesToSkip`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `suggestBytesToSkip`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 142-145
```cpp
private:
  DecodeStatus getARMInstruction(MCInst &Instr, uint64_t &Size,
                                 ArrayRef<uint8_t> Bytes, uint64_t Address,
                                 raw_ostream &CStream) const;
```
- EN: Declares `getARMInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getARMInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 147-149
```cpp
  DecodeStatus getThumbInstruction(MCInst &Instr, uint64_t &Size,
                                   ArrayRef<uint8_t> Bytes, uint64_t Address,
                                   raw_ostream &CStream) const;
```
- EN: Declares `getThumbInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getThumbInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 151-153
```cpp
  bool isVectorPredicable(const MCInst &MI) const;
  DecodeStatus checkThumbPredicate(MCInst &) const;
  void UpdateThumbPredicate(DecodeStatus &S, MCInst &MI) const;
```
- EN: Declares `isVectorPredicable`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isVectorPredicable`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 155-156
```cpp
  llvm::endianness InstructionEndianness;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-158
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 160-162
```cpp
typedef DecodeStatus OperandDecoder(MCInst &Inst, unsigned Val,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder);
```
- EN: Declares `OperandDecoder`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `OperandDecoder`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 164-178
```cpp
/// tryAddingSymbolicOperand - trys to add a symbolic operand in place of the
/// immediate Value in the MCInst.  The immediate Value has had any PC
/// adjustment made by the caller.  If the instruction is a branch instruction
/// then isBranch is true, else false.  If the getOpInfo() function was set as
/// part of the setupForSymbolicDisassembly() call then that function is called
/// to get any symbolic information at the Address for this instruction.  If
/// that returns non-zero then the symbolic information it returns is used to
/// create an MCExpr and that is added as an operand to the MCInst.  If
/// getOpInfo() returns zero and isBranch is true then a symbol look up for
/// Value is done and if a symbol is found an MCExpr is created with that, else
/// an MCExpr with Value is created.  This function returns true if it adds an
/// operand to the MCInst and false otherwise.
static bool tryAddingSymbolicOperand(uint64_t Address, int32_t Value,
                                     bool isBranch, uint64_t InstSize,
                                     MCInst &MI,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 179-184
```cpp
                                     const MCDisassembler *Decoder) {
  // FIXME: Does it make sense for value to be negative?
  return Decoder->tryAddingSymbolicOperand(MI, (uint32_t)Value, Address,
                                           isBranch, /*Offset=*/0, /*OpSize=*/0,
                                           InstSize);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 186-198
```cpp
/// tryAddingPcLoadReferenceComment - trys to add a comment as to what is being
/// referenced by a load instruction with the base register that is the Pc.
/// These can often be values in a literal pool near the Address of the
/// instruction.  The Address of the instruction and its immediate Value are
/// used as a possible literal pool entry.  The SymbolLookUp call back will
/// return the name of a symbol referenced by the literal pool's entry if
/// the referenced address is that of a symbol.  Or it will return a pointer to
/// a literal 'C' string if the referenced address of the literal pool's entry
/// is an address into a section with 'C' string literals.
static void tryAddingPcLoadReferenceComment(uint64_t Address, int Value,
                                            const MCDisassembler *Decoder) {
  Decoder->tryAddingPcLoadReferenceComment(Value, Address);
}
```
- EN: Implements `tryAddingPcLoadReferenceComment`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `tryAddingPcLoadReferenceComment`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 200-200
```cpp
// Register class decoding functions.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 202-207
```cpp
static const uint16_t GPRDecoderTable[] = {
  ARM::R0, ARM::R1, ARM::R2, ARM::R3,
  ARM::R4, ARM::R5, ARM::R6, ARM::R7,
  ARM::R8, ARM::R9, ARM::R10, ARM::R11,
  ARM::R12, ARM::SP, ARM::LR, ARM::PC
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-214
```cpp
static const uint16_t CLRMGPRDecoderTable[] = {
  ARM::R0, ARM::R1, ARM::R2, ARM::R3,
  ARM::R4, ARM::R5, ARM::R6, ARM::R7,
  ARM::R8, ARM::R9, ARM::R10, ARM::R11,
  ARM::R12, 0, ARM::LR, ARM::APSR
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 216-220
```cpp
static DecodeStatus DecodeGPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeGPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 222-225
```cpp
  unsigned Register = GPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 227-231
```cpp
static DecodeStatus DecodeCLRMGPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeCLRMGPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCLRMGPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 233-235
```cpp
  unsigned Register = CLRMGPRDecoderTable[RegNo];
  if (Register == 0)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 237-239
```cpp
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 241-244
```cpp
static DecodeStatus DecodeGPRnopcRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeGPRnopcRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRnopcRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 246-247
```cpp
  if (RegNo == 15)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 249-249
```cpp
  Check(S, DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder));
```
- EN: Declares `Check`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Check`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 251-252
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 254-257
```cpp
static DecodeStatus DecodeGPRnospRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeGPRnospRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRnospRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 259-260
```cpp
  if (RegNo == 13)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 262-262
```cpp
  Check(S, DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder));
```
- EN: Declares `Check`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Check`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 264-265
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 267-270
```cpp
static DecodeStatus
DecodeGPRwithAPSRRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeGPRwithAPSRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRwithAPSRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 272-276
```cpp
  if (RegNo == 15)
  {
    Inst.addOperand(MCOperand::createReg(ARM::APSR_NZCV));
    return MCDisassembler::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 278-280
```cpp
  Check(S, DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 282-285
```cpp
static DecodeStatus
DecodeGPRwithZRRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeGPRwithZRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRwithZRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 287-291
```cpp
  if (RegNo == 15)
  {
    Inst.addOperand(MCOperand::createReg(ARM::ZR));
    return MCDisassembler::Success;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 293-294
```cpp
  if (RegNo == 13)
    Check(S, MCDisassembler::SoftFail);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 296-298
```cpp
  Check(S, DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-308
```cpp
static DecodeStatus
DecodeGPRwithZRnospRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  if (RegNo == 13)
    return MCDisassembler::Fail;
  Check(S, DecodeGPRwithZRRegisterClass(Inst, RegNo, Address, Decoder));
  return S;
}
```
- EN: Implements `DecodeGPRwithZRnospRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRwithZRnospRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 310-316
```cpp
static DecodeStatus DecodetGPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
  return DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Implements `DecodetGPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodetGPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 318-321
```cpp
static const uint16_t GPRPairDecoderTable[] = {
  ARM::R0_R1, ARM::R2_R3,   ARM::R4_R5,  ARM::R6_R7,
  ARM::R8_R9, ARM::R10_R11, ARM::R12_SP
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 323-326
```cpp
static DecodeStatus DecodeGPRPairRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeGPRPairRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRPairRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 328-331
```cpp
  // According to the Arm ARM RegNo = 14 is undefined, but we return fail
  // rather than SoftFail as there is no GPRPair table entry for index 7.
  if (RegNo > 13)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 333-334
```cpp
  if (RegNo & 1)
     S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 336-339
```cpp
  unsigned RegisterPair = GPRPairDecoderTable[RegNo/2];
  Inst.addOperand(MCOperand::createReg(RegisterPair));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 341-345
```cpp
static DecodeStatus
DecodeGPRPairnospRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                               const MCDisassembler *Decoder) {
  if (RegNo > 13)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeGPRPairnospRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRPairnospRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 347-348
```cpp
  unsigned RegisterPair = GPRPairDecoderTable[RegNo/2];
  Inst.addOperand(MCOperand::createReg(RegisterPair));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 350-353
```cpp
  if ((RegNo & 1) || RegNo > 10)
     return MCDisassembler::SoftFail;
  return MCDisassembler::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 355-359
```cpp
static DecodeStatus DecodeGPRspRegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo != 13)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeGPRspRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRspRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 361-364
```cpp
  unsigned Register = GPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 366-380
```cpp
static DecodeStatus DecodetcGPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  unsigned Register = 0;
  switch (RegNo) {
    case 0:
      Register = ARM::R0;
      break;
    case 1:
      Register = ARM::R1;
      break;
    case 2:
      Register = ARM::R2;
      break;
    case 3:
```
- EN: Implements `DecodetcGPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodetcGPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 381-391
```cpp
      Register = ARM::R3;
      break;
    case 9:
      Register = ARM::R9;
      break;
    case 12:
      Register = ARM::R12;
      break;
    default:
      return MCDisassembler::Fail;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 393-395
```cpp
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 397-400
```cpp
static DecodeStatus DecoderGPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecoderGPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecoderGPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-403
```cpp
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 405-406
```cpp
  if ((RegNo == 13 && !featureBits[ARM::HasV8Ops]) || RegNo == 15)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 408-410
```cpp
  Check(S, DecodeGPRRegisterClass(Inst, RegNo, Address, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 412-421
```cpp
static const MCPhysReg SPRDecoderTable[] = {
     ARM::S0,  ARM::S1,  ARM::S2,  ARM::S3,
     ARM::S4,  ARM::S5,  ARM::S6,  ARM::S7,
     ARM::S8,  ARM::S9, ARM::S10, ARM::S11,
    ARM::S12, ARM::S13, ARM::S14, ARM::S15,
    ARM::S16, ARM::S17, ARM::S18, ARM::S19,
    ARM::S20, ARM::S21, ARM::S22, ARM::S23,
    ARM::S24, ARM::S25, ARM::S26, ARM::S27,
    ARM::S28, ARM::S29, ARM::S30, ARM::S31
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 423-427
```cpp
static DecodeStatus DecodeSPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 31)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeSPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 429-432
```cpp
  unsigned Register = SPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 434-438
```cpp
static DecodeStatus DecodeHPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  return DecodeSPRRegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Implements `DecodeHPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeHPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 440-449
```cpp
static const MCPhysReg DPRDecoderTable[] = {
     ARM::D0,  ARM::D1,  ARM::D2,  ARM::D3,
     ARM::D4,  ARM::D5,  ARM::D6,  ARM::D7,
     ARM::D8,  ARM::D9, ARM::D10, ARM::D11,
    ARM::D12, ARM::D13, ARM::D14, ARM::D15,
    ARM::D16, ARM::D17, ARM::D18, ARM::D19,
    ARM::D20, ARM::D21, ARM::D22, ARM::D23,
    ARM::D24, ARM::D25, ARM::D26, ARM::D27,
    ARM::D28, ARM::D29, ARM::D30, ARM::D31
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 451-458
```cpp
// Does this instruction/subtarget permit use of registers d16-d31?
static bool PermitsD32(const MCInst &Inst, const MCDisassembler *Decoder) {
  if (Inst.getOpcode() == ARM::VSCCLRMD || Inst.getOpcode() == ARM::VSCCLRMS)
    return true;
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
  return featureBits[ARM::FeatureD32];
}
```
- EN: Implements `PermitsD32`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `PermitsD32`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 460-464
```cpp
static DecodeStatus DecodeDPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > (PermitsD32(Inst, Decoder) ? 31u : 15u))
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeDPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 466-469
```cpp
  unsigned Register = DPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 471-477
```cpp
static DecodeStatus DecodeDPR_8RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
  return DecodeDPRRegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Implements `DecodeDPR_8RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDPR_8RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 479-485
```cpp
static DecodeStatus DecodeSPR_8RegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;
  return DecodeSPRRegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Implements `DecodeSPR_8RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSPR_8RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 487-493
```cpp
static DecodeStatus DecodeDPR_VFP2RegisterClass(MCInst &Inst, unsigned RegNo,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if (RegNo > 15)
    return MCDisassembler::Fail;
  return DecodeDPRRegisterClass(Inst, RegNo, Address, Decoder);
}
```
- EN: Implements `DecodeDPR_VFP2RegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDPR_VFP2RegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 495-500
```cpp
static const MCPhysReg QPRDecoderTable[] = {
     ARM::Q0,  ARM::Q1,  ARM::Q2,  ARM::Q3,
     ARM::Q4,  ARM::Q5,  ARM::Q6,  ARM::Q7,
     ARM::Q8,  ARM::Q9, ARM::Q10, ARM::Q11,
    ARM::Q12, ARM::Q13, ARM::Q14, ARM::Q15
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 502-507
```cpp
static DecodeStatus DecodeQPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (RegNo > 31 || (RegNo & 1) != 0)
    return MCDisassembler::Fail;
  RegNo >>= 1;
```
- EN: Implements `DecodeQPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeQPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 509-512
```cpp
  unsigned Register = QPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 514-521
```cpp
static const MCPhysReg DPairDecoderTable[] = {
  ARM::Q0,  ARM::D1_D2,   ARM::Q1,  ARM::D3_D4,   ARM::Q2,  ARM::D5_D6,
  ARM::Q3,  ARM::D7_D8,   ARM::Q4,  ARM::D9_D10,  ARM::Q5,  ARM::D11_D12,
  ARM::Q6,  ARM::D13_D14, ARM::Q7,  ARM::D15_D16, ARM::Q8,  ARM::D17_D18,
  ARM::Q9,  ARM::D19_D20, ARM::Q10, ARM::D21_D22, ARM::Q11, ARM::D23_D24,
  ARM::Q12, ARM::D25_D26, ARM::Q13, ARM::D27_D28, ARM::Q14, ARM::D29_D30,
  ARM::Q15
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 523-527
```cpp
static DecodeStatus DecodeDPairRegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 30)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeDPairRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDPairRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 529-532
```cpp
  unsigned Register = DPairDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 534-543
```cpp
static const MCPhysReg DPairSpacedDecoderTable[] = {
  ARM::D0_D2,   ARM::D1_D3,   ARM::D2_D4,   ARM::D3_D5,
  ARM::D4_D6,   ARM::D5_D7,   ARM::D6_D8,   ARM::D7_D9,
  ARM::D8_D10,  ARM::D9_D11,  ARM::D10_D12, ARM::D11_D13,
  ARM::D12_D14, ARM::D13_D15, ARM::D14_D16, ARM::D15_D17,
  ARM::D16_D18, ARM::D17_D19, ARM::D18_D20, ARM::D19_D21,
  ARM::D20_D22, ARM::D21_D23, ARM::D22_D24, ARM::D23_D25,
  ARM::D24_D26, ARM::D25_D27, ARM::D26_D28, ARM::D27_D29,
  ARM::D28_D30, ARM::D29_D31
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 545-549
```cpp
static DecodeStatus
DecodeDPairSpacedRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
                               const MCDisassembler *Decoder) {
  if (RegNo > 29)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeDPairSpacedRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDPairSpacedRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 551-554
```cpp
  unsigned Register = DPairSpacedDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 556-560
```cpp
static DecodeStatus DecodeMQPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (RegNo > 7)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMQPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMQPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 562-565
```cpp
  unsigned Register = QPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 567-570
```cpp
static const MCPhysReg QQPRDecoderTable[] = {
     ARM::Q0_Q1,  ARM::Q1_Q2,  ARM::Q2_Q3,  ARM::Q3_Q4,
     ARM::Q4_Q5,  ARM::Q5_Q6,  ARM::Q6_Q7
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 572-576
```cpp
static DecodeStatus DecodeMQQPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  if (RegNo > 6)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMQQPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMQQPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 578-581
```cpp
  unsigned Register = QQPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 583-586
```cpp
static const MCPhysReg QQQQPRDecoderTable[] = {
     ARM::Q0_Q1_Q2_Q3,  ARM::Q1_Q2_Q3_Q4,  ARM::Q2_Q3_Q4_Q5,
     ARM::Q3_Q4_Q5_Q6,  ARM::Q4_Q5_Q6_Q7
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 588-592
```cpp
static DecodeStatus DecodeMQQQQPRRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if (RegNo > 4)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMQQQQPRRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMQQQQPRRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 594-597
```cpp
  unsigned Register = QQQQPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 599-599
```cpp
// Operand decoding functions.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 601-615
```cpp
static DecodeStatus DecodePredicateOperand(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  if (Val == 0xF) return MCDisassembler::Fail;
  // AL predicate is not allowed on Thumb1 branches.
  if (Inst.getOpcode() == ARM::tBcc && Val == 0xE)
    return MCDisassembler::Fail;
  const MCInstrInfo *MCII =
      static_cast<const ARMDisassembler *>(Decoder)->MCII.get();
  if (Val != ARMCC::AL && !MCII->get(Inst.getOpcode()).isPredicable())
    Check(S, MCDisassembler::SoftFail);
  Inst.addOperand(MCOperand::createImm(Val));
  if (Val == ARMCC::AL) {
    Inst.addOperand(MCOperand::createReg(ARM::NoRegister));
```
- EN: Implements `DecodePredicateOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePredicateOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 616-619
```cpp
  } else
    Inst.addOperand(MCOperand::createReg(ARM::CPSR));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 621-635
```cpp
// This overload is used to decode a `pred` operand that is not encoded into
// instruction. This is the case for almost all predicable Thumb instructions
// (exceptions are tBcc and t2Bcc). Some predicable Thumb instructions have ARM
// equivalents where they are not predicable (always executed). This function
// is used to decode `pred` operand of these ARM instructions, too.
static DecodeStatus DecodePredicateOperand(MCInst &Inst,
                                           const MCDisassembler *Decoder) {
  const auto *D = static_cast<const ARMDisassembler *>(Decoder);
  unsigned CC = ARMCC::AL;
  if (D->getSubtargetInfo().hasFeature(ARM::ModeThumb))
    CC = D->ITBlock.getITCC();
  MCRegister CondReg = CC == ARMCC::AL ? ARM::NoRegister : ARM::CPSR;
  Inst.addOperand(MCOperand::createImm(CC));
  Inst.addOperand(MCOperand::createReg(CondReg));
  return MCDisassembler::Success;
```
- EN: Implements `DecodePredicateOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePredicateOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 636-636
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 638-646
```cpp
static DecodeStatus DecodeCCOutOperand(MCInst &Inst, unsigned Val,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  if (Val)
    Inst.addOperand(MCOperand::createReg(ARM::CPSR));
  else
    Inst.addOperand(MCOperand::createReg(ARM::NoRegister));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeCCOutOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCCOutOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 648-657
```cpp
// This overload is called when decoding `s_cc_out` operand, which is not
// encoded into instruction. It is only used in Thumb1 instructions.
static DecodeStatus DecodeCCOutOperand(MCInst &Inst,
                                       const MCDisassembler *Decoder) {
  const auto *D = static_cast<const ARMDisassembler *>(Decoder);
  // Thumb1 instructions define CPSR unless they are inside an IT block.
  MCRegister CCR = D->ITBlock.instrInITBlock() ? ARM::NoRegister : ARM::CPSR;
  Inst.addOperand(MCOperand::createReg(CCR));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeCCOutOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCCOutOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 659-663
```cpp
static DecodeStatus DecodeVpredNOperand(MCInst &Inst,
                                        const MCDisassembler *Decoder) {
  const auto *D = static_cast<const ARMDisassembler *>(Decoder);
  unsigned VCC = D->VPTBlock.getVPTPred();
  MCRegister CondReg = VCC == ARMVCC::None ? ARM::NoRegister : ARM::P0;
```
- EN: Implements `DecodeVpredNOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVpredNOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 665-667
```cpp
  Inst.addOperand(MCOperand::createImm(VCC));             // $cond
  Inst.addOperand(MCOperand::createReg(CondReg));         // $cond_reg
  Inst.addOperand(MCOperand::createReg(ARM::NoRegister)); // $tp_reg
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 669-670
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 672-676
```cpp
static DecodeStatus DecodeVpredROperand(MCInst &Inst,
                                        const MCDisassembler *Decoder) {
  const auto *D = static_cast<const ARMDisassembler *>(Decoder);
  unsigned VCC = D->VPTBlock.getVPTPred();
  MCRegister CondReg = VCC == ARMVCC::None ? ARM::NoRegister : ARM::P0;
```
- EN: Implements `DecodeVpredROperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVpredROperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 678-680
```cpp
  Inst.addOperand(MCOperand::createImm(VCC));             // $cond
  Inst.addOperand(MCOperand::createReg(CondReg));         // $cond_reg
  Inst.addOperand(MCOperand::createReg(ARM::NoRegister)); // $tp_reg
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 682-688
```cpp
  // The last sub-operand ($inactive) is tied to an output operand.
  // The output operand has already been decoded, so just copy it.
  const MCInstrDesc &MCID = D->MCII->get(Inst.getOpcode());
  unsigned InactiveOpIdx = Inst.getNumOperands();
  int TiedOpIdx = MCID.getOperandConstraint(InactiveOpIdx, MCOI::TIED_TO);
  assert(TiedOpIdx >= 0 &&
         "Inactive register in vpred_r is not tied to an output!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 690-691
```cpp
  // Make a copy of the operand to ensure it is not invalidated when MI grows.
  Inst.addOperand(MCOperand(Inst.getOperand(TiedOpIdx))); // $inactive
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 693-694
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 696-699
```cpp
static DecodeStatus DecodeSORegImmOperand(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSORegImmOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSORegImmOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 701-703
```cpp
  unsigned Rm = fieldFromInstruction(Val, 0, 4);
  unsigned type = fieldFromInstruction(Val, 5, 2);
  unsigned imm = fieldFromInstruction(Val, 7, 5);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 705-707
```cpp
  // Register-immediate
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 709-723
```cpp
  ARM_AM::ShiftOpc Shift = ARM_AM::lsl;
  switch (type) {
    case 0:
      Shift = ARM_AM::lsl;
      break;
    case 1:
      Shift = ARM_AM::lsr;
      break;
    case 2:
      Shift = ARM_AM::asr;
      break;
    case 3:
      Shift = ARM_AM::ror;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 725-726
```cpp
  if (Shift == ARM_AM::ror && imm == 0)
    Shift = ARM_AM::rrx;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 728-729
```cpp
  unsigned Op = Shift | (imm << 3);
  Inst.addOperand(MCOperand::createImm(Op));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 731-732
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 734-737
```cpp
static DecodeStatus DecodeSORegRegOperand(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSORegRegOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSORegRegOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 739-741
```cpp
  unsigned Rm = fieldFromInstruction(Val, 0, 4);
  unsigned type = fieldFromInstruction(Val, 5, 2);
  unsigned Rs = fieldFromInstruction(Val, 8, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 743-747
```cpp
  // Register-register
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rs, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 749-763
```cpp
  ARM_AM::ShiftOpc Shift = ARM_AM::lsl;
  switch (type) {
    case 0:
      Shift = ARM_AM::lsl;
      break;
    case 1:
      Shift = ARM_AM::lsr;
      break;
    case 2:
      Shift = ARM_AM::asr;
      break;
    case 3:
      Shift = ARM_AM::ror;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 765-765
```cpp
  Inst.addOperand(MCOperand::createImm(Shift));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 767-768
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 770-773
```cpp
static DecodeStatus DecodeRegListOperand(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeRegListOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRegListOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 775-789
```cpp
  bool NeedDisjointWriteback = false;
  MCRegister WritebackReg;
  bool CLRM = false;
  switch (Inst.getOpcode()) {
  default:
    break;
  case ARM::LDMIA_UPD:
  case ARM::LDMDB_UPD:
  case ARM::LDMIB_UPD:
  case ARM::LDMDA_UPD:
  case ARM::t2LDMIA_UPD:
  case ARM::t2LDMDB_UPD:
  case ARM::t2STMIA_UPD:
  case ARM::t2STMDB_UPD:
    NeedDisjointWriteback = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 790-795
```cpp
    WritebackReg = Inst.getOperand(0).getReg();
    break;
  case ARM::t2CLRM:
    CLRM = true;
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 797-811
```cpp
  // Empty register lists are not allowed.
  if (Val == 0) return MCDisassembler::Fail;
  for (unsigned i = 0; i < 16; ++i) {
    if (Val & (1 << i)) {
      if (CLRM) {
        if (!Check(S, DecodeCLRMGPRRegisterClass(Inst, i, Address, Decoder))) {
          return MCDisassembler::Fail;
        }
      } else {
        if (!Check(S, DecodeGPRRegisterClass(Inst, i, Address, Decoder)))
          return MCDisassembler::Fail;
        // Writeback not allowed if Rn is in the target list.
        if (NeedDisjointWriteback && WritebackReg == Inst.end()[-1].getReg())
          Check(S, MCDisassembler::SoftFail);
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 812-813
```cpp
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 815-816
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 818-821
```cpp
static DecodeStatus DecodeSPRRegListOperand(MCInst &Inst, unsigned Val,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSPRRegListOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSPRRegListOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 823-824
```cpp
  unsigned Vd = fieldFromInstruction(Val, 8, 5);
  unsigned regs = fieldFromInstruction(Val, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 826-831
```cpp
  // In case of unpredictable encoding, tweak the operands.
  if (regs == 0 || (Vd + regs) > 32) {
    regs = Vd + regs > 32 ? 32 - Vd : regs;
    regs = std::max( 1u, regs);
    S = MCDisassembler::SoftFail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 833-838
```cpp
  if (!Check(S, DecodeSPRRegisterClass(Inst, Vd, Address, Decoder)))
    return MCDisassembler::Fail;
  for (unsigned i = 0; i < (regs - 1); ++i) {
    if (!Check(S, DecodeSPRRegisterClass(Inst, ++Vd, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 840-841
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 843-846
```cpp
static DecodeStatus DecodeDPRRegListOperand(MCInst &Inst, unsigned Val,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeDPRRegListOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDPRRegListOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 848-849
```cpp
  unsigned Vd = fieldFromInstruction(Val, 8, 5);
  unsigned regs = fieldFromInstruction(Val, 1, 7);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 851-858
```cpp
  // In case of unpredictable encoding, tweak the operands.
  unsigned MaxReg = PermitsD32(Inst, Decoder) ? 32 : 16;
  if (regs == 0 || (Vd + regs) > MaxReg) {
    regs = Vd + regs > MaxReg ? MaxReg - Vd : regs;
    regs = std::max( 1u, regs);
    regs = std::min(MaxReg, regs);
    S = MCDisassembler::SoftFail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 860-865
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Vd, Address, Decoder)))
      return MCDisassembler::Fail;
  for (unsigned i = 0; i < (regs - 1); ++i) {
    if (!Check(S, DecodeDPRRegisterClass(Inst, ++Vd, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 867-868
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 870-879
```cpp
static DecodeStatus DecodeBitfieldMaskOperand(MCInst &Inst, unsigned Val,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  // This operand encodes a mask of contiguous zeros between a specified MSB
  // and LSB.  To decode it, we create the mask of all bits MSB-and-lower,
  // the mask of all bits LSB-and-lower, and then xor them to create
  // the mask of that's all ones on [msb, lsb].  Finally we not it to
  // create the final mask.
  unsigned msb = fieldFromInstruction(Val, 5, 5);
  unsigned lsb = fieldFromInstruction(Val, 0, 5);
```
- EN: Implements `DecodeBitfieldMaskOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBitfieldMaskOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 881-888
```cpp
  DecodeStatus S = MCDisassembler::Success;
  if (lsb > msb) {
    Check(S, MCDisassembler::SoftFail);
    // The check above will cause the warning for the "potentially undefined
    // instruction encoding" but we can't build a bad MCOperand value here
    // with a lsb > msb or else printing the MCInst will cause a crash.
    lsb = msb;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 890-892
```cpp
  uint32_t msb_mask = 0xFFFFFFFF;
  if (msb != 31) msb_mask = (1U << (msb+1)) - 1;
  uint32_t lsb_mask = (1U << lsb) - 1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 894-896
```cpp
  Inst.addOperand(MCOperand::createImm(~(msb_mask ^ lsb_mask)));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 898-901
```cpp
static DecodeStatus DecodeCopMemInstruction(MCInst &Inst, unsigned Insn,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeCopMemInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCopMemInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 903-910
```cpp
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned CRd = fieldFromInstruction(Insn, 12, 4);
  unsigned coproc = fieldFromInstruction(Insn, 8, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 8);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned U = fieldFromInstruction(Insn, 23, 1);
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 912-926
```cpp
  switch (Inst.getOpcode()) {
    case ARM::LDC_OFFSET:
    case ARM::LDC_PRE:
    case ARM::LDC_POST:
    case ARM::LDC_OPTION:
    case ARM::LDCL_OFFSET:
    case ARM::LDCL_PRE:
    case ARM::LDCL_POST:
    case ARM::LDCL_OPTION:
    case ARM::STC_OFFSET:
    case ARM::STC_PRE:
    case ARM::STC_POST:
    case ARM::STC_OPTION:
    case ARM::STCL_OFFSET:
    case ARM::STCL_PRE:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 927-941
```cpp
    case ARM::STCL_POST:
    case ARM::STCL_OPTION:
    case ARM::t2LDC_OFFSET:
    case ARM::t2LDC_PRE:
    case ARM::t2LDC_POST:
    case ARM::t2LDC_OPTION:
    case ARM::t2LDCL_OFFSET:
    case ARM::t2LDCL_PRE:
    case ARM::t2LDCL_POST:
    case ARM::t2LDCL_OPTION:
    case ARM::t2STC_OFFSET:
    case ARM::t2STC_PRE:
    case ARM::t2STC_POST:
    case ARM::t2STC_OPTION:
    case ARM::t2STCL_OFFSET:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 942-956
```cpp
    case ARM::t2STCL_PRE:
    case ARM::t2STCL_POST:
    case ARM::t2STCL_OPTION:
    case ARM::t2LDC2_OFFSET:
    case ARM::t2LDC2L_OFFSET:
    case ARM::t2LDC2_PRE:
    case ARM::t2LDC2L_PRE:
    case ARM::t2STC2_OFFSET:
    case ARM::t2STC2L_OFFSET:
    case ARM::t2STC2_PRE:
    case ARM::t2STC2L_PRE:
    case ARM::LDC2_OFFSET:
    case ARM::LDC2L_OFFSET:
    case ARM::LDC2_PRE:
    case ARM::LDC2L_PRE:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 957-971
```cpp
    case ARM::STC2_OFFSET:
    case ARM::STC2L_OFFSET:
    case ARM::STC2_PRE:
    case ARM::STC2L_PRE:
    case ARM::t2LDC2_OPTION:
    case ARM::t2STC2_OPTION:
    case ARM::t2LDC2_POST:
    case ARM::t2LDC2L_POST:
    case ARM::t2STC2_POST:
    case ARM::t2STC2L_POST:
    case ARM::LDC2_POST:
    case ARM::LDC2L_POST:
    case ARM::STC2_POST:
    case ARM::STC2L_POST:
      if (coproc == 0xA || coproc == 0xB ||
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 972-979
```cpp
          (featureBits[ARM::HasV8_1MMainlineOps] &&
           (coproc == 0x8 || coproc == 0x9 || coproc == 0xA || coproc == 0xB ||
            coproc == 0xE || coproc == 0xF)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 981-982
```cpp
  if (featureBits[ARM::HasV8Ops] && (coproc != 14))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 984-987
```cpp
  Inst.addOperand(MCOperand::createImm(coproc));
  Inst.addOperand(MCOperand::createImm(CRd));
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 989-1003
```cpp
  switch (Inst.getOpcode()) {
    case ARM::t2LDC2_OFFSET:
    case ARM::t2LDC2L_OFFSET:
    case ARM::t2LDC2_PRE:
    case ARM::t2LDC2L_PRE:
    case ARM::t2STC2_OFFSET:
    case ARM::t2STC2L_OFFSET:
    case ARM::t2STC2_PRE:
    case ARM::t2STC2L_PRE:
    case ARM::LDC2_OFFSET:
    case ARM::LDC2L_OFFSET:
    case ARM::LDC2_PRE:
    case ARM::LDC2L_PRE:
    case ARM::STC2_OFFSET:
    case ARM::STC2L_OFFSET:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1004-1018
```cpp
    case ARM::STC2_PRE:
    case ARM::STC2L_PRE:
    case ARM::t2LDC_OFFSET:
    case ARM::t2LDCL_OFFSET:
    case ARM::t2LDC_PRE:
    case ARM::t2LDCL_PRE:
    case ARM::t2STC_OFFSET:
    case ARM::t2STCL_OFFSET:
    case ARM::t2STC_PRE:
    case ARM::t2STCL_PRE:
    case ARM::LDC_OFFSET:
    case ARM::LDCL_OFFSET:
    case ARM::LDC_PRE:
    case ARM::LDCL_PRE:
    case ARM::STC_OFFSET:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1019-1033
```cpp
    case ARM::STCL_OFFSET:
    case ARM::STC_PRE:
    case ARM::STCL_PRE:
      imm = ARM_AM::getAM5Opc(U ? ARM_AM::add : ARM_AM::sub, imm);
      Inst.addOperand(MCOperand::createImm(imm));
      break;
    case ARM::t2LDC2_POST:
    case ARM::t2LDC2L_POST:
    case ARM::t2STC2_POST:
    case ARM::t2STC2L_POST:
    case ARM::LDC2_POST:
    case ARM::LDC2L_POST:
    case ARM::STC2_POST:
    case ARM::STC2L_POST:
    case ARM::t2LDC_POST:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1034-1048
```cpp
    case ARM::t2LDCL_POST:
    case ARM::t2STC_POST:
    case ARM::t2STCL_POST:
    case ARM::LDC_POST:
    case ARM::LDCL_POST:
    case ARM::STC_POST:
    case ARM::STCL_POST:
      imm |= U << 8;
      [[fallthrough]];
    default:
      // The 'option' variant doesn't encode 'U' in the immediate since
      // the immediate is unsigned [0,255].
      Inst.addOperand(MCOperand::createImm(imm));
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1050-1064
```cpp
  switch (Inst.getOpcode()) {
    case ARM::LDC_OFFSET:
    case ARM::LDC_PRE:
    case ARM::LDC_POST:
    case ARM::LDC_OPTION:
    case ARM::LDCL_OFFSET:
    case ARM::LDCL_PRE:
    case ARM::LDCL_POST:
    case ARM::LDCL_OPTION:
    case ARM::STC_OFFSET:
    case ARM::STC_PRE:
    case ARM::STC_POST:
    case ARM::STC_OPTION:
    case ARM::STCL_OFFSET:
    case ARM::STCL_PRE:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1065-1079
```cpp
    case ARM::STCL_POST:
    case ARM::STCL_OPTION:
      if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    case ARM::t2LDC2L_OFFSET:
    case ARM::t2LDC2L_OPTION:
    case ARM::t2LDC2L_POST:
    case ARM::t2LDC2L_PRE:
    case ARM::t2LDC2_OFFSET:
    case ARM::t2LDC2_OPTION:
    case ARM::t2LDC2_POST:
    case ARM::t2LDC2_PRE:
    case ARM::t2LDCL_OFFSET:
    case ARM::t2LDCL_OPTION:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1080-1094
```cpp
    case ARM::t2LDCL_POST:
    case ARM::t2LDCL_PRE:
    case ARM::t2LDC_OFFSET:
    case ARM::t2LDC_OPTION:
    case ARM::t2LDC_POST:
    case ARM::t2LDC_PRE:
    case ARM::t2STC2L_OFFSET:
    case ARM::t2STC2L_OPTION:
    case ARM::t2STC2L_POST:
    case ARM::t2STC2L_PRE:
    case ARM::t2STC2_OFFSET:
    case ARM::t2STC2_OPTION:
    case ARM::t2STC2_POST:
    case ARM::t2STC2_PRE:
    case ARM::t2STCL_OFFSET:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1095-1106
```cpp
    case ARM::t2STCL_OPTION:
    case ARM::t2STCL_POST:
    case ARM::t2STCL_PRE:
    case ARM::t2STC_OFFSET:
    case ARM::t2STC_OPTION:
    case ARM::t2STC_POST:
    case ARM::t2STC_PRE:
      DecodePredicateOperand(Inst, Decoder);
      break;
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1108-1109
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1111-1114
```cpp
static DecodeStatus
DecodeAddrMode2IdxInstruction(MCInst &Inst, unsigned Insn, uint64_t Address,
                              const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeAddrMode2IdxInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrMode2IdxInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1116-1123
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 12);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned reg = fieldFromInstruction(Insn, 25, 1);
  unsigned P = fieldFromInstruction(Insn, 24, 1);
  unsigned W = fieldFromInstruction(Insn, 21, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1125-1139
```cpp
  // On stores, the writeback operand precedes Rt.
  switch (Inst.getOpcode()) {
    case ARM::STR_POST_IMM:
    case ARM::STR_POST_REG:
    case ARM::STRB_POST_IMM:
    case ARM::STRB_POST_REG:
    case ARM::STRT_POST_REG:
    case ARM::STRT_POST_IMM:
    case ARM::STRBT_POST_REG:
    case ARM::STRBT_POST_IMM:
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1140-1140
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1142-1143
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1145-1159
```cpp
  // On loads, the writeback operand comes after Rt.
  switch (Inst.getOpcode()) {
    case ARM::LDR_POST_IMM:
    case ARM::LDR_POST_REG:
    case ARM::LDRB_POST_IMM:
    case ARM::LDRB_POST_REG:
    case ARM::LDRBT_POST_REG:
    case ARM::LDRBT_POST_IMM:
    case ARM::LDRT_POST_REG:
    case ARM::LDRT_POST_IMM:
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1160-1160
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1162-1163
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1165-1167
```cpp
  ARM_AM::AddrOpc Op = ARM_AM::add;
  if (!fieldFromInstruction(Insn, 23, 1))
    Op = ARM_AM::sub;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1169-1174
```cpp
  bool writeback = (P == 0) || (W == 1);
  unsigned idx_mode = 0;
  if (P && writeback)
    idx_mode = ARMII::IndexModePre;
  else if (!P && writeback)
    idx_mode = ARMII::IndexModePost;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1176-1177
```cpp
  if (writeback && (Rn == 15 || Rn == Rt))
    S = MCDisassembler::SoftFail; // UNPREDICTABLE
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1179-1193
```cpp
  if (reg) {
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
    ARM_AM::ShiftOpc Opc = ARM_AM::lsl;
    switch( fieldFromInstruction(Insn, 5, 2)) {
      case 0:
        Opc = ARM_AM::lsl;
        break;
      case 1:
        Opc = ARM_AM::lsr;
        break;
      case 2:
        Opc = ARM_AM::asr;
        break;
      case 3:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1194-1202
```cpp
        Opc = ARM_AM::ror;
        break;
      default:
        return MCDisassembler::Fail;
    }
    unsigned amt = fieldFromInstruction(Insn, 7, 5);
    if (Opc == ARM_AM::ror && amt == 0)
      Opc = ARM_AM::rrx;
    unsigned imm = ARM_AM::getAM2Opc(Op, amt, Opc, idx_mode);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1204-1209
```cpp
    Inst.addOperand(MCOperand::createImm(imm));
  } else {
    Inst.addOperand(MCOperand::createReg(0));
    unsigned tmp = ARM_AM::getAM2Opc(Op, imm, ARM_AM::lsl, idx_mode);
    Inst.addOperand(MCOperand::createImm(tmp));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1211-1212
```cpp
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1214-1215
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1217-1220
```cpp
static DecodeStatus DecodeSORegMemOperand(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSORegMemOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSORegMemOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1222-1226
```cpp
  unsigned Rn = fieldFromInstruction(Val, 13, 4);
  unsigned Rm = fieldFromInstruction(Val,  0, 4);
  unsigned type = fieldFromInstruction(Val, 5, 2);
  unsigned imm = fieldFromInstruction(Val, 7, 5);
  unsigned U = fieldFromInstruction(Val, 12, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1228-1242
```cpp
  ARM_AM::ShiftOpc ShOp = ARM_AM::lsl;
  switch (type) {
    case 0:
      ShOp = ARM_AM::lsl;
      break;
    case 1:
      ShOp = ARM_AM::lsr;
      break;
    case 2:
      ShOp = ARM_AM::asr;
      break;
    case 3:
      ShOp = ARM_AM::ror;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1244-1245
```cpp
  if (ShOp == ARM_AM::ror && imm == 0)
    ShOp = ARM_AM::rrx;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1247-1256
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  unsigned shift;
  if (U)
    shift = ARM_AM::getAM2Opc(ARM_AM::add, imm, ShOp);
  else
    shift = ARM_AM::getAM2Opc(ARM_AM::sub, imm, ShOp);
  Inst.addOperand(MCOperand::createImm(shift));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1258-1259
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1261-1265
```cpp
static DecodeStatus DecodeTSBInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  if (Inst.getOpcode() != ARM::TSB && Inst.getOpcode() != ARM::t2TSB)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeTSBInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeTSBInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1267-1274
```cpp
  // The "csync" operand is not encoded into the "tsb" instruction (as this is
  // the only available operand), but LLVM expects the instruction to have one
  // operand, so we need to add the csync when decoding.
  Inst.addOperand(MCOperand::createImm(ARM_TSB::CSYNC));
  if (Inst.getOpcode() == ARM::t2TSB)
    DecodePredicateOperand(Inst, Decoder);
  return MCDisassembler::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1276-1279
```cpp
static DecodeStatus DecodeAddrMode3Instruction(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeAddrMode3Instruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrMode3Instruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1281-1290
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned type = fieldFromInstruction(Insn, 22, 1);
  unsigned imm = fieldFromInstruction(Insn, 8, 4);
  unsigned U = ((~fieldFromInstruction(Insn, 23, 1)) & 1) << 8;
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned W = fieldFromInstruction(Insn, 21, 1);
  unsigned P = fieldFromInstruction(Insn, 24, 1);
  unsigned Rt2 = Rt + 1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1292-1292
```cpp
  bool writeback = (W == 1) | (P == 0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1294-1308
```cpp
  // For {LD,ST}RD, Rt must be even, else undefined.
  switch (Inst.getOpcode()) {
    case ARM::STRD:
    case ARM::STRD_PRE:
    case ARM::STRD_POST:
    case ARM::LDRD:
    case ARM::LDRD_PRE:
    case ARM::LDRD_POST:
      if (Rt & 0x1) S = MCDisassembler::SoftFail;
      break;
    default:
      break;
  }
  switch (Inst.getOpcode()) {
    case ARM::STRD:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1309-1312
```cpp
    case ARM::STRD_PRE:
    case ARM::STRD_POST:
      if (P == 0 && W == 1)
        S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1314-1328
```cpp
      if (writeback && (Rn == 15 || Rn == Rt || Rn == Rt2))
        S = MCDisassembler::SoftFail;
      if (type && Rm == 15)
        S = MCDisassembler::SoftFail;
      if (Rt2 == 15)
        S = MCDisassembler::SoftFail;
      if (!type && fieldFromInstruction(Insn, 8, 4))
        S = MCDisassembler::SoftFail;
      break;
    case ARM::STRH:
    case ARM::STRH_PRE:
    case ARM::STRH_POST:
      if (Rt == 15)
        S = MCDisassembler::SoftFail;
      if (writeback && (Rn == 15 || Rn == Rt))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1329-1343
```cpp
        S = MCDisassembler::SoftFail;
      if (!type && Rm == 15)
        S = MCDisassembler::SoftFail;
      break;
    case ARM::LDRD:
    case ARM::LDRD_PRE:
    case ARM::LDRD_POST:
      if (type && Rn == 15) {
        if (Rt2 == 15)
          S = MCDisassembler::SoftFail;
        break;
      }
      if (P == 0 && W == 1)
        S = MCDisassembler::SoftFail;
      if (!type && (Rt2 == 15 || Rm == 15 || Rm == Rt || Rm == Rt2))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1344-1358
```cpp
        S = MCDisassembler::SoftFail;
      if (!type && writeback && Rn == 15)
        S = MCDisassembler::SoftFail;
      if (writeback && (Rn == Rt || Rn == Rt2))
        S = MCDisassembler::SoftFail;
      break;
    case ARM::LDRH:
    case ARM::LDRH_PRE:
    case ARM::LDRH_POST:
      if (type && Rn == 15) {
        if (Rt == 15)
          S = MCDisassembler::SoftFail;
        break;
      }
      if (Rt == 15)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1359-1373
```cpp
        S = MCDisassembler::SoftFail;
      if (!type && Rm == 15)
        S = MCDisassembler::SoftFail;
      if (!type && writeback && (Rn == 15 || Rn == Rt))
        S = MCDisassembler::SoftFail;
      break;
    case ARM::LDRSH:
    case ARM::LDRSH_PRE:
    case ARM::LDRSH_POST:
    case ARM::LDRSB:
    case ARM::LDRSB_PRE:
    case ARM::LDRSB_POST:
      if (type && Rn == 15) {
        if (Rt == 15)
          S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1374-1385
```cpp
        break;
      }
      if (type && (Rt == 15 || (writeback && Rn == Rt)))
        S = MCDisassembler::SoftFail;
      if (!type && (Rt == 15 || Rm == 15))
        S = MCDisassembler::SoftFail;
      if (!type && writeback && (Rn == 15 || Rn == Rt))
        S = MCDisassembler::SoftFail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1387-1391
```cpp
  if (writeback) { // Writeback
    if (P)
      U |= ARMII::IndexModePre << 9;
    else
      U |= ARMII::IndexModePost << 9;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1393-1407
```cpp
    // On stores, the writeback operand precedes Rt.
    switch (Inst.getOpcode()) {
    case ARM::STRD:
    case ARM::STRD_PRE:
    case ARM::STRD_POST:
    case ARM::STRH:
    case ARM::STRH_PRE:
    case ARM::STRH_POST:
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1409-1423
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  switch (Inst.getOpcode()) {
    case ARM::STRD:
    case ARM::STRD_PRE:
    case ARM::STRD_POST:
    case ARM::LDRD:
    case ARM::LDRD_PRE:
    case ARM::LDRD_POST:
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rt+1, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1425-1439
```cpp
  if (writeback) {
    // On loads, the writeback operand comes after Rt.
    switch (Inst.getOpcode()) {
    case ARM::LDRD:
    case ARM::LDRD_PRE:
    case ARM::LDRD_POST:
    case ARM::LDRH:
    case ARM::LDRH_PRE:
    case ARM::LDRH_POST:
    case ARM::LDRSH:
    case ARM::LDRSH_PRE:
    case ARM::LDRSH_POST:
    case ARM::LDRSB:
    case ARM::LDRSB_PRE:
    case ARM::LDRSB_POST:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1440-1448
```cpp
    case ARM::LDRHTr:
    case ARM::LDRSBTr:
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1450-1451
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1453-1460
```cpp
  if (type) {
    Inst.addOperand(MCOperand::createReg(0));
    Inst.addOperand(MCOperand::createImm(U | (imm << 4) | Rm));
  } else {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
    Inst.addOperand(MCOperand::createImm(U));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1462-1463
```cpp
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1465-1466
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1468-1471
```cpp
static DecodeStatus DecodeRFEInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeRFEInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRFEInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1473-1475
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1477-1478
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1480-1486
```cpp
static DecodeStatus DecodeCPSInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  unsigned imod = fieldFromInstruction(Insn, 18, 2);
  unsigned M = fieldFromInstruction(Insn, 17, 1);
  unsigned iflags = fieldFromInstruction(Insn, 6, 3);
  unsigned mode = fieldFromInstruction(Insn, 0, 5);
```
- EN: Implements `DecodeCPSInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCPSInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1488-1488
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1490-1495
```cpp
  // This decoder is called from multiple location that do not check
  // the full encoding is valid before they do.
  if (fieldFromInstruction(Insn, 5, 1) != 0 ||
      fieldFromInstruction(Insn, 16, 1) != 0 ||
      fieldFromInstruction(Insn, 20, 8) != 0x10)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1497-1500
```cpp
  // imod == '01' --> UNPREDICTABLE
  // NOTE: Even though this is technically UNPREDICTABLE, we choose to
  // return failure here.  The '01' imod value is unprintable, so there's
  // nothing useful we could do even if we returned UNPREDICTABLE.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1502-1502
```cpp
  if (imod == 1) return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1504-1518
```cpp
  if (imod && M) {
    Inst.setOpcode(ARM::CPS3p);
    Inst.addOperand(MCOperand::createImm(imod));
    Inst.addOperand(MCOperand::createImm(iflags));
    Inst.addOperand(MCOperand::createImm(mode));
  } else if (imod && !M) {
    Inst.setOpcode(ARM::CPS2p);
    Inst.addOperand(MCOperand::createImm(imod));
    Inst.addOperand(MCOperand::createImm(iflags));
    if (mode) S = MCDisassembler::SoftFail;
  } else if (!imod && M) {
    Inst.setOpcode(ARM::CPS1p);
    Inst.addOperand(MCOperand::createImm(mode));
    if (iflags) S = MCDisassembler::SoftFail;
  } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1519-1523
```cpp
    // imod == '00' && M == '0' --> UNPREDICTABLE
    Inst.setOpcode(ARM::CPS1p);
    Inst.addOperand(MCOperand::createImm(mode));
    S = MCDisassembler::SoftFail;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1525-1526
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1528-1531
```cpp
static DecodeStatus DecodeQADDInstruction(MCInst &Inst, unsigned Insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeQADDInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeQADDInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1533-1536
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1538-1539
```cpp
  if (pred == 0xF)
    return DecodeCPSInstruction(Inst, Insn, Address, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1541-1550
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1552-1556
```cpp
static DecodeStatus
DecodeMemMultipleWritebackInstruction(MCInst &Inst, unsigned Insn,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeMemMultipleWritebackInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemMultipleWritebackInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1558-1560
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned reglist = fieldFromInstruction(Insn, 0, 16);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1562-1576
```cpp
  if (pred == 0xF) {
    // Ambiguous with RFE and SRS
    switch (Inst.getOpcode()) {
      case ARM::LDMDA:
        Inst.setOpcode(ARM::RFEDA);
        break;
      case ARM::LDMDA_UPD:
        Inst.setOpcode(ARM::RFEDA_UPD);
        break;
      case ARM::LDMDB:
        Inst.setOpcode(ARM::RFEDB);
        break;
      case ARM::LDMDB_UPD:
        Inst.setOpcode(ARM::RFEDB_UPD);
        break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1577-1591
```cpp
      case ARM::LDMIA:
        Inst.setOpcode(ARM::RFEIA);
        break;
      case ARM::LDMIA_UPD:
        Inst.setOpcode(ARM::RFEIA_UPD);
        break;
      case ARM::LDMIB:
        Inst.setOpcode(ARM::RFEIB);
        break;
      case ARM::LDMIB_UPD:
        Inst.setOpcode(ARM::RFEIB_UPD);
        break;
      case ARM::STMDA:
        Inst.setOpcode(ARM::SRSDA);
        break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1592-1606
```cpp
      case ARM::STMDA_UPD:
        Inst.setOpcode(ARM::SRSDA_UPD);
        break;
      case ARM::STMDB:
        Inst.setOpcode(ARM::SRSDB);
        break;
      case ARM::STMDB_UPD:
        Inst.setOpcode(ARM::SRSDB_UPD);
        break;
      case ARM::STMIA:
        Inst.setOpcode(ARM::SRSIA);
        break;
      case ARM::STMIA_UPD:
        Inst.setOpcode(ARM::SRSIA_UPD);
        break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1607-1615
```cpp
      case ARM::STMIB:
        Inst.setOpcode(ARM::SRSIB);
        break;
      case ARM::STMIB_UPD:
        Inst.setOpcode(ARM::SRSIB_UPD);
        break;
      default:
        return MCDisassembler::Fail;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1617-1622
```cpp
    // For stores (which become SRS's, the only operand is the mode.
    if (fieldFromInstruction(Insn, 20, 1) == 0) {
      // Check SRS encoding constraints
      if (!(fieldFromInstruction(Insn, 22, 1) == 1 &&
            fieldFromInstruction(Insn, 20, 1) == 0))
        return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1624-1627
```cpp
      Inst.addOperand(
          MCOperand::createImm(fieldFromInstruction(Insn, 0, 4)));
      return S;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1629-1630
```cpp
    return DecodeRFEInstruction(Inst, Insn, Address, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1632-1639
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail; // Tied
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeRegListOperand(Inst, reglist, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1641-1642
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1644-1651
```cpp
// Check for UNPREDICTABLE predicated ESB instruction
static DecodeStatus DecodeHINTInstruction(MCInst &Inst, unsigned Insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned imm8 = fieldFromInstruction(Insn, 0, 8);
  const FeatureBitset &FeatureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Implements `DecodeHINTInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeHINTInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1653-1653
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1655-1655
```cpp
  Inst.addOperand(MCOperand::createImm(imm8));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1657-1658
```cpp
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1660-1663
```cpp
  // ESB is unpredictable if pred != AL. Without the RAS extension, it is a NOP,
  // so all predicates should be allowed.
  if (imm8 == 0x10 && pred != 0xe && ((FeatureBits[ARM::FeatureRAS]) != 0))
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1665-1666
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1668-1674
```cpp
static DecodeStatus DecodeT2CPSInstruction(MCInst &Inst, unsigned Insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  unsigned imod = fieldFromInstruction(Insn, 9, 2);
  unsigned M = fieldFromInstruction(Insn, 8, 1);
  unsigned iflags = fieldFromInstruction(Insn, 5, 3);
  unsigned mode = fieldFromInstruction(Insn, 0, 5);
```
- EN: Implements `DecodeT2CPSInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2CPSInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1676-1676
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1678-1681
```cpp
  // imod == '01' --> UNPREDICTABLE
  // NOTE: Even though this is technically UNPREDICTABLE, we choose to
  // return failure here.  The '01' imod value is unprintable, so there's
  // nothing useful we could do even if we returned UNPREDICTABLE.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1683-1683
```cpp
  if (imod == 1) return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1685-1699
```cpp
  if (imod && M) {
    Inst.setOpcode(ARM::t2CPS3p);
    Inst.addOperand(MCOperand::createImm(imod));
    Inst.addOperand(MCOperand::createImm(iflags));
    Inst.addOperand(MCOperand::createImm(mode));
  } else if (imod && !M) {
    Inst.setOpcode(ARM::t2CPS2p);
    Inst.addOperand(MCOperand::createImm(imod));
    Inst.addOperand(MCOperand::createImm(iflags));
    if (mode) S = MCDisassembler::SoftFail;
  } else if (!imod && M) {
    Inst.setOpcode(ARM::t2CPS1p);
    Inst.addOperand(MCOperand::createImm(mode));
    if (iflags) S = MCDisassembler::SoftFail;
  } else {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1700-1707
```cpp
    // imod == '00' && M == '0' --> this is a HINT instruction
    int imm = fieldFromInstruction(Insn, 0, 8);
    // HINT are defined only for immediate in [0..4]
    if(imm > 4) return MCDisassembler::Fail;
    Inst.setOpcode(ARM::t2HINT);
    Inst.addOperand(MCOperand::createImm(imm));
    DecodePredicateOperand(Inst, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1709-1710
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1712-1715
```cpp
static DecodeStatus
DecodeT2HintSpaceInstruction(MCInst &Inst, unsigned Insn, uint64_t Address,
                             const MCDisassembler *Decoder) {
  unsigned imm = fieldFromInstruction(Insn, 0, 8);
```
- EN: Implements `DecodeT2HintSpaceInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2HintSpaceInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1717-1717
```cpp
  unsigned Opcode = ARM::t2HINT;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1719-1727
```cpp
  if (imm == 0x0D) {
    Opcode = ARM::t2PACBTI;
  } else if (imm == 0x1D) {
    Opcode = ARM::t2PAC;
  } else if (imm == 0x2D) {
    Opcode = ARM::t2AUT;
  } else if (imm == 0x0F) {
    Opcode = ARM::t2BTI;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1729-1733
```cpp
  Inst.setOpcode(Opcode);
  if (Opcode == ARM::t2HINT) {
    Inst.addOperand(MCOperand::createImm(imm));
    DecodePredicateOperand(Inst, Decoder);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1735-1736
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1738-1741
```cpp
static DecodeStatus DecodeT2MOVTWInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2MOVTWInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2MOVTWInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1743-1744
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 8, 4);
  unsigned imm = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1746-1749
```cpp
  imm |= (fieldFromInstruction(Insn, 0, 8) << 0);
  imm |= (fieldFromInstruction(Insn, 12, 3) << 8);
  imm |= (fieldFromInstruction(Insn, 16, 4) << 12);
  imm |= (fieldFromInstruction(Insn, 26, 1) << 11);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1751-1755
```cpp
  if (Inst.getOpcode() == ARM::t2MOVTi16)
    if (!Check(S, DecoderGPRRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1757-1758
```cpp
  if (!tryAddingSymbolicOperand(Address, imm, false, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1760-1762
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1764-1767
```cpp
static DecodeStatus DecodeArmMOVTWInstruction(MCInst &Inst, unsigned Insn,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeArmMOVTWInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeArmMOVTWInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1769-1771
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned imm = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1773-1774
```cpp
  imm |= (fieldFromInstruction(Insn, 0, 12) << 0);
  imm |= (fieldFromInstruction(Insn, 16, 4) << 12);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1776-1778
```cpp
  if (Inst.getOpcode() == ARM::MOVTi16)
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1780-1781
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1783-1784
```cpp
  if (!tryAddingSymbolicOperand(Address, imm, false, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1786-1787
```cpp
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1789-1790
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1792-1795
```cpp
static DecodeStatus DecodeSMLAInstruction(MCInst &Inst, unsigned Insn,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSMLAInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSMLAInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1797-1801
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 16, 4);
  unsigned Rn = fieldFromInstruction(Insn, 0, 4);
  unsigned Rm = fieldFromInstruction(Insn, 8, 4);
  unsigned Ra = fieldFromInstruction(Insn, 12, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1803-1804
```cpp
  if (pred == 0xF)
    return DecodeCPSInstruction(Inst, Insn, Address, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1806-1813
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Ra, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1815-1816
```cpp
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1818-1819
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1821-1824
```cpp
static DecodeStatus DecodeSETPANInstruction(MCInst &Inst, unsigned Insn,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSETPANInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSETPANInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1826-1826
```cpp
  unsigned Imm = fieldFromInstruction(Insn, 9, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1828-1829
```cpp
  const FeatureBitset &FeatureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1831-1833
```cpp
  if (!FeatureBits[ARM::HasV8_1aOps] ||
      !FeatureBits[ARM::HasV8Ops])
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1835-1842
```cpp
  // Decoder can be called from DecodeTST, which does not check the full
  // encoding is valid.
  if (fieldFromInstruction(Insn, 20,12) != 0xf11 ||
      fieldFromInstruction(Insn, 4,4) != 0)
    return MCDisassembler::Fail;
  if (fieldFromInstruction(Insn, 10,10) != 0 ||
      fieldFromInstruction(Insn, 0,4) != 0)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1844-1845
```cpp
  Inst.setOpcode(ARM::SETPAN);
  Inst.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1847-1848
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1850-1853
```cpp
static DecodeStatus DecodeTSTInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeTSTInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeTSTInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1855-1857
```cpp
  unsigned Pred = fieldFromInstruction(Insn, 28, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1859-1860
```cpp
  if (Pred == 0xF)
    return DecodeSETPANInstruction(Inst, Insn, Address, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1862-1867
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, Pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1869-1870
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1872-1875
```cpp
static DecodeStatus DecodeAddrModeImm12Operand(MCInst &Inst, unsigned Val,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeAddrModeImm12Operand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrModeImm12Operand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1877-1879
```cpp
  unsigned add = fieldFromInstruction(Val, 12, 1);
  unsigned imm = fieldFromInstruction(Val, 0, 12);
  unsigned Rn = fieldFromInstruction(Val, 13, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1881-1882
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1884-1888
```cpp
  if (!add) imm *= -1;
  if (imm == 0 && !add) imm = INT32_MIN;
  Inst.addOperand(MCOperand::createImm(imm));
  if (Rn == 15)
    tryAddingPcLoadReferenceComment(Address, Address + imm + 8, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1890-1891
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1893-1896
```cpp
static DecodeStatus DecodeAddrMode5Operand(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeAddrMode5Operand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrMode5Operand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1898-1901
```cpp
  unsigned Rn = fieldFromInstruction(Val, 9, 4);
  // U == 1 to add imm, 0 to subtract it.
  unsigned U = fieldFromInstruction(Val, 8, 1);
  unsigned imm = fieldFromInstruction(Val, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1903-1904
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1906-1909
```cpp
  if (U)
    Inst.addOperand(MCOperand::createImm(ARM_AM::getAM5Opc(ARM_AM::add, imm)));
  else
    Inst.addOperand(MCOperand::createImm(ARM_AM::getAM5Opc(ARM_AM::sub, imm)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1911-1912
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1914-1917
```cpp
static DecodeStatus DecodeAddrMode5FP16Operand(MCInst &Inst, unsigned Val,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeAddrMode5FP16Operand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrMode5FP16Operand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1919-1922
```cpp
  unsigned Rn = fieldFromInstruction(Val, 9, 4);
  // U == 1 to add imm, 0 to subtract it.
  unsigned U = fieldFromInstruction(Val, 8, 1);
  unsigned imm = fieldFromInstruction(Val, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1924-1925
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1927-1930
```cpp
  if (U)
    Inst.addOperand(MCOperand::createImm(ARM_AM::getAM5FP16Opc(ARM_AM::add, imm)));
  else
    Inst.addOperand(MCOperand::createImm(ARM_AM::getAM5FP16Opc(ARM_AM::sub, imm)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1932-1933
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1935-1939
```cpp
static DecodeStatus DecodeAddrMode7Operand(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  return DecodeGPRRegisterClass(Inst, Val, Address, Decoder);
}
```
- EN: Implements `DecodeAddrMode7Operand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrMode7Operand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1941-1944
```cpp
static DecodeStatus DecodeT2BInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus Status = MCDisassembler::Success;
```
- EN: Implements `DecodeT2BInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2BInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1946-1960
```cpp
  // Note the J1 and J2 values are from the encoded instruction.  So here
  // change them to I1 and I2 values via as documented:
  // I1 = NOT(J1 EOR S);
  // I2 = NOT(J2 EOR S);
  // and build the imm32 with one trailing zero as documented:
  // imm32 = SignExtend(S:I1:I2:imm10:imm11:'0', 32);
  unsigned S = fieldFromInstruction(Insn, 26, 1);
  unsigned J1 = fieldFromInstruction(Insn, 13, 1);
  unsigned J2 = fieldFromInstruction(Insn, 11, 1);
  unsigned I1 = !(J1 ^ S);
  unsigned I2 = !(J2 ^ S);
  unsigned imm10 = fieldFromInstruction(Insn, 16, 10);
  unsigned imm11 = fieldFromInstruction(Insn, 0, 11);
  unsigned tmp = (S << 23) | (I1 << 22) | (I2 << 21) | (imm10 << 11) | imm11;
  int imm32 = SignExtend32<25>(tmp << 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1961-1963
```cpp
  if (!tryAddingSymbolicOperand(Address, Address + imm32 + 4,
                                true, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(imm32));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1965-1967
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return Status;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1969-1972
```cpp
static DecodeStatus DecodeBranchImmInstruction(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeBranchImmInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBranchImmInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1974-1975
```cpp
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 24) << 2;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1977-1984
```cpp
  if (pred == 0xF) {
    Inst.setOpcode(ARM::BLXi);
    imm |= fieldFromInstruction(Insn, 24, 1) << 1;
    if (!tryAddingSymbolicOperand(Address, Address + SignExtend32<26>(imm) + 8,
                                  true, 4, Inst, Decoder))
      Inst.addOperand(MCOperand::createImm(SignExtend32<26>(imm)));
    return S;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1986-1988
```cpp
  if (!tryAddingSymbolicOperand(Address, Address + SignExtend32<26>(imm) + 8,
                                true, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(SignExtend32<26>(imm)));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1990-1994
```cpp
  // We already have BL_pred for BL w/ predicate, no need to add addition
  // predicate operands for BL
  if (Inst.getOpcode() != ARM::BL)
    if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1996-1997
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1999-2002
```cpp
static DecodeStatus DecodeAddrMode6Operand(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeAddrMode6Operand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeAddrMode6Operand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2004-2005
```cpp
  unsigned Rm = fieldFromInstruction(Val, 0, 4);
  unsigned align = fieldFromInstruction(Val, 4, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2007-2012
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!align)
    Inst.addOperand(MCOperand::createImm(0));
  else
    Inst.addOperand(MCOperand::createImm(4 << align));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2014-2015
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2017-2020
```cpp
static DecodeStatus DecodeVLDInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLDInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLDInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2022-2027
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned wb = fieldFromInstruction(Insn, 16, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  Rn |= fieldFromInstruction(Insn, 4, 2) << 4;
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2029-2043
```cpp
  // First output register
  switch (Inst.getOpcode()) {
  case ARM::VLD1q16: case ARM::VLD1q32: case ARM::VLD1q64: case ARM::VLD1q8:
  case ARM::VLD1q16wb_fixed: case ARM::VLD1q16wb_register:
  case ARM::VLD1q32wb_fixed: case ARM::VLD1q32wb_register:
  case ARM::VLD1q64wb_fixed: case ARM::VLD1q64wb_register:
  case ARM::VLD1q8wb_fixed: case ARM::VLD1q8wb_register:
  case ARM::VLD2d16: case ARM::VLD2d32: case ARM::VLD2d8:
  case ARM::VLD2d16wb_fixed: case ARM::VLD2d16wb_register:
  case ARM::VLD2d32wb_fixed: case ARM::VLD2d32wb_register:
  case ARM::VLD2d8wb_fixed: case ARM::VLD2d8wb_register:
    if (!Check(S, DecodeDPairRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  case ARM::VLD2b16:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2044-2058
```cpp
  case ARM::VLD2b32:
  case ARM::VLD2b8:
  case ARM::VLD2b16wb_fixed:
  case ARM::VLD2b16wb_register:
  case ARM::VLD2b32wb_fixed:
  case ARM::VLD2b32wb_register:
  case ARM::VLD2b8wb_fixed:
  case ARM::VLD2b8wb_register:
    if (!Check(S, DecodeDPairSpacedRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  default:
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2060-2074
```cpp
  // Second output register
  switch (Inst.getOpcode()) {
    case ARM::VLD3d8:
    case ARM::VLD3d16:
    case ARM::VLD3d32:
    case ARM::VLD3d8_UPD:
    case ARM::VLD3d16_UPD:
    case ARM::VLD3d32_UPD:
    case ARM::VLD4d8:
    case ARM::VLD4d16:
    case ARM::VLD4d32:
    case ARM::VLD4d8_UPD:
    case ARM::VLD4d16_UPD:
    case ARM::VLD4d32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+1)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2075-2089
```cpp
        return MCDisassembler::Fail;
      break;
    case ARM::VLD3q8:
    case ARM::VLD3q16:
    case ARM::VLD3q32:
    case ARM::VLD3q8_UPD:
    case ARM::VLD3q16_UPD:
    case ARM::VLD3q32_UPD:
    case ARM::VLD4q8:
    case ARM::VLD4q16:
    case ARM::VLD4q32:
    case ARM::VLD4q8_UPD:
    case ARM::VLD4q16_UPD:
    case ARM::VLD4q32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+2)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2090-2094
```cpp
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2096-2110
```cpp
  // Third output register
  switch(Inst.getOpcode()) {
    case ARM::VLD3d8:
    case ARM::VLD3d16:
    case ARM::VLD3d32:
    case ARM::VLD3d8_UPD:
    case ARM::VLD3d16_UPD:
    case ARM::VLD3d32_UPD:
    case ARM::VLD4d8:
    case ARM::VLD4d16:
    case ARM::VLD4d32:
    case ARM::VLD4d8_UPD:
    case ARM::VLD4d16_UPD:
    case ARM::VLD4d32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+2)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2111-2125
```cpp
        return MCDisassembler::Fail;
      break;
    case ARM::VLD3q8:
    case ARM::VLD3q16:
    case ARM::VLD3q32:
    case ARM::VLD3q8_UPD:
    case ARM::VLD3q16_UPD:
    case ARM::VLD3q32_UPD:
    case ARM::VLD4q8:
    case ARM::VLD4q16:
    case ARM::VLD4q32:
    case ARM::VLD4q8_UPD:
    case ARM::VLD4q16_UPD:
    case ARM::VLD4q32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+4)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2126-2130
```cpp
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2132-2146
```cpp
  // Fourth output register
  switch (Inst.getOpcode()) {
    case ARM::VLD4d8:
    case ARM::VLD4d16:
    case ARM::VLD4d32:
    case ARM::VLD4d8_UPD:
    case ARM::VLD4d16_UPD:
    case ARM::VLD4d32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+3)%32, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    case ARM::VLD4q8:
    case ARM::VLD4q16:
    case ARM::VLD4q32:
    case ARM::VLD4q8_UPD:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2147-2154
```cpp
    case ARM::VLD4q16_UPD:
    case ARM::VLD4q32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+6)%32, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2156-2170
```cpp
  // Writeback operand
  switch (Inst.getOpcode()) {
    case ARM::VLD1d8wb_fixed:
    case ARM::VLD1d16wb_fixed:
    case ARM::VLD1d32wb_fixed:
    case ARM::VLD1d64wb_fixed:
    case ARM::VLD1d8wb_register:
    case ARM::VLD1d16wb_register:
    case ARM::VLD1d32wb_register:
    case ARM::VLD1d64wb_register:
    case ARM::VLD1q8wb_fixed:
    case ARM::VLD1q16wb_fixed:
    case ARM::VLD1q32wb_fixed:
    case ARM::VLD1q64wb_fixed:
    case ARM::VLD1q8wb_register:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2171-2185
```cpp
    case ARM::VLD1q16wb_register:
    case ARM::VLD1q32wb_register:
    case ARM::VLD1q64wb_register:
    case ARM::VLD1d8Twb_fixed:
    case ARM::VLD1d8Twb_register:
    case ARM::VLD1d16Twb_fixed:
    case ARM::VLD1d16Twb_register:
    case ARM::VLD1d32Twb_fixed:
    case ARM::VLD1d32Twb_register:
    case ARM::VLD1d64Twb_fixed:
    case ARM::VLD1d64Twb_register:
    case ARM::VLD1d8Qwb_fixed:
    case ARM::VLD1d8Qwb_register:
    case ARM::VLD1d16Qwb_fixed:
    case ARM::VLD1d16Qwb_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2186-2200
```cpp
    case ARM::VLD1d32Qwb_fixed:
    case ARM::VLD1d32Qwb_register:
    case ARM::VLD1d64Qwb_fixed:
    case ARM::VLD1d64Qwb_register:
    case ARM::VLD2d8wb_fixed:
    case ARM::VLD2d16wb_fixed:
    case ARM::VLD2d32wb_fixed:
    case ARM::VLD2q8wb_fixed:
    case ARM::VLD2q16wb_fixed:
    case ARM::VLD2q32wb_fixed:
    case ARM::VLD2d8wb_register:
    case ARM::VLD2d16wb_register:
    case ARM::VLD2d32wb_register:
    case ARM::VLD2q8wb_register:
    case ARM::VLD2q16wb_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2201-2215
```cpp
    case ARM::VLD2q32wb_register:
    case ARM::VLD2b8wb_fixed:
    case ARM::VLD2b16wb_fixed:
    case ARM::VLD2b32wb_fixed:
    case ARM::VLD2b8wb_register:
    case ARM::VLD2b16wb_register:
    case ARM::VLD2b32wb_register:
      Inst.addOperand(MCOperand::createImm(0));
      break;
    case ARM::VLD3d8_UPD:
    case ARM::VLD3d16_UPD:
    case ARM::VLD3d32_UPD:
    case ARM::VLD3q8_UPD:
    case ARM::VLD3q16_UPD:
    case ARM::VLD3q32_UPD:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2216-2227
```cpp
    case ARM::VLD4d8_UPD:
    case ARM::VLD4d16_UPD:
    case ARM::VLD4d32_UPD:
    case ARM::VLD4q8_UPD:
    case ARM::VLD4q16_UPD:
    case ARM::VLD4q32_UPD:
      if (!Check(S, DecodeGPRRegisterClass(Inst, wb, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2229-2231
```cpp
  // AddrMode6 Base (register+alignment)
  if (!Check(S, DecodeAddrMode6Operand(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2233-2247
```cpp
  // AddrMode6 Offset (register)
  switch (Inst.getOpcode()) {
  default:
    // The below have been updated to have explicit am6offset split
    // between fixed and register offset. For those instructions not
    // yet updated, we need to add an additional reg0 operand for the
    // fixed variant.
    //
    // The fixed offset encodes as Rm == 0xd, so we check for that.
    if (Rm == 0xd) {
      Inst.addOperand(MCOperand::createReg(0));
      break;
    }
    // Fall through to handle the register offset variant.
    [[fallthrough]];
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2248-2262
```cpp
  case ARM::VLD1d8wb_fixed:
  case ARM::VLD1d16wb_fixed:
  case ARM::VLD1d32wb_fixed:
  case ARM::VLD1d64wb_fixed:
  case ARM::VLD1d8Twb_fixed:
  case ARM::VLD1d16Twb_fixed:
  case ARM::VLD1d32Twb_fixed:
  case ARM::VLD1d64Twb_fixed:
  case ARM::VLD1d8Qwb_fixed:
  case ARM::VLD1d16Qwb_fixed:
  case ARM::VLD1d32Qwb_fixed:
  case ARM::VLD1d64Qwb_fixed:
  case ARM::VLD1d8wb_register:
  case ARM::VLD1d16wb_register:
  case ARM::VLD1d32wb_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2263-2277
```cpp
  case ARM::VLD1d64wb_register:
  case ARM::VLD1q8wb_fixed:
  case ARM::VLD1q16wb_fixed:
  case ARM::VLD1q32wb_fixed:
  case ARM::VLD1q64wb_fixed:
  case ARM::VLD1q8wb_register:
  case ARM::VLD1q16wb_register:
  case ARM::VLD1q32wb_register:
  case ARM::VLD1q64wb_register:
    // The fixed offset post-increment encodes Rm == 0xd. The no-writeback
    // variant encodes Rm == 0xf. Anything else is a register offset post-
    // increment and we need to add the register operand to the instruction.
    if (Rm != 0xD && Rm != 0xF &&
        !Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2278-2289
```cpp
    break;
  case ARM::VLD2d8wb_fixed:
  case ARM::VLD2d16wb_fixed:
  case ARM::VLD2d32wb_fixed:
  case ARM::VLD2b8wb_fixed:
  case ARM::VLD2b16wb_fixed:
  case ARM::VLD2b32wb_fixed:
  case ARM::VLD2q8wb_fixed:
  case ARM::VLD2q16wb_fixed:
  case ARM::VLD2q32wb_fixed:
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2291-2293
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2295-2298
```cpp
static DecodeStatus DecodeVSTInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVSTInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVSTInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2300-2305
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned wb = fieldFromInstruction(Insn, 16, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  Rn |= fieldFromInstruction(Insn, 4, 2) << 4;
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2307-2321
```cpp
  // Writeback Operand
  switch (Inst.getOpcode()) {
    case ARM::VST1d8wb_fixed:
    case ARM::VST1d16wb_fixed:
    case ARM::VST1d32wb_fixed:
    case ARM::VST1d64wb_fixed:
    case ARM::VST1d8wb_register:
    case ARM::VST1d16wb_register:
    case ARM::VST1d32wb_register:
    case ARM::VST1d64wb_register:
    case ARM::VST1q8wb_fixed:
    case ARM::VST1q16wb_fixed:
    case ARM::VST1q32wb_fixed:
    case ARM::VST1q64wb_fixed:
    case ARM::VST1q8wb_register:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2322-2336
```cpp
    case ARM::VST1q16wb_register:
    case ARM::VST1q32wb_register:
    case ARM::VST1q64wb_register:
    case ARM::VST1d8Twb_fixed:
    case ARM::VST1d16Twb_fixed:
    case ARM::VST1d32Twb_fixed:
    case ARM::VST1d64Twb_fixed:
    case ARM::VST1d8Twb_register:
    case ARM::VST1d16Twb_register:
    case ARM::VST1d32Twb_register:
    case ARM::VST1d64Twb_register:
    case ARM::VST1d8Qwb_fixed:
    case ARM::VST1d16Qwb_fixed:
    case ARM::VST1d32Qwb_fixed:
    case ARM::VST1d64Qwb_fixed:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2337-2351
```cpp
    case ARM::VST1d8Qwb_register:
    case ARM::VST1d16Qwb_register:
    case ARM::VST1d32Qwb_register:
    case ARM::VST1d64Qwb_register:
    case ARM::VST2d8wb_fixed:
    case ARM::VST2d16wb_fixed:
    case ARM::VST2d32wb_fixed:
    case ARM::VST2d8wb_register:
    case ARM::VST2d16wb_register:
    case ARM::VST2d32wb_register:
    case ARM::VST2q8wb_fixed:
    case ARM::VST2q16wb_fixed:
    case ARM::VST2q32wb_fixed:
    case ARM::VST2q8wb_register:
    case ARM::VST2q16wb_register:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2352-2366
```cpp
    case ARM::VST2q32wb_register:
    case ARM::VST2b8wb_fixed:
    case ARM::VST2b16wb_fixed:
    case ARM::VST2b32wb_fixed:
    case ARM::VST2b8wb_register:
    case ARM::VST2b16wb_register:
    case ARM::VST2b32wb_register:
      if (Rm == 0xF)
        return MCDisassembler::Fail;
      Inst.addOperand(MCOperand::createImm(0));
      break;
    case ARM::VST3d8_UPD:
    case ARM::VST3d16_UPD:
    case ARM::VST3d32_UPD:
    case ARM::VST3q8_UPD:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2367-2380
```cpp
    case ARM::VST3q16_UPD:
    case ARM::VST3q32_UPD:
    case ARM::VST4d8_UPD:
    case ARM::VST4d16_UPD:
    case ARM::VST4d32_UPD:
    case ARM::VST4q8_UPD:
    case ARM::VST4q16_UPD:
    case ARM::VST4q32_UPD:
      if (!Check(S, DecodeGPRRegisterClass(Inst, wb, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2382-2384
```cpp
  // AddrMode6 Base (register+alignment)
  if (!Check(S, DecodeAddrMode6Operand(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2386-2400
```cpp
  // AddrMode6 Offset (register)
  switch (Inst.getOpcode()) {
    default:
      if (Rm == 0xD)
        Inst.addOperand(MCOperand::createReg(0));
      else if (Rm != 0xF) {
        if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
          return MCDisassembler::Fail;
      }
      break;
    case ARM::VST1d8wb_fixed:
    case ARM::VST1d16wb_fixed:
    case ARM::VST1d32wb_fixed:
    case ARM::VST1d64wb_fixed:
    case ARM::VST1q8wb_fixed:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2401-2415
```cpp
    case ARM::VST1q16wb_fixed:
    case ARM::VST1q32wb_fixed:
    case ARM::VST1q64wb_fixed:
    case ARM::VST1d8Twb_fixed:
    case ARM::VST1d16Twb_fixed:
    case ARM::VST1d32Twb_fixed:
    case ARM::VST1d64Twb_fixed:
    case ARM::VST1d8Qwb_fixed:
    case ARM::VST1d16Qwb_fixed:
    case ARM::VST1d32Qwb_fixed:
    case ARM::VST1d64Qwb_fixed:
    case ARM::VST2d8wb_fixed:
    case ARM::VST2d16wb_fixed:
    case ARM::VST2d32wb_fixed:
    case ARM::VST2q8wb_fixed:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2416-2422
```cpp
    case ARM::VST2q16wb_fixed:
    case ARM::VST2q32wb_fixed:
    case ARM::VST2b8wb_fixed:
    case ARM::VST2b16wb_fixed:
    case ARM::VST2b32wb_fixed:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2424-2438
```cpp
  // First input register
  switch (Inst.getOpcode()) {
  case ARM::VST1q16:
  case ARM::VST1q32:
  case ARM::VST1q64:
  case ARM::VST1q8:
  case ARM::VST1q16wb_fixed:
  case ARM::VST1q16wb_register:
  case ARM::VST1q32wb_fixed:
  case ARM::VST1q32wb_register:
  case ARM::VST1q64wb_fixed:
  case ARM::VST1q64wb_register:
  case ARM::VST1q8wb_fixed:
  case ARM::VST1q8wb_register:
  case ARM::VST2d16:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2439-2453
```cpp
  case ARM::VST2d32:
  case ARM::VST2d8:
  case ARM::VST2d16wb_fixed:
  case ARM::VST2d16wb_register:
  case ARM::VST2d32wb_fixed:
  case ARM::VST2d32wb_register:
  case ARM::VST2d8wb_fixed:
  case ARM::VST2d8wb_register:
    if (!Check(S, DecodeDPairRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  case ARM::VST2b16:
  case ARM::VST2b32:
  case ARM::VST2b8:
  case ARM::VST2b16wb_fixed:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2454-2465
```cpp
  case ARM::VST2b16wb_register:
  case ARM::VST2b32wb_fixed:
  case ARM::VST2b32wb_register:
  case ARM::VST2b8wb_fixed:
  case ARM::VST2b8wb_register:
    if (!Check(S, DecodeDPairSpacedRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  default:
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2467-2481
```cpp
  // Second input register
  switch (Inst.getOpcode()) {
    case ARM::VST3d8:
    case ARM::VST3d16:
    case ARM::VST3d32:
    case ARM::VST3d8_UPD:
    case ARM::VST3d16_UPD:
    case ARM::VST3d32_UPD:
    case ARM::VST4d8:
    case ARM::VST4d16:
    case ARM::VST4d32:
    case ARM::VST4d8_UPD:
    case ARM::VST4d16_UPD:
    case ARM::VST4d32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+1)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2482-2496
```cpp
        return MCDisassembler::Fail;
      break;
    case ARM::VST3q8:
    case ARM::VST3q16:
    case ARM::VST3q32:
    case ARM::VST3q8_UPD:
    case ARM::VST3q16_UPD:
    case ARM::VST3q32_UPD:
    case ARM::VST4q8:
    case ARM::VST4q16:
    case ARM::VST4q32:
    case ARM::VST4q8_UPD:
    case ARM::VST4q16_UPD:
    case ARM::VST4q32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+2)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2497-2501
```cpp
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2503-2517
```cpp
  // Third input register
  switch (Inst.getOpcode()) {
    case ARM::VST3d8:
    case ARM::VST3d16:
    case ARM::VST3d32:
    case ARM::VST3d8_UPD:
    case ARM::VST3d16_UPD:
    case ARM::VST3d32_UPD:
    case ARM::VST4d8:
    case ARM::VST4d16:
    case ARM::VST4d32:
    case ARM::VST4d8_UPD:
    case ARM::VST4d16_UPD:
    case ARM::VST4d32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+2)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2518-2532
```cpp
        return MCDisassembler::Fail;
      break;
    case ARM::VST3q8:
    case ARM::VST3q16:
    case ARM::VST3q32:
    case ARM::VST3q8_UPD:
    case ARM::VST3q16_UPD:
    case ARM::VST3q32_UPD:
    case ARM::VST4q8:
    case ARM::VST4q16:
    case ARM::VST4q32:
    case ARM::VST4q8_UPD:
    case ARM::VST4q16_UPD:
    case ARM::VST4q32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+4)%32, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2533-2537
```cpp
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2539-2553
```cpp
  // Fourth input register
  switch (Inst.getOpcode()) {
    case ARM::VST4d8:
    case ARM::VST4d16:
    case ARM::VST4d32:
    case ARM::VST4d8_UPD:
    case ARM::VST4d16_UPD:
    case ARM::VST4d32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+3)%32, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    case ARM::VST4q8:
    case ARM::VST4q16:
    case ARM::VST4q32:
    case ARM::VST4q8_UPD:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2554-2561
```cpp
    case ARM::VST4q16_UPD:
    case ARM::VST4q32_UPD:
      if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+6)%32, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2563-2565
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2567-2574
```cpp
static DecodeStatus DecodeVLDST1Instruction(MCInst &Inst, unsigned Insn,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  unsigned type = fieldFromInstruction(Insn, 8, 4);
  unsigned align = fieldFromInstruction(Insn, 4, 2);
  if (type == 6 && (align & 2)) return MCDisassembler::Fail;
  if (type == 7 && (align & 2)) return MCDisassembler::Fail;
  if (type == 10 && align == 3) return MCDisassembler::Fail;
```
- EN: Implements `DecodeVLDST1Instruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLDST1Instruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2576-2579
```cpp
  unsigned load = fieldFromInstruction(Insn, 21, 1);
  return load ? DecodeVLDInstruction(Inst, Insn, Address, Decoder)
              : DecodeVSTInstruction(Inst, Insn, Address, Decoder);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2581-2585
```cpp
static DecodeStatus DecodeVLDST2Instruction(MCInst &Inst, unsigned Insn,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  unsigned size = fieldFromInstruction(Insn, 6, 2);
  if (size == 3) return MCDisassembler::Fail;
```
- EN: Implements `DecodeVLDST2Instruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLDST2Instruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2587-2590
```cpp
  unsigned type = fieldFromInstruction(Insn, 8, 4);
  unsigned align = fieldFromInstruction(Insn, 4, 2);
  if (type == 8 && align == 3) return MCDisassembler::Fail;
  if (type == 9 && align == 3) return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2592-2595
```cpp
  unsigned load = fieldFromInstruction(Insn, 21, 1);
  return load ? DecodeVLDInstruction(Inst, Insn, Address, Decoder)
              : DecodeVSTInstruction(Inst, Insn, Address, Decoder);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2597-2601
```cpp
static DecodeStatus DecodeVLDST3Instruction(MCInst &Inst, unsigned Insn,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  unsigned size = fieldFromInstruction(Insn, 6, 2);
  if (size == 3) return MCDisassembler::Fail;
```
- EN: Implements `DecodeVLDST3Instruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLDST3Instruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2603-2604
```cpp
  unsigned align = fieldFromInstruction(Insn, 4, 2);
  if (align & 2) return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2606-2609
```cpp
  unsigned load = fieldFromInstruction(Insn, 21, 1);
  return load ? DecodeVLDInstruction(Inst, Insn, Address, Decoder)
              : DecodeVSTInstruction(Inst, Insn, Address, Decoder);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2611-2615
```cpp
static DecodeStatus DecodeVLDST4Instruction(MCInst &Inst, unsigned Insn,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  unsigned size = fieldFromInstruction(Insn, 6, 2);
  if (size == 3) return MCDisassembler::Fail;
```
- EN: Implements `DecodeVLDST4Instruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLDST4Instruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2617-2620
```cpp
  unsigned load = fieldFromInstruction(Insn, 21, 1);
  return load ? DecodeVLDInstruction(Inst, Insn, Address, Decoder)
              : DecodeVSTInstruction(Inst, Insn, Address, Decoder);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2622-2625
```cpp
static DecodeStatus DecodeVLD1DupInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD1DupInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD1DupInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2627-2632
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned align = fieldFromInstruction(Insn, 4, 1);
  unsigned size = fieldFromInstruction(Insn, 6, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2634-2636
```cpp
  if (size == 0 && align == 1)
    return MCDisassembler::Fail;
  align *= (1 << size);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2638-2652
```cpp
  switch (Inst.getOpcode()) {
  case ARM::VLD1DUPq16: case ARM::VLD1DUPq32: case ARM::VLD1DUPq8:
  case ARM::VLD1DUPq16wb_fixed: case ARM::VLD1DUPq16wb_register:
  case ARM::VLD1DUPq32wb_fixed: case ARM::VLD1DUPq32wb_register:
  case ARM::VLD1DUPq8wb_fixed: case ARM::VLD1DUPq8wb_register:
    if (!Check(S, DecodeDPairRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  default:
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  }
  if (Rm != 0xF) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2653-2654
```cpp
      return MCDisassembler::Fail;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2656-2658
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2660-2665
```cpp
  // The fixed offset post-increment encodes Rm == 0xd. The no-writeback
  // variant encodes Rm == 0xf. Anything else is a register offset post-
  // increment and we need to add the register operand to the instruction.
  if (Rm != 0xD && Rm != 0xF &&
      !Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2667-2669
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2671-2674
```cpp
static DecodeStatus DecodeVLD2DupInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD2DupInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD2DupInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2676-2682
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned align = fieldFromInstruction(Insn, 4, 1);
  unsigned size = 1 << fieldFromInstruction(Insn, 6, 2);
  align *= 2*size;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2684-2698
```cpp
  switch (Inst.getOpcode()) {
  case ARM::VLD2DUPd16: case ARM::VLD2DUPd32: case ARM::VLD2DUPd8:
  case ARM::VLD2DUPd16wb_fixed: case ARM::VLD2DUPd16wb_register:
  case ARM::VLD2DUPd32wb_fixed: case ARM::VLD2DUPd32wb_register:
  case ARM::VLD2DUPd8wb_fixed: case ARM::VLD2DUPd8wb_register:
    if (!Check(S, DecodeDPairRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  case ARM::VLD2DUPd16x2: case ARM::VLD2DUPd32x2: case ARM::VLD2DUPd8x2:
  case ARM::VLD2DUPd16x2wb_fixed: case ARM::VLD2DUPd16x2wb_register:
  case ARM::VLD2DUPd32x2wb_fixed: case ARM::VLD2DUPd32x2wb_register:
  case ARM::VLD2DUPd8x2wb_fixed: case ARM::VLD2DUPd8x2wb_register:
    if (!Check(S, DecodeDPairSpacedRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2699-2703
```cpp
  default:
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  }
```
- EN: Declares `Check`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Check`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2705-2706
```cpp
  if (Rm != 0xF)
    Inst.addOperand(MCOperand::createImm(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2708-2710
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2712-2715
```cpp
  if (Rm != 0xD && Rm != 0xF) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2717-2719
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2721-2724
```cpp
static DecodeStatus DecodeVLD3DupInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD3DupInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD3DupInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2726-2730
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned inc = fieldFromInstruction(Insn, 5, 1) + 1;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2732-2741
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+inc)%32, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+2*inc)%32, Address, Decoder)))
    return MCDisassembler::Fail;
  if (Rm != 0xF) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2743-2745
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2747-2752
```cpp
  if (Rm == 0xD)
    Inst.addOperand(MCOperand::createReg(0));
  else if (Rm != 0xF) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2754-2756
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2758-2761
```cpp
static DecodeStatus DecodeVLD4DupInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD4DupInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD4DupInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2763-2769
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned size = fieldFromInstruction(Insn, 6, 2);
  unsigned inc = fieldFromInstruction(Insn, 5, 1) + 1;
  unsigned align = fieldFromInstruction(Insn, 4, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2771-2782
```cpp
  if (size == 0x3) {
    if (align == 0)
      return MCDisassembler::Fail;
    align = 16;
  } else {
    if (size == 2) {
      align *= 8;
    } else {
      size = 1 << size;
      align *= 4*size;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2784-2795
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+inc)%32, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+2*inc)%32, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, (Rd+3*inc)%32, Address, Decoder)))
    return MCDisassembler::Fail;
  if (Rm != 0xF) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2797-2799
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2801-2806
```cpp
  if (Rm == 0xD)
    Inst.addOperand(MCOperand::createReg(0));
  else if (Rm != 0xF) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2808-2810
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2812-2815
```cpp
static DecodeStatus DecodeVMOVModImmInstruction(MCInst &Inst, unsigned Insn,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVMOVModImmInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVMOVModImmInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2817-2824
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned imm = fieldFromInstruction(Insn, 0, 4);
  imm |= fieldFromInstruction(Insn, 16, 3) << 4;
  imm |= fieldFromInstruction(Insn, 24, 1) << 7;
  imm |= fieldFromInstruction(Insn, 8, 4) << 8;
  imm |= fieldFromInstruction(Insn, 5, 1) << 12;
  unsigned Q = fieldFromInstruction(Insn, 6, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2826-2832
```cpp
  if (Q) {
    if (!Check(S, DecodeQPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  } else {
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2834-2834
```cpp
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2836-2850
```cpp
  switch (Inst.getOpcode()) {
    case ARM::VORRiv4i16:
    case ARM::VORRiv2i32:
    case ARM::VBICiv4i16:
    case ARM::VBICiv2i32:
      if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
    case ARM::VORRiv8i16:
    case ARM::VORRiv4i32:
    case ARM::VBICiv8i16:
    case ARM::VBICiv4i32:
      if (!Check(S, DecodeQPRRegisterClass(Inst, Rd, Address, Decoder)))
        return MCDisassembler::Fail;
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2851-2853
```cpp
    default:
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2855-2857
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2859-2862
```cpp
static DecodeStatus DecodeMVEModImmInstruction(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeMVEModImmInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEModImmInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2864-2871
```cpp
  unsigned Qd = ((fieldFromInstruction(Insn, 22, 1) << 3) |
                 fieldFromInstruction(Insn, 13, 3));
  unsigned cmode = fieldFromInstruction(Insn, 8, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 4);
  imm |= fieldFromInstruction(Insn, 16, 3) << 4;
  imm |= fieldFromInstruction(Insn, 28, 1) << 7;
  imm |= cmode                             << 8;
  imm |= fieldFromInstruction(Insn, 5, 1)  << 12;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2873-2874
```cpp
  if (cmode == 0xF && Inst.getOpcode() == ARM::MVE_VMVNimmi32)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2876-2877
```cpp
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2879-2879
```cpp
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2881-2883
```cpp
  Check(S, DecodeVpredROperand(Inst, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2885-2888
```cpp
static DecodeStatus DecodeMVEVADCInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeMVEVADCInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEVADCInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2890-2894
```cpp
  unsigned Qd = fieldFromInstruction(Insn, 13, 3);
  Qd |= fieldFromInstruction(Insn, 22, 1) << 3;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createReg(ARM::FPSCR_NZCV));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2896-2905
```cpp
  unsigned Qn = fieldFromInstruction(Insn, 17, 3);
  Qn |= fieldFromInstruction(Insn, 7, 1) << 3;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qn, Address, Decoder)))
    return MCDisassembler::Fail;
  unsigned Qm = fieldFromInstruction(Insn, 1, 3);
  Qm |= fieldFromInstruction(Insn, 5, 1) << 3;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!fieldFromInstruction(Insn, 12, 1)) // I bit clear => need input FPSCR
    Inst.addOperand(MCOperand::createReg(ARM::FPSCR_NZCV));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2907-2909
```cpp
  Check(S, DecodeVpredROperand(Inst, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2911-2914
```cpp
static DecodeStatus DecodeVSHLMaxInstruction(MCInst &Inst, unsigned Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVSHLMaxInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVSHLMaxInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2916-2920
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  Rm |= fieldFromInstruction(Insn, 5, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 18, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2922-2926
```cpp
  if (!Check(S, DecodeQPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(8 << size));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2928-2930
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2932-2937
```cpp
static DecodeStatus DecodeShiftRight8Imm(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(8 - Val));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeShiftRight8Imm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeShiftRight8Imm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2939-2944
```cpp
static DecodeStatus DecodeShiftRight16Imm(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(16 - Val));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeShiftRight16Imm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeShiftRight16Imm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2946-2951
```cpp
static DecodeStatus DecodeShiftRight32Imm(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(32 - Val));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeShiftRight32Imm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeShiftRight32Imm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2953-2958
```cpp
static DecodeStatus DecodeShiftRight64Imm(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm(64 - Val));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeShiftRight64Imm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeShiftRight64Imm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2960-2963
```cpp
static DecodeStatus DecodeTBLInstruction(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeTBLInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeTBLInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 2965-2971
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  Rn |= fieldFromInstruction(Insn, 7, 1) << 4;
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  Rm |= fieldFromInstruction(Insn, 5, 1) << 4;
  unsigned op = fieldFromInstruction(Insn, 6, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2973-2978
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (op) {
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail; // Writeback
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2980-2989
```cpp
  switch (Inst.getOpcode()) {
  case ARM::VTBL2:
  case ARM::VTBX2:
    if (!Check(S, DecodeDPairRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  default:
    if (!Check(S, DecodeDPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2991-2992
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 2994-2996
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 2998-3001
```cpp
static DecodeStatus DecodeThumbAddSpecialReg(MCInst &Inst, uint16_t Insn,
                                             uint64_t Address,
                                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeThumbAddSpecialReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddSpecialReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3003-3004
```cpp
  unsigned dst = fieldFromInstruction(Insn, 8, 3);
  unsigned imm = fieldFromInstruction(Insn, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3006-3007
```cpp
  if (!Check(S, DecodetGPRRegisterClass(Inst, dst, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3009-3017
```cpp
  switch(Inst.getOpcode()) {
    default:
      return MCDisassembler::Fail;
    case ARM::tADR:
      break; // tADR does not explicitly represent the PC as an operand.
    case ARM::tADDrSPi:
      Inst.addOperand(MCOperand::createReg(ARM::SP));
      break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3019-3022
```cpp
  Inst.addOperand(MCOperand::createImm(imm));
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3024-3031
```cpp
static DecodeStatus DecodeThumbBROperand(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  if (!tryAddingSymbolicOperand(Address, Address + SignExtend32<12>(Val<<1) + 4,
                                true, 2, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(SignExtend32<12>(Val << 1)));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeThumbBROperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbBROperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3033-3040
```cpp
static DecodeStatus DecodeT2BROperand(MCInst &Inst, unsigned Val,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  if (!tryAddingSymbolicOperand(Address, Address + SignExtend32<21>(Val) + 4,
                                true, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(SignExtend32<21>(Val)));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeT2BROperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2BROperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3042-3049
```cpp
static DecodeStatus DecodeThumbCmpBROperand(MCInst &Inst, unsigned Val,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  if (!tryAddingSymbolicOperand(Address, Address + (Val<<1) + 4,
                                true, 2, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(Val << 1));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeThumbCmpBROperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbCmpBROperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3051-3054
```cpp
static DecodeStatus DecodeThumbAddrModeRR(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeThumbAddrModeRR`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddrModeRR`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3056-3057
```cpp
  unsigned Rn = fieldFromInstruction(Val, 0, 3);
  unsigned Rm = fieldFromInstruction(Val, 3, 3);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3059-3062
```cpp
  if (!Check(S, DecodetGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodetGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3064-3065
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3067-3070
```cpp
static DecodeStatus DecodeThumbAddrModeIS(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeThumbAddrModeIS`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddrModeIS`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3072-3073
```cpp
  unsigned Rn = fieldFromInstruction(Val, 0, 3);
  unsigned imm = fieldFromInstruction(Val, 3, 5);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3075-3077
```cpp
  if (!Check(S, DecodetGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3079-3080
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3082-3085
```cpp
static DecodeStatus DecodeThumbAddrModePC(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  unsigned imm = Val << 2;
```
- EN: Implements `DecodeThumbAddrModePC`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddrModePC`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3087-3088
```cpp
  Inst.addOperand(MCOperand::createImm(imm));
  tryAddingPcLoadReferenceComment(Address, (Address & ~2u) + imm + 4, Decoder);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3090-3091
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3093-3097
```cpp
static DecodeStatus DecodeThumbAddrModeSP(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createReg(ARM::SP));
  Inst.addOperand(MCOperand::createImm(Val));
```
- EN: Implements `DecodeThumbAddrModeSP`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddrModeSP`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3099-3100
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3102-3105
```cpp
static DecodeStatus DecodeT2AddrModeSOReg(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeSOReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeSOReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3107-3109
```cpp
  unsigned Rn = fieldFromInstruction(Val, 6, 4);
  unsigned Rm = fieldFromInstruction(Val, 2, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3111-3121
```cpp
  // Thumb stores cannot use PC as dest register.
  switch (Inst.getOpcode()) {
  case ARM::t2STRHs:
  case ARM::t2STRBs:
  case ARM::t2STRs:
    if (Rn == 15)
      return MCDisassembler::Fail;
    break;
  default:
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3123-3127
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3129-3130
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3132-3135
```cpp
static DecodeStatus DecodeT2LoadLabel(MCInst &Inst, unsigned Insn,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LoadLabel`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LoadLabel`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3137-3139
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned U = fieldFromInstruction(Insn, 23, 1);
  int imm = fieldFromInstruction(Insn, 0, 12);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3141-3142
```cpp
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3144-3144
```cpp
  bool hasV7Ops = featureBits[ARM::HasV7Ops];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3146-3160
```cpp
  if (Rt == 15) {
    switch (Inst.getOpcode()) {
      case ARM::t2LDRBpci:
      case ARM::t2LDRHpci:
        Inst.setOpcode(ARM::t2PLDpci);
        break;
      case ARM::t2LDRSBpci:
        Inst.setOpcode(ARM::t2PLIpci);
        break;
      case ARM::t2LDRSHpci:
        return MCDisassembler::Fail;
      default:
        break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3162-3172
```cpp
  switch(Inst.getOpcode()) {
  case ARM::t2PLDpci:
    break;
  case ARM::t2PLIpci:
    if (!hasV7Ops)
      return MCDisassembler::Fail;
    break;
  default:
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3174-3181
```cpp
  if (!U) {
    // Special case for #-0.
    if (imm == 0)
      imm = INT32_MIN;
    else
      imm = -imm;
  }
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3183-3185
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3187-3190
```cpp
static DecodeStatus DecodeT2LoadShift(MCInst &Inst, unsigned Insn,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LoadShift`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LoadShift`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3192-3193
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3195-3196
```cpp
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3198-3199
```cpp
  bool hasMP = featureBits[ARM::FeatureMP];
  bool hasV7Ops = featureBits[ARM::HasV7Ops];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3201-3215
```cpp
  if (Rn == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRBs:
      Inst.setOpcode(ARM::t2LDRBpci);
      break;
    case ARM::t2LDRHs:
      Inst.setOpcode(ARM::t2LDRHpci);
      break;
    case ARM::t2LDRSHs:
      Inst.setOpcode(ARM::t2LDRSHpci);
      break;
    case ARM::t2LDRSBs:
      Inst.setOpcode(ARM::t2LDRSBpci);
      break;
    case ARM::t2LDRs:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3216-3226
```cpp
      Inst.setOpcode(ARM::t2LDRpci);
      break;
    case ARM::t2PLDs:
      Inst.setOpcode(ARM::t2PLDpci);
      break;
    case ARM::t2PLIs:
      Inst.setOpcode(ARM::t2PLIpci);
      break;
    default:
      return MCDisassembler::Fail;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3228-3229
```cpp
    return DecodeT2LoadLabel(Inst, Insn, Address, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3231-3244
```cpp
  if (Rt == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRSHs:
      return MCDisassembler::Fail;
    case ARM::t2LDRHs:
      Inst.setOpcode(ARM::t2PLDWs);
      break;
    case ARM::t2LDRSBs:
      Inst.setOpcode(ARM::t2PLIs);
      break;
    default:
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3246-3260
```cpp
  switch (Inst.getOpcode()) {
    case ARM::t2PLDs:
      break;
    case ARM::t2PLIs:
      if (!hasV7Ops)
        return MCDisassembler::Fail;
      break;
    case ARM::t2PLDWs:
      if (!hasV7Ops || !hasMP)
        return MCDisassembler::Fail;
      break;
    default:
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
        return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3262-3266
```cpp
  unsigned addrmode = fieldFromInstruction(Insn, 4, 2);
  addrmode |= fieldFromInstruction(Insn, 0, 4) << 2;
  addrmode |= fieldFromInstruction(Insn, 16, 4) << 6;
  if (!Check(S, DecodeT2AddrModeSOReg(Inst, addrmode, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3268-3270
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3272-3279
```cpp
static DecodeStatus DecodeT2Imm8(MCInst &Inst, unsigned Val, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  int imm = Val & 0xFF;
  if (Val == 0)
    imm = INT32_MIN;
  else if (!(Val & 0x100))
    imm *= -1;
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Implements `DecodeT2Imm8`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2Imm8`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3281-3282
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3284-3287
```cpp
static DecodeStatus DecodeT2AddrModeImm8(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeImm8`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeImm8`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3289-3290
```cpp
  unsigned Rn = fieldFromInstruction(Val, 9, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 9);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3292-3305
```cpp
  // Thumb stores cannot use PC as dest register.
  switch (Inst.getOpcode()) {
  case ARM::t2STRT:
  case ARM::t2STRBT:
  case ARM::t2STRHT:
  case ARM::t2STRi8:
  case ARM::t2STRHi8:
  case ARM::t2STRBi8:
    if (Rn == 15)
      return MCDisassembler::Fail;
    break;
  default:
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3307-3321
```cpp
  // Some instructions always use an additive offset.
  switch (Inst.getOpcode()) {
    case ARM::t2LDRT:
    case ARM::t2LDRBT:
    case ARM::t2LDRHT:
    case ARM::t2LDRSBT:
    case ARM::t2LDRSHT:
    case ARM::t2STRT:
    case ARM::t2STRBT:
    case ARM::t2STRHT:
      imm |= 0x100;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3323-3326
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeT2Imm8(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3328-3329
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3331-3334
```cpp
static DecodeStatus DecodeT2LoadImm8(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LoadImm8`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LoadImm8`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3336-3342
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned U = fieldFromInstruction(Insn, 9, 1);
  unsigned imm = fieldFromInstruction(Insn, 0, 8);
  imm |= (U << 8);
  imm |= (Rn << 9);
  unsigned add = fieldFromInstruction(Insn, 9, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3344-3345
```cpp
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3347-3348
```cpp
  bool hasMP = featureBits[ARM::FeatureMP];
  bool hasV7Ops = featureBits[ARM::HasV7Ops];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3350-3364
```cpp
  if (Rn == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRi8:
      Inst.setOpcode(ARM::t2LDRpci);
      break;
    case ARM::t2LDRBi8:
      Inst.setOpcode(ARM::t2LDRBpci);
      break;
    case ARM::t2LDRSBi8:
      Inst.setOpcode(ARM::t2LDRSBpci);
      break;
    case ARM::t2LDRHi8:
      Inst.setOpcode(ARM::t2LDRHpci);
      break;
    case ARM::t2LDRSHi8:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3365-3377
```cpp
      Inst.setOpcode(ARM::t2LDRSHpci);
      break;
    case ARM::t2PLDi8:
      Inst.setOpcode(ARM::t2PLDpci);
      break;
    case ARM::t2PLIi8:
      Inst.setOpcode(ARM::t2PLIpci);
      break;
    default:
      return MCDisassembler::Fail;
    }
    return DecodeT2LoadLabel(Inst, Insn, Address, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3379-3393
```cpp
  if (Rt == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRSHi8:
      return MCDisassembler::Fail;
    case ARM::t2LDRHi8:
      if (!add)
        Inst.setOpcode(ARM::t2PLDWi8);
      break;
    case ARM::t2LDRSBi8:
      Inst.setOpcode(ARM::t2PLIi8);
      break;
    default:
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3395-3409
```cpp
  switch (Inst.getOpcode()) {
  case ARM::t2PLDi8:
    break;
  case ARM::t2PLIi8:
    if (!hasV7Ops)
      return MCDisassembler::Fail;
    break;
  case ARM::t2PLDWi8:
      if (!hasV7Ops || !hasMP)
        return MCDisassembler::Fail;
      break;
  default:
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3411-3415
```cpp
  if (!Check(S, DecodeT2AddrModeImm8(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3417-3420
```cpp
static DecodeStatus DecodeT2AddrModeImm12(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeImm12`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeImm12`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3422-3423
```cpp
  unsigned Rn = fieldFromInstruction(Val, 13, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 12);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3425-3435
```cpp
  // Thumb stores cannot use PC as dest register.
  switch (Inst.getOpcode()) {
  case ARM::t2STRi12:
  case ARM::t2STRBi12:
  case ARM::t2STRHi12:
    if (Rn == 15)
      return MCDisassembler::Fail;
    break;
  default:
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3437-3439
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3441-3442
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3444-3447
```cpp
static DecodeStatus DecodeT2LoadImm12(MCInst &Inst, unsigned Insn,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LoadImm12`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LoadImm12`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3449-3452
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 12);
  imm |= (Rn << 13);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3454-3455
```cpp
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3457-3458
```cpp
  bool hasMP = featureBits[ARM::FeatureMP];
  bool hasV7Ops = featureBits[ARM::HasV7Ops];
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3460-3474
```cpp
  if (Rn == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRi12:
      Inst.setOpcode(ARM::t2LDRpci);
      break;
    case ARM::t2LDRHi12:
      Inst.setOpcode(ARM::t2LDRHpci);
      break;
    case ARM::t2LDRSHi12:
      Inst.setOpcode(ARM::t2LDRSHpci);
      break;
    case ARM::t2LDRBi12:
      Inst.setOpcode(ARM::t2LDRBpci);
      break;
    case ARM::t2LDRSBi12:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3475-3487
```cpp
      Inst.setOpcode(ARM::t2LDRSBpci);
      break;
    case ARM::t2PLDi12:
      Inst.setOpcode(ARM::t2PLDpci);
      break;
    case ARM::t2PLIi12:
      Inst.setOpcode(ARM::t2PLIpci);
      break;
    default:
      return MCDisassembler::Fail;
    }
    return DecodeT2LoadLabel(Inst, Insn, Address, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3489-3502
```cpp
  if (Rt == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRSHi12:
      return MCDisassembler::Fail;
    case ARM::t2LDRHi12:
      Inst.setOpcode(ARM::t2PLDWi12);
      break;
    case ARM::t2LDRSBi12:
      Inst.setOpcode(ARM::t2PLIi12);
      break;
    default:
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3504-3518
```cpp
  switch (Inst.getOpcode()) {
  case ARM::t2PLDi12:
    break;
  case ARM::t2PLIi12:
    if (!hasV7Ops)
      return MCDisassembler::Fail;
    break;
  case ARM::t2PLDWi12:
      if (!hasV7Ops || !hasMP)
        return MCDisassembler::Fail;
      break;
  default:
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3520-3524
```cpp
  if (!Check(S, DecodeT2AddrModeImm12(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3526-3528
```cpp
static DecodeStatus DecodeT2LoadT(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LoadT`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LoadT`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3530-3533
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 8);
  imm |= (Rn << 9);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3535-3549
```cpp
  if (Rn == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDRT:
      Inst.setOpcode(ARM::t2LDRpci);
      break;
    case ARM::t2LDRBT:
      Inst.setOpcode(ARM::t2LDRBpci);
      break;
    case ARM::t2LDRHT:
      Inst.setOpcode(ARM::t2LDRHpci);
      break;
    case ARM::t2LDRSBT:
      Inst.setOpcode(ARM::t2LDRSBpci);
      break;
    case ARM::t2LDRSHT:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3550-3556
```cpp
      Inst.setOpcode(ARM::t2LDRSHpci);
      break;
    default:
      return MCDisassembler::Fail;
    }
    return DecodeT2LoadLabel(Inst, Insn, Address, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3558-3564
```cpp
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeT2AddrModeImm8(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3566-3571
```cpp
static DecodeStatus DecodeT2Imm8S4(MCInst &Inst, unsigned Val, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  if (Val == 0)
    Inst.addOperand(MCOperand::createImm(INT32_MIN));
  else {
    int imm = Val & 0xFF;
```
- EN: Implements `DecodeT2Imm8S4`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2Imm8S4`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3573-3575
```cpp
    if (!(Val & 0x100)) imm *= -1;
    Inst.addOperand(MCOperand::createImm(imm * 4));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3577-3578
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3580-3585
```cpp
static DecodeStatus DecodeT2Imm7S4(MCInst &Inst, unsigned Val, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  if (Val == 0)
    Inst.addOperand(MCOperand::createImm(INT32_MIN));
  else {
    int imm = Val & 0x7F;
```
- EN: Implements `DecodeT2Imm7S4`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2Imm7S4`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3587-3590
```cpp
    if (!(Val & 0x80))
      imm *= -1;
    Inst.addOperand(MCOperand::createImm(imm * 4));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3592-3593
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3595-3598
```cpp
static DecodeStatus DecodeT2AddrModeImm8s4(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeImm8s4`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeImm8s4`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3600-3601
```cpp
  unsigned Rn = fieldFromInstruction(Val, 9, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 9);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3603-3606
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeT2Imm8S4(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3608-3609
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3611-3614
```cpp
static DecodeStatus DecodeT2AddrModeImm7s4(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeImm7s4`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeImm7s4`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3616-3617
```cpp
  unsigned Rn = fieldFromInstruction(Val, 8, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3619-3622
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeT2Imm7S4(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3624-3625
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3627-3630
```cpp
static DecodeStatus DecodeT2AddrModeImm0_1020s4(MCInst &Inst, unsigned Val,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeImm0_1020s4`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeImm0_1020s4`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3632-3633
```cpp
  unsigned Rn = fieldFromInstruction(Val, 8, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3635-3636
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3638-3638
```cpp
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3640-3641
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3643-3653
```cpp
template <int shift>
static DecodeStatus DecodeT2Imm7(MCInst &Inst, unsigned Val, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  int imm = Val & 0x7F;
  if (Val == 0)
    imm = INT32_MIN;
  else if (!(Val & 0x80))
    imm *= -1;
  if (imm != INT32_MIN)
    imm *= (1U << shift);
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Implements `DecodeT2Imm7`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2Imm7`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3655-3656
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3658-3662
```cpp
template <int shift>
static DecodeStatus DecodeTAddrModeImm7(MCInst &Inst, unsigned Val,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeTAddrModeImm7`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeTAddrModeImm7`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3664-3665
```cpp
  unsigned Rn = fieldFromInstruction(Val, 8, 3);
  unsigned imm = fieldFromInstruction(Val, 0, 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3667-3670
```cpp
  if (!Check(S, DecodetGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeT2Imm7<shift>(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3672-3673
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3675-3679
```cpp
template <int shift, int WriteBack>
static DecodeStatus DecodeT2AddrModeImm7(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2AddrModeImm7`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddrModeImm7`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3681-3689
```cpp
  unsigned Rn = fieldFromInstruction(Val, 8, 4);
  unsigned imm = fieldFromInstruction(Val, 0, 8);
  if (WriteBack) {
    if (!Check(S, DecoderGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  } else if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeT2Imm7<shift>(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3691-3692
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3694-3697
```cpp
static DecodeStatus DecodeT2LdStPre(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LdStPre`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LdStPre`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3699-3704
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned addr = fieldFromInstruction(Insn, 0, 8);
  addr |= fieldFromInstruction(Insn, 9, 1) << 8;
  addr |= Rn << 9;
  unsigned load = fieldFromInstruction(Insn, 20, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3706-3720
```cpp
  if (Rn == 15) {
    switch (Inst.getOpcode()) {
    case ARM::t2LDR_PRE:
    case ARM::t2LDR_POST:
      Inst.setOpcode(ARM::t2LDRpci);
      break;
    case ARM::t2LDRB_PRE:
    case ARM::t2LDRB_POST:
      Inst.setOpcode(ARM::t2LDRBpci);
      break;
    case ARM::t2LDRH_PRE:
    case ARM::t2LDRH_POST:
      Inst.setOpcode(ARM::t2LDRHpci);
      break;
    case ARM::t2LDRSB_PRE:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3721-3735
```cpp
    case ARM::t2LDRSB_POST:
      if (Rt == 15)
        Inst.setOpcode(ARM::t2PLIpci);
      else
        Inst.setOpcode(ARM::t2LDRSBpci);
      break;
    case ARM::t2LDRSH_PRE:
    case ARM::t2LDRSH_POST:
      Inst.setOpcode(ARM::t2LDRSHpci);
      break;
    default:
      return MCDisassembler::Fail;
    }
    return DecodeT2LoadLabel(Inst, Insn, Address, Decoder);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3737-3740
```cpp
  if (!load) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3742-3743
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3745-3748
```cpp
  if (load) {
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3750-3751
```cpp
  if (!Check(S, DecodeT2AddrModeImm8(Inst, addr, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3753-3755
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3757-3760
```cpp
static DecodeStatus DecodeThumbAddSPImm(MCInst &Inst, uint16_t Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  unsigned imm = fieldFromInstruction(Insn, 0, 7);
```
- EN: Implements `DecodeThumbAddSPImm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddSPImm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3762-3764
```cpp
  Inst.addOperand(MCOperand::createReg(ARM::SP));
  Inst.addOperand(MCOperand::createReg(ARM::SP));
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3766-3768
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3770-3773
```cpp
static DecodeStatus DecodeThumbAddSPReg(MCInst &Inst, uint16_t Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeThumbAddSPReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbAddSPReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3775-3777
```cpp
  if (Inst.getOpcode() == ARM::tADDrSP) {
    unsigned Rdm = fieldFromInstruction(Insn, 0, 3);
    Rdm |= fieldFromInstruction(Insn, 7, 1) << 3;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3779-3785
```cpp
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rdm, Address, Decoder)))
      return MCDisassembler::Fail;
    Inst.addOperand(MCOperand::createReg(ARM::SP));
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rdm, Address, Decoder)))
      return MCDisassembler::Fail;
  } else if (Inst.getOpcode() == ARM::tADDspr) {
    unsigned Rm = fieldFromInstruction(Insn, 3, 4);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3787-3791
```cpp
    Inst.addOperand(MCOperand::createReg(ARM::SP));
    Inst.addOperand(MCOperand::createReg(ARM::SP));
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3793-3795
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3797-3801
```cpp
static DecodeStatus DecodeThumbCPS(MCInst &Inst, uint16_t Insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  unsigned imod = fieldFromInstruction(Insn, 4, 1) | 0x2;
  unsigned flags = fieldFromInstruction(Insn, 0, 3);
```
- EN: Implements `DecodeThumbCPS`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbCPS`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3803-3804
```cpp
  Inst.addOperand(MCOperand::createImm(imod));
  Inst.addOperand(MCOperand::createImm(flags));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3806-3807
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3809-3814
```cpp
static DecodeStatus DecodePostIdxReg(MCInst &Inst, unsigned Insn,
                                     uint64_t Address,
                                     const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned add = fieldFromInstruction(Insn, 4, 1);
```
- EN: Implements `DecodePostIdxReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePostIdxReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3816-3818
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(add));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3820-3821
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3823-3828
```cpp
static DecodeStatus DecodeMveAddrModeRQ(MCInst &Inst, unsigned Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Rn = fieldFromInstruction(Insn, 3, 4);
  unsigned Qm = fieldFromInstruction(Insn, 0, 3);
```
- EN: Implements `DecodeMveAddrModeRQ`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMveAddrModeRQ`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3830-3833
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3835-3836
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3838-3844
```cpp
template <int shift>
static DecodeStatus DecodeMveAddrModeQ(MCInst &Inst, unsigned Insn,
                                       uint64_t Address,
                                       const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Qm = fieldFromInstruction(Insn, 8, 3);
  int imm = fieldFromInstruction(Insn, 0, 7);
```
- EN: Implements `DecodeMveAddrModeQ`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMveAddrModeQ`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3846-3847
```cpp
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3849-3857
```cpp
  if(!fieldFromInstruction(Insn, 7, 1)) {
    if (imm == 0)
      imm = INT32_MIN;                 // indicate -0
    else
      imm *= -1;
  }
  if (imm != INT32_MIN)
    imm *= (1U << shift);
  Inst.addOperand(MCOperand::createImm(imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3859-3860
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3862-3876
```cpp
static DecodeStatus DecodeThumbBLXOffset(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  // Val is passed in as S:J1:J2:imm10H:imm10L:'0'
  // Note only one trailing zero not two.  Also the J1 and J2 values are from
  // the encoded instruction.  So here change to I1 and I2 values via:
  // I1 = NOT(J1 EOR S);
  // I2 = NOT(J2 EOR S);
  // and build the imm32 with two trailing zeros as documented:
  // imm32 = SignExtend(S:I1:I2:imm10H:imm10L:'00', 32);
  unsigned S = (Val >> 23) & 1;
  unsigned J1 = (Val >> 22) & 1;
  unsigned J2 = (Val >> 21) & 1;
  unsigned I1 = !(J1 ^ S);
  unsigned I2 = !(J2 ^ S);
```
- EN: Implements `DecodeThumbBLXOffset`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbBLXOffset`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3877-3878
```cpp
  unsigned tmp = (Val & ~0x600000) | (I1 << 22) | (I2 << 21);
  int imm32 = SignExtend32<25>(tmp << 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3880-3885
```cpp
  if (!tryAddingSymbolicOperand(Address,
                                (Address & ~2u) + imm32 + 4,
                                true, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(imm32));
  return MCDisassembler::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3887-3891
```cpp
static DecodeStatus DecodeCoprocessor(MCInst &Inst, unsigned Val,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  if (Val == 0xA || Val == 0xB)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeCoprocessor`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeCoprocessor`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3893-3894
```cpp
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3896-3897
```cpp
  if (!isValidCoprocessorNumber(Val, featureBits))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3899-3901
```cpp
  Inst.addOperand(MCOperand::createImm(Val));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3903-3908
```cpp
static DecodeStatus DecodeThumbTableBranch(MCInst &Inst, unsigned Insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  const FeatureBitset &FeatureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeThumbTableBranch`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbTableBranch`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3910-3911
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3913-3920
```cpp
  if (Rn == 13 && !FeatureBits[ARM::HasV8Ops]) S = MCDisassembler::SoftFail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3922-3926
```cpp
static DecodeStatus DecodeMemBarrierOption(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  if (Val & ~0xf)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMemBarrierOption`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMemBarrierOption`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3928-3930
```cpp
  Inst.addOperand(MCOperand::createImm(Val));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3932-3935
```cpp
static DecodeStatus DecodeThumb2BCCInstruction(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeThumb2BCCInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumb2BCCInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3937-3951
```cpp
  unsigned pred = fieldFromInstruction(Insn, 22, 4);
  if (pred == 0xE || pred == 0xF) {
    unsigned opc = fieldFromInstruction(Insn, 4, 28);
    switch (opc) {
      default:
        return MCDisassembler::Fail;
      case 0xf3bf8f4:
        Inst.setOpcode(ARM::t2DSB);
        break;
      case 0xf3bf8f5:
        Inst.setOpcode(ARM::t2DMB);
        break;
      case 0xf3bf8f6:
        Inst.setOpcode(ARM::t2ISB);
        break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3952-3952
```cpp
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3954-3956
```cpp
    unsigned imm = fieldFromInstruction(Insn, 0, 4);
    return DecodeMemBarrierOption(Inst, imm, Address, Decoder);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3958-3962
```cpp
  unsigned brtarget = fieldFromInstruction(Insn, 0, 11) << 1;
  brtarget |= fieldFromInstruction(Insn, 11, 1) << 19;
  brtarget |= fieldFromInstruction(Insn, 13, 1) << 18;
  brtarget |= fieldFromInstruction(Insn, 16, 6) << 12;
  brtarget |= fieldFromInstruction(Insn, 26, 1) << 20;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3964-3967
```cpp
  if (!Check(S, DecodeT2BROperand(Inst, brtarget, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 3969-3970
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 3972-3986
```cpp
// Decode a shifted immediate operand.  These basically consist
// of an 8-bit value, and a 4-bit directive that specifies either
// a splat operation or a rotation.
static DecodeStatus DecodeT2SOImm(MCInst &Inst, unsigned Val, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  unsigned ctrl = fieldFromInstruction(Val, 10, 2);
  if (ctrl == 0) {
    unsigned byte = fieldFromInstruction(Val, 8, 2);
    unsigned imm = fieldFromInstruction(Val, 0, 8);
    switch (byte) {
      case 0:
        Inst.addOperand(MCOperand::createImm(imm));
        break;
      case 1:
        Inst.addOperand(MCOperand::createImm((imm << 16) | imm));
```
- EN: Implements `DecodeT2SOImm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2SOImm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 3987-4001
```cpp
        break;
      case 2:
        Inst.addOperand(MCOperand::createImm((imm << 24) | (imm << 8)));
        break;
      case 3:
        Inst.addOperand(MCOperand::createImm((imm << 24) | (imm << 16) |
                                             (imm << 8)  |  imm));
        break;
    }
  } else {
    unsigned unrot = fieldFromInstruction(Val, 0, 7) | 0x80;
    unsigned rot = fieldFromInstruction(Val, 7, 5);
    unsigned imm = llvm::rotr<uint32_t>(unrot, rot);
    Inst.addOperand(MCOperand::createImm(imm));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4003-4004
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4006-4013
```cpp
static DecodeStatus DecodeThumbBCCTargetOperand(MCInst &Inst, unsigned Val,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if (!tryAddingSymbolicOperand(Address, Address + SignExtend32<9>(Val<<1) + 4,
                                true, 2, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(SignExtend32<9>(Val << 1)));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeThumbBCCTargetOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbBCCTargetOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4015-4029
```cpp
static DecodeStatus DecodeThumbBLTargetOperand(MCInst &Inst, unsigned Val,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  // Val is passed in as S:J1:J2:imm10:imm11
  // Note no trailing zero after imm11.  Also the J1 and J2 values are from
  // the encoded instruction.  So here change to I1 and I2 values via:
  // I1 = NOT(J1 EOR S);
  // I2 = NOT(J2 EOR S);
  // and build the imm32 with one trailing zero as documented:
  // imm32 = SignExtend(S:I1:I2:imm10:imm11:'0', 32);
  unsigned S = (Val >> 23) & 1;
  unsigned J1 = (Val >> 22) & 1;
  unsigned J2 = (Val >> 21) & 1;
  unsigned I1 = !(J1 ^ S);
  unsigned I2 = !(J2 ^ S);
```
- EN: Implements `DecodeThumbBLTargetOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeThumbBLTargetOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4030-4031
```cpp
  unsigned tmp = (Val & ~0x600000) | (I1 << 22) | (I2 << 21);
  int imm32 = SignExtend32<25>(tmp << 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4033-4037
```cpp
  if (!tryAddingSymbolicOperand(Address, Address + imm32 + 4,
                                true, 4, Inst, Decoder))
    Inst.addOperand(MCOperand::createImm(imm32));
  return MCDisassembler::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4039-4043
```cpp
static DecodeStatus DecodeInstSyncBarrierOption(MCInst &Inst, unsigned Val,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if (Val & ~0xf)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeInstSyncBarrierOption`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeInstSyncBarrierOption`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4045-4047
```cpp
  Inst.addOperand(MCOperand::createImm(Val));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4049-4053
```cpp
static DecodeStatus DecodeMSRMask(MCInst &Inst, unsigned Val, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  const FeatureBitset &FeatureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Implements `DecodeMSRMask`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMSRMask`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4055-4056
```cpp
  if (FeatureBits[ARM::FeatureMClass]) {
    unsigned ValLow = Val & 0xff;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4058-4072
```cpp
    // Validate the SYSm value first.
    switch (ValLow) {
    case  0: // apsr
    case  1: // iapsr
    case  2: // eapsr
    case  3: // xpsr
    case  5: // ipsr
    case  6: // epsr
    case  7: // iepsr
    case  8: // msp
    case  9: // psp
    case 16: // primask
    case 20: // control
      break;
    case 17: // basepri
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4073-4087
```cpp
    case 18: // basepri_max
    case 19: // faultmask
      if (!(FeatureBits[ARM::HasV7Ops]))
        // Values basepri, basepri_max and faultmask are only valid for v7m.
        return MCDisassembler::Fail;
      break;
    case 0x8a: // msplim_ns
    case 0x8b: // psplim_ns
    case 0x91: // basepri_ns
    case 0x93: // faultmask_ns
      if (!(FeatureBits[ARM::HasV8MMainlineOps]))
        return MCDisassembler::Fail;
      [[fallthrough]];
    case 10:   // msplim
    case 11:   // psplim
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4088-4102
```cpp
    case 0x88: // msp_ns
    case 0x89: // psp_ns
    case 0x90: // primask_ns
    case 0x94: // control_ns
    case 0x98: // sp_ns
      if (!(FeatureBits[ARM::Feature8MSecExt]))
        return MCDisassembler::Fail;
      break;
    case 0x20: // pac_key_p_0
    case 0x21: // pac_key_p_1
    case 0x22: // pac_key_p_2
    case 0x23: // pac_key_p_3
    case 0x24: // pac_key_u_0
    case 0x25: // pac_key_u_1
    case 0x26: // pac_key_u_2
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4103-4117
```cpp
    case 0x27: // pac_key_u_3
    case 0xa0: // pac_key_p_0_ns
    case 0xa1: // pac_key_p_1_ns
    case 0xa2: // pac_key_p_2_ns
    case 0xa3: // pac_key_p_3_ns
    case 0xa4: // pac_key_u_0_ns
    case 0xa5: // pac_key_u_1_ns
    case 0xa6: // pac_key_u_2_ns
    case 0xa7: // pac_key_u_3_ns
      if (!(FeatureBits[ARM::FeaturePACBTI]))
        return MCDisassembler::Fail;
      break;
    default:
      // Architecturally defined as unpredictable
      S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4118-4119
```cpp
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4121-4135
```cpp
    if (Inst.getOpcode() == ARM::t2MSR_M) {
      unsigned Mask = fieldFromInstruction(Val, 10, 2);
      if (!(FeatureBits[ARM::HasV7Ops])) {
        // The ARMv6-M MSR bits {11-10} can be only 0b10, other values are
        // unpredictable.
        if (Mask != 2)
          S = MCDisassembler::SoftFail;
      }
      else {
        // The ARMv7-M architecture stores an additional 2-bit mask value in
        // MSR bits {11-10}. The mask is used only with apsr, iapsr, eapsr and
        // xpsr, it has to be 0b10 in other cases. Bit mask{1} indicates if
        // the NZCVQ bits should be moved by the instruction. Bit mask{0}
        // indicates the move for the GE{3:0} bits, the mask{0} bit can be set
        // only if the processor includes the DSP extension.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4136-4148
```cpp
        if (Mask == 0 || (Mask != 2 && ValLow > 3) ||
            (!(FeatureBits[ARM::FeatureDSP]) && (Mask & 1)))
          S = MCDisassembler::SoftFail;
      }
    }
  } else {
    // A/R class
    if (Val == 0)
      return MCDisassembler::Fail;
  }
  Inst.addOperand(MCOperand::createImm(Val));
  return S;
}
```
- EN: Declares `if`, packaging target-specific state and APIs around `ARMDisassembler`.
- CN: 这里声明 `if`，把与 `ARMDisassembler` 相关的目标特定状态和 API 组织在一起。

### Lines 4150-4154
```cpp
static DecodeStatus DecodeBankedReg(MCInst &Inst, unsigned Val,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  unsigned R = fieldFromInstruction(Val, 5, 1);
  unsigned SysM = fieldFromInstruction(Val, 0, 5);
```
- EN: Implements `DecodeBankedReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBankedReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4156-4160
```cpp
  // The table of encodings for these banked registers comes from B9.2.3 of the
  // ARM ARM. There are patterns, but nothing regular enough to make this logic
  // neater. So by fiat, these values are UNPREDICTABLE:
  if (!ARMBankedReg::lookupBankedRegByEncoding((R << 5) | SysM))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4162-4164
```cpp
  Inst.addOperand(MCOperand::createImm(Val));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4166-4169
```cpp
static DecodeStatus DecodeDoubleRegLoad(MCInst &Inst, unsigned Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeDoubleRegLoad`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDoubleRegLoad`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4171-4173
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4175-4176
```cpp
  if (Rn == 0xF)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4178-4183
```cpp
  if (!Check(S, DecodeGPRPairRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4185-4186
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4188-4191
```cpp
static DecodeStatus DecodeDoubleRegStore(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeDoubleRegStore`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeDoubleRegStore`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4193-4196
```cpp
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  unsigned Rt = fieldFromInstruction(Insn, 0, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4198-4199
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4201-4202
```cpp
  if (Rn == 0xF || Rd == Rn || Rd == Rt || Rd == Rt+1)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4204-4209
```cpp
  if (!Check(S, DecodeGPRPairRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4211-4212
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4214-4217
```cpp
static DecodeStatus DecodeLDRPreImm(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeLDRPreImm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLDRPreImm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4219-4224
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 12);
  imm |= fieldFromInstruction(Insn, 16, 4) << 13;
  imm |= fieldFromInstruction(Insn, 23, 1) << 12;
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4226-4226
```cpp
  if (Rn == 0xF || Rn == Rt) S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4228-4235
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeAddrModeImm12Operand(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4237-4238
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4240-4243
```cpp
static DecodeStatus DecodeLDRPreReg(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeLDRPreReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLDRPreReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4245-4251
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 12);
  imm |= fieldFromInstruction(Insn, 16, 4) << 13;
  imm |= fieldFromInstruction(Insn, 23, 1) << 12;
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4253-4254
```cpp
  if (Rn == 0xF || Rn == Rt) S = MCDisassembler::SoftFail;
  if (Rm == 0xF) S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4256-4263
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeSORegMemOperand(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4265-4266
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4268-4271
```cpp
static DecodeStatus DecodeSTRPreImm(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSTRPreImm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSTRPreImm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4273-4278
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 12);
  imm |= fieldFromInstruction(Insn, 16, 4) << 13;
  imm |= fieldFromInstruction(Insn, 23, 1) << 12;
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4280-4280
```cpp
  if (Rn == 0xF || Rn == Rt) S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4282-4289
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeAddrModeImm12Operand(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4291-4292
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4294-4297
```cpp
static DecodeStatus DecodeSTRPreReg(MCInst &Inst, unsigned Insn,
                                    uint64_t Address,
                                    const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeSTRPreReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSTRPreReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4299-4304
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned imm = fieldFromInstruction(Insn, 0, 12);
  imm |= fieldFromInstruction(Insn, 16, 4) << 13;
  imm |= fieldFromInstruction(Insn, 23, 1) << 12;
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4306-4306
```cpp
  if (Rn == 0xF || Rn == Rt) S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4308-4315
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeSORegMemOperand(Inst, imm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4317-4318
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4320-4322
```cpp
static DecodeStatus DecodeVLD1LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD1LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD1LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4324-4328
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4330-4344
```cpp
  unsigned align = 0;
  unsigned index = 0;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      if (fieldFromInstruction(Insn, 4, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 5, 3);
      break;
    case 1:
      if (fieldFromInstruction(Insn, 5, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 6, 2);
      if (fieldFromInstruction(Insn, 4, 1))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4345-4350
```cpp
        align = 2;
      break;
    case 2:
      if (fieldFromInstruction(Insn, 6, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 7, 1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4352-4361
```cpp
      switch (fieldFromInstruction(Insn, 4, 2)) {
        case 0 :
          align = 0; break;
        case 3:
          align = 4; break;
        default:
          return MCDisassembler::Fail;
      }
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4363-4377
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
        return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4378-4378
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4380-4382
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4384-4386
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4388-4390
```cpp
static DecodeStatus DecodeVST1LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVST1LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVST1LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4392-4396
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4398-4412
```cpp
  unsigned align = 0;
  unsigned index = 0;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      if (fieldFromInstruction(Insn, 4, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 5, 3);
      break;
    case 1:
      if (fieldFromInstruction(Insn, 5, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 6, 2);
      if (fieldFromInstruction(Insn, 4, 1))
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4413-4418
```cpp
        align = 2;
      break;
    case 2:
      if (fieldFromInstruction(Insn, 6, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 7, 1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4420-4429
```cpp
      switch (fieldFromInstruction(Insn, 4, 2)) {
        case 0:
          align = 0; break;
        case 3:
          align = 4; break;
        default:
          return MCDisassembler::Fail;
      }
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4431-4444
```cpp
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4446-4448
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4450-4452
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4454-4456
```cpp
static DecodeStatus DecodeVLD2LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD2LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD2LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4458-4462
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4464-4478
```cpp
  unsigned align = 0;
  unsigned index = 0;
  unsigned inc = 1;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      index = fieldFromInstruction(Insn, 5, 3);
      if (fieldFromInstruction(Insn, 4, 1))
        align = 2;
      break;
    case 1:
      index = fieldFromInstruction(Insn, 6, 2);
      if (fieldFromInstruction(Insn, 4, 1))
        align = 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4479-4491
```cpp
      if (fieldFromInstruction(Insn, 5, 1))
        inc = 2;
      break;
    case 2:
      if (fieldFromInstruction(Insn, 5, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 7, 1);
      if (fieldFromInstruction(Insn, 4, 1) != 0)
        align = 8;
      if (fieldFromInstruction(Insn, 6, 1))
        inc = 2;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4493-4507
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
        return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4508-4510
```cpp
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4512-4516
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4518-4520
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4522-4524
```cpp
static DecodeStatus DecodeVST2LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVST2LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVST2LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4526-4530
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4532-4546
```cpp
  unsigned align = 0;
  unsigned index = 0;
  unsigned inc = 1;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      index = fieldFromInstruction(Insn, 5, 3);
      if (fieldFromInstruction(Insn, 4, 1))
        align = 2;
      break;
    case 1:
      index = fieldFromInstruction(Insn, 6, 2);
      if (fieldFromInstruction(Insn, 4, 1))
        align = 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4547-4559
```cpp
      if (fieldFromInstruction(Insn, 5, 1))
        inc = 2;
      break;
    case 2:
      if (fieldFromInstruction(Insn, 5, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 7, 1);
      if (fieldFromInstruction(Insn, 4, 1) != 0)
        align = 8;
      if (fieldFromInstruction(Insn, 6, 1))
        inc = 2;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4561-4574
```cpp
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
        return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4576-4580
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4582-4584
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4586-4588
```cpp
static DecodeStatus DecodeVLD3LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD3LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD3LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4590-4594
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4596-4610
```cpp
  unsigned align = 0;
  unsigned index = 0;
  unsigned inc = 1;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      if (fieldFromInstruction(Insn, 4, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 5, 3);
      break;
    case 1:
      if (fieldFromInstruction(Insn, 4, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 6, 2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4611-4621
```cpp
      if (fieldFromInstruction(Insn, 5, 1))
        inc = 2;
      break;
    case 2:
      if (fieldFromInstruction(Insn, 4, 2))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 7, 1);
      if (fieldFromInstruction(Insn, 6, 1))
        inc = 2;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4623-4628
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+2*inc, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4630-4643
```cpp
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4645-4651
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+2*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4653-4655
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4657-4659
```cpp
static DecodeStatus DecodeVST3LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVST3LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVST3LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4661-4665
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4667-4681
```cpp
  unsigned align = 0;
  unsigned index = 0;
  unsigned inc = 1;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      if (fieldFromInstruction(Insn, 4, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 5, 3);
      break;
    case 1:
      if (fieldFromInstruction(Insn, 4, 1))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 6, 2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4682-4692
```cpp
      if (fieldFromInstruction(Insn, 5, 1))
        inc = 2;
      break;
    case 2:
      if (fieldFromInstruction(Insn, 4, 2))
        return MCDisassembler::Fail; // UNDEFINED
      index = fieldFromInstruction(Insn, 7, 1);
      if (fieldFromInstruction(Insn, 6, 1))
        inc = 2;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4694-4707
```cpp
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4709-4715
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+2*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4717-4719
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4721-4723
```cpp
static DecodeStatus DecodeVLD4LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVLD4LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVLD4LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4725-4729
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4731-4745
```cpp
  unsigned align = 0;
  unsigned index = 0;
  unsigned inc = 1;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      if (fieldFromInstruction(Insn, 4, 1))
        align = 4;
      index = fieldFromInstruction(Insn, 5, 3);
      break;
    case 1:
      if (fieldFromInstruction(Insn, 4, 1))
        align = 8;
      index = fieldFromInstruction(Insn, 6, 2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4746-4757
```cpp
      if (fieldFromInstruction(Insn, 5, 1))
        inc = 2;
      break;
    case 2:
      switch (fieldFromInstruction(Insn, 4, 2)) {
        case 0:
          align = 0; break;
        case 3:
          return MCDisassembler::Fail;
        default:
          align = 4 << fieldFromInstruction(Insn, 4, 2); break;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4759-4763
```cpp
      index = fieldFromInstruction(Insn, 7, 1);
      if (fieldFromInstruction(Insn, 6, 1))
        inc = 2;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4765-4772
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+2*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+3*inc, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4774-4787
```cpp
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
        return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4789-4797
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+2*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+3*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4799-4801
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4803-4805
```cpp
static DecodeStatus DecodeVST4LN(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVST4LN`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVST4LN`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4807-4811
```cpp
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm = fieldFromInstruction(Insn, 0, 4);
  unsigned Rd = fieldFromInstruction(Insn, 12, 4);
  Rd |= fieldFromInstruction(Insn, 22, 1) << 4;
  unsigned size = fieldFromInstruction(Insn, 10, 2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4813-4827
```cpp
  unsigned align = 0;
  unsigned index = 0;
  unsigned inc = 1;
  switch (size) {
    default:
      return MCDisassembler::Fail;
    case 0:
      if (fieldFromInstruction(Insn, 4, 1))
        align = 4;
      index = fieldFromInstruction(Insn, 5, 3);
      break;
    case 1:
      if (fieldFromInstruction(Insn, 4, 1))
        align = 8;
      index = fieldFromInstruction(Insn, 6, 2);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4828-4839
```cpp
      if (fieldFromInstruction(Insn, 5, 1))
        inc = 2;
      break;
    case 2:
      switch (fieldFromInstruction(Insn, 4, 2)) {
        case 0:
          align = 0; break;
        case 3:
          return MCDisassembler::Fail;
        default:
          align = 4 << fieldFromInstruction(Insn, 4, 2); break;
      }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4841-4845
```cpp
      index = fieldFromInstruction(Insn, 7, 1);
      if (fieldFromInstruction(Insn, 6, 1))
        inc = 2;
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4847-4860
```cpp
  if (Rm != 0xF) { // Writeback
    if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(align));
  if (Rm != 0xF) {
    if (Rm != 0xD) {
      if (!Check(S, DecodeGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
    } else
      Inst.addOperand(MCOperand::createReg(0));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4862-4870
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+2*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Rd+3*inc, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(index));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4872-4874
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4876-4883
```cpp
static DecodeStatus DecodeVMOVSRR(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Rt  = fieldFromInstruction(Insn, 12, 4);
  unsigned Rt2 = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm  = fieldFromInstruction(Insn,  5, 1);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  Rm |= fieldFromInstruction(Insn, 0, 4) << 1;
```
- EN: Implements `DecodeVMOVSRR`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVMOVSRR`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4885-4886
```cpp
  if (Rt == 0xF || Rt2 == 0xF || Rm == 0x1F)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4888-4897
```cpp
  if (!Check(S, DecodeSPRRegisterClass(Inst, Rm  , Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeSPRRegisterClass(Inst, Rm+1, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt  , Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt2 , Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4899-4900
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4902-4909
```cpp
static DecodeStatus DecodeVMOVRRS(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Rt  = fieldFromInstruction(Insn, 12, 4);
  unsigned Rt2 = fieldFromInstruction(Insn, 16, 4);
  unsigned Rm  = fieldFromInstruction(Insn,  5, 1);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
  Rm |= fieldFromInstruction(Insn, 0, 4) << 1;
```
- EN: Implements `DecodeVMOVRRS`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVMOVRRS`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4911-4912
```cpp
  if (Rt == 0xF || Rt2 == 0xF || Rm == 0x1F)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4914-4923
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt  , Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt2 , Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeSPRRegisterClass(Inst, Rm  , Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeSPRRegisterClass(Inst, Rm+1, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4925-4926
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4928-4932
```cpp
static DecodeStatus DecodeIT(MCInst &Inst, unsigned Insn, uint64_t Address,
                             const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned pred = fieldFromInstruction(Insn, 4, 4);
  unsigned mask = fieldFromInstruction(Insn, 0, 4);
```
- EN: Implements `DecodeIT`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeIT`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4934-4937
```cpp
  if (pred == 0xF) {
    pred = 0xE;
    S = MCDisassembler::SoftFail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4939-4940
```cpp
  if (mask == 0x0)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4942-4950
```cpp
  // IT masks are encoded as a sequence of replacement low-order bits
  // for the condition code. So if the low bit of the starting
  // condition code is 1, then we have to flip all the bits above the
  // terminating bit (which is the lowest 1 bit).
  if (pred & 1) {
    unsigned LowBit = mask & -mask;
    unsigned BitsAboveLowBit = 0xF & (-LowBit << 1);
    mask ^= BitsAboveLowBit;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4952-4955
```cpp
  Inst.addOperand(MCOperand::createImm(pred));
  Inst.addOperand(MCOperand::createImm(mask));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4957-4960
```cpp
static DecodeStatus DecodeT2LDRDPreInstruction(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2LDRDPreInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2LDRDPreInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 4962-4969
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rt2 = fieldFromInstruction(Insn, 8, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned addr = fieldFromInstruction(Insn, 0, 8);
  unsigned W = fieldFromInstruction(Insn, 21, 1);
  unsigned U = fieldFromInstruction(Insn, 23, 1);
  unsigned P = fieldFromInstruction(Insn, 24, 1);
  bool writeback = (W == 1) | (P == 0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4971-4971
```cpp
  addr |= (U << 8) | (Rn << 9);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4973-4976
```cpp
  if (writeback && (Rn == Rt || Rn == Rt2))
    Check(S, MCDisassembler::SoftFail);
  if (Rt == Rt2)
    Check(S, MCDisassembler::SoftFail);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4978-4989
```cpp
  // Rt
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  // Rt2
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rt2, Address, Decoder)))
    return MCDisassembler::Fail;
  // Writeback operand
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  // addr
  if (!Check(S, DecodeT2AddrModeImm8s4(Inst, addr, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 4991-4993
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 4995-4998
```cpp
static DecodeStatus DecodeT2STRDPreInstruction(MCInst &Inst, unsigned Insn,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2STRDPreInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2STRDPreInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5000-5007
```cpp
  unsigned Rt = fieldFromInstruction(Insn, 12, 4);
  unsigned Rt2 = fieldFromInstruction(Insn, 8, 4);
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  unsigned addr = fieldFromInstruction(Insn, 0, 8);
  unsigned W = fieldFromInstruction(Insn, 21, 1);
  unsigned U = fieldFromInstruction(Insn, 23, 1);
  unsigned P = fieldFromInstruction(Insn, 24, 1);
  bool writeback = (W == 1) | (P == 0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5009-5009
```cpp
  addr |= (U << 8) | (Rn << 9);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5011-5012
```cpp
  if (writeback && (Rn == Rt || Rn == Rt2))
    Check(S, MCDisassembler::SoftFail);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5014-5025
```cpp
  // Writeback operand
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  // Rt
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  // Rt2
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rt2, Address, Decoder)))
    return MCDisassembler::Fail;
  // addr
  if (!Check(S, DecodeT2AddrModeImm8s4(Inst, addr, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5027-5029
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5031-5038
```cpp
static DecodeStatus DecodeT2Adr(MCInst &Inst, uint32_t Insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  unsigned sign1 = fieldFromInstruction(Insn, 21, 1);
  unsigned sign2 = fieldFromInstruction(Insn, 23, 1);
  if (sign1 != sign2) return MCDisassembler::Fail;
  const unsigned Rd = fieldFromInstruction(Insn, 8, 4);
  assert(Inst.getNumOperands() == 0 && "We should receive an empty Inst");
  DecodeStatus S = DecoderGPRRegisterClass(Inst, Rd, Address, Decoder);
```
- EN: Implements `DecodeT2Adr`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2Adr`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5040-5054
```cpp
  unsigned Val = fieldFromInstruction(Insn, 0, 8);
  Val |= fieldFromInstruction(Insn, 12, 3) << 8;
  Val |= fieldFromInstruction(Insn, 26, 1) << 11;
  // If sign, then it is decreasing the address.
  if (sign1) {
    // Following ARMv7 Architecture Manual, when the offset
    // is zero, it is decoded as a subw, not as a adr.w
    if (!Val) {
      Inst.setOpcode(ARM::t2SUBri12);
      Inst.addOperand(MCOperand::createReg(ARM::PC));
    } else
      Val = -Val;
  }
  Inst.addOperand(MCOperand::createImm(Val));
  DecodePredicateOperand(Inst, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5055-5056
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5058-5061
```cpp
static DecodeStatus DecodeT2ShifterImmOperand(MCInst &Inst, uint32_t Val,
                                              uint64_t Address,
                                              const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeT2ShifterImmOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2ShifterImmOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5063-5067
```cpp
  // Shift of "asr #32" is not allowed in Thumb2 mode.
  if (Val == 0x20) S = MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(Val));
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5069-5074
```cpp
static DecodeStatus DecodeSwap(MCInst &Inst, unsigned Insn, uint64_t Address,
                               const MCDisassembler *Decoder) {
  unsigned Rt   = fieldFromInstruction(Insn, 12, 4);
  unsigned Rt2  = fieldFromInstruction(Insn, 0,  4);
  unsigned Rn   = fieldFromInstruction(Insn, 16, 4);
  unsigned pred = fieldFromInstruction(Insn, 28, 4);
```
- EN: Implements `DecodeSwap`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeSwap`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5076-5077
```cpp
  if (pred == 0xF)
    return DecodeCPSInstruction(Inst, Insn, Address, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5079-5079
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5081-5082
```cpp
  if (Rt == Rn || Rn == Rt2)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5084-5091
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt2, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5093-5094
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5096-5100
```cpp
static DecodeStatus DecodeVCVTD(MCInst &Inst, unsigned Insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
  bool hasFullFP16 = featureBits[ARM::FeatureFullFP16];
```
- EN: Implements `DecodeVCVTD`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVCVTD`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5102-5108
```cpp
  unsigned Vd = (fieldFromInstruction(Insn, 12, 4) << 0);
  Vd |= (fieldFromInstruction(Insn, 22, 1) << 4);
  unsigned Vm = (fieldFromInstruction(Insn, 0, 4) << 0);
  Vm |= (fieldFromInstruction(Insn, 5, 1) << 4);
  unsigned imm = fieldFromInstruction(Insn, 16, 6);
  unsigned cmode = fieldFromInstruction(Insn, 8, 4);
  unsigned op = fieldFromInstruction(Insn, 5, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5110-5110
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5112-5126
```cpp
  // If the top 3 bits of imm are clear, this is a VMOV (immediate)
  if (!(imm & 0x38)) {
    if (cmode == 0xF) {
      if (op == 1) return MCDisassembler::Fail;
      Inst.setOpcode(ARM::VMOVv2f32);
    }
    if (hasFullFP16) {
      if (cmode == 0xE) {
        if (op == 1) {
          Inst.setOpcode(ARM::VMOVv1i64);
        } else {
          Inst.setOpcode(ARM::VMOVv8i8);
        }
      }
      if (cmode == 0xD) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5127-5141
```cpp
        if (op == 1) {
          Inst.setOpcode(ARM::VMVNv2i32);
        } else {
          Inst.setOpcode(ARM::VMOVv2i32);
        }
      }
      if (cmode == 0xC) {
        if (op == 1) {
          Inst.setOpcode(ARM::VMVNv2i32);
        } else {
          Inst.setOpcode(ARM::VMOVv2i32);
        }
      }
    }
    return DecodeVMOVModImmInstruction(Inst, Insn, Address, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5142-5142
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5144-5144
```cpp
  if (!(imm & 0x20)) return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5146-5150
```cpp
  if (!Check(S, DecodeDPRRegisterClass(Inst, Vd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Vm, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(64 - imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5152-5154
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5156-5160
```cpp
static DecodeStatus DecodeVCVTQ(MCInst &Inst, unsigned Insn, uint64_t Address,
                                const MCDisassembler *Decoder) {
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
  bool hasFullFP16 = featureBits[ARM::FeatureFullFP16];
```
- EN: Implements `DecodeVCVTQ`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVCVTQ`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5162-5168
```cpp
  unsigned Vd = (fieldFromInstruction(Insn, 12, 4) << 0);
  Vd |= (fieldFromInstruction(Insn, 22, 1) << 4);
  unsigned Vm = (fieldFromInstruction(Insn, 0, 4) << 0);
  Vm |= (fieldFromInstruction(Insn, 5, 1) << 4);
  unsigned imm = fieldFromInstruction(Insn, 16, 6);
  unsigned cmode = fieldFromInstruction(Insn, 8, 4);
  unsigned op = fieldFromInstruction(Insn, 5, 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5170-5170
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5172-5186
```cpp
  // If the top 3 bits of imm are clear, this is a VMOV (immediate)
  if (!(imm & 0x38)) {
    if (cmode == 0xF) {
      if (op == 1) return MCDisassembler::Fail;
      Inst.setOpcode(ARM::VMOVv4f32);
    }
    if (hasFullFP16) {
      if (cmode == 0xE) {
        if (op == 1) {
          Inst.setOpcode(ARM::VMOVv2i64);
        } else {
          Inst.setOpcode(ARM::VMOVv16i8);
        }
      }
      if (cmode == 0xD) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5187-5201
```cpp
        if (op == 1) {
          Inst.setOpcode(ARM::VMVNv4i32);
        } else {
          Inst.setOpcode(ARM::VMOVv4i32);
        }
      }
      if (cmode == 0xC) {
        if (op == 1) {
          Inst.setOpcode(ARM::VMVNv4i32);
        } else {
          Inst.setOpcode(ARM::VMOVv4i32);
        }
      }
    }
    return DecodeVMOVModImmInstruction(Inst, Insn, Address, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5202-5202
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5204-5204
```cpp
  if (!(imm & 0x20)) return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5206-5210
```cpp
  if (!Check(S, DecodeQPRRegisterClass(Inst, Vd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeQPRRegisterClass(Inst, Vm, Address, Decoder)))
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(64 - imm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5212-5214
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5216-5227
```cpp
static DecodeStatus
DecodeNEONComplexLane64Instruction(MCInst &Inst, unsigned Insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  unsigned Vd = (fieldFromInstruction(Insn, 12, 4) << 0);
  Vd |= (fieldFromInstruction(Insn, 22, 1) << 4);
  unsigned Vn = (fieldFromInstruction(Insn, 16, 4) << 0);
  Vn |= (fieldFromInstruction(Insn, 7, 1) << 4);
  unsigned Vm = (fieldFromInstruction(Insn, 0, 4) << 0);
  Vm |= (fieldFromInstruction(Insn, 5, 1) << 4);
  unsigned q = (fieldFromInstruction(Insn, 6, 1) << 0);
  unsigned rotate = (fieldFromInstruction(Insn, 20, 2) << 0);
```
- EN: Implements `DecodeNEONComplexLane64Instruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeNEONComplexLane64Instruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5229-5229
```cpp
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5231-5231
```cpp
  auto DestRegDecoder = q ? DecodeQPRRegisterClass : DecodeDPRRegisterClass;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5233-5244
```cpp
  if (!Check(S, DestRegDecoder(Inst, Vd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DestRegDecoder(Inst, Vd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DestRegDecoder(Inst, Vn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeDPRRegisterClass(Inst, Vm, Address, Decoder)))
    return MCDisassembler::Fail;
  // The lane index does not have any bits in the encoding, because it can only
  // be 0.
  Inst.addOperand(MCOperand::createImm(0));
  Inst.addOperand(MCOperand::createImm(rotate));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5246-5247
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5249-5251
```cpp
static DecodeStatus DecodeLDR(MCInst &Inst, unsigned Val, uint64_t Address,
                              const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeLDR`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLDR`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5253-5257
```cpp
  unsigned Rn = fieldFromInstruction(Val, 16, 4);
  unsigned Rt = fieldFromInstruction(Val, 12, 4);
  unsigned Rm = fieldFromInstruction(Val, 0, 4);
  Rm |= (fieldFromInstruction(Val, 23, 1) << 4);
  unsigned Cond = fieldFromInstruction(Val, 28, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5259-5260
```cpp
  if (fieldFromInstruction(Val, 8, 4) != 0 || Rn == Rt)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5262-5271
```cpp
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeAddrMode7Operand(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePostIdxReg(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodePredicateOperand(Inst, Cond, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5273-5274
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5276-5279
```cpp
static DecodeStatus DecoderForMRRC2AndMCRR2(MCInst &Inst, unsigned Val,
                                            uint64_t Address,
                                            const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecoderForMRRC2AndMCRR2`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecoderForMRRC2AndMCRR2`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5281-5285
```cpp
  unsigned CRm = fieldFromInstruction(Val, 0, 4);
  unsigned opc1 = fieldFromInstruction(Val, 4, 4);
  unsigned cop = fieldFromInstruction(Val, 8, 4);
  unsigned Rt = fieldFromInstruction(Val, 12, 4);
  unsigned Rt2 = fieldFromInstruction(Val, 16, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5287-5288
```cpp
  if ((cop & ~0x1) == 0xa)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5290-5291
```cpp
  if (Rt == Rt2)
    S = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5293-5301
```cpp
  // We have to check if the instruction is MRRC2
  // or MCRR2 when constructing the operands for
  // Inst. Reason is because MRRC2 stores to two
  // registers so it's tablegen desc has two
  // outputs whereas MCRR doesn't store to any
  // registers so all of it's operands are listed
  // as inputs, therefore the operand order for
  // MRRC2 needs to be [Rt, Rt2, cop, opc1, CRm]
  // and MCRR2 operand order is [cop, opc1, Rt, Rt2, CRm]
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 5303-5317
```cpp
  if (Inst.getOpcode() == ARM::MRRC2) {
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt, Address, Decoder)))
      return MCDisassembler::Fail;
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt2, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  Inst.addOperand(MCOperand::createImm(cop));
  Inst.addOperand(MCOperand::createImm(opc1));
  if (Inst.getOpcode() == ARM::MCRR2) {
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt, Address, Decoder)))
      return MCDisassembler::Fail;
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rt2, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  Inst.addOperand(MCOperand::createImm(CRm));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5319-5320
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5322-5327
```cpp
static DecodeStatus DecodeForVMRSandVMSR(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  const FeatureBitset &featureBits =
      Decoder->getSubtargetInfo().getFeatureBits();
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeForVMRSandVMSR`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeForVMRSandVMSR`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5329-5338
```cpp
  // Add explicit operand for the destination sysreg, for cases where
  // we have to model it for code generation purposes.
  switch (Inst.getOpcode()) {
  case ARM::VMSR_FPSCR_NZCVQC:
    Inst.addOperand(MCOperand::createReg(ARM::FPSCR_NZCV));
    break;
  case ARM::VMSR_P0:
    Inst.addOperand(MCOperand::createReg(ARM::VPR));
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5340-5341
```cpp
  if (Inst.getOpcode() != ARM::FMSTAT) {
    unsigned Rt = fieldFromInstruction(Val, 12, 4);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5343-5349
```cpp
    if (featureBits[ARM::ModeThumb] && !featureBits[ARM::HasV8Ops]) {
      if (Rt == 13 || Rt == 15)
        S = MCDisassembler::SoftFail;
      Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder));
    } else
      Check(S, DecodeGPRnopcRegisterClass(Inst, Rt, Address, Decoder));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5351-5359
```cpp
  // Add explicit operand for the source sysreg, similarly to above.
  switch (Inst.getOpcode()) {
  case ARM::VMRS_FPSCR_NZCVQC:
    Inst.addOperand(MCOperand::createReg(ARM::FPSCR_NZCV));
    break;
  case ARM::VMRS_P0:
    Inst.addOperand(MCOperand::createReg(ARM::VPR));
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5361-5368
```cpp
  if (featureBits[ARM::ModeThumb]) {
    Inst.addOperand(MCOperand::createImm(ARMCC::AL));
    Inst.addOperand(MCOperand::createReg(0));
  } else {
    unsigned pred = fieldFromInstruction(Val, 28, 4);
    if (!Check(S, DecodePredicateOperand(Inst, pred, Address, Decoder)))
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5370-5371
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5373-5379
```cpp
template <bool isSigned, bool isNeg, bool zeroPermitted, int size>
static DecodeStatus DecodeBFLabelOperand(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  if (Val == 0 && !zeroPermitted)
    S = MCDisassembler::Fail;
```
- EN: Implements `DecodeBFLabelOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBFLabelOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5381-5385
```cpp
  uint64_t DecVal;
  if (isSigned)
    DecVal = SignExtend32<size + 1>(Val << 1);
  else
    DecVal = (Val << 1);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5387-5391
```cpp
  if (!tryAddingSymbolicOperand(Address, Address + DecVal + 4, true, 4, Inst,
                                Decoder))
    Inst.addOperand(MCOperand::createImm(isNeg ? -DecVal : DecVal));
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5393-5395
```cpp
static DecodeStatus DecodeBFAfterTargetOperand(MCInst &Inst, unsigned Val,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
```
- EN: Implements `DecodeBFAfterTargetOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeBFAfterTargetOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5397-5403
```cpp
  uint64_t LocImm = Inst.getOperand(0).getImm();
  Val = LocImm + (2 << Val);
  if (!tryAddingSymbolicOperand(Address, Address + Val + 4, true, 4, Inst,
                                Decoder))
    Inst.addOperand(MCOperand::createImm(Val));
  return MCDisassembler::Success;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5405-5412
```cpp
static DecodeStatus DecodePredNoALOperand(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  if (Val >= ARMCC::AL)  // also exclude the non-condition NV
    return MCDisassembler::Fail;
  Inst.addOperand(MCOperand::createImm(Val));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodePredNoALOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePredNoALOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5414-5416
```cpp
static DecodeStatus DecodeLOLoop(MCInst &Inst, unsigned Insn, uint64_t Address,
                                 const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeLOLoop`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLOLoop`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5418-5421
```cpp
  if (Inst.getOpcode() == ARM::MVE_LCTP) {
    DecodePredicateOperand(Inst, Decoder);
    return S;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5423-5437
```cpp
  unsigned Imm = fieldFromInstruction(Insn, 11, 1) |
                 fieldFromInstruction(Insn, 1, 10) << 1;
  switch (Inst.getOpcode()) {
  case ARM::t2LEUpdate:
  case ARM::MVE_LETP:
    Inst.addOperand(MCOperand::createReg(ARM::LR));
    Inst.addOperand(MCOperand::createReg(ARM::LR));
    [[fallthrough]];
  case ARM::t2LE:
    if (!Check(S, DecodeBFLabelOperand<false, true, true, 11>(
                   Inst, Imm, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  case ARM::t2WLS:
  case ARM::MVE_WLSTP_8:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5438-5452
```cpp
  case ARM::MVE_WLSTP_16:
  case ARM::MVE_WLSTP_32:
  case ARM::MVE_WLSTP_64:
    Inst.addOperand(MCOperand::createReg(ARM::LR));
    if (!Check(S,
               DecoderGPRRegisterClass(Inst, fieldFromInstruction(Insn, 16, 4),
                                       Address, Decoder)) ||
        !Check(S, DecodeBFLabelOperand<false, false, true, 11>(
                   Inst, Imm, Address, Decoder)))
      return MCDisassembler::Fail;
    break;
  case ARM::t2DLS:
  case ARM::MVE_DLSTP_8:
  case ARM::MVE_DLSTP_16:
  case ARM::MVE_DLSTP_32:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5453-5463
```cpp
  case ARM::MVE_DLSTP_64:
    unsigned Rn = fieldFromInstruction(Insn, 16, 4);
    if (Rn == 0xF) {
      // Enforce all the rest of the instruction bits in LCTP, which
      // won't have been reliably checked based on LCTP's own tablegen
      // record, because we came to this decode by a roundabout route.
      uint32_t CanonicalLCTP = 0xF00FE001, SBZMask = 0x00300FFE;
      if ((Insn & ~SBZMask) != CanonicalLCTP)
        return MCDisassembler::Fail;   // a mandatory bit is wrong: hard fail
      if (Insn != CanonicalLCTP)
        Check(S, MCDisassembler::SoftFail); // an SBZ bit is wrong: soft fail
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5465-5477
```cpp
      Inst.setOpcode(ARM::MVE_LCTP);
      DecodePredicateOperand(Inst, Decoder);
    } else {
      Inst.addOperand(MCOperand::createReg(ARM::LR));
      if (!Check(S, DecoderGPRRegisterClass(Inst,
                                            fieldFromInstruction(Insn, 16, 4),
                                            Address, Decoder)))
        return MCDisassembler::Fail;
    }
    break;
  }
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5479-5482
```cpp
static DecodeStatus DecodeLongShiftOperand(MCInst &Inst, unsigned Val,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeLongShiftOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLongShiftOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5484-5485
```cpp
  if (Val == 0)
    Val = 32;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5487-5487
```cpp
  Inst.addOperand(MCOperand::createImm(Val));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5489-5490
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5492-5496
```cpp
static DecodeStatus DecodetGPROddRegisterClass(MCInst &Inst, unsigned RegNo,
                                               uint64_t Address,
                                               const MCDisassembler *Decoder) {
  if ((RegNo) + 1 > 11)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodetGPROddRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodetGPROddRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5498-5501
```cpp
  unsigned Register = GPRDecoderTable[(RegNo) + 1];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5503-5507
```cpp
static DecodeStatus DecodetGPREvenRegisterClass(MCInst &Inst, unsigned RegNo,
                                                uint64_t Address,
                                                const MCDisassembler *Decoder) {
  if ((RegNo) > 14)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodetGPREvenRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodetGPREvenRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5509-5512
```cpp
  unsigned Register = GPRDecoderTable[(RegNo)];
  Inst.addOperand(MCOperand::createReg(Register));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5514-5521
```cpp
static DecodeStatus
DecodeGPRwithAPSR_NZCVnospRegisterClass(MCInst &Inst, unsigned RegNo,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  if (RegNo == 15) {
    Inst.addOperand(MCOperand::createReg(ARM::APSR_NZCV));
    return MCDisassembler::Success;
  }
```
- EN: Implements `DecodeGPRwithAPSR_NZCVnospRegisterClass`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeGPRwithAPSR_NZCVnospRegisterClass`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5523-5524
```cpp
  unsigned Register = GPRDecoderTable[RegNo];
  Inst.addOperand(MCOperand::createReg(Register));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5526-5527
```cpp
  if (RegNo == 13)
    return MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5529-5530
```cpp
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5532-5534
```cpp
static DecodeStatus DecodeVSCCLRM(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVSCCLRM`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVSCCLRM`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5536-5550
```cpp
  Inst.addOperand(MCOperand::createImm(ARMCC::AL));
  Inst.addOperand(MCOperand::createReg(0));
  unsigned regs = fieldFromInstruction(Insn, 0, 8);
  if (regs == 0) {
    // Register list contains only VPR
  } else if (Inst.getOpcode() == ARM::VSCCLRMD) {
    unsigned reglist = regs | (fieldFromInstruction(Insn, 12, 4) << 8) |
                       (fieldFromInstruction(Insn, 22, 1) << 12);
    if (!Check(S, DecodeDPRRegListOperand(Inst, reglist, Address, Decoder))) {
      return MCDisassembler::Fail;
    }
  } else {
    unsigned Vd = (fieldFromInstruction(Insn, 12, 4) << 1) |
                  fieldFromInstruction(Insn, 22, 1);
    // Registers past s31 are permitted and treated as being half of a d
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5551-5564
```cpp
    // register, though both halves of each d register must be present.
    unsigned max_reg = Vd + regs;
    if (max_reg > 64 || (max_reg > 32 && (max_reg & 1)))
      S = MCDisassembler::SoftFail;
    unsigned max_sreg = std::min(32u, max_reg);
    unsigned max_dreg = std::min(32u, max_reg / 2);
    for (unsigned i = Vd; i < max_sreg; ++i)
      if (!Check(S, DecodeSPRRegisterClass(Inst, i, Address, Decoder)))
        return MCDisassembler::Fail;
    for (unsigned i = 16; i < max_dreg; ++i)
      if (!Check(S, DecodeDPRRegisterClass(Inst, i, Address, Decoder)))
        return MCDisassembler::Fail;
  }
  Inst.addOperand(MCOperand::createReg(ARM::VPR));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5566-5567
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5569-5572
```cpp
static DecodeStatus DecodeVPTMaskOperand(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVPTMaskOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVPTMaskOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5574-5583
```cpp
  // Parse VPT mask and encode it in the MCInst as an immediate with the same
  // format as the it_mask.  That is, from the second 'e|t' encode 'e' as 1 and
  // 't' as 0 and finish with a 1.
  unsigned Imm = 0;
  // We always start with a 't'.
  unsigned CurBit = 0;
  for (int i = 3; i >= 0; --i) {
    // If the bit we are looking at is not the same as last one, invert the
    // CurBit, if it is the same leave it as is.
    CurBit ^= (Val >> i) & 1U;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5585-5586
```cpp
    // Encode the CurBit at the right place in the immediate.
    Imm |= (CurBit << i);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5588-5593
```cpp
    // If we are done, finish the encoding with a 1.
    if ((Val & ~(~0U << i)) == 0) {
      Imm |= 1U << i;
      break;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5595-5595
```cpp
  Inst.addOperand(MCOperand::createImm(Imm));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5597-5598
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5600-5605
```cpp
static DecodeStatus
DecodeRestrictedIPredicateOperand(MCInst &Inst, unsigned Val, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm((Val & 0x1) == 0 ? ARMCC::EQ : ARMCC::NE));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeRestrictedIPredicateOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRestrictedIPredicateOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5607-5621
```cpp
static DecodeStatus
DecodeRestrictedSPredicateOperand(MCInst &Inst, unsigned Val, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  unsigned Code;
  switch (Val & 0x3) {
  case 0:
    Code = ARMCC::GE;
    break;
  case 1:
    Code = ARMCC::LT;
    break;
  case 2:
    Code = ARMCC::GT;
    break;
  case 3:
```
- EN: Implements `DecodeRestrictedSPredicateOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRestrictedSPredicateOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5622-5627
```cpp
    Code = ARMCC::LE;
    break;
  }
  Inst.addOperand(MCOperand::createImm(Code));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5629-5634
```cpp
static DecodeStatus
DecodeRestrictedUPredicateOperand(MCInst &Inst, unsigned Val, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  Inst.addOperand(MCOperand::createImm((Val & 0x1) == 0 ? ARMCC::HS : ARMCC::HI));
  return MCDisassembler::Success;
}
```
- EN: Implements `DecodeRestrictedUPredicateOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRestrictedUPredicateOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5636-5650
```cpp
static DecodeStatus
DecodeRestrictedFPPredicateOperand(MCInst &Inst, unsigned Val, uint64_t Address,
                                   const MCDisassembler *Decoder) {
  unsigned Code;
  switch (Val) {
  default:
    return MCDisassembler::Fail;
  case 0:
    Code = ARMCC::EQ;
    break;
  case 1:
    Code = ARMCC::NE;
    break;
  case 4:
    Code = ARMCC::GE;
```
- EN: Implements `DecodeRestrictedFPPredicateOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeRestrictedFPPredicateOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5651-5661
```cpp
    break;
  case 5:
    Code = ARMCC::LT;
    break;
  case 6:
    Code = ARMCC::GT;
    break;
  case 7:
    Code = ARMCC::LE;
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5663-5665
```cpp
  Inst.addOperand(MCOperand::createImm(Code));
  return MCDisassembler::Success;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5667-5670
```cpp
static DecodeStatus DecodeVCVTImmOperand(MCInst &Inst, unsigned Val,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeVCVTImmOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVCVTImmOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5672-5672
```cpp
  unsigned DecodedVal = 64 - Val;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5674-5688
```cpp
  switch (Inst.getOpcode()) {
  case ARM::MVE_VCVTf16s16_fix:
  case ARM::MVE_VCVTs16f16_fix:
  case ARM::MVE_VCVTf16u16_fix:
  case ARM::MVE_VCVTu16f16_fix:
    if (DecodedVal > 16)
      return MCDisassembler::Fail;
    break;
  case ARM::MVE_VCVTf32s32_fix:
  case ARM::MVE_VCVTs32f32_fix:
  case ARM::MVE_VCVTf32u32_fix:
  case ARM::MVE_VCVTu32f32_fix:
    if (DecodedVal > 32)
      return MCDisassembler::Fail;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5689-5689
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5691-5691
```cpp
  Inst.addOperand(MCOperand::createImm(64 - Val));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5693-5694
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5696-5710
```cpp
static unsigned FixedRegForVSTRVLDR_SYSREG(unsigned Opcode) {
  switch (Opcode) {
  case ARM::VSTR_P0_off:
  case ARM::VSTR_P0_pre:
  case ARM::VSTR_P0_post:
  case ARM::VLDR_P0_off:
  case ARM::VLDR_P0_pre:
  case ARM::VLDR_P0_post:
    return ARM::P0;
  case ARM::VSTR_FPSCR_NZCVQC_off:
  case ARM::VSTR_FPSCR_NZCVQC_pre:
  case ARM::VSTR_FPSCR_NZCVQC_post:
  case ARM::VLDR_FPSCR_NZCVQC_off:
  case ARM::VLDR_FPSCR_NZCVQC_pre:
  case ARM::VLDR_FPSCR_NZCVQC_post:
```
- EN: Implements `FixedRegForVSTRVLDR_SYSREG`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `FixedRegForVSTRVLDR_SYSREG`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5711-5715
```cpp
    return ARM::FPSCR;
  default:
    return 0;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5717-5731
```cpp
template <bool Writeback>
static DecodeStatus DecodeVSTRVLDR_SYSREG(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  switch (Inst.getOpcode()) {
  case ARM::VSTR_FPSCR_pre:
  case ARM::VSTR_FPSCR_NZCVQC_pre:
  case ARM::VLDR_FPSCR_pre:
  case ARM::VLDR_FPSCR_NZCVQC_pre:
  case ARM::VSTR_FPSCR_off:
  case ARM::VSTR_FPSCR_NZCVQC_off:
  case ARM::VLDR_FPSCR_off:
  case ARM::VLDR_FPSCR_NZCVQC_off:
  case ARM::VSTR_FPSCR_post:
  case ARM::VSTR_FPSCR_NZCVQC_post:
```
- EN: Implements `DecodeVSTRVLDR_SYSREG`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeVSTRVLDR_SYSREG`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5732-5735
```cpp
  case ARM::VLDR_FPSCR_post:
  case ARM::VLDR_FPSCR_NZCVQC_post:
    const FeatureBitset &featureBits =
        Decoder->getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5737-5739
```cpp
    if (!featureBits[ARM::HasMVEIntegerOps] && !featureBits[ARM::FeatureVFP2])
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5741-5746
```cpp
  DecodeStatus S = MCDisassembler::Success;
  if (unsigned Sysreg = FixedRegForVSTRVLDR_SYSREG(Inst.getOpcode()))
    Inst.addOperand(MCOperand::createReg(Sysreg));
  unsigned Rn = fieldFromInstruction(Val, 16, 4);
  unsigned addr = fieldFromInstruction(Val, 0, 7) |
                  (fieldFromInstruction(Val, 23, 1) << 7) | (Rn << 8);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5748-5753
```cpp
  if (Writeback) {
    if (!Check(S, DecodeGPRnopcRegisterClass(Inst, Rn, Address, Decoder)))
      return MCDisassembler::Fail;
  }
  if (!Check(S, DecodeT2AddrModeImm7s4(Inst, addr, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5755-5756
```cpp
  Inst.addOperand(MCOperand::createImm(ARMCC::AL));
  Inst.addOperand(MCOperand::createReg(0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5758-5759
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5761-5765
```cpp
static inline DecodeStatus
DecodeMVE_MEM_pre(MCInst &Inst, unsigned Val, uint64_t Address,
                  const MCDisassembler *Decoder, unsigned Rn,
                  OperandDecoder RnDecoder, OperandDecoder AddrDecoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeMVE_MEM_pre`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVE_MEM_pre`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5767-5769
```cpp
  unsigned Qd = fieldFromInstruction(Val, 13, 3);
  unsigned addr = fieldFromInstruction(Val, 0, 7) |
                  (fieldFromInstruction(Val, 23, 1) << 7) | (Rn << 8);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5771-5776
```cpp
  if (!Check(S, RnDecoder(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, AddrDecoder(Inst, addr, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5778-5780
```cpp
  Check(S, DecodeVpredNOperand(Inst, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5782-5790
```cpp
template <int shift>
static DecodeStatus DecodeMVE_MEM_1_pre(MCInst &Inst, unsigned Val,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  return DecodeMVE_MEM_pre(Inst, Val, Address, Decoder,
                           fieldFromInstruction(Val, 16, 3),
                           DecodetGPRRegisterClass,
                           DecodeTAddrModeImm7<shift>);
}
```
- EN: Implements `DecodeMVE_MEM_1_pre`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVE_MEM_1_pre`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5792-5800
```cpp
template <int shift>
static DecodeStatus DecodeMVE_MEM_2_pre(MCInst &Inst, unsigned Val,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  return DecodeMVE_MEM_pre(Inst, Val, Address, Decoder,
                           fieldFromInstruction(Val, 16, 4),
                           DecoderGPRRegisterClass,
                           DecodeT2AddrModeImm7<shift,1>);
}
```
- EN: Implements `DecodeMVE_MEM_2_pre`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVE_MEM_2_pre`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5802-5810
```cpp
template <int shift>
static DecodeStatus DecodeMVE_MEM_3_pre(MCInst &Inst, unsigned Val,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  return DecodeMVE_MEM_pre(Inst, Val, Address, Decoder,
                           fieldFromInstruction(Val, 17, 3),
                           DecodeMQPRRegisterClass,
                           DecodeMveAddrModeQ<shift>);
}
```
- EN: Implements `DecodeMVE_MEM_3_pre`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVE_MEM_3_pre`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5812-5816
```cpp
template <unsigned MinLog, unsigned MaxLog>
static DecodeStatus DecodePowerTwoOperand(MCInst &Inst, unsigned Val,
                                          uint64_t Address,
                                          const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodePowerTwoOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodePowerTwoOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5818-5819
```cpp
  if (Val < MinLog || Val > MaxLog)
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5821-5823
```cpp
  Inst.addOperand(MCOperand::createImm(1LL << Val));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5825-5829
```cpp
template <unsigned start>
static DecodeStatus
DecodeMVEPairVectorIndexOperand(MCInst &Inst, unsigned Val, uint64_t Address,
                                const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeMVEPairVectorIndexOperand`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEPairVectorIndexOperand`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5831-5831
```cpp
  Inst.addOperand(MCOperand::createImm(start + Val));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5833-5834
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5836-5844
```cpp
static DecodeStatus DecodeMVEVMOVQtoDReg(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Rt = fieldFromInstruction(Insn, 0, 4);
  unsigned Rt2 = fieldFromInstruction(Insn, 16, 4);
  unsigned Qd = ((fieldFromInstruction(Insn, 22, 1) << 3) |
                 fieldFromInstruction(Insn, 13, 3));
  unsigned index = fieldFromInstruction(Insn, 4, 1);
```
- EN: Implements `DecodeMVEVMOVQtoDReg`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEVMOVQtoDReg`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5846-5855
```cpp
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt2, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMVEPairVectorIndexOperand<2>(Inst, index, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMVEPairVectorIndexOperand<0>(Inst, index, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5857-5859
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5861-5869
```cpp
static DecodeStatus DecodeMVEVMOVDRegtoQ(MCInst &Inst, unsigned Insn,
                                         uint64_t Address,
                                         const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Rt = fieldFromInstruction(Insn, 0, 4);
  unsigned Rt2 = fieldFromInstruction(Insn, 16, 4);
  unsigned Qd = ((fieldFromInstruction(Insn, 22, 1) << 3) |
                 fieldFromInstruction(Insn, 13, 3));
  unsigned index = fieldFromInstruction(Insn, 4, 1);
```
- EN: Implements `DecodeMVEVMOVDRegtoQ`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEVMOVDRegtoQ`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5871-5882
```cpp
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rt2, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMVEPairVectorIndexOperand<2>(Inst, index, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMVEPairVectorIndexOperand<0>(Inst, index, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5884-5886
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5888-5891
```cpp
static DecodeStatus
DecodeMVEOverlappingLongShift(MCInst &Inst, unsigned Insn, uint64_t Address,
                              const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeMVEOverlappingLongShift`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEOverlappingLongShift`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5893-5895
```cpp
  unsigned RdaLo = fieldFromInstruction(Insn, 17, 3) << 1;
  unsigned RdaHi = fieldFromInstruction(Insn, 9, 3) << 1;
  unsigned Rm = fieldFromInstruction(Insn, 12, 4);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5897-5903
```cpp
  if (RdaHi == 14) {
    // This value of RdaHi (really indicating pc, because RdaHi has to
    // be an odd-numbered register, so the low bit will be set by the
    // decode function below) indicates that we must decode as SQRSHR
    // or UQRSHL, which both have a single Rda register field with all
    // four bits.
    unsigned Rda = fieldFromInstruction(Insn, 16, 4);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5905-5916
```cpp
    switch (Inst.getOpcode()) {
      case ARM::MVE_ASRLr:
      case ARM::MVE_SQRSHRL:
        Inst.setOpcode(ARM::MVE_SQRSHR);
        break;
      case ARM::MVE_LSLLr:
      case ARM::MVE_UQRSHLL:
        Inst.setOpcode(ARM::MVE_UQRSHL);
        break;
      default:
        llvm_unreachable("Unexpected starting opcode!");
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5918-5920
```cpp
    // Rda as output parameter
    if (!Check(S, DecoderGPRRegisterClass(Inst, Rda, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5922-5924
```cpp
    // Rda again as input parameter
    if (!Check(S, DecoderGPRRegisterClass(Inst, Rda, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5926-5928
```cpp
    // Rm, the amount to shift by
    if (!Check(S, DecoderGPRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5930-5930
```cpp
    DecodePredicateOperand(Inst, Decoder);
```
- EN: Declares `DecodePredicateOperand`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `DecodePredicateOperand`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5932-5933
```cpp
    if (fieldFromInstruction (Insn, 6, 3) != 4)
      return MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5935-5936
```cpp
    if (Rda == Rm)
      return MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5938-5939
```cpp
    return S;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5941-5942
```cpp
  // Otherwise, we decode as whichever opcode our caller has already
  // put into Inst. Those all look the same:
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 5944-5948
```cpp
  // RdaLo,RdaHi as output parameters
  if (!Check(S, DecodetGPREvenRegisterClass(Inst, RdaLo, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodetGPROddRegisterClass(Inst, RdaHi, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5950-5954
```cpp
  // RdaLo,RdaHi again as input parameters
  if (!Check(S, DecodetGPREvenRegisterClass(Inst, RdaLo, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodetGPROddRegisterClass(Inst, RdaHi, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5956-5958
```cpp
  // Rm, the amount to shift by
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rm, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5960-5965
```cpp
  if (Inst.getOpcode() == ARM::MVE_SQRSHRL ||
      Inst.getOpcode() == ARM::MVE_UQRSHLL) {
    unsigned Saturate = fieldFromInstruction(Insn, 7, 1);
    // Saturate, the bit position for saturation
    Inst.addOperand(MCOperand::createImm(Saturate));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5967-5969
```cpp
  DecodePredicateOperand(Inst, Decoder);
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 5971-5979
```cpp
static DecodeStatus DecodeMVEVCVTt1fp(MCInst &Inst, unsigned Insn,
                                      uint64_t Address,
                                      const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  unsigned Qd = ((fieldFromInstruction(Insn, 22, 1) << 3) |
                 fieldFromInstruction(Insn, 13, 3));
  unsigned Qm = ((fieldFromInstruction(Insn, 5, 1) << 3) |
                 fieldFromInstruction(Insn, 1, 3));
  unsigned imm6 = fieldFromInstruction(Insn, 16, 6);
```
- EN: Implements `DecodeMVEVCVTt1fp`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEVCVTt1fp`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 5981-5989
```cpp
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qd, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qm, Address, Decoder)))
    return MCDisassembler::Fail;
  if (!Check(S, DecodeVCVTImmOperand(Inst, imm6, Address, Decoder)))
    return MCDisassembler::Fail;
  Check(S, DecodeVpredROperand(Inst, Decoder));
  return S;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 5991-5998
```cpp
template <bool scalar, OperandDecoder predicate_decoder>
static DecodeStatus DecodeMVEVCMP(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  Inst.addOperand(MCOperand::createReg(ARM::VPR));
  unsigned Qn = fieldFromInstruction(Insn, 17, 3);
  if (!Check(S, DecodeMQPRRegisterClass(Inst, Qn, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeMVEVCMP`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEVCMP`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6000-6000
```cpp
  unsigned fc;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6002-6016
```cpp
  if (scalar) {
    fc = fieldFromInstruction(Insn, 12, 1) << 2 |
         fieldFromInstruction(Insn, 7, 1) |
         fieldFromInstruction(Insn, 5, 1) << 1;
    unsigned Rm = fieldFromInstruction(Insn, 0, 4);
    if (!Check(S, DecodeGPRwithZRRegisterClass(Inst, Rm, Address, Decoder)))
      return MCDisassembler::Fail;
  } else {
    fc = fieldFromInstruction(Insn, 12, 1) << 2 |
         fieldFromInstruction(Insn, 7, 1) |
         fieldFromInstruction(Insn, 0, 1) << 1;
    unsigned Qm = fieldFromInstruction(Insn, 5, 1) << 4 |
                  fieldFromInstruction(Insn, 1, 3);
    if (!Check(S, DecodeMQPRRegisterClass(Inst, Qm, Address, Decoder)))
      return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6017-6017
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6019-6020
```cpp
  if (!Check(S, predicate_decoder(Inst, fc, Address, Decoder)))
    return MCDisassembler::Fail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6022-6024
```cpp
  Check(S, DecodeVpredNOperand(Inst, Decoder));
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6026-6035
```cpp
static DecodeStatus DecodeMveVCTP(MCInst &Inst, unsigned Insn, uint64_t Address,
                                  const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  Inst.addOperand(MCOperand::createReg(ARM::VPR));
  unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  if (!Check(S, DecoderGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  Check(S, DecodeVpredNOperand(Inst, Decoder));
  return S;
}
```
- EN: Implements `DecodeMveVCTP`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMveVCTP`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6037-6045
```cpp
static DecodeStatus DecodeMVEVPNOT(MCInst &Inst, unsigned Insn,
                                   uint64_t Address,
                                   const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
  Inst.addOperand(MCOperand::createReg(ARM::VPR));
  Inst.addOperand(MCOperand::createReg(ARM::VPR));
  Check(S, DecodeVpredNOperand(Inst, Decoder));
  return S;
}
```
- EN: Implements `DecodeMVEVPNOT`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeMVEVPNOT`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6047-6060
```cpp
static DecodeStatus DecodeT2AddSubSPImm(MCInst &Inst, unsigned Insn,
                                        uint64_t Address,
                                        const MCDisassembler *Decoder) {
  const unsigned Rd = fieldFromInstruction(Insn, 8, 4);
  const unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  const unsigned Imm12 = fieldFromInstruction(Insn, 26, 1) << 11 |
                         fieldFromInstruction(Insn, 12, 3) << 8 |
                         fieldFromInstruction(Insn, 0, 8);
  const unsigned TypeT3 = fieldFromInstruction(Insn, 25, 1);
  unsigned sign1 = fieldFromInstruction(Insn, 21, 1);
  unsigned sign2 = fieldFromInstruction(Insn, 23, 1);
  unsigned S = fieldFromInstruction(Insn, 20, 1);
  if (sign1 != sign2)
    return MCDisassembler::Fail;
```
- EN: Implements `DecodeT2AddSubSPImm`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeT2AddSubSPImm`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6062-6076
```cpp
  // T3 does a zext of imm12, where T2 does a ThumbExpandImm (T2SOImm)
  DecodeStatus DS = MCDisassembler::Success;
  if ((!Check(DS,
              DecodeGPRspRegisterClass(Inst, Rd, Address, Decoder))) || // dst
      (!Check(DS, DecodeGPRspRegisterClass(Inst, Rn, Address, Decoder))))
    return MCDisassembler::Fail;
  if (TypeT3) {
    Inst.setOpcode(sign1 ? ARM::t2SUBspImm12 : ARM::t2ADDspImm12);
    Inst.addOperand(MCOperand::createImm(Imm12)); // zext imm12
    DecodePredicateOperand(Inst, Decoder);
  } else {
    Inst.setOpcode(sign1 ? ARM::t2SUBspImm : ARM::t2ADDspImm);
    if (!Check(DS, DecodeT2SOImm(Inst, Imm12, Address, Decoder))) // imm12
      return MCDisassembler::Fail;
    DecodePredicateOperand(Inst, Decoder);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6077-6079
```cpp
    if (!Check(DS, DecodeCCOutOperand(Inst, S, Address, Decoder))) // cc_out
      return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6081-6082
```cpp
  return DS;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6084-6087
```cpp
static DecodeStatus DecodeLazyLoadStoreMul(MCInst &Inst, unsigned Insn,
                                           uint64_t Address,
                                           const MCDisassembler *Decoder) {
  DecodeStatus S = MCDisassembler::Success;
```
- EN: Implements `DecodeLazyLoadStoreMul`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `DecodeLazyLoadStoreMul`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6089-6099
```cpp
  const unsigned Rn = fieldFromInstruction(Insn, 16, 4);
  // Adding Rn, holding memory location to save/load to/from, the only argument
  // that is being encoded.
  // '$Rn' in the assembly.
  if (!Check(S, DecodeGPRRegisterClass(Inst, Rn, Address, Decoder)))
    return MCDisassembler::Fail;
  // An optional predicate, '$p' in the assembly.
  DecodePredicateOperand(Inst, Decoder);
  // An immediate that represents a floating point registers list. '$regs' in
  // the assembly.
  Inst.addOperand(MCOperand::createImm(0)); // Arbitrary value, has no effect.
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6101-6102
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6104-6104
```cpp
#include "ARMGenDisassemblerTables.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 6106-6120
```cpp
// Post-decoding checks
static DecodeStatus checkDecodedInstruction(MCInst &MI, uint64_t &Size,
                                            uint64_t Address, raw_ostream &CS,
                                            uint32_t Insn,
                                            DecodeStatus Result) {
  switch (MI.getOpcode()) {
    case ARM::HVC: {
      // HVC is undefined if condition = 0xf otherwise unpredictable
      // if condition != 0xe
      uint32_t Cond = (Insn >> 28) & 0xF;
      if (Cond == 0xF)
        return MCDisassembler::Fail;
      if (Cond != 0xE)
        return MCDisassembler::SoftFail;
      return Result;
```
- EN: Implements `checkDecodedInstruction`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `checkDecodedInstruction`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6121-6135
```cpp
    }
    case ARM::t2ADDri:
    case ARM::t2ADDri12:
    case ARM::t2ADDrr:
    case ARM::t2ADDrs:
    case ARM::t2SUBri:
    case ARM::t2SUBri12:
    case ARM::t2SUBrr:
    case ARM::t2SUBrs:
      if (MI.getOperand(0).getReg() == ARM::SP &&
          MI.getOperand(1).getReg() != ARM::SP)
        return MCDisassembler::SoftFail;
      return Result;
    default: return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6136-6136
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6138-6144
```cpp
uint64_t ARMDisassembler::suggestBytesToSkip(ArrayRef<uint8_t> Bytes,
                                             uint64_t Address) const {
  // In Arm state, instructions are always 4 bytes wide, so there's no
  // point in skipping any smaller number of bytes if an instruction
  // can't be decoded.
  if (!STI.hasFeature(ARM::ModeThumb))
    return 4;
```
- EN: Implements `ARMDisassembler::suggestBytesToSkip`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::suggestBytesToSkip`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6146-6159
```cpp
  // In a Thumb instruction stream, a halfword is a standalone 2-byte
  // instruction if and only if its value is less than 0xE800.
  // Otherwise, it's the first halfword of a 4-byte instruction.
  //
  // So, if we can see the upcoming halfword, we can judge on that
  // basis, and maybe skip a whole 4-byte instruction that we don't
  // know how to decode, without accidentally trying to interpret its
  // second half as something else.
  //
  // If we don't have the instruction data available, we just have to
  // recommend skipping the minimum sensible distance, which is 2
  // bytes.
  if (Bytes.size() < 2)
    return 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6161-6164
```cpp
  uint16_t Insn16 = llvm::support::endian::read<uint16_t>(
      Bytes.data(), InstructionEndianness);
  return Insn16 < 0xE800 ? 2 : 4;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6166-6176
```cpp
DecodeStatus ARMDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
                                             ArrayRef<uint8_t> Bytes,
                                             uint64_t Address,
                                             raw_ostream &CS) const {
  DecodeStatus S;
  if (STI.hasFeature(ARM::ModeThumb))
    S = getThumbInstruction(MI, Size, Bytes, Address, CS);
  else
    S = getARMInstruction(MI, Size, Bytes, Address, CS);
  if (S == DecodeStatus::Fail)
    return S;
```
- EN: Implements `ARMDisassembler::getInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::getInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6178-6184
```cpp
  // Verify that the decoded instruction has the correct number of operands.
  const MCInstrDesc &MCID = MCII->get(MI.getOpcode());
  if (!MCID.isVariadic() && MI.getNumOperands() != MCID.getNumOperands()) {
    reportFatalInternalError(MCII->getName(MI.getOpcode()) + ": expected " +
                             Twine(MCID.getNumOperands()) + " operands, got " +
                             Twine(MI.getNumOperands()) + "\n");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6186-6187
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6189-6193
```cpp
DecodeStatus ARMDisassembler::getARMInstruction(MCInst &MI, uint64_t &Size,
                                                ArrayRef<uint8_t> Bytes,
                                                uint64_t Address,
                                                raw_ostream &CS) const {
  CommentStream = &CS;
```
- EN: Implements `ARMDisassembler::getARMInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::getARMInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6195-6197
```cpp
  assert(!STI.hasFeature(ARM::ModeThumb) &&
         "Asked to disassemble an ARM instruction but Subtarget is in Thumb "
         "mode!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6199-6203
```cpp
  // We want to read exactly 4 bytes of data.
  if (Bytes.size() < 4) {
    Size = 0;
    return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6205-6207
```cpp
  // Encoded as a 32-bit word in the stream.
  uint32_t Insn = llvm::support::endian::read<uint32_t>(Bytes.data(),
                                                        InstructionEndianness);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6209-6215
```cpp
  // Calling the auto-generated decoder function.
  DecodeStatus Result =
      decodeInstruction(DecoderTableARM32, MI, Insn, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
    return checkDecodedInstruction(MI, Size, Address, CS, Insn, Result);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6217-6222
```cpp
  const uint8_t *Tables[] = {
      DecoderTableVFP32,      DecoderTableVFPV832,
      DecoderTableNEONData32, DecoderTableNEONLoadStore32,
      DecoderTableNEONDup32,  DecoderTablev8NEON32,
      DecoderTablev8Crypto32,
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6224-6230
```cpp
  for (const uint8_t *Table : Tables) {
    Result = decodeInstruction(Table, MI, Insn, Address, this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      return Result;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6232-6237
```cpp
  Result =
      decodeInstruction(DecoderTableCoProc32, MI, Insn, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
    return checkDecodedInstruction(MI, Size, Address, CS, Insn, Result);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6239-6241
```cpp
  Size = 4;
  return MCDisassembler::Fail;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6243-6250
```cpp
bool ARMDisassembler::isVectorPredicable(const MCInst &MI) const {
  const MCInstrDesc &MCID = MCII->get(MI.getOpcode());
  for (unsigned i = 0; i < MCID.NumOperands; ++i) {
    if (ARM::isVpred(MCID.operands()[i].OperandType))
      return true;
  }
  return false;
}
```
- EN: Implements `ARMDisassembler::isVectorPredicable`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::isVectorPredicable`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6252-6258
```cpp
// Most Thumb instructions don't have explicit predicates in the encoding,
// but rather get their predicates from IT context. Here, we check that the
// decoded instruction is allowed to have the decoded predicate and advance
// IT/VPT block states.
MCDisassembler::DecodeStatus
ARMDisassembler::checkThumbPredicate(MCInst &MI) const {
  MCDisassembler::DecodeStatus S = Success;
```
- EN: Implements `ARMDisassembler::checkThumbPredicate`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::checkThumbPredicate`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6260-6260
```cpp
  const FeatureBitset &FeatureBits = getSubtargetInfo().getFeatureBits();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6262-6276
```cpp
  switch (MI.getOpcode()) {
    case ARM::tBcc:
    case ARM::t2Bcc:
    case ARM::tCBZ:
    case ARM::tCBNZ:
    case ARM::tCPS:
    case ARM::t2CPS3p:
    case ARM::t2CPS2p:
    case ARM::t2CPS1p:
    case ARM::t2CSEL:
    case ARM::t2CSINC:
    case ARM::t2CSINV:
    case ARM::t2CSNEG:
    case ARM::tMOVSr:
    case ARM::tSETEND:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6277-6291
```cpp
      // Some instructions (mostly conditional branches) are not
      // allowed in IT blocks.
      if (ITBlock.instrInITBlock())
        S = SoftFail;
      else
        return Success;
      break;
    case ARM::t2HINT:
      if (MI.getOperand(0).getImm() == 0x10 && (FeatureBits[ARM::FeatureRAS]) != 0)
        S = SoftFail;
      break;
    case ARM::tB:
    case ARM::t2B:
    case ARM::t2TBB:
    case ARM::t2TBH:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6292-6299
```cpp
      // Some instructions (mostly unconditional branches) can
      // only appears at the end of, or outside of, an IT.
      if (ITBlock.instrInITBlock() && !ITBlock.instrLastInITBlock())
        S = SoftFail;
      break;
    default:
      break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6301-6305
```cpp
  // Warn on non-VPT predicable instruction in a VPT block and a VPT
  // predicable instruction in an IT block
  if ((!isVectorPredicable(MI) && VPTBlock.instrInVPTBlock()) ||
      (isVectorPredicable(MI) && ITBlock.instrInITBlock()))
    S = SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6307-6310
```cpp
  if (ITBlock.instrInITBlock())
    ITBlock.advanceITState();
  else if (VPTBlock.instrInVPTBlock())
    VPTBlock.advanceVPTState();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6312-6313
```cpp
  return S;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6315-6329
```cpp
// Thumb VFP and some NEON instructions are a special case. Because we share
// their encodings between ARM and Thumb modes, and they are predicable in ARM
// mode, the auto-generated decoder will give them an (incorrect)
// predicate operand.  We need to rewrite these operands based on the IT
// context as a post-pass.
void ARMDisassembler::UpdateThumbPredicate(DecodeStatus &S, MCInst &MI) const {
  unsigned CC;
  CC = ITBlock.getITCC();
  if (CC == 0xF)
    CC = ARMCC::AL;
  if (ITBlock.instrInITBlock())
    ITBlock.advanceITState();
  else if (VPTBlock.instrInVPTBlock()) {
    CC = VPTBlock.getVPTPred();
    VPTBlock.advanceVPTState();
```
- EN: Implements `ARMDisassembler::UpdateThumbPredicate`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::UpdateThumbPredicate`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6330-6330
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6332-6346
```cpp
  const MCInstrDesc &MCID = MCII->get(MI.getOpcode());
  ArrayRef<MCOperandInfo> OpInfo = MCID.operands();
  MCInst::iterator I = MI.begin();
  unsigned short NumOps = MCID.NumOperands;
  for (unsigned i = 0; i < NumOps; ++i, ++I) {
    if (OpInfo[i].isPredicate() ) {
      if (CC != ARMCC::AL && !MCID.isPredicable())
        Check(S, SoftFail);
      I->setImm(CC);
      ++I;
      if (CC == ARMCC::AL)
        I->setReg(ARM::NoRegister);
      else
        I->setReg(ARM::CPSR);
      return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6347-6349
```cpp
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6351-6355
```cpp
DecodeStatus ARMDisassembler::getThumbInstruction(MCInst &MI, uint64_t &Size,
                                                  ArrayRef<uint8_t> Bytes,
                                                  uint64_t Address,
                                                  raw_ostream &CS) const {
  CommentStream = &CS;
```
- EN: Implements `ARMDisassembler::getThumbInstruction`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMDisassembler::getThumbInstruction`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6357-6358
```cpp
  assert(STI.hasFeature(ARM::ModeThumb) &&
         "Asked to disassemble in Thumb mode but Subtarget is in ARM mode!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6360-6364
```cpp
  // We want to read exactly 2 bytes of data.
  if (Bytes.size() < 2) {
    Size = 0;
    return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6366-6374
```cpp
  uint16_t Insn16 = llvm::support::endian::read<uint16_t>(
      Bytes.data(), InstructionEndianness);
  DecodeStatus Result =
      decodeInstruction(DecoderTableThumb16, MI, Insn16, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 2;
    Check(Result, checkThumbPredicate(MI));
    return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6376-6382
```cpp
  Result = decodeInstruction(DecoderTableThumbSBit16, MI, Insn16, Address, this,
                             STI);
  if (Result) {
    Size = 2;
    Check(Result, checkThumbPredicate(MI));
    return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6384-6387
```cpp
  Result =
      decodeInstruction(DecoderTableThumb216, MI, Insn16, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 2;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6389-6392
```cpp
    // Nested IT blocks are UNPREDICTABLE.  Must be checked before we add
    // the Thumb predicate.
    if (MI.getOpcode() == ARM::t2IT && ITBlock.instrInITBlock())
      Result = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6394-6394
```cpp
    Check(Result, checkThumbPredicate(MI));
```
- EN: Declares `Check`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Check`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6396-6402
```cpp
    // If we find an IT instruction, we need to parse its condition
    // code and mask operands so that we can apply them correctly
    // to the subsequent instructions.
    if (MI.getOpcode() == ARM::t2IT) {
      unsigned Firstcond = MI.getOperand(0).getImm();
      unsigned Mask = MI.getOperand(1).getImm();
      ITBlock.setITState(Firstcond, Mask);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6404-6407
```cpp
      // An IT instruction that would give a 'NV' predicate is unpredictable.
      if (Firstcond == ARMCC::AL && !isPowerOf2_32(Mask))
        CS << "unpredictable IT predicate sequence";
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6409-6410
```cpp
    return Result;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6412-6416
```cpp
  // We want to read exactly 4 bytes of data.
  if (Bytes.size() < 4) {
    Size = 0;
    return MCDisassembler::Fail;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6418-6420
```cpp
  uint32_t Insn32 =
      (uint32_t(Insn16) << 16) | llvm::support::endian::read<uint16_t>(
                                     Bytes.data() + 2, InstructionEndianness);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6422-6425
```cpp
  Result =
      decodeInstruction(DecoderTableMVE32, MI, Insn32, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6427-6430
```cpp
    // Nested VPT blocks are UNPREDICTABLE. Must be checked before we add
    // the VPT predicate.
    if (isVPTOpcode(MI.getOpcode()) && VPTBlock.instrInVPTBlock())
      Result = MCDisassembler::SoftFail;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6432-6432
```cpp
    Check(Result, checkThumbPredicate(MI));
```
- EN: Declares `Check`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Check`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6434-6437
```cpp
    if (isVPTOpcode(MI.getOpcode())) {
      unsigned Mask = MI.getOperand(0).getImm();
      VPTBlock.setVPTState(Mask);
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6439-6440
```cpp
    return Result;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 6442-6448
```cpp
  Result =
      decodeInstruction(DecoderTableThumb32, MI, Insn32, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
    Check(Result, checkThumbPredicate(MI));
    return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6450-6456
```cpp
  Result =
      decodeInstruction(DecoderTableThumb232, MI, Insn32, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
    Check(Result, checkThumbPredicate(MI));
    return checkDecodedInstruction(MI, Size, Address, CS, Insn32, Result);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6458-6466
```cpp
  if (fieldFromInstruction(Insn32, 28, 4) == 0xE) {
    Result =
        decodeInstruction(DecoderTableVFP32, MI, Insn32, Address, this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      UpdateThumbPredicate(Result, MI);
      return Result;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6468-6473
```cpp
  Result =
      decodeInstruction(DecoderTableVFPV832, MI, Insn32, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
    return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6475-6483
```cpp
  if (fieldFromInstruction(Insn32, 28, 4) == 0xE) {
    Result = decodeInstruction(DecoderTableNEONDup32, MI, Insn32, Address, this,
                               STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      UpdateThumbPredicate(Result, MI);
      return Result;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6485-6496
```cpp
  if (fieldFromInstruction(Insn32, 24, 8) == 0xF9) {
    uint32_t NEONLdStInsn = Insn32;
    NEONLdStInsn &= 0xF0FFFFFF;
    NEONLdStInsn |= 0x04000000;
    Result = decodeInstruction(DecoderTableNEONLoadStore32, MI, NEONLdStInsn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      Check(Result, checkThumbPredicate(MI));
      return Result;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6498-6509
```cpp
  if (fieldFromInstruction(Insn32, 24, 4) == 0xF) {
    uint32_t NEONDataInsn = Insn32;
    NEONDataInsn &= 0xF0FFFFFF; // Clear bits 27-24
    NEONDataInsn |= (NEONDataInsn & 0x10000000) >> 4; // Move bit 28 to bit 24
    NEONDataInsn |= 0x12000000; // Set bits 28 and 25
    Result = decodeInstruction(DecoderTableNEONData32, MI, NEONDataInsn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      Check(Result, checkThumbPredicate(MI));
      return Result;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6511-6520
```cpp
    uint32_t NEONCryptoInsn = Insn32;
    NEONCryptoInsn &= 0xF0FFFFFF; // Clear bits 27-24
    NEONCryptoInsn |= (NEONCryptoInsn & 0x10000000) >> 4; // Move bit 28 to bit 24
    NEONCryptoInsn |= 0x12000000; // Set bits 28 and 25
    Result = decodeInstruction(DecoderTablev8Crypto32, MI, NEONCryptoInsn,
                               Address, this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      return Result;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6522-6530
```cpp
    uint32_t NEONv8Insn = Insn32;
    NEONv8Insn &= 0xF3FFFFFF; // Clear bits 27-26
    Result = decodeInstruction(DecoderTablev8NEON32, MI, NEONv8Insn, Address,
                               this, STI);
    if (Result != MCDisassembler::Fail) {
      Size = 4;
      return Result;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6532-6542
```cpp
  uint32_t Coproc = fieldFromInstruction(Insn32, 8, 4);
  const uint8_t *DecoderTable = ARM::isCDECoproc(Coproc, STI)
                                    ? DecoderTableThumb2CDE32
                                    : DecoderTableThumb2CoProc32;
  Result =
      decodeInstruction(DecoderTable, MI, Insn32, Address, this, STI);
  if (Result != MCDisassembler::Fail) {
    Size = 4;
    Check(Result, checkThumbPredicate(MI));
    return Result;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6544-6550
```cpp
  // Advance IT state to prevent next instruction inheriting
  // the wrong IT state.
  if (ITBlock.instrInITBlock())
    ITBlock.advanceITState();
  Size = 0;
  return MCDisassembler::Fail;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 6552-6556
```cpp
static MCDisassembler *createARMDisassembler(const Target &T,
                                             const MCSubtargetInfo &STI,
                                             MCContext &Ctx) {
  return new ARMDisassembler(STI, Ctx, T.createMCInstrInfo());
}
```
- EN: Implements `createARMDisassembler`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createARMDisassembler`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 6558-6568
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeARMDisassembler() {
  TargetRegistry::RegisterMCDisassembler(getTheARMLETarget(),
                                         createARMDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheARMBETarget(),
                                         createARMDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheThumbLETarget(),
                                         createARMDisassembler);
  TargetRegistry::RegisterMCDisassembler(getTheThumbBETarget(),
                                         createARMDisassembler);
}
```
- EN: Implements `TargetRegistry::RegisterMCDisassembler`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `TargetRegistry::RegisterMCDisassembler`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: instruction decoding and disassembly.
  - CN: 核心职责：指令解码与反汇编。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMBaseInstrInfo.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `MCTargetDesc/ARMMCTargetDesc.h`, `TargetInfo/ARMTargetInfo.h`, `Utils/ARMBaseInfo.h`, `ARMGenDisassemblerTables.inc`.
  - CN: 后端本地头文件：`ARMBaseInstrInfo.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `MCTargetDesc/ARMMCTargetDesc.h`, `TargetInfo/ARMTargetInfo.h`, `Utils/ARMBaseInfo.h`, `ARMGenDisassemblerTables.inc`。
- EN: LLVM infrastructure headers: `llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrDesc.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCSubtargetInfo.h` ... (+6 more).
  - CN: LLVM 基础设施头文件：`llvm/MC/MCContext.h`, `llvm/MC/MCDecoder.h`, `llvm/MC/MCDecoderOps.h`, `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrDesc.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCSubtargetInfo.h` ... (+6 more)。
- EN: Standard/system headers: `algorithm`, `cassert`, `cstdint`, `vector`.
  - CN: 标准库/系统头文件：`algorithm`, `cassert`, `cstdint`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
