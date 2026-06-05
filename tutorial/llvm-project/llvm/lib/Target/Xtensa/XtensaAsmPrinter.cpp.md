# XtensaAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaAsmPrinter.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Lowers MachineInstr to the MC layer and emits target-specific assembly/object output.
  - **CN**: 把 MachineInstr 降到 MC 层，并输出目标相关的汇编/目标文件内容。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- XtensaAsmPrinter.cpp Xtensa LLVM Assembly Printer ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a printer that converts from our internal representation
// of machine-dependent LLVM code to GAS-format Xtensa assembly language.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-30
```cpp

#include "XtensaAsmPrinter.h"
#include "MCTargetDesc/XtensaInstPrinter.h"
#include "MCTargetDesc/XtensaMCAsmInfo.h"
#include "MCTargetDesc/XtensaTargetStreamer.h"
#include "TargetInfo/XtensaTargetInfo.h"
#include "XtensaConstantPoolValue.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineModuleInfoImpls.h"
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInstBuilder.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCSymbolELF.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaAsmPrinter.h`, `MCTargetDesc/XtensaInstPrinter.h`, `MCTargetDesc/XtensaMCAsmInfo.h`, `MCTargetDesc/XtensaTargetStreamer.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaAsmPrinter.h`, `MCTargetDesc/XtensaInstPrinter.h`, `MCTargetDesc/XtensaMCAsmInfo.h`, `MCTargetDesc/XtensaTargetStreamer.h`。

### Lines 31-45
```cpp
#include "llvm/MC/TargetRegistry.h"

using namespace llvm;

static Xtensa::Specifier
getModifierSpecifier(XtensaCP::XtensaCPModifier Modifier) {
  switch (Modifier) {
  case XtensaCP::no_modifier:
    return Xtensa::S_None;
  case XtensaCP::TPOFF:
    return Xtensa::S_TPOFF;
  }
  report_fatal_error("Invalid XtensaCPModifier!");
}

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/TargetRegistry.h`。

