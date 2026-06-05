# LoongArchInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchInstrInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target instruction information for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责定义或实现目标指令信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //=- LoongArchInstrInfo.cpp - LoongArch Instruction Information -*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the LoongArch implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchInstrInfo.h"
  14: #include "LoongArch.h"
  15: #include "LoongArchMachineFunctionInfo.h"
  16: #include "LoongArchRegisterInfo.h"
  17: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  18: #include "MCTargetDesc/LoongArchMatInt.h"
  19: #include "llvm/CodeGen/RegisterScavenging.h"
  20: #include "llvm/CodeGen/StackMaps.h"
  21: #include "llvm/MC/MCContext.h"
  22: #include "llvm/MC/MCInstBuilder.h"
  23: #include "llvm/Support/CommandLine.h"
  24: 
  25: using namespace llvm;
  26: 
  27: static cl::opt<bool> DisableRelocSched(
  28:     "loongarch-disable-reloc-sched",
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchInstrInfo.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchRegisterInfo.h`, `LoongArchMCTargetDesc.h`, `LoongArchMatInt.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchInstrInfo.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchRegisterInfo.h`, `LoongArchMCTargetDesc.h`, `LoongArchMatInt.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 29-56 / 第 29-56 行
```cpp
  29:     cl::desc("Disable scheduling of instructions with target flags"),
  30:     cl::init(false), cl::Hidden);
  31: 
  32: #define GET_INSTRINFO_CTOR_DTOR
  33: #include "LoongArchGenInstrInfo.inc"
  34: 
  35: LoongArchInstrInfo::LoongArchInstrInfo(const LoongArchSubtarget &STI)
  36:     : LoongArchGenInstrInfo(STI, RegInfo, LoongArch::ADJCALLSTACKDOWN,
  37:                             LoongArch::ADJCALLSTACKUP),
  38:       RegInfo(STI.getHwMode()), STI(STI) {}
  39: 
  40: MCInst LoongArchInstrInfo::getNop() const {
  41:   return MCInstBuilder(LoongArch::ANDI)
  42:       .addReg(LoongArch::R0)
  43:       .addReg(LoongArch::R0)
  44:       .addImm(0);
  45: }
  46: 
  47: void LoongArchInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
  48:                                      MachineBasicBlock::iterator MBBI,
  49:                                      const DebugLoc &DL, Register DstReg,
  50:                                      Register SrcReg, bool KillSrc,
  51:                                      bool RenamableDest,
  52:                                      bool RenamableSrc) const {
  53:   if (LoongArch::GPRRegClass.contains(DstReg, SrcReg)) {
  54:     BuildMI(MBB, MBBI, DL, get(LoongArch::OR), DstReg)
  55:         .addReg(SrcReg, getKillRegState(KillSrc))
  56:         .addReg(LoongArch::R0);
```
- **EN**: It imports dependencies such as `LoongArchGenInstrInfo.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchGenInstrInfo`, `LoongArchInstrInfo::copyPhysReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `LoongArchGenInstrInfo.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchGenInstrInfo`, `LoongArchInstrInfo::copyPhysReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57:     return;
  58:   }
  59: 
  60:   // VR->VR copies.
  61:   if (LoongArch::LSX128RegClass.contains(DstReg, SrcReg)) {
  62:     BuildMI(MBB, MBBI, DL, get(LoongArch::VORI_B), DstReg)
  63:         .addReg(SrcReg, getKillRegState(KillSrc))
  64:         .addImm(0);
  65:     return;
  66:   }
  67: 
  68:   // XR->XR copies.
  69:   if (LoongArch::LASX256RegClass.contains(DstReg, SrcReg)) {
  70:     BuildMI(MBB, MBBI, DL, get(LoongArch::XVORI_B), DstReg)
  71:         .addReg(SrcReg, getKillRegState(KillSrc))
  72:         .addImm(0);
  73:     return;
  74:   }
  75: 
  76:   // GPR->CFR copy.
  77:   if (LoongArch::CFRRegClass.contains(DstReg) &&
  78:       LoongArch::GPRRegClass.contains(SrcReg)) {
  79:     BuildMI(MBB, MBBI, DL, get(LoongArch::MOVGR2CF), DstReg)
  80:         .addReg(SrcReg, getKillRegState(KillSrc));
  81:     return;
  82:   }
  83:   // CFR->GPR copy.
  84:   if (LoongArch::GPRRegClass.contains(DstReg) &&
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```cpp
  85:       LoongArch::CFRRegClass.contains(SrcReg)) {
  86:     BuildMI(MBB, MBBI, DL, get(LoongArch::MOVCF2GR), DstReg)
  87:         .addReg(SrcReg, getKillRegState(KillSrc));
  88:     return;
  89:   }
  90:   // CFR->CFR copy.
  91:   if (LoongArch::CFRRegClass.contains(DstReg, SrcReg)) {
  92:     BuildMI(MBB, MBBI, DL, get(LoongArch::PseudoCopyCFR), DstReg)
  93:         .addReg(SrcReg, getKillRegState(KillSrc));
  94:     return;
  95:   }
  96: 
  97:   // FPR->FPR copies.
  98:   unsigned Opc;
  99:   if (LoongArch::FPR32RegClass.contains(DstReg, SrcReg)) {
 100:     Opc = LoongArch::FMOV_S;
 101:   } else if (LoongArch::FPR64RegClass.contains(DstReg, SrcReg)) {
 102:     Opc = LoongArch::FMOV_D;
 103:   } else if (LoongArch::GPRRegClass.contains(DstReg) &&
 104:              LoongArch::FPR32RegClass.contains(SrcReg)) {
 105:     // FPR32 -> GPR copies
 106:     Opc = LoongArch::MOVFR2GR_S;
 107:   } else if (LoongArch::GPRRegClass.contains(DstReg) &&
 108:              LoongArch::FPR64RegClass.contains(SrcReg)) {
 109:     // FPR64 -> GPR copies
 110:     Opc = LoongArch::MOVFR2GR_D;
 111:   } else {
 112:     // TODO: support other copies.
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:     llvm_unreachable("Impossible reg-to-reg copy");
 114:   }
 115: 
 116:   BuildMI(MBB, MBBI, DL, get(Opc), DstReg)
 117:       .addReg(SrcReg, getKillRegState(KillSrc));
 118: }
 119: 
 120: void LoongArchInstrInfo::storeRegToStackSlot(
 121:     MachineBasicBlock &MBB, MachineBasicBlock::iterator I, Register SrcReg,
 122:     bool IsKill, int FI, const TargetRegisterClass *RC,
 123: 
 124:     Register VReg, MachineInstr::MIFlag Flags) const {
 125:   MachineFunction *MF = MBB.getParent();
 126:   MachineFrameInfo &MFI = MF->getFrameInfo();
 127: 
 128:   unsigned Opcode;
 129:   if (LoongArch::GPRRegClass.hasSubClassEq(RC))
 130:     Opcode = TRI.getRegSizeInBits(LoongArch::GPRRegClass) == 32
 131:                  ? LoongArch::ST_W
 132:                  : LoongArch::ST_D;
 133:   else if (LoongArch::FPR32RegClass.hasSubClassEq(RC))
 134:     Opcode = LoongArch::FST_S;
 135:   else if (LoongArch::FPR64RegClass.hasSubClassEq(RC))
 136:     Opcode = LoongArch::FST_D;
 137:   else if (LoongArch::LSX128RegClass.hasSubClassEq(RC))
 138:     Opcode = LoongArch::VST;
 139:   else if (LoongArch::LASX256RegClass.hasSubClassEq(RC))
 140:     Opcode = LoongArch::XVST;
```
- **EN**: The range implements or declares functions including `BuildMI`, `LoongArchInstrInfo::storeRegToStackSlot`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `LoongArchInstrInfo::storeRegToStackSlot` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 141-168 / 第 141-168 行
```cpp
 141:   else if (LoongArch::CFRRegClass.hasSubClassEq(RC))
 142:     Opcode = LoongArch::PseudoST_CFR;
 143:   else
 144:     llvm_unreachable("Can't store this register to stack slot");
 145: 
 146:   MachineMemOperand *MMO = MF->getMachineMemOperand(
 147:       MachinePointerInfo::getFixedStack(*MF, FI), MachineMemOperand::MOStore,
 148:       MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
 149: 
 150:   BuildMI(MBB, I, DebugLoc(), get(Opcode))
 151:       .addReg(SrcReg, getKillRegState(IsKill))
 152:       .addFrameIndex(FI)
 153:       .addImm(0)
 154:       .addMemOperand(MMO);
 155: }
 156: 
 157: void LoongArchInstrInfo::loadRegFromStackSlot(
 158:     MachineBasicBlock &MBB, MachineBasicBlock::iterator I, Register DstReg,
 159:     int FI, const TargetRegisterClass *RC, Register VReg, unsigned SubReg,
 160:     MachineInstr::MIFlag Flags) const {
 161:   MachineFunction *MF = MBB.getParent();
 162:   MachineFrameInfo &MFI = MF->getFrameInfo();
 163:   DebugLoc DL;
 164:   if (I != MBB.end())
 165:     DL = I->getDebugLoc();
 166: 
 167:   unsigned Opcode;
 168:   if (LoongArch::GPRRegClass.hasSubClassEq(RC))
```
- **EN**: The range implements or declares functions including `BuildMI`, `LoongArchInstrInfo::loadRegFromStackSlot`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `LoongArchInstrInfo::loadRegFromStackSlot` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 169-196 / 第 169-196 行
```cpp
 169:     Opcode = RegInfo.getRegSizeInBits(LoongArch::GPRRegClass) == 32
 170:                  ? LoongArch::LD_W
 171:                  : LoongArch::LD_D;
 172:   else if (LoongArch::FPR32RegClass.hasSubClassEq(RC))
 173:     Opcode = LoongArch::FLD_S;
 174:   else if (LoongArch::FPR64RegClass.hasSubClassEq(RC))
 175:     Opcode = LoongArch::FLD_D;
 176:   else if (LoongArch::LSX128RegClass.hasSubClassEq(RC))
 177:     Opcode = LoongArch::VLD;
 178:   else if (LoongArch::LASX256RegClass.hasSubClassEq(RC))
 179:     Opcode = LoongArch::XVLD;
 180:   else if (LoongArch::CFRRegClass.hasSubClassEq(RC))
 181:     Opcode = LoongArch::PseudoLD_CFR;
 182:   else
 183:     llvm_unreachable("Can't load this register from stack slot");
 184: 
 185:   MachineMemOperand *MMO = MF->getMachineMemOperand(
 186:       MachinePointerInfo::getFixedStack(*MF, FI), MachineMemOperand::MOLoad,
 187:       MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
 188: 
 189:   BuildMI(MBB, I, DL, get(Opcode), DstReg)
 190:       .addFrameIndex(FI)
 191:       .addImm(0)
 192:       .addMemOperand(MMO);
 193: }
 194: 
 195: Register LoongArchInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
 196:                                                  int &FrameIndex) const {
```
- **EN**: The range implements or declares functions including `BuildMI`, `LoongArchInstrInfo::isLoadFromStackSlot`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `LoongArchInstrInfo::isLoadFromStackSlot` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 197-224 / 第 197-224 行
```cpp
 197:   TypeSize Dummy = TypeSize::getZero();
 198:   return isLoadFromStackSlot(MI, FrameIndex, Dummy);
 199: }
 200: 
 201: Register LoongArchInstrInfo::isLoadFromStackSlot(const MachineInstr &MI,
 202:                                                  int &FrameIndex,
 203:                                                  TypeSize &MemBytes) const {
 204:   switch (MI.getOpcode()) {
 205:   default:
 206:     return Register();
 207:   case LoongArch::LD_W:
 208:   case LoongArch::FLD_S:
 209:     MemBytes = TypeSize::getFixed(4);
 210:     break;
 211:   case LoongArch::LD_D:
 212:   case LoongArch::FLD_D:
 213:     MemBytes = TypeSize::getFixed(8);
 214:     break;
 215:   case LoongArch::VLD:
 216:     MemBytes = TypeSize::getFixed(16);
 217:     break;
 218:   case LoongArch::XVLD:
 219:     MemBytes = TypeSize::getFixed(32);
 220:     break;
 221:   }
 222: 
 223:   if ((MI.getOperand(1).isFI()) &&  // is a stack slot
 224:       (MI.getOperand(2).isImm()) && // the imm is zero
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::isLoadFromStackSlot`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::isLoadFromStackSlot` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:       (MI.getOperand(2).getImm() == 0)) {
 226:     FrameIndex = MI.getOperand(1).getIndex();
 227:     return MI.getOperand(0).getReg();
 228:   }
 229: 
 230:   return Register();
 231: }
 232: 
 233: Register LoongArchInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
 234:                                                 int &FrameIndex) const {
 235:   TypeSize Dummy = TypeSize::getZero();
 236:   return isStoreToStackSlot(MI, FrameIndex, Dummy);
 237: }
 238: 
 239: Register LoongArchInstrInfo::isStoreToStackSlot(const MachineInstr &MI,
 240:                                                 int &FrameIndex,
 241:                                                 TypeSize &MemBytes) const {
 242:   switch (MI.getOpcode()) {
 243:   default:
 244:     return Register();
 245:   case LoongArch::ST_W:
 246:   case LoongArch::FST_S:
 247:     MemBytes = TypeSize::getFixed(4);
 248:     break;
 249:   case LoongArch::ST_D:
 250:   case LoongArch::FST_D:
 251:     MemBytes = TypeSize::getFixed(8);
 252:     break;
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::isStoreToStackSlot`, `LoongArchInstrInfo::isStoreToStackSlot`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::isStoreToStackSlot`, `LoongArchInstrInfo::isStoreToStackSlot` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   case LoongArch::VST:
 254:     MemBytes = TypeSize::getFixed(16);
 255:     break;
 256:   case LoongArch::XVST:
 257:     MemBytes = TypeSize::getFixed(32);
 258:     break;
 259:   }
 260: 
 261:   if ((MI.getOperand(1).isFI()) &&  // is a stack slot
 262:       (MI.getOperand(2).isImm()) && // the imm is zero
 263:       (MI.getOperand(2).getImm() == 0)) {
 264:     FrameIndex = MI.getOperand(1).getIndex();
 265:     return MI.getOperand(0).getReg();
 266:   }
 267: 
 268:   return Register();
 269: }
 270: 
 271: void LoongArchInstrInfo::movImm(MachineBasicBlock &MBB,
 272:                                 MachineBasicBlock::iterator MBBI,
 273:                                 const DebugLoc &DL, Register DstReg,
 274:                                 uint64_t Val, MachineInstr::MIFlag Flag) const {
 275:   Register SrcReg = LoongArch::R0;
 276: 
 277:   if (!STI.is64Bit() && !isInt<32>(Val))
 278:     report_fatal_error("Should only materialize 32-bit constants for LA32");
 279: 
 280:   auto Seq = LoongArchMatInt::generateInstSeq(Val);
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::movImm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::movImm` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```cpp
 281:   assert(!Seq.empty());
 282: 
 283:   for (auto &Inst : Seq) {
 284:     switch (Inst.Opc) {
 285:     case LoongArch::LU12I_W:
 286:       BuildMI(MBB, MBBI, DL, get(Inst.Opc), DstReg)
 287:           .addImm(Inst.Imm)
 288:           .setMIFlag(Flag);
 289:       break;
 290:     case LoongArch::ADDI_W:
 291:     case LoongArch::ORI:
 292:     case LoongArch::LU32I_D: // "rj" is needed due to InstrInfo pattern
 293:     case LoongArch::LU52I_D:
 294:       BuildMI(MBB, MBBI, DL, get(Inst.Opc), DstReg)
 295:           .addReg(SrcReg, RegState::Kill)
 296:           .addImm(Inst.Imm)
 297:           .setMIFlag(Flag);
 298:       break;
 299:     case LoongArch::BSTRINS_D:
 300:       BuildMI(MBB, MBBI, DL, get(Inst.Opc), DstReg)
 301:           .addReg(SrcReg, RegState::Kill)
 302:           .addReg(SrcReg, RegState::Kill)
 303:           .addImm(Inst.Imm >> 32)
 304:           .addImm(Inst.Imm & 0xFF)
 305:           .setMIFlag(Flag);
 306:       break;
 307:     default:
 308:       assert(false && "Unknown insn emitted by LoongArchMatInt");
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 309-336 / 第 309-336 行
```cpp
 309:     }
 310: 
 311:     // Only the first instruction has $zero as its source.
 312:     SrcReg = DstReg;
 313:   }
 314: }
 315: 
 316: unsigned LoongArchInstrInfo::getInstSizeInBytes(const MachineInstr &MI) const {
 317:   unsigned Opcode = MI.getOpcode();
 318: 
 319:   if (Opcode == TargetOpcode::INLINEASM ||
 320:       Opcode == TargetOpcode::INLINEASM_BR) {
 321:     const MachineFunction *MF = MI.getParent()->getParent();
 322:     const MCAsmInfo &MAI = MF->getTarget().getMCAsmInfo();
 323:     return getInlineAsmLength(MI.getOperand(0).getSymbolName(), MAI);
 324:   }
 325: 
 326:   unsigned NumBytes = 0;
 327:   const MCInstrDesc &Desc = MI.getDesc();
 328: 
 329:   // Size should be preferably set in
 330:   // llvm/lib/Target/LoongArch/LoongArch*InstrInfo.td (default case).
 331:   // Specific cases handle instructions of variable sizes.
 332:   switch (Desc.getOpcode()) {
 333:   default:
 334:     return Desc.getSize();
 335:   case TargetOpcode::STATEPOINT:
 336:     NumBytes = StatepointOpers(&MI).getNumPatchBytes();
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::getInstSizeInBytes`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::getInstSizeInBytes` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:     assert(NumBytes % 4 == 0 && "Invalid number of NOP bytes requested!");
 338:     // No patch bytes means a normal call inst (i.e. `bl`) is emitted.
 339:     if (NumBytes == 0)
 340:       NumBytes = 4;
 341:     break;
 342:   case TargetOpcode::PATCHABLE_FUNCTION_ENTER: {
 343:     const MachineFunction *MF = MI.getParent()->getParent();
 344:     const Function &F = MF->getFunction();
 345:     if (F.hasFnAttribute("patchable-function-entry")) {
 346:       unsigned Num =
 347:           F.getFnAttributeAsParsedInteger("patchable-function-entry");
 348:       return Num * 4;
 349:     }
 350:     [[fallthrough]];
 351:   }
 352:   case TargetOpcode::PATCHABLE_FUNCTION_EXIT:
 353:   case TargetOpcode::PATCHABLE_TAIL_CALL:
 354:     // Size of xray sled (branch + 11 nops).
 355:     return 12 * 4;
 356:   case TargetOpcode::BUNDLE:
 357:     return getInstBundleSize(MI);
 358:   }
 359:   return NumBytes;
 360: }
 361: 
 362: bool LoongArchInstrInfo::isAsCheapAsAMove(const MachineInstr &MI) const {
 363:   const unsigned Opcode = MI.getOpcode();
 364:   switch (Opcode) {
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::isAsCheapAsAMove`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::isAsCheapAsAMove` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   default:
 366:     break;
 367:   case LoongArch::ADDI_D:
 368:   case LoongArch::ORI:
 369:   case LoongArch::XORI:
 370:     return (MI.getOperand(1).isReg() &&
 371:             MI.getOperand(1).getReg() == LoongArch::R0) ||
 372:            (MI.getOperand(2).isImm() && MI.getOperand(2).getImm() == 0);
 373:   }
 374:   return MI.isAsCheapAsAMove();
 375: }
 376: 
 377: MachineBasicBlock *
 378: LoongArchInstrInfo::getBranchDestBlock(const MachineInstr &MI) const {
 379:   assert(MI.getDesc().isBranch() && "Unexpected opcode!");
 380:   // The branch target is always the last operand.
 381:   return MI.getOperand(MI.getNumExplicitOperands() - 1).getMBB();
 382: }
 383: 
 384: static void parseCondBranch(MachineInstr &LastInst, MachineBasicBlock *&Target,
 385:                             SmallVectorImpl<MachineOperand> &Cond) {
 386:   // Block ends with fall-through condbranch.
 387:   assert(LastInst.getDesc().isConditionalBranch() &&
 388:          "Unknown conditional branch");
 389:   int NumOp = LastInst.getNumExplicitOperands();
 390:   Target = LastInst.getOperand(NumOp - 1).getMBB();
 391: 
 392:   Cond.push_back(MachineOperand::CreateImm(LastInst.getOpcode()));
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::getBranchDestBlock`, `parseCondBranch`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::getBranchDestBlock`, `parseCondBranch` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   for (int i = 0; i < NumOp - 1; i++)
 394:     Cond.push_back(LastInst.getOperand(i));
 395: }
 396: 
 397: bool LoongArchInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
 398:                                        MachineBasicBlock *&TBB,
 399:                                        MachineBasicBlock *&FBB,
 400:                                        SmallVectorImpl<MachineOperand> &Cond,
 401:                                        bool AllowModify) const {
 402:   TBB = FBB = nullptr;
 403:   Cond.clear();
 404: 
 405:   // If the block has no terminators, it just falls into the block after it.
 406:   MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
 407:   if (I == MBB.end() || !isUnpredicatedTerminator(*I))
 408:     return false;
 409: 
 410:   // Count the number of terminators and find the first unconditional or
 411:   // indirect branch.
 412:   MachineBasicBlock::iterator FirstUncondOrIndirectBr = MBB.end();
 413:   int NumTerminators = 0;
 414:   for (auto J = I.getReverse(); J != MBB.rend() && isUnpredicatedTerminator(*J);
 415:        J++) {
 416:     NumTerminators++;
 417:     if (J->getDesc().isUnconditionalBranch() ||
 418:         J->getDesc().isIndirectBranch()) {
 419:       FirstUncondOrIndirectBr = J.getReverse();
 420:     }
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::analyzeBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::analyzeBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 421-448 / 第 421-448 行
```cpp
 421:   }
 422: 
 423:   // If AllowModify is true, we can erase any terminators after
 424:   // FirstUncondOrIndirectBR.
 425:   if (AllowModify && FirstUncondOrIndirectBr != MBB.end()) {
 426:     while (std::next(FirstUncondOrIndirectBr) != MBB.end()) {
 427:       std::next(FirstUncondOrIndirectBr)->eraseFromParent();
 428:       NumTerminators--;
 429:     }
 430:     I = FirstUncondOrIndirectBr;
 431:   }
 432: 
 433:   // Handle a single unconditional branch.
 434:   if (NumTerminators == 1 && I->getDesc().isUnconditionalBranch()) {
 435:     TBB = getBranchDestBlock(*I);
 436:     return false;
 437:   }
 438: 
 439:   // Handle a single conditional branch.
 440:   if (NumTerminators == 1 && I->getDesc().isConditionalBranch()) {
 441:     parseCondBranch(*I, TBB, Cond);
 442:     return false;
 443:   }
 444: 
 445:   // Handle a conditional branch followed by an unconditional branch.
 446:   if (NumTerminators == 2 && std::prev(I)->getDesc().isConditionalBranch() &&
 447:       I->getDesc().isUnconditionalBranch()) {
 448:     parseCondBranch(*std::prev(I), TBB, Cond);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 449-476 / 第 449-476 行
```cpp
 449:     FBB = getBranchDestBlock(*I);
 450:     return false;
 451:   }
 452: 
 453:   // Otherwise, we can't handle this.
 454:   return true;
 455: }
 456: 
 457: bool LoongArchInstrInfo::isBranchOffsetInRange(unsigned BranchOp,
 458:                                                int64_t BrOffset) const {
 459:   switch (BranchOp) {
 460:   default:
 461:     llvm_unreachable("Unknown branch instruction!");
 462:   case LoongArch::BEQ:
 463:   case LoongArch::BNE:
 464:   case LoongArch::BLT:
 465:   case LoongArch::BGE:
 466:   case LoongArch::BLTU:
 467:   case LoongArch::BGEU:
 468:     return isInt<18>(BrOffset);
 469:   case LoongArch::BEQZ:
 470:   case LoongArch::BNEZ:
 471:   case LoongArch::BCEQZ:
 472:   case LoongArch::BCNEZ:
 473:     return isInt<23>(BrOffset);
 474:   case LoongArch::B:
 475:   case LoongArch::PseudoBR:
 476:     return isInt<28>(BrOffset);
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::isBranchOffsetInRange`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::isBranchOffsetInRange` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 477-504 / 第 477-504 行
```cpp
 477:   }
 478: }
 479: 
 480: bool LoongArchInstrInfo::isSafeToMove(const MachineInstr &MI,
 481:                                       const MachineBasicBlock *MBB,
 482:                                       const MachineFunction &MF) const {
 483:   if (DisableRelocSched) {
 484:     for (const MachineOperand &MO : MI.operands())
 485:       if (MO.getTargetFlags())
 486:         return false;
 487:   }
 488: 
 489:   auto MII = MI.getIterator();
 490:   auto MIE = MBB->end();
 491: 
 492:   // According to psABI v2.30:
 493:   //
 494:   // https://github.com/loongson/la-abi-specs/releases/tag/v2.30
 495:   //
 496:   // The following instruction patterns are prohibited from being reordered:
 497:   //
 498:   // * pcalau12i $a0, %pc_hi20(s)
 499:   //   addi.d $a1, $zero, %pc_lo12(s)
 500:   //   lu32i.d $a1, %pc64_lo20(s)
 501:   //   lu52i.d $a1, $a1, %pc64_hi12(s)
 502:   //
 503:   // * pcalau12i $a0, %got_pc_hi20(s) | %ld_pc_hi20(s) | %gd_pc_hi20(s)
 504:   //   addi.d $a1, $zero, %got_pc_lo12(s)
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::isSafeToMove`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::isSafeToMove` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 505-532 / 第 505-532 行
```cpp
 505:   //   lu32i.d $a1, %got64_pc_lo20(s)
 506:   //   lu52i.d $a1, $a1, %got64_pc_hi12(s)
 507:   //
 508:   // * pcalau12i $a0, %ie_pc_hi20(s)
 509:   //   addi.d $a1, $zero, %ie_pc_lo12(s)
 510:   //   lu32i.d $a1, %ie64_pc_lo20(s)
 511:   //   lu52i.d $a1, $a1, %ie64_pc_hi12(s)
 512:   //
 513:   // * pcalau12i $a0, %desc_pc_hi20(s)
 514:   //   addi.d $a1, $zero, %desc_pc_lo12(s)
 515:   //   lu32i.d $a1, %desc64_pc_lo20(s)
 516:   //   lu52i.d $a1, $a1, %desc64_pc_hi12(s)
 517:   //
 518:   // For simplicity, only pcalau12i and lu52i.d are marked as scheduling
 519:   // boundaries, and the instructions between them are guaranteed to be
 520:   // ordered according to data dependencies.
 521:   switch (MI.getOpcode()) {
 522:   case LoongArch::PCALAU12I: {
 523:     auto AddI = std::next(MII);
 524:     if (AddI == MIE || AddI->getOpcode() != LoongArch::ADDI_D)
 525:       break;
 526:     auto Lu32I = std::next(AddI);
 527:     if (Lu32I == MIE || Lu32I->getOpcode() != LoongArch::LU32I_D)
 528:       break;
 529:     auto MO0 = MI.getOperand(1).getTargetFlags();
 530:     auto MO1 = AddI->getOperand(2).getTargetFlags();
 531:     auto MO2 = Lu32I->getOperand(2).getTargetFlags();
 532:     if (MO0 == LoongArchII::MO_PCREL_HI && MO1 == LoongArchII::MO_PCREL_LO &&
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:         MO2 == LoongArchII::MO_PCREL64_LO)
 534:       return false;
 535:     if ((MO0 == LoongArchII::MO_GOT_PC_HI || MO0 == LoongArchII::MO_LD_PC_HI ||
 536:          MO0 == LoongArchII::MO_GD_PC_HI) &&
 537:         MO1 == LoongArchII::MO_GOT_PC_LO && MO2 == LoongArchII::MO_GOT_PC64_LO)
 538:       return false;
 539:     if (MO0 == LoongArchII::MO_IE_PC_HI && MO1 == LoongArchII::MO_IE_PC_LO &&
 540:         MO2 == LoongArchII::MO_IE_PC64_LO)
 541:       return false;
 542:     if (MO0 == LoongArchII::MO_DESC_PC_HI &&
 543:         MO1 == LoongArchII::MO_DESC_PC_LO &&
 544:         MO2 == LoongArchII::MO_DESC64_PC_LO)
 545:       return false;
 546:     break;
 547:   }
 548:   case LoongArch::LU52I_D: {
 549:     auto MO = MI.getOperand(2).getTargetFlags();
 550:     if (MO == LoongArchII::MO_PCREL64_HI || MO == LoongArchII::MO_GOT_PC64_HI ||
 551:         MO == LoongArchII::MO_IE_PC64_HI || MO == LoongArchII::MO_DESC64_PC_HI)
 552:       return false;
 553:     break;
 554:   }
 555:   default:
 556:     break;
 557:   }
 558: 
 559:   const auto &STI = MF.getSubtarget<LoongArchSubtarget>();
 560:   if (STI.hasFeature(LoongArch::FeatureRelax)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-588 / 第 561-588 行
```cpp
 561:     // When linker relaxation enabled, the following instruction patterns are
 562:     // prohibited from being reordered:
 563:     //
 564:     // * pcalau12i $a0, %pc_hi20(s)
 565:     //   addi.w/d $a0, $a0, %pc_lo12(s)
 566:     //
 567:     // * pcalau12i $a0, %got_pc_hi20(s)
 568:     //   ld.w/d $a0, $a0, %got_pc_lo12(s)
 569:     //
 570:     // * pcalau12i $a0, %ld_pc_hi20(s) | %gd_pc_hi20(s)
 571:     //   addi.w/d $a0, $a0, %got_pc_lo12(s)
 572:     //
 573:     // * pcalau12i $a0, %desc_pc_hi20(s)
 574:     //   addi.w/d  $a0, $a0, %desc_pc_lo12(s)
 575:     //   ld.w/d    $ra, $a0, %desc_ld(s)
 576:     //   jirl      $ra, $ra, %desc_call(s)
 577:     unsigned AddiOp = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 578:     unsigned LdOp = STI.is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
 579:     switch (MI.getOpcode()) {
 580:     case LoongArch::PCALAU12I: {
 581:       auto MO0 = LoongArchII::getDirectFlags(MI.getOperand(1));
 582:       auto SecondOp = std::next(MII);
 583:       if (MO0 == LoongArchII::MO_DESC_PC_HI) {
 584:         if (SecondOp == MIE || SecondOp->getOpcode() != AddiOp)
 585:           break;
 586:         auto Ld = std::next(SecondOp);
 587:         if (Ld == MIE || Ld->getOpcode() != LdOp)
 588:           break;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```cpp
 589:         auto MO1 = LoongArchII::getDirectFlags(SecondOp->getOperand(2));
 590:         auto MO2 = LoongArchII::getDirectFlags(Ld->getOperand(2));
 591:         if (MO1 == LoongArchII::MO_DESC_PC_LO && MO2 == LoongArchII::MO_DESC_LD)
 592:           return false;
 593:         break;
 594:       }
 595:       if (SecondOp == MIE ||
 596:           (SecondOp->getOpcode() != AddiOp && SecondOp->getOpcode() != LdOp))
 597:         break;
 598:       auto MO1 = LoongArchII::getDirectFlags(SecondOp->getOperand(2));
 599:       if (MO0 == LoongArchII::MO_PCREL_HI && SecondOp->getOpcode() == AddiOp &&
 600:           MO1 == LoongArchII::MO_PCREL_LO)
 601:         return false;
 602:       if (MO0 == LoongArchII::MO_GOT_PC_HI && SecondOp->getOpcode() == LdOp &&
 603:           MO1 == LoongArchII::MO_GOT_PC_LO)
 604:         return false;
 605:       if ((MO0 == LoongArchII::MO_LD_PC_HI ||
 606:            MO0 == LoongArchII::MO_GD_PC_HI) &&
 607:           SecondOp->getOpcode() == AddiOp && MO1 == LoongArchII::MO_GOT_PC_LO)
 608:         return false;
 609:       break;
 610:     }
 611:     case LoongArch::ADDI_W:
 612:     case LoongArch::ADDI_D: {
 613:       auto MO = LoongArchII::getDirectFlags(MI.getOperand(2));
 614:       if (MO == LoongArchII::MO_PCREL_LO || MO == LoongArchII::MO_GOT_PC_LO)
 615:         return false;
 616:       break;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```cpp
 617:     }
 618:     case LoongArch::LD_W:
 619:     case LoongArch::LD_D: {
 620:       auto MO = LoongArchII::getDirectFlags(MI.getOperand(2));
 621:       if (MO == LoongArchII::MO_GOT_PC_LO)
 622:         return false;
 623:       break;
 624:     }
 625:     case LoongArch::PseudoDESC_CALL: {
 626:       auto MO = LoongArchII::getDirectFlags(MI.getOperand(2));
 627:       if (MO == LoongArchII::MO_DESC_CALL)
 628:         return false;
 629:       break;
 630:     }
 631:     default:
 632:       break;
 633:     }
 634:   }
 635: 
 636:   return true;
 637: }
 638: 
 639: bool LoongArchInstrInfo::isSchedulingBoundary(const MachineInstr &MI,
 640:                                               const MachineBasicBlock *MBB,
 641:                                               const MachineFunction &MF) const {
 642:   if (TargetInstrInfo::isSchedulingBoundary(MI, MBB, MF))
 643:     return true;
 644: 
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::isSchedulingBoundary`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::isSchedulingBoundary` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```cpp
 645:   if (!isSafeToMove(MI, MBB, MF))
 646:     return true;
 647: 
 648:   return false;
 649: }
 650: 
 651: unsigned LoongArchInstrInfo::removeBranch(MachineBasicBlock &MBB,
 652:                                           int *BytesRemoved) const {
 653:   if (BytesRemoved)
 654:     *BytesRemoved = 0;
 655:   MachineBasicBlock::iterator I = MBB.getLastNonDebugInstr();
 656:   if (I == MBB.end())
 657:     return 0;
 658: 
 659:   if (!I->getDesc().isBranch())
 660:     return 0;
 661: 
 662:   // Remove the branch.
 663:   if (BytesRemoved)
 664:     *BytesRemoved += getInstSizeInBytes(*I);
 665:   I->eraseFromParent();
 666: 
 667:   I = MBB.end();
 668: 
 669:   if (I == MBB.begin())
 670:     return 1;
 671:   --I;
 672:   if (!I->getDesc().isConditionalBranch())
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::removeBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::removeBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 673-700 / 第 673-700 行
```cpp
 673:     return 1;
 674: 
 675:   // Remove the branch.
 676:   if (BytesRemoved)
 677:     *BytesRemoved += getInstSizeInBytes(*I);
 678:   I->eraseFromParent();
 679:   return 2;
 680: }
 681: 
 682: // Inserts a branch into the end of the specific MachineBasicBlock, returning
 683: // the number of instructions inserted.
 684: unsigned LoongArchInstrInfo::insertBranch(
 685:     MachineBasicBlock &MBB, MachineBasicBlock *TBB, MachineBasicBlock *FBB,
 686:     ArrayRef<MachineOperand> Cond, const DebugLoc &DL, int *BytesAdded) const {
 687:   if (BytesAdded)
 688:     *BytesAdded = 0;
 689: 
 690:   // Shouldn't be a fall through.
 691:   assert(TBB && "insertBranch must not be told to insert a fallthrough");
 692:   assert(Cond.size() <= 3 && Cond.size() != 1 &&
 693:          "LoongArch branch conditions have at most two components!");
 694: 
 695:   // Unconditional branch.
 696:   if (Cond.empty()) {
 697:     MachineInstr &MI = *BuildMI(&MBB, DL, get(LoongArch::PseudoBR)).addMBB(TBB);
 698:     if (BytesAdded)
 699:       *BytesAdded += getInstSizeInBytes(MI);
 700:     return 1;
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::insertBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::insertBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 701-728 / 第 701-728 行
```cpp
 701:   }
 702: 
 703:   // Either a one or two-way conditional branch.
 704:   MachineInstrBuilder MIB = BuildMI(&MBB, DL, get(Cond[0].getImm()));
 705:   for (unsigned i = 1; i < Cond.size(); ++i)
 706:     MIB.add(Cond[i]);
 707:   MIB.addMBB(TBB);
 708:   if (BytesAdded)
 709:     *BytesAdded += getInstSizeInBytes(*MIB);
 710: 
 711:   // One-way conditional branch.
 712:   if (!FBB)
 713:     return 1;
 714: 
 715:   // Two-way conditional branch.
 716:   MachineInstr &MI = *BuildMI(&MBB, DL, get(LoongArch::PseudoBR)).addMBB(FBB);
 717:   if (BytesAdded)
 718:     *BytesAdded += getInstSizeInBytes(MI);
 719:   return 2;
 720: }
 721: 
 722: void LoongArchInstrInfo::insertIndirectBranch(MachineBasicBlock &MBB,
 723:                                               MachineBasicBlock &DestBB,
 724:                                               MachineBasicBlock &RestoreBB,
 725:                                               const DebugLoc &DL,
 726:                                               int64_t BrOffset,
 727:                                               RegScavenger *RS) const {
 728:   assert(RS && "RegScavenger required for long branching");
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::insertIndirectBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::insertIndirectBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 729-756 / 第 729-756 行
```cpp
 729:   assert(MBB.empty() &&
 730:          "new block should be inserted for expanding unconditional branch");
 731:   assert(MBB.pred_size() == 1);
 732: 
 733:   MachineFunction *MF = MBB.getParent();
 734:   MachineRegisterInfo &MRI = MF->getRegInfo();
 735:   const TargetRegisterInfo *TRI = MF->getSubtarget().getRegisterInfo();
 736:   LoongArchMachineFunctionInfo *LAFI =
 737:       MF->getInfo<LoongArchMachineFunctionInfo>();
 738:   bool Has32S = STI.hasFeature(LoongArch::Feature32S);
 739: 
 740:   if (!isInt<32>(BrOffset))
 741:     report_fatal_error(
 742:         "Branch offsets outside of the signed 32-bit range not supported");
 743: 
 744:   Register ScratchReg = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
 745:   MachineInstr *PCAI = nullptr;
 746:   MachineInstr *ADDI = nullptr;
 747:   auto II = MBB.end();
 748:   unsigned ADDIOp = STI.is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 749: 
 750:   if (Has32S) {
 751:     PCAI = BuildMI(MBB, II, DL, get(LoongArch::PCALAU12I), ScratchReg)
 752:                .addMBB(&DestBB, LoongArchII::MO_PCREL_HI);
 753:     ADDI = BuildMI(MBB, II, DL, get(ADDIOp), ScratchReg)
 754:                .addReg(ScratchReg)
 755:                .addMBB(&DestBB, LoongArchII::MO_PCREL_LO);
 756:   } else {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 757-784 / 第 757-784 行
```cpp
 757:     MCSymbol *PCAddSymbol = MF->getContext().createNamedTempSymbol("pcadd_hi");
 758:     PCAI = BuildMI(MBB, II, DL, get(LoongArch::PCADDU12I), ScratchReg)
 759:                .addMBB(&DestBB, LoongArchII::MO_PCADD_HI);
 760:     PCAI->setPreInstrSymbol(*MF, PCAddSymbol);
 761:     ADDI = BuildMI(MBB, II, DL, get(ADDIOp), ScratchReg)
 762:                .addReg(ScratchReg)
 763:                .addSym(PCAddSymbol, LoongArchII::MO_PCADD_LO);
 764:   }
 765:   BuildMI(MBB, II, DL, get(LoongArch::PseudoBRIND))
 766:       .addReg(ScratchReg, RegState::Kill)
 767:       .addImm(0);
 768: 
 769:   RS->enterBasicBlockEnd(MBB);
 770:   Register Scav = RS->scavengeRegisterBackwards(
 771:       LoongArch::GPRRegClass, PCAI->getIterator(), /*RestoreAfter=*/false,
 772:       /*SPAdj=*/0, /*AllowSpill=*/false);
 773:   if (Scav != LoongArch::NoRegister)
 774:     RS->setRegUsed(Scav);
 775:   else {
 776:     // When there is no scavenged register, it needs to specify a register.
 777:     // Specify t8 register because it won't be used too often.
 778:     Scav = LoongArch::R20;
 779:     int FrameIndex = LAFI->getBranchRelaxationSpillFrameIndex();
 780:     if (FrameIndex == -1)
 781:       report_fatal_error("The function size is incorrectly estimated.");
 782:     storeRegToStackSlot(MBB, PCAI, Scav, /*IsKill=*/true, FrameIndex,
 783:                         &LoongArch::GPRRegClass, Register());
 784:     TRI->eliminateFrameIndex(std::prev(PCAI->getIterator()),
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 785-812 / 第 785-812 行
```cpp
 785:                              /*SpAdj=*/0, /*FIOperandNum=*/1);
 786:     PCAI->getOperand(1).setMBB(&RestoreBB);
 787:     if (Has32S)
 788:       ADDI->getOperand(2).setMBB(&RestoreBB);
 789:     loadRegFromStackSlot(RestoreBB, RestoreBB.end(), Scav, FrameIndex,
 790:                          &LoongArch::GPRRegClass, Register());
 791:     TRI->eliminateFrameIndex(RestoreBB.back(),
 792:                              /*SpAdj=*/0, /*FIOperandNum=*/1);
 793:   }
 794:   MRI.replaceRegWith(ScratchReg, Scav);
 795:   MRI.clearVirtRegs();
 796: }
 797: 
 798: static unsigned getOppositeBranchOpc(unsigned Opc) {
 799:   switch (Opc) {
 800:   default:
 801:     llvm_unreachable("Unrecognized conditional branch");
 802:   case LoongArch::BEQ:
 803:     return LoongArch::BNE;
 804:   case LoongArch::BNE:
 805:     return LoongArch::BEQ;
 806:   case LoongArch::BEQZ:
 807:     return LoongArch::BNEZ;
 808:   case LoongArch::BNEZ:
 809:     return LoongArch::BEQZ;
 810:   case LoongArch::BCEQZ:
 811:     return LoongArch::BCNEZ;
 812:   case LoongArch::BCNEZ:
```
- **EN**: The range implements or declares functions including `getOppositeBranchOpc`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getOppositeBranchOpc` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 813-840 / 第 813-840 行
```cpp
 813:     return LoongArch::BCEQZ;
 814:   case LoongArch::BLT:
 815:     return LoongArch::BGE;
 816:   case LoongArch::BGE:
 817:     return LoongArch::BLT;
 818:   case LoongArch::BLTU:
 819:     return LoongArch::BGEU;
 820:   case LoongArch::BGEU:
 821:     return LoongArch::BLTU;
 822:   }
 823: }
 824: 
 825: bool LoongArchInstrInfo::reverseBranchCondition(
 826:     SmallVectorImpl<MachineOperand> &Cond) const {
 827:   assert((Cond.size() && Cond.size() <= 3) && "Invalid branch condition!");
 828:   Cond[0].setImm(getOppositeBranchOpc(Cond[0].getImm()));
 829:   return false;
 830: }
 831: 
 832: std::pair<unsigned, unsigned>
 833: LoongArchInstrInfo::decomposeMachineOperandsTargetFlags(unsigned TF) const {
 834:   const unsigned Mask = LoongArchII::MO_DIRECT_FLAG_MASK;
 835:   return std::make_pair(TF & Mask, TF & ~Mask);
 836: }
 837: 
 838: ArrayRef<std::pair<unsigned, const char *>>
 839: LoongArchInstrInfo::getSerializableDirectMachineOperandTargetFlags() const {
 840:   using namespace LoongArchII;
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::reverseBranchCondition`, `LoongArchInstrInfo::decomposeMachineOperandsTargetFlags`, `LoongArchInstrInfo::getSerializableDirectMachineOperandTargetFlags`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::reverseBranchCondition`, `LoongArchInstrInfo::decomposeMachineOperandsTargetFlags`, `LoongArchInstrInfo::getSerializableDirectMachineOperandTargetFlags` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 841-868 / 第 841-868 行
```cpp
 841:   // TODO: Add more target flags.
 842:   static const std::pair<unsigned, const char *> TargetFlags[] = {
 843:       {MO_CALL, "loongarch-call"},
 844:       {MO_CALL_PLT, "loongarch-call-plt"},
 845:       {MO_PCREL_HI, "loongarch-pcrel-hi"},
 846:       {MO_PCREL_LO, "loongarch-pcrel-lo"},
 847:       {MO_PCREL64_LO, "loongarch-pcrel64-lo"},
 848:       {MO_PCREL64_HI, "loongarch-pcrel64-hi"},
 849:       {MO_GOT_PC_HI, "loongarch-got-pc-hi"},
 850:       {MO_GOT_PC_LO, "loongarch-got-pc-lo"},
 851:       {MO_GOT_PC64_LO, "loongarch-got-pc64-lo"},
 852:       {MO_GOT_PC64_HI, "loongarch-got-pc64-hi"},
 853:       {MO_LE_HI, "loongarch-le-hi"},
 854:       {MO_LE_LO, "loongarch-le-lo"},
 855:       {MO_LE64_LO, "loongarch-le64-lo"},
 856:       {MO_LE64_HI, "loongarch-le64-hi"},
 857:       {MO_IE_PC_HI, "loongarch-ie-pc-hi"},
 858:       {MO_IE_PC_LO, "loongarch-ie-pc-lo"},
 859:       {MO_IE_PC64_LO, "loongarch-ie-pc64-lo"},
 860:       {MO_IE_PC64_HI, "loongarch-ie-pc64-hi"},
 861:       {MO_LD_PC_HI, "loongarch-ld-pc-hi"},
 862:       {MO_GD_PC_HI, "loongarch-gd-pc-hi"},
 863:       {MO_CALL30, "loongarch-call30"},
 864:       {MO_CALL36, "loongarch-call36"},
 865:       {MO_DESC_PC_HI, "loongarch-desc-pc-hi"},
 866:       {MO_DESC_PC_LO, "loongarch-desc-pc-lo"},
 867:       {MO_DESC64_PC_LO, "loongarch-desc64-pc-lo"},
 868:       {MO_DESC64_PC_HI, "loongarch-desc64-pc-hi"},
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 869-896 / 第 869-896 行
```cpp
 869:       {MO_DESC_LD, "loongarch-desc-ld"},
 870:       {MO_DESC_CALL, "loongarch-desc-call"},
 871:       {MO_LE_HI_R, "loongarch-le-hi-r"},
 872:       {MO_LE_ADD_R, "loongarch-le-add-r"},
 873:       {MO_LE_LO_R, "loongarch-le-lo-r"},
 874:       {MO_PCADD_HI, "loongarch-pcadd-hi"},
 875:       {MO_PCADD_LO, "loongarch-pcadd-lo"},
 876:       {MO_GOT_PCADD_HI, "loongarch-got-pcadd-hi"},
 877:       {MO_GOT_PCADD_LO, "loongarch-got-pcadd-lo"},
 878:       {MO_IE_PCADD_HI, "loongarch-ie-pcadd-hi"},
 879:       {MO_IE_PCADD_LO, "loongarch-ie-pcadd-lo"},
 880:       {MO_LD_PCADD_HI, "loongarch-ld-pcadd-hi"},
 881:       {MO_LD_PCADD_LO, "loongarch-ld-pcadd-lo"},
 882:       {MO_GD_PCADD_HI, "loongarch-gd-pcadd-hi"},
 883:       {MO_GD_PCADD_LO, "loongarch-gd-pcadd-lo"},
 884:       {MO_DESC_PCADD_HI, "loongarch-pcadd-desc-hi"},
 885:       {MO_DESC_PCADD_LO, "loongarch-pcadd-desc-lo"}};
 886:   return ArrayRef(TargetFlags);
 887: }
 888: 
 889: ArrayRef<std::pair<unsigned, const char *>>
 890: LoongArchInstrInfo::getSerializableBitmaskMachineOperandTargetFlags() const {
 891:   using namespace LoongArchII;
 892:   static const std::pair<unsigned, const char *> TargetFlags[] = {
 893:       {MO_RELAX, "loongarch-relax"}};
 894:   return ArrayRef(TargetFlags);
 895: }
 896: 
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::getSerializableBitmaskMachineOperandTargetFlags`.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::getSerializableBitmaskMachineOperandTargetFlags` 等函数。

