# ARMMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMMCInstLower.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains code to lower ARM MachineInstrs to their corresponding MCInst records.
- 用途 (CN): 实现 ARM 后端中的 `ARMMCInstLower`，重点处理从 MachineInstr/MachineOperand 到 MC 层对象的降级。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- ARMMCInstLower.cpp - Convert ARM MachineInstr to an MCInst --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains code to lower ARM MachineInstrs to their corresponding
// MCInst records.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-28
```cpp
#include "ARM.h"
#include "ARMAsmPrinter.h"
#include "ARMBaseInstrInfo.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMSubtarget.h"
#include "MCTargetDesc/ARMAddressingModes.h"
#include "MCTargetDesc/ARMBaseInfo.h"
#include "MCTargetDesc/ARMMCAsmInfo.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/Constants.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 29-34
```cpp
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstBuilder.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/ErrorHandling.h"
#include <cassert>
#include <cstdint>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 36-36
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 38-42
```cpp
MCOperand ARMAsmPrinter::GetSymbolRef(const MachineOperand &MO,
                                      const MCSymbol *Symbol) {
  auto Specifier = ARM::S_None;
  if (MO.getTargetFlags() & ARMII::MO_SBREL)
    Specifier = ARM::S_SBREL;
```
- EN: Implements `ARMAsmPrinter::GetSymbolRef`, a query/helper routine centered on machine operands. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMAsmPrinter::GetSymbolRef`，它是一个围绕机器操作数展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 44-58
```cpp
  const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
  switch (MO.getTargetFlags() & ARMII::MO_OPTION_MASK) {
  default:
    llvm_unreachable("Unknown target flag on symbol operand");
  case ARMII::MO_NO_FLAG:
    break;
  case ARMII::MO_LO16:
    Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
    Expr = ARM::createLower16(Expr, OutContext);
    break;
  case ARMII::MO_HI16:
    Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
    Expr = ARM::createUpper16(Expr, OutContext);
    break;
  case ARMII::MO_LO_0_7:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 59-73
```cpp
    Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
    Expr = ARM::createLower0_7(Expr, OutContext);
    break;
  case ARMII::MO_LO_8_15:
    Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
    Expr = ARM::createLower8_15(Expr, OutContext);
    break;
  case ARMII::MO_HI_0_7:
    Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
    Expr = ARM::createUpper0_7(Expr, OutContext);
    break;
  case ARMII::MO_HI_8_15:
    Expr = MCSymbolRefExpr::create(Symbol, Specifier, OutContext);
    Expr = ARM::createUpper8_15(Expr, OutContext);
    break;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 74-74
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 76-81
```cpp
  if (!MO.isJTI() && MO.getOffset())
    Expr = MCBinaryExpr::createAdd(Expr,
                                   MCConstantExpr::create(MO.getOffset(),
                                                          OutContext),
                                   OutContext);
  return MCOperand::createExpr(Expr);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 83-83
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 85-99
```cpp
bool ARMAsmPrinter::lowerOperand(const MachineOperand &MO,
                                 MCOperand &MCOp) {
  switch (MO.getType()) {
  default: llvm_unreachable("unknown operand type");
  case MachineOperand::MO_Register:
    // Ignore all implicit register operands.
    if (MO.isImplicit())
      return false;
    assert(!MO.getSubReg() && "Subregs should be eliminated!");
    MCOp = MCOperand::createReg(MO.getReg());
    break;
  case MachineOperand::MO_Immediate:
    MCOp = MCOperand::createImm(MO.getImm());
    break;
  case MachineOperand::MO_MachineBasicBlock:
```
- EN: Implements `ARMAsmPrinter::lowerOperand`, a lowering routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMAsmPrinter::lowerOperand`，它是一个围绕机器基本块展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-114
```cpp
    MCOp = MCOperand::createExpr(MCSymbolRefExpr::create(
        MO.getMBB()->getSymbol(), OutContext));
    break;
  case MachineOperand::MO_GlobalAddress:
    MCOp = GetSymbolRef(MO,
                        GetARMGVSymbol(MO.getGlobal(), MO.getTargetFlags()));
    break;
  case MachineOperand::MO_ExternalSymbol:
    MCOp = GetSymbolRef(MO,
                        GetExternalSymbolSymbol(MO.getSymbolName()));
    break;
  case MachineOperand::MO_JumpTableIndex:
    MCOp = GetSymbolRef(MO, GetJTISymbol(MO.getIndex()));
    break;
  case MachineOperand::MO_ConstantPoolIndex:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 115-129
```cpp
    assert(!MF->getSubtarget<ARMSubtarget>().genExecuteOnly() &&
           "execute-only should not generate constant pools");
    MCOp = GetSymbolRef(MO, GetCPISymbol(MO.getIndex()));
    break;
  case MachineOperand::MO_BlockAddress:
    MCOp = GetSymbolRef(MO, GetBlockAddressSymbol(MO.getBlockAddress()));
    break;
  case MachineOperand::MO_FPImmediate: {
    APFloat Val = MO.getFPImm()->getValueAPF();
    bool ignored;
    Val.convert(APFloat::IEEEdouble(), APFloat::rmTowardZero, &ignored);
    MCOp = MCOperand::createDFPImm(bit_cast<uint64_t>(Val.convertToDouble()));
    break;
  }
  case MachineOperand::MO_RegisterMask:
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 130-134
```cpp
    // Ignore call clobbers.
    return false;
  }
  return true;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 136-138
