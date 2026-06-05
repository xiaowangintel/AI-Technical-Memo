# M68kInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target instruction information for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责定义或实现目标指令信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- M68kInstrInfo.cpp - M68k Instruction Information --------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains the M68k declaration of the TargetInstrInfo class.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "M68kInstrInfo.h"
  15: 
  16: #include "M68kInstrBuilder.h"
  17: #include "M68kMachineFunction.h"
  18: #include "M68kTargetMachine.h"
  19: #include "MCTargetDesc/M68kMCCodeEmitter.h"
  20: 
  21: #include "llvm/ADT/STLExtras.h"
  22: #include "llvm/ADT/ScopeExit.h"
  23: #include "llvm/CodeGen/LivePhysRegs.h"
  24: #include "llvm/CodeGen/LiveVariables.h"
  25: #include "llvm/CodeGen/MachineInstrBuilder.h"
  26: #include "llvm/CodeGen/MachineOperand.h"
  27: #include "llvm/CodeGen/MachineRegisterInfo.h"
  28: #include "llvm/CodeGenTypes/MachineValueType.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kInstrInfo.h`, `M68kInstrBuilder.h`, `M68kMachineFunction.h`, `M68kTargetMachine.h`, `M68kMCCodeEmitter.h`, `STLExtras.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kInstrInfo.h`, `M68kInstrBuilder.h`, `M68kMachineFunction.h`, `M68kTargetMachine.h`, `M68kMCCodeEmitter.h`, `STLExtras.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 29-56 / 第 29-56 行
