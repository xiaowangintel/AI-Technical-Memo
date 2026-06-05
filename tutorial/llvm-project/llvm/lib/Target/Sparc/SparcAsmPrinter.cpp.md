# SparcAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcAsmPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Lowers MachineInstr to the MC layer and emits target-specific assembly/object output.
  - **CN**: 把 MachineInstr 降到 MC 层，并输出目标相关的汇编/目标文件内容。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- SparcAsmPrinter.cpp - Sparc LLVM assembly writer ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a printer that converts from our internal representation
// of machine-dependent LLVM code to GAS-format SPARC assembly language.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-30
```cpp

#include "MCTargetDesc/SparcInstPrinter.h"
#include "MCTargetDesc/SparcMCAsmInfo.h"
#include "MCTargetDesc/SparcMCTargetDesc.h"
#include "MCTargetDesc/SparcTargetStreamer.h"
#include "Sparc.h"
#include "SparcInstrInfo.h"
#include "SparcTargetMachine.h"
#include "TargetInfo/SparcTargetInfo.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineModuleInfoImpls.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/IR/Mangler.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/SparcInstPrinter.h`, `MCTargetDesc/SparcMCAsmInfo.h`, `MCTargetDesc/SparcMCTargetDesc.h`, `MCTargetDesc/SparcTargetStreamer.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/SparcInstPrinter.h`, `MCTargetDesc/SparcMCAsmInfo.h`, `MCTargetDesc/SparcMCTargetDesc.h`, `MCTargetDesc/SparcTargetStreamer.h`。

### Lines 31-40
```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

#define DEBUG_TYPE "asm-printer"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCInst.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCInst.h`, `llvm/MC/MCStreamer.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/TargetRegistry.h`。

### Lines 41-52
```cpp
namespace {
class SparcAsmPrinter : public AsmPrinter {
  SparcTargetStreamer &getTargetStreamer() {
    return static_cast<SparcTargetStreamer &>(
        *OutStreamer->getTargetStreamer());
  }

public:
  explicit SparcAsmPrinter(TargetMachine &TM,
                           std::unique_ptr<MCStreamer> Streamer)
      : AsmPrinter(TM, std::move(Streamer), ID) {}

```
- **EN**: Introduces declarations for `SparcAsmPrinter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SparcAsmPrinter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 53-64
```cpp
  StringRef getPassName() const override { return "Sparc Assembly Printer"; }

  void printOperand(const MachineInstr *MI, int opNum, raw_ostream &OS);
  void printMemOperand(const MachineInstr *MI, int opNum, raw_ostream &OS);

  void emitFunctionBodyStart() override;
  void emitInstruction(const MachineInstr *MI) override;

  static const char *getRegisterName(MCRegister Reg) {
    return SparcInstPrinter::getRegisterName(Reg);
  }

