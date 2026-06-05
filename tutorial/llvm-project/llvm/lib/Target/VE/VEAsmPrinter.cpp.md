# VEAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEAsmPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Lowers MachineInstr to the MC layer and emits target-specific assembly/object output.
  - **CN**: 把 MachineInstr 降到 MC 层，并输出目标相关的汇编/目标文件内容。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- VEAsmPrinter.cpp - VE LLVM assembly writer ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a printer that converts from our internal representation
// of machine-dependent LLVM code to GAS-format VE assembly language.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-30
```cpp

#include "MCTargetDesc/VEInstPrinter.h"
#include "MCTargetDesc/VEMCAsmInfo.h"
#include "MCTargetDesc/VETargetStreamer.h"
#include "TargetInfo/VETargetInfo.h"
#include "VE.h"
#include "VEInstrInfo.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineModuleInfoImpls.h"
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/IR/Mangler.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/TargetRegistry.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEInstPrinter.h`, `MCTargetDesc/VEMCAsmInfo.h`, `MCTargetDesc/VETargetStreamer.h`, `TargetInfo/VETargetInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEInstPrinter.h`, `MCTargetDesc/VEMCAsmInfo.h`, `MCTargetDesc/VETargetStreamer.h`, `TargetInfo/VETargetInfo.h`。

### Lines 31-42
```cpp
#include "llvm/Support/Compiler.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "ve-asmprinter"

namespace {
class VEAsmPrinter : public AsmPrinter {
  VETargetStreamer &getTargetStreamer() {
    return static_cast<VETargetStreamer &>(*OutStreamer->getTargetStreamer());
  }

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/Compiler.h`, `llvm/Support/raw_ostream.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/Compiler.h`, `llvm/Support/raw_ostream.h`。

### Lines 43-55
```cpp
public:
  explicit VEAsmPrinter(TargetMachine &TM, std::unique_ptr<MCStreamer> Streamer)
      : AsmPrinter(TM, std::move(Streamer), ID) {}

  StringRef getPassName() const override { return "VE Assembly Printer"; }

  void lowerGETGOTAndEmitMCInsts(const MachineInstr *MI,
                                 const MCSubtargetInfo &STI);
  void lowerGETFunPLTAndEmitMCInsts(const MachineInstr *MI,
                                    const MCSubtargetInfo &STI);
  void lowerGETTLSAddrAndEmitMCInsts(const MachineInstr *MI,
                                     const MCSubtargetInfo &STI);

```
- **EN**: Implements logic around `VEAsmPrinter`, `AsmPrinter`, `getPassName`, `lowerGETGOTAndEmitMCInsts`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `VEAsmPrinter`, `AsmPrinter`, `getPassName`, `lowerGETGOTAndEmitMCInsts`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 56-66
```cpp
  void emitInstruction(const MachineInstr *MI) override;

  static const char *getRegisterName(MCRegister Reg) {
    return VEInstPrinter::getRegisterName(Reg);
  }
  void printOperand(const MachineInstr *MI, int OpNum, raw_ostream &OS);
  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                       const char *ExtraCode, raw_ostream &O) override;
  bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
                             const char *ExtraCode, raw_ostream &O) override;

```
- **EN**: Implements logic around `emitInstruction`, `getRegisterName`, `printOperand`, `PrintAsmOperand`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitInstruction`, `getRegisterName`, `printOperand`, `PrintAsmOperand`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 67-77
```cpp
  static char ID;
};
} // end of anonymous namespace

static MCOperand createVEMCOperand(VE::Specifier Kind, MCSymbol *Sym,
                                   MCContext &OutContext) {
  const MCSymbolRefExpr *MCSym = MCSymbolRefExpr::create(Sym, OutContext);
  return MCOperand::createExpr(
      MCSpecifierExpr::create(MCSym, Kind, OutContext));
}

