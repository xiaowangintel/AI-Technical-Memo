# BPFMIPeephole.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFMIPeephole.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-------------- BPFMIPeephole.cpp - MI Peephole Cleanups  -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass performs peephole optimizations to cleanup ugly code sequences at
  10: // MachineInstruction layer.
  11: //
  12: // Currently, there are two optimizations implemented:
  13: //  - One pre-RA MachineSSA pass to eliminate type promotion sequences, those
  14: //    zero extend 32-bit subregisters to 64-bit registers, if the compiler
  15: //    could prove the subregisters is defined by 32-bit operations in which
  16: //    case the upper half of the underlying 64-bit registers were zeroed
  17: //    implicitly.
  18: //
  19: //  - One post-RA PreEmit pass to do final cleanup on some redundant
  20: //    instructions generated due to bad RA on subregister.
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。

### Lines 21-40

```cpp
  21: //===----------------------------------------------------------------------===//
  22: 
  23: #include "BPF.h"
  24: #include "BPFInstrInfo.h"
  25: #include "BPFTargetMachine.h"
  26: #include "llvm/ADT/Statistic.h"
  27: #include "llvm/ADT/StringExtras.h"
  28: #include "llvm/CodeGen/LivePhysRegs.h"
  29: #include "llvm/CodeGen/MachineFrameInfo.h"
  30: #include "llvm/CodeGen/MachineFunctionPass.h"
  31: #include "llvm/CodeGen/MachineInstrBuilder.h"
  32: #include "llvm/CodeGen/MachineRegisterInfo.h"
  33: #include "llvm/Support/Debug.h"
  34: #include <set>
  35: 
  36: using namespace llvm;
  37: 
  38: #define DEBUG_TYPE "bpf-mi-zext-elim"
  39: 
  40: static cl::opt<int> GotolAbsLowBound("gotol-abs-low-bound", cl::Hidden,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41:   cl::init(INT16_MAX >> 1), cl::desc("Specify gotol lower bound"));
  42: 
  43: STATISTIC(ZExtElemNum, "Number of zero extension shifts eliminated");
  44: 
  45: namespace {
  46: 
  47: struct BPFMIPeephole : public MachineFunctionPass {
  48: 
  49:   static char ID;
  50:   const BPFInstrInfo *TII;
  51:   MachineFunction *MF;
  52:   MachineRegisterInfo *MRI;
  53: 
  54:   BPFMIPeephole() : MachineFunctionPass(ID) {}
  55: 
  56: private:
  57:   // Initialize class variables.
  58:   void initialize(MachineFunction &MFParm);
  59: 
  60:   bool isCopyFrom32Def(MachineInstr *CopyMI);
```

- EN: This chunk introduces interfaces or data structures such as BPFMIPeephole, variables, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 BPFMIPeephole, variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 61-80

```cpp
  61:   bool isInsnFrom32Def(MachineInstr *DefInsn);
  62:   bool isPhiFrom32Def(MachineInstr *MovMI);
  63:   bool isMovFrom32Def(MachineInstr *MovMI);
  64:   bool eliminateZExtSeq();
  65:   bool eliminateZExt();
  66: 
  67:   std::set<MachineInstr *> PhiInsns;
  68: 
  69: public:
  70: 
  71:   // Main entry point for this pass.
  72:   bool runOnMachineFunction(MachineFunction &MF) override {
  73:     if (skipFunction(MF.getFunction()))
  74:       return false;
  75: 
  76:     initialize(MF);
  77: 
  78:     // First try to eliminate (zext, lshift, rshift) and then
  79:     // try to eliminate zext.
  80:     bool ZExtSeqExist, ZExtExist;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 81-100

```cpp
  81:     ZExtSeqExist = eliminateZExtSeq();
  82:     ZExtExist = eliminateZExt();
  83:     return ZExtSeqExist || ZExtExist;
  84:   }
  85: };
  86: 
  87: // Initialize class variables.
  88: void BPFMIPeephole::initialize(MachineFunction &MFParm) {
  89:   MF = &MFParm;
  90:   MRI = &MF->getRegInfo();
  91:   TII = MF->getSubtarget<BPFSubtarget>().getInstrInfo();
  92:   LLVM_DEBUG(dbgs() << "*** BPF MachineSSA ZEXT Elim peephole pass ***\n\n");
  93: }
  94: 
  95: bool BPFMIPeephole::isCopyFrom32Def(MachineInstr *CopyMI)
  96: {
  97:   MachineOperand &opnd = CopyMI->getOperand(1);
  98: 
  99:   if (!opnd.isReg())
 100:     return false;
```

- EN: This chunk introduces interfaces or data structures such as variables, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as initialize, isCopyFrom32Def contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。 initialize, isCopyFrom32Def 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101: 
 102:   // Return false if getting value from a 32bit physical register.
 103:   // Most likely, this physical register is aliased to
 104:   // function call return value or current function parameters.
 105:   Register Reg = opnd.getReg();
 106:   if (!Reg.isVirtual())
 107:     return false;
 108: 
 109:   if (MRI->getRegClass(Reg) == &BPF::GPRRegClass)
 110:     return false;
 111: 
 112:   MachineInstr *DefInsn = MRI->getVRegDef(Reg);
 113:   if (!isInsnFrom32Def(DefInsn))
 114:     return false;
 115: 
 116:   return true;
 117: }
 118: 
 119: bool BPFMIPeephole::isPhiFrom32Def(MachineInstr *PhiMI)
 120: {
```

- EN: Function bodies or method definitions such as isPhiFrom32Def contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isPhiFrom32Def 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:   for (unsigned i = 1, e = PhiMI->getNumOperands(); i < e; i += 2) {
 122:     MachineOperand &opnd = PhiMI->getOperand(i);
 123: 
 124:     if (!opnd.isReg())
 125:       return false;
 126: 
 127:     MachineInstr *PhiDef = MRI->getVRegDef(opnd.getReg());
 128:     if (!PhiDef)
 129:       return false;
 130:     if (PhiDef->isPHI()) {
 131:       if (!PhiInsns.insert(PhiDef).second)
 132:         return false;
 133:       if (!isPhiFrom32Def(PhiDef))
 134:         return false;
 135:     }
 136:     if (PhiDef->getOpcode() == BPF::COPY && !isCopyFrom32Def(PhiDef))
 137:       return false;
 138:   }
 139: 
 140:   return true;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141: }
 142: 
 143: // The \p DefInsn instruction defines a virtual register.
 144: bool BPFMIPeephole::isInsnFrom32Def(MachineInstr *DefInsn)
 145: {
 146:   if (!DefInsn)
 147:     return false;
 148: 
 149:   if (DefInsn->isPHI()) {
 150:     if (!PhiInsns.insert(DefInsn).second)
 151:       return false;
 152:     if (!isPhiFrom32Def(DefInsn))
 153:       return false;
 154:   } else if (DefInsn->getOpcode() == BPF::COPY) {
 155:     if (!isCopyFrom32Def(DefInsn))
 156:       return false;
 157:   }
 158: 
 159:   return true;
 160: }
