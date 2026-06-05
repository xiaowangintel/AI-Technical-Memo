# VEMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VEMCInstLower.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Converts MachineInstr into MCInst form before final emission.
  - **CN**: 在最终发射前把 MachineInstr 转换为 MCInst 形式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VEMCInstLower.cpp - Convert VE MachineInstr to MCInst -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-12
```cpp
//
// This file contains code to lower VE MachineInstrs to their corresponding
// MCInst records.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp

#include "MCTargetDesc/VEMCAsmInfo.h"
#include "VE.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/Mangler.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEMCAsmInfo.h`, `VE.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEMCAsmInfo.h`, `VE.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`。

### Lines 23-27
```cpp
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"

using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`。

### Lines 28-32
```cpp
static MCOperand LowerSymbolOperand(const MachineInstr *MI,
                                    const MachineOperand &MO,
                                    const MCSymbol *Symbol, AsmPrinter &AP) {
  auto Kind = (VE::Specifier)MO.getTargetFlags();

```
- **EN**: Implements logic around `LowerSymbolOperand`, `getTargetFlags`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerSymbolOperand`, `getTargetFlags` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 33-42
```cpp
  const MCExpr *Expr = MCSymbolRefExpr::create(Symbol, AP.OutContext);
  // Add offset iff MO is not jump table info or machine basic block.
  if (!MO.isJTI() && !MO.isMBB() && MO.getOffset())
    Expr = MCBinaryExpr::createAdd(
        Expr, MCConstantExpr::create(MO.getOffset(), AP.OutContext),
        AP.OutContext);
  Expr = MCSpecifierExpr::create(Expr, Kind, AP.OutContext);
  return MCOperand::createExpr(Expr);
}

```
- **EN**: Implements logic around `create`, `createAdd`, `createExpr`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `create`, `createAdd`, `createExpr` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 43-48
```cpp
static MCOperand LowerOperand(const MachineInstr *MI, const MachineOperand &MO,
                              AsmPrinter &AP) {
  switch (MO.getType()) {
  default:
    report_fatal_error("unsupported operand type");

```
- **EN**: Implements logic around `LowerOperand`, `report_fatal_error`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerOperand`, `report_fatal_error` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 49-53
```cpp
  case MachineOperand::MO_Register:
    if (MO.isImplicit())
      break;
    return MCOperand::createReg(MO.getReg());

```
- **EN**: Implements logic around `createReg`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `createReg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 54-63
```cpp
  case MachineOperand::MO_BlockAddress:
    return LowerSymbolOperand(
        MI, MO, AP.GetBlockAddressSymbol(MO.getBlockAddress()), AP);
  case MachineOperand::MO_ConstantPoolIndex:
    return LowerSymbolOperand(MI, MO, AP.GetCPISymbol(MO.getIndex()), AP);
  case MachineOperand::MO_ExternalSymbol:
    return LowerSymbolOperand(
        MI, MO, AP.GetExternalSymbolSymbol(MO.getSymbolName()), AP);
  case MachineOperand::MO_GlobalAddress:
    return LowerSymbolOperand(MI, MO, AP.getSymbol(MO.getGlobal()), AP);
```
- **EN**: Implements logic around `LowerSymbolOperand`, `GetBlockAddressSymbol`, `GetExternalSymbolSymbol`; this block returns target-specific results.
- **CN**: 围绕 `LowerSymbolOperand`, `GetBlockAddressSymbol`, `GetExternalSymbolSymbol` 实现具体逻辑；这一段返回目标相关结果。

### Lines 64-70
```cpp
  case MachineOperand::MO_Immediate:
    return MCOperand::createImm(MO.getImm());
  case MachineOperand::MO_JumpTableIndex:
    return LowerSymbolOperand(MI, MO, AP.GetJTISymbol(MO.getIndex()), AP);
  case MachineOperand::MO_MachineBasicBlock:
    return LowerSymbolOperand(MI, MO, MO.getMBB()->getSymbol(), AP);

```
- **EN**: Implements logic around `createImm`, `LowerSymbolOperand`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createImm`, `LowerSymbolOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 71-76
```cpp
  case MachineOperand::MO_RegisterMask:
    break;
  }
  return MCOperand();
}

```
- **EN**: Implements logic around `MCOperand`; this block returns target-specific results.
- **CN**: 围绕 `MCOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 77-83
```cpp
void llvm::LowerVEMachineInstrToMCInst(const MachineInstr *MI, MCInst &OutMI,
                                       AsmPrinter &AP) {
  OutMI.setOpcode(MI->getOpcode());

  for (const MachineOperand &MO : MI->operands()) {
    MCOperand MCOp = LowerOperand(MI, MO, AP);

```
- **EN**: Implements logic around `LowerVEMachineInstrToMCInst`, `setOpcode`, `LowerOperand`; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `LowerVEMachineInstrToMCInst`, `setOpcode`, `LowerOperand` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 84-87
```cpp
    if (MCOp.isValid())
      OutMI.addOperand(MCOp);
  }
}
```
- **EN**: Implements logic around `addOperand`; this block applies conditional target rules.
- **CN**: 围绕 `addOperand` 实现具体逻辑；这一段应用条件化的目标规则。

## Key Concepts / 关键概念

- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/VEMCAsmInfo.h`, `VE.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCAsmInfo.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR
