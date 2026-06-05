# M68kRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kRegisterInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target register information for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责定义或实现目标寄存器信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kRegisterInfo.cpp - CPU0 Register Information --------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains the CPU0 implementation of the TargetRegisterInfo class.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "M68kRegisterInfo.h"
  15: 
  16: #include "M68k.h"
  17: #include "M68kMachineFunction.h"
  18: #include "M68kSubtarget.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kRegisterInfo.h`, `M68k.h`, `M68kMachineFunction.h`, `M68kSubtarget.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kRegisterInfo.h`, `M68k.h`, `M68kMachineFunction.h`, `M68kSubtarget.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: #include "MCTargetDesc/M68kMCTargetDesc.h"
  21: 
  22: #include "llvm/CodeGen/MachineFrameInfo.h"
  23: #include "llvm/CodeGen/MachineRegisterInfo.h"
  24: #include "llvm/IR/Function.h"
  25: #include "llvm/IR/Type.h"
  26: #include "llvm/Support/CommandLine.h"
  27: #include "llvm/Support/Debug.h"
  28: #include "llvm/Support/ErrorHandling.h"
  29: #include "llvm/Support/raw_ostream.h"
  30: 
  31: #define GET_REGINFO_TARGET_DESC
  32: #include "M68kGenRegisterInfo.inc"
  33: 
  34: #define DEBUG_TYPE "m68k-reg-info"
  35: 
  36: using namespace llvm;