```
- **EN**: Implements logic around `createVEMCOperand`, `create`, `createExpr`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createVEMCOperand`, `create`, `createExpr` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 78-91
```cpp
static MCOperand createGOTRelExprOp(VE::Specifier Kind, MCSymbol *GOTLabel,
                                    MCContext &OutContext) {
  const MCSymbolRefExpr *GOT = MCSymbolRefExpr::create(GOTLabel, OutContext);
  return MCOperand::createExpr(MCSpecifierExpr::create(GOT, Kind, OutContext));
}

static void emitSIC(MCStreamer &OutStreamer, MCOperand &RD,
                    const MCSubtargetInfo &STI) {
  MCInst SICInst;
  SICInst.setOpcode(VE::SIC);
  SICInst.addOperand(RD);
  OutStreamer.emitInstruction(SICInst, STI);
}

```
- **EN**: Implements logic around `createGOTRelExprOp`, `create`, `createExpr`, `emitSIC`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createGOTRelExprOp`, `create`, `createExpr`, `emitSIC`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 92-103
```cpp
static void emitBSIC(MCStreamer &OutStreamer, MCOperand &R1, MCOperand &R2,
                     const MCSubtargetInfo &STI) {
  MCInst BSICInst;
  BSICInst.setOpcode(VE::BSICrii);
  BSICInst.addOperand(R1);
  BSICInst.addOperand(R2);
  MCOperand czero = MCOperand::createImm(0);
  BSICInst.addOperand(czero);
  BSICInst.addOperand(czero);
  OutStreamer.emitInstruction(BSICInst, STI);
}

```
- **EN**: Implements logic around `emitBSIC`, `setOpcode`, `addOperand`, `createImm`, ...; this block works at the MC layer.
- **CN**: 围绕 `emitBSIC`, `setOpcode`, `addOperand`, `createImm`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 104-115
```cpp
static void emitLEAzzi(MCStreamer &OutStreamer, MCOperand &Imm, MCOperand &RD,
                       const MCSubtargetInfo &STI) {
  MCInst LEAInst;
  LEAInst.setOpcode(VE::LEAzii);
  LEAInst.addOperand(RD);
  MCOperand CZero = MCOperand::createImm(0);
  LEAInst.addOperand(CZero);
  LEAInst.addOperand(CZero);
  LEAInst.addOperand(Imm);
  OutStreamer.emitInstruction(LEAInst, STI);
}

```
- **EN**: Implements logic around `emitLEAzzi`, `setOpcode`, `addOperand`, `createImm`, ...; this block works at the MC layer.
- **CN**: 围绕 `emitLEAzzi`, `setOpcode`, `addOperand`, `createImm`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 116-127
```cpp
static void emitLEASLzzi(MCStreamer &OutStreamer, MCOperand &Imm, MCOperand &RD,
                         const MCSubtargetInfo &STI) {
  MCInst LEASLInst;
  LEASLInst.setOpcode(VE::LEASLzii);
  LEASLInst.addOperand(RD);
  MCOperand CZero = MCOperand::createImm(0);
  LEASLInst.addOperand(CZero);
  LEASLInst.addOperand(CZero);
  LEASLInst.addOperand(Imm);
  OutStreamer.emitInstruction(LEASLInst, STI);
}

```
- **EN**: Implements logic around `emitLEASLzzi`, `setOpcode`, `addOperand`, `createImm`, ...; this block works at the MC layer.
- **CN**: 围绕 `emitLEASLzzi`, `setOpcode`, `addOperand`, `createImm`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 128-139
```cpp
static void emitLEAzii(MCStreamer &OutStreamer, MCOperand &RS1, MCOperand &Imm,
                       MCOperand &RD, const MCSubtargetInfo &STI) {
  MCInst LEAInst;
  LEAInst.setOpcode(VE::LEAzii);
  LEAInst.addOperand(RD);
  MCOperand CZero = MCOperand::createImm(0);
  LEAInst.addOperand(CZero);
  LEAInst.addOperand(RS1);
  LEAInst.addOperand(Imm);
  OutStreamer.emitInstruction(LEAInst, STI);
}

```
- **EN**: Implements logic around `emitLEAzii`, `setOpcode`, `addOperand`, `createImm`, ...; this block works at the MC layer.
- **CN**: 围绕 `emitLEAzii`, `setOpcode`, `addOperand`, `createImm`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 140-151
```cpp
static void emitLEASLrri(MCStreamer &OutStreamer, MCOperand &RS1,
                         MCOperand &RS2, MCOperand &Imm, MCOperand &RD,
                         const MCSubtargetInfo &STI) {
  MCInst LEASLInst;
  LEASLInst.setOpcode(VE::LEASLrri);
  LEASLInst.addOperand(RD);
  LEASLInst.addOperand(RS1);
  LEASLInst.addOperand(RS2);
  LEASLInst.addOperand(Imm);
  OutStreamer.emitInstruction(LEASLInst, STI);
}

