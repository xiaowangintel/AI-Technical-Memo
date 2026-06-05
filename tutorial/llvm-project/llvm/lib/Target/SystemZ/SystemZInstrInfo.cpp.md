# SystemZInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target instruction information for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责定义或实现目标指令信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: //===-- SystemZInstrInfo.cpp - SystemZ instruction information ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the SystemZ implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "SystemZInstrInfo.h"
  14: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  15: #include "SystemZ.h"
  16: #include "SystemZInstrBuilder.h"
  17: #include "SystemZSubtarget.h"
  18: #include "llvm/ADT/Statistic.h"
  19: #include "llvm/CodeGen/LiveInterval.h"
  20: #include "llvm/CodeGen/LiveIntervals.h"
  21: #include "llvm/CodeGen/LiveRegUnits.h"
  22: #include "llvm/CodeGen/LiveVariables.h"
  23: #include "llvm/CodeGen/MachineBasicBlock.h"
  24: #include "llvm/CodeGen/MachineFrameInfo.h"
  25: #include "llvm/CodeGen/MachineFunction.h"
  26: #include "llvm/CodeGen/MachineInstr.h"
  27: #include "llvm/CodeGen/MachineMemOperand.h"
  28: #include "llvm/CodeGen/MachineOperand.h"
  29: #include "llvm/CodeGen/MachineRegisterInfo.h"
  30: #include "llvm/CodeGen/SlotIndexes.h"
  31: #include "llvm/CodeGen/StackMaps.h"
  32: #include "llvm/CodeGen/TargetInstrInfo.h"
  33: #include "llvm/CodeGen/TargetOpcodes.h"
  34: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  35: #include "llvm/CodeGen/VirtRegMap.h"
  36: #include "llvm/MC/MCInstBuilder.h"
  37: #include "llvm/MC/MCInstrDesc.h"
  38: #include "llvm/MC/MCRegisterInfo.h"
  39: #include "llvm/Support/BranchProbability.h"
  40: #include "llvm/Support/ErrorHandling.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZInstrInfo.h`, `SystemZMCTargetDesc.h`, `SystemZ.h`, `SystemZInstrBuilder.h`, `SystemZSubtarget.h`, `Statistic.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZInstrInfo.h`, `SystemZMCTargetDesc.h`, `SystemZ.h`, `SystemZInstrBuilder.h`, `SystemZSubtarget.h`, `Statistic.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 41-80 / 第 41-80 行