```

- EN: Function bodies or method definitions such as isInsnFrom32Def contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isInsnFrom32Def 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161: 
 162: bool BPFMIPeephole::isMovFrom32Def(MachineInstr *MovMI)
 163: {
 164:   MachineInstr *DefInsn = MRI->getVRegDef(MovMI->getOperand(1).getReg());
 165: 
 166:   LLVM_DEBUG(dbgs() << "  Def of Mov Src:");
 167:   LLVM_DEBUG(DefInsn->dump());
 168: 
 169:   PhiInsns.clear();
 170:   if (!isInsnFrom32Def(DefInsn))
 171:     return false;
 172: 
 173:   LLVM_DEBUG(dbgs() << "  One ZExt elim sequence identified.\n");
 174: 
 175:   return true;
 176: }
 177: 
 178: bool BPFMIPeephole::eliminateZExtSeq() {
 179:   MachineInstr* ToErase = nullptr;
 180:   bool Eliminated = false;
```

- EN: Function bodies or method definitions such as isMovFrom32Def, eliminateZExtSeq contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isMovFrom32Def, eliminateZExtSeq 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181: 
 182:   for (MachineBasicBlock &MBB : *MF) {
 183:     for (MachineInstr &MI : MBB) {
 184:       // If the previous instruction was marked for elimination, remove it now.
 185:       if (ToErase) {
 186:         ToErase->eraseFromParent();
 187:         ToErase = nullptr;
 188:       }
 189: 
 190:       // Eliminate the 32-bit to 64-bit zero extension sequence when possible.
 191:       //
 192:       //   MOV_32_64 rB, wA
 193:       //   SLL_ri    rB, rB, 32
 194:       //   SRL_ri    rB, rB, 32
 195:       if (MI.getOpcode() == BPF::SRL_ri &&
 196:           MI.getOperand(2).getImm() == 32) {
 197:         Register DstReg = MI.getOperand(0).getReg();
 198:         Register ShfReg = MI.getOperand(1).getReg();
 199:         MachineInstr *SllMI = MRI->getVRegDef(ShfReg);
 200: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201:         LLVM_DEBUG(dbgs() << "Starting SRL found:");
 202:         LLVM_DEBUG(MI.dump());
 203: 
 204:         if (!SllMI ||
 205:             SllMI->isPHI() ||
 206:             SllMI->getOpcode() != BPF::SLL_ri ||
 207:             SllMI->getOperand(2).getImm() != 32)
 208:           continue;
 209: 
 210:         LLVM_DEBUG(dbgs() << "  SLL found:");
 211:         LLVM_DEBUG(SllMI->dump());
 212: 
 213:         MachineInstr *MovMI = MRI->getVRegDef(SllMI->getOperand(1).getReg());
 214:         if (!MovMI ||
 215:             MovMI->isPHI() ||
 216:             MovMI->getOpcode() != BPF::MOV_32_64)
 217:           continue;
 218: 
 219:         LLVM_DEBUG(dbgs() << "  Type cast Mov found:");
 220:         LLVM_DEBUG(MovMI->dump());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221: 
 222:         Register SubReg = MovMI->getOperand(1).getReg();
 223:         if (!isMovFrom32Def(MovMI)) {
 224:           LLVM_DEBUG(dbgs()
 225:                      << "  One ZExt elim sequence failed qualifying elim.\n");
 226:           continue;
 227:         }
 228: 
 229:         BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(BPF::SUBREG_TO_REG), DstReg)
 230:             .addReg(SubReg)
 231:             .addImm(BPF::sub_32);
 232: 
 233:         SllMI->eraseFromParent();
 234:         MovMI->eraseFromParent();
 235:         // MI is the right shift, we can't erase it in it's own iteration.
 236:         // Mark it to ToErase, and erase in the next iteration.
 237:         ToErase = &MI;
 238:         ZExtElemNum++;
 239:         Eliminated = true;
 240:       }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:     }
 242:   }
 243: 
 244:   return Eliminated;
 245: }
 246: 
 247: bool BPFMIPeephole::eliminateZExt() {
 248:   MachineInstr* ToErase = nullptr;
 249:   bool Eliminated = false;
 250: 
 251:   for (MachineBasicBlock &MBB : *MF) {
 252:     for (MachineInstr &MI : MBB) {
 253:       // If the previous instruction was marked for elimination, remove it now.
 254:       if (ToErase) {
 255:         ToErase->eraseFromParent();
 256:         ToErase = nullptr;
 257:       }
 258: 
 259:       if (MI.getOpcode() != BPF::MOV_32_64)
 260:         continue;
```

- EN: Function bodies or method definitions such as eliminateZExt contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: eliminateZExt 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261: 
 262:       // Eliminate MOV_32_64 if possible.
 263:       //   MOV_32_64 rA, wB
 264:       //
 265:       // If wB has been zero extended, replace it with a SUBREG_TO_REG.
 266:       // This is to workaround BPF programs where pkt->{data, data_end}
 267:       // is encoded as u32, but actually the verifier populates them
 268:       // as 64bit pointer. The MOV_32_64 will zero out the top 32 bits.
 269:       LLVM_DEBUG(dbgs() << "Candidate MOV_32_64 instruction:");
 270:       LLVM_DEBUG(MI.dump());
 271: 
 272:       if (!isMovFrom32Def(&MI))
 273:         continue;
 274: 
 275:       LLVM_DEBUG(dbgs() << "Removing the MOV_32_64 instruction\n");
 276: 
 277:       Register dst = MI.getOperand(0).getReg();
 278:       Register src = MI.getOperand(1).getReg();
 279: 
 280:       // Build a SUBREG_TO_REG instruction.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:       BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(BPF::SUBREG_TO_REG), dst)
 282:           .addReg(src)
 283:           .addImm(BPF::sub_32);
 284: 
 285:       ToErase = &MI;
 286:       Eliminated = true;
 287:     }
 288:   }
 289: 
 290:   return Eliminated;
 291: }
 292: 
 293: } // end default namespace
 294: 
 295: INITIALIZE_PASS(BPFMIPeephole, DEBUG_TYPE,
 296:                 "BPF MachineSSA Peephole Optimization For ZEXT Eliminate",
 297:                 false, false)
 298: 
 299: char BPFMIPeephole::ID = 0;
 300: FunctionPass* llvm::createBPFMIPeepholePass() { return new BPFMIPeephole(); }
```

- EN: Function bodies or method definitions such as createBPFMIPeepholePass contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: createBPFMIPeepholePass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 301-320

```cpp
 301: 
 302: STATISTIC(RedundantMovElemNum, "Number of redundant moves eliminated");
 303: 
 304: namespace {
 305: 
 306: struct BPFMIPreEmitPeephole : public MachineFunctionPass {
 307: 
 308:   static char ID;
 309:   MachineFunction *MF;
 310:   const TargetRegisterInfo *TRI;
 311:   const BPFInstrInfo *TII;
 312:   bool SupportGotol;
 313: 
 314:   BPFMIPreEmitPeephole() : MachineFunctionPass(ID) {}
 315: 
 316: private:
 317:   // Initialize class variables.
 318:   void initialize(MachineFunction &MFParm);
 319: 
 320:   bool in16BitRange(int Num);
```

- EN: This chunk introduces interfaces or data structures such as BPFMIPreEmitPeephole, variables, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 BPFMIPreEmitPeephole, variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 321-340

```cpp
 321:   bool eliminateRedundantMov();
 322:   bool adjustBranch();
 323:   bool insertMissingCallerSavedSpills();
 324:   bool removeMayGotoZero();
 325:   bool addExitAfterUnreachable();
 326:   bool expandStackArgPseudos();
 327: 
 328: public:
 329: 
 330:   // Main entry point for this pass.
 331:   bool runOnMachineFunction(MachineFunction &MF) override {
 332:     if (skipFunction(MF.getFunction()))
 333:       return false;
 334: 
 335:     initialize(MF);
 336: 
 337:     bool Changed;
 338:     Changed = eliminateRedundantMov();
 339:     if (SupportGotol)
 340:       Changed = adjustBranch() || Changed;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:     Changed |= insertMissingCallerSavedSpills();
 342:     Changed |= removeMayGotoZero();
 343:     Changed |= addExitAfterUnreachable();
 344:     Changed |= expandStackArgPseudos();
 345:     return Changed;
 346:   }
 347: };
 348: 
 349: // Initialize class variables.
 350: void BPFMIPreEmitPeephole::initialize(MachineFunction &MFParm) {
 351:   MF = &MFParm;
 352:   TII = MF->getSubtarget<BPFSubtarget>().getInstrInfo();
 353:   TRI = MF->getSubtarget<BPFSubtarget>().getRegisterInfo();
 354:   SupportGotol = MF->getSubtarget<BPFSubtarget>().hasGotol();
 355:   LLVM_DEBUG(dbgs() << "*** BPF PreEmit peephole pass ***\n\n");
 356: }
 357: 
 358: bool BPFMIPreEmitPeephole::eliminateRedundantMov() {
 359:   MachineInstr* ToErase = nullptr;
 360:   bool Eliminated = false;
```

- EN: This chunk introduces interfaces or data structures such as variables, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as initialize, eliminateRedundantMov contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。 initialize, eliminateRedundantMov 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 361-380

```cpp
 361: 
 362:   for (MachineBasicBlock &MBB : *MF) {
 363:     for (MachineInstr &MI : MBB) {
 364:       // If the previous instruction was marked for elimination, remove it now.
 365:       if (ToErase) {
 366:         LLVM_DEBUG(dbgs() << "  Redundant Mov Eliminated:");
 367:         LLVM_DEBUG(ToErase->dump());
 368:         ToErase->eraseFromParent();
 369:         ToErase = nullptr;
 370:       }
 371: 
 372:       // Eliminate identical move:
 373:       //
 374:       //   MOV rA, rA
 375:       //
 376:       // Note that we cannot remove
 377:       //   MOV_32_64  rA, wA
 378:       //   MOV_rr_32  wA, wA
 379:       // as these two instructions having side effects, zeroing out
 380:       // top 32 bits of rA.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 381-400

```cpp
 381:       unsigned Opcode = MI.getOpcode();
 382:       if (Opcode == BPF::MOV_rr) {
 383:         Register dst = MI.getOperand(0).getReg();
 384:         Register src = MI.getOperand(1).getReg();
 385: 
 386:         if (dst != src)
 387:           continue;
 388: 
 389:         ToErase = &MI;
 390:         RedundantMovElemNum++;
 391:         Eliminated = true;
 392:       }
 393:     }
 394:   }
 395: 
 396:   return Eliminated;
 397: }
 398: 
 399: bool BPFMIPreEmitPeephole::in16BitRange(int Num) {
 400:   // Well, the cut-off is not precisely at 16bit range since
```

- EN: Function bodies or method definitions such as in16BitRange contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: in16BitRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 401-420

```cpp
 401:   // new codes are added during the transformation. So let us
 402:   // a little bit conservative.
 403:   return Num >= -GotolAbsLowBound && Num <= GotolAbsLowBound;
 404: }
 405: 
 406: // Before cpu=v4, only 16bit branch target offset (-0x8000 to 0x7fff)
 407: // is supported for both unconditional (JMP) and condition (JEQ, JSGT,
 408: // etc.) branches. In certain cases, e.g., full unrolling, the branch
 409: // target offset might exceed 16bit range. If this happens, the llvm
 410: // will generate incorrect code as the offset is truncated to 16bit.
 411: //
 412: // To fix this rare case, a new insn JMPL is introduced. This new
 413: // insn supports supports 32bit branch target offset. The compiler
 414: // does not use this insn during insn selection. Rather, BPF backend
 415: // will estimate the branch target offset and do JMP -> JMPL and
 416: // JEQ -> JEQ + JMPL conversion if the estimated branch target offset
 417: // is beyond 16bit.
 418: bool BPFMIPreEmitPeephole::adjustBranch() {
 419:   bool Changed = false;
 420:   int CurrNumInsns = 0;
```

- EN: Function bodies or method definitions such as adjustBranch contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: adjustBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 421-440

```cpp
 421:   DenseMap<MachineBasicBlock *, int> SoFarNumInsns;
 422:   DenseMap<MachineBasicBlock *, MachineBasicBlock *> FollowThroughBB;
 423:   std::vector<MachineBasicBlock *> MBBs;
 424: 
 425:   MachineBasicBlock *PrevBB = nullptr;
 426:   for (MachineBasicBlock &MBB : *MF) {
 427:     // MBB.size() is the number of insns in this basic block, including some
 428:     // debug info, e.g., DEBUG_VALUE, so we may over-count a little bit.
 429:     // Typically we have way more normal insns than DEBUG_VALUE insns.
 430:     // Also, if we indeed need to convert conditional branch like JEQ to
 431:     // JEQ + JMPL, we actually introduced some new insns like below.
 432:     CurrNumInsns += (int)MBB.size();
 433:     SoFarNumInsns[&MBB] = CurrNumInsns;
 434:     if (PrevBB != nullptr)
 435:       FollowThroughBB[PrevBB] = &MBB;
 436:     PrevBB = &MBB;
 437:     // A list of original BBs to make later traveral easier.
 438:     MBBs.push_back(&MBB);
 439:   }
 440:   FollowThroughBB[PrevBB] = nullptr;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 441-460

```cpp
 441: 
 442:   for (unsigned i = 0; i < MBBs.size(); i++) {
 443:     // We have four cases here:
 444:     //  (1). no terminator, simple follow through.
 445:     //  (2). jmp to another bb.
 446:     //  (3). conditional jmp to another bb or follow through.
 447:     //  (4). conditional jmp followed by an unconditional jmp.
 448:     MachineInstr *CondJmp = nullptr, *UncondJmp = nullptr;
 449: 
 450:     MachineBasicBlock *MBB = MBBs[i];
 451:     for (MachineInstr &Term : MBB->terminators()) {
 452:       if (Term.isConditionalBranch()) {
 453:         assert(CondJmp == nullptr);
 454:         CondJmp = &Term;
 455:       } else if (Term.isUnconditionalBranch()) {
 456:         assert(UncondJmp == nullptr);
 457:         UncondJmp = &Term;
 458:       }
 459:     }
 460: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461:     // (1). no terminator, simple follow through.
 462:     if (!CondJmp && !UncondJmp)
 463:       continue;
 464: 
 465:     MachineBasicBlock *CondTargetBB, *JmpBB;
 466:     CurrNumInsns = SoFarNumInsns[MBB];
 467: 
 468:     // (2). jmp to another bb.
 469:     if (!CondJmp && UncondJmp) {
 470:       JmpBB = UncondJmp->getOperand(0).getMBB();
 471:       if (in16BitRange(SoFarNumInsns[JmpBB] - JmpBB->size() - CurrNumInsns))
 472:         continue;
 473: 
 474:       // replace this insn as a JMPL.
 475:       BuildMI(MBB, UncondJmp->getDebugLoc(), TII->get(BPF::JMPL)).addMBB(JmpBB);
 476:       UncondJmp->eraseFromParent();
 477:       Changed = true;
 478:       continue;
 479:     }
 480: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 481-500

```cpp
 481:     const BasicBlock *TermBB = MBB->getBasicBlock();
 482:     int Dist;
 483: 
 484:     // (3). conditional jmp to another bb or follow through.
 485:     if (!UncondJmp) {
 486:       CondTargetBB = CondJmp->getOperand(2).getMBB();
 487:       MachineBasicBlock *FollowBB = FollowThroughBB[MBB];
 488:       Dist = SoFarNumInsns[CondTargetBB] - CondTargetBB->size() - CurrNumInsns;
 489:       if (in16BitRange(Dist))
 490:         continue;
 491: 
 492:       // We have
 493:       //   B2: ...
 494:       //       if (cond) goto B5
 495:       //   B3: ...
 496:       // where B2 -> B5 is beyond 16bit range.
 497:       //
 498:       // We do not have 32bit cond jmp insn. So we try to do
 499:       // the following.
 500:       //   B2:     ...
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501:       //           if (cond) goto New_B1
 502:       //   New_B0  goto B3
 503:       //   New_B1: gotol B5
 504:       //   B3: ...
 505:       // Basically two new basic blocks are created.
 506:       MachineBasicBlock *New_B0 = MF->CreateMachineBasicBlock(TermBB);
 507:       MachineBasicBlock *New_B1 = MF->CreateMachineBasicBlock(TermBB);
 508: 
 509:       // Insert New_B0 and New_B1 into function block list.
 510:       MachineFunction::iterator MBB_I  = ++MBB->getIterator();
 511:       MF->insert(MBB_I, New_B0);
 512:       MF->insert(MBB_I, New_B1);
 513: 
 514:       // replace B2 cond jump
 515:       if (CondJmp->getOperand(1).isReg())
 516:         BuildMI(*MBB, MachineBasicBlock::iterator(*CondJmp), CondJmp->getDebugLoc(), TII->get(CondJmp->getOpcode()))
 517:             .addReg(CondJmp->getOperand(0).getReg())
 518:             .addReg(CondJmp->getOperand(1).getReg())
 519:             .addMBB(New_B1);
 520:       else
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 521-540

```cpp
 521:         BuildMI(*MBB, MachineBasicBlock::iterator(*CondJmp), CondJmp->getDebugLoc(), TII->get(CondJmp->getOpcode()))
 522:             .addReg(CondJmp->getOperand(0).getReg())
 523:             .addImm(CondJmp->getOperand(1).getImm())
 524:             .addMBB(New_B1);
 525: 
 526:       // it is possible that CondTargetBB and FollowBB are the same. But the
 527:       // above Dist checking should already filtered this case.
 528:       MBB->removeSuccessor(CondTargetBB);
 529:       MBB->removeSuccessor(FollowBB);
 530:       MBB->addSuccessor(New_B0);
 531:       MBB->addSuccessor(New_B1);
 532: 
 533:       // Populate insns in New_B0 and New_B1.
 534:       BuildMI(New_B0, CondJmp->getDebugLoc(), TII->get(BPF::JMP)).addMBB(FollowBB);
 535:       BuildMI(New_B1, CondJmp->getDebugLoc(), TII->get(BPF::JMPL))
 536:           .addMBB(CondTargetBB);
 537: 
 538:       New_B0->addSuccessor(FollowBB);
 539:       New_B1->addSuccessor(CondTargetBB);
 540:       CondJmp->eraseFromParent();
```

- EN: This range continues the implementation of the backend component described by BPFMIPeephole.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 541-560

```cpp
 541:       Changed = true;
 542:       continue;
 543:     }
 544: 
 545:     //  (4). conditional jmp followed by an unconditional jmp.
 546:     CondTargetBB = CondJmp->getOperand(2).getMBB();
 547:     JmpBB = UncondJmp->getOperand(0).getMBB();
 548: 
 549:     // We have
 550:     //   B2: ...
 551:     //       if (cond) goto B5
 552:     //       JMP B7
 553:     //   B3: ...
 554:     //
 555:     // If only B2->B5 is out of 16bit range, we can do
 556:     //   B2: ...
 557:     //       if (cond) goto new_B
 558:     //       JMP B7
 559:     //   New_B: gotol B5
 560:     //   B3: ...
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 561-580

```cpp
 561:     //
 562:     // If only 'JMP B7' is out of 16bit range, we can replace
 563:     // 'JMP B7' with 'JMPL B7'.
 564:     //
 565:     // If both B2->B5 and 'JMP B7' is out of range, just do
 566:     // both the above transformations.
 567:     Dist = SoFarNumInsns[CondTargetBB] - CondTargetBB->size() - CurrNumInsns;
 568:     if (!in16BitRange(Dist)) {
 569:       MachineBasicBlock *New_B = MF->CreateMachineBasicBlock(TermBB);
 570: 
 571:       // Insert New_B0 into function block list.
 572:       MF->insert(++MBB->getIterator(), New_B);
 573: 
 574:       // replace B2 cond jump
 575:       if (CondJmp->getOperand(1).isReg())
 576:         BuildMI(*MBB, MachineBasicBlock::iterator(*CondJmp), CondJmp->getDebugLoc(), TII->get(CondJmp->getOpcode()))
 577:             .addReg(CondJmp->getOperand(0).getReg())
 578:             .addReg(CondJmp->getOperand(1).getReg())
 579:             .addMBB(New_B);
 580:       else
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 581-600

```cpp
 581:         BuildMI(*MBB, MachineBasicBlock::iterator(*CondJmp), CondJmp->getDebugLoc(), TII->get(CondJmp->getOpcode()))
 582:             .addReg(CondJmp->getOperand(0).getReg())
 583:             .addImm(CondJmp->getOperand(1).getImm())
 584:             .addMBB(New_B);
 585: 
 586:       if (CondTargetBB != JmpBB)
 587:         MBB->removeSuccessor(CondTargetBB);
 588:       MBB->addSuccessor(New_B);
 589: 
 590:       // Populate insn in New_B.
 591:       BuildMI(New_B, CondJmp->getDebugLoc(), TII->get(BPF::JMPL)).addMBB(CondTargetBB);
 592: 
 593:       New_B->addSuccessor(CondTargetBB);
 594:       CondJmp->eraseFromParent();
 595:       Changed = true;
 596:     }
 597: 
 598:     if (!in16BitRange(SoFarNumInsns[JmpBB] - CurrNumInsns)) {
 599:       BuildMI(MBB, UncondJmp->getDebugLoc(), TII->get(BPF::JMPL)).addMBB(JmpBB);
 600:       UncondJmp->eraseFromParent();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 601-620

```cpp
 601:       Changed = true;
 602:     }
 603:   }
 604: 
 605:   return Changed;
 606: }
 607: 
 608: static const unsigned CallerSavedRegs[] = {BPF::R0, BPF::R1, BPF::R2,
 609:                                            BPF::R3, BPF::R4, BPF::R5};
 610: 
 611: struct BPFFastCall {
 612:   MachineInstr *MI;
 613:   unsigned LiveCallerSavedRegs;
 614: };
 615: 
 616: static void collectBPFFastCalls(const TargetRegisterInfo *TRI,
 617:                                 LivePhysRegs &LiveRegs, MachineBasicBlock &BB,
 618:                                 SmallVectorImpl<BPFFastCall> &Calls) {
 619:   LiveRegs.init(*TRI);
 620:   LiveRegs.addLiveOuts(BB);
```

- EN: This chunk introduces interfaces or data structures such as BPFFastCall, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as collectBPFFastCalls contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 BPFFastCall 等接口或数据结构，用于组织该文件暴露的目标专用行为。 collectBPFFastCalls 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 621-640

```cpp
 621:   Calls.clear();
 622:   for (MachineInstr &MI : llvm::reverse(BB)) {
 623:     if (MI.isCall()) {
 624:       unsigned LiveCallerSavedRegs = 0;
 625:       for (MCRegister R : CallerSavedRegs) {
 626:         bool DoSpillFill = false;
 627:         for (MCPhysReg SR : TRI->subregs(R))
 628:           DoSpillFill |= !MI.definesRegister(SR, TRI) && LiveRegs.contains(SR);
 629:         if (!DoSpillFill)
 630:           continue;
 631:         LiveCallerSavedRegs |= 1 << R;
 632:       }
 633:       if (LiveCallerSavedRegs)
 634:         Calls.push_back({&MI, LiveCallerSavedRegs});
 635:     }
 636:     LiveRegs.stepBackward(MI);
 637:   }
 638: }
 639: 
 640: static int64_t computeMinFixedObjOffset(MachineFrameInfo &MFI,
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 641-660

```cpp
 641:                                         unsigned SlotSize) {
 642:   int64_t MinFixedObjOffset = 0;
 643:   // Same logic as in X86FrameLowering::adjustFrameForMsvcCxxEh()
 644:   for (int I = MFI.getObjectIndexBegin(); I < MFI.getObjectIndexEnd(); ++I) {
 645:     if (MFI.isDeadObjectIndex(I))
 646:       continue;
 647:     MinFixedObjOffset = std::min(MinFixedObjOffset, MFI.getObjectOffset(I));
 648:   }
 649:   MinFixedObjOffset -=
 650:       (SlotSize + MinFixedObjOffset % SlotSize) & (SlotSize - 1);
 651:   return MinFixedObjOffset;
 652: }
 653: 
 654: bool BPFMIPreEmitPeephole::insertMissingCallerSavedSpills() {
 655:   MachineFrameInfo &MFI = MF->getFrameInfo();
 656:   SmallVector<BPFFastCall, 8> Calls;
 657:   LivePhysRegs LiveRegs;
 658:   const unsigned SlotSize = 8;
 659:   int64_t MinFixedObjOffset = computeMinFixedObjOffset(MFI, SlotSize);
 660:   bool Changed = false;
```

- EN: Function bodies or method definitions such as insertMissingCallerSavedSpills contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: insertMissingCallerSavedSpills 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 661-680

```cpp
 661:   for (MachineBasicBlock &BB : *MF) {
 662:     collectBPFFastCalls(TRI, LiveRegs, BB, Calls);
 663:     Changed |= !Calls.empty();
 664:     for (BPFFastCall &Call : Calls) {
 665:       int64_t CurOffset = MinFixedObjOffset;
 666:       for (MCRegister Reg : CallerSavedRegs) {
 667:         if (((1 << Reg) & Call.LiveCallerSavedRegs) == 0)
 668:           continue;
 669:         // Allocate stack object
 670:         CurOffset -= SlotSize;
 671:         MFI.CreateFixedSpillStackObject(SlotSize, CurOffset);
 672:         // Generate spill
 673:         BuildMI(BB, Call.MI->getIterator(), Call.MI->getDebugLoc(),
 674:                 TII->get(BPF::STD))
 675:             .addReg(Reg, RegState::Kill)
 676:             .addReg(BPF::R10)
 677:             .addImm(CurOffset);
 678:         // Generate fill
 679:         BuildMI(BB, ++Call.MI->getIterator(), Call.MI->getDebugLoc(),
 680:                 TII->get(BPF::LDD))
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 681-700

```cpp
 681:             .addReg(Reg, RegState::Define)
 682:             .addReg(BPF::R10)
 683:             .addImm(CurOffset);
 684:       }
 685:     }
 686:   }
 687:   return Changed;
 688: }
 689: 
 690: bool BPFMIPreEmitPeephole::removeMayGotoZero() {
 691:   bool Changed = false;
 692:   MachineBasicBlock *Prev_MBB, *Curr_MBB = nullptr;
 693: 
 694:   for (MachineBasicBlock &MBB : make_early_inc_range(reverse(*MF))) {
 695:     Prev_MBB = Curr_MBB;
 696:     Curr_MBB = &MBB;
 697:     if (Prev_MBB == nullptr || Curr_MBB->empty())
 698:       continue;
 699: 
 700:     MachineInstr &MI = Curr_MBB->back();
```

- EN: Function bodies or method definitions such as removeMayGotoZero contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeMayGotoZero 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 701-720

```cpp
 701:     if (MI.getOpcode() != TargetOpcode::INLINEASM_BR)
 702:       continue;
 703: 
 704:     const char *AsmStr = MI.getOperand(0).getSymbolName();
 705:     SmallVector<StringRef, 4> AsmPieces;
 706:     SplitString(AsmStr, AsmPieces, ";\n");
 707: 
 708:     // Do not support multiple insns in one inline asm.
 709:     if (AsmPieces.size() != 1)
 710:       continue;
 711: 
 712:     // The asm insn must be a may_goto insn.
 713:     SmallVector<StringRef, 4> AsmOpPieces;
 714:     SplitString(AsmPieces[0], AsmOpPieces, " ");
 715:     if (AsmOpPieces.size() != 2 || AsmOpPieces[0] != "may_goto")
 716:       continue;
 717:     // Enforce the format of 'may_goto <label>'.
 718:     if (AsmOpPieces[1] != "${0:l}" && AsmOpPieces[1] != "$0")
 719:       continue;
 720: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 721-740

```cpp
 721:     // Get the may_goto branch target.
 722:     MachineOperand &MO = MI.getOperand(InlineAsm::MIOp_FirstOperand + 1);
 723:     if (!MO.isMBB() || MO.getMBB() != Prev_MBB)
 724:       continue;
 725: 
 726:     Changed = true;
 727:     if (Curr_MBB->begin() == MI) {
 728:       // Single 'may_goto' insn in the same basic block.
 729:       Curr_MBB->removeSuccessor(Prev_MBB);
 730:       for (MachineBasicBlock *Pred : Curr_MBB->predecessors())
 731:         Pred->replaceSuccessor(Curr_MBB, Prev_MBB);
 732:       Curr_MBB->eraseFromParent();
 733:       Curr_MBB = Prev_MBB;
 734:     } else {
 735:       // Remove 'may_goto' insn.
 736:       MI.eraseFromParent();
 737:     }
 738:   }
 739: 
 740:   return Changed;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 741-760

```cpp
 741: }
 742: 
 743: // If the last insn in a funciton is 'JAL &bpf_unreachable', let us add an
 744: // 'exit' insn after that insn. This will ensure no fallthrough at the last
 745: // insn, making kernel verification easier.
 746: bool BPFMIPreEmitPeephole::addExitAfterUnreachable() {
 747:   MachineBasicBlock &MBB = MF->back();
 748:   MachineInstr &MI = MBB.back();
 749:   if (MI.getOpcode() != BPF::JAL || !MI.getOperand(0).isGlobal() ||
 750:       MI.getOperand(0).getGlobal()->getName() != BPF_TRAP)
 751:     return false;
 752: 
 753:   BuildMI(&MBB, MI.getDebugLoc(), TII->get(BPF::RET));
 754:   return true;
 755: }
 756: 
 757: bool BPFMIPreEmitPeephole::expandStackArgPseudos() {
 758:   bool Changed = false;
 759: 
 760:   for (MachineBasicBlock &MBB : *MF) {
```

- EN: Function bodies or method definitions such as addExitAfterUnreachable, expandStackArgPseudos contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: addExitAfterUnreachable, expandStackArgPseudos 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 761-780

```cpp
 761:     for (auto It = MBB.begin(), End = MBB.end(); It != End;) {
 762:       MachineInstr &MI = *It++;
 763:       DebugLoc DL = MI.getDebugLoc();
 764: 
 765:       switch (MI.getOpcode()) {
 766:       default:
 767:         break;
 768: 
 769:       case BPF::LOAD_STACK_ARG_PSEUDO: {
 770:         Register DstReg = MI.getOperand(0).getReg();
 771:         int16_t Off = MI.getOperand(1).getImm();
 772: 
 773:         BuildMI(MBB, MI, DL, TII->get(BPF::LDD), DstReg)
 774:             .addReg(BPF::R11)
 775:             .addImm(Off);
 776:         MI.eraseFromParent();
 777:         Changed = true;
 778:         break;
 779:       }
 780: 
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 781-800

```cpp
 781:       case BPF::STORE_STACK_ARG_PSEUDO: {
 782:         int16_t Off = MI.getOperand(0).getImm();
 783:         const MachineOperand &SrcMO = MI.getOperand(1);
 784:         Register SrcReg = SrcMO.getReg();
 785:         bool IsKill = SrcMO.isKill();
 786: 
 787:         BuildMI(MBB, MI, DL, TII->get(BPF::STD))
 788:             .addReg(SrcReg, getKillRegState(IsKill))
 789:             .addReg(BPF::R11)
 790:             .addImm(Off);
 791:         MI.eraseFromParent();
 792:         Changed = true;
 793:         break;
 794:       }
 795: 
 796:       case BPF::STORE_STACK_ARG_IMM_PSEUDO: {
 797:         int16_t Off = MI.getOperand(0).getImm();
 798:         int32_t Val = MI.getOperand(1).getImm();
 799: 
 800:         BuildMI(MBB, MI, DL, TII->get(BPF::STD_imm))
```

- EN: This range continues the implementation of the backend component described by BPFMIPeephole.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 801-820

```cpp
 801:             .addImm(Val)
 802:             .addReg(BPF::R11)
 803:             .addImm(Off);
 804:         MI.eraseFromParent();
 805:         Changed = true;
 806:         break;
 807:       }
 808:       }
 809:     }
 810:   }
 811: 
 812:   return Changed;
 813: }
 814: 
 815: } // end default namespace
 816: 
 817: INITIALIZE_PASS(BPFMIPreEmitPeephole, "bpf-mi-pemit-peephole",
 818:                 "BPF PreEmit Peephole Optimization", false, false)
 819: 
 820: char BPFMIPreEmitPeephole::ID = 0;
```

- EN: This range continues the implementation of the backend component described by BPFMIPeephole.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 821-824

```cpp
 821: FunctionPass* llvm::createBPFMIPreEmitPeepholePass()
 822: {
 823:   return new BPFMIPreEmitPeephole();
 824: }
```

- EN: Function bodies or method definitions such as createBPFMIPreEmitPeepholePass contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: createBPFMIPreEmitPeepholePass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFInstrInfo.h`, `BPFTargetMachine.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringExtras.h`, `llvm/CodeGen/LivePhysRegs.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunctionPass.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
