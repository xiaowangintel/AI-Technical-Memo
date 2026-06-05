# XCoreMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreMCInstLower.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Converts MachineInstr into MCInst form before final emission.
  - **CN**: 在最终发射前把 MachineInstr 转换为 MCInst 形式。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XCoreMCInstLower.cpp - Convert XCore MachineInstr to MCInst -------===//
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
///
/// \file
/// This file contains code to lower XCore MachineInstrs to their
/// corresponding MCInst records.
///
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 13-22
```cpp
//===----------------------------------------------------------------------===//
#include "XCoreMCInstLower.h"
#include "llvm/CodeGen/AsmPrinter.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/IR/Mangler.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreMCInstLower.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreMCInstLower.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`。

### Lines 23-28
```cpp

using namespace llvm;

XCoreMCInstLower::XCoreMCInstLower(class AsmPrinter &asmprinter)
    : Printer(asmprinter) {}

```
- **EN**: Introduces declarations for `llvm`, `AsmPrinter`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm`, `AsmPrinter` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 29-35
```cpp
void XCoreMCInstLower::Initialize(MCContext *C) { Ctx = C; }

MCOperand XCoreMCInstLower::LowerSymbolOperand(const MachineOperand &MO,
                                               MachineOperandType MOTy,
                                               unsigned Offset) const {
  const MCSymbol *Symbol;

```
- **EN**: Implements logic around `Initialize`, `LowerSymbolOperand`; this block works at the MC layer.
- **CN**: 围绕 `Initialize`, `LowerSymbolOperand` 实现具体逻辑；这一段工作在 MC 层。

### Lines 36-45
```cpp
  switch (MOTy) {
    case MachineOperand::MO_MachineBasicBlock:
      Symbol = MO.getMBB()->getSymbol();
      break;
    case MachineOperand::MO_GlobalAddress:
      Symbol = Printer.getSymbol(MO.getGlobal());
      Offset += MO.getOffset();
      break;
    case MachineOperand::MO_BlockAddress:
      Symbol = Printer.GetBlockAddressSymbol(MO.getBlockAddress());
```
- **EN**: Implements logic around `getMBB`, `getSymbol`, `getOffset`, `GetBlockAddressSymbol`; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getMBB`, `getSymbol`, `getOffset`, `GetBlockAddressSymbol` 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 46-55
```cpp
      Offset += MO.getOffset();
      break;
    case MachineOperand::MO_ExternalSymbol:
      Symbol = Printer.GetExternalSymbolSymbol(MO.getSymbolName());
      Offset += MO.getOffset();
      break;
    case MachineOperand::MO_JumpTableIndex:
      Symbol = Printer.GetJTISymbol(MO.getIndex());
      break;
    case MachineOperand::MO_ConstantPoolIndex:
```
- **EN**: Implements logic around `getOffset`, `GetExternalSymbolSymbol`, `GetJTISymbol`.
- **CN**: 围绕 `getOffset`, `GetExternalSymbolSymbol`, `GetJTISymbol` 实现具体逻辑。

### Lines 56-62
```cpp
      Symbol = Printer.GetCPISymbol(MO.getIndex());
      Offset += MO.getOffset();
      break;
    default:
      llvm_unreachable("<unknown operand type>");
  }

```
- **EN**: Implements logic around `GetCPISymbol`, `getOffset`, `llvm_unreachable`.
- **CN**: 围绕 `GetCPISymbol`, `getOffset`, `llvm_unreachable` 实现具体逻辑。

### Lines 63-69
```cpp
  const MCSymbolRefExpr *MCSym = MCSymbolRefExpr::create(Symbol, *Ctx);
  if (!Offset)
    return MCOperand::createExpr(MCSym);

  // Assume offset is never negative.
  assert(Offset > 0);

```
- **EN**: Implements logic around `create`, `createExpr`, `assert`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `create`, `createExpr`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 70-74
```cpp
  const MCConstantExpr *OffsetExpr =  MCConstantExpr::create(Offset, *Ctx);
  const MCBinaryExpr *Add = MCBinaryExpr::createAdd(MCSym, OffsetExpr, *Ctx);
  return MCOperand::createExpr(Add);
}

```
- **EN**: Implements logic around `create`, `createAdd`, `createExpr`; this block returns target-specific results.
- **CN**: 围绕 `create`, `createAdd`, `createExpr` 实现具体逻辑；这一段返回目标相关结果。

### Lines 75-84
```cpp
MCOperand XCoreMCInstLower::LowerOperand(const MachineOperand &MO,
                                         unsigned offset) const {
  MachineOperandType MOTy = MO.getType();

  switch (MOTy) {
    default: llvm_unreachable("unknown operand type");
    case MachineOperand::MO_Register:
      // Ignore all implicit register operands.
      if (MO.isImplicit()) break;
      return MCOperand::createReg(MO.getReg());
```
- **EN**: Implements logic around `LowerOperand`, `getType`, `llvm_unreachable`, `createReg`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `LowerOperand`, `getType`, `llvm_unreachable`, `createReg` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 85-94
```cpp
    case MachineOperand::MO_Immediate:
      return MCOperand::createImm(MO.getImm() + offset);
    case MachineOperand::MO_MachineBasicBlock:
    case MachineOperand::MO_GlobalAddress:
    case MachineOperand::MO_ExternalSymbol:
    case MachineOperand::MO_JumpTableIndex:
    case MachineOperand::MO_ConstantPoolIndex:
    case MachineOperand::MO_BlockAddress:
      return LowerSymbolOperand(MO, MOTy, offset);
    case MachineOperand::MO_RegisterMask:
```
- **EN**: Implements logic around `createImm`, `LowerSymbolOperand`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createImm`, `LowerSymbolOperand` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 95-100
```cpp
      break;
  }

  return MCOperand();
}

```
- **EN**: Implements logic around `MCOperand`; this block returns target-specific results.
- **CN**: 围绕 `MCOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 101-106
```cpp
void XCoreMCInstLower::Lower(const MachineInstr *MI, MCInst &OutMI) const {
  OutMI.setOpcode(MI->getOpcode());

  for (const MachineOperand &MO : MI->operands()) {
    MCOperand MCOp = LowerOperand(MO);

```
- **EN**: Implements logic around `Lower`, `setOpcode`, `LowerOperand`; this block works at the MachineInstr/MachineFunction layer; works at the MC layer.
- **CN**: 围绕 `Lower`, `setOpcode`, `LowerOperand` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层，工作在 MC 层。

### Lines 107-110
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

- **Direct includes / 直接包含**: `XCoreMCInstLower.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/IR/Mangler.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR
