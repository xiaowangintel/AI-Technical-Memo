# MipsMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsMCInstLower.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains code to lower Mips MachineInstrs to their corresponding MCInst records.
- 用途 (CN): 实现 Mips 后端中的 `MipsMCInstLower`，重点处理从 MachineInstr/MachineOperand 到 MC 层对象的降级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- MipsMCInstLower.cpp - Convert Mips MachineInstr to MCInst ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains code to lower Mips MachineInstrs to their corresponding
// MCInst records.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-24
```cpp
#include "MipsMCInstLower.h"
#include "MCTargetDesc/MipsBaseInfo.h"
#include "MCTargetDesc/MipsMCAsmInfo.h"
#include "MipsAsmPrinter.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/Support/ErrorHandling.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 26-26
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 28-29
```cpp
MipsMCInstLower::MipsMCInstLower(MipsAsmPrinter &asmprinter)
  : AsmPrinter(asmprinter) {}
```
- EN: Implements `MipsMCInstLower::MipsMCInstLower`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::MipsMCInstLower`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 31-33
```cpp
void MipsMCInstLower::Initialize(MCContext *C) {
  Ctx = C;
}
```
- EN: Implements `MipsMCInstLower::Initialize`, a target-specific routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::Initialize`，它是一个围绕MC 指令构造展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 35-42
```cpp
MCOperand MipsMCInstLower::LowerSymbolOperand(const MachineOperand &MO,
                                              MachineOperandType MOTy,
                                              int64_t Offset) const {
  Mips::Specifier TargetKind = Mips::S_None;
  bool IsGpOff = false;
  const MCSymbol *Symbol;
  SmallString<128> Name;
  unsigned TargetFlags = MO.getTargetFlags();
```
- EN: Implements `MipsMCInstLower::LowerSymbolOperand`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::LowerSymbolOperand`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 44-48
```cpp
  if (TargetFlags & MipsII::MO_DLLIMPORT) {
    // Handle dllimport linkage
    Name += "__imp_";
    TargetFlags &= ~MipsII::MO_DLLIMPORT;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 50-64
```cpp
  switch (TargetFlags) {
  default:
    llvm_unreachable("Invalid target flag!");
  case MipsII::MO_NO_FLAG:
    break;
  case MipsII::MO_GPREL:
    TargetKind = Mips::S_GPREL;
    break;
  case MipsII::MO_GOT_CALL:
    TargetKind = Mips::S_GOT_CALL;
    break;
  case MipsII::MO_GOT:
    TargetKind = Mips::S_GOT;
    break;
  case MipsII::MO_ABS_HI:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 65-79
```cpp
    TargetKind = Mips::S_HI;
    break;
  case MipsII::MO_ABS_LO:
    TargetKind = Mips::S_LO;
    break;
  case MipsII::MO_TLSGD:
    TargetKind = Mips::S_TLSGD;
    break;
  case MipsII::MO_TLSLDM:
    TargetKind = Mips::S_TLSLDM;
    break;
  case MipsII::MO_DTPREL_HI:
    TargetKind = Mips::S_DTPREL_HI;
    break;
  case MipsII::MO_DTPREL_LO:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 80-94
```cpp
    TargetKind = Mips::S_DTPREL_LO;
    break;
  case MipsII::MO_GOTTPREL:
    TargetKind = Mips::S_GOTTPREL;
    break;
  case MipsII::MO_TPREL_HI:
    TargetKind = Mips::S_TPREL_HI;
    break;
  case MipsII::MO_TPREL_LO:
    TargetKind = Mips::S_TPREL_LO;
    break;
  case MipsII::MO_GPOFF_HI:
    TargetKind = Mips::S_HI;
    IsGpOff = true;
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 95-109
```cpp
  case MipsII::MO_GPOFF_LO:
    TargetKind = Mips::S_LO;
    IsGpOff = true;
    break;
  case MipsII::MO_GOT_DISP:
    TargetKind = Mips::S_GOT_DISP;
    break;
  case MipsII::MO_GOT_HI16:
    TargetKind = Mips::S_GOT_HI16;
    break;
  case MipsII::MO_GOT_LO16:
    TargetKind = Mips::S_GOT_LO16;
    break;
  case MipsII::MO_GOT_PAGE:
    TargetKind = Mips::S_GOT_PAGE;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 110-124
```cpp
    break;
  case MipsII::MO_GOT_OFST:
    TargetKind = Mips::S_GOT_OFST;
    break;
  case MipsII::MO_HIGHER:
    TargetKind = Mips::S_HIGHER;
    break;
  case MipsII::MO_HIGHEST:
    TargetKind = Mips::S_HIGHEST;
    break;
  case MipsII::MO_CALL_HI16:
    TargetKind = Mips::S_CALL_HI16;
    break;
  case MipsII::MO_CALL_LO16:
    TargetKind = Mips::S_CALL_LO16;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-128
```cpp
    break;
  case MipsII::MO_JALR:
    return MCOperand();
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 130-133
```cpp
  switch (MOTy) {
  case MachineOperand::MO_MachineBasicBlock:
    Symbol = MO.getMBB()->getSymbol();
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 135-139
```cpp
  case MachineOperand::MO_GlobalAddress:
    AsmPrinter.getNameWithPrefix(Name, MO.getGlobal());
    Symbol = Ctx->getOrCreateSymbol(Name);
    Offset += MO.getOffset();
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-144
```cpp
  case MachineOperand::MO_BlockAddress:
    Symbol = AsmPrinter.GetBlockAddressSymbol(MO.getBlockAddress());
    Offset += MO.getOffset();
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 146-149
```cpp
  case MachineOperand::MO_ExternalSymbol:
    Symbol = AsmPrinter.GetExternalSymbolSymbol(MO.getSymbolName());
    Offset += MO.getOffset();
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 151-154
```cpp
  case MachineOperand::MO_MCSymbol:
    Symbol = MO.getMCSymbol();
    Offset += MO.getOffset();
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 156-158
```cpp
  case MachineOperand::MO_JumpTableIndex:
    Symbol = AsmPrinter.GetJTISymbol(MO.getIndex());
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 160-163
```cpp
  case MachineOperand::MO_ConstantPoolIndex:
    Symbol = AsmPrinter.GetCPISymbol(MO.getIndex());
    Offset += MO.getOffset();
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 165-167
```cpp
  default:
    llvm_unreachable("<unknown operand type>");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 169-169
```cpp
  const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, *Ctx);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 171-175
```cpp
  if (Offset) {
    // Note: Offset can also be negative
    Expr = MCBinaryExpr::createAdd(Expr, MCConstantExpr::create(Offset, *Ctx),
                                   *Ctx);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 177-180
```cpp
  if (IsGpOff)
    Expr = Mips::createGpOff(Expr, TargetKind, *Ctx);
  else if (TargetKind != Mips::S_None)
    Expr = MCSpecifierExpr::create(Expr, TargetKind, *Ctx);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 182-183
```cpp
  return MCOperand::createExpr(Expr);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 185-187
```cpp
MCOperand MipsMCInstLower::LowerOperand(const MachineOperand &MO,
                                        int64_t offset) const {
  MachineOperandType MOTy = MO.getType();
```
- EN: Implements `MipsMCInstLower::LowerOperand`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::LowerOperand`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 189-203
```cpp
  switch (MOTy) {
  default: llvm_unreachable("unknown operand type");
  case MachineOperand::MO_Register:
    // Ignore all implicit register operands.
    if (MO.isImplicit()) break;
    return MCOperand::createReg(MO.getReg());
  case MachineOperand::MO_Immediate:
    return MCOperand::createImm(MO.getImm() + offset);
  case MachineOperand::MO_MachineBasicBlock:
  case MachineOperand::MO_GlobalAddress:
  case MachineOperand::MO_ExternalSymbol:
  case MachineOperand::MO_MCSymbol:
  case MachineOperand::MO_JumpTableIndex:
  case MachineOperand::MO_ConstantPoolIndex:
  case MachineOperand::MO_BlockAddress:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 204-207
```cpp
    return LowerSymbolOperand(MO, MOTy, offset);
  case MachineOperand::MO_RegisterMask:
    break;
 }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-210
```cpp
  return MCOperand();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 212-217
```cpp
MCOperand MipsMCInstLower::createSub(MachineBasicBlock *BB1,
                                     MachineBasicBlock *BB2,
                                     Mips::Specifier Kind) const {
  const MCSymbolRefExpr *Sym1 = MCSymbolRefExpr::create(BB1->getSymbol(), *Ctx);
  const MCSymbolRefExpr *Sym2 = MCSymbolRefExpr::create(BB2->getSymbol(), *Ctx);
  const MCBinaryExpr *Sub = MCBinaryExpr::createSub(Sym1, Sym2, *Ctx);
```
- EN: Implements `MipsMCInstLower::createSub`, a query/helper routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::createSub`，它是一个围绕MC 指令构造展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 219-220
```cpp
  return MCOperand::createExpr(MCSpecifierExpr::create(Sub, Kind, *Ctx));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 222-224
```cpp
void MipsMCInstLower::
lowerLongBranchLUi(const MachineInstr *MI, MCInst &OutMI) const {
  OutMI.setOpcode(Mips::LUi);
```
- EN: Implements `lowerLongBranchLUi`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `lowerLongBranchLUi`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 226-227
```cpp
  // Lower register operand.
  OutMI.addOperand(LowerOperand(MI->getOperand(0)));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 229-243
```cpp
  Mips::Specifier Spec;
  unsigned TargetFlags = MI->getOperand(1).getTargetFlags();
  switch (TargetFlags) {
  case MipsII::MO_HIGHEST:
    Spec = Mips::S_HIGHEST;
    break;
  case MipsII::MO_HIGHER:
    Spec = Mips::S_HIGHER;
    break;
  case MipsII::MO_ABS_HI:
    Spec = Mips::S_HI;
    break;
  case MipsII::MO_ABS_LO:
    Spec = Mips::S_LO;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 244-246
```cpp
  default:
    report_fatal_error("Unexpected flags for lowerLongBranchLUi");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 248-258
```cpp
  if (MI->getNumOperands() == 2) {
    const MCExpr *Expr =
        MCSymbolRefExpr::create(MI->getOperand(1).getMBB()->getSymbol(), *Ctx);
    const auto *MipsExpr = MCSpecifierExpr::create(Expr, Spec, *Ctx);
    OutMI.addOperand(MCOperand::createExpr(MipsExpr));
  } else if (MI->getNumOperands() == 3) {
    // Create %hi($tgt-$baltgt).
    OutMI.addOperand(createSub(MI->getOperand(1).getMBB(),
                               MI->getOperand(2).getMBB(), Spec));
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 260-262
```cpp
void MipsMCInstLower::lowerLongBranchADDiu(const MachineInstr *MI,
                                           MCInst &OutMI, int Opcode) const {
  OutMI.setOpcode(Opcode);
```
- EN: Implements `MipsMCInstLower::lowerLongBranchADDiu`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::lowerLongBranchADDiu`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 264-278
```cpp
  Mips::Specifier Spec;
  unsigned TargetFlags = MI->getOperand(2).getTargetFlags();
  switch (TargetFlags) {
  case MipsII::MO_HIGHEST:
    Spec = Mips::S_HIGHEST;
    break;
  case MipsII::MO_HIGHER:
    Spec = Mips::S_HIGHER;
    break;
  case MipsII::MO_ABS_HI:
    Spec = Mips::S_HI;
    break;
  case MipsII::MO_ABS_LO:
    Spec = Mips::S_LO;
    break;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 279-281
```cpp
  default:
    report_fatal_error("Unexpected flags for lowerLongBranchADDiu");
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 283-287
```cpp
  // Lower two register operands.
  for (unsigned I = 0, E = 2; I != E; ++I) {
    const MachineOperand &MO = MI->getOperand(I);
    OutMI.addOperand(LowerOperand(MO));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 289-300
```cpp
  if (MI->getNumOperands() == 3) {
    // Lower register operand.
    const MCExpr *Expr =
        MCSymbolRefExpr::create(MI->getOperand(2).getMBB()->getSymbol(), *Ctx);
    const auto *MipsExpr = MCSpecifierExpr::create(Expr, Spec, *Ctx);
    OutMI.addOperand(MCOperand::createExpr(MipsExpr));
  } else if (MI->getNumOperands() == 4) {
    // Create %lo($tgt-$baltgt) or %hi($tgt-$baltgt).
    OutMI.addOperand(createSub(MI->getOperand(2).getMBB(),
                               MI->getOperand(3).getMBB(), Spec));
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 302-316
```cpp
bool MipsMCInstLower::lowerLongBranch(const MachineInstr *MI,
                                      MCInst &OutMI) const {
  switch (MI->getOpcode()) {
  default:
    return false;
  case Mips::LONG_BRANCH_LUi:
  case Mips::LONG_BRANCH_LUi2Op:
  case Mips::LONG_BRANCH_LUi2Op_64:
    lowerLongBranchLUi(MI, OutMI);
    return true;
  case Mips::LONG_BRANCH_ADDiu:
  case Mips::LONG_BRANCH_ADDiu2Op:
    lowerLongBranchADDiu(MI, OutMI, Mips::ADDiu);
    return true;
  case Mips::LONG_BRANCH_DADDiu:
```
- EN: Implements `MipsMCInstLower::lowerLongBranch`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::lowerLongBranch`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 317-321
```cpp
  case Mips::LONG_BRANCH_DADDiu2Op:
    lowerLongBranchADDiu(MI, OutMI, Mips::DADDiu);
    return true;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 323-325
```cpp
void MipsMCInstLower::Lower(const MachineInstr *MI, MCInst &OutMI) const {
  if (lowerLongBranch(MI, OutMI))
    return;
```
- EN: Implements `MipsMCInstLower::Lower`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsMCInstLower::Lower`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 327-327
```cpp
  OutMI.setOpcode(MI->getOpcode());
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 329-330
```cpp
  for (const MachineOperand &MO : MI->operands()) {
    MCOperand MCOp = LowerOperand(MO);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 332-335
```cpp
    if (MCOp.isValid())
      OutMI.addOperand(MCOp);
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

## Key Concepts / 关键概念

- EN: Primary role: lowering from MachineInstr/MachineOperand to MC layer objects.
  - CN: 核心职责：从 MachineInstr/MachineOperand 到 MC 层对象的降级。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsMCInstLower.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MipsAsmPrinter.h`.
  - CN: 后端本地头文件：`MipsMCInstLower.h`, `MCTargetDesc/MipsBaseInfo.h`, `MCTargetDesc/MipsMCAsmInfo.h`, `MipsAsmPrinter.h`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/Support/ErrorHandling.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/Support/ErrorHandling.h`。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