### Lines 46-62
```cpp
void XtensaAsmPrinter::emitInstruction(const MachineInstr *MI) {
  unsigned Opc = MI->getOpcode();

  switch (Opc) {
  case Xtensa::BR_JT:
    EmitToStreamer(
        *OutStreamer,
        MCInstBuilder(Xtensa::JX).addReg(MI->getOperand(0).getReg()));
    return;
  default:
    MCInst LoweredMI;
    lowerToMCInst(MI, LoweredMI);
    EmitToStreamer(*OutStreamer, LoweredMI);
    return;
  }
}

```
- **EN**: Implements logic around `emitInstruction`, `getOpcode`, `EmitToStreamer`, `MCInstBuilder`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `emitInstruction`, `getOpcode`, `EmitToStreamer`, `MCInstBuilder`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 63-80
```cpp
void XtensaAsmPrinter::emitMachineConstantPoolValue(
    MachineConstantPoolValue *MCPV) {
  XtensaConstantPoolValue *XtensaCPV =
      static_cast<XtensaConstantPoolValue *>(MCPV);
  MCSymbol *MCSym;

  if (XtensaCPV->isBlockAddress()) {
    const BlockAddress *BA =
        cast<XtensaConstantPoolConstant>(XtensaCPV)->getBlockAddress();
    MCSym = GetBlockAddressSymbol(BA);
  } else if (XtensaCPV->isMachineBasicBlock()) {
    const MachineBasicBlock *MBB =
        cast<XtensaConstantPoolMBB>(XtensaCPV)->getMBB();
    MCSym = MBB->getSymbol();
  } else if (XtensaCPV->isJumpTable()) {
    unsigned Idx = cast<XtensaConstantPoolJumpTable>(XtensaCPV)->getIndex();
    MCSym = this->GetJTISymbol(Idx, false);
  } else {
```
- **EN**: Implements logic around `emitMachineConstantPoolValue`, `cast<XtensaConstantPoolConstant>`, `GetBlockAddressSymbol`, `cast<XtensaConstantPoolMBB>`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `emitMachineConstantPoolValue`, `cast<XtensaConstantPoolConstant>`, `GetBlockAddressSymbol`, `cast<XtensaConstantPoolMBB>`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 81-94
```cpp
    assert(XtensaCPV->isExtSymbol() && "unrecognized constant pool value");
    XtensaConstantPoolSymbol *XtensaSym =
        cast<XtensaConstantPoolSymbol>(XtensaCPV);
    const char *SymName = XtensaSym->getSymbol();

    if (XtensaSym->isPrivateLinkage()) {
      const DataLayout &DL = getDataLayout();
      MCSym = OutContext.getOrCreateSymbol(Twine(DL.getInternalSymbolPrefix()) +
                                           SymName);
    } else {
      MCSym = OutContext.getOrCreateSymbol(SymName);
    }
  }

```
- **EN**: Implements logic around `assert`, `cast<XtensaConstantPoolSymbol>`, `getSymbol`, `getDataLayout`, ...; this block applies conditional target rules.
- **CN**: 围绕 `assert`, `cast<XtensaConstantPoolSymbol>`, `getSymbol`, `getDataLayout`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 95-106
```cpp
  MCSymbol *LblSym = GetCPISymbol(XtensaCPV->getLabelId());
  auto *TS =
      static_cast<XtensaTargetStreamer *>(OutStreamer->getTargetStreamer());
  auto Spec = getModifierSpecifier(XtensaCPV->getModifier());

  if (XtensaCPV->getModifier() != XtensaCP::no_modifier) {
    std::string SymName(MCSym->getName());
    StringRef Modifier = XtensaCPV->getModifierText();
    SymName += Modifier;
    MCSym = OutContext.getOrCreateSymbol(SymName);
  }

```
- **EN**: Implements logic around `GetCPISymbol`, `getTargetStreamer`, `getModifierSpecifier`, `SymName`, ...; this block applies conditional target rules.
- **CN**: 围绕 `GetCPISymbol`, `getTargetStreamer`, `getModifierSpecifier`, `SymName`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 107-124
```cpp
  const MCExpr *Expr = MCSymbolRefExpr::create(MCSym, Spec, OutContext);
  TS->emitLiteral(LblSym, Expr, false);
}

void XtensaAsmPrinter::emitMachineConstantPoolEntry(
    const MachineConstantPoolEntry &CPE, int i) {
  if (CPE.isMachineConstantPoolEntry()) {
    XtensaConstantPoolValue *XtensaCPV =
        static_cast<XtensaConstantPoolValue *>(CPE.Val.MachineCPVal);
    XtensaCPV->setLabelId(i);
    emitMachineConstantPoolValue(CPE.Val.MachineCPVal);
  } else {
    MCSymbol *LblSym = GetCPISymbol(i);
    auto *TS =
        static_cast<XtensaTargetStreamer *>(OutStreamer->getTargetStreamer());
    const Constant *C = CPE.Val.ConstVal;
    const MCExpr *Value = nullptr;

```
- **EN**: Implements logic around `create`, `emitLiteral`, `emitMachineConstantPoolEntry`, `setLabelId`, ...; this block applies conditional target rules.
- **CN**: 围绕 `create`, `emitLiteral`, `emitMachineConstantPoolEntry`, `setLabelId`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 125-136
```cpp
    Type *Ty = C->getType();
    if (const auto *CFP = dyn_cast<ConstantFP>(C)) {
      Value = MCConstantExpr::create(
          CFP->getValueAPF().bitcastToAPInt().getSExtValue(), OutContext);
    } else if (const auto *CI = dyn_cast<ConstantInt>(C)) {
      Value = MCConstantExpr::create(CI->getValue().getSExtValue(), OutContext);
    } else if (isa<PointerType>(Ty)) {
      Value = lowerConstant(C);
    } else {
      llvm_unreachable("unexpected constant pool entry type");
    }

```
- **EN**: Implements logic around `getType`, `create`, `getValueAPF`, `lowerConstant`, ...; this block applies conditional target rules.
- **CN**: 围绕 `getType`, `create`, `getValueAPF`, `lowerConstant`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 137-151
```cpp
    TS->emitLiteral(LblSym, Value, false);
  }
}

// EmitConstantPool - Print to the current output stream assembly
// representations of the constants in the constant pool MCP. This is
// used to print out constants which have been "spilled to memory" by
// the code generator.
void XtensaAsmPrinter::emitConstantPool() {
  const Function &F = MF->getFunction();
  const MachineConstantPool *MCP = MF->getConstantPool();
  const std::vector<MachineConstantPoolEntry> &CP = MCP->getConstants();
  if (CP.empty())
    return;

```
- **EN**: Implements logic around `emitLiteral`, `emitConstantPool`, `getFunction`, `getConstantPool`, ...; this block applies conditional target rules.
- **CN**: 围绕 `emitLiteral`, `emitConstantPool`, `getFunction`, `getConstantPool`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 152-163
```cpp
  OutStreamer->pushSection();

  auto *TS =
      static_cast<XtensaTargetStreamer *>(OutStreamer->getTargetStreamer());
  MCSection *CS = getObjFileLowering().SectionForGlobal(&F, TM);
  TS->startLiteralSection(CS);

  int CPIdx = 0;
  for (const MachineConstantPoolEntry &CPE : CP) {
    emitMachineConstantPoolEntry(CPE, CPIdx++);
  }

```
- **EN**: Implements logic around `pushSection`, `getTargetStreamer`, `getObjFileLowering`, `startLiteralSection`, ....
- **CN**: 围绕 `pushSection`, `getTargetStreamer`, `getObjFileLowering`, `startLiteralSection`, ... 实现具体逻辑。

