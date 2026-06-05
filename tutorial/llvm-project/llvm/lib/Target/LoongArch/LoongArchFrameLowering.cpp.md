# LoongArchFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchFrameLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements stack frame lowering and prologue/epilogue logic for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责实现栈帧降低以及序言/尾声逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- LoongArchFrameLowering.cpp - LoongArch Frame Information -*- C++ -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the LoongArch implementation of TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchFrameLowering.h"
  14: #include "LoongArchMachineFunctionInfo.h"
  15: #include "LoongArchSubtarget.h"
  16: #include "MCTargetDesc/LoongArchBaseInfo.h"
  17: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  18: #include "llvm/CodeGen/MachineFrameInfo.h"
  19: #include "llvm/CodeGen/MachineFunction.h"
  20: #include "llvm/CodeGen/MachineInstrBuilder.h"
  21: #include "llvm/CodeGen/MachineRegisterInfo.h"
  22: #include "llvm/CodeGen/RegisterScavenging.h"
  23: #include "llvm/IR/DiagnosticInfo.h"
  24: #include "llvm/MC/MCDwarf.h"
  25: 
  26: using namespace llvm;
  27: 
  28: #define DEBUG_TYPE "loongarch-frame-lowering"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchFrameLowering.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchSubtarget.h`, `LoongArchBaseInfo.h`, `LoongArchMCTargetDesc.h`, `MachineFrameInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchFrameLowering.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchSubtarget.h`, `LoongArchBaseInfo.h`, `LoongArchMCTargetDesc.h`, `MachineFrameInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29: 
  30: // Return true if the specified function should have a dedicated frame
  31: // pointer register.  This is true if frame pointer elimination is
  32: // disabled, if it needs dynamic stack realignment, if the function has
  33: // variable sized allocas, or if the frame address is taken.
  34: bool LoongArchFrameLowering::hasFPImpl(const MachineFunction &MF) const {
  35:   const TargetRegisterInfo *RegInfo = MF.getSubtarget().getRegisterInfo();
  36: 
  37:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  38:   return MF.getTarget().Options.DisableFramePointerElim(MF) ||
  39:          RegInfo->hasStackRealignment(MF) || MFI.hasVarSizedObjects() ||
  40:          MFI.isFrameAddressTaken();
  41: }
  42: 
  43: bool LoongArchFrameLowering::hasBP(const MachineFunction &MF) const {
  44:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  45:   const TargetRegisterInfo *TRI = STI.getRegisterInfo();
  46: 
  47:   return MFI.hasVarSizedObjects() && TRI->hasStackRealignment(MF);
  48: }
  49: 
  50: void LoongArchFrameLowering::adjustReg(MachineBasicBlock &MBB,
  51:                                        MachineBasicBlock::iterator MBBI,
  52:                                        const DebugLoc &DL, Register DestReg,
  53:                                        Register SrcReg, int64_t Val,
  54:                                        MachineInstr::MIFlag Flag) const {
  55:   const LoongArchInstrInfo *TII = STI.getInstrInfo();
  56:   bool IsLA64 = STI.is64Bit();
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::hasFPImpl`, `LoongArchFrameLowering::hasBP`, `LoongArchFrameLowering::adjustReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::hasFPImpl`, `LoongArchFrameLowering::hasBP`, `LoongArchFrameLowering::adjustReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57:   unsigned Addi = IsLA64 ? LoongArch::ADDI_D : LoongArch::ADDI_W;
  58: 
  59:   if (DestReg == SrcReg && Val == 0)
  60:     return;
  61: 
  62:   if (isInt<12>(Val)) {
  63:     // addi.w/d $DstReg, $SrcReg, Val
  64:     BuildMI(MBB, MBBI, DL, TII->get(Addi), DestReg)
  65:         .addReg(SrcReg)
  66:         .addImm(Val)
  67:         .setMIFlag(Flag);
  68:     return;
  69:   }
  70: 
  71:   // Try to split the offset across two ADDIs. We need to keep the stack pointer
  72:   // aligned after each ADDI. We need to determine the maximum value we can put
  73:   // in each ADDI. In the negative direction, we can use -2048 which is always
  74:   // sufficiently aligned. In the positive direction, we need to find the
  75:   // largest 12-bit immediate that is aligned. Exclude -4096 since it can be
  76:   // created with LU12I.W.
  77:   assert(getStackAlign().value() < 2048 && "Stack alignment too large");
  78:   int64_t MaxPosAdjStep = 2048 - getStackAlign().value();
  79:   if (Val > -4096 && Val <= (2 * MaxPosAdjStep)) {
  80:     int64_t FirstAdj = Val < 0 ? -2048 : MaxPosAdjStep;
  81:     Val -= FirstAdj;
  82:     BuildMI(MBB, MBBI, DL, TII->get(Addi), DestReg)
  83:         .addReg(SrcReg)
  84:         .addImm(FirstAdj)
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 85-112 / 第 85-112 行
```cpp
  85:         .setMIFlag(Flag);
  86:     BuildMI(MBB, MBBI, DL, TII->get(Addi), DestReg)
  87:         .addReg(DestReg, RegState::Kill)
  88:         .addImm(Val)
  89:         .setMIFlag(Flag);
  90:     return;
  91:   }
  92: 
  93:   unsigned Opc = IsLA64 ? LoongArch::ADD_D : LoongArch::ADD_W;
  94:   if (Val < 0) {
  95:     Val = -Val;
  96:     Opc = IsLA64 ? LoongArch::SUB_D : LoongArch::SUB_W;
  97:   }
  98: 
  99:   MachineRegisterInfo &MRI = MBB.getParent()->getRegInfo();
 100:   Register ScratchReg = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
 101:   TII->movImm(MBB, MBBI, DL, ScratchReg, Val, Flag);
 102:   BuildMI(MBB, MBBI, DL, TII->get(Opc), DestReg)
 103:       .addReg(SrcReg)
 104:       .addReg(ScratchReg, RegState::Kill)
 105:       .setMIFlag(Flag);
 106: }
 107: 
 108: // Determine the size of the frame and maximum call frame size.
 109: void LoongArchFrameLowering::determineFrameLayout(MachineFunction &MF) const {
 110:   MachineFrameInfo &MFI = MF.getFrameInfo();
 111: 
 112:   // Get the number of bytes to allocate from the FrameInfo.
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `LoongArchFrameLowering::determineFrameLayout`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `LoongArchFrameLowering::determineFrameLayout` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:   uint64_t FrameSize = MFI.getStackSize();
 114: 
 115:   // Make sure the frame is aligned.
 116:   FrameSize = alignTo(FrameSize, getStackAlign());
 117: 
 118:   // Update frame info.
 119:   MFI.setStackSize(FrameSize);
 120: }
 121: 
 122: static uint64_t estimateFunctionSizeInBytes(const LoongArchInstrInfo *TII,
 123:                                             const MachineFunction &MF) {
 124:   uint64_t FuncSize = 0;
 125:   for (auto &MBB : MF)
 126:     for (auto &MI : MBB)
 127:       FuncSize += TII->getInstSizeInBytes(MI);
 128:   return FuncSize;
 129: }
 130: 
 131: static bool needScavSlotForCFR(MachineFunction &MF) {
 132:   if (!MF.getSubtarget<LoongArchSubtarget>().hasBasicF())
 133:     return false;
 134:   for (auto &MBB : MF)
 135:     for (auto &MI : MBB)
 136:       if (MI.getOpcode() == LoongArch::PseudoST_CFR)
 137:         return true;
 138:   return false;
 139: }
 140: 
```
- **EN**: The range implements or declares functions including `estimateFunctionSizeInBytes`, `needScavSlotForCFR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `estimateFunctionSizeInBytes`, `needScavSlotForCFR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 141-168 / 第 141-168 行
```cpp
 141: void LoongArchFrameLowering::processFunctionBeforeFrameFinalized(
 142:     MachineFunction &MF, RegScavenger *RS) const {
 143:   const LoongArchRegisterInfo *RI = STI.getRegisterInfo();
 144:   const TargetRegisterClass &RC = LoongArch::GPRRegClass;
 145:   const LoongArchInstrInfo *TII = STI.getInstrInfo();
 146:   LoongArchMachineFunctionInfo *LAFI =
 147:       MF.getInfo<LoongArchMachineFunctionInfo>();
 148:   MachineFrameInfo &MFI = MF.getFrameInfo();
 149: 
 150:   unsigned ScavSlotsNum = 0;
 151: 
 152:   // Far branches beyond 27-bit offset require a spill slot for scratch
 153:   // register.
 154:   bool IsLargeFunction = !isInt<27>(estimateFunctionSizeInBytes(TII, MF));
 155:   if (IsLargeFunction)
 156:     ScavSlotsNum = 1;
 157: 
 158:   // estimateStackSize has been observed to under-estimate the final stack
 159:   // size, so give ourselves wiggle-room by checking for stack size
 160:   // representable an 11-bit signed field rather than 12-bits.
 161:   // For [x]vstelm.{b/h/w/d} memory instructions with 8 imm offset, 7-bit
 162:   // signed field is fine.
 163:   unsigned EstimateStackSize = MFI.estimateStackSize(MF);
 164:   if (!isInt<11>(EstimateStackSize) ||
 165:       (MF.getSubtarget<LoongArchSubtarget>().hasExtLSX() &&
 166:        !isInt<7>(EstimateStackSize)))
 167:     ScavSlotsNum = std::max(ScavSlotsNum, 1u);
 168: 
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::processFunctionBeforeFrameFinalized`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::processFunctionBeforeFrameFinalized` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   // For CFR spill.
 170:   if (needScavSlotForCFR(MF))
 171:     ++ScavSlotsNum;
 172: 
 173:   // Create emergency spill slots.
 174:   for (unsigned i = 0; i < ScavSlotsNum; ++i) {
 175:     int FI =
 176:         MFI.CreateSpillStackObject(RI->getSpillSize(RC), RI->getSpillAlign(RC));
 177:     RS->addScavengingFrameIndex(FI);
 178:     if (IsLargeFunction && LAFI->getBranchRelaxationSpillFrameIndex() == -1)
 179:       LAFI->setBranchRelaxationSpillFrameIndex(FI);
 180:     LLVM_DEBUG(dbgs() << "Allocated FI(" << FI
 181:                       << ") as the emergency spill slot.\n");
 182:   }
 183: }
 184: 
 185: void LoongArchFrameLowering::emitPrologue(MachineFunction &MF,
 186:                                           MachineBasicBlock &MBB) const {
 187:   MachineFrameInfo &MFI = MF.getFrameInfo();
 188:   auto *LoongArchFI = MF.getInfo<LoongArchMachineFunctionInfo>();
 189:   const LoongArchRegisterInfo *RI = STI.getRegisterInfo();
 190:   const LoongArchInstrInfo *TII = STI.getInstrInfo();
 191:   MachineBasicBlock::iterator MBBI = MBB.begin();
 192:   bool IsLA64 = STI.is64Bit();
 193: 
 194:   Register SPReg = LoongArch::R3;
 195:   Register FPReg = LoongArch::R22;
 196: 
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::emitPrologue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::emitPrologue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 197-224 / 第 197-224 行
```cpp
 197:   // Debug location must be unknown since the first debug location is used
 198:   // to determine the end of the prologue.
 199:   DebugLoc DL;
 200:   // All calls are tail calls in GHC calling conv, and functions have no
 201:   // prologue/epilogue.
 202:   if (MF.getFunction().getCallingConv() == CallingConv::GHC)
 203:     return;
 204:   // Determine the correct frame layout
 205:   determineFrameLayout(MF);
 206: 
 207:   // First, compute final stack size.
 208:   uint64_t StackSize = MFI.getStackSize();
 209:   uint64_t RealStackSize = StackSize;
 210: 
 211:   // Early exit if there is no need to allocate space in the stack.
 212:   if (StackSize == 0 && !MFI.adjustsStack())
 213:     return;
 214: 
 215:   uint64_t FirstSPAdjustAmount = getFirstSPAdjustAmount(MF);
 216:   // Split the SP adjustment to reduce the offsets of callee saved spill.
 217:   if (FirstSPAdjustAmount)
 218:     StackSize = FirstSPAdjustAmount;
 219: 
 220:   // Adjust stack.
 221:   adjustReg(MBB, MBBI, DL, SPReg, SPReg, -StackSize, MachineInstr::FrameSetup);
 222:   // Emit ".cfi_def_cfa_offset StackSize".
 223:   unsigned CFIIndex =
 224:       MF.addFrameInst(MCCFIInstruction::cfiDefCfaOffset(nullptr, StackSize));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 225-252 / 第 225-252 行
```cpp
 225:   BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 226:       .addCFIIndex(CFIIndex)
 227:       .setMIFlag(MachineInstr::FrameSetup);
 228: 
 229:   const auto &CSI = MFI.getCalleeSavedInfo();
 230: 
 231:   // The frame pointer is callee-saved, and code has been generated for us to
 232:   // save it to the stack. We need to skip over the storing of callee-saved
 233:   // registers as the frame pointer must be modified after it has been saved
 234:   // to the stack, not before.
 235:   std::advance(MBBI, CSI.size());
 236: 
 237:   // Iterate over list of callee-saved registers and emit .cfi_offset
 238:   // directives.
 239:   for (const auto &Entry : CSI) {
 240:     int64_t Offset = MFI.getObjectOffset(Entry.getFrameIdx());
 241:     unsigned CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 242:         nullptr, RI->getDwarfRegNum(Entry.getReg(), true), Offset));
 243:     BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 244:         .addCFIIndex(CFIIndex)
 245:         .setMIFlag(MachineInstr::FrameSetup);
 246:   }
 247: 
 248:   // Generate new FP.
 249:   if (hasFP(MF)) {
 250:     adjustReg(MBB, MBBI, DL, FPReg, SPReg,
 251:               StackSize - LoongArchFI->getVarArgsSaveSize(),
 252:               MachineInstr::FrameSetup);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 253-280 / 第 253-280 行
```cpp
 253: 
 254:     // Emit ".cfi_def_cfa $fp, LoongArchFI->getVarArgsSaveSize()"
 255:     unsigned CFIIndex = MF.addFrameInst(
 256:         MCCFIInstruction::cfiDefCfa(nullptr, RI->getDwarfRegNum(FPReg, true),
 257:                                     LoongArchFI->getVarArgsSaveSize()));
 258:     BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 259:         .addCFIIndex(CFIIndex)
 260:         .setMIFlag(MachineInstr::FrameSetup);
 261:   }
 262: 
 263:   // Emit the second SP adjustment after saving callee saved registers.
 264:   if (FirstSPAdjustAmount) {
 265:     uint64_t SecondSPAdjustAmount = RealStackSize - FirstSPAdjustAmount;
 266:     assert(SecondSPAdjustAmount > 0 &&
 267:            "SecondSPAdjustAmount should be greater than zero");
 268:     adjustReg(MBB, MBBI, DL, SPReg, SPReg, -SecondSPAdjustAmount,
 269:               MachineInstr::FrameSetup);
 270: 
 271:     if (!hasFP(MF)) {
 272:       // If we are using a frame-pointer, and thus emitted ".cfi_def_cfa fp, 0",
 273:       // don't emit an sp-based .cfi_def_cfa_offset
 274:       // Emit ".cfi_def_cfa_offset RealStackSize"
 275:       unsigned CFIIndex = MF.addFrameInst(
 276:           MCCFIInstruction::cfiDefCfaOffset(nullptr, RealStackSize));
 277:       BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 278:           .addCFIIndex(CFIIndex)
 279:           .setMIFlag(MachineInstr::FrameSetup);
 280:     }
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-308 / 第 281-308 行
```cpp
 281:   }
 282: 
 283:   if (hasFP(MF)) {
 284:     // Realign stack.
 285:     if (RI->hasStackRealignment(MF)) {
 286:       unsigned Align = Log2(MFI.getMaxAlign());
 287:       assert(Align > 0 && "The stack realignment size is invalid!");
 288:       BuildMI(MBB, MBBI, DL,
 289:               TII->get(IsLA64 ? LoongArch::BSTRINS_D : LoongArch::BSTRINS_W),
 290:               SPReg)
 291:           .addReg(SPReg)
 292:           .addReg(LoongArch::R0)
 293:           .addImm(Align - 1)
 294:           .addImm(0)
 295:           .setMIFlag(MachineInstr::FrameSetup);
 296:       // FP will be used to restore the frame in the epilogue, so we need
 297:       // another base register BP to record SP after re-alignment. SP will
 298:       // track the current stack after allocating variable sized objects.
 299:       if (hasBP(MF)) {
 300:         // move BP, $sp
 301:         BuildMI(MBB, MBBI, DL, TII->get(LoongArch::OR),
 302:                 LoongArchABI::getBPReg())
 303:             .addReg(SPReg)
 304:             .addReg(LoongArch::R0)
 305:             .setMIFlag(MachineInstr::FrameSetup);
 306:       }
 307:     }
 308:   }
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 309-336 / 第 309-336 行
```cpp
 309: }
 310: 
 311: void LoongArchFrameLowering::emitEpilogue(MachineFunction &MF,
 312:                                           MachineBasicBlock &MBB) const {
 313:   const LoongArchRegisterInfo *RI = STI.getRegisterInfo();
 314:   MachineFrameInfo &MFI = MF.getFrameInfo();
 315:   auto *LoongArchFI = MF.getInfo<LoongArchMachineFunctionInfo>();
 316:   Register SPReg = LoongArch::R3;
 317:   // All calls are tail calls in GHC calling conv, and functions have no
 318:   // prologue/epilogue.
 319:   if (MF.getFunction().getCallingConv() == CallingConv::GHC)
 320:     return;
 321:   MachineBasicBlock::iterator MBBI = MBB.getFirstTerminator();
 322:   DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
 323: 
 324:   const auto &CSI = MFI.getCalleeSavedInfo();
 325:   // Skip to before the restores of callee-saved registers.
 326:   auto LastFrameDestroy = MBBI;
 327:   if (!CSI.empty())
 328:     LastFrameDestroy = std::prev(MBBI, CSI.size());
 329: 
 330:   // Get the number of bytes from FrameInfo.
 331:   uint64_t StackSize = MFI.getStackSize();
 332: 
 333:   // Restore the stack pointer.
 334:   if (RI->hasStackRealignment(MF) || MFI.hasVarSizedObjects()) {
 335:     assert(hasFP(MF) && "frame pointer should not have been eliminated");
 336:     adjustReg(MBB, LastFrameDestroy, DL, SPReg, LoongArch::R22,
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::emitEpilogue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::emitEpilogue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 337-364 / 第 337-364 行
```cpp
 337:               -StackSize + LoongArchFI->getVarArgsSaveSize(),
 338:               MachineInstr::FrameDestroy);
 339:   }
 340: 
 341:   uint64_t FirstSPAdjustAmount = getFirstSPAdjustAmount(MF);
 342:   if (FirstSPAdjustAmount) {
 343:     uint64_t SecondSPAdjustAmount = StackSize - FirstSPAdjustAmount;
 344:     assert(SecondSPAdjustAmount > 0 &&
 345:            "SecondSPAdjustAmount should be greater than zero");
 346: 
 347:     adjustReg(MBB, LastFrameDestroy, DL, SPReg, SPReg, SecondSPAdjustAmount,
 348:               MachineInstr::FrameDestroy);
 349:     StackSize = FirstSPAdjustAmount;
 350:   }
 351: 
 352:   // Deallocate stack
 353:   adjustReg(MBB, MBBI, DL, SPReg, SPReg, StackSize, MachineInstr::FrameDestroy);
 354: }
 355: 
 356: // We would like to split the SP adjustment to reduce prologue/epilogue
 357: // as following instructions. In this way, the offset of the callee saved
 358: // register could fit in a single store.
 359: // e.g.
 360: //   addi.d  $sp, $sp, -2032
 361: //   st.d    $ra, $sp,  2024
 362: //   st.d    $fp, $sp,  2016
 363: //   addi.d  $sp, $sp,   -16
 364: uint64_t LoongArchFrameLowering::getFirstSPAdjustAmount(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 365-392 / 第 365-392 行
```cpp
 365:     const MachineFunction &MF) const {
 366:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 367:   const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
 368: 
 369:   // Return the FirstSPAdjustAmount if the StackSize can not fit in a signed
 370:   // 12-bit and there exists a callee-saved register needing to be pushed.
 371:   if (!isInt<12>(MFI.getStackSize()) && (CSI.size() > 0)) {
 372:     // FirstSPAdjustAmount is chosen as (2048 - StackAlign) because 2048 will
 373:     // cause sp = sp + 2048 in the epilogue to be split into multiple
 374:     // instructions. Offsets smaller than 2048 can fit in a single load/store
 375:     // instruction, and we have to stick with the stack alignment.
 376:     // So (2048 - StackAlign) will satisfy the stack alignment.
 377:     return 2048 - getStackAlign().value();
 378:   }
 379:   return 0;
 380: }
 381: 
 382: void LoongArchFrameLowering::determineCalleeSaves(MachineFunction &MF,
 383:                                                   BitVector &SavedRegs,
 384:                                                   RegScavenger *RS) const {
 385:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
 386:   // Unconditionally spill RA and FP only if the function uses a frame
 387:   // pointer.
 388:   if (hasFP(MF)) {
 389:     SavedRegs.set(LoongArch::R1);
 390:     SavedRegs.set(LoongArch::R22);
 391:   }
 392:   // Mark BP as used if function has dedicated base pointer.
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::determineCalleeSaves`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::determineCalleeSaves` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   if (hasBP(MF))
 394:     SavedRegs.set(LoongArchABI::getBPReg());
 395: }
 396: 
 397: // Do not preserve stack space within prologue for outgoing variables if the
 398: // function contains variable size objects.
 399: // Let eliminateCallFramePseudoInstr preserve stack space for it.
 400: bool LoongArchFrameLowering::hasReservedCallFrame(
 401:     const MachineFunction &MF) const {
 402:   return !MF.getFrameInfo().hasVarSizedObjects();
 403: }
 404: 
 405: // Eliminate ADJCALLSTACKDOWN, ADJCALLSTACKUP pseudo instructions.
 406: MachineBasicBlock::iterator
 407: LoongArchFrameLowering::eliminateCallFramePseudoInstr(
 408:     MachineFunction &MF, MachineBasicBlock &MBB,
 409:     MachineBasicBlock::iterator MI) const {
 410:   Register SPReg = LoongArch::R3;
 411:   DebugLoc DL = MI->getDebugLoc();
 412: 
 413:   if (!hasReservedCallFrame(MF)) {
 414:     // If space has not been reserved for a call frame, ADJCALLSTACKDOWN and
 415:     // ADJCALLSTACKUP must be converted to instructions manipulating the stack
 416:     // pointer. This is necessary when there is a variable length stack
 417:     // allocation (e.g. alloca), which means it's not possible to allocate
 418:     // space for outgoing arguments from within the function prologue.
 419:     int64_t Amount = MI->getOperand(0).getImm();
 420: 
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::hasReservedCallFrame`, `LoongArchFrameLowering::eliminateCallFramePseudoInstr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::hasReservedCallFrame`, `LoongArchFrameLowering::eliminateCallFramePseudoInstr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 421-448 / 第 421-448 行
```cpp
 421:     if (Amount != 0) {
 422:       // Ensure the stack remains aligned after adjustment.
 423:       Amount = alignSPAdjust(Amount);
 424: 
 425:       if (MI->getOpcode() == LoongArch::ADJCALLSTACKDOWN)
 426:         Amount = -Amount;
 427: 
 428:       adjustReg(MBB, MI, DL, SPReg, SPReg, Amount, MachineInstr::NoFlags);
 429:     }
 430:   }
 431: 
 432:   return MBB.erase(MI);
 433: }
 434: 
 435: bool LoongArchFrameLowering::spillCalleeSavedRegisters(
 436:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 437:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 438:   if (CSI.empty())
 439:     return true;
 440: 
 441:   MachineFunction *MF = MBB.getParent();
 442:   const TargetInstrInfo &TII = *MF->getSubtarget().getInstrInfo();
 443: 
 444:   // Insert the spill to the stack frame.
 445:   for (auto &CS : CSI) {
 446:     MCRegister Reg = CS.getReg();
 447:     // If the register is RA and the return address is taken by method
 448:     // LoongArchTargetLowering::lowerRETURNADDR, don't set kill flag.
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::spillCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::spillCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 449-476 / 第 449-476 行
```cpp
 449:     bool IsKill =
 450:         !(Reg == LoongArch::R1 && MF->getFrameInfo().isReturnAddressTaken());
 451:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(Reg);
 452:     TII.storeRegToStackSlot(MBB, MI, Reg, IsKill, CS.getFrameIdx(), RC,
 453:                             Register());
 454:   }
 455: 
 456:   return true;
 457: }
 458: 
 459: StackOffset LoongArchFrameLowering::getFrameIndexReference(
 460:     const MachineFunction &MF, int FI, Register &FrameReg) const {
 461:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 462:   const TargetRegisterInfo *RI = MF.getSubtarget().getRegisterInfo();
 463:   auto *LoongArchFI = MF.getInfo<LoongArchMachineFunctionInfo>();
 464:   uint64_t StackSize = MFI.getStackSize();
 465:   uint64_t FirstSPAdjustAmount = getFirstSPAdjustAmount(MF);
 466: 
 467:   // Callee-saved registers should be referenced relative to the stack
 468:   // pointer (positive offset), otherwise use the frame pointer (negative
 469:   // offset).
 470:   const auto &CSI = MFI.getCalleeSavedInfo();
 471:   int MinCSFI = 0;
 472:   int MaxCSFI = -1;
 473:   StackOffset Offset =
 474:       StackOffset::getFixed(MFI.getObjectOffset(FI) - getOffsetOfLocalArea() +
 475:                             MFI.getOffsetAdjustment());
 476: 
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::getFrameIndexReference`.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::getFrameIndexReference` 等函数。

### Lines 477-504 / 第 477-504 行
```cpp
 477:   if (CSI.size()) {
 478:     MinCSFI = CSI[0].getFrameIdx();
 479:     MaxCSFI = CSI[CSI.size() - 1].getFrameIdx();
 480:   }
 481: 
 482:   if (FI >= MinCSFI && FI <= MaxCSFI) {
 483:     FrameReg = LoongArch::R3;
 484:     if (FirstSPAdjustAmount)
 485:       Offset += StackOffset::getFixed(FirstSPAdjustAmount);
 486:     else
 487:       Offset += StackOffset::getFixed(StackSize);
 488:   } else if (RI->hasStackRealignment(MF) && !MFI.isFixedObjectIndex(FI)) {
 489:     // If the stack was realigned, the frame pointer is set in order to allow
 490:     // SP to be restored, so we need another base register to record the stack
 491:     // after realignment.
 492:     FrameReg = hasBP(MF) ? LoongArchABI::getBPReg() : LoongArch::R3;
 493:     Offset += StackOffset::getFixed(StackSize);
 494:   } else {
 495:     FrameReg = RI->getFrameRegister(MF);
 496:     if (hasFP(MF))
 497:       Offset += StackOffset::getFixed(LoongArchFI->getVarArgsSaveSize());
 498:     else
 499:       Offset += StackOffset::getFixed(StackSize);
 500:   }
 501: 
 502:   return Offset;
 503: }
 504: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 505-512 / 第 505-512 行
```cpp
 505: bool LoongArchFrameLowering::enableShrinkWrapping(
 506:     const MachineFunction &MF) const {
 507:   // Keep the conventional code flow when not optimizing.
 508:   if (MF.getFunction().hasOptNone())
 509:     return false;
 510: 
 511:   return true;
 512: }
```
- **EN**: The range implements or declares functions including `LoongArchFrameLowering::enableShrinkWrapping`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchFrameLowering::enableShrinkWrapping` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `LoongArchFrameLowering.h`
- `LoongArchMachineFunctionInfo.h`
- `LoongArchSubtarget.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/RegisterScavenging.h`
- `llvm/IR/DiagnosticInfo.h`
- `llvm/MC/MCDwarf.h`
