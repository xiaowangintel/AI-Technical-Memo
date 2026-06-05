# AMDGPUInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/MCTargetDesc/AMDGPUInstPrinter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUInstPrinter for the LLVM MC target description layer. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM MC 目标描述层中的 AMDGPUInstPrinter 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-43: File banner, includes, and setup
```cpp
//===-- AMDGPUInstPrinter.cpp - AMDGPU MC Inst -> ASM ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
// \file
//===----------------------------------------------------------------------===//

#include "AMDGPUInstPrinter.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIDefines.h"
#include "Utils/AMDGPUAsmUtils.h"
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/TargetParser/TargetParser.h"

using namespace llvm;
using namespace llvm::AMDGPU;

void AMDGPUInstPrinter::printRegName(raw_ostream &OS, MCRegister Reg) {
  // FIXME: The current implementation of
  // AsmParser::parseRegisterOrRegisterNumber in MC implies we either emit this
  // as an integer or we provide a name which represents a physical register.
  // For CFI instructions we really want to emit a name for the DWARF register
  // instead, because there may be multiple DWARF registers corresponding to a
  // single physical register. One case where this problem manifests is with
  // wave32/wave64 where using the physical register name is ambiguous: if we
  // write e.g. `.cfi_undefined v0` we lose information about the wavefront
  // size which we need to encode the register in the final DWARF. Ideally we
  // would extend MC to support parsing DWARF register names so we could do
  // something like `.cfi_undefined dwarf_wave32_v0`. For now we just live with
  // non-pretty DWARF register names in assembly text.
  OS << Reg.id();
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUInstPrinter::printRegName`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUInstPrinter::printRegName`。

### Lines 44-85: Implements AMDGPUInstPrinter::printInst
```cpp
void AMDGPUInstPrinter::printInst(const MCInst *MI, uint64_t Address,
                                  StringRef Annot, const MCSubtargetInfo &STI,
                                  raw_ostream &OS) {
  printInstruction(MI, Address, STI, OS);
  printAnnotation(OS, Annot);
}

void AMDGPUInstPrinter::printU16ImmOperand(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isExpr()) {
    MAI.printExpr(O, *Op.getExpr());
    return;
  }

  // It's possible to end up with a 32-bit literal used with a 16-bit operand
  // with ignored high bits. Print as 32-bit anyway in that case.
  int64_t Imm = Op.getImm();
  if (isInt<16>(Imm) || isUInt<16>(Imm))
    O << formatHex(static_cast<uint64_t>(Imm & 0xffff));
  else
    printU32ImmOperand(MI, OpNo, STI, O);
}

void AMDGPUInstPrinter::printU16ImmDecOperand(const MCInst *MI, unsigned OpNo,
                                              raw_ostream &O) {
  O << formatDec(MI->getOperand(OpNo).getImm() & 0xffff);
}

void AMDGPUInstPrinter::printU32ImmOperand(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isExpr()) {
    MAI.printExpr(O, *Op.getExpr());
    return;
  }

  O << formatHex(Op.getImm() & 0xffffffff);
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printInst. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printInst`, `AMDGPUInstPrinter::printU16ImmOperand`, `AMDGPUInstPrinter::printU16ImmDecOperand`.
**CN:** 本节包含与 AMDGPUInstPrinter::printInst 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printInst`, `AMDGPUInstPrinter::printU16ImmOperand`, `AMDGPUInstPrinter::printU16ImmDecOperand`。

### Lines 86-129: Implements AMDGPUInstPrinter::printFP64ImmOperand
```cpp
void AMDGPUInstPrinter::printFP64ImmOperand(const MCInst *MI, unsigned OpNo,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  // KIMM64
  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isExpr()) {
    MAI.printExpr(O, *Op.getExpr());
    return;
  }

  printLiteral64(Op.getImm(), O, /*IsFP=*/true);
}

void AMDGPUInstPrinter::printNamedBit(const MCInst *MI, unsigned OpNo,
                                      raw_ostream &O, StringRef BitName) {
  if (MI->getOperand(OpNo).getImm()) {
    O << ' ' << BitName;
  }
}

void AMDGPUInstPrinter::printOffset(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  uint32_t Imm = MI->getOperand(OpNo).getImm();
  if (Imm != 0) {
    O << " offset:";

    // GFX12+ uses a 24-bit signed offset for VBUFFER.
    const MCInstrDesc &Desc = MII.get(MI->getOpcode());
    bool IsVBuffer = Desc.TSFlags & (SIInstrFlags::MUBUF | SIInstrFlags::MTBUF);
    if (IsVBuffer && AMDGPU::isGFX12Plus(STI))
      O << formatDec(SignExtend32<24>(Imm));
    else
      printU16ImmDecOperand(MI, OpNo, O);
  }
}

void AMDGPUInstPrinter::printFlatOffset(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  uint32_t Imm = MI->getOperand(OpNo).getImm();
  if (Imm != 0) {
    O << " offset:";

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printFP64ImmOperand. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printFP64ImmOperand`, `AMDGPUInstPrinter::printNamedBit`, `AMDGPUInstPrinter::printOffset`.
**CN:** 本节包含与 AMDGPUInstPrinter::printFP64ImmOperand 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printFP64ImmOperand`, `AMDGPUInstPrinter::printNamedBit`, `AMDGPUInstPrinter::printOffset`。

### Lines 130-173: Conditional logic and checks
```cpp
    const MCInstrDesc &Desc = MII.get(MI->getOpcode());
    bool AllowNegative = (Desc.TSFlags & (SIInstrFlags::FlatGlobal |
                                          SIInstrFlags::FlatScratch)) ||
                         STI.hasFeature(AMDGPU::FeatureFlatSignedOffset);

    if (AllowNegative) // Signed offset
      O << formatDec(SignExtend32(Imm, AMDGPU::getNumFlatOffsetBits(STI)));
    else // Unsigned offset
      printU16ImmDecOperand(MI, OpNo, O);
  }
}

void AMDGPUInstPrinter::printSMRDOffset8(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  printU32ImmOperand(MI, OpNo, STI, O);
}

void AMDGPUInstPrinter::printSMEMOffset(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  O << formatHex(MI->getOperand(OpNo).getImm());
}

void AMDGPUInstPrinter::printSMRDLiteralOffset(const MCInst *MI, unsigned OpNo,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  printU32ImmOperand(MI, OpNo, STI, O);
}

