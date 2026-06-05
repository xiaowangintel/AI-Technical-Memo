# PPCMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCMCInstLower.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCMCInstLower.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCMCInstLower.cpp - Convert PPC MachineInstr to an MCInst --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
//
// This file contains code to lower PPC MachineInstrs to their corresponding
// MCInst records.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file contains code to lower PPC MachineInstrs to their corresponding". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file contains code to lower PPC MachineInstrs to their corresponding”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/PPCMCAsmInfo.h"
#include "PPC.h"
#include "PPCMachineFunctionInfo.h"
#include "PPCSubtarget.h"
#include "llvm/ADT/SmallString.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 19-25

```cpp
#include "llvm/ADT/Twine.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineModuleInfoImpls.h"
#include "llvm/CodeGen/TargetLowering.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/GlobalValue.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 26-63

```cpp
#include "llvm/IR/Mangler.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
using namespace llvm;

static MCSymbol *GetSymbolFromOperand(const MachineOperand &MO,
                                      AsmPrinter &AP) {
  if (MO.isGlobal()) {
    // Get the symbol from the global, accounting for XCOFF-specific
    // intricacies (see TargetLoweringObjectFileXCOFF::getTargetSymbol).
    const GlobalValue *GV = MO.getGlobal();
    return AP.getSymbol(GV);
  }

  assert(MO.isSymbol() && "Isn't a symbol reference");

  SmallString<128> Name;
  const DataLayout &DL = AP.getDataLayout();
  Mangler::getNameWithPrefix(Name, MO.getSymbolName(), DL);

  MCContext &Ctx = AP.OutContext;
  MCSymbol *Sym = Ctx.getOrCreateSymbol(Name);
  return Sym;
}

