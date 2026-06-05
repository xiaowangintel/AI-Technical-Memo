# MipsAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsAsmPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to GAS-format MIPS assembly language.
- 用途 (CN): 实现 Mips 后端中的 `MipsAsmPrinter`，重点处理汇编打印与 MC 发射。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- MipsAsmPrinter.cpp - Mips LLVM Assembly Printer --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains a printer that converts from our internal representation
// of machine-dependent LLVM code to GAS-format MIPS assembly language.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-28
```cpp
#include "MipsAsmPrinter.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsInstPrinter.h"
#include "MCTargetDesc/MipsMCAsmInfo.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "MCTargetDesc/MipsTargetStreamer.h"
#include "Mips.h"
#include "MipsMCInstLower.h"
#include "MipsMachineFunction.h"
#include "MipsSubtarget.h"
#include "MipsTargetMachine.h"
#include "TargetInfo/MipsTargetInfo.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringRef.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 29-43
```cpp
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/ELF.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineConstantPool.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Function.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 44-58
```cpp
#include "llvm/IR/InlineAsm.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstBuilder.h"
#include "llvm/MC/MCObjectFileInfo.h"
#include "llvm/MC/MCSectionELF.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Casting.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 59-67
```cpp
#include "llvm/Target/TargetLoweringObjectFile.h"
#include "llvm/Target/TargetMachine.h"
#include "llvm/TargetParser/Triple.h"
#include <cassert>
#include <cstdint>
#include <map>
#include <memory>
#include <string>
#include <vector>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 69-69
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 71-71
```cpp
#define DEBUG_TYPE "mips-asm-printer"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 73-73
```cpp
extern cl::opt<bool> EmitJalrReloc;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 75-77
```cpp
MipsTargetStreamer &MipsAsmPrinter::getTargetStreamer() const {
  return static_cast<MipsTargetStreamer &>(*OutStreamer->getTargetStreamer());
}
```
- EN: Implements `MipsAsmPrinter::getTargetStreamer`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::getTargetStreamer`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 79-80
```cpp
bool MipsAsmPrinter::runOnMachineFunction(MachineFunction &MF) {
  Subtarget = &MF.getSubtarget<MipsSubtarget>();
```
- EN: Implements `MipsAsmPrinter::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 82-86
```cpp
  MipsFI = MF.getInfo<MipsFunctionInfo>();
  if (Subtarget->inMips16Mode())
    for (const auto &I : MipsFI->StubsNeeded)
      StubsNeeded.insert(I);
  MCP = MF.getConstantPool();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 88-88
```cpp
  AsmPrinter::runOnMachineFunction(MF);
```
- EN: Declares `AsmPrinter::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `AsmPrinter::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 90-90
```cpp
  emitXRayTable();
```
- EN: Declares `emitXRayTable`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitXRayTable`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 92-93
```cpp
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-98
```cpp
bool MipsAsmPrinter::lowerOperand(const MachineOperand &MO, MCOperand &MCOp) {
  MCOp = MCInstLowering.LowerOperand(MO);
  return MCOp.isValid();
}
```
- EN: Implements `MipsAsmPrinter::lowerOperand`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::lowerOperand`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-100
```cpp
#include "MipsGenMCPseudoLowering.inc"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 102-108
```cpp
// Lower PseudoReturn/PseudoIndirectBranch/PseudoIndirectBranch64 to JR, JR_MM,
// JALR, or JALR64 as appropriate for the target.
void MipsAsmPrinter::emitPseudoIndirectBranch(MCStreamer &OutStreamer,
                                              const MachineInstr *MI) {
  bool HasLinkReg = false;
  bool InMicroMipsMode = Subtarget->inMicroMipsMode();
  MCInst TmpInst0;
```
- EN: Implements `MipsAsmPrinter::emitPseudoIndirectBranch`, a emission/printing routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitPseudoIndirectBranch`，它是一个围绕MC streamer 交互展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-124
```cpp
  if (Subtarget->hasMips64r6()) {
    // MIPS64r6 should use (JALR64 ZERO_64, $rs)
    TmpInst0.setOpcode(Mips::JALR64);
    HasLinkReg = true;
  } else if (Subtarget->hasMips32r6()) {
    // MIPS32r6 should use (JALR ZERO, $rs)
    if (InMicroMipsMode)
      TmpInst0.setOpcode(Mips::JRC16_MMR6);
    else {
      TmpInst0.setOpcode(Mips::JALR);
      HasLinkReg = true;
    }
  } else if (Subtarget->inMicroMipsMode())
    // microMIPS should use (JR_MM $rs)
    TmpInst0.setOpcode(Mips::JR_MM);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 125-128
```cpp
  else {
    // Everything else should use (JR $rs)
    TmpInst0.setOpcode(Mips::JR);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 130-130
```cpp
  MCOperand MCOp;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 132-135
```cpp
  if (HasLinkReg) {
    unsigned ZeroReg = Subtarget->isGP64bit() ? Mips::ZERO_64 : Mips::ZERO;
    TmpInst0.addOperand(MCOperand::createReg(ZeroReg));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 137-138
```cpp
  lowerOperand(MI->getOperand(0), MCOp);
  TmpInst0.addOperand(MCOp);
```
- EN: Declares `lowerOperand`, a lowering routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lowerOperand`，它是一个围绕目标相关状态展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 140-141
```cpp
  EmitToStreamer(OutStreamer, TmpInst0);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-157
```cpp
// If there is an MO_JALR operand, insert:
//
// .reloc tmplabel, R_{MICRO}MIPS_JALR, symbol
// tmplabel:
//
// This is an optimization hint for the linker which may then replace
// an indirect call with a direct branch.
static void emitDirectiveRelocJalr(const MachineInstr &MI,
                                   MCContext &OutContext,
                                   TargetMachine &TM,
                                   MCStreamer &OutStreamer,
                                   const MipsSubtarget &Subtarget) {
  for (const MachineOperand &MO :
       llvm::drop_begin(MI.operands(), MI.getDesc().getNumOperands())) {
    if (MO.isMCSymbol() && (MO.getTargetFlags() & MipsII::MO_JALR)) {
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 158-172
```cpp
      MCSymbol *Callee = MO.getMCSymbol();
      if (Callee && !Callee->getName().empty()) {
        MCSymbol *OffsetLabel = OutContext.createTempSymbol();
        const MCExpr *OffsetExpr =
            MCSymbolRefExpr::create(OffsetLabel, OutContext);
        const MCExpr *CaleeExpr =
            MCSymbolRefExpr::create(Callee, OutContext);
        OutStreamer.emitRelocDirective(
            *OffsetExpr,
            Subtarget.inMicroMipsMode() ? "R_MICROMIPS_JALR" : "R_MIPS_JALR",
            CaleeExpr);
        OutStreamer.emitLabel(OffsetLabel);
        return;
      }
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 173-174
```cpp
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 176-179
```cpp
void MipsAsmPrinter::emitInstruction(const MachineInstr *MI) {
  // FIXME: Enable feature predicate checks once all the test pass.
  // Mips_MC::verifyInstructionPredicates(MI->getOpcode(),
  //                                      getSubtargetInfo().getFeatureBits());
```
- EN: Implements `MipsAsmPrinter::emitInstruction`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitInstruction`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 181-183
```cpp
  MipsTargetStreamer &TS = getTargetStreamer();
  unsigned Opc = MI->getOpcode();
  TS.forbidModuleDirective();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 185-187
```cpp
  if (MI->isDebugValue()) {
    SmallString<128> Str;
    raw_svector_ostream OS(Str);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 189-193
```cpp
    PrintDebugValueComment(MI, OS);
    return;
  }
  if (MI->isDebugLabel())
    return;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 195-209
```cpp
  // If we just ended a constant pool, mark it as such.
  if (InConstantPool && Opc != Mips::CONSTPOOL_ENTRY) {
    OutStreamer->emitDataRegion(MCDR_DataRegionEnd);
    InConstantPool = false;
  }
  if (Opc == Mips::CONSTPOOL_ENTRY) {
    // CONSTPOOL_ENTRY - This instruction represents a floating
    // constant pool in the function.  The first operand is the ID#
    // for this instruction, the second is the index into the
    // MachineConstantPool that this is, the third is the size in
    // bytes of this constant pool entry.
    // The required alignment is specified on the basic block holding this MI.
    //
    unsigned LabelId = (unsigned)MI->getOperand(0).getImm();
    unsigned CPIdx = (unsigned)MI->getOperand(1).getIndex();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 211-215
```cpp
    // If this is the first entry of the pool, mark it.
    if (!InConstantPool) {
      OutStreamer->emitDataRegion(MCDR_DataRegion);
      InConstantPool = true;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 217-217
```cpp
    OutStreamer->emitLabel(GetCPISymbol(LabelId));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 219-225
```cpp
    const MachineConstantPoolEntry &MCPE = MCP->getConstants()[CPIdx];
    if (MCPE.isMachineConstantPoolEntry())
      emitMachineConstantPoolValue(MCPE.Val.MachineCPVal);
    else
      emitGlobalConstant(MF->getDataLayout(), MCPE.Val.ConstVal);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 227-237
```cpp
  switch (Opc) {
  case Mips::PATCHABLE_FUNCTION_ENTER:
    LowerPATCHABLE_FUNCTION_ENTER(*MI);
    return;
  case Mips::PATCHABLE_FUNCTION_EXIT:
    LowerPATCHABLE_FUNCTION_EXIT(*MI);
    return;
  case Mips::PATCHABLE_TAIL_CALL:
    LowerPATCHABLE_TAIL_CALL(*MI);
    return;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 239-242
```cpp
  if (EmitJalrReloc &&
      (MI->isReturn() || MI->isCall() || MI->isIndirectBranch())) {
    emitDirectiveRelocJalr(*MI, OutContext, TM, *OutStreamer, *Subtarget);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 244-245
```cpp
  MachineBasicBlock::const_instr_iterator I = MI->getIterator();
  MachineBasicBlock::const_instr_iterator E = MI->getParent()->instr_end();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 247-252
```cpp
  do {
    // Do any auto-generated pseudo lowerings.
    if (MCInst OutInst; lowerPseudoInstExpansion(&*I, OutInst)) {
      EmitToStreamer(*OutStreamer, OutInst);
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 254-256
```cpp
    // Skip the BUNDLE pseudo instruction and lower the contents
    if (I->isBundle())
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 258-266
```cpp
    if (I->getOpcode() == Mips::PseudoReturn ||
        I->getOpcode() == Mips::PseudoReturn64 ||
        I->getOpcode() == Mips::PseudoIndirectBranch ||
        I->getOpcode() == Mips::PseudoIndirectBranch64 ||
        I->getOpcode() == Mips::TAILCALLREG ||
        I->getOpcode() == Mips::TAILCALLREG64) {
      emitPseudoIndirectBranch(*OutStreamer, &*I);
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 268-277
```cpp
    // The inMips16Mode() test is not permanent.
    // Some instructions are marked as pseudo right now which
    // would make the test fail for the wrong reason but
    // that will be fixed soon. We need this here because we are
    // removing another test for this situation downstream in the
    // callchain.
    //
    if (I->isPseudo() && !Subtarget->inMips16Mode()
        && !isLongBranchPseudo(I->getOpcode()))
      llvm_unreachable("Pseudo opcode found in emitInstruction()");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-283
```cpp
    MCInst TmpInst0;
    MCInstLowering.Lower(&*I, TmpInst0);
    EmitToStreamer(*OutStreamer, TmpInst0);
  } while ((++I != E) && I->isInsideBundle()); // Delay slot check
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 285-299
```cpp
//===----------------------------------------------------------------------===//
//
//  Mips Asm Directives
//
//  -- Frame directive "frame Stackpointer, Stacksize, RARegister"
//  Describe the stack frame.
//
//  -- Mask directives "(f)mask  bitmask, offset"
//  Tells the assembler which registers are saved and where.
//  bitmask - contain a little endian bitset indicating which registers are
//            saved on function prologue (e.g. with a 0x80000000 mask, the
//            assembler knows the register 31 (RA) is saved at prologue.
//  offset  - the position before stack pointer subtraction indicating where
//            the first saved register on prologue is located. (e.g. with a
//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 300-314
```cpp
//  Consider the following function prologue:
//
//    .frame  $fp,48,$ra
//    .mask   0xc0000000,-8
//       addiu $sp, $sp, -48
//       sw $ra, 40($sp)
//       sw $fp, 36($sp)
//
//    With a 0xc0000000 mask, the assembler knows the register 31 (RA) and
//    30 (FP) are saved at prologue. As the save order on prologue is from
//    left to right, RA is saved first. A -8 offset means that after the
//    stack pointer subtration, the first register in the mask (RA) will be
//    saved at address 48-8=40.
//
//===----------------------------------------------------------------------===//
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 316-318
```cpp
//===----------------------------------------------------------------------===//
// Mask directives
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 320-325
```cpp
// Create a bitmask with all callee saved registers for CPU or Floating Point
// registers. For CPU registers consider RA, GP and FP for saving if necessary.
void MipsAsmPrinter::printSavedRegsBitmask() {
  // CPU and FPU Saved Registers Bitmasks
  unsigned CPUBitmask = 0, FPUBitmask = 0;
  int CPUTopSavedRegOff, FPUTopSavedRegOff;
```
- EN: Implements `MipsAsmPrinter::printSavedRegsBitmask`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::printSavedRegsBitmask`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 327-336
```cpp
  // Set the CPU and FPU Bitmasks
  const MachineFrameInfo &MFI = MF->getFrameInfo();
  const TargetRegisterInfo *TRI = MF->getSubtarget().getRegisterInfo();
  const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
  // size of stack area to which FP callee-saved regs are saved.
  unsigned CPURegSize = TRI->getRegSizeInBits(Mips::GPR32RegClass) / 8;
  unsigned FGR32RegSize = TRI->getRegSizeInBits(Mips::FGR32RegClass) / 8;
  unsigned AFGR64RegSize = TRI->getRegSizeInBits(Mips::AFGR64RegClass) / 8;
  bool HasAFGR64Reg = false;
  unsigned CSFPRegsSize = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 338-340
```cpp
  for (const auto &I : CSI) {
    Register Reg = I.getReg();
    unsigned RegNum = TRI->getEncodingValue(Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 342-353
```cpp
    // If it's a floating point register, set the FPU Bitmask.
    // If it's a general purpose register, set the CPU Bitmask.
    if (Mips::FGR32RegClass.contains(Reg)) {
      FPUBitmask |= (1 << RegNum);
      CSFPRegsSize += FGR32RegSize;
    } else if (Mips::AFGR64RegClass.contains(Reg)) {
      FPUBitmask |= (3 << RegNum);
      CSFPRegsSize += AFGR64RegSize;
      HasAFGR64Reg = true;
    } else if (Mips::GPR32RegClass.contains(Reg))
      CPUBitmask |= (1 << RegNum);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 355-357
```cpp
  // FP Regs are saved right below where the virtual frame pointer points to.
  FPUTopSavedRegOff = FPUBitmask ?
    (HasAFGR64Reg ? -AFGR64RegSize : -FGR32RegSize) : 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 359-360
```cpp
  // CPU Regs are saved below FP Regs.
  CPUTopSavedRegOff = CPUBitmask ? -CSFPRegsSize - CPURegSize : 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 362-364
```cpp
  MipsTargetStreamer &TS = getTargetStreamer();
  // Print CPUBitmask
  TS.emitMask(CPUBitmask, CPUTopSavedRegOff);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 366-368
```cpp
  // Print FPUBitmask
  TS.emitFMask(FPUBitmask, FPUTopSavedRegOff);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 370-372
```cpp
//===----------------------------------------------------------------------===//
// Frame and Set directives
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 374-376
```cpp
/// Frame Directive
void MipsAsmPrinter::emitFrameDirective() {
  const TargetRegisterInfo &RI = *MF->getSubtarget().getRegisterInfo();
```
- EN: Implements `MipsAsmPrinter::emitFrameDirective`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitFrameDirective`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 378-380
```cpp
  Register stackReg = RI.getFrameRegister(*MF);
  MCRegister returnReg = RI.getRARegister();
  unsigned stackSize = MF->getFrameInfo().getStackSize();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 382-383
```cpp
  getTargetStreamer().emitFrame(stackReg, stackSize, returnReg);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 385-393
```cpp
/// Emit Set directives.
const char *MipsAsmPrinter::getCurrentABIString() const {
  switch (static_cast<MipsTargetMachine &>(TM).getABI().GetEnumValue()) {
  case MipsABIInfo::ABI::O32:  return "abi32";
  case MipsABIInfo::ABI::N32:  return "abiN32";
  case MipsABIInfo::ABI::N64:  return "abi64";
  default: llvm_unreachable("Unknown Mips ABI");
  }
}
```
- EN: Implements `MipsAsmPrinter::getCurrentABIString`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::getCurrentABIString`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 395-396
```cpp
void MipsAsmPrinter::emitFunctionEntryLabel() {
  MipsTargetStreamer &TS = getTargetStreamer();
```
- EN: Implements `MipsAsmPrinter::emitFunctionEntryLabel`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitFunctionEntryLabel`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 398-403
```cpp
  if (Subtarget->inMicroMipsMode()) {
    TS.emitDirectiveSetMicroMips();
    TS.setUsesMicroMips();
    TS.updateABIInfo(*Subtarget);
  } else
    TS.emitDirectiveSetNoMicroMips();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 405-408
```cpp
  if (Subtarget->inMips16Mode())
    TS.emitDirectiveSetMips16();
  else
    TS.emitDirectiveSetNoMips16();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 410-412
```cpp
  TS.emitDirectiveEnt(*CurrentFnSym);
  OutStreamer->emitLabel(CurrentFnSym);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 414-417
```cpp
/// EmitFunctionBodyStart - Targets can override this to emit stuff before
/// the first basic block in the function.
void MipsAsmPrinter::emitFunctionBodyStart() {
  MipsTargetStreamer &TS = getTargetStreamer();
```
- EN: Implements `MipsAsmPrinter::emitFunctionBodyStart`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitFunctionBodyStart`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 419-419
```cpp
  MCInstLowering.Initialize(&MF->getContext());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 421-423
```cpp
  bool IsNakedFunction = MF->getFunction().hasFnAttribute(Attribute::Naked);
  if (!IsNakedFunction)
    emitFrameDirective();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 425-426
```cpp
  if (!IsNakedFunction)
    printSavedRegsBitmask();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 428-433
```cpp
  if (!Subtarget->inMips16Mode()) {
    TS.emitDirectiveSetNoReorder();
    TS.emitDirectiveSetNoMacro();
    TS.emitDirectiveSetNoAt();
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 435-438
```cpp
/// EmitFunctionBodyEnd - Targets can override this to emit stuff after
/// the last basic block in the function.
void MipsAsmPrinter::emitFunctionBodyEnd() {
  MipsTargetStreamer &TS = getTargetStreamer();
```
- EN: Implements `MipsAsmPrinter::emitFunctionBodyEnd`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitFunctionBodyEnd`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 440-454
```cpp
  // There are instruction for this macros, but they must
  // always be at the function end, and we can't emit and
  // break with BB logic.
  if (!Subtarget->inMips16Mode()) {
    TS.emitDirectiveSetAt();
    TS.emitDirectiveSetMacro();
    TS.emitDirectiveSetReorder();
  }
  TS.emitDirectiveEnd(CurrentFnSym->getName());
  // Make sure to terminate any constant pools that were at the end
  // of the function.
  if (!InConstantPool)
    return;
  InConstantPool = false;
  OutStreamer->emitDataRegion(MCDR_DataRegionEnd);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 455-455
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 457-462
```cpp
void MipsAsmPrinter::emitBasicBlockEnd(const MachineBasicBlock &MBB) {
  AsmPrinter::emitBasicBlockEnd(MBB);
  MipsTargetStreamer &TS = getTargetStreamer();
  if (MBB.empty())
    TS.emitDirectiveInsn();
}
```
- EN: Implements `MipsAsmPrinter::emitBasicBlockEnd`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitBasicBlockEnd`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 464-469
```cpp
// Print out an operand for an inline asm expression.
bool MipsAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNum,
                                     const char *ExtraCode, raw_ostream &O) {
  // Does this asm operand have a single letter operand modifier?
  if (ExtraCode && ExtraCode[0]) {
    if (ExtraCode[1] != 0) return true; // Unknown modifier.
```
- EN: Implements `MipsAsmPrinter::PrintAsmOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::PrintAsmOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 471-485
```cpp
    const MachineOperand &MO = MI->getOperand(OpNum);
    switch (ExtraCode[0]) {
    default:
      // See if this is a generic print operand
      return AsmPrinter::PrintAsmOperand(MI, OpNum, ExtraCode, O);
    case 'X': // hex const int
      if (!MO.isImm())
        return true;
      O << "0x" << Twine::utohexstr(MO.getImm());
      return false;
    case 'x': // hex const int (low 16 bits)
      if (!MO.isImm())
        return true;
      O << "0x" << Twine::utohexstr(MO.getImm() & 0xffff);
      return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 486-500
```cpp
    case 'd': // decimal const int
      if (!MO.isImm())
        return true;
      O << MO.getImm();
      return false;
    case 'm': // decimal const int minus 1
      if (!MO.isImm())
        return true;
      O << MO.getImm() - 1;
      return false;
    case 'y': // exact log2
      if (!MO.isImm())
        return true;
      if (!isPowerOf2_64(MO.getImm()))
        return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 501-515
```cpp
      O << Log2_64(MO.getImm());
      return false;
    case 'z':
      // $0 if zero, regular printing otherwise
      if (MO.isImm() && MO.getImm() == 0) {
        O << "$0";
        return false;
      }
      // If not, call printOperand as normal.
      break;
    case 'D': // Second part of a double word register operand
    case 'L': // Low order register of a double word register operand
    case 'M': // High order register of a double word register operand
    {
      if (OpNum == 0)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 516-530
```cpp
        return true;
      const MachineOperand &FlagsOP = MI->getOperand(OpNum - 1);
      if (!FlagsOP.isImm())
        return true;
      const InlineAsm::Flag Flags(FlagsOP.getImm());
      const unsigned NumVals = Flags.getNumOperandRegisters();
      // Number of registers represented by this operand. We are looking
      // for 2 for 32 bit mode and 1 for 64 bit mode.
      if (NumVals != 2) {
        if (Subtarget->isGP64bit() && NumVals == 1 && MO.isReg()) {
          Register Reg = MO.getReg();
          O << '$' << MipsInstPrinter::getRegisterName(Reg);
          return false;
        }
        return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 531-531
```cpp
      }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 533-547
```cpp
      unsigned RegOp = OpNum;
      if (!Subtarget->isGP64bit()){
        // Endianness reverses which register holds the high or low value
        // between M and L.
        switch(ExtraCode[0]) {
        case 'M':
          RegOp = (Subtarget->isLittle()) ? OpNum + 1 : OpNum;
          break;
        case 'L':
          RegOp = (Subtarget->isLittle()) ? OpNum : OpNum + 1;
          break;
        case 'D': // Always the second part
          RegOp = OpNum + 1;
        }
        if (RegOp >= MI->getNumOperands())
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 548-562
```cpp
          return true;
        const MachineOperand &MO = MI->getOperand(RegOp);
        if (!MO.isReg())
          return true;
        Register Reg = MO.getReg();
        O << '$' << MipsInstPrinter::getRegisterName(Reg);
        return false;
      }
      break;
    }
    case 'w': {
      MCRegister w = getMSARegFromFReg(MO.getReg());
      if (w != Mips::NoRegister) {
        O << '$' << MipsInstPrinter::getRegisterName(w);
        return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 563-567
```cpp
      }
      break;
    }
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 569-571
```cpp
  printOperand(MI, OpNum, O);
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 573-584
```cpp
bool MipsAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
                                           unsigned OpNum,
                                           const char *ExtraCode,
                                           raw_ostream &O) {
  assert(OpNum + 1 < MI->getNumOperands() && "Insufficient operands");
  const MachineOperand &BaseMO = MI->getOperand(OpNum);
  const MachineOperand &OffsetMO = MI->getOperand(OpNum + 1);
  assert(BaseMO.isReg() &&
         "Unexpected base pointer for inline asm memory operand.");
  assert(OffsetMO.isImm() &&
         "Unexpected offset for inline asm memory operand.");
  int Offset = OffsetMO.getImm();
```
- EN: Implements `MipsAsmPrinter::PrintAsmMemoryOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::PrintAsmMemoryOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 586-600
```cpp
  // Currently we are expecting either no ExtraCode or 'D','M','L'.
  if (ExtraCode) {
    switch (ExtraCode[0]) {
    case 'D':
      Offset += 4;
      break;
    case 'M':
      if (Subtarget->isLittle())
        Offset += 4;
      break;
    case 'L':
      if (!Subtarget->isLittle())
        Offset += 4;
      break;
    default:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 601-603
```cpp
      return true; // Unknown modifier.
    }
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 605-606
```cpp
  O << Offset << "($" << MipsInstPrinter::getRegisterName(BaseMO.getReg())
    << ")";
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 608-609
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 611-614
```cpp
void MipsAsmPrinter::printOperand(const MachineInstr *MI, int opNum,
                                  raw_ostream &O) {
  const MachineOperand &MO = MI->getOperand(opNum);
  bool closeP = false;
```
- EN: Implements `MipsAsmPrinter::printOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::printOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 616-617
```cpp
  if (MO.getTargetFlags())
    closeP = true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 619-633
```cpp
  switch(MO.getTargetFlags()) {
  case MipsII::MO_GPREL:    O << "%gp_rel("; break;
  case MipsII::MO_GOT_CALL: O << "%call16("; break;
  case MipsII::MO_GOT:      O << "%got(";    break;
  case MipsII::MO_ABS_HI:   O << "%hi(";     break;
  case MipsII::MO_ABS_LO:   O << "%lo(";     break;
  case MipsII::MO_HIGHER:   O << "%higher("; break;
  case MipsII::MO_HIGHEST:  O << "%highest(("; break;
  case MipsII::MO_TLSGD:    O << "%tlsgd(";  break;
  case MipsII::MO_GOTTPREL: O << "%gottprel("; break;
  case MipsII::MO_TPREL_HI: O << "%tprel_hi("; break;
  case MipsII::MO_TPREL_LO: O << "%tprel_lo("; break;
  case MipsII::MO_GPOFF_HI: O << "%hi(%neg(%gp_rel("; break;
  case MipsII::MO_GPOFF_LO: O << "%lo(%neg(%gp_rel("; break;
  case MipsII::MO_GOT_DISP: O << "%got_disp("; break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 634-636
```cpp
  case MipsII::MO_GOT_PAGE: O << "%got_page("; break;
  case MipsII::MO_GOT_OFST: O << "%got_ofst("; break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 638-642
```cpp
  switch (MO.getType()) {
    case MachineOperand::MO_Register:
      O << '$'
        << StringRef(MipsInstPrinter::getRegisterName(MO.getReg())).lower();
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 644-646
```cpp
    case MachineOperand::MO_Immediate:
      O << MO.getImm();
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 648-650
```cpp
    case MachineOperand::MO_MachineBasicBlock:
      MO.getMBB()->getSymbol()->print(O, MAI);
      return;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 652-654
```cpp
    case MachineOperand::MO_GlobalAddress:
      PrintSymbolOperand(MO, O);
      break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 656-660
```cpp
    case MachineOperand::MO_BlockAddress: {
      MCSymbol *BA = GetBlockAddressSymbol(MO.getBlockAddress());
      O << BA->getName();
      break;
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 662-667
```cpp
    case MachineOperand::MO_ConstantPoolIndex:
      O << getDataLayout().getInternalSymbolPrefix() << "CPI"
        << getFunctionNumber() << "_" << MO.getIndex();
      if (MO.getOffset())
        O << "+" << MO.getOffset();
      break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 669-671
```cpp
    default:
      llvm_unreachable("<unknown operand type>");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 673-674
```cpp
  if (closeP) O << ")";
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 676-680
```cpp
void MipsAsmPrinter::
printMemOperand(const MachineInstr *MI, int opNum, raw_ostream &O) {
  // Load/Store memory operands -- imm($reg)
  // If PIC target the target is loaded as the
  // pattern lw $25,%call16($28)
```
- EN: Implements `printMemOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `printMemOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 682-691
```cpp
  // opNum can be invalid if instruction has reglist as operand.
  // MemOperand is always last operand of instruction (base + offset).
  switch (MI->getOpcode()) {
  default:
    break;
  case Mips::SWM32_MM:
  case Mips::LWM32_MM:
    opNum = MI->getNumOperands() - 2;
    break;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 693-697
```cpp
  printOperand(MI, opNum+1, O);
  O << "(";
  printOperand(MI, opNum, O);
  O << ")";
}
```
- EN: Declares `printOperand`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `printOperand`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 699-706
```cpp
void MipsAsmPrinter::
printMemOperandEA(const MachineInstr *MI, int opNum, raw_ostream &O) {
  // when using stack locations for not load/store instructions
  // print the same way as all normal 3 operand instructions.
  printOperand(MI, opNum, O);
  O << ", ";
  printOperand(MI, opNum+1, O);
}
```
- EN: Implements `printMemOperandEA`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `printMemOperandEA`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 708-712
```cpp
void MipsAsmPrinter::printFCCOperand(const MachineInstr *MI, int opNum,
                                     raw_ostream &O) {
  const MachineOperand &MO = MI->getOperand(opNum);
  O << Mips::MipsFCCToString((Mips::CondCode)MO.getImm());
}
```
- EN: Implements `MipsAsmPrinter::printFCCOperand`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::printFCCOperand`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 714-720
```cpp
void MipsAsmPrinter::
printRegisterList(const MachineInstr *MI, int opNum, raw_ostream &O) {
  for (int i = opNum, e = MI->getNumOperands(); i != e; ++i) {
    if (i != opNum) O << ", ";
    printOperand(MI, i, O);
  }
}
```
- EN: Implements `printRegisterList`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `printRegisterList`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 722-723
```cpp
void MipsAsmPrinter::emitStartOfAsmFile(Module &M) {
  const Triple &TT = TM.getTargetTriple();
```
- EN: Implements `MipsAsmPrinter::emitStartOfAsmFile`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitStartOfAsmFile`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 725-726
```cpp
  if (TT.isOSBinFormatELF()) {
    MipsTargetStreamer &TS = getTargetStreamer();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 728-731
```cpp
    // MipsTargetStreamer has an initialization order problem when emitting an
    // object file directly (see MipsTargetELFStreamer for full details). Work
    // around it by re-initializing the PIC state here.
    TS.setPic(OutContext.getObjectFileInfo()->isPositionIndependent());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 733-737
```cpp
    // Try to get target-features from the first function.
    StringRef FS = TM.getTargetFeatureString();
    Module::iterator F = M.begin();
    if (FS.empty() && M.size() && F->hasFnAttribute("target-features"))
      FS = F->getFnAttribute("target-features").getValueAsString();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 739-741
```cpp
    std::string strFS = FS.str();
    if (M.size() && F->getFnAttribute("use-soft-float").getValueAsBool())
      strFS += strFS.empty() ? "+soft-float" : ",+soft-float";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 743-750
```cpp
    // Compute MIPS architecture attributes based on the default subtarget
    // that we'd have constructed.
    // FIXME: For ifunc related functions we could iterate over and look
    // for a feature string that doesn't match the default one.
    StringRef CPU = MIPS_MC::selectMipsCPU(TT, TM.getTargetCPU());
    const MipsTargetMachine &MTM = static_cast<const MipsTargetMachine &>(TM);
    const MipsSubtarget STI(TT, CPU, StringRef(strFS), MTM.isLittleEndian(),
                            MTM, std::nullopt);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 752-762
```cpp
    bool IsABICalls = STI.isABICalls();
    const MipsABIInfo &ABI = MTM.getABI();
    if (IsABICalls) {
      TS.emitDirectiveAbiCalls();
      // FIXME: This condition should be a lot more complicated that it is here.
      //        Ideally it should test for properties of the ABI and not the ABI
      //        itself.
      //        For the moment, I'm only correcting enough to make MIPS-IV work.
      if (!isPositionIndependent() && STI.hasSym32())
        TS.emitDirectiveOptionPic0();
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 764-767
```cpp
    // Tell the assembler which ABI we are using
    std::string SectionName = std::string(".mdebug.") + getCurrentABIString();
    OutStreamer->switchSection(
        OutContext.getELFSection(SectionName, ELF::SHT_PROGBITS, 0));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 769-772
```cpp
    // NaN: At the moment we only support:
    // 1. .nan legacy (default)
    // 2. .nan 2008
    STI.isNaN2008() ? TS.emitDirectiveNaN2008() : TS.emitDirectiveNaNLegacy();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 774-774
```cpp
    // TODO: handle O64 ABI
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 776-776
```cpp
    TS.updateABIInfo(STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 778-783
```cpp
    // We should always emit a '.module fp=...' but binutils 2.24 does not
    // accept it. We therefore emit it when it contradicts the ABI defaults
    // (-mfpxx or -mfp64) and omit it otherwise.
    if ((ABI.IsO32() && (STI.isABI_FPXX() || STI.isFP64bit())) ||
        STI.useSoftFloat())
      TS.emitDirectiveModuleFP();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 785-789
```cpp
    // We should always emit a '.module [no]oddspreg' but binutils 2.24 does not
    // accept it. We therefore emit it when it contradicts the default or an
    // option has changed the default (i.e. FPXX) and omit it otherwise.
    if (ABI.IsO32() && (!STI.useOddSPReg() || STI.isABI_FPXX()))
      TS.emitDirectiveModuleOddSPReg();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 791-794
```cpp
    // Switch to the .text section.
    OutStreamer->switchSection(getObjFileLowering().getTextSection());
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 796-797
```cpp
void MipsAsmPrinter::emitInlineAsmStart() const {
  MipsTargetStreamer &TS = getTargetStreamer();
```
- EN: Implements `MipsAsmPrinter::emitInlineAsmStart`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitInlineAsmStart`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 799-810
```cpp
  // GCC's choice of assembler options for inline assembly code ('at', 'macro'
  // and 'reorder') is different from LLVM's choice for generated code ('noat',
  // 'nomacro' and 'noreorder').
  // In order to maintain compatibility with inline assembly code which depends
  // on GCC's assembler options being used, we have to switch to those options
  // for the duration of the inline assembly block and then switch back.
  TS.emitDirectiveSetPush();
  TS.emitDirectiveSetAt();
  TS.emitDirectiveSetMacro();
  TS.emitDirectiveSetReorder();
  OutStreamer->addBlankLine();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 812-817
```cpp
void MipsAsmPrinter::emitInlineAsmEnd(const MCSubtargetInfo &StartInfo,
                                      const MCSubtargetInfo *EndInfo,
                                      const MachineInstr *MI) {
  OutStreamer->addBlankLine();
  getTargetStreamer().emitDirectiveSetPop();
}
```
- EN: Implements `MipsAsmPrinter::emitInlineAsmEnd`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitInlineAsmEnd`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 819-833
```cpp
void MipsAsmPrinter::emitJumpTableEntry(const MachineJumpTableInfo &MJTI,
                                        const MachineBasicBlock *MBB,
                                        unsigned uid) const {
  MCSymbol *MBBSym = MBB->getSymbol();
  switch (MJTI.getEntryKind()) {
  case MachineJumpTableInfo::EK_BlockAddress:
    OutStreamer->emitValue(MCSymbolRefExpr::create(MBBSym, OutContext),
                           getDataLayout().getPointerSize());
    break;
  case MachineJumpTableInfo::EK_GPRel32BlockAddress:
    // Each entry is a GP-relative value targeting the block symbol.
    getTargetStreamer().emitGPRel32Value(
        MCSymbolRefExpr::create(MBBSym, OutContext));
    break;
  case MachineJumpTableInfo::EK_GPRel64BlockAddress:
```
- EN: Implements `MipsAsmPrinter::emitJumpTableEntry`, a emission/printing routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitJumpTableEntry`，它是一个围绕机器基本块展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 834-840
```cpp
    getTargetStreamer().emitGPRel64Value(
        MCSymbolRefExpr::create(MBBSym, OutContext));
    break;
  default:
    llvm_unreachable("");
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 842-848
```cpp
void MipsAsmPrinter::EmitJal(const MCSubtargetInfo &STI, MCSymbol *Symbol) {
  MCInst I;
  I.setOpcode(Mips::JAL);
  I.addOperand(
      MCOperand::createExpr(MCSymbolRefExpr::create(Symbol, OutContext)));
  OutStreamer->emitInstruction(I, STI);
}
```
- EN: Implements `MipsAsmPrinter::EmitJal`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitJal`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 850-856
```cpp
void MipsAsmPrinter::EmitInstrReg(const MCSubtargetInfo &STI, unsigned Opcode,
                                  unsigned Reg) {
  MCInst I;
  I.setOpcode(Opcode);
  I.addOperand(MCOperand::createReg(Reg));
  OutStreamer->emitInstruction(I, STI);
}
```
- EN: Implements `MipsAsmPrinter::EmitInstrReg`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitInstrReg`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 858-872
```cpp
void MipsAsmPrinter::EmitInstrRegReg(const MCSubtargetInfo &STI,
                                     unsigned Opcode, unsigned Reg1,
                                     unsigned Reg2) {
  MCInst I;
  //
  // Because of the current td files for Mips32, the operands for MTC1
  // appear backwards from their normal assembly order. It's not a trivial
  // change to fix this in the td file so we adjust for it here.
  //
  if (Opcode == Mips::MTC1) {
    unsigned Temp = Reg1;
    Reg1 = Reg2;
    Reg2 = Temp;
  }
  I.setOpcode(Opcode);
```
- EN: Implements `MipsAsmPrinter::EmitInstrRegReg`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitInstrRegReg`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 873-876
```cpp
  I.addOperand(MCOperand::createReg(Reg1));
  I.addOperand(MCOperand::createReg(Reg2));
  OutStreamer->emitInstruction(I, STI);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 878-887
```cpp
void MipsAsmPrinter::EmitInstrRegRegReg(const MCSubtargetInfo &STI,
                                        unsigned Opcode, unsigned Reg1,
                                        unsigned Reg2, unsigned Reg3) {
  MCInst I;
  I.setOpcode(Opcode);
  I.addOperand(MCOperand::createReg(Reg1));
  I.addOperand(MCOperand::createReg(Reg2));
  I.addOperand(MCOperand::createReg(Reg3));
  OutStreamer->emitInstruction(I, STI);
}
```
- EN: Implements `MipsAsmPrinter::EmitInstrRegRegReg`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitInstrRegRegReg`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 889-900
```cpp
void MipsAsmPrinter::EmitMovFPIntPair(const MCSubtargetInfo &STI,
                                      unsigned MovOpc, unsigned Reg1,
                                      unsigned Reg2, unsigned FPReg1,
                                      unsigned FPReg2, bool LE) {
  if (!LE) {
    unsigned temp = Reg1;
    Reg1 = Reg2;
    Reg2 = temp;
  }
  EmitInstrRegReg(STI, MovOpc, Reg1, FPReg1);
  EmitInstrRegReg(STI, MovOpc, Reg2, FPReg2);
}
```
- EN: Implements `MipsAsmPrinter::EmitMovFPIntPair`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitMovFPIntPair`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 902-905
```cpp
void MipsAsmPrinter::EmitSwapFPIntParams(const MCSubtargetInfo &STI,
                                         Mips16HardFloatInfo::FPParamVariant PV,
                                         bool LE, bool ToFP) {
  using namespace Mips16HardFloatInfo;
```
- EN: Implements `MipsAsmPrinter::EmitSwapFPIntParams`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitSwapFPIntParams`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 907-921
```cpp
  unsigned MovOpc = ToFP ? Mips::MTC1 : Mips::MFC1;
  switch (PV) {
  case FSig:
    EmitInstrRegReg(STI, MovOpc, Mips::A0, Mips::F12);
    break;
  case FFSig:
    EmitMovFPIntPair(STI, MovOpc, Mips::A0, Mips::A1, Mips::F12, Mips::F14, LE);
    break;
  case FDSig:
    EmitInstrRegReg(STI, MovOpc, Mips::A0, Mips::F12);
    EmitMovFPIntPair(STI, MovOpc, Mips::A2, Mips::A3, Mips::F14, Mips::F15, LE);
    break;
  case DSig:
    EmitMovFPIntPair(STI, MovOpc, Mips::A0, Mips::A1, Mips::F12, Mips::F13, LE);
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 922-933
```cpp
  case DDSig:
    EmitMovFPIntPair(STI, MovOpc, Mips::A0, Mips::A1, Mips::F12, Mips::F13, LE);
    EmitMovFPIntPair(STI, MovOpc, Mips::A2, Mips::A3, Mips::F14, Mips::F15, LE);
    break;
  case DFSig:
    EmitMovFPIntPair(STI, MovOpc, Mips::A0, Mips::A1, Mips::F12, Mips::F13, LE);
    EmitInstrRegReg(STI, MovOpc, Mips::A2, Mips::F14);
    break;
  case NoSig:
    return;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 935-938
```cpp
void MipsAsmPrinter::EmitSwapFPIntRetval(
    const MCSubtargetInfo &STI, Mips16HardFloatInfo::FPReturnVariant RV,
    bool LE) {
  using namespace Mips16HardFloatInfo;
```
- EN: Implements `MipsAsmPrinter::EmitSwapFPIntRetval`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitSwapFPIntRetval`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 940-954
```cpp
  unsigned MovOpc = Mips::MFC1;
  switch (RV) {
  case FRet:
    EmitInstrRegReg(STI, MovOpc, Mips::V0, Mips::F0);
    break;
  case DRet:
    EmitMovFPIntPair(STI, MovOpc, Mips::V0, Mips::V1, Mips::F0, Mips::F1, LE);
    break;
  case CFRet:
    EmitMovFPIntPair(STI, MovOpc, Mips::V0, Mips::V1, Mips::F0, Mips::F1, LE);
    break;
  case CDRet:
    EmitMovFPIntPair(STI, MovOpc, Mips::V0, Mips::V1, Mips::F0, Mips::F1, LE);
    EmitMovFPIntPair(STI, MovOpc, Mips::A0, Mips::A1, Mips::F2, Mips::F3, LE);
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 955-958
```cpp
  case NoFPRet:
    break;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 960-962
```cpp
void MipsAsmPrinter::EmitFPCallStub(
    const char *Symbol, const Mips16HardFloatInfo::FuncSignature *Signature) {
  using namespace Mips16HardFloatInfo;
```
- EN: Implements `MipsAsmPrinter::EmitFPCallStub`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitFPCallStub`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 964-971
```cpp
  MCSymbol *MSymbol = OutContext.getOrCreateSymbol(StringRef(Symbol));
  bool LE = getDataLayout().isLittleEndian();
  // Construct a local MCSubtargetInfo here.
  // This is because the MachineFunction won't exist (but have not yet been
  // freed) and since we're at the global level we can use the default
  // constructed subtarget.
  std::unique_ptr<MCSubtargetInfo> STI(TM.getTarget().createMCSubtargetInfo(
      TM.getTargetTriple(), TM.getTargetCPU(), TM.getTargetFeatureString()));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 973-987
```cpp
  //
  // .global xxxx
  //
  OutStreamer->emitSymbolAttribute(MSymbol, MCSA_Global);
  const char *RetType;
  //
  // make the comment field identifying the return and parameter
  // types of the floating point stub
  // # Stub function to call rettype xxxx (params)
  //
  switch (Signature->RetSig) {
  case FRet:
    RetType = "float";
    break;
  case DRet:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 988-1002
```cpp
    RetType = "double";
    break;
  case CFRet:
    RetType = "complex";
    break;
  case CDRet:
    RetType = "double complex";
    break;
  case NoFPRet:
    RetType = "";
    break;
  }
  const char *Parms;
  switch (Signature->ParamSig) {
  case FSig:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1003-1017
```cpp
    Parms = "float";
    break;
  case FFSig:
    Parms = "float, float";
    break;
  case FDSig:
    Parms = "float, double";
    break;
  case DSig:
    Parms = "double";
    break;
  case DDSig:
    Parms = "double, double";
    break;
  case DFSig:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1018-1032
```cpp
    Parms = "double, float";
    break;
  case NoSig:
    Parms = "";
    break;
  }
  OutStreamer->AddComment("\t# Stub function to call " + Twine(RetType) + " " +
                          Twine(Symbol) + " (" + Twine(Parms) + ")");
  //
  // probably not necessary but we save and restore the current section state
  //
  OutStreamer->pushSection();
  //
  // .section mips16.call.fpxxxx,"ax",@progbits
  //
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1033-1047
```cpp
  MCSectionELF *M = OutContext.getELFSection(
      ".mips16.call.fp." + std::string(Symbol), ELF::SHT_PROGBITS,
      ELF::SHF_ALLOC | ELF::SHF_EXECINSTR);
  OutStreamer->switchSection(M);
  //
  // .align 2
  //
  OutStreamer->emitValueToAlignment(Align(4));
  MipsTargetStreamer &TS = getTargetStreamer();
  //
  // .set nomips16
  // .set nomicromips
  //
  TS.emitDirectiveSetNoMips16();
  TS.emitDirectiveSetNoMicroMips();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1048-1059
```cpp
  //
  // .ent __call_stub_fp_xxxx
  // .type  __call_stub_fp_xxxx,@function
  //  __call_stub_fp_xxxx:
  //
  std::string x = "__call_stub_fp_" + std::string(Symbol);
  MCSymbol *Stub = OutContext.getOrCreateSymbol(StringRef(x));
  TS.emitDirectiveEnt(*Stub);
  MCSymbol *MType =
      OutContext.getOrCreateSymbol("__call_stub_fp_" + Twine(Symbol));
  OutStreamer->emitSymbolAttribute(MType, MCSA_ELF_TypeFunction);
  OutStreamer->emitLabel(Stub);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1061-1075
```cpp
  // Only handle non-pic for now.
  assert(!isPositionIndependent() &&
         "should not be here if we are compiling pic");
  TS.emitDirectiveSetReorder();
  //
  // We need to add a MipsMCExpr class to MCTargetDesc to fully implement
  // stubs without raw text but this current patch is for compiler generated
  // functions and they all return some value.
  // The calling sequence for non pic is different in that case and we need
  // to implement %lo and %hi in order to handle the case of no return value
  // See the corresponding method in Mips16HardFloat for details.
  //
  // mov the return address to S2.
  // we have no stack space to store it and we are about to make another call.
  // We need to make sure that the enclosing function knows to save S2
```
- EN: Declares `to`, packaging target-specific state and APIs around `MipsAsmPrinter`.
- CN: 这里声明 `to`，把与 `MipsAsmPrinter` 相关的目标特定状态和 API 组织在一起。

### Lines 1076-1078
```cpp
  // This should have already been handled.
  //
  // Mov $18, $31
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1080-1080
```cpp
  EmitInstrRegRegReg(*STI, Mips::OR, Mips::S2, Mips::RA, Mips::ZERO);
```
- EN: Declares `EmitInstrRegRegReg`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitInstrRegRegReg`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1082-1082
```cpp
  EmitSwapFPIntParams(*STI, Signature->ParamSig, LE, true);
```
- EN: Declares `EmitSwapFPIntParams`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSwapFPIntParams`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1084-1086
```cpp
  // Jal xxxx
  //
  EmitJal(*STI, MSymbol);
```
- EN: Declares `EmitJal`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitJal`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1088-1095
```cpp
  // fix return values
  EmitSwapFPIntRetval(*STI, Signature->RetSig, LE);
  //
  // do the return
  // if (Signature->RetSig == NoFPRet)
  //  llvm_unreachable("should not be any stubs here with no return value");
  // else
  EmitInstrReg(*STI, Mips::JR, Mips::S2);
```
- EN: Declares `EmitSwapFPIntRetval`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSwapFPIntRetval`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1097-1105
```cpp
  MCSymbol *Tmp = OutContext.createTempSymbol();
  OutStreamer->emitLabel(Tmp);
  const MCSymbolRefExpr *E = MCSymbolRefExpr::create(Stub, OutContext);
  const MCSymbolRefExpr *T = MCSymbolRefExpr::create(Tmp, OutContext);
  const MCExpr *T_min_E = MCBinaryExpr::createSub(T, E, OutContext);
  OutStreamer->emitELFSize(Stub, T_min_E);
  TS.emitDirectiveEnd(x);
  OutStreamer->popSection();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1107-1121
```cpp
void MipsAsmPrinter::emitEndOfAsmFile(Module &M) {
  // Emit needed stubs
  //
  for (std::map<
           const char *,
           const Mips16HardFloatInfo::FuncSignature *>::const_iterator
           it = StubsNeeded.begin();
       it != StubsNeeded.end(); ++it) {
    const char *Symbol = it->first;
    const Mips16HardFloatInfo::FuncSignature *Signature = it->second;
    EmitFPCallStub(Symbol, Signature);
  }
  // return to the text section
  OutStreamer->switchSection(OutContext.getObjectFileInfo()->getTextSection());
}
```
- EN: Implements `MipsAsmPrinter::emitEndOfAsmFile`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitEndOfAsmFile`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1123-1137
```cpp
void MipsAsmPrinter::EmitSled(const MachineInstr &MI, SledKind Kind) {
  const uint8_t NoopsInSledCount = Subtarget->isGP64bit() ? 15 : 11;
  // For mips32 we want to emit the following pattern:
  //
  // .Lxray_sled_N:
  //   ALIGN
  //   B .tmpN
  //   11 NOP instructions (44 bytes)
  //   ADDIU T9, T9, 52
  // .tmpN
  //
  // We need the 44 bytes (11 instructions) because at runtime, we'd
  // be patching over the full 48 bytes (12 instructions) with the following
  // pattern:
  //
```
- EN: Implements `MipsAsmPrinter::EmitSled`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::EmitSled`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1138-1152
```cpp
  //   ADDIU    SP, SP, -8
  //   NOP
  //   SW       RA, 4(SP)
  //   SW       T9, 0(SP)
  //   LUI      T9, %hi(__xray_FunctionEntry/Exit)
  //   ORI      T9, T9, %lo(__xray_FunctionEntry/Exit)
  //   LUI      T0, %hi(function_id)
  //   JALR     T9
  //   ORI      T0, T0, %lo(function_id)
  //   LW       T9, 0(SP)
  //   LW       RA, 4(SP)
  //   ADDIU    SP, SP, 8
  //
  // We add 52 bytes to t9 because we want to adjust the function pointer to
  // the actual start of function i.e. the address just after the noop sled.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1153-1167
```cpp
  // We do this because gp displacement relocation is emitted at the start of
  // of the function i.e after the nop sled and to correctly calculate the
  // global offset table address, t9 must hold the address of the instruction
  // containing the gp displacement relocation.
  // FIXME: Is this correct for the static relocation model?
  //
  // For mips64 we want to emit the following pattern:
  //
  // .Lxray_sled_N:
  //   ALIGN
  //   B .tmpN
  //   15 NOP instructions (60 bytes)
  // .tmpN
  //
  // We need the 60 bytes (15 instructions) because at runtime, we'd
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1168-1182
```cpp
  // be patching over the full 64 bytes (16 instructions) with the following
  // pattern:
  //
  //   DADDIU   SP, SP, -16
  //   NOP
  //   SD       RA, 8(SP)
  //   SD       T9, 0(SP)
  //   LUI      T9, %highest(__xray_FunctionEntry/Exit)
  //   ORI      T9, T9, %higher(__xray_FunctionEntry/Exit)
  //   DSLL     T9, T9, 16
  //   ORI      T9, T9, %hi(__xray_FunctionEntry/Exit)
  //   DSLL     T9, T9, 16
  //   ORI      T9, T9, %lo(__xray_FunctionEntry/Exit)
  //   LUI      T0, %hi(function_id)
  //   JALR     T9
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 1183-1191
```cpp
  //   ADDIU    T0, T0, %lo(function_id)
  //   LD       T9, 0(SP)
  //   LD       RA, 8(SP)
  //   DADDIU   SP, SP, 16
  //
  OutStreamer->emitCodeAlignment(Align(4), &getSubtargetInfo());
  auto CurSled = OutContext.createTempSymbol("xray_sled_", true);
  OutStreamer->emitLabel(CurSled);
  auto Target = OutContext.createTempSymbol();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1193-1199
```cpp
  // Emit "B .tmpN" instruction, which jumps over the nop sled to the actual
  // start of function
  const MCExpr *TargetExpr = MCSymbolRefExpr::create(Target, OutContext);
  EmitToStreamer(*OutStreamer, MCInstBuilder(Mips::BEQ)
                                   .addReg(Mips::ZERO)
                                   .addReg(Mips::ZERO)
                                   .addExpr(TargetExpr));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1201-1205
```cpp
  for (int8_t I = 0; I < NoopsInSledCount; I++)
    EmitToStreamer(*OutStreamer, MCInstBuilder(Mips::SLL)
                                     .addReg(Mips::ZERO)
                                     .addReg(Mips::ZERO)
                                     .addImm(0));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1207-1207
```cpp
  OutStreamer->emitLabel(Target);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1209-1215
```cpp
  if (!Subtarget->isGP64bit()) {
    EmitToStreamer(*OutStreamer,
                   MCInstBuilder(Mips::ADDiu)
                       .addReg(Mips::T9)
                       .addReg(Mips::T9)
                       .addImm(0x34));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 1217-1218
```cpp
  recordSled(CurSled, MI, Kind, 2);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1220-1222
```cpp
void MipsAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI) {
  EmitSled(MI, SledKind::FUNCTION_ENTER);
}
```
- EN: Implements `MipsAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1224-1226
```cpp
void MipsAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI) {
  EmitSled(MI, SledKind::FUNCTION_EXIT);
}
```
- EN: Implements `MipsAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1228-1230
```cpp
void MipsAsmPrinter::LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI) {
  EmitSled(MI, SledKind::TAIL_CALL);
}
```
- EN: Implements `MipsAsmPrinter::LowerPATCHABLE_TAIL_CALL`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::LowerPATCHABLE_TAIL_CALL`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1232-1235
```cpp
void MipsAsmPrinter::PrintDebugValueComment(const MachineInstr *MI,
                                           raw_ostream &OS) {
  // TODO: implement
}
```
- EN: Implements `MipsAsmPrinter::PrintDebugValueComment`, a emission/printing routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::PrintDebugValueComment`，它是一个围绕机器指令展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1237-1251
```cpp
// Emit .dtprelword or .dtpreldword directive
// and value for debug thread local expression.
void MipsAsmPrinter::emitDebugValue(const MCExpr *Value, unsigned Size) const {
  if (auto *MipsExpr = dyn_cast<MCSpecifierExpr>(Value)) {
    if (MipsExpr && MipsExpr->getSpecifier() == Mips::S_DTPREL) {
      switch (Size) {
      case 4:
        getTargetStreamer().emitDTPRel32Value(MipsExpr->getSubExpr());
        break;
      case 8:
        getTargetStreamer().emitDTPRel64Value(MipsExpr->getSubExpr());
        break;
      default:
        llvm_unreachable("Unexpected size of expression value.");
      }
```
- EN: Implements `MipsAsmPrinter::emitDebugValue`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::emitDebugValue`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1252-1256
```cpp
      return;
    }
  }
  AsmPrinter::emitDebugValue(Value, Size);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1258-1266
```cpp
bool MipsAsmPrinter::isLongBranchPseudo(int Opcode) const {
  return (Opcode == Mips::LONG_BRANCH_LUi
          || Opcode == Mips::LONG_BRANCH_LUi2Op
          || Opcode == Mips::LONG_BRANCH_LUi2Op_64
          || Opcode == Mips::LONG_BRANCH_ADDiu
          || Opcode == Mips::LONG_BRANCH_ADDiu2Op
          || Opcode == Mips::LONG_BRANCH_DADDiu
          || Opcode == Mips::LONG_BRANCH_DADDiu2Op);
}
```
- EN: Implements `MipsAsmPrinter::isLongBranchPseudo`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsAsmPrinter::isLongBranchPseudo`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 1268-1268
```cpp
char MipsAsmPrinter::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1270-1271
```cpp
INITIALIZE_PASS(MipsAsmPrinter, "mips-asm-printer", "Mips Assembly Printer",
                false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 1273-1280
```cpp
// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeMipsAsmPrinter() {
  RegisterAsmPrinter<MipsAsmPrinter> X(getTheMipsTarget());
  RegisterAsmPrinter<MipsAsmPrinter> Y(getTheMipselTarget());
  RegisterAsmPrinter<MipsAsmPrinter> A(getTheMips64Target());
  RegisterAsmPrinter<MipsAsmPrinter> B(getTheMips64elTarget());
}
```
- EN: Implements `LLVMInitializeMipsAsmPrinter`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVMInitializeMipsAsmPrinter`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: assembly printing and MC emission.
  - CN: 核心职责：汇编打印与 MC 发射。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsAsmPrinter.h`, `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsInstPrinter.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MCTargetDesc/MipsTargetStreamer.h`, `Mips.h` ... (+6 more).
  - CN: 后端本地头文件：`MipsAsmPrinter.h`, `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsInstPrinter.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `MCTargetDesc/MipsTargetStreamer.h`, `Mips.h` ... (+6 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/ELF.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h` ... (+27 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/BinaryFormat/ELF.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h` ... (+27 more)。
- EN: Standard/system headers: `cassert`, `cstdint`, `map`, `memory`, `string`, `vector`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`, `map`, `memory`, `string`, `vector`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
