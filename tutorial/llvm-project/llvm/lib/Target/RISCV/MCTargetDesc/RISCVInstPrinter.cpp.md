# RISCVInstPrinter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVInstPrinter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements printing LLVM MC instructions as RISC-V assembly syntax. / 实现将 LLVM MC 指令打印为 RISC-V 汇编语法。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVInstPrinter.cpp - Convert RISC-V MCInst to asm syntax --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This class prints an RISC-V MCInst to a .s file.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-25: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVInstPrinter.h"
#include "RISCVBaseInfo.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstPrinter.h"
#include "llvm/MC/MCInstrAnalysis.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
using namespace llvm;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 26-36: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
#define DEBUG_TYPE "asm-printer"

// Include the auto-generated portion of the assembly writer.
#define PRINT_ALIAS_INSTR
#include "RISCVGenAsmWriter.inc"

static cl::opt<bool>
    NoAliases("riscv-no-aliases",
              cl::desc("Disable the emission of assembler pseudo instructions"),
              cl::init(false), cl::Hidden);
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 37-46: Definitions and supporting logic / 定义与支撑逻辑
```cpp
static cl::opt<bool> EmitX8AsFP("riscv-emit-x8-as-fp",
                                cl::desc("Emit x8 as fp instead of s0"),
                                cl::init(false), cl::Hidden);

// Print architectural register names rather than the ABI names (such as x2
// instead of sp).
// TODO: Make RISCVInstPrinter::getRegisterName non-static so that this can a
// member.
static bool ArchRegNames;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 47-56: Commentary and design intent / 注释与设计意图
```cpp
// The command-line flags above are used by llvm-mc and llc. They can be used by
// `llvm-objdump`, but we override their values here to handle options passed to
// `llvm-objdump` with `-M` (which matches GNU objdump). There did not seem to
// be an easier way to allow these options in all these tools, without doing it
// this way.
bool RISCVInstPrinter::applyTargetSpecificCLOption(StringRef Opt) {
  if (Opt == "no-aliases") {
    PrintAliases = false;
    return true;
  }
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 57-66: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Opt == "numeric") {
    ArchRegNames = true;
    return true;
  }
  if (Opt == "emit-x8-as-fp") {
    if (!ArchRegNames)
      EmitX8AsFP = true;
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 67-83: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return false;
}

void RISCVInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                 StringRef Annot, const MCSubtargetInfo &STI,
                                 raw_ostream &O) {
  bool Res = false;
  const MCInst *NewMI = MI;
  MCInst UncompressedMI;
  if (PrintAliases && !NoAliases)
    Res = RISCVRVC::uncompress(UncompressedMI, *MI, STI);
  if (Res)
    NewMI = &UncompressedMI;
  if (!PrintAliases || NoAliases || !printAliasInstr(NewMI, Address, STI, O))
    printInstruction(NewMI, Address, STI, O);
  printAnnotation(O, Annot);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 84-93: Function implementation: RISCVInstPrinter::printRegName / 函数实现：RISCVInstPrinter::printRegName
```cpp

void RISCVInstPrinter::printRegName(raw_ostream &O, MCRegister Reg) {
  markup(O, Markup::Register) << getRegisterName(Reg);
}

void RISCVInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNo);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 94-103: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (MO.isReg()) {
    printRegName(O, MO.getReg());
    return;
  }

  if (MO.isImm()) {
    printImm(MI, OpNo, STI, O);
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 104-115: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  assert(MO.isExpr() && "Unknown operand kind in printOperand");
  MAI.printExpr(O, *MO.getExpr());
}

void RISCVInstPrinter::printBranchOperand(const MCInst *MI, uint64_t Address,
                                          unsigned OpNo,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  // Do not print the numeric target address when symbolizing.
  if (SymbolizeOperands)
    return;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 116-127: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  const MCOperand &MO = MI->getOperand(OpNo);
  if (!MO.isImm())
    return printOperand(MI, OpNo, STI, O);

  if (PrintBranchImmAsAddress) {
    uint64_t Target = Address + MO.getImm();
    if (!STI.hasFeature(RISCV::Feature64Bit))
      Target &= 0xffffffff;
    markup(O, Markup::Target) << formatHex(Target);
  } else {
    markup(O, Markup::Target) << formatImm(MO.getImm());
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 128-141: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

void RISCVInstPrinter::printCSRSystemRegister(const MCInst *MI, unsigned OpNo,
                                              const MCSubtargetInfo &STI,
                                              raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();
  auto Range = RISCVSysReg::lookupSysRegByEncoding(Imm);
  for (auto &Reg : Range) {
    if (Reg.IsAltName || Reg.IsDeprecatedName)
      continue;
    if (Reg.haveRequiredFeatures(STI.getFeatureBits())) {
      markup(O, Markup::Register) << Reg.Name;
      return;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 142-151: Function implementation: RISCVInstPrinter::printFenceArg / 函数实现：RISCVInstPrinter::printFenceArg
```cpp
  }
  markup(O, Markup::Register) << formatImm(Imm);
}