### Lines 164-181
```cpp
  OutStreamer->popSection();
}

void XtensaAsmPrinter::printOperand(const MachineInstr *MI, int OpNo,
                                    raw_ostream &O) {
  const MachineOperand &MO = MI->getOperand(OpNo);

  switch (MO.getType()) {
  case MachineOperand::MO_Register:
  case MachineOperand::MO_Immediate: {
    MCOperand MC = lowerOperand(MI->getOperand(OpNo));
    XtensaInstPrinter::printOperand(MC, O);
    break;
  }
  default:
    llvm_unreachable("unknown operand type");
  }
}
```
- **EN**: Implements logic around `popSection`, `printOperand`, `getOperand`, `lowerOperand`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `popSection`, `printOperand`, `getOperand`, `lowerOperand`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 182-190
```cpp

bool XtensaAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
                                       const char *ExtraCode, raw_ostream &O) {
  // Print the operand if there is no operand modifier.
  if (!ExtraCode || !ExtraCode[0]) {
    printOperand(MI, OpNo, O);
    return false;
  }

```
- **EN**: Implements logic around `PrintAsmOperand`, `printOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PrintAsmOperand`, `printOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 191-201
```cpp
  // Fallback to the default implementation.
  return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
}

bool XtensaAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
                                             unsigned OpNo,
                                             const char *ExtraCode,
                                             raw_ostream &OS) {
  if (ExtraCode && ExtraCode[0])
    return true; // Unknown modifier.

```
- **EN**: Implements logic around `PrintAsmOperand`, `PrintAsmMemoryOperand`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `PrintAsmOperand`, `PrintAsmMemoryOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 202-210
```cpp
  assert(OpNo + 1 < MI->getNumOperands() && "Insufficient operands");

  const MachineOperand &Base = MI->getOperand(OpNo);
  const MachineOperand &Offset = MI->getOperand(OpNo + 1);

  assert(Base.isReg() &&
         "Unexpected base pointer for inline asm memory operand.");
  assert(Offset.isImm() && "Unexpected offset for inline asm memory operand.");

```
- **EN**: Implements logic around `assert`, `getOperand`.
- **CN**: 围绕 `assert`, `getOperand` 实现具体逻辑。

### Lines 211-223
```cpp
  OS << XtensaInstPrinter::getRegisterName(Base.getReg());
  OS << ", ";
  OS << Offset.getImm();

  return false;
}

MCSymbol *
XtensaAsmPrinter::GetConstantPoolIndexSymbol(const MachineOperand &MO) const {
  // Create a symbol for the name.
  return GetCPISymbol(MO.getIndex());
}

```
- **EN**: Implements logic around `getRegisterName`, `getImm`, `GetConstantPoolIndexSymbol`, `GetCPISymbol`; this block returns target-specific results.
- **CN**: 围绕 `getRegisterName`, `getImm`, `GetConstantPoolIndexSymbol`, `GetCPISymbol` 实现具体逻辑；这一段返回目标相关结果。

### Lines 224-241
```cpp
MCSymbol *XtensaAsmPrinter::GetJumpTableSymbol(const MachineOperand &MO) const {
  return GetJTISymbol(MO.getIndex());
}

MCOperand
XtensaAsmPrinter::LowerSymbolOperand(const MachineOperand &MO,
                                     MachineOperand::MachineOperandType MOTy,
                                     unsigned Offset) const {
  const MCSymbol *Symbol;
  switch (MOTy) {
  case MachineOperand::MO_GlobalAddress:
    Symbol = getSymbol(MO.getGlobal());
    Offset += MO.getOffset();
    break;
  case MachineOperand::MO_MachineBasicBlock:
    Symbol = MO.getMBB()->getSymbol();
    break;
  case MachineOperand::MO_BlockAddress:
```
- **EN**: Implements logic around `GetJumpTableSymbol`, `GetJTISymbol`, `LowerSymbolOperand`, `getSymbol`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `GetJumpTableSymbol`, `GetJTISymbol`, `LowerSymbolOperand`, `getSymbol`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 242-259
```cpp
    Symbol = GetBlockAddressSymbol(MO.getBlockAddress());
    Offset += MO.getOffset();
    break;
  case MachineOperand::MO_ExternalSymbol:
    Symbol = GetExternalSymbolSymbol(MO.getSymbolName());
    Offset += MO.getOffset();
    break;
  case MachineOperand::MO_JumpTableIndex:
    Symbol = GetJumpTableSymbol(MO);
    break;
  case MachineOperand::MO_ConstantPoolIndex:
    Symbol = GetConstantPoolIndexSymbol(MO);
    Offset += MO.getOffset();
    break;
  default:
    report_fatal_error("<unknown operand type>");
  }

```
- **EN**: Implements logic around `GetBlockAddressSymbol`, `getOffset`, `GetExternalSymbolSymbol`, `GetJumpTableSymbol`, ....
- **CN**: 围绕 `GetBlockAddressSymbol`, `getOffset`, `GetExternalSymbolSymbol`, `GetJumpTableSymbol`, ... 实现具体逻辑。

### Lines 260-269
```cpp
  const MCExpr *ME = MCSymbolRefExpr::create(Symbol, OutContext);
  if (Offset) {
    // Assume offset is never negative.
    assert(Offset > 0);

    const MCConstantExpr *OffsetExpr =
        MCConstantExpr::create(Offset, OutContext);
    ME = MCBinaryExpr::createAdd(ME, OffsetExpr, OutContext);
  }

```
- **EN**: Implements logic around `create`, `assert`, `createAdd`; this block applies conditional target rules.
- **CN**: 围绕 `create`, `assert`, `createAdd` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 270-287
```cpp
  return MCOperand::createExpr(ME);
}

MCOperand XtensaAsmPrinter::lowerOperand(const MachineOperand &MO,
                                         unsigned Offset) const {
  MachineOperand::MachineOperandType MOTy = MO.getType();

  switch (MOTy) {
  case MachineOperand::MO_Register:
    // Ignore all implicit register operands.
    if (MO.isImplicit())
      break;
    return MCOperand::createReg(MO.getReg());
  case MachineOperand::MO_Immediate:
    return MCOperand::createImm(MO.getImm() + Offset);
  case MachineOperand::MO_RegisterMask:
    break;
  case MachineOperand::MO_GlobalAddress:
```
- **EN**: Implements logic around `createExpr`, `lowerOperand`, `getType`, `createReg`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `createExpr`, `lowerOperand`, `getType`, `createReg`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 288-297
```cpp
  case MachineOperand::MO_MachineBasicBlock:
  case MachineOperand::MO_BlockAddress:
  case MachineOperand::MO_ExternalSymbol:
  case MachineOperand::MO_JumpTableIndex:
  case MachineOperand::MO_ConstantPoolIndex:
    return LowerSymbolOperand(MO, MOTy, Offset);
  default:
    report_fatal_error("unknown operand type");
  }

```
- **EN**: Implements logic around `LowerSymbolOperand`, `report_fatal_error`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerSymbolOperand`, `report_fatal_error` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 298-308
```cpp
  return MCOperand();
}