```
- **EN**: Implements logic around `emitLEASLrri`, `setOpcode`, `addOperand`, `emitInstruction`; this block works at the MC layer.
- **CN**: 围绕 `emitLEASLrri`, `setOpcode`, `addOperand`, `emitInstruction` 实现具体逻辑；这一段工作在 MC 层。

### Lines 152-162
```cpp
static void emitBinary(MCStreamer &OutStreamer, unsigned Opcode, MCOperand &RS1,
                       MCOperand &Src2, MCOperand &RD,
                       const MCSubtargetInfo &STI) {
  MCInst Inst;
  Inst.setOpcode(Opcode);
  Inst.addOperand(RD);
  Inst.addOperand(RS1);
  Inst.addOperand(Src2);
  OutStreamer.emitInstruction(Inst, STI);
}

```
- **EN**: Implements logic around `emitBinary`, `setOpcode`, `addOperand`, `emitInstruction`; this block works at the MC layer.
- **CN**: 围绕 `emitBinary`, `setOpcode`, `addOperand`, `emitInstruction` 实现具体逻辑；这一段工作在 MC 层。

### Lines 163-171
```cpp
static void emitANDrm(MCStreamer &OutStreamer, MCOperand &RS1, MCOperand &Imm,
                      MCOperand &RD, const MCSubtargetInfo &STI) {
  emitBinary(OutStreamer, VE::ANDrm, RS1, Imm, RD, STI);
}

static void emitHiLo(MCStreamer &OutStreamer, MCSymbol *GOTSym,
                     VE::Specifier HiKind, VE::Specifier LoKind, MCOperand &RD,
                     MCContext &OutContext, const MCSubtargetInfo &STI) {

```
- **EN**: Implements logic around `emitANDrm`, `emitBinary`, `emitHiLo`; this block works at the MC layer.
- **CN**: 围绕 `emitANDrm`, `emitBinary`, `emitHiLo` 实现具体逻辑；这一段工作在 MC 层。

### Lines 172-184
```cpp
  MCOperand hi = createVEMCOperand(HiKind, GOTSym, OutContext);
  MCOperand lo = createVEMCOperand(LoKind, GOTSym, OutContext);
  emitLEAzzi(OutStreamer, lo, RD, STI);
  MCOperand M032 = MCOperand::createImm(M0(32));
  emitANDrm(OutStreamer, RD, M032, RD, STI);
  emitLEASLzzi(OutStreamer, hi, RD, STI);
}

void VEAsmPrinter::lowerGETGOTAndEmitMCInsts(const MachineInstr *MI,
                                             const MCSubtargetInfo &STI) {
  MCSymbol *GOTLabel =
      OutContext.getOrCreateSymbol(Twine("_GLOBAL_OFFSET_TABLE_"));

```
- **EN**: Implements logic around `createVEMCOperand`, `emitLEAzzi`, `createImm`, `emitANDrm`, ...; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `createVEMCOperand`, `emitLEAzzi`, `createImm`, `emitANDrm`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 185-202
```cpp
  const MachineOperand &MO = MI->getOperand(0);
  MCOperand MCRegOP = MCOperand::createReg(MO.getReg());

  if (!isPositionIndependent()) {
    // Just load the address of GOT to MCRegOP.
    switch (TM.getCodeModel()) {
    default:
      llvm_unreachable("Unsupported absolute code model");
    case CodeModel::Small:
    case CodeModel::Medium:
    case CodeModel::Large:
      emitHiLo(*OutStreamer, GOTLabel, VE::S_HI32, VE::S_LO32, MCRegOP,
               OutContext, STI);
      break;
    }
    return;
  }

```
- **EN**: Implements logic around `getOperand`, `createReg`, `llvm_unreachable`, `emitHiLo`; this block uses `switch`-based dispatch; applies conditional target rules.
- **CN**: 围绕 `getOperand`, `createReg`, `llvm_unreachable`, `emitHiLo` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则。

### Lines 203-219
```cpp
  MCOperand RegGOT = MCOperand::createReg(VE::SX15); // GOT
  MCOperand RegPLT = MCOperand::createReg(VE::SX16); // PLT

  // lea %got, _GLOBAL_OFFSET_TABLE_@PC_LO(-24)
  // and %got, %got, (32)0
  // sic %plt
  // lea.sl %got, _GLOBAL_OFFSET_TABLE_@PC_HI(%plt, %got)
  MCOperand cim24 = MCOperand::createImm(-24);
  MCOperand loImm = createGOTRelExprOp(VE::S_PC_LO32, GOTLabel, OutContext);
  emitLEAzii(*OutStreamer, cim24, loImm, MCRegOP, STI);
  MCOperand M032 = MCOperand::createImm(M0(32));
  emitANDrm(*OutStreamer, MCRegOP, M032, MCRegOP, STI);
  emitSIC(*OutStreamer, RegPLT, STI);
  MCOperand hiImm = createGOTRelExprOp(VE::S_PC_HI32, GOTLabel, OutContext);
  emitLEASLrri(*OutStreamer, RegGOT, RegPLT, hiImm, MCRegOP, STI);
}

