# HexagonMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonMCInstLower.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file contains code to lower Hexagon MachineInstrs to their corresponding
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonMCInstLower.cpp - Convert Hexagon MachineInstr to an MCInst -===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file contains code to lower Hexagon MachineInstrs to their corresponding
    10: // MCInst records.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "HexagonAsmPrinter.h"
    15: #include "MCTargetDesc/HexagonMCExpr.h"
    16: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    17: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    18: #include "llvm/ADT/APFloat.h"
    19: #include "llvm/ADT/APInt.h"
    20: #include "llvm/CodeGen/MachineBasicBlock.h"
    21: #include "llvm/CodeGen/MachineInstr.h"
    22: #include "llvm/CodeGen/MachineOperand.h"
    23: #include "llvm/IR/Constants.h"
    24: #include "llvm/MC/MCContext.h"
    25: #include "llvm/MC/MCExpr.h"
```
- EN: It imports headers such as HexagonAsmPrinter.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (12 total), establishing the LLVM/Hexagon APIs used below. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstLower, HexagonAsmPrinter, HexagonMCExpr, HexagonMCInstrInfo, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里引入了 HexagonAsmPrinter.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (12 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstLower, HexagonAsmPrinter, HexagonMCExpr, HexagonMCInstrInfo, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: #include "llvm/MC/MCInst.h"
    27: #include "llvm/Support/ErrorHandling.h"
    28: #include "llvm/Support/raw_ostream.h"
    29: #include <cassert>
    30: 
    31: using namespace llvm;
    32: 
    33: namespace llvm {
    34: 
    35: void HexagonLowerToMC(const MCInstrInfo &MCII, const MachineInstr *MI,
    36:                       MCInst &MCB, HexagonAsmPrinter &AP);
    37: 
    38: } // end namespace llvm
    39: 
    40: static MCOperand GetSymbolRef(const MachineOperand &MO, const MCSymbol *Symbol,
    41:                               HexagonAsmPrinter &Printer, bool MustExtend) {
    42:   MCContext &MC = Printer.OutContext;
    43:   const MCExpr *ME;
    44: 
    45:   // Populate the relocation type based on Hexagon target flags
    46:   // set on an operand
    47:   HexagonMCExpr::VariantKind RelocationType;
    48:   switch (MO.getTargetFlags() & ~HexagonII::HMOTF_ConstExtended) {
    49:   default:
    50:     RelocationType = HexagonMCExpr::VK_None;
```
- EN: It imports headers such as llvm/MC/MCInst.h, llvm/Support/ErrorHandling.h, llvm/Support/raw_ostream.h, cassert, establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonLowerToMC, GetSymbolRef, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 llvm/MC/MCInst.h, llvm/Support/ErrorHandling.h, llvm/Support/raw_ostream.h, cassert 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonLowerToMC, GetSymbolRef 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-75 / 第 51-75 行

```cpp
    51:     break;
    52:   case HexagonII::MO_PCREL:
    53:     RelocationType = HexagonMCExpr::VK_PCREL;
    54:     break;
    55:   case HexagonII::MO_GOT:
    56:     RelocationType = HexagonMCExpr::VK_GOT;
    57:     break;
    58:   case HexagonII::MO_LO16:
    59:     RelocationType = HexagonMCExpr::VK_LO16;
    60:     break;
    61:   case HexagonII::MO_HI16:
    62:     RelocationType = HexagonMCExpr::VK_HI16;
    63:     break;
    64:   case HexagonII::MO_GPREL:
    65:     RelocationType = HexagonMCExpr::VK_GPREL;
    66:     break;
    67:   case HexagonII::MO_GDGOT:
    68:     RelocationType = HexagonMCExpr::VK_GD_GOT;
    69:     break;
    70:   case HexagonII::MO_GDPLT:
    71:     RelocationType = HexagonMCExpr::VK_GD_PLT;
    72:     break;
    73:   case HexagonII::MO_IE:
    74:     RelocationType = HexagonMCExpr::VK_IE;
    75:     break;
```
- EN: Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:   case HexagonII::MO_IEGOT:
    77:     RelocationType = HexagonMCExpr::VK_IE_GOT;
    78:     break;
    79:   case HexagonII::MO_TPREL:
    80:     RelocationType = HexagonMCExpr::VK_TPREL;
    81:     break;
    82:   }
    83: 
    84:   ME = MCSymbolRefExpr::create(Symbol, RelocationType, MC);
    85: 
    86:   if (!MO.isJTI() && MO.getOffset())
    87:     ME = MCBinaryExpr::createAdd(ME, MCConstantExpr::create(MO.getOffset(), MC),
    88:                                  MC);
    89: 
    90:   ME = HexagonMCExpr::create(ME, MC);
    91:   HexagonMCInstrInfo::setMustExtend(*ME, MustExtend);
    92:   return MCOperand::createExpr(ME);
    93: }
    94: 
    95: // Create an MCInst from a MachineInstr
    96: void llvm::HexagonLowerToMC(const MCInstrInfo &MCII, const MachineInstr *MI,
    97:                             MCInst &MCB, HexagonAsmPrinter &AP) {
    98:   if (MI->getOpcode() == Hexagon::ENDLOOP0) {
    99:     HexagonMCInstrInfo::setInnerLoop(MCB);
   100:     return;
```
- EN: It declares or implements routines such as MCSymbolRefExpr::create, HexagonMCExpr::create, HexagonMCInstrInfo::setMustExtend, MCOperand::createExpr, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonMCExpr, HexagonMCInstrInfo, HexagonLowerToMC, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MCSymbolRefExpr::create, HexagonMCExpr::create, HexagonMCInstrInfo::setMustExtend, MCOperand::createExpr, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCExpr, HexagonMCInstrInfo, HexagonLowerToMC, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   }
   102:   if (MI->getOpcode() == Hexagon::ENDLOOP1) {
   103:     HexagonMCInstrInfo::setOuterLoop(MCB);
   104:     return;
   105:   }
   106:   if (MI->getOpcode() == Hexagon::PATCHABLE_FUNCTION_ENTER) {
   107:     AP.EmitSled(*MI, HexagonAsmPrinter::SledKind::FUNCTION_ENTER);
   108:     return;
   109:   }
   110:   if (MI->getOpcode() == Hexagon::PATCHABLE_FUNCTION_EXIT) {
   111:     AP.EmitSled(*MI, HexagonAsmPrinter::SledKind::FUNCTION_EXIT);
   112:     return;
   113:   }
   114:   if (MI->getOpcode() == Hexagon::PATCHABLE_TAIL_CALL) {
   115:     AP.EmitSled(*MI, HexagonAsmPrinter::SledKind::TAIL_CALL);
   116:     return;
   117:   }
   118:   if (MI->getOpcode() == Hexagon::PATCHABLE_EVENT_CALL) {
   119:     AP.LowerPATCHABLE_EVENT_CALL(*MI, false);
   120:     return;
   121:   }
   122:   if (MI->getOpcode() == Hexagon::PATCHABLE_TYPED_EVENT_CALL) {
   123:     AP.LowerPATCHABLE_EVENT_CALL(*MI, true);
   124:     return;
   125:   }
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::setOuterLoop, EmitSled, LowerPATCHABLE_EVENT_CALL, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonAsmPrinter, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::setOuterLoop, EmitSled, LowerPATCHABLE_EVENT_CALL 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonAsmPrinter，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```cpp
   126: 
   127:   MCInst *MCI = AP.OutContext.createMCInst();
   128:   MCI->setOpcode(MI->getOpcode());
   129:   assert(MCI->getOpcode() == static_cast<unsigned>(MI->getOpcode()) &&
   130:          "MCI opcode should have been set on construction");
   131: 
   132:   for (const MachineOperand &MO : MI->operands()) {
   133:     MCOperand MCO;
   134:     bool MustExtend = MO.getTargetFlags() & HexagonII::HMOTF_ConstExtended;
   135: 
   136:     switch (MO.getType()) {
   137:     default:
   138:       MI->print(errs());
   139:       llvm_unreachable("unknown operand type");
   140:     case MachineOperand::MO_RegisterMask:
   141:       continue;
   142:     case MachineOperand::MO_Register:
   143:       // Ignore all implicit register operands.
   144:       if (MO.isImplicit())
   145:         continue;
   146:       MCO = MCOperand::createReg(MO.getReg());
   147:       break;
   148:     case MachineOperand::MO_FPImmediate: {
   149:       APFloat Val = MO.getFPImm()->getValueAPF();
   150:       // FP immediates are used only when setting GPRs, so they may be dealt
```
- EN: It declares or implements routines such as createMCInst, setOpcode, assert, print, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 createMCInst, setOpcode, assert, print, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 151-175 / 第 151-175 行

```cpp
   151:       // with like regular immediates from this point on.
   152:       auto Expr = HexagonMCExpr::create(
   153:           MCConstantExpr::create(*Val.bitcastToAPInt().getRawData(),
   154:                                  AP.OutContext),
   155:           AP.OutContext);
   156:       HexagonMCInstrInfo::setMustExtend(*Expr, MustExtend);
   157:       MCO = MCOperand::createExpr(Expr);
   158:       break;
   159:     }
   160:     case MachineOperand::MO_Immediate: {
   161:       auto Expr = HexagonMCExpr::create(
   162:           MCConstantExpr::create(MO.getImm(), AP.OutContext), AP.OutContext);
   163:       HexagonMCInstrInfo::setMustExtend(*Expr, MustExtend);
   164:       MCO = MCOperand::createExpr(Expr);
   165:       break;
   166:     }
   167:     case MachineOperand::MO_MachineBasicBlock: {
   168:       MCExpr const *Expr = MCSymbolRefExpr::create(MO.getMBB()->getSymbol(),
   169:                                                    AP.OutContext);
   170:       Expr = HexagonMCExpr::create(Expr, AP.OutContext);
   171:       HexagonMCInstrInfo::setMustExtend(*Expr, MustExtend);
   172:       MCO = MCOperand::createExpr(Expr);
   173:       break;
   174:     }
   175:     case MachineOperand::MO_GlobalAddress:
```
- EN: It declares or implements routines such as HexagonMCExpr::create, HexagonMCInstrInfo::setMustExtend, MCOperand::createExpr, MCSymbolRefExpr::create, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCExpr::create, HexagonMCInstrInfo::setMustExtend, MCOperand::createExpr, MCSymbolRefExpr::create 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 176-199 / 第 176-199 行

```cpp
   176:       MCO = GetSymbolRef(MO, AP.getSymbol(MO.getGlobal()), AP, MustExtend);
   177:       break;
   178:     case MachineOperand::MO_ExternalSymbol:
   179:       MCO = GetSymbolRef(MO, AP.GetExternalSymbolSymbol(MO.getSymbolName()),
   180:                          AP, MustExtend);
   181:       break;
   182:     case MachineOperand::MO_JumpTableIndex:
   183:       MCO = GetSymbolRef(MO, AP.GetJTISymbol(MO.getIndex()), AP, MustExtend);
   184:       break;
   185:     case MachineOperand::MO_ConstantPoolIndex:
   186:       MCO = GetSymbolRef(MO, AP.GetCPISymbol(MO.getIndex()), AP, MustExtend);
   187:       break;
   188:     case MachineOperand::MO_BlockAddress:
   189:       MCO = GetSymbolRef(MO, AP.GetBlockAddressSymbol(MO.getBlockAddress()), AP,
   190:                          MustExtend);
   191:       break;
   192:     }
   193: 
   194:     MCI->addOperand(MCO);
   195:   }
   196:   AP.HexagonProcessInstruction(*MCI, *MI);
   197:   HexagonMCInstrInfo::extendIfNeeded(AP.OutContext, MCII, MCB, *MCI);
   198:   MCB.addOperand(MCOperand::createInst(MCI));
   199: }
```
- EN: It declares or implements routines such as GetSymbolRef, addOperand, HexagonProcessInstruction, HexagonMCInstrInfo::extendIfNeeded, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonProcessInstruction, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 GetSymbolRef, addOperand, HexagonProcessInstruction, HexagonMCInstrInfo::extendIfNeeded 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonProcessInstruction, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonAsmPrinter.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/APFloat.h, llvm/ADT/APInt.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/IR/Constants.h, ... (16 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCInstLower, HexagonAsmPrinter, HexagonMCExpr, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonLowerToMC, HexagonII, HexagonProcessInstruction`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
