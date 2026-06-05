# LanaiFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiFrameLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the Lanai implementation of TargetFrameLowering class.
- 目的（中文）: 实现栈帧布局、函数序言/尾声生成以及相关调用序列细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiFrameLowering.cpp - Lanai Frame Information ------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the Lanai implementation of TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LanaiFrameLowering.h"
  14: 
  15: #include "LanaiAluCode.h"
  16: #include "LanaiInstrInfo.h"
  17: #include "LanaiSubtarget.h"
  18: #include "llvm/CodeGen/MachineFrameInfo.h"
  19: #include "llvm/CodeGen/MachineFunction.h"
  20: #include "llvm/CodeGen/MachineInstrBuilder.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 21-40

```cpp
  21: 
  22: using namespace llvm;
  23: 
  24: // Determines the size of the frame and maximum call frame size.
  25: void LanaiFrameLowering::determineFrameLayout(MachineFunction &MF) const {
  26:   MachineFrameInfo &MFI = MF.getFrameInfo();
  27:   const LanaiRegisterInfo *LRI = STI.getRegisterInfo();
  28: 
  29:   // Get the number of bytes to allocate from the FrameInfo.
  30:   unsigned FrameSize = MFI.getStackSize();
  31: 
  32:   // Get the alignment.
  33:   Align StackAlign =
  34:       LRI->hasStackRealignment(MF) ? MFI.getMaxAlign() : getStackAlign();
  35: 
  36:   // Get the maximum call frame size of all the calls.
  37:   unsigned MaxCallFrameSize = MFI.getMaxCallFrameSize();
  38: 
  39:   // If we have dynamic alloca then MaxCallFrameSize needs to be aligned so
  40:   // that allocations will be aligned.
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as determineFrameLayout contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 determineFrameLayout 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 41-60

```cpp
  41:   if (MFI.hasVarSizedObjects())
  42:     MaxCallFrameSize = alignTo(MaxCallFrameSize, StackAlign);
  43: 
  44:   // Update maximum call frame size.
  45:   MFI.setMaxCallFrameSize(MaxCallFrameSize);
  46: 
  47:   // Include call frame size in total.
  48:   if (!(hasReservedCallFrame(MF) && MFI.adjustsStack()))
  49:     FrameSize += MaxCallFrameSize;
  50: 
  51:   // Make sure the frame is aligned.
  52:   FrameSize = alignTo(FrameSize, StackAlign);
  53: 
  54:   // Update frame info.
  55:   MFI.setStackSize(FrameSize);
  56: }
  57: 
  58: // Iterates through each basic block in a machine function and replaces
  59: // ADJDYNALLOC pseudo instructions with a Lanai:ADDI with the
  60: // maximum call frame size as the immediate.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 61-80

```cpp
  61: void LanaiFrameLowering::replaceAdjDynAllocPseudo(MachineFunction &MF) const {
  62:   const LanaiInstrInfo &LII = *STI.getInstrInfo();
  63:   unsigned MaxCallFrameSize = MF.getFrameInfo().getMaxCallFrameSize();
  64: 
  65:   for (MachineBasicBlock &MBB : MF) {
  66:     for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
  67:       if (MI.getOpcode() == Lanai::ADJDYNALLOC) {
  68:         DebugLoc DL = MI.getDebugLoc();
  69:         Register Dst = MI.getOperand(0).getReg();
  70:         Register Src = MI.getOperand(1).getReg();
  71: 
  72:         BuildMI(MBB, MI, DL, LII.get(Lanai::ADD_I_LO), Dst)
  73:             .addReg(Src)
  74:             .addImm(MaxCallFrameSize);
  75:         MI.eraseFromParent();
  76:       }
  77:     }
  78:   }
  79: }
  80: 
```

- EN: Function bodies or method definitions such as replaceAdjDynAllocPseudo contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: replaceAdjDynAllocPseudo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 81-100

```cpp
  81: // Generates the following sequence for function entry:
  82: //   st %fp,-4[*%sp]        !push old FP
  83: //   add %sp,8,%fp          !generate new FP
  84: //   sub %sp,0x4,%sp        !allocate stack space (as needed)
  85: void LanaiFrameLowering::emitPrologue(MachineFunction &MF,
  86:                                       MachineBasicBlock &MBB) const {
  87:   assert(&MF.front() == &MBB && "Shrink-wrapping not yet supported");
  88: 
  89:   MachineFrameInfo &MFI = MF.getFrameInfo();
  90:   const LanaiInstrInfo &LII = *STI.getInstrInfo();
  91:   MachineBasicBlock::iterator MBBI = MBB.begin();
  92: 
  93:   // Debug location must be unknown since the first debug location is used
  94:   // to determine the end of the prologue.
  95:   DebugLoc DL;
  96: 
  97:   // Determine the correct frame layout
  98:   determineFrameLayout(MF);
  99: 
 100:   // FIXME: This appears to be overallocating.  Needs investigation.
```

- EN: Function bodies or method definitions such as emitPrologue contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitPrologue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 101-120

```cpp
 101:   // Get the number of bytes to allocate from the FrameInfo.
 102:   unsigned StackSize = MFI.getStackSize();
 103: 
 104:   // Push old FP
 105:   // st %fp,-4[*%sp]
 106:   BuildMI(MBB, MBBI, DL, LII.get(Lanai::SW_RI))
 107:       .addReg(Lanai::FP)
 108:       .addReg(Lanai::SP)
 109:       .addImm(-4)
 110:       .addImm(LPAC::makePreOp(LPAC::ADD))
 111:       .setMIFlag(MachineInstr::FrameSetup);
 112: 
 113:   // Generate new FP
 114:   // add %sp,8,%fp
 115:   BuildMI(MBB, MBBI, DL, LII.get(Lanai::ADD_I_LO), Lanai::FP)
 116:       .addReg(Lanai::SP)
 117:       .addImm(8)
 118:       .setMIFlag(MachineInstr::FrameSetup);
 119: 
 120:   // Allocate space on the stack if needed
```