```
- **EN**: Implements logic around `createReg`, `createImm`, `createGOTRelExprOp`, `emitLEAzii`, ....
- **CN**: 围绕 `createReg`, `createImm`, `createGOTRelExprOp`, `emitLEAzii`, ... 实现具体逻辑。

### Lines 220-237
```cpp
void VEAsmPrinter::lowerGETFunPLTAndEmitMCInsts(const MachineInstr *MI,
                                                const MCSubtargetInfo &STI) {
  const MachineOperand &MO = MI->getOperand(0);
  MCOperand MCRegOP = MCOperand::createReg(MO.getReg());
  const MachineOperand &Addr = MI->getOperand(1);
  MCSymbol *AddrSym = nullptr;

  switch (Addr.getType()) {
  default:
    llvm_unreachable("<unknown operand type>");
    return;
  case MachineOperand::MO_MachineBasicBlock:
    report_fatal_error("MBB is not supported yet");
    return;
  case MachineOperand::MO_ConstantPoolIndex:
    report_fatal_error("ConstantPool is not supported yet");
    return;
  case MachineOperand::MO_ExternalSymbol:
```
- **EN**: Implements logic around `lowerGETFunPLTAndEmitMCInsts`, `getOperand`, `createReg`, `llvm_unreachable`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `lowerGETFunPLTAndEmitMCInsts`, `getOperand`, `createReg`, `llvm_unreachable`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 238-249
```cpp
    AddrSym = GetExternalSymbolSymbol(Addr.getSymbolName());
    break;
  case MachineOperand::MO_GlobalAddress:
    AddrSym = getSymbol(Addr.getGlobal());
    break;
  }

  if (!isPositionIndependent()) {
    llvm_unreachable("Unsupported uses of %plt in not PIC code");
    return;
  }

```
- **EN**: Implements logic around `GetExternalSymbolSymbol`, `getSymbol`, `llvm_unreachable`; this block applies conditional target rules.
- **CN**: 围绕 `GetExternalSymbolSymbol`, `getSymbol`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 250-265
```cpp
  MCOperand RegPLT = MCOperand::createReg(VE::SX16); // PLT

  // lea %dst, func@plt_lo(-24)
  // and %dst, %dst, (32)0
  // sic %plt                            ; FIXME: is it safe to use %plt here?
  // lea.sl %dst, func@plt_hi(%plt, %dst)
  MCOperand cim24 = MCOperand::createImm(-24);
  MCOperand loImm = createGOTRelExprOp(VE::S_PLT_LO32, AddrSym, OutContext);
  emitLEAzii(*OutStreamer, cim24, loImm, MCRegOP, STI);
  MCOperand M032 = MCOperand::createImm(M0(32));
  emitANDrm(*OutStreamer, MCRegOP, M032, MCRegOP, STI);
  emitSIC(*OutStreamer, RegPLT, STI);
  MCOperand hiImm = createGOTRelExprOp(VE::S_PLT_HI32, AddrSym, OutContext);
  emitLEASLrri(*OutStreamer, MCRegOP, RegPLT, hiImm, MCRegOP, STI);
}

