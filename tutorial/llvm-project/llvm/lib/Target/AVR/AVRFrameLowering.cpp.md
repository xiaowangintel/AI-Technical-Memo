# AVRFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRFrameLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the AVR implementation of TargetFrameLowering class.
- 目的（中文）: 实现栈帧布局、函数序言/尾声生成以及相关调用序列细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRFrameLowering.cpp - AVR Frame Information ----------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the AVR implementation of TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "AVRFrameLowering.h"
  14: 
  15: #include "AVR.h"
  16: #include "AVRInstrInfo.h"
  17: #include "AVRMachineFunctionInfo.h"
  18: #include "AVRTargetMachine.h"
  19: #include "MCTargetDesc/AVRMCTargetDesc.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineFrameInfo.h"
  22: #include "llvm/CodeGen/MachineFunction.h"
  23: #include "llvm/CodeGen/MachineFunctionPass.h"
  24: #include "llvm/CodeGen/MachineInstrBuilder.h"
  25: #include "llvm/CodeGen/MachineRegisterInfo.h"
  26: 
  27: namespace llvm {
  28: 
  29: AVRFrameLowering::AVRFrameLowering()
  30:     : TargetFrameLowering(TargetFrameLowering::StackGrowsDown, Align(1), -2) {}
  31: 
  32: bool AVRFrameLowering::canSimplifyCallFramePseudos(
  33:     const MachineFunction &MF) const {
  34:   // Always simplify call frame pseudo instructions, even when
  35:   // hasReservedCallFrame is false.
  36:   return true;
  37: }
  38: 
  39: bool AVRFrameLowering::hasReservedCallFrame(const MachineFunction &MF) const {
  40:   // Reserve call frame memory in function prologue under the following
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as TargetFrameLowering, canSimplifyCallFramePseudos, hasReservedCallFrame contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 TargetFrameLowering, canSimplifyCallFramePseudos, hasReservedCallFrame 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 41-60

```cpp
  41:   // conditions:
  42:   // - Y pointer is reserved to be the frame pointer.
  43:   // - The function does not contain variable sized objects.
  44: 
  45:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  46:   return hasFP(MF) && !MFI.hasVarSizedObjects();
  47: }
  48: 
  49: void AVRFrameLowering::emitPrologue(MachineFunction &MF,
  50:                                     MachineBasicBlock &MBB) const {
  51:   MachineBasicBlock::iterator MBBI = MBB.begin();
  52:   DebugLoc DL = (MBBI != MBB.end()) ? MBBI->getDebugLoc() : DebugLoc();
  53:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
  54:   const AVRInstrInfo &TII = *STI.getInstrInfo();
  55:   const AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
  56:   const MachineRegisterInfo &MRI = MF.getRegInfo();
  57:   bool HasFP = hasFP(MF);
  58: 
  59:   // Interrupt handlers re-enable interrupts in function entry.
  60:   if (AFI->isInterruptHandler()) {
```

- EN: Function bodies or method definitions such as emitPrologue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitPrologue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 61-80

```cpp
  61:     BuildMI(MBB, MBBI, DL, TII.get(AVR::BSETs))
  62:         .addImm(0x07)
  63:         .setMIFlag(MachineInstr::FrameSetup);
  64:   }
  65: 
  66:   // Emit special prologue code to save R1, R0 and SREG in interrupt/signal
  67:   // handlers before saving any other registers.
  68:   if (AFI->isInterruptOrSignalHandler()) {
  69:     BuildMI(MBB, MBBI, DL, TII.get(AVR::PUSHRr))
  70:         .addReg(STI.getTmpRegister(), RegState::Kill)
  71:         .setMIFlag(MachineInstr::FrameSetup);
  72: 
  73:     BuildMI(MBB, MBBI, DL, TII.get(AVR::INRdA), STI.getTmpRegister())
  74:         .addImm(STI.getIORegSREG())
  75:         .setMIFlag(MachineInstr::FrameSetup);
  76:     BuildMI(MBB, MBBI, DL, TII.get(AVR::PUSHRr))
  77:         .addReg(STI.getTmpRegister(), RegState::Kill)
  78:         .setMIFlag(MachineInstr::FrameSetup);
  79:     if (!MRI.reg_empty(STI.getZeroRegister())) {
  80:       BuildMI(MBB, MBBI, DL, TII.get(AVR::PUSHRr))
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 81-100

```cpp
  81:           .addReg(STI.getZeroRegister(), RegState::Kill)
  82:           .setMIFlag(MachineInstr::FrameSetup);
  83:       BuildMI(MBB, MBBI, DL, TII.get(AVR::EORRdRr))
  84:           .addReg(STI.getZeroRegister(), RegState::Define)
  85:           .addReg(STI.getZeroRegister(), RegState::Kill)
  86:           .addReg(STI.getZeroRegister(), RegState::Kill)
  87:           .setMIFlag(MachineInstr::FrameSetup);
  88:     }
  89:   }
  90: 
  91:   // Early exit if the frame pointer is not needed in this function.
  92:   if (!HasFP) {
  93:     return;
  94:   }
  95: 
  96:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  97:   unsigned FrameSize = MFI.getStackSize() - AFI->getCalleeSavedFrameSize();
  98: 
  99:   // Skip the callee-saved push instructions.
 100:   while (
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 101-120

```cpp
 101:       (MBBI != MBB.end()) && MBBI->getFlag(MachineInstr::FrameSetup) &&
 102:       (MBBI->getOpcode() == AVR::PUSHRr || MBBI->getOpcode() == AVR::PUSHWRr)) {
 103:     ++MBBI;
 104:   }
 105: 
 106:   // Update Y with the new base value.
 107:   BuildMI(MBB, MBBI, DL, TII.get(AVR::SPREAD), AVR::R29R28)
 108:       .addReg(AVR::SP)
 109:       .setMIFlag(MachineInstr::FrameSetup);
 110: 
 111:   // Mark the FramePtr as live-in in every block except the entry.
 112:   for (MachineBasicBlock &MBBJ : llvm::drop_begin(MF)) {
 113:     MBBJ.addLiveIn(AVR::R29R28);
 114:   }
 115: 
 116:   if (!FrameSize) {
 117:     return;
 118:   }
 119: 
 120:   // Reserve the necessary frame memory by doing FP -= <size>.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 121-140

```cpp
 121:   unsigned Opcode = (isUInt<6>(FrameSize) && STI.hasADDSUBIW()) ? AVR::SBIWRdK
 122:                                                                 : AVR::SUBIWRdK;
 123: 
 124:   MachineInstr *MI = BuildMI(MBB, MBBI, DL, TII.get(Opcode), AVR::R29R28)
 125:                          .addReg(AVR::R29R28, RegState::Kill)
 126:                          .addImm(FrameSize)
 127:                          .setMIFlag(MachineInstr::FrameSetup);
 128:   // The SREG implicit def is dead.
 129:   MI->getOperand(3).setIsDead();
 130: 
 131:   // Write back R29R28 to SP and temporarily disable interrupts.
 132:   BuildMI(MBB, MBBI, DL, TII.get(AVR::SPWRITE), AVR::SP)
 133:       .addReg(AVR::R29R28)
 134:       .setMIFlag(MachineInstr::FrameSetup);
 135: }
 136: 
 137: static void restoreStatusRegister(MachineFunction &MF, MachineBasicBlock &MBB) {
 138:   const AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
 139:   const MachineRegisterInfo &MRI = MF.getRegInfo();
 140: 
```

- EN: Function bodies or method definitions such as restoreStatusRegister contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: restoreStatusRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 141-160

```cpp
 141:   MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
 142: 
 143:   DebugLoc DL = MBBI->getDebugLoc();
 144:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 145:   const AVRInstrInfo &TII = *STI.getInstrInfo();
 146: 
 147:   // Emit special epilogue code to restore R1, R0 and SREG in interrupt/signal
 148:   // handlers at the very end of the function, just before reti.
 149:   if (AFI->isInterruptOrSignalHandler()) {
 150:     if (!MRI.reg_empty(STI.getZeroRegister())) {
 151:       BuildMI(MBB, MBBI, DL, TII.get(AVR::POPRd), STI.getZeroRegister());
 152:     }
 153:     BuildMI(MBB, MBBI, DL, TII.get(AVR::POPRd), STI.getTmpRegister());
 154:     BuildMI(MBB, MBBI, DL, TII.get(AVR::OUTARr))
 155:         .addImm(STI.getIORegSREG())
 156:         .addReg(STI.getTmpRegister(), RegState::Kill);
 157:     BuildMI(MBB, MBBI, DL, TII.get(AVR::POPRd), STI.getTmpRegister());
 158:   }
 159: }
 160: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 161-180

