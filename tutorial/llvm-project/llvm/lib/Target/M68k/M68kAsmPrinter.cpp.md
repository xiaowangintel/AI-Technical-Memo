# M68kAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kAsmPrinter.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file emits textual assembly from LLVM machine instructions for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责将 LLVM 机器指令输出为文本汇编。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kAsmPrinter.cpp - M68k LLVM Assembly Printer ---------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains a printer that converts from our internal representation
  11: /// of machine-dependent LLVM code to GAS-format M68k assembly language.
  12: ///
  13: //===----------------------------------------------------------------------===//
  14: 
  15: // TODO Conform to Motorola ASM syntax
  16: 
  17: #include "M68kAsmPrinter.h"
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kAsmPrinter.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kAsmPrinter.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "M68k.h"
  20: #include "M68kMachineFunction.h"
  21: #include "MCTargetDesc/M68kInstPrinter.h"
  22: #include "TargetInfo/M68kTargetInfo.h"
  23: 
  24: #include "llvm/MC/TargetRegistry.h"
  25: 
  26: using namespace llvm;
  27: 
  28: #define DEBUG_TYPE "m68k-asm-printer"
  29: 
  30: bool M68kAsmPrinter::runOnMachineFunction(MachineFunction &MF) {
  31:   MMFI = MF.getInfo<M68kMachineFunctionInfo>();
  32:   MCInstLowering = std::make_unique<M68kMCInstLower>(MF, *this);
  33:   AsmPrinter::runOnMachineFunction(MF);
  34:   return true;
  35: }
  36: 