```cpp
  41: #include "llvm/Support/MathExtras.h"
  42: #include "llvm/Target/TargetMachine.h"
  43: #include <cassert>
  44: #include <cstdint>
  45: #include <iterator>
  46: 
  47: using namespace llvm;
  48: 
  49: #define GET_INSTRINFO_CTOR_DTOR
  50: #define GET_INSTRMAP_INFO
  51: #include "SystemZGenInstrInfo.inc"
  52: 
  53: #define DEBUG_TYPE "systemz-II"
  54: 
  55: // Return a mask with Count low bits set.
  56: static uint64_t allOnes(unsigned int Count) {
  57:   return Count == 0 ? 0 : (uint64_t(1) << (Count - 1) << 1) - 1;
  58: }
  59: 
  60: // Pin the vtable to this file.
  61: void SystemZInstrInfo::anchor() {}
  62: 
  63: SystemZInstrInfo::SystemZInstrInfo(const SystemZSubtarget &sti)
  64:     : SystemZGenInstrInfo(sti, RI, -1, -1),
  65:       RI(sti.getSpecialRegisters()->getReturnFunctionAddressRegister(),
  66:          sti.getHwMode()),
  67:       STI(sti) {}
  68: 
  69: // MI is a 128-bit load or store.  Split it into two 64-bit loads or stores,
  70: // each having the opcode given by NewOpcode.
  71: void SystemZInstrInfo::splitMove(MachineBasicBlock::iterator MI,
  72:                                  unsigned NewOpcode) const {
  73:   MachineBasicBlock *MBB = MI->getParent();
  74:   MachineFunction &MF = *MBB->getParent();
  75: 
  76:   // Get two load or store instructions.  Use the original instruction for
  77:   // one of them and create a clone for the other.
  78:   MachineInstr *HighPartMI = MF.CloneMachineInstr(&*MI);
  79:   MachineInstr *LowPartMI = &*MI;
  80:   MBB->insert(LowPartMI, HighPartMI);
```
- **EN**: It imports dependencies such as `MathExtras.h`, `TargetMachine.h`, `cassert`, `cstdint`, `iterator`, `SystemZGenInstrInfo.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `allOnes`, `SystemZInstrInfo::anchor`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `MathExtras.h`, `TargetMachine.h`, `cassert`, `cstdint`, `iterator`, `SystemZGenInstrInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `allOnes`, `SystemZInstrInfo::anchor` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 81-120 / 第 81-120 行
```cpp
  81: 
  82:   // Set up the two 64-bit registers and remember super reg and its flags.
  83:   MachineOperand &HighRegOp = HighPartMI->getOperand(0);
  84:   MachineOperand &LowRegOp = LowPartMI->getOperand(0);
  85:   Register Reg128 = LowRegOp.getReg();
  86:   RegState Reg128Killed = getKillRegState(LowRegOp.isKill());
  87:   RegState Reg128Undef = getUndefRegState(LowRegOp.isUndef());
  88:   HighRegOp.setReg(RI.getSubReg(HighRegOp.getReg(), SystemZ::subreg_h64));
  89:   LowRegOp.setReg(RI.getSubReg(LowRegOp.getReg(), SystemZ::subreg_l64));
  90: 
  91:   // The address in the first (high) instruction is already correct.
  92:   // Adjust the offset in the second (low) instruction.
  93:   MachineOperand &HighOffsetOp = HighPartMI->getOperand(2);
  94:   MachineOperand &LowOffsetOp = LowPartMI->getOperand(2);
  95:   LowOffsetOp.setImm(LowOffsetOp.getImm() + 8);
  96: 
  97:   // Set the opcodes.
  98:   unsigned HighOpcode = getOpcodeForOffset(NewOpcode, HighOffsetOp.getImm());
  99:   unsigned LowOpcode = getOpcodeForOffset(NewOpcode, LowOffsetOp.getImm());
 100:   assert(HighOpcode && LowOpcode && "Both offsets should be in range");
 101:   HighPartMI->setDesc(get(HighOpcode));
 102:   LowPartMI->setDesc(get(LowOpcode));
 103: 
 104:   MachineInstr *FirstMI = HighPartMI;
 105:   if (MI->mayStore()) {
 106:     FirstMI->getOperand(0).setIsKill(false);
 107:     // Add implicit uses of the super register in case one of the subregs is
 108:     // undefined. We could track liveness and skip storing an undefined
 109:     // subreg, but this is hopefully rare (discovered with llvm-stress).
 110:     // If Reg128 was killed, set kill flag on MI.
 111:     RegState Reg128UndefImpl = (Reg128Undef | RegState::Implicit);
 112:     MachineInstrBuilder(MF, HighPartMI).addReg(Reg128, Reg128UndefImpl);
 113:     MachineInstrBuilder(MF, LowPartMI).addReg(Reg128, (Reg128UndefImpl | Reg128Killed));
 114:   } else {
 115:     // If HighPartMI clobbers any of the address registers, it needs to come
 116:     // after LowPartMI.
 117:     auto overlapsAddressReg = [&](Register Reg) -> bool {
 118:       return RI.regsOverlap(Reg, MI->getOperand(1).getReg()) ||
 119:              RI.regsOverlap(Reg, MI->getOperand(3).getReg());
 120:     };
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 121-160 / 第 121-160 行
```cpp
 121:     if (overlapsAddressReg(HighRegOp.getReg())) {
 122:       assert(!overlapsAddressReg(LowRegOp.getReg()) &&
 123:              "Both loads clobber address!");
 124:       MBB->splice(HighPartMI, MBB, LowPartMI);
 125:       FirstMI = LowPartMI;
 126:     }
 127:   }
 128: 
 129:   // Clear the kill flags on the address registers in the first instruction.
 130:   FirstMI->getOperand(1).setIsKill(false);
 131:   FirstMI->getOperand(3).setIsKill(false);
 132: }
 133: 
 134: // Split ADJDYNALLOC instruction MI.
 135: void SystemZInstrInfo::splitAdjDynAlloc(MachineBasicBlock::iterator MI) const {
 136:   MachineBasicBlock *MBB = MI->getParent();
 137:   MachineFunction &MF = *MBB->getParent();
 138:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
 139:   MachineOperand &OffsetMO = MI->getOperand(2);
 140:   SystemZCallingConventionRegisters *Regs = STI.getSpecialRegisters();
 141: 
 142:   uint64_t Offset = (MFFrame.getMaxCallFrameSize() +
 143:                      Regs->getCallFrameSize() +
 144:                      Regs->getStackPointerBias() +
 145:                      OffsetMO.getImm());
 146:   unsigned NewOpcode = getOpcodeForOffset(SystemZ::LA, Offset);
 147:   assert(NewOpcode && "No support for huge argument lists yet");
 148:   MI->setDesc(get(NewOpcode));
 149:   OffsetMO.setImm(Offset);
 150: }
 151: 
 152: // MI is an RI-style pseudo instruction.  Replace it with LowOpcode
 153: // if the first operand is a low GR32 and HighOpcode if the first operand
 154: // is a high GR32.  ConvertHigh is true if LowOpcode takes a signed operand
 155: // and HighOpcode takes an unsigned 32-bit operand.  In those cases,
 156: // MI has the same kind of operand as LowOpcode, so needs to be converted
 157: // if HighOpcode is used.
 158: void SystemZInstrInfo::expandRIPseudo(MachineInstr &MI, unsigned LowOpcode,
 159:                                       unsigned HighOpcode,
 160:                                       bool ConvertHigh) const {
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::splitAdjDynAlloc`, `SystemZInstrInfo::expandRIPseudo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::splitAdjDynAlloc`, `SystemZInstrInfo::expandRIPseudo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 161-200 / 第 161-200 行
```cpp
 161:   Register Reg = MI.getOperand(0).getReg();
 162:   bool IsHigh = SystemZ::isHighReg(Reg);
 163:   MI.setDesc(get(IsHigh ? HighOpcode : LowOpcode));
 164:   if (IsHigh && ConvertHigh)
 165:     MI.getOperand(1).setImm(uint32_t(MI.getOperand(1).getImm()));
 166: }
 167: 
 168: // MI is a three-operand RIE-style pseudo instruction.  Replace it with
 169: // LowOpcodeK if the registers are both low GR32s, otherwise use a move
 170: // followed by HighOpcode or LowOpcode, depending on whether the target
 171: // is a high or low GR32.
 172: void SystemZInstrInfo::expandRIEPseudo(MachineInstr &MI, unsigned LowOpcode,
 173:                                        unsigned LowOpcodeK,
 174:                                        unsigned HighOpcode) const {
 175:   Register DestReg = MI.getOperand(0).getReg();
 176:   Register SrcReg = MI.getOperand(1).getReg();
 177:   bool DestIsHigh = SystemZ::isHighReg(DestReg);
 178:   bool SrcIsHigh = SystemZ::isHighReg(SrcReg);
 179:   if (!DestIsHigh && !SrcIsHigh)
 180:     MI.setDesc(get(LowOpcodeK));
 181:   else {
 182:     if (DestReg != SrcReg) {
 183:       emitGRX32Move(*MI.getParent(), MI, MI.getDebugLoc(), DestReg, SrcReg,
 184:                     SystemZ::LR, 32, MI.getOperand(1).isKill(),
 185:                     MI.getOperand(1).isUndef());
 186:       MI.getOperand(1).setReg(DestReg);
 187:     }
 188:     MI.setDesc(get(DestIsHigh ? HighOpcode : LowOpcode));
 189:     MI.tieOperands(0, 1);
 190:   }
 191: }
 192: 
 193: // MI is an RXY-style pseudo instruction.  Replace it with LowOpcode
 194: // if the first operand is a low GR32 and HighOpcode if the first operand
 195: // is a high GR32.
 196: void SystemZInstrInfo::expandRXYPseudo(MachineInstr &MI, unsigned LowOpcode,
 197:                                        unsigned HighOpcode) const {
 198:   Register Reg = MI.getOperand(0).getReg();
 199:   unsigned Opcode = getOpcodeForOffset(
 200:       SystemZ::isHighReg(Reg) ? HighOpcode : LowOpcode,
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::expandRIEPseudo`, `SystemZInstrInfo::expandRXYPseudo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::expandRIEPseudo`, `SystemZInstrInfo::expandRXYPseudo` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 201-240 / 第 201-240 行
```cpp
 201:       MI.getOperand(2).getImm());
 202:   MI.setDesc(get(Opcode));
 203: }
 204: 
 205: // MI is a load-on-condition pseudo instruction with a single register
 206: // (source or destination) operand.  Replace it with LowOpcode if the
 207: // register is a low GR32 and HighOpcode if the register is a high GR32.
 208: void SystemZInstrInfo::expandLOCPseudo(MachineInstr &MI, unsigned LowOpcode,
 209:                                        unsigned HighOpcode) const {
 210:   Register Reg = MI.getOperand(0).getReg();
 211:   unsigned Opcode = SystemZ::isHighReg(Reg) ? HighOpcode : LowOpcode;
 212:   MI.setDesc(get(Opcode));
 213: }
 214: 
 215: // MI is an RR-style pseudo instruction that zero-extends the low Size bits
 216: // of one GRX32 into another.  Replace it with LowOpcode if both operands
 217: // are low registers, otherwise use RISB[LH]G.
 218: void SystemZInstrInfo::expandZExtPseudo(MachineInstr &MI, unsigned LowOpcode,
 219:                                         unsigned Size) const {
 220:   MachineInstrBuilder MIB =
 221:     emitGRX32Move(*MI.getParent(), MI, MI.getDebugLoc(),
 222:                MI.getOperand(0).getReg(), MI.getOperand(1).getReg(), LowOpcode,
 223:                Size, MI.getOperand(1).isKill(), MI.getOperand(1).isUndef());
 224: 
 225:   // Keep the remaining operands as-is.
 226:   for (const MachineOperand &MO : llvm::drop_begin(MI.operands(), 2))
 227:     MIB.add(MO);
 228: 
 229:   MI.eraseFromParent();
 230: }
 231: 
 232: void SystemZInstrInfo::expandLoadStackGuard(MachineInstr *MI) const {
 233:   MachineBasicBlock *MBB = MI->getParent();
 234:   MachineFunction &MF = *MBB->getParent();
 235:   const Register Reg64 = MI->getOperand(0).getReg();
 236:   const Register Reg32 = RI.getSubReg(Reg64, SystemZ::subreg_l32);
 237: 
 238:   // EAR can only load the low subregister so us a shift for %a0 to produce
 239:   // the GR containing %a0 and %a1.
 240: 
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::expandLOCPseudo`, `SystemZInstrInfo::expandZExtPseudo`, `SystemZInstrInfo::expandLoadStackGuard`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::expandLOCPseudo`, `SystemZInstrInfo::expandZExtPseudo`, `SystemZInstrInfo::expandLoadStackGuard` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 241-280 / 第 241-280 行
```cpp
 241:   // ear <reg>, %a0
 242:   BuildMI(*MBB, MI, MI->getDebugLoc(), get(SystemZ::EAR), Reg32)
 243:     .addReg(SystemZ::A0)
 244:     .addReg(Reg64, RegState::ImplicitDefine);
 245: 
 246:   // sllg <reg>, <reg>, 32
 247:   BuildMI(*MBB, MI, MI->getDebugLoc(), get(SystemZ::SLLG), Reg64)
 248:     .addReg(Reg64)
 249:     .addReg(0)
 250:     .addImm(32);
 251: 
 252:   // ear <reg>, %a1
 253:   BuildMI(*MBB, MI, MI->getDebugLoc(), get(SystemZ::EAR), Reg32)
 254:     .addReg(SystemZ::A1);
 255: 
 256:   // lg <reg>, 40(<reg>)
 257:   MI->setDesc(get(SystemZ::LG));
 258:   MachineInstrBuilder(MF, MI).addReg(Reg64).addImm(40).addReg(0);
 259: }
 260: 
 261: // Emit a zero-extending move from 32-bit GPR SrcReg to 32-bit GPR
 262: // DestReg before MBBI in MBB.  Use LowLowOpcode when both DestReg and SrcReg
 263: // are low registers, otherwise use RISB[LH]G.  Size is the number of bits
 264: // taken from the low end of SrcReg (8 for LLCR, 16 for LLHR and 32 for LR).
 265: // KillSrc is true if this move is the last use of SrcReg.
 266: MachineInstrBuilder
 267: SystemZInstrInfo::emitGRX32Move(MachineBasicBlock &MBB,
 268:                                 MachineBasicBlock::iterator MBBI,
 269:                                 const DebugLoc &DL, unsigned DestReg,
 270:                                 unsigned SrcReg, unsigned LowLowOpcode,
 271:                                 unsigned Size, bool KillSrc,
 272:                                 bool UndefSrc) const {
 273:   unsigned Opcode;
 274:   bool DestIsHigh = SystemZ::isHighReg(DestReg);
 275:   bool SrcIsHigh = SystemZ::isHighReg(SrcReg);
 276:   if (DestIsHigh && SrcIsHigh)
 277:     Opcode = SystemZ::RISBHH;
 278:   else if (DestIsHigh && !SrcIsHigh)
 279:     Opcode = SystemZ::RISBHL;
 280:   else if (!DestIsHigh && SrcIsHigh)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `SystemZInstrInfo::emitGRX32Move`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `SystemZInstrInfo::emitGRX32Move` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-320 / 第 281-320 行
```cpp
 281:     Opcode = SystemZ::RISBLH;
 282:   else {
 283:     return BuildMI(MBB, MBBI, DL, get(LowLowOpcode), DestReg)
 284:       .addReg(SrcReg, getKillRegState(KillSrc) | getUndefRegState(UndefSrc));
 285:   }
 286:   unsigned Rotate = (DestIsHigh != SrcIsHigh ? 32 : 0);
 287:   return BuildMI(MBB, MBBI, DL, get(Opcode), DestReg)
 288:     .addReg(DestReg, RegState::Undef)
 289:     .addReg(SrcReg, getKillRegState(KillSrc) | getUndefRegState(UndefSrc))
 290:     .addImm(32 - Size).addImm(128 + 31).addImm(Rotate);
 291: }
 292: 
 293: MachineInstr *SystemZInstrInfo::commuteInstructionImpl(MachineInstr &MI,
 294:                                                        bool NewMI,
 295:                                                        unsigned OpIdx1,
 296:                                                        unsigned OpIdx2) const {
 297:   auto cloneIfNew = [NewMI](MachineInstr &MI) -> MachineInstr & {
 298:     if (NewMI)
 299:       return *MI.getParent()->getParent()->CloneMachineInstr(&MI);
 300:     return MI;
 301:   };
 302: 
 303:   switch (MI.getOpcode()) {
 304:   case SystemZ::SELRMux:
 305:   case SystemZ::SELFHR:
 306:   case SystemZ::SELR:
 307:   case SystemZ::SELGR:
 308:   case SystemZ::LOCRMux:
 309:   case SystemZ::LOCFHR:
 310:   case SystemZ::LOCR:
 311:   case SystemZ::LOCGR: {
 312:     auto &WorkingMI = cloneIfNew(MI);
 313:     // Invert condition.
 314:     unsigned CCValid = WorkingMI.getOperand(3).getImm();
 315:     unsigned CCMask = WorkingMI.getOperand(4).getImm();
 316:     WorkingMI.getOperand(4).setImm(CCMask ^ CCValid);
 317:     return TargetInstrInfo::commuteInstructionImpl(WorkingMI, /*NewMI=*/false,
 318:                                                    OpIdx1, OpIdx2);
 319:   }
 320:   default:
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 321-360 / 第 321-360 行
```cpp
 321:     return TargetInstrInfo::commuteInstructionImpl(MI, NewMI, OpIdx1, OpIdx2);
 322:   }
 323: }
 324: 
 325: // If MI is a simple load or store for a frame object, return the register
 326: // it loads or stores and set FrameIndex to the index of the frame object.
 327: // Return 0 otherwise.
 328: //
 329: // Flag is SimpleBDXLoad for loads and SimpleBDXStore for stores.
 330: static int isSimpleMove(const MachineInstr &MI, int &FrameIndex,
 331:                         unsigned Flag) {
 332:   const MCInstrDesc &MCID = MI.getDesc();
 333:   if ((MCID.TSFlags & Flag) && MI.getOperand(1).isFI() &&
 334:       MI.getOperand(2).getImm() == 0 && MI.getOperand(3).getReg() == 0) {
 335:     FrameIndex = MI.getOperand(1).getIndex();
 336:     return MI.getOperand(0).getReg();
 337:   }
 338:   return 0;
 339: }
 340: 
 341: Register SystemZInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
 342:                                                int &FrameIndex) const {
 343:   return isSimpleMove(MI, FrameIndex, SystemZII::SimpleBDXLoad);
 344: }
 345: 
 346: Register SystemZInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
 347:                                               int &FrameIndex) const {
 348:   return isSimpleMove(MI, FrameIndex, SystemZII::SimpleBDXStore);
 349: }
 350: 
 351: Register SystemZInstrInfo::isLoadFromStackSlotPostFE(const MachineInstr &MI,
 352:                                                      int &FrameIndex) const {
 353:   // if this is not a simple load from memory, it's not a load from stack slot
 354:   // either.
 355:   const MCInstrDesc &MCID = MI.getDesc();
 356:   if (!(MCID.TSFlags & SystemZII::SimpleBDXLoad))
 357:     return 0;
 358: 
 359:   // This version of isLoadFromStackSlot should only be used post frame-index
 360:   // elimination.
```
- **EN**: The range implements or declares functions including `isSimpleMove`, `SystemZInstrInfo::isLoadFromStackSlot`, `SystemZInstrInfo::isStoreToStackSlot`, `SystemZInstrInfo::isLoadFromStackSlotPostFE`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSimpleMove`, `SystemZInstrInfo::isLoadFromStackSlot`, `SystemZInstrInfo::isStoreToStackSlot`, `SystemZInstrInfo::isLoadFromStackSlotPostFE` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 361-400 / 第 361-400 行
```cpp
 361:   assert(!MI.getOperand(1).isFI());
 362: 
 363:   // Now attempt to derive frame index from MachineMemOperands.
 364:   SmallVector<const MachineMemOperand *, 1> Accesses;
 365:   if (hasLoadFromStackSlot(MI, Accesses)) {
 366:     FrameIndex =
 367:         cast<FixedStackPseudoSourceValue>(Accesses.front()->getPseudoValue())
 368:             ->getFrameIndex();
 369:     return MI.getOperand(0).getReg();
 370:   }
 371:   return 0;
 372: }
 373: 
 374: Register SystemZInstrInfo::isStoreToStackSlotPostFE(const MachineInstr &MI,
 375:                                                     int &FrameIndex) const {
 376:   // if this is not a simple store to memory, it's not a store to stack slot
 377:   // either.
 378:   const MCInstrDesc &MCID = MI.getDesc();
 379:   if (!(MCID.TSFlags & SystemZII::SimpleBDXStore))
 380:     return 0;
 381: 
 382:   // This version of isStoreToStackSlot should only be used post frame-index
 383:   // elimination.
 384:   assert(!MI.getOperand(1).isFI());
 385: 
 386:   // Now attempt to derive frame index from MachineMemOperands.
 387:   SmallVector<const MachineMemOperand *, 1> Accesses;
 388:   if (hasStoreToStackSlot(MI, Accesses)) {
 389:     FrameIndex =
 390:         cast<FixedStackPseudoSourceValue>(Accesses.front()->getPseudoValue())
 391:             ->getFrameIndex();
 392:     return MI.getOperand(0).getReg();
 393:   }
 394:   return 0;
 395: }
 396: 
 397: bool SystemZInstrInfo::isStackSlotCopy(const MachineInstr &MI,
 398:                                        int &DestFrameIndex,
 399:                                        int &SrcFrameIndex) const {
 400:   // Check for MVC 0(Length,FI1),0(FI2)
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::isStoreToStackSlotPostFE`, `SystemZInstrInfo::isStackSlotCopy`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::isStoreToStackSlotPostFE`, `SystemZInstrInfo::isStackSlotCopy` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 401-440 / 第 401-440 行
```cpp
 401:   const MachineFrameInfo &MFI = MI.getParent()->getParent()->getFrameInfo();
 402:   if (MI.getOpcode() != SystemZ::MVC || !MI.getOperand(0).isFI() ||
 403:       MI.getOperand(1).getImm() != 0 || !MI.getOperand(3).isFI() ||
 404:       MI.getOperand(4).getImm() != 0)
 405:     return false;
 406: 
 407:   // Check that Length covers the full slots.
 408:   int64_t Length = MI.getOperand(2).getImm();
 409:   unsigned FI1 = MI.getOperand(0).getIndex();
 410:   unsigned FI2 = MI.getOperand(3).getIndex();
 411:   if (MFI.getObjectSize(FI1) != Length ||
 412:       MFI.getObjectSize(FI2) != Length)
 413:     return false;
 414: 
 415:   DestFrameIndex = FI1;
 416:   SrcFrameIndex = FI2;
 417:   return true;
 418: }
 419: 
 420: bool SystemZInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 421:                                      MachineBasicBlock *&TBB,
 422:                                      MachineBasicBlock *&FBB,
 423:                                      SmallVectorImpl<MachineOperand> &Cond,
 424:                                      bool AllowModify) const {
 425:   // Most of the code and comments here are boilerplate.
 426: 
 427:   // Start from the bottom of the block and work up, examining the
 428:   // terminator instructions.
 429:   MachineBasicBlock::iterator I = MBB.end();
 430:   while (I != MBB.begin()) {
 431:     --I;
 432:     if (I->isDebugInstr())
 433:       continue;
 434: 
 435:     // Working from the bottom, when we see a non-terminator instruction, we're
 436:     // done.
 437:     if (!isUnpredicatedTerminator(*I))
 438:       break;
 439: 
 440:     // A terminator that isn't a branch can't easily be handled by this
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::analyzeBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::analyzeBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 441-480 / 第 441-480 行
```cpp
 441:     // analysis.
 442:     if (!I->isBranch())
 443:       return true;
 444: 
 445:     // Can't handle indirect branches.
 446:     SystemZII::Branch Branch(getBranchInfo(*I));
 447:     if (!Branch.hasMBBTarget())
 448:       return true;
 449: 
 450:     // Punt on compound branches.
 451:     if (Branch.Type != SystemZII::BranchNormal)
 452:       return true;
 453: 
 454:     if (Branch.CCMask == SystemZ::CCMASK_ANY) {
 455:       // Handle unconditional branches.
 456:       if (!AllowModify) {
 457:         TBB = Branch.getMBBTarget();
 458:         continue;
 459:       }
 460: 
 461:       // If the block has any instructions after a JMP, delete them.
 462:       MBB.erase(std::next(I), MBB.end());
 463: 
 464:       Cond.clear();
 465:       FBB = nullptr;
 466: 
 467:       // Delete the JMP if it's equivalent to a fall-through.
 468:       if (MBB.isLayoutSuccessor(Branch.getMBBTarget())) {
 469:         TBB = nullptr;
 470:         I->eraseFromParent();
 471:         I = MBB.end();
 472:         continue;
 473:       }
 474: 
 475:       // TBB is used to indicate the unconditinal destination.
 476:       TBB = Branch.getMBBTarget();
 477:       continue;
 478:     }
 479: 
 480:     // Working from the bottom, handle the first conditional branch.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 481-520 / 第 481-520 行
```cpp
 481:     if (Cond.empty()) {
 482:       // FIXME: add X86-style branch swap
 483:       FBB = TBB;
 484:       TBB = Branch.getMBBTarget();
 485:       Cond.push_back(MachineOperand::CreateImm(Branch.CCValid));
 486:       Cond.push_back(MachineOperand::CreateImm(Branch.CCMask));
 487:       continue;
 488:     }
 489: 
 490:     // Handle subsequent conditional branches.
 491:     assert(Cond.size() == 2 && TBB && "Should have seen a conditional branch");
 492: 
 493:     // Only handle the case where all conditional branches branch to the same
 494:     // destination.
 495:     if (TBB != Branch.getMBBTarget())
 496:       return true;
 497: 
 498:     // If the conditions are the same, we can leave them alone.
 499:     unsigned OldCCValid = Cond[0].getImm();
 500:     unsigned OldCCMask = Cond[1].getImm();
 501:     if (OldCCValid == Branch.CCValid && OldCCMask == Branch.CCMask)
 502:       continue;
 503: 
 504:     // FIXME: Try combining conditions like X86 does.  Should be easy on Z!
 505:     return false;
 506:   }
 507: 
 508:   return false;
 509: }
 510: 
 511: unsigned SystemZInstrInfo::removeBranch(MachineBasicBlock &MBB,
 512:                                         int *BytesRemoved) const {
 513:   assert(!BytesRemoved && "code size not handled");
 514: 
 515:   // Most of the code and comments here are boilerplate.
 516:   MachineBasicBlock::iterator I = MBB.end();
 517:   unsigned Count = 0;
 518: 
 519:   while (I != MBB.begin()) {
 520:     --I;
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::removeBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::removeBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 521-560 / 第 521-560 行
```cpp
 521:     if (I->isDebugInstr())
 522:       continue;
 523:     if (!I->isBranch())
 524:       break;
 525:     if (!getBranchInfo(*I).hasMBBTarget())
 526:       break;
 527:     // Remove the branch.
 528:     I->eraseFromParent();
 529:     I = MBB.end();
 530:     ++Count;
 531:   }
 532: 
 533:   return Count;
 534: }
 535: 
 536: bool SystemZInstrInfo::
 537: reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const {
 538:   assert(Cond.size() == 2 && "Invalid condition");
 539:   Cond[1].setImm(Cond[1].getImm() ^ Cond[0].getImm());
 540:   return false;
 541: }
 542: 
 543: unsigned SystemZInstrInfo::insertBranch(MachineBasicBlock &MBB,
 544:                                         MachineBasicBlock *TBB,
 545:                                         MachineBasicBlock *FBB,
 546:                                         ArrayRef<MachineOperand> Cond,
 547:                                         const DebugLoc &DL,
 548:                                         int *BytesAdded) const {
 549:   // In this function we output 32-bit branches, which should always
 550:   // have enough range.  They can be shortened and relaxed by later code
 551:   // in the pipeline, if desired.
 552: 
 553:   // Shouldn't be a fall through.
 554:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
 555:   assert((Cond.size() == 2 || Cond.size() == 0) &&
 556:          "SystemZ branch conditions have one component!");
 557:   assert(!BytesAdded && "code size not handled");
 558: 
 559:   if (Cond.empty()) {
 560:     // Unconditional branch?
```
- **EN**: The range implements or declares functions including `reverseBranchCondition`, `SystemZInstrInfo::insertBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `reverseBranchCondition`, `SystemZInstrInfo::insertBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-600 / 第 561-600 行
```cpp
 561:     assert(!FBB && "Unconditional branch with multiple successors!");
 562:     BuildMI(&MBB, DL, get(SystemZ::J)).addMBB(TBB);
 563:     return 1;
 564:   }
 565: 
 566:   // Conditional branch.
 567:   unsigned Count = 0;
 568:   unsigned CCValid = Cond[0].getImm();
 569:   unsigned CCMask = Cond[1].getImm();
 570:   BuildMI(&MBB, DL, get(SystemZ::BRC))
 571:     .addImm(CCValid).addImm(CCMask).addMBB(TBB);
 572:   ++Count;
 573: 
 574:   if (FBB) {
 575:     // Two-way Conditional branch. Insert the second branch.
 576:     BuildMI(&MBB, DL, get(SystemZ::J)).addMBB(FBB);
 577:     ++Count;
 578:   }
 579:   return Count;
 580: }
 581: 
 582: bool SystemZInstrInfo::analyzeCompare(const MachineInstr &MI, Register &SrcReg,
 583:                                       Register &SrcReg2, int64_t &Mask,
 584:                                       int64_t &Value) const {
 585:   assert(MI.isCompare() && "Caller should have checked for a comparison");
 586: 
 587:   if (MI.getNumExplicitOperands() == 2 && MI.getOperand(0).isReg() &&
 588:       MI.getOperand(1).isImm()) {
 589:     SrcReg = MI.getOperand(0).getReg();
 590:     SrcReg2 = 0;
 591:     Value = MI.getOperand(1).getImm();
 592:     Mask = ~0;
 593:     return true;
 594:   }
 595: 
 596:   return false;
 597: }
 598: 
 599: bool SystemZInstrInfo::canInsertSelect(const MachineBasicBlock &MBB,
 600:                                        ArrayRef<MachineOperand> Pred,
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZInstrInfo::analyzeCompare`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZInstrInfo::analyzeCompare` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 601-640 / 第 601-640 行
```cpp
 601:                                        Register DstReg, Register TrueReg,
 602:                                        Register FalseReg, int &CondCycles,
 603:                                        int &TrueCycles,
 604:                                        int &FalseCycles) const {
 605:   // Not all subtargets have LOCR instructions.
 606:   if (!STI.hasLoadStoreOnCond())
 607:     return false;
 608:   if (Pred.size() != 2)
 609:     return false;
 610: 
 611:   // Check register classes.
 612:   const MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
 613:   const TargetRegisterClass *RC =
 614:     RI.getCommonSubClass(MRI.getRegClass(TrueReg), MRI.getRegClass(FalseReg));
 615:   if (!RC)
 616:     return false;
 617: 
 618:   // We have LOCR instructions for 32 and 64 bit general purpose registers.
 619:   if ((STI.hasLoadStoreOnCond2() &&
 620:        SystemZ::GRX32BitRegClass.hasSubClassEq(RC)) ||
 621:       SystemZ::GR32BitRegClass.hasSubClassEq(RC) ||
 622:       SystemZ::GR64BitRegClass.hasSubClassEq(RC)) {
 623:     CondCycles = 2;
 624:     TrueCycles = 2;
 625:     FalseCycles = 2;
 626:     return true;
 627:   }
 628: 
 629:   // Can't do anything else.
 630:   return false;
 631: }
 632: 
 633: void SystemZInstrInfo::insertSelect(MachineBasicBlock &MBB,
 634:                                     MachineBasicBlock::iterator I,
 635:                                     const DebugLoc &DL, Register DstReg,
 636:                                     ArrayRef<MachineOperand> Pred,
 637:                                     Register TrueReg,
 638:                                     Register FalseReg) const {
 639:   MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
 640:   const TargetRegisterClass *RC = MRI.getRegClass(DstReg);
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::insertSelect`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::insertSelect` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 641-680 / 第 641-680 行
```cpp
 641: 
 642:   assert(Pred.size() == 2 && "Invalid condition");
 643:   unsigned CCValid = Pred[0].getImm();
 644:   unsigned CCMask = Pred[1].getImm();
 645: 
 646:   unsigned Opc;
 647:   if (SystemZ::GRX32BitRegClass.hasSubClassEq(RC)) {
 648:     if (STI.hasMiscellaneousExtensions3())
 649:       Opc = SystemZ::SELRMux;
 650:     else if (STI.hasLoadStoreOnCond2())
 651:       Opc = SystemZ::LOCRMux;
 652:     else {
 653:       Opc = SystemZ::LOCR;
 654:       MRI.constrainRegClass(DstReg, &SystemZ::GR32BitRegClass);
 655:       Register TReg = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
 656:       Register FReg = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
 657:       BuildMI(MBB, I, DL, get(TargetOpcode::COPY), TReg).addReg(TrueReg);
 658:       BuildMI(MBB, I, DL, get(TargetOpcode::COPY), FReg).addReg(FalseReg);
 659:       TrueReg = TReg;
 660:       FalseReg = FReg;
 661:     }
 662:   } else if (SystemZ::GR64BitRegClass.hasSubClassEq(RC)) {
 663:     if (STI.hasMiscellaneousExtensions3())
 664:       Opc = SystemZ::SELGR;
 665:     else
 666:       Opc = SystemZ::LOCGR;
 667:   } else
 668:     llvm_unreachable("Invalid register class");
 669: 
 670:   BuildMI(MBB, I, DL, get(Opc), DstReg)
 671:     .addReg(FalseReg).addReg(TrueReg)
 672:     .addImm(CCValid).addImm(CCMask);
 673: }
 674: 
 675: bool SystemZInstrInfo::foldImmediate(MachineInstr &UseMI, MachineInstr &DefMI,
 676:                                      Register Reg,
 677:                                      MachineRegisterInfo *MRI) const {
 678:   unsigned DefOpc = DefMI.getOpcode();
 679: 
 680:   if (DefOpc == SystemZ::VGBM) {
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZInstrInfo::foldImmediate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZInstrInfo::foldImmediate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 681-720 / 第 681-720 行
```cpp
 681:     int64_t ImmVal = DefMI.getOperand(1).getImm();
 682:     if (ImmVal != 0) // TODO: Handle other values
 683:       return false;
 684: 
 685:     // Fold gr128 = COPY (vr128 VGBM imm)
 686:     //
 687:     // %tmp:gr64 = LGHI 0
 688:     // to  gr128 = REG_SEQUENCE %tmp, %tmp
 689:     assert(DefMI.getOperand(0).getReg() == Reg);
 690: 
 691:     if (!UseMI.isCopy())
 692:       return false;
 693: 
 694:     Register CopyDstReg = UseMI.getOperand(0).getReg();
 695:     if (CopyDstReg.isVirtual() &&
 696:         MRI->getRegClass(CopyDstReg) == &SystemZ::GR128BitRegClass &&
 697:         MRI->hasOneNonDBGUse(Reg)) {
 698:       // TODO: Handle physical registers
 699:       // TODO: Handle gr64 uses with subregister indexes
 700:       // TODO: Should this multi-use cases?
 701:       Register TmpReg = MRI->createVirtualRegister(&SystemZ::GR64BitRegClass);
 702:       MachineBasicBlock &MBB = *UseMI.getParent();
 703: 
 704:       loadImmediate(MBB, UseMI.getIterator(), TmpReg, ImmVal);
 705: 
 706:       UseMI.setDesc(get(SystemZ::REG_SEQUENCE));
 707:       UseMI.getOperand(1).setReg(TmpReg);
 708:       MachineInstrBuilder(*MBB.getParent(), &UseMI)
 709:           .addImm(SystemZ::subreg_h64)
 710:           .addReg(TmpReg)
 711:           .addImm(SystemZ::subreg_l64);
 712: 
 713:       if (MRI->use_nodbg_empty(Reg))
 714:         DefMI.eraseFromParent();
 715:       return true;
 716:     }
 717: 
 718:     return false;
 719:   }
 720: 
```
- **EN**: The range implements or declares functions including `MachineInstrBuilder`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `MachineInstrBuilder` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 721-760 / 第 721-760 行
```cpp
 721:   if (DefOpc != SystemZ::LHIMux && DefOpc != SystemZ::LHI &&
 722:       DefOpc != SystemZ::LGHI)
 723:     return false;
 724:   if (DefMI.getOperand(0).getReg() != Reg)
 725:     return false;
 726:   int32_t ImmVal = (int32_t)DefMI.getOperand(1).getImm();
 727: 
 728:   unsigned UseOpc = UseMI.getOpcode();
 729:   unsigned NewUseOpc;
 730:   unsigned UseIdx;
 731:   int CommuteIdx = -1;
 732:   bool TieOps = false;
 733:   switch (UseOpc) {
 734:   case SystemZ::SELRMux:
 735:     TieOps = true;
 736:     [[fallthrough]];
 737:   case SystemZ::LOCRMux:
 738:     if (!STI.hasLoadStoreOnCond2())
 739:       return false;
 740:     NewUseOpc = SystemZ::LOCHIMux;
 741:     if (UseMI.getOperand(2).getReg() == Reg)
 742:       UseIdx = 2;
 743:     else if (UseMI.getOperand(1).getReg() == Reg)
 744:       UseIdx = 2, CommuteIdx = 1;
 745:     else
 746:       return false;
 747:     break;
 748:   case SystemZ::SELGR:
 749:     TieOps = true;
 750:     [[fallthrough]];
 751:   case SystemZ::LOCGR:
 752:     if (!STI.hasLoadStoreOnCond2())
 753:       return false;
 754:     NewUseOpc = SystemZ::LOCGHI;
 755:     if (UseMI.getOperand(2).getReg() == Reg)
 756:       UseIdx = 2;
 757:     else if (UseMI.getOperand(1).getReg() == Reg)
 758:       UseIdx = 2, CommuteIdx = 1;
 759:     else
 760:       return false;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 761-800 / 第 761-800 行
```cpp
 761:     break;
 762:   default:
 763:     return false;
 764:   }
 765: 
 766:   if (CommuteIdx != -1)
 767:     if (!commuteInstruction(UseMI, false, CommuteIdx, UseIdx))
 768:       return false;
 769: 
 770:   bool DeleteDef = MRI->hasOneNonDBGUse(Reg);
 771:   UseMI.setDesc(get(NewUseOpc));
 772:   if (TieOps)
 773:     UseMI.tieOperands(0, 1);
 774:   UseMI.getOperand(UseIdx).ChangeToImmediate(ImmVal);
 775:   if (DeleteDef)
 776:     DefMI.eraseFromParent();
 777: 
 778:   return true;
 779: }
 780: 
 781: bool SystemZInstrInfo::isPredicable(const MachineInstr &MI) const {
 782:   unsigned Opcode = MI.getOpcode();
 783:   if (Opcode == SystemZ::Return ||
 784:       Opcode == SystemZ::Return_XPLINK ||
 785:       Opcode == SystemZ::Trap ||
 786:       Opcode == SystemZ::CallJG ||
 787:       Opcode == SystemZ::CallBR)
 788:     return true;
 789:   return false;
 790: }
 791: 
 792: bool SystemZInstrInfo::
 793: isProfitableToIfCvt(MachineBasicBlock &MBB,
 794:                     unsigned NumCycles, unsigned ExtraPredCycles,
 795:                     BranchProbability Probability) const {
 796:   // Avoid using conditional returns at the end of a loop (since then
 797:   // we'd need to emit an unconditional branch to the beginning anyway,
 798:   // making the loop body longer).  This doesn't apply for low-probability
 799:   // loops (eg. compare-and-swap retry), so just decide based on branch
 800:   // probability instead of looping structure.
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::isPredicable`, `isProfitableToIfCvt`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::isPredicable`, `isProfitableToIfCvt` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 801-840 / 第 801-840 行
```cpp
 801:   // However, since Compare and Trap instructions cost the same as a regular
 802:   // Compare instruction, we should allow the if conversion to convert this
 803:   // into a Conditional Compare regardless of the branch probability.
 804:   if (MBB.getLastNonDebugInstr()->getOpcode() != SystemZ::Trap &&
 805:       MBB.succ_empty() && Probability < BranchProbability(1, 8))
 806:     return false;
 807:   // For now only convert single instructions.
 808:   return NumCycles == 1;
 809: }
 810: 
 811: bool SystemZInstrInfo::
 812: isProfitableToIfCvt(MachineBasicBlock &TMBB,
 813:                     unsigned NumCyclesT, unsigned ExtraPredCyclesT,
 814:                     MachineBasicBlock &FMBB,
 815:                     unsigned NumCyclesF, unsigned ExtraPredCyclesF,
 816:                     BranchProbability Probability) const {
 817:   // For now avoid converting mutually-exclusive cases.
 818:   return false;
 819: }
 820: 
 821: bool SystemZInstrInfo::
 822: isProfitableToDupForIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
 823:                           BranchProbability Probability) const {
 824:   // For now only duplicate single instructions.
 825:   return NumCycles == 1;
 826: }
 827: 
 828: bool SystemZInstrInfo::PredicateInstruction(
 829:     MachineInstr &MI, ArrayRef<MachineOperand> Pred) const {
 830:   assert(Pred.size() == 2 && "Invalid condition");
 831:   unsigned CCValid = Pred[0].getImm();
 832:   unsigned CCMask = Pred[1].getImm();
 833:   assert(CCMask > 0 && CCMask < 15 && "Invalid predicate");
 834:   unsigned Opcode = MI.getOpcode();
 835:   if (Opcode == SystemZ::Trap) {
 836:     MI.setDesc(get(SystemZ::CondTrap));
 837:     MachineInstrBuilder(*MI.getParent()->getParent(), MI)
 838:       .addImm(CCValid).addImm(CCMask)
 839:       .addReg(SystemZ::CC, RegState::Implicit);
 840:     return true;
```
- **EN**: The range implements or declares functions including `isProfitableToIfCvt`, `isProfitableToDupForIfCvt`, `SystemZInstrInfo::PredicateInstruction`, `MachineInstrBuilder`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `isProfitableToIfCvt`, `isProfitableToDupForIfCvt`, `SystemZInstrInfo::PredicateInstruction`, `MachineInstrBuilder` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 841-880 / 第 841-880 行
```cpp
 841:   }
 842:   if (Opcode == SystemZ::Return || Opcode == SystemZ::Return_XPLINK) {
 843:     MI.setDesc(get(Opcode == SystemZ::Return ? SystemZ::CondReturn
 844:                                              : SystemZ::CondReturn_XPLINK));
 845:     MachineInstrBuilder(*MI.getParent()->getParent(), MI)
 846:         .addImm(CCValid)
 847:         .addImm(CCMask)
 848:         .addReg(SystemZ::CC, RegState::Implicit);
 849:     return true;
 850:   }
 851:   if (Opcode == SystemZ::CallJG) {
 852:     MachineOperand FirstOp = MI.getOperand(0);
 853:     const uint32_t *RegMask = MI.getOperand(1).getRegMask();
 854:     MI.removeOperand(1);
 855:     MI.removeOperand(0);
 856:     MI.setDesc(get(SystemZ::CallBRCL));
 857:     MachineInstrBuilder(*MI.getParent()->getParent(), MI)
 858:         .addImm(CCValid)
 859:         .addImm(CCMask)
 860:         .add(FirstOp)
 861:         .addRegMask(RegMask)
 862:         .addReg(SystemZ::CC, RegState::Implicit);
 863:     return true;
 864:   }
 865:   if (Opcode == SystemZ::CallBR) {
 866:     MachineOperand Target = MI.getOperand(0);
 867:     const uint32_t *RegMask = MI.getOperand(1).getRegMask();
 868:     MI.removeOperand(1);
 869:     MI.removeOperand(0);
 870:     MI.setDesc(get(SystemZ::CallBCR));
 871:     MachineInstrBuilder(*MI.getParent()->getParent(), MI)
 872:       .addImm(CCValid).addImm(CCMask)
 873:       .add(Target)
 874:       .addRegMask(RegMask)
 875:       .addReg(SystemZ::CC, RegState::Implicit);
 876:     return true;
 877:   }
 878:   return false;
 879: }
 880: 
