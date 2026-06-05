# CSKYFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYFrameLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the CSKY implementation of TargetFrameLowering class.
- 目的（中文）: 实现栈帧布局、函数序言/尾声生成以及相关调用序列细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYFrameLowering.cpp - CSKY Frame Information ------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the CSKY implementation of TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYFrameLowering.h"
  14: #include "CSKYMachineFunctionInfo.h"
  15: #include "CSKYSubtarget.h"
  16: #include "llvm/CodeGen/MachineConstantPool.h"
  17: #include "llvm/CodeGen/MachineFrameInfo.h"
  18: #include "llvm/CodeGen/MachineFunction.h"
  19: #include "llvm/CodeGen/MachineInstrBuilder.h"
  20: #include "llvm/CodeGen/MachineRegisterInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/RegisterScavenging.h"
  22: #include "llvm/IR/DiagnosticInfo.h"
  23: #include "llvm/MC/MCDwarf.h"
  24: 
  25: using namespace llvm;
  26: 
  27: #define DEBUG_TYPE "csky-frame-lowering"
  28: 
  29: // Returns the register used to hold the frame pointer.
  30: static Register getFPReg(const CSKYSubtarget &STI) { return CSKY::R8; }
  31: 
  32: // To avoid the BP value clobbered by a function call, we need to choose a
  33: // callee saved register to save the value.
  34: static Register getBPReg(const CSKYSubtarget &STI) { return CSKY::R7; }
  35: 
  36: bool CSKYFrameLowering::hasFPImpl(const MachineFunction &MF) const {
  37:   const TargetRegisterInfo *RegInfo = MF.getSubtarget().getRegisterInfo();
  38: 
  39:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  40:   return MF.getTarget().Options.DisableFramePointerElim(MF) ||
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as getFPReg, getBPReg, hasFPImpl contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 getFPReg, getBPReg, hasFPImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 41-60

```cpp
  41:          RegInfo->hasStackRealignment(MF) || MFI.hasVarSizedObjects() ||
  42:          MFI.isFrameAddressTaken();
  43: }
  44: 
  45: bool CSKYFrameLowering::hasBP(const MachineFunction &MF) const {
  46:   const MachineFrameInfo &MFI = MF.getFrameInfo();
  47: 
  48:   return MFI.hasVarSizedObjects();
  49: }
  50: 
  51: // Determines the size of the frame and maximum call frame size.
  52: void CSKYFrameLowering::determineFrameLayout(MachineFunction &MF) const {
  53:   MachineFrameInfo &MFI = MF.getFrameInfo();
  54:   const CSKYRegisterInfo *RI = STI.getRegisterInfo();
  55: 
  56:   // Get the number of bytes to allocate from the FrameInfo.
  57:   uint64_t FrameSize = MFI.getStackSize();
  58: 
  59:   // Get the alignment.
  60:   Align StackAlign = getStackAlign();
```

- EN: Function bodies or method definitions such as hasBP, determineFrameLayout contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: hasBP, determineFrameLayout 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 61-80

```cpp
  61:   if (RI->hasStackRealignment(MF)) {
  62:     Align MaxStackAlign = std::max(StackAlign, MFI.getMaxAlign());
  63:     FrameSize += (MaxStackAlign.value() - StackAlign.value());
  64:     StackAlign = MaxStackAlign;
  65:   }
  66: 
  67:   // Set Max Call Frame Size
  68:   uint64_t MaxCallSize = alignTo(MFI.getMaxCallFrameSize(), StackAlign);
  69:   MFI.setMaxCallFrameSize(MaxCallSize);
  70: 
  71:   // Make sure the frame is aligned.
  72:   FrameSize = alignTo(FrameSize, StackAlign);
  73: 
  74:   // Update frame info.
  75:   MFI.setStackSize(FrameSize);
  76: }
  77: 
  78: void CSKYFrameLowering::emitPrologue(MachineFunction &MF,
  79:                                      MachineBasicBlock &MBB) const {
  80:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
```

- EN: Function bodies or method definitions such as emitPrologue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitPrologue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 81-100

```cpp
  81:   MachineFrameInfo &MFI = MF.getFrameInfo();
  82:   const CSKYRegisterInfo *RI = STI.getRegisterInfo();
  83:   const CSKYInstrInfo *TII = STI.getInstrInfo();
  84:   MachineBasicBlock::iterator MBBI = MBB.begin();
  85:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
  86:   const MachineRegisterInfo &MRI = MF.getRegInfo();
  87: 
  88:   Register FPReg = getFPReg(STI);
  89:   Register SPReg = CSKY::R14;
  90:   Register BPReg = getBPReg(STI);
  91: 
  92:   // Debug location must be unknown since the first debug location is used
  93:   // to determine the end of the prologue.
  94:   DebugLoc DL;
  95: 
  96:   if (MF.getFunction().hasFnAttribute("interrupt"))
  97:     BuildMI(MBB, MBBI, DL, TII->get(CSKY::NIE));
  98: 
  99:   // Determine the correct frame layout
 100:   determineFrameLayout(MF);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 101-120

```cpp
 101: 
 102:   // FIXME (note copied from Lanai): This appears to be overallocating.  Needs
 103:   // investigation. Get the number of bytes to allocate from the FrameInfo.
 104:   uint64_t StackSize = MFI.getStackSize();
 105: 
 106:   // Early exit if there is no need to allocate on the stack
 107:   if (StackSize == 0 && !MFI.adjustsStack())
 108:     return;
 109: 
 110:   const auto &CSI = MFI.getCalleeSavedInfo();
 111: 
 112:   unsigned spillAreaSize = CFI->getCalleeSaveAreaSize();
 113: 
 114:   uint64_t ActualSize = spillAreaSize + CFI->getVarArgsSaveSize();
 115: 
 116:   // First part stack allocation.
 117:   adjustReg(MBB, MBBI, DL, SPReg, SPReg, -(static_cast<int64_t>(ActualSize)),
 118:             MachineInstr::NoFlags);
 119: 
 120:   // Emit ".cfi_def_cfa_offset FirstSPAdjustAmount"
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 121-140

```cpp
 121:   unsigned CFIIndex =
 122:       MF.addFrameInst(MCCFIInstruction::cfiDefCfaOffset(nullptr, ActualSize));
 123:   BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 124:       .addCFIIndex(CFIIndex);
 125: 
 126:   // The frame pointer is callee-saved, and code has been generated for us to
 127:   // save it to the stack. We need to skip over the storing of callee-saved
 128:   // registers as the frame pointer must be modified after it has been saved
 129:   // to the stack, not before.
 130:   // FIXME: assumes exactly one instruction is used to save each callee-saved
 131:   // register.
 132:   std::advance(MBBI, CSI.size());
 133: 
 134:   // Iterate over list of callee-saved registers and emit .cfi_offset
 135:   // directives.
 136:   for (const auto &Entry : CSI) {
 137:     int64_t Offset = MFI.getObjectOffset(Entry.getFrameIdx());
 138:     MCRegister Reg = Entry.getReg();
 139: 
 140:     unsigned Num = TRI->getRegSizeInBits(Reg, MRI) / 32;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 141-160

```cpp
 141:     for (unsigned i = 0; i < Num; i++) {
 142:       unsigned CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 143:           nullptr, RI->getDwarfRegNum(Reg, true) + i, Offset + i * 4));
 144:       BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 145:           .addCFIIndex(CFIIndex);
 146:     }
 147:   }
 148: 
 149:   // Generate new FP.
 150:   if (hasFP(MF)) {
 151:     BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::COPY), FPReg)
 152:         .addReg(SPReg)
 153:         .setMIFlag(MachineInstr::FrameSetup);
 154: 
 155:     // Emit ".cfi_def_cfa_register $fp"
 156:     unsigned CFIIndex = MF.addFrameInst(MCCFIInstruction::createDefCfaRegister(
 157:         nullptr, RI->getDwarfRegNum(FPReg, true)));
 158:     BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 159:         .addCFIIndex(CFIIndex);
 160: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 161-180