- EN: At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 121-140

```cpp
 121:   // sub %sp,StackSize,%sp
 122:   if (StackSize != 0) {
 123:     BuildMI(MBB, MBBI, DL, LII.get(Lanai::SUB_I_LO), Lanai::SP)
 124:         .addReg(Lanai::SP)
 125:         .addImm(StackSize)
 126:         .setMIFlag(MachineInstr::FrameSetup);
 127:   }
 128: 
 129:   // Replace ADJDYNANALLOC
 130:   if (MFI.hasVarSizedObjects())
 131:     replaceAdjDynAllocPseudo(MF);
 132: }
 133: 
 134: MachineBasicBlock::iterator LanaiFrameLowering::eliminateCallFramePseudoInstr(
 135:     MachineFunction & /*MF*/, MachineBasicBlock &MBB,
 136:     MachineBasicBlock::iterator I) const {
 137:   // Discard ADJCALLSTACKDOWN, ADJCALLSTACKUP instructions.
 138:   return MBB.erase(I);
 139: }
 140: 
```

- EN: Function bodies or method definitions such as eliminateCallFramePseudoInstr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: eliminateCallFramePseudoInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 141-160

```cpp
 141: // The function epilogue should not depend on the current stack pointer!
 142: // It should use the frame pointer only.  This is mandatory because
 143: // of alloca; we also take advantage of it to omit stack adjustments
 144: // before returning.
 145: //
 146: // Note that when we go to restore the preserved register values we must
 147: // not try to address their slots by using offsets from the stack pointer.
 148: // That's because the stack pointer may have been moved during the function
 149: // execution due to a call to alloca().  Rather, we must restore all
 150: // preserved registers via offsets from the frame pointer value.
 151: //
 152: // Note also that when the current frame is being "popped" (by adjusting
 153: // the value of the stack pointer) on function exit, we must (for the
 154: // sake of alloca) set the new value of the stack pointer based upon
 155: // the current value of the frame pointer.  We can't just add what we
 156: // believe to be the (static) frame size to the stack pointer because
 157: // if we did that, and alloca() had been called during this function,
 158: // we would end up returning *without* having fully deallocated all of
 159: // the space grabbed by alloca.  If that happened, and a function
 160: // containing one or more alloca() calls was called over and over again,
```

- EN: At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 161-180

```cpp
 161: // then the stack would grow without limit!
 162: //
 163: // RET is lowered to
 164: //      ld -4[%fp],%pc  # modify %pc (two delay slots)
 165: // as the return address is in the stack frame and mov to pc is allowed.
 166: // emitEpilogue emits
 167: //      mov %fp,%sp     # restore the stack pointer
 168: //      ld -8[%fp],%fp  # restore the caller's frame pointer
 169: // before RET and the delay slot filler will move RET such that these
 170: // instructions execute in the delay slots of the load to PC.
 171: void LanaiFrameLowering::emitEpilogue(MachineFunction & /*MF*/,
 172:                                       MachineBasicBlock &MBB) const {
 173:   MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
 174:   const LanaiInstrInfo &LII = *STI.getInstrInfo();
 175:   DebugLoc DL = MBBI->getDebugLoc();
 176: 
 177:   // Restore the stack pointer using the callee's frame pointer value.
 178:   BuildMI(MBB, MBBI, DL, LII.get(Lanai::ADD_I_LO), Lanai::SP)
 179:       .addReg(Lanai::FP)
 180:       .addImm(0);
```

- EN: Function bodies or method definitions such as emitEpilogue contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitEpilogue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 181-200

```cpp
 181: 
 182:   // Restore the frame pointer from the stack.
 183:   BuildMI(MBB, MBBI, DL, LII.get(Lanai::LDW_RI), Lanai::FP)
 184:       .addReg(Lanai::FP)
 185:       .addImm(-8)
 186:       .addImm(LPAC::ADD);
 187: }
 188: 
 189: void LanaiFrameLowering::determineCalleeSaves(MachineFunction &MF,
 190:                                               BitVector &SavedRegs,
 191:                                               RegScavenger *RS) const {
 192:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
 193: 
 194:   MachineFrameInfo &MFI = MF.getFrameInfo();
 195:   const LanaiRegisterInfo *LRI = STI.getRegisterInfo();
 196:   int Offset = -4;
 197: 
 198:   // Reserve 4 bytes for the saved RCA
 199:   MFI.CreateFixedObject(4, Offset, true);
 200:   Offset -= 4;
```

- EN: Function bodies or method definitions such as determineCalleeSaves contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: determineCalleeSaves 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 201-210

```cpp
 201: 
 202:   // Reserve 4 bytes for the saved FP
 203:   MFI.CreateFixedObject(4, Offset, true);
 204:   Offset -= 4;
 205: 
 206:   if (LRI->hasBasePointer(MF)) {
 207:     MFI.CreateFixedObject(4, Offset, true);
 208:     SavedRegs.reset(LRI->getBaseRegister());
 209:   }
 210: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

## Key Concepts / 关键概念

- Prologue and epilogue emission / 序言与尾声生成
- Stack object layout / 栈对象布局
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiFrameLowering.h`, `LanaiAluCode.h`, `LanaiInstrInfo.h`, `LanaiSubtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `LanaiFrameLowering.h`
