# LoongArchExpandAtomicPseudoInsts.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchExpandAtomicPseudoInsts.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the LoongArch backend.
- **用途 (CN)**: 提供 LoongArch 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //==- LoongArchExpandAtomicPseudoInsts.cpp - Expand atomic pseudo instrs. -===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains a pass that expands atomic pseudo instructions into
  10: // target instructions. This pass should be run at the last possible moment,
  11: // avoiding the possibility for other passes to break the requirements for
  12: // forward progress in the LL/SC block.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: #include "LoongArch.h"
  17: #include "LoongArchInstrInfo.h"
  18: #include "LoongArchTargetMachine.h"
  19: 
  20: #include "llvm/CodeGen/LivePhysRegs.h"
  21: #include "llvm/CodeGen/MachineFunctionPass.h"
  22: #include "llvm/CodeGen/MachineInstrBuilder.h"
  23: 
  24: using namespace llvm;
  25: 
  26: #define LoongArch_EXPAND_ATOMIC_PSEUDO_NAME                                    \
  27:   "LoongArch atomic pseudo instruction expansion pass"
  28: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArch.h`, `LoongArchInstrInfo.h`, `LoongArchTargetMachine.h`, `LivePhysRegs.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArch.h`, `LoongArchInstrInfo.h`, `LoongArchTargetMachine.h`, `LivePhysRegs.h`, `MachineFunctionPass.h`, `MachineInstrBuilder.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29: namespace {
  30: 
  31: class LoongArchExpandAtomicPseudo : public MachineFunctionPass {
  32: public:
  33:   const LoongArchInstrInfo *TII;
  34:   static char ID;
  35: 
  36:   LoongArchExpandAtomicPseudo() : MachineFunctionPass(ID) {}
  37: 
  38:   bool runOnMachineFunction(MachineFunction &MF) override;
  39: 
  40:   StringRef getPassName() const override {
  41:     return LoongArch_EXPAND_ATOMIC_PSEUDO_NAME;
  42:   }
  43: 
  44: private:
  45:   bool expandMBB(MachineBasicBlock &MBB);
  46:   bool expandMI(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  47:                 MachineBasicBlock::iterator &NextMBBI);
  48:   bool expandAtomicBinOp(MachineBasicBlock &MBB,
  49:                          MachineBasicBlock::iterator MBBI, AtomicRMWInst::BinOp,
  50:                          bool IsMasked, int Width,
  51:                          MachineBasicBlock::iterator &NextMBBI);
  52:   bool expandAtomicMinMaxOp(MachineBasicBlock &MBB,
  53:                             MachineBasicBlock::iterator MBBI,
  54:                             AtomicRMWInst::BinOp, bool IsMasked, int Width,
  55:                             MachineBasicBlock::iterator &NextMBBI);
  56:   bool expandAtomicCmpXchg(MachineBasicBlock &MBB,
```
- **EN**: This block declares or refines TableGen records such as `LoongArchExpandAtomicPseudo`. The range implements or declares functions including `LoongArchExpandAtomicPseudo`.
- **CN**: 该代码块声明或细化了 `LoongArchExpandAtomicPseudo` 等 TableGen 记录。 这一段实现或声明了 `LoongArchExpandAtomicPseudo` 等函数。

### Lines 57-84 / 第 57-84 行
```cpp
  57:                            MachineBasicBlock::iterator MBBI, bool IsMasked,
  58:                            int Width, MachineBasicBlock::iterator &NextMBBI);
  59:   bool expandAtomicCmpXchg128(MachineBasicBlock &MBB,
  60:                               MachineBasicBlock::iterator,
  61:                               MachineBasicBlock::iterator &NextMBBI);
  62: };
  63: 
  64: char LoongArchExpandAtomicPseudo::ID = 0;
  65: 
  66: bool LoongArchExpandAtomicPseudo::runOnMachineFunction(MachineFunction &MF) {
  67:   TII =
  68:       static_cast<const LoongArchInstrInfo *>(MF.getSubtarget().getInstrInfo());
  69:   bool Modified = false;
  70:   for (auto &MBB : MF)
  71:     Modified |= expandMBB(MBB);
  72:   return Modified;
  73: }
  74: 
  75: bool LoongArchExpandAtomicPseudo::expandMBB(MachineBasicBlock &MBB) {
  76:   bool Modified = false;
  77: 
  78:   MachineBasicBlock::iterator MBBI = MBB.begin(), E = MBB.end();
  79:   while (MBBI != E) {
  80:     MachineBasicBlock::iterator NMBBI = std::next(MBBI);
  81:     Modified |= expandMI(MBB, MBBI, NMBBI);
  82:     MBBI = NMBBI;
  83:   }
  84: 
```
- **EN**: The range implements or declares functions including `LoongArchExpandAtomicPseudo::runOnMachineFunction`, `LoongArchExpandAtomicPseudo::expandMBB`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchExpandAtomicPseudo::runOnMachineFunction`, `LoongArchExpandAtomicPseudo::expandMBB` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   return Modified;
  86: }
  87: 
  88: bool LoongArchExpandAtomicPseudo::expandMI(
  89:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  90:     MachineBasicBlock::iterator &NextMBBI) {
  91:   switch (MBBI->getOpcode()) {
  92:   case LoongArch::PseudoMaskedAtomicSwap32:
  93:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xchg, true, 32,
  94:                              NextMBBI);
  95:   case LoongArch::PseudoAtomicSwap32:
  96:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xchg, false, 32,
  97:                              NextMBBI);
  98:   case LoongArch::PseudoMaskedAtomicLoadAdd32:
  99:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Add, true, 32, NextMBBI);
 100:   case LoongArch::PseudoMaskedAtomicLoadSub32:
 101:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Sub, true, 32, NextMBBI);
 102:   case LoongArch::PseudoAtomicLoadNand32:
 103:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Nand, false, 32,
 104:                              NextMBBI);
 105:   case LoongArch::PseudoAtomicLoadNand64:
 106:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Nand, false, 64,
 107:                              NextMBBI);
 108:   case LoongArch::PseudoMaskedAtomicLoadNand32:
 109:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Nand, true, 32,
 110:                              NextMBBI);
 111:   case LoongArch::PseudoAtomicLoadAdd32:
 112:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Add, false, 32,
