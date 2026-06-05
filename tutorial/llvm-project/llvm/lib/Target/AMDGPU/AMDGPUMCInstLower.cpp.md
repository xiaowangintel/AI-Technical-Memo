# AMDGPUMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMCInstLower.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUMCInstLower for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUMCInstLower 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-34: File banner, includes, and setup
```cpp
//===- AMDGPUMCInstLower.cpp - Lower AMDGPU MachineInstr to an MCInst -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Code to lower AMDGPU MachineInstrs to their corresponding MCInst.
//
//===----------------------------------------------------------------------===//
//

#include "AMDGPUMCInstLower.h"
#include "AMDGPU.h"
#include "AMDGPUAsmPrinter.h"
#include "AMDGPUMachineFunctionInfo.h"
#include "MCTargetDesc/AMDGPUInstPrinter.h"
#include "MCTargetDesc/AMDGPUMCExpr.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCObjectStreamer.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/Endian.h"
```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 35-68: Header dependencies and setup
```cpp
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/Format.h"
#include <algorithm>

using namespace llvm;

#include "AMDGPUGenMCPseudoLowering.inc"

AMDGPUMCInstLower::AMDGPUMCInstLower(MCContext &ctx,
                                     const TargetSubtargetInfo &st,
                                     const AsmPrinter &ap):
  Ctx(ctx), ST(st), AP(ap) { }

static AMDGPUMCExpr::Specifier getSpecifier(unsigned MOFlags) {
  switch (MOFlags) {
  default:
    return AMDGPUMCExpr::S_None;
  case SIInstrInfo::MO_GOTPCREL:
  case SIInstrInfo::MO_GOTPCREL64:
    return AMDGPUMCExpr::S_GOTPCREL;
  case SIInstrInfo::MO_GOTPCREL32_LO:
    return AMDGPUMCExpr::S_GOTPCREL32_LO;
  case SIInstrInfo::MO_GOTPCREL32_HI:
    return AMDGPUMCExpr::S_GOTPCREL32_HI;
  case SIInstrInfo::MO_REL32_LO:
    return AMDGPUMCExpr::S_REL32_LO;
  case SIInstrInfo::MO_REL32_HI:
    return AMDGPUMCExpr::S_REL32_HI;
  case SIInstrInfo::MO_REL64:
    return AMDGPUMCExpr::S_REL64;
  case SIInstrInfo::MO_ABS32_LO:
    return AMDGPUMCExpr::S_ABS32_LO;
  case SIInstrInfo::MO_ABS32_HI:
    return AMDGPUMCExpr::S_ABS32_HI;
```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `AMDGPUMCInstLower::AMDGPUMCInstLower`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`AMDGPUMCInstLower::AMDGPUMCInstLower`。

### Lines 69-102: Implements AMDGPUMCInstLower::lowerOperand
```cpp
  case SIInstrInfo::MO_ABS64:
    return AMDGPUMCExpr::S_ABS64;
  }
}

bool AMDGPUMCInstLower::lowerOperand(const MachineOperand &MO,
                                     MCOperand &MCOp) const {
  switch (MO.getType()) {
  default:
    break;
  case MachineOperand::MO_Immediate:
    MCOp = MCOperand::createImm(MO.getImm());
    return true;
  case MachineOperand::MO_Register:
    MCOp = MCOperand::createReg(AMDGPU::getMCReg(MO.getReg(), ST));
    return true;
  case MachineOperand::MO_MachineBasicBlock:
    MCOp = MCOperand::createExpr(
        MCSymbolRefExpr::create(MO.getMBB()->getSymbol(), Ctx));
    return true;
  case MachineOperand::MO_GlobalAddress: {
    const GlobalValue *GV = MO.getGlobal();
    SmallString<128> SymbolName;
    AP.getNameWithPrefix(SymbolName, GV);
    MCSymbol *Sym = Ctx.getOrCreateSymbol(SymbolName);
    const MCExpr *Expr =
        MCSymbolRefExpr::create(Sym, getSpecifier(MO.getTargetFlags()), Ctx);
    int64_t Offset = MO.getOffset();
    if (Offset != 0) {
      Expr = MCBinaryExpr::createAdd(Expr,
                                     MCConstantExpr::create(Offset, Ctx), Ctx);
    }
    MCOp = MCOperand::createExpr(Expr);
    return true;
```
**EN:** This section contains concrete logic for AMDGPUMCInstLower::lowerOperand. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUMCInstLower::lowerOperand`, `MCOperand::createImm`, `MCOperand::createReg`.
**CN:** 本节包含与 AMDGPUMCInstLower::lowerOperand 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUMCInstLower::lowerOperand`, `MCOperand::createImm`, `MCOperand::createReg`。

