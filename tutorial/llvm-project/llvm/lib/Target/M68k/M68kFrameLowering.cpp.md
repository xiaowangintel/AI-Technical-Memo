# M68kFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kFrameLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements stack frame lowering and prologue/epilogue logic for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责实现栈帧降低以及序言/尾声逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- M68kFrameLowering.cpp - M68k Frame Information ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains the M68k implementation of TargetFrameLowering class.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "M68kFrameLowering.h"
  15: 
  16: #include "M68kInstrBuilder.h"
  17: #include "M68kInstrInfo.h"
  18: #include "M68kMachineFunction.h"
  19: #include "M68kSubtarget.h"
  20: 
  21: #include "llvm/ADT/SmallSet.h"
  22: #include "llvm/CodeGen/MachineFrameInfo.h"
  23: #include "llvm/CodeGen/MachineFunction.h"
  24: #include "llvm/CodeGen/MachineInstrBuilder.h"
  25: #include "llvm/CodeGen/MachineModuleInfo.h"
  26: #include "llvm/CodeGen/MachineRegisterInfo.h"
  27: #include "llvm/IR/DataLayout.h"
  28: #include "llvm/IR/Function.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kFrameLowering.h`, `M68kInstrBuilder.h`, `M68kInstrInfo.h`, `M68kMachineFunction.h`, `M68kSubtarget.h`, `SmallSet.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kFrameLowering.h`, `M68kInstrBuilder.h`, `M68kInstrInfo.h`, `M68kMachineFunction.h`, `M68kSubtarget.h`, `SmallSet.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 29-56 / 第 29-56 行
