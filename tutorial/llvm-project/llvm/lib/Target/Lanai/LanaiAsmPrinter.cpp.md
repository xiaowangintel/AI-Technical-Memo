# LanaiAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiAsmPrinter.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to the Lanai assembly language.
- 目的（中文）: 实现汇编打印器，把 LLVM 机器指令降级为文本汇编或 MC 层输出。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiAsmPrinter.cpp - Lanai LLVM assembly writer ------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a printer that converts from our internal representation
  10: // of machine-dependent LLVM code to the Lanai assembly language.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LanaiAluCode.h"
  15: #include "LanaiCondCode.h"
  16: #include "LanaiMCInstLower.h"
  17: #include "LanaiTargetMachine.h"
  18: #include "MCTargetDesc/LanaiInstPrinter.h"
  19: #include "TargetInfo/LanaiTargetInfo.h"
  20: #include "llvm/CodeGen/AsmPrinter.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineConstantPool.h"
  22: #include "llvm/CodeGen/MachineInstr.h"
  23: #include "llvm/IR/Mangler.h"
  24: #include "llvm/MC/MCAsmInfo.h"
  25: #include "llvm/MC/MCInst.h"
  26: #include "llvm/MC/MCInstBuilder.h"
  27: #include "llvm/MC/MCStreamer.h"
  28: #include "llvm/MC/MCSymbol.h"
  29: #include "llvm/MC/TargetRegistry.h"
  30: #include "llvm/Support/Compiler.h"
  31: #include "llvm/Support/raw_ostream.h"
  32: 
  33: #define DEBUG_TYPE "asm-printer"
  34: 
  35: using namespace llvm;
  36: 
  37: namespace {
  38: class LanaiAsmPrinter : public AsmPrinter {
  39: public:
  40:   explicit LanaiAsmPrinter(TargetMachine &TM,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as LanaiAsmPrinter, which organize the target-specific behavior exposed by the file. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 LanaiAsmPrinter 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 41-60

```cpp
  41:                            std::unique_ptr<MCStreamer> Streamer)
  42:       : AsmPrinter(TM, std::move(Streamer), ID) {}
  43: 
  44:   StringRef getPassName() const override { return "Lanai Assembly Printer"; }
  45: 
  46:   void printOperand(const MachineInstr *MI, int OpNum, raw_ostream &O);
  47:   bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
  48:                        const char *ExtraCode, raw_ostream &O) override;
  49:   void emitInstruction(const MachineInstr *MI) override;
  50:   bool isBlockOnlyReachableByFallthrough(
  51:       const MachineBasicBlock *MBB) const override;
  52: 
  53: private:
  54:   void customEmitInstruction(const MachineInstr *MI);
  55:   void emitCallInstruction(const MachineInstr *MI);
  56: 
  57: public:
  58:   static char ID;
  59: };
  60: } // end of anonymous namespace
```

- EN: Function bodies or method definitions such as AsmPrinter contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: AsmPrinter 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 61-80

```cpp
  61: 
  62: void LanaiAsmPrinter::printOperand(const MachineInstr *MI, int OpNum,
  63:                                    raw_ostream &O) {
  64:   const MachineOperand &MO = MI->getOperand(OpNum);
  65: 
  66:   switch (MO.getType()) {
  67:   case MachineOperand::MO_Register:
  68:     O << LanaiInstPrinter::getRegisterName(MO.getReg());
  69:     break;
  70: 
  71:   case MachineOperand::MO_Immediate:
  72:     O << MO.getImm();
  73:     break;
  74: 
  75:   case MachineOperand::MO_MachineBasicBlock:
  76:     O << *MO.getMBB()->getSymbol();
  77:     break;
  78: 
  79:   case MachineOperand::MO_GlobalAddress:
  80:     O << *getSymbol(MO.getGlobal());
```

- EN: Function bodies or method definitions such as printOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: printOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 81-100

```cpp
  81:     break;
  82: 
  83:   case MachineOperand::MO_BlockAddress: {
  84:     MCSymbol *BA = GetBlockAddressSymbol(MO.getBlockAddress());
  85:     O << BA->getName();
  86:     break;
  87:   }
  88: 
  89:   case MachineOperand::MO_ExternalSymbol:
  90:     O << *GetExternalSymbolSymbol(MO.getSymbolName());
  91:     break;
  92: 
  93:   case MachineOperand::MO_JumpTableIndex:
  94:     O << MAI.getInternalSymbolPrefix() << "JTI" << getFunctionNumber() << '_'
  95:       << MO.getIndex();
  96:     break;
  97: 
  98:   case MachineOperand::MO_ConstantPoolIndex:
  99:     O << MAI.getInternalSymbolPrefix() << "CPI" << getFunctionNumber() << '_'
 100:       << MO.getIndex();
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 101-120

```cpp
 101:     return;
 102: 
 103:   default:
 104:     llvm_unreachable("<unknown operand type>");
 105:   }
 106: }
 107: 
 108: // PrintAsmOperand - Print out an operand for an inline asm expression.
 109: bool LanaiAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
 110:                                       const char *ExtraCode, raw_ostream &O) {
 111:   // Does this asm operand have a single letter operand modifier?
 112:   if (ExtraCode && ExtraCode[0]) {
 113:     if (ExtraCode[1])
 114:       return true; // Unknown modifier.
 115: 
 116:     switch (ExtraCode[0]) {
 117:     // The highest-numbered register of a pair.
 118:     case 'H': {
 119:       if (OpNo == 0)
 120:         return true;
```

- EN: Function bodies or method definitions such as PrintAsmOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: PrintAsmOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 121-140

```cpp
 121:       const MachineOperand &FlagsOP = MI->getOperand(OpNo - 1);
 122:       if (!FlagsOP.isImm())
 123:         return true;
 124:       const InlineAsm::Flag Flags(FlagsOP.getImm());
 125:       const unsigned NumVals = Flags.getNumOperandRegisters();
 126:       if (NumVals != 2)
 127:         return true;
 128:       unsigned RegOp = OpNo + 1;
 129:       if (RegOp >= MI->getNumOperands())
 130:         return true;
 131:       const MachineOperand &MO = MI->getOperand(RegOp);
 132:       if (!MO.isReg())
 133:         return true;
 134:       Register Reg = MO.getReg();
 135:       O << LanaiInstPrinter::getRegisterName(Reg);
 136:       return false;
 137:     }
 138:     default:
 139:       return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
 140:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 141-160

```cpp
 141:   }
 142:   printOperand(MI, OpNo, O);
 143:   return false;
 144: }
 145: 
 146: //===----------------------------------------------------------------------===//
 147: void LanaiAsmPrinter::emitCallInstruction(const MachineInstr *MI) {
 148:   assert((MI->getOpcode() == Lanai::CALL || MI->getOpcode() == Lanai::CALLR) &&
 149:          "Unsupported call function");
 150: 
 151:   LanaiMCInstLower MCInstLowering(OutContext, *this);
 152:   MCSubtargetInfo STI = getSubtargetInfo();
 153:   // Insert save rca instruction immediately before the call.
 154:   // TODO: We should generate a pc-relative mov instruction here instead
 155:   // of pc + 16 (should be mov .+16 %rca).
 156:   OutStreamer->emitInstruction(MCInstBuilder(Lanai::ADD_I_LO)
 157:                                    .addReg(Lanai::RCA)
 158:                                    .addReg(Lanai::PC)
 159:                                    .addImm(16),
 160:                                STI);
```

- EN: Function bodies or method definitions such as emitCallInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: emitCallInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 161-180

```cpp
 161: 
 162:   // Push rca onto the stack.
 163:   //   st %rca, [--%sp]
 164:   OutStreamer->emitInstruction(MCInstBuilder(Lanai::SW_RI)
 165:                                    .addReg(Lanai::RCA)
 166:                                    .addReg(Lanai::SP)
 167:                                    .addImm(-4)
 168:                                    .addImm(LPAC::makePreOp(LPAC::ADD)),
 169:                                STI);
 170: 
 171:   // Lower the call instruction.
 172:   if (MI->getOpcode() == Lanai::CALL) {
 173:     MCInst TmpInst;
 174:     MCInstLowering.Lower(MI, TmpInst);
 175:     TmpInst.setOpcode(Lanai::BT);
 176:     OutStreamer->emitInstruction(TmpInst, STI);
 177:   } else {
 178:     OutStreamer->emitInstruction(MCInstBuilder(Lanai::ADD_R)
 179:                                      .addReg(Lanai::PC)
 180:                                      .addReg(MI->getOperand(0).getReg())
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 181-200

```cpp
 181:                                      .addReg(Lanai::R0)
 182:                                      .addImm(LPCC::ICC_T),
 183:                                  STI);
 184:   }
 185: }
 186: 
 187: void LanaiAsmPrinter::customEmitInstruction(const MachineInstr *MI) {
 188:   LanaiMCInstLower MCInstLowering(OutContext, *this);
 189:   MCSubtargetInfo STI = getSubtargetInfo();
 190:   MCInst TmpInst;
 191:   MCInstLowering.Lower(MI, TmpInst);
 192:   OutStreamer->emitInstruction(TmpInst, STI);
 193: }
 194: 
 195: void LanaiAsmPrinter::emitInstruction(const MachineInstr *MI) {
 196:   Lanai_MC::verifyInstructionPredicates(MI->getOpcode(),
 197:                                         getSubtargetInfo().getFeatureBits());
 198: 
 199:   MachineBasicBlock::const_instr_iterator I = MI->getIterator();
 200:   MachineBasicBlock::const_instr_iterator E = MI->getParent()->instr_end();
```

- EN: Function bodies or method definitions such as customEmitInstruction, emitInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: customEmitInstruction, emitInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 201-220

```cpp
 201: 
 202:   do {
 203:     if (I->isCall()) {
 204:       emitCallInstruction(&*I);
 205:       continue;
 206:     }
 207: 
 208:     customEmitInstruction(&*I);
 209:   } while ((++I != E) && I->isInsideBundle());
 210: }
 211: 
 212: // isBlockOnlyReachableByFallthough - Return true if the basic block has
 213: // exactly one predecessor and the control transfer mechanism between
 214: // the predecessor and this block is a fall-through.
 215: // FIXME: could the overridden cases be handled in analyzeBranch?
 216: bool LanaiAsmPrinter::isBlockOnlyReachableByFallthrough(
 217:     const MachineBasicBlock *MBB) const {
 218:   // The predecessor has to be immediately before this block.
 219:   const MachineBasicBlock *Pred = *MBB->pred_begin();
 220: 
```

- EN: Function bodies or method definitions such as isBlockOnlyReachableByFallthrough contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: isBlockOnlyReachableByFallthrough 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 221-240

```cpp
 221:   // If the predecessor is a switch statement, assume a jump table
 222:   // implementation, so it is not a fall through.
 223:   if (const BasicBlock *B = Pred->getBasicBlock())
 224:     if (isa<SwitchInst>(B->getTerminator()))
 225:       return false;
 226: 
 227:   // Check default implementation
 228:   if (!AsmPrinter::isBlockOnlyReachableByFallthrough(MBB))
 229:     return false;
 230: 
 231:   // Otherwise, check the last instruction.
 232:   // Check if the last terminator is an unconditional branch.
 233:   MachineBasicBlock::const_iterator I = Pred->end();
 234:   while (I != Pred->begin() && !(--I)->isTerminator()) {
 235:   }
 236: 
 237:   return !I->isBarrier();
 238: }
 239: 
 240: char LanaiAsmPrinter::ID = 0;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，它参与将机器指令打印或降级为汇编输出。

### Lines 241-249

```cpp
 241: 
 242: INITIALIZE_PASS(LanaiAsmPrinter, "lanai-asm-printer", "Lanai Assembly Printer",
 243:                 false, false)
 244: 
 245: // Force static initialization.
 246: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
 247: LLVMInitializeLanaiAsmPrinter() {
 248:   RegisterAsmPrinter<LanaiAsmPrinter> X(getTheLanaiTarget());
 249: }
```

- EN: In file-level terms, it participates in printing or lowering machine instructions into assembly output.
- 中文: 从文件角色看，它参与将机器指令打印或降级为汇编输出。

## Key Concepts / 关键概念

- MachineInstr to MC lowering / MachineInstr 到 MC 降级
- Assembly syntax emission / 汇编语法输出
- Machine instruction manipulation / 机器指令操作
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiAluCode.h`, `LanaiCondCode.h`, `LanaiMCInstLower.h`, `LanaiTargetMachine.h`, `MCTargetDesc/LanaiInstPrinter.h`, `TargetInfo/LanaiTargetInfo.h`, `llvm/CodeGen/AsmPrinter.h`, `llvm/CodeGen/MachineConstantPool.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