```cpp
 161:     // Second part stack allocation.
 162:     adjustReg(MBB, MBBI, DL, SPReg, SPReg,
 163:               -(static_cast<int64_t>(StackSize - ActualSize)),
 164:               MachineInstr::NoFlags);
 165: 
 166:     // Realign Stack
 167:     const CSKYRegisterInfo *RI = STI.getRegisterInfo();
 168:     if (RI->hasStackRealignment(MF)) {
 169:       Align MaxAlignment = MFI.getMaxAlign();
 170: 
 171:       const CSKYInstrInfo *TII = STI.getInstrInfo();
 172:       if (STI.hasE2() && isUInt<12>(~(-(int)MaxAlignment.value()))) {
 173:         BuildMI(MBB, MBBI, DL, TII->get(CSKY::ANDNI32), SPReg)
 174:             .addReg(SPReg)
 175:             .addImm(~(-(int)MaxAlignment.value()));
 176:       } else {
 177:         unsigned ShiftAmount = Log2(MaxAlignment);
 178: 
 179:         if (STI.hasE2()) {
 180:           Register VR =
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 181-200

```cpp
 181:               MF.getRegInfo().createVirtualRegister(&CSKY::GPRRegClass);
 182:           BuildMI(MBB, MBBI, DL, TII->get(CSKY::LSRI32), VR)
 183:               .addReg(SPReg)
 184:               .addImm(ShiftAmount);
 185:           BuildMI(MBB, MBBI, DL, TII->get(CSKY::LSLI32), SPReg)
 186:               .addReg(VR)
 187:               .addImm(ShiftAmount);
 188:         } else {
 189:           Register VR =
 190:               MF.getRegInfo().createVirtualRegister(&CSKY::mGPRRegClass);
 191:           BuildMI(MBB, MBBI, DL, TII->get(CSKY::MOV16), VR).addReg(SPReg);
 192:           BuildMI(MBB, MBBI, DL, TII->get(CSKY::LSRI16), VR)
 193:               .addReg(VR)
 194:               .addImm(ShiftAmount);
 195:           BuildMI(MBB, MBBI, DL, TII->get(CSKY::LSLI16), VR)
 196:               .addReg(VR)
 197:               .addImm(ShiftAmount);
 198:           BuildMI(MBB, MBBI, DL, TII->get(CSKY::MOV16), SPReg).addReg(VR);
 199:         }
 200:       }
