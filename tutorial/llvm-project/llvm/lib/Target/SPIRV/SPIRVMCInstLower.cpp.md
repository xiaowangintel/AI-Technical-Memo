# SPIRVMCInstLower.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVMCInstLower.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains code to lower SPIR-V MachineInstrs to their corresponding MCInst records.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //=- SPIRVMCInstLower.cpp - Convert SPIR-V MachineInstr to MCInst -*- C++ -*-=//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains code to lower SPIR-V MachineInstrs to their corresponding
10: // MCInst records.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVMCInstLower.h"
15: #include "SPIRVModuleAnalysis.h"
16: #include "SPIRVUtils.h"
17: #include "llvm/CodeGen/MachineInstr.h"
18: #include "llvm/IR/Constants.h"
19:
20: using namespace llvm;
21:
22: void SPIRVMCInstLower::lower(const MachineInstr *MI, MCInst &OutMI,
23:                              SPIRV::ModuleAnalysisInfo *MAI) const {
24:   OutMI.setOpcode(MI->getOpcode());
25:   // Propagate previously set flags
26:   if (MI->getAsmPrinterFlags() & SPIRV::ASM_PRINTER_WIDTH16)
27:     OutMI.setFlags(SPIRV::INST_PRINTER_WIDTH16);
28:   if (MI->getAsmPrinterFlags() & SPIRV::ASM_PRINTER_WIDTH64)
29:     OutMI.setFlags(SPIRV::INST_PRINTER_WIDTH64);
30:   const MachineFunction *MF = MI->getMF();
31:   for (unsigned i = 0, e = MI->getNumOperands(); i != e; ++i) {
32:     const MachineOperand &MO = MI->getOperand(i);
33:     MCOperand MCOp;
34:     switch (MO.getType()) {
35:     default:
36:       llvm_unreachable("unknown operand type");
37:     case MachineOperand::MO_GlobalAddress: {
38:       MCRegister Reg =
39:           MAI->getGlobalObjReg(dyn_cast<GlobalObject>(MO.getGlobal()));
40:       if (!Reg.isValid()) {
```
- EN: This range implements operational logic in helpers such as setOpcode, setFlags, getMF, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 setOpcode、setFlags、getMF、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-75
```cpp
41:         std::string DiagMsg;
42:         raw_string_ostream OS(DiagMsg);
43:         MI->print(OS);
44:         DiagMsg = "Unknown global object in:" + DiagMsg;
45:         report_fatal_error(DiagMsg.c_str());
46:       }
47:       MCOp = MCOperand::createReg(Reg);
48:       break;
49:     }
50:     case MachineOperand::MO_MachineBasicBlock:
51:       MCOp = MCOperand::createReg(MAI->getOrCreateMBBRegister(*MO.getMBB()));
52:       break;
53:     case MachineOperand::MO_Register: {
54:       MCRegister NewReg = MAI->getRegisterAlias(MF, MO.getReg());
55:       MCOp = MCOperand::createReg(NewReg.isValid() ? NewReg
56:                                                    : MO.getReg().asMCReg());
57:       break;
58:     }
59:     case MachineOperand::MO_Immediate:
60:       if (MI->getOpcode() == SPIRV::OpExtInst && i == 2) {
61:         MCRegister Reg = MAI->getExtInstSetReg(MO.getImm());
62:         MCOp = MCOperand::createReg(Reg);
63:       } else {
64:         MCOp = MCOperand::createImm(MO.getImm());
65:       }
66:       break;
67:     case MachineOperand::MO_FPImmediate:
68:       MCOp = MCOperand::createDFPImm(
69:           MO.getFPImm()->getValueAPF().convertToFloat());
70:       break;
71:     }
72:
73:     OutMI.addOperand(MCOp);
74:   }
75: }
```
- EN: This range implements operational logic in helpers such as OS, print, report_fatal_error, MCOperand::createReg, translating backend policy into executable code.
- CN: 这一段实现了 OS、print、report_fatal_error、MCOperand::createReg 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include setOpcode, setFlags, getMF, getOperand, llvm_unreachable, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 setOpcode, setFlags, getMF, getOperand, llvm_unreachable，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVMCInstLower.h`
  - `SPIRVModuleAnalysis.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineInstr.h`
  - `llvm/IR/Constants.h`