```
- **EN**: Implements logic around `createReg`, `createImm`, `createGOTRelExprOp`, `emitLEAzii`, ....
- **CN**: 围绕 `createReg`, `createImm`, `createGOTRelExprOp`, `emitLEAzii`, ... 实现具体逻辑。

### Lines 266-283
```cpp
void VEAsmPrinter::lowerGETTLSAddrAndEmitMCInsts(const MachineInstr *MI,
                                                 const MCSubtargetInfo &STI) {
  const MachineOperand &Addr = MI->getOperand(0);
  MCSymbol *AddrSym = nullptr;

  switch (Addr.getType()) {
  default:
    llvm_unreachable("<unknown operand type>");
    return;
  case MachineOperand::MO_MachineBasicBlock:
    report_fatal_error("MBB is not supported yet");
    return;
  case MachineOperand::MO_ConstantPoolIndex:
    report_fatal_error("ConstantPool is not supported yet");
    return;
  case MachineOperand::MO_ExternalSymbol:
    AddrSym = GetExternalSymbolSymbol(Addr.getSymbolName());
    break;
```
- **EN**: Implements logic around `lowerGETTLSAddrAndEmitMCInsts`, `getOperand`, `llvm_unreachable`, `report_fatal_error`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `lowerGETTLSAddrAndEmitMCInsts`, `getOperand`, `llvm_unreachable`, `report_fatal_error`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 284-293
```cpp
  case MachineOperand::MO_GlobalAddress:
    AddrSym = getSymbol(Addr.getGlobal());
    break;
  }

  MCOperand RegLR = MCOperand::createReg(VE::SX10);  // LR
  MCOperand RegS0 = MCOperand::createReg(VE::SX0);   // S0
  MCOperand RegS12 = MCOperand::createReg(VE::SX12); // S12
  MCSymbol *GetTLSLabel = OutContext.getOrCreateSymbol(Twine("__tls_get_addr"));

```
- **EN**: Implements logic around `getSymbol`, `createReg`, `getOrCreateSymbol`.
- **CN**: 围绕 `getSymbol`, `createReg`, `getOrCreateSymbol` 实现具体逻辑。

### Lines 294-311
```cpp
  // lea %s0, sym@tls_gd_lo(-24)
  // and %s0, %s0, (32)0
  // sic %lr
  // lea.sl %s0, sym@tls_gd_hi(%lr, %s0)
  // lea %s12, __tls_get_addr@plt_lo(8)
  // and %s12, %s12, (32)0
  // lea.sl %s12, __tls_get_addr@plt_hi(%s12, %lr)
  // bsic %lr, (, %s12)
  MCOperand cim24 = MCOperand::createImm(-24);
  MCOperand loImm = createGOTRelExprOp(VE::S_TLS_GD_LO32, AddrSym, OutContext);
  emitLEAzii(*OutStreamer, cim24, loImm, RegS0, STI);
  MCOperand M032 = MCOperand::createImm(M0(32));
  emitANDrm(*OutStreamer, RegS0, M032, RegS0, STI);
  emitSIC(*OutStreamer, RegLR, STI);
  MCOperand hiImm = createGOTRelExprOp(VE::S_TLS_GD_HI32, AddrSym, OutContext);
  emitLEASLrri(*OutStreamer, RegS0, RegLR, hiImm, RegS0, STI);
  MCOperand ci8 = MCOperand::createImm(8);
  MCOperand loImm2 =
```
- **EN**: Implements logic around `createImm`, `createGOTRelExprOp`, `emitLEAzii`, `emitANDrm`, ....
- **CN**: 围绕 `createImm`, `createGOTRelExprOp`, `emitLEAzii`, `emitANDrm`, ... 实现具体逻辑。

### Lines 312-320
```cpp
      createGOTRelExprOp(VE::S_PLT_LO32, GetTLSLabel, OutContext);
  emitLEAzii(*OutStreamer, ci8, loImm2, RegS12, STI);
  emitANDrm(*OutStreamer, RegS12, M032, RegS12, STI);
  MCOperand hiImm2 =
      createGOTRelExprOp(VE::S_PLT_HI32, GetTLSLabel, OutContext);
  emitLEASLrri(*OutStreamer, RegS12, RegLR, hiImm2, RegS12, STI);
  emitBSIC(*OutStreamer, RegLR, RegS12, STI);
}