```
- **EN**: Implements logic around `getPassName`, `printOperand`, `printMemOperand`, `emitFunctionBodyStart`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getPassName`, `printOperand`, `printMemOperand`, `emitFunctionBodyStart`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 65-74
```cpp
  bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                       const char *ExtraCode, raw_ostream &O) override;
  bool PrintAsmMemoryOperand(const MachineInstr *MI, unsigned OpNo,
                             const char *ExtraCode, raw_ostream &O) override;

  void LowerGETPCXAndEmitMCInsts(const MachineInstr *MI,
                                 const MCSubtargetInfo &STI);

  MCOperand lowerOperand(const MachineOperand &MO) const;

```
- **EN**: Implements logic around `PrintAsmOperand`, `PrintAsmMemoryOperand`, `LowerGETPCXAndEmitMCInsts`, `lowerOperand`; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `PrintAsmOperand`, `PrintAsmMemoryOperand`, `LowerGETPCXAndEmitMCInsts`, `lowerOperand` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 75-92
```cpp
private:
  void lowerToMCInst(const MachineInstr *MI, MCInst &OutMI);

public:
  static char ID;
};
} // end of anonymous namespace

static MCOperand createSparcMCOperand(uint16_t Kind, MCSymbol *Sym,
                                      MCContext &OutContext) {
  const MCSymbolRefExpr *MCSym = MCSymbolRefExpr::create(Sym, OutContext);
  auto *expr = MCSpecifierExpr::create(MCSym, Kind, OutContext);
  return MCOperand::createExpr(expr);
}
static MCOperand createPCXCallOP(MCSymbol *Label,
                                 MCContext &OutContext) {
  return MCOperand::createExpr(MCSymbolRefExpr::create(Label, OutContext));
}
```
- **EN**: Implements logic around `lowerToMCInst`, `createSparcMCOperand`, `create`, `createExpr`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `lowerToMCInst`, `createSparcMCOperand`, `create`, `createExpr`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 93-102
```cpp

static MCOperand createPCXRelExprOp(uint16_t Spec, MCSymbol *GOTLabel,
                                    MCSymbol *StartLabel, MCSymbol *CurLabel,
                                    MCContext &OutContext) {
  const MCSymbolRefExpr *GOT = MCSymbolRefExpr::create(GOTLabel, OutContext);
  const MCSymbolRefExpr *Start = MCSymbolRefExpr::create(StartLabel,
                                                         OutContext);
  const MCSymbolRefExpr *Cur = MCSymbolRefExpr::create(CurLabel,
                                                       OutContext);

```
- **EN**: Implements logic around `createPCXRelExprOp`, `create`; this block works at the MC layer.
- **CN**: 围绕 `createPCXRelExprOp`, `create` 实现具体逻辑；这一段工作在 MC 层。

### Lines 103-118
```cpp
  const MCBinaryExpr *Sub = MCBinaryExpr::createSub(Cur, Start, OutContext);
  const MCBinaryExpr *Add = MCBinaryExpr::createAdd(GOT, Sub, OutContext);
  auto *expr = MCSpecifierExpr::create(Add, Spec, OutContext);
  return MCOperand::createExpr(expr);
}

static void EmitCall(MCStreamer &OutStreamer,
                     MCOperand &Callee,
                     const MCSubtargetInfo &STI)
{
  MCInst CallInst;
  CallInst.setOpcode(SP::CALL);
  CallInst.addOperand(Callee);
  OutStreamer.emitInstruction(CallInst, STI);
}

```
- **EN**: Implements logic around `createSub`, `createAdd`, `create`, `createExpr`, ...; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createSub`, `createAdd`, `create`, `createExpr`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 119-127
```cpp
static void EmitRDPC(MCStreamer &OutStreamer, MCOperand &RD,
                     const MCSubtargetInfo &STI) {
  MCInst RDPCInst;
  RDPCInst.setOpcode(SP::RDASR);
  RDPCInst.addOperand(RD);
  RDPCInst.addOperand(MCOperand::createReg(SP::ASR5));
  OutStreamer.emitInstruction(RDPCInst, STI);
}

```
- **EN**: Implements logic around `EmitRDPC`, `setOpcode`, `addOperand`, `emitInstruction`; this block works at the MC layer.
- **CN**: 围绕 `EmitRDPC`, `setOpcode`, `addOperand`, `emitInstruction` 实现具体逻辑；这一段工作在 MC 层。

### Lines 128-138
```cpp
static void EmitSETHI(MCStreamer &OutStreamer,
                      MCOperand &Imm, MCOperand &RD,
                      const MCSubtargetInfo &STI)
{
  MCInst SETHIInst;
  SETHIInst.setOpcode(SP::SETHIi);
  SETHIInst.addOperand(RD);
  SETHIInst.addOperand(Imm);
  OutStreamer.emitInstruction(SETHIInst, STI);
}