void AMDGPUInstPrinter::printCPol(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  auto Imm = MI->getOperand(OpNo).getImm();

  if (AMDGPU::isGFX12Plus(STI)) {
    const int64_t TH = Imm & CPol::TH;
    const int64_t Scope = Imm & CPol::SCOPE;

    if (Imm & CPol::SCAL)
      O << " scale_offset";

    printTH(MI, TH, Scope, O);
    printScope(Scope, O);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getNumFlatOffsetBits`, `AMDGPUInstPrinter::printSMRDOffset8`, `AMDGPUInstPrinter::printSMEMOffset`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getNumFlatOffsetBits`, `AMDGPUInstPrinter::printSMRDOffset8`, `AMDGPUInstPrinter::printSMEMOffset`。

### Lines 174-206: Conditional logic and checks
```cpp
    if (Imm & CPol::NV)
      O << " nv";

    return;
  }

  if (Imm & CPol::GLC)
    O << ((AMDGPU::isGFX940(STI) &&
           !(MII.get(MI->getOpcode()).TSFlags & SIInstrFlags::SMRD)) ? " sc0"
                                                                     : " glc");
  if (Imm & CPol::SLC)
    O << (AMDGPU::isGFX940(STI) ? " nt" : " slc");
  if ((Imm & CPol::DLC) && AMDGPU::isGFX10Plus(STI))
    O << " dlc";
  if ((Imm & CPol::SCC) && AMDGPU::isGFX90A(STI))
    O << (AMDGPU::isGFX940(STI) ? " sc1" : " scc");
  if (Imm & ~CPol::ALL_pregfx12)
    O << " /* unexpected cache policy bit */";
}

void AMDGPUInstPrinter::printTH(const MCInst *MI, int64_t TH, int64_t Scope,
                                raw_ostream &O) {
  // For th = 0 do not print this field
  if (TH == 0)
    return;

  const unsigned Opcode = MI->getOpcode();
  const MCInstrDesc &TID = MII.get(Opcode);
  unsigned THType = AMDGPU::getTemporalHintType(TID);
  bool IsStore = (THType == AMDGPU::CPol::TH_TYPE_STORE);

  O << " th:";

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isGFX940`, `AMDGPU::isGFX10Plus`, `AMDGPU::isGFX90A`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isGFX940`, `AMDGPU::isGFX10Plus`, `AMDGPU::isGFX90A`。

### Lines 207-250: Switch-based control flow
```cpp
  if (THType == AMDGPU::CPol::TH_TYPE_ATOMIC) {
    O << "TH_ATOMIC_";
    if (TH & AMDGPU::CPol::TH_ATOMIC_CASCADE) {
      if (Scope >= AMDGPU::CPol::SCOPE_DEV)
        O << "CASCADE" << (TH & AMDGPU::CPol::TH_ATOMIC_NT ? "_NT" : "_RT");
      else
        O << formatHex(TH);
    } else if (TH & AMDGPU::CPol::TH_ATOMIC_NT)
      O << "NT" << (TH & AMDGPU::CPol::TH_ATOMIC_RETURN ? "_RETURN" : "");
    else if (TH & AMDGPU::CPol::TH_ATOMIC_RETURN)
      O << "RETURN";
    else
      O << formatHex(TH);
  } else {
    if (!IsStore && TH == AMDGPU::CPol::TH_RESERVED)
      O << formatHex(TH);
    else {
      O << (IsStore ? "TH_STORE_" : "TH_LOAD_");
      switch (TH) {
      case AMDGPU::CPol::TH_NT:
        O << "NT";
        break;
      case AMDGPU::CPol::TH_HT:
        O << "HT";
        break;
      case AMDGPU::CPol::TH_BYPASS: // or LU or WB
        O << (Scope == AMDGPU::CPol::SCOPE_SYS ? "BYPASS"
                                               : (IsStore ? "WB" : "LU"));
        break;
      case AMDGPU::CPol::TH_NT_RT:
        O << "NT_RT";
        break;
      case AMDGPU::CPol::TH_RT_NT:
        O << "RT_NT";
        break;
      case AMDGPU::CPol::TH_NT_HT:
        O << "NT_HT";
        break;
      case AMDGPU::CPol::TH_NT_WB:
        O << "NT_WB";
        break;
      default:
        llvm_unreachable("unexpected th value");
      }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 251-290: Implements AMDGPUInstPrinter::printScope
```cpp
    }
  }
}

void AMDGPUInstPrinter::printScope(int64_t Scope, raw_ostream &O) {
  if (Scope == CPol::SCOPE_CU)
    return;

  O << " scope:";

  if (Scope == CPol::SCOPE_SE)
    O << "SCOPE_SE";
  else if (Scope == CPol::SCOPE_DEV)
    O << "SCOPE_DEV";
  else if (Scope == CPol::SCOPE_SYS)
    O << "SCOPE_SYS";
  else
    llvm_unreachable("unexpected scope policy value");
}

void AMDGPUInstPrinter::printDim(const MCInst *MI, unsigned OpNo,
                                 const MCSubtargetInfo &STI, raw_ostream &O) {
  unsigned Dim = MI->getOperand(OpNo).getImm();
  O << " dim:SQ_RSRC_IMG_";

  const AMDGPU::MIMGDimInfo *DimInfo = AMDGPU::getMIMGDimInfoByEncoding(Dim);
  if (DimInfo)
    O << DimInfo->AsmSuffix;
  else
    O << Dim;
}

void AMDGPUInstPrinter::printR128A16(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI, raw_ostream &O) {
  if (STI.hasFeature(AMDGPU::FeatureR128A16))
    printNamedBit(MI, OpNo, O, "a16");
  else
    printNamedBit(MI, OpNo, O, "r128");
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printScope. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printScope`, `AMDGPUInstPrinter::printDim`, `AMDGPU::getMIMGDimInfoByEncoding`.
**CN:** 本节包含与 AMDGPUInstPrinter::printScope 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printScope`, `AMDGPUInstPrinter::printDim`, `AMDGPU::getMIMGDimInfoByEncoding`。

### Lines 291-334: Implements AMDGPUInstPrinter::printFORMAT
```cpp
void AMDGPUInstPrinter::printFORMAT(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
}

void AMDGPUInstPrinter::printSymbolicFormat(const MCInst *MI,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  using namespace llvm::AMDGPU::MTBUFFormat;

  int OpNo =
    AMDGPU::getNamedOperandIdx(MI->getOpcode(), AMDGPU::OpName::format);
  assert(OpNo != -1);

  unsigned Val = MI->getOperand(OpNo).getImm();
  if (AMDGPU::isGFX10Plus(STI)) {
    if (Val == UFMT_DEFAULT)
      return;
    if (isValidUnifiedFormat(Val, STI)) {
      O << " format:[" << getUnifiedFormatName(Val, STI) << ']';
    } else {
      O << " format:" << Val;
    }
  } else {
    if (Val == DFMT_NFMT_DEFAULT)
      return;
    if (isValidDfmtNfmt(Val, STI)) {
      unsigned Dfmt;
      unsigned Nfmt;
      decodeDfmtNfmt(Val, Dfmt, Nfmt);
      O << " format:[";
      if (Dfmt != DFMT_DEFAULT) {
        O << getDfmtName(Dfmt);
        if (Nfmt != NFMT_DEFAULT) {
          O << ',';
        }
      }
      if (Nfmt != NFMT_DEFAULT) {
        O << getNfmtName(Nfmt, STI);
      }
      O << ']';
    } else {
      O << " format:" << Val;
    }
```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printFORMAT. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printFORMAT`, `AMDGPUInstPrinter::printSymbolicFormat`, `AMDGPU::getNamedOperandIdx`.
**CN:** 本节包含与 AMDGPUInstPrinter::printFORMAT 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printFORMAT`, `AMDGPUInstPrinter::printSymbolicFormat`, `AMDGPU::getNamedOperandIdx`。

### Lines 335-370: Defines getRegForPrinting
```cpp
  }
}

// \returns a low 256 vgpr representing a high vgpr \p Reg [v256..v1023] or
// \p Reg itself otherwise.
static MCRegister getRegForPrinting(MCRegister Reg, const MCRegisterInfo &MRI) {
  unsigned Enc = MRI.getEncodingValue(Reg);
  unsigned Idx = Enc & AMDGPU::HWEncoding::REG_IDX_MASK;
  if (Idx < 0x100)
    return Reg;

  unsigned RegNo = Idx % 0x100;
  const MCRegisterClass *RC = getVGPRPhysRegClass(Reg, MRI);
  if (RC->getID() == AMDGPU::VGPR_16RegClassID) {
    // This class has 2048 registers with interleaved lo16 and hi16.
    RegNo *= 2;
    if (Enc & AMDGPU::HWEncoding::IS_HI16)
      ++RegNo;
  }

  return RC->getRegister(RegNo);
}

// Restore MSBs of a VGPR above 255 from the MCInstrAnalysis.
static MCRegister getRegFromMIA(MCRegister Reg, unsigned OpNo,
                                const MCInstrDesc &Desc,
                                const MCRegisterInfo &MRI,
                                const AMDGPUMCInstrAnalysis &MIA) {
  unsigned VgprMSBs = MIA.getVgprMSBs();
  if (!VgprMSBs)
    return Reg;

  unsigned Enc = MRI.getEncodingValue(Reg);
  if (!(Enc & AMDGPU::HWEncoding::IS_VGPR))
    return Reg;

```
**EN:** This section contains concrete logic for getRegForPrinting. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getRegForPrinting 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 371-413: Preprocessor guards and macros
```cpp
  auto Ops = AMDGPU::getVGPRLoweringOperandTables(Desc);
  if (!Ops.first)
    return Reg;
  unsigned Opc = Desc.getOpcode();
  unsigned I;
  for (I = 0; I < 4; ++I) {
    if (Ops.first[I] != AMDGPU::OpName::NUM_OPERAND_NAMES &&
        (unsigned)AMDGPU::getNamedOperandIdx(Opc, Ops.first[I]) == OpNo)
      break;
    if (Ops.second && Ops.second[I] != AMDGPU::OpName::NUM_OPERAND_NAMES &&
        (unsigned)AMDGPU::getNamedOperandIdx(Opc, Ops.second[I]) == OpNo)
      break;
  }
  if (I == 4)
    return Reg;
  unsigned OpMSBs = (VgprMSBs >> (I * 2)) & 3;
  if (!OpMSBs)
    return Reg;
  if (MCRegister NewReg = AMDGPU::getVGPRWithMSBs(Reg, OpMSBs, MRI))
    return NewReg;
  return Reg;
}

void AMDGPUInstPrinter::printRegOperand(MCRegister Reg, raw_ostream &O,
                                        const MCRegisterInfo &MRI) {
#if !defined(NDEBUG)
  switch (Reg.id()) {
  case AMDGPU::FP_REG:
  case AMDGPU::SP_REG:
  case AMDGPU::PRIVATE_RSRC_REG:
    llvm_unreachable("pseudo-register should not ever be emitted");
  default:
    break;
  }
#endif

  MCRegister PrintReg = getRegForPrinting(Reg, MRI);
  O << getRegisterName(PrintReg);

  if (PrintReg != Reg)
    O << " /*" << getRegisterName(Reg) << "*/";
}

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `AMDGPU::getVGPRLoweringOperandTables`, `AMDGPU::getNamedOperandIdx`, `AMDGPU::getVGPRWithMSBs`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`AMDGPU::getVGPRLoweringOperandTables`, `AMDGPU::getNamedOperandIdx`, `AMDGPU::getVGPRWithMSBs`。

### Lines 414-448: Implements AMDGPUInstPrinter::printRegOperand
```cpp
void AMDGPUInstPrinter::printRegOperand(MCRegister Reg, unsigned Opc,
                                        unsigned OpNo, raw_ostream &O,
                                        const MCRegisterInfo &MRI) {
  if (MIA)
    Reg = getRegFromMIA(Reg, OpNo, MII.get(Opc), MRI,
                        *static_cast<const AMDGPUMCInstrAnalysis *>(MIA));
  printRegOperand(Reg, O, MRI);
}

void AMDGPUInstPrinter::printVOPDst(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI, raw_ostream &O) {
  auto Opcode = MI->getOpcode();
  auto Flags = MII.get(Opcode).TSFlags;
  if (OpNo == 0) {
    if (Flags & SIInstrFlags::VOP3 && Flags & SIInstrFlags::DPP)
      O << "_e64_dpp";
    else if (Flags & SIInstrFlags::VOP3) {
      if (!getVOP3IsSingle(Opcode))
        O << "_e64";
    } else if (Flags & SIInstrFlags::DPP)
      O << "_dpp";
    else if (Flags & SIInstrFlags::SDWA)
      O << "_sdwa";
    else if (((Flags & SIInstrFlags::VOP1) && !getVOP1IsSingle(Opcode)) ||
             ((Flags & SIInstrFlags::VOP2) && !getVOP2IsSingle(Opcode)))
      O << "_e32";
    O << " ";
  }

  printRegularOperand(MI, OpNo, STI, O);

  // Print default vcc/vcc_lo operand.
  switch (Opcode) {
  default: break;

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printRegOperand. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printRegOperand`, `AMDGPUInstPrinter::printVOPDst`.
**CN:** 本节包含与 AMDGPUInstPrinter::printRegOperand 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printRegOperand`, `AMDGPUInstPrinter::printVOPDst`。

### Lines 449-492: Implementation details and local logic
```cpp
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_sdwa_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_sdwa_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_sdwa_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx11:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx11:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx11:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx11:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx11:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx11:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx11:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx11:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx11:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx12:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx12:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx12:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx12:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx12:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx12:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx12:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx12:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx12:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx13:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx13:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx13:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx13:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx13:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx13:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx13:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx13:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx13:
    printDefaultVccOperand(false, STI, O);
    break;
  }
}

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 493-529: Implements AMDGPUInstPrinter::printVINTRPDst
```cpp
void AMDGPUInstPrinter::printVINTRPDst(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI, raw_ostream &O) {
  if (AMDGPU::isSI(STI) || AMDGPU::isCI(STI))
    O << " ";
  else
    O << "_e32 ";

  printRegularOperand(MI, OpNo, STI, O);
}

void AMDGPUInstPrinter::printAVLdSt32Align2RegOp(const MCInst *MI,
                                                 unsigned OpNo,
                                                 const MCSubtargetInfo &STI,
                                                 raw_ostream &O) {
  MCRegister Reg = MI->getOperand(OpNo).getReg();

  // On targets with an even alignment requirement
  if (MCRegister SubReg = MRI.getSubReg(Reg, AMDGPU::sub0))
    Reg = SubReg;
  printRegOperand(Reg, O, MRI);
}

void AMDGPUInstPrinter::printImmediateInt16(uint32_t Imm,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  int32_t SImm = static_cast<int32_t>(Imm);
  if (isInlinableIntLiteral(SImm)) {
    O << SImm;
    return;
  }

  if (printImmediateFloat32(Imm, STI, O))
    return;

  O << formatHex(static_cast<uint64_t>(Imm & 0xffff));
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printVINTRPDst. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printVINTRPDst`, `AMDGPU::isSI`, `AMDGPU::isCI`.
**CN:** 本节包含与 AMDGPUInstPrinter::printVINTRPDst 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printVINTRPDst`, `AMDGPU::isSI`, `AMDGPU::isCI`。

### Lines 530-555: Defines printImmediateFP16
```cpp
static bool printImmediateFP16(uint32_t Imm, const MCSubtargetInfo &STI,
                               raw_ostream &O) {
  if (Imm == 0x3C00)
    O << "1.0";
  else if (Imm == 0xBC00)
    O << "-1.0";
  else if (Imm == 0x3800)
    O << "0.5";
  else if (Imm == 0xB800)
    O << "-0.5";
  else if (Imm == 0x4000)
    O << "2.0";
  else if (Imm == 0xC000)
    O << "-2.0";
  else if (Imm == 0x4400)
    O << "4.0";
  else if (Imm == 0xC400)
    O << "-4.0";
  else if (Imm == 0x3118 && STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    O << "0.15915494";
  else
    return false;

  return true;
}

```
**EN:** This section contains concrete logic for printImmediateFP16. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 printImmediateFP16 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 556-596: Defines printImmediateBFloat16
```cpp
static bool printImmediateBFloat16(uint32_t Imm, const MCSubtargetInfo &STI,
                                   raw_ostream &O) {
  if (Imm == 0x3F80)
    O << "1.0";
  else if (Imm == 0xBF80)
    O << "-1.0";
  else if (Imm == 0x3F00)
    O << "0.5";
  else if (Imm == 0xBF00)
    O << "-0.5";
  else if (Imm == 0x4000)
    O << "2.0";
  else if (Imm == 0xC000)
    O << "-2.0";
  else if (Imm == 0x4080)
    O << "4.0";
  else if (Imm == 0xC080)
    O << "-4.0";
  else if (Imm == 0x3E22 && STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    O << "0.15915494";
  else
    return false;

  return true;
}

void AMDGPUInstPrinter::printImmediateBF16(uint32_t Imm,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  int16_t SImm = static_cast<int16_t>(Imm);
  if (isInlinableIntLiteral(SImm)) {
    O << SImm;
    return;
  }

  if (printImmediateBFloat16(static_cast<uint16_t>(Imm), STI, O))
    return;

  O << formatHex(static_cast<uint64_t>(Imm));
}

```
**EN:** This section contains concrete logic for printImmediateBFloat16. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printImmediateBF16`.
**CN:** 本节包含与 printImmediateBFloat16 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printImmediateBF16`。

### Lines 597-622: Implements AMDGPUInstPrinter::printImmediateF16
```cpp
void AMDGPUInstPrinter::printImmediateF16(uint32_t Imm,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  int16_t SImm = static_cast<int16_t>(Imm);
  if (isInlinableIntLiteral(SImm)) {
    O << SImm;
    return;
  }

  uint16_t HImm = static_cast<uint16_t>(Imm);
  if (printImmediateFP16(HImm, STI, O))
    return;

  uint64_t Imm16 = static_cast<uint16_t>(Imm);
  O << formatHex(Imm16);
}

void AMDGPUInstPrinter::printImmediateV216(uint32_t Imm, uint8_t OpType,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  int32_t SImm = static_cast<int32_t>(Imm);
  if (isInlinableIntLiteral(SImm)) {
    O << SImm;
    return;
  }

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printImmediateF16. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printImmediateF16`, `AMDGPUInstPrinter::printImmediateV216`.
**CN:** 本节包含与 AMDGPUInstPrinter::printImmediateF16 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printImmediateF16`, `AMDGPUInstPrinter::printImmediateV216`。

### Lines 623-665: Switch-based control flow
```cpp
  switch (OpType) {
  case AMDGPU::OPERAND_REG_IMM_V2INT16:
  case AMDGPU::OPERAND_REG_INLINE_C_V2INT16:
    if (printImmediateFloat32(Imm, STI, O))
      return;
    break;
  case AMDGPU::OPERAND_REG_IMM_V2FP16:
  case AMDGPU::OPERAND_REG_INLINE_C_V2FP16:
    if (isUInt<16>(Imm) &&
        printImmediateFP16(static_cast<uint16_t>(Imm), STI, O))
      return;
    break;
  case AMDGPU::OPERAND_REG_IMM_V2FP16_SPLAT: {
    if (AMDGPU::isGFX11Plus(STI)) {
      // For GFX11+, the inline constant is duplicated to both channels, so we
      // need to check if the low and high 16 bits are the same, and then if
      // they can be printed as inline constant values.
      uint16_t Lo16 = static_cast<uint16_t>(Imm & 0xFFFF);
      uint16_t Hi16 = static_cast<uint16_t>((Imm >> 16) & 0xFFFF);
      if (Lo16 == Hi16 &&
          printImmediateFP16(static_cast<uint16_t>(Imm), STI, O))
        return;
    } else {
      // For pre-GFX11, the inline constant is in the low 16 bits, so we need
      // to check if it can be printed as inline constant value.
      if (isUInt<16>(Imm) &&
          printImmediateFP16(static_cast<uint16_t>(Imm), STI, O))
        return;
    }
    break;
  }
  case AMDGPU::OPERAND_REG_IMM_V2BF16:
  case AMDGPU::OPERAND_REG_INLINE_C_V2BF16:
    if (isUInt<16>(Imm) &&
        printImmediateBFloat16(static_cast<uint16_t>(Imm), STI, O))
      return;
    break;
  case AMDGPU::OPERAND_REG_IMM_NOINLINE_V2FP16:
    break;
  default:
    llvm_unreachable("bad operand type");
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isGFX11Plus`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isGFX11Plus`。

### Lines 666-707: Declares formatHex
```cpp
  O << formatHex(static_cast<uint64_t>(Imm));
}

bool AMDGPUInstPrinter::printImmediateFloat32(uint32_t Imm,
                                              const MCSubtargetInfo &STI,
                                              raw_ostream &O) {
  if (Imm == llvm::bit_cast<uint32_t>(0.0f))
    O << "0.0";
  else if (Imm == llvm::bit_cast<uint32_t>(1.0f))
    O << "1.0";
  else if (Imm == llvm::bit_cast<uint32_t>(-1.0f))
    O << "-1.0";
  else if (Imm == llvm::bit_cast<uint32_t>(0.5f))
    O << "0.5";
  else if (Imm == llvm::bit_cast<uint32_t>(-0.5f))
    O << "-0.5";
  else if (Imm == llvm::bit_cast<uint32_t>(2.0f))
    O << "2.0";
  else if (Imm == llvm::bit_cast<uint32_t>(-2.0f))
    O << "-2.0";
  else if (Imm == llvm::bit_cast<uint32_t>(4.0f))
    O << "4.0";
  else if (Imm == llvm::bit_cast<uint32_t>(-4.0f))
    O << "-4.0";
  else if (Imm == 0x3e22f983 &&
           STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    O << "0.15915494";
  else
    return false;

  return true;
}

void AMDGPUInstPrinter::printImmediate32(uint32_t Imm,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  int32_t SImm = static_cast<int32_t>(Imm);
  if (isInlinableIntLiteral(SImm)) {
    O << SImm;
    return;
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUInstPrinter::printImmediateFloat32`, `AMDGPUInstPrinter::printImmediate32`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUInstPrinter::printImmediateFloat32`, `AMDGPUInstPrinter::printImmediate32`。

### Lines 708-747: Declares formatHex
```cpp
  if (printImmediateFloat32(Imm, STI, O))
    return;

  O << formatHex(static_cast<uint64_t>(Imm));
}

void AMDGPUInstPrinter::printImmediate64(uint64_t Imm,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O, bool IsFP) {
  int64_t SImm = static_cast<int64_t>(Imm);
  if (SImm >= -16 && SImm <= 64) {
    O << SImm;
    return;
  }

  if (Imm == llvm::bit_cast<uint64_t>(0.0))
    O << "0.0";
  else if (Imm == llvm::bit_cast<uint64_t>(1.0))
    O << "1.0";
  else if (Imm == llvm::bit_cast<uint64_t>(-1.0))
    O << "-1.0";
  else if (Imm == llvm::bit_cast<uint64_t>(0.5))
    O << "0.5";
  else if (Imm == llvm::bit_cast<uint64_t>(-0.5))
    O << "-0.5";
  else if (Imm == llvm::bit_cast<uint64_t>(2.0))
    O << "2.0";
  else if (Imm == llvm::bit_cast<uint64_t>(-2.0))
    O << "-2.0";
  else if (Imm == llvm::bit_cast<uint64_t>(4.0))
    O << "4.0";
  else if (Imm == llvm::bit_cast<uint64_t>(-4.0))
    O << "-4.0";
  else if (Imm == 0x3fc45f306dc9c882 &&
           STI.hasFeature(AMDGPU::FeatureInv2PiInlineImm))
    O << "0.15915494309189532";
  else
    printLiteral64(Imm, O, IsFP);
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUInstPrinter::printImmediate64`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUInstPrinter::printImmediate64`。

### Lines 748-790: Implements AMDGPUInstPrinter::printLiteral64
```cpp
void AMDGPUInstPrinter::printLiteral64(uint64_t Imm, raw_ostream &O,
                                       bool IsFP) {
  if (IsFP && Lo_32(Imm) == 0)
    O << formatHex(static_cast<uint64_t>(Hi_32(Imm)));
  else
    O << formatHex(Imm);
}

void AMDGPUInstPrinter::printBLGP(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI,
                                  raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();
  if (!Imm)
    return;

  if (AMDGPU::isGFX940(STI)) {
    switch (MI->getOpcode()) {
    case AMDGPU::V_MFMA_F64_16X16X4F64_gfx940_acd:
    case AMDGPU::V_MFMA_F64_16X16X4F64_gfx940_vcd:
    case AMDGPU::V_MFMA_F64_4X4X4F64_gfx940_acd:
    case AMDGPU::V_MFMA_F64_4X4X4F64_gfx940_vcd:
      O << " neg:[" << (Imm & 1) << ',' << ((Imm >> 1) & 1) << ','
        << ((Imm >> 2) & 1) << ']';
      return;
    }
  }

  O << " blgp:" << Imm;
}

void AMDGPUInstPrinter::printDefaultVccOperand(bool FirstOperand,
                                               const MCSubtargetInfo &STI,
                                               raw_ostream &O) {
  if (!FirstOperand)
    O << ", ";
  printRegOperand(STI.hasFeature(AMDGPU::FeatureWavefrontSize32)
                      ? AMDGPU::VCC_LO
                      : AMDGPU::VCC,
                  O, MRI);
  if (FirstOperand)
    O << ", ";
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printLiteral64. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printLiteral64`, `AMDGPUInstPrinter::printBLGP`, `AMDGPU::isGFX940`.
**CN:** 本节包含与 AMDGPUInstPrinter::printLiteral64 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printLiteral64`, `AMDGPUInstPrinter::printBLGP`, `AMDGPU::isGFX940`。

### Lines 791-834: Implements AMDGPUInstPrinter::needsImpliedVcc
```cpp
bool AMDGPUInstPrinter::needsImpliedVcc(const MCInstrDesc &Desc,
                                        unsigned OpNo) const {
  return OpNo == 0 && (Desc.TSFlags & SIInstrFlags::DPP) &&
         (Desc.TSFlags & SIInstrFlags::VOPC) &&
         !isVOPCAsmOnly(Desc.getOpcode()) &&
         (Desc.hasImplicitDefOfPhysReg(AMDGPU::VCC) ||
          Desc.hasImplicitDefOfPhysReg(AMDGPU::VCC_LO));
}

// Print default vcc/vcc_lo operand of VOPC.
void AMDGPUInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  unsigned Opc = MI->getOpcode();
  const MCInstrDesc &Desc = MII.get(Opc);
  int ModIdx = AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::src0_modifiers);
  // 0, 1 and 2 are the first printed operands in different cases
  // If there are printed modifiers, printOperandAndFPInputMods or
  // printOperandAndIntInputMods will be called instead
  if ((OpNo == 0 ||
       (OpNo == 1 && (Desc.TSFlags & SIInstrFlags::DPP) && ModIdx != -1)) &&
      (Desc.TSFlags & SIInstrFlags::VOPC) && !isVOPCAsmOnly(Desc.getOpcode()) &&
      (Desc.hasImplicitDefOfPhysReg(AMDGPU::VCC) ||
       Desc.hasImplicitDefOfPhysReg(AMDGPU::VCC_LO)))
    printDefaultVccOperand(true, STI, O);

  printRegularOperand(MI, OpNo, STI, O);
}

// Print operands after vcc or modifier handling.
void AMDGPUInstPrinter::printRegularOperand(const MCInst *MI, unsigned OpNo,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O) {
  const MCInstrDesc &Desc = MII.get(MI->getOpcode());

  if (OpNo >= MI->getNumOperands()) {
    O << "/*Missing OP" << OpNo << "*/";
    return;
  }

  const MCOperand &Op = MI->getOperand(OpNo);
  if (Op.isReg()) {
    printRegOperand(Op.getReg(), MI->getOpcode(), OpNo, O, MRI);

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::needsImpliedVcc. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::needsImpliedVcc`, `AMDGPUInstPrinter::printOperand`, `AMDGPU::getNamedOperandIdx`.
**CN:** 本节包含与 AMDGPUInstPrinter::needsImpliedVcc 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::needsImpliedVcc`, `AMDGPUInstPrinter::printOperand`, `AMDGPU::getNamedOperandIdx`。

### Lines 835-878: Switch-based control flow
```cpp
    // Check if operand register class contains register used.
    // Intention: print disassembler message when invalid code is decoded,
    // for example sgpr register used in VReg or VISrc(VReg or imm) operand.
    const MCOperandInfo &OpInfo = Desc.operands()[OpNo];
    if (OpInfo.RegClass != -1) {
      int16_t RCID = MII.getOpRegClassID(
          OpInfo, STI.getHwMode(MCSubtargetInfo::HwMode_RegInfo));
      const MCRegisterClass &RC = MRI.getRegClass(RCID);
      auto Reg = mc2PseudoReg(Op.getReg());
      if (!RC.contains(Reg) && !isInlineValue(Reg)) {
        bool IsWaveSizeOp = OpInfo.isLookupRegClassByHwMode() &&
                            (OpInfo.RegClass == AMDGPU::SReg_1 ||
                             OpInfo.RegClass == AMDGPU::SReg_1_XEXEC);
        // Suppress this comment for a mismatched wavesize. Some users expect to
        // be able to assemble and disassemble modules with mixed wavesizes, but
        // we do not know the subtarget in different functions in MC.
        //
        // TODO: Should probably print it anyway, maybe a more specific version.
        if (!IsWaveSizeOp) {
          O << "/*Invalid register, operand has \'" << MRI.getRegClassName(&RC)
            << "\' register class*/";
        }
      }
    }
  } else if (Op.isImm()) {
    const uint8_t OpTy = Desc.operands()[OpNo].OperandType;
    switch (OpTy) {
    case AMDGPU::OPERAND_REG_IMM_INT32:
    case AMDGPU::OPERAND_REG_IMM_FP32:
    case AMDGPU::OPERAND_REG_INLINE_C_INT32:
    case AMDGPU::OPERAND_REG_INLINE_C_FP32:
    case AMDGPU::OPERAND_REG_INLINE_AC_INT32:
    case AMDGPU::OPERAND_REG_INLINE_AC_FP32:
    case AMDGPU::OPERAND_REG_IMM_V2INT32:
    case AMDGPU::OPERAND_REG_IMM_V2FP32:
    case MCOI::OPERAND_IMMEDIATE:
    case AMDGPU::OPERAND_INLINE_SPLIT_BARRIER_INT32:
      printImmediate32(Op.getImm(), STI, O);
      break;
    case AMDGPU::OPERAND_REG_IMM_INT64:
    case AMDGPU::OPERAND_REG_INLINE_C_INT64:
      printImmediate64(Op.getImm(), STI, O, false);
      break;
    case AMDGPU::OPERAND_REG_IMM_FP64:
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 879-922: Conditional logic and checks
```cpp
    case AMDGPU::OPERAND_REG_INLINE_C_FP64:
    case AMDGPU::OPERAND_REG_INLINE_AC_FP64:
      printImmediate64(Op.getImm(), STI, O, true);
      break;
    case AMDGPU::OPERAND_REG_INLINE_C_INT16:
    case AMDGPU::OPERAND_REG_IMM_INT16:
      printImmediateInt16(Op.getImm(), STI, O);
      break;
    case AMDGPU::OPERAND_REG_INLINE_C_FP16:
    case AMDGPU::OPERAND_REG_IMM_FP16:
      printImmediateF16(Op.getImm(), STI, O);
      break;
    case AMDGPU::OPERAND_REG_INLINE_C_BF16:
    case AMDGPU::OPERAND_REG_IMM_BF16:
      printImmediateBF16(Op.getImm(), STI, O);
      break;
    case AMDGPU::OPERAND_REG_IMM_V2INT16:
    case AMDGPU::OPERAND_REG_IMM_V2BF16:
    case AMDGPU::OPERAND_REG_IMM_V2FP16:
    case AMDGPU::OPERAND_REG_IMM_V2FP16_SPLAT:
    case AMDGPU::OPERAND_REG_IMM_NOINLINE_V2FP16:
    case AMDGPU::OPERAND_REG_INLINE_C_V2INT16:
    case AMDGPU::OPERAND_REG_INLINE_C_V2BF16:
    case AMDGPU::OPERAND_REG_INLINE_C_V2FP16:
      printImmediateV216(Op.getImm(), OpTy, STI, O);
      break;
    case MCOI::OPERAND_UNKNOWN:
    case MCOI::OPERAND_PCREL:
      O << formatDec(Op.getImm());
      break;
    case MCOI::OPERAND_REGISTER:
      // Disassembler does not fail when operand should not allow immediate
      // operands but decodes them into 32bit immediate operand.
      printImmediate32(Op.getImm(), STI, O);
      O << "/*Invalid immediate*/";
      break;
    default:
      // We hit this for the immediate instruction bits that don't yet have a
      // custom printer.
      llvm_unreachable("unexpected immediate operand type");
    }
  } else if (Op.isExpr()) {
    const MCExpr *Exp = Op.getExpr();
    MAI.printExpr(O, *Exp);
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 923-966: Switch-based control flow
```cpp
  } else {
    O << "/*INV_OP*/";
  }

  // Print default vcc/vcc_lo operand of v_cndmask_b32_e32.
  switch (MI->getOpcode()) {
  default: break;

  case AMDGPU::V_CNDMASK_B32_e32_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx10:
  case AMDGPU::V_CNDMASK_B32_dpp8_gfx10:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx10:
  case AMDGPU::V_CNDMASK_B32_e32_gfx11:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx11:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx11:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx11:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx11:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx11:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx11:
  case AMDGPU::V_CNDMASK_B32_dpp8_gfx11:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx11:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx11:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx11:
  case AMDGPU::V_CNDMASK_B32_e32_gfx12:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx12:
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx12:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx12:
  case AMDGPU::V_CNDMASK_B32_dpp_gfx12:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx12:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx12:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx12:
  case AMDGPU::V_CNDMASK_B32_dpp8_gfx12:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx12:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx12:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx12:
  case AMDGPU::V_CNDMASK_B32_e32_gfx13:
  case AMDGPU::V_ADD_CO_CI_U32_e32_gfx13:
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 967-1009: Conditional logic and checks
```cpp
  case AMDGPU::V_SUB_CO_CI_U32_e32_gfx13:
  case AMDGPU::V_SUBREV_CO_CI_U32_e32_gfx13:
  case AMDGPU::V_CNDMASK_B32_dpp_gfx13:
  case AMDGPU::V_ADD_CO_CI_U32_dpp_gfx13:
  case AMDGPU::V_SUB_CO_CI_U32_dpp_gfx13:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp_gfx13:
  case AMDGPU::V_CNDMASK_B32_dpp8_gfx13:
  case AMDGPU::V_ADD_CO_CI_U32_dpp8_gfx13:
  case AMDGPU::V_SUB_CO_CI_U32_dpp8_gfx13:
  case AMDGPU::V_SUBREV_CO_CI_U32_dpp8_gfx13:

  case AMDGPU::V_CNDMASK_B32_e32_gfx6_gfx7:
  case AMDGPU::V_CNDMASK_B32_e32_vi:
    if ((int)OpNo == AMDGPU::getNamedOperandIdx(MI->getOpcode(),
                                                AMDGPU::OpName::src1))
      printDefaultVccOperand(OpNo == 0, STI, O);
    break;
  }

  if (Desc.TSFlags & SIInstrFlags::MTBUF) {
    int SOffsetIdx =
      AMDGPU::getNamedOperandIdx(MI->getOpcode(), AMDGPU::OpName::soffset);
    assert(SOffsetIdx != -1);
    if ((int)OpNo == SOffsetIdx)
      printSymbolicFormat(MI, STI, O);
  }
}

void AMDGPUInstPrinter::printOperandAndFPInputMods(const MCInst *MI,
                                                   unsigned OpNo,
                                                   const MCSubtargetInfo &STI,
                                                   raw_ostream &O) {
  const MCInstrDesc &Desc = MII.get(MI->getOpcode());
  if (needsImpliedVcc(Desc, OpNo))
    printDefaultVccOperand(true, STI, O);

  unsigned InputModifiers = MI->getOperand(OpNo).getImm();

  // Use 'neg(...)' instead of '-' to avoid ambiguity.
  // This is important for integer literals because
  // -1 is not the same value as neg(1).
  bool NegMnemo = false;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getNamedOperandIdx`, `AMDGPUInstPrinter::printOperandAndFPInputMods`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getNamedOperandIdx`, `AMDGPUInstPrinter::printOperandAndFPInputMods`。

### Lines 1010-1047: Switch-based control flow
```cpp
  if (InputModifiers & SISrcMods::NEG) {
    if (OpNo + 1 < MI->getNumOperands() &&
        (InputModifiers & SISrcMods::ABS) == 0) {
      const MCOperand &Op = MI->getOperand(OpNo + 1);
      NegMnemo = Op.isImm();
    }
    if (NegMnemo) {
      O << "neg(";
    } else {
      O << '-';
    }
  }

  if (InputModifiers & SISrcMods::ABS)
    O << '|';
  printRegularOperand(MI, OpNo + 1, STI, O);
  if (InputModifiers & SISrcMods::ABS)
    O << '|';

  if (NegMnemo) {
    O << ')';
  }

  // Print default vcc/vcc_lo operand of VOP2b.
  switch (MI->getOpcode()) {
  default:
    break;

  case AMDGPU::V_CNDMASK_B32_sdwa_gfx10:
  case AMDGPU::V_CNDMASK_B32_dpp_gfx10:
  case AMDGPU::V_CNDMASK_B32_dpp_gfx11:
    if ((int)OpNo + 1 ==
        AMDGPU::getNamedOperandIdx(MI->getOpcode(), AMDGPU::OpName::src1))
      printDefaultVccOperand(OpNo == 0, STI, O);
    break;
  }
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getNamedOperandIdx`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getNamedOperandIdx`。

### Lines 1048-1090: Implements AMDGPUInstPrinter::printOperandAndIntInputMods
```cpp
void AMDGPUInstPrinter::printOperandAndIntInputMods(const MCInst *MI,
                                                    unsigned OpNo,
                                                    const MCSubtargetInfo &STI,
                                                    raw_ostream &O) {
  const MCInstrDesc &Desc = MII.get(MI->getOpcode());
  if (needsImpliedVcc(Desc, OpNo))
    printDefaultVccOperand(true, STI, O);

  unsigned InputModifiers = MI->getOperand(OpNo).getImm();
  if (InputModifiers & SISrcMods::SEXT)
    O << "sext(";
  printRegularOperand(MI, OpNo + 1, STI, O);
  if (InputModifiers & SISrcMods::SEXT)
    O << ')';

  // Print default vcc/vcc_lo operand of VOP2b.
  switch (MI->getOpcode()) {
  default: break;

  case AMDGPU::V_ADD_CO_CI_U32_sdwa_gfx10:
  case AMDGPU::V_SUB_CO_CI_U32_sdwa_gfx10:
  case AMDGPU::V_SUBREV_CO_CI_U32_sdwa_gfx10:
    if ((int)OpNo + 1 == AMDGPU::getNamedOperandIdx(MI->getOpcode(),
                                                    AMDGPU::OpName::src1))
      printDefaultVccOperand(OpNo == 0, STI, O);
    break;
  }
}

void AMDGPUInstPrinter::printDPP8(const MCInst *MI, unsigned OpNo,
                                  const MCSubtargetInfo &STI,
                                  raw_ostream &O) {
  if (!AMDGPU::isGFX10Plus(STI))
    llvm_unreachable("dpp8 is not supported on ASICs earlier than GFX10");

  unsigned Imm = MI->getOperand(OpNo).getImm();
  O << "dpp8:[" << formatDec(Imm & 0x7);
  for (size_t i = 1; i < 8; ++i) {
    O << ',' << formatDec((Imm >> (3 * i)) & 0x7);
  }
  O << ']';
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printOperandAndIntInputMods. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printOperandAndIntInputMods`, `AMDGPU::getNamedOperandIdx`, `AMDGPUInstPrinter::printDPP8`.
**CN:** 本节包含与 AMDGPUInstPrinter::printOperandAndIntInputMods 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printOperandAndIntInputMods`, `AMDGPU::getNamedOperandIdx`, `AMDGPUInstPrinter::printDPP8`。

### Lines 1091-1134: Implements AMDGPUInstPrinter::printDPPCtrl
```cpp
void AMDGPUInstPrinter::printDPPCtrl(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  using namespace AMDGPU::DPP;

  unsigned Imm = MI->getOperand(OpNo).getImm();
  const MCInstrDesc &Desc = MII.get(MI->getOpcode());

  if (!AMDGPU::isLegalDPALU_DPPControl(STI, Imm) &&
      AMDGPU::isDPALU_DPP(Desc, MII, STI)) {
    O << " /* DP ALU dpp only supports "
      << (isGFX12(STI) ? "row_share" : "row_newbcast") << " */";
    return;
  }
  if (Imm <= DppCtrl::QUAD_PERM_LAST) {
    O << "quad_perm:[";
    O << formatDec(Imm & 0x3)         << ',';
    O << formatDec((Imm & 0xc)  >> 2) << ',';
    O << formatDec((Imm & 0x30) >> 4) << ',';
    O << formatDec((Imm & 0xc0) >> 6) << ']';
  } else if ((Imm >= DppCtrl::ROW_SHL_FIRST) &&
             (Imm <= DppCtrl::ROW_SHL_LAST)) {
    O << "row_shl:" << formatDec(Imm - DppCtrl::ROW_SHL0);
  } else if ((Imm >= DppCtrl::ROW_SHR_FIRST) &&
             (Imm <= DppCtrl::ROW_SHR_LAST)) {
    O << "row_shr:" << formatDec(Imm - DppCtrl::ROW_SHR0);
  } else if ((Imm >= DppCtrl::ROW_ROR_FIRST) &&
             (Imm <= DppCtrl::ROW_ROR_LAST)) {
    O << "row_ror:" << formatDec(Imm - DppCtrl::ROW_ROR0);
  } else if (Imm == DppCtrl::WAVE_SHL1) {
    if (AMDGPU::isGFX10Plus(STI)) {
      O << "/* wave_shl is not supported starting from GFX10 */";
      return;
    }
    O << "wave_shl:1";
  } else if (Imm == DppCtrl::WAVE_ROL1) {
    if (AMDGPU::isGFX10Plus(STI)) {
      O << "/* wave_rol is not supported starting from GFX10 */";
      return;
    }
    O << "wave_rol:1";
  } else if (Imm == DppCtrl::WAVE_SHR1) {
    if (AMDGPU::isGFX10Plus(STI)) {
      O << "/* wave_shr is not supported starting from GFX10 */";
```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printDPPCtrl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printDPPCtrl`, `AMDGPU::isLegalDPALU_DPPControl`, `AMDGPU::isDPALU_DPP`.
**CN:** 本节包含与 AMDGPUInstPrinter::printDPPCtrl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printDPPCtrl`, `AMDGPU::isLegalDPALU_DPPControl`, `AMDGPU::isDPALU_DPP`。

### Lines 1135-1178: Conditional logic and checks
```cpp
      return;
    }
    O << "wave_shr:1";
  } else if (Imm == DppCtrl::WAVE_ROR1) {
    if (AMDGPU::isGFX10Plus(STI)) {
      O << "/* wave_ror is not supported starting from GFX10 */";
      return;
    }
    O << "wave_ror:1";
  } else if (Imm == DppCtrl::ROW_MIRROR) {
    O << "row_mirror";
  } else if (Imm == DppCtrl::ROW_HALF_MIRROR) {
    O << "row_half_mirror";
  } else if (Imm == DppCtrl::BCAST15) {
    if (AMDGPU::isGFX10Plus(STI)) {
      O << "/* row_bcast is not supported starting from GFX10 */";
      return;
    }
    O << "row_bcast:15";
  } else if (Imm == DppCtrl::BCAST31) {
    if (AMDGPU::isGFX10Plus(STI)) {
      O << "/* row_bcast is not supported starting from GFX10 */";
      return;
    }
    O << "row_bcast:31";
  } else if ((Imm >= DppCtrl::ROW_SHARE_FIRST) &&
             (Imm <= DppCtrl::ROW_SHARE_LAST)) {
    if (AMDGPU::isGFX90A(STI)) {
      O << "row_newbcast:";
    } else if (AMDGPU::isGFX10Plus(STI)) {
      O << "row_share:";
    } else {
      O << " /* row_newbcast/row_share is not supported on ASICs earlier "
           "than GFX90A/GFX10 */";
      return;
    }
    O << formatDec(Imm - DppCtrl::ROW_SHARE_FIRST);
  } else if ((Imm >= DppCtrl::ROW_XMASK_FIRST) &&
             (Imm <= DppCtrl::ROW_XMASK_LAST)) {
    if (!AMDGPU::isGFX10Plus(STI)) {
      O << "/* row_xmask is not supported on ASICs earlier than GFX10 */";
      return;
    }
    O << "row_xmask:" << formatDec(Imm - DppCtrl::ROW_XMASK_FIRST);
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isGFX10Plus`, `AMDGPU::isGFX90A`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isGFX10Plus`, `AMDGPU::isGFX90A`。

### Lines 1179-1218: Implements AMDGPUInstPrinter::printDppBoundCtrl
```cpp
  } else {
    O << "/* Invalid dpp_ctrl value */";
  }
}

void AMDGPUInstPrinter::printDppBoundCtrl(const MCInst *MI, unsigned OpNo,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNo).getImm();
  if (Imm) {
    O << " bound_ctrl:1";
  }
}

void AMDGPUInstPrinter::printDppFI(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  using namespace llvm::AMDGPU::DPP;
  unsigned Imm = MI->getOperand(OpNo).getImm();
  if (Imm == DPP_FI_1 || Imm == DPP8_FI_1) {
    O << " fi:1";
  }
}

void AMDGPUInstPrinter::printSDWASel(const MCInst *MI, unsigned OpNo,
                                     raw_ostream &O) {
  using namespace llvm::AMDGPU::SDWA;

  unsigned Imm = MI->getOperand(OpNo).getImm();
  switch (Imm) {
  case SdwaSel::BYTE_0: O << "BYTE_0"; break;
  case SdwaSel::BYTE_1: O << "BYTE_1"; break;
  case SdwaSel::BYTE_2: O << "BYTE_2"; break;
  case SdwaSel::BYTE_3: O << "BYTE_3"; break;
  case SdwaSel::WORD_0: O << "WORD_0"; break;
  case SdwaSel::WORD_1: O << "WORD_1"; break;
  case SdwaSel::DWORD: O << "DWORD"; break;
  default: llvm_unreachable("Invalid SDWA data select operand");
  }
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printDppBoundCtrl. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printDppBoundCtrl`, `AMDGPUInstPrinter::printDppFI`, `AMDGPUInstPrinter::printSDWASel`.
**CN:** 本节包含与 AMDGPUInstPrinter::printDppBoundCtrl 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printDppBoundCtrl`, `AMDGPUInstPrinter::printDppFI`, `AMDGPUInstPrinter::printSDWASel`。

### Lines 1219-1261: Implements AMDGPUInstPrinter::printSDWADstSel
```cpp
void AMDGPUInstPrinter::printSDWADstSel(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  O << "dst_sel:";
  printSDWASel(MI, OpNo, O);
}

void AMDGPUInstPrinter::printSDWASrc0Sel(const MCInst *MI, unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  O << "src0_sel:";
  printSDWASel(MI, OpNo, O);
}

void AMDGPUInstPrinter::printSDWASrc1Sel(const MCInst *MI, unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O) {
  O << "src1_sel:";
  printSDWASel(MI, OpNo, O);
}

void AMDGPUInstPrinter::printSDWADstUnused(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  using namespace llvm::AMDGPU::SDWA;

  O << "dst_unused:";
  unsigned Imm = MI->getOperand(OpNo).getImm();
  switch (Imm) {
  case DstUnused::UNUSED_PAD: O << "UNUSED_PAD"; break;
  case DstUnused::UNUSED_SEXT: O << "UNUSED_SEXT"; break;
  case DstUnused::UNUSED_PRESERVE: O << "UNUSED_PRESERVE"; break;
  default: llvm_unreachable("Invalid SDWA dest_unused operand");
  }
}

void AMDGPUInstPrinter::printExpSrcN(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI, raw_ostream &O,
                                     unsigned N) {
  unsigned Opc = MI->getOpcode();
  int EnIdx = AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::en);
  unsigned En = MI->getOperand(EnIdx).getImm();

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printSDWADstSel. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printSDWADstSel`, `AMDGPUInstPrinter::printSDWASrc0Sel`, `AMDGPUInstPrinter::printSDWASrc1Sel`.
**CN:** 本节包含与 AMDGPUInstPrinter::printSDWADstSel 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printSDWADstSel`, `AMDGPUInstPrinter::printSDWASrc0Sel`, `AMDGPUInstPrinter::printSDWASrc1Sel`。

### Lines 1262-1305: Implements AMDGPU::getNamedOperandIdx
```cpp
  int ComprIdx = AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::compr);

  // If compr is set, print as src0, src0, src1, src1
  if (MI->getOperand(ComprIdx).getImm())
    OpNo = OpNo - N + N / 2;

  if (En & (1 << N))
    printRegOperand(MI->getOperand(OpNo).getReg(), Opc, OpNo, O, MRI);
  else
    O << "off";
}

void AMDGPUInstPrinter::printExpSrc0(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  printExpSrcN(MI, OpNo, STI, O, 0);
}

void AMDGPUInstPrinter::printExpSrc1(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  printExpSrcN(MI, OpNo, STI, O, 1);
}

void AMDGPUInstPrinter::printExpSrc2(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  printExpSrcN(MI, OpNo, STI, O, 2);
}

void AMDGPUInstPrinter::printExpSrc3(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  printExpSrcN(MI, OpNo, STI, O, 3);
}

void AMDGPUInstPrinter::printExpTgt(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  using namespace llvm::AMDGPU::Exp;

  // This is really a 6 bit field.
  unsigned Id = MI->getOperand(OpNo).getImm() & ((1 << 6) - 1);

```
**EN:** This section contains concrete logic for AMDGPU::getNamedOperandIdx. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getNamedOperandIdx`, `AMDGPUInstPrinter::printExpSrc0`, `AMDGPUInstPrinter::printExpSrc1`.
**CN:** 本节包含与 AMDGPU::getNamedOperandIdx 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getNamedOperandIdx`, `AMDGPUInstPrinter::printExpSrc0`, `AMDGPUInstPrinter::printExpSrc1`。

### Lines 1306-1349: Defines allOpsDefaultValue
```cpp
  int Index;
  StringRef TgtName;
  if (getTgtName(Id, TgtName, Index) && isSupportedTgtId(Id, STI)) {
    O << ' ' << TgtName;
    if (Index >= 0)
      O << Index;
  } else {
    O << " invalid_target_" << Id;
  }
}

static bool allOpsDefaultValue(const int* Ops, int NumOps, int Mod,
                               bool IsPacked, bool HasDstSel) {
  int DefaultValue = IsPacked && (Mod == SISrcMods::OP_SEL_1);

  for (int I = 0; I < NumOps; ++I) {
    if (!!(Ops[I] & Mod) != DefaultValue)
      return false;
  }

  if (HasDstSel && (Ops[0] & SISrcMods::DST_OP_SEL) != 0)
    return false;

  return true;
}

void AMDGPUInstPrinter::printPackedModifier(const MCInst *MI,
                                            StringRef Name,
                                            unsigned Mod,
                                            raw_ostream &O) {
  unsigned Opc = MI->getOpcode();
  int NumOps = 0;
  int Ops[3];

  std::pair<AMDGPU::OpName, AMDGPU::OpName> MOps[] = {
      {AMDGPU::OpName::src0_modifiers, AMDGPU::OpName::src0},
      {AMDGPU::OpName::src1_modifiers, AMDGPU::OpName::src1},
      {AMDGPU::OpName::src2_modifiers, AMDGPU::OpName::src2}};
  int DefaultValue = (Mod == SISrcMods::OP_SEL_1);

  for (auto [SrcMod, Src] : MOps) {
    if (!AMDGPU::hasNamedOperand(Opc, Src))
      break;

```
**EN:** This section contains concrete logic for allOpsDefaultValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printPackedModifier`, `AMDGPU::hasNamedOperand`.
**CN:** 本节包含与 allOpsDefaultValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printPackedModifier`, `AMDGPU::hasNamedOperand`。

### Lines 1350-1393: Conditional logic and checks
```cpp
    int ModIdx = AMDGPU::getNamedOperandIdx(Opc, SrcMod);
    Ops[NumOps++] =
        (ModIdx != -1) ? MI->getOperand(ModIdx).getImm() : DefaultValue;
  }

  // Some instructions, e.g. v_interp_p2_f16 in GFX9, have src0, src2, but no
  // src1.
  if (NumOps == 1 && AMDGPU::hasNamedOperand(Opc, AMDGPU::OpName::src2) &&
      !AMDGPU::hasNamedOperand(Opc, AMDGPU::OpName::src1)) {
    Ops[NumOps++] = DefaultValue; // Set src1_modifiers to default.
    int Mod2Idx =
        AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::src2_modifiers);
    assert(Mod2Idx != -1);
    Ops[NumOps++] = MI->getOperand(Mod2Idx).getImm();
  }

  const bool HasDst =
      (AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::vdst) != -1) ||
      (AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::sdst) != -1);

  // Print three values of neg/opsel for wmma instructions (prints 0 when there
  // is no src_modifier operand instead of not printing anything).
  if (MII.get(MI->getOpcode()).TSFlags & SIInstrFlags::IsSWMMAC ||
      MII.get(MI->getOpcode()).TSFlags & SIInstrFlags::IsWMMA) {
    NumOps = 0;
    int DefaultValue = Mod == SISrcMods::OP_SEL_1;
    for (AMDGPU::OpName OpName :
         {AMDGPU::OpName::src0_modifiers, AMDGPU::OpName::src1_modifiers,
          AMDGPU::OpName::src2_modifiers}) {
      int Idx = AMDGPU::getNamedOperandIdx(Opc, OpName);
      if (Idx != -1)
        Ops[NumOps++] = MI->getOperand(Idx).getImm();
      else
        Ops[NumOps++] = DefaultValue;
    }
  }

  const bool HasDstSel =
      HasDst && NumOps > 0 && Mod == SISrcMods::OP_SEL_0 &&
      MII.get(MI->getOpcode()).TSFlags & SIInstrFlags::VOP3_OPSEL;

  const bool IsPacked =
    MII.get(MI->getOpcode()).TSFlags & SIInstrFlags::IsPacked;

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getNamedOperandIdx`, `AMDGPU::hasNamedOperand`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getNamedOperandIdx`, `AMDGPU::hasNamedOperand`。

### Lines 1394-1435: Declares Sep
```cpp
  if (allOpsDefaultValue(Ops, NumOps, Mod, IsPacked, HasDstSel))
    return;

  O << Name;
  ListSeparator Sep(",");
  for (int I = 0; I < NumOps; ++I)
    O << Sep << !!(Ops[I] & Mod);

  if (HasDstSel) {
    O << ',' << !!(Ops[0] & SISrcMods::DST_OP_SEL);
  }

  O << ']';
}

void AMDGPUInstPrinter::printOpSel(const MCInst *MI, unsigned,
                                   const MCSubtargetInfo &STI,
                                   raw_ostream &O) {
  unsigned Opc = MI->getOpcode();
  if (isCvt_F32_Fp8_Bf8_e64(Opc)) {
    auto SrcMod =
        AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::src0_modifiers);
    unsigned Mod = MI->getOperand(SrcMod).getImm();
    unsigned Index0 = !!(Mod & SISrcMods::OP_SEL_0);
    unsigned Index1 = !!(Mod & SISrcMods::OP_SEL_1);
    if (Index0 || Index1)
      O << " op_sel:[" << Index0 << ',' << Index1 << ']';
    return;
  }
  if (isPermlane16(Opc)) {
    auto FIN = AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::src0_modifiers);
    auto BCN = AMDGPU::getNamedOperandIdx(Opc, AMDGPU::OpName::src1_modifiers);
    unsigned FI = !!(MI->getOperand(FIN).getImm() & SISrcMods::OP_SEL_0);
    unsigned BC = !!(MI->getOperand(BCN).getImm() & SISrcMods::OP_SEL_0);
    if (FI || BC)
      O << " op_sel:[" << FI << ',' << BC << ']';
    return;
  }

  printPackedModifier(MI, " op_sel:[", SISrcMods::OP_SEL_0, O);
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUInstPrinter::printOpSel`, `AMDGPU::getNamedOperandIdx`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUInstPrinter::printOpSel`, `AMDGPU::getNamedOperandIdx`。

### Lines 1436-1473: Implements AMDGPUInstPrinter::printOpSelHi
```cpp
void AMDGPUInstPrinter::printOpSelHi(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  printPackedModifier(MI, " op_sel_hi:[", SISrcMods::OP_SEL_1, O);
}

void AMDGPUInstPrinter::printNegLo(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI,
                                   raw_ostream &O) {
  printPackedModifier(MI, " neg_lo:[", SISrcMods::NEG, O);
}

void AMDGPUInstPrinter::printNegHi(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI,
                                   raw_ostream &O) {
  printPackedModifier(MI, " neg_hi:[", SISrcMods::NEG_HI, O);
}

void AMDGPUInstPrinter::printIndexKey8bit(const MCInst *MI, unsigned OpNo,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  auto Imm = MI->getOperand(OpNo).getImm() & 0x7;
  if (Imm == 0)
    return;

  O << " index_key:" << Imm;
}

void AMDGPUInstPrinter::printIndexKey16bit(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  auto Imm = MI->getOperand(OpNo).getImm() & 0x7;
  if (Imm == 0)
    return;

  O << " index_key:" << Imm;
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printOpSelHi. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printOpSelHi`, `AMDGPUInstPrinter::printNegLo`, `AMDGPUInstPrinter::printNegHi`.
**CN:** 本节包含与 AMDGPUInstPrinter::printOpSelHi 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printOpSelHi`, `AMDGPUInstPrinter::printNegLo`, `AMDGPUInstPrinter::printNegHi`。

### Lines 1474-1516: Implements AMDGPUInstPrinter::printIndexKey32bit
```cpp
void AMDGPUInstPrinter::printIndexKey32bit(const MCInst *MI, unsigned OpNo,
                                           const MCSubtargetInfo &STI,
                                           raw_ostream &O) {
  auto Imm = MI->getOperand(OpNo).getImm() & 0x7;
  if (Imm == 0)
    return;

  O << " index_key:" << Imm;
}

void AMDGPUInstPrinter::printMatrixFMT(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O, char AorB) {
  auto Imm = MI->getOperand(OpNo).getImm() & 0x7;
  if (Imm == 0)
    return;

  O << " matrix_" << AorB << "_fmt:";
  if (Imm < static_cast<int64_t>(std::size(WMMAMods::ModMatrixFmt)))
    O << WMMAMods::ModMatrixFmt[Imm];
  else
    O << Imm;
}

void AMDGPUInstPrinter::printMatrixAFMT(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  printMatrixFMT(MI, OpNo, STI, O, 'a');
}

void AMDGPUInstPrinter::printMatrixBFMT(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  printMatrixFMT(MI, OpNo, STI, O, 'b');
}

void AMDGPUInstPrinter::printMatrixScale(const MCInst *MI, unsigned OpNo,
                                         const MCSubtargetInfo &STI,
                                         raw_ostream &O, char AorB) {
  auto Imm = MI->getOperand(OpNo).getImm() & 1;
  if (Imm == 0)
    return;

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printIndexKey32bit. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printIndexKey32bit`, `AMDGPUInstPrinter::printMatrixFMT`, `std::size`.
**CN:** 本节包含与 AMDGPUInstPrinter::printIndexKey32bit 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printIndexKey32bit`, `AMDGPUInstPrinter::printMatrixFMT`, `std::size`。

### Lines 1517-1555: Implements AMDGPUInstPrinter::printMatrixAScale
```cpp
  O << " matrix_" << AorB << "_scale:";
  if (Imm < static_cast<int64_t>(std::size(WMMAMods::ModMatrixScale)))
    O << WMMAMods::ModMatrixScale[Imm];
  else
    O << Imm;
}

void AMDGPUInstPrinter::printMatrixAScale(const MCInst *MI, unsigned OpNo,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  printMatrixScale(MI, OpNo, STI, O, 'a');
}

void AMDGPUInstPrinter::printMatrixBScale(const MCInst *MI, unsigned OpNo,
                                          const MCSubtargetInfo &STI,
                                          raw_ostream &O) {
  printMatrixScale(MI, OpNo, STI, O, 'b');
}

void AMDGPUInstPrinter::printMatrixScaleFmt(const MCInst *MI, unsigned OpNo,
                                            const MCSubtargetInfo &STI,
                                            raw_ostream &O, char AorB) {
  auto Imm = MI->getOperand(OpNo).getImm() & 3;
  if (Imm == 0)
    return;

  O << " matrix_" << AorB << "_scale_fmt:";
  if (Imm < static_cast<int64_t>(std::size(WMMAMods::ModMatrixScaleFmt)))
    O << WMMAMods::ModMatrixScaleFmt[Imm];
  else
    O << Imm;
}

void AMDGPUInstPrinter::printMatrixAScaleFmt(const MCInst *MI, unsigned OpNo,
                                             const MCSubtargetInfo &STI,
                                             raw_ostream &O) {
  printMatrixScaleFmt(MI, OpNo, STI, O, 'a');
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printMatrixAScale. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::size`, `AMDGPUInstPrinter::printMatrixAScale`, `AMDGPUInstPrinter::printMatrixBScale`.
**CN:** 本节包含与 AMDGPUInstPrinter::printMatrixAScale 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::size`, `AMDGPUInstPrinter::printMatrixAScale`, `AMDGPUInstPrinter::printMatrixBScale`。

### Lines 1556-1594: Implements AMDGPUInstPrinter::printMatrixBScaleFmt
```cpp
void AMDGPUInstPrinter::printMatrixBScaleFmt(const MCInst *MI, unsigned OpNo,
                                             const MCSubtargetInfo &STI,
                                             raw_ostream &O) {
  printMatrixScaleFmt(MI, OpNo, STI, O, 'b');
}

void AMDGPUInstPrinter::printInterpSlot(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  unsigned Imm = MI->getOperand(OpNum).getImm();
  switch (Imm) {
  case 0:
    O << "p10";
    break;
  case 1:
    O << "p20";
    break;
  case 2:
    O << "p0";
    break;
  default:
    O << "invalid_param_" << Imm;
  }
}

void AMDGPUInstPrinter::printInterpAttr(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  unsigned Attr = MI->getOperand(OpNum).getImm();
  O << "attr" << Attr;
}

void AMDGPUInstPrinter::printInterpAttrChan(const MCInst *MI, unsigned OpNum,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  unsigned Chan = MI->getOperand(OpNum).getImm();
  O << '.' << "xyzw"[Chan & 0x3];
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printMatrixBScaleFmt. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printMatrixBScaleFmt`, `AMDGPUInstPrinter::printInterpSlot`, `AMDGPUInstPrinter::printInterpAttr`.
**CN:** 本节包含与 AMDGPUInstPrinter::printMatrixBScaleFmt 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printMatrixBScaleFmt`, `AMDGPUInstPrinter::printInterpSlot`, `AMDGPUInstPrinter::printInterpAttr`。

### Lines 1595-1633: Implements AMDGPUInstPrinter::printGPRIdxMode
```cpp
void AMDGPUInstPrinter::printGPRIdxMode(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  using namespace llvm::AMDGPU::VGPRIndexMode;
  unsigned Val = MI->getOperand(OpNo).getImm();

  if ((Val & ~ENABLE_MASK) != 0) {
    O << formatHex(static_cast<uint64_t>(Val));
  } else {
    O << "gpr_idx(";
    ListSeparator Sep(",");
    for (unsigned ModeId = ID_MIN; ModeId <= ID_MAX; ++ModeId) {
      if (Val & (1 << ModeId))
        O << Sep << IdSymbolic[ModeId];
    }
    O << ')';
  }
}

void AMDGPUInstPrinter::printMemOperand(const MCInst *MI, unsigned OpNo,
                                        const MCSubtargetInfo &STI,
                                        raw_ostream &O) {
  printRegularOperand(MI, OpNo, STI, O);
  O  << ", ";
  printRegularOperand(MI, OpNo + 1, STI, O);
}

void AMDGPUInstPrinter::printIfSet(const MCInst *MI, unsigned OpNo,
                                   raw_ostream &O, StringRef Asm,
                                   StringRef Default) {
  const MCOperand &Op = MI->getOperand(OpNo);
  assert(Op.isImm());
  if (Op.getImm() == 1) {
    O << Asm;
  } else {
    O << Default;
  }
}

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printGPRIdxMode. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printGPRIdxMode`, `AMDGPUInstPrinter::printMemOperand`, `AMDGPUInstPrinter::printIfSet`.
**CN:** 本节包含与 AMDGPUInstPrinter::printGPRIdxMode 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printGPRIdxMode`, `AMDGPUInstPrinter::printMemOperand`, `AMDGPUInstPrinter::printIfSet`。

### Lines 1634-1667: Implements AMDGPUInstPrinter::printIfSet
```cpp
void AMDGPUInstPrinter::printIfSet(const MCInst *MI, unsigned OpNo,
                                   raw_ostream &O, char Asm) {
  const MCOperand &Op = MI->getOperand(OpNo);
  assert(Op.isImm());
  if (Op.getImm() == 1)
    O << Asm;
}

void AMDGPUInstPrinter::printOModSI(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  int Imm = MI->getOperand(OpNo).getImm();
  if (Imm == SIOutMods::MUL2)
    O << " mul:2";
  else if (Imm == SIOutMods::MUL4)
    O << " mul:4";
  else if (Imm == SIOutMods::DIV2)
    O << " div:2";
}

void AMDGPUInstPrinter::printSendMsg(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  using namespace llvm::AMDGPU::SendMsg;

  const unsigned Imm16 = MI->getOperand(OpNo).getImm();

  uint16_t MsgId;
  uint16_t OpId;
  uint16_t StreamId;
  decodeMsg(Imm16, MsgId, OpId, StreamId, STI);

  StringRef MsgName = getMsgName(MsgId, STI);

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printIfSet. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printIfSet`, `AMDGPUInstPrinter::printOModSI`, `AMDGPUInstPrinter::printSendMsg`.
**CN:** 本节包含与 AMDGPUInstPrinter::printIfSet 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printIfSet`, `AMDGPUInstPrinter::printOModSI`, `AMDGPUInstPrinter::printSendMsg`。

### Lines 1668-1708: Namespace declarations and scope setup
```cpp
  if (!MsgName.empty() && isValidMsgOp(MsgId, OpId, STI) &&
      isValidMsgStream(MsgId, OpId, StreamId, STI)) {
    O << "sendmsg(" << MsgName;
    if (msgRequiresOp(MsgId, STI)) {
      O << ", " << getMsgOpName(MsgId, OpId, STI);
      if (msgSupportsStream(MsgId, OpId, STI)) {
        O << ", " << StreamId;
      }
    }
    O << ')';
  } else if (encodeMsg(MsgId, OpId, StreamId) == Imm16) {
    O << "sendmsg(" << MsgId << ", " << OpId << ", " << StreamId << ')';
  } else {
    O << Imm16; // Unknown imm16 code.
  }
}

void AMDGPUInstPrinter::printWaitEvent(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  using namespace llvm::AMDGPU::WaitEvent;
  const uint16_t Imm16 = static_cast<uint16_t>(MI->getOperand(OpNo).getImm());

  StringRef EventName = getWaitEventMaskName(Imm16, STI);
  if (EventName.empty())
    O << formatHex(static_cast<uint64_t>(Imm16));
  else
    O << EventName;
}

static void printSwizzleBitmask(const uint16_t AndMask,
                                const uint16_t OrMask,
                                const uint16_t XorMask,
                                raw_ostream &O) {
  using namespace llvm::AMDGPU::Swizzle;

  uint16_t Probe0 = ((0            & AndMask) | OrMask) ^ XorMask;
  uint16_t Probe1 = ((BITMASK_MASK & AndMask) | OrMask) ^ XorMask;

  O << "\"";

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions. Main symbols: `AMDGPUInstPrinter::printWaitEvent`.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。 主要符号：`AMDGPUInstPrinter::printWaitEvent`。

### Lines 1709-1742: Namespace declarations and scope setup
```cpp
  for (unsigned Mask = 1 << (BITMASK_WIDTH - 1); Mask > 0; Mask >>= 1) {
    uint16_t p0 = Probe0 & Mask;
    uint16_t p1 = Probe1 & Mask;

    if (p0 == p1) {
      if (p0 == 0) {
        O << "0";
      } else {
        O << "1";
      }
    } else {
      if (p0 == 0) {
        O << "p";
      } else {
        O << "i";
      }
    }
  }

  O << "\"";
}

void AMDGPUInstPrinter::printSwizzle(const MCInst *MI, unsigned OpNo,
                                     const MCSubtargetInfo &STI,
                                     raw_ostream &O) {
  using namespace llvm::AMDGPU::Swizzle;

  uint16_t Imm = MI->getOperand(OpNo).getImm();
  if (Imm == 0) {
    return;
  }

  O << " offset:";

```
**EN:** This section arranges declarations into namespace scope so related AMDGPU functionality stays organized and avoids symbol collisions. Main symbols: `AMDGPUInstPrinter::printSwizzle`.
**CN:** 本节将声明组织到命名空间作用域中，使相关 AMDGPU 功能保持清晰并避免符号冲突。 主要符号：`AMDGPUInstPrinter::printSwizzle`。

### Lines 1743-1786: Conditional logic and checks
```cpp
  // Rotate and FFT modes
  if (Imm >= ROTATE_MODE_LO && AMDGPU::isGFX9Plus(STI)) {
    if (Imm >= FFT_MODE_LO) {
      O << "swizzle(" << IdSymbolic[ID_FFT] << ',' << (Imm & FFT_SWIZZLE_MASK)
        << ')';
    } else if (Imm >= ROTATE_MODE_LO) {
      O << "swizzle(" << IdSymbolic[ID_ROTATE] << ','
        << ((Imm >> ROTATE_DIR_SHIFT) & ROTATE_DIR_MASK) << ','
        << ((Imm >> ROTATE_SIZE_SHIFT) & ROTATE_SIZE_MASK) << ')';
    }
    return;
  }

  // Basic mode
  if ((Imm & QUAD_PERM_ENC_MASK) == QUAD_PERM_ENC) {
    O << "swizzle(" << IdSymbolic[ID_QUAD_PERM];
    for (unsigned I = 0; I < LANE_NUM; ++I) {
      O << ",";
      O << formatDec(Imm & LANE_MASK);
      Imm >>= LANE_SHIFT;
    }
    O << ")";

  } else if ((Imm & BITMASK_PERM_ENC_MASK) == BITMASK_PERM_ENC) {

    uint16_t AndMask = (Imm >> BITMASK_AND_SHIFT) & BITMASK_MASK;
    uint16_t OrMask  = (Imm >> BITMASK_OR_SHIFT)  & BITMASK_MASK;
    uint16_t XorMask = (Imm >> BITMASK_XOR_SHIFT) & BITMASK_MASK;

    if (AndMask == BITMASK_MAX && OrMask == 0 && llvm::popcount(XorMask) == 1) {

      O << "swizzle(" << IdSymbolic[ID_SWAP];
      O << ",";
      O << formatDec(XorMask);
      O << ")";

    } else if (AndMask == BITMASK_MAX && OrMask == 0 && XorMask > 0 &&
               isPowerOf2_64(XorMask + 1)) {

      O << "swizzle(" << IdSymbolic[ID_REVERSE];
      O << ",";
      O << formatDec(XorMask + 1);
      O << ")";

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::isGFX9Plus`, `llvm::popcount`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::isGFX9Plus`, `llvm::popcount`。

### Lines 1787-1829: Conditional logic and checks
```cpp
    } else {

      uint16_t GroupSize = BITMASK_MAX - AndMask + 1;
      if (GroupSize > 1 &&
          isPowerOf2_64(GroupSize) &&
          OrMask < GroupSize &&
          XorMask == 0) {

        O << "swizzle(" << IdSymbolic[ID_BROADCAST];
        O << ",";
        O << formatDec(GroupSize);
        O << ",";
        O << formatDec(OrMask);
        O << ")";

      } else {
        O << "swizzle(" << IdSymbolic[ID_BITMASK_PERM];
        O << ",";
        printSwizzleBitmask(AndMask, OrMask, XorMask, O);
        O << ")";
      }
    }
  } else {
    printU16ImmDecOperand(MI, OpNo, O);
  }
}

void AMDGPUInstPrinter::printSWaitCnt(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  AMDGPU::IsaVersion ISA = AMDGPU::getIsaVersion(STI.getCPU());

  unsigned SImm16 = MI->getOperand(OpNo).getImm();
  unsigned Vmcnt, Expcnt, Lgkmcnt;
  decodeWaitcnt(ISA, SImm16, Vmcnt, Expcnt, Lgkmcnt);

  bool IsDefaultVmcnt = Vmcnt == getVmcntBitMask(ISA);
  bool IsDefaultExpcnt = Expcnt == getExpcntBitMask(ISA);
  bool IsDefaultLgkmcnt = Lgkmcnt == getLgkmcntBitMask(ISA);
  bool PrintAll = IsDefaultVmcnt && IsDefaultExpcnt && IsDefaultLgkmcnt;

  ListSeparator Sep(" ");

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUInstPrinter::printSWaitCnt`, `AMDGPU::getIsaVersion`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUInstPrinter::printSWaitCnt`, `AMDGPU::getIsaVersion`。

### Lines 1830-1872: Implements AMDGPUInstPrinter::printDepCtr
```cpp
  if (!IsDefaultVmcnt || PrintAll)
    O << Sep << "vmcnt(" << Vmcnt << ')';

  if (!IsDefaultExpcnt || PrintAll)
    O << Sep << "expcnt(" << Expcnt << ')';

  if (!IsDefaultLgkmcnt || PrintAll)
    O << Sep << "lgkmcnt(" << Lgkmcnt << ')';
}

void AMDGPUInstPrinter::printDepCtr(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  using namespace llvm::AMDGPU::DepCtr;

  uint64_t Imm16 = MI->getOperand(OpNo).getImm() & 0xffff;

  bool HasNonDefaultVal = false;
  if (isSymbolicDepCtrEncoding(Imm16, HasNonDefaultVal, STI)) {
    int Id = 0;
    StringRef Name;
    unsigned Val;
    bool IsDefault;
    ListSeparator Sep(" ");
    while (decodeDepCtr(Imm16, Id, Name, Val, IsDefault, STI)) {
      if (!IsDefault || !HasNonDefaultVal)
        O << Sep << Name << '(' << Val << ')';
    }
  } else {
    O << formatHex(Imm16);
  }
}

void AMDGPUInstPrinter::printSDelayALU(const MCInst *MI, unsigned OpNo,
                                       const MCSubtargetInfo &STI,
                                       raw_ostream &O) {
  const char *BadInstId = "/* invalid instid value */";
  static const std::array<const char *, 12> InstIds = {
      "NO_DEP",        "VALU_DEP_1",    "VALU_DEP_2",
      "VALU_DEP_3",    "VALU_DEP_4",    "TRANS32_DEP_1",
      "TRANS32_DEP_2", "TRANS32_DEP_3", "FMA_ACCUM_CYCLE_1",
      "SALU_CYCLE_1",  "SALU_CYCLE_2",  "SALU_CYCLE_3"};

```
**EN:** This section contains concrete logic for AMDGPUInstPrinter::printDepCtr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUInstPrinter::printDepCtr`, `AMDGPUInstPrinter::printSDelayALU`.
**CN:** 本节包含与 AMDGPUInstPrinter::printDepCtr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUInstPrinter::printDepCtr`, `AMDGPUInstPrinter::printSDelayALU`。

### Lines 1873-1912: Declares getOperand
```cpp
  const char *BadInstSkip = "/* invalid instskip value */";
  static const std::array<const char *, 6> InstSkips = {
      "SAME", "NEXT", "SKIP_1", "SKIP_2", "SKIP_3", "SKIP_4"};

  unsigned SImm16 = MI->getOperand(OpNo).getImm();
  const char *Prefix = "";

  unsigned Value = SImm16 & 0xF;
  if (Value) {
    const char *Name = Value < InstIds.size() ? InstIds[Value] : BadInstId;
    O << Prefix << "instid0(" << Name << ')';
    Prefix = " | ";
  }

  Value = (SImm16 >> 4) & 7;
  if (Value) {
    const char *Name =
        Value < InstSkips.size() ? InstSkips[Value] : BadInstSkip;
    O << Prefix << "instskip(" << Name << ')';
    Prefix = " | ";
  }

  Value = (SImm16 >> 7) & 0xF;
  if (Value) {
    const char *Name = Value < InstIds.size() ? InstIds[Value] : BadInstId;
    O << Prefix << "instid1(" << Name << ')';
    Prefix = " | ";
  }

  if (!*Prefix)
    O << "0";
}

void AMDGPUInstPrinter::printHwreg(const MCInst *MI, unsigned OpNo,
                                   const MCSubtargetInfo &STI, raw_ostream &O) {
  using namespace llvm::AMDGPU::Hwreg;
  unsigned Val = MI->getOperand(OpNo).getImm();
  auto [Id, Offset, Width] = HwregEncoding::decode(Val);
  StringRef HwRegName = getHwreg(Id, STI);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUInstPrinter::printHwreg`, `HwregEncoding::decode`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUInstPrinter::printHwreg`, `HwregEncoding::decode`。

### Lines 1913-1950: Declares hwreg
```cpp
  O << "hwreg(";
  if (!HwRegName.empty()) {
    O << HwRegName;
  } else {
    O << Id;
  }
  if (Width != HwregSize::Default || Offset != HwregOffset::Default)
    O << ", " << Offset << ", " << Width;
  O << ')';
}

void AMDGPUInstPrinter::printEndpgm(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  uint16_t Imm = MI->getOperand(OpNo).getImm();
  if (Imm == 0) {
    return;
  }

  O << ' ' << formatDec(Imm);
}

void AMDGPUInstPrinter::printNamedInt(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O, StringRef Prefix,
                                      bool PrintInHex, bool AlwaysPrint) {
  int64_t V = MI->getOperand(OpNo).getImm();
  if (AlwaysPrint || V != 0)
    O << ' ' << Prefix << ':' << (PrintInHex ? formatHex(V) : formatDec(V));
}

void AMDGPUInstPrinter::printBitOp3(const MCInst *MI, unsigned OpNo,
                                    const MCSubtargetInfo &STI,
                                    raw_ostream &O) {
  uint8_t Imm = MI->getOperand(OpNo).getImm();
  if (!Imm)
    return;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPUInstPrinter::printEndpgm`, `AMDGPUInstPrinter::printNamedInt`, `AMDGPUInstPrinter::printBitOp3`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPUInstPrinter::printEndpgm`, `AMDGPUInstPrinter::printNamedInt`, `AMDGPUInstPrinter::printBitOp3`。

### Lines 1951-1968: Header dependencies and setup
```cpp
  O << " bitop3:";
  if (Imm <= 10)
    O << formatDec(Imm);
  else
    O << formatHex(static_cast<uint64_t>(Imm));
}

void AMDGPUInstPrinter::printScaleSel(const MCInst *MI, unsigned OpNo,
                                      const MCSubtargetInfo &STI,
                                      raw_ostream &O) {
  uint8_t Imm = MI->getOperand(OpNo).getImm();
  if (!Imm)
    return;

  O << " scale_sel:" << formatDec(Imm);
}

#include "AMDGPUGenAsmWriter.inc"
```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUInstPrinter::printScaleSel`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUInstPrinter::printScaleSel`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUInstPrinter::printRegName`, `AMDGPUInstPrinter::printInst`, `AMDGPUInstPrinter::printU16ImmOperand`, `AMDGPUInstPrinter::printU16ImmDecOperand`, `AMDGPUInstPrinter::printU32ImmOperand`, `AMDGPUInstPrinter::printFP64ImmOperand`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUInstPrinter.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIDefines.h"`
- `"Utils/AMDGPUAsmUtils.h"`
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/StringExtras.h"`
- `"llvm/MC/MCAsmInfo.h"`
- `"llvm/MC/MCExpr.h"`
- `"llvm/MC/MCInst.h"`
- `"llvm/MC/MCInstrDesc.h"`
- `"llvm/MC/MCInstrInfo.h"`
- `"llvm/MC/MCRegisterInfo.h"`
- `"llvm/MC/MCSubtargetInfo.h"`
- `"llvm/TargetParser/TargetParser.h"`
- `"AMDGPUGenAsmWriter.inc"`