```cpp
  29: #include "llvm/MC/TargetRegistry.h"
  30: #include "llvm/Support/ErrorHandling.h"
  31: #include "llvm/Support/Regex.h"
  32: 
  33: #include <functional>
  34: 
  35: using namespace llvm;
  36: 
  37: #define DEBUG_TYPE "M68k-instr-info"
  38: 
  39: #define GET_INSTRINFO_CTOR_DTOR
  40: #include "M68kGenInstrInfo.inc"
  41: 
  42: // Pin the vtable to this file.
  43: void M68kInstrInfo::anchor() {}
  44: 
  45: M68kInstrInfo::M68kInstrInfo(const M68kSubtarget &STI)
  46:     : M68kGenInstrInfo(STI, RI, M68k::ADJCALLSTACKDOWN, M68k::ADJCALLSTACKUP, 0,
  47:                        M68k::RET),
  48:       Subtarget(STI), RI(STI) {}
  49: 
  50: static M68k::CondCode getCondFromBranchOpc(unsigned BrOpc) {
  51:   switch (BrOpc) {
  52:   default:
  53:     return M68k::COND_INVALID;
  54:   case M68k::Beq8:
  55:     return M68k::COND_EQ;
  56:   case M68k::Bne8:
```
- **EN**: It imports dependencies such as `TargetRegistry.h`, `ErrorHandling.h`, `Regex.h`, `functional`, `M68kGenInstrInfo.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `M68kInstrInfo::anchor`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 它引入了 `TargetRegistry.h`, `ErrorHandling.h`, `Regex.h`, `functional`, `M68kGenInstrInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `M68kInstrInfo::anchor` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 57-84 / 第 57-84 行
```cpp
  57:     return M68k::COND_NE;
  58:   case M68k::Blt8:
  59:     return M68k::COND_LT;
  60:   case M68k::Ble8:
  61:     return M68k::COND_LE;
  62:   case M68k::Bgt8:
  63:     return M68k::COND_GT;
  64:   case M68k::Bge8:
  65:     return M68k::COND_GE;
  66:   case M68k::Bcs8:
  67:     return M68k::COND_CS;
  68:   case M68k::Bls8:
  69:     return M68k::COND_LS;
  70:   case M68k::Bhi8:
  71:     return M68k::COND_HI;
  72:   case M68k::Bcc8:
  73:     return M68k::COND_CC;
  74:   case M68k::Bmi8:
  75:     return M68k::COND_MI;
  76:   case M68k::Bpl8:
  77:     return M68k::COND_PL;
  78:   case M68k::Bvs8:
  79:     return M68k::COND_VS;
  80:   case M68k::Bvc8:
  81:     return M68k::COND_VC;
  82:   }
  83: }
  84: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 85-112 / 第 85-112 行
```cpp
  85: bool M68kInstrInfo::AnalyzeBranchImpl(MachineBasicBlock &MBB,
  86:                                       MachineBasicBlock *&TBB,
  87:                                       MachineBasicBlock *&FBB,
  88:                                       SmallVectorImpl<MachineOperand> &Cond,
  89:                                       bool AllowModify) const {
  90: 
  91:   auto UncondBranch =
  92:       std::pair<MachineBasicBlock::reverse_iterator, MachineBasicBlock *>{
  93:           MBB.rend(), nullptr};
  94: 
  95:   // Erase any instructions if allowed at the end of the scope.
  96:   std::vector<std::reference_wrapper<llvm::MachineInstr>> EraseList;
  97:   llvm::scope_exit FinalizeOnReturn([&EraseList] {
  98:     for (auto &Ref : EraseList)
  99:       Ref.get().eraseFromParent();
 100:   });
 101: 
 102:   // Start from the bottom of the block and work up, examining the
 103:   // terminator instructions.
 104:   for (auto iter = MBB.rbegin(); iter != MBB.rend(); iter = std::next(iter)) {
 105: 
 106:     unsigned Opcode = iter->getOpcode();
 107: 
 108:     if (iter->isDebugInstr())
 109:       continue;
 110: 
 111:     // Working from the bottom, when we see a non-terminator instruction, we're
 112:     // done.
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::AnalyzeBranchImpl`, `FinalizeOnReturn`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::AnalyzeBranchImpl`, `FinalizeOnReturn` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 113-140 / 第 113-140 行
```cpp
 113:     if (!isUnpredicatedTerminator(*iter))
 114:       break;
 115: 
 116:     // A terminator that isn't a branch can't easily be handled by this
 117:     // analysis.
 118:     if (!iter->isBranch())
 119:       return true;
 120: 
 121:     // Handle unconditional branches.
 122:     if (Opcode == M68k::BRA8 || Opcode == M68k::BRA16) {
 123:       if (!iter->getOperand(0).isMBB())
 124:         return true;
 125:       UncondBranch = {iter, iter->getOperand(0).getMBB()};
 126: 
 127:       // TBB is used to indicate the unconditional destination.
 128:       TBB = UncondBranch.second;
 129: 
 130:       if (!AllowModify)
 131:         continue;
 132: 
 133:       // If the block has any instructions after a JMP, erase them.
 134:       EraseList.insert(EraseList.begin(), MBB.rbegin(), iter);
 135: 
 136:       Cond.clear();
 137:       FBB = nullptr;
 138: 
 139:       // Erase the JMP if it's equivalent to a fall-through.
 140:       if (MBB.isLayoutSuccessor(UncondBranch.second)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```cpp
 141:         TBB = nullptr;
 142:         EraseList.push_back(*iter);
 143:         UncondBranch = {MBB.rend(), nullptr};
 144:       }
 145: 
 146:       continue;
 147:     }
 148: 
 149:     // Handle conditional branches.
 150:     auto BranchCode = M68k::GetCondFromBranchOpc(Opcode);
 151: 
 152:     // Can't handle indirect branch.
 153:     if (BranchCode == M68k::COND_INVALID)
 154:       return true;
 155: 
 156:     // In practice we should never have an undef CCR operand, if we do
 157:     // abort here as we are not prepared to preserve the flag.
 158:     // ??? Is this required?
 159:     // if (iter->getOperand(1).isUndef())
 160:     //   return true;
 161: 
 162:     // Working from the bottom, handle the first conditional branch.
 163:     if (Cond.empty()) {
 164:       if (!iter->getOperand(0).isMBB())
 165:         return true;
 166:       MachineBasicBlock *CondBranchTarget = iter->getOperand(0).getMBB();
 167: 
 168:       // If we see something like this:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:       //
 170:       //     bcc l1
 171:       //     bra l2
 172:       //     ...
 173:       //   l1:
 174:       //     ...
 175:       //   l2:
 176:       if (UncondBranch.first != MBB.rend()) {
 177: 
 178:         assert(std::next(UncondBranch.first) == iter && "Wrong block layout.");
 179: 
 180:         // And we are allowed to modify the block and the target block of the
 181:         // conditional branch is the direct successor of this block:
 182:         //
 183:         //     bcc l1
 184:         //     bra l2
 185:         //   l1:
 186:         //     ...
 187:         //   l2:
 188:         //
 189:         // we change it to this if allowed:
 190:         //
 191:         //     bncc l2
 192:         //   l1:
 193:         //     ...
 194:         //   l2:
 195:         //
 196:         // Which is a bit more efficient.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 197-224 / 第 197-224 行
```cpp
 197:         if (AllowModify && MBB.isLayoutSuccessor(CondBranchTarget)) {
 198: 
 199:           BranchCode = GetOppositeBranchCondition(BranchCode);
 200:           unsigned BNCC = GetCondBranchFromCond(BranchCode);
 201: 
 202:           BuildMI(MBB, *UncondBranch.first, MBB.rfindDebugLoc(iter), get(BNCC))
 203:               .addMBB(UncondBranch.second);
 204: 
 205:           EraseList.push_back(*iter);
 206:           EraseList.push_back(*UncondBranch.first);
 207: 
 208:           TBB = UncondBranch.second;
 209:           FBB = nullptr;
 210:           Cond.push_back(MachineOperand::CreateImm(BranchCode));
 211: 
 212:           // Otherwise preserve TBB, FBB and Cond as requested
 213:         } else {
 214:           TBB = CondBranchTarget;
 215:           FBB = UncondBranch.second;
 216:           Cond.push_back(MachineOperand::CreateImm(BranchCode));
 217:         }
 218: 
 219:         UncondBranch = {MBB.rend(), nullptr};
 220:         continue;
 221:       }
 222: 
 223:       TBB = CondBranchTarget;
 224:       FBB = nullptr;
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:       Cond.push_back(MachineOperand::CreateImm(BranchCode));
 226: 
 227:       continue;
 228:     }
 229: 
 230:     // Handle subsequent conditional branches. Only handle the case where all
 231:     // conditional branches branch to the same destination and their condition
 232:     // opcodes fit one of the special multi-branch idioms.
 233:     assert(Cond.size() == 1);
 234:     assert(TBB);
 235: 
 236:     // If the conditions are the same, we can leave them alone.
 237:     auto OldBranchCode = static_cast<M68k::CondCode>(Cond[0].getImm());
 238:     if (!iter->getOperand(0).isMBB())
 239:       return true;
 240:     auto NewTBB = iter->getOperand(0).getMBB();
 241:     if (OldBranchCode == BranchCode && TBB == NewTBB)
 242:       continue;
 243: 
 244:     // If they differ we cannot do much here.
 245:     return true;
 246:   }
 247: 
 248:   return false;
 249: }
 250: 
 251: bool M68kInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 252:                                   MachineBasicBlock *&TBB,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 253-280 / 第 253-280 行
```cpp
 253:                                   MachineBasicBlock *&FBB,
 254:                                   SmallVectorImpl<MachineOperand> &Cond,
 255:                                   bool AllowModify) const {
 256:   return AnalyzeBranchImpl(MBB, TBB, FBB, Cond, AllowModify);
 257: }
 258: 
 259: unsigned M68kInstrInfo::removeBranch(MachineBasicBlock &MBB,
 260:                                      int *BytesRemoved) const {
 261:   assert(!BytesRemoved && "code size not handled");
 262: 
 263:   MachineBasicBlock::iterator I = MBB.end();
 264:   unsigned Count = 0;
 265: 
 266:   while (I != MBB.begin()) {
 267:     --I;
 268:     if (I->isDebugValue())
 269:       continue;
 270:     if (I->getOpcode() != M68k::BRA8 &&
 271:         getCondFromBranchOpc(I->getOpcode()) == M68k::COND_INVALID)
 272:       break;
 273:     // Remove the branch.
 274:     I->eraseFromParent();
 275:     I = MBB.end();
 276:     ++Count;
 277:   }
 278: 
 279:   return Count;
 280: }
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::removeBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::removeBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-308 / 第 281-308 行
```cpp
 281: 
 282: unsigned M68kInstrInfo::insertBranch(
 283:     MachineBasicBlock &MBB, MachineBasicBlock *TBB, MachineBasicBlock *FBB,
 284:     ArrayRef<MachineOperand> Cond, const DebugLoc &DL, int *BytesAdded) const {
 285:   // Shouldn't be a fall through.
 286:   assert(TBB && "InsertBranch must not be told to insert a fallthrough");
 287:   assert((Cond.size() == 1 || Cond.size() == 0) &&
 288:          "M68k branch conditions have one component!");
 289:   assert(!BytesAdded && "code size not handled");
 290: 
 291:   if (Cond.empty()) {
 292:     // Unconditional branch?
 293:     assert(!FBB && "Unconditional branch with multiple successors!");
 294:     BuildMI(&MBB, DL, get(M68k::BRA8)).addMBB(TBB);
 295:     return 1;
 296:   }
 297: 
 298:   // If FBB is null, it is implied to be a fall-through block.
 299:   bool FallThru = FBB == nullptr;
 300: 
 301:   // Conditional branch.
 302:   unsigned Count = 0;
 303:   M68k::CondCode CC = (M68k::CondCode)Cond[0].getImm();
 304:   unsigned Opc = GetCondBranchFromCond(CC);
 305:   BuildMI(&MBB, DL, get(Opc)).addMBB(TBB);
 306:   ++Count;
 307:   if (!FallThru) {
 308:     // Two-way Conditional branch. Insert the second branch.
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::insertBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::insertBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 309-336 / 第 309-336 行
```cpp
 309:     BuildMI(&MBB, DL, get(M68k::BRA8)).addMBB(FBB);
 310:     ++Count;
 311:   }
 312:   return Count;
 313: }
 314: 
 315: void M68kInstrInfo::AddSExt(MachineBasicBlock &MBB,
 316:                             MachineBasicBlock::iterator I, DebugLoc DL,
 317:                             unsigned Reg, MVT From, MVT To) const {
 318:   if (From == MVT::i8) {
 319:     unsigned R = Reg;
 320:     // EXT16 requires i16 register
 321:     if (To == MVT::i32) {
 322:       R = RI.getSubReg(Reg, M68k::MxSubRegIndex16Lo);
 323:       assert(R && "No viable SUB register available");
 324:     }
 325:     BuildMI(MBB, I, DL, get(M68k::EXT16), R).addReg(R);
 326:   }
 327: 
 328:   if (To == MVT::i32)
 329:     BuildMI(MBB, I, DL, get(M68k::EXT32), Reg).addReg(Reg);
 330: }
 331: 
 332: void M68kInstrInfo::AddZExt(MachineBasicBlock &MBB,
 333:                             MachineBasicBlock::iterator I, DebugLoc DL,
 334:                             unsigned Reg, MVT From, MVT To) const {
 335: 
 336:   unsigned Mask, And;
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::AddSExt`, `M68kInstrInfo::AddZExt`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::AddSExt`, `M68kInstrInfo::AddZExt` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 337-364 / 第 337-364 行
```cpp
 337:   if (From == MVT::i8)
 338:     Mask = 0xFF;
 339:   else
 340:     Mask = 0xFFFF;
 341: 
 342:   if (To == MVT::i16)
 343:     And = M68k::AND16di;
 344:   else // i32
 345:     And = M68k::AND32di;
 346: 
 347:   // TODO use xor r,r to decrease size
 348:   BuildMI(MBB, I, DL, get(And), Reg).addReg(Reg).addImm(Mask);
 349: }
 350: 
 351: // Convert MOVI to the appropriate instruction (sequence) for setting
 352: // the register to an immediate value.
 353: bool M68kInstrInfo::ExpandMOVI(MachineInstrBuilder &MIB, MVT MVTSize) const {
 354:   Register Reg = MIB->getOperand(0).getReg();
 355:   int64_t Imm = MIB->getOperand(1).getImm();
 356:   bool IsAddressReg = false;
 357: 
 358:   const auto *DR32 = RI.getRegClass(M68k::DR32RegClassID);
 359:   const auto *AR32 = RI.getRegClass(M68k::AR32RegClassID);
 360:   const auto *AR16 = RI.getRegClass(M68k::AR16RegClassID);
 361: 
 362:   if (AR16->contains(Reg) || AR32->contains(Reg))
 363:     IsAddressReg = true;
 364: 
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::ExpandMOVI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::ExpandMOVI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   // We need to assign to the full register to make IV happy
 366:   Register SReg =
 367:       MVTSize == MVT::i32
 368:           ? Reg
 369:           : Register(RI.getMatchingMegaReg(Reg, IsAddressReg ? AR32 : DR32));
 370:   assert(SReg && "No viable MEGA register available");
 371: 
 372:   LLVM_DEBUG(dbgs() << "Expand " << *MIB.getInstr() << " to ");
 373: 
 374:   // Sign extention doesn't matter if we only use the bottom 8 bits
 375:   if (MVTSize == MVT::i8 || (!IsAddressReg && Imm >= -128 && Imm <= 127)) {
 376:     LLVM_DEBUG(dbgs() << "MOVEQ\n");
 377: 
 378:     MIB->setDesc(get(M68k::MOVQ));
 379:     MIB->getOperand(0).setReg(SReg);
 380: 
 381:     // Counter the effects of sign-extension with a bitwise not.
 382:     // This is only faster and smaller for 32 bit values.
 383:   } else if (DR32->contains(Reg) && isUInt<8>(Imm)) {
 384:     LLVM_DEBUG(dbgs() << "MOVEQ and NOT\n");
 385: 
 386:     MachineBasicBlock &MBB = *MIB->getParent();
 387:     DebugLoc DL = MIB->getDebugLoc();
 388: 
 389:     unsigned SubReg = RI.getSubReg(Reg, M68k::MxSubRegIndex8Lo);
 390:     assert(SubReg && "No viable SUB register available");
 391: 
 392:     BuildMI(MBB, MIB.getInstr(), DL, get(M68k::MOVQ), SReg).addImm(~Imm & 0xFF);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 393-420 / 第 393-420 行
```cpp
 393:     BuildMI(MBB, MIB.getInstr(), DL, get(M68k::NOT8d), SubReg).addReg(SubReg);
 394: 
 395:     MIB->removeFromParent();
 396: 
 397:     // Special case for setting address register to NULL (0)
 398:   } else if (IsAddressReg && Imm == 0) {
 399:     LLVM_DEBUG(dbgs() << "SUBA\n");
 400: 
 401:     MachineBasicBlock &MBB = *MIB->getParent();
 402:     DebugLoc DL = MIB->getDebugLoc();
 403: 
 404:     BuildMI(MBB, MIB.getInstr(), DL, get(M68k::SUB32ar), SReg)
 405:         .addReg(SReg, RegState::Undef)
 406:         .addReg(SReg, RegState::Undef);
 407: 
 408:     MIB->removeFromParent();
 409: 
 410:     // movea.w implicitly sign extends to the full register width,
 411:     // so exploit that if the immediate fits in the correct range.
 412:     //
 413:     // TODO: use lea imm.w, %an for further constants when 16-bit
 414:     // absolute addressing is implemented.
 415:   } else if (AR32->contains(Reg) && isUInt<16>(Imm)) {
 416:     LLVM_DEBUG(dbgs() << "MOVEA w/ implicit extend\n");
 417: 
 418:     unsigned SubReg = RI.getSubReg(Reg, M68k::MxSubRegIndex16Lo);
 419:     assert(SubReg && "No viable SUB register available");
 420: 
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 421-448 / 第 421-448 行
```cpp
 421:     MIB->setDesc(get(M68k::MOV16ai));
 422:     MIB->getOperand(0).setReg(SubReg);
 423: 
 424:     // Fall back to a move with immediate
 425:   } else {
 426:     LLVM_DEBUG(dbgs() << "MOVE\n");
 427:     MIB->setDesc(get(MVTSize == MVT::i16 ? M68k::MOV16ri : M68k::MOV32ri));
 428:   }
 429: 
 430:   return true;
 431: }
 432: 
 433: bool M68kInstrInfo::ExpandMOVX_RR(MachineInstrBuilder &MIB, MVT MVTDst,
 434:                                   MVT MVTSrc) const {
 435:   unsigned Move = MVTDst == MVT::i16 ? M68k::MOV16rr : M68k::MOV32rr;
 436:   Register Dst = MIB->getOperand(0).getReg();
 437:   Register Src = MIB->getOperand(1).getReg();
 438: 
 439:   assert(Dst != Src && "You cannot use the same Regs with MOVX_RR");
 440: 
 441:   const auto &TRI = getRegisterInfo();
 442: 
 443:   const auto *RCDst = TRI.getMaximalPhysRegClass(Dst, MVTDst);
 444:   const auto *RCSrc = TRI.getMaximalPhysRegClass(Src, MVTSrc);
 445: 
 446:   assert(RCDst && RCSrc && "Wrong use of MOVX_RR");
 447:   assert(RCDst != RCSrc && "You cannot use the same Reg Classes with MOVX_RR");
 448:   (void)RCSrc;
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::ExpandMOVX_RR`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::ExpandMOVX_RR` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 449-476 / 第 449-476 行
```cpp
 449: 
 450:   // We need to find the super source register that matches the size of Dst
 451:   unsigned SSrc = RI.getMatchingMegaReg(Src, RCDst);
 452:   assert(SSrc && "No viable MEGA register available");
 453: 
 454:   DebugLoc DL = MIB->getDebugLoc();
 455: 
 456:   // If it happens to that super source register is the destination register
 457:   // we do nothing
 458:   if (Dst == SSrc) {
 459:     LLVM_DEBUG(dbgs() << "Remove " << *MIB.getInstr() << '\n');
 460:     MIB->eraseFromParent();
 461:   } else { // otherwise we need to MOV
 462:     LLVM_DEBUG(dbgs() << "Expand " << *MIB.getInstr() << " to MOV\n");
 463:     MIB->setDesc(get(Move));
 464:     MIB->getOperand(1).setReg(SSrc);
 465:   }
 466: 
 467:   return true;
 468: }
 469: 
 470: /// Expand SExt MOVE pseudos into a MOV and a EXT if the operands are two
 471: /// different registers or just EXT if it is the same register
 472: bool M68kInstrInfo::ExpandMOVSZX_RR(MachineInstrBuilder &MIB, bool IsSigned,
 473:                                     MVT MVTDst, MVT MVTSrc) const {
 474:   LLVM_DEBUG(dbgs() << "Expand " << *MIB.getInstr() << " to ");
 475: 
 476:   unsigned Move;
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::ExpandMOVSZX_RR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::ExpandMOVSZX_RR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 477-504 / 第 477-504 行
```cpp
 477: 
 478:   if (MVTDst == MVT::i16)
 479:     Move = M68k::MOV16rr;
 480:   else // i32
 481:     Move = M68k::MOV32rr;
 482: 
 483:   Register Dst = MIB->getOperand(0).getReg();
 484:   Register Src = MIB->getOperand(1).getReg();
 485: 
 486:   assert(Dst != Src && "You cannot use the same Regs with MOVSX_RR");
 487: 
 488:   const auto &TRI = getRegisterInfo();
 489: 
 490:   const auto *RCDst = TRI.getMaximalPhysRegClass(Dst, MVTDst);
 491:   const auto *RCSrc = TRI.getMaximalPhysRegClass(Src, MVTSrc);
 492: 
 493:   assert(RCDst && RCSrc && "Wrong use of MOVSX_RR");
 494:   assert(RCDst != RCSrc && "You cannot use the same Reg Classes with MOVSX_RR");
 495:   (void)RCSrc;
 496: 
 497:   // We need to find the super source register that matches the size of Dst
 498:   unsigned SSrc = RI.getMatchingMegaReg(Src, RCDst);
 499:   assert(SSrc && "No viable MEGA register available");
 500: 
 501:   MachineBasicBlock &MBB = *MIB->getParent();
 502:   DebugLoc DL = MIB->getDebugLoc();
 503: 
 504:   if (Dst != SSrc) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 505-532 / 第 505-532 行
```cpp
 505:     LLVM_DEBUG(dbgs() << "Move and " << '\n');
 506:     BuildMI(MBB, MIB.getInstr(), DL, get(Move), Dst).addReg(SSrc);
 507:   }
 508: 
 509:   if (IsSigned) {
 510:     LLVM_DEBUG(dbgs() << "Sign Extend" << '\n');
 511:     AddSExt(MBB, MIB.getInstr(), DL, Dst, MVTSrc, MVTDst);
 512:   } else {
 513:     LLVM_DEBUG(dbgs() << "Zero Extend" << '\n');
 514:     AddZExt(MBB, MIB.getInstr(), DL, Dst, MVTSrc, MVTDst);
 515:   }
 516: 
 517:   MIB->eraseFromParent();
 518: 
 519:   return true;
 520: }
 521: 
 522: bool M68kInstrInfo::ExpandMOVSZX_RM(MachineInstrBuilder &MIB, bool IsSigned,
 523:                                     const MCInstrDesc &Desc, MVT MVTDst,
 524:                                     MVT MVTSrc) const {
 525:   LLVM_DEBUG(dbgs() << "Expand " << *MIB.getInstr() << " to LOAD and ");
 526: 
 527:   Register Dst = MIB->getOperand(0).getReg();
 528: 
 529:   // We need the subreg of Dst to make instruction verifier happy because the
 530:   // real machine instruction consumes and produces values of the same size and
 531:   // the registers the will be used here fall into different classes and this
 532:   // makes IV cry. We could use a bigger operation, but this will put some
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::ExpandMOVSZX_RM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::ExpandMOVSZX_RM` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:   // pressure on cache and memory, so no.
 534:   unsigned SubDst =
 535:       RI.getSubReg(Dst, MVTSrc == MVT::i8 ? M68k::MxSubRegIndex8Lo
 536:                                           : M68k::MxSubRegIndex16Lo);
 537:   assert(SubDst && "No viable SUB register available");
 538: 
 539:   // Make this a plain move
 540:   MIB->setDesc(Desc);
 541:   MIB->getOperand(0).setReg(SubDst);
 542: 
 543:   MachineBasicBlock::iterator I = MIB.getInstr();
 544:   I++;
 545:   MachineBasicBlock &MBB = *MIB->getParent();
 546:   DebugLoc DL = MIB->getDebugLoc();
 547: 
 548:   if (IsSigned) {
 549:     LLVM_DEBUG(dbgs() << "Sign Extend" << '\n');
 550:     AddSExt(MBB, I, DL, Dst, MVTSrc, MVTDst);
 551:   } else {
 552:     LLVM_DEBUG(dbgs() << "Zero Extend" << '\n');
 553:     AddZExt(MBB, I, DL, Dst, MVTSrc, MVTDst);
 554:   }
 555: 
 556:   return true;
 557: }
 558: 
 559: bool M68kInstrInfo::ExpandPUSH_POP(MachineInstrBuilder &MIB,
 560:                                    const MCInstrDesc &Desc, bool IsPush) const {
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::ExpandPUSH_POP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::ExpandPUSH_POP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-588 / 第 561-588 行
```cpp
 561:   MachineBasicBlock::iterator I = MIB.getInstr();
 562:   I++;
 563:   MachineBasicBlock &MBB = *MIB->getParent();
 564:   MachineOperand MO = MIB->getOperand(0);
 565:   DebugLoc DL = MIB->getDebugLoc();
 566:   if (IsPush)
 567:     BuildMI(MBB, I, DL, Desc).addReg(RI.getStackRegister()).add(MO);
 568:   else
 569:     BuildMI(MBB, I, DL, Desc, MO.getReg()).addReg(RI.getStackRegister());
 570: 
 571:   MIB->eraseFromParent();
 572:   return true;
 573: }
 574: 
 575: bool M68kInstrInfo::ExpandMOVEM(MachineInstrBuilder &MIB,
 576:                                 const MCInstrDesc &Desc, bool IsRM) const {
 577:   int Reg = 0, Offset = 0, Base = 0;
 578:   auto DL = MIB->getDebugLoc();
 579:   auto MI = MIB.getInstr();
 580:   auto &MBB = *MIB->getParent();
 581: 
 582:   if (IsRM) {
 583:     Reg = MIB->getOperand(0).getReg();
 584:     Offset = MIB->getOperand(1).getImm();
 585:     Base = MIB->getOperand(2).getReg();
 586:   } else {
 587:     Offset = MIB->getOperand(0).getImm();
 588:     Base = MIB->getOperand(1).getReg();
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::ExpandMOVEM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::ExpandMOVEM` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```cpp
 589:     Reg = MIB->getOperand(2).getReg();
 590:   }
 591: 
 592:   unsigned Mask = 1 << RI.getSpillRegisterOrder(Reg);
 593:   if (IsRM) {
 594:     BuildMI(MBB, MI, DL, Desc)
 595:         .addImm(Mask)
 596:         .addImm(Offset)
 597:         .addReg(Base)
 598:         .addReg(Reg, RegState::ImplicitDefine)
 599:         .copyImplicitOps(*MIB);
 600:   } else {
 601:     BuildMI(MBB, MI, DL, Desc)
 602:         .addImm(Offset)
 603:         .addReg(Base)
 604:         .addImm(Mask)
 605:         .addReg(Reg, RegState::Implicit)
 606:         .copyImplicitOps(*MIB);
 607:   }
 608: 
 609:   MIB->eraseFromParent();
 610: 
 611:   return true;
 612: }
 613: 
 614: /// Expand a single-def pseudo instruction to a two-addr
 615: /// instruction with two undef reads of the register being defined.
 616: /// This is used for mapping:
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```cpp
 617: ///   %d0 = SETCS_C32d
 618: /// to:
 619: ///   %d0 = SUBX32dd %d0<undef>, %d0<undef>
 620: ///
 621: static bool Expand2AddrUndef(MachineInstrBuilder &MIB,
 622:                              const MCInstrDesc &Desc) {
 623:   assert(Desc.getNumOperands() == 3 && "Expected two-addr instruction.");
 624:   Register Reg = MIB->getOperand(0).getReg();
 625:   MIB->setDesc(Desc);
 626: 
 627:   // MachineInstr::addOperand() will insert explicit operands before any
 628:   // implicit operands.
 629:   MIB.addReg(Reg, RegState::Undef).addReg(Reg, RegState::Undef);
 630:   // But we don't trust that.
 631:   assert(MIB->getOperand(1).getReg() == Reg &&
 632:          MIB->getOperand(2).getReg() == Reg && "Misplaced operand");
 633:   return true;
 634: }
 635: 
 636: bool M68kInstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
 637:   MachineInstrBuilder MIB(*MI.getParent()->getParent(), MI);
 638:   switch (MI.getOpcode()) {
 639:   case M68k::PUSH8d:
 640:     return ExpandPUSH_POP(MIB, get(M68k::MOV8ed), true);
 641:   case M68k::PUSH16d:
 642:     return ExpandPUSH_POP(MIB, get(M68k::MOV16er), true);
 643:   case M68k::PUSH32r:
 644:     return ExpandPUSH_POP(MIB, get(M68k::MOV32er), true);
```
- **EN**: The range implements or declares functions including `Expand2AddrUndef`, `M68kInstrInfo::expandPostRAPseudo`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `Expand2AddrUndef`, `M68kInstrInfo::expandPostRAPseudo` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 645-672 / 第 645-672 行
```cpp
 645: 
 646:   case M68k::POP8d:
 647:     return ExpandPUSH_POP(MIB, get(M68k::MOV8do), false);
 648:   case M68k::POP16d:
 649:     return ExpandPUSH_POP(MIB, get(M68k::MOV16ro), false);
 650:   case M68k::POP32r:
 651:     return ExpandPUSH_POP(MIB, get(M68k::MOV32ro), false);
 652: 
 653:   case M68k::SETCS_C8d:
 654:     return Expand2AddrUndef(MIB, get(M68k::SUBX8dd));
 655:   case M68k::SETCS_C16d:
 656:     return Expand2AddrUndef(MIB, get(M68k::SUBX16dd));
 657:   case M68k::SETCS_C32d:
 658:     return Expand2AddrUndef(MIB, get(M68k::SUBX32dd));
 659:   }
 660:   return false;
 661: }
 662: 
 663: bool M68kInstrInfo::isPCRelRegisterOperandLegal(
 664:     const MachineOperand &MO) const {
 665:   assert(MO.isReg());
 666: 
 667:   // Check whether this MO belongs to an instruction with addressing mode 'k',
 668:   // Refer to TargetInstrInfo.h for more information about this function.
 669: 
 670:   const MachineInstr *MI = MO.getParent();
 671:   const unsigned NameIndices = M68kInstrNameIndices[MI->getOpcode()];
 672:   StringRef InstrName(&M68kInstrNameData[NameIndices]);
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::isPCRelRegisterOperandLegal`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::isPCRelRegisterOperandLegal` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 673-700 / 第 673-700 行
```cpp
 673:   const unsigned OperandNo = MO.getOperandNo();
 674: 
 675:   // If this machine operand is the 2nd operand, then check
 676:   // whether the instruction has destination addressing mode 'k'.
 677:   if (OperandNo == 1)
 678:     return Regex("[A-Z]+(8|16|32)k[a-z](_TC)?$").match(InstrName);
 679: 
 680:   // If this machine operand is the last one, then check
 681:   // whether the instruction has source addressing mode 'k'.
 682:   if (OperandNo == MI->getNumExplicitOperands() - 1)
 683:     return Regex("[A-Z]+(8|16|32)[a-z]k(_TC)?$").match(InstrName);
 684: 
 685:   return false;
 686: }
 687: 
 688: void M68kInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
 689:                                 MachineBasicBlock::iterator MI,
 690:                                 const DebugLoc &DL, Register DstReg,
 691:                                 Register SrcReg, bool KillSrc,
 692:                                 bool RenamableDest, bool RenamableSrc) const {
 693:   const auto &Subtarget = MBB.getParent()->getSubtarget<M68kSubtarget>();
 694:   unsigned Opc = 0;
 695: 
 696:   // First deal with the normal symmetric copies.
 697:   if (M68k::XR32RegClass.contains(DstReg, SrcReg))
 698:     Opc = M68k::MOV32rr;
 699:   else if (M68k::XR16RegClass.contains(DstReg, SrcReg))
 700:     Opc = M68k::MOV16rr;
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::copyPhysReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::copyPhysReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 701-728 / 第 701-728 行
```cpp
 701:   else if (M68k::DR8RegClass.contains(DstReg, SrcReg))
 702:     Opc = M68k::MOV8dd;
 703: 
 704:   if (Opc) {
 705:     BuildMI(MBB, MI, DL, get(Opc), DstReg)
 706:         .addReg(SrcReg, getKillRegState(KillSrc));
 707:     return;
 708:   }
 709: 
 710:   // Now deal with asymmetrically sized copies. The cases that follow are upcast
 711:   // moves.
 712:   //
 713:   // NOTE
 714:   // These moves are not aware of type nature of these values and thus
 715:   // won't do any SExt or ZExt and upper bits will basically contain garbage.
 716:   MachineInstrBuilder MIB(*MBB.getParent(), MI);
 717:   if (M68k::DR8RegClass.contains(SrcReg)) {
 718:     if (M68k::XR16RegClass.contains(DstReg))
 719:       Opc = M68k::MOVXd16d8;
 720:     else if (M68k::XR32RegClass.contains(DstReg))
 721:       Opc = M68k::MOVXd32d8;
 722:   } else if (M68k::XR16RegClass.contains(SrcReg) &&
 723:              M68k::XR32RegClass.contains(DstReg))
 724:     Opc = M68k::MOVXd32d16;
 725: 
 726:   if (Opc) {
 727:     BuildMI(MBB, MI, DL, get(Opc), DstReg)
 728:         .addReg(SrcReg, getKillRegState(KillSrc));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 729-756 / 第 729-756 行
```cpp
 729:     return;
 730:   }
 731: 
 732:   bool FromCCR = SrcReg == M68k::CCR;
 733:   bool FromSR = SrcReg == M68k::SR;
 734:   bool ToCCR = DstReg == M68k::CCR;
 735:   bool ToSR = DstReg == M68k::SR;
 736: 
 737:   if (FromCCR) {
 738:     Opc = M68k::MOV16dc;
 739:     if (!Subtarget.atLeastM68010()) {
 740:       Opc = M68k::MOV16ds;
 741:       SrcReg = M68k::SR;
 742:     }
 743:     if (!M68k::DR8RegClass.contains(DstReg) &&
 744:         !M68k::DR16RegClass.contains(DstReg) &&
 745:         !M68k::DR32RegClass.contains(DstReg)) {
 746:       LLVM_DEBUG(dbgs() << "Cannot copy CCR to " << RI.getName(DstReg) << '\n');
 747:       llvm_unreachable("Invalid register for MOVE from CCR");
 748:     }
 749:   } else if (ToCCR) {
 750:     Opc = M68k::MOV16cd;
 751:     if (M68k::DR8RegClass.contains(SrcReg)) {
 752:       // Promote used register to the next class
 753:       SrcReg = getRegisterInfo().getMatchingSuperReg(
 754:           SrcReg, M68k::MxSubRegIndex8Lo, &M68k::DR16RegClass);
 755:     } else if (!M68k::DR16RegClass.contains(SrcReg) &&
 756:                !M68k::DR32RegClass.contains(SrcReg)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 757-784 / 第 757-784 行
```cpp
 757:       LLVM_DEBUG(dbgs() << "Cannot copy " << RI.getName(SrcReg) << " to CCR\n");
 758:       llvm_unreachable("Invalid register for MOVE to CCR");
 759:     }
 760:   } else if (FromSR || ToSR) {
 761:     llvm_unreachable("Cannot emit SR copy instruction");
 762:   }
 763: 
 764:   if (Opc) {
 765:     BuildMI(MBB, MI, DL, get(Opc), DstReg)
 766:         .addReg(SrcReg, getKillRegState(KillSrc));
 767:     return;
 768:   }
 769: 
 770:   LLVM_DEBUG(dbgs() << "Cannot copy " << RI.getName(SrcReg) << " to "
 771:                     << RI.getName(DstReg) << '\n');
 772:   llvm_unreachable("Cannot emit physreg copy instruction");
 773: }
 774: 
 775: namespace {
 776: unsigned getLoadStoreRegOpcode(unsigned Reg, const TargetRegisterClass *RC,
 777:                                const TargetRegisterInfo *TRI,
 778:                                const M68kSubtarget &STI, bool load) {
 779:   switch (TRI->getSpillSize(*RC)) {
 780:   default:
 781:     LLVM_DEBUG(
 782:         dbgs() << "Cannot determine appropriate opcode for load/store to/from "
 783:                << TRI->getName(Reg) << " of class " << TRI->getRegClassName(RC)
 784:                << " with spill size " << TRI->getSpillSize(*RC) << '\n');
```
- **EN**: The range implements or declares functions including `getLoadStoreRegOpcode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getLoadStoreRegOpcode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 785-812 / 第 785-812 行
```cpp
 785:     llvm_unreachable("Unknown spill size");
 786:   case 2:
 787:     if (M68k::XR16RegClass.hasSubClassEq(RC))
 788:       return load ? M68k::MOVM16mp_P : M68k::MOVM16pm_P;
 789:     if (M68k::DR8RegClass.hasSubClassEq(RC))
 790:       return load ? M68k::MOVM16mp_P : M68k::MOVM16pm_P;
 791:     if (M68k::CCRCRegClass.hasSubClassEq(RC))
 792:       return load ? M68k::MOVM16mp_P : M68k::MOVM16pm_P;
 793:     llvm_unreachable("Unknown 2-byte regclass");
 794:   case 4:
 795:     if (M68k::XR32RegClass.hasSubClassEq(RC))
 796:       return load ? M68k::MOVM32mp_P : M68k::MOVM32pm_P;
 797:     llvm_unreachable("Unknown 4-byte regclass");
 798:   }
 799: }
 800: 
 801: unsigned getStoreRegOpcode(unsigned SrcReg, const TargetRegisterClass *RC,
 802:                            const TargetRegisterInfo *TRI,
 803:                            const M68kSubtarget &STI) {
 804:   return getLoadStoreRegOpcode(SrcReg, RC, TRI, STI, false);
 805: }
 806: 
 807: unsigned getLoadRegOpcode(unsigned DstReg, const TargetRegisterClass *RC,
 808:                           const TargetRegisterInfo *TRI,
 809:                           const M68kSubtarget &STI) {
 810:   return getLoadStoreRegOpcode(DstReg, RC, TRI, STI, true);
 811: }
 812: } // end anonymous namespace