```
- **EN**: The range implements or declares functions including `LoongArchExpandAtomicPseudo::expandMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `LoongArchExpandAtomicPseudo::expandMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 113-140 / 第 113-140 行
```cpp
 113:                              NextMBBI);
 114:   case LoongArch::PseudoAtomicLoadSub32:
 115:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Sub, false, 32,
 116:                              NextMBBI);
 117:   case LoongArch::PseudoAtomicLoadAnd32:
 118:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::And, false, 32,
 119:                              NextMBBI);
 120:   case LoongArch::PseudoAtomicLoadOr32:
 121:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Or, false, 32, NextMBBI);
 122:   case LoongArch::PseudoAtomicLoadXor32:
 123:     return expandAtomicBinOp(MBB, MBBI, AtomicRMWInst::Xor, false, 32,
 124:                              NextMBBI);
 125:   case LoongArch::PseudoAtomicLoadUMax32:
 126:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMax, false, 32,
 127:                                 NextMBBI);
 128:   case LoongArch::PseudoAtomicLoadUMin32:
 129:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMin, false, 32,
 130:                                 NextMBBI);
 131:   case LoongArch::PseudoAtomicLoadMax32:
 132:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Max, false, 32,
 133:                                 NextMBBI);
 134:   case LoongArch::PseudoAtomicLoadMin32:
 135:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Min, false, 32,
 136:                                 NextMBBI);
 137:   case LoongArch::PseudoMaskedAtomicLoadUMax32:
 138:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMax, true, 32,
 139:                                 NextMBBI);
 140:   case LoongArch::PseudoMaskedAtomicLoadUMin32:
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 141-168 / 第 141-168 行
```cpp
 141:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::UMin, true, 32,
 142:                                 NextMBBI);
 143:   case LoongArch::PseudoCmpXchg32:
 144:     return expandAtomicCmpXchg(MBB, MBBI, false, 32, NextMBBI);
 145:   case LoongArch::PseudoCmpXchg64:
 146:     return expandAtomicCmpXchg(MBB, MBBI, false, 64, NextMBBI);
 147:   case LoongArch::PseudoCmpXchg128:
 148:   case LoongArch::PseudoCmpXchg128Acquire:
 149:     return expandAtomicCmpXchg128(MBB, MBBI, NextMBBI);
 150:   case LoongArch::PseudoMaskedCmpXchg32:
 151:     return expandAtomicCmpXchg(MBB, MBBI, true, 32, NextMBBI);
 152:   case LoongArch::PseudoMaskedAtomicLoadMax32:
 153:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Max, true, 32,
 154:                                 NextMBBI);
 155:   case LoongArch::PseudoMaskedAtomicLoadMin32:
 156:     return expandAtomicMinMaxOp(MBB, MBBI, AtomicRMWInst::Min, true, 32,
 157:                                 NextMBBI);
 158:   }
 159:   return false;
 160: }
 161: 
 162: static void doAtomicBinOpExpansion(const LoongArchInstrInfo *TII,
 163:                                    MachineInstr &MI, DebugLoc DL,
 164:                                    MachineBasicBlock *ThisMBB,
 165:                                    MachineBasicBlock *LoopMBB,
 166:                                    MachineBasicBlock *DoneMBB,
 167:                                    AtomicRMWInst::BinOp BinOp, int Width) {
 168:   Register DestReg = MI.getOperand(0).getReg();
```
- **EN**: The range implements or declares functions including `doAtomicBinOpExpansion`.
- **CN**: 这一段实现或声明了 `doAtomicBinOpExpansion` 等函数。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   Register ScratchReg = MI.getOperand(1).getReg();
 170:   Register AddrReg = MI.getOperand(2).getReg();
 171:   Register IncrReg = MI.getOperand(3).getReg();
 172: 
 173:   // .loop:
 174:   //   ll.[w|d] dest, (addr)
 175:   //   binop scratch, dest, val
 176:   //   sc.[w|d] scratch, scratch, (addr)
 177:   //   beqz scratch, loop
 178:   BuildMI(LoopMBB, DL,
 179:           TII->get(Width == 32 ? LoongArch::LL_W : LoongArch::LL_D), DestReg)
 180:       .addReg(AddrReg)
 181:       .addImm(0);
 182:   switch (BinOp) {
 183:   default:
 184:     llvm_unreachable("Unexpected AtomicRMW BinOp");
 185:   case AtomicRMWInst::Xchg:
 186:     BuildMI(LoopMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 187:         .addReg(IncrReg)
 188:         .addReg(LoongArch::R0);
 189:     break;
 190:   case AtomicRMWInst::Nand:
 191:     BuildMI(LoopMBB, DL, TII->get(LoongArch::AND), ScratchReg)
 192:         .addReg(DestReg)
 193:         .addReg(IncrReg);
 194:     BuildMI(LoopMBB, DL, TII->get(LoongArch::NOR), ScratchReg)
 195:         .addReg(ScratchReg)
 196:         .addReg(LoongArch::R0);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 197-224 / 第 197-224 行
```cpp
 197:     break;
 198:   case AtomicRMWInst::Add:
 199:     BuildMI(LoopMBB, DL, TII->get(LoongArch::ADD_W), ScratchReg)
 200:         .addReg(DestReg)
 201:         .addReg(IncrReg);
 202:     break;
 203:   case AtomicRMWInst::Sub:
 204:     BuildMI(LoopMBB, DL, TII->get(LoongArch::SUB_W), ScratchReg)
 205:         .addReg(DestReg)
 206:         .addReg(IncrReg);
 207:     break;
 208:   case AtomicRMWInst::And:
 209:     BuildMI(LoopMBB, DL, TII->get(LoongArch::AND), ScratchReg)
 210:         .addReg(DestReg)
 211:         .addReg(IncrReg);
 212:     break;
 213:   case AtomicRMWInst::Or:
 214:     BuildMI(LoopMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 215:         .addReg(DestReg)
 216:         .addReg(IncrReg);
 217:     break;
 218:   case AtomicRMWInst::Xor:
 219:     BuildMI(LoopMBB, DL, TII->get(LoongArch::XOR), ScratchReg)
 220:         .addReg(DestReg)
 221:         .addReg(IncrReg);
 222:     break;
 223:   }
 224:   BuildMI(LoopMBB, DL,
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 225-252 / 第 225-252 行
```cpp
 225:           TII->get(Width == 32 ? LoongArch::SC_W : LoongArch::SC_D), ScratchReg)
 226:       .addReg(ScratchReg)
 227:       .addReg(AddrReg)
 228:       .addImm(0);
 229:   BuildMI(LoopMBB, DL, TII->get(LoongArch::BEQ))
 230:       .addReg(ScratchReg)
 231:       .addReg(LoongArch::R0)
 232:       .addMBB(LoopMBB);
 233: }
 234: 
 235: static void insertMaskedMerge(const LoongArchInstrInfo *TII, DebugLoc DL,
 236:                               MachineBasicBlock *MBB, Register DestReg,
 237:                               Register OldValReg, Register NewValReg,
 238:                               Register MaskReg, Register ScratchReg) {
 239:   assert(OldValReg != ScratchReg && "OldValReg and ScratchReg must be unique");
 240:   assert(OldValReg != MaskReg && "OldValReg and MaskReg must be unique");
 241:   assert(ScratchReg != MaskReg && "ScratchReg and MaskReg must be unique");
 242: 
 243:   // res = oldval ^ ((oldval ^ newval) & masktargetdata);
 244:   BuildMI(MBB, DL, TII->get(LoongArch::XOR), ScratchReg)
 245:       .addReg(OldValReg)
 246:       .addReg(NewValReg);
 247:   BuildMI(MBB, DL, TII->get(LoongArch::AND), ScratchReg)
 248:       .addReg(ScratchReg)
 249:       .addReg(MaskReg);
 250:   BuildMI(MBB, DL, TII->get(LoongArch::XOR), DestReg)
 251:       .addReg(OldValReg)
 252:       .addReg(ScratchReg);
```
- **EN**: The range implements or declares functions including `BuildMI`, `insertMaskedMerge`, `BuildMI`, `BuildMI`, `BuildMI`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `insertMaskedMerge`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 253-280 / 第 253-280 行
```cpp
 253: }
 254: 
 255: static void doMaskedAtomicBinOpExpansion(
 256:     const LoongArchInstrInfo *TII, MachineInstr &MI, DebugLoc DL,
 257:     MachineBasicBlock *ThisMBB, MachineBasicBlock *LoopMBB,
 258:     MachineBasicBlock *DoneMBB, AtomicRMWInst::BinOp BinOp, int Width) {
 259:   assert(Width == 32 && "Should never need to expand masked 64-bit operations");
 260:   Register DestReg = MI.getOperand(0).getReg();
 261:   Register ScratchReg = MI.getOperand(1).getReg();
 262:   Register AddrReg = MI.getOperand(2).getReg();
 263:   Register IncrReg = MI.getOperand(3).getReg();
 264:   Register MaskReg = MI.getOperand(4).getReg();
 265: 
 266:   // .loop:
 267:   //   ll.w destreg, (alignedaddr)
 268:   //   binop scratch, destreg, incr
 269:   //   xor scratch, destreg, scratch
 270:   //   and scratch, scratch, masktargetdata
 271:   //   xor scratch, destreg, scratch
 272:   //   sc.w scratch, scratch, (alignedaddr)
 273:   //   beqz scratch, loop
 274:   BuildMI(LoopMBB, DL, TII->get(LoongArch::LL_W), DestReg)
 275:       .addReg(AddrReg)
 276:       .addImm(0);
 277:   switch (BinOp) {
 278:   default:
 279:     llvm_unreachable("Unexpected AtomicRMW BinOp");
 280:   case AtomicRMWInst::Xchg:
```
- **EN**: The range implements or declares functions including `doMaskedAtomicBinOpExpansion`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `doMaskedAtomicBinOpExpansion`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-308 / 第 281-308 行
```cpp
 281:     BuildMI(LoopMBB, DL, TII->get(LoongArch::ADDI_W), ScratchReg)
 282:         .addReg(IncrReg)
 283:         .addImm(0);
 284:     break;
 285:   case AtomicRMWInst::Add:
 286:     BuildMI(LoopMBB, DL, TII->get(LoongArch::ADD_W), ScratchReg)
 287:         .addReg(DestReg)
 288:         .addReg(IncrReg);
 289:     break;
 290:   case AtomicRMWInst::Sub:
 291:     BuildMI(LoopMBB, DL, TII->get(LoongArch::SUB_W), ScratchReg)
 292:         .addReg(DestReg)
 293:         .addReg(IncrReg);
 294:     break;
 295:   case AtomicRMWInst::Nand:
 296:     BuildMI(LoopMBB, DL, TII->get(LoongArch::AND), ScratchReg)
 297:         .addReg(DestReg)
 298:         .addReg(IncrReg);
 299:     BuildMI(LoopMBB, DL, TII->get(LoongArch::NOR), ScratchReg)
 300:         .addReg(ScratchReg)
 301:         .addReg(LoongArch::R0);
 302:     // TODO: support other AtomicRMWInst.
 303:   }
 304: 
 305:   insertMaskedMerge(TII, DL, LoopMBB, ScratchReg, DestReg, ScratchReg, MaskReg,
 306:                     ScratchReg);
 307: 
 308:   BuildMI(LoopMBB, DL, TII->get(LoongArch::SC_W), ScratchReg)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 309-336 / 第 309-336 行
```cpp
 309:       .addReg(ScratchReg)
 310:       .addReg(AddrReg)
 311:       .addImm(0);
 312:   BuildMI(LoopMBB, DL, TII->get(LoongArch::BEQ))
 313:       .addReg(ScratchReg)
 314:       .addReg(LoongArch::R0)
 315:       .addMBB(LoopMBB);
 316: }
 317: 
 318: bool LoongArchExpandAtomicPseudo::expandAtomicBinOp(
 319:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 320:     AtomicRMWInst::BinOp BinOp, bool IsMasked, int Width,
 321:     MachineBasicBlock::iterator &NextMBBI) {
 322:   MachineInstr &MI = *MBBI;
 323:   DebugLoc DL = MI.getDebugLoc();
 324: 
 325:   MachineFunction *MF = MBB.getParent();
 326:   auto LoopMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 327:   auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 328: 
 329:   // Insert new MBBs.
 330:   MF->insert(++MBB.getIterator(), LoopMBB);
 331:   MF->insert(++LoopMBB->getIterator(), DoneMBB);
 332: 
 333:   // Set up successors and transfer remaining instructions to DoneMBB.
 334:   LoopMBB->addSuccessor(LoopMBB);
 335:   LoopMBB->addSuccessor(DoneMBB);
 336:   DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
```
- **EN**: The range implements or declares functions including `BuildMI`, `LoongArchExpandAtomicPseudo::expandAtomicBinOp`.
- **CN**: 这一段实现或声明了 `BuildMI`, `LoongArchExpandAtomicPseudo::expandAtomicBinOp` 等函数。

### Lines 337-364 / 第 337-364 行
```cpp
 337:   DoneMBB->transferSuccessors(&MBB);
 338:   MBB.addSuccessor(LoopMBB);
 339: 
 340:   if (IsMasked)
 341:     doMaskedAtomicBinOpExpansion(TII, MI, DL, &MBB, LoopMBB, DoneMBB, BinOp,
 342:                                  Width);
 343:   else
 344:     doAtomicBinOpExpansion(TII, MI, DL, &MBB, LoopMBB, DoneMBB, BinOp, Width);
 345: 
 346:   NextMBBI = MBB.end();
 347:   MI.eraseFromParent();
 348: 
 349:   LivePhysRegs LiveRegs;
 350:   computeAndAddLiveIns(LiveRegs, *LoopMBB);
 351:   computeAndAddLiveIns(LiveRegs, *DoneMBB);
 352: 
 353:   return true;
 354: }
 355: 
 356: static void insertSext(const LoongArchInstrInfo *TII, DebugLoc DL,
 357:                        MachineBasicBlock *MBB, Register ValReg,
 358:                        Register ShamtReg) {
 359:   BuildMI(MBB, DL, TII->get(LoongArch::SLL_W), ValReg)
 360:       .addReg(ValReg)
 361:       .addReg(ShamtReg);
 362:   BuildMI(MBB, DL, TII->get(LoongArch::SRA_W), ValReg)
 363:       .addReg(ValReg)
 364:       .addReg(ShamtReg);
```
- **EN**: The range implements or declares functions including `insertSext`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `insertSext`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365: }
 366: 
 367: bool LoongArchExpandAtomicPseudo::expandAtomicMinMaxOp(
 368:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 369:     AtomicRMWInst::BinOp BinOp, bool IsMasked, int Width,
 370:     MachineBasicBlock::iterator &NextMBBI) {
 371:   assert(Width == 32 && "Should never need to expand masked 64-bit operations");
 372: 
 373:   MachineInstr &MI = *MBBI;
 374:   DebugLoc DL = MI.getDebugLoc();
 375:   MachineFunction *MF = MBB.getParent();
 376:   auto LoopHeadMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 377:   auto LoopIfBodyMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 378:   auto LoopTailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 379:   auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 380: 
 381:   // Insert new MBBs.
 382:   MF->insert(++MBB.getIterator(), LoopHeadMBB);
 383:   MF->insert(++LoopHeadMBB->getIterator(), LoopIfBodyMBB);
 384:   MF->insert(++LoopIfBodyMBB->getIterator(), LoopTailMBB);
 385:   MF->insert(++LoopTailMBB->getIterator(), DoneMBB);
 386: 
 387:   // Set up successors and transfer remaining instructions to DoneMBB.
 388:   LoopHeadMBB->addSuccessor(LoopIfBodyMBB);
 389:   LoopHeadMBB->addSuccessor(LoopTailMBB);
 390:   LoopIfBodyMBB->addSuccessor(LoopTailMBB);
 391:   LoopTailMBB->addSuccessor(LoopHeadMBB);
 392:   LoopTailMBB->addSuccessor(DoneMBB);
```
- **EN**: The range implements or declares functions including `LoongArchExpandAtomicPseudo::expandAtomicMinMaxOp`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchExpandAtomicPseudo::expandAtomicMinMaxOp` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
 394:   DoneMBB->transferSuccessors(&MBB);
 395:   MBB.addSuccessor(LoopHeadMBB);
 396: 
 397:   Register DestReg = MI.getOperand(0).getReg();
 398:   Register ScratchReg = MI.getOperand(1).getReg();
 399:   Register AddrReg = MI.getOperand(IsMasked ? 3 : 2).getReg();
 400:   Register IncrReg = MI.getOperand(IsMasked ? 4 : 3).getReg();
 401:   Register CmprReg = DestReg;
 402: 
 403:   //
 404:   // .loophead:
 405:   //   ll.w destreg, (alignedaddr)
 406:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::LL_W), DestReg)
 407:       .addReg(AddrReg)
 408:       .addImm(0);
 409:   //   and cmpr, destreg, mask
 410:   if (IsMasked) {
 411:     Register MaskReg = MI.getOperand(5).getReg();
 412:     CmprReg = MI.getOperand(2).getReg();
 413:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::AND), CmprReg)
 414:         .addReg(DestReg)
 415:         .addReg(MaskReg);
 416:   }
 417:   //   move scratch, destreg
 418:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 419:       .addReg(DestReg)
 420:       .addReg(LoongArch::R0);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421: 
 422:   switch (BinOp) {
 423:   default:
 424:     llvm_unreachable("Unexpected AtomicRMW BinOp");
 425:   // bgeu cmpr, incr, .looptail
 426:   case AtomicRMWInst::UMax:
 427:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BGEU))
 428:         .addReg(CmprReg)
 429:         .addReg(IncrReg)
 430:         .addMBB(LoopTailMBB);
 431:     break;
 432:   // bgeu incr, cmpr, .looptail
 433:   case AtomicRMWInst::UMin:
 434:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BGEU))
 435:         .addReg(IncrReg)
 436:         .addReg(CmprReg)
 437:         .addMBB(LoopTailMBB);
 438:     break;
 439:   case AtomicRMWInst::Max:
 440:     if (IsMasked)
 441:       insertSext(TII, DL, LoopHeadMBB, CmprReg, MI.getOperand(6).getReg());
 442:     // bge cmpr, incr, .looptail
 443:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BGE))
 444:         .addReg(CmprReg)
 445:         .addReg(IncrReg)
 446:         .addMBB(LoopTailMBB);
 447:     break;
 448:   case AtomicRMWInst::Min:
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 449-476 / 第 449-476 行
```cpp
 449:     if (IsMasked)
 450:       insertSext(TII, DL, LoopHeadMBB, CmprReg, MI.getOperand(6).getReg());
 451:     // bge incr, cmpr, .looptail
 452:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BGE))
 453:         .addReg(IncrReg)
 454:         .addReg(CmprReg)
 455:         .addMBB(LoopTailMBB);
 456:     break;
 457:     // TODO: support other AtomicRMWInst.
 458:   }
 459: 
 460:   // .loopifbody:
 461:   if (IsMasked) {
 462:     Register MaskReg = MI.getOperand(5).getReg();
 463:     // xor scratch, destreg, incr
 464:     // and scratch, scratch, mask
 465:     // xor scratch, destreg, scratch
 466:     insertMaskedMerge(TII, DL, LoopIfBodyMBB, ScratchReg, DestReg, IncrReg,
 467:                       MaskReg, ScratchReg);
 468:   } else {
 469:     // move scratch, incr
 470:     BuildMI(LoopIfBodyMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 471:         .addReg(IncrReg)
 472:         .addReg(LoongArch::R0);
 473:   }
 474: 
 475:   // .looptail:
 476:   //   sc.w scratch, scratch, (addr)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 477-504 / 第 477-504 行
```cpp
 477:   //   beqz scratch, loop
 478:   BuildMI(LoopTailMBB, DL, TII->get(LoongArch::SC_W), ScratchReg)
 479:       .addReg(ScratchReg)
 480:       .addReg(AddrReg)
 481:       .addImm(0);
 482:   BuildMI(LoopTailMBB, DL, TII->get(LoongArch::BEQ))
 483:       .addReg(ScratchReg)
 484:       .addReg(LoongArch::R0)
 485:       .addMBB(LoopHeadMBB);
 486: 
 487:   NextMBBI = MBB.end();
 488:   MI.eraseFromParent();
 489: 
 490:   LivePhysRegs LiveRegs;
 491:   computeAndAddLiveIns(LiveRegs, *LoopHeadMBB);
 492:   computeAndAddLiveIns(LiveRegs, *LoopIfBodyMBB);
 493:   computeAndAddLiveIns(LiveRegs, *LoopTailMBB);
 494:   computeAndAddLiveIns(LiveRegs, *DoneMBB);
 495: 
 496:   return true;
 497: }
 498: 
 499: bool LoongArchExpandAtomicPseudo::expandAtomicCmpXchg(
 500:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, bool IsMasked,
 501:     int Width, MachineBasicBlock::iterator &NextMBBI) {
 502:   MachineInstr &MI = *MBBI;
 503:   DebugLoc DL = MI.getDebugLoc();
 504:   MachineFunction *MF = MBB.getParent();
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `LoongArchExpandAtomicPseudo::expandAtomicCmpXchg`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `LoongArchExpandAtomicPseudo::expandAtomicCmpXchg` 等函数。

### Lines 505-532 / 第 505-532 行
```cpp
 505:   auto LoopHeadMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 506:   auto LoopTailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 507:   auto TailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 508:   auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 509: 
 510:   // Insert new MBBs.
 511:   MF->insert(++MBB.getIterator(), LoopHeadMBB);
 512:   MF->insert(++LoopHeadMBB->getIterator(), LoopTailMBB);
 513:   MF->insert(++LoopTailMBB->getIterator(), TailMBB);
 514:   MF->insert(++TailMBB->getIterator(), DoneMBB);
 515: 
 516:   // Set up successors and transfer remaining instructions to DoneMBB.
 517:   LoopHeadMBB->addSuccessor(LoopTailMBB);
 518:   LoopHeadMBB->addSuccessor(TailMBB);
 519:   LoopTailMBB->addSuccessor(DoneMBB);
 520:   LoopTailMBB->addSuccessor(LoopHeadMBB);
 521:   TailMBB->addSuccessor(DoneMBB);
 522:   DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
 523:   DoneMBB->transferSuccessors(&MBB);
 524:   MBB.addSuccessor(LoopHeadMBB);
 525: 
 526:   Register DestReg = MI.getOperand(0).getReg();
 527:   Register ScratchReg = MI.getOperand(1).getReg();
 528:   Register AddrReg = MI.getOperand(2).getReg();
 529:   Register CmpValReg = MI.getOperand(3).getReg();
 530:   Register NewValReg = MI.getOperand(4).getReg();
 531: 
 532:   if (!IsMasked) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:     // .loophead:
 534:     //   ll.[w|d] dest, (addr)
 535:     //   bne dest, cmpval, tail
 536:     BuildMI(LoopHeadMBB, DL,
 537:             TII->get(Width == 32 ? LoongArch::LL_W : LoongArch::LL_D), DestReg)
 538:         .addReg(AddrReg)
 539:         .addImm(0);
 540:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BNE))
 541:         .addReg(DestReg)
 542:         .addReg(CmpValReg)
 543:         .addMBB(TailMBB);
 544:     // .looptail:
 545:     //   move scratch, newval
 546:     //   sc.[w|d] scratch, scratch, (addr)
 547:     //   beqz scratch, loophead
 548:     //   b done
 549:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 550:         .addReg(NewValReg)
 551:         .addReg(LoongArch::R0);
 552:     BuildMI(LoopTailMBB, DL,
 553:             TII->get(Width == 32 ? LoongArch::SC_W : LoongArch::SC_D),
 554:             ScratchReg)
 555:         .addReg(ScratchReg)
 556:         .addReg(AddrReg)
 557:         .addImm(0);
 558:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::BEQ))
 559:         .addReg(ScratchReg)
 560:         .addReg(LoongArch::R0)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 561-588 / 第 561-588 行
```cpp
 561:         .addMBB(LoopHeadMBB);
 562:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::B)).addMBB(DoneMBB);
 563:   } else {
 564:     // .loophead:
 565:     //   ll.[w|d] dest, (addr)
 566:     //   and scratch, dest, mask
 567:     //   bne scratch, cmpval, tail
 568:     Register MaskReg = MI.getOperand(5).getReg();
 569:     BuildMI(LoopHeadMBB, DL,
 570:             TII->get(Width == 32 ? LoongArch::LL_W : LoongArch::LL_D), DestReg)
 571:         .addReg(AddrReg)
 572:         .addImm(0);
 573:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::AND), ScratchReg)
 574:         .addReg(DestReg)
 575:         .addReg(MaskReg);
 576:     BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BNE))
 577:         .addReg(ScratchReg)
 578:         .addReg(CmpValReg)
 579:         .addMBB(TailMBB);
 580: 
 581:     // .looptail:
 582:     //   andn scratch, dest, mask
 583:     //   or scratch, scratch, newval
 584:     //   sc.[w|d] scratch, scratch, (addr)
 585:     //   beqz scratch, loophead
 586:     //   b done
 587:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::ANDN), ScratchReg)
 588:         .addReg(DestReg)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 589-616 / 第 589-616 行
```cpp
 589:         .addReg(MaskReg);
 590:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 591:         .addReg(ScratchReg)
 592:         .addReg(NewValReg);
 593:     BuildMI(LoopTailMBB, DL,
 594:             TII->get(Width == 32 ? LoongArch::SC_W : LoongArch::SC_D),
 595:             ScratchReg)
 596:         .addReg(ScratchReg)
 597:         .addReg(AddrReg)
 598:         .addImm(0);
 599:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::BEQ))
 600:         .addReg(ScratchReg)
 601:         .addReg(LoongArch::R0)
 602:         .addMBB(LoopHeadMBB);
 603:     BuildMI(LoopTailMBB, DL, TII->get(LoongArch::B)).addMBB(DoneMBB);
 604:   }
 605: 
 606:   AtomicOrdering FailureOrdering =
 607:       static_cast<AtomicOrdering>(MI.getOperand(IsMasked ? 6 : 5).getImm());
 608:   int hint;
 609: 
 610:   switch (FailureOrdering) {
 611:   case AtomicOrdering::Acquire:
 612:   case AtomicOrdering::AcquireRelease:
 613:   case AtomicOrdering::SequentiallyConsistent:
 614:     // acquire
 615:     hint = 0b10100;
 616:     break;
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 617-644 / 第 617-644 行
```cpp
 617:   default:
 618:     hint = 0x700;
 619:   }
 620: 
 621:   // .tail:
 622:   //   dbar 0x700 | acquire
 623: 
 624:   if (!(hint == 0x700 && MF->getSubtarget<LoongArchSubtarget>().hasLD_SEQ_SA()))
 625:     BuildMI(TailMBB, DL, TII->get(LoongArch::DBAR)).addImm(hint);
 626: 
 627:   NextMBBI = MBB.end();
 628:   MI.eraseFromParent();
 629: 
 630:   LivePhysRegs LiveRegs;
 631:   computeAndAddLiveIns(LiveRegs, *LoopHeadMBB);
 632:   computeAndAddLiveIns(LiveRegs, *LoopTailMBB);
 633:   computeAndAddLiveIns(LiveRegs, *TailMBB);
 634:   computeAndAddLiveIns(LiveRegs, *DoneMBB);
 635: 
 636:   return true;
 637: }
 638: 
 639: bool LoongArchExpandAtomicPseudo::expandAtomicCmpXchg128(
 640:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 641:     MachineBasicBlock::iterator &NextMBBI) {
 642:   MachineInstr &MI = *MBBI;
 643:   DebugLoc DL = MI.getDebugLoc();
 644:   MachineFunction *MF = MBB.getParent();
```
- **EN**: The range implements or declares functions including `LoongArchExpandAtomicPseudo::expandAtomicCmpXchg128`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchExpandAtomicPseudo::expandAtomicCmpXchg128` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```cpp
 645:   auto LoopHeadMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 646:   auto LoopTailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 647:   auto TailMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 648:   auto DoneMBB = MF->CreateMachineBasicBlock(MBB.getBasicBlock());
 649: 
 650:   // Insert new MBBs
 651:   MF->insert(++MBB.getIterator(), LoopHeadMBB);
 652:   MF->insert(++LoopHeadMBB->getIterator(), LoopTailMBB);
 653:   MF->insert(++LoopTailMBB->getIterator(), TailMBB);
 654:   MF->insert(++TailMBB->getIterator(), DoneMBB);
 655: 
 656:   // Set up successors and transfer remaining instructions to DoneMBB.
 657:   LoopHeadMBB->addSuccessor(LoopTailMBB);
 658:   LoopHeadMBB->addSuccessor(TailMBB);
 659:   LoopTailMBB->addSuccessor(DoneMBB);
 660:   LoopTailMBB->addSuccessor(LoopHeadMBB);
 661:   TailMBB->addSuccessor(DoneMBB);
 662:   DoneMBB->splice(DoneMBB->end(), &MBB, MI, MBB.end());
 663:   DoneMBB->transferSuccessors(&MBB);
 664:   MBB.addSuccessor(LoopHeadMBB);
 665: 
 666:   Register DestLoReg = MI.getOperand(0).getReg();
 667:   Register DestHiReg = MI.getOperand(1).getReg();
 668:   Register ScratchReg = MI.getOperand(2).getReg();
 669:   Register AddrReg = MI.getOperand(3).getReg();
 670:   Register CmpValLoReg = MI.getOperand(4).getReg();
 671:   Register CmpValHiReg = MI.getOperand(5).getReg();
 672:   Register NewValLoReg = MI.getOperand(6).getReg();
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 673-700 / 第 673-700 行
```cpp
 673:   Register NewValHiReg = MI.getOperand(7).getReg();
 674: 
 675:   // .loophead:
 676:   //   ll.d res_lo, (addr)
 677:   //   dbar acquire
 678:   //   ld.d res_hi, (addr), 8
 679:   //   bne dest_lo, cmpval_lo, tail
 680:   //   bne dest_hi, cmpval_hi, tail
 681:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::LL_D), DestLoReg)
 682:       .addReg(AddrReg)
 683:       .addImm(0);
 684:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::DBAR)).addImm(0b10100);
 685:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::LD_D), DestHiReg)
 686:       .addReg(AddrReg)
 687:       .addImm(8);
 688:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BNE))
 689:       .addReg(DestLoReg)
 690:       .addReg(CmpValLoReg)
 691:       .addMBB(TailMBB);
 692:   BuildMI(LoopHeadMBB, DL, TII->get(LoongArch::BNE))
 693:       .addReg(DestHiReg)
 694:       .addReg(CmpValHiReg)
 695:       .addMBB(TailMBB);
 696:   // .looptail:
 697:   //   move scratch, newval_lo
 698:   //   sc.q scratch, newval_hi, (addr)
 699:   //   beqz scratch, loophead
 700:   //   b done
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 701-728 / 第 701-728 行
```cpp
 701:   BuildMI(LoopTailMBB, DL, TII->get(LoongArch::OR), ScratchReg)
 702:       .addReg(NewValLoReg)
 703:       .addReg(LoongArch::R0);
 704:   BuildMI(LoopTailMBB, DL, TII->get(LoongArch::SC_Q), ScratchReg)
 705:       .addReg(ScratchReg)
 706:       .addReg(NewValHiReg)
 707:       .addReg(AddrReg);
 708:   BuildMI(LoopTailMBB, DL, TII->get(LoongArch::BEQ))
 709:       .addReg(ScratchReg)
 710:       .addReg(LoongArch::R0)
 711:       .addMBB(LoopHeadMBB);
 712:   BuildMI(LoopTailMBB, DL, TII->get(LoongArch::B)).addMBB(DoneMBB);
 713:   int hint;
 714: 
 715:   switch (MI.getOpcode()) {
 716:   case LoongArch::PseudoCmpXchg128Acquire:
 717:     // acquire acqrel seqcst
 718:     hint = 0b10100;
 719:     break;
 720:   case LoongArch::PseudoCmpXchg128:
 721:     hint = 0x700;
 722:     break;
 723:   default:
 724:     llvm_unreachable("Unexpected opcode");
 725:   }
 726: 
 727:   // .tail:
 728:   //   dbar 0x700 | acquire
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 729-755 / 第 729-755 行
```cpp
 729:   if (!(hint == 0x700 && MF->getSubtarget<LoongArchSubtarget>().hasLD_SEQ_SA()))
 730:     BuildMI(TailMBB, DL, TII->get(LoongArch::DBAR)).addImm(hint);
 731: 
 732:   NextMBBI = MBB.end();
 733:   MI.eraseFromParent();
 734: 
 735:   LivePhysRegs LiveRegs;
 736:   computeAndAddLiveIns(LiveRegs, *LoopHeadMBB);
 737:   computeAndAddLiveIns(LiveRegs, *LoopTailMBB);
 738:   computeAndAddLiveIns(LiveRegs, *TailMBB);
 739:   computeAndAddLiveIns(LiveRegs, *DoneMBB);
 740: 
 741:   return true;
 742: }
 743: 
 744: } // end namespace
 745: 
 746: INITIALIZE_PASS(LoongArchExpandAtomicPseudo, "loongarch-expand-atomic-pseudo",
 747:                 LoongArch_EXPAND_ATOMIC_PSEUDO_NAME, false, false)
 748: 
 749: namespace llvm {
 750: 
 751: FunctionPass *createLoongArchExpandAtomicPseudoPass() {
 752:   return new LoongArchExpandAtomicPseudo();
 753: }
 754: 
 755: } // end namespace llvm
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `LoongArch.h`
- `LoongArchInstrInfo.h`
- `LoongArchTargetMachine.h`
- `llvm/CodeGen/LivePhysRegs.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