void XtensaAsmPrinter::lowerToMCInst(const MachineInstr *MI,
                                     MCInst &OutMI) const {
  OutMI.setOpcode(MI->getOpcode());

  for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
    const MachineOperand &MO = MI->getOperand(i);
    MCOperand MCOp = lowerOperand(MO);

```
- **EN**: Implements logic around `MCOperand`, `lowerToMCInst`, `setOpcode`, `getOperand`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `MCOperand`, `lowerToMCInst`, `setOpcode`, `getOperand`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 309-318
```cpp
    if (MCOp.isValid())
      OutMI.addOperand(MCOp);
  }
}

char XtensaAsmPrinter::ID = 0;

INITIALIZE_PASS(XtensaAsmPrinter, "xtensa-asm-printer",
                "Xtensa Assembly Printer", false, false)

```
- **EN**: Implements logic around `addOperand`, `INITIALIZE_PASS`; this block applies conditional target rules.
- **CN**: 围绕 `addOperand`, `INITIALIZE_PASS` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 319-321
```cpp
extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeXtensaAsmPrinter() {
  RegisterAsmPrinter<XtensaAsmPrinter> A(getTheXtensaTarget());
}
```
- **EN**: Implements logic around `A`.
- **CN**: 围绕 `A` 实现具体逻辑。

## Key Concepts / 关键概念

- **Assembly emission / 汇编发射**:
  - **EN**: Lowers machine instructions to printable/emittable MC form
  - **CN**: 把机器指令降低为可打印/可发射的 MC 形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XtensaAsmPrinter.h`, `MCTargetDesc/XtensaInstPrinter.h`, `MCTargetDesc/XtensaMCAsmInfo.h`, `MCTargetDesc/XtensaTargetStreamer.h`, `TargetInfo/XtensaTargetInfo.h`, `XtensaConstantPoolValue.h`, `llvm/ADT/StringExtras.h`, `llvm/BinaryFormat/ELF.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineModuleInfoImpls.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/MC/MCExpr.h` ... (+6 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen
