# ARCInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCInstrInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the ARC implementation of the TargetInstrInfo class.
- 目的（中文）: 定义目标指令属性以及 LLVM CodeGen 各 Pass 使用的辅助方法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCInstrInfo.cpp - ARC Instruction Information -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the ARC implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "ARCInstrInfo.h"
  14: #include "ARC.h"
  15: #include "ARCMachineFunctionInfo.h"
  16: #include "ARCSubtarget.h"
  17: #include "MCTargetDesc/ARCInfo.h"
  18: #include "llvm/CodeGen/MachineFrameInfo.h"
  19: #include "llvm/CodeGen/MachineInstrBuilder.h"
  20: #include "llvm/CodeGen/MachineMemOperand.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 21-40

```cpp
  21: #include "llvm/MC/TargetRegistry.h"
  22: #include "llvm/Support/Debug.h"
  23: 
  24: using namespace llvm;
  25: 
  26: #define GET_INSTRINFO_CTOR_DTOR
  27: #include "ARCGenInstrInfo.inc"
  28: 
  29: #define DEBUG_TYPE "arc-inst-info"
  30: 
  31: enum AddrIncType {
  32:     NoAddInc = 0,
  33:     PreInc   = 1,
  34:     PostInc  = 2,
  35:     Scaled   = 3
  36: };
  37: 
  38: enum TSFlagsConstants {
  39:     TSF_AddrModeOff = 0,
  40:     TSF_AddModeMask = 3
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 41-60

```cpp
  41: };
  42: 
  43: // Pin the vtable to this file.
  44: void ARCInstrInfo::anchor() {}
  45: 
  46: ARCInstrInfo::ARCInstrInfo(const ARCSubtarget &ST)
  47:     : ARCGenInstrInfo(ST, RI, ARC::ADJCALLSTACKDOWN, ARC::ADJCALLSTACKUP),
  48:       RI(ST) {}
  49: 
  50: static bool isZeroImm(const MachineOperand &Op) {
  51:   return Op.isImm() && Op.getImm() == 0;
  52: }
  53: 
  54: static bool isLoad(int Opcode) {
  55:   return Opcode == ARC::LD_rs9 || Opcode == ARC::LDH_rs9 ||
  56:          Opcode == ARC::LDB_rs9;
  57: }
  58: 
  59: static bool isStore(int Opcode) {
  60:   return Opcode == ARC::ST_rs9 || Opcode == ARC::STH_rs9 ||
```

- EN: Function bodies or method definitions such as anchor, ARCGenInstrInfo, isZeroImm contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: anchor, ARCGenInstrInfo, isZeroImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 61-80

```cpp
  61:          Opcode == ARC::STB_rs9;
  62: }
  63: 
  64: /// If the specified machine instruction is a direct
  65: /// load from a stack slot, return the virtual or physical register number of
  66: /// the destination along with the FrameIndex of the loaded stack slot.  If
  67: /// not, return 0.  This predicate must return 0 if the instruction has
  68: /// any side effects other than loading from the stack slot.
  69: Register ARCInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
  70:                                            int &FrameIndex) const {
  71:   int Opcode = MI.getOpcode();
  72:   if (isLoad(Opcode)) {
  73:     if ((MI.getOperand(1).isFI()) &&  // is a stack slot
  74:         (MI.getOperand(2).isImm()) && // the imm is zero
  75:         (isZeroImm(MI.getOperand(2)))) {
  76:       FrameIndex = MI.getOperand(1).getIndex();
  77:       return MI.getOperand(0).getReg();
  78:     }
  79:   }
  80:   return 0;
```

- EN: Function bodies or method definitions such as isLoadFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isLoadFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 81-100

```cpp
  81: }
  82: 
  83: /// If the specified machine instruction is a direct
  84: /// store to a stack slot, return the virtual or physical register number of
  85: /// the source reg along with the FrameIndex of the loaded stack slot.  If
  86: /// not, return 0.  This predicate must return 0 if the instruction has
  87: /// any side effects other than storing to the stack slot.
  88: Register ARCInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
  89:                                           int &FrameIndex) const {
  90:   int Opcode = MI.getOpcode();
  91:   if (isStore(Opcode)) {
  92:     if ((MI.getOperand(1).isFI()) &&  // is a stack slot
  93:         (MI.getOperand(2).isImm()) && // the imm is zero
  94:         (isZeroImm(MI.getOperand(2)))) {
  95:       FrameIndex = MI.getOperand(1).getIndex();
  96:       return MI.getOperand(0).getReg();
  97:     }
  98:   }
  99:   return 0;
 100: }