```

- EN: At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 201-220

```cpp
 201:     }
 202: 
 203:     // FP will be used to restore the frame in the epilogue, so we need
 204:     // another base register BP to record SP after re-alignment. SP will
 205:     // track the current stack after allocating variable sized objects.
 206:     if (hasBP(MF)) {
 207:       // move BP, SP
 208:       BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::COPY), BPReg).addReg(SPReg);
 209:     }
 210: 
 211:   } else {
 212:     adjustReg(MBB, MBBI, DL, SPReg, SPReg,
 213:               -(static_cast<int64_t>(StackSize - ActualSize)),
 214:               MachineInstr::NoFlags);
 215:     // Emit ".cfi_def_cfa_offset StackSize"
 216:     unsigned CFIIndex = MF.addFrameInst(
 217:         MCCFIInstruction::cfiDefCfaOffset(nullptr, MFI.getStackSize()));
 218:     BuildMI(MBB, MBBI, DL, TII->get(TargetOpcode::CFI_INSTRUCTION))
 219:         .addCFIIndex(CFIIndex);
 220:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 221-240

```cpp
 221: }
 222: 
 223: void CSKYFrameLowering::emitEpilogue(MachineFunction &MF,
 224:                                      MachineBasicBlock &MBB) const {
 225:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
 226: 
 227:   MachineFrameInfo &MFI = MF.getFrameInfo();
 228:   Register FPReg = getFPReg(STI);
 229:   Register SPReg = CSKY::R14;
 230: 
 231:   // Get the insert location for the epilogue. If there were no terminators in
 232:   // the block, get the last instruction.
 233:   MachineBasicBlock::iterator MBBI = MBB.end();
 234:   DebugLoc DL;
 235:   if (!MBB.empty()) {
 236:     MBBI = MBB.getFirstTerminator();
 237:     if (MBBI == MBB.end())
 238:       MBBI = MBB.getLastNonDebugInstr();
 239:     DL = MBBI->getDebugLoc();
 240: 
```

- EN: Function bodies or method definitions such as emitEpilogue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitEpilogue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 241-260

```cpp
 241:     // If this is not a terminator, the actual insert location should be after
 242:     // the last instruction.
 243:     if (!MBBI->isTerminator())
 244:       MBBI = std::next(MBBI);
 245:   }
 246: 
 247:   const auto &CSI = MFI.getCalleeSavedInfo();
 248:   uint64_t StackSize = MFI.getStackSize();
 249: 
 250:   uint64_t ActualSize =
 251:       CFI->getCalleeSaveAreaSize() + CFI->getVarArgsSaveSize();
 252: 
 253:   // Skip to before the restores of callee-saved registers
 254:   // FIXME: assumes exactly one instruction is used to restore each
 255:   // callee-saved register.
 256:   auto LastFrameDestroy = MBBI;
 257:   if (!CSI.empty())
 258:     LastFrameDestroy = std::prev(MBBI, CSI.size());
 259: 
 260:   if (hasFP(MF)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 261-280

```cpp
 261:     const CSKYInstrInfo *TII = STI.getInstrInfo();
 262:     BuildMI(MBB, LastFrameDestroy, DL, TII->get(TargetOpcode::COPY), SPReg)
 263:         .addReg(FPReg)
 264:         .setMIFlag(MachineInstr::NoFlags);
 265:   } else {
 266:     adjustReg(MBB, LastFrameDestroy, DL, SPReg, SPReg, (StackSize - ActualSize),
 267:               MachineInstr::FrameDestroy);
 268:   }
 269: 
 270:   adjustReg(MBB, MBBI, DL, SPReg, SPReg, ActualSize,
 271:             MachineInstr::FrameDestroy);
 272: }
 273: 
 274: static unsigned EstimateFunctionSizeInBytes(const MachineFunction &MF,
 275:                                             const CSKYInstrInfo &TII) {
 276:   unsigned FnSize = 0;
 277:   for (auto &MBB : MF) {
 278:     for (auto &MI : MBB)
 279:       FnSize += TII.getInstSizeInBytes(MI);
 280:   }
```

- EN: Function bodies or method definitions such as EstimateFunctionSizeInBytes contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: EstimateFunctionSizeInBytes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 281-300

```cpp
 281:   FnSize += MF.getConstantPool()->getConstants().size() * 4;
 282:   return FnSize;
 283: }
 284: 
 285: static unsigned estimateRSStackSizeLimit(MachineFunction &MF,
 286:                                          const CSKYSubtarget &STI) {
 287:   unsigned Limit = (1 << 12) - 1;
 288: 
 289:   for (auto &MBB : MF) {
 290:     for (auto &MI : MBB) {
 291:       if (MI.isDebugInstr())
 292:         continue;
 293: 
 294:       for (unsigned i = 0, e = MI.getNumOperands(); i != e; ++i) {
 295:         if (!MI.getOperand(i).isFI())
 296:           continue;
 297: 
 298:         if (MI.getOpcode() == CSKY::SPILL_CARRY ||
 299:             MI.getOpcode() == CSKY::RESTORE_CARRY ||
 300:             MI.getOpcode() == CSKY::STORE_PAIR ||
```

- EN: Function bodies or method definitions such as estimateRSStackSizeLimit contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: estimateRSStackSizeLimit 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 301-320

```cpp
 301:             MI.getOpcode() == CSKY::LOAD_PAIR) {
 302:           Limit = std::min(Limit, ((1U << 12) - 1) * 4);
 303:           break;
 304:         }
 305: 
 306:         if (MI.getOpcode() == CSKY::ADDI32) {
 307:           Limit = std::min(Limit, (1U << 12));
 308:           break;
 309:         }
 310: 
 311:         if (MI.getOpcode() == CSKY::ADDI16XZ) {
 312:           Limit = std::min(Limit, (1U << 3));
 313:           break;
 314:         }
 315: 
 316:         // ADDI16 will not require an extra register,
 317:         // it can reuse the destination.
 318:         if (MI.getOpcode() == CSKY::ADDI16)
 319:           break;
 320: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 321-340

```cpp
 321:         // Otherwise check the addressing mode.
 322:         switch (MI.getDesc().TSFlags & CSKYII::AddrModeMask) {
 323:         default:
 324:           LLVM_DEBUG(MI.dump());
 325:           llvm_unreachable(
 326:               "Unhandled addressing mode in stack size limit calculation");
 327:         case CSKYII::AddrMode32B:
 328:           Limit = std::min(Limit, (1U << 12) - 1);
 329:           break;
 330:         case CSKYII::AddrMode32H:
 331:           Limit = std::min(Limit, ((1U << 12) - 1) * 2);
 332:           break;
 333:         case CSKYII::AddrMode32WD:
 334:           Limit = std::min(Limit, ((1U << 12) - 1) * 4);
 335:           break;
 336:         case CSKYII::AddrMode16B:
 337:           Limit = std::min(Limit, (1U << 5) - 1);
 338:           break;
 339:         case CSKYII::AddrMode16H:
 340:           Limit = std::min(Limit, ((1U << 5) - 1) * 2);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 341-360

```cpp
 341:           break;
 342:         case CSKYII::AddrMode16W:
 343:           Limit = std::min(Limit, ((1U << 5) - 1) * 4);
 344:           break;
 345:         case CSKYII::AddrMode32SDF:
 346:           Limit = std::min(Limit, ((1U << 8) - 1) * 4);
 347:           break;
 348:         }
 349:         break; // At most one FI per instruction
 350:       }
 351:     }
 352:   }
 353: 
 354:   return Limit;
 355: }
 356: 
 357: void CSKYFrameLowering::determineCalleeSaves(MachineFunction &MF,
 358:                                              BitVector &SavedRegs,
 359:                                              RegScavenger *RS) const {
 360:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
```

- EN: Function bodies or method definitions such as determineCalleeSaves contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: determineCalleeSaves 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 361-380

```cpp
 361: 
 362:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
 363:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
 364:   const CSKYInstrInfo *TII = STI.getInstrInfo();
 365:   const MachineRegisterInfo &MRI = MF.getRegInfo();
 366:   MachineFrameInfo &MFI = MF.getFrameInfo();
 367: 
 368:   if (hasFP(MF))
 369:     SavedRegs.set(CSKY::R8);
 370: 
 371:   // Mark BP as used if function has dedicated base pointer.
 372:   if (hasBP(MF))
 373:     SavedRegs.set(CSKY::R7);
 374: 
 375:   // If interrupt is enabled and there are calls in the handler,
 376:   // unconditionally save all Caller-saved registers and
 377:   // all FP registers, regardless whether they are used.
 378:   if (MF.getFunction().hasFnAttribute("interrupt") && MFI.hasCalls()) {
 379: 
 380:     static const MCPhysReg CSRegs[] = {CSKY::R0,  CSKY::R1,  CSKY::R2, CSKY::R3,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 381-400

```cpp
 381:                                        CSKY::R12, CSKY::R13, 0};
 382: 
 383:     for (unsigned i = 0; CSRegs[i]; ++i)
 384:       SavedRegs.set(CSRegs[i]);
 385: 
 386:     if (STI.hasHighRegisters()) {
 387: 
 388:       static const MCPhysReg CSHRegs[] = {CSKY::R18, CSKY::R19, CSKY::R20,
 389:                                           CSKY::R21, CSKY::R22, CSKY::R23,
 390:                                           CSKY::R24, CSKY::R25, 0};
 391: 
 392:       for (unsigned i = 0; CSHRegs[i]; ++i)
 393:         SavedRegs.set(CSHRegs[i]);
 394:     }
 395: 
 396:     static const MCPhysReg CSF32Regs[] = {
 397:         CSKY::F8_32,  CSKY::F9_32,  CSKY::F10_32,
 398:         CSKY::F11_32, CSKY::F12_32, CSKY::F13_32,
 399:         CSKY::F14_32, CSKY::F15_32, 0};
 400:     static const MCPhysReg CSF64Regs[] = {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 401-420

```cpp
 401:         CSKY::F8_64,  CSKY::F9_64,  CSKY::F10_64,
 402:         CSKY::F11_64, CSKY::F12_64, CSKY::F13_64,
 403:         CSKY::F14_64, CSKY::F15_64, 0};
 404: 
 405:     const MCPhysReg *FRegs = NULL;
 406:     if (STI.hasFPUv2DoubleFloat() || STI.hasFPUv3DoubleFloat())
 407:       FRegs = CSF64Regs;
 408:     else if (STI.hasFPUv2SingleFloat() || STI.hasFPUv3SingleFloat())
 409:       FRegs = CSF32Regs;
 410: 
 411:     if (FRegs != NULL) {
 412:       const MCPhysReg *Regs = MF.getRegInfo().getCalleeSavedRegs();
 413: 
 414:       for (unsigned i = 0; Regs[i]; ++i)
 415:         if (CSKY::FPR32RegClass.contains(Regs[i]) ||
 416:             CSKY::FPR64RegClass.contains(Regs[i])) {
 417:           unsigned x = 0;
 418:           for (; FRegs[x]; ++x)
 419:             if (FRegs[x] == Regs[i])
 420:               break;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 421-440

```cpp
 421:           if (FRegs[x] == 0)
 422:             SavedRegs.set(Regs[i]);
 423:         }
 424:     }
 425:   }
 426: 
 427:   unsigned CSStackSize = 0;
 428:   for (unsigned Reg : SavedRegs.set_bits()) {
 429:     auto RegSize = TRI->getRegSizeInBits(Reg, MRI) / 8;
 430:     CSStackSize += RegSize;
 431:   }
 432: 
 433:   CFI->setCalleeSaveAreaSize(CSStackSize);
 434: 
 435:   uint64_t Limit = estimateRSStackSizeLimit(MF, STI);
 436: 
 437:   bool BigFrame = (MFI.estimateStackSize(MF) + CSStackSize >= Limit);
 438: 
 439:   if (BigFrame || CFI->isCRSpilled() || !STI.hasE2()) {
 440:     const TargetRegisterClass *RC = &CSKY::GPRRegClass;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 441-460

```cpp
 441:     unsigned size = TRI->getSpillSize(*RC);
 442:     Align align = TRI->getSpillAlign(*RC);
 443: 
 444:     RS->addScavengingFrameIndex(MFI.CreateSpillStackObject(size, align));
 445:   }
 446: 
 447:   unsigned FnSize = EstimateFunctionSizeInBytes(MF, *TII);
 448:   // Force R15 to be spilled if the function size is > 65534. This enables
 449:   // use of BSR to implement far jump.
 450:   if (FnSize >= ((1 << (16 - 1)) * 2))
 451:     SavedRegs.set(CSKY::R15);
 452: 
 453:   CFI->setLRIsSpilled(SavedRegs.test(CSKY::R15));
 454: }
 455: 
 456: // Not preserve stack space within prologue for outgoing variables when the
 457: // function contains variable size objects and let eliminateCallFramePseudoInstr
 458: // preserve stack space for it.
 459: bool CSKYFrameLowering::hasReservedCallFrame(const MachineFunction &MF) const {
 460:   return !MF.getFrameInfo().hasVarSizedObjects();
```

- EN: Function bodies or method definitions such as hasReservedCallFrame contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: hasReservedCallFrame 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 461-480

```cpp
 461: }
 462: 
 463: bool CSKYFrameLowering::spillCalleeSavedRegisters(
 464:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 465:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 466:   if (CSI.empty())
 467:     return true;
 468: 
 469:   MachineFunction *MF = MBB.getParent();
 470:   const TargetInstrInfo &TII = *MF->getSubtarget().getInstrInfo();
 471:   DebugLoc DL;
 472:   if (MI != MBB.end() && !MI->isDebugInstr())
 473:     DL = MI->getDebugLoc();
 474: 
 475:   for (auto &CS : CSI) {
 476:     // Insert the spill to the stack frame.
 477:     MCRegister Reg = CS.getReg();
 478:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(Reg);
 479:     TII.storeRegToStackSlot(MBB, MI, Reg, true, CS.getFrameIdx(), RC,
 480:                             Register());
```

- EN: Function bodies or method definitions such as spillCalleeSavedRegisters contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: spillCalleeSavedRegisters 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 481-500

```cpp
 481:   }
 482: 
 483:   return true;
 484: }
 485: 
 486: bool CSKYFrameLowering::restoreCalleeSavedRegisters(
 487:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 488:     MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 489:   if (CSI.empty())
 490:     return true;
 491: 
 492:   MachineFunction *MF = MBB.getParent();
 493:   const TargetInstrInfo &TII = *MF->getSubtarget().getInstrInfo();
 494:   DebugLoc DL;
 495:   if (MI != MBB.end() && !MI->isDebugInstr())
 496:     DL = MI->getDebugLoc();
 497: 
 498:   for (auto &CS : reverse(CSI)) {
 499:     MCRegister Reg = CS.getReg();
 500:     const TargetRegisterClass *RC = TRI->getMinimalPhysRegClass(Reg);
```

- EN: Function bodies or method definitions such as restoreCalleeSavedRegisters contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: restoreCalleeSavedRegisters 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 501-520

```cpp
 501:     TII.loadRegFromStackSlot(MBB, MI, Reg, CS.getFrameIdx(), RC, Register());
 502:     assert(MI != MBB.begin() && "loadRegFromStackSlot didn't insert any code!");
 503:   }
 504: 
 505:   return true;
 506: }
 507: 
 508: // Eliminate ADJCALLSTACKDOWN, ADJCALLSTACKUP pseudo instructions.
 509: MachineBasicBlock::iterator CSKYFrameLowering::eliminateCallFramePseudoInstr(
 510:     MachineFunction &MF, MachineBasicBlock &MBB,
 511:     MachineBasicBlock::iterator MI) const {
 512:   Register SPReg = CSKY::R14;
 513:   DebugLoc DL = MI->getDebugLoc();
 514: 
 515:   if (!hasReservedCallFrame(MF)) {
 516:     // If space has not been reserved for a call frame, ADJCALLSTACKDOWN and
 517:     // ADJCALLSTACKUP must be converted to instructions manipulating the stack
 518:     // pointer. This is necessary when there is a variable length stack
 519:     // allocation (e.g. alloca), which means it's not possible to allocate
 520:     // space for outgoing arguments from within the function prologue.
```

- EN: Function bodies or method definitions such as eliminateCallFramePseudoInstr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: eliminateCallFramePseudoInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 521-540

```cpp
 521:     int64_t Amount = MI->getOperand(0).getImm();
 522: 
 523:     if (Amount != 0) {
 524:       // Ensure the stack remains aligned after adjustment.
 525:       Amount = alignSPAdjust(Amount);
 526: 
 527:       if (MI->getOpcode() == CSKY::ADJCALLSTACKDOWN)
 528:         Amount = -Amount;
 529: 
 530:       adjustReg(MBB, MI, DL, SPReg, SPReg, Amount, MachineInstr::NoFlags);
 531:     }
 532:   }
 533: 
 534:   return MBB.erase(MI);
 535: }
 536: 
 537: void CSKYFrameLowering::adjustReg(MachineBasicBlock &MBB,
 538:                                   MachineBasicBlock::iterator MBBI,
 539:                                   const DebugLoc &DL, Register DestReg,
 540:                                   Register SrcReg, int64_t Val,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 541-560

```cpp
 541:                                   MachineInstr::MIFlag Flag) const {
 542:   const CSKYInstrInfo *TII = STI.getInstrInfo();
 543: 
 544:   if (DestReg == SrcReg && Val == 0)
 545:     return;
 546: 
 547:   // TODO: Add 16-bit instruction support with immediate num
 548:   if (STI.hasE2() && isUInt<12>(std::abs(Val) - 1)) {
 549:     BuildMI(MBB, MBBI, DL, TII->get(Val < 0 ? CSKY::SUBI32 : CSKY::ADDI32),
 550:             DestReg)
 551:         .addReg(SrcReg)
 552:         .addImm(std::abs(Val))
 553:         .setMIFlag(Flag);
 554:   } else if (!STI.hasE2() && isShiftedUInt<7, 2>(std::abs(Val))) {
 555:     BuildMI(MBB, MBBI, DL,
 556:             TII->get(Val < 0 ? CSKY::SUBI16SPSP : CSKY::ADDI16SPSP), CSKY::R14)
 557:         .addReg(CSKY::R14, RegState::Kill)
 558:         .addImm(std::abs(Val))
 559:         .setMIFlag(Flag);
 560:   } else {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 561-580

```cpp
 561: 
 562:     unsigned Op = 0;
 563: 
 564:     if (STI.hasE2()) {
 565:       Op = Val < 0 ? CSKY::SUBU32 : CSKY::ADDU32;
 566:     } else {
 567:       assert(SrcReg == DestReg);
 568:       Op = Val < 0 ? CSKY::SUBU16XZ : CSKY::ADDU16XZ;
 569:     }
 570: 
 571:     Register ScratchReg = TII->movImm(MBB, MBBI, DL, std::abs(Val), Flag);
 572: 
 573:     BuildMI(MBB, MBBI, DL, TII->get(Op), DestReg)
 574:         .addReg(SrcReg)
 575:         .addReg(ScratchReg, RegState::Kill)
 576:         .setMIFlag(Flag);
 577:   }
 578: }
 579: 
 580: StackOffset
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 581-600

```cpp
 581: CSKYFrameLowering::getFrameIndexReference(const MachineFunction &MF, int FI,
 582:                                           Register &FrameReg) const {
 583:   const CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
 584:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 585:   const TargetRegisterInfo *RI = MF.getSubtarget().getRegisterInfo();
 586:   const auto &CSI = MFI.getCalleeSavedInfo();
 587: 
 588:   int MinCSFI = 0;
 589:   int MaxCSFI = -1;
 590: 
 591:   int Offset = MFI.getObjectOffset(FI) + MFI.getOffsetAdjustment();
 592: 
 593:   if (CSI.size()) {
 594:     MinCSFI = CSI[0].getFrameIdx();
 595:     MaxCSFI = CSI[CSI.size() - 1].getFrameIdx();
 596:   }
 597: 
 598:   if (FI >= MinCSFI && FI <= MaxCSFI) {
 599:     FrameReg = CSKY::R14;
 600:     Offset += CFI->getVarArgsSaveSize() + CFI->getCalleeSaveAreaSize();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 601-620

```cpp
 601:   } else if (RI->hasStackRealignment(MF)) {
 602:     assert(hasFP(MF));
 603:     if (!MFI.isFixedObjectIndex(FI)) {
 604:       FrameReg = hasBP(MF) ? getBPReg(STI) : CSKY::R14;
 605:       Offset += MFI.getStackSize();
 606:     } else {
 607:       FrameReg = getFPReg(STI);
 608:       Offset += CFI->getVarArgsSaveSize() + CFI->getCalleeSaveAreaSize();
 609:     }
 610:   } else {
 611:     if (MFI.isFixedObjectIndex(FI) && hasFP(MF)) {
 612:       FrameReg = getFPReg(STI);
 613:       Offset += CFI->getVarArgsSaveSize() + CFI->getCalleeSaveAreaSize();
 614:     } else {
 615:       FrameReg = hasBP(MF) ? getBPReg(STI) : CSKY::R14;
 616:       Offset += MFI.getStackSize();
 617:     }
 618:   }
 619: 
 620:   return StackOffset::getFixed(Offset);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 621-621

```cpp
 621: }
```

- EN: At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

## Key Concepts / 关键概念

- Prologue and epilogue emission / 序言与尾声生成
- Stack object layout / 栈对象布局
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYFrameLowering.h`, `CSKYMachineFunctionInfo.h`, `CSKYSubtarget.h`, `llvm/CodeGen/MachineConstantPool.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `CSKYFrameLowering.h`