### Lines 103-132: Conditional logic and checks
```cpp
  }
  case MachineOperand::MO_ExternalSymbol: {
    MCSymbol *Sym = Ctx.getOrCreateSymbol(StringRef(MO.getSymbolName()));
    const MCSymbolRefExpr *Expr = MCSymbolRefExpr::create(Sym, Ctx);
    MCOp = MCOperand::createExpr(Expr);
    return true;
  }
  case MachineOperand::MO_RegisterMask:
    // Regmasks are like implicit defs.
    return false;
  case MachineOperand::MO_MCSymbol:
    if (MO.getTargetFlags() == SIInstrInfo::MO_FAR_BRANCH_OFFSET) {
      MCSymbol *Sym = MO.getMCSymbol();
      MCOp = MCOperand::createExpr(Sym->getVariableValue());
      return true;
    }
    break;
  }
  llvm_unreachable("unknown operand type");
}

// Lower true16 D16 Pseudo instruction to d16_lo/d16_hi MCInst based on
// Dst/Data's .l/.h selection
void AMDGPUMCInstLower::lowerT16D16Helper(const MachineInstr *MI,
                                          MCInst &OutMI) const {
  unsigned Opcode = MI->getOpcode();
  const auto *TII = static_cast<const SIInstrInfo*>(ST.getInstrInfo());
  const SIRegisterInfo &TRI = TII->getRegisterInfo();
  const auto *Info = AMDGPU::getT16D16Helper(Opcode);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCSymbolRefExpr::create`, `MCOperand::createExpr`, `AMDGPUMCInstLower::lowerT16D16Helper`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCSymbolRefExpr::create`, `MCOperand::createExpr`, `AMDGPUMCInstLower::lowerT16D16Helper`。

### Lines 133-159: Conditional logic and checks
```cpp
  llvm::AMDGPU::OpName OpName;
  if (TII->isDS(Opcode)) {
    if (MI->mayLoad())
      OpName = llvm::AMDGPU::OpName::vdst;
    else if (MI->mayStore())
      OpName = llvm::AMDGPU::OpName::data0;
    else
      llvm_unreachable("LDS load or store expected");
  } else {
    OpName = AMDGPU::hasNamedOperand(Opcode, llvm::AMDGPU::OpName::vdata)
                 ? llvm::AMDGPU::OpName::vdata
                 : llvm::AMDGPU::OpName::vdst;
  }

  // select Dst/Data
  int VDstOrVDataIdx = AMDGPU::getNamedOperandIdx(Opcode, OpName);
  const MachineOperand &MIVDstOrVData = MI->getOperand(VDstOrVDataIdx);

  // select hi/lo MCInst
  bool IsHi = AMDGPU::isHi16Reg(MIVDstOrVData.getReg(), TRI);
  Opcode = IsHi ? Info->HiOp : Info->LoOp;

  int MCOpcode = TII->pseudoToMCOpcode(Opcode);
  assert(MCOpcode != -1 &&
         "Pseudo instruction doesn't have a target-specific version");
  OutMI.setOpcode(MCOpcode);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::hasNamedOperand`, `AMDGPU::getNamedOperandIdx`, `AMDGPU::isHi16Reg`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::hasNamedOperand`, `AMDGPU::getNamedOperandIdx`, `AMDGPU::isHi16Reg`。