```

- EN: Function bodies or method definitions such as isStoreToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isStoreToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 101-120

```cpp
 101: 
 102: /// Return the inverse of passed condition, i.e. turning COND_E to COND_NE.
 103: static ARCCC::CondCode getOppositeBranchCondition(ARCCC::CondCode CC) {
 104:   switch (CC) {
 105:   default:
 106:     llvm_unreachable("Illegal condition code!");
 107:   case ARCCC::EQ:
 108:     return ARCCC::NE;
 109:   case ARCCC::NE:
 110:     return ARCCC::EQ;
 111:   case ARCCC::LO:
 112:     return ARCCC::HS;
 113:   case ARCCC::HS:
 114:     return ARCCC::LO;
 115:   case ARCCC::GT:
 116:     return ARCCC::LE;
 117:   case ARCCC::GE:
 118:     return ARCCC::LT;
 119:   case ARCCC::VS:
 120:     return ARCCC::VC;
```

- EN: Function bodies or method definitions such as getOppositeBranchCondition contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getOppositeBranchCondition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 121-140

```cpp
 121:   case ARCCC::VC:
 122:     return ARCCC::VS;
 123:   case ARCCC::LT:
 124:     return ARCCC::GE;
 125:   case ARCCC::LE:
 126:     return ARCCC::GT;
 127:   case ARCCC::HI:
 128:     return ARCCC::LS;
 129:   case ARCCC::LS:
 130:     return ARCCC::HI;
 131:   case ARCCC::NZ:
 132:     return ARCCC::Z;
 133:   case ARCCC::Z:
 134:     return ARCCC::NZ;
 135:   }
 136: }
 137: 
 138: static bool isUncondBranchOpcode(int Opc) { return Opc == ARC::BR; }
 139: 
 140: static bool isCondBranchOpcode(int Opc) {
```

- EN: Function bodies or method definitions such as isUncondBranchOpcode, isCondBranchOpcode contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isUncondBranchOpcode, isCondBranchOpcode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 141-160

```cpp
 141:   return Opc == ARC::BRcc_rr_p || Opc == ARC::BRcc_ru6_p;
 142: }
 143: 
 144: static bool isJumpOpcode(int Opc) { return Opc == ARC::J; }
 145: 
 146: /// Analyze the branching code at the end of MBB, returning
 147: /// true if it cannot be understood (e.g. it's a switch dispatch or isn't
 148: /// implemented for a target).  Upon success, this returns false and returns
 149: /// with the following information in various cases:
 150: ///
 151: /// 1. If this block ends with no branches (it just falls through to its succ)
 152: ///    just return false, leaving TBB/FBB null.
 153: /// 2. If this block ends with only an unconditional branch, it sets TBB to be
 154: ///    the destination block.
 155: /// 3. If this block ends with a conditional branch and it falls through to a
 156: ///    successor block, it sets TBB to be the branch destination block and a
 157: ///    list of operands that evaluate the condition. These operands can be
 158: ///    passed to other TargetInstrInfo methods to create new branches.
 159: /// 4. If this block ends with a conditional branch followed by an
 160: ///    unconditional branch, it returns the 'true' destination in TBB, the