```cpp
 161: void AVRFrameLowering::emitEpilogue(MachineFunction &MF,
 162:                                     MachineBasicBlock &MBB) const {
 163:   const AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
 164: 
 165:   // Early exit if the frame pointer is not needed in this function except for
 166:   // signal/interrupt handlers where special code generation is required.
 167:   if (!hasFP(MF) && !AFI->isInterruptOrSignalHandler()) {
 168:     return;
 169:   }
 170: 
 171:   MachineBasicBlock::iterator MBBI = MBB.getLastNonDebugInstr();
 172:   assert(MBBI->getDesc().isReturn() &&
 173:          "Can only insert epilog into returning blocks");
 174: 
 175:   DebugLoc DL = MBBI->getDebugLoc();
 176:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 177:   unsigned FrameSize = MFI.getStackSize() - AFI->getCalleeSavedFrameSize();
 178:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 179:   const AVRInstrInfo &TII = *STI.getInstrInfo();
 180: 
```

- EN: Function bodies or method definitions such as emitEpilogue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitEpilogue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 181-200

```cpp
 181:   // Early exit if there is no need to restore the frame pointer.
 182:   if (!FrameSize && !MF.getFrameInfo().hasVarSizedObjects()) {
 183:     restoreStatusRegister(MF, MBB);
 184:     return;
 185:   }
 186: 
 187:   // Skip the callee-saved pop instructions.
 188:   while (MBBI != MBB.begin()) {
 189:     MachineBasicBlock::iterator PI = std::prev(MBBI);
 190:     int Opc = PI->getOpcode();
 191: 
 192:     if (Opc != AVR::POPRd && Opc != AVR::POPWRd && !PI->isTerminator()) {
 193:       break;
 194:     }
 195: 
 196:     --MBBI;
 197:   }
 198: 
 199:   if (FrameSize) {
 200:     unsigned Opcode;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 201-220

```cpp
 201: 
 202:     // Select the optimal opcode depending on how big it is.
 203:     if (isUInt<6>(FrameSize) && STI.hasADDSUBIW()) {
 204:       Opcode = AVR::ADIWRdK;
 205:     } else {
 206:       Opcode = AVR::SUBIWRdK;
 207:       FrameSize = -FrameSize;
 208:     }
 209: 
 210:     // Restore the frame pointer by doing FP += <size>.
 211:     MachineInstr *MI = BuildMI(MBB, MBBI, DL, TII.get(Opcode), AVR::R29R28)
 212:                            .addReg(AVR::R29R28, RegState::Kill)
 213:                            .addImm(FrameSize);
 214:     // The SREG implicit def is dead.
 215:     MI->getOperand(3).setIsDead();
 216:   }
 217: 
 218:   // Write back R29R28 to SP and temporarily disable interrupts.
 219:   BuildMI(MBB, MBBI, DL, TII.get(AVR::SPWRITE), AVR::SP)
 220:       .addReg(AVR::R29R28, RegState::Kill);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 221-240