void RISCVInstPrinter::printFenceArg(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  unsigned FenceArg = MI->getOperand(OpNo).getImm();
  assert (((FenceArg >> 4) == 0) && "Invalid immediate in printFenceArg");
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 152-162: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  if ((FenceArg & RISCVFenceField::I) != 0)
    O << 'i';
  if ((FenceArg & RISCVFenceField::O) != 0)
    O << 'o';
  if ((FenceArg & RISCVFenceField::R) != 0)
    O << 'r';
  if ((FenceArg & RISCVFenceField::W) != 0)
    O << 'w';
  if (FenceArg == 0)
    O << "0";
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 163-172: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVInstPrinter::printFRMArg(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  auto FRMArg =
      static_cast<RISCVFPRndMode::RoundingMode>(MI->getOperand(OpNo).getImm());
  if (PrintAliases && !NoAliases && FRMArg == RISCVFPRndMode::RoundingMode::DYN)
    return;
  O << ", " << RISCVFPRndMode::roundingModeToString(FRMArg);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 173-184: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
void RISCVInstPrinter::printFRMArgLegacy(const MCInst *MI, unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  auto FRMArg =
      static_cast<RISCVFPRndMode::RoundingMode>(MI->getOperand(OpNo).getImm());
  // Never print rounding mode if it's the default 'rne'. This ensures the
  // output can still be parsed by older tools that erroneously failed to
  // accept a rounding mode.
  if (FRMArg == RISCVFPRndMode::RoundingMode::RNE)
    return;
  O << ", " << RISCVFPRndMode::roundingModeToString(FRMArg);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 185-204: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVInstPrinter::printFPImmOperand(const MCInst *MI, unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();
  if (Imm == 1) {
    markup(O, Markup::Immediate) << "min";
  } else if (Imm == 30) {
    markup(O, Markup::Immediate) << "inf";
  } else if (Imm == 31) {
    markup(O, Markup::Immediate) << "nan";
  } else {
    float FPVal = RISCVLoadFPImm::getFPImm(Imm);
    // If the value is an integer, print a .0 fraction. Otherwise, use %g to
    // which will not print trailing zeros and will use scientific notation
    // if it is shorter than printing as a decimal. The smallest value requires
    // 12 digits of precision including the decimal.
    if (FPVal == (int)(FPVal))
      markup(O, Markup::Immediate) << format("%.1f", FPVal);
    else
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 205-218: Function implementation: RISCVInstPrinter::printZeroOffsetMemOp / 函数实现：RISCVInstPrinter::printZeroOffsetMemOp
```cpp
      markup(O, Markup::Immediate) << format("%.12g", FPVal);
  }
}

void RISCVInstPrinter::printZeroOffsetMemOp(const MCInst *MI, unsigned OpNo,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNo);

  assert(MO.isReg() && "printZeroOffsetMemOp can only print register operands");
  O << "(";
  printRegName(O, MO.getReg());
  O << ")";
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 219-234: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

void RISCVInstPrinter::printVTypeI(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();
  // Print the raw immediate for reserved values: vlmul[2:0]=4, vsew[2:0]=0b1xx,
  // altfmt=1 without zvfbfa or zvfofp8min extension, or non-zero in bits 9 and
  // above.
  if (!RISCVVType::isValidVType(Imm) ||
      (RISCVVType::isAltFmt(Imm) &&
       !(STI.hasFeature(RISCV::FeatureStdExtZvfbfa) ||
         STI.hasFeature(RISCV::FeatureStdExtZvfofp8min) ||
         STI.hasFeature(RISCV::FeatureVendorXSfvfbfexp16e))) ||
      (Imm >> 9) != 0) {
    O << formatImm(Imm);
    return;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 235-251: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Print the text form.
  RISCVVType::printVType(Imm, O);
}

void RISCVInstPrinter::printXSfmmVType(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();
  assert(RISCVVType::isValidXSfmmVType(Imm));
  unsigned SEW = RISCVVType::getSEW(Imm);
  O << "e" << SEW;
  bool AltFmt = RISCVVType::isAltFmt(Imm);
  if (AltFmt)
    O << "alt";
  unsigned Widen = RISCVVType::getXSfmmWiden(Imm);
  O << ", w" << Widen;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 252-262: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// Print a Zcmp RList. If we are printing architectural register names rather
// than ABI register names, we need to print "{x1, x8-x9, x18-x27}" for all
// registers. Otherwise, we print "{ra, s0-s11}".
void RISCVInstPrinter::printRegList(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI, raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();

  assert(Imm >= RISCVZC::RLISTENCODE::RA &&
         Imm <= RISCVZC::RLISTENCODE::RA_S0_S11 && "Invalid Rlist");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 263-275: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  O << "{";
  printRegName(O, RISCV::X1);

  if (Imm >= RISCVZC::RLISTENCODE::RA_S0) {
    O << ", ";
    printRegName(O, RISCV::X8);
  }

  if (Imm >= RISCVZC::RLISTENCODE::RA_S0_S1) {
    O << '-';
    if (Imm == RISCVZC::RLISTENCODE::RA_S0_S1 || ArchRegNames)
      printRegName(O, RISCV::X9);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 276-293: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (Imm >= RISCVZC::RLISTENCODE::RA_S0_S2) {
    if (ArchRegNames)
      O << ", ";
    if (Imm == RISCVZC::RLISTENCODE::RA_S0_S2 || ArchRegNames)
      printRegName(O, RISCV::X18);
  }

  if (Imm >= RISCVZC::RLISTENCODE::RA_S0_S3) {
    if (ArchRegNames)
      O << '-';
    unsigned Offset = (Imm - RISCVZC::RLISTENCODE::RA_S0_S3);
    // Encodings for S3-S9 are contiguous. There is no encoding for S10, so we
    // must skip to S11(X27).
    if (Imm == RISCVZC::RLISTENCODE::RA_S0_S11)
      ++Offset;
    printRegName(O, RISCV::X19 + Offset);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 294-304: Function implementation: RISCVInstPrinter::printRegReg / 函数实现：RISCVInstPrinter::printRegReg
```cpp

  O << "}";
}

void RISCVInstPrinter::printRegReg(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &OffsetMO = MI->getOperand(OpNo + 1);

  assert(OffsetMO.isReg() && "printRegReg can only print register operands");
  printRegName(O, OffsetMO.getReg());
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 305-324: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  O << "(";
  const MCOperand &BaseMO = MI->getOperand(OpNo);
  assert(BaseMO.isReg() && "printRegReg can only print register operands");
  printRegName(O, BaseMO.getReg());
  O << ")";
}

void RISCVInstPrinter::printStackAdj(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI, raw_ostream &O,
                                     bool Negate) {
  int64_t Imm = MI->getOperand(OpNo).getImm();
  bool IsRV64 = STI.hasFeature(RISCV::Feature64Bit);
  int64_t StackAdj = 0;
  auto RlistVal = MI->getOperand(0).getImm();
  auto Base = RISCVZC::getStackAdjBase(RlistVal, IsRV64);
  StackAdj = Imm + Base;
  assert((StackAdj >= Base && StackAdj <= Base + 48) &&
         "Incorrect stack adjust");
  if (Negate)
    StackAdj = -StackAdj;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 325-335: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // RAII guard for ANSI color escape sequences
  WithMarkup ScopedMarkup = markup(O, Markup::Immediate);
  O << StackAdj;
}

void RISCVInstPrinter::printVMaskReg(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNo);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 336-348: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  assert(MO.isReg() && "printVMaskReg can only print register operands");
  if (MO.getReg() == RISCV::NoRegister)
    return;
  O << ", ";
  printRegName(O, MO.getReg());
  O << ".t";
}

void RISCVInstPrinter::printVScaleReg(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  const MCOperand &MO = MI->getOperand(OpNo);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 349-368: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  assert(MO.isReg() && "printVScaleReg can only print register operands");
  O << ", ";
  printRegName(O, MO.getReg());
  O << ".scale";
}

void RISCVInstPrinter::printImm(const MCInst *MI, unsigned OpNo,
                                const MCSubtargetInfo &STI, raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  const unsigned Opcode = MI->getOpcode();
  uint64_t Imm = Op.getImm();
  if (STI.getTargetTriple().isOSBinFormatMachO() &&
      (Opcode == RISCV::ANDI || Opcode == RISCV::ORI || Opcode == RISCV::XORI ||
       Opcode == RISCV::C_ANDI || Opcode == RISCV::AUIPC ||
       Opcode == RISCV::LUI)) {
    if (!STI.hasFeature(RISCV::Feature64Bit))
      Imm &= 0xffffffff;
    markup(O, Markup::Immediate) << formatHex(Imm);
  } else
    markup(O, Markup::Immediate) << formatImm(Imm);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 369-381: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
}

const char *RISCVInstPrinter::getRegisterName(MCRegister Reg) {
  // When PrintAliases is enabled, and EmitX8AsFP is enabled, x8 will be printed
  // as fp instead of s0. Note that these similar registers are not replaced:
  // - X8_H: used for f16 register in zhinx
  // - X8_W: used for f32 register in zfinx
  // - X8_X9: used for GPR Pair
  if (!ArchRegNames && EmitX8AsFP && Reg == RISCV::X8)
    return "fp";
  return getRegisterName(Reg, ArchRegNames ? RISCV::NoRegAltName
                                           : RISCV::ABIRegAltName);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**
- **Assembly printing** / **汇编打印**

## Dependencies / 依赖关系
- `RISCVInstPrinter.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCAsmInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCExpr.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCInst.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCInstPrinter.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCInstrAnalysis.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCSubtargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCSymbol.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/CommandLine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/ErrorHandling.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenAsmWriter.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