```
- **EN**: Implements logic around `createGOTRelExprOp`, `emitLEAzii`, `emitANDrm`, `emitLEASLrri`, ....
- **CN**: 围绕 `createGOTRelExprOp`, `emitLEAzii`, `emitANDrm`, `emitLEASLrri`, ... 实现具体逻辑。

### Lines 321-338
```cpp
void VEAsmPrinter::emitInstruction(const MachineInstr *MI) {
  VE_MC::verifyInstructionPredicates(MI->getOpcode(),
                                     getSubtargetInfo().getFeatureBits());

  switch (MI->getOpcode()) {
  default:
    break;
  case TargetOpcode::DBG_VALUE:
    // FIXME: Debug Value.
    return;
  case VE::GETGOT:
    lowerGETGOTAndEmitMCInsts(MI, getSubtargetInfo());
    return;
  case VE::GETFUNPLT:
    lowerGETFunPLTAndEmitMCInsts(MI, getSubtargetInfo());
    return;
  case VE::GETTLSADDR:
    lowerGETTLSAddrAndEmitMCInsts(MI, getSubtargetInfo());
```
- **EN**: Implements logic around `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `lowerGETGOTAndEmitMCInsts`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `lowerGETGOTAndEmitMCInsts`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 339-350
```cpp
    return;
  }

  MachineBasicBlock::const_instr_iterator I = MI->getIterator();
  MachineBasicBlock::const_instr_iterator E = MI->getParent()->instr_end();
  do {
    MCInst TmpInst;
    LowerVEMachineInstrToMCInst(&*I, TmpInst, *this);
    EmitToStreamer(*OutStreamer, TmpInst);
  } while ((++I != E) && I->isInsideBundle()); // Delay slot check.
}

```
- **EN**: Implements logic around `getIterator`, `getParent`, `LowerVEMachineInstrToMCInst`, `EmitToStreamer`; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `getIterator`, `getParent`, `LowerVEMachineInstrToMCInst`, `EmitToStreamer` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 351-366
```cpp
void VEAsmPrinter::printOperand(const MachineInstr *MI, int OpNum,
                                raw_ostream &O) {
  const MachineOperand &MO = MI->getOperand(OpNum);

  switch (MO.getType()) {
  case MachineOperand::MO_Register:
    O << "%" << StringRef(getRegisterName(MO.getReg())).lower();
    break;
  case MachineOperand::MO_Immediate:
    O << (int)MO.getImm();
    break;
  default:
    llvm_unreachable("<unknown operand type>");
  }
}

```
- **EN**: Implements logic around `printOperand`, `getOperand`, `StringRef`, `getImm`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `getOperand`, `StringRef`, `getImm`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 367-383
```cpp
// PrintAsmOperand - Print out an operand for an inline asm expression.
bool VEAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                                   const char *ExtraCode, raw_ostream &O) {
  if (ExtraCode && ExtraCode[0]) {
    if (ExtraCode[1] != 0)
      return true; // Unknown modifier.

    switch (ExtraCode[0]) {
    default:
      // See if this is a generic print operand
      return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
    case 'r':
    case 'v':
      break;
    }
  }

```
- **EN**: Implements logic around `PrintAsmOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PrintAsmOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 384-394
```cpp
  printOperand(MI, OpNo, O);

  return false;
}

bool VEAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
                                         const char *ExtraCode,
                                         raw_ostream &O) {
  if (ExtraCode && ExtraCode[0])
    return true;  // Unknown modifier

```
- **EN**: Implements logic around `printOperand`, `PrintAsmMemoryOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `PrintAsmMemoryOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 395-412
```cpp
  if (MI->getOperand(OpNo+1).isImm() &&
      MI->getOperand(OpNo+1).getImm() == 0) {
    // don't print "+0"
  } else {
    printOperand(MI, OpNo+1, O);
  }
  if (MI->getOperand(OpNo).isImm() &&
      MI->getOperand(OpNo).getImm() == 0) {
    if (MI->getOperand(OpNo+1).isImm() &&
        MI->getOperand(OpNo+1).getImm() == 0) {
      O << "0";
    } else {
      // don't print "(0)"
    }
  } else {
    O << "(";
    printOperand(MI, OpNo, O);
    O << ")";
```
- **EN**: Implements logic around `getOperand`, `printOperand`; this block applies conditional target rules.
- **CN**: 围绕 `getOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 413-421
```cpp
  }
  return false;
}

char VEAsmPrinter::ID = 0;

INITIALIZE_PASS(VEAsmPrinter, "ve-asm-printer", "VE Assembly Printer", false,
                false)

```
- **EN**: Implements logic around `INITIALIZE_PASS`; this block returns target-specific results.
- **CN**: 围绕 `INITIALIZE_PASS` 实现具体逻辑；这一段返回目标相关结果。

### Lines 422-425
```cpp
// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeVEAsmPrinter() {
  RegisterAsmPrinter<VEAsmPrinter> X(getTheVETarget());
}
```
- **EN**: Implements logic around `X`.
- **CN**: 围绕 `X` 实现具体逻辑。

## Key Concepts / 关键概念

- **Assembly emission / 汇编发射**:
  - **EN**: Lowers machine instructions to printable/emittable MC form
  - **CN**: 把机器指令降低为可打印/可发射的 MC 形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/VEInstPrinter.h`, `MCTargetDesc/VEMCAsmInfo.h`, `MCTargetDesc/VETargetStreamer.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `VEInstrInfo.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineModuleInfoImpls.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCAsmInfo.h` ... (+7 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR, Support