```
- **EN**: Implements logic around `EmitSETHI`, `setOpcode`, `addOperand`, `emitInstruction`; this block works at the MC layer.
- **CN**: 围绕 `EmitSETHI`, `setOpcode`, `addOperand`, `emitInstruction` 实现具体逻辑；这一段工作在 MC 层。

### Lines 139-150
```cpp
static void EmitBinary(MCStreamer &OutStreamer, unsigned Opcode,
                       MCOperand &RS1, MCOperand &Src2, MCOperand &RD,
                       const MCSubtargetInfo &STI)
{
  MCInst Inst;
  Inst.setOpcode(Opcode);
  Inst.addOperand(RD);
  Inst.addOperand(RS1);
  Inst.addOperand(Src2);
  OutStreamer.emitInstruction(Inst, STI);
}

```
- **EN**: Implements logic around `EmitBinary`, `setOpcode`, `addOperand`, `emitInstruction`; this block works at the MC layer.
- **CN**: 围绕 `EmitBinary`, `setOpcode`, `addOperand`, `emitInstruction` 实现具体逻辑；这一段工作在 MC 层。

### Lines 151-162
```cpp
static void EmitOR(MCStreamer &OutStreamer,
                   MCOperand &RS1, MCOperand &Imm, MCOperand &RD,
                   const MCSubtargetInfo &STI) {
  EmitBinary(OutStreamer, SP::ORri, RS1, Imm, RD, STI);
}

static void EmitADD(MCStreamer &OutStreamer,
                    MCOperand &RS1, MCOperand &RS2, MCOperand &RD,
                    const MCSubtargetInfo &STI) {
  EmitBinary(OutStreamer, SP::ADDrr, RS1, RS2, RD, STI);
}

```
- **EN**: Implements logic around `EmitOR`, `EmitBinary`, `EmitADD`.
- **CN**: 围绕 `EmitOR`, `EmitBinary`, `EmitADD` 实现具体逻辑。

### Lines 163-177
```cpp
static void EmitSHL(MCStreamer &OutStreamer,
                    MCOperand &RS1, MCOperand &Imm, MCOperand &RD,
                    const MCSubtargetInfo &STI) {
  EmitBinary(OutStreamer, SP::SLLri, RS1, Imm, RD, STI);
}

static void emitHiLo(MCStreamer &OutStreamer, MCSymbol *GOTSym, uint16_t HiKind,
                     uint16_t LoKind, MCOperand &RD, MCContext &OutContext,
                     const MCSubtargetInfo &STI) {
  MCOperand hi = createSparcMCOperand(HiKind, GOTSym, OutContext);
  MCOperand lo = createSparcMCOperand(LoKind, GOTSym, OutContext);
  EmitSETHI(OutStreamer, hi, RD, STI);
  EmitOR(OutStreamer, RD, lo, RD, STI);
}