### Lines 897-924 / 第 897-924 行
```cpp
 897: bool LoongArchInstrInfo::canFoldIntoAddrMode(const MachineInstr &MemI,
 898:                                              Register Reg,
 899:                                              const MachineInstr &AddrI,
 900:                                              ExtAddrMode &AM) const {
 901:   enum MemIOffsetType {
 902:     Imm14Shift2,
 903:     Imm12,
 904:     Imm11Shift1,
 905:     Imm10Shift2,
 906:     Imm9Shift3,
 907:     Imm8,
 908:     Imm8Shift1,
 909:     Imm8Shift2,
 910:     Imm8Shift3
 911:   };
 912: 
 913:   MemIOffsetType OT;
 914:   switch (MemI.getOpcode()) {
 915:   default:
 916:     return false;
 917:   case LoongArch::LDPTR_W:
 918:   case LoongArch::LDPTR_D:
 919:   case LoongArch::STPTR_W:
 920:   case LoongArch::STPTR_D:
 921:     OT = Imm14Shift2;
 922:     break;
 923:   case LoongArch::LD_B:
 924:   case LoongArch::LD_H:
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::canFoldIntoAddrMode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::canFoldIntoAddrMode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 925-952 / 第 925-952 行
```cpp
 925:   case LoongArch::LD_W:
 926:   case LoongArch::LD_D:
 927:   case LoongArch::LD_BU:
 928:   case LoongArch::LD_HU:
 929:   case LoongArch::LD_WU:
 930:   case LoongArch::ST_B:
 931:   case LoongArch::ST_H:
 932:   case LoongArch::ST_W:
 933:   case LoongArch::ST_D:
 934:   case LoongArch::FLD_S:
 935:   case LoongArch::FLD_D:
 936:   case LoongArch::FST_S:
 937:   case LoongArch::FST_D:
 938:   case LoongArch::VLD:
 939:   case LoongArch::VST:
 940:   case LoongArch::XVLD:
 941:   case LoongArch::XVST:
 942:   case LoongArch::VLDREPL_B:
 943:   case LoongArch::XVLDREPL_B:
 944:     OT = Imm12;
 945:     break;
 946:   case LoongArch::VLDREPL_H:
 947:   case LoongArch::XVLDREPL_H:
 948:     OT = Imm11Shift1;
 949:     break;
 950:   case LoongArch::VLDREPL_W:
 951:   case LoongArch::XVLDREPL_W:
 952:     OT = Imm10Shift2;
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 953-980 / 第 953-980 行
```cpp
 953:     break;
 954:   case LoongArch::VLDREPL_D:
 955:   case LoongArch::XVLDREPL_D:
 956:     OT = Imm9Shift3;
 957:     break;
 958:   case LoongArch::VSTELM_B:
 959:   case LoongArch::XVSTELM_B:
 960:     OT = Imm8;
 961:     break;
 962:   case LoongArch::VSTELM_H:
 963:   case LoongArch::XVSTELM_H:
 964:     OT = Imm8Shift1;
 965:     break;
 966:   case LoongArch::VSTELM_W:
 967:   case LoongArch::XVSTELM_W:
 968:     OT = Imm8Shift2;
 969:     break;
 970:   case LoongArch::VSTELM_D:
 971:   case LoongArch::XVSTELM_D:
 972:     OT = Imm8Shift3;
 973:     break;
 974:   }
 975: 
 976:   if (MemI.getOperand(0).getReg() == Reg)
 977:     return false;
 978: 
 979:   if ((AddrI.getOpcode() != LoongArch::ADDI_W &&
 980:        AddrI.getOpcode() != LoongArch::ADDI_D) ||
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 981-1008 / 第 981-1008 行
```cpp
 981:       !AddrI.getOperand(1).isReg() || !AddrI.getOperand(2).isImm())
 982:     return false;
 983: 
 984:   int64_t OldOffset = MemI.getOperand(2).getImm();
 985:   int64_t Disp = AddrI.getOperand(2).getImm();
 986:   int64_t NewOffset = OldOffset + Disp;
 987:   if (!STI.is64Bit())
 988:     NewOffset = SignExtend64<32>(NewOffset);
 989: 
 990:   if (!(OT == Imm14Shift2 && isShiftedInt<14, 2>(NewOffset) && STI.hasUAL()) &&
 991:       !(OT == Imm12 && isInt<12>(NewOffset)) &&
 992:       !(OT == Imm11Shift1 && isShiftedInt<11, 1>(NewOffset)) &&
 993:       !(OT == Imm10Shift2 && isShiftedInt<10, 2>(NewOffset)) &&
 994:       !(OT == Imm9Shift3 && isShiftedInt<9, 3>(NewOffset)) &&
 995:       !(OT == Imm8 && isInt<8>(NewOffset)) &&
 996:       !(OT == Imm8Shift1 && isShiftedInt<8, 1>(NewOffset)) &&
 997:       !(OT == Imm8Shift2 && isShiftedInt<8, 2>(NewOffset)) &&
 998:       !(OT == Imm8Shift3 && isShiftedInt<8, 3>(NewOffset)))
 999:     return false;
