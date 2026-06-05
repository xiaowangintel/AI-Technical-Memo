# ARCMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCMCInstLower.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains code to lower ARC MachineInstrs to their corresponding MCInst records.
- 目的（中文）: 将 MachineInstr 降级为 MCInst，用于汇编输出或目标文件生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCMCInstLower.cpp - ARC MachineInstr to MCInst ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains code to lower ARC MachineInstrs to their
  11: /// corresponding MCInst records.
  12: ///
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "ARCMCInstLower.h"
  16: #include "llvm/CodeGen/AsmPrinter.h"
  17: #include "llvm/CodeGen/MachineFunction.h"
  18: #include "llvm/CodeGen/MachineInstr.h"
  19: #include "llvm/CodeGen/MachineOperand.h"
  20: #include "llvm/MC/MCContext.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCExpr.h"
  22: #include "llvm/MC/MCInst.h"
  23: 
  24: using namespace llvm;
  25: 
  26: ARCMCInstLower::ARCMCInstLower(MCContext *C, AsmPrinter &AsmPrinter)
  27:     : Ctx(C), Printer(AsmPrinter) {}
  28: 
  29: MCOperand ARCMCInstLower::LowerSymbolOperand(const MachineOperand &MO,
  30:                                              MachineOperandType MOTy,
  31:                                              unsigned Offset) const {
  32:   const MCSymbol *Symbol;
  33: 
  34:   switch (MOTy) {
  35:   case MachineOperand::MO_MachineBasicBlock:
  36:     Symbol = MO.getMBB()->getSymbol();
  37:     break;
  38:   case MachineOperand::MO_GlobalAddress:
  39:     Symbol = Printer.getSymbol(MO.getGlobal());
  40:     Offset += MO.getOffset();
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as Ctx, LowerSymbolOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 Ctx, LowerSymbolOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 41-60

```cpp
  41:     break;
  42:   case MachineOperand::MO_BlockAddress:
  43:     Symbol = Printer.GetBlockAddressSymbol(MO.getBlockAddress());
  44:     Offset += MO.getOffset();
  45:     break;
  46:   case MachineOperand::MO_ExternalSymbol:
  47:     Symbol = Printer.GetExternalSymbolSymbol(MO.getSymbolName());
  48:     Offset += MO.getOffset();
  49:     break;
  50:   case MachineOperand::MO_JumpTableIndex:
  51:     Symbol = Printer.GetJTISymbol(MO.getIndex());
  52:     break;
  53:   case MachineOperand::MO_ConstantPoolIndex:
  54:     Symbol = Printer.GetCPISymbol(MO.getIndex());
  55:     Offset += MO.getOffset();
  56:     break;
  57:   default:
  58:     llvm_unreachable("<unknown operand type>");
  59:   }
  60: 
```

- EN: This range continues the implementation of the backend component described by ARCMCInstLower.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61:   assert(Symbol && "Symbol creation failed.\n");
  62:   const MCSymbolRefExpr *MCSym = MCSymbolRefExpr::create(Symbol, *Ctx);
  63: 
  64:   if (!Offset)
  65:     return MCOperand::createExpr(MCSym);
  66: 
  67:   // Assume offset is never negative.
  68:   assert(Offset > 0);
  69: 
  70:   const MCConstantExpr *OffsetExpr = MCConstantExpr::create(Offset, *Ctx);
  71:   const MCBinaryExpr *Add = MCBinaryExpr::createAdd(MCSym, OffsetExpr, *Ctx);
  72:   return MCOperand::createExpr(Add);
  73: }
  74: 
  75: MCOperand ARCMCInstLower::LowerOperand(const MachineOperand &MO,
  76:                                        unsigned Offset) const {
  77:   MachineOperandType MOTy = MO.getType();
  78: 
  79:   switch (MOTy) {
  80:   default:
```

- EN: Function bodies or method definitions such as LowerOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: LowerOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:     llvm_unreachable("unknown operand type");
  82:   case MachineOperand::MO_Register:
  83:     // Ignore all implicit register operands.
  84:     if (MO.isImplicit())
  85:       break;
  86:     return MCOperand::createReg(MO.getReg());
  87:   case MachineOperand::MO_Immediate:
  88:     return MCOperand::createImm(MO.getImm() + Offset);
  89:   case MachineOperand::MO_MachineBasicBlock:
  90:   case MachineOperand::MO_GlobalAddress:
  91:   case MachineOperand::MO_ExternalSymbol:
  92:   case MachineOperand::MO_JumpTableIndex:
  93:   case MachineOperand::MO_ConstantPoolIndex:
  94:   case MachineOperand::MO_BlockAddress:
  95:     return LowerSymbolOperand(MO, MOTy, Offset);
  96:   case MachineOperand::MO_RegisterMask:
  97:     break;
  98:   }
  99: 
 100:   return {};
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-112

```cpp
 101: }
 102: 
 103: void ARCMCInstLower::Lower(const MachineInstr *MI, MCInst &OutMI) const {
 104:   OutMI.setOpcode(MI->getOpcode());
 105: 
 106:   for (const MachineOperand &MO : MI->operands()) {
 107:     MCOperand MCOp = LowerOperand(MO);
 108: 
 109:     if (MCOp.isValid())
 110:       OutMI.addOperand(MCOp);
 111:   }
 112: }
```

- EN: Function bodies or method definitions such as Lower contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: Lower 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- Assembly printing / 汇编打印

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCMCInstLower.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineOperand.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Local companions / 本地配套文件: `ARCMCInstLower.h`
