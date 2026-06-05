# SystemZPostRewrite.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZPostRewrite.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //==---- SystemZPostRewrite.cpp - Select pseudos after RegAlloc ---*- C++ -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a pass that is run immediately after VirtRegRewriter
  10: // but before MachineCopyPropagation. The purpose is to lower pseudos to
  11: // target instructions before any later pass might substitute a register for
  12: // another.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: #include "SystemZ.h"
  17: #include "SystemZInstrInfo.h"
  18: #include "SystemZSubtarget.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZ.h`, `SystemZInstrInfo.h`, `SystemZSubtarget.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZ.h`, `SystemZInstrInfo.h`, `SystemZSubtarget.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/ADT/Statistic.h"
  20: #include "llvm/CodeGen/LivePhysRegs.h"
  21: #include "llvm/CodeGen/MachineFunctionPass.h"
  22: #include "llvm/CodeGen/MachineInstr.h"
  23: #include "llvm/CodeGen/MachineInstrBuilder.h"
  24: using namespace llvm;
  25: 
  26: #define DEBUG_TYPE "systemz-postrewrite"
  27: STATISTIC(MemFoldCopies, "Number of copies inserted before folded mem ops.");
  28: STATISTIC(LOCRMuxJumps, "Number of LOCRMux jump-sequences (lower is better)");
  29: 
  30: namespace {
  31: 
  32: class SystemZPostRewrite : public MachineFunctionPass {
  33: public:
  34:   static char ID;
  35:   SystemZPostRewrite() : MachineFunctionPass(ID) {}
  36: 
```
- **EN**: It imports dependencies such as `Statistic.h`, `LivePhysRegs.h`, `MachineFunctionPass.h`, `MachineInstr.h`, `MachineInstrBuilder.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZPostRewrite`. The range implements or declares functions including `SystemZPostRewrite`.
- **CN**: 它引入了 `Statistic.h`, `LivePhysRegs.h`, `MachineFunctionPass.h`, `MachineInstr.h`, `MachineInstrBuilder.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZPostRewrite` 等 TableGen 记录。 这一段实现或声明了 `SystemZPostRewrite` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   const SystemZInstrInfo *TII;
  38: 
  39:   bool runOnMachineFunction(MachineFunction &Fn) override;
  40: 
  41: private:
  42:   void selectLOCRMux(MachineBasicBlock &MBB,
  43:                      MachineBasicBlock::iterator MBBI,
  44:                      MachineBasicBlock::iterator &NextMBBI,
  45:                      unsigned LowOpcode,
  46:                      unsigned HighOpcode);
  47:   void selectSELRMux(MachineBasicBlock &MBB,
  48:                      MachineBasicBlock::iterator MBBI,
  49:                      MachineBasicBlock::iterator &NextMBBI,
  50:                      unsigned LowOpcode,
  51:                      unsigned HighOpcode);
  52:   bool expandCondMove(MachineBasicBlock &MBB,
  53:                       MachineBasicBlock::iterator MBBI,
  54:                       MachineBasicBlock::iterator &NextMBBI);
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   bool selectMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  56:                 MachineBasicBlock::iterator &NextMBBI);
  57:   bool selectMBB(MachineBasicBlock &MBB);
  58: };
  59: 
  60: char SystemZPostRewrite::ID = 0;
  61: 
  62: } // end anonymous namespace
  63: 
  64: INITIALIZE_PASS(SystemZPostRewrite, "systemz-post-rewrite",
  65:                 "SystemZ Post Rewrite pass", false, false)
  66: 
  67: /// Returns an instance of the Post Rewrite pass.
  68: FunctionPass *llvm::createSystemZPostRewritePass(SystemZTargetMachine &TM) {
  69:   return new SystemZPostRewrite();
  70: }
  71: 
  72: // MI is a load-register-on-condition pseudo instruction.  Replace it with
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73: // LowOpcode if source and destination are both low GR32s and HighOpcode if
  74: // source and destination are both high GR32s. Otherwise, a branch sequence
  75: // is created.
  76: void SystemZPostRewrite::selectLOCRMux(MachineBasicBlock &MBB,
  77:                                        MachineBasicBlock::iterator MBBI,
  78:                                        MachineBasicBlock::iterator &NextMBBI,
  79:                                        unsigned LowOpcode,
  80:                                        unsigned HighOpcode) {
  81:   Register DestReg = MBBI->getOperand(0).getReg();
  82:   Register SrcReg = MBBI->getOperand(2).getReg();
  83:   bool DestIsHigh = SystemZ::isHighReg(DestReg);
  84:   bool SrcIsHigh = SystemZ::isHighReg(SrcReg);
  85: 
  86:   if (!DestIsHigh && !SrcIsHigh)
  87:     MBBI->setDesc(TII->get(LowOpcode));
  88:   else if (DestIsHigh && SrcIsHigh)
  89:     MBBI->setDesc(TII->get(HighOpcode));
  90:   else
```
- **EN**: The range implements or declares functions including `SystemZPostRewrite::selectLOCRMux`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPostRewrite::selectLOCRMux` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     expandCondMove(MBB, MBBI, NextMBBI);
  92: }
  93: 
  94: // MI is a select pseudo instruction.  Replace it with LowOpcode if source
  95: // and destination are all low GR32s and HighOpcode if source and destination
  96: // are all high GR32s. Otherwise, a branch sequence is created.
  97: void SystemZPostRewrite::selectSELRMux(MachineBasicBlock &MBB,
  98:                                        MachineBasicBlock::iterator MBBI,
  99:                                        MachineBasicBlock::iterator &NextMBBI,
 100:                                        unsigned LowOpcode,
 101:                                        unsigned HighOpcode) {
 102:   Register DestReg = MBBI->getOperand(0).getReg();
 103:   MachineOperand &Src1MO = MBBI->getOperand(1);
 104:   MachineOperand &Src2MO = MBBI->getOperand(2);
 105:   Register Src1Reg = Src1MO.getReg();
 106:   Register Src2Reg = Src2MO.getReg();
 107:   bool DestIsHigh = SystemZ::isHighReg(DestReg);
 108:   bool Src1IsHigh = SystemZ::isHighReg(Src1Reg);
```
- **EN**: The range implements or declares functions including `SystemZPostRewrite::selectSELRMux`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPostRewrite::selectSELRMux` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   bool Src2IsHigh = SystemZ::isHighReg(Src2Reg);
 110:   // A copy instruction that we might create, held here for the purpose of
 111:   // debug instr value tracking.
 112:   MachineInstr *CopyInst = nullptr;
 113: 
 114:   // In rare cases both sources are the same register (after
 115:   // machine-cse). This must be handled as it may lead to wrong-code (after
 116:   // machine-cp) if the kill flag on Src1 isn't cleared (with
 117:   // expandCondMove()).
 118:   if (Src1Reg == Src2Reg) {
 119:     CopyInst = BuildMI(*MBBI->getParent(), MBBI, MBBI->getDebugLoc(),
 120:                        TII->get(SystemZ::COPY), DestReg)
 121:                    .addReg(Src1Reg, getRegState(Src1MO) & getRegState(Src2MO));
 122:     MBB.getParent()->substituteDebugValuesForInst(*MBBI, *CopyInst, 1);
 123:     MBBI->eraseFromParent();
 124:     return;
 125:   }
 126: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   // If sources and destination aren't all high or all low, we may be able to
 128:   // simplify the operation by moving one of the sources to the destination
 129:   // first.  But only if this doesn't clobber the other source.
 130:   if (DestReg != Src1Reg && DestReg != Src2Reg) {
 131:     if (DestIsHigh != Src1IsHigh) {
 132:       CopyInst = BuildMI(*MBBI->getParent(), MBBI, MBBI->getDebugLoc(),
 133:                          TII->get(SystemZ::COPY), DestReg)
 134:                      .addReg(Src1Reg, getRegState(Src1MO));
 135:       Src1MO.setReg(DestReg);
 136:       Src1Reg = DestReg;
 137:       Src1IsHigh = DestIsHigh;
 138:     } else if (DestIsHigh != Src2IsHigh) {
 139:       CopyInst = BuildMI(*MBBI->getParent(), MBBI, MBBI->getDebugLoc(),
 140:                          TII->get(SystemZ::COPY), DestReg)
 141:                      .addReg(Src2Reg, getRegState(Src2MO));
 142:       Src2MO.setReg(DestReg);
 143:       Src2Reg = DestReg;
 144:       Src2IsHigh = DestIsHigh;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     }
 146:   }
 147:   // if a copy instruction was inserted, record the debug value substitution
 148:   if (CopyInst)
 149:     MBB.getParent()->substituteDebugValuesForInst(*MBBI, *CopyInst, 1);
 150: 
 151:   // If the destination (now) matches one source, prefer this to be first.
 152:   if (DestReg != Src1Reg && DestReg == Src2Reg) {
 153:     TII->commuteInstruction(*MBBI, false, 1, 2);
 154:     std::swap(Src1Reg, Src2Reg);
 155:     std::swap(Src1IsHigh, Src2IsHigh);
 156:   }
 157: 
 158:   if (!DestIsHigh && !Src1IsHigh && !Src2IsHigh)
 159:     MBBI->setDesc(TII->get(LowOpcode));
 160:   else if (DestIsHigh && Src1IsHigh && Src2IsHigh)
 161:     MBBI->setDesc(TII->get(HighOpcode));
 162:   else
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     // Given the simplification above, we must already have a two-operand case.
 164:     expandCondMove(MBB, MBBI, NextMBBI);
 165: }
 166: 
 167: // Replace MBBI by a branch sequence that performs a conditional move of
 168: // operand 2 to the destination register. Operand 1 is expected to be the
 169: // same register as the destination.
 170: bool SystemZPostRewrite::expandCondMove(MachineBasicBlock &MBB,
 171:                                         MachineBasicBlock::iterator MBBI,
 172:                                         MachineBasicBlock::iterator &NextMBBI) {
 173:   MachineFunction &MF = *MBB.getParent();
 174:   const BasicBlock *BB = MBB.getBasicBlock();
 175:   MachineInstr &MI = *MBBI;
 176:   DebugLoc DL = MI.getDebugLoc();
 177:   Register DestReg = MI.getOperand(0).getReg();
 178:   Register SrcReg = MI.getOperand(2).getReg();
 179:   unsigned CCValid = MI.getOperand(3).getImm();
 180:   unsigned CCMask = MI.getOperand(4).getImm();
```
- **EN**: The range implements or declares functions including `SystemZPostRewrite::expandCondMove`.
- **CN**: 这一段实现或声明了 `SystemZPostRewrite::expandCondMove` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   assert(DestReg == MI.getOperand(1).getReg() &&
 182:          "Expected destination and first source operand to be the same.");
 183: 
 184:   LivePhysRegs LiveRegs(TII->getRegisterInfo());
 185:   LiveRegs.addLiveOuts(MBB);
 186:   for (auto I = std::prev(MBB.end()); I != MBBI; --I)
 187:     LiveRegs.stepBackward(*I);
 188: 
 189:   // Splice MBB at MI, moving the rest of the block into RestMBB.
 190:   MachineBasicBlock *RestMBB = MF.CreateMachineBasicBlock(BB);
 191:   MF.insert(std::next(MachineFunction::iterator(MBB)), RestMBB);
 192:   RestMBB->splice(RestMBB->begin(), &MBB, MI, MBB.end());
 193:   RestMBB->transferSuccessors(&MBB);
 194:   for (MCPhysReg R : LiveRegs)
 195:     RestMBB->addLiveIn(R);
 196: 
 197:   // Create a new block MoveMBB to hold the move instruction.
 198:   MachineBasicBlock *MoveMBB = MF.CreateMachineBasicBlock(BB);
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   MF.insert(std::next(MachineFunction::iterator(MBB)), MoveMBB);
 200:   MoveMBB->addLiveIn(SrcReg);
 201:   for (MCPhysReg R : LiveRegs)
 202:     MoveMBB->addLiveIn(R);
 203: 
 204:   // At the end of MBB, create a conditional branch to RestMBB if the
 205:   // condition is false, otherwise fall through to MoveMBB.
 206:   BuildMI(&MBB, DL, TII->get(SystemZ::BRC))
 207:     .addImm(CCValid).addImm(CCMask ^ CCValid).addMBB(RestMBB);
 208:   MBB.addSuccessor(RestMBB);
 209:   MBB.addSuccessor(MoveMBB);
 210: 
 211:   // In MoveMBB, emit an instruction to move SrcReg into DestReg,
 212:   // then fall through to RestMBB.
 213:   MachineInstr *CopyInst =
 214:       BuildMI(*MoveMBB, MoveMBB->end(), DL, TII->get(SystemZ::COPY), DestReg)
 215:           .addReg(MI.getOperand(2).getReg(), getRegState(MI.getOperand(2)));
 216:   // record the debug value substitution for CopyInst
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   MBB.getParent()->substituteDebugValuesForInst(*MBBI, *CopyInst, 1);
 218:   MoveMBB->addSuccessor(RestMBB);
 219: 
 220:   NextMBBI = MBB.end();
 221:   MI.eraseFromParent();
 222:   LOCRMuxJumps++;
 223:   return true;
 224: }
 225: 
 226: /// If MBBI references a pseudo instruction that should be selected here,
 227: /// do it and return true.  Otherwise return false.
 228: bool SystemZPostRewrite::selectMI(MachineBasicBlock &MBB,
 229:                                   MachineBasicBlock::iterator MBBI,
 230:                                   MachineBasicBlock::iterator &NextMBBI) {
 231:   MachineInstr &MI = *MBBI;
 232:   unsigned Opcode = MI.getOpcode();
 233: 
 234:   // Note: If this could be done during regalloc in foldMemoryOperandImpl()
```
- **EN**: The range implements or declares functions including `SystemZPostRewrite::selectMI`.
- **CN**: 这一段实现或声明了 `SystemZPostRewrite::selectMI` 等函数。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   // while also updating the LiveIntervals, there would be no need for the
 236:   // MemFoldPseudo to begin with.
 237:   int TargetMemOpcode = SystemZ::getTargetMemOpcode(Opcode);
 238:   if (TargetMemOpcode != -1) {
 239:     MI.setDesc(TII->get(TargetMemOpcode));
 240:     MI.tieOperands(0, 1);
 241:     Register DstReg = MI.getOperand(0).getReg();
 242:     MachineOperand &SrcMO = MI.getOperand(1);
 243:     if (DstReg != SrcMO.getReg()) {
 244:       BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(SystemZ::COPY), DstReg)
 245:         .addReg(SrcMO.getReg());
 246:       SrcMO.setReg(DstReg);
 247:       MemFoldCopies++;
 248:     }
 249:     return true;
 250:   }
 251: 
 252:   switch (Opcode) {
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253:   case SystemZ::LOCRMux:
 254:     selectLOCRMux(MBB, MBBI, NextMBBI, SystemZ::LOCR, SystemZ::LOCFHR);
 255:     return true;
 256:   case SystemZ::SELRMux:
 257:     selectSELRMux(MBB, MBBI, NextMBBI, SystemZ::SELR, SystemZ::SELFHR);
 258:     return true;
 259:   }
 260: 
 261:   return false;
 262: }
 263: 
 264: /// Iterate over the instructions in basic block MBB and select any
 265: /// pseudo instructions.  Return true if anything was modified.
 266: bool SystemZPostRewrite::selectMBB(MachineBasicBlock &MBB) {
 267:   bool Modified = false;
 268: 
 269:   MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
 270:   while (MBBI != E) {
```
- **EN**: The range implements or declares functions including `SystemZPostRewrite::selectMBB`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZPostRewrite::selectMBB` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 271-288 / 第 271-288 行
```cpp
 271:     MachineBasicBlock::iterator NMBBI = std::next(MBBI);
 272:     Modified |= selectMI(MBB, MBBI, NMBBI);
 273:     MBBI = NMBBI;
 274:   }
 275: 
 276:   return Modified;
 277: }
 278: 
 279: bool SystemZPostRewrite::runOnMachineFunction(MachineFunction &MF) {
 280:   TII = MF.getSubtarget<SystemZSubtarget>().getInstrInfo();
 281: 
 282:   bool Modified = false;
 283:   for (auto &MBB : MF)
 284:     Modified |= selectMBB(MBB);
 285: 
 286:   return Modified;
 287: }
 288: 
```
- **EN**: The range implements or declares functions including `SystemZPostRewrite::runOnMachineFunction`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZPostRewrite::runOnMachineFunction` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZ.h`
- `SystemZInstrInfo.h`
- `SystemZSubtarget.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/LivePhysRegs.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