1000: 
1001:   AM.BaseReg = AddrI.getOperand(1).getReg();
1002:   AM.ScaledReg = 0;
1003:   AM.Scale = 0;
1004:   AM.Displacement = NewOffset;
1005:   AM.Form = ExtAddrMode::Formula::Basic;
1006:   return true;
1007: }
1008: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1009-1036 / 第 1009-1036 行
```cpp
1009: MachineInstr *
1010: LoongArchInstrInfo::emitLdStWithAddr(MachineInstr &MemI,
1011:                                      const ExtAddrMode &AM) const {
1012:   const DebugLoc &DL = MemI.getDebugLoc();
1013:   MachineBasicBlock &MBB = *MemI.getParent();
1014: 
1015:   assert(AM.ScaledReg == 0 && AM.Scale == 0 &&
1016:          "Addressing mode not supported for folding");
1017: 
1018:   unsigned MemIOp = MemI.getOpcode();
1019:   switch (MemIOp) {
1020:   default:
1021:     return BuildMI(MBB, MemI, DL, get(MemIOp))
1022:         .addReg(MemI.getOperand(0).getReg(), getDefRegState(MemI.mayLoad()))
1023:         .addReg(AM.BaseReg)
1024:         .addImm(AM.Displacement)
1025:         .setMemRefs(MemI.memoperands())
1026:         .setMIFlags(MemI.getFlags());
1027:   case LoongArch::VSTELM_B:
1028:   case LoongArch::VSTELM_H:
1029:   case LoongArch::VSTELM_W:
1030:   case LoongArch::VSTELM_D:
1031:   case LoongArch::XVSTELM_B:
1032:   case LoongArch::XVSTELM_H:
1033:   case LoongArch::XVSTELM_W:
1034:   case LoongArch::XVSTELM_D:
1035:     return BuildMI(MBB, MemI, DL, get(MemIOp))
1036:         .addReg(MemI.getOperand(0).getReg())
```
- **EN**: The range implements or declares functions including `LoongArchInstrInfo::emitLdStWithAddr`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchInstrInfo::emitLdStWithAddr`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1037-1049 / 第 1037-1049 行
```cpp
1037:         .addReg(AM.BaseReg)
1038:         .addImm(AM.Displacement)
1039:         .addImm(MemI.getOperand(3).getImm())
1040:         .setMemRefs(MemI.memoperands())
1041:         .setMIFlags(MemI.getFlags());
1042:   }
1043: }
1044: 
1045: // Returns true if this is the sext.w pattern, addi.w rd, rs, 0.
1046: bool LoongArch::isSEXT_W(const MachineInstr &MI) {
1047:   return MI.getOpcode() == LoongArch::ADDI_W && MI.getOperand(1).isReg() &&
1048:          MI.getOperand(2).isImm() && MI.getOperand(2).getImm() == 0;
1049: }
```
- **EN**: The range implements or declares functions including `LoongArch::isSEXT_W`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArch::isSEXT_W` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `LoongArchInstrInfo.h`
- `LoongArch.h`
- `LoongArchMachineFunctionInfo.h`
- `LoongArchRegisterInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `MCTargetDesc/LoongArchMatInt.h`
- `llvm/CodeGen/RegisterScavenging.h`
- `llvm/CodeGen/StackMaps.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCInstBuilder.h`
- `llvm/Support/CommandLine.h`
- `LoongArchGenInstrInfo.inc`
