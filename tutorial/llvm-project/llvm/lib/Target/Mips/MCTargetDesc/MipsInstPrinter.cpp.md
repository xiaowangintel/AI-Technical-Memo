# MipsInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MCTargetDesc/MipsInstPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsInstPrinter` for the Mips backend, focusing on instruction pretty-printing.
- 用途 (CN): 实现 Mips 后端中的 `MipsInstPrinter`，重点处理指令格式化打印。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsInstPrinter.cpp - Convert Mips MCInst to assembly syntax ------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class prints an Mips MCInst to a .s file.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-24
```cpp
#include "MipsInstPrinter.h"
#include "Mips.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 26-26
```cpp
#define DEBUG_TYPE "asm-printer"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 28-29
```cpp
#define PRINT_ALIAS_INSTR
#include "MipsGenAsmWriter.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 31-35
```cpp
template<unsigned R>
static bool isReg(const MCInst &MI, unsigned OpNo) {
  assert(MI.getOperand(OpNo).isReg() && "Register operand expected.");
  return MI.getOperand(OpNo).getReg() == R;
}
```
- EN: Implements `isReg`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isReg`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 37-51
```cpp
const char* Mips::MipsFCCToString(Mips::CondCode CC) {
  switch (CC) {
  case FCOND_F:
  case FCOND_T:   return "f";
  case FCOND_UN:
  case FCOND_OR:  return "un";
  case FCOND_OEQ:
  case FCOND_UNE: return "eq";
  case FCOND_UEQ:
  case FCOND_ONE: return "ueq";
  case FCOND_OLT:
  case FCOND_UGE: return "olt";
  case FCOND_ULT:
  case FCOND_OGE: return "ult";
  case FCOND_OLE:
```
- EN: Implements `Mips::MipsFCCToString`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `Mips::MipsFCCToString`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 52-66
```cpp
  case FCOND_UGT: return "ole";
  case FCOND_ULE:
  case FCOND_OGT: return "ule";
  case FCOND_SF:
  case FCOND_ST:  return "sf";
  case FCOND_NGLE:
  case FCOND_GLE: return "ngle";
  case FCOND_SEQ:
  case FCOND_SNE: return "seq";
  case FCOND_NGL:
  case FCOND_GL:  return "ngl";
  case FCOND_LT:
  case FCOND_NLT: return "lt";
  case FCOND_NGE:
  case FCOND_GE:  return "nge";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 67-73
```cpp
  case FCOND_LE:
  case FCOND_NLE: return "le";
  case FCOND_NGT:
  case FCOND_GT:  return "ngt";
  }
  llvm_unreachable("Impossible condition code!");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 75-78