```
- **EN**: Implements logic around `EmitSHL`, `EmitBinary`, `emitHiLo`, `createSparcMCOperand`, ...; this block works at the MC layer.
- **CN**: 围绕 `EmitSHL`, `EmitBinary`, `emitHiLo`, `createSparcMCOperand`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 178-187
```cpp
void SparcAsmPrinter::LowerGETPCXAndEmitMCInsts(const MachineInstr *MI,
                                                const MCSubtargetInfo &STI)
{
  MCSymbol *GOTLabel   =
    OutContext.getOrCreateSymbol(Twine("_GLOBAL_OFFSET_TABLE_"));

  const MachineOperand &MO = MI->getOperand(0);
  assert(MO.getReg() != SP::O7 &&
         "%o7 is assigned as destination for getpcx!");

```
- **EN**: Implements logic around `LowerGETPCXAndEmitMCInsts`, `getOrCreateSymbol`, `getOperand`, `assert`; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `LowerGETPCXAndEmitMCInsts`, `getOrCreateSymbol`, `getOperand`, `assert` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 188-205
```cpp
  MCOperand MCRegOP = MCOperand::createReg(MO.getReg());


  if (!isPositionIndependent()) {
    // Just load the address of GOT to MCRegOP.
    switch(TM.getCodeModel()) {
    default:
      llvm_unreachable("Unsupported absolute code model");
    case CodeModel::Small:
      emitHiLo(*OutStreamer, GOTLabel, ELF::R_SPARC_HI22, ELF::R_SPARC_LO10,
               MCRegOP, OutContext, STI);
      break;
    case CodeModel::Medium: {
      emitHiLo(*OutStreamer, GOTLabel, ELF::R_SPARC_H44, ELF::R_SPARC_M44,
               MCRegOP, OutContext, STI);
      MCOperand imm = MCOperand::createExpr(MCConstantExpr::create(12,
                                                                   OutContext));
      EmitSHL(*OutStreamer, MCRegOP, imm, MCRegOP, STI);
```
- **EN**: Implements logic around `createReg`, `llvm_unreachable`, `emitHiLo`, `createExpr`, ...; this block uses `switch`-based dispatch; applies conditional target rules; maps fixups or relocations.
- **CN**: 围绕 `createReg`, `llvm_unreachable`, `emitHiLo`, `createExpr`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，映射 fixup 或重定位。

### Lines 206-223
```cpp
      MCOperand lo =
          createSparcMCOperand(ELF::R_SPARC_L44, GOTLabel, OutContext);
      EmitOR(*OutStreamer, MCRegOP, lo, MCRegOP, STI);
      break;
    }
    case CodeModel::Large: {
      emitHiLo(*OutStreamer, GOTLabel, ELF::R_SPARC_HH22, ELF::R_SPARC_HM10,
               MCRegOP, OutContext, STI);
      MCOperand imm = MCOperand::createExpr(MCConstantExpr::create(32,
                                                                   OutContext));
      EmitSHL(*OutStreamer, MCRegOP, imm, MCRegOP, STI);
      // Use register %o7 to load the lower 32 bits.
      MCOperand RegO7 = MCOperand::createReg(SP::O7);
      emitHiLo(*OutStreamer, GOTLabel, ELF::R_SPARC_HI22, ELF::R_SPARC_LO10,
               RegO7, OutContext, STI);
      EmitADD(*OutStreamer, MCRegOP, RegO7, MCRegOP, STI);
    }
    }
```
- **EN**: Implements logic around `createSparcMCOperand`, `EmitOR`, `emitHiLo`, `createExpr`, ...; this block maps fixups or relocations.
- **CN**: 围绕 `createSparcMCOperand`, `EmitOR`, `emitHiLo`, `createExpr`, ... 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 224-232
```cpp
    return;
  }

  MCSymbol *StartLabel = OutContext.createTempSymbol();
  MCSymbol *EndLabel   = OutContext.createTempSymbol();
  MCSymbol *SethiLabel = OutContext.createTempSymbol();

  MCOperand RegO7   = MCOperand::createReg(SP::O7);

```
- **EN**: Implements logic around `createTempSymbol`, `createReg`.
- **CN**: 围绕 `createTempSymbol`, `createReg` 实现具体逻辑。

### Lines 233-250
```cpp
  // <StartLabel>:
  //   <GET-PC> // This will be either `call <EndLabel>` or `rd %pc, %o7`.
  // <SethiLabel>:
  //     sethi %hi(_GLOBAL_OFFSET_TABLE_+(<SethiLabel>-<StartLabel>)), <MO>
  // <EndLabel>:
  //   or  <MO>, %lo(_GLOBAL_OFFSET_TABLE_+(<EndLabel>-<StartLabel>))), <MO>
  //   add <MO>, %o7, <MO>

  OutStreamer->emitLabel(StartLabel);
  if (!STI.getTargetTriple().isSPARC64() ||
      STI.hasFeature(Sparc::TuneSlowRDPC)) {
    MCOperand Callee = createPCXCallOP(EndLabel, OutContext);
    EmitCall(*OutStreamer, Callee, STI);
  } else {
    // TODO find out whether it is possible to store PC
    // in other registers, to enable leaf function optimization.
    // (On the other hand, approx. over 97.8% of GETPCXes happen
    // in non-leaf functions, so would this be worth the effort?)
```
- **EN**: Implements logic around `emitLabel`, `hasFeature`, `createPCXCallOP`, `EmitCall`; this block applies conditional target rules.
- **CN**: 围绕 `emitLabel`, `hasFeature`, `createPCXCallOP`, `EmitCall` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 251-263
```cpp
    EmitRDPC(*OutStreamer, RegO7, STI);
  }
  OutStreamer->emitLabel(SethiLabel);
  MCOperand hiImm = createPCXRelExprOp(ELF::R_SPARC_PC22, GOTLabel, StartLabel,
                                       SethiLabel, OutContext);
  EmitSETHI(*OutStreamer, hiImm, MCRegOP, STI);
  OutStreamer->emitLabel(EndLabel);
  MCOperand loImm = createPCXRelExprOp(ELF::R_SPARC_PC10, GOTLabel, StartLabel,
                                       EndLabel, OutContext);
  EmitOR(*OutStreamer, MCRegOP, loImm, MCRegOP, STI);
  EmitADD(*OutStreamer, MCRegOP, RegO7, MCRegOP, STI);
}