```
- **EN**: It imports dependencies such as `M68k.h`, `M68kMachineFunction.h`, `M68kInstPrinter.h`, `M68kTargetInfo.h`, `TargetRegistry.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `M68kAsmPrinter::runOnMachineFunction`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `M68k.h`, `M68kMachineFunction.h`, `M68kInstPrinter.h`, `M68kTargetInfo.h`, `TargetRegistry.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `M68kAsmPrinter::runOnMachineFunction` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: void M68kAsmPrinter::printOperand(const MachineInstr *MI, int OpNum,
  38:                                   raw_ostream &OS) {
  39:   const MachineOperand &MO = MI->getOperand(OpNum);
  40:   switch (MO.getType()) {
  41:   case MachineOperand::MO_Register:
  42:     OS << "%" << M68kInstPrinter::getRegisterName(MO.getReg());
  43:     break;
  44:   case MachineOperand::MO_Immediate:
  45:     OS << '#' << MO.getImm();
  46:     break;
  47:   case MachineOperand::MO_MachineBasicBlock:
  48:     MO.getMBB()->getSymbol()->print(OS, MAI);
  49:     break;
  50:   case MachineOperand::MO_GlobalAddress:
  51:     PrintSymbolOperand(MO, OS);
  52:     break;
  53:   case MachineOperand::MO_BlockAddress:
  54:     GetBlockAddressSymbol(MO.getBlockAddress())->print(OS, MAI);
```
- **EN**: The range implements or declares functions including `M68kAsmPrinter::printOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `M68kAsmPrinter::printOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 55-72 / 第 55-72 行
```cpp
  55:     break;
  56:   case MachineOperand::MO_ConstantPoolIndex: {
  57:     const DataLayout &DL = getDataLayout();
  58:     OS << DL.getInternalSymbolPrefix() << "CPI" << getFunctionNumber() << '_'
  59:        << MO.getIndex();
  60:     break;
  61:   }
  62:   default:
  63:     llvm_unreachable("not implemented");
  64:   }
  65: }
  66: 
  67: bool M68kAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
  68:                                      const char *ExtraCode, raw_ostream &OS) {
  69:   // Print the operand if there is no operand modifier.
  70:   if (!ExtraCode || !ExtraCode[0]) {
  71:     printOperand(MI, OpNo, OS);
  72:     return false;
```
- **EN**: The range implements or declares functions including `M68kAsmPrinter::PrintAsmOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kAsmPrinter::PrintAsmOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   }
  74: 
  75:   // Fallback to the default implementation.
  76:   return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, OS);
  77: }
  78: 
  79: void M68kAsmPrinter::printDisp(const MachineInstr *MI, unsigned opNum,
  80:                                raw_ostream &O) {
  81:   // Print immediate displacement without the '#' predix
  82:   const MachineOperand &Op = MI->getOperand(opNum);
  83:   if (Op.isImm()) {
  84:     O << Op.getImm();
  85:     return;
  86:   }
  87:   // Displacement is relocatable, so we're pretty permissive about what
  88:   // can be put here.
  89:   printOperand(MI, opNum, O);
  90: }
```
- **EN**: The range implements or declares functions including `M68kAsmPrinter::printDisp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmPrinter::printDisp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92: void M68kAsmPrinter::printAbsMem(const MachineInstr *MI, unsigned OpNum,
  93:                                  raw_ostream &O) {
  94:   const MachineOperand &MO = MI->getOperand(OpNum);
  95:   if (MO.isImm())
  96:     O << format("$%0" PRIx64, (uint64_t)MO.getImm());
  97:   else
  98:     PrintAsmMemoryOperand(MI, OpNum, nullptr, O);
  99: }
 100: 
 101: bool M68kAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
 102:                                            unsigned OpNo, const char *ExtraCode,
 103:                                            raw_ostream &OS) {
 104:   const MachineOperand &MO = MI->getOperand(OpNo);
 105:   switch (MO.getType()) {
 106:   case MachineOperand::MO_Immediate:
 107:     // Immediate value that goes here is the addressing mode kind we set
 108:     // in M68kDAGToDAGISel::SelectInlineAsmMemoryOperand.
```
- **EN**: The range implements or declares functions including `M68kAsmPrinter::printAbsMem`, `M68kAsmPrinter::PrintAsmMemoryOperand`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmPrinter::printAbsMem`, `M68kAsmPrinter::PrintAsmMemoryOperand` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     using namespace M68k;
 110:     // Skip the addressing mode kind operand.
 111:     ++OpNo;
 112:     // Decode MemAddrModeKind.
 113:     switch (static_cast<MemAddrModeKind>(MO.getImm())) {
 114:     case MemAddrModeKind::j:
 115:       printARIMem(MI, OpNo, OS);
 116:       break;
 117:     case MemAddrModeKind::o:
 118:       printARIPIMem(MI, OpNo, OS);
 119:       break;
 120:     case MemAddrModeKind::e:
 121:       printARIPDMem(MI, OpNo, OS);
 122:       break;
 123:     case MemAddrModeKind::p:
 124:       printARIDMem(MI, OpNo, OS);
 125:       break;
 126:     case MemAddrModeKind::f:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     case MemAddrModeKind::F:
 128:       printARIIMem(MI, OpNo, OS);
 129:       break;
 130:     case MemAddrModeKind::k:
 131:       printPCIMem(MI, 0, OpNo, OS);
 132:       break;
 133:     case MemAddrModeKind::q:
 134:       printPCDMem(MI, 0, OpNo, OS);
 135:       break;
 136:     case MemAddrModeKind::b:
 137:       printAbsMem(MI, OpNo, OS);
 138:       break;
 139:     default:
 140:       llvm_unreachable("Unrecognized memory addressing mode");
 141:     }
 142:     return false;
 143:   case MachineOperand::MO_GlobalAddress:
 144:     PrintSymbolOperand(MO, OS);
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     return false;
 146:   case MachineOperand::MO_BlockAddress:
 147:     GetBlockAddressSymbol(MO.getBlockAddress())->print(OS, MAI);
 148:     return false;
 149:   case MachineOperand::MO_Register:
 150:     // This is a special case where it is treated as a memory reference, with
 151:     // the register holding the address value. Thus, we print it as ARI here.
 152:     if (M68kII::isAddressRegister(MO.getReg())) {
 153:       printARIMem(MI, OpNo, OS);
 154:       return false;
 155:     }
 156:     break;
 157:   default:
 158:     break;
 159:   }
 160:   return AsmPrinter::PrintAsmMemoryOperand(MI, OpNo, ExtraCode, OS);
 161: }
 162: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163: void M68kAsmPrinter::emitInstruction(const MachineInstr *MI) {
 164:   M68k_MC::verifyInstructionPredicates(MI->getOpcode(),
 165:                                        getSubtargetInfo().getFeatureBits());
 166: 
 167:   switch (MI->getOpcode()) {
 168:   default: {
 169:     if (MI->isPseudo()) {
 170:       LLVM_DEBUG(dbgs() << "Pseudo opcode(" << MI->getOpcode()
 171:                         << ") found in EmitInstruction()\n");
 172:       llvm_unreachable("Cannot proceed");
 173:     }
 174:     break;
 175:   }
 176:   case M68k::TAILJMPj:
 177:   case M68k::TAILJMPq:
 178:     // Lower these as normal, but add some comments.
 179:     OutStreamer->AddComment("TAILCALL");
 180:     break;
```
- **EN**: The range implements or declares functions including `M68kAsmPrinter::emitInstruction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmPrinter::emitInstruction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   }
 182: 
 183:   MCInst TmpInst0;
 184:   MCInstLowering->Lower(MI, TmpInst0);
 185:   OutStreamer->emitInstruction(TmpInst0, getSubtargetInfo());
 186: }
 187: 
 188: void M68kAsmPrinter::emitFunctionBodyStart() {}
 189: 
 190: void M68kAsmPrinter::emitFunctionBodyEnd() {}
 191: 
 192: void M68kAsmPrinter::emitEndOfAsmFile(Module &M) {}
 193: 
 194: char M68kAsmPrinter::ID = 0;
 195: 
 196: INITIALIZE_PASS(M68kAsmPrinter, "m68k-asm-printer", "M68k Assembly Printer",
 197:                 false, false)
 198: 
```
- **EN**: The range implements or declares functions including `M68kAsmPrinter::emitFunctionBodyStart`.
- **CN**: 这一段实现或声明了 `M68kAsmPrinter::emitFunctionBodyStart` 等函数。

### Lines 199-201 / 第 199-201 行
```cpp
 199: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeM68kAsmPrinter() {
 200:   RegisterAsmPrinter<M68kAsmPrinter> X(getTheM68kTarget());
 201: }
```
- **EN**: This span continues the file's main responsibility: this file emits textual assembly from LLVM machine instructions for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `M68kAsmPrinter.h`
- `M68k.h`
- `M68kMachineFunction.h`
- `MCTargetDesc/M68kInstPrinter.h`
- `TargetInfo/M68kTargetInfo.h`
- `llvm/MC/TargetRegistry.h`