```cpp
 221: 
 222:   restoreStatusRegister(MF, MBB);
 223: }
 224: 
 225: // Return true if the specified function should have a dedicated frame
 226: // pointer register. This is true if the function meets any of the following
 227: // conditions:
 228: //  - a register has been spilled
 229: //  - has allocas
 230: //  - input arguments are passed using the stack
 231: //
 232: // Notice that strictly this is not a frame pointer because it contains SP after
 233: // frame allocation instead of having the original SP in function entry.
 234: bool AVRFrameLowering::hasFPImpl(const MachineFunction &MF) const {
 235:   const AVRMachineFunctionInfo *FuncInfo = MF.getInfo<AVRMachineFunctionInfo>();
 236: 
 237:   return (FuncInfo->getHasSpills() || FuncInfo->getHasAllocas() ||
 238:           FuncInfo->getHasStackArgs() ||
 239:           MF.getFrameInfo().hasVarSizedObjects());
 240: }
```

- EN: Function bodies or method definitions such as hasFPImpl contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: hasFPImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 241-260

```cpp
 241: 
 242: bool AVRFrameLowering::spillCalleeSavedRegisters(
 243:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 244:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 245:   if (CSI.empty()) {
 246:     return false;
 247:   }
 248: 
 249:   unsigned CalleeFrameSize = 0;
 250:   DebugLoc DL = MBB.findDebugLoc(MI);
 251:   MachineFunction &MF = *MBB.getParent();
 252:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 253:   const TargetInstrInfo &TII = *STI.getInstrInfo();
 254:   AVRMachineFunctionInfo *AVRFI = MF.getInfo<AVRMachineFunctionInfo>();
 255: 
 256:   for (const CalleeSavedInfo &I : llvm::reverse(CSI)) {
 257:     MCRegister Reg = I.getReg();
 258:     bool IsNotLiveIn = !MBB.isLiveIn(Reg);
 259: 
 260:     // Check if Reg is a sub register of a 16-bit livein register, and then
```