```cpp
void llvm::LowerARMMachineInstrToMCInst(const MachineInstr *MI, MCInst &OutMI,
                                        ARMAsmPrinter &AP) {
  OutMI.setOpcode(MI->getOpcode());
```
- EN: Implements `llvm::LowerARMMachineInstrToMCInst`, a lowering routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::LowerARMMachineInstrToMCInst`，它是一个围绕MC 指令构造展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 140-154
```cpp
  // In the MC layer, we keep modified immediates in their encoded form
  bool EncodeImms = false;
  switch (MI->getOpcode()) {
  default: break;
  case ARM::MOVi:
  case ARM::MVNi:
  case ARM::CMPri:
  case ARM::CMNri:
  case ARM::TSTri:
  case ARM::TEQri:
  case ARM::MSRi:
  case ARM::ADCri:
  case ARM::ADDri:
  case ARM::ADDSri:
  case ARM::SBCri:
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 155-166
```cpp
  case ARM::SUBri:
  case ARM::SUBSri:
  case ARM::ANDri:
  case ARM::ORRri:
  case ARM::EORri:
  case ARM::BICri:
  case ARM::RSBri:
  case ARM::RSBSri:
  case ARM::RSCri:
    EncodeImms = true;
    break;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 168-179
```cpp
  for (const MachineOperand &MO : MI->operands()) {
    MCOperand MCOp;
    if (AP.lowerOperand(MO, MCOp)) {
      if (MCOp.isImm() && EncodeImms) {
        int32_t Enc = ARM_AM::getSOImmVal(MCOp.getImm());
        if (Enc != -1)
          MCOp.setImm(Enc);
      }
      OutMI.addOperand(MCOp);
    }
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 181-195
```cpp
void ARMAsmPrinter::EmitSled(const MachineInstr &MI, SledKind Kind)
{
  const MachineFunction *MF = MI.getParent()->getParent();
  if (MF->getInfo<ARMFunctionInfo>()->isThumbFunction()) {
    const Function &Fn = MF->getFunction();
    Fn.getContext().diagnose(DiagnosticInfoUnsupported(
        Fn,
        "An attempt to perform XRay instrumentation for a Thumb function (not "
        "supported). Detected when emitting a sled.",
        MI.getDebugLoc()));
    return;
  }
  static const int8_t NoopsInSledCount = 6;
  // We want to emit the following pattern:
  //
```
- EN: Implements `ARMAsmPrinter::EmitSled`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMAsmPrinter::EmitSled`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 196-210
```cpp
  // .Lxray_sled_N:
  //   ALIGN
  //   B #20
  //   ; 6 NOP instructions (24 bytes)
  // .tmpN
  //
  // We need the 24 bytes (6 instructions) because at runtime, we'd be patching
  // over the full 28 bytes (7 instructions) with the following pattern:
  //
  //   PUSH{ r0, lr }
  //   MOVW r0, #<lower 16 bits of function ID>
  //   MOVT r0, #<higher 16 bits of function ID>
  //   MOVW ip, #<lower 16 bits of address of __xray_FunctionEntry/Exit>
  //   MOVT ip, #<higher 16 bits of address of __xray_FunctionEntry/Exit>
  //   BLX ip
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 211-216
```cpp
  //   POP{ r0, lr }
  //
  OutStreamer->emitCodeAlignment(Align(4), &getSubtargetInfo());
  auto CurSled = OutContext.createTempSymbol("xray_sled_", true);
  OutStreamer->emitLabel(CurSled);
  auto Target = OutContext.createTempSymbol();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 218-224
```cpp
  // Emit "B #20" instruction, which jumps over the next 24 bytes (because
  // register pc is 8 bytes ahead of the jump instruction by the moment CPU
  // is executing it).
  // By analogy to ARMAsmPrinter::lowerPseudoInstExpansion() |case ARM::B|.
  // It is not clear why |addReg(0)| is needed (the last operand).
  EmitToStreamer(*OutStreamer, MCInstBuilder(ARM::Bcc).addImm(20)
    .addImm(ARMCC::AL).addReg(0));
```
- EN: Declares `EmitToStreamer`, a emission/printing routine centered on MC instruction construction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitToStreamer`，它是一个围绕MC 指令构造展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 226-226
```cpp
  emitNops(NoopsInSledCount);
```
- EN: Declares `emitNops`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitNops`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 228-230
```cpp
  OutStreamer->emitLabel(Target);
  recordSled(CurSled, MI, Kind, 2);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 232-235
```cpp
void ARMAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER(const MachineInstr &MI)
{
  EmitSled(MI, SledKind::FUNCTION_ENTER);
}
```
- EN: Implements `ARMAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMAsmPrinter::LowerPATCHABLE_FUNCTION_ENTER`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 237-240
```cpp
void ARMAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT(const MachineInstr &MI)
{
  EmitSled(MI, SledKind::FUNCTION_EXIT);
}
```
- EN: Implements `ARMAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMAsmPrinter::LowerPATCHABLE_FUNCTION_EXIT`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 242-245
```cpp
void ARMAsmPrinter::LowerPATCHABLE_TAIL_CALL(const MachineInstr &MI)
{
  EmitSled(MI, SledKind::TAIL_CALL);
}
```
- EN: Implements `ARMAsmPrinter::LowerPATCHABLE_TAIL_CALL`, a lowering routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMAsmPrinter::LowerPATCHABLE_TAIL_CALL`，它是一个围绕机器指令展开的降级例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: lowering from MachineInstr/MachineOperand to MC layer objects.
  - CN: 核心职责：从 MachineInstr/MachineOperand 到 MC 层对象的降级。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARM.h`, `ARMAsmPrinter.h`, `ARMBaseInstrInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `MCTargetDesc/ARMMCAsmInfo.h`.
  - CN: 后端本地头文件：`ARM.h`, `ARMAsmPrinter.h`, `ARMBaseInstrInfo.h`, `ARMMachineFunctionInfo.h`, `ARMSubtarget.h`, `MCTargetDesc/ARMAddressingModes.h`, `MCTargetDesc/ARMBaseInfo.h`, `MCTargetDesc/ARMMCAsmInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/APFloat.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/IR/Constants.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h` ... (+3 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/APFloat.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/IR/Constants.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h` ... (+3 more)。
- EN: Standard/system headers: `cassert`, `cstdint`.
  - CN: 标准库/系统头文件：`cassert`, `cstdint`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