static MCOperand GetSymbolRef(const MachineOperand &MO, const MCSymbol *Symbol,
                              AsmPrinter &Printer) {
  MCContext &Ctx = Printer.OutContext;
  PPCMCExpr::Specifier RefKind = PPC::S_None;

  unsigned access = MO.getTargetFlags();

  switch (access) {
    case PPCII::MO_TPREL_LO:
      RefKind = PPC::S_TPREL_LO;
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 64-101

```cpp
      break;
    case PPCII::MO_TPREL_HA:
      RefKind = PPC::S_TPREL_HA;
      break;
    case PPCII::MO_DTPREL_LO:
      RefKind = PPC::S_DTPREL_LO;
      break;
    case PPCII::MO_TLSLD_LO:
      RefKind = PPC::S_GOT_TLSLD_LO;
      break;
    case PPCII::MO_TOC_LO:
      RefKind = PPC::S_TOC_LO;
      break;
    case PPCII::MO_TLS:
      RefKind = PPC::S_TLS;
      break;
    case PPCII::MO_TLS_PCREL_FLAG:
      RefKind = PPC::S_TLS_PCREL;
      break;
  }

  const TargetMachine &TM = Printer.TM;
  const MachineInstr *MI = MO.getParent();
  const MachineFunction *MF = MI->getMF();

  if (MO.getTargetFlags() == PPCII::MO_PLT)
    RefKind = PPC::S_PLT;
  else if (MO.getTargetFlags() == PPCII::MO_PCREL_FLAG)
    RefKind = PPC::S_PCREL;
  else if (MO.getTargetFlags() == PPCII::MO_GOT_PCREL_FLAG)
    RefKind = PPC::S_GOT_PCREL;
  else if (MO.getTargetFlags() == PPCII::MO_TPREL_PCREL_FLAG)
    RefKind = PPC::S_TPREL;
  else if (MO.getTargetFlags() == PPCII::MO_GOT_TLSGD_PCREL_FLAG)
    RefKind = PPC::S_GOT_TLSGD_PCREL;
  else if (MO.getTargetFlags() == PPCII::MO_GOT_TLSLD_PCREL_FLAG)
    RefKind = PPC::S_GOT_TLSLD_PCREL;
  else if (MO.getTargetFlags() == PPCII::MO_GOT_TPREL_PCREL_FLAG)
```
- **EN**: Declares function entry points including `getParent`, `getMF`, `getTargetFlags` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `getParent`, `getMF`, `getTargetFlags`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 102-139

```cpp
    RefKind = PPC::S_GOT_TPREL_PCREL;
  else if (MO.getTargetFlags() == PPCII::MO_TPREL_FLAG ||
           MO.getTargetFlags() == PPCII::MO_TLSLD_FLAG) {
    assert(MO.isGlobal() && "Only expecting a global MachineOperand here!");
    TLSModel::Model Model = TM.getTLSModel(MO.getGlobal());
    const PPCFunctionInfo *FuncInfo = MF->getInfo<PPCFunctionInfo>();
    // For the local-[exec|dynamic] TLS model, we may generate the offset from
    // the TLS base as an immediate operand (instead of using a TOC entry). Set
    // the relocation type in case the result is used for purposes other than a
    // TOC reference. In TOC reference cases, this result is discarded.
    if (Model == TLSModel::LocalExec)
      RefKind = PPC::S_AIX_TLSLE;
    else if (Model == TLSModel::LocalDynamic &&
             FuncInfo->isAIXFuncUseTLSIEForLD())
      // On AIX, TLS model opt may have turned local-dynamic accesses into
      // initial-exec accesses.
      RefKind = PPC::S_AIX_TLSIE;
    else if (Model == TLSModel::LocalDynamic)
      RefKind = PPC::S_AIX_TLSLD;
  }

  const Module *M = MF->getFunction().getParent();
  const PPCSubtarget *Subtarget = &(MF->getSubtarget<PPCSubtarget>());

  unsigned MIOpcode = MI->getOpcode();
  assert((Subtarget->isUsingPCRelativeCalls() || MIOpcode != PPC::BL8_NOTOC) &&
         "BL8_NOTOC is only valid when using PC Relative Calls.");
  if (Subtarget->isUsingPCRelativeCalls()) {
    if (MIOpcode == PPC::TAILB || MIOpcode == PPC::TAILB8 ||
        MIOpcode == PPC::TCRETURNdi || MIOpcode == PPC::TCRETURNdi8 ||
        MIOpcode == PPC::BL8_NOTOC || MIOpcode == PPC::BL8_NOTOC_RM) {
      RefKind = PPC::S_NOTOC;
    }
    if (MO.getTargetFlags() == PPCII::MO_PCREL_OPT_FLAG)
      RefKind = PPC::S_PCREL_OPT;
  }

  const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, RefKind, Ctx);
```
- **EN**: Implements helper routine(s) `getTargetFlags`, `isGlobal`, `getTLSModel` for this portion of the PowerPC backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `getTargetFlags`, `isGlobal`, `getTLSModel`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 140-177

```cpp
  // If -msecure-plt -fPIC, add 32768 to symbol.
  if (Subtarget->isSecurePlt() && TM.isPositionIndependent() &&
      M->getPICLevel() == PICLevel::BigPIC &&
      MO.getTargetFlags() == PPCII::MO_PLT)
    Expr =
        MCBinaryExpr::createAdd(Expr, MCConstantExpr::create(32768, Ctx), Ctx);

  if (!MO.isJTI() && MO.getOffset())
    Expr = MCBinaryExpr::createAdd(Expr,
                                   MCConstantExpr::create(MO.getOffset(), Ctx),
                                   Ctx);

  // Subtract off the PIC base if required.
  if (MO.getTargetFlags() == PPCII::MO_PIC_FLAG ||
      MO.getTargetFlags() == PPCII::MO_PIC_HA_FLAG ||
      MO.getTargetFlags() == PPCII::MO_PIC_LO_FLAG) {
    const MachineFunction *MF = MO.getParent()->getParent()->getParent();

    const MCExpr *PB = MCSymbolRefExpr::create(MF->getPICBaseSymbol(), Ctx);
    Expr = MCBinaryExpr::createSub(Expr, PB, Ctx);
  }

  // Add ha16() / lo16() markers if required.
  switch (access) {
    case PPCII::MO_LO:
    case PPCII::MO_PIC_LO_FLAG:
      Expr = MCSpecifierExpr::create(Expr, PPC::S_LO, Ctx);
      break;
    case PPCII::MO_HA:
    case PPCII::MO_PIC_HA_FLAG:
      Expr = MCSpecifierExpr::create(Expr, PPC::S_HA, Ctx);
      break;
  }

  return MCOperand::createExpr(Expr);
}

void llvm::LowerPPCMachineInstrToMCInst(const MachineInstr *MI, MCInst &OutMI,
```
- **EN**: Implements helper routine(s) `isSecurePlt`, `isPositionIndependent`, `getPICLevel` for this portion of the PowerPC backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `isSecurePlt`, `isPositionIndependent`, `getPICLevel`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 178-215

```cpp
                                        AsmPrinter &AP) {
  OutMI.setOpcode(MI->getOpcode());

  for (const MachineOperand &MO : MI->operands()) {
    MCOperand MCOp;
    if (LowerPPCMachineOperandToMCOperand(MO, MCOp, AP))
      OutMI.addOperand(MCOp);
  }
}

bool llvm::LowerPPCMachineOperandToMCOperand(const MachineOperand &MO,
                                             MCOperand &OutMO, AsmPrinter &AP) {
  switch (MO.getType()) {
  default:
    llvm_unreachable("unknown operand type");
  case MachineOperand::MO_Register:
    assert(!MO.getSubReg() && "Subregs should be eliminated!");
    assert(MO.getReg() > PPC::NoRegister &&
           MO.getReg() < PPC::NUM_TARGET_REGS &&
           "Invalid register for this target!");
    // ISA instructions refer to the containing dmr reg.
    if (PPC::isDMRROWpRegister(MO.getReg())) {
      OutMO =
          MCOperand::createReg(PPC::DMR0 + (MO.getReg() - PPC::DMRROWp0) / 4);
      return true;
    }
    // Ignore all implicit register operands.
    if (MO.isImplicit())
      return false;
    OutMO = MCOperand::createReg(MO.getReg());
    return true;
  case MachineOperand::MO_Immediate:
    OutMO = MCOperand::createImm(MO.getImm());
    return true;
  case MachineOperand::MO_MachineBasicBlock:
    OutMO = MCOperand::createExpr(
        MCSymbolRefExpr::create(MO.getMBB()->getSymbol(), AP.OutContext));
    return true;
```
- **EN**: Implements helper routine(s) `setOpcode`, `getOpcode`, `operands` for this portion of the PowerPC backend MC layer support for the backend. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `setOpcode`, `getOpcode`, `operands`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 216-236

```cpp
  case MachineOperand::MO_GlobalAddress:
  case MachineOperand::MO_ExternalSymbol:
    OutMO = GetSymbolRef(MO, GetSymbolFromOperand(MO, AP), AP);
    return true;
  case MachineOperand::MO_JumpTableIndex:
    OutMO = GetSymbolRef(MO, AP.GetJTISymbol(MO.getIndex()), AP);
    return true;
  case MachineOperand::MO_ConstantPoolIndex:
    OutMO = GetSymbolRef(MO, AP.GetCPISymbol(MO.getIndex()), AP);
    return true;
  case MachineOperand::MO_BlockAddress:
    OutMO =
        GetSymbolRef(MO, AP.GetBlockAddressSymbol(MO.getBlockAddress()), AP);
    return true;
  case MachineOperand::MO_MCSymbol:
    OutMO = GetSymbolRef(MO, MO.getMCSymbol(), AP);
    return true;
  case MachineOperand::MO_RegisterMask:
    return false;
  }
}
```
- **EN**: Declares function entry points including `GetSymbolRef`, `GetSymbolFromOperand`, `GetJTISymbol` that other backend components call later. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `GetSymbolRef`, `GetSymbolFromOperand`, `GetJTISymbol`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/PPCMCAsmInfo.h`
- `PPC.h`
- `PPCMachineFunctionInfo.h`
- `PPCSubtarget.h`
- `llvm/ADT/SmallString.h`
- `llvm/ADT/Twine.h`
- `llvm/CodeGen/AsmPrinter.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineModuleInfoImpls.h`
- `llvm/CodeGen/TargetLowering.h`
- `llvm/IR/DataLayout.h`
- `llvm/IR/GlobalValue.h`
- `llvm/IR/Mangler.h`
- `llvm/IR/Module.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/Target/TargetLoweringObjectFile.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