- EN: Function bodies or method definitions such as spillCalleeSavedRegisters contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: spillCalleeSavedRegisters 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 261-280

```cpp
 261:     // add it to the livein list.
 262:     if (IsNotLiveIn)
 263:       for (const auto &LiveIn : MBB.liveins())
 264:         if (STI.getRegisterInfo()->isSubRegister(LiveIn.PhysReg, Reg)) {
 265:           IsNotLiveIn = false;
 266:           MBB.addLiveIn(Reg);
 267:           break;
 268:         }
 269: 
 270:     assert(TRI->getRegSizeInBits(*TRI->getMinimalPhysRegClass(Reg)) == 8 &&
 271:            "Invalid register size");
 272: 
 273:     // Add the callee-saved register as live-in only if it is not already a
 274:     // live-in register, this usually happens with arguments that are passed
 275:     // through callee-saved registers.
 276:     if (IsNotLiveIn) {
 277:       MBB.addLiveIn(Reg);
 278:     }
 279: 
 280:     // Do not kill the register when it is an input argument.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 281-300

```cpp
 281:     BuildMI(MBB, MI, DL, TII.get(AVR::PUSHRr))
 282:         .addReg(Reg, getKillRegState(IsNotLiveIn))
 283:         .setMIFlag(MachineInstr::FrameSetup);
 284:     ++CalleeFrameSize;
 285:   }
 286: 
 287:   AVRFI->setCalleeSavedFrameSize(CalleeFrameSize);
 288: 
 289:   return true;
 290: }
 291: 
 292: bool AVRFrameLowering::restoreCalleeSavedRegisters(
 293:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 294:     MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 295:   if (CSI.empty()) {
 296:     return false;
 297:   }
 298: 
 299:   DebugLoc DL = MBB.findDebugLoc(MI);
 300:   const MachineFunction &MF = *MBB.getParent();
```

- EN: Function bodies or method definitions such as restoreCalleeSavedRegisters contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: restoreCalleeSavedRegisters 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 301-320

```cpp
 301:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 302:   const TargetInstrInfo &TII = *STI.getInstrInfo();
 303: 
 304:   for (const CalleeSavedInfo &CCSI : CSI) {
 305:     MCRegister Reg = CCSI.getReg();
 306: 
 307:     assert(TRI->getRegSizeInBits(*TRI->getMinimalPhysRegClass(Reg)) == 8 &&
 308:            "Invalid register size");
 309: 
 310:     BuildMI(MBB, MI, DL, TII.get(AVR::POPRd), Reg);
 311:   }
 312: 
 313:   return true;
 314: }
 315: 
 316: /// Replace pseudo store instructions that pass arguments through the stack with
 317: /// real instructions.
 318: static void fixStackStores(MachineBasicBlock &MBB,
 319:                            MachineBasicBlock::iterator StartMI,
 320:                            const TargetInstrInfo &TII) {
```

- EN: Function bodies or method definitions such as fixStackStores contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: fixStackStores 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 321-340

```cpp
 321:   // Iterate through the BB until we hit a call instruction or we reach the end.
 322:   for (MachineInstr &MI :
 323:        llvm::make_early_inc_range(llvm::make_range(StartMI, MBB.end()))) {
 324:     if (MI.isCall())
 325:       break;
 326: 
 327:     unsigned Opcode = MI.getOpcode();
 328: 
 329:     // Only care of pseudo store instructions where SP is the base pointer.
 330:     if (Opcode != AVR::STDSPQRr && Opcode != AVR::STDWSPQRr)
 331:       continue;
 332: 
 333:     assert(MI.getOperand(0).getReg() == AVR::SP &&
 334:            "SP is expected as base pointer");
 335: 
 336:     // Replace this instruction with a regular store. Use Y as the base
 337:     // pointer since it is guaranteed to contain a copy of SP.
 338:     unsigned STOpc =
 339:         (Opcode == AVR::STDWSPQRr) ? AVR::STDWPtrQRr : AVR::STDPtrQRr;
 340: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 341-360

```cpp
 341:     MI.setDesc(TII.get(STOpc));
 342:     MI.getOperand(0).setReg(AVR::R31R30);
 343:   }
 344: }
 345: 
 346: MachineBasicBlock::iterator AVRFrameLowering::eliminateCallFramePseudoInstr(
 347:     MachineFunction &MF, MachineBasicBlock &MBB,
 348:     MachineBasicBlock::iterator MI) const {
 349:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 350:   const AVRInstrInfo &TII = *STI.getInstrInfo();
 351: 
 352:   if (hasReservedCallFrame(MF)) {
 353:     return MBB.erase(MI);
 354:   }
 355: 
 356:   DebugLoc DL = MI->getDebugLoc();
 357:   unsigned int Opcode = MI->getOpcode();
 358:   int Amount = TII.getFrameSize(*MI);
 359: 
 360:   if (Amount == 0) {
```

- EN: Function bodies or method definitions such as eliminateCallFramePseudoInstr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: eliminateCallFramePseudoInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 361-380

```cpp
 361:     return MBB.erase(MI);
 362:   }
 363: 
 364:   assert(getStackAlign() == Align(1) && "Unsupported stack alignment");
 365: 
 366:   if (Opcode == TII.getCallFrameSetupOpcode()) {
 367:     // Update the stack pointer.
 368:     // In many cases this can be done far more efficiently by pushing the
 369:     // relevant values directly to the stack. However, doing that correctly
 370:     // (in the right order, possibly skipping some empty space for undef
 371:     // values, etc) is tricky and thus left to be optimized in the future.
 372:     BuildMI(MBB, MI, DL, TII.get(AVR::SPREAD), AVR::R31R30).addReg(AVR::SP);
 373: 
 374:     MachineInstr *New =
 375:         BuildMI(MBB, MI, DL, TII.get(AVR::SUBIWRdK), AVR::R31R30)
 376:             .addReg(AVR::R31R30, RegState::Kill)
 377:             .addImm(Amount);
 378:     New->getOperand(3).setIsDead();
 379: 
 380:     BuildMI(MBB, MI, DL, TII.get(AVR::SPWRITE), AVR::SP).addReg(AVR::R31R30);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 381-400

```cpp
 381: 
 382:     // Make sure the remaining stack stores are converted to real store
 383:     // instructions.
 384:     fixStackStores(MBB, MI, TII);
 385:   } else {
 386:     assert(Opcode == TII.getCallFrameDestroyOpcode());
 387: 
 388:     // Note that small stack changes could be implemented more efficiently
 389:     // with a few pop instructions instead of the 8-9 instructions now
 390:     // required.
 391: 
 392:     // Select the best opcode to adjust SP based on the offset size.
 393:     unsigned AddOpcode;
 394: 
 395:     if (isUInt<6>(Amount) && STI.hasADDSUBIW()) {
 396:       AddOpcode = AVR::ADIWRdK;
 397:     } else {
 398:       AddOpcode = AVR::SUBIWRdK;
 399:       Amount = -Amount;
 400:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 401-420

```cpp
 401: 
 402:     // Build the instruction sequence.
 403:     BuildMI(MBB, MI, DL, TII.get(AVR::SPREAD), AVR::R31R30).addReg(AVR::SP);
 404: 
 405:     MachineInstr *New = BuildMI(MBB, MI, DL, TII.get(AddOpcode), AVR::R31R30)
 406:                             .addReg(AVR::R31R30, RegState::Kill)
 407:                             .addImm(Amount);
 408:     New->getOperand(3).setIsDead();
 409: 
 410:     BuildMI(MBB, MI, DL, TII.get(AVR::SPWRITE), AVR::SP)
 411:         .addReg(AVR::R31R30, RegState::Kill);
 412:   }
 413: 
 414:   return MBB.erase(MI);
 415: }
 416: 
 417: void AVRFrameLowering::determineCalleeSaves(MachineFunction &MF,
 418:                                             BitVector &SavedRegs,
 419:                                             RegScavenger *RS) const {
 420:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
```

- EN: Function bodies or method definitions such as determineCalleeSaves contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: determineCalleeSaves 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 421-440

```cpp
 421: 
 422:   // If we have a frame pointer, the Y register needs to be saved as well.
 423:   if (hasFP(MF)) {
 424:     SavedRegs.set(AVR::R29);
 425:     SavedRegs.set(AVR::R28);
 426:   }
 427: }
 428: /// The frame analyzer pass.
 429: ///
 430: /// Scans the function for allocas and used arguments
 431: /// that are passed through the stack.
 432: struct AVRFrameAnalyzer : public MachineFunctionPass {
 433:   static char ID;
 434:   AVRFrameAnalyzer() : MachineFunctionPass(ID) {}
 435: 
 436:   bool runOnMachineFunction(MachineFunction &MF) override {
 437:     const MachineFrameInfo &MFI = MF.getFrameInfo();
 438:     AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
 439: 
 440:     // If there are no fixed frame indexes during this stage it means there
```

- EN: This chunk introduces interfaces or data structures such as AVRFrameAnalyzer, which organize the target-specific behavior exposed by the file. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一段引入了 AVRFrameAnalyzer 等接口或数据结构，用于组织该文件暴露的目标专用行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 441-460

```cpp
 441:     // are allocas present in the function.
 442:     if (MFI.getNumObjects() != MFI.getNumFixedObjects()) {
 443:       // Check for the type of allocas present in the function. We only care
 444:       // about fixed size allocas so do not give false positives if only
 445:       // variable sized allocas are present.
 446:       for (unsigned i = 0, e = MFI.getObjectIndexEnd(); i != e; ++i) {
 447:         // Variable sized objects have size 0.
 448:         if (MFI.getObjectSize(i)) {
 449:           AFI->setHasAllocas(true);
 450:           break;
 451:         }
 452:       }
 453:     }
 454: 
 455:     // If there are fixed frame indexes present, scan the function to see if
 456:     // they are really being used.
 457:     if (MFI.getNumFixedObjects() == 0) {
 458:       return false;
 459:     }
 460: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 461-480

```cpp
 461:     // Ok fixed frame indexes present, now scan the function to see if they
 462:     // are really being used, otherwise we can ignore them.
 463:     for (const MachineBasicBlock &BB : MF) {
 464:       for (const MachineInstr &MI : BB) {
 465:         int Opcode = MI.getOpcode();
 466: 
 467:         if ((Opcode != AVR::LDDRdPtrQ) && (Opcode != AVR::LDDWRdPtrQ) &&
 468:             (Opcode != AVR::STDPtrQRr) && (Opcode != AVR::STDWPtrQRr) &&
 469:             (Opcode != AVR::FRMIDX)) {
 470:           continue;
 471:         }
 472: 
 473:         for (const MachineOperand &MO : MI.operands()) {
 474:           if (!MO.isFI()) {
 475:             continue;
 476:           }
 477: 
 478:           if (MFI.isFixedObjectIndex(MO.getIndex())) {
 479:             AFI->setHasStackArgs(true);
 480:             return false;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 481-497

```cpp
 481:           }
 482:         }
 483:       }
 484:     }
 485: 
 486:     return false;
 487:   }
 488: 
 489:   StringRef getPassName() const override { return "AVR Frame Analyzer"; }
 490: };
 491: 
 492: char AVRFrameAnalyzer::ID = 0;
 493: 
 494: /// Creates instance of the frame analyzer pass.
 495: FunctionPass *createAVRFrameAnalyzerPass() { return new AVRFrameAnalyzer(); }
 496: 
 497: } // end of namespace llvm
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

## Key Concepts / 关键概念

- Prologue and epilogue emission / 序言与尾声生成
- Stack object layout / 栈对象布局
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRFrameLowering.h`, `AVR.h`, `AVRInstrInfo.h`, `AVRMachineFunctionInfo.h`, `AVRTargetMachine.h`, `MCTargetDesc/AVRMCTargetDesc.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `AVRFrameLowering.h`