```
- **EN**: It imports dependencies such as `M68kMCTargetDesc.h`, `MachineFrameInfo.h`, `MachineRegisterInfo.h`, `Function.h`, `Type.h`, `CommandLine.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file. Generated `.inc` fragments are pulled in here, a common LLVM technique for TableGen-produced code.
- **CN**: 它引入了 `M68kMCTargetDesc.h`, `MachineFrameInfo.h`, `MachineRegisterInfo.h`, `Function.h`, `Type.h`, `CommandLine.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。 这里引入了生成的 `.inc` 片段，这是 LLVM 使用 TableGen 产物的常见方式。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38: static cl::opt<bool> EnableBasePointer(
  39:     "m68k-use-base-pointer", cl::Hidden, cl::init(true),
  40:     cl::desc("Enable use of a base pointer for complex stack frames"));
  41: 
  42: // Pin the vtable to this file.
  43: void M68kRegisterInfo::anchor() {}
  44: 
  45: M68kRegisterInfo::M68kRegisterInfo(const M68kSubtarget &ST)
  46:     // FIXME x26 not sure it this the correct value, it expects RA, but M68k
  47:     // passes IP anyway, how this works?
  48:     : M68kGenRegisterInfo(M68k::A0, 0, 0, M68k::PC), Subtarget(ST) {
  49:   StackPtr = M68k::SP;
  50:   FramePtr = M68k::A6;
  51:   GlobalBasePtr = M68k::A5;
  52:   BasePtr = M68k::A4;
  53: }
  54: 
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::anchor`.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::anchor` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55: //===----------------------------------------------------------------------===//
  56: // Callee Saved Registers methods
  57: //===----------------------------------------------------------------------===//
  58: 
  59: const MCPhysReg *
  60: M68kRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  61:   return CSR_STD_SaveList;
  62: }
  63: 
  64: const uint32_t *
  65: M68kRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
  66:                                        CallingConv::ID) const {
  67:   return CSR_STD_RegMask;
  68: }
  69: 
  70: const TargetRegisterClass *
  71: M68kRegisterInfo::getRegsForTailCall(const MachineFunction &MF) const {
  72:   return &M68k::XR32_TCRegClass;
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::getCalleeSavedRegs`, `M68kRegisterInfo::getCallPreservedMask`, `M68kRegisterInfo::getRegsForTailCall`.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::getCalleeSavedRegs`, `M68kRegisterInfo::getCallPreservedMask`, `M68kRegisterInfo::getRegsForTailCall` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73: }
  74: 
  75: unsigned
  76: M68kRegisterInfo::getMatchingMegaReg(unsigned Reg,
  77:                                      const TargetRegisterClass *RC) const {
  78:   for (MCPhysReg Super : superregs(Reg))
  79:     if (RC->contains(Super))
  80:       return Super;
  81:   return 0;
  82: }
  83: 
  84: const TargetRegisterClass *
  85: M68kRegisterInfo::getMaximalPhysRegClass(unsigned reg, MVT VT) const {
  86:   assert(Register::isPhysicalRegister(reg) &&
  87:          "reg must be a physical register");
  88: 
  89:   // Pick the most sub register class of the right type that contains
  90:   // this physreg.
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::getMatchingMegaReg`, `M68kRegisterInfo::getMaximalPhysRegClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::getMatchingMegaReg`, `M68kRegisterInfo::getMaximalPhysRegClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   const TargetRegisterClass *BestRC = nullptr;
  92:   for (regclass_iterator I = regclass_begin(), E = regclass_end(); I != E;
  93:        ++I) {
  94:     const TargetRegisterClass *RC = *I;
  95:     if ((VT == MVT::Other || isTypeLegalForClass(*RC, VT)) &&
  96:         RC->contains(reg) &&
  97:         (!BestRC ||
  98:          (BestRC->hasSubClass(RC) && RC->getNumRegs() > BestRC->getNumRegs())))
  99:       BestRC = RC;
 100:   }
 101: 
 102:   assert(BestRC && "Couldn't find the register class");
 103:   return BestRC;
 104: }
 105: 
 106: int M68kRegisterInfo::getRegisterOrder(unsigned Reg,
 107:                                        const TargetRegisterClass &TRC) const {
 108:   for (unsigned i = 0; i < TRC.getNumRegs(); ++i) {
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::getRegisterOrder`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::getRegisterOrder` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     if (regsOverlap(Reg, TRC.getRegister(i))) {
 110:       return i;
 111:     }
 112:   }
 113:   return -1;
 114: }
 115: 
 116: int M68kRegisterInfo::getSpillRegisterOrder(unsigned Reg) const {
 117:   int Result = getRegisterOrder(Reg, *getRegClass(M68k::SPILLRegClassID));
 118:   assert(Result >= 0 && "Can not determine spill order");
 119:   return Result;
 120: }
 121: 
 122: BitVector M68kRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
 123:   const M68kFrameLowering *TFI = getFrameLowering(MF);
 124: 
 125:   BitVector Reserved(getNumRegs());
 126: 
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::getSpillRegisterOrder`, `M68kRegisterInfo::getReservedRegs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::getSpillRegisterOrder`, `M68kRegisterInfo::getReservedRegs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   // Set a register's and its sub-registers and aliases as reserved.
 128:   auto setBitVector = [&Reserved, this](unsigned Reg) {
 129:     for (MCRegAliasIterator I(Reg, this, /* self */ true); I.isValid(); ++I) {
 130:       Reserved.set(*I);
 131:     }
 132:     for (MCPhysReg I : subregs_inclusive(Reg)) {
 133:       Reserved.set(I);
 134:     }
 135:   };
 136: 
 137:   // Registers reserved by users
 138:   for (size_t Reg = 0, Total = getNumRegs(); Reg != Total; ++Reg) {
 139:     if (MF.getSubtarget<M68kSubtarget>().isRegisterReservedByUser(Reg))
 140:       setBitVector(Reg);
 141:   }
 142: 
 143:   setBitVector(M68k::PC);
 144:   setBitVector(M68k::SP);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 145-162 / 第 145-162 行
```cpp
 145: 
 146:   if (TFI->hasFP(MF)) {
 147:     setBitVector(FramePtr);
 148:   }
 149: 
 150:   // Set the base-pointer register and its aliases as reserved if needed.
 151:   if (hasBasePointer(MF)) {
 152:     CallingConv::ID CC = MF.getFunction().getCallingConv();
 153:     const uint32_t *RegMask = getCallPreservedMask(MF, CC);
 154:     if (MachineOperand::clobbersPhysReg(RegMask, getBaseRegister()))
 155:       report_fatal_error("Stack realignment in presence of dynamic allocas is "
 156:                          "not supported with"
 157:                          "this calling convention.");
 158: 
 159:     setBitVector(getBaseRegister());
 160:   }
 161: 
 162:   return Reserved;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163: }
 164: 
 165: bool M68kRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
 166:                                            int SPAdj, unsigned FIOperandNum,
 167:                                            RegScavenger *RS) const {
 168:   MachineInstr &MI = *II;
 169:   MachineFunction &MF = *MI.getParent()->getParent();
 170:   const M68kFrameLowering *TFI = getFrameLowering(MF);
 171: 
 172:   // We have either (i,An,Rn) or (i,An) EA form
 173:   // NOTE Base contains the FI and we need to backtrace a bit to get Disp
 174:   MachineOperand &Disp = MI.getOperand(FIOperandNum - 1);
 175:   MachineOperand &Base = MI.getOperand(FIOperandNum);
 176: 
 177:   int Imm = (int)(Disp.getImm());
 178:   int FIndex = (int)(Base.getIndex());
 179: 
 180:   // FIXME tail call: implement jmp from mem
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::eliminateFrameIndex`.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::eliminateFrameIndex` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   bool AfterFPPop = false;
 182: 
 183:   unsigned BasePtr;
 184:   if (hasBasePointer(MF))
 185:     BasePtr = (FIndex < 0 ? FramePtr : getBaseRegister());
 186:   else if (hasStackRealignment(MF))
 187:     BasePtr = (FIndex < 0 ? FramePtr : StackPtr);
 188:   else if (AfterFPPop)
 189:     BasePtr = StackPtr;
 190:   else
 191:     BasePtr = (TFI->hasFP(MF) ? FramePtr : StackPtr);
 192: 
 193:   Base.ChangeToRegister(BasePtr, false);
 194: 
 195:   // Now add the frame object offset to the offset from FP.
 196:   int64_t FIOffset;
 197:   Register IgnoredFrameReg;
 198:   if (AfterFPPop) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:     // Tail call jmp happens after FP is popped.
 200:     const MachineFrameInfo &MFI = MF.getFrameInfo();
 201:     FIOffset = MFI.getObjectOffset(FIndex) - TFI->getOffsetOfLocalArea();
 202:   } else {
 203:     FIOffset =
 204:         TFI->getFrameIndexReference(MF, FIndex, IgnoredFrameReg).getFixed();
 205:   }
 206: 
 207:   if (BasePtr == StackPtr)
 208:     FIOffset += SPAdj;
 209: 
 210:   Disp.ChangeToImmediate(FIOffset + Imm);
 211:   return false;
 212: }
 213: 
 214: bool M68kRegisterInfo::requiresRegisterScavenging(
 215:     const MachineFunction &MF) const {
 216:   return false;
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::requiresRegisterScavenging`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::requiresRegisterScavenging` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217: }
 218: 
 219: bool M68kRegisterInfo::trackLivenessAfterRegAlloc(
 220:     const MachineFunction &MF) const {
 221:   return true;
 222: }
 223: 
 224: static bool CantUseSP(const MachineFrameInfo &MFI) {
 225:   return MFI.hasVarSizedObjects() || MFI.hasOpaqueSPAdjustment();
 226: }
 227: 
 228: bool M68kRegisterInfo::hasBasePointer(const MachineFunction &MF) const {
 229:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 230: 
 231:   if (!EnableBasePointer)
 232:     return false;
 233: 
 234:   // When we need stack realignment, we can't address the stack from the frame
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::trackLivenessAfterRegAlloc`, `CantUseSP`, `M68kRegisterInfo::hasBasePointer`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::trackLivenessAfterRegAlloc`, `CantUseSP`, `M68kRegisterInfo::hasBasePointer` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   // pointer.  When we have dynamic allocas or stack-adjusting inline asm, we
 236:   // can't address variables from the stack pointer.  MS inline asm can
 237:   // reference locals while also adjusting the stack pointer.  When we can't
 238:   // use both the SP and the FP, we need a separate base pointer register.
 239:   bool CantUseFP = hasStackRealignment(MF);
 240:   return CantUseFP && CantUseSP(MFI);
 241: }
 242: 
 243: bool M68kRegisterInfo::canRealignStack(const MachineFunction &MF) const {
 244:   if (!TargetRegisterInfo::canRealignStack(MF))
 245:     return false;
 246: 
 247:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 248:   const MachineRegisterInfo *MRI = &MF.getRegInfo();
 249: 
 250:   // Stack realignment requires a frame pointer.  If we already started
 251:   // register allocation with frame pointer elimination, it is too late now.
 252:   if (!MRI->canReserveReg(FramePtr))
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::canRealignStack`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::canRealignStack` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-269 / 第 253-269 行
```cpp
 253:     return false;
 254: 
 255:   // If a base pointer is necessary. Check that it isn't too late to reserve it.
 256:   if (CantUseSP(MFI))
 257:     return MRI->canReserveReg(BasePtr);
 258: 
 259:   return true;
 260: }
 261: 
 262: Register M68kRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
 263:   const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
 264:   return TFI->hasFP(MF) ? FramePtr : StackPtr;
 265: }
 266: 
 267: const TargetRegisterClass *M68kRegisterInfo::intRegClass(unsigned size) const {
 268:   return &M68k::DR32RegClass;
 269: }
```
- **EN**: The range implements or declares functions including `M68kRegisterInfo::getFrameRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kRegisterInfo::getFrameRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68kRegisterInfo.h`
- `M68k.h`
- `M68kMachineFunction.h`
- `M68kSubtarget.h`
- `MCTargetDesc/M68kMCTargetDesc.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/IR/Function.h`
- `llvm/IR/Type.h`
- `llvm/Support/CommandLine.h`
- `llvm/Support/Debug.h`
- `llvm/Support/ErrorHandling.h`
- `llvm/Support/raw_ostream.h`
- `M68kGenRegisterInfo.inc`