```
- **EN**: The range implements or declares functions including `MachineInstrBuilder`, `MachineInstrBuilder`, `MachineInstrBuilder`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `MachineInstrBuilder`, `MachineInstrBuilder`, `MachineInstrBuilder` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 881-920 / 第 881-920 行
```cpp
 881: void SystemZInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
 882:                                    MachineBasicBlock::iterator MBBI,
 883:                                    const DebugLoc &DL, Register DestReg,
 884:                                    Register SrcReg, bool KillSrc,
 885:                                    bool RenamableDest,
 886:                                    bool RenamableSrc) const {
 887:   // Split 128-bit GPR moves into two 64-bit moves. Add implicit uses of the
 888:   // super register in case one of the subregs is undefined.
 889:   // This handles ADDR128 too.
 890:   if (SystemZ::GR128BitRegClass.contains(DestReg, SrcReg)) {
 891:     copyPhysReg(MBB, MBBI, DL, RI.getSubReg(DestReg, SystemZ::subreg_h64),
 892:                 RI.getSubReg(SrcReg, SystemZ::subreg_h64), KillSrc);
 893:     MachineInstrBuilder(*MBB.getParent(), std::prev(MBBI))
 894:       .addReg(SrcReg, RegState::Implicit);
 895:     copyPhysReg(MBB, MBBI, DL, RI.getSubReg(DestReg, SystemZ::subreg_l64),
 896:                 RI.getSubReg(SrcReg, SystemZ::subreg_l64), KillSrc);
 897:     MachineInstrBuilder(*MBB.getParent(), std::prev(MBBI))
 898:       .addReg(SrcReg, (getKillRegState(KillSrc) | RegState::Implicit));
 899:     return;
 900:   }
 901: 
 902:   if (SystemZ::GRX32BitRegClass.contains(DestReg, SrcReg)) {
 903:     emitGRX32Move(MBB, MBBI, DL, DestReg, SrcReg, SystemZ::LR, 32, KillSrc,
 904:                   false);
 905:     return;
 906:   }
 907: 
 908:   // Move 128-bit floating-point values between VR128 and FP128.
 909:   if (SystemZ::VR128BitRegClass.contains(DestReg) &&
 910:       SystemZ::FP128BitRegClass.contains(SrcReg)) {
 911:     MCRegister SrcRegHi =
 912:         RI.getMatchingSuperReg(RI.getSubReg(SrcReg, SystemZ::subreg_h64),
 913:                                SystemZ::subreg_h64, &SystemZ::VR128BitRegClass);
 914:     MCRegister SrcRegLo =
 915:         RI.getMatchingSuperReg(RI.getSubReg(SrcReg, SystemZ::subreg_l64),
 916:                                SystemZ::subreg_h64, &SystemZ::VR128BitRegClass);
 917: 
 918:     BuildMI(MBB, MBBI, DL, get(SystemZ::VMRHG), DestReg)
 919:       .addReg(SrcRegHi, getKillRegState(KillSrc))
 920:       .addReg(SrcRegLo, getKillRegState(KillSrc));
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::copyPhysReg`, `MachineInstrBuilder`, `MachineInstrBuilder`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::copyPhysReg`, `MachineInstrBuilder`, `MachineInstrBuilder`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 921-960 / 第 921-960 行
```cpp
 921:     return;
 922:   }
 923:   if (SystemZ::FP128BitRegClass.contains(DestReg) &&
 924:       SystemZ::VR128BitRegClass.contains(SrcReg)) {
 925:     MCRegister DestRegHi =
 926:         RI.getMatchingSuperReg(RI.getSubReg(DestReg, SystemZ::subreg_h64),
 927:                                SystemZ::subreg_h64, &SystemZ::VR128BitRegClass);
 928:     MCRegister DestRegLo =
 929:         RI.getMatchingSuperReg(RI.getSubReg(DestReg, SystemZ::subreg_l64),
 930:                                SystemZ::subreg_h64, &SystemZ::VR128BitRegClass);
 931: 
 932:     if (DestRegHi != SrcReg.asMCReg())
 933:       copyPhysReg(MBB, MBBI, DL, DestRegHi, SrcReg, false);
 934:     BuildMI(MBB, MBBI, DL, get(SystemZ::VREPG), DestRegLo)
 935:       .addReg(SrcReg, getKillRegState(KillSrc)).addImm(1);
 936:     return;
 937:   }
 938: 
 939:   if (SystemZ::FP128BitRegClass.contains(DestReg) &&
 940:       SystemZ::GR128BitRegClass.contains(SrcReg)) {
 941:     MCRegister DestRegHi = RI.getSubReg(DestReg, SystemZ::subreg_h64);
 942:     MCRegister DestRegLo = RI.getSubReg(DestReg, SystemZ::subreg_l64);
 943:     MCRegister SrcRegHi = RI.getSubReg(SrcReg, SystemZ::subreg_h64);
 944:     MCRegister SrcRegLo = RI.getSubReg(SrcReg, SystemZ::subreg_l64);
 945: 
 946:     BuildMI(MBB, MBBI, DL, get(SystemZ::LDGR), DestRegHi)
 947:         .addReg(SrcRegHi)
 948:         .addReg(DestReg, RegState::ImplicitDefine);
 949: 
 950:     BuildMI(MBB, MBBI, DL, get(SystemZ::LDGR), DestRegLo)
 951:         .addReg(SrcRegLo, getKillRegState(KillSrc));
 952:     return;
 953:   }
 954: 
 955:   // Move CC value from a GR32.
 956:   if (DestReg == SystemZ::CC) {
 957:     unsigned Opcode =
 958:       SystemZ::GR32BitRegClass.contains(SrcReg) ? SystemZ::TMLH : SystemZ::TMHH;
 959:     BuildMI(MBB, MBBI, DL, get(Opcode))
 960:       .addReg(SrcReg, getKillRegState(KillSrc))
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:       .addImm(3 << (SystemZ::IPM_CC - 16));
 962:     return;
 963:   }
 964: 
 965:   if (SystemZ::GR128BitRegClass.contains(DestReg) &&
 966:       SystemZ::VR128BitRegClass.contains(SrcReg)) {
 967:     MCRegister DestH64 = RI.getSubReg(DestReg, SystemZ::subreg_h64);
 968:     MCRegister DestL64 = RI.getSubReg(DestReg, SystemZ::subreg_l64);
 969: 
 970:     BuildMI(MBB, MBBI, DL, get(SystemZ::VLGVG), DestH64)
 971:         .addReg(SrcReg)
 972:         .addReg(SystemZ::NoRegister)
 973:         .addImm(0)
 974:         .addDef(DestReg, RegState::Implicit);
 975:     BuildMI(MBB, MBBI, DL, get(SystemZ::VLGVG), DestL64)
 976:         .addReg(SrcReg, getKillRegState(KillSrc))
 977:         .addReg(SystemZ::NoRegister)
 978:         .addImm(1);
 979:     return;
 980:   }
 981: 
 982:   if (SystemZ::VR128BitRegClass.contains(DestReg) &&
 983:       SystemZ::GR128BitRegClass.contains(SrcReg)) {
 984:     BuildMI(MBB, MBBI, DL, get(SystemZ::VLVGP), DestReg)
 985:         .addReg(RI.getSubReg(SrcReg, SystemZ::subreg_h64))
 986:         .addReg(RI.getSubReg(SrcReg, SystemZ::subreg_l64));
 987:     return;
 988:   }
 989: 
 990:   // Everything else needs only one instruction.
 991:   unsigned Opcode;
 992:   if (SystemZ::GR64BitRegClass.contains(DestReg, SrcReg))
 993:     Opcode = SystemZ::LGR;
 994:   else if (SystemZ::FP16BitRegClass.contains(DestReg, SrcReg))
 995:     Opcode = STI.hasVector() ? SystemZ::LDR16 : SystemZ::LER16;
 996:   else if (SystemZ::FP32BitRegClass.contains(DestReg, SrcReg))
 997:     // For z13 we prefer LDR over LER to avoid partial register dependencies.
 998:     Opcode = STI.hasVector() ? SystemZ::LDR32 : SystemZ::LER;
 999:   else if (SystemZ::FP64BitRegClass.contains(DestReg, SrcReg))
1000:     Opcode = SystemZ::LDR;
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:   else if (SystemZ::FP128BitRegClass.contains(DestReg, SrcReg))
1002:     Opcode = SystemZ::LXR;
1003:   else if (SystemZ::VR16BitRegClass.contains(DestReg, SrcReg))
1004:     Opcode = SystemZ::VLR16;
1005:   else if (SystemZ::VR32BitRegClass.contains(DestReg, SrcReg))
1006:     Opcode = SystemZ::VLR32;
1007:   else if (SystemZ::VR64BitRegClass.contains(DestReg, SrcReg))
1008:     Opcode = SystemZ::VLR64;
1009:   else if (SystemZ::VR128BitRegClass.contains(DestReg, SrcReg))
1010:     Opcode = SystemZ::VLR;
1011:   else if (SystemZ::AR32BitRegClass.contains(DestReg, SrcReg))
1012:     Opcode = SystemZ::CPYA;
1013:   else if (SystemZ::GR64BitRegClass.contains(DestReg) &&
1014:            SystemZ::FP64BitRegClass.contains(SrcReg))
1015:     Opcode = SystemZ::LGDR;
1016:   else if (SystemZ::FP64BitRegClass.contains(DestReg) &&
1017:            SystemZ::GR64BitRegClass.contains(SrcReg))
1018:     Opcode = SystemZ::LDGR;
1019:   else
1020:     llvm_unreachable("Impossible reg-to-reg copy");
1021: 
1022:   BuildMI(MBB, MBBI, DL, get(Opcode), DestReg)
1023:     .addReg(SrcReg, getKillRegState(KillSrc));
1024: }
1025: 
1026: void SystemZInstrInfo::storeRegToStackSlot(
1027:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
1028:     bool isKill, int FrameIdx, const TargetRegisterClass *RC,
1029: 
1030:     Register VReg, MachineInstr::MIFlag Flags) const {
1031:   DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
1032: 
1033:   // Callers may expect a single instruction, so keep 128-bit moves
1034:   // together for now and lower them after register allocation.
1035:   unsigned LoadOpcode, StoreOpcode;
1036:   getLoadStoreOpcodes(RC, LoadOpcode, StoreOpcode);
1037:   addFrameReference(BuildMI(MBB, MBBI, DL, get(StoreOpcode))
1038:                         .addReg(SrcReg, getKillRegState(isKill)),
1039:                     FrameIdx);
1040: }
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZInstrInfo::storeRegToStackSlot`, `addFrameReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZInstrInfo::storeRegToStackSlot`, `addFrameReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041: 
1042: void SystemZInstrInfo::loadRegFromStackSlot(MachineBasicBlock &MBB,
1043:                                             MachineBasicBlock::iterator MBBI,
1044:                                             Register DestReg, int FrameIdx,
1045:                                             const TargetRegisterClass *RC,
1046:                                             Register VReg, unsigned SubReg,
1047:                                             MachineInstr::MIFlag Flags) const {
1048:   DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
1049: 
1050:   // Callers may expect a single instruction, so keep 128-bit moves
1051:   // together for now and lower them after register allocation.
1052:   unsigned LoadOpcode, StoreOpcode;
1053:   getLoadStoreOpcodes(RC, LoadOpcode, StoreOpcode);
1054:   addFrameReference(BuildMI(MBB, MBBI, DL, get(LoadOpcode), DestReg),
1055:                     FrameIdx);
1056: }
1057: 
1058: // Return true if MI is a simple load or store with a 12-bit displacement
1059: // and no index.  Flag is SimpleBDXLoad for loads and SimpleBDXStore for stores.
1060: static bool isSimpleBD12Move(const MachineInstr *MI, unsigned Flag) {
1061:   const MCInstrDesc &MCID = MI->getDesc();
1062:   return ((MCID.TSFlags & Flag) &&
1063:           isUInt<12>(MI->getOperand(2).getImm()) &&
1064:           MI->getOperand(3).getReg() == 0);
1065: }
1066: 
1067: namespace {
1068: 
1069: struct LogicOp {
1070:   LogicOp() = default;
1071:   LogicOp(unsigned regSize, unsigned immLSB, unsigned immSize)
1072:     : RegSize(regSize), ImmLSB(immLSB), ImmSize(immSize) {}
1073: 
1074:   explicit operator bool() const { return RegSize; }
1075: 
1076:   unsigned RegSize = 0;
1077:   unsigned ImmLSB = 0;
1078:   unsigned ImmSize = 0;
1079: };
1080: 
```
- **EN**: It introduces interface types such as `LogicOp`, shaping how other backend components interact with this file. The range implements or declares functions including `SystemZInstrInfo::loadRegFromStackSlot`, `isSimpleBD12Move`, `LogicOp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `LogicOp` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `SystemZInstrInfo::loadRegFromStackSlot`, `isSimpleBD12Move`, `LogicOp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081: } // end anonymous namespace
1082: 
1083: static LogicOp interpretAndImmediate(unsigned Opcode) {
1084:   switch (Opcode) {
1085:   case SystemZ::NILMux: return LogicOp(32,  0, 16);
1086:   case SystemZ::NIHMux: return LogicOp(32, 16, 16);
1087:   case SystemZ::NILL64: return LogicOp(64,  0, 16);
1088:   case SystemZ::NILH64: return LogicOp(64, 16, 16);
1089:   case SystemZ::NIHL64: return LogicOp(64, 32, 16);
1090:   case SystemZ::NIHH64: return LogicOp(64, 48, 16);
1091:   case SystemZ::NIFMux: return LogicOp(32,  0, 32);
1092:   case SystemZ::NILF64: return LogicOp(64,  0, 32);
1093:   case SystemZ::NIHF64: return LogicOp(64, 32, 32);
1094:   default:              return LogicOp();
1095:   }
1096: }
1097: 
1098: static void transferDeadCC(MachineInstr *OldMI, MachineInstr *NewMI) {
1099:   if (OldMI->registerDefIsDead(SystemZ::CC, /*TRI=*/nullptr)) {
1100:     MachineOperand *CCDef =
1101:         NewMI->findRegisterDefOperand(SystemZ::CC, /*TRI=*/nullptr);
1102:     if (CCDef != nullptr)
1103:       CCDef->setIsDead(true);
1104:   }
1105: }
1106: 
1107: static void transferMIFlag(MachineInstr *OldMI, MachineInstr *NewMI,
1108:                            MachineInstr::MIFlag Flag) {
1109:   if (OldMI->getFlag(Flag))
1110:     NewMI->setFlag(Flag);
1111: }
1112: 
1113: MachineInstr *
1114: SystemZInstrInfo::convertToThreeAddress(MachineInstr &MI, LiveVariables *LV,
1115:                                         LiveIntervals *LIS) const {
1116:   MachineBasicBlock *MBB = MI.getParent();
1117: 
1118:   // Try to convert an AND into an RISBG-type instruction.
1119:   // TODO: It might be beneficial to select RISBG and shorten to AND instead.
1120:   if (LogicOp And = interpretAndImmediate(MI.getOpcode())) {
```
- **EN**: The range implements or declares functions including `interpretAndImmediate`, `transferDeadCC`, `transferMIFlag`, `SystemZInstrInfo::convertToThreeAddress`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `interpretAndImmediate`, `transferDeadCC`, `transferMIFlag`, `SystemZInstrInfo::convertToThreeAddress` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121:     uint64_t Imm = MI.getOperand(2).getImm() << And.ImmLSB;
1122:     // AND IMMEDIATE leaves the other bits of the register unchanged.
1123:     Imm |= allOnes(And.RegSize) & ~(allOnes(And.ImmSize) << And.ImmLSB);
1124:     unsigned Start, End;
1125:     if (isRxSBGMask(Imm, And.RegSize, Start, End)) {
1126:       unsigned NewOpcode;
1127:       if (And.RegSize == 64) {
1128:         NewOpcode = SystemZ::RISBG;
1129:         // Prefer RISBGN if available, since it does not clobber CC.
1130:         if (STI.hasMiscellaneousExtensions())
1131:           NewOpcode = SystemZ::RISBGN;
1132:       } else {
1133:         NewOpcode = SystemZ::RISBMux;
1134:         Start &= 31;
1135:         End &= 31;
1136:       }
1137:       MachineOperand &Dest = MI.getOperand(0);
1138:       MachineOperand &Src = MI.getOperand(1);
1139:       MachineInstrBuilder MIB =
1140:           BuildMI(*MBB, MI, MI.getDebugLoc(), get(NewOpcode))
1141:               .add(Dest)
1142:               .addReg(0)
1143:               .addReg(Src.getReg(), getKillRegState(Src.isKill()),
1144:                       Src.getSubReg())
1145:               .addImm(Start)
1146:               .addImm(End + 128)
1147:               .addImm(0);
1148:       if (LV) {
1149:         unsigned NumOps = MI.getNumOperands();
1150:         for (unsigned I = 1; I < NumOps; ++I) {
1151:           MachineOperand &Op = MI.getOperand(I);
1152:           if (Op.isReg() && Op.isKill())
1153:             LV->replaceKillInstruction(Op.getReg(), MI, *MIB);
1154:         }
1155:       }
1156:       if (LIS)
1157:         LIS->ReplaceMachineInstrInMaps(MI, *MIB);
1158:       transferDeadCC(&MI, MIB);
1159:       return MIB;
1160:     }
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:   }
1162:   return nullptr;
1163: }
1164: 
1165: bool SystemZInstrInfo::isAssociativeAndCommutative(const MachineInstr &Inst,
1166:                                                    bool Invert) const {
1167:   unsigned Opc = Inst.getOpcode();
1168:   if (Invert) {
1169:     auto InverseOpcode = getInverseOpcode(Opc);
1170:     if (!InverseOpcode)
1171:       return false;
1172:     Opc = *InverseOpcode;
1173:   }
1174: 
1175:   switch (Opc) {
1176:   default:
1177:     break;
1178:   // Adds and multiplications.
1179:   case SystemZ::WFADB:
1180:   case SystemZ::WFASB:
1181:   case SystemZ::WFAXB:
1182:   case SystemZ::VFADB:
1183:   case SystemZ::VFASB:
1184:   case SystemZ::WFMDB:
1185:   case SystemZ::WFMSB:
1186:   case SystemZ::WFMXB:
1187:   case SystemZ::VFMDB:
1188:   case SystemZ::VFMSB:
1189:     return (Inst.getFlag(MachineInstr::MIFlag::FmReassoc) &&
1190:             Inst.getFlag(MachineInstr::MIFlag::FmNsz));
1191:   }
1192: 
1193:   return false;
1194: }
1195: 
1196: std::optional<unsigned>
1197: SystemZInstrInfo::getInverseOpcode(unsigned Opcode) const {
1198:   // fadd => fsub
1199:   switch (Opcode) {
1200:   case SystemZ::WFADB:
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::isAssociativeAndCommutative`, `SystemZInstrInfo::getInverseOpcode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::isAssociativeAndCommutative`, `SystemZInstrInfo::getInverseOpcode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:     return SystemZ::WFSDB;
1202:   case SystemZ::WFASB:
1203:     return SystemZ::WFSSB;
1204:   case SystemZ::WFAXB:
1205:     return SystemZ::WFSXB;
1206:   case SystemZ::VFADB:
1207:     return SystemZ::VFSDB;
1208:   case SystemZ::VFASB:
1209:     return SystemZ::VFSSB;
1210:   // fsub => fadd
1211:   case SystemZ::WFSDB:
1212:     return SystemZ::WFADB;
1213:   case SystemZ::WFSSB:
1214:     return SystemZ::WFASB;
1215:   case SystemZ::WFSXB:
1216:     return SystemZ::WFAXB;
1217:   case SystemZ::VFSDB:
1218:     return SystemZ::VFADB;
1219:   case SystemZ::VFSSB:
1220:     return SystemZ::VFASB;
1221:   default:
1222:     return std::nullopt;
1223:   }
1224: }
1225: 
1226: MachineInstr *SystemZInstrInfo::foldMemoryOperandImpl(
1227:     MachineFunction &MF, MachineInstr &MI, ArrayRef<unsigned> Ops,
1228:     int FrameIndex, MachineInstr *&CopyMI, LiveIntervals *LIS,
1229:     VirtRegMap *VRM) const {
1230:   MachineBasicBlock::iterator InsertPt = MI;
1231:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
1232:   MachineRegisterInfo &MRI = MF.getRegInfo();
1233:   const MachineFrameInfo &MFI = MF.getFrameInfo();
1234:   unsigned Size = MFI.getObjectSize(FrameIndex);
1235:   unsigned Opcode = MI.getOpcode();
1236: 
1237:   // Check CC liveness if new instruction introduces a dead def of CC.
1238:   SlotIndex MISlot = SlotIndex();
1239:   LiveRange *CCLiveRange = nullptr;
1240:   bool CCLiveAtMI = true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241:   if (LIS) {
1242:     MISlot = LIS->getSlotIndexes()->getInstructionIndex(MI).getRegSlot();
1243:     auto CCUnits = TRI->regunits(MCRegister::from(SystemZ::CC));
1244:     assert(range_size(CCUnits) == 1 && "CC only has one reg unit.");
1245:     CCLiveRange = &LIS->getRegUnit(*CCUnits.begin());
1246:     CCLiveAtMI = CCLiveRange->liveAt(MISlot);
1247:   }
1248: 
1249:   if (Ops.size() == 2 && Ops[0] == 0 && Ops[1] == 1) {
1250:     if (!CCLiveAtMI && (Opcode == SystemZ::LA || Opcode == SystemZ::LAY) &&
1251:         isInt<8>(MI.getOperand(2).getImm()) && !MI.getOperand(3).getReg()) {
1252:       // LA(Y) %reg, CONST(%reg) -> AGSI %mem, CONST
1253:       MachineInstr *BuiltMI = BuildMI(*InsertPt->getParent(), InsertPt,
1254:                                       MI.getDebugLoc(), get(SystemZ::AGSI))
1255:         .addFrameIndex(FrameIndex)
1256:         .addImm(0)
1257:         .addImm(MI.getOperand(2).getImm());
1258:       BuiltMI->findRegisterDefOperand(SystemZ::CC, /*TRI=*/nullptr)
1259:           ->setIsDead(true);
1260:       CCLiveRange->createDeadDef(MISlot, LIS->getVNInfoAllocator());
1261:       return BuiltMI;
1262:     }
1263:     return nullptr;
1264:   }
1265: 
1266:   // All other cases require a single operand.
1267:   if (Ops.size() != 1)
1268:     return nullptr;
1269: 
1270:   unsigned OpNum = Ops[0];
1271:   const TargetRegisterClass *RC =
1272:       MF.getRegInfo().getRegClass(MI.getOperand(OpNum).getReg());
1273:   assert((Size * 8 == TRI->getRegSizeInBits(*RC) ||
1274:           (RC == &SystemZ::FP16BitRegClass && Size == 4 && !STI.hasVector())) &&
1275:          "Invalid size combination");
1276:   (void)RC;
1277: 
1278:   if ((Opcode == SystemZ::AHI || Opcode == SystemZ::AGHI) && OpNum == 0 &&
1279:       isInt<8>(MI.getOperand(2).getImm())) {
1280:     // A(G)HI %reg, CONST -> A(G)SI %mem, CONST
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281:     Opcode = (Opcode == SystemZ::AHI ? SystemZ::ASI : SystemZ::AGSI);
1282:     MachineInstr *BuiltMI =
1283:         BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(), get(Opcode))
1284:             .addFrameIndex(FrameIndex)
1285:             .addImm(0)
1286:             .addImm(MI.getOperand(2).getImm());
1287:     transferDeadCC(&MI, BuiltMI);
1288:     transferMIFlag(&MI, BuiltMI, MachineInstr::NoSWrap);
1289:     return BuiltMI;
1290:   }
1291: 
1292:   if ((Opcode == SystemZ::ALFI && OpNum == 0 &&
1293:        isInt<8>((int32_t)MI.getOperand(2).getImm())) ||
1294:       (Opcode == SystemZ::ALGFI && OpNum == 0 &&
1295:        isInt<8>(MI.getOperand(2).getImm()))) {
1296:     // AL(G)FI %reg, CONST -> AL(G)SI %mem, CONST
1297:     Opcode = (Opcode == SystemZ::ALFI ? SystemZ::ALSI : SystemZ::ALGSI);
1298:     MachineInstr *BuiltMI =
1299:         BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(), get(Opcode))
1300:             .addFrameIndex(FrameIndex)
1301:             .addImm(0)
1302:             .addImm((int8_t)MI.getOperand(2).getImm());
1303:     transferDeadCC(&MI, BuiltMI);
1304:     return BuiltMI;
1305:   }
1306: 
1307:   if ((Opcode == SystemZ::SLFI && OpNum == 0 &&
1308:        isInt<8>((int32_t)-MI.getOperand(2).getImm())) ||
1309:       (Opcode == SystemZ::SLGFI && OpNum == 0 &&
1310:        isInt<8>((-MI.getOperand(2).getImm())))) {
1311:     // SL(G)FI %reg, CONST -> AL(G)SI %mem, -CONST
1312:     Opcode = (Opcode == SystemZ::SLFI ? SystemZ::ALSI : SystemZ::ALGSI);
1313:     MachineInstr *BuiltMI =
1314:         BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(), get(Opcode))
1315:             .addFrameIndex(FrameIndex)
1316:             .addImm(0)
1317:             .addImm((int8_t)-MI.getOperand(2).getImm());
1318:     transferDeadCC(&MI, BuiltMI);
1319:     return BuiltMI;
1320:   }
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321: 
1322:   unsigned MemImmOpc = 0;
1323:   switch (Opcode) {
1324:   case SystemZ::LHIMux:
1325:   case SystemZ::LHI:    MemImmOpc = SystemZ::MVHI;  break;
1326:   case SystemZ::LGHI:   MemImmOpc = SystemZ::MVGHI; break;
1327:   case SystemZ::CHIMux:
1328:   case SystemZ::CHI:    MemImmOpc = SystemZ::CHSI;  break;
1329:   case SystemZ::CGHI:   MemImmOpc = SystemZ::CGHSI; break;
1330:   case SystemZ::CLFIMux:
1331:   case SystemZ::CLFI:
1332:     if (isUInt<16>(MI.getOperand(1).getImm()))
1333:       MemImmOpc = SystemZ::CLFHSI;
1334:     break;
1335:   case SystemZ::CLGFI:
1336:     if (isUInt<16>(MI.getOperand(1).getImm()))
1337:       MemImmOpc = SystemZ::CLGHSI;
1338:     break;
1339:   default: break;
1340:   }
1341:   if (MemImmOpc)
1342:     return BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(),
1343:                    get(MemImmOpc))
1344:                .addFrameIndex(FrameIndex)
1345:                .addImm(0)
1346:                .addImm(MI.getOperand(1).getImm());
1347: 
1348:   if (Opcode == SystemZ::LGDR || Opcode == SystemZ::LDGR) {
1349:     bool Op0IsGPR = (Opcode == SystemZ::LGDR);
1350:     bool Op1IsGPR = (Opcode == SystemZ::LDGR);
1351:     // If we're spilling the destination of an LDGR or LGDR, store the
1352:     // source register instead.
1353:     if (OpNum == 0) {
1354:       unsigned StoreOpcode = Op1IsGPR ? SystemZ::STG : SystemZ::STD;
1355:       return BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(),
1356:                      get(StoreOpcode))
1357:           .add(MI.getOperand(1))
1358:           .addFrameIndex(FrameIndex)
1359:           .addImm(0)
1360:           .addReg(0);
```
- **EN**: The range implements or declares functions including `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:     }
1362:     // If we're spilling the source of an LDGR or LGDR, load the
1363:     // destination register instead.
1364:     if (OpNum == 1) {
1365:       unsigned LoadOpcode = Op0IsGPR ? SystemZ::LG : SystemZ::LD;
1366:       return BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(),
1367:                      get(LoadOpcode))
1368:         .add(MI.getOperand(0))
1369:         .addFrameIndex(FrameIndex)
1370:         .addImm(0)
1371:         .addReg(0);
1372:     }
1373:   }
1374: 
1375:   // Look for cases where the source of a simple store or the destination
1376:   // of a simple load is being spilled.  Try to use MVC instead.
1377:   //
1378:   // Although MVC is in practice a fast choice in these cases, it is still
1379:   // logically a bytewise copy.  This means that we cannot use it if the
1380:   // load or store is volatile.  We also wouldn't be able to use MVC if
1381:   // the two memories partially overlap, but that case cannot occur here,
1382:   // because we know that one of the memories is a full frame index.
1383:   //
1384:   // For performance reasons, we also want to avoid using MVC if the addresses
1385:   // might be equal.  We don't worry about that case here, because spill slot
1386:   // coloring happens later, and because we have special code to remove
1387:   // MVCs that turn out to be redundant.
1388:   if (OpNum == 0 && MI.hasOneMemOperand()) {
1389:     MachineMemOperand *MMO = *MI.memoperands_begin();
1390:     if (MMO->getSize() == Size && !MMO->isVolatile() && !MMO->isAtomic()) {
1391:       // Handle conversion of loads.
1392:       if (isSimpleBD12Move(&MI, SystemZII::SimpleBDXLoad)) {
1393:         return BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(),
1394:                        get(SystemZ::MVC))
1395:             .addFrameIndex(FrameIndex)
1396:             .addImm(0)
1397:             .addImm(Size)
1398:             .add(MI.getOperand(1))
1399:             .addImm(MI.getOperand(2).getImm())
1400:             .addMemOperand(MMO);
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401:       }
1402:       // Handle conversion of stores.
1403:       if (isSimpleBD12Move(&MI, SystemZII::SimpleBDXStore)) {
1404:         return BuildMI(*InsertPt->getParent(), InsertPt, MI.getDebugLoc(),
1405:                        get(SystemZ::MVC))
1406:             .add(MI.getOperand(1))
1407:             .addImm(MI.getOperand(2).getImm())
1408:             .addImm(Size)
1409:             .addFrameIndex(FrameIndex)
1410:             .addImm(0)
1411:             .addMemOperand(MMO);
1412:       }
1413:     }
1414:   }
1415: 
1416:   // If the spilled operand is the final one or the instruction is
1417:   // commutable, try to change <INSN>R into <INSN>.  Don't introduce a def of
1418:   // CC if it is live and MI does not define it.
1419:   unsigned NumOps = MI.getNumExplicitOperands();
1420:   int MemOpcode = SystemZ::getMemOpcode(Opcode);
1421:   if (MemOpcode == -1 ||
1422:       (CCLiveAtMI && !MI.definesRegister(SystemZ::CC, /*TRI=*/nullptr) &&
1423:        get(MemOpcode).hasImplicitDefOfPhysReg(SystemZ::CC)))
1424:     return nullptr;
1425: 
1426:   // Check if all other vregs have a usable allocation in the case of vector
1427:   // to FP conversion.
1428:   const MCInstrDesc &MCID = MI.getDesc();
1429:   for (unsigned I = 0, E = MCID.getNumOperands(); I != E; ++I) {
1430:     const MCOperandInfo &MCOI = MCID.operands()[I];
1431:     if (MCOI.OperandType != MCOI::OPERAND_REGISTER || I == OpNum)
1432:       continue;
1433:     const TargetRegisterClass *RC = TRI->getRegClass(MCOI.RegClass);
1434:     if (RC == &SystemZ::VR32BitRegClass || RC == &SystemZ::VR64BitRegClass) {
1435:       Register Reg = MI.getOperand(I).getReg();
1436:       Register PhysReg = Reg.isVirtual()
1437:                              ? (VRM ? Register(VRM->getPhys(Reg)) : Register())
1438:                              : Reg;
1439:       if (!PhysReg ||
1440:           !(SystemZ::FP32BitRegClass.contains(PhysReg) ||
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441:             SystemZ::FP64BitRegClass.contains(PhysReg) ||
1442:             SystemZ::VF128BitRegClass.contains(PhysReg)))
1443:         return nullptr;
1444:     }
1445:   }
1446:   // Fused multiply and add/sub need to have the same dst and accumulator reg.
1447:   bool FusedFPOp = (Opcode == SystemZ::WFMADB || Opcode == SystemZ::WFMASB ||
1448:                     Opcode == SystemZ::WFMSDB || Opcode == SystemZ::WFMSSB);
1449:   if (FusedFPOp) {
1450:     Register DstReg = VRM->getPhys(MI.getOperand(0).getReg());
1451:     Register AccReg = VRM->getPhys(MI.getOperand(3).getReg());
1452:     if (OpNum == 0 || OpNum == 3 || DstReg != AccReg)
1453:       return nullptr;
1454:   }
1455: 
1456:   // Try to swap compare operands if possible.
1457:   bool NeedsCommute = false;
1458:   if ((MI.getOpcode() == SystemZ::CR || MI.getOpcode() == SystemZ::CGR ||
1459:        MI.getOpcode() == SystemZ::CLR || MI.getOpcode() == SystemZ::CLGR ||
1460:        MI.getOpcode() == SystemZ::WFCDB || MI.getOpcode() == SystemZ::WFCSB ||
1461:        MI.getOpcode() == SystemZ::WFKDB || MI.getOpcode() == SystemZ::WFKSB) &&
1462:       OpNum == 0 && prepareCompareSwapOperands(MI))
1463:     NeedsCommute = true;
1464: 
1465:   bool CCOperands = false;
1466:   if (MI.getOpcode() == SystemZ::LOCRMux || MI.getOpcode() == SystemZ::LOCGR ||
1467:       MI.getOpcode() == SystemZ::SELRMux || MI.getOpcode() == SystemZ::SELGR) {
1468:     assert(MI.getNumOperands() == 6 && NumOps == 5 &&
1469:            "LOCR/SELR instruction operands corrupt?");
1470:     NumOps -= 2;
1471:     CCOperands = true;
1472:   }
1473: 
1474:   // See if this is a 3-address instruction that is convertible to 2-address
1475:   // and suitable for folding below.  Only try this with virtual registers
1476:   // and a provided VRM (during regalloc).
1477:   if (NumOps == 3 && SystemZ::getTargetMemOpcode(MemOpcode) != -1) {
1478:     if (VRM == nullptr)
1479:       return nullptr;
1480:     else {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481:       Register DstReg = MI.getOperand(0).getReg();
1482:       Register DstPhys =
1483:           (DstReg.isVirtual() ? Register(VRM->getPhys(DstReg)) : DstReg);
1484:       Register SrcReg = (OpNum == 2 ? MI.getOperand(1).getReg()
1485:                                     : ((OpNum == 1 && MI.isCommutable())
1486:                                            ? MI.getOperand(2).getReg()
1487:                                            : Register()));
1488:       if (DstPhys && !SystemZ::GRH32BitRegClass.contains(DstPhys) && SrcReg &&
1489:           SrcReg.isVirtual() && DstPhys == VRM->getPhys(SrcReg))
1490:         NeedsCommute = (OpNum == 1);
1491:       else
1492:         return nullptr;
1493:     }
1494:   }
1495: 
1496:   if ((OpNum == NumOps - 1) || NeedsCommute || FusedFPOp) {
1497:     const MCInstrDesc &MemDesc = get(MemOpcode);
1498:     uint64_t AccessBytes = SystemZII::getAccessSize(MemDesc.TSFlags);
1499:     assert(AccessBytes != 0 && "Size of access should be known");
1500:     assert(AccessBytes <= Size && "Access outside the frame index");
1501:     uint64_t Offset = Size - AccessBytes;
1502:     MachineInstrBuilder MIB = BuildMI(*InsertPt->getParent(), InsertPt,
1503:                                       MI.getDebugLoc(), get(MemOpcode));
1504:     if (MI.isCompare()) {
1505:       assert(NumOps == 2 && "Expected 2 register operands for a compare.");
1506:       MIB.add(MI.getOperand(NeedsCommute ? 1 : 0));
1507:     }
1508:     else if (FusedFPOp) {
1509:       MIB.add(MI.getOperand(0));
1510:       MIB.add(MI.getOperand(3));
1511:       MIB.add(MI.getOperand(OpNum == 1 ? 2 : 1));
1512:     }
1513:     else {
1514:       MIB.add(MI.getOperand(0));
1515:       if (NeedsCommute)
1516:         MIB.add(MI.getOperand(2));
1517:       else
1518:         for (unsigned I = 1; I < OpNum; ++I)
1519:           MIB.add(MI.getOperand(I));
1520:     }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521:     MIB.addFrameIndex(FrameIndex).addImm(Offset);
1522:     if (MemDesc.TSFlags & SystemZII::HasIndex)
1523:       MIB.addReg(0);
1524:     if (CCOperands) {
1525:       unsigned CCValid = MI.getOperand(NumOps).getImm();
1526:       unsigned CCMask = MI.getOperand(NumOps + 1).getImm();
1527:       MIB.addImm(CCValid);
1528:       MIB.addImm(NeedsCommute ? CCMask ^ CCValid : CCMask);
1529:     }
1530:     if (MIB->definesRegister(SystemZ::CC, /*TRI=*/nullptr) &&
1531:         (!MI.definesRegister(SystemZ::CC, /*TRI=*/nullptr) ||
1532:          MI.registerDefIsDead(SystemZ::CC, /*TRI=*/nullptr))) {
1533:       MIB->addRegisterDead(SystemZ::CC, TRI);
1534:       if (CCLiveRange)
1535:         CCLiveRange->createDeadDef(MISlot, LIS->getVNInfoAllocator());
1536:     }
1537:     // Constrain the register classes if converted from a vector opcode. The
1538:     // allocated regs are in an FP reg-class per previous check above.
1539:     for (const MachineOperand &MO : MIB->operands())
1540:       if (MO.isReg() && MO.getReg().isVirtual()) {
1541:         Register Reg = MO.getReg();
1542:         if (MRI.getRegClass(Reg) == &SystemZ::VR32BitRegClass)
1543:           MRI.setRegClass(Reg, &SystemZ::FP32BitRegClass);
1544:         else if (MRI.getRegClass(Reg) == &SystemZ::VR64BitRegClass)
1545:           MRI.setRegClass(Reg, &SystemZ::FP64BitRegClass);
1546:         else if (MRI.getRegClass(Reg) == &SystemZ::VR128BitRegClass)
1547:           MRI.setRegClass(Reg, &SystemZ::VF128BitRegClass);
1548:       }
1549: 
1550:     transferDeadCC(&MI, MIB);
1551:     transferMIFlag(&MI, MIB, MachineInstr::NoSWrap);
1552:     transferMIFlag(&MI, MIB, MachineInstr::NoFPExcept);
1553:     return MIB;
1554:   }
1555: 
1556:   return nullptr;
1557: }
1558: 
1559: MachineInstr *SystemZInstrInfo::foldMemoryOperandImpl(
1560:     MachineFunction &MF, MachineInstr &MI, ArrayRef<unsigned> Ops,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1561-1600 / 第 1561-1600 行
```cpp
1561:     MachineInstr &LoadMI, MachineInstr *&CopyMI, LiveIntervals *LIS,
1562:     VirtRegMap *VRM) const {
1563:   MachineBasicBlock::iterator InsertPt = MI;
1564:   MachineRegisterInfo *MRI = &MF.getRegInfo();
1565:   MachineBasicBlock *MBB = MI.getParent();
1566: 
1567:   // For reassociable FP operations, any loads have been purposefully left
1568:   // unfolded so that MachineCombiner can do its work on reg/reg
1569:   // opcodes. After that, as many loads as possible are now folded.
1570:   // TODO: This may be beneficial with other opcodes as well as machine-sink
1571:   // can move loads close to their user in a different MBB, which the isel
1572:   // matcher did not see.
1573:   unsigned LoadOpc = 0;
1574:   unsigned RegMemOpcode = 0;
1575:   const TargetRegisterClass *FPRC = nullptr;
1576:   RegMemOpcode = MI.getOpcode() == SystemZ::WFADB   ? SystemZ::ADB
1577:                  : MI.getOpcode() == SystemZ::WFSDB ? SystemZ::SDB
1578:                  : MI.getOpcode() == SystemZ::WFMDB ? SystemZ::MDB
1579:                                                     : 0;
1580:   if (RegMemOpcode) {
1581:     LoadOpc = SystemZ::VL64;
1582:     FPRC = &SystemZ::FP64BitRegClass;
1583:   } else {
1584:     RegMemOpcode = MI.getOpcode() == SystemZ::WFASB   ? SystemZ::AEB
1585:                    : MI.getOpcode() == SystemZ::WFSSB ? SystemZ::SEB
1586:                    : MI.getOpcode() == SystemZ::WFMSB ? SystemZ::MEEB
1587:                                                       : 0;
1588:     if (RegMemOpcode) {
1589:       LoadOpc = SystemZ::VL32;
1590:       FPRC = &SystemZ::FP32BitRegClass;
1591:     }
1592:   }
1593:   if (!RegMemOpcode || LoadMI.getOpcode() != LoadOpc)
1594:     return nullptr;
1595: 
1596:   // If RegMemOpcode clobbers CC, first make sure CC is not live at this point.
1597:   if (get(RegMemOpcode).hasImplicitDefOfPhysReg(SystemZ::CC)) {
1598:     for (MachineBasicBlock::iterator MII = InsertPt;;) {
1599:       if (MII == MBB->begin()) {
1600:         if (MBB->isLiveIn(SystemZ::CC))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1601-1640 / 第 1601-1640 行
```cpp
1601:           return nullptr;
1602:         break;
1603:       }
1604:       --MII;
1605:       if (MII->definesRegister(SystemZ::CC, /*TRI=*/nullptr)) {
1606:         if (!MII->registerDefIsDead(SystemZ::CC, /*TRI=*/nullptr))
1607:           return nullptr;
1608:         break;
1609:       }
1610:     }
1611:   }
1612: 
1613:   Register FoldAsLoadDefReg = LoadMI.getOperand(0).getReg();
1614:   if (Ops.size() != 1 || FoldAsLoadDefReg != MI.getOperand(Ops[0]).getReg())
1615:     return nullptr;
1616:   Register DstReg = MI.getOperand(0).getReg();
1617:   MachineOperand LHS = MI.getOperand(1);
1618:   MachineOperand RHS = MI.getOperand(2);
1619:   MachineOperand &RegMO = RHS.getReg() == FoldAsLoadDefReg ? LHS : RHS;
1620:   if ((RegMemOpcode == SystemZ::SDB || RegMemOpcode == SystemZ::SEB) &&
1621:       FoldAsLoadDefReg != RHS.getReg())
1622:     return nullptr;
1623: 
1624:   MachineOperand &Base = LoadMI.getOperand(1);
1625:   MachineOperand &Disp = LoadMI.getOperand(2);
1626:   MachineOperand &Indx = LoadMI.getOperand(3);
1627:   MachineInstrBuilder MIB =
1628:       BuildMI(*MI.getParent(), InsertPt, MI.getDebugLoc(), get(RegMemOpcode), DstReg)
1629:           .add(RegMO)
1630:           .add(Base)
1631:           .add(Disp)
1632:           .add(Indx);
1633:   MIB->addRegisterDead(SystemZ::CC, &RI);
1634:   MRI->setRegClass(DstReg, FPRC);
1635:   MRI->setRegClass(RegMO.getReg(), FPRC);
1636:   transferMIFlag(&MI, MIB, MachineInstr::NoFPExcept);
1637: 
1638:   return MIB;
1639: }
1640: 
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1641-1680 / 第 1641-1680 行
```cpp
1641: bool SystemZInstrInfo::expandPostRAPseudo(MachineInstr &MI) const {
1642:   switch (MI.getOpcode()) {
1643:   case SystemZ::L128:
1644:     splitMove(MI, SystemZ::LG);
1645:     return true;
1646: 
1647:   case SystemZ::ST128:
1648:     splitMove(MI, SystemZ::STG);
1649:     return true;
1650: 
1651:   case SystemZ::LX:
1652:     splitMove(MI, SystemZ::LD);
1653:     return true;
1654: 
1655:   case SystemZ::STX:
1656:     splitMove(MI, SystemZ::STD);
1657:     return true;
1658: 
1659:   case SystemZ::LBMux:
1660:     expandRXYPseudo(MI, SystemZ::LB, SystemZ::LBH);
1661:     return true;
1662: 
1663:   case SystemZ::LHMux:
1664:     expandRXYPseudo(MI, SystemZ::LH, SystemZ::LHH);
1665:     return true;
1666: 
1667:   case SystemZ::LLCRMux:
1668:     expandZExtPseudo(MI, SystemZ::LLCR, 8);
1669:     return true;
1670: 
1671:   case SystemZ::LLHRMux:
1672:     expandZExtPseudo(MI, SystemZ::LLHR, 16);
1673:     return true;
1674: 
1675:   case SystemZ::LLCMux:
1676:     expandRXYPseudo(MI, SystemZ::LLC, SystemZ::LLCH);
1677:     return true;
1678: 
1679:   case SystemZ::LLHMux:
1680:     expandRXYPseudo(MI, SystemZ::LLH, SystemZ::LLHH);
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::expandPostRAPseudo`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::expandPostRAPseudo` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 1681-1720 / 第 1681-1720 行
```cpp
1681:     return true;
1682: 
1683:   case SystemZ::LMux:
1684:     expandRXYPseudo(MI, SystemZ::L, SystemZ::LFH);
1685:     return true;
1686: 
1687:   case SystemZ::LOCMux:
1688:     expandLOCPseudo(MI, SystemZ::LOC, SystemZ::LOCFH);
1689:     return true;
1690: 
1691:   case SystemZ::LOCHIMux:
1692:     expandLOCPseudo(MI, SystemZ::LOCHI, SystemZ::LOCHHI);
1693:     return true;
1694: 
1695:   case SystemZ::STCMux:
1696:     expandRXYPseudo(MI, SystemZ::STC, SystemZ::STCH);
1697:     return true;
1698: 
1699:   case SystemZ::STHMux:
1700:     expandRXYPseudo(MI, SystemZ::STH, SystemZ::STHH);
1701:     return true;
1702: 
1703:   case SystemZ::STMux:
1704:     expandRXYPseudo(MI, SystemZ::ST, SystemZ::STFH);
1705:     return true;
1706: 
1707:   case SystemZ::STOCMux:
1708:     expandLOCPseudo(MI, SystemZ::STOC, SystemZ::STOCFH);
1709:     return true;
1710: 
1711:   case SystemZ::LHIMux:
1712:     expandRIPseudo(MI, SystemZ::LHI, SystemZ::IIHF, true);
1713:     return true;
1714: 
1715:   case SystemZ::IIFMux:
1716:     expandRIPseudo(MI, SystemZ::IILF, SystemZ::IIHF, false);
1717:     return true;
1718: 
1719:   case SystemZ::IILMux:
1720:     expandRIPseudo(MI, SystemZ::IILL, SystemZ::IIHL, false);
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1721-1760 / 第 1721-1760 行
```cpp
1721:     return true;
1722: 
1723:   case SystemZ::IIHMux:
1724:     expandRIPseudo(MI, SystemZ::IILH, SystemZ::IIHH, false);
1725:     return true;
1726: 
1727:   case SystemZ::NIFMux:
1728:     expandRIPseudo(MI, SystemZ::NILF, SystemZ::NIHF, false);
1729:     return true;
1730: 
1731:   case SystemZ::NILMux:
1732:     expandRIPseudo(MI, SystemZ::NILL, SystemZ::NIHL, false);
1733:     return true;
1734: 
1735:   case SystemZ::NIHMux:
1736:     expandRIPseudo(MI, SystemZ::NILH, SystemZ::NIHH, false);
1737:     return true;
1738: 
1739:   case SystemZ::OIFMux:
1740:     expandRIPseudo(MI, SystemZ::OILF, SystemZ::OIHF, false);
1741:     return true;
1742: 
1743:   case SystemZ::OILMux:
1744:     expandRIPseudo(MI, SystemZ::OILL, SystemZ::OIHL, false);
1745:     return true;
1746: 
1747:   case SystemZ::OIHMux:
1748:     expandRIPseudo(MI, SystemZ::OILH, SystemZ::OIHH, false);
1749:     return true;
1750: 
1751:   case SystemZ::XIFMux:
1752:     expandRIPseudo(MI, SystemZ::XILF, SystemZ::XIHF, false);
1753:     return true;
1754: 
1755:   case SystemZ::TMLMux:
1756:     expandRIPseudo(MI, SystemZ::TMLL, SystemZ::TMHL, false);
1757:     return true;
1758: 
1759:   case SystemZ::TMHMux:
1760:     expandRIPseudo(MI, SystemZ::TMLH, SystemZ::TMHH, false);
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1761-1800 / 第 1761-1800 行
```cpp
1761:     return true;
1762: 
1763:   case SystemZ::AHIMux:
1764:     expandRIPseudo(MI, SystemZ::AHI, SystemZ::AIH, false);
1765:     return true;
1766: 
1767:   case SystemZ::AHIMuxK:
1768:     expandRIEPseudo(MI, SystemZ::AHI, SystemZ::AHIK, SystemZ::AIH);
1769:     return true;
1770: 
1771:   case SystemZ::AFIMux:
1772:     expandRIPseudo(MI, SystemZ::AFI, SystemZ::AIH, false);
1773:     return true;
1774: 
1775:   case SystemZ::CHIMux:
1776:     expandRIPseudo(MI, SystemZ::CHI, SystemZ::CIH, false);
1777:     return true;
1778: 
1779:   case SystemZ::CFIMux:
1780:     expandRIPseudo(MI, SystemZ::CFI, SystemZ::CIH, false);
1781:     return true;
1782: 
1783:   case SystemZ::CLFIMux:
1784:     expandRIPseudo(MI, SystemZ::CLFI, SystemZ::CLIH, false);
1785:     return true;
1786: 
1787:   case SystemZ::CMux:
1788:     expandRXYPseudo(MI, SystemZ::C, SystemZ::CHF);
1789:     return true;
1790: 
1791:   case SystemZ::CLMux:
1792:     expandRXYPseudo(MI, SystemZ::CL, SystemZ::CLHF);
1793:     return true;
1794: 
1795:   case SystemZ::RISBMux: {
1796:     bool DestIsHigh = SystemZ::isHighReg(MI.getOperand(0).getReg());
1797:     bool SrcIsHigh = SystemZ::isHighReg(MI.getOperand(2).getReg());
1798:     if (SrcIsHigh == DestIsHigh)
1799:       MI.setDesc(get(DestIsHigh ? SystemZ::RISBHH : SystemZ::RISBLL));
1800:     else {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1801-1840 / 第 1801-1840 行
```cpp
1801:       MI.setDesc(get(DestIsHigh ? SystemZ::RISBHL : SystemZ::RISBLH));
1802:       MI.getOperand(5).setImm(MI.getOperand(5).getImm() ^ 32);
1803:     }
1804:     return true;
1805:   }
1806: 
1807:   case SystemZ::ADJDYNALLOC:
1808:     splitAdjDynAlloc(MI);
1809:     return true;
1810: 
1811:   case TargetOpcode::LOAD_STACK_GUARD:
1812:     expandLoadStackGuard(&MI);
1813:     return true;
1814: 
1815:   default:
1816:     return false;
1817:   }
1818: }
1819: 
1820: unsigned SystemZInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
1821:   if (MI.isInlineAsm()) {
1822:     const MachineFunction *MF = MI.getParent()->getParent();
1823:     const char *AsmStr = MI.getOperand(0).getSymbolName();
1824:     return getInlineAsmLength(AsmStr, MF->getTarget().getMCAsmInfo());
1825:   }
1826:   else if (MI.getOpcode() == SystemZ::PATCHPOINT)
1827:     return PatchPointOpers(&MI).getNumPatchBytes();
1828:   else if (MI.getOpcode() == SystemZ::STACKMAP)
1829:     return MI.getOperand(1).getImm();
1830:   else if (MI.getOpcode() == SystemZ::FENTRY_CALL)
1831:     return 6;
1832:   if (MI.getOpcode() == TargetOpcode::PATCHABLE_FUNCTION_ENTER)
1833:     return 18;
1834:   if (MI.getOpcode() == TargetOpcode::PATCHABLE_RET)
1835:     return 18 + (MI.getOperand(0).getImm() == SystemZ::CondReturn ? 4 : 0);
1836:   if (MI.getOpcode() == TargetOpcode::BUNDLE)
1837:     return getInstBundleSize(MI);
1838: 
1839:   return MI.getDesc().getSize();
1840: }
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getInstSizeInBytes`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getInstSizeInBytes` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1841-1880 / 第 1841-1880 行
```cpp
1841: 
1842: SystemZII::Branch
1843: SystemZInstrInfo::getBranchInfo(const MachineInstr &MI) const {
1844:   switch (MI.getOpcode()) {
1845:   case SystemZ::BR:
1846:   case SystemZ::BI:
1847:   case SystemZ::J:
1848:   case SystemZ::JG:
1849:     return SystemZII::Branch(SystemZII::BranchNormal, SystemZ::CCMASK_ANY,
1850:                              SystemZ::CCMASK_ANY, &MI.getOperand(0));
1851: 
1852:   case SystemZ::BRC:
1853:   case SystemZ::BRCL:
1854:     return SystemZII::Branch(SystemZII::BranchNormal, MI.getOperand(0).getImm(),
1855:                              MI.getOperand(1).getImm(), &MI.getOperand(2));
1856: 
1857:   case SystemZ::BRCT:
1858:   case SystemZ::BRCTH:
1859:     return SystemZII::Branch(SystemZII::BranchCT, SystemZ::CCMASK_ICMP,
1860:                              SystemZ::CCMASK_CMP_NE, &MI.getOperand(2));
1861: 
1862:   case SystemZ::BRCTG:
1863:     return SystemZII::Branch(SystemZII::BranchCTG, SystemZ::CCMASK_ICMP,
1864:                              SystemZ::CCMASK_CMP_NE, &MI.getOperand(2));
1865: 
1866:   case SystemZ::CIJ:
1867:   case SystemZ::CRJ:
1868:     return SystemZII::Branch(SystemZII::BranchC, SystemZ::CCMASK_ICMP,
1869:                              MI.getOperand(2).getImm(), &MI.getOperand(3));
1870: 
1871:   case SystemZ::CLIJ:
1872:   case SystemZ::CLRJ:
1873:     return SystemZII::Branch(SystemZII::BranchCL, SystemZ::CCMASK_ICMP,
1874:                              MI.getOperand(2).getImm(), &MI.getOperand(3));
1875: 
1876:   case SystemZ::CGIJ:
1877:   case SystemZ::CGRJ:
1878:     return SystemZII::Branch(SystemZII::BranchCG, SystemZ::CCMASK_ICMP,
1879:                              MI.getOperand(2).getImm(), &MI.getOperand(3));
1880: 
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getBranchInfo`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getBranchInfo` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 1881-1920 / 第 1881-1920 行
```cpp
1881:   case SystemZ::CLGIJ:
1882:   case SystemZ::CLGRJ:
1883:     return SystemZII::Branch(SystemZII::BranchCLG, SystemZ::CCMASK_ICMP,
1884:                              MI.getOperand(2).getImm(), &MI.getOperand(3));
1885: 
1886:   case SystemZ::INLINEASM_BR:
1887:     // Don't try to analyze asm goto, so pass nullptr as branch target argument.
1888:     return SystemZII::Branch(SystemZII::AsmGoto, 0, 0, nullptr);
1889: 
1890:   default:
1891:     llvm_unreachable("Unrecognized branch opcode");
1892:   }
1893: }
1894: 
1895: void SystemZInstrInfo::getLoadStoreOpcodes(const TargetRegisterClass *RC,
1896:                                            unsigned &LoadOpcode,
1897:                                            unsigned &StoreOpcode) const {
1898:   if (RC == &SystemZ::GR32BitRegClass || RC == &SystemZ::ADDR32BitRegClass) {
1899:     LoadOpcode = SystemZ::L;
1900:     StoreOpcode = SystemZ::ST;
1901:   } else if (RC == &SystemZ::GRH32BitRegClass) {
1902:     LoadOpcode = SystemZ::LFH;
1903:     StoreOpcode = SystemZ::STFH;
1904:   } else if (RC == &SystemZ::GRX32BitRegClass) {
1905:     LoadOpcode = SystemZ::LMux;
1906:     StoreOpcode = SystemZ::STMux;
1907:   } else if (RC == &SystemZ::GR64BitRegClass ||
1908:              RC == &SystemZ::ADDR64BitRegClass) {
1909:     LoadOpcode = SystemZ::LG;
1910:     StoreOpcode = SystemZ::STG;
1911:   } else if (RC == &SystemZ::GR128BitRegClass ||
1912:              RC == &SystemZ::ADDR128BitRegClass) {
1913:     LoadOpcode = SystemZ::L128;
1914:     StoreOpcode = SystemZ::ST128;
1915:   } else if (RC == &SystemZ::FP16BitRegClass && !STI.hasVector()) {
1916:     LoadOpcode = SystemZ::LE16;
1917:     StoreOpcode = SystemZ::STE16;
1918:   } else if (RC == &SystemZ::FP32BitRegClass) {
1919:     LoadOpcode = SystemZ::LE;
1920:     StoreOpcode = SystemZ::STE;
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getLoadStoreOpcodes`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getLoadStoreOpcodes` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1921-1960 / 第 1921-1960 行
```cpp
1921:   } else if (RC == &SystemZ::FP64BitRegClass) {
1922:     LoadOpcode = SystemZ::LD;
1923:     StoreOpcode = SystemZ::STD;
1924:   } else if (RC == &SystemZ::FP128BitRegClass) {
1925:     LoadOpcode = SystemZ::LX;
1926:     StoreOpcode = SystemZ::STX;
1927:   } else if (RC == &SystemZ::FP16BitRegClass ||
1928:              RC == &SystemZ::VR16BitRegClass) {
1929:     LoadOpcode = SystemZ::VL16;
1930:     StoreOpcode = SystemZ::VST16;
1931:   } else if (RC == &SystemZ::VR32BitRegClass) {
1932:     LoadOpcode = SystemZ::VL32;
1933:     StoreOpcode = SystemZ::VST32;
1934:   } else if (RC == &SystemZ::VR64BitRegClass) {
1935:     LoadOpcode = SystemZ::VL64;
1936:     StoreOpcode = SystemZ::VST64;
1937:   } else if (RC == &SystemZ::VF128BitRegClass ||
1938:              RC == &SystemZ::VR128BitRegClass) {
1939:     LoadOpcode = SystemZ::VL;
1940:     StoreOpcode = SystemZ::VST;
1941:   } else
1942:     llvm_unreachable("Unsupported regclass to load or store");
1943: }
1944: 
1945: unsigned SystemZInstrInfo::getOpcodeForOffset(unsigned Opcode,
1946:                                               int64_t Offset,
1947:                                               const MachineInstr *MI) const {
1948:   const MCInstrDesc &MCID = get(Opcode);
1949:   int64_t Offset2 = (MCID.TSFlags & SystemZII::Is128Bit ? Offset + 8 : Offset);
1950:   if (isUInt<12>(Offset) && isUInt<12>(Offset2)) {
1951:     // Get the instruction to use for unsigned 12-bit displacements.
1952:     int Disp12Opcode = SystemZ::getDisp12Opcode(Opcode);
1953:     if (Disp12Opcode >= 0)
1954:       return Disp12Opcode;
1955: 
1956:     // All address-related instructions can use unsigned 12-bit
1957:     // displacements.
1958:     return Opcode;
1959:   }
1960:   if (isInt<20>(Offset) && isInt<20>(Offset2)) {
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getOpcodeForOffset`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getOpcodeForOffset` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1961-2000 / 第 1961-2000 行
```cpp
1961:     // Get the instruction to use for signed 20-bit displacements.
1962:     int Disp20Opcode = SystemZ::getDisp20Opcode(Opcode);
1963:     if (Disp20Opcode >= 0)
1964:       return Disp20Opcode;
1965: 
1966:     // Check whether Opcode allows signed 20-bit displacements.
1967:     if (MCID.TSFlags & SystemZII::Has20BitOffset)
1968:       return Opcode;
1969: 
1970:     // If a VR32/VR64 reg ended up in an FP register, use the FP opcode.
1971:     if (MI && MI->getOperand(0).isReg()) {
1972:       Register Reg = MI->getOperand(0).getReg();
1973:       if (Reg.isPhysical() && SystemZMC::getFirstReg(Reg) < 16) {
1974:         switch (Opcode) {
1975:         case SystemZ::VL32:
1976:           return SystemZ::LEY;
1977:         case SystemZ::VST32:
1978:           return SystemZ::STEY;
1979:         case SystemZ::VL64:
1980:           return SystemZ::LDY;
1981:         case SystemZ::VST64:
1982:           return SystemZ::STDY;
1983:         default: break;
1984:         }
1985:       }
1986:     }
1987:   }
1988:   return 0;
1989: }
1990: 
1991: bool SystemZInstrInfo::hasDisplacementPairInsn(unsigned Opcode) const {
1992:   const MCInstrDesc &MCID = get(Opcode);
1993:   if (MCID.TSFlags & SystemZII::Has20BitOffset)
1994:     return SystemZ::getDisp12Opcode(Opcode) >= 0;
1995:   return SystemZ::getDisp20Opcode(Opcode) >= 0;
1996: }
1997: 
1998: unsigned SystemZInstrInfo::getLoadAndTest(unsigned Opcode) const {
1999:   switch (Opcode) {
2000:   case SystemZ::L:      return SystemZ::LT;
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::hasDisplacementPairInsn`, `SystemZInstrInfo::getLoadAndTest`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::hasDisplacementPairInsn`, `SystemZInstrInfo::getLoadAndTest` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2001-2040 / 第 2001-2040 行
```cpp
2001:   case SystemZ::LY:     return SystemZ::LT;
2002:   case SystemZ::LG:     return SystemZ::LTG;
2003:   case SystemZ::LGF:    return SystemZ::LTGF;
2004:   case SystemZ::LR:     return SystemZ::LTR;
2005:   case SystemZ::LGFR:   return SystemZ::LTGFR;
2006:   case SystemZ::LGR:    return SystemZ::LTGR;
2007:   case SystemZ::LCDFR:  return SystemZ::LCDBR;
2008:   case SystemZ::LPDFR:  return SystemZ::LPDBR;
2009:   case SystemZ::LNDFR:  return SystemZ::LNDBR;
2010:   case SystemZ::LCDFR_32:  return SystemZ::LCEBR;
2011:   case SystemZ::LPDFR_32:  return SystemZ::LPEBR;
2012:   case SystemZ::LNDFR_32:  return SystemZ::LNEBR;
2013:   // On zEC12 we prefer to use RISBGN.  But if there is a chance to
2014:   // actually use the condition code, we may turn it back into RISGB.
2015:   // Note that RISBG is not really a "load-and-test" instruction,
2016:   // but sets the same condition code values, so is OK to use here.
2017:   case SystemZ::RISBGN: return SystemZ::RISBG;
2018:   default:              return 0;
2019:   }
2020: }
2021: 
2022: bool SystemZInstrInfo::isRxSBGMask(uint64_t Mask, unsigned BitSize,
2023:                                    unsigned &Start, unsigned &End) const {
2024:   // Reject trivial all-zero masks.
2025:   Mask &= allOnes(BitSize);
2026:   if (Mask == 0)
2027:     return false;
2028: 
2029:   // Handle the 1+0+ or 0+1+0* cases.  Start then specifies the index of
2030:   // the msb and End specifies the index of the lsb.
2031:   unsigned LSB, Length;
2032:   if (isShiftedMask_64(Mask, LSB, Length)) {
2033:     Start = 63 - (LSB + Length - 1);
2034:     End = 63 - LSB;
2035:     return true;
2036:   }
2037: 
2038:   // Handle the wrap-around 1+0+1+ cases.  Start then specifies the msb
2039:   // of the low 1s and End specifies the lsb of the high 1s.
2040:   if (isShiftedMask_64(Mask ^ allOnes(BitSize), LSB, Length)) {
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::isRxSBGMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::isRxSBGMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2041-2080 / 第 2041-2080 行
```cpp
2041:     assert(LSB > 0 && "Bottom bit must be set");
2042:     assert(LSB + Length < BitSize && "Top bit must be set");
2043:     Start = 63 - (LSB - 1);
2044:     End = 63 - (LSB + Length);
2045:     return true;
2046:   }
2047: 
2048:   return false;
2049: }
2050: 
2051: unsigned SystemZInstrInfo::getFusedCompare(unsigned Opcode,
2052:                                            SystemZII::FusedCompareType Type,
2053:                                            const MachineInstr *MI) const {
2054:   switch (Opcode) {
2055:   case SystemZ::CHI:
2056:   case SystemZ::CGHI:
2057:     if (!(MI && isInt<8>(MI->getOperand(1).getImm())))
2058:       return 0;
2059:     break;
2060:   case SystemZ::CLFI:
2061:   case SystemZ::CLGFI:
2062:     if (!(MI && isUInt<8>(MI->getOperand(1).getImm())))
2063:       return 0;
2064:     break;
2065:   case SystemZ::CL:
2066:   case SystemZ::CLG:
2067:     if (!STI.hasMiscellaneousExtensions())
2068:       return 0;
2069:     if (!(MI && MI->getOperand(3).getReg() == 0))
2070:       return 0;
2071:     break;
2072:   }
2073:   switch (Type) {
2074:   case SystemZII::CompareAndBranch:
2075:     switch (Opcode) {
2076:     case SystemZ::CR:
2077:       return SystemZ::CRJ;
2078:     case SystemZ::CGR:
2079:       return SystemZ::CGRJ;
2080:     case SystemZ::CHI:
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getFusedCompare`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getFusedCompare` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2081-2120 / 第 2081-2120 行
```cpp
2081:       return SystemZ::CIJ;
2082:     case SystemZ::CGHI:
2083:       return SystemZ::CGIJ;
2084:     case SystemZ::CLR:
2085:       return SystemZ::CLRJ;
2086:     case SystemZ::CLGR:
2087:       return SystemZ::CLGRJ;
2088:     case SystemZ::CLFI:
2089:       return SystemZ::CLIJ;
2090:     case SystemZ::CLGFI:
2091:       return SystemZ::CLGIJ;
2092:     default:
2093:       return 0;
2094:     }
2095:   case SystemZII::CompareAndReturn:
2096:     switch (Opcode) {
2097:     case SystemZ::CR:
2098:       return SystemZ::CRBReturn;
2099:     case SystemZ::CGR:
2100:       return SystemZ::CGRBReturn;
2101:     case SystemZ::CHI:
2102:       return SystemZ::CIBReturn;
2103:     case SystemZ::CGHI:
2104:       return SystemZ::CGIBReturn;
2105:     case SystemZ::CLR:
2106:       return SystemZ::CLRBReturn;
2107:     case SystemZ::CLGR:
2108:       return SystemZ::CLGRBReturn;
2109:     case SystemZ::CLFI:
2110:       return SystemZ::CLIBReturn;
2111:     case SystemZ::CLGFI:
2112:       return SystemZ::CLGIBReturn;
2113:     default:
2114:       return 0;
2115:     }
2116:   case SystemZII::CompareAndSibcall:
2117:     switch (Opcode) {
2118:     case SystemZ::CR:
2119:       return SystemZ::CRBCall;
2120:     case SystemZ::CGR:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 2121-2160 / 第 2121-2160 行
```cpp
2121:       return SystemZ::CGRBCall;
2122:     case SystemZ::CHI:
2123:       return SystemZ::CIBCall;
2124:     case SystemZ::CGHI:
2125:       return SystemZ::CGIBCall;
2126:     case SystemZ::CLR:
2127:       return SystemZ::CLRBCall;
2128:     case SystemZ::CLGR:
2129:       return SystemZ::CLGRBCall;
2130:     case SystemZ::CLFI:
2131:       return SystemZ::CLIBCall;
2132:     case SystemZ::CLGFI:
2133:       return SystemZ::CLGIBCall;
2134:     default:
2135:       return 0;
2136:     }
2137:   case SystemZII::CompareAndTrap:
2138:     switch (Opcode) {
2139:     case SystemZ::CR:
2140:       return SystemZ::CRT;
2141:     case SystemZ::CGR:
2142:       return SystemZ::CGRT;
2143:     case SystemZ::CHI:
2144:       return SystemZ::CIT;
2145:     case SystemZ::CGHI:
2146:       return SystemZ::CGIT;
2147:     case SystemZ::CLR:
2148:       return SystemZ::CLRT;
2149:     case SystemZ::CLGR:
2150:       return SystemZ::CLGRT;
2151:     case SystemZ::CLFI:
2152:       return SystemZ::CLFIT;
2153:     case SystemZ::CLGFI:
2154:       return SystemZ::CLGIT;
2155:     case SystemZ::CL:
2156:       return SystemZ::CLT;
2157:     case SystemZ::CLG:
2158:       return SystemZ::CLGT;
2159:     default:
2160:       return 0;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 2161-2200 / 第 2161-2200 行
```cpp
2161:     }
2162:   }
2163:   return 0;
2164: }
2165: 
2166: bool SystemZInstrInfo::isLoadAndTestAsCmp(const MachineInstr &MI) const {
2167:   // If we during isel used a load-and-test as a compare with 0, the
2168:   // def operand is dead.
2169:   return (MI.getOpcode() == SystemZ::LTEBR ||
2170:           MI.getOpcode() == SystemZ::LTDBR ||
2171:           MI.getOpcode() == SystemZ::LTXBR) &&
2172:          MI.getOperand(0).isDead();
2173: }
2174: 
2175: bool SystemZInstrInfo::isCompareZero(const MachineInstr &Compare) const {
2176:   if (isLoadAndTestAsCmp(Compare))
2177:     return true;
2178:   return Compare.isCompare() && Compare.getNumExplicitOperands() == 2 &&
2179:          Compare.getOperand(1).isImm() && Compare.getOperand(1).getImm() == 0;
2180: }
2181: 
2182: Register
2183: SystemZInstrInfo::getCompareSourceReg(const MachineInstr &Compare) const {
2184:   assert(isCompareZero(Compare) && "Expected a compare with 0.");
2185:   return Compare.getOperand(isLoadAndTestAsCmp(Compare) ? 1 : 0).getReg();
2186: }
2187: 
2188: bool SystemZInstrInfo::
2189: prepareCompareSwapOperands(MachineBasicBlock::iterator const MBBI) const {
2190:   assert(MBBI->isCompare() && MBBI->getOperand(0).isReg() &&
2191:          MBBI->getOperand(1).isReg() && !MBBI->mayLoad() &&
2192:          "Not a compare reg/reg.");
2193: 
2194:   MachineBasicBlock *MBB = MBBI->getParent();
2195:   bool CCLive = true;
2196:   SmallVector<MachineInstr *, 4> CCUsers;
2197:   for (MachineInstr &MI : llvm::make_range(std::next(MBBI), MBB->end())) {
2198:     if (MI.readsRegister(SystemZ::CC, /*TRI=*/nullptr)) {
2199:       unsigned Flags = MI.getDesc().TSFlags;
2200:       if ((Flags & SystemZII::CCMaskFirst) || (Flags & SystemZII::CCMaskLast))
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::isLoadAndTestAsCmp`, `SystemZInstrInfo::isCompareZero`, `SystemZInstrInfo::getCompareSourceReg`, `prepareCompareSwapOperands`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::isLoadAndTestAsCmp`, `SystemZInstrInfo::isCompareZero`, `SystemZInstrInfo::getCompareSourceReg`, `prepareCompareSwapOperands` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2201-2240 / 第 2201-2240 行
```cpp
2201:         CCUsers.push_back(&MI);
2202:       else
2203:         return false;
2204:     }
2205:     if (MI.definesRegister(SystemZ::CC, /*TRI=*/nullptr)) {
2206:       CCLive = false;
2207:       break;
2208:     }
2209:   }
2210:   if (CCLive) {
2211:     LiveRegUnits LiveRegs(*MBB->getParent()->getSubtarget().getRegisterInfo());
2212:     LiveRegs.addLiveOuts(*MBB);
2213:     if (!LiveRegs.available(SystemZ::CC))
2214:       return false;
2215:   }
2216: 
2217:   // Update all CC users.
2218:   for (unsigned Idx = 0; Idx < CCUsers.size(); ++Idx) {
2219:     unsigned Flags = CCUsers[Idx]->getDesc().TSFlags;
2220:     unsigned FirstOpNum = ((Flags & SystemZII::CCMaskFirst) ?
2221:                            0 : CCUsers[Idx]->getNumExplicitOperands() - 2);
2222:     MachineOperand &CCMaskMO = CCUsers[Idx]->getOperand(FirstOpNum + 1);
2223:     unsigned NewCCMask = SystemZ::reverseCCMask(CCMaskMO.getImm());
2224:     CCMaskMO.setImm(NewCCMask);
2225:   }
2226: 
2227:   return true;
2228: }
2229: 
2230: unsigned SystemZ::reverseCCMask(unsigned CCMask) {
2231:   return ((CCMask & SystemZ::CCMASK_CMP_EQ) |
2232:           ((CCMask & SystemZ::CCMASK_CMP_GT) ? SystemZ::CCMASK_CMP_LT : 0) |
2233:           ((CCMask & SystemZ::CCMASK_CMP_LT) ? SystemZ::CCMASK_CMP_GT : 0) |
2234:           (CCMask & SystemZ::CCMASK_CMP_UO));
2235: }
2236: 
2237: MachineBasicBlock *SystemZ::emitBlockAfter(MachineBasicBlock *MBB) {
2238:   MachineFunction &MF = *MBB->getParent();
2239:   MachineBasicBlock *NewMBB = MF.CreateMachineBasicBlock(MBB->getBasicBlock());
2240:   MF.insert(std::next(MachineFunction::iterator(MBB)), NewMBB);
```
- **EN**: The range implements or declares functions including `SystemZ::reverseCCMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZ::reverseCCMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2241-2280 / 第 2241-2280 行
```cpp
2241:   return NewMBB;
2242: }
2243: 
2244: MachineBasicBlock *SystemZ::splitBlockAfter(MachineBasicBlock::iterator MI,
2245:                                             MachineBasicBlock *MBB) {
2246:   MachineBasicBlock *NewMBB = emitBlockAfter(MBB);
2247:   NewMBB->splice(NewMBB->begin(), MBB,
2248:                  std::next(MachineBasicBlock::iterator(MI)), MBB->end());
2249:   NewMBB->transferSuccessorsAndUpdatePHIs(MBB);
2250:   return NewMBB;
2251: }
2252: 
2253: MachineBasicBlock *SystemZ::splitBlockBefore(MachineBasicBlock::iterator MI,
2254:                                              MachineBasicBlock *MBB) {
2255:   MachineBasicBlock *NewMBB = emitBlockAfter(MBB);
2256:   NewMBB->splice(NewMBB->begin(), MBB, MI, MBB->end());
2257:   NewMBB->transferSuccessorsAndUpdatePHIs(MBB);
2258:   return NewMBB;
2259: }
2260: 
2261: unsigned SystemZInstrInfo::getLoadAndTrap(unsigned Opcode) const {
2262:   if (!STI.hasLoadAndTrap())
2263:     return 0;
2264:   switch (Opcode) {
2265:   case SystemZ::L:
2266:   case SystemZ::LY:
2267:     return SystemZ::LAT;
2268:   case SystemZ::LG:
2269:     return SystemZ::LGAT;
2270:   case SystemZ::LFH:
2271:     return SystemZ::LFHAT;
2272:   case SystemZ::LLGF:
2273:     return SystemZ::LLGFAT;
2274:   case SystemZ::LLGT:
2275:     return SystemZ::LLGTAT;
2276:   }
2277:   return 0;
2278: }
2279: 
2280: void SystemZInstrInfo::loadImmediate(MachineBasicBlock &MBB,
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getLoadAndTrap`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getLoadAndTrap` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2281-2320 / 第 2281-2320 行
```cpp
2281:                                      MachineBasicBlock::iterator MBBI,
2282:                                      unsigned Reg, uint64_t Value) const {
2283:   DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
2284:   unsigned Opcode = 0;
2285:   if (isInt<16>(Value))
2286:     Opcode = SystemZ::LGHI;
2287:   else if (SystemZ::isImmLL(Value))
2288:     Opcode = SystemZ::LLILL;
2289:   else if (SystemZ::isImmLH(Value)) {
2290:     Opcode = SystemZ::LLILH;
2291:     Value >>= 16;
2292:   }
2293:   else if (isInt<32>(Value))
2294:     Opcode = SystemZ::LGFI;
2295:   if (Opcode) {
2296:     BuildMI(MBB, MBBI, DL, get(Opcode), Reg).addImm(Value);
2297:     return;
2298:   }
2299: 
2300:   MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
2301:   assert (MRI.isSSA() &&  "Huge values only handled before reg-alloc .");
2302:   Register Reg0 = MRI.createVirtualRegister(&SystemZ::GR64BitRegClass);
2303:   Register Reg1 = MRI.createVirtualRegister(&SystemZ::GR64BitRegClass);
2304:   BuildMI(MBB, MBBI, DL, get(SystemZ::IMPLICIT_DEF), Reg0);
2305:   BuildMI(MBB, MBBI, DL, get(SystemZ::IIHF64), Reg1)
2306:     .addReg(Reg0).addImm(Value >> 32);
2307:   BuildMI(MBB, MBBI, DL, get(SystemZ::IILF64), Reg)
2308:     .addReg(Reg1).addImm(Value & ((uint64_t(1) << 32) - 1));
2309: }
2310: 
2311: bool SystemZInstrInfo::verifyInstruction(const MachineInstr &MI,
2312:                                          StringRef &ErrInfo) const {
2313:   const MCInstrDesc &MCID = MI.getDesc();
2314:   for (unsigned I = 0, E = MI.getNumOperands(); I != E; ++I) {
2315:     if (I >= MCID.getNumOperands())
2316:       break;
2317:     const MachineOperand &Op = MI.getOperand(I);
2318:     const MCOperandInfo &MCOI = MCID.operands()[I];
2319:     // Addressing modes have register and immediate operands. Op should be a
2320:     // register (or frame index) operand if MCOI.RegClass contains a valid
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `SystemZInstrInfo::verifyInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `SystemZInstrInfo::verifyInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2321-2360 / 第 2321-2360 行
```cpp
2321:     // register class, or an immediate otherwise.
2322:     if (MCOI.OperandType == MCOI::OPERAND_MEMORY &&
2323:         ((MCOI.RegClass != -1 && !Op.isReg() && !Op.isFI()) ||
2324:          (MCOI.RegClass == -1 && !Op.isImm()))) {
2325:       ErrInfo = "Addressing mode operands corrupt!";
2326:       return false;
2327:     }
2328:   }
2329: 
2330:   return true;
2331: }
2332: 
2333: bool SystemZInstrInfo::
2334: areMemAccessesTriviallyDisjoint(const MachineInstr &MIa,
2335:                                 const MachineInstr &MIb) const {
2336: 
2337:   if (!MIa.hasOneMemOperand() || !MIb.hasOneMemOperand())
2338:     return false;
2339: 
2340:   // If mem-operands show that the same address Value is used by both
2341:   // instructions, check for non-overlapping offsets and widths. Not
2342:   // sure if a register based analysis would be an improvement...
2343: 
2344:   MachineMemOperand *MMOa = *MIa.memoperands_begin();
2345:   MachineMemOperand *MMOb = *MIb.memoperands_begin();
2346:   const Value *VALa = MMOa->getValue();
2347:   const Value *VALb = MMOb->getValue();
2348:   bool SameVal = (VALa && VALb && (VALa == VALb));
2349:   if (!SameVal) {
2350:     const PseudoSourceValue *PSVa = MMOa->getPseudoValue();
2351:     const PseudoSourceValue *PSVb = MMOb->getPseudoValue();
2352:     if (PSVa && PSVb && (PSVa == PSVb))
2353:       SameVal = true;
2354:   }
2355:   if (SameVal) {
2356:     int OffsetA = MMOa->getOffset(), OffsetB = MMOb->getOffset();
2357:     LocationSize WidthA = MMOa->getSize(), WidthB = MMOb->getSize();
2358:     int LowOffset = OffsetA < OffsetB ? OffsetA : OffsetB;
2359:     int HighOffset = OffsetA < OffsetB ? OffsetB : OffsetA;
2360:     LocationSize LowWidth = (LowOffset == OffsetA) ? WidthA : WidthB;
```
- **EN**: The range implements or declares functions including `areMemAccessesTriviallyDisjoint`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `areMemAccessesTriviallyDisjoint` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2361-2400 / 第 2361-2400 行
```cpp
2361:     if (LowWidth.hasValue() &&
2362:         LowOffset + (int)LowWidth.getValue() <= HighOffset)
2363:       return true;
2364:   }
2365: 
2366:   return false;
2367: }
2368: 
2369: bool SystemZInstrInfo::getConstValDefinedInReg(const MachineInstr &MI,
2370:                                                const Register Reg,
2371:                                                int64_t &ImmVal) const {
2372: 
2373:   if (MI.getOpcode() == SystemZ::VGBM && Reg == MI.getOperand(0).getReg()) {
2374:     ImmVal = MI.getOperand(1).getImm();
2375:     // TODO: Handle non-0 values
2376:     return ImmVal == 0;
2377:   }
2378: 
2379:   return false;
2380: }
2381: 
2382: std::optional<DestSourcePair>
2383: SystemZInstrInfo::isCopyInstrImpl(const MachineInstr &MI) const {
2384:   // if MI is a simple single-register copy operation, return operand pair
2385:   if (MI.isMoveReg())
2386:     return DestSourcePair(MI.getOperand(0), MI.getOperand(1));
2387: 
2388:   return std::nullopt;
2389: }
2390: 
2391: std::pair<unsigned, unsigned>
2392: SystemZInstrInfo::decomposeMachineOperandsTargetFlags(unsigned TF) const {
2393:   return std::make_pair(TF, 0u);
2394: }
2395: 
2396: ArrayRef<std::pair<unsigned, const char *>>
2397: SystemZInstrInfo::getSerializableDirectMachineOperandTargetFlags() const {
2398:   using namespace SystemZII;
2399: 
2400:   static const std::pair<unsigned, const char *> TargetFlags[] = {
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getConstValDefinedInReg`, `SystemZInstrInfo::isCopyInstrImpl`, `SystemZInstrInfo::decomposeMachineOperandsTargetFlags`, `SystemZInstrInfo::getSerializableDirectMachineOperandTargetFlags`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getConstValDefinedInReg`, `SystemZInstrInfo::isCopyInstrImpl`, `SystemZInstrInfo::decomposeMachineOperandsTargetFlags`, `SystemZInstrInfo::getSerializableDirectMachineOperandTargetFlags` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2401-2409 / 第 2401-2409 行
```cpp
2401:       {MO_ADA_DATA_SYMBOL_ADDR, "systemz-ada-datasymboladdr"},
2402:       {MO_ADA_INDIRECT_FUNC_DESC, "systemz-ada-indirectfuncdesc"},
2403:       {MO_ADA_DIRECT_FUNC_DESC, "systemz-ada-directfuncdesc"}};
2404:   return ArrayRef(TargetFlags);
2405: }
2406: 
2407: MCInst SystemZInstrInfo::getNop() const {
2408:   return MCInstBuilder(SystemZ::NOPR).addReg(0);
2409: }
```
- **EN**: The range implements or declares functions including `SystemZInstrInfo::getNop`.
- **CN**: 这一段实现或声明了 `SystemZInstrInfo::getNop` 等函数。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。

## Dependencies / 依赖关系
- `SystemZInstrInfo.h`
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `SystemZ.h`
- `SystemZInstrBuilder.h`
- `SystemZSubtarget.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/LiveInterval.h`
- `llvm/CodeGen/LiveIntervals.h`
- `llvm/CodeGen/LiveRegUnits.h`
- `llvm/CodeGen/LiveVariables.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/CodeGen/MachineMemOperand.h`
- `llvm/CodeGen/MachineOperand.h`
- `...` (18 more include dependencies omitted for brevity / 其余 18 个 include 依赖已省略)