```cpp
void MipsInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  markup(OS, Markup::Register)
      << '$' << StringRef(getRegisterName(Reg)).lower();
}
```
- EN: Implements `MipsInstPrinter::printRegName`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printRegName`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 80-94
```cpp
void MipsInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                StringRef Annot, const MCSubtargetInfo &STI,
                                raw_ostream &O) {
  switch (MI->getOpcode()) {
  default:
    break;
  case Mips::RDHWR:
  case Mips::RDHWR64:
    O << "\t.set\tpush\n";
    O << "\t.set\tmips32r2\n";
    break;
  case Mips::Save16:
    O << "\tsave\t";
    printSaveRestore(MI, STI, O);
    O << " # 16 bit inst\n";
```
- EN: Implements `MipsInstPrinter::printInst`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printInst`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 95-109
```cpp
    return;
  case Mips::SaveX16:
    O << "\tsave\t";
    printSaveRestore(MI, STI, O);
    O << "\n";
    return;
  case Mips::Restore16:
    O << "\trestore\t";
    printSaveRestore(MI, STI, O);
    O << " # 16 bit inst\n";
    return;
  case Mips::RestoreX16:
    O << "\trestore\t";
    printSaveRestore(MI, STI, O);
    O << "\n";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 110-111
```cpp
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 113-117
```cpp
  // Try to print any aliases first.
  if (!printAliasInstr(MI, Address, STI, O) &&
      !printAlias(*MI, Address, STI, O))
    printInstruction(MI, Address, STI, O);
  printAnnotation(O, Annot);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 119-126
```cpp
  switch (MI->getOpcode()) {
  default:
    break;
  case Mips::RDHWR:
  case Mips::RDHWR64:
    O << "\n\t.set\tpop";
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 128-134
```cpp
void MipsInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isReg()) {
    printRegName(O, Op.getReg());
    return;
  }
```
- EN: Implements `MipsInstPrinter::printOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 136-139
```cpp
  if (Op.isImm()) {
    markup(O, Markup::Immediate) << formatImm(Op.getImm());
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 141-143
```cpp
  assert(Op.isExpr() && "unknown operand kind in printOperand");
  MAI.printExpr(O, *Op.getExpr());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 145-150
```cpp
void MipsInstPrinter::printJumpOperand(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (!Op.isImm())
    return printOperand(MI, OpNo, STI, O);
```
- EN: Implements `MipsInstPrinter::printJumpOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printJumpOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 152-156
```cpp
  if (PrintBranchImmAsAddress)
    markup(O, Markup::Immediate) << formatHex(Op.getImm());
  else
    markup(O, Markup::Immediate) << formatImm(Op.getImm());
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 158-164
```cpp
void MipsInstPrinter::printBranchOperand(const MCInst *MI, uint64_t Address,
                                         unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (!Op.isImm())
    return printOperand(MI, OpNo, STI, O);
```
- EN: Implements `MipsInstPrinter::printBranchOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printBranchOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 166-176
```cpp
  if (PrintBranchImmAsAddress) {
    uint64_t Target = Address + Op.getImm();
    if (STI.hasFeature(Mips::FeatureMips32))
      Target &= 0xffffffff;
    else if (STI.hasFeature(Mips::FeatureMips16))
      Target &= 0xffff;
    markup(O, Markup::Immediate) << formatHex(Target);
  } else {
    markup(O, Markup::Immediate) << formatImm(Op.getImm());
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 178-189
```cpp
template <unsigned Bits, unsigned Offset>
void MipsInstPrinter::printUImm(const MCInst *MI, int opNum,
                                const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(opNum);
  if (MO.isImm()) {
    uint64_t Imm = MO.getImm();
    Imm -= Offset;
    Imm &= (1 << Bits) - 1;
    Imm += Offset;
    markup(O, Markup::Immediate) << formatImm(Imm);
    return;
  }
```
- EN: Implements `MipsInstPrinter::printUImm`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printUImm`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-192
```cpp
  printOperand(MI, opNum, STI, O);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 194-199
```cpp
void MipsInstPrinter::printMemOperand(const MCInst *MI, int opNum,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  // Load/Store memory operands -- imm($reg)
  // If PIC target the target is loaded as the
  // pattern lw $25,%call16($28)
```
- EN: Implements `MipsInstPrinter::printMemOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printMemOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 201-214
```cpp
  // opNum can be invalid if instruction had reglist as operand.
  // MemOperand is always last operand of instruction (base + offset).
  switch (MI->getOpcode()) {
  default:
    break;
  case Mips::SWM32_MM:
  case Mips::LWM32_MM:
  case Mips::SWM16_MM:
  case Mips::SWM16_MMR6:
  case Mips::LWM16_MM:
  case Mips::LWM16_MMR6:
    opNum = MI->getNumOperands() - 2;
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 216-221
```cpp
  WithMarkup M = markup(O, Markup::Memory);
  printOperand(MI, opNum + 1, STI, O);
  O << "(";
  printOperand(MI, opNum, STI, O);
  O << ")";
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 223-231
```cpp
void MipsInstPrinter::printMemOperandEA(const MCInst *MI, int opNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  // when using stack locations for not load/store instructions
  // print the same way as all normal 3 operand instructions.
  printOperand(MI, opNum, STI, O);
  O << ", ";
  printOperand(MI, opNum + 1, STI, O);
}
```
- EN: Implements `MipsInstPrinter::printMemOperandEA`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printMemOperandEA`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 233-238
```cpp
void MipsInstPrinter::printFCCOperand(const MCInst *MI, int opNum,
                                      const MCSubtargetInfo & /* STI */,
                                      raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(opNum);
  O << MipsFCCToString((Mips::CondCode)MO.getImm());
}
```
- EN: Implements `MipsInstPrinter::printFCCOperand`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printFCCOperand`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 240-243
```cpp
void MipsInstPrinter::
printSHFMask(const MCInst *MI, int opNum, raw_ostream &O) {
  llvm_unreachable("TODO");
}
```
- EN: Implements `printSHFMask`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `printSHFMask`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 245-255
```cpp
bool MipsInstPrinter::printAlias(const char *Str, const MCInst &MI,
                                 uint64_t Address, unsigned OpNo,
                                 const MCSubtargetInfo &STI, raw_ostream &OS,
                                 bool IsBranch) {
  OS << "\t" << Str << "\t";
  if (IsBranch)
    printBranchOperand(&MI, Address, OpNo, STI, OS);
  else
    printOperand(&MI, OpNo, STI, OS);
  return true;
}
```
- EN: Implements `MipsInstPrinter::printAlias`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printAlias`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 257-268
```cpp
bool MipsInstPrinter::printAlias(const char *Str, const MCInst &MI,
                                 uint64_t Address, unsigned OpNo0,
                                 unsigned OpNo1, const MCSubtargetInfo &STI,
                                 raw_ostream &OS, bool IsBranch) {
  printAlias(Str, MI, Address, OpNo0, STI, OS, IsBranch);
  OS << ", ";
  if (IsBranch)
    printBranchOperand(&MI, Address, OpNo1, STI, OS);
  else
    printOperand(&MI, OpNo1, STI, OS);
  return true;
}
```
- EN: Implements `MipsInstPrinter::printAlias`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printAlias`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 270-284
```cpp
bool MipsInstPrinter::printAlias(const MCInst &MI, uint64_t Address,
                                 const MCSubtargetInfo &STI, raw_ostream &OS) {
  switch (MI.getOpcode()) {
  case Mips::BEQ:
  case Mips::BEQ_MM:
    // beq $zero, $zero, $L2 => b $L2
    // beq $r0, $zero, $L2 => beqz $r0, $L2
    return (isReg<Mips::ZERO>(MI, 0) && isReg<Mips::ZERO>(MI, 1) &&
            printAlias("b", MI, Address, 2, STI, OS, true)) ||
           (isReg<Mips::ZERO>(MI, 1) &&
            printAlias("beqz", MI, Address, 0, 2, STI, OS, true));
  case Mips::BEQ64:
    // beq $r0, $zero, $L2 => beqz $r0, $L2
    return isReg<Mips::ZERO_64>(MI, 1) &&
           printAlias("beqz", MI, Address, 0, 2, STI, OS, true);
```
- EN: Implements `MipsInstPrinter::printAlias`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printAlias`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 285-299
```cpp
  case Mips::BNE:
  case Mips::BNE_MM:
    // bne $r0, $zero, $L2 => bnez $r0, $L2
    return isReg<Mips::ZERO>(MI, 1) &&
           printAlias("bnez", MI, Address, 0, 2, STI, OS, true);
  case Mips::BNE64:
    // bne $r0, $zero, $L2 => bnez $r0, $L2
    return isReg<Mips::ZERO_64>(MI, 1) &&
           printAlias("bnez", MI, Address, 0, 2, STI, OS, true);
  case Mips::BGEZAL:
    // bgezal $zero, $L1 => bal $L1
    return isReg<Mips::ZERO>(MI, 0) &&
           printAlias("bal", MI, Address, 1, STI, OS, true);
  case Mips::BC1T:
    // bc1t $fcc0, $L1 => bc1t $L1
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-314
```cpp
    return isReg<Mips::FCC0>(MI, 0) &&
           printAlias("bc1t", MI, Address, 1, STI, OS, true);
  case Mips::BC1F:
    // bc1f $fcc0, $L1 => bc1f $L1
    return isReg<Mips::FCC0>(MI, 0) &&
           printAlias("bc1f", MI, Address, 1, STI, OS, true);
  case Mips::JALR:
    // jalr $zero, $r1 => jr $r1
    // jalr $ra, $r1 => jalr $r1
    return (isReg<Mips::ZERO>(MI, 0) &&
            printAlias("jr", MI, Address, 1, STI, OS)) ||
           (isReg<Mips::RA>(MI, 0) &&
            printAlias("jalr", MI, Address, 1, STI, OS));
  case Mips::JALR64:
    // jalr $zero, $r1 => jr $r1
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 315-329
```cpp
    // jalr $ra, $r1 => jalr $r1
    return (isReg<Mips::ZERO_64>(MI, 0) &&
            printAlias("jr", MI, Address, 1, STI, OS)) ||
           (isReg<Mips::RA_64>(MI, 0) &&
            printAlias("jalr", MI, Address, 1, STI, OS));
  case Mips::NOR:
  case Mips::NOR_MM:
  case Mips::NOR_MMR6:
    // nor $r0, $r1, $zero => not $r0, $r1
    return isReg<Mips::ZERO>(MI, 2) &&
           printAlias("not", MI, Address, 0, 1, STI, OS);
  case Mips::NOR64:
    // nor $r0, $r1, $zero => not $r0, $r1
    return isReg<Mips::ZERO_64>(MI, 2) &&
           printAlias("not", MI, Address, 0, 1, STI, OS);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 330-339
```cpp
  case Mips::OR:
  case Mips::ADDu:
    // or $r0, $r1, $zero => move $r0, $r1
    // addu $r0, $r1, $zero => move $r0, $r1
    return isReg<Mips::ZERO>(MI, 2) &&
           printAlias("move", MI, Address, 0, 1, STI, OS);
  default:
    return false;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 341-351
```cpp
void MipsInstPrinter::printSaveRestore(const MCInst *MI,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
    if (i != 0) O << ", ";
    if (MI->getOperand(i).isReg())
      printRegName(O, MI->getOperand(i).getReg());
    else
      printUImm<16>(MI, i, STI, O);
  }
}
```
- EN: Implements `MipsInstPrinter::printSaveRestore`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printSaveRestore`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 353-363
```cpp
void MipsInstPrinter::printRegisterList(const MCInst *MI, int opNum,
                                        const MCSubtargetInfo & /* STI */,
                                        raw_ostream &O) {
  // - 2 because register List is always first operand of instruction and it is
  // always followed by memory operand (base + offset).
  for (int i = opNum, e = MI->getNumOperands() - 2; i != e; ++i) {
    if (i != opNum)
      O << ", ";
    printRegName(O, MI->getOperand(i).getReg());
  }
}
```
- EN: Implements `MipsInstPrinter::printRegisterList`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsInstPrinter::printRegisterList`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: instruction pretty-printing.
  - CN: 核心职责：指令格式化打印。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsInstPrinter.h`, `Mips.h`, `MipsGenAsmWriter.inc`.
  - CN: 后端本地头文件：`MipsInstPrinter.h`, `Mips.h`, `MipsGenAsmWriter.inc`。
- EN: LLVM infrastructure headers: `llvm/ADT/StringExtras.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/ErrorHandling.h` ... (+1 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringExtras.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/Support/ErrorHandling.h` ... (+1 more)。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