```cpp
  29: #include "llvm/Support/Alignment.h"
  30: #include "llvm/Support/CommandLine.h"
  31: #include "llvm/Target/TargetMachine.h"
  32: #include "llvm/Target/TargetOptions.h"
  33: 
  34: using namespace llvm;
  35: 
  36: #define DEBUG_TYPE "m68k-frame"
  37: 
  38: M68kFrameLowering::M68kFrameLowering(const M68kSubtarget &STI, Align Alignment)
  39:     : TargetFrameLowering(StackGrowsDown, Alignment, -4), STI(STI),
  40:       TII(*STI.getInstrInfo()), TRI(STI.getRegisterInfo()) {
  41:   SlotSize = STI.getSlotSize();
  42:   StackPtr = TRI->getStackRegister();
  43: }
  44: 
  45: bool M68kFrameLowering::hasFPImpl(const MachineFunction &MF) const {
  46:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  47:   const TargetRegisterInfo *TRI = STI.getRegisterInfo();
  48: 
  49:   return MF.getTarget().Options.DisableFramePointerElim(MF) ||
  50:          MFI.hasVarSizedObjects() || MFI.isFrameAddressTaken() ||
  51:          TRI->hasStackRealignment(MF);
  52: }
  53: 
  54: // FIXME Make sure no other factors prevent us from reserving call frame
  55: bool M68kFrameLowering::hasReservedCallFrame(const MachineFunction &MF) const {
  56:   return !MF.getFrameInfo().hasVarSizedObjects() &&
```
- **EN**: It imports dependencies such as `Alignment.h`, `CommandLine.h`, `TargetMachine.h`, `TargetOptions.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `TargetFrameLowering`, `M68kFrameLowering::hasFPImpl`, `M68kFrameLowering::hasReservedCallFrame`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `Alignment.h`, `CommandLine.h`, `TargetMachine.h`, `TargetOptions.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `TargetFrameLowering`, `M68kFrameLowering::hasFPImpl`, `M68kFrameLowering::hasReservedCallFrame` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 57-84 / 第 57-84 行
```cpp
  57:          !MF.getInfo<M68kMachineFunctionInfo>()->getHasPushSequences();
  58: }
  59: 
  60: bool M68kFrameLowering::canSimplifyCallFramePseudos(
  61:     const MachineFunction &MF) const {
  62:   return hasReservedCallFrame(MF) ||
  63:          (hasFP(MF) && !TRI->hasStackRealignment(MF)) ||
  64:          TRI->hasBasePointer(MF);
  65: }
  66: 
  67: bool M68kFrameLowering::needsFrameIndexResolution(
  68:     const MachineFunction &MF) const {
  69:   return MF.getFrameInfo().hasStackObjects() ||
  70:          MF.getInfo<M68kMachineFunctionInfo>()->getHasPushSequences();
  71: }
  72: 
  73: // NOTE: this only has a subset of the full frame index logic. In
  74: // particular, the FI < 0 and AfterFPPop logic is handled in
  75: // M68kRegisterInfo::eliminateFrameIndex, but not here. Possibly
  76: // (probably?) it should be moved into here.
  77: StackOffset
  78: M68kFrameLowering::getFrameIndexReference(const MachineFunction &MF, int FI,
  79:                                           Register &FrameReg) const {
  80:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  81: 
  82:   // We can't calculate offset from frame pointer if the stack is realigned,
  83:   // so enforce usage of stack/base pointer.  The base pointer is used when we
  84:   // have dynamic allocas in addition to dynamic realignment.
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::canSimplifyCallFramePseudos`, `M68kFrameLowering::needsFrameIndexResolution`, `M68kFrameLowering::getFrameIndexReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::canSimplifyCallFramePseudos`, `M68kFrameLowering::needsFrameIndexResolution`, `M68kFrameLowering::getFrameIndexReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   if (TRI->hasBasePointer(MF))
  86:     FrameReg = TRI->getBaseRegister();
  87:   else if (TRI->hasStackRealignment(MF))
  88:     FrameReg = TRI->getStackRegister();
  89:   else
  90:     FrameReg = TRI->getFrameRegister(MF);
  91: 
  92:   // Offset will hold the offset from the stack pointer at function entry to the
  93:   // object.
  94:   // We need to factor in additional offsets applied during the prologue to the
  95:   // frame, base, and stack pointer depending on which is used.
  96:   int Offset = MFI.getObjectOffset(FI) - getOffsetOfLocalArea();
  97:   const M68kMachineFunctionInfo *MMFI = MF.getInfo<M68kMachineFunctionInfo>();
  98:   uint64_t StackSize = MFI.getStackSize();
  99:   bool HasFP = hasFP(MF);
 100: 
 101:   // TODO: Support tail calls
 102:   if (TRI->hasBasePointer(MF)) {
 103:     assert(HasFP && "VLAs and dynamic stack realign, but no FP?!");
 104:     if (FI < 0) {
 105:       // Skip the saved FP.
 106:       return StackOffset::getFixed(Offset + SlotSize);
 107:     }
 108: 
 109:     assert((-(Offset + StackSize)) % MFI.getObjectAlign(FI).value() == 0);
 110:     return StackOffset::getFixed(Offset + StackSize);
 111:   }
 112:   if (TRI->hasStackRealignment(MF)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 113-140 / 第 113-140 行
```cpp
 113:     if (FI < 0) {
 114:       // Skip the saved FP.
 115:       return StackOffset::getFixed(Offset + SlotSize);
 116:     }
 117: 
 118:     assert((-(Offset + StackSize)) % MFI.getObjectAlign(FI).value() == 0);
 119:     return StackOffset::getFixed(Offset + StackSize);
 120:   }
 121: 
 122:   if (!HasFP)
 123:     return StackOffset::getFixed(Offset + StackSize);
 124: 
 125:   // Skip the saved FP.
 126:   Offset += SlotSize;
 127: 
 128:   // Skip the RETADDR move area
 129:   int TailCallReturnAddrDelta = MMFI->getTCReturnAddrDelta();
 130:   if (TailCallReturnAddrDelta < 0)
 131:     Offset -= TailCallReturnAddrDelta;
 132: 
 133:   return StackOffset::getFixed(Offset);
 134: }
 135: 
 136: /// Return a caller-saved register that isn't live
 137: /// when it reaches the "return" instruction. We can then pop a stack object
 138: /// to this register without worry about clobbering it.
 139: static unsigned findDeadCallerSavedReg(MachineBasicBlock &MBB,
 140:                                        MachineBasicBlock::iterator &MBBI,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 141-168 / 第 141-168 行
```cpp
 141:                                        const M68kRegisterInfo *TRI) {
 142:   const MachineFunction *MF = MBB.getParent();
 143:   if (MF->callsEHReturn())
 144:     return 0;
 145: 
 146:   const TargetRegisterClass &AvailableRegs = *TRI->getRegsForTailCall(*MF);
 147: 
 148:   if (MBBI == MBB.end())
 149:     return 0;
 150: 
 151:   switch (MBBI->getOpcode()) {
 152:   default:
 153:     return 0;
 154:   case TargetOpcode::PATCHABLE_RET:
 155:   case M68k::RET: {
 156:     SmallSet<uint16_t, 8> Uses;
 157: 
 158:     for (unsigned i = 0, e = MBBI->getNumOperands(); i != e; ++i) {
 159:       MachineOperand &MO = MBBI->getOperand(i);
 160:       if (!MO.isReg() || MO.isDef())
 161:         continue;
 162:       Register Reg = MO.getReg();
 163:       if (!Reg)
 164:         continue;
 165:       for (MCRegAliasIterator AI(Reg, TRI, true); AI.isValid(); ++AI)
 166:         Uses.insert(*AI);
 167:     }
 168: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 169-196 / 第 169-196 行
```cpp
 169:     for (auto CS : AvailableRegs)
 170:       if (!Uses.count(CS))
 171:         return CS;
 172:   }
 173:   }
 174: 
 175:   return 0;
 176: }
 177: 
 178: static bool isRegLiveIn(MachineBasicBlock &MBB, unsigned Reg) {
 179:   return llvm::any_of(MBB.liveins(),
 180:                       [Reg](MachineBasicBlock::RegisterMaskPair RegMask) {
 181:                         return RegMask.PhysReg == Reg;
 182:                       });
 183: }
 184: 
 185: uint64_t
 186: M68kFrameLowering::calculateMaxStackAlign(const MachineFunction &MF) const {
 187:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 188:   uint64_t MaxAlign = MFI.getMaxAlign().value(); // Desired stack alignment.
 189:   unsigned StackAlign = getStackAlignment();     // ABI alignment
 190:   if (MF.getFunction().hasFnAttribute("stackrealign")) {
 191:     if (MFI.hasCalls())
 192:       MaxAlign = (StackAlign > MaxAlign) ? StackAlign : MaxAlign;
 193:     else if (MaxAlign < SlotSize)
 194:       MaxAlign = SlotSize;
 195:   }
 196:   return MaxAlign;
```
- **EN**: The range implements or declares functions including `isRegLiveIn`, `M68kFrameLowering::calculateMaxStackAlign`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `isRegLiveIn`, `M68kFrameLowering::calculateMaxStackAlign` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```cpp
 197: }
 198: 
 199: void M68kFrameLowering::BuildStackAlignAND(MachineBasicBlock &MBB,
 200:                                            MachineBasicBlock::iterator MBBI,
 201:                                            const DebugLoc &DL, unsigned Reg,
 202:                                            uint64_t MaxAlign) const {
 203:   uint64_t Val = -MaxAlign;
 204:   unsigned AndOp = M68k::AND32di;
 205:   unsigned MovOp = M68k::MOV32rr;
 206: 
 207:   // This function is normally used with SP which is Address Register, but AND,
 208:   // or any other logical instructions in M68k do not support ARs so we need
 209:   // to use a temp Data Register to perform the op.
 210:   unsigned Tmp = M68k::D0;
 211: 
 212:   BuildMI(MBB, MBBI, DL, TII.get(MovOp), Tmp)
 213:       .addReg(Reg)
 214:       .setMIFlag(MachineInstr::FrameSetup);
 215: 
 216:   MachineInstr *MI = BuildMI(MBB, MBBI, DL, TII.get(AndOp), Tmp)
 217:                          .addReg(Tmp)
 218:                          .addImm(Val)
 219:                          .setMIFlag(MachineInstr::FrameSetup);
 220: 
 221:   // The CCR implicit def is dead.
 222:   MI->getOperand(3).setIsDead();
 223: 
 224:   BuildMI(MBB, MBBI, DL, TII.get(MovOp), Reg)
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::BuildStackAlignAND`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::BuildStackAlignAND`, `BuildMI`, `BuildMI` 等函数。

### Lines 225-252 / 第 225-252 行
```cpp
 225:       .addReg(Tmp)
 226:       .setMIFlag(MachineInstr::FrameSetup);
 227: }
 228: 
 229: MachineBasicBlock::iterator M68kFrameLowering::eliminateCallFramePseudoInstr(
 230:     MachineFunction &MF, MachineBasicBlock &MBB,
 231:     MachineBasicBlock::iterator I) const {
 232:   bool ReserveCallFrame = hasReservedCallFrame(MF);
 233:   unsigned Opcode = I->getOpcode();
 234:   bool IsDestroy = Opcode == TII.getCallFrameDestroyOpcode();
 235:   DebugLoc DL = I->getDebugLoc();
 236:   uint64_t Amount = I->getOperand(0).getImm();
 237:   uint64_t InternalAmt = (IsDestroy || Amount) ? I->getOperand(1).getImm() : 0;
 238:   I = MBB.erase(I);
 239: 
 240:   if (!ReserveCallFrame) {
 241:     // If the stack pointer can be changed after prologue, turn the
 242:     // adjcallstackup instruction into a 'sub %SP, <amt>' and the
 243:     // adjcallstackdown instruction into 'add %SP, <amt>'
 244: 
 245:     // We need to keep the stack aligned properly.  To do this, we round the
 246:     // amount of space needed for the outgoing arguments up to the next
 247:     // alignment boundary.
 248:     unsigned StackAlign = getStackAlignment();
 249:     Amount = alignTo(Amount, StackAlign);
 250: 
 251:     bool DwarfCFI = MF.needsFrameMoves();
 252: 
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::eliminateCallFramePseudoInstr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::eliminateCallFramePseudoInstr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:     // If we have any exception handlers in this function, and we adjust
 254:     // the SP before calls, we may need to indicate this to the unwinder
 255:     // using GNU_ARGS_SIZE. Note that this may be necessary even when
 256:     // Amount == 0, because the preceding function may have set a non-0
 257:     // GNU_ARGS_SIZE.
 258:     // TODO: We don't need to reset this between subsequent functions,
 259:     // if it didn't change.
 260:     bool HasDwarfEHHandlers = !MF.getLandingPads().empty();
 261: 
 262:     if (HasDwarfEHHandlers && !IsDestroy &&
 263:         MF.getInfo<M68kMachineFunctionInfo>()->getHasPushSequences()) {
 264:       BuildCFI(MBB, I, DL,
 265:                MCCFIInstruction::createGnuArgsSize(nullptr, Amount));
 266:     }
 267: 
 268:     if (Amount == 0)
 269:       return I;
 270: 
 271:     // Factor out the amount that gets handled inside the sequence
 272:     // (Pushes of argument for frame setup, callee pops for frame destroy)
 273:     Amount -= InternalAmt;
 274: 
 275:     // TODO: This is needed only if we require precise CFA.
 276:     // If this is a callee-pop calling convention, emit a CFA adjust for
 277:     // the amount the callee popped.
 278:     if (IsDestroy && InternalAmt && DwarfCFI && !hasFP(MF))
 279:       BuildCFI(MBB, I, DL,
 280:                MCCFIInstruction::createAdjustCfaOffset(nullptr, -InternalAmt));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-308 / 第 281-308 行
```cpp
 281: 
 282:     // Add Amount to SP to destroy a frame, or subtract to setup.
 283:     int64_t StackAdjustment = IsDestroy ? Amount : -Amount;
 284:     int64_t CfaAdjustment = -StackAdjustment;
 285: 
 286:     if (StackAdjustment) {
 287:       // Merge with any previous or following adjustment instruction. Note: the
 288:       // instructions merged with here do not have CFI, so their stack
 289:       // adjustments do not feed into CfaAdjustment.
 290:       StackAdjustment += mergeSPUpdates(MBB, I, true);
 291:       StackAdjustment += mergeSPUpdates(MBB, I, false);
 292: 
 293:       if (StackAdjustment) {
 294:         BuildStackAdjustment(MBB, I, DL, StackAdjustment, false);
 295:       }
 296:     }
 297: 
 298:     if (DwarfCFI && !hasFP(MF)) {
 299:       // If we don't have FP, but need to generate unwind information,
 300:       // we need to set the correct CFA offset after the stack adjustment.
 301:       // How much we adjust the CFA offset depends on whether we're emitting
 302:       // CFI only for EH purposes or for debugging. EH only requires the CFA
 303:       // offset to be correct at each call site, while for debugging we want
 304:       // it to be more precise.
 305: 
 306:       // TODO: When not using precise CFA, we also need to adjust for the
 307:       // InternalAmt here.
 308:       if (CfaAdjustment) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 309-336 / 第 309-336 行
```cpp
 309:         BuildCFI(
 310:             MBB, I, DL,
 311:             MCCFIInstruction::createAdjustCfaOffset(nullptr, CfaAdjustment));
 312:       }
 313:     }
 314: 
 315:     return I;
 316:   }
 317: 
 318:   if (IsDestroy && InternalAmt) {
 319:     // If we are performing frame pointer elimination and if the callee pops
 320:     // something off the stack pointer, add it back.  We do this until we have
 321:     // more advanced stack pointer tracking ability.
 322:     // We are not tracking the stack pointer adjustment by the callee, so make
 323:     // sure we restore the stack pointer immediately after the call, there may
 324:     // be spill code inserted between the CALL and ADJCALLSTACKUP instructions.
 325:     MachineBasicBlock::iterator CI = I;
 326:     MachineBasicBlock::iterator B = MBB.begin();
 327:     while (CI != B && !std::prev(CI)->isCall())
 328:       --CI;
 329:     BuildStackAdjustment(MBB, CI, DL, -InternalAmt, /*InEpilogue=*/false);
 330:   }
 331: 
 332:   return I;
 333: }
 334: 
 335: /// Emit a series of instructions to increment / decrement the stack pointer by
 336: /// a constant value.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 337-364 / 第 337-364 行
```cpp
 337: void M68kFrameLowering::emitSPUpdate(MachineBasicBlock &MBB,
 338:                                      MachineBasicBlock::iterator &MBBI,
 339:                                      int64_t NumBytes, bool InEpilogue) const {
 340:   bool IsSub = NumBytes < 0;
 341:   uint64_t Offset = IsSub ? -NumBytes : NumBytes;
 342: 
 343:   uint64_t Chunk = (1LL << 31) - 1;
 344:   DebugLoc DL = MBB.findDebugLoc(MBBI);
 345: 
 346:   while (Offset) {
 347:     if (Offset > Chunk) {
 348:       // Rather than emit a long series of instructions for large offsets,
 349:       // load the offset into a register and do one sub/add
 350:       Register Reg;
 351: 
 352:       if (IsSub && !isRegLiveIn(MBB, M68k::D0))
 353:         Reg = M68k::D0;
 354:       else
 355:         Reg = findDeadCallerSavedReg(MBB, MBBI, TRI);
 356: 
 357:       if (Reg) {
 358:         unsigned Opc = M68k::MOV32ri;
 359:         BuildMI(MBB, MBBI, DL, TII.get(Opc), Reg).addImm(Offset);
 360:         Opc = IsSub ? M68k::SUB32ar : M68k::ADD32ar;
 361:         MachineInstr *MI = BuildMI(MBB, MBBI, DL, TII.get(Opc), StackPtr)
 362:                                .addReg(StackPtr)
 363:                                .addReg(Reg);
 364:         // ??? still no CCR
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::emitSPUpdate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::emitSPUpdate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 365-392 / 第 365-392 行
```cpp
 365:         MI->getOperand(3).setIsDead(); // The CCR implicit def is dead.
 366:         Offset = 0;
 367:         continue;
 368:       }
 369:     }
 370: 
 371:     uint64_t ThisVal = std::min(Offset, Chunk);
 372: 
 373:     MachineInstrBuilder MI = BuildStackAdjustment(
 374:         MBB, MBBI, DL, IsSub ? -ThisVal : ThisVal, InEpilogue);
 375:     if (IsSub)
 376:       MI.setMIFlag(MachineInstr::FrameSetup);
 377:     else
 378:       MI.setMIFlag(MachineInstr::FrameDestroy);
 379: 
 380:     Offset -= ThisVal;
 381:   }
 382: }
 383: 
 384: int M68kFrameLowering::mergeSPUpdates(MachineBasicBlock &MBB,
 385:                                       MachineBasicBlock::iterator &MBBI,
 386:                                       bool MergeWithPrevious) const {
 387:   if ((MergeWithPrevious && MBBI == MBB.begin()) ||
 388:       (!MergeWithPrevious && MBBI == MBB.end()))
 389:     return 0;
 390: 
 391:   MachineBasicBlock::iterator PI = MergeWithPrevious ? std::prev(MBBI) : MBBI;
 392:   MachineBasicBlock::iterator NI =
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::mergeSPUpdates`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::mergeSPUpdates` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```cpp
 393:       MergeWithPrevious ? nullptr : std::next(MBBI);
 394:   unsigned Opc = PI->getOpcode();
 395:   int Offset = 0;
 396: 
 397:   if (!MergeWithPrevious && NI != MBB.end() &&
 398:       NI->getOpcode() == TargetOpcode::CFI_INSTRUCTION) {
 399:     // Don't merge with the next instruction if it has CFI.
 400:     return Offset;
 401:   }
 402: 
 403:   if (Opc == M68k::ADD32ai && PI->getOperand(0).getReg() == StackPtr) {
 404:     assert(PI->getOperand(1).getReg() == StackPtr);
 405:     Offset += PI->getOperand(2).getImm();
 406:     MBB.erase(PI);
 407:     if (!MergeWithPrevious)
 408:       MBBI = NI;
 409:   } else if (Opc == M68k::SUB32ai && PI->getOperand(0).getReg() == StackPtr) {
 410:     assert(PI->getOperand(1).getReg() == StackPtr);
 411:     Offset -= PI->getOperand(2).getImm();
 412:     MBB.erase(PI);
 413:     if (!MergeWithPrevious)
 414:       MBBI = NI;
 415:   }
 416: 
 417:   return Offset;
 418: }
 419: 
 420: MachineInstrBuilder M68kFrameLowering::BuildStackAdjustment(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 421-448 / 第 421-448 行
```cpp
 421:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 422:     const DebugLoc &DL, int64_t Offset, bool InEpilogue) const {
 423:   assert(Offset != 0 && "zero offset stack adjustment requested");
 424: 
 425:   // TODO can `lea` be used to adjust stack?
 426: 
 427:   bool IsSub = Offset < 0;
 428:   uint64_t AbsOffset = IsSub ? -Offset : Offset;
 429:   unsigned Opc = IsSub ? M68k::SUB32ai : M68k::ADD32ai;
 430: 
 431:   MachineInstrBuilder MI = BuildMI(MBB, MBBI, DL, TII.get(Opc), StackPtr)
 432:                                .addReg(StackPtr)
 433:                                .addImm(AbsOffset);
 434:   // FIXME Update CCR as well. For now we just
 435:   // conservatively say CCR implicit def is dead
 436:   MI->getOperand(3).setIsDead();
 437:   return MI;
 438: }
 439: 
 440: void M68kFrameLowering::BuildCFI(MachineBasicBlock &MBB,
 441:                                  MachineBasicBlock::iterator MBBI,
 442:                                  const DebugLoc &DL,
 443:                                  const MCCFIInstruction &CFIInst) const {
 444:   MachineFunction &MF = *MBB.getParent();
 445:   unsigned CFIIndex = MF.addFrameInst(CFIInst);
 446:   BuildMI(MBB, MBBI, DL, TII.get(TargetOpcode::CFI_INSTRUCTION))
 447:       .addCFIIndex(CFIIndex);
 448: }
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::BuildCFI`, `BuildMI`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::BuildCFI`, `BuildMI` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 449-476 / 第 449-476 行
```cpp
 449: 
 450: void M68kFrameLowering::emitPrologueCalleeSavedFrameMoves(
 451:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 452:     const DebugLoc &DL) const {
 453:   MachineFunction &MF = *MBB.getParent();
 454:   MachineFrameInfo &MFI = MF.getFrameInfo();
 455:   const MCRegisterInfo *MRI = MF.getContext().getRegisterInfo();
 456: 
 457:   // Add callee saved registers to move list.
 458:   const auto &CSI = MFI.getCalleeSavedInfo();
 459:   if (CSI.empty())
 460:     return;
 461: 
 462:   // Calculate offsets.
 463:   for (const auto &I : CSI) {
 464:     int64_t Offset = MFI.getObjectOffset(I.getFrameIdx());
 465:     MCRegister Reg = I.getReg();
 466: 
 467:     unsigned DwarfReg = MRI->getDwarfRegNum(Reg, true);
 468:     BuildCFI(MBB, MBBI, DL,
 469:              MCCFIInstruction::createOffset(nullptr, DwarfReg, Offset));
 470:   }
 471: }
 472: 
 473: void M68kFrameLowering::emitPrologue(MachineFunction &MF,
 474:                                      MachineBasicBlock &MBB) const {
 475:   assert(&STI == &MF.getSubtarget<M68kSubtarget>() &&
 476:          "MF used frame lowering for wrong subtarget");
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::emitPrologueCalleeSavedFrameMoves`, `M68kFrameLowering::emitPrologue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::emitPrologueCalleeSavedFrameMoves`, `M68kFrameLowering::emitPrologue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 477-504 / 第 477-504 行
```cpp
 477: 
 478:   MachineBasicBlock::iterator MBBI = MBB.begin();
 479:   MachineFrameInfo &MFI = MF.getFrameInfo();
 480:   M68kMachineFunctionInfo *MMFI = MF.getInfo<M68kMachineFunctionInfo>();
 481:   uint64_t MaxAlign = calculateMaxStackAlign(MF); // Desired stack alignment.
 482:   uint64_t StackSize = MFI.getStackSize(); // Number of bytes to allocate.
 483:   bool HasFP = hasFP(MF);
 484:   bool NeedsDwarfCFI = MF.needsFrameMoves();
 485:   Register FramePtr = TRI->getFrameRegister(MF);
 486:   const unsigned MachineFramePtr = FramePtr;
 487:   unsigned BasePtr = TRI->getBaseRegister();
 488: 
 489:   // Debug location must be unknown since the first debug location is used
 490:   // to determine the end of the prologue.
 491:   DebugLoc DL;
 492: 
 493:   // Add RETADDR move area to callee saved frame size.
 494:   int TailCallReturnAddrDelta = MMFI->getTCReturnAddrDelta();
 495: 
 496:   if (TailCallReturnAddrDelta < 0) {
 497:     MMFI->setCalleeSavedFrameSize(MMFI->getCalleeSavedFrameSize() -
 498:                                   TailCallReturnAddrDelta);
 499:   }
 500: 
 501:   // Insert stack pointer adjustment for later moving of return addr.  Only
 502:   // applies to tail call optimized functions where the callee argument stack
 503:   // size is bigger than the callers.
 504:   if (TailCallReturnAddrDelta < 0) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-532 / 第 505-532 行
```cpp
 505:     BuildStackAdjustment(MBB, MBBI, DL, TailCallReturnAddrDelta,
 506:                          /*InEpilogue=*/false)
 507:         .setMIFlag(MachineInstr::FrameSetup);
 508:   }
 509: 
 510:   // Mapping for machine moves:
 511:   //
 512:   //   DST: VirtualFP AND
 513:   //        SRC: VirtualFP              => DW_CFA_def_cfa_offset
 514:   //        ELSE                        => DW_CFA_def_cfa
 515:   //
 516:   //   SRC: VirtualFP AND
 517:   //        DST: Register               => DW_CFA_def_cfa_register
 518:   //
 519:   //   ELSE
 520:   //        OFFSET < 0                  => DW_CFA_offset_extended_sf
 521:   //        REG < 64                    => DW_CFA_offset + Reg
 522:   //        ELSE                        => DW_CFA_offset_extended
 523: 
 524:   uint64_t NumBytes = 0;
 525:   int stackGrowth = -SlotSize;
 526: 
 527:   if (HasFP) {
 528:     // Calculate required stack adjustment.
 529:     uint64_t FrameSize = StackSize - SlotSize;
 530:     // If required, include space for extra hidden slot for stashing base
 531:     // pointer.
 532:     if (MMFI->getRestoreBasePointer())
```
- **EN**: The range implements or declares functions including `BuildStackAdjustment`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildStackAdjustment` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 533-560 / 第 533-560 行
```cpp
 533:       FrameSize += SlotSize;
 534: 
 535:     NumBytes = FrameSize - MMFI->getCalleeSavedFrameSize();
 536: 
 537:     // Callee-saved registers are pushed on stack before the stack is realigned.
 538:     if (TRI->hasStackRealignment(MF))
 539:       NumBytes = alignTo(NumBytes, MaxAlign);
 540: 
 541:     // Get the offset of the stack slot for the FP register, which is
 542:     // guaranteed to be the last slot by processFunctionBeforeFrameFinalized.
 543:     // Update the frame offset adjustment.
 544:     MFI.setOffsetAdjustment(-NumBytes);
 545: 
 546:     BuildMI(MBB, MBBI, DL, TII.get(M68k::LINK16))
 547:         .addReg(M68k::WA6, RegState::Kill)
 548:         .addImm(-NumBytes)
 549:         .setMIFlag(MachineInstr::FrameSetup);
 550: 
 551:     if (NeedsDwarfCFI) {
 552:       // Mark the place where FP was saved.
 553:       // Define the current CFA rule to use the provided offset.
 554:       assert(StackSize);
 555:       BuildCFI(MBB, MBBI, DL,
 556:                MCCFIInstruction::cfiDefCfaOffset(nullptr, 2 * stackGrowth));
 557: 
 558:       // Change the rule for the FramePtr to be an "offset" rule.
 559:       int DwarfFramePtr = TRI->getDwarfRegNum(MachineFramePtr, true);
 560:       assert(DwarfFramePtr > 0);
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-588 / 第 561-588 行
```cpp
 561:       BuildCFI(MBB, MBBI, DL,
 562:                MCCFIInstruction::createOffset(nullptr, DwarfFramePtr,
 563:                                               2 * stackGrowth));
 564:     }
 565: 
 566:     if (NeedsDwarfCFI) {
 567:       // Mark effective beginning of when frame pointer becomes valid.
 568:       // Define the current CFA to use the FP register.
 569:       unsigned DwarfFramePtr = TRI->getDwarfRegNum(MachineFramePtr, true);
 570:       BuildCFI(MBB, MBBI, DL,
 571:                MCCFIInstruction::createDefCfaRegister(nullptr, DwarfFramePtr));
 572:     }
 573: 
 574:     // Mark the FramePtr as live-in in every block. Don't do this again for
 575:     // funclet prologues.
 576:     for (MachineBasicBlock &EveryMBB : MF)
 577:       EveryMBB.addLiveIn(MachineFramePtr);
 578:   } else {
 579:     NumBytes = StackSize - MMFI->getCalleeSavedFrameSize();
 580:   }
 581: 
 582:   // Skip the callee-saved push instructions.
 583:   bool PushedRegs = false;
 584:   int StackOffset = 2 * stackGrowth;
 585: 
 586:   while (MBBI != MBB.end() && MBBI->getFlag(MachineInstr::FrameSetup) &&
 587:          MBBI->getOpcode() == M68k::PUSH32r) {
 588:     PushedRegs = true;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 589-616 / 第 589-616 行
```cpp
 589:     ++MBBI;
 590: 
 591:     if (!HasFP && NeedsDwarfCFI) {
 592:       // Mark callee-saved push instruction.
 593:       // Define the current CFA rule to use the provided offset.
 594:       assert(StackSize);
 595:       BuildCFI(MBB, MBBI, DL,
 596:                MCCFIInstruction::cfiDefCfaOffset(nullptr, StackOffset));
 597:       StackOffset += stackGrowth;
 598:     }
 599:   }
 600: 
 601:   // Realign stack after we pushed callee-saved registers (so that we'll be
 602:   // able to calculate their offsets from the frame pointer).
 603:   if (TRI->hasStackRealignment(MF)) {
 604:     assert(HasFP && "There should be a frame pointer if stack is realigned.");
 605:     BuildStackAlignAND(MBB, MBBI, DL, StackPtr, MaxAlign);
 606:   }
 607: 
 608:   // If there is an SUB32ri of SP immediately before this instruction, merge
 609:   // the two. This can be the case when tail call elimination is enabled and
 610:   // the callee has more arguments then the caller.
 611:   NumBytes -= mergeSPUpdates(MBB, MBBI, true);
 612: 
 613:   // Adjust stack pointer: ESP -= numbytes.
 614:   if (!HasFP)
 615:     emitSPUpdate(MBB, MBBI, -(int64_t)NumBytes, /*InEpilogue=*/false);
 616: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 617-644 / 第 617-644 行
```cpp
 617:   unsigned SPOrEstablisher = StackPtr;
 618: 
 619:   // If we need a base pointer, set it up here. It's whatever the value
 620:   // of the stack pointer is at this point. Any variable size objects
 621:   // will be allocated after this, so we can still use the base pointer
 622:   // to reference locals.
 623:   if (TRI->hasBasePointer(MF)) {
 624:     // Update the base pointer with the current stack pointer.
 625:     BuildMI(MBB, MBBI, DL, TII.get(M68k::MOV32aa), BasePtr)
 626:         .addReg(SPOrEstablisher)
 627:         .setMIFlag(MachineInstr::FrameSetup);
 628:     if (MMFI->getRestoreBasePointer()) {
 629:       // Stash value of base pointer.  Saving SP instead of FP shortens
 630:       // dependence chain. Used by SjLj EH.
 631:       unsigned Opm = M68k::MOV32ja;
 632:       M68k::addRegIndirectWithDisp(BuildMI(MBB, MBBI, DL, TII.get(Opm)),
 633:                                    FramePtr, true,
 634:                                    MMFI->getRestoreBasePointerOffset())
 635:           .addReg(SPOrEstablisher)
 636:           .setMIFlag(MachineInstr::FrameSetup);
 637:     }
 638:   }
 639: 
 640:   if (((!HasFP && NumBytes) || PushedRegs) && NeedsDwarfCFI) {
 641:     // Mark end of stack pointer adjustment.
 642:     if (!HasFP && NumBytes) {
 643:       // Define the current CFA rule to use the provided offset.
 644:       assert(StackSize);
```
- **EN**: The range implements or declares functions including `M68k::addRegIndirectWithDisp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68k::addRegIndirectWithDisp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 645-672 / 第 645-672 行
```cpp
 645:       BuildCFI(
 646:           MBB, MBBI, DL,
 647:           MCCFIInstruction::cfiDefCfaOffset(nullptr, -StackSize + stackGrowth));
 648:     }
 649: 
 650:     // Emit DWARF info specifying the offsets of the callee-saved registers.
 651:     if (PushedRegs)
 652:       emitPrologueCalleeSavedFrameMoves(MBB, MBBI, DL);
 653:   }
 654: 
 655:   // TODO Interrupt handlers
 656:   // M68k Interrupt handling function cannot assume anything about the
 657:   // direction flag (DF in CCR register). Clear this flag by creating "cld"
 658:   // instruction in each prologue of interrupt handler function. The "cld"
 659:   // instruction should only in these cases:
 660:   // 1. The interrupt handling function uses any of the "rep" instructions.
 661:   // 2. Interrupt handling function calls another function.
 662: }
 663: 
 664: static bool isTailCallOpcode(unsigned Opc) {
 665:   return Opc == M68k::TCRETURNj || Opc == M68k::TCRETURNq;
 666: }
 667: 
 668: void M68kFrameLowering::emitEpilogue(MachineFunction &MF,
 669:                                      MachineBasicBlock &MBB) const {
 670:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 671:   M68kMachineFunctionInfo *MMFI = MF.getInfo<M68kMachineFunctionInfo>();
 672:   MachineBasicBlock::iterator MBBI = MBB.getFirstTerminator();
```
- **EN**: The range implements or declares functions including `isTailCallOpcode`, `M68kFrameLowering::emitEpilogue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isTailCallOpcode`, `M68kFrameLowering::emitEpilogue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 673-700 / 第 673-700 行
```cpp
 673:   std::optional<unsigned> RetOpcode;
 674:   if (MBBI != MBB.end())
 675:     RetOpcode = MBBI->getOpcode();
 676:   DebugLoc DL;
 677:   if (MBBI != MBB.end())
 678:     DL = MBBI->getDebugLoc();
 679:   Register FramePtr = TRI->getFrameRegister(MF);
 680:   unsigned MachineFramePtr = FramePtr;
 681: 
 682:   // Get the number of bytes to allocate from the FrameInfo.
 683:   uint64_t StackSize = MFI.getStackSize();
 684:   uint64_t MaxAlign = calculateMaxStackAlign(MF);
 685:   unsigned CSSize = MMFI->getCalleeSavedFrameSize();
 686:   uint64_t NumBytes = 0;
 687: 
 688:   if (hasFP(MF)) {
 689:     // Calculate required stack adjustment.
 690:     uint64_t FrameSize = StackSize - SlotSize;
 691:     NumBytes = FrameSize - CSSize;
 692: 
 693:     // Callee-saved registers were pushed on stack before the stack was
 694:     // realigned.
 695:     if (TRI->hasStackRealignment(MF))
 696:       NumBytes = alignTo(FrameSize, MaxAlign);
 697: 
 698:   } else {
 699:     NumBytes = StackSize - CSSize;
 700:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 701-728 / 第 701-728 行
```cpp
 701: 
 702:   // Skip the callee-saved pop instructions.
 703:   while (MBBI != MBB.begin()) {
 704:     MachineBasicBlock::iterator PI = std::prev(MBBI);
 705:     unsigned Opc = PI->getOpcode();
 706: 
 707:     if ((Opc != M68k::POP32r || !PI->getFlag(MachineInstr::FrameDestroy)) &&
 708:         Opc != M68k::DBG_VALUE && !PI->isTerminator())
 709:       break;
 710: 
 711:     --MBBI;
 712:   }
 713:   MachineBasicBlock::iterator FirstCSPop = MBBI;
 714: 
 715:   if (MBBI != MBB.end())
 716:     DL = MBBI->getDebugLoc();
 717: 
 718:   // If there is an ADD32ri or SUB32ri of SP immediately before this
 719:   // instruction, merge the two instructions.
 720:   if (NumBytes || MFI.hasVarSizedObjects())
 721:     NumBytes += mergeSPUpdates(MBB, MBBI, true);
 722: 
 723:   // If dynamic alloca is used, then reset SP to point to the last callee-saved
 724:   // slot before popping them off! Same applies for the case, when stack was
 725:   // realigned. Don't do this if this was a funclet epilogue, since the funclets
 726:   // will not do realignment or dynamic stack allocation.
 727:   if ((TRI->hasStackRealignment(MF) || MFI.hasVarSizedObjects())) {
 728:     if (TRI->hasStackRealignment(MF))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 729-756 / 第 729-756 行
```cpp
 729:       MBBI = FirstCSPop;
 730:     uint64_t LEAAmount = -CSSize;
 731: 
 732:     // 'move %FramePtr, SP' will not be recognized as an epilogue sequence.
 733:     // However, we may use this sequence if we have a frame pointer because the
 734:     // effects of the prologue can safely be undone.
 735:     if (LEAAmount != 0) {
 736:       unsigned Opc = M68k::LEA32p;
 737:       M68k::addRegIndirectWithDisp(
 738:           BuildMI(MBB, MBBI, DL, TII.get(Opc), StackPtr), FramePtr, false,
 739:           LEAAmount);
 740:       --MBBI;
 741:     } else {
 742:       BuildMI(MBB, MBBI, DL, TII.get(M68k::UNLK))
 743:           .addReg(MachineFramePtr, RegState::Kill)
 744:           .setMIFlag(MachineInstr::FrameDestroy);
 745:       --MBBI;
 746:     }
 747:   } else if (hasFP(MF)) {
 748:     BuildMI(MBB, MBBI, DL, TII.get(M68k::UNLK))
 749:         .addReg(MachineFramePtr, RegState::Kill)
 750:         .setMIFlag(MachineInstr::FrameDestroy);
 751:   } else if (NumBytes) {
 752:     // Adjust stack pointer back: SP += numbytes.
 753:     emitSPUpdate(MBB, MBBI, NumBytes, /*InEpilogue=*/true);
 754:     --MBBI;
 755:   }
 756: 
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 757-784 / 第 757-784 行
```cpp
 757:   if (!RetOpcode || !isTailCallOpcode(*RetOpcode)) {
 758:     // Add the return addr area delta back since we are not tail calling.
 759:     int Offset = -1 * MMFI->getTCReturnAddrDelta();
 760:     assert(Offset >= 0 && "TCDelta should never be positive");
 761:     if (Offset) {
 762:       MBBI = MBB.getFirstTerminator();
 763: 
 764:       // Check for possible merge with preceding ADD instruction.
 765:       Offset += mergeSPUpdates(MBB, MBBI, true);
 766:       emitSPUpdate(MBB, MBBI, Offset, /*InEpilogue=*/true);
 767:     }
 768:   }
 769: }
 770: 
 771: void M68kFrameLowering::determineCalleeSaves(MachineFunction &MF,
 772:                                              BitVector &SavedRegs,
 773:                                              RegScavenger *RS) const {
 774:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
 775: 
 776:   MachineFrameInfo &MFI = MF.getFrameInfo();
 777: 
 778:   M68kMachineFunctionInfo *M68kFI = MF.getInfo<M68kMachineFunctionInfo>();
 779:   int64_t TailCallReturnAddrDelta = M68kFI->getTCReturnAddrDelta();
 780: 
 781:   if (TailCallReturnAddrDelta < 0) {
 782:     // create RETURNADDR area
 783:     //   arg
 784:     //   arg
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::determineCalleeSaves`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::determineCalleeSaves` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 785-812 / 第 785-812 行
```cpp
 785:     //   RETADDR
 786:     //   { ...
 787:     //     RETADDR area
 788:     //     ...
 789:     //   }
 790:     //   [FP]
 791:     MFI.CreateFixedObject(-TailCallReturnAddrDelta,
 792:                           TailCallReturnAddrDelta - SlotSize, true);
 793:   }
 794: 
 795:   // Spill the BasePtr if it's used.
 796:   if (TRI->hasBasePointer(MF)) {
 797:     SavedRegs.set(TRI->getBaseRegister());
 798:   }
 799: }
 800: 
 801: bool M68kFrameLowering::assignCalleeSavedSpillSlots(
 802:     MachineFunction &MF, const TargetRegisterInfo *TRI,
 803:     std::vector<CalleeSavedInfo> &CSI) const {
 804:   MachineFrameInfo &MFI = MF.getFrameInfo();
 805:   M68kMachineFunctionInfo *M68kFI = MF.getInfo<M68kMachineFunctionInfo>();
 806: 
 807:   int SpillSlotOffset = getOffsetOfLocalArea() + M68kFI->getTCReturnAddrDelta();
 808: 
 809:   if (hasFP(MF)) {
 810:     // emitPrologue always spills frame register the first thing.
 811:     SpillSlotOffset -= SlotSize;
 812:     MFI.CreateFixedSpillStackObject(SlotSize, SpillSlotOffset);
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::assignCalleeSavedSpillSlots`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::assignCalleeSavedSpillSlots` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 813-840 / 第 813-840 行
```cpp
 813: 
 814:     // Since emitPrologue and emitEpilogue will handle spilling and restoring of
 815:     // the frame register, we can delete it from CSI list and not have to worry
 816:     // about avoiding it later.
 817:     Register FPReg = TRI->getFrameRegister(MF);
 818:     for (unsigned i = 0, e = CSI.size(); i < e; ++i) {
 819:       if (TRI->regsOverlap(CSI[i].getReg(), FPReg)) {
 820:         CSI.erase(CSI.begin() + i);
 821:         break;
 822:       }
 823:     }
 824:   }
 825: 
 826:   // The rest is fine
 827:   return false;
 828: }
 829: 
 830: bool M68kFrameLowering::spillCalleeSavedRegisters(
 831:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 832:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 833:   auto &MRI = *static_cast<const M68kRegisterInfo *>(TRI);
 834:   auto DL = MBB.findDebugLoc(MI);
 835: 
 836:   int FI = 0;
 837:   unsigned Mask = 0;
 838:   for (const auto &Info : CSI) {
 839:     FI = std::max(FI, Info.getFrameIdx());
 840:     MCRegister Reg = Info.getReg();
```
- **EN**: The range implements or declares functions including `M68kFrameLowering::spillCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kFrameLowering::spillCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 841-868 / 第 841-868 行
```cpp
 841:     unsigned Shift = MRI.getSpillRegisterOrder(Reg);
 842:     Mask |= 1 << Shift;
 843:   }
 844: 
 845:   auto I =
 846:       M68k::addFrameReference(BuildMI(MBB, MI, DL, TII.get(M68k::MOVM32pm)), FI)
 847:           .addImm(Mask)
 848:           .setMIFlag(MachineInstr::FrameSetup);
 849: 
 850:   // Append implicit registers and mem locations
 851:   const MachineFunction &MF = *MBB.getParent();
 852:   const MachineRegisterInfo &RI = MF.getRegInfo();
 853:   for (const auto &Info : CSI) {
 854:     MCRegister Reg = Info.getReg();
 855:     bool IsLiveIn = RI.isLiveIn(Reg);
 856:     if (!IsLiveIn)
 857:       MBB.addLiveIn(Reg);
 858:     I.addReg(Reg, IsLiveIn ? RegState::Implicit : RegState::ImplicitKill);
 859:     M68k::addMemOperand(I, Info.getFrameIdx(), 0);
 860:   }
 861: 
 862:   return true;
 863: }
 864: 
 865: bool M68kFrameLowering::restoreCalleeSavedRegisters(
 866:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 867:     MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 868:   auto &MRI = *static_cast<const M68kRegisterInfo *>(TRI);
```
- **EN**: The range implements or declares functions including `M68k::addFrameReference`, `M68kFrameLowering::restoreCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68k::addFrameReference`, `M68kFrameLowering::restoreCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 869-891 / 第 869-891 行
```cpp
 869:   auto DL = MBB.findDebugLoc(MI);
 870: 
 871:   int FI = 0;
 872:   unsigned Mask = 0;
 873:   for (const auto &Info : CSI) {
 874:     FI = std::max(FI, Info.getFrameIdx());
 875:     MCRegister Reg = Info.getReg();
 876:     unsigned Shift = MRI.getSpillRegisterOrder(Reg);
 877:     Mask |= 1 << Shift;
 878:   }
 879: 
 880:   auto I = M68k::addFrameReference(
 881:                BuildMI(MBB, MI, DL, TII.get(M68k::MOVM32mp)).addImm(Mask), FI)
 882:                .setMIFlag(MachineInstr::FrameDestroy);
 883: 
 884:   // Append implicit registers and mem locations
 885:   for (const auto &Info : CSI) {
 886:     I.addReg(Info.getReg(), RegState::ImplicitDefine);
 887:     M68k::addMemOperand(I, Info.getFrameIdx(), 0);
 888:   }
 889: 
 890:   return true;
 891: }
```
- **EN**: The range implements or declares functions including `BuildMI`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Frame lowering**: Builds stack frames, callee-save handling, and prologue/epilogue sequences. / 构建栈帧、被调用者保存寄存器处理以及序言/尾声序列。

## Dependencies / 依赖关系
- `M68kFrameLowering.h`
- `M68kInstrBuilder.h`
- `M68kInstrInfo.h`
- `M68kMachineFunction.h`
- `M68kSubtarget.h`
- `llvm/ADT/SmallSet.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineModuleInfo.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/IR/DataLayout.h`
- `llvm/IR/Function.h`
- `llvm/Support/Alignment.h`
- `llvm/Support/CommandLine.h`
- `llvm/Target/TargetMachine.h`
- `...` (1 more include dependencies omitted for brevity / 其余 1 个 include 依赖已省略)