### Lines 160-183: Conditional logic and checks
```cpp
  // lower operands
  for (int I = 0, E = MI->getNumExplicitOperands(); I < E; I++) {
    const MachineOperand &MO = MI->getOperand(I);
    MCOperand MCOp;
    if (I == VDstOrVDataIdx)
      MCOp = MCOperand::createReg(TRI.get32BitRegister(MIVDstOrVData.getReg()));
    else
      lowerOperand(MO, MCOp);
    OutMI.addOperand(MCOp);
  }

  if (AMDGPU::hasNamedOperand(MCOpcode, AMDGPU::OpName::vdst_in)) {
    MCOperand MCOp;
    lowerOperand(MIVDstOrVData, MCOp);
    OutMI.addOperand(MCOp);
  }
}

void AMDGPUMCInstLower::lowerT16FmaMixFP16(const MachineInstr *MI,
                                           MCInst &OutMI) const {
  unsigned Opcode = MI->getOpcode();
  const auto *TII = static_cast<const SIInstrInfo *>(ST.getInstrInfo());
  const SIRegisterInfo &TRI = TII->getRegisterInfo();

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `MCOperand::createReg`, `AMDGPU::hasNamedOperand`, `AMDGPUMCInstLower::lowerT16FmaMixFP16`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`MCOperand::createReg`, `AMDGPU::hasNamedOperand`, `AMDGPUMCInstLower::lowerT16FmaMixFP16`。

### Lines 184-215: Implements AMDGPU::getNamedOperandIdx
```cpp
  int VDstIdx = AMDGPU::getNamedOperandIdx(Opcode, llvm::AMDGPU::OpName::vdst);
  const MachineOperand &VDst = MI->getOperand(VDstIdx);
  bool IsHi = AMDGPU::isHi16Reg(VDst.getReg(), TRI);
  switch (Opcode) {
  case AMDGPU::V_FMA_MIX_F16_t16:
    Opcode = IsHi ? AMDGPU::V_FMA_MIXHI_F16 : AMDGPU::V_FMA_MIXLO_F16;
    break;
  case AMDGPU::V_FMA_MIX_BF16_t16:
    Opcode = IsHi ? AMDGPU::V_FMA_MIXHI_BF16 : AMDGPU::V_FMA_MIXLO_BF16;
    break;
  }
  int MCOpcode = TII->pseudoToMCOpcode(Opcode);
  assert(MCOpcode != -1 &&
         "Pseudo instruction doesn't have a target-specific version");
  OutMI.setOpcode(MCOpcode);

  // lower operands
  for (int I = 0, E = MI->getNumExplicitOperands(); I < E; I++) {
    const MachineOperand &MO = MI->getOperand(I);
    MCOperand MCOp;
    if (I == VDstIdx)
      MCOp = MCOperand::createReg(TRI.get32BitRegister(VDst.getReg()));
    else
      lowerOperand(MO, MCOp);
    OutMI.addOperand(MCOp);
  }
}

