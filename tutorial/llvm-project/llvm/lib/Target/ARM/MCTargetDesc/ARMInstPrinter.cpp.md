# ARMInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/MCTargetDesc/ARMInstPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMInstPrinter` for the ARM backend, focusing on instruction pretty-printing.
- 用途 (CN): 实现 ARM 后端中的 `ARMInstPrinter`，重点处理指令格式化打印。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMInstPrinter.cpp - Convert ARM MCInst to assembly syntax --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class prints an ARM MCInst to a .s file.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "ARMInstPrinter.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "Utils/ARMBaseInfo.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrAnalysis.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/SubtargetFeature.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-29
```cpp
#include <cassert>
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 31-31
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 33-33
```cpp
#define DEBUG_TYPE "asm-printer"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 35-36
```cpp
#define PRINT_ALIAS_INSTR
#include "ARMGenAsmWriter.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 38-43
```cpp
/// translateShiftImm - Convert shift immediate from 0-31 to 1-32 for printing.
///
/// getSORegOffset returns an integer from 0-31, representing '32' as 0.
static unsigned translateShiftImm(unsigned imm) {
  // lsr #32 and asr #32 exist, but should be encoded as a 0.
  assert((imm & ~0x1f) == 0 && "Invalid shift encoding");
```
- EN: Implements `translateShiftImm`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `translateShiftImm`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 45-48
```cpp
  if (imm == 0)
    return 32;
  return imm;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 50-54
```cpp
static void printRegImmShift(raw_ostream &O, ARM_AM::ShiftOpc ShOpc,
                             unsigned ShImm, ARMInstPrinter &printer) {
  if (ShOpc == ARM_AM::no_shift || (ShOpc == ARM_AM::lsl && !ShImm))
    return;
  O << ", ";
```
- EN: Implements `printRegImmShift`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `printRegImmShift`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 56-57
```cpp
  assert(!(ShOpc == ARM_AM::ror && !ShImm) && "Cannot have ror #0");
  O << getShiftOpcStr(ShOpc);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 59-64
```cpp
  if (ShOpc != ARM_AM::rrx) {
    O << " ";
    printer.markup(O, llvm::MCInstPrinter::Markup::Immediate)
        << "#" << translateShiftImm(ShImm);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 66-68
```cpp
ARMInstPrinter::ARMInstPrinter(const MCAsmInfo &MAI, const MCInstrInfo &MII,
                               const MCRegisterInfo &MRI)
    : MCInstPrinter(MAI, MII, MRI) {}
```
- EN: Implements `ARMInstPrinter::ARMInstPrinter`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::ARMInstPrinter`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 70-80
```cpp
bool ARMInstPrinter::applyTargetSpecificCLOption(StringRef Opt) {
  if (Opt == "reg-names-std") {
    DefaultAltIdx = ARM::NoRegAltName;
    return true;
  }
  if (Opt == "reg-names-raw") {
    DefaultAltIdx = ARM::RegNamesRaw;
    return true;
  }
  return false;
}
```
- EN: Implements `ARMInstPrinter::applyTargetSpecificCLOption`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::applyTargetSpecificCLOption`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-84
```cpp
void ARMInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  markup(OS, Markup::Register) << getRegisterName(Reg, DefaultAltIdx);
}
```
- EN: Implements `ARMInstPrinter::printRegName`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printRegName`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 86-89
```cpp
void ARMInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                               StringRef Annot, const MCSubtargetInfo &STI,
                               raw_ostream &O) {
  unsigned Opcode = MI->getOpcode();
```
- EN: Implements `ARMInstPrinter::printInst`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printInst`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 91-105
```cpp
  switch (Opcode) {
  case ARM::VLLDM: {
    const MCOperand &Reg = MI->getOperand(0);
    O << '\t' << "vlldm" << '\t';
    printRegName(O, Reg.getReg());
    O << ", "
      << "{d0 - d15}";
    return;
  }
  case ARM::VLLDM_T2: {
    const MCOperand &Reg = MI->getOperand(0);
    O << '\t' << "vlldm" << '\t';
    printRegName(O, Reg.getReg());
    O << ", "
      << "{d0 - d31}";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 106-120
```cpp
    return;
  }
  case ARM::VLSTM: {
    const MCOperand &Reg = MI->getOperand(0);
    O << '\t' << "vlstm" << '\t';
    printRegName(O, Reg.getReg());
    O << ", "
      << "{d0 - d15}";
    return;
  }
  case ARM::VLSTM_T2: {
    const MCOperand &Reg = MI->getOperand(0);
    O << '\t' << "vlstm" << '\t';
    printRegName(O, Reg.getReg());
    O << ", "
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 121-130
```cpp
      << "{d0 - d31}";
    return;
  }
  // Check for MOVs and print canonical forms, instead.
  case ARM::MOVsr: {
    // FIXME: Thumb variants?
    const MCOperand &Dst = MI->getOperand(0);
    const MCOperand &MO1 = MI->getOperand(1);
    const MCOperand &MO2 = MI->getOperand(2);
    const MCOperand &MO3 = MI->getOperand(3);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-134
```cpp
    O << '\t' << ARM_AM::getShiftOpcStr(ARM_AM::getSORegShOp(MO3.getImm()));
    printSBitModifierOperand(MI, 6, STI, O);
    printPredicateOperand(MI, 4, STI, O);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 136-139
```cpp
    O << '\t';
    printRegName(O, Dst.getReg());
    O << ", ";
    printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-146
```cpp
    O << ", ";
    printRegName(O, MO2.getReg());
    assert(ARM_AM::getSORegOffset(MO3.getImm()) == 0);
    printAnnotation(O, Annot);
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 148-152
```cpp
  case ARM::MOVsi: {
    // FIXME: Thumb variants?
    const MCOperand &Dst = MI->getOperand(0);
    const MCOperand &MO1 = MI->getOperand(1);
    const MCOperand &MO2 = MI->getOperand(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 154-156
```cpp
    O << '\t' << ARM_AM::getShiftOpcStr(ARM_AM::getSORegShOp(MO2.getImm()));
    printSBitModifierOperand(MI, 5, STI, O);
    printPredicateOperand(MI, 3, STI, O);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-161
```cpp
    O << '\t';
    printRegName(O, Dst.getReg());
    O << ", ";
    printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 163-166
```cpp
    if (ARM_AM::getSORegShOp(MO2.getImm()) == ARM_AM::rrx) {
      printAnnotation(O, Annot);
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 168-173
```cpp
    O << ", ";
    markup(O, Markup::Immediate)
        << "#" << translateShiftImm(ARM_AM::getSORegOffset(MO2.getImm()));
    printAnnotation(O, Annot);
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 175-189
```cpp
  // A8.6.123 PUSH
  case ARM::STMDB_UPD:
  case ARM::t2STMDB_UPD:
    if (MI->getOperand(0).getReg() == ARM::SP && MI->getNumOperands() > 5) {
      // Should only print PUSH if there are at least two registers in the list.
      O << '\t' << "push";
      printPredicateOperand(MI, 2, STI, O);
      if (Opcode == ARM::t2STMDB_UPD)
        O << ".w";
      O << '\t';
      printRegisterList(MI, 4, STI, O);
      printAnnotation(O, Annot);
      return;
    } else
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 191-202
```cpp
  case ARM::STR_PRE_IMM:
    if (MI->getOperand(2).getReg() == ARM::SP &&
        MI->getOperand(3).getImm() == -4) {
      O << '\t' << "push";
      printPredicateOperand(MI, 4, STI, O);
      O << "\t{";
      printRegName(O, MI->getOperand(1).getReg());
      O << "}";
      printAnnotation(O, Annot);
      return;
    } else
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 204-218
```cpp
  // A8.6.122 POP
  case ARM::LDMIA_UPD:
  case ARM::t2LDMIA_UPD:
    if (MI->getOperand(0).getReg() == ARM::SP && MI->getNumOperands() > 5) {
      // Should only print POP if there are at least two registers in the list.
      O << '\t' << "pop";
      printPredicateOperand(MI, 2, STI, O);
      if (Opcode == ARM::t2LDMIA_UPD)
        O << ".w";
      O << '\t';
      printRegisterList(MI, 4, STI, O);
      printAnnotation(O, Annot);
      return;
    } else
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 220-231
```cpp
  case ARM::LDR_POST_IMM:
    if (MI->getOperand(2).getReg() == ARM::SP &&
        MI->getOperand(4).getImm() == 4) {
      O << '\t' << "pop";
      printPredicateOperand(MI, 5, STI, O);
      O << "\t{";
      printRegName(O, MI->getOperand(0).getReg());
      O << "}";
      printAnnotation(O, Annot);
      return;
    } else
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 233-244
```cpp
  // A8.6.355 VPUSH
  case ARM::VSTMSDB_UPD:
  case ARM::VSTMDDB_UPD:
    if (MI->getOperand(0).getReg() == ARM::SP) {
      O << '\t' << "vpush";
      printPredicateOperand(MI, 2, STI, O);
      O << '\t';
      printRegisterList(MI, 4, STI, O);
      printAnnotation(O, Annot);
      return;
    } else
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 246-257
```cpp
  // A8.6.354 VPOP
  case ARM::VLDMSIA_UPD:
  case ARM::VLDMDIA_UPD:
    if (MI->getOperand(0).getReg() == ARM::SP) {
      O << '\t' << "vpop";
      printPredicateOperand(MI, 2, STI, O);
      O << '\t';
      printRegisterList(MI, 4, STI, O);
      printAnnotation(O, Annot);
      return;
    } else
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 259-265
```cpp
  case ARM::tLDMIA: {
    bool Writeback = true;
    MCRegister BaseReg = MI->getOperand(0).getReg();
    for (unsigned i = 3; i < MI->getNumOperands(); ++i) {
      if (MI->getOperand(i).getReg() == BaseReg)
        Writeback = false;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 267-267
```cpp
    O << "\tldm";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 269-278
```cpp
    printPredicateOperand(MI, 1, STI, O);
    O << '\t';
    printRegName(O, BaseReg);
    if (Writeback)
      O << "!";
    O << ", ";
    printRegisterList(MI, 3, STI, O);
    printAnnotation(O, Annot);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 280-294
```cpp
  // Combine 2 GPRs from disassembler into a GPRPair to match with instr def.
  // ldrexd/strexd require even/odd GPR pair. To enforce this constraint,
  // a single GPRPair reg operand is used in the .td file to replace the two
  // GPRs. However, when decoding them, the two GRPs cannot be automatically
  // expressed as a GPRPair, so we have to manually merge them.
  // FIXME: We would really like to be able to tablegen'erate this.
  case ARM::LDREXD:
  case ARM::STREXD:
  case ARM::LDAEXD:
  case ARM::STLEXD: {
    const MCRegisterClass &MRC = MRI.getRegClass(ARM::GPRRegClassID);
    bool isStore = Opcode == ARM::STREXD || Opcode == ARM::STLEXD;
    MCRegister Reg = MI->getOperand(isStore ? 1 : 0).getReg();
    if (MRC.contains(Reg)) {
      MCInst NewMI;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 295-296
```cpp
      MCOperand NewReg;
      NewMI.setOpcode(Opcode);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 298-302
```cpp
      if (isStore)
        NewMI.addOperand(MI->getOperand(0));
      NewReg = MCOperand::createReg(MRI.getMatchingSuperReg(
          Reg, ARM::gsub_0, &MRI.getRegClass(ARM::GPRPairRegClassID)));
      NewMI.addOperand(NewReg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 304-318
```cpp
      // Copy the rest operands into NewMI.
      for (unsigned i = isStore ? 3 : 2; i < MI->getNumOperands(); ++i)
        NewMI.addOperand(MI->getOperand(i));
      printInstruction(&NewMI, Address, STI, O);
      return;
    }
    break;
  }
  case ARM::TSB:
  case ARM::t2TSB:
    O << "\ttsb\tcsync";
    return;
  case ARM::t2DSB:
    switch (MI->getOperand(0).getImm()) {
    default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 319-331
```cpp
      if (!printAliasInstr(MI, Address, STI, O))
        printInstruction(MI, Address, STI, O);
      break;
    case 0:
      O << "\tssbb";
      break;
    case 4:
      O << "\tpssbb";
      break;
    }
    printAnnotation(O, Annot);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 333-334
```cpp
  if (!printAliasInstr(MI, Address, STI, O))
    printInstruction(MI, Address, STI, O);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 336-337
```cpp
  printAnnotation(O, Annot);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 339-353
```cpp
void ARMInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isReg()) {
    MCRegister Reg = Op.getReg();
    printRegName(O, Reg);
  } else if (Op.isImm()) {
    markup(O, Markup::Immediate) << '#' << formatImm(Op.getImm());
  } else {
    assert(Op.isExpr() && "unknown operand kind in printOperand");
    const MCExpr *Expr = Op.getExpr();
    switch (Expr->getKind()) {
    case MCExpr::Binary:
      O << '#';
      MAI.printExpr(O, *Expr);
```
- EN: Implements `ARMInstPrinter::printOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 354-368
```cpp
      break;
    case MCExpr::Constant: {
      // If a symbolic branch target was added as a constant expression then
      // print that address in hex. And only print 32 unsigned bits for the
      // address.
      const MCConstantExpr *Constant = cast<MCConstantExpr>(Expr);
      int64_t TargetAddress;
      if (!Constant->evaluateAsAbsolute(TargetAddress)) {
        O << '#';
        MAI.printExpr(O, *Expr);
      } else {
        O << "0x";
        O.write_hex(static_cast<uint32_t>(TargetAddress));
      }
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 369-377
```cpp
    }
    default:
      // FIXME: Should we always treat this as if it is a constant literal and
      // prefix it with '#'?
      MAI.printExpr(O, *Expr);
      break;
    }
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 379-391
```cpp
void ARMInstPrinter::printOperand(const MCInst *MI, uint64_t Address,
                                  unsigned OpNum, const MCSubtargetInfo &STI,
                                  raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNum);
  if (!Op.isImm() || !PrintBranchImmAsAddress || getUseMarkup())
    return printOperand(MI, OpNum, STI, O);
  uint64_t Target = ARM_MC::evaluateBranchTarget(MII.get(MI->getOpcode()),
                                                 Address, Op.getImm());
  Target &= 0xffffffff;
  O << formatHex(Target);
  if (CommentStream)
    *CommentStream << "imm = #" << formatImm(Op.getImm()) << '\n';
}
```
- EN: Implements `ARMInstPrinter::printOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 393-400
```cpp
void ARMInstPrinter::printThumbLdrLabelOperand(const MCInst *MI, unsigned OpNum,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  if (MO1.isExpr()) {
    MAI.printExpr(O, *MO1.getExpr());
    return;
  }
```
- EN: Implements `ARMInstPrinter::printThumbLdrLabelOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbLdrLabelOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 402-403
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[pc, ";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 405-406
```cpp
  int32_t OffImm = (int32_t)MO1.getImm();
  bool isSub = OffImm < 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 408-417
```cpp
  // Special value for #-0. All others are normal.
  if (OffImm == INT32_MIN)
    OffImm = 0;
  if (isSub) {
    markup(O, Markup::Immediate) << "#-" << formatImm(-OffImm);
  } else {
    markup(O, Markup::Immediate) << "#" << formatImm(OffImm);
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 419-429
```cpp
// so_reg is a 4-operand unit corresponding to register forms of the A5.1
// "Addressing Mode 1 - Data-processing operands" forms.  This includes:
//    REG 0   0           - e.g. R5
//    REG REG 0,SH_OPC    - e.g. R5, ROR R3
//    REG 0   IMM,SH_OPC  - e.g. R5, LSL #3
void ARMInstPrinter::printSORegRegOperand(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
  const MCOperand &MO3 = MI->getOperand(OpNum + 2);
```
- EN: Implements `ARMInstPrinter::printSORegRegOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printSORegRegOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 431-431
```cpp
  printRegName(O, MO1.getReg());
```
- EN: Declares `printRegName`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `printRegName`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 433-437
```cpp
  // Print the shift opc.
  ARM_AM::ShiftOpc ShOpc = ARM_AM::getSORegShOp(MO3.getImm());
  O << ", " << ARM_AM::getShiftOpcStr(ShOpc);
  if (ShOpc == ARM_AM::rrx)
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 439-442
```cpp
  O << ' ';
  printRegName(O, MO2.getReg());
  assert(ARM_AM::getSORegOffset(MO3.getImm()) == 0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 444-448
```cpp
void ARMInstPrinter::printSORegImmOperand(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printSORegImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printSORegImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 450-450
```cpp
  printRegName(O, MO1.getReg());
```
- EN: Declares `printRegName`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `printRegName`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 452-455
```cpp
  // Print the shift opc.
  printRegImmShift(O, ARM_AM::getSORegShOp(MO2.getImm()),
                   ARM_AM::getSORegOffset(MO2.getImm()), *this);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 457-459
```cpp
//===--------------------------------------------------------------------===//
// Addressing Mode #2
//===--------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 461-466
```cpp
void ARMInstPrinter::printAM2PreOrOffsetIndexOp(const MCInst *MI, unsigned Op,
                                                const MCSubtargetInfo &STI,
                                                raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(Op);
  const MCOperand &MO2 = MI->getOperand(Op + 1);
  const MCOperand &MO3 = MI->getOperand(Op + 2);
```
- EN: Implements `ARMInstPrinter::printAM2PreOrOffsetIndexOp`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAM2PreOrOffsetIndexOp`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 468-470
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 472-481
```cpp
  if (!MO2.getReg()) {
    if (ARM_AM::getAM2Offset(MO3.getImm())) { // Don't print +0.
      O << ", ";
      markup(O, Markup::Immediate)
          << "#" << ARM_AM::getAddrOpcStr(ARM_AM::getAM2Op(MO3.getImm()))
          << ARM_AM::getAM2Offset(MO3.getImm());
    }
    O << "]";
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 483-485
```cpp
  O << ", ";
  O << ARM_AM::getAddrOpcStr(ARM_AM::getAM2Op(MO3.getImm()));
  printRegName(O, MO2.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 487-490
```cpp
  printRegImmShift(O, ARM_AM::getAM2ShiftOpc(MO3.getImm()),
                   ARM_AM::getAM2Offset(MO3.getImm()), *this);
  O << "]";
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 492-496
```cpp
void ARMInstPrinter::printAddrModeTBB(const MCInst *MI, unsigned Op,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(Op);
  const MCOperand &MO2 = MI->getOperand(Op + 1);
```
- EN: Implements `ARMInstPrinter::printAddrModeTBB`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrModeTBB`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 498-504
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  O << ", ";
  printRegName(O, MO2.getReg());
  O << "]";
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 506-519
```cpp
void ARMInstPrinter::printAddrModeTBH(const MCInst *MI, unsigned Op,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(Op);
  const MCOperand &MO2 = MI->getOperand(Op + 1);
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  O << ", ";
  printRegName(O, MO2.getReg());
  O << ", lsl ";
  markup(O, Markup::Immediate) << "#1";
  O << "]";
}
```
- EN: Implements `ARMInstPrinter::printAddrModeTBH`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrModeTBH`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 521-524
```cpp
void ARMInstPrinter::printAddrMode2Operand(const MCInst *MI, unsigned Op,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(Op);
```
- EN: Implements `ARMInstPrinter::printAddrMode2Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode2Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 526-529
```cpp
  if (!MO1.isReg()) { // FIXME: This is for CP entries, but isn't right.
    printOperand(MI, Op, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 531-535
```cpp
#ifndef NDEBUG
  const MCOperand &MO3 = MI->getOperand(Op + 2);
  unsigned IdxMode = ARM_AM::getAM2IdxMode(MO3.getImm());
  assert(IdxMode != ARMII::IndexModePost && "Should be pre or offset index op");
#endif
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 537-538
```cpp
  printAM2PreOrOffsetIndexOp(MI, Op, STI, O);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 540-545
```cpp
void ARMInstPrinter::printAddrMode2OffsetOperand(const MCInst *MI,
                                                 unsigned OpNum,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printAddrMode2OffsetOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode2OffsetOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 547-553
```cpp
  if (!MO1.getReg()) {
    unsigned ImmOffs = ARM_AM::getAM2Offset(MO2.getImm());
    markup(O, Markup::Immediate)
        << '#' << ARM_AM::getAddrOpcStr(ARM_AM::getAM2Op(MO2.getImm()))
        << ImmOffs;
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 555-556
```cpp
  O << ARM_AM::getAddrOpcStr(ARM_AM::getAM2Op(MO2.getImm()));
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 558-560
```cpp
  printRegImmShift(O, ARM_AM::getAM2ShiftOpc(MO2.getImm()),
                   ARM_AM::getAM2Offset(MO2.getImm()), *this);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 562-564
```cpp
//===--------------------------------------------------------------------===//
// Addressing Mode #3
//===--------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 566-571
```cpp
void ARMInstPrinter::printAM3PreOrOffsetIndexOp(const MCInst *MI, unsigned Op,
                                                raw_ostream &O,
                                                bool AlwaysPrintImm0) {
  const MCOperand &MO1 = MI->getOperand(Op);
  const MCOperand &MO2 = MI->getOperand(Op + 1);
  const MCOperand &MO3 = MI->getOperand(Op + 2);
```
- EN: Implements `ARMInstPrinter::printAM3PreOrOffsetIndexOp`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAM3PreOrOffsetIndexOp`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 573-575
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << '[';
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 577-582
```cpp
  if (MO2.getReg()) {
    O << ", " << getAddrOpcStr(ARM_AM::getAM3Op(MO3.getImm()));
    printRegName(O, MO2.getReg());
    O << ']';
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 584-586
```cpp
  // If the op is sub we have to print the immediate even if it is 0
  unsigned ImmOffs = ARM_AM::getAM3Offset(MO3.getImm());
  ARM_AM::AddrOpc op = ARM_AM::getAM3Op(MO3.getImm());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 588-593
```cpp
  if (AlwaysPrintImm0 || ImmOffs || (op == ARM_AM::sub)) {
    O << ", ";
    markup(O, Markup::Immediate) << "#" << ARM_AM::getAddrOpcStr(op) << ImmOffs;
  }
  O << ']';
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 595-603
```cpp
template <bool AlwaysPrintImm0>
void ARMInstPrinter::printAddrMode3Operand(const MCInst *MI, unsigned Op,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(Op);
  if (!MO1.isReg()) { //  For label symbolic references.
    printOperand(MI, Op, STI, O);
    return;
  }
```
- EN: Implements `ARMInstPrinter::printAddrMode3Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode3Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 605-609
```cpp
  assert(ARM_AM::getAM3IdxMode(MI->getOperand(Op + 2).getImm()) !=
             ARMII::IndexModePost &&
         "unexpected idxmode");
  printAM3PreOrOffsetIndexOp(MI, Op, O, AlwaysPrintImm0);
}
```
- EN: Declares `ARM_AM::getAM3IdxMode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARM_AM::getAM3IdxMode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 611-616
```cpp
void ARMInstPrinter::printAddrMode3OffsetOperand(const MCInst *MI,
                                                 unsigned OpNum,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printAddrMode3OffsetOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode3OffsetOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 618-622
```cpp
  if (MO1.getReg()) {
    O << getAddrOpcStr(ARM_AM::getAM3Op(MO2.getImm()));
    printRegName(O, MO1.getReg());
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 624-628
```cpp
  unsigned ImmOffs = ARM_AM::getAM3Offset(MO2.getImm());
  markup(O, Markup::Immediate)
      << '#' << ARM_AM::getAddrOpcStr(ARM_AM::getAM3Op(MO2.getImm()))
      << ImmOffs;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 630-637
```cpp
void ARMInstPrinter::printPostIdxImm8Operand(const MCInst *MI, unsigned OpNum,
                                             const MCSubtargetInfo &STI,
                                             raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);
  unsigned Imm = MO.getImm();
  markup(O, Markup::Immediate)
      << '#' << ((Imm & 256) ? "" : "-") << (Imm & 0xff);
}
```
- EN: Implements `ARMInstPrinter::printPostIdxImm8Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPostIdxImm8Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 639-643
```cpp
void ARMInstPrinter::printPostIdxRegOperand(const MCInst *MI, unsigned OpNum,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printPostIdxRegOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPostIdxRegOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 645-647
```cpp
  O << (MO2.getImm() ? "" : "-");
  printRegName(O, MO1.getReg());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 649-656
```cpp
void ARMInstPrinter::printPostIdxImm8s4Operand(const MCInst *MI, unsigned OpNum,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);
  unsigned Imm = MO.getImm();
  markup(O, Markup::Immediate)
      << '#' << ((Imm & 256) ? "" : "-") << ((Imm & 0xff) << 2);
}
```
- EN: Implements `ARMInstPrinter::printPostIdxImm8s4Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPostIdxImm8s4Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 658-663
```cpp
template<int shift>
void ARMInstPrinter::printMveAddrModeRQOperand(const MCInst *MI, unsigned OpNum,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printMveAddrModeRQOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMveAddrModeRQOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 665-669
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  O << ", ";
  printRegName(O, MO2.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 671-672
```cpp
  if (shift > 0)
    printRegImmShift(O, ARM_AM::uxtw, shift, *this);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 674-675
```cpp
  O << "]";
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 677-683
```cpp
void ARMInstPrinter::printLdStmModeOperand(const MCInst *MI, unsigned OpNum,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  ARM_AM::AMSubMode Mode =
      ARM_AM::getAM4SubMode(MI->getOperand(OpNum).getImm());
  O << ARM_AM::getAMSubModeStr(Mode);
}
```
- EN: Implements `ARMInstPrinter::printLdStmModeOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printLdStmModeOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 685-690
```cpp
template <bool AlwaysPrintImm0>
void ARMInstPrinter::printAddrMode5Operand(const MCInst *MI, unsigned OpNum,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printAddrMode5Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode5Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 692-695
```cpp
  if (!MO1.isReg()) { // FIXME: This is for CP entries, but isn't right.
    printOperand(MI, OpNum, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 697-699
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 701-709
```cpp
  unsigned ImmOffs = ARM_AM::getAM5Offset(MO2.getImm());
  ARM_AM::AddrOpc Op = ARM_AM::getAM5Op(MO2.getImm());
  if (AlwaysPrintImm0 || ImmOffs || Op == ARM_AM::sub) {
    O << ", ";
    markup(O, Markup::Immediate)
        << "#" << ARM_AM::getAddrOpcStr(Op) << ImmOffs * 4;
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 711-716
```cpp
template <bool AlwaysPrintImm0>
void ARMInstPrinter::printAddrMode5FP16Operand(const MCInst *MI, unsigned OpNum,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum+1);
```
- EN: Implements `ARMInstPrinter::printAddrMode5FP16Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode5FP16Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 718-721
```cpp
  if (!MO1.isReg()) {   // FIXME: This is for CP entries, but isn't right.
    printOperand(MI, OpNum, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 723-725
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 727-736
```cpp
  unsigned ImmOffs = ARM_AM::getAM5FP16Offset(MO2.getImm());
  unsigned Op = ARM_AM::getAM5FP16Op(MO2.getImm());
  if (AlwaysPrintImm0 || ImmOffs || Op == ARM_AM::sub) {
    O << ", ";
    markup(O, Markup::Immediate)
        << "#" << ARM_AM::getAddrOpcStr(ARM_AM::getAM5FP16Op(MO2.getImm()))
        << ImmOffs * 2;
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 738-742
```cpp
void ARMInstPrinter::printAddrMode6Operand(const MCInst *MI, unsigned OpNum,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printAddrMode6Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode6Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 744-751
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  if (MO2.getImm()) {
    O << ":" << (MO2.getImm() << 3);
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 753-761
```cpp
void ARMInstPrinter::printAddrMode7Operand(const MCInst *MI, unsigned OpNum,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  O << "]";
}
```
- EN: Implements `ARMInstPrinter::printAddrMode7Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode7Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 763-774
```cpp
void ARMInstPrinter::printAddrMode6OffsetOperand(const MCInst *MI,
                                                 unsigned OpNum,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);
  if (!MO.getReg())
    O << "!";
  else {
    O << ", ";
    printRegName(O, MO.getReg());
  }
}
```
- EN: Implements `ARMInstPrinter::printAddrMode6OffsetOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrMode6OffsetOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 776-788
```cpp
void ARMInstPrinter::printBitfieldInvMaskImmOperand(const MCInst *MI,
                                                    unsigned OpNum,
                                                    const MCSubtargetInfo &STI,
                                                    raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);
  uint32_t v = ~MO.getImm();
  int32_t lsb = llvm::countr_zero(v);
  int32_t width = llvm::bit_width(v) - lsb;
  assert(MO.isImm() && "Not a valid bf_inv_mask_imm value!");
  markup(O, Markup::Immediate) << '#' << lsb;
  O << ", ";
  markup(O, Markup::Immediate) << '#' << width;
}
```
- EN: Implements `ARMInstPrinter::printBitfieldInvMaskImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printBitfieldInvMaskImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 790-795
```cpp
void ARMInstPrinter::printMemBOption(const MCInst *MI, unsigned OpNum,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  unsigned val = MI->getOperand(OpNum).getImm();
  O << ARM_MB::MemBOptToString(val, STI.hasFeature(ARM::HasV8Ops));
}
```
- EN: Implements `ARMInstPrinter::printMemBOption`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMemBOption`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 797-802
```cpp
void ARMInstPrinter::printInstSyncBOption(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  unsigned val = MI->getOperand(OpNum).getImm();
  O << ARM_ISB::InstSyncBOptToString(val);
}
```
- EN: Implements `ARMInstPrinter::printInstSyncBOption`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printInstSyncBOption`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 804-809
```cpp
void ARMInstPrinter::printTraceSyncBOption(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  unsigned val = MI->getOperand(OpNum).getImm();
  O << ARM_TSB::TraceSyncBOptToString(val);
}
```
- EN: Implements `ARMInstPrinter::printTraceSyncBOption`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printTraceSyncBOption`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 811-824
```cpp
void ARMInstPrinter::printShiftImmOperand(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  unsigned ShiftOp = MI->getOperand(OpNum).getImm();
  bool isASR = (ShiftOp & (1 << 5)) != 0;
  unsigned Amt = ShiftOp & 0x1f;
  if (isASR) {
    O << ", asr ";
    markup(O, Markup::Immediate) << "#" << (Amt == 0 ? 32 : Amt);
  } else if (Amt) {
    O << ", lsl ";
    markup(O, Markup::Immediate) << "#" << Amt;
  }
}
```
- EN: Implements `ARMInstPrinter::printShiftImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printShiftImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 826-835
```cpp
void ARMInstPrinter::printPKHLSLShiftImm(const MCInst *MI, unsigned OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNum).getImm();
  if (Imm == 0)
    return;
  assert(Imm > 0 && Imm < 32 && "Invalid PKH shift immediate value!");
  O << ", lsl ";
  markup(O, Markup::Immediate) << "#" << Imm;
}
```
- EN: Implements `ARMInstPrinter::printPKHLSLShiftImm`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPKHLSLShiftImm`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 837-847
```cpp
void ARMInstPrinter::printPKHASRShiftImm(const MCInst *MI, unsigned OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNum).getImm();
  // A shift amount of 32 is encoded as 0.
  if (Imm == 0)
    Imm = 32;
  assert(Imm > 0 && Imm <= 32 && "Invalid PKH shift immediate value!");
  O << ", asr ";
  markup(O, Markup::Immediate) << "#" << Imm;
}
```
- EN: Implements `ARMInstPrinter::printPKHASRShiftImm`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPKHASRShiftImm`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 849-858
```cpp
void ARMInstPrinter::printRegisterList(const MCInst *MI, unsigned OpNum,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  if (MI->getOpcode() != ARM::t2CLRM && MI->getOpcode() != ARM::VSCCLRMS) {
    assert(is_sorted(drop_begin(*MI, OpNum),
                     [&](const MCOperand &LHS, const MCOperand &RHS) {
                       return MRI.getEncodingValue(LHS.getReg()) <
                              MRI.getEncodingValue(RHS.getReg());
                     }));
  }
```
- EN: Implements `ARMInstPrinter::printRegisterList`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printRegisterList`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 860-867
```cpp
  O << "{";
  for (unsigned i = OpNum, e = MI->getNumOperands(); i != e; ++i) {
    if (i != OpNum)
      O << ", ";
    printRegName(O, MI->getOperand(i).getReg());
  }
  O << "}";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 869-876
```cpp
void ARMInstPrinter::printGPRPairOperand(const MCInst *MI, unsigned OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNum).getReg();
  printRegName(O, MRI.getSubReg(Reg, ARM::gsub_0));
  O << ", ";
  printRegName(O, MRI.getSubReg(Reg, ARM::gsub_1));
}
```
- EN: Implements `ARMInstPrinter::printGPRPairOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printGPRPairOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 878-886
```cpp
void ARMInstPrinter::printSetendOperand(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNum);
  if (Op.getImm())
    O << "be";
  else
    O << "le";
}
```
- EN: Implements `ARMInstPrinter::printSetendOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printSetendOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 888-892
```cpp
void ARMInstPrinter::printCPSIMod(const MCInst *MI, unsigned OpNum,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNum);
  O << ARM_PROC::IModToString(Op.getImm());
}
```
- EN: Implements `ARMInstPrinter::printCPSIMod`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printCPSIMod`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 894-900
```cpp
void ARMInstPrinter::printCPSIFlag(const MCInst *MI, unsigned OpNum,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNum);
  unsigned IFlags = Op.getImm();
  for (int i = 2; i >= 0; --i)
    if (IFlags & (1 << i))
      O << ARM_PROC::IFlagsToString(1 << i);
```
- EN: Implements `ARMInstPrinter::printCPSIFlag`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printCPSIFlag`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 902-904
```cpp
  if (IFlags == 0)
    O << "none";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 906-911
```cpp
void ARMInstPrinter::printMSRMaskOperand(const MCInst *MI, unsigned OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNum);
  const FeatureBitset &FeatureBits = STI.getFeatureBits();
  if (FeatureBits[ARM::FeatureMClass]) {
```
- EN: Implements `ARMInstPrinter::printMSRMaskOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMSRMaskOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 913-914
```cpp
    unsigned SYSm = Op.getImm() & 0xFFF; // 12-bit SYSm
    unsigned Opcode = MI->getOpcode();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 916-923
```cpp
    // For writes, handle extended mask bits if the DSP extension is present.
    if (Opcode == ARM::t2MSR_M && FeatureBits[ARM::FeatureDSP]) {
      auto TheReg =ARMSysReg::lookupMClassSysRegBy12bitSYSmValue(SYSm);
      if (TheReg && TheReg->isInRequiredFeatures({ARM::FeatureDSP})) {
          O << TheReg->Name;
          return;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 925-935
```cpp
    // Handle the basic 8-bit mask.
    SYSm &= 0xff;
    if (Opcode == ARM::t2MSR_M && FeatureBits [ARM::HasV7Ops]) {
      // ARMv7-M deprecates using MSR APSR without a _<bits> qualifier as an
      // alias for MSR APSR_nzcvq.
      auto TheReg = ARMSysReg::lookupMClassSysRegAPSRNonDeprecated(SYSm);
      if (TheReg) {
          O << TheReg->Name;
          return;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 937-941
```cpp
    auto TheReg = ARMSysReg::lookupMClassSysRegBy8bitSYSmValue(SYSm);
    if (TheReg) {
      O << TheReg->Name;
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 943-943
```cpp
    O << SYSm;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 945-946
```cpp
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 948-951
```cpp
  // As special cases, CPSR_f, CPSR_s and CPSR_fs prefer printing as
  // APSR_nzcvq, APSR_g and APSRnzcvqg, respectively.
  unsigned SpecRegRBit = Op.getImm() >> 4;
  unsigned Mask = Op.getImm() & 0xf;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 953-967
```cpp
  if (!SpecRegRBit && (Mask == 8 || Mask == 4 || Mask == 12)) {
    O << "APSR_";
    switch (Mask) {
    default:
      llvm_unreachable("Unexpected mask value!");
    case 4:
      O << "g";
      return;
    case 8:
      O << "nzcvq";
      return;
    case 12:
      O << "nzcvqg";
      return;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 968-968
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 970-973
```cpp
  if (SpecRegRBit)
    O << "SPSR";
  else
    O << "CPSR";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 975-986
```cpp
  if (Mask) {
    O << '_';
    if (Mask & 8)
      O << 'f';
    if (Mask & 4)
      O << 's';
    if (Mask & 2)
      O << 'x';
    if (Mask & 1)
      O << 'c';
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 988-994
```cpp
void ARMInstPrinter::printBankedRegOperand(const MCInst *MI, unsigned OpNum,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  uint32_t Banked = MI->getOperand(OpNum).getImm();
  auto TheReg = ARMBankedReg::lookupBankedRegByEncoding(Banked);
  assert(TheReg && "invalid banked register operand");
  std::string Name = TheReg->Name;
```
- EN: Implements `ARMInstPrinter::printBankedRegOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printBankedRegOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 996-1000
```cpp
  uint32_t isSPSR = (Banked & 0x20) >> 5;
  if (isSPSR)
    Name.replace(0, 4, "SPSR"); // convert 'spsr_' to 'SPSR_'
  O << Name;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1002-1011
```cpp
void ARMInstPrinter::printPredicateOperand(const MCInst *MI, unsigned OpNum,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  ARMCC::CondCodes CC = (ARMCC::CondCodes)MI->getOperand(OpNum).getImm();
  // Handle the undefined 15 CC value here for printing so we don't abort().
  if ((unsigned)CC == 15)
    O << "<und>";
  else if (CC != ARMCC::AL)
    O << ARMCondCodeToString(CC);
}
```
- EN: Implements `ARMInstPrinter::printPredicateOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPredicateOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1013-1020
```cpp
void ARMInstPrinter::printMandatoryRestrictedPredicateOperand(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  if ((ARMCC::CondCodes)MI->getOperand(OpNum).getImm() == ARMCC::HS)
    O << "cs";
  else
    printMandatoryPredicateOperand(MI, OpNum, STI, O);
}
```
- EN: Implements `ARMInstPrinter::printMandatoryRestrictedPredicateOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMandatoryRestrictedPredicateOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1022-1028
```cpp
void ARMInstPrinter::printMandatoryPredicateOperand(const MCInst *MI,
                                                    unsigned OpNum,
                                                    const MCSubtargetInfo &STI,
                                                    raw_ostream &O) {
  ARMCC::CondCodes CC = (ARMCC::CondCodes)MI->getOperand(OpNum).getImm();
  O << ARMCondCodeToString(CC);
}
```
- EN: Implements `ARMInstPrinter::printMandatoryPredicateOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMandatoryPredicateOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1030-1036
```cpp
void ARMInstPrinter::printMandatoryInvertedPredicateOperand(const MCInst *MI,
                                                            unsigned OpNum,
                                                            const MCSubtargetInfo &STI,
                                                            raw_ostream &O) {
  ARMCC::CondCodes CC = (ARMCC::CondCodes)MI->getOperand(OpNum).getImm();
  O << ARMCondCodeToString(ARMCC::getOppositeCondition(CC));
}
```
- EN: Implements `ARMInstPrinter::printMandatoryInvertedPredicateOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMandatoryInvertedPredicateOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1038-1046
```cpp
void ARMInstPrinter::printSBitModifierOperand(const MCInst *MI, unsigned OpNum,
                                              const MCSubtargetInfo &STI,
                                              raw_ostream &O) {
  if (MI->getOperand(OpNum).getReg()) {
    assert(MI->getOperand(OpNum).getReg() == ARM::CPSR &&
           "Expect ARM CPSR register!");
    O << 's';
  }
}
```
- EN: Implements `ARMInstPrinter::printSBitModifierOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printSBitModifierOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1048-1052
```cpp
void ARMInstPrinter::printNoHashImmediate(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  O << MI->getOperand(OpNum).getImm();
}
```
- EN: Implements `ARMInstPrinter::printNoHashImmediate`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printNoHashImmediate`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1054-1058
```cpp
void ARMInstPrinter::printPImmediate(const MCInst *MI, unsigned OpNum,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  O << "p" << MI->getOperand(OpNum).getImm();
}
```
- EN: Implements `ARMInstPrinter::printPImmediate`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPImmediate`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1060-1064
```cpp
void ARMInstPrinter::printCImmediate(const MCInst *MI, unsigned OpNum,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  O << "c" << MI->getOperand(OpNum).getImm();
}
```
- EN: Implements `ARMInstPrinter::printCImmediate`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printCImmediate`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1066-1070
```cpp
void ARMInstPrinter::printCoprocOptionImm(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  O << "{" << MI->getOperand(OpNum).getImm() << "}";
}
```
- EN: Implements `ARMInstPrinter::printCoprocOptionImm`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printCoprocOptionImm`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1072-1075
```cpp
void ARMInstPrinter::printPCLabel(const MCInst *MI, unsigned OpNum,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  llvm_unreachable("Unhandled PC-relative pseudo-instruction!");
}
```
- EN: Implements `ARMInstPrinter::printPCLabel`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printPCLabel`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1077-1081
```cpp
template <unsigned scale>
void ARMInstPrinter::printAdrLabelOperand(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);
```
- EN: Implements `ARMInstPrinter::printAdrLabelOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAdrLabelOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1083-1086
```cpp
  if (MO.isExpr()) {
    MAI.printExpr(O, *MO.getExpr());
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1088-1088
```cpp
  int32_t OffImm = (int32_t)MO.getImm() << scale;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1090-1097
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Immediate);
  if (OffImm == INT32_MIN)
    O << "#-0";
  else if (OffImm < 0)
    O << "#-" << -OffImm;
  else
    O << "#" << OffImm;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1099-1104
```cpp
void ARMInstPrinter::printThumbS4ImmOperand(const MCInst *MI, unsigned OpNum,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  markup(O, Markup::Immediate)
      << "#" << formatImm(MI->getOperand(OpNum).getImm() * 4);
}
```
- EN: Implements `ARMInstPrinter::printThumbS4ImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbS4ImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1106-1111
```cpp
void ARMInstPrinter::printThumbSRImm(const MCInst *MI, unsigned OpNum,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNum).getImm();
  markup(O, Markup::Immediate) << "#" << formatImm((Imm == 0 ? 32 : Imm));
}
```
- EN: Implements `ARMInstPrinter::printThumbSRImm`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbSRImm`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1113-1126
```cpp
void ARMInstPrinter::printThumbITMask(const MCInst *MI, unsigned OpNum,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  // (3 - the number of trailing zeros) is the number of then / else.
  unsigned Mask = MI->getOperand(OpNum).getImm();
  unsigned NumTZ = llvm::countr_zero(Mask);
  assert(NumTZ <= 3 && "Invalid IT mask!");
  for (unsigned Pos = 3, e = NumTZ; Pos > e; --Pos) {
    if ((Mask >> Pos) & 1)
      O << 'e';
    else
      O << 't';
  }
}
```
- EN: Implements `ARMInstPrinter::printThumbITMask`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbITMask`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1128-1132
```cpp
void ARMInstPrinter::printThumbAddrModeRROperand(const MCInst *MI, unsigned Op,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(Op);
  const MCOperand &MO2 = MI->getOperand(Op + 1);
```
- EN: Implements `ARMInstPrinter::printThumbAddrModeRROperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbAddrModeRROperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1134-1137
```cpp
  if (!MO1.isReg()) { // FIXME: This is for CP entries, but isn't right.
    printOperand(MI, Op, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1139-1147
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  if (MCRegister RegNum = MO2.getReg()) {
    O << ", ";
    printRegName(O, RegNum);
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1149-1155
```cpp
void ARMInstPrinter::printThumbAddrModeImm5SOperand(const MCInst *MI,
                                                    unsigned Op,
                                                    const MCSubtargetInfo &STI,
                                                    raw_ostream &O,
                                                    unsigned Scale) {
  const MCOperand &MO1 = MI->getOperand(Op);
  const MCOperand &MO2 = MI->getOperand(Op + 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1157-1160
```cpp
  if (!MO1.isReg()) { // FIXME: This is for CP entries, but isn't right.
    printOperand(MI, Op, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1162-1170
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  if (unsigned ImmOffs = MO2.getImm()) {
    O << ", ";
    markup(O, Markup::Immediate) << "#" << formatImm(ImmOffs * Scale);
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1172-1177
```cpp
void ARMInstPrinter::printThumbAddrModeImm5S1Operand(const MCInst *MI,
                                                     unsigned Op,
                                                     const MCSubtargetInfo &STI,
                                                     raw_ostream &O) {
  printThumbAddrModeImm5SOperand(MI, Op, STI, O, 1);
}
```
- EN: Implements `ARMInstPrinter::printThumbAddrModeImm5S1Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbAddrModeImm5S1Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1179-1184
```cpp
void ARMInstPrinter::printThumbAddrModeImm5S2Operand(const MCInst *MI,
                                                     unsigned Op,
                                                     const MCSubtargetInfo &STI,
                                                     raw_ostream &O) {
  printThumbAddrModeImm5SOperand(MI, Op, STI, O, 2);
}
```
- EN: Implements `ARMInstPrinter::printThumbAddrModeImm5S2Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbAddrModeImm5S2Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1186-1191
```cpp
void ARMInstPrinter::printThumbAddrModeImm5S4Operand(const MCInst *MI,
                                                     unsigned Op,
                                                     const MCSubtargetInfo &STI,
                                                     raw_ostream &O) {
  printThumbAddrModeImm5SOperand(MI, Op, STI, O, 4);
}
```
- EN: Implements `ARMInstPrinter::printThumbAddrModeImm5S4Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbAddrModeImm5S4Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1193-1197
```cpp
void ARMInstPrinter::printThumbAddrModeSPOperand(const MCInst *MI, unsigned Op,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  printThumbAddrModeImm5SOperand(MI, Op, STI, O, 4);
}
```
- EN: Implements `ARMInstPrinter::printThumbAddrModeSPOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printThumbAddrModeSPOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1199-1207
```cpp
// Constant shifts t2_so_reg is a 2-operand unit corresponding to the Thumb2
// register with shift forms.
// REG 0   0           - e.g. R5
// REG IMM, SH_OPC     - e.g. R5, LSL #3
void ARMInstPrinter::printT2SOOperand(const MCInst *MI, unsigned OpNum,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printT2SOOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printT2SOOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1209-1210
```cpp
  MCRegister Reg = MO1.getReg();
  printRegName(O, Reg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1212-1216
```cpp
  // Print the shift opc.
  assert(MO2.isImm() && "Not a valid t2_so_reg value!");
  printRegImmShift(O, ARM_AM::getSORegShOp(MO2.getImm()),
                   ARM_AM::getSORegOffset(MO2.getImm()), *this);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1218-1223
```cpp
template <bool AlwaysPrintImm0>
void ARMInstPrinter::printAddrModeImm12Operand(const MCInst *MI, unsigned OpNum,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printAddrModeImm12Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printAddrModeImm12Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1225-1228
```cpp
  if (!MO1.isReg()) { // FIXME: This is for CP entries, but isn't right.
    printOperand(MI, OpNum, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1230-1232
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1234-1247
```cpp
  int32_t OffImm = (int32_t)MO2.getImm();
  bool isSub = OffImm < 0;
  // Special value for #-0. All others are normal.
  if (OffImm == INT32_MIN)
    OffImm = 0;
  if (isSub) {
    O << ", ";
    markup(O, Markup::Immediate) << "#-" << formatImm(-OffImm);
  } else if (AlwaysPrintImm0 || OffImm > 0) {
    O << ", ";
    markup(O, Markup::Immediate) << "#" << formatImm(OffImm);
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1249-1255
```cpp
template <bool AlwaysPrintImm0>
void ARMInstPrinter::printT2AddrModeImm8Operand(const MCInst *MI,
                                                unsigned OpNum,
                                                const MCSubtargetInfo &STI,
                                                raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1257-1259
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1261-1274
```cpp
  int32_t OffImm = (int32_t)MO2.getImm();
  bool isSub = OffImm < 0;
  // Don't print +0.
  if (OffImm == INT32_MIN)
    OffImm = 0;
  if (isSub) {
    O << ", ";
    markup(O, Markup::Immediate) << "#-" << -OffImm;
  } else if (AlwaysPrintImm0 || OffImm > 0) {
    O << ", ";
    markup(O, Markup::Immediate) << "#" << OffImm;
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1276-1282
```cpp
template <bool AlwaysPrintImm0>
void ARMInstPrinter::printT2AddrModeImm8s4Operand(const MCInst *MI,
                                                  unsigned OpNum,
                                                  const MCSubtargetInfo &STI,
                                                  raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1284-1287
```cpp
  if (!MO1.isReg()) { //  For label symbolic references.
    printOperand(MI, OpNum, STI, O);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1289-1291
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1293-1294
```cpp
  int32_t OffImm = (int32_t)MO2.getImm();
  bool isSub = OffImm < 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1296-1296
```cpp
  assert(((OffImm & 0x3) == 0) && "Not a valid immediate!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1298-1309
```cpp
  // Don't print +0.
  if (OffImm == INT32_MIN)
    OffImm = 0;
  if (isSub) {
    O << ", ";
    markup(O, Markup::Immediate) << "#-" << -OffImm;
  } else if (AlwaysPrintImm0 || OffImm > 0) {
    O << ", ";
    markup(O, Markup::Immediate) << "#" << OffImm;
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1311-1315
```cpp
void ARMInstPrinter::printT2AddrModeImm0_1020s4Operand(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
```
- EN: Implements `ARMInstPrinter::printT2AddrModeImm0_1020s4Operand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printT2AddrModeImm0_1020s4Operand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1317-1325
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
  if (MO2.getImm()) {
    O << ", ";
    markup(O, Markup::Immediate) << "#" << formatImm(MO2.getImm() * 4);
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1327-1340
```cpp
void ARMInstPrinter::printT2AddrModeImm8OffsetOperand(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  int32_t OffImm = (int32_t)MO1.getImm();
  O << ", ";
  WithMarkup ScopedMarkup = markup(O, Markup::Immediate);
  if (OffImm == INT32_MIN)
    O << "#-0";
  else if (OffImm < 0)
    O << "#-" << -OffImm;
  else
    O << "#" << OffImm;
}
```
- EN: Implements `ARMInstPrinter::printT2AddrModeImm8OffsetOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printT2AddrModeImm8OffsetOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1342-1346
```cpp
void ARMInstPrinter::printT2AddrModeImm8s4OffsetOperand(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  int32_t OffImm = (int32_t)MO1.getImm();
```
- EN: Implements `ARMInstPrinter::printT2AddrModeImm8s4OffsetOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printT2AddrModeImm8s4OffsetOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1348-1348
```cpp
  assert(((OffImm & 0x3) == 0) && "Not a valid immediate!");
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1350-1358
```cpp
  O << ", ";
  WithMarkup ScopedMarkup = markup(O, Markup::Immediate);
  if (OffImm == INT32_MIN)
    O << "#-0";
  else if (OffImm < 0)
    O << "#-" << -OffImm;
  else
    O << "#" << OffImm;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1360-1366
```cpp
void ARMInstPrinter::printT2AddrModeSoRegOperand(const MCInst *MI,
                                                 unsigned OpNum,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  const MCOperand &MO1 = MI->getOperand(OpNum);
  const MCOperand &MO2 = MI->getOperand(OpNum + 1);
  const MCOperand &MO3 = MI->getOperand(OpNum + 2);
```
- EN: Implements `ARMInstPrinter::printT2AddrModeSoRegOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printT2AddrModeSoRegOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1368-1370
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Memory);
  O << "[";
  printRegName(O, MO1.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1372-1374
```cpp
  assert(MO2.getReg() && "Invalid so_reg load / store address!");
  O << ", ";
  printRegName(O, MO2.getReg());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1376-1383
```cpp
  unsigned ShAmt = MO3.getImm();
  if (ShAmt) {
    assert(ShAmt <= 3 && "Not a valid Thumb2 addressing mode!");
    O << ", lsl ";
    markup(O, Markup::Immediate) << "#" << ShAmt;
  }
  O << "]";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1385-1390
```cpp
void ARMInstPrinter::printFPImmOperand(const MCInst *MI, unsigned OpNum,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNum);
  markup(O, Markup::Immediate) << '#' << ARM_AM::getFPImmFloat(MO.getImm());
}
```
- EN: Implements `ARMInstPrinter::printFPImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printFPImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1392-1397
```cpp
void ARMInstPrinter::printVMOVModImmOperand(const MCInst *MI, unsigned OpNum,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  unsigned EncodedImm = MI->getOperand(OpNum).getImm();
  unsigned EltBits;
  uint64_t Val = ARM_AM::decodeVMOVModImm(EncodedImm, EltBits);
```
- EN: Implements `ARMInstPrinter::printVMOVModImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVMOVModImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1399-1402
```cpp
  WithMarkup ScopedMarkup = markup(O, Markup::Immediate);
  O << "#0x";
  O.write_hex(Val);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1404-1409
```cpp
void ARMInstPrinter::printImmPlusOneOperand(const MCInst *MI, unsigned OpNum,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNum).getImm();
  markup(O, Markup::Immediate) << "#" << formatImm(Imm + 1);
}
```
- EN: Implements `ARMInstPrinter::printImmPlusOneOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printImmPlusOneOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1411-1420
```cpp
void ARMInstPrinter::printRotImmOperand(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNum).getImm();
  if (Imm == 0)
    return;
  assert(Imm <= 3 && "illegal ror immediate!");
  O << ", ror ";
  markup(O, Markup::Immediate) << "#" << 8 * Imm;
}
```
- EN: Implements `ARMInstPrinter::printRotImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printRotImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1422-1425
```cpp
void ARMInstPrinter::printModImmOperand(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  MCOperand Op = MI->getOperand(OpNum);
```
- EN: Implements `ARMInstPrinter::printModImmOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printModImmOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1427-1429
```cpp
  // Support for fixups (MCFixup)
  if (Op.isExpr())
    return printOperand(MI, OpNum, STI, O);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1431-1432
```cpp
  unsigned Bits = Op.getImm() & 0xFF;
  unsigned Rot = (Op.getImm() & 0xF00) >> 7;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1434-1444
```cpp
  bool PrintUnsigned = false;
  switch (MI->getOpcode()) {
  case ARM::MOVi:
    // Movs to PC should be treated unsigned
    PrintUnsigned = (MI->getOperand(OpNum - 1).getReg() == ARM::PC);
    break;
  case ARM::MSRi:
    // Movs to special registers should be treated unsigned
    PrintUnsigned = true;
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1446-1455
```cpp
  int32_t Rotated = llvm::rotr<uint32_t>(Bits, Rot);
  if (ARM_AM::getSOImmVal(Rotated) == Op.getImm()) {
    // #rot has the least possible value
    O << "#";
    if (PrintUnsigned)
      markup(O, Markup::Immediate) << static_cast<uint32_t>(Rotated);
    else
      markup(O, Markup::Immediate) << Rotated;
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1457-1462
```cpp
  // Explicit #bits, #rot implied
  O << "#";
  markup(O, Markup::Immediate) << Bits;
  O << ", #";
  markup(O, Markup::Immediate) << Rot;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1464-1467
```cpp
void ARMInstPrinter::printFBits16(const MCInst *MI, unsigned OpNum,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  markup(O, Markup::Immediate) << "#" << 16 - MI->getOperand(OpNum).getImm();
}
```
- EN: Implements `ARMInstPrinter::printFBits16`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printFBits16`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1469-1472
```cpp
void ARMInstPrinter::printFBits32(const MCInst *MI, unsigned OpNum,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  markup(O, Markup::Immediate) << "#" << 32 - MI->getOperand(OpNum).getImm();
}
```
- EN: Implements `ARMInstPrinter::printFBits32`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printFBits32`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1474-1478
```cpp
void ARMInstPrinter::printVectorIndex(const MCInst *MI, unsigned OpNum,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  O << "[" << MI->getOperand(OpNum).getImm() << "]";
}
```
- EN: Implements `ARMInstPrinter::printVectorIndex`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorIndex`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1480-1486
```cpp
void ARMInstPrinter::printVectorListOne(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << "}";
}
```
- EN: Implements `ARMInstPrinter::printVectorListOne`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorListOne`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1488-1499
```cpp
void ARMInstPrinter::printVectorListTwo(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNum).getReg();
  MCRegister Reg0 = MRI.getSubReg(Reg, ARM::dsub_0);
  MCRegister Reg1 = MRI.getSubReg(Reg, ARM::dsub_1);
  O << "{";
  printRegName(O, Reg0);
  O << ", ";
  printRegName(O, Reg1);
  O << "}";
}
```
- EN: Implements `ARMInstPrinter::printVectorListTwo`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorListTwo`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1501-1512
```cpp
void ARMInstPrinter::printVectorListTwoSpaced(const MCInst *MI, unsigned OpNum,
                                              const MCSubtargetInfo &STI,
                                              raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNum).getReg();
  MCRegister Reg0 = MRI.getSubReg(Reg, ARM::dsub_0);
  MCRegister Reg1 = MRI.getSubReg(Reg, ARM::dsub_2);
  O << "{";
  printRegName(O, Reg0);
  O << ", ";
  printRegName(O, Reg1);
  O << "}";
}
```
- EN: Implements `ARMInstPrinter::printVectorListTwoSpaced`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorListTwoSpaced`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1514-1527
```cpp
void ARMInstPrinter::printVectorListThree(const MCInst *MI, unsigned OpNum,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 1);
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << "}";
}
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1529-1543
```cpp
void ARMInstPrinter::printVectorListFour(const MCInst *MI, unsigned OpNum,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 1);
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 3);
  O << "}";
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1544-1544
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1546-1553
```cpp
void ARMInstPrinter::printVectorListOneAllLanes(const MCInst *MI,
                                                unsigned OpNum,
                                                const MCSubtargetInfo &STI,
                                                raw_ostream &O) {
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << "[]}";
}
```
- EN: Implements `ARMInstPrinter::printVectorListOneAllLanes`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorListOneAllLanes`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1555-1567
```cpp
void ARMInstPrinter::printVectorListTwoAllLanes(const MCInst *MI,
                                                unsigned OpNum,
                                                const MCSubtargetInfo &STI,
                                                raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNum).getReg();
  MCRegister Reg0 = MRI.getSubReg(Reg, ARM::dsub_0);
  MCRegister Reg1 = MRI.getSubReg(Reg, ARM::dsub_1);
  O << "{";
  printRegName(O, Reg0);
  O << "[], ";
  printRegName(O, Reg1);
  O << "[]}";
}
```
- EN: Implements `ARMInstPrinter::printVectorListTwoAllLanes`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorListTwoAllLanes`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1569-1583
```cpp
void ARMInstPrinter::printVectorListThreeAllLanes(const MCInst *MI,
                                                  unsigned OpNum,
                                                  const MCSubtargetInfo &STI,
                                                  raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 1);
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << "[]}";
}
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1585-1599
```cpp
void ARMInstPrinter::printVectorListFourAllLanes(const MCInst *MI,
                                                 unsigned OpNum,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 1);
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 3);
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1600-1601
```cpp
  O << "[]}";
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1603-1614
```cpp
void ARMInstPrinter::printVectorListTwoSpacedAllLanes(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNum).getReg();
  MCRegister Reg0 = MRI.getSubReg(Reg, ARM::dsub_0);
  MCRegister Reg1 = MRI.getSubReg(Reg, ARM::dsub_2);
  O << "{";
  printRegName(O, Reg0);
  O << "[], ";
  printRegName(O, Reg1);
  O << "[]}";
}
```
- EN: Implements `ARMInstPrinter::printVectorListTwoSpacedAllLanes`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVectorListTwoSpacedAllLanes`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1616-1629
```cpp
void ARMInstPrinter::printVectorListThreeSpacedAllLanes(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 4);
  O << "[]}";
}
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1631-1645
```cpp
void ARMInstPrinter::printVectorListFourSpacedAllLanes(
    const MCInst *MI, unsigned OpNum, const MCSubtargetInfo &STI,
    raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 4);
  O << "[], ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 6);
  O << "[]}";
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1646-1646
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1648-1662
```cpp
void ARMInstPrinter::printVectorListThreeSpaced(const MCInst *MI,
                                                unsigned OpNum,
                                                const MCSubtargetInfo &STI,
                                                raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 4);
  O << "}";
}
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1664-1678
```cpp
void ARMInstPrinter::printVectorListFourSpaced(const MCInst *MI, unsigned OpNum,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  // Normally, it's not safe to use register enum values directly with
  // addition to get the next register, but for VFP registers, the
  // sort order is guaranteed because they're all of the form D<n>.
  O << "{";
  printRegName(O, MI->getOperand(OpNum).getReg());
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 2);
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 4);
  O << ", ";
  printRegName(O, MI->getOperand(OpNum).getReg() + 6);
  O << "}";
```
- EN: Defines enumeration `values` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `values`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 1679-1679
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1681-1693
```cpp
template<unsigned NumRegs>
void ARMInstPrinter::printMVEVectorList(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNum).getReg();
  const char *Prefix = "{";
  for (unsigned i = 0; i < NumRegs; i++) {
    O << Prefix;
    printRegName(O, MRI.getSubReg(Reg, ARM::qsub_0 + i));
    Prefix = ", ";
  }
  O << "}";
}
```
- EN: Implements `ARMInstPrinter::printMVEVectorList`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMVEVectorList`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1695-1701
```cpp
template<int64_t Angle, int64_t Remainder>
void ARMInstPrinter::printComplexRotationOp(const MCInst *MI, unsigned OpNo,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  unsigned Val = MI->getOperand(OpNo).getImm();
  O << "#" << (Val * Angle) + Remainder;
}
```
- EN: Implements `ARMInstPrinter::printComplexRotationOp`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printComplexRotationOp`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1703-1709
```cpp
void ARMInstPrinter::printVPTPredicateOperand(const MCInst *MI, unsigned OpNum,
                                              const MCSubtargetInfo &STI,
                                              raw_ostream &O) {
  ARMVCC::VPTCodes CC = (ARMVCC::VPTCodes)MI->getOperand(OpNum).getImm();
  if (CC != ARMVCC::None)
    O << ARMVPTPredToString(CC);
}
```
- EN: Implements `ARMInstPrinter::printVPTPredicateOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVPTPredicateOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1711-1725
```cpp
void ARMInstPrinter::printVPTMask(const MCInst *MI, unsigned OpNum,
                                  const MCSubtargetInfo &STI,
                                  raw_ostream &O) {
  // (3 - the number of trailing zeroes) is the number of them / else.
  unsigned Mask = MI->getOperand(OpNum).getImm();
  unsigned NumTZ = llvm::countr_zero(Mask);
  assert(NumTZ <= 3 && "Invalid VPT mask!");
  for (unsigned Pos = 3, e = NumTZ; Pos > e; --Pos) {
    bool T = ((Mask >> Pos) & 1) == 0;
    if (T)
      O << 't';
    else
      O << 'e';
  }
}
```
- EN: Implements `ARMInstPrinter::printVPTMask`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printVPTMask`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1727-1733
```cpp
void ARMInstPrinter::printMveSaturateOp(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  uint32_t Val = MI->getOperand(OpNum).getImm();
  assert(Val <= 1 && "Invalid MVE saturate operand");
  O << "#" << (Val == 1 ? 48 : 64);
}
```
- EN: Implements `ARMInstPrinter::printMveSaturateOp`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMInstPrinter::printMveSaturateOp`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: instruction pretty-printing.
  - CN: 核心职责：指令格式化打印。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMInstPrinter.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `Utils/ARMBaseInfo.h`, `ARMGenAsmWriter.inc`.
  - CN: 后端本地头文件：`ARMInstPrinter.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `Utils/ARMBaseInfo.h`, `ARMGenAsmWriter.inc`。
- EN: LLVM infrastructure headers: `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrAnalysis.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/Support/Casting.h` ... (+3 more).
  - CN: LLVM 基础设施头文件：`llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrAnalysis.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/Support/Casting.h` ... (+3 more)。
- EN: Standard/system headers: `cassert`, `cstdint`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
