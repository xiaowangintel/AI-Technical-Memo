# BPFInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFInstrInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the BPF implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFInstrInfo.cpp - BPF Instruction Information ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the BPF implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "BPFInstrInfo.h"
  14: #include "BPF.h"
  15: #include "BPFSubtarget.h"
  16: #include "llvm/ADT/SmallVector.h"
  17: #include "llvm/CodeGen/MachineBasicBlock.h"
  18: #include "llvm/CodeGen/MachineInstrBuilder.h"
  19: #include "llvm/IR/DebugLoc.h"
  20: #include "llvm/Support/ErrorHandling.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: #include <cassert>
  22: #include <iterator>
  23: 
  24: #define GET_INSTRINFO_CTOR_DTOR
  25: #include "BPFGenInstrInfo.inc"
  26: 
  27: using namespace llvm;
  28: 
  29: BPFInstrInfo::BPFInstrInfo(const BPFSubtarget &STI)
  30:     : BPFGenInstrInfo(STI, RI, BPF::ADJCALLSTACKDOWN, BPF::ADJCALLSTACKUP) {}
  31: 
  32: void BPFInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
  33:                                MachineBasicBlock::iterator I,
  34:                                const DebugLoc &DL, Register DestReg,
  35:                                Register SrcReg, bool KillSrc,
  36:                                bool RenamableDest, bool RenamableSrc) const {
  37:   if (BPF::GPRRegClass.contains(DestReg, SrcReg))
  38:     BuildMI(MBB, I, DL, get(BPF::MOV_rr), DestReg)
  39:         .addReg(SrcReg, getKillRegState(KillSrc));
  40:   else if (BPF::GPR32RegClass.contains(DestReg, SrcReg))
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as BPFGenInstrInfo, copyPhysReg contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 BPFGenInstrInfo, copyPhysReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41:     BuildMI(MBB, I, DL, get(BPF::MOV_rr_32), DestReg)
  42:         .addReg(SrcReg, getKillRegState(KillSrc));
  43:   else
  44:     llvm_unreachable("Impossible reg-to-reg copy");
  45: }
  46: 
  47: void BPFInstrInfo::expandMEMCPY(MachineBasicBlock::iterator MI) const {
  48:   Register DstReg = MI->getOperand(0).getReg();
  49:   Register SrcReg = MI->getOperand(1).getReg();
  50:   uint64_t CopyLen = MI->getOperand(2).getImm();
  51:   uint64_t Alignment = MI->getOperand(3).getImm();
  52:   Register ScratchReg = MI->getOperand(4).getReg();
  53:   MachineBasicBlock *BB = MI->getParent();
  54:   DebugLoc dl = MI->getDebugLoc();
  55:   unsigned LdOpc, StOpc;
  56: 
  57:   switch (Alignment) {
  58:   case 1:
  59:     LdOpc = BPF::LDB;
  60:     StOpc = BPF::STB;
```

- EN: Function bodies or method definitions such as expandMEMCPY contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: expandMEMCPY 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:     break;
  62:   case 2:
  63:     LdOpc = BPF::LDH;
  64:     StOpc = BPF::STH;
  65:     break;
  66:   case 4:
  67:     LdOpc = BPF::LDW;
  68:     StOpc = BPF::STW;
  69:     break;
  70:   case 8:
  71:     LdOpc = BPF::LDD;
  72:     StOpc = BPF::STD;
  73:     break;
  74:   default:
  75:     llvm_unreachable("unsupported memcpy alignment");
  76:   }
  77: 
  78:   unsigned IterationNum = CopyLen >> Log2_64(Alignment);
  79:   for(unsigned I = 0; I < IterationNum; ++I) {
  80:     BuildMI(*BB, MI, dl, get(LdOpc))
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81:             .addReg(ScratchReg, RegState::Define).addReg(SrcReg)
  82:             .addImm(I * Alignment);
  83:     BuildMI(*BB, MI, dl, get(StOpc))
  84:             .addReg(ScratchReg, RegState::Kill).addReg(DstReg)
  85:             .addImm(I * Alignment);
  86:   }
  87: 
  88:   unsigned BytesLeft = CopyLen & (Alignment - 1);
  89:   unsigned Offset = IterationNum * Alignment;
  90:   bool Hanging4Byte = BytesLeft & 0x4;
  91:   bool Hanging2Byte = BytesLeft & 0x2;
  92:   bool Hanging1Byte = BytesLeft & 0x1;
  93:   if (Hanging4Byte) {
  94:     BuildMI(*BB, MI, dl, get(BPF::LDW))
  95:             .addReg(ScratchReg, RegState::Define).addReg(SrcReg).addImm(Offset);
  96:     BuildMI(*BB, MI, dl, get(BPF::STW))
  97:             .addReg(ScratchReg, RegState::Kill).addReg(DstReg).addImm(Offset);
  98:     Offset += 4;
  99:   }
 100:   if (Hanging2Byte) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-120

```cpp
 101:     BuildMI(*BB, MI, dl, get(BPF::LDH))
 102:             .addReg(ScratchReg, RegState::Define).addReg(SrcReg).addImm(Offset);
 103:     BuildMI(*BB, MI, dl, get(BPF::STH))
 104:             .addReg(ScratchReg, RegState::Kill).addReg(DstReg).addImm(Offset);
 105:     Offset += 2;
 106:   }
 107:   if (Hanging1Byte) {
 108:     BuildMI(*BB, MI, dl, get(BPF::LDB))
 109:             .addReg(ScratchReg, RegState::Define).addReg(SrcReg).addImm(Offset);
 110:     BuildMI(*BB, MI, dl, get(BPF::STB))
 111:             .addReg(ScratchReg, RegState::Kill).addReg(DstReg).addImm(Offset);
 112:   }
 113: 
 114:   BB->erase(MI);
 115: }
 116: 
 117: bool BPFInstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
 118:   if (MI.getOpcode() == BPF::MEMCPY) {
 119:     expandMEMCPY(MI);
 120:     return true;
```

- EN: Function bodies or method definitions such as expandPostRAPseudo contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: expandPostRAPseudo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 121-140

```cpp
 121:   }
 122: 
 123:   return false;
 124: }
 125: 
 126: void BPFInstrInfo::storeRegToStackSlot(MachineBasicBlock &MBB,
 127:                                        MachineBasicBlock::iterator I,
 128:                                        Register SrcReg, bool IsKill, int FI,
 129:                                        const TargetRegisterClass *RC,
 130:                                        Register VReg,
 131:                                        MachineInstr::MIFlag Flags) const {
 132:   DebugLoc DL;
 133:   if (I != MBB.end())
 134:     DL = I->getDebugLoc();
 135: 
 136:   if (RC == &BPF::GPRRegClass)
 137:     BuildMI(MBB, I, DL, get(BPF::STD))
 138:         .addReg(SrcReg, getKillRegState(IsKill))
 139:         .addFrameIndex(FI)
 140:         .addImm(0);
```

- EN: Function bodies or method definitions such as storeRegToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: storeRegToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 141-160

```cpp
 141:   else if (RC == &BPF::GPR32RegClass)
 142:     BuildMI(MBB, I, DL, get(BPF::STW32))
 143:         .addReg(SrcReg, getKillRegState(IsKill))
 144:         .addFrameIndex(FI)
 145:         .addImm(0);
 146:   else
 147:     llvm_unreachable("Can't store this register to stack slot");
 148: }
 149: 
 150: void BPFInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
 151:                                         MachineBasicBlock::iterator I,
 152:                                         Register DestReg, int FI,
 153:                                         const TargetRegisterClass *RC,
 154:                                         Register VReg, unsigned SubReg,
 155:                                         MachineInstr::MIFlag Flags) const {
 156:   DebugLoc DL;
 157:   if (I != MBB.end())
 158:     DL = I->getDebugLoc();
 159: 
 160:   if (RC == &BPF::GPRRegClass)
```

- EN: Function bodies or method definitions such as loadRegFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: loadRegFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 161-180

```cpp
 161:     BuildMI(MBB, I, DL, get(BPF::LDD), DestReg).addFrameIndex(FI).addImm(0);
 162:   else if (RC == &BPF::GPR32RegClass)
 163:     BuildMI(MBB, I, DL, get(BPF::LDW32), DestReg).addFrameIndex(FI).addImm(0);
 164:   else
 165:     llvm_unreachable("Can't load this register from stack slot");
 166: }
 167: 
 168: bool BPFInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 169:                                  MachineBasicBlock *&TBB,
 170:                                  MachineBasicBlock *&FBB,
 171:                                  SmallVectorImpl<MachineOperand> &Cond,
 172:                                  bool AllowModify) const {
 173:   // Start from the bottom of the block and work up, examining the
 174:   // terminator instructions.
 175:   MachineBasicBlock::iterator I = MBB.end();
 176:   while (I != MBB.begin()) {
 177:     --I;
 178:     if (I->isDebugInstr())
 179:       continue;
 180: 
```

- EN: Function bodies or method definitions such as analyzeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: analyzeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 181-200

```cpp
 181:     // Working from the bottom, when we see a non-terminator
 182:     // instruction, we're done.
 183:     if (!isUnpredicatedTerminator(*I))
 184:       break;
 185: 
 186:     // From base method doc: ... returning true if it cannot be understood ...
 187:     // Indirect branch has multiple destinations and no true/false concepts.
 188:     if (I->isIndirectBranch())
 189:       return true;
 190: 
 191:     // A terminator that isn't a branch can't easily be handled
 192:     // by this analysis.
 193:     if (!I->isBranch())
 194:       return true;
 195: 
 196:     // Handle unconditional branches.
 197:     if (I->getOpcode() == BPF::JMP) {
 198:       if (!AllowModify) {
 199:         TBB = I->getOperand(0).getMBB();
 200:         continue;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 201-220

```cpp
 201:       }
 202: 
 203:       // If the block has any instructions after a J, delete them.
 204:       MBB.erase(std::next(I), MBB.end());
 205:       Cond.clear();
 206:       FBB = nullptr;
 207: 
 208:       // Delete the J if it's equivalent to a fall-through.
 209:       if (MBB.isLayoutSuccessor(I->getOperand(0).getMBB())) {
 210:         TBB = nullptr;
 211:         I->eraseFromParent();
 212:         I = MBB.end();
 213:         continue;
 214:       }
 215: 
 216:       // TBB is used to indicate the unconditinal destination.
 217:       TBB = I->getOperand(0).getMBB();
 218:       continue;
 219:     }
 220:     // Cannot handle conditional branches
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 221-240

```cpp
 221:     return true;
 222:   }
 223: 
 224:   return false;
 225: }
 226: 
 227: unsigned BPFInstrInfo::insertBranch(MachineBasicBlock &MBB,
 228:                                     MachineBasicBlock *TBB,
 229:                                     MachineBasicBlock *FBB,
 230:                                     ArrayRef<MachineOperand> Cond,
 231:                                     const DebugLoc &DL,
 232:                                     int *BytesAdded) const {
 233:   assert(!BytesAdded && "code size not handled");
 234: 
 235:   // Shouldn't be a fall through.
 236:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
 237: 
 238:   if (Cond.empty()) {
 239:     // Unconditional branch
 240:     assert(!FBB && "Unconditional branch with multiple successors!");
```

- EN: Function bodies or method definitions such as insertBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: insertBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 241-260

```cpp
 241:     BuildMI(&MBB, DL, get(BPF::JMP)).addMBB(TBB);
 242:     return 1;
 243:   }
 244: 
 245:   llvm_unreachable("Unexpected conditional branch");
 246: }
 247: 
 248: unsigned BPFInstrInfo::removeBranch(MachineBasicBlock &MBB,
 249:                                     int *BytesRemoved) const {
 250:   assert(!BytesRemoved && "code size not handled");
 251: 
 252:   MachineBasicBlock::iterator I = MBB.end();
 253:   unsigned Count = 0;
 254: 
 255:   while (I != MBB.begin()) {
 256:     --I;
 257:     if (I->isDebugInstr())
 258:       continue;
 259:     if (I->getOpcode() != BPF::JMP)
 260:       break;
```

- EN: Function bodies or method definitions such as removeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: removeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 261-280

```cpp
 261:     // Remove the branch.
 262:     I->eraseFromParent();
 263:     I = MBB.end();
 264:     ++Count;
 265:   }
 266: 
 267:   return Count;
 268: }
 269: 
 270: int BPFInstrInfo::getJumpTableIndex(const MachineInstr &MI) const {
 271:   if (MI.getOpcode() != BPF::JX)
 272:     return -1;
 273: 
 274:   // The pattern looks like:
 275:   // %0 = LD_imm64 %jump-table.0   ; load jump-table address
 276:   // %1 = ADD_rr %0, $another_reg  ; address + offset
 277:   // %2 = LDD %1, 0                ; load the actual label
 278:   // JX %2
 279:   const MachineFunction &MF = *MI.getParent()->getParent();
 280:   const MachineRegisterInfo &MRI = MF.getRegInfo();
```

- EN: Function bodies or method definitions such as getJumpTableIndex contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getJumpTableIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 281-300

```cpp
 281: 
 282:   Register Reg = MI.getOperand(0).getReg();
 283:   if (!Reg.isVirtual())
 284:     return -1;
 285:   MachineInstr *Ldd = MRI.getUniqueVRegDef(Reg);
 286:   if (Ldd == nullptr || Ldd->getOpcode() != BPF::LDD)
 287:     return -1;
 288: 
 289:   Reg = Ldd->getOperand(1).getReg();
 290:   if (!Reg.isVirtual())
 291:     return -1;
 292:   MachineInstr *Add = MRI.getUniqueVRegDef(Reg);
 293:   if (Add == nullptr || Add->getOpcode() != BPF::ADD_rr)
 294:     return -1;
 295: 
 296:   Reg = Add->getOperand(1).getReg();
 297:   if (!Reg.isVirtual())
 298:     return -1;
 299:   MachineInstr *LDimm64 = MRI.getUniqueVRegDef(Reg);
 300:   if (LDimm64 == nullptr || LDimm64->getOpcode() != BPF::LD_imm64)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 301-308

```cpp
 301:     return -1;
 302: 
 303:   const MachineOperand &MO = LDimm64->getOperand(1);
 304:   if (!MO.isJTI())
 305:     return -1;
 306: 
 307:   return MO.getIndex();
 308: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPFInstrInfo.h`, `BPF.h`, `BPFSubtarget.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/IR/DebugLoc.h`, `llvm/Support/ErrorHandling.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `BPFGenInstrInfo.inc`
- Local companions / 本地配套文件: `BPFInstrInfo.h`, `BPFInstrInfo.td`