```
- **EN**: The range implements or declares functions including `getStoreRegOpcode`, `getLoadRegOpcode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getStoreRegOpcode`, `getLoadRegOpcode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 813-840 / 第 813-840 行
```cpp
 813: 
 814: bool M68kInstrInfo::getStackSlotRange(const TargetRegisterClass *RC,
 815:                                       unsigned SubIdx, unsigned &Size,
 816:                                       unsigned &Offset,
 817:                                       const MachineFunction &MF) const {
 818:   // The slot size must be the maximum size so we can easily use MOVEM.L
 819:   Size = 4;
 820:   Offset = 0;
 821:   return true;
 822: }
 823: 
 824: void M68kInstrInfo::storeRegToStackSlot(
 825:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI, Register SrcReg,
 826:     bool IsKill, int FrameIndex, const TargetRegisterClass *RC, Register VReg,
 827:     MachineInstr::MIFlag Flags) const {
 828:   const MachineFrameInfo &MFI = MBB.getParent()->getFrameInfo();
 829:   assert(MFI.getObjectSize(FrameIndex) >= TRI.getSpillSize(*RC) &&
 830:          "Stack slot is too small to store");
 831:   (void)MFI;
 832: 
 833:   unsigned Opc = getStoreRegOpcode(SrcReg, RC, &TRI, Subtarget);
 834:   DebugLoc DL = MBB.findDebugLoc(MI);
 835:   // (0,FrameIndex) <- $reg
 836:   M68k::addFrameReference(BuildMI(MBB, MI, DL, get(Opc)), FrameIndex)
 837:       .addReg(SrcReg, getKillRegState(IsKill));
 838: }
 839: 
 840: void M68kInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::getStackSlotRange`, `M68kInstrInfo::storeRegToStackSlot`, `M68k::addFrameReference`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::getStackSlotRange`, `M68kInstrInfo::storeRegToStackSlot`, `M68k::addFrameReference` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 841-868 / 第 841-868 行
```cpp
 841:                                          MachineBasicBlock::iterator MI,
 842:                                          Register DstReg, int FrameIndex,
 843:                                          const TargetRegisterClass *RC,
 844:                                          Register VReg, unsigned SubReg,
 845:                                          MachineInstr::MIFlag Flags) const {
 846:   const MachineFrameInfo &MFI = MBB.getParent()->getFrameInfo();
 847:   assert(MFI.getObjectSize(FrameIndex) >= TRI.getSpillSize(*RC) &&
 848:          "Stack slot is too small to load");
 849:   (void)MFI;
 850: 
 851:   unsigned Opc = getLoadRegOpcode(DstReg, RC, &TRI, Subtarget);
 852:   DebugLoc DL = MBB.findDebugLoc(MI);
 853:   M68k::addFrameReference(BuildMI(MBB, MI, DL, get(Opc), DstReg), FrameIndex);
 854: }
 855: 
 856: /// Return a virtual register initialized with the global base register
 857: /// value. Output instructions required to initialize the register in the
 858: /// function entry block, if necessary.
 859: ///
 860: /// TODO Move this function to M68kMachineFunctionInfo.
 861: unsigned M68kInstrInfo::getGlobalBaseReg(MachineFunction *MF) const {
 862:   M68kMachineFunctionInfo *MxFI = MF->getInfo<M68kMachineFunctionInfo>();
 863:   unsigned GlobalBaseReg = MxFI->getGlobalBaseReg();
 864:   if (GlobalBaseReg != 0)
 865:     return GlobalBaseReg;
 866: 
 867:   // Create the register. The code to initialize it is inserted later,
 868:   // by the M68kGlobalBaseReg pass (below).
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::getGlobalBaseReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::getGlobalBaseReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 869-896 / 第 869-896 行
```cpp
 869:   //
 870:   // NOTE
 871:   // Normally M68k uses A5 register as global base pointer but this will
 872:   // create unnecessary spill if we use less then 4 registers in code; since A5
 873:   // is callee-save anyway we could try to allocate caller-save first and if
 874:   // lucky get one, otherwise it does not really matter which callee-save to
 875:   // use.
 876:   MachineRegisterInfo &RegInfo = MF->getRegInfo();
 877:   GlobalBaseReg = RegInfo.createVirtualRegister(&M68k::AR32_NOSPRegClass);
 878:   MxFI->setGlobalBaseReg(GlobalBaseReg);
 879:   return GlobalBaseReg;
 880: }
 881: 
 882: std::pair<unsigned, unsigned>
 883: M68kInstrInfo::decomposeMachineOperandsTargetFlags(unsigned TF) const {
 884:   return std::make_pair(TF, 0u);
 885: }
 886: 
 887: ArrayRef<std::pair<unsigned, const char *>>
 888: M68kInstrInfo::getSerializableDirectMachineOperandTargetFlags() const {
 889:   using namespace M68kII;
 890:   static const std::pair<unsigned, const char *> TargetFlags[] = {
 891:       {MO_ABSOLUTE_ADDRESS, "m68k-absolute"},
 892:       {MO_PC_RELATIVE_ADDRESS, "m68k-pcrel"},
 893:       {MO_GOT, "m68k-got"},
 894:       {MO_GOTOFF, "m68k-gotoff"},
 895:       {MO_GOTPCREL, "m68k-gotpcrel"},
 896:       {MO_PLT, "m68k-plt"},
```
- **EN**: The range implements or declares functions including `M68kInstrInfo::decomposeMachineOperandsTargetFlags`, `M68kInstrInfo::getSerializableDirectMachineOperandTargetFlags`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kInstrInfo::decomposeMachineOperandsTargetFlags`, `M68kInstrInfo::getSerializableDirectMachineOperandTargetFlags` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 897-924 / 第 897-924 行
```cpp
 897:       {MO_TLSGD, "m68k-tlsgd"},
 898:       {MO_TLSLD, "m68k-tlsld"},
 899:       {MO_TLSLDM, "m68k-tlsldm"},
 900:       {MO_TLSIE, "m68k-tlsie"},
 901:       {MO_TLSLE, "m68k-tlsle"}};
 902:   return ArrayRef(TargetFlags);
 903: }
 904: 
 905: #undef DEBUG_TYPE
 906: #define DEBUG_TYPE "m68k-create-global-base-reg"
 907: 
 908: #define PASS_NAME "M68k PIC Global Base Reg Initialization"
 909: 
 910: namespace {
 911: /// This initializes the PIC global base register
 912: struct M68kGlobalBaseReg : public MachineFunctionPass {
 913:   static char ID;
 914:   M68kGlobalBaseReg() : MachineFunctionPass(ID) {}
 915: 
 916:   bool runOnMachineFunction(MachineFunction &MF) override {
 917:     const M68kSubtarget &STI = MF.getSubtarget<M68kSubtarget>();
 918:     M68kMachineFunctionInfo *MxFI = MF.getInfo<M68kMachineFunctionInfo>();
 919: 
 920:     unsigned GlobalBaseReg = MxFI->getGlobalBaseReg();
 921: 
 922:     // If we didn't need a GlobalBaseReg, don't insert code.
 923:     if (GlobalBaseReg == 0)
 924:       return false;
```
- **EN**: It introduces interface types such as `M68kGlobalBaseReg`, shaping how other backend components interact with this file. The range implements or declares functions including `M68kGlobalBaseReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `M68kGlobalBaseReg` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `M68kGlobalBaseReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 925-951 / 第 925-951 行
```cpp
 925: 
 926:     // Insert the set of GlobalBaseReg into the first MBB of the function
 927:     MachineBasicBlock &FirstMBB = MF.front();
 928:     MachineBasicBlock::iterator MBBI = FirstMBB.begin();
 929:     DebugLoc DL = FirstMBB.findDebugLoc(MBBI);
 930:     const M68kInstrInfo *TII = STI.getInstrInfo();
 931: 
 932:     // Generate lea (__GLOBAL_OFFSET_TABLE_,%PC), %A5
 933:     BuildMI(FirstMBB, MBBI, DL, TII->get(M68k::LEA32q), GlobalBaseReg)
 934:         .addExternalSymbol("_GLOBAL_OFFSET_TABLE_", M68kII::MO_GOTPCREL);
 935: 
 936:     return true;
 937:   }
 938: 
 939:   void getAnalysisUsage(AnalysisUsage &AU) const override {
 940:     AU.setPreservesCFG();
 941:     MachineFunctionPass::getAnalysisUsage(AU);
 942:   }
 943: };
 944: char M68kGlobalBaseReg::ID = 0;
 945: } // namespace
 946: 
 947: INITIALIZE_PASS(M68kGlobalBaseReg, DEBUG_TYPE, PASS_NAME, false, false)
 948: 
 949: FunctionPass *llvm::createM68kGlobalBaseRegPass() {
 950:   return new M68kGlobalBaseReg();
 951: }
```
- **EN**: The range implements or declares functions including `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `M68kInstrInfo.h`
- `M68kInstrBuilder.h`
- `M68kMachineFunction.h`
- `M68kTargetMachine.h`
- `MCTargetDesc/M68kMCCodeEmitter.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/ScopeExit.h`
- `llvm/CodeGen/LivePhysRegs.h`
- `llvm/CodeGen/LiveVariables.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineOperand.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGenTypes/MachineValueType.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/Regex.h`
- `...` (2 more include dependencies omitted for brevity / 其余 2 个 include 依赖已省略)