void AMDGPUMCInstLower::lower(const MachineInstr *MI, MCInst &OutMI) const {
  unsigned Opcode = MI->getOpcode();
  const auto *TII = static_cast<const SIInstrInfo *>(ST.getInstrInfo());

```
**EN:** This section contains concrete logic for AMDGPU::getNamedOperandIdx. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPU::getNamedOperandIdx`, `AMDGPU::isHi16Reg`, `MCOperand::createReg`.
**CN:** 本节包含与 AMDGPU::getNamedOperandIdx 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPU::getNamedOperandIdx`, `AMDGPU::isHi16Reg`, `MCOperand::createReg`。

### Lines 216-244: Conditional logic and checks
```cpp
  // FIXME: Should be able to handle this with lowerPseudoInstExpansion. We
  // need to select it to the subtarget specific version, and there's no way to
  // do that with a single pseudo source operation.
  if (Opcode == AMDGPU::S_SETPC_B64_return)
    Opcode = AMDGPU::S_SETPC_B64;
  else if (Opcode == AMDGPU::SI_CALL) {
    // SI_CALL is just S_SWAPPC_B64 with an additional operand to track the
    // called function (which we need to remove here).
    OutMI.setOpcode(TII->pseudoToMCOpcode(AMDGPU::S_SWAPPC_B64));
    MCOperand Dest, Src;
    lowerOperand(MI->getOperand(0), Dest);
    lowerOperand(MI->getOperand(1), Src);
    OutMI.addOperand(Dest);
    OutMI.addOperand(Src);
    return;
  } else if (Opcode == AMDGPU::SI_TCRETURN ||
             Opcode == AMDGPU::SI_TCRETURN_GFX ||
             Opcode == AMDGPU::SI_TCRETURN_CHAIN) {
    // TODO: How to use branch immediate and avoid register+add?
    Opcode = AMDGPU::S_SETPC_B64;
  } else if (AMDGPU::getT16D16Helper(Opcode)) {
    lowerT16D16Helper(MI, OutMI);
    return;
  } else if (Opcode == AMDGPU::V_FMA_MIX_F16_t16 ||
             Opcode == AMDGPU::V_FMA_MIX_BF16_t16) {
    lowerT16FmaMixFP16(MI, OutMI);
    return;
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::getT16D16Helper`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::getT16D16Helper`。

### Lines 245-276: Declares pseudoToMCOpcode
```cpp
  int MCOpcode = TII->pseudoToMCOpcode(Opcode);
  if (MCOpcode == -1) {
    LLVMContext &C = MI->getMF()->getFunction().getContext();
    C.emitError("AMDGPUMCInstLower::lower - Pseudo instruction doesn't have "
                "a target-specific version: " + Twine(MI->getOpcode()));
    return;
  }

  OutMI.setOpcode(MCOpcode);

  for (const MachineOperand &MO : MI->explicit_operands()) {
    MCOperand MCOp;
    lowerOperand(MO, MCOp);
    OutMI.addOperand(MCOp);
  }

  int FIIdx = AMDGPU::getNamedOperandIdx(MCOpcode, AMDGPU::OpName::fi);
  if (FIIdx >= (int)OutMI.getNumOperands())
    OutMI.addOperand(MCOperand::createImm(0));
}

bool AMDGPUAsmPrinter::lowerOperand(const MachineOperand &MO,
                                    MCOperand &MCOp) const {
  const GCNSubtarget &STI = MF->getSubtarget<GCNSubtarget>();
  AMDGPUMCInstLower MCInstLowering(OutContext, STI, *this);
  return MCInstLowering.lowerOperand(MO, MCOp);
}

const MCExpr *AMDGPUAsmPrinter::lowerConstant(const Constant *CV,
                                              const Constant *BaseCV,
                                              uint64_t Offset) {

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `AMDGPU::getNamedOperandIdx`, `MCOperand::createImm`, `AMDGPUAsmPrinter::lowerOperand`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`AMDGPU::getNamedOperandIdx`, `MCOperand::createImm`, `AMDGPUAsmPrinter::lowerOperand`。

### Lines 277-310: Conditional logic and checks
```cpp
  // Intercept LDS variables with known addresses
  if (const GlobalVariable *GV = dyn_cast<const GlobalVariable>(CV)) {
    if (std::optional<uint32_t> Address =
            AMDGPUMachineFunctionInfo::getLDSAbsoluteAddress(*GV)) {
      auto *IntTy = Type::getInt32Ty(CV->getContext());
      return AsmPrinter::lowerConstant(ConstantInt::get(IntTy, *Address),
                                       BaseCV, Offset);
    }
  }

  if (const MCExpr *E = lowerAddrSpaceCast(CV, OutContext))
    return E;
  return AsmPrinter::lowerConstant(CV, BaseCV, Offset);
}

static void emitVGPRBlockComment(const MachineInstr *MI, const SIInstrInfo *TII,
                                 const TargetRegisterInfo *TRI,
                                 const SIMachineFunctionInfo *MFI,
                                 MCStreamer &OS) {
  // The instruction will only transfer a subset of the registers in the block,
  // based on the mask that is stored in m0. We could search for the instruction
  // that sets m0, but most of the time we'll already have the mask stored in
  // the machine function info. Try to use that. This assumes that we only use
  // block loads/stores for CSR spills.
  Register RegBlock =
      TII->getNamedOperand(*MI, MI->mayLoad() ? AMDGPU::OpName::vdst
                                              : AMDGPU::OpName::vdata)
          ->getReg();
  Register FirstRegInBlock = TRI->getSubReg(RegBlock, AMDGPU::sub0);
  uint32_t Mask = MFI->getMaskForVGPRBlockOps(RegBlock);

  if (!Mask)
    return; // Nothing to report

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPUMachineFunctionInfo::getLDSAbsoluteAddress`, `Type::getInt32Ty`, `AsmPrinter::lowerConstant`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPUMachineFunctionInfo::getLDSAbsoluteAddress`, `Type::getInt32Ty`, `AsmPrinter::lowerConstant`。

### Lines 311-344: Declares emitRawComment
```cpp
  SmallString<512> TransferredRegs;
  for (unsigned I = 0; I < sizeof(Mask) * 8; ++I) {
    if (Mask & (1 << I)) {
      (llvm::Twine(" ") + TRI->getRegAsmName(FirstRegInBlock + I))
          .toVector(TransferredRegs);
    }
  }

  OS.emitRawComment(" transferring at most " + TransferredRegs);
}

void AMDGPUAsmPrinter::emitInstruction(const MachineInstr *MI) {
  if (MI->isCall())
    collectCallEdge(*MI);

  // FIXME: Enable feature predicate checks once all the test pass.
  // AMDGPU_MC::verifyInstructionPredicates(MI->getOpcode(),
  //                                        getSubtargetInfo().getFeatureBits());

  if (MCInst OutInst; lowerPseudoInstExpansion(MI, OutInst)) {
    EmitToStreamer(*OutStreamer, OutInst);
    return;
  }

  const GCNSubtarget &STI = MF->getSubtarget<GCNSubtarget>();
  AMDGPUMCInstLower MCInstLowering(OutContext, STI, *this);

  StringRef Err;
  if (!STI.getInstrInfo()->verifyInstruction(*MI, Err)) {
    LLVMContext &C = MI->getMF()->getFunction().getContext();
    C.emitError("Illegal instruction detected: " + Err);
    MI->print(errs());
  }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `llvm::Twine`, `AMDGPUAsmPrinter::emitInstruction`, `AMDGPU_MC::verifyInstructionPredicates`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`llvm::Twine`, `AMDGPUAsmPrinter::emitInstruction`, `AMDGPU_MC::verifyInstructionPredicates`。

### Lines 345-373: Conditional logic and checks
```cpp
  if (MI->isBundle()) {
    const MachineBasicBlock *MBB = MI->getParent();
    MachineBasicBlock::const_instr_iterator I = ++MI->getIterator();
    while (I != MBB->instr_end() && I->isInsideBundle()) {
      emitInstruction(&*I);
      ++I;
    }
  } else {
    // We don't want these pseudo instructions encoded. They are
    // placeholder instructions and should only be printed as
    // comments.
    if (MI->getOpcode() == AMDGPU::SI_RETURN_TO_EPILOG) {
      if (isVerbose())
        OutStreamer->emitRawComment(" return to shader part epilog");
      return;
    }

    if (MI->getOpcode() == AMDGPU::WAVE_BARRIER) {
      if (isVerbose())
        OutStreamer->emitRawComment(" wave barrier");
      return;
    }

    if (MI->getOpcode() == AMDGPU::ASYNCMARK) {
      if (isVerbose())
        OutStreamer->emitRawComment(" asyncmark");
      return;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 374-404: Conditional logic and checks
```cpp
    if (MI->getOpcode() == AMDGPU::WAIT_ASYNCMARK) {
      if (isVerbose()) {
        OutStreamer->emitRawComment(" wait_asyncmark(" +
                                    Twine(MI->getOperand(0).getImm()) + ")");
      }
      return;
    }

    if (MI->getOpcode() == AMDGPU::SCHED_BARRIER) {
      if (isVerbose()) {
        std::string HexString;
        raw_string_ostream HexStream(HexString);
        HexStream << format_hex(MI->getOperand(0).getImm(), 10, true);
        OutStreamer->emitRawComment(" sched_barrier mask(" + HexString + ")");
      }
      return;
    }

    if (MI->getOpcode() == AMDGPU::SCHED_GROUP_BARRIER) {
      if (isVerbose()) {
        std::string HexString;
        raw_string_ostream HexStream(HexString);
        HexStream << format_hex(MI->getOperand(0).getImm(), 10, true);
        OutStreamer->emitRawComment(
            " sched_group_barrier mask(" + HexString + ") size(" +
            Twine(MI->getOperand(1).getImm()) + ") SyncID(" +
            Twine(MI->getOperand(2).getImm()) + ")");
      }
      return;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 405-436: Conditional logic and checks
```cpp
    if (MI->getOpcode() == AMDGPU::IGLP_OPT) {
      if (isVerbose()) {
        std::string HexString;
        raw_string_ostream HexStream(HexString);
        HexStream << format_hex(MI->getOperand(0).getImm(), 10, true);
        OutStreamer->emitRawComment(" iglp_opt mask(" + HexString + ")");
      }
      return;
    }

    if (MI->getOpcode() == AMDGPU::SI_MASKED_UNREACHABLE) {
      if (isVerbose())
        OutStreamer->emitRawComment(" divergent unreachable");
      return;
    }

    if (MI->isMetaInstruction()) {
      if (isVerbose())
        OutStreamer->emitRawComment(" meta instruction");
      return;
    }

    unsigned Opc = MI->getOpcode();
    if (LLVM_UNLIKELY(Opc == TargetOpcode::STATEPOINT ||
                      Opc == TargetOpcode::STACKMAP ||
                      Opc == TargetOpcode::PATCHPOINT)) {
      LLVMContext &Ctx = MI->getMF()->getFunction().getContext();
      Ctx.emitError("unhandled statepoint-like instruction");
      OutStreamer->emitRawComment("unsupported statepoint/stackmap/patchpoint");
      return;
    }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 437-467: Conditional logic and checks
```cpp
    if (isVerbose())
      if (STI.getInstrInfo()->isBlockLoadStore(MI->getOpcode()))
        emitVGPRBlockComment(MI, STI.getInstrInfo(), STI.getRegisterInfo(),
                             MF->getInfo<SIMachineFunctionInfo>(),
                             *OutStreamer);

    if (isVerbose() && (MI->getOpcode() == AMDGPU::S_SET_VGPR_MSB ||
                        (MI->getOpcode() == AMDGPU::S_SETREG_IMM32_B32 &&
                         STI.has1024AddressableVGPRs()))) {
      std::optional<unsigned> V;
      if (MI->getOpcode() == AMDGPU::S_SETREG_IMM32_B32)
        V = AMDGPU::convertSetRegImmToVgprMSBs(*MI,
                                               STI.hasSetregVGPRMSBFixup());
      else
        V = MI->getOperand(0).getImm() & 0xff;
      if (V.has_value())
        OutStreamer->AddComment(
            " msbs: dst=" + Twine(*V >> 6) + " src0=" + Twine(*V & 3) +
            " src1=" + Twine((*V >> 2) & 3) + " src2=" + Twine((*V >> 4) & 3));
    }

    MCInst TmpInst;
    MCInstLowering.lower(MI, TmpInst);
    EmitToStreamer(*OutStreamer, TmpInst);

    if (DumpCodeInstEmitter) {
      // Disassemble instruction/operands to text
      DisasmLines.resize(DisasmLines.size() + 1);
      std::string &DisasmLine = DisasmLines.back();
      raw_string_ostream DisasmStream(DisasmLine);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::convertSetRegImmToVgprMSBs`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::convertSetRegImmToVgprMSBs`。

### Lines 468-491: Implementation details and local logic
```cpp
      AMDGPUInstPrinter InstPrinter(TM.getMCAsmInfo(), *STI.getInstrInfo(),
                                    *STI.getRegisterInfo());
      InstPrinter.printInst(&TmpInst, 0, StringRef(), STI, DisasmStream);

      // Disassemble instruction/operands to hex representation.
      SmallVector<MCFixup, 4> Fixups;
      SmallVector<char, 16> CodeBytes;

      DumpCodeInstEmitter->encodeInstruction(
          TmpInst, CodeBytes, Fixups, MF->getSubtarget<MCSubtargetInfo>());
      HexLines.resize(HexLines.size() + 1);
      std::string &HexLine = HexLines.back();
      raw_string_ostream HexStream(HexLine);

      for (size_t i = 0; i < CodeBytes.size(); i += 4) {
        unsigned int CodeDWord =
            support::endian::read32le(CodeBytes.data() + i);
        HexStream << format("%s%08X", (i > 0 ? " " : ""), CodeDWord);
      }

      DisasmLineMaxLen = std::max(DisasmLineMaxLen, DisasmLine.size());
    }
  }
}
```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend. Main symbols: `endian::read32le`, `std::max`.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。 主要符号：`endian::read32le`, `std::max`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `AMDGPUMCInstLower::AMDGPUMCInstLower`, `AMDGPUMCInstLower::lowerOperand`, `MCOperand::createImm`, `MCOperand::createReg`, `AMDGPU::getMCReg`, `MCOperand::createExpr`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUMCInstLower.h"`
- `"AMDGPU.h"`
- `"AMDGPUAsmPrinter.h"`
- `"AMDGPUMachineFunctionInfo.h"`
- `"MCTargetDesc/AMDGPUInstPrinter.h"`
- `"MCTargetDesc/AMDGPUMCExpr.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/MachineBasicBlock.h"`
- `"llvm/CodeGen/MachineInstr.h"`
- `"llvm/IR/Constants.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/GlobalVariable.h"`
- `"llvm/MC/MCCodeEmitter.h"`
- `"llvm/MC/MCContext.h"`
- `"llvm/MC/MCExpr.h"`
- `"llvm/MC/MCInst.h"`
- `"llvm/MC/MCObjectStreamer.h"`
- `"llvm/MC/MCStreamer.h"`
- `"llvm/Support/Endian.h"`
- ... and 4 more direct dependencies / 以及另外 4 个直接依赖