```
- **EN**: Implements logic around `EmitRDPC`, `emitLabel`, `createPCXRelExprOp`, `EmitSETHI`, ...; this block maps fixups or relocations.
- **CN**: 围绕 `EmitRDPC`, `emitLabel`, `createPCXRelExprOp`, `EmitSETHI`, ... 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 264-273
```cpp
MCOperand SparcAsmPrinter::lowerOperand(const MachineOperand &MO) const {
  switch (MO.getType()) {
  default:
    llvm_unreachable("unknown operand type");
    break;
  case MachineOperand::MO_Register:
    if (MO.isImplicit())
      break;
    return MCOperand::createReg(MO.getReg());

```
- **EN**: Implements logic around `lowerOperand`, `llvm_unreachable`, `createReg`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `lowerOperand`, `llvm_unreachable`, `createReg` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 274-291
```cpp
  case MachineOperand::MO_Immediate:
    return MCOperand::createImm(MO.getImm());

  case MachineOperand::MO_MachineBasicBlock:
  case MachineOperand::MO_GlobalAddress:
  case MachineOperand::MO_BlockAddress:
  case MachineOperand::MO_ExternalSymbol:
  case MachineOperand::MO_ConstantPoolIndex: {
    auto RelType = MO.getTargetFlags();
    const MCSymbol *Symbol = nullptr;
    switch (MO.getType()) {
    default:
      llvm_unreachable("");
    case MachineOperand::MO_MachineBasicBlock:
      Symbol = MO.getMBB()->getSymbol();
      break;
    case MachineOperand::MO_GlobalAddress:
      Symbol = getSymbol(MO.getGlobal());
```
- **EN**: Implements logic around `createImm`, `getTargetFlags`, `llvm_unreachable`, `getMBB`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createImm`, `getTargetFlags`, `llvm_unreachable`, `getMBB`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 292-303
```cpp
      break;
    case MachineOperand::MO_BlockAddress:
      Symbol = GetBlockAddressSymbol(MO.getBlockAddress());
      break;
    case MachineOperand::MO_ExternalSymbol:
      Symbol = GetExternalSymbolSymbol(MO.getSymbolName());
      break;
    case MachineOperand::MO_ConstantPoolIndex:
      Symbol = GetCPISymbol(MO.getIndex());
      break;
    }

```
- **EN**: Implements logic around `GetBlockAddressSymbol`, `GetExternalSymbolSymbol`, `GetCPISymbol`.
- **CN**: 围绕 `GetBlockAddressSymbol`, `GetExternalSymbolSymbol`, `GetCPISymbol` 实现具体逻辑。

### Lines 304-315
```cpp
    const MCExpr *expr = MCSymbolRefExpr::create(Symbol, OutContext);
    if (RelType)
      expr = MCSpecifierExpr::create(expr, RelType, OutContext);
    return MCOperand::createExpr(expr);
  }

  case MachineOperand::MO_RegisterMask:
    break;
  }
  return MCOperand();
}

```
- **EN**: Implements logic around `create`, `createExpr`, `MCOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `create`, `createExpr`, `MCOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 316-325
```cpp
void SparcAsmPrinter::lowerToMCInst(const MachineInstr *MI, MCInst &OutMI) {
  OutMI.setOpcode(MI->getOpcode());

  for (const MachineOperand &MO : MI->operands()) {
    MCOperand MCOp = lowerOperand(MO);
    if (MCOp.isValid())
      OutMI.addOperand(MCOp);
  }
}

```
- **EN**: Implements logic around `lowerToMCInst`, `setOpcode`, `lowerOperand`, `addOperand`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `lowerToMCInst`, `setOpcode`, `lowerOperand`, `addOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 326-338
```cpp
void SparcAsmPrinter::emitInstruction(const MachineInstr *MI) {
  Sparc_MC::verifyInstructionPredicates(MI->getOpcode(),
                                        getSubtargetInfo().getFeatureBits());
  if (MI->isBundle()) {
    const MachineBasicBlock *MBB = MI->getParent();
    MachineBasicBlock::const_instr_iterator I = ++MI->getIterator();
    while (I != MBB->instr_end() && I->isInsideBundle()) {
      emitInstruction(&*I);
      ++I;
    }
    return;
  }

```
- **EN**: Implements logic around `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `getParent`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitInstruction`, `verifyInstructionPredicates`, `getSubtargetInfo`, `getParent`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 339-356
```cpp
  switch (MI->getOpcode()) {
  default: break;
  case TargetOpcode::DBG_VALUE:
    // FIXME: Debug Value.
    return;
  case SP::CASArr:
  case SP::SWAPrr:
  case SP::SWAPri:
    if (MF->getSubtarget<SparcSubtarget>().fixTN0011())
      OutStreamer->emitCodeAlignment(Align(16), &getSubtargetInfo());
    break;
  case SP::GETPCX:
    LowerGETPCXAndEmitMCInsts(MI, getSubtargetInfo());
    return;
  }
  MachineBasicBlock::const_instr_iterator I = MI->getIterator();
  MachineBasicBlock::const_instr_iterator E = MI->getParent()->instr_end();
  do {
```
- **EN**: Implements logic around `emitCodeAlignment`, `LowerGETPCXAndEmitMCInsts`, `getIterator`, `getParent`; this block uses `switch`-based dispatch; applies conditional target rules; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `emitCodeAlignment`, `LowerGETPCXAndEmitMCInsts`, `getIterator`, `getParent` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 357-366
```cpp
    MCInst TmpInst;
    lowerToMCInst(&*I, TmpInst);
    EmitToStreamer(*OutStreamer, TmpInst);
  } while ((++I != E) && I->isInsideBundle()); // Delay slot check.
}

void SparcAsmPrinter::emitFunctionBodyStart() {
  if (!MF->getSubtarget<SparcSubtarget>().is64Bit())
    return;

```
- **EN**: Implements logic around `lowerToMCInst`, `EmitToStreamer`, `emitFunctionBodyStart`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `lowerToMCInst`, `EmitToStreamer`, `emitFunctionBodyStart` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 367-380
```cpp
  const MachineRegisterInfo &MRI = MF->getRegInfo();
  const unsigned globalRegs[] = { SP::G2, SP::G3, SP::G6, SP::G7, 0 };
  for (unsigned i = 0; globalRegs[i] != 0; ++i) {
    unsigned reg = globalRegs[i];
    if (MRI.use_empty(reg))
      continue;

    if  (reg == SP::G6 || reg == SP::G7)
      getTargetStreamer().emitSparcRegisterIgnore(reg);
    else
      getTargetStreamer().emitSparcRegisterScratch(reg);
  }
}

```
- **EN**: Implements logic around `getRegInfo`, `getTargetStreamer`; this block applies conditional target rules.
- **CN**: 围绕 `getRegInfo`, `getTargetStreamer` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 381-389
```cpp
void SparcAsmPrinter::printOperand(const MachineInstr *MI, int opNum,
                                   raw_ostream &O) {
  const DataLayout &DL = getDataLayout();
  const MachineOperand &MO = MI->getOperand(opNum);
  switch (MO.getType()) {
  case MachineOperand::MO_Register:
    O << "%" << StringRef(getRegisterName(MO.getReg())).lower();
    break;

```
- **EN**: Implements logic around `printOperand`, `getDataLayout`, `getOperand`, `StringRef`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `getDataLayout`, `getOperand`, `StringRef` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 390-407
```cpp
  case MachineOperand::MO_Immediate:
    O << MO.getImm();
    break;
  case MachineOperand::MO_MachineBasicBlock:
    MO.getMBB()->getSymbol()->print(O, MAI);
    return;
  case MachineOperand::MO_GlobalAddress:
    PrintSymbolOperand(MO, O);
    break;
  case MachineOperand::MO_BlockAddress:
    O <<  GetBlockAddressSymbol(MO.getBlockAddress())->getName();
    break;
  case MachineOperand::MO_ExternalSymbol:
    O << MO.getSymbolName();
    break;
  case MachineOperand::MO_ConstantPoolIndex:
    O << DL.getInternalSymbolPrefix() << "CPI" << getFunctionNumber() << "_"
      << MO.getIndex();
```
- **EN**: Implements logic around `getImm`, `getMBB`, `PrintSymbolOperand`, `GetBlockAddressSymbol`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getImm`, `getMBB`, `PrintSymbolOperand`, `GetBlockAddressSymbol`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 408-416
```cpp
    break;
  case MachineOperand::MO_Metadata:
    MO.getMetadata()->printAsOperand(O, MMI->getModule());
    break;
  default:
    llvm_unreachable("<unknown operand type>");
  }
}

```
- **EN**: Implements logic around `getMetadata`, `llvm_unreachable`.
- **CN**: 围绕 `getMetadata`, `llvm_unreachable` 实现具体逻辑。

### Lines 417-427
```cpp
void SparcAsmPrinter::printMemOperand(const MachineInstr *MI, int opNum,
                                      raw_ostream &O) {
  printOperand(MI, opNum, O);

  if (MI->getOperand(opNum+1).isReg() &&
      MI->getOperand(opNum+1).getReg() == SP::G0)
    return;   // don't print "+%g0"
  if (MI->getOperand(opNum+1).isImm() &&
      MI->getOperand(opNum+1).getImm() == 0)
    return;   // don't print "+0"

```
- **EN**: Implements logic around `printMemOperand`, `printOperand`, `getOperand`; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printMemOperand`, `printOperand`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 428-439
```cpp
  O << "+";
  printOperand(MI, opNum+1, O);
}

/// PrintAsmOperand - Print out an operand for an inline asm expression.
///
bool SparcAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                                      const char *ExtraCode,
                                      raw_ostream &O) {
  if (ExtraCode && ExtraCode[0]) {
    if (ExtraCode[1] != 0) return true; // Unknown modifier.

```
- **EN**: Implements logic around `printOperand`, `PrintAsmOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `PrintAsmOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 440-451
```cpp
    switch (ExtraCode[0]) {
    default:
      // See if this is a generic print operand
      return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
    case 'L': // Low order register of a twin word register operand
    case 'H': // High order register of a twin word register operand
    {
      const SparcSubtarget &Subtarget = MF->getSubtarget<SparcSubtarget>();
      const MachineOperand &MO = MI->getOperand(OpNo);
      const SparcRegisterInfo *RegisterInfo = Subtarget.getRegisterInfo();
      Register MOReg = MO.getReg();

```
- **EN**: Implements logic around `PrintAsmOperand`, `getSubtarget<SparcSubtarget>`, `getOperand`, `getRegisterInfo`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `PrintAsmOperand`, `getSubtarget<SparcSubtarget>`, `getOperand`, `getRegisterInfo`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 452-469
```cpp
      Register HiReg, LoReg;
      if (!SP::IntPairRegClass.contains(MOReg)) {
        // If we aren't given a register pair already, find out which pair it
        // belongs to. Note that here, the specified register operand, which
        // refers to the high part of the twinword, needs to be an even-numbered
        // register.
        MOReg = RegisterInfo->getMatchingSuperReg(MOReg, SP::sub_even,
                                                  &SP::IntPairRegClass);
        if (!MOReg) {
          SMLoc Loc;
          OutContext.reportError(
              Loc, "Hi part of pair should point to an even-numbered register");
          OutContext.reportError(
              Loc, "(note that in some cases it might be necessary to manually "
                   "bind the input/output registers instead of relying on "
                   "automatic allocation)");
          return true;
        }
```
- **EN**: Implements logic around `getMatchingSuperReg`, `reportError`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getMatchingSuperReg`, `reportError` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 470-484
```cpp
      }

      HiReg = RegisterInfo->getSubReg(MOReg, SP::sub_even);
      LoReg = RegisterInfo->getSubReg(MOReg, SP::sub_odd);

      Register Reg;
      switch (ExtraCode[0]) {
      case 'L':
        Reg = LoReg;
        break;
      case 'H':
        Reg = HiReg;
        break;
      }

```
- **EN**: Implements logic around `getSubReg`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `getSubReg` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 485-493
```cpp
      O << '%' << SparcInstPrinter::getRegisterName(Reg);
      return false;
    }
    case 'f':
    case 'r':
     break;
    }
  }

```
- **EN**: Implements logic around `getRegisterName`; this block returns target-specific results.
- **CN**: 围绕 `getRegisterName` 实现具体逻辑；这一段返回目标相关结果。

### Lines 494-505
```cpp
  printOperand(MI, OpNo, O);

  return false;
}

bool SparcAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
                                            unsigned OpNo,
                                            const char *ExtraCode,
                                            raw_ostream &O) {
  if (ExtraCode && ExtraCode[0])
    return true;  // Unknown modifier

```
- **EN**: Implements logic around `printOperand`, `PrintAsmMemoryOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `printOperand`, `PrintAsmMemoryOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 506-514
```cpp
  O << '[';
  printMemOperand(MI, OpNo, O);
  O << ']';

  return false;
}

char SparcAsmPrinter::ID = 0;

```
- **EN**: Implements logic around `printMemOperand`; this block returns target-specific results.
- **CN**: 围绕 `printMemOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 515-524
```cpp
INITIALIZE_PASS(SparcAsmPrinter, "sparc-asm-printer", "Sparc Assembly Printer",
                false, false)

// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeSparcAsmPrinter() {
  RegisterAsmPrinter<SparcAsmPrinter> X(getTheSparcTarget());
  RegisterAsmPrinter<SparcAsmPrinter> Y(getTheSparcV9Target());
  RegisterAsmPrinter<SparcAsmPrinter> Z(getTheSparcelTarget());
}
```
- **EN**: Implements logic around `INITIALIZE_PASS`, `X`, `Y`, `Z`.
- **CN**: 围绕 `INITIALIZE_PASS`, `X`, `Y`, `Z` 实现具体逻辑。

## Key Concepts / 关键概念

- **Assembly emission / 汇编发射**:
  - **EN**: Lowers machine instructions to printable/emittable MC form
  - **CN**: 把机器指令降低为可打印/可发射的 MC 形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/SparcInstPrinter.h`, `MCTargetDesc/SparcMCAsmInfo.h`, `MCTargetDesc/SparcMCTargetDesc.h`, `MCTargetDesc/SparcTargetStreamer.h`, `Sparc.h`, `SparcInstrInfo.h`, `SparcTargetMachine.h`, `TargetInfo/SparcTargetInfo.h`, `llvm/BinaryFormat/ELF.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineModuleInfoImpls.h` ... (+11 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR, Support
