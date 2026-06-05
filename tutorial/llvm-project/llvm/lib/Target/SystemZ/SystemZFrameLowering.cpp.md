# SystemZFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZFrameLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责实现栈帧降低以及序言/尾声逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: //===-- SystemZFrameLowering.cpp - Frame lowering for SystemZ -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZFrameLowering.h"
  10: #include "SystemZCallingConv.h"
  11: #include "SystemZInstrInfo.h"
  12: #include "SystemZMachineFunctionInfo.h"
  13: #include "SystemZRegisterInfo.h"
  14: #include "SystemZSubtarget.h"
  15: #include "llvm/ADT/STLExtras.h"
  16: #include "llvm/CodeGen/LivePhysRegs.h"
  17: #include "llvm/CodeGen/MachineInstrBuilder.h"
  18: #include "llvm/CodeGen/MachineModuleInfo.h"
  19: #include "llvm/CodeGen/MachineRegisterInfo.h"
  20: #include "llvm/CodeGen/RegisterScavenging.h"
  21: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  22: #include "llvm/IR/CallingConv.h"
  23: #include "llvm/IR/Function.h"
  24: #include "llvm/Target/TargetMachine.h"
  25: 
  26: using namespace llvm;
  27: 
  28: namespace {
  29: // The ABI-defined register save slots, relative to the CFA (i.e.
  30: // incoming stack pointer + SystemZMC::ELFCallFrameSize).
  31: static const TargetFrameLowering::SpillSlot ELFSpillOffsetTable[] = {
  32:   { SystemZ::R2D,  0x10 },
  33:   { SystemZ::R3D,  0x18 },
  34:   { SystemZ::R4D,  0x20 },
  35:   { SystemZ::R5D,  0x28 },
  36:   { SystemZ::R6D,  0x30 },
  37:   { SystemZ::R7D,  0x38 },
  38:   { SystemZ::R8D,  0x40 },
  39:   { SystemZ::R9D,  0x48 },
  40:   { SystemZ::R10D, 0x50 },
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZFrameLowering.h`, `SystemZCallingConv.h`, `SystemZInstrInfo.h`, `SystemZMachineFunctionInfo.h`, `SystemZRegisterInfo.h`, `SystemZSubtarget.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZFrameLowering.h`, `SystemZCallingConv.h`, `SystemZInstrInfo.h`, `SystemZMachineFunctionInfo.h`, `SystemZRegisterInfo.h`, `SystemZSubtarget.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 41-80 / 第 41-80 行
```cpp
  41:   { SystemZ::R11D, 0x58 },
  42:   { SystemZ::R12D, 0x60 },
  43:   { SystemZ::R13D, 0x68 },
  44:   { SystemZ::R14D, 0x70 },
  45:   { SystemZ::R15D, 0x78 },
  46:   { SystemZ::F0D,  0x80 },
  47:   { SystemZ::F2D,  0x88 },
  48:   { SystemZ::F4D,  0x90 },
  49:   { SystemZ::F6D,  0x98 }
  50: };
  51: 
  52: static const TargetFrameLowering::SpillSlot XPLINKSpillOffsetTable[] = {
  53:     {SystemZ::R4D, 0x00},  {SystemZ::R5D, 0x08},  {SystemZ::R6D, 0x10},
  54:     {SystemZ::R7D, 0x18},  {SystemZ::R8D, 0x20},  {SystemZ::R9D, 0x28},
  55:     {SystemZ::R10D, 0x30}, {SystemZ::R11D, 0x38}, {SystemZ::R12D, 0x40},
  56:     {SystemZ::R13D, 0x48}, {SystemZ::R14D, 0x50}, {SystemZ::R15D, 0x58}};
  57: } // end anonymous namespace
  58: 
  59: SystemZFrameLowering::SystemZFrameLowering(StackDirection D, Align StackAl,
  60:                                            int LAO, Align TransAl,
  61:                                            bool StackReal, unsigned PointerSize)
  62:     : TargetFrameLowering(D, StackAl, LAO, TransAl, StackReal),
  63:       PointerSize(PointerSize) {}
  64: 
  65: std::unique_ptr<SystemZFrameLowering>
  66: SystemZFrameLowering::create(const SystemZSubtarget &STI) {
  67:   unsigned PtrSz =
  68:       STI.getTargetLowering()->getTargetMachine().getPointerSize(0);
  69:   if (STI.isTargetXPLINK64())
  70:     return std::make_unique<SystemZXPLINKFrameLowering>(PtrSz);
  71:   return std::make_unique<SystemZELFFrameLowering>(PtrSz);
  72: }
  73: 
  74: namespace {
  75: struct SZFrameSortingObj {
  76:   bool IsValid = false;     // True if we care about this Object.
  77:   uint32_t ObjectIndex = 0; // Index of Object into MFI list.
  78:   uint64_t ObjectSize = 0;  // Size of Object in bytes.
  79:   uint32_t D12Count = 0;    // 12-bit displacement only.
  80:   uint32_t DPairCount = 0;  // 12 or 20 bit displacement.
```
- **EN**: It introduces interface types such as `SZFrameSortingObj`, shaping how other backend components interact with this file. The range implements or declares functions including `TargetFrameLowering`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SZFrameSortingObj` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `TargetFrameLowering` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 81-120 / 第 81-120 行
```cpp
  81: };
  82: typedef std::vector<SZFrameSortingObj> SZFrameObjVec;
  83: } // namespace
  84: 
  85: // TODO: Move to base class.
  86: void SystemZELFFrameLowering::orderFrameObjects(
  87:     const MachineFunction &MF, SmallVectorImpl<int> &ObjectsToAllocate) const {
  88:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  89:   auto *TII = MF.getSubtarget<SystemZSubtarget>().getInstrInfo();
  90: 
  91:   // Make a vector of sorting objects to track all MFI objects and mark those
  92:   // to be sorted as valid.
  93:   if (ObjectsToAllocate.size() <= 1)
  94:     return;
  95:   SZFrameObjVec SortingObjects(MFI.getObjectIndexEnd());
  96:   for (auto &Obj : ObjectsToAllocate) {
  97:     SortingObjects[Obj].IsValid = true;
  98:     SortingObjects[Obj].ObjectIndex = Obj;
  99:     SortingObjects[Obj].ObjectSize = MFI.getObjectSize(Obj);
 100:   }
 101: 
 102:   // Examine uses for each object and record short (12-bit) and "pair"
 103:   // displacement types.
 104:   for (auto &MBB : MF)
 105:     for (auto &MI : MBB) {
 106:       if (MI.isDebugInstr())
 107:         continue;
 108:       for (unsigned I = 0, E = MI.getNumOperands(); I != E; ++I) {
 109:         const MachineOperand &MO = MI.getOperand(I);
 110:         if (!MO.isFI())
 111:           continue;
 112:         int Index = MO.getIndex();
 113:         if (Index >= 0 && Index < MFI.getObjectIndexEnd() &&
 114:             SortingObjects[Index].IsValid) {
 115:           if (TII->hasDisplacementPairInsn(MI.getOpcode()))
 116:             SortingObjects[Index].DPairCount++;
 117:           else if (!(MI.getDesc().TSFlags & SystemZII::Has20BitOffset))
 118:             SortingObjects[Index].D12Count++;
 119:         }
 120:       }
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::orderFrameObjects`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::orderFrameObjects` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 121-160 / 第 121-160 行
```cpp
 121:     }
 122: 
 123:   // Sort all objects for short/paired displacements, which should be
 124:   // sufficient as it seems like all frame objects typically are within the
 125:   // long displacement range.  Sorting works by computing the "density" as
 126:   // Count / ObjectSize. The comparisons of two such fractions are refactored
 127:   // by multiplying both sides with A.ObjectSize * B.ObjectSize, in order to
 128:   // eliminate the (fp) divisions.  A higher density object needs to go after
 129:   // in the list in order for it to end up lower on the stack.
 130:   auto CmpD12 = [](const SZFrameSortingObj &A, const SZFrameSortingObj &B) {
 131:     // Put all invalid and variable sized objects at the end.
 132:     if (!A.IsValid || !B.IsValid)
 133:       return A.IsValid;
 134:     if (!A.ObjectSize || !B.ObjectSize)
 135:       return A.ObjectSize > 0;
 136:     uint64_t ADensityCmp = A.D12Count * B.ObjectSize;
 137:     uint64_t BDensityCmp = B.D12Count * A.ObjectSize;
 138:     if (ADensityCmp != BDensityCmp)
 139:       return ADensityCmp < BDensityCmp;
 140:     return A.DPairCount * B.ObjectSize < B.DPairCount * A.ObjectSize;
 141:   };
 142:   llvm::stable_sort(SortingObjects, CmpD12);
 143: 
 144:   // Now modify the original list to represent the final order that
 145:   // we want.
 146:   unsigned Idx = 0;
 147:   for (auto &Obj : SortingObjects) {
 148:     // All invalid items are sorted at the end, so it's safe to stop.
 149:     if (!Obj.IsValid)
 150:       break;
 151:     ObjectsToAllocate[Idx++] = Obj.ObjectIndex;
 152:   }
 153: }
 154: 
 155: bool SystemZFrameLowering::hasReservedCallFrame(
 156:     const MachineFunction &MF) const {
 157:   // The ELF ABI requires us to allocate 160 bytes of stack space for the
 158:   // callee, with any outgoing stack arguments being placed above that. It
 159:   // seems better to make that area a permanent feature of the frame even if
 160:   // we're using a frame pointer. Similarly, 64-bit XPLINK requires 96 bytes
```
- **EN**: The range implements or declares functions including `SystemZFrameLowering::hasReservedCallFrame`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZFrameLowering::hasReservedCallFrame` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 161-200 / 第 161-200 行
```cpp
 161:   // of stack space for the register save area.
 162:   return true;
 163: }
 164: 
 165: void SystemZFrameLowering::emitIncrement(MachineBasicBlock &MBB,
 166:                                          MachineBasicBlock::iterator &MBBI,
 167:                                          const DebugLoc &DL, Register Reg,
 168:                                          int64_t NumBytes,
 169:                                          const TargetInstrInfo *TII) const {
 170:   while (NumBytes) {
 171:     unsigned Opcode;
 172:     int64_t ThisVal = NumBytes;
 173:     if (isInt<16>(NumBytes))
 174:       Opcode = SystemZ::AGHI;
 175:     else {
 176:       Opcode = SystemZ::AGFI;
 177:       // Make sure we maintain stack alignment.
 178:       int64_t MinVal = -uint64_t(1) << 31;
 179:       int64_t MaxVal = (int64_t(1) << 31) - getStackAlignment();
 180:       if (ThisVal < MinVal)
 181:         ThisVal = MinVal;
 182:       else if (ThisVal > MaxVal)
 183:         ThisVal = MaxVal;
 184:     }
 185:     MachineInstr *MI = BuildMI(MBB, MBBI, DL, TII->get(Opcode), Reg)
 186:                            .addReg(Reg)
 187:                            .addImm(ThisVal);
 188:     // The CC implicit def is dead.
 189:     MI->getOperand(3).setIsDead();
 190:     NumBytes -= ThisVal;
 191:   }
 192: }
 193: 
 194: bool SystemZELFFrameLowering::assignCalleeSavedSpillSlots(
 195:     MachineFunction &MF, const TargetRegisterInfo *TRI,
 196:     std::vector<CalleeSavedInfo> &CSI) const {
 197:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 198:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
 199:   bool IsVarArg = MF.getFunction().isVarArg();
 200:   if (CSI.empty())
```
- **EN**: The range implements or declares functions including `SystemZFrameLowering::emitIncrement`, `SystemZELFFrameLowering::assignCalleeSavedSpillSlots`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZFrameLowering::emitIncrement`, `SystemZELFFrameLowering::assignCalleeSavedSpillSlots` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 201-240 / 第 201-240 行
```cpp
 201:     return true; // Early exit if no callee saved registers are modified!
 202: 
 203:   unsigned LowGPR = 0;
 204:   unsigned HighGPR = SystemZ::R15D;
 205:   int StartSPOffset = SystemZMC::ELFCallFrameSize;
 206:   for (auto &CS : CSI) {
 207:     MCRegister Reg = CS.getReg();
 208:     int Offset = getRegSpillOffset(MF, Reg);
 209:     if (Offset) {
 210:       if (SystemZ::GR64BitRegClass.contains(Reg) && StartSPOffset > Offset) {
 211:         LowGPR = Reg;
 212:         StartSPOffset = Offset;
 213:       }
 214:       Offset -= SystemZMC::ELFCallFrameSize;
 215:       int FrameIdx =
 216:           MFFrame.CreateFixedSpillStackObject(getPointerSize(), Offset);
 217:       CS.setFrameIdx(FrameIdx);
 218:     } else
 219:       CS.setFrameIdx(INT32_MAX);
 220:   }
 221: 
 222:   // Save the range of call-saved registers, for use by the
 223:   // prologue/epilogue inserters.
 224:   ZFI->setRestoreGPRRegs(LowGPR, HighGPR, StartSPOffset);
 225:   if (IsVarArg) {
 226:     // Also save the GPR varargs, if any.  R6D is call-saved, so would
 227:     // already be included, but we also need to handle the call-clobbered
 228:     // argument registers.
 229:     Register FirstGPR = ZFI->getVarArgsFirstGPR();
 230:     if (FirstGPR < SystemZ::ELFNumArgGPRs) {
 231:       unsigned Reg = SystemZ::ELFArgGPRs[FirstGPR];
 232:       int Offset = getRegSpillOffset(MF, Reg);
 233:       if (StartSPOffset > Offset) {
 234:         LowGPR = Reg; StartSPOffset = Offset;
 235:       }
 236:     }
 237:   }
 238:   ZFI->setSpillGPRRegs(LowGPR, HighGPR, StartSPOffset);
 239: 
 240:   // Create fixed stack objects for the remaining registers.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 241-280 / 第 241-280 行
```cpp
 241:   int CurrOffset = -SystemZMC::ELFCallFrameSize;
 242:   if (usePackedStack(MF))
 243:     CurrOffset += StartSPOffset;
 244: 
 245:   for (auto &CS : CSI) {
 246:     if (CS.getFrameIdx() != INT32_MAX)
 247:       continue;
 248:     MCRegister Reg = CS.getReg();
 249:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(Reg);
 250:     unsigned Size = TRI->getSpillSize(*RC);
 251:     CurrOffset -= Size;
 252:     assert(CurrOffset % 8 == 0 &&
 253:            "8-byte alignment required for for all register save slots");
 254:     int FrameIdx = MFFrame.CreateFixedSpillStackObject(Size, CurrOffset);
 255:     CS.setFrameIdx(FrameIdx);
 256:   }
 257: 
 258:   return true;
 259: }
 260: 
 261: void SystemZELFFrameLowering::determineCalleeSaves(MachineFunction &MF,
 262:                                                    BitVector &SavedRegs,
 263:                                                    RegScavenger *RS) const {
 264:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
 265: 
 266:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
 267:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
 268:   bool HasFP = hasFP(MF);
 269:   SystemZMachineFunctionInfo *MFI = MF.getInfo<SystemZMachineFunctionInfo>();
 270:   bool IsVarArg = MF.getFunction().isVarArg();
 271: 
 272:   // va_start stores incoming FPR varargs in the normal way, but delegates
 273:   // the saving of incoming GPR varargs to spillCalleeSavedRegisters().
 274:   // Record these pending uses, which typically include the call-saved
 275:   // argument register R6D.
 276:   if (IsVarArg)
 277:     for (unsigned I = MFI->getVarArgsFirstGPR(); I < SystemZ::ELFNumArgGPRs; ++I)
 278:       SavedRegs.set(SystemZ::ELFArgGPRs[I]);
 279: 
 280:   // If there are any landing pads, entering them will modify r6/r7.
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::determineCalleeSaves`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::determineCalleeSaves` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 281-320 / 第 281-320 行
```cpp
 281:   if (!MF.getLandingPads().empty()) {
 282:     SavedRegs.set(SystemZ::R6D);
 283:     SavedRegs.set(SystemZ::R7D);
 284:   }
 285: 
 286:   // If the function requires a frame pointer, record that the hard
 287:   // frame pointer will be clobbered.
 288:   if (HasFP)
 289:     SavedRegs.set(SystemZ::R11D);
 290: 
 291:   // If the function calls other functions, record that the return
 292:   // address register will be clobbered.
 293:   if (MFFrame.hasCalls())
 294:     SavedRegs.set(SystemZ::R14D);
 295: 
 296:   // If we are saving GPRs other than the stack pointer, we might as well
 297:   // save and restore the stack pointer at the same time, via STMG and LMG.
 298:   // This allows the deallocation to be done by the LMG, rather than needing
 299:   // a separate %r15 addition.
 300:   const MCPhysReg *CSRegs = TRI->getCalleeSavedRegs(&MF);
 301:   for (unsigned I = 0; CSRegs[I]; ++I) {
 302:     unsigned Reg = CSRegs[I];
 303:     if (SystemZ::GR64BitRegClass.contains(Reg) && SavedRegs.test(Reg)) {
 304:       SavedRegs.set(SystemZ::R15D);
 305:       break;
 306:     }
 307:   }
 308: }
 309: 
 310: SystemZELFFrameLowering::SystemZELFFrameLowering(unsigned PointerSize)
 311:     : SystemZFrameLowering(TargetFrameLowering::StackGrowsDown, Align(8), 0,
 312:                            Align(8), /* StackRealignable */ false, PointerSize),
 313:       RegSpillOffsets(0) {
 314: 
 315:   // Due to the SystemZ ABI, the DWARF CFA (Canonical Frame Address) is not
 316:   // equal to the incoming stack pointer, but to incoming stack pointer plus
 317:   // 160.  Instead of using a Local Area Offset, the Register save area will
 318:   // be occupied by fixed frame objects, and all offsets are actually
 319:   // relative to CFA.
 320: 
```
- **EN**: The range implements or declares functions including `SystemZFrameLowering`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZFrameLowering` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 321-360 / 第 321-360 行
```cpp
 321:   // Create a mapping from register number to save slot offset.
 322:   // These offsets are relative to the start of the register save area.
 323:   RegSpillOffsets.grow(SystemZ::NUM_TARGET_REGS);
 324:   for (const auto &Entry : ELFSpillOffsetTable)
 325:     RegSpillOffsets[Entry.Reg] = Entry.Offset;
 326: }
 327: 
 328: // Add GPR64 to the save instruction being built by MIB, which is in basic
 329: // block MBB.  IsImplicit says whether this is an explicit operand to the
 330: // instruction, or an implicit one that comes between the explicit start
 331: // and end registers.
 332: static void addSavedGPR(MachineBasicBlock &MBB, MachineInstrBuilder &MIB,
 333:                         unsigned GPR64, bool IsImplicit) {
 334:   const TargetRegisterInfo *RI =
 335:       MBB.getParent()->getSubtarget().getRegisterInfo();
 336:   Register GPR32 = RI->getSubReg(GPR64, SystemZ::subreg_l32);
 337:   bool IsLive = MBB.isLiveIn(GPR64) || MBB.isLiveIn(GPR32);
 338:   if (!IsLive || !IsImplicit) {
 339:     MIB.addReg(GPR64, getImplRegState(IsImplicit) | getKillRegState(!IsLive));
 340:     if (!IsLive)
 341:       MBB.addLiveIn(GPR64);
 342:   }
 343: }
 344: 
 345: bool SystemZELFFrameLowering::spillCalleeSavedRegisters(
 346:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 347:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 348:   if (CSI.empty())
 349:     return false;
 350: 
 351:   MachineFunction &MF = *MBB.getParent();
 352:   const TargetInstrInfo *TII = MF.getSubtarget().getInstrInfo();
 353:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 354:   bool IsVarArg = MF.getFunction().isVarArg();
 355:   DebugLoc DL;
 356: 
 357:   // Save GPRs
 358:   SystemZ::GPRRegs SpillGPRs = ZFI->getSpillGPRRegs();
 359:   if (SpillGPRs.LowGPR) {
 360:     assert(SpillGPRs.LowGPR != SpillGPRs.HighGPR &&
```
- **EN**: The range implements or declares functions including `addSavedGPR`, `SystemZELFFrameLowering::spillCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `addSavedGPR`, `SystemZELFFrameLowering::spillCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 361-400 / 第 361-400 行
```cpp
 361:            "Should be saving %r15 and something else");
 362: 
 363:     // Build an STMG instruction.
 364:     MachineInstrBuilder MIB = BuildMI(MBB, MBBI, DL, TII->get(SystemZ::STMG));
 365: 
 366:     // Add the explicit register operands.
 367:     addSavedGPR(MBB, MIB, SpillGPRs.LowGPR, false);
 368:     addSavedGPR(MBB, MIB, SpillGPRs.HighGPR, false);
 369: 
 370:     // Add the address.
 371:     MIB.addReg(SystemZ::R15D).addImm(SpillGPRs.GPROffset);
 372: 
 373:     // Make sure all call-saved GPRs are included as operands and are
 374:     // marked as live on entry.
 375:     for (const CalleeSavedInfo &I : CSI) {
 376:       MCRegister Reg = I.getReg();
 377:       if (SystemZ::GR64BitRegClass.contains(Reg))
 378:         addSavedGPR(MBB, MIB, Reg, true);
 379:     }
 380: 
 381:     // ...likewise GPR varargs.
 382:     if (IsVarArg)
 383:       for (unsigned I = ZFI->getVarArgsFirstGPR(); I < SystemZ::ELFNumArgGPRs; ++I)
 384:         addSavedGPR(MBB, MIB, SystemZ::ELFArgGPRs[I], true);
 385:   }
 386: 
 387:   // Save FPRs/VRs in the normal TargetInstrInfo way.
 388:   for (const CalleeSavedInfo &I : CSI) {
 389:     MCRegister Reg = I.getReg();
 390:     if (SystemZ::FP64BitRegClass.contains(Reg)) {
 391:       MBB.addLiveIn(Reg);
 392:       TII->storeRegToStackSlot(MBB, MBBI, Reg, true, I.getFrameIdx(),
 393:                                &SystemZ::FP64BitRegClass, Register());
 394:     }
 395:     if (SystemZ::VR128BitRegClass.contains(Reg)) {
 396:       MBB.addLiveIn(Reg);
 397:       TII->storeRegToStackSlot(MBB, MBBI, Reg, true, I.getFrameIdx(),
 398:                                &SystemZ::VR128BitRegClass, Register());
 399:     }
 400:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 401-440 / 第 401-440 行
```cpp
 401: 
 402:   return true;
 403: }
 404: 
 405: bool SystemZELFFrameLowering::restoreCalleeSavedRegisters(
 406:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 407:     MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 408:   if (CSI.empty())
 409:     return false;
 410: 
 411:   MachineFunction &MF = *MBB.getParent();
 412:   const TargetInstrInfo *TII = MF.getSubtarget().getInstrInfo();
 413:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 414:   bool HasFP = hasFP(MF);
 415:   DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
 416: 
 417:   // Restore FPRs/VRs in the normal TargetInstrInfo way.
 418:   for (const CalleeSavedInfo &I : CSI) {
 419:     MCRegister Reg = I.getReg();
 420:     if (SystemZ::FP64BitRegClass.contains(Reg))
 421:       TII->loadRegFromStackSlot(MBB, MBBI, Reg, I.getFrameIdx(),
 422:                                 &SystemZ::FP64BitRegClass, Register());
 423:     if (SystemZ::VR128BitRegClass.contains(Reg))
 424:       TII->loadRegFromStackSlot(MBB, MBBI, Reg, I.getFrameIdx(),
 425:                                 &SystemZ::VR128BitRegClass, Register());
 426:   }
 427: 
 428:   // Restore call-saved GPRs (but not call-clobbered varargs, which at
 429:   // this point might hold return values).
 430:   SystemZ::GPRRegs RestoreGPRs = ZFI->getRestoreGPRRegs();
 431:   if (RestoreGPRs.LowGPR) {
 432:     // If we saved any of %r2-%r5 as varargs, we should also be saving
 433:     // and restoring %r6.  If we're saving %r6 or above, we should be
 434:     // restoring it too.
 435:     assert(RestoreGPRs.LowGPR != RestoreGPRs.HighGPR &&
 436:            "Should be loading %r15 and something else");
 437: 
 438:     // Build an LMG instruction.
 439:     MachineInstrBuilder MIB = BuildMI(MBB, MBBI, DL, TII->get(SystemZ::LMG));
 440: 
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::restoreCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::restoreCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 441-480 / 第 441-480 行
```cpp
 441:     // Add the explicit register operands.
 442:     MIB.addReg(RestoreGPRs.LowGPR, RegState::Define);
 443:     MIB.addReg(RestoreGPRs.HighGPR, RegState::Define);
 444: 
 445:     // Add the address.
 446:     MIB.addReg(HasFP ? SystemZ::R11D : SystemZ::R15D);
 447:     MIB.addImm(RestoreGPRs.GPROffset);
 448: 
 449:     // Do a second scan adding regs as being defined by instruction
 450:     for (const CalleeSavedInfo &I : CSI) {
 451:       MCRegister Reg = I.getReg();
 452:       if (Reg != RestoreGPRs.LowGPR && Reg != RestoreGPRs.HighGPR &&
 453:           SystemZ::GR64BitRegClass.contains(Reg))
 454:         MIB.addReg(Reg, RegState::ImplicitDefine);
 455:     }
 456:   }
 457: 
 458:   return true;
 459: }
 460: 
 461: void SystemZELFFrameLowering::processFunctionBeforeFrameFinalized(
 462:     MachineFunction &MF, RegScavenger *RS) const {
 463:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
 464:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 465:   MachineRegisterInfo *MRI = &MF.getRegInfo();
 466:   bool BackChain = MF.getSubtarget<SystemZSubtarget>().hasBackChain();
 467: 
 468:   if (!usePackedStack(MF) || BackChain)
 469:     // Create the incoming register save area.
 470:     getOrCreateFramePointerSaveIndex(MF);
 471: 
 472:   // Get the size of our stack frame to be allocated ...
 473:   uint64_t StackSize = (MFFrame.estimateStackSize(MF) +
 474:                         SystemZMC::ELFCallFrameSize);
 475:   // ... and the maximum offset we may need to reach into the
 476:   // caller's frame to access the save area or stack arguments.
 477:   int64_t MaxArgOffset = 0;
 478:   for (int I = MFFrame.getObjectIndexBegin(); I != 0; ++I)
 479:     if (MFFrame.getObjectOffset(I) >= 0) {
 480:       int64_t ArgOffset = MFFrame.getObjectOffset(I) +
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::processFunctionBeforeFrameFinalized`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::processFunctionBeforeFrameFinalized` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 481-520 / 第 481-520 行
```cpp
 481:                           MFFrame.getObjectSize(I);
 482:       MaxArgOffset = std::max(MaxArgOffset, ArgOffset);
 483:     }
 484: 
 485:   uint64_t MaxReach = StackSize + MaxArgOffset;
 486:   if (!isUInt<12>(MaxReach)) {
 487:     // We may need register scavenging slots if some parts of the frame
 488:     // are outside the reach of an unsigned 12-bit displacement.
 489:     // Create 2 for the case where both addresses in an MVC are
 490:     // out of range.
 491:     RS->addScavengingFrameIndex(
 492:         MFFrame.CreateSpillStackObject(getPointerSize(), Align(8)));
 493:     RS->addScavengingFrameIndex(
 494:         MFFrame.CreateSpillStackObject(getPointerSize(), Align(8)));
 495:   }
 496: 
 497:   // If R6 is used as an argument register it is still callee saved. If it in
 498:   // this case is not clobbered (and restored) it should never be marked as
 499:   // killed.
 500:   if (MF.front().isLiveIn(SystemZ::R6D) &&
 501:       ZFI->getRestoreGPRRegs().LowGPR != SystemZ::R6D)
 502:     for (auto &MO : MRI->use_nodbg_operands(SystemZ::R6D))
 503:       MO.setIsKill(false);
 504: }
 505: 
 506: // Add CFI for the new CFA offset.
 507: static void buildCFAOffs(MachineBasicBlock &MBB,
 508:                          MachineBasicBlock::iterator MBBI,
 509:                          const DebugLoc &DL, int Offset,
 510:                          const SystemZInstrInfo *ZII) {
 511:   unsigned CFIIndex = MBB.getParent()->addFrameInst(
 512:     MCCFIInstruction::cfiDefCfaOffset(nullptr, -Offset));
 513:   BuildMI(MBB, MBBI, DL, ZII->get(TargetOpcode::CFI_INSTRUCTION))
 514:     .addCFIIndex(CFIIndex);
 515: }
 516: 
 517: // Add CFI for the new frame location.
 518: static void buildDefCFAReg(MachineBasicBlock &MBB,
 519:                            MachineBasicBlock::iterator MBBI,
 520:                            const DebugLoc &DL, unsigned Reg,
```
- **EN**: The range implements or declares functions including `buildCFAOffs`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `buildCFAOffs`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 521-560 / 第 521-560 行
```cpp
 521:                            const SystemZInstrInfo *ZII) {
 522:   MachineFunction &MF = *MBB.getParent();
 523:   const MCRegisterInfo *MRI = MF.getContext().getRegisterInfo();
 524:   unsigned RegNum = MRI->getDwarfRegNum(Reg, true);
 525:   unsigned CFIIndex = MF.addFrameInst(
 526:                         MCCFIInstruction::createDefCfaRegister(nullptr, RegNum));
 527:   BuildMI(MBB, MBBI, DL, ZII->get(TargetOpcode::CFI_INSTRUCTION))
 528:     .addCFIIndex(CFIIndex);
 529: }
 530: 
 531: void SystemZELFFrameLowering::emitPrologue(MachineFunction &MF,
 532:                                            MachineBasicBlock &MBB) const {
 533:   assert(&MF.front() == &MBB && "Shrink-wrapping not yet supported");
 534:   const SystemZSubtarget &STI = MF.getSubtarget<SystemZSubtarget>();
 535:   const SystemZTargetLowering &TLI = *STI.getTargetLowering();
 536:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
 537:   auto *ZII = STI.getInstrInfo();
 538:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 539:   MachineBasicBlock::iterator MBBI = MBB.begin();
 540:   const MCRegisterInfo *MRI = MF.getContext().getRegisterInfo();
 541:   const std::vector<CalleeSavedInfo> &CSI = MFFrame.getCalleeSavedInfo();
 542:   bool HasFP = hasFP(MF);
 543: 
 544:   // In GHC calling convention C stack space, including the ABI-defined
 545:   // 160-byte base area, is (de)allocated by GHC itself.  This stack space may
 546:   // be used by LLVM as spill slots for the tail recursive GHC functions.  Thus
 547:   // do not allocate stack space here, too.
 548:   if (MF.getFunction().getCallingConv() == CallingConv::GHC) {
 549:     if (MFFrame.getStackSize() > 2048 * sizeof(long)) {
 550:       report_fatal_error(
 551:           "Pre allocated stack space for GHC function is too small");
 552:     }
 553:     if (HasFP) {
 554:       report_fatal_error(
 555:           "In GHC calling convention a frame pointer is not supported");
 556:     }
 557:     MFFrame.setStackSize(MFFrame.getStackSize() + SystemZMC::ELFCallFrameSize);
 558:     return;
 559:   }
 560: 
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZELFFrameLowering::emitPrologue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZELFFrameLowering::emitPrologue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-600 / 第 561-600 行
```cpp
 561:   // Debug location must be unknown since the first debug location is used
 562:   // to determine the end of the prologue.
 563:   DebugLoc DL;
 564:   // Add mcount instrumentation if necessary.
 565:   if (MF.getFunction()
 566:           .getFnAttribute("systemz-instrument-function-entry")
 567:           .getValueAsString() == "mcount") {
 568: 
 569:     // Store return address 8 bytes above stack pointer.
 570:     BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::STG))
 571:         .addReg(SystemZ::R14D)
 572:         .addReg(SystemZ::R15D)
 573:         .addImm(8)
 574:         .addReg(0);
 575: 
 576:     // Call mcount (Regmask from CC AnyReg since mcount preserves all normal
 577:     // argument registers).
 578:     const uint32_t *Mask = MF.getSubtarget<SystemZSubtarget>()
 579:                                .getSpecialRegisters()
 580:                                ->getCallPreservedMask(MF, CallingConv::AnyReg);
 581:     BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::CallBRASL))
 582:         .addExternalSymbol("mcount")
 583:         .addRegMask(Mask);
 584: 
 585:     // Reload return address from 8 bytes above stack pointer.
 586:     BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::LG))
 587:         .addReg(SystemZ::R14D, RegState::Define)
 588:         .addReg(SystemZ::R15D)
 589:         .addImm(8)
 590:         .addReg(0);
 591:   }
 592: 
 593:   // The current offset of the stack pointer from the CFA.
 594:   int64_t SPOffsetFromCFA = -SystemZMC::ELFCFAOffsetFromInitialSP;
 595: 
 596:   if (ZFI->getSpillGPRRegs().LowGPR) {
 597:     // Skip over the GPR saves.
 598:     if (MBBI != MBB.end() && MBBI->getOpcode() == SystemZ::STMG)
 599:       ++MBBI;
 600:     else
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 601-640 / 第 601-640 行
```cpp
 601:       llvm_unreachable("Couldn't skip over GPR saves");
 602: 
 603:     // Add CFI for the GPR saves.
 604:     for (auto &Save : CSI) {
 605:       MCRegister Reg = Save.getReg();
 606:       if (SystemZ::GR64BitRegClass.contains(Reg)) {
 607:         int FI = Save.getFrameIdx();
 608:         int64_t Offset = MFFrame.getObjectOffset(FI);
 609:         unsigned CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 610:             nullptr, MRI->getDwarfRegNum(Reg, true), Offset));
 611:         BuildMI(MBB, MBBI, DL, ZII->get(TargetOpcode::CFI_INSTRUCTION))
 612:             .addCFIIndex(CFIIndex);
 613:       }
 614:     }
 615:   }
 616: 
 617:   uint64_t StackSize = MFFrame.getStackSize();
 618:   // We need to allocate the ABI-defined 160-byte base area whenever
 619:   // we allocate stack space for our own use and whenever we call another
 620:   // function.
 621:   bool HasStackObject = false;
 622:   for (unsigned i = 0, e = MFFrame.getObjectIndexEnd(); i != e; ++i)
 623:     if (!MFFrame.isDeadObjectIndex(i)) {
 624:       HasStackObject = true;
 625:       break;
 626:     }
 627:   if (HasStackObject || MFFrame.hasCalls())
 628:     StackSize += SystemZMC::ELFCallFrameSize;
 629:   // Don't allocate the incoming reg save area.
 630:   StackSize = StackSize > SystemZMC::ELFCallFrameSize
 631:                   ? StackSize - SystemZMC::ELFCallFrameSize
 632:                   : 0;
 633:   MFFrame.setStackSize(StackSize);
 634: 
 635:   if (StackSize) {
 636:     // Allocate StackSize bytes.
 637:     int64_t Delta = -int64_t(StackSize);
 638:     const unsigned ProbeSize = TLI.getStackProbeSize(MF);
 639:     bool FreeProbe = (ZFI->getSpillGPRRegs().GPROffset &&
 640:            (ZFI->getSpillGPRRegs().GPROffset + StackSize) < ProbeSize);
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 641-680 / 第 641-680 行
```cpp
 641:     if (!FreeProbe &&
 642:         MF.getSubtarget().getTargetLowering()->hasInlineStackProbe(MF)) {
 643:       // Stack probing may involve looping, but splitting the prologue block
 644:       // is not possible at this point since it would invalidate the
 645:       // SaveBlocks / RestoreBlocks sets of PEI in the single block function
 646:       // case. Build a pseudo to be handled later by inlineStackProbe().
 647:       BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::PROBED_STACKALLOC))
 648:         .addImm(StackSize);
 649:     }
 650:     else {
 651:       bool StoreBackchain = MF.getSubtarget<SystemZSubtarget>().hasBackChain();
 652:       // If we need backchain, save current stack pointer.  R1 is free at
 653:       // this point.
 654:       if (StoreBackchain)
 655:         BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::LGR))
 656:           .addReg(SystemZ::R1D, RegState::Define).addReg(SystemZ::R15D);
 657:       emitIncrement(MBB, MBBI, DL, SystemZ::R15D, Delta, ZII);
 658:       buildCFAOffs(MBB, MBBI, DL, SPOffsetFromCFA + Delta, ZII);
 659:       if (StoreBackchain)
 660:         BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::STG))
 661:           .addReg(SystemZ::R1D, RegState::Kill).addReg(SystemZ::R15D)
 662:           .addImm(getBackchainOffset(MF)).addReg(0);
 663:     }
 664:     SPOffsetFromCFA += Delta;
 665:   }
 666: 
 667:   if (HasFP) {
 668:     // Copy the base of the frame to R11.
 669:     BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::LGR), SystemZ::R11D)
 670:       .addReg(SystemZ::R15D);
 671: 
 672:     // Add CFI for the new frame location.
 673:     buildDefCFAReg(MBB, MBBI, DL, SystemZ::R11D, ZII);
 674: 
 675:     // Mark the FramePtr as live at the beginning of every block except
 676:     // the entry block.  (We'll have marked R11 as live on entry when
 677:     // saving the GPRs.)
 678:     for (MachineBasicBlock &MBBJ : llvm::drop_begin(MF))
 679:       MBBJ.addLiveIn(SystemZ::R11D);
 680:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 681-720 / 第 681-720 行
```cpp
 681: 
 682:   // Skip over the FPR/VR saves.
 683:   SmallVector<unsigned, 8> CFIIndexes;
 684:   for (auto &Save : CSI) {
 685:     MCRegister Reg = Save.getReg();
 686:     if (SystemZ::FP64BitRegClass.contains(Reg)) {
 687:       if (MBBI != MBB.end() &&
 688:           (MBBI->getOpcode() == SystemZ::STD ||
 689:            MBBI->getOpcode() == SystemZ::STDY))
 690:         ++MBBI;
 691:       else
 692:         llvm_unreachable("Couldn't skip over FPR save");
 693:     } else if (SystemZ::VR128BitRegClass.contains(Reg)) {
 694:       if (MBBI != MBB.end() &&
 695:           MBBI->getOpcode() == SystemZ::VST)
 696:         ++MBBI;
 697:       else
 698:         llvm_unreachable("Couldn't skip over VR save");
 699:     } else
 700:       continue;
 701: 
 702:     // Add CFI for the this save.
 703:     unsigned DwarfReg = MRI->getDwarfRegNum(Reg, true);
 704:     Register IgnoredFrameReg;
 705:     int64_t Offset =
 706:         getFrameIndexReference(MF, Save.getFrameIdx(), IgnoredFrameReg)
 707:             .getFixed();
 708: 
 709:     unsigned CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 710:           nullptr, DwarfReg, SPOffsetFromCFA + Offset));
 711:     CFIIndexes.push_back(CFIIndex);
 712:   }
 713:   // Complete the CFI for the FPR/VR saves, modelling them as taking effect
 714:   // after the last save.
 715:   for (auto CFIIndex : CFIIndexes) {
 716:     BuildMI(MBB, MBBI, DL, ZII->get(TargetOpcode::CFI_INSTRUCTION))
 717:         .addCFIIndex(CFIIndex);
 718:   }
 719: }
 720: 
```
- **EN**: The range implements or declares functions including `getFrameIndexReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `getFrameIndexReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 721-760 / 第 721-760 行
```cpp
 721: void SystemZELFFrameLowering::emitEpilogue(MachineFunction &MF,
 722:                                            MachineBasicBlock &MBB) const {
 723:   MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
 724:   auto *ZII =
 725:       static_cast<const SystemZInstrInfo *>(MF.getSubtarget().getInstrInfo());
 726:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 727:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
 728: 
 729:   // See SystemZELFFrameLowering::emitPrologue
 730:   if (MF.getFunction().getCallingConv() == CallingConv::GHC)
 731:     return;
 732: 
 733:   // Skip the return instruction.
 734:   assert(MBBI->isReturn() && "Can only insert epilogue into returning blocks");
 735: 
 736:   uint64_t StackSize = MFFrame.getStackSize();
 737:   if (ZFI->getRestoreGPRRegs().LowGPR) {
 738:     --MBBI;
 739:     unsigned Opcode = MBBI->getOpcode();
 740:     if (Opcode != SystemZ::LMG)
 741:       llvm_unreachable("Expected to see callee-save register restore code");
 742: 
 743:     unsigned AddrOpNo = 2;
 744:     DebugLoc DL = MBBI->getDebugLoc();
 745:     uint64_t Offset = StackSize + MBBI->getOperand(AddrOpNo + 1).getImm();
 746:     unsigned NewOpcode = ZII->getOpcodeForOffset(Opcode, Offset);
 747: 
 748:     // If the offset is too large, use the largest stack-aligned offset
 749:     // and add the rest to the base register (the stack or frame pointer).
 750:     if (!NewOpcode) {
 751:       uint64_t NumBytes = Offset - 0x7fff8;
 752:       emitIncrement(MBB, MBBI, DL, MBBI->getOperand(AddrOpNo).getReg(),
 753:                     NumBytes, ZII);
 754:       Offset -= NumBytes;
 755:       NewOpcode = ZII->getOpcodeForOffset(Opcode, Offset);
 756:       assert(NewOpcode && "No restore instruction available");
 757:     }
 758: 
 759:     MBBI->setDesc(ZII->get(NewOpcode));
 760:     MBBI->getOperand(AddrOpNo + 1).ChangeToImmediate(Offset);
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::emitEpilogue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::emitEpilogue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 761-800 / 第 761-800 行
```cpp
 761:   } else if (StackSize) {
 762:     DebugLoc DL = MBBI->getDebugLoc();
 763:     emitIncrement(MBB, MBBI, DL, SystemZ::R15D, StackSize, ZII);
 764:   }
 765: }
 766: 
 767: void SystemZELFFrameLowering::inlineStackProbe(
 768:     MachineFunction &MF, MachineBasicBlock &PrologMBB) const {
 769:   auto *ZII =
 770:     static_cast<const SystemZInstrInfo *>(MF.getSubtarget().getInstrInfo());
 771:   const SystemZSubtarget &STI = MF.getSubtarget<SystemZSubtarget>();
 772:   const SystemZTargetLowering &TLI = *STI.getTargetLowering();
 773: 
 774:   MachineInstr *StackAllocMI = nullptr;
 775:   for (MachineInstr &MI : PrologMBB)
 776:     if (MI.getOpcode() == SystemZ::PROBED_STACKALLOC) {
 777:       StackAllocMI = &MI;
 778:       break;
 779:     }
 780:   if (StackAllocMI == nullptr)
 781:     return;
 782:   uint64_t StackSize = StackAllocMI->getOperand(0).getImm();
 783:   const unsigned ProbeSize = TLI.getStackProbeSize(MF);
 784:   uint64_t NumFullBlocks = StackSize / ProbeSize;
 785:   uint64_t Residual = StackSize % ProbeSize;
 786:   int64_t SPOffsetFromCFA = -SystemZMC::ELFCFAOffsetFromInitialSP;
 787:   MachineBasicBlock *MBB = &PrologMBB;
 788:   MachineBasicBlock::iterator MBBI = StackAllocMI;
 789:   const DebugLoc DL = StackAllocMI->getDebugLoc();
 790: 
 791:   // Allocate a block of Size bytes on the stack and probe it.
 792:   auto allocateAndProbe = [&](MachineBasicBlock &InsMBB,
 793:                               MachineBasicBlock::iterator InsPt, unsigned Size,
 794:                               bool EmitCFI) -> void {
 795:     emitIncrement(InsMBB, InsPt, DL, SystemZ::R15D, -int64_t(Size), ZII);
 796:     if (EmitCFI) {
 797:       SPOffsetFromCFA -= Size;
 798:       buildCFAOffs(InsMBB, InsPt, DL, SPOffsetFromCFA, ZII);
 799:     }
 800:     // Probe by means of a volatile compare.
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::inlineStackProbe`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::inlineStackProbe` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 801-840 / 第 801-840 行
```cpp
 801:     MachineMemOperand *MMO = MF.getMachineMemOperand(MachinePointerInfo(),
 802:       MachineMemOperand::MOVolatile | MachineMemOperand::MOLoad, 8, Align(1));
 803:     BuildMI(InsMBB, InsPt, DL, ZII->get(SystemZ::CG))
 804:       .addReg(SystemZ::R0D, RegState::Undef)
 805:       .addReg(SystemZ::R15D).addImm(Size - 8).addReg(0)
 806:       .addMemOperand(MMO);
 807:   };
 808: 
 809:   bool StoreBackchain = MF.getSubtarget<SystemZSubtarget>().hasBackChain();
 810:   if (StoreBackchain)
 811:     BuildMI(*MBB, MBBI, DL, ZII->get(SystemZ::LGR))
 812:       .addReg(SystemZ::R1D, RegState::Define).addReg(SystemZ::R15D);
 813: 
 814:   MachineBasicBlock *DoneMBB = nullptr;
 815:   MachineBasicBlock *LoopMBB = nullptr;
 816:   if (NumFullBlocks < 3) {
 817:     // Emit unrolled probe statements.
 818:     for (unsigned int i = 0; i < NumFullBlocks; i++)
 819:       allocateAndProbe(*MBB, MBBI, ProbeSize, true/*EmitCFI*/);
 820:   } else {
 821:     // Emit a loop probing the pages.
 822:     uint64_t LoopAlloc = ProbeSize * NumFullBlocks;
 823:     SPOffsetFromCFA -= LoopAlloc;
 824: 
 825:     // Use R0D to hold the exit value.
 826:     BuildMI(*MBB, MBBI, DL, ZII->get(SystemZ::LGR), SystemZ::R0D)
 827:       .addReg(SystemZ::R15D);
 828:     buildDefCFAReg(*MBB, MBBI, DL, SystemZ::R0D, ZII);
 829:     emitIncrement(*MBB, MBBI, DL, SystemZ::R0D, -int64_t(LoopAlloc), ZII);
 830:     buildCFAOffs(*MBB, MBBI, DL, -int64_t(SystemZMC::ELFCallFrameSize + LoopAlloc),
 831:                  ZII);
 832: 
 833:     DoneMBB = SystemZ::splitBlockBefore(MBBI, MBB);
 834:     LoopMBB = SystemZ::emitBlockAfter(MBB);
 835:     MBB->addSuccessor(LoopMBB);
 836:     LoopMBB->addSuccessor(LoopMBB);
 837:     LoopMBB->addSuccessor(DoneMBB);
 838: 
 839:     MBB = LoopMBB;
 840:     allocateAndProbe(*MBB, MBB->end(), ProbeSize, false/*EmitCFI*/);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 841-880 / 第 841-880 行
```cpp
 841:     BuildMI(*MBB, MBB->end(), DL, ZII->get(SystemZ::CLGR))
 842:       .addReg(SystemZ::R15D).addReg(SystemZ::R0D);
 843:     BuildMI(*MBB, MBB->end(), DL, ZII->get(SystemZ::BRC))
 844:       .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_GT).addMBB(MBB);
 845: 
 846:     MBB = DoneMBB;
 847:     MBBI = DoneMBB->begin();
 848:     buildDefCFAReg(*MBB, MBBI, DL, SystemZ::R15D, ZII);
 849:   }
 850: 
 851:   if (Residual)
 852:     allocateAndProbe(*MBB, MBBI, Residual, true/*EmitCFI*/);
 853: 
 854:   if (StoreBackchain)
 855:     BuildMI(*MBB, MBBI, DL, ZII->get(SystemZ::STG))
 856:       .addReg(SystemZ::R1D, RegState::Kill).addReg(SystemZ::R15D)
 857:       .addImm(getBackchainOffset(MF)).addReg(0);
 858: 
 859:   StackAllocMI->eraseFromParent();
 860:   if (DoneMBB != nullptr) {
 861:     // Compute the live-in lists for the new blocks.
 862:     fullyRecomputeLiveIns({DoneMBB, LoopMBB});
 863:   }
 864: }
 865: 
 866: bool SystemZELFFrameLowering::hasFPImpl(const MachineFunction &MF) const {
 867:   return (MF.getTarget().Options.DisableFramePointerElim(MF) ||
 868:           MF.getFrameInfo().hasVarSizedObjects());
 869: }
 870: 
 871: StackOffset SystemZELFFrameLowering::getFrameIndexReference(
 872:     const MachineFunction &MF, int FI, Register &FrameReg) const {
 873:   // Our incoming SP is actually SystemZMC::ELFCallFrameSize below the CFA, so
 874:   // add that difference here.
 875:   StackOffset Offset =
 876:       TargetFrameLowering::getFrameIndexReference(MF, FI, FrameReg);
 877:   return Offset + StackOffset::getFixed(SystemZMC::ELFCallFrameSize);
 878: }
 879: 
 880: unsigned SystemZELFFrameLowering::getRegSpillOffset(MachineFunction &MF,
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `SystemZELFFrameLowering::hasFPImpl`, `SystemZELFFrameLowering::getFrameIndexReference`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `SystemZELFFrameLowering::hasFPImpl`, `SystemZELFFrameLowering::getFrameIndexReference` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 881-920 / 第 881-920 行
```cpp
 881:                                                     Register Reg) const {
 882:   bool IsVarArg = MF.getFunction().isVarArg();
 883:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
 884:   bool BackChain = Subtarget.hasBackChain();
 885:   bool SoftFloat = Subtarget.hasSoftFloat();
 886:   unsigned Offset = RegSpillOffsets[Reg];
 887:   if (usePackedStack(MF) && !(IsVarArg && !SoftFloat)) {
 888:     if (SystemZ::GR64BitRegClass.contains(Reg))
 889:       // Put all GPRs at the top of the Register save area with packed
 890:       // stack. Make room for the backchain if needed.
 891:       Offset += BackChain ? 24 : 32;
 892:     else
 893:       Offset = 0;
 894:   }
 895:   return Offset;
 896: }
 897: 
 898: int SystemZELFFrameLowering::getOrCreateFramePointerSaveIndex(
 899:     MachineFunction &MF) const {
 900:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 901:   int FI = ZFI->getFramePointerSaveIndex();
 902:   if (!FI) {
 903:     MachineFrameInfo &MFFrame = MF.getFrameInfo();
 904:     int Offset = getBackchainOffset(MF) - SystemZMC::ELFCallFrameSize;
 905:     FI = MFFrame.CreateFixedObject(getPointerSize(), Offset, false);
 906:     ZFI->setFramePointerSaveIndex(FI);
 907:   }
 908:   return FI;
 909: }
 910: 
 911: bool SystemZELFFrameLowering::usePackedStack(MachineFunction &MF) const {
 912:   bool HasPackedStackAttr = MF.getFunction().hasFnAttribute("packed-stack");
 913:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
 914:   bool BackChain = Subtarget.hasBackChain();
 915:   bool SoftFloat = Subtarget.hasSoftFloat();
 916:   if (HasPackedStackAttr && BackChain && !SoftFloat)
 917:     report_fatal_error("packed-stack + backchain + hard-float is unsupported.");
 918:   bool CallConv = MF.getFunction().getCallingConv() != CallingConv::GHC;
 919:   return HasPackedStackAttr && CallConv;
 920: }
```
- **EN**: The range implements or declares functions including `SystemZELFFrameLowering::getOrCreateFramePointerSaveIndex`, `SystemZELFFrameLowering::usePackedStack`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZELFFrameLowering::getOrCreateFramePointerSaveIndex`, `SystemZELFFrameLowering::usePackedStack` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 921-960 / 第 921-960 行
```cpp
 921: 
 922: SystemZXPLINKFrameLowering::SystemZXPLINKFrameLowering(unsigned PointerSize)
 923:     : SystemZFrameLowering(TargetFrameLowering::StackGrowsDown, Align(32), 0,
 924:                            Align(32), /* StackRealignable */ false,
 925:                            PointerSize),
 926:       RegSpillOffsets(-1) {
 927: 
 928:   // Create a mapping from register number to save slot offset.
 929:   // These offsets are relative to the start of the local are area.
 930:   RegSpillOffsets.grow(SystemZ::NUM_TARGET_REGS);
 931:   for (const auto &Entry : XPLINKSpillOffsetTable)
 932:     RegSpillOffsets[Entry.Reg] = Entry.Offset;
 933: }
 934: 
 935: int SystemZXPLINKFrameLowering::getOrCreateFramePointerSaveIndex(
 936:     MachineFunction &MF) const {
 937:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
 938:   int FI = ZFI->getFramePointerSaveIndex();
 939:   if (!FI) {
 940:     MachineFrameInfo &MFFrame = MF.getFrameInfo();
 941:     FI = MFFrame.CreateFixedObject(getPointerSize(), 0, false);
 942:     MFFrame.setStackID(FI, TargetStackID::NoAlloc);
 943:     ZFI->setFramePointerSaveIndex(FI);
 944:   }
 945:   return FI;
 946: }
 947: 
 948: // Checks if the function is a potential candidate for being a XPLeaf routine.
 949: static bool isXPLeafCandidate(const MachineFunction &MF) {
 950:   const MachineFrameInfo &MFFrame = MF.getFrameInfo();
 951:   const MachineRegisterInfo &MRI = MF.getRegInfo();
 952:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
 953:   auto *Regs =
 954:       static_cast<SystemZXPLINK64Registers *>(Subtarget.getSpecialRegisters());
 955: 
 956:   // If function calls other functions including alloca, then it is not a XPLeaf
 957:   // routine.
 958:   if (MFFrame.hasCalls())
 959:     return false;
 960: 
```
- **EN**: The range implements or declares functions including `SystemZFrameLowering`, `SystemZXPLINKFrameLowering::getOrCreateFramePointerSaveIndex`, `isXPLeafCandidate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZFrameLowering`, `SystemZXPLINKFrameLowering::getOrCreateFramePointerSaveIndex`, `isXPLeafCandidate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:   // If the function has var Sized Objects, then it is not a XPLeaf routine.
 962:   if (MFFrame.hasVarSizedObjects())
 963:     return false;
 964: 
 965:   // If the function adjusts the stack, then it is not a XPLeaf routine.
 966:   if (MFFrame.adjustsStack())
 967:     return false;
 968: 
 969:   // If function modifies the stack pointer register, then it is not a XPLeaf
 970:   // routine.
 971:   if (MRI.isPhysRegModified(Regs->getStackPointerRegister()))
 972:     return false;
 973: 
 974:   // If function modifies the ADA register, then it is not a XPLeaf routine.
 975:   if (MRI.isPhysRegModified(Regs->getAddressOfCalleeRegister()))
 976:     return false;
 977: 
 978:   // If function modifies the return address register, then it is not a XPLeaf
 979:   // routine.
 980:   if (MRI.isPhysRegModified(Regs->getReturnFunctionAddressRegister()))
 981:     return false;
 982: 
 983:   // If the backchain pointer should be stored, then it is not a XPLeaf routine.
 984:   if (MF.getSubtarget<SystemZSubtarget>().hasBackChain())
 985:     return false;
 986: 
 987:   // If function acquires its own stack frame, then it is not a XPLeaf routine.
 988:   // At the time this function is called, only slots for local variables are
 989:   // allocated, so this is a very rough estimate.
 990:   if (MFFrame.estimateStackSize(MF) > 0)
 991:     return false;
 992: 
 993:   return true;
 994: }
 995: 
 996: bool SystemZXPLINKFrameLowering::assignCalleeSavedSpillSlots(
 997:     MachineFunction &MF, const TargetRegisterInfo *TRI,
 998:     std::vector<CalleeSavedInfo> &CSI) const {
 999:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
1000:   SystemZMachineFunctionInfo *MFI = MF.getInfo<SystemZMachineFunctionInfo>();
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::assignCalleeSavedSpillSlots`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::assignCalleeSavedSpillSlots` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1002:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1003:   auto &GRRegClass = SystemZ::GR64BitRegClass;
1004: 
1005:   // At this point, the result of isXPLeafCandidate() is not accurate because
1006:   // the size of the save area has not yet been determined. If
1007:   // isXPLeafCandidate() indicates a potential leaf function, and there are no
1008:   // callee-save registers, then it is indeed a leaf function, and we can early
1009:   // exit.
1010:   // TODO: It is possible for leaf functions to use callee-saved registers.
1011:   // It can use the 0-2k range between R4 and the caller's stack frame without
1012:   // acquiring its own stack frame.
1013:   bool IsLeaf = CSI.empty() && isXPLeafCandidate(MF);
1014:   if (IsLeaf)
1015:     return true;
1016: 
1017:   // For non-leaf functions:
1018:   // - the address of callee (entry point) register R6 must be saved
1019:   CSI.push_back(CalleeSavedInfo(Regs.getAddressOfCalleeRegister()));
1020:   CSI.back().setRestored(false);
1021: 
1022:   // The return address register R7 must be saved and restored.
1023:   CSI.push_back(CalleeSavedInfo(Regs.getReturnFunctionAddressRegister()));
1024: 
1025:   // If the function needs a frame pointer, or if the backchain pointer should
1026:   // be stored, then save the stack pointer register R4.
1027:   if (hasFP(MF) || Subtarget.hasBackChain())
1028:     CSI.push_back(CalleeSavedInfo(Regs.getStackPointerRegister()));
1029: 
1030:   // If this function has an associated personality function then the
1031:   // environment register R5 must be saved in the DSA.
1032:   if (!MF.getLandingPads().empty()) {
1033:     CSI.push_back(CalleeSavedInfo(Regs.getADARegister()));
1034:     CSI.back().setRestored(false);
1035:   }
1036: 
1037:   // Scan the call-saved GPRs and find the bounds of the register spill area.
1038:   Register LowRestoreGPR = 0;
1039:   int LowRestoreOffset = INT32_MAX;
1040:   Register LowSpillGPR = 0;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041:   int LowSpillOffset = INT32_MAX;
1042:   Register HighGPR = 0;
1043:   int HighOffset = -1;
1044: 
1045:   // Query index of the saved frame pointer.
1046:   int FPSI = MFI->getFramePointerSaveIndex();
1047: 
1048:   for (auto &CS : CSI) {
1049:     MCRegister Reg = CS.getReg();
1050:     int Offset = RegSpillOffsets[Reg];
1051:     if (Offset >= 0) {
1052:       if (GRRegClass.contains(Reg)) {
1053:         if (LowSpillOffset > Offset) {
1054:           LowSpillOffset = Offset;
1055:           LowSpillGPR = Reg;
1056:         }
1057:         if (CS.isRestored() && LowRestoreOffset > Offset) {
1058:           LowRestoreOffset = Offset;
1059:           LowRestoreGPR = Reg;
1060:         }
1061: 
1062:         if (Offset > HighOffset) {
1063:           HighOffset = Offset;
1064:           HighGPR = Reg;
1065:         }
1066:         // Non-volatile GPRs are saved in the dedicated register save area at
1067:         // the bottom of the stack and are not truly part of the "normal" stack
1068:         // frame. Mark the frame index as NoAlloc to indicate it as such.
1069:         unsigned RegSize = getPointerSize();
1070:         int FrameIdx =
1071:             (FPSI && Offset == 0)
1072:                 ? FPSI
1073:                 : MFFrame.CreateFixedSpillStackObject(RegSize, Offset);
1074:         CS.setFrameIdx(FrameIdx);
1075:         MFFrame.setStackID(FrameIdx, TargetStackID::NoAlloc);
1076:       }
1077:     } else {
1078:       MCRegister Reg = CS.getReg();
1079:       const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(Reg);
1080:       Align Alignment = TRI->getSpillAlign(*RC);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081:       unsigned Size = TRI->getSpillSize(*RC);
1082:       Alignment = std::min(Alignment, getStackAlign());
1083:       int FrameIdx = MFFrame.CreateStackObject(Size, Alignment, true);
1084:       CS.setFrameIdx(FrameIdx);
1085:     }
1086:   }
1087: 
1088:   // Save the range of call-saved registers, for use by the
1089:   // prologue/epilogue inserters.
1090:   if (LowRestoreGPR)
1091:     MFI->setRestoreGPRRegs(LowRestoreGPR, HighGPR, LowRestoreOffset);
1092: 
1093:   // Save the range of call-saved registers, for use by the epilogue inserter.
1094:   assert(LowSpillGPR && "Expected registers to spill");
1095:   MFI->setSpillGPRRegs(LowSpillGPR, HighGPR, LowSpillOffset);
1096: 
1097:   return true;
1098: }
1099: 
1100: void SystemZXPLINKFrameLowering::determineCalleeSaves(MachineFunction &MF,
1101:                                                       BitVector &SavedRegs,
1102:                                                       RegScavenger *RS) const {
1103:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
1104: 
1105:   bool HasFP = hasFP(MF);
1106:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1107:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1108: 
1109:   // If the function requires a frame pointer, record that the hard
1110:   // frame pointer will be clobbered.
1111:   if (HasFP)
1112:     SavedRegs.set(Regs.getFramePointerRegister());
1113: }
1114: 
1115: bool SystemZXPLINKFrameLowering::spillCalleeSavedRegisters(
1116:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
1117:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
1118:   if (CSI.empty())
1119:     return true;
1120: 
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::determineCalleeSaves`, `SystemZXPLINKFrameLowering::spillCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::determineCalleeSaves`, `SystemZXPLINKFrameLowering::spillCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121:   MachineFunction &MF = *MBB.getParent();
1122:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
1123:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1124:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
1125:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1126:   SystemZ::GPRRegs SpillGPRs = ZFI->getSpillGPRRegs();
1127:   DebugLoc DL;
1128: 
1129:   // Save GPRs
1130:   if (SpillGPRs.LowGPR) {
1131:     assert(SpillGPRs.LowGPR != SpillGPRs.HighGPR &&
1132:            "Should be saving multiple registers");
1133: 
1134:     // Build an STM/STMG instruction.
1135:     MachineInstrBuilder MIB = BuildMI(MBB, MBBI, DL, TII->get(SystemZ::STMG));
1136: 
1137:     // Add the explicit register operands.
1138:     addSavedGPR(MBB, MIB, SpillGPRs.LowGPR, false);
1139:     addSavedGPR(MBB, MIB, SpillGPRs.HighGPR, false);
1140: 
1141:     // Add the address r4
1142:     MIB.addReg(Regs.getStackPointerRegister());
1143: 
1144:     // Add the partial offset
1145:     // We cannot add the actual offset as, at the stack is not finalized
1146:     MIB.addImm(SpillGPRs.GPROffset);
1147: 
1148:     // Make sure all call-saved GPRs are included as operands and are
1149:     // marked as live on entry.
1150:     auto &GRRegClass = SystemZ::GR64BitRegClass;
1151:     for (const CalleeSavedInfo &I : CSI) {
1152:       MCRegister Reg = I.getReg();
1153:       if (GRRegClass.contains(Reg))
1154:         addSavedGPR(MBB, MIB, Reg, true);
1155:     }
1156:   }
1157: 
1158:   // Spill FPRs to the stack in the normal TargetInstrInfo way
1159:   // Registers in CSI are in inverted order so registers with large
1160:   // numbers will be assigned to high address.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:   // Reverse the order at spilling and restoring so instructions on
1162:   // registers with small numbers are emitted first.
1163:   for (const CalleeSavedInfo &I : llvm::reverse(CSI)) {
1164:     MCRegister Reg = I.getReg();
1165:     if (SystemZ::FP64BitRegClass.contains(Reg)) {
1166:       MBB.addLiveIn(Reg);
1167:       TII->storeRegToStackSlot(MBB, MBBI, Reg, true, I.getFrameIdx(),
1168:                                &SystemZ::FP64BitRegClass, Register());
1169:     }
1170:     if (SystemZ::VR128BitRegClass.contains(Reg)) {
1171:       MBB.addLiveIn(Reg);
1172:       TII->storeRegToStackSlot(MBB, MBBI, Reg, true, I.getFrameIdx(),
1173:                                &SystemZ::VR128BitRegClass, Register());
1174:     }
1175:   }
1176: 
1177:   return true;
1178: }
1179: 
1180: bool SystemZXPLINKFrameLowering::restoreCalleeSavedRegisters(
1181:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
1182:     MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
1183: 
1184:   if (CSI.empty())
1185:     return false;
1186: 
1187:   MachineFunction &MF = *MBB.getParent();
1188:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
1189:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1190:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
1191:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1192: 
1193:   DebugLoc DL = MBBI != MBB.end() ? MBBI->getDebugLoc() : DebugLoc();
1194: 
1195:   // Restore FPRs in the normal TargetInstrInfo way.
1196:   for (const CalleeSavedInfo &I : llvm::reverse(CSI)) {
1197:     MCRegister Reg = I.getReg();
1198:     if (SystemZ::FP64BitRegClass.contains(Reg))
1199:       TII->loadRegFromStackSlot(MBB, MBBI, Reg, I.getFrameIdx(),
1200:                                 &SystemZ::FP64BitRegClass, Register());
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::restoreCalleeSavedRegisters`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::restoreCalleeSavedRegisters` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:     if (SystemZ::VR128BitRegClass.contains(Reg))
1202:       TII->loadRegFromStackSlot(MBB, MBBI, Reg, I.getFrameIdx(),
1203:                                 &SystemZ::VR128BitRegClass, Register());
1204:   }
1205: 
1206:   // Restore call-saved GPRs (but not call-clobbered varargs, which at
1207:   // this point might hold return values).
1208:   SystemZ::GPRRegs RestoreGPRs = ZFI->getRestoreGPRRegs();
1209:   if (RestoreGPRs.LowGPR) {
1210:     assert(isInt<20>(Regs.getStackPointerBias() + RestoreGPRs.GPROffset));
1211:     if (RestoreGPRs.LowGPR == RestoreGPRs.HighGPR)
1212:       // Build an LG/L instruction.
1213:       BuildMI(MBB, MBBI, DL, TII->get(SystemZ::LG), RestoreGPRs.LowGPR)
1214:           .addReg(Regs.getStackPointerRegister())
1215:           .addImm(Regs.getStackPointerBias() + RestoreGPRs.GPROffset)
1216:           .addReg(0);
1217:     else {
1218:       // Build an LMG/LM instruction.
1219:       MachineInstrBuilder MIB = BuildMI(MBB, MBBI, DL, TII->get(SystemZ::LMG));
1220: 
1221:       // Add the explicit register operands.
1222:       MIB.addReg(RestoreGPRs.LowGPR, RegState::Define);
1223:       MIB.addReg(RestoreGPRs.HighGPR, RegState::Define);
1224: 
1225:       // Add the address.
1226:       MIB.addReg(Regs.getStackPointerRegister());
1227:       MIB.addImm(Regs.getStackPointerBias() + RestoreGPRs.GPROffset);
1228: 
1229:       // Do a second scan adding regs as being defined by instruction
1230:       for (const CalleeSavedInfo &I : CSI) {
1231:         MCRegister Reg = I.getReg();
1232:         if (Reg > RestoreGPRs.LowGPR && Reg < RestoreGPRs.HighGPR)
1233:           MIB.addReg(Reg, RegState::ImplicitDefine);
1234:       }
1235:     }
1236:   }
1237: 
1238:   return true;
1239: }
1240: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241: void SystemZXPLINKFrameLowering::emitPrologue(MachineFunction &MF,
1242:                                               MachineBasicBlock &MBB) const {
1243:   assert(&MF.front() == &MBB && "Shrink-wrapping not yet supported");
1244:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1245:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
1246:   MachineBasicBlock::iterator MBBI = MBB.begin();
1247:   auto *ZII = Subtarget.getInstrInfo();
1248:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1249:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
1250:   MachineInstr *StoreInstr = nullptr;
1251: 
1252:   determineFrameLayout(MF);
1253: 
1254:   bool HasFP = hasFP(MF);
1255:   // Debug location must be unknown since the first debug location is used
1256:   // to determine the end of the prologue.
1257:   DebugLoc DL;
1258:   uint64_t Offset = 0;
1259: 
1260:   const uint64_t StackSize = MFFrame.getStackSize();
1261: 
1262:   if (ZFI->getSpillGPRRegs().LowGPR) {
1263:     // Skip over the GPR saves.
1264:     if ((MBBI != MBB.end()) && ((MBBI->getOpcode() == SystemZ::STMG))) {
1265:       const int Operand = 3;
1266:       // Now we can set the offset for the operation, since now the Stack
1267:       // has been finalized.
1268:       Offset = Regs.getStackPointerBias() + MBBI->getOperand(Operand).getImm();
1269:       // Maximum displacement for STMG instruction.
1270:       if (isInt<20>(Offset - StackSize))
1271:         Offset -= StackSize;
1272:       else
1273:         StoreInstr = &*MBBI;
1274:       MBBI->getOperand(Operand).setImm(Offset);
1275:       ++MBBI;
1276:     } else
1277:       llvm_unreachable("Couldn't skip over GPR saves");
1278:   }
1279: 
1280:   if (StackSize) {
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::emitPrologue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::emitPrologue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281:     MachineBasicBlock::iterator InsertPt = StoreInstr ? StoreInstr : MBBI;
1282:     // Allocate StackSize bytes.
1283:     int64_t Delta = -int64_t(StackSize);
1284: 
1285:     // In case the STM(G) instruction also stores SP (R4), but the displacement
1286:     // is too large, the SP register is manipulated first before storing,
1287:     // resulting in the wrong value stored and retrieved later. In this case, we
1288:     // need to temporarily save the value of SP, and store it later to memory.
1289:     if (StoreInstr && HasFP) {
1290:       // Insert LR r0,r4 before STMG instruction.
1291:       BuildMI(MBB, InsertPt, DL, ZII->get(SystemZ::LGR))
1292:           .addReg(SystemZ::R0D, RegState::Define)
1293:           .addReg(SystemZ::R4D);
1294:       // Insert ST r0,xxx(,r4) after STMG instruction.
1295:       BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::STG))
1296:           .addReg(SystemZ::R0D, RegState::Kill)
1297:           .addReg(SystemZ::R4D)
1298:           .addImm(Offset)
1299:           .addReg(0);
1300:     }
1301: 
1302:     emitIncrement(MBB, InsertPt, DL, Regs.getStackPointerRegister(), Delta,
1303:                   ZII);
1304: 
1305:     // If the requested stack size is larger than the guard page, then we need
1306:     // to check if we need to call the stack extender. This requires adding a
1307:     // conditional branch, but splitting the prologue block is not possible at
1308:     // this point since it would invalidate the SaveBlocks / RestoreBlocks sets
1309:     // of PEI in the single block function case. Build a pseudo to be handled
1310:     // later by inlineStackProbe().
1311:     const uint64_t GuardPageSize = 1024 * 1024;
1312:     if (StackSize > GuardPageSize) {
1313:       assert(StoreInstr && "Wrong insertion point");
1314:       BuildMI(MBB, InsertPt, DL, ZII->get(SystemZ::XPLINK_STACKALLOC));
1315:     }
1316:   }
1317: 
1318:   if (HasFP) {
1319:     // Copy the base of the frame to Frame Pointer Register.
1320:     BuildMI(MBB, MBBI, DL, ZII->get(SystemZ::LGR),
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321:             Regs.getFramePointerRegister())
1322:         .addReg(Regs.getStackPointerRegister());
1323: 
1324:     // Mark the FramePtr as live at the beginning of every block except
1325:     // the entry block.  (We'll have marked R8 as live on entry when
1326:     // saving the GPRs.)
1327:     for (MachineBasicBlock &B : llvm::drop_begin(MF))
1328:       B.addLiveIn(Regs.getFramePointerRegister());
1329:   }
1330: 
1331:   // Save GPRs used for varargs, if any.
1332:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
1333:   bool IsVarArg = MF.getFunction().isVarArg();
1334: 
1335:   if (IsVarArg) {
1336:     // FixedRegs is the number of used registers, accounting for shadow
1337:     // registers.
1338:     unsigned FixedRegs = ZFI->getVarArgsFirstGPR() + ZFI->getVarArgsFirstFPR();
1339:     auto &GPRs = SystemZ::XPLINK64ArgGPRs;
1340:     for (unsigned I = FixedRegs; I < SystemZ::XPLINK64NumArgGPRs; I++) {
1341:       uint64_t StartOffset = MFFrame.getOffsetAdjustment() +
1342:                              MFFrame.getStackSize() + Regs.getCallFrameSize() +
1343:                              getOffsetOfLocalArea() + I * getPointerSize();
1344:       unsigned Reg = GPRs[I];
1345:       BuildMI(MBB, MBBI, DL, TII->get(SystemZ::STG))
1346:           .addReg(Reg)
1347:           .addReg(Regs.getStackPointerRegister())
1348:           .addImm(StartOffset)
1349:           .addReg(0);
1350:       if (!MBB.isLiveIn(Reg))
1351:         MBB.addLiveIn(Reg);
1352:     }
1353:   }
1354: }
1355: 
1356: void SystemZXPLINKFrameLowering::emitEpilogue(MachineFunction &MF,
1357:                                               MachineBasicBlock &MBB) const {
1358:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1359:   MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
1360:   SystemZMachineFunctionInfo *ZFI = MF.getInfo<SystemZMachineFunctionInfo>();
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZXPLINKFrameLowering::emitEpilogue`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZXPLINKFrameLowering::emitEpilogue` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
1362:   auto *ZII = Subtarget.getInstrInfo();
1363:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1364: 
1365:   // Skip the return instruction.
1366:   assert(MBBI->isReturn() && "Can only insert epilogue into returning blocks");
1367: 
1368:   uint64_t StackSize = MFFrame.getStackSize();
1369:   if (StackSize) {
1370:     unsigned SPReg = Regs.getStackPointerRegister();
1371:     if (ZFI->getRestoreGPRRegs().LowGPR != SPReg) {
1372:       DebugLoc DL = MBBI->getDebugLoc();
1373:       emitIncrement(MBB, MBBI, DL, SPReg, StackSize, ZII);
1374:     }
1375:   }
1376: }
1377: 
1378: // Emit a compare of the stack pointer against the stack floor, and a call to
1379: // the LE stack extender if needed.
1380: void SystemZXPLINKFrameLowering::inlineStackProbe(
1381:     MachineFunction &MF, MachineBasicBlock &PrologMBB) const {
1382:   auto *ZII =
1383:       static_cast<const SystemZInstrInfo *>(MF.getSubtarget().getInstrInfo());
1384: 
1385:   MachineInstr *StackAllocMI = nullptr;
1386:   for (MachineInstr &MI : PrologMBB)
1387:     if (MI.getOpcode() == SystemZ::XPLINK_STACKALLOC) {
1388:       StackAllocMI = &MI;
1389:       break;
1390:     }
1391:   if (StackAllocMI == nullptr)
1392:     return;
1393: 
1394:   bool NeedSaveSP = hasFP(MF);
1395:   bool NeedSaveArg = PrologMBB.isLiveIn(SystemZ::R3D);
1396:   const int64_t SaveSlotR3 = 2192;
1397: 
1398:   MachineBasicBlock &MBB = PrologMBB;
1399:   const DebugLoc DL = StackAllocMI->getDebugLoc();
1400: 
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::inlineStackProbe`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::inlineStackProbe` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401:   // The 2nd half of block MBB after split.
1402:   MachineBasicBlock *NextMBB;
1403: 
1404:   // Add new basic block for the call to the stack overflow function.
1405:   MachineBasicBlock *StackExtMBB =
1406:       MF.CreateMachineBasicBlock(MBB.getBasicBlock());
1407:   MF.push_back(StackExtMBB);
1408: 
1409:   // LG r3,72(,r3)
1410:   BuildMI(StackExtMBB, DL, ZII->get(SystemZ::LG), SystemZ::R3D)
1411:       .addReg(SystemZ::R3D)
1412:       .addImm(72)
1413:       .addReg(0);
1414:   // BASR r3,r3
1415:   BuildMI(StackExtMBB, DL, ZII->get(SystemZ::CallBASR_STACKEXT))
1416:       .addReg(SystemZ::R3D);
1417:   if (NeedSaveArg) {
1418:     if (!NeedSaveSP) {
1419:       // LGR r0,r3
1420:       BuildMI(MBB, StackAllocMI, DL, ZII->get(SystemZ::LGR))
1421:           .addReg(SystemZ::R0D, RegState::Define)
1422:           .addReg(SystemZ::R3D);
1423:     } else {
1424:       // In this case, the incoming value of r4 is saved in r0 so the
1425:       // latter register is unavailable. Store r3 in its corresponding
1426:       // slot in the parameter list instead. Do this at the start of
1427:       // the prolog before r4 is manipulated by anything else.
1428:       // STG r3, 2192(r4)
1429:       BuildMI(MBB, MBB.begin(), DL, ZII->get(SystemZ::STG))
1430:           .addReg(SystemZ::R3D)
1431:           .addReg(SystemZ::R4D)
1432:           .addImm(SaveSlotR3)
1433:           .addReg(0);
1434:     }
1435:   }
1436:   // LLGT r3,1208
1437:   BuildMI(MBB, StackAllocMI, DL, ZII->get(SystemZ::LLGT), SystemZ::R3D)
1438:       .addReg(0)
1439:       .addImm(1208)
1440:       .addReg(0);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441:   // CG r4,64(,r3)
1442:   BuildMI(MBB, StackAllocMI, DL, ZII->get(SystemZ::CG))
1443:       .addReg(SystemZ::R4D)
1444:       .addReg(SystemZ::R3D)
1445:       .addImm(64)
1446:       .addReg(0);
1447:   // JLL b'0100',F'37'
1448:   BuildMI(MBB, StackAllocMI, DL, ZII->get(SystemZ::BRC))
1449:       .addImm(SystemZ::CCMASK_ICMP)
1450:       .addImm(SystemZ::CCMASK_CMP_LT)
1451:       .addMBB(StackExtMBB);
1452: 
1453:   NextMBB = SystemZ::splitBlockBefore(StackAllocMI, &MBB);
1454:   MBB.addSuccessor(NextMBB);
1455:   MBB.addSuccessor(StackExtMBB);
1456:   if (NeedSaveArg) {
1457:     if (!NeedSaveSP) {
1458:       // LGR r3, r0
1459:       BuildMI(*NextMBB, StackAllocMI, DL, ZII->get(SystemZ::LGR))
1460:           .addReg(SystemZ::R3D, RegState::Define)
1461:           .addReg(SystemZ::R0D, RegState::Kill);
1462:     } else {
1463:       // In this case, the incoming value of r4 is saved in r0 so the
1464:       // latter register is unavailable. We stored r3 in its corresponding
1465:       // slot in the parameter list instead and we now restore it from there.
1466:       // LGR r3, r0
1467:       BuildMI(*NextMBB, StackAllocMI, DL, ZII->get(SystemZ::LGR))
1468:           .addReg(SystemZ::R3D, RegState::Define)
1469:           .addReg(SystemZ::R0D);
1470:       // LG r3, 2192(r3)
1471:       BuildMI(*NextMBB, StackAllocMI, DL, ZII->get(SystemZ::LG))
1472:           .addReg(SystemZ::R3D, RegState::Define)
1473:           .addReg(SystemZ::R3D)
1474:           .addImm(SaveSlotR3)
1475:           .addReg(0);
1476:     }
1477:   }
1478: 
1479:   // Add jump back from stack extension BB.
1480:   BuildMI(StackExtMBB, DL, ZII->get(SystemZ::J)).addMBB(NextMBB);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481:   StackExtMBB->addSuccessor(NextMBB);
1482: 
1483:   StackAllocMI->eraseFromParent();
1484: 
1485:   // Compute the live-in lists for the new blocks.
1486:   fullyRecomputeLiveIns({StackExtMBB, NextMBB});
1487: }
1488: 
1489: bool SystemZXPLINKFrameLowering::hasFPImpl(const MachineFunction &MF) const {
1490:   return (MF.getFrameInfo().hasVarSizedObjects());
1491: }
1492: 
1493: void SystemZXPLINKFrameLowering::processFunctionBeforeFrameFinalized(
1494:     MachineFunction &MF, RegScavenger *RS) const {
1495:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
1496:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1497:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
1498: 
1499:   // Setup stack frame offset
1500:   MFFrame.setOffsetAdjustment(Regs.getStackPointerBias());
1501: 
1502:   // Nothing to do for leaf functions.
1503:   uint64_t StackSize = MFFrame.estimateStackSize(MF);
1504:   if (StackSize == 0 && MFFrame.getCalleeSavedInfo().empty())
1505:     return;
1506: 
1507:   // Although the XPLINK specifications for AMODE64 state that minimum size
1508:   // of the param area is minimum 32 bytes and no rounding is otherwise
1509:   // specified, we round this area in 64 bytes increments to be compatible
1510:   // with existing compilers.
1511:   MFFrame.setMaxCallFrameSize(
1512:       std::max(64U, (unsigned)alignTo(MFFrame.getMaxCallFrameSize(), 64)));
1513: 
1514:   // Add frame values with positive object offsets. Since the displacement from
1515:   // the SP/FP is calculated by ObjectOffset + StackSize + Bias, object offsets
1516:   // with positive values are in the caller's stack frame. We need to include
1517:   // that since it is accessed by displacement to SP/FP.
1518:   int64_t LargestArgOffset = 0;
1519:   for (int I = MFFrame.getObjectIndexBegin(); I != 0; ++I) {
1520:     if (MFFrame.getObjectOffset(I) >= 0) {
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::hasFPImpl`, `SystemZXPLINKFrameLowering::processFunctionBeforeFrameFinalized`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::hasFPImpl`, `SystemZXPLINKFrameLowering::processFunctionBeforeFrameFinalized` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521:       int64_t ObjOffset = MFFrame.getObjectOffset(I) + MFFrame.getObjectSize(I);
1522:       LargestArgOffset = std::max(ObjOffset, LargestArgOffset);
1523:     }
1524:   }
1525: 
1526:   uint64_t MaxReach = (StackSize + Regs.getCallFrameSize() +
1527:                        Regs.getStackPointerBias() + LargestArgOffset);
1528: 
1529:   if (!isUInt<12>(MaxReach)) {
1530:     // We may need register scavenging slots if some parts of the frame
1531:     // are outside the reach of an unsigned 12-bit displacement.
1532:     RS->addScavengingFrameIndex(MFFrame.CreateSpillStackObject(8, Align(8)));
1533:     RS->addScavengingFrameIndex(MFFrame.CreateSpillStackObject(8, Align(8)));
1534:   }
1535: }
1536: 
1537: // Determines the size of the frame, and creates the deferred spill objects.
1538: void SystemZXPLINKFrameLowering::determineFrameLayout(
1539:     MachineFunction &MF) const {
1540:   MachineFrameInfo &MFFrame = MF.getFrameInfo();
1541:   const SystemZSubtarget &Subtarget = MF.getSubtarget<SystemZSubtarget>();
1542:   auto *Regs =
1543:       static_cast<SystemZXPLINK64Registers *>(Subtarget.getSpecialRegisters());
1544: 
1545:   uint64_t StackSize = MFFrame.getStackSize();
1546:   if (StackSize == 0)
1547:     return;
1548: 
1549:   // Add the size of the register save area and the reserved area to the size.
1550:   StackSize += Regs->getCallFrameSize();
1551:   MFFrame.setStackSize(StackSize);
1552: 
1553:   // We now know the stack size. Update the stack objects for the register save
1554:   // area now. This has no impact on the stack frame layout, as this is already
1555:   // computed. However, it makes sure that all callee saved registers have a
1556:   // valid offset assigned.
1557:   for (int FrameIdx = MFFrame.getObjectIndexBegin(); FrameIdx != 0;
1558:        ++FrameIdx) {
1559:     if (MFFrame.getStackID(FrameIdx) == TargetStackID::NoAlloc) {
1560:       int64_t SPOffset = MFFrame.getObjectOffset(FrameIdx);
```
- **EN**: The range implements or declares functions including `SystemZXPLINKFrameLowering::determineFrameLayout`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZXPLINKFrameLowering::determineFrameLayout` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1561-1565 / 第 1561-1565 行
```cpp
1561:       SPOffset -= StackSize;
1562:       MFFrame.setObjectOffset(FrameIdx, SPOffset);
1563:     }
1564:   }
1565: }
```
- **EN**: This span continues the file's main responsibility: this file implements stack frame lowering and prologue/epilogue logic for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `SystemZFrameLowering.h`
- `SystemZCallingConv.h`
- `SystemZInstrInfo.h`
- `SystemZMachineFunctionInfo.h`
- `SystemZRegisterInfo.h`
- `SystemZSubtarget.h`
- `llvm/ADT/STLExtras.h`
- `llvm/CodeGen/LivePhysRegs.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineModuleInfo.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/RegisterScavenging.h`
- `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- `llvm/IR/CallingConv.h`
- `llvm/IR/Function.h`
- `llvm/Target/TargetMachine.h`