```

- EN: Function bodies or method definitions such as isJumpOpcode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: isJumpOpcode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 161-180

```cpp
 161: ///    'false' destination in FBB, and a list of operands that evaluate the
 162: ///    condition.  These operands can be passed to other TargetInstrInfo
 163: ///    methods to create new branches.
 164: ///
 165: /// Note that RemoveBranch and insertBranch must be implemented to support
 166: /// cases where this method returns success.
 167: ///
 168: /// If AllowModify is true, then this routine is allowed to modify the basic
 169: /// block (e.g. delete instructions after the unconditional branch).
 170: 
 171: bool ARCInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 172:                                  MachineBasicBlock *&TBB,
 173:                                  MachineBasicBlock *&FBB,
 174:                                  SmallVectorImpl<MachineOperand> &Cond,
 175:                                  bool AllowModify) const {
 176:   TBB = FBB = nullptr;
 177:   MachineBasicBlock::iterator I = MBB.end();
 178:   if (I == MBB.begin())
 179:     return false;
 180:   --I;
```

- EN: Function bodies or method definitions such as analyzeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: analyzeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 181-200

```cpp
 181: 
 182:   while (isPredicated(*I) || I->isTerminator() || I->isDebugValue()) {
 183:     // Flag to be raised on unanalyzeable instructions. This is useful in cases
 184:     // where we want to clean up on the end of the basic block before we bail
 185:     // out.
 186:     bool CantAnalyze = false;
 187: 
 188:     // Skip over DEBUG values and predicated nonterminators.
 189:     while (I->isDebugInstr() || !I->isTerminator()) {
 190:       if (I == MBB.begin())
 191:         return false;
 192:       --I;
 193:     }
 194: 
 195:     if (isJumpOpcode(I->getOpcode())) {
 196:       // Indirect branches and jump tables can't be analyzed, but we still want
 197:       // to clean up any instructions at the tail of the basic block.
 198:       CantAnalyze = true;
 199:     } else if (isUncondBranchOpcode(I->getOpcode())) {
 200:       TBB = I->getOperand(0).getMBB();
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 201-220

```cpp
 201:     } else if (isCondBranchOpcode(I->getOpcode())) {
 202:       // Bail out if we encounter multiple conditional branches.
 203:       if (!Cond.empty())
 204:         return true;
 205: 
 206:       assert(!FBB && "FBB should have been null.");
 207:       FBB = TBB;
 208:       TBB = I->getOperand(0).getMBB();
 209:       Cond.push_back(I->getOperand(1));
 210:       Cond.push_back(I->getOperand(2));
 211:       Cond.push_back(I->getOperand(3));
 212:     } else if (I->isReturn()) {
 213:       // Returns can't be analyzed, but we should run cleanup.
 214:       CantAnalyze = !isPredicated(*I);
 215:     } else {
 216:       // We encountered other unrecognized terminator. Bail out immediately.
 217:       return true;
 218:     }
 219: 
 220:     // Cleanup code - to be run for unpredicated unconditional branches and
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 221-240

```cpp
 221:     //                returns.
 222:     if (!isPredicated(*I) && (isUncondBranchOpcode(I->getOpcode()) ||
 223:                               isJumpOpcode(I->getOpcode()) || I->isReturn())) {
 224:       // Forget any previous condition branch information - it no longer
 225:       // applies.
 226:       Cond.clear();
 227:       FBB = nullptr;
 228: 
 229:       // If we can modify the function, delete everything below this
 230:       // unconditional branch.
 231:       if (AllowModify) {
 232:         MachineBasicBlock::iterator DI = std::next(I);
 233:         while (DI != MBB.end()) {
 234:           MachineInstr &InstToDelete = *DI;
 235:           ++DI;
 236:           InstToDelete.eraseFromParent();
 237:         }
 238:       }
 239:     }
 240: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 241-260

```cpp
 241:     if (CantAnalyze)
 242:       return true;
 243: 
 244:     if (I == MBB.begin())
 245:       return false;
 246: 
 247:     --I;
 248:   }
 249: 
 250:   // We made it past the terminators without bailing out - we must have
 251:   // analyzed this branch successfully.
 252:   return false;
 253: }
 254: 
 255: unsigned ARCInstrInfo::removeBranch(MachineBasicBlock &MBB,
 256:                                     int *BytesRemoved) const {
 257:   assert(!BytesRemoved && "Code size not handled");
 258:   MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
 259:   if (I == MBB.end())
 260:     return 0;
```

- EN: Function bodies or method definitions such as removeBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: removeBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 261-280

```cpp
 261: 
 262:   if (!isUncondBranchOpcode(I->getOpcode()) &&
 263:       !isCondBranchOpcode(I->getOpcode()))
 264:     return 0;
 265: 
 266:   // Remove the branch.
 267:   I->eraseFromParent();
 268: 
 269:   I = MBB.end();
 270: 
 271:   if (I == MBB.begin())
 272:     return 1;
 273:   --I;
 274:   if (!isCondBranchOpcode(I->getOpcode()))
 275:     return 1;
 276: 
 277:   // Remove the branch.
 278:   I->eraseFromParent();
 279:   return 2;
 280: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 281-300

```cpp
 281: 
 282: void ARCInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
 283:                                MachineBasicBlock::iterator I,
 284:                                const DebugLoc &DL, Register DestReg,
 285:                                Register SrcReg, bool KillSrc,
 286:                                bool RenamableDest, bool RenamableSrc) const {
 287:   assert(ARC::GPR32RegClass.contains(SrcReg) &&
 288:          "Only GPR32 src copy supported.");
 289:   assert(ARC::GPR32RegClass.contains(DestReg) &&
 290:          "Only GPR32 dest copy supported.");
 291:   BuildMI(MBB, I, DL, get(ARC::MOV_rr), DestReg)
 292:       .addReg(SrcReg, getKillRegState(KillSrc));
 293: }
 294: 
 295: void ARCInstrInfo::storeRegToStackSlot(
 296:     MachineBasicBlock &MBB, MachineBasicBlock::iterator I, Register SrcReg,
 297:     bool IsKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
 298:     MachineInstr::MIFlag Flags) const {
 299:   DebugLoc DL = MBB.findDebugLoc(I);
 300:   MachineFunction &MF = *MBB.getParent();
```

- EN: Function bodies or method definitions such as copyPhysReg, storeRegToStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: copyPhysReg, storeRegToStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 301-320

```cpp
 301:   MachineFrameInfo &MFI = MF.getFrameInfo();
 302: 
 303:   MachineMemOperand *MMO = MF.getMachineMemOperand(
 304:       MachinePointerInfo::getFixedStack(MF, FrameIndex),
 305:       MachineMemOperand::MOStore, MFI.getObjectSize(FrameIndex),
 306:       MFI.getObjectAlign(FrameIndex));
 307: 
 308:   assert(MMO && "Couldn't get MachineMemOperand for store to stack.");
 309:   assert(TRI.getSpillSize(*RC) == 4 &&
 310:          "Only support 4-byte stores to stack now.");
 311:   assert(ARC::GPR32RegClass.hasSubClassEq(RC) &&
 312:          "Only support GPR32 stores to stack now.");
 313:   LLVM_DEBUG(dbgs() << "Created store reg=" << printReg(SrcReg, &TRI)
 314:                     << " to FrameIndex=" << FrameIndex << "\n");
 315:   BuildMI(MBB, I, DL, get(ARC::ST_rs9))
 316:       .addReg(SrcReg, getKillRegState(IsKill))
 317:       .addFrameIndex(FrameIndex)
 318:       .addImm(0)
 319:       .addMemOperand(MMO);
 320: }
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 321-340

```cpp
 321: 
 322: void ARCInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
 323:                                         MachineBasicBlock::iterator I,
 324:                                         Register DestReg, int FrameIndex,
 325:                                         const TargetRegisterClass *RC,
 326:                                         Register VReg, unsigned SubReg,
 327:                                         MachineInstr::MIFlag Flags) const {
 328:   DebugLoc DL = MBB.findDebugLoc(I);
 329:   MachineFunction &MF = *MBB.getParent();
 330:   MachineFrameInfo &MFI = MF.getFrameInfo();
 331:   MachineMemOperand *MMO = MF.getMachineMemOperand(
 332:       MachinePointerInfo::getFixedStack(MF, FrameIndex),
 333:       MachineMemOperand::MOLoad, MFI.getObjectSize(FrameIndex),
 334:       MFI.getObjectAlign(FrameIndex));
 335: 
 336:   assert(MMO && "Couldn't get MachineMemOperand for store to stack.");
 337:   assert(TRI.getSpillSize(*RC) == 4 &&
 338:          "Only support 4-byte loads from stack now.");
 339:   assert(ARC::GPR32RegClass.hasSubClassEq(RC) &&
 340:          "Only support GPR32 stores to stack now.");
```

- EN: Function bodies or method definitions such as loadRegFromStackSlot contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: loadRegFromStackSlot 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 341-360

```cpp
 341:   LLVM_DEBUG(dbgs() << "Created load reg=" << printReg(DestReg, &TRI)
 342:                     << " from FrameIndex=" << FrameIndex << "\n");
 343:   BuildMI(MBB, I, DL, get(ARC::LD_rs9))
 344:       .addReg(DestReg, RegState::Define)
 345:       .addFrameIndex(FrameIndex)
 346:       .addImm(0)
 347:       .addMemOperand(MMO);
 348: }
 349: 
 350: /// Return the inverse opcode of the specified Branch instruction.
 351: bool ARCInstrInfo::reverseBranchCondition(
 352:     SmallVectorImpl<MachineOperand> &Cond) const {
 353:   assert((Cond.size() == 3) && "Invalid ARC branch condition!");
 354:   Cond[2].setImm(getOppositeBranchCondition((ARCCC::CondCode)Cond[2].getImm()));
 355:   return false;
 356: }
 357: 
 358: MachineBasicBlock::iterator
 359: ARCInstrInfo::loadImmediate(MachineBasicBlock &MBB,
 360:                             MachineBasicBlock::iterator MI, unsigned Reg,
```

- EN: Function bodies or method definitions such as reverseBranchCondition contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: reverseBranchCondition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 361-380

```cpp
 361:                             uint64_t Value) const {
 362:   DebugLoc DL = MBB.findDebugLoc(MI);
 363:   if (isInt<12>(Value)) {
 364:     return BuildMI(MBB, MI, DL, get(ARC::MOV_rs12), Reg)
 365:         .addImm(Value)
 366:         .getInstr();
 367:   }
 368:   llvm_unreachable("Need Arc long immediate instructions.");
 369: }
 370: 
 371: unsigned ARCInstrInfo::insertBranch(MachineBasicBlock &MBB,
 372:                                     MachineBasicBlock *TBB,
 373:                                     MachineBasicBlock *FBB,
 374:                                     ArrayRef<MachineOperand> Cond,
 375:                                     const DebugLoc &DL, int *BytesAdded) const {
 376:   assert(!BytesAdded && "Code size not handled.");
 377: 
 378:   // Shouldn't be a fall through.
 379:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
 380:   assert((Cond.size() == 3 || Cond.size() == 0) &&
```

- EN: Function bodies or method definitions such as insertBranch contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: insertBranch 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 381-400

```cpp
 381:          "ARC branch conditions have two components!");
 382: 
 383:   if (Cond.empty()) {
 384:     BuildMI(&MBB, DL, get(ARC::BR)).addMBB(TBB);
 385:     return 1;
 386:   }
 387:   int BccOpc = Cond[1].isImm() ? ARC::BRcc_ru6_p : ARC::BRcc_rr_p;
 388:   MachineInstrBuilder MIB = BuildMI(&MBB, DL, get(BccOpc));
 389:   MIB.addMBB(TBB);
 390:   for (unsigned i = 0; i < 3; i++) {
 391:     MIB.add(Cond[i]);
 392:   }
 393: 
 394:   // One-way conditional branch.
 395:   if (!FBB) {
 396:     return 1;
 397:   }
 398: 
 399:   // Two-way conditional branch.
 400:   BuildMI(&MBB, DL, get(ARC::BR)).addMBB(FBB);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 401-420

```cpp
 401:   return 2;
 402: }
 403: 
 404: unsigned ARCInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
 405:   if (MI.isInlineAsm()) {
 406:     const MachineFunction *MF = MI.getParent()->getParent();
 407:     const char *AsmStr = MI.getOperand(0).getSymbolName();
 408:     return getInlineAsmLength(AsmStr, MF->getTarget().getMCAsmInfo());
 409:   }
 410:   return MI.getDesc().getSize();
 411: }
 412: 
 413: bool ARCInstrInfo::isPostIncrement(const MachineInstr &MI) const {
 414:   const MCInstrDesc &MID = MI.getDesc();
 415:   const uint64_t F = MID.TSFlags;
 416:   return ((F >> TSF_AddrModeOff) & TSF_AddModeMask) == PostInc;
 417: }
 418: 
 419: bool ARCInstrInfo::isPreIncrement(const MachineInstr &MI) const {
 420:   const MCInstrDesc &MID = MI.getDesc();
```

- EN: Function bodies or method definitions such as getInstSizeInBytes, isPostIncrement, isPreIncrement contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getInstSizeInBytes, isPostIncrement, isPreIncrement 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 421-440

```cpp
 421:   const uint64_t F = MID.TSFlags;
 422:   return ((F >> TSF_AddrModeOff) & TSF_AddModeMask) == PreInc;
 423: }
 424: 
 425: bool ARCInstrInfo::getBaseAndOffsetPosition(const MachineInstr &MI,
 426:                                         unsigned &BasePos,
 427:                                         unsigned &OffsetPos) const {
 428:   if (!MI.mayLoad() && !MI.mayStore())
 429:     return false;
 430: 
 431:   BasePos = 1;
 432:   OffsetPos = 2;
 433: 
 434:   if (isPostIncrement(MI) || isPreIncrement(MI)) {
 435:     BasePos++;
 436:     OffsetPos++;
 437:   }
 438: 
 439:   if (!MI.getOperand(BasePos).isReg() || !MI.getOperand(OffsetPos).isImm())
 440:     return false;
```

- EN: Function bodies or method definitions such as getBaseAndOffsetPosition contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: getBaseAndOffsetPosition 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

### Lines 441-443

```cpp
 441: 
 442:   return true;
 443: }
```

- EN: At the backend level, this range exposes instruction semantics and utility logic used by multiple passes.
- 中文: 在后端层面，这一段提供多种 Pass 共享的指令语义和辅助逻辑。

## Key Concepts / 关键概念

- Instruction semantics / 指令语义
- CodeGen helper hooks / CodeGen 辅助钩子
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCInstrInfo.h`, `ARC.h`, `ARCMachineFunctionInfo.h`, `ARCSubtarget.h`, `MCTargetDesc/ARCInfo.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineMemOperand.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Generated or companion files / 生成或配套文件: `ARCGenInstrInfo.inc`
- Local companions / 本地配套文件: `ARCInstrInfo.h`, `ARCInstrInfo.td`
