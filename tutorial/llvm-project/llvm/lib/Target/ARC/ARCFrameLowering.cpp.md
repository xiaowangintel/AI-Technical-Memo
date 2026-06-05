# ARCFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCFrameLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the ARC implementation of the TargetFrameLowering class.
- 目的（中文）: 实现栈帧布局、函数序言/尾声生成以及相关调用序列细节。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCFrameLowering.cpp - ARC Frame Information -------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the ARC implementation of the TargetFrameLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "ARCFrameLowering.h"
  14: #include "ARCMachineFunctionInfo.h"
  15: #include "ARCSubtarget.h"
  16: #include "llvm/CodeGen/MachineInstrBuilder.h"
  17: #include "llvm/CodeGen/MachineModuleInfo.h"
  18: #include "llvm/CodeGen/RegisterScavenging.h"
  19: #include "llvm/CodeGen/TargetRegisterInfo.h"
  20: #include "llvm/IR/Function.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 21-40

```cpp
  21: #include "llvm/Support/Debug.h"
  22: 
  23: #define DEBUG_TYPE "arc-frame-lowering"
  24: 
  25: using namespace llvm;
  26: 
  27: static cl::opt<bool>
  28:     UseSaveRestoreFunclet("arc-save-restore-funclet", cl::Hidden,
  29:                           cl::desc("Use arc callee save/restore functions"),
  30:                           cl::init(true));
  31: 
  32: static const char *store_funclet_name[] = {
  33:     "__st_r13_to_r15", "__st_r13_to_r16", "__st_r13_to_r17", "__st_r13_to_r18",
  34:     "__st_r13_to_r19", "__st_r13_to_r20", "__st_r13_to_r21", "__st_r13_to_r22",
  35:     "__st_r13_to_r23", "__st_r13_to_r24", "__st_r13_to_r25",
  36: };
  37: 
  38: static const char *load_funclet_name[] = {
  39:     "__ld_r13_to_r15", "__ld_r13_to_r16", "__ld_r13_to_r17", "__ld_r13_to_r18",
  40:     "__ld_r13_to_r19", "__ld_r13_to_r20", "__ld_r13_to_r21", "__ld_r13_to_r22",
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 41-60

```cpp
  41:     "__ld_r13_to_r23", "__ld_r13_to_r24", "__ld_r13_to_r25",
  42: };
  43: 
  44: static void generateStackAdjustment(MachineBasicBlock &MBB,
  45:                                     MachineBasicBlock::iterator MBBI,
  46:                                     const ARCInstrInfo &TII, DebugLoc dl,
  47:                                     int Amount, int StackPtr) {
  48:   unsigned AdjOp;
  49:   if (!Amount)
  50:     return;
  51:   bool Positive;
  52:   unsigned AbsAmount;
  53:   if (Amount < 0) {
  54:     AbsAmount = -Amount;
  55:     Positive = false;
  56:   } else {
  57:     AbsAmount = Amount;
  58:     Positive = true;
  59:   }
  60: 
```

- EN: Function bodies or method definitions such as generateStackAdjustment contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: generateStackAdjustment 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 61-80

```cpp
  61:   LLVM_DEBUG(dbgs() << "Internal: adjust stack by: " << Amount << ","
  62:                     << AbsAmount << "\n");
  63: 
  64:   assert((AbsAmount % 4 == 0) && "Stack adjustments must be 4-byte aligned.");
  65:   if (isUInt<6>(AbsAmount))
  66:     AdjOp = Positive ? ARC::ADD_rru6 : ARC::SUB_rru6;
  67:   else if (isInt<12>(AbsAmount))
  68:     AdjOp = Positive ? ARC::ADD_rrs12 : ARC::SUB_rrs12;
  69:   else
  70:     AdjOp = Positive ? ARC::ADD_rrlimm : ARC::SUB_rrlimm;
  71: 
  72:   BuildMI(MBB, MBBI, dl, TII.get(AdjOp), StackPtr)
  73:       .addReg(StackPtr)
  74:       .addImm(AbsAmount);
  75: }
  76: 
  77: static unsigned determineLastCalleeSave(ArrayRef<CalleeSavedInfo> CSI) {
  78:   unsigned Last = 0;
  79:   for (auto Reg : CSI) {
  80:     assert(Reg.getReg() >= ARC::R13 && Reg.getReg() <= ARC::R25 &&
```

- EN: Function bodies or method definitions such as determineLastCalleeSave contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: determineLastCalleeSave 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 81-100

```cpp
  81:            "Unexpected callee saved reg.");
  82:     if (Reg.getReg() > Last)
  83:       Last = Reg.getReg();
  84:   }
  85:   return Last;
  86: }
  87: 
  88: void ARCFrameLowering::determineCalleeSaves(MachineFunction &MF,
  89:                                             BitVector &SavedRegs,
  90:                                             RegScavenger *RS) const {
  91:   LLVM_DEBUG(dbgs() << "Determine Callee Saves: " << MF.getName() << "\n");
  92:   TargetFrameLowering::determineCalleeSaves(MF, SavedRegs, RS);
  93:   SavedRegs.set(ARC::BLINK);
  94: }
  95: 
  96: void ARCFrameLowering::adjustStackToMatchRecords(
  97:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
  98:     bool Allocate) const {
  99:   MachineFunction &MF = *MBB.getParent();
 100:   int ScalarAlloc = MF.getFrameInfo().getStackSize();
```

- EN: Function bodies or method definitions such as determineCalleeSaves, adjustStackToMatchRecords contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: determineCalleeSaves, adjustStackToMatchRecords 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 101-120

```cpp
 101: 
 102:   if (Allocate) {
 103:     // Allocate by adjusting by the negative of what the record holder tracked
 104:     // it tracked a positive offset in a downward growing stack.
 105:     ScalarAlloc = -ScalarAlloc;
 106:   }
 107: 
 108:   generateStackAdjustment(MBB, MBBI, *ST.getInstrInfo(), DebugLoc(),
 109:                           ScalarAlloc, ARC::SP);
 110: }
 111: 
 112: /// Insert prolog code into the function.
 113: /// For ARC, this inserts a call to a function that puts required callee saved
 114: /// registers onto the stack, when enough callee saved registers are required.
 115: void ARCFrameLowering::emitPrologue(MachineFunction &MF,
 116:                                     MachineBasicBlock &MBB) const {
 117:   LLVM_DEBUG(dbgs() << "Emit Prologue: " << MF.getName() << "\n");
 118:   auto *AFI = MF.getInfo<ARCFunctionInfo>();
 119:   MCContext &Context = MF.getContext();
 120:   const MCRegisterInfo *MRI = Context.getRegisterInfo();
```

- EN: Function bodies or method definitions such as emitPrologue contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitPrologue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 121-140

```cpp
 121:   const ARCInstrInfo *TII = MF.getSubtarget<ARCSubtarget>().getInstrInfo();
 122:   MachineBasicBlock::iterator MBBI = MBB.begin();
 123:   // Debug location must be unknown since the first debug location is used
 124:   // to determine the end of the prologue.
 125:   DebugLoc dl;
 126:   MachineFrameInfo &MFI = MF.getFrameInfo();
 127:   const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
 128:   unsigned Last = determineLastCalleeSave(CSI);
 129:   unsigned StackSlotsUsedByFunclet = 0;
 130:   bool SavedBlink = false;
 131:   unsigned AlreadyAdjusted = 0;
 132:   if (MF.getFunction().isVarArg()) {
 133:     // Add in the varargs area here first.
 134:     LLVM_DEBUG(dbgs() << "Varargs\n");
 135:     unsigned VarArgsBytes = MFI.getObjectSize(AFI->getVarArgsFrameIndex());
 136:     unsigned Opc = ARC::SUB_rrlimm;
 137:     if (isUInt<6>(VarArgsBytes))
 138:       Opc = ARC::SUB_rru6;
 139:     else if (isInt<12>(VarArgsBytes))
 140:       Opc = ARC::SUB_rrs12;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 141-160

```cpp
 141:     BuildMI(MBB, MBBI, dl, TII->get(Opc), ARC::SP)
 142:         .addReg(ARC::SP)
 143:         .addImm(VarArgsBytes);
 144:   }
 145:   if (hasFP(MF)) {
 146:     LLVM_DEBUG(dbgs() << "Saving FP\n");
 147:     BuildMI(MBB, MBBI, dl, TII->get(ARC::ST_AW_rs9))
 148:         .addReg(ARC::SP, RegState::Define)
 149:         .addReg(ARC::FP)
 150:         .addReg(ARC::SP)
 151:         .addImm(-4);
 152:     AlreadyAdjusted += 4;
 153:   }
 154:   if (UseSaveRestoreFunclet && Last > ARC::R14) {
 155:     LLVM_DEBUG(dbgs() << "Creating store funclet.\n");
 156:     // BL to __save_r13_to_<TRI->getRegAsmName()>
 157:     StackSlotsUsedByFunclet = Last - ARC::R12;
 158:     BuildMI(MBB, MBBI, dl, TII->get(ARC::PUSH_S_BLINK));
 159:     BuildMI(MBB, MBBI, dl, TII->get(ARC::SUB_rru6))
 160:         .addReg(ARC::SP)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 161-180

```cpp
 161:         .addReg(ARC::SP)
 162:         .addImm(4 * StackSlotsUsedByFunclet);
 163:     BuildMI(MBB, MBBI, dl, TII->get(ARC::BL))
 164:         .addExternalSymbol(store_funclet_name[Last - ARC::R15])
 165:         .addReg(ARC::BLINK, RegState::Implicit | RegState::Kill);
 166:     AlreadyAdjusted += 4 * (StackSlotsUsedByFunclet + 1);
 167:     SavedBlink = true;
 168:   }
 169:   // If we haven't saved BLINK, but we need to...do that now.
 170:   if (MFI.hasCalls() && !SavedBlink) {
 171:     LLVM_DEBUG(dbgs() << "Creating save blink.\n");
 172:     BuildMI(MBB, MBBI, dl, TII->get(ARC::PUSH_S_BLINK));
 173:     AlreadyAdjusted += 4;
 174:   }
 175:   if (AFI->MaxCallStackReq > 0)
 176:     MFI.setStackSize(MFI.getStackSize() + AFI->MaxCallStackReq);
 177:   // We have already saved some of the stack...
 178:   LLVM_DEBUG(dbgs() << "Adjusting stack by: "
 179:                     << (MFI.getStackSize() - AlreadyAdjusted) << "\n");
 180:   generateStackAdjustment(MBB, MBBI, *ST.getInstrInfo(), dl,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 181-200

```cpp
 181:                           -(MFI.getStackSize() - AlreadyAdjusted), ARC::SP);
 182: 
 183:   if (hasFP(MF)) {
 184:     LLVM_DEBUG(dbgs() << "Setting FP from SP.\n");
 185:     BuildMI(MBB, MBBI, dl,
 186:             TII->get(isUInt<6>(MFI.getStackSize()) ? ARC::ADD_rru6
 187:                                                    : ARC::ADD_rrlimm),
 188:             ARC::FP)
 189:         .addReg(ARC::SP)
 190:         .addImm(MFI.getStackSize());
 191:   }
 192: 
 193:   // Emit CFI records:
 194:   // .cfi_def_cfa_offset StackSize
 195:   // .cfi_offset fp, -StackSize
 196:   // .cfi_offset blink, -StackSize+4
 197:   unsigned CFIIndex = MF.addFrameInst(
 198:       MCCFIInstruction::cfiDefCfaOffset(nullptr, MFI.getStackSize()));
 199:   BuildMI(MBB, MBBI, dl, TII->get(TargetOpcode::CFI_INSTRUCTION))
 200:       .addCFIIndex(CFIIndex)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 201-220

```cpp
 201:       .setMIFlags(MachineInstr::FrameSetup);
 202: 
 203:   int CurOffset = -4;
 204:   if (hasFP(MF)) {
 205:     CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 206:         nullptr, MRI->getDwarfRegNum(ARC::FP, true), CurOffset));
 207:     BuildMI(MBB, MBBI, dl, TII->get(TargetOpcode::CFI_INSTRUCTION))
 208:         .addCFIIndex(CFIIndex)
 209:         .setMIFlags(MachineInstr::FrameSetup);
 210:     CurOffset -= 4;
 211:   }
 212: 
 213:   if (MFI.hasCalls()) {
 214:     CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 215:         nullptr, MRI->getDwarfRegNum(ARC::BLINK, true), CurOffset));
 216:     BuildMI(MBB, MBBI, dl, TII->get(TargetOpcode::CFI_INSTRUCTION))
 217:         .addCFIIndex(CFIIndex)
 218:         .setMIFlags(MachineInstr::FrameSetup);
 219:   }
 220:   // CFI for the rest of the registers.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 221-240

```cpp
 221:   for (const auto &Entry : CSI) {
 222:     MCRegister Reg = Entry.getReg();
 223:     int FI = Entry.getFrameIdx();
 224:     // Skip BLINK and FP.
 225:     if ((hasFP(MF) && Reg == ARC::FP) || (MFI.hasCalls() && Reg == ARC::BLINK))
 226:       continue;
 227:     CFIIndex = MF.addFrameInst(MCCFIInstruction::createOffset(
 228:         nullptr, MRI->getDwarfRegNum(Reg, true), MFI.getObjectOffset(FI)));
 229:     BuildMI(MBB, MBBI, dl, TII->get(TargetOpcode::CFI_INSTRUCTION))
 230:         .addCFIIndex(CFIIndex)
 231:         .setMIFlags(MachineInstr::FrameSetup);
 232:   }
 233: }
 234: 
 235: /// Insert epilog code into the function.
 236: /// For ARC, this inserts a call to a function that restores callee saved
 237: /// registers onto the stack, when enough callee saved registers are required.
 238: void ARCFrameLowering::emitEpilogue(MachineFunction &MF,
 239:                                     MachineBasicBlock &MBB) const {
 240:   LLVM_DEBUG(dbgs() << "Emit Epilogue: " << MF.getName() << "\n");
```

- EN: Function bodies or method definitions such as emitEpilogue contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitEpilogue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 241-260

```cpp
 241:   auto *AFI = MF.getInfo<ARCFunctionInfo>();
 242:   const ARCInstrInfo *TII = MF.getSubtarget<ARCSubtarget>().getInstrInfo();
 243:   MachineBasicBlock::iterator MBBI = MBB.getFirstTerminator();
 244:   MachineFrameInfo &MFI = MF.getFrameInfo();
 245:   uint64_t StackSize = MF.getFrameInfo().getStackSize();
 246:   bool SavedBlink = false;
 247:   unsigned AmountAboveFunclet = 0;
 248:   // If we have variable sized frame objects, then we have to move
 249:   // the stack pointer to a known spot (fp - StackSize).
 250:   // Then, replace the frame pointer by (new) [sp,StackSize-4].
 251:   // Then, move the stack pointer the rest of the way (sp = sp + StackSize).
 252:   if (hasFP(MF)) {
 253:     unsigned Opc = ARC::SUB_rrlimm;
 254:     if (isUInt<6>(StackSize))
 255:       Opc = ARC::SUB_rru6;
 256:     BuildMI(MBB, MBBI, DebugLoc(), TII->get(Opc), ARC::SP)
 257:         .addReg(ARC::FP)
 258:         .addImm(StackSize);
 259:     AmountAboveFunclet += 4;
 260:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 261-280

```cpp
 261: 
 262:   // Now, move the stack pointer to the bottom of the save area for the funclet.
 263:   const std::vector<CalleeSavedInfo> &CSI = MFI.getCalleeSavedInfo();
 264:   unsigned Last = determineLastCalleeSave(CSI);
 265:   unsigned StackSlotsUsedByFunclet = 0;
 266:   // Now, restore the callee save registers.
 267:   if (UseSaveRestoreFunclet && Last > ARC::R14) {
 268:     // BL to __ld_r13_to_<TRI->getRegAsmName()>
 269:     StackSlotsUsedByFunclet = Last - ARC::R12;
 270:     AmountAboveFunclet += 4 * (StackSlotsUsedByFunclet + 1);
 271:     SavedBlink = true;
 272:   }
 273: 
 274:   if (MFI.hasCalls() && !SavedBlink) {
 275:     AmountAboveFunclet += 4;
 276:     SavedBlink = true;
 277:   }
 278: 
 279:   // Move the stack pointer up to the point of the funclet.
 280:   if (unsigned MoveAmount = StackSize - AmountAboveFunclet) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 281-300

```cpp
 281:     unsigned Opc = ARC::ADD_rrlimm;
 282:     if (isUInt<6>(MoveAmount))
 283:       Opc = ARC::ADD_rru6;
 284:     else if (isInt<12>(MoveAmount))
 285:       Opc = ARC::ADD_rrs12;
 286:     BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII->get(Opc), ARC::SP)
 287:         .addReg(ARC::SP)
 288:         .addImm(StackSize - AmountAboveFunclet);
 289:   }
 290: 
 291:   if (StackSlotsUsedByFunclet) {
 292:     // This part of the adjustment will always be < 64 bytes.
 293:     BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII->get(ARC::BL))
 294:         .addExternalSymbol(load_funclet_name[Last - ARC::R15])
 295:         .addReg(ARC::BLINK, RegState::Implicit | RegState::Kill);
 296:     unsigned Opc = ARC::ADD_rrlimm;
 297:     if (isUInt<6>(4 * StackSlotsUsedByFunclet))
 298:       Opc = ARC::ADD_rru6;
 299:     else if (isInt<12>(4 * StackSlotsUsedByFunclet))
 300:       Opc = ARC::ADD_rrs12;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 301-320

```cpp
 301:     BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII->get(Opc), ARC::SP)
 302:         .addReg(ARC::SP)
 303:         .addImm(4 * (StackSlotsUsedByFunclet));
 304:   }
 305:   // Now, pop blink if necessary.
 306:   if (SavedBlink) {
 307:     BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII->get(ARC::POP_S_BLINK));
 308:   }
 309:   // Now, pop fp if necessary.
 310:   if (hasFP(MF)) {
 311:     BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII->get(ARC::LD_AB_rs9))
 312:         .addReg(ARC::FP, RegState::Define)
 313:         .addReg(ARC::SP, RegState::Define)
 314:         .addReg(ARC::SP)
 315:         .addImm(4);
 316:   }
 317: 
 318:   // Relieve the varargs area if necessary.
 319:   if (MF.getFunction().isVarArg()) {
 320:     // Add in the varargs area here first.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 321-340

```cpp
 321:     LLVM_DEBUG(dbgs() << "Varargs\n");
 322:     unsigned VarArgsBytes = MFI.getObjectSize(AFI->getVarArgsFrameIndex());
 323:     unsigned Opc = ARC::ADD_rrlimm;
 324:     if (isUInt<6>(VarArgsBytes))
 325:       Opc = ARC::ADD_rru6;
 326:     else if (isInt<12>(VarArgsBytes))
 327:       Opc = ARC::ADD_rrs12;
 328:     BuildMI(MBB, MBBI, MBB.findDebugLoc(MBBI), TII->get(Opc))
 329:         .addReg(ARC::SP)
 330:         .addReg(ARC::SP)
 331:         .addImm(VarArgsBytes);
 332:   }
 333: }
 334: 
 335: static std::vector<CalleeSavedInfo>::iterator
 336: getSavedReg(std::vector<CalleeSavedInfo> &V, unsigned reg) {
 337:   for (auto I = V.begin(), E = V.end(); I != E; ++I) {
 338:     if (reg == I->getReg())
 339:       return I;
 340:   }
```

- EN: Function bodies or method definitions such as getSavedReg contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: getSavedReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 341-360

```cpp
 341:   return V.end();
 342: }
 343: 
 344: bool ARCFrameLowering::assignCalleeSavedSpillSlots(
 345:     MachineFunction &MF, const TargetRegisterInfo *TRI,
 346:     std::vector<CalleeSavedInfo> &CSI) const {
 347:   // Use this opportunity to assign the spill slots for all of the potential
 348:   // callee save registers (blink, fp, r13->r25) that we care about the
 349:   // placement for.  We can calculate all of that data here.
 350:   int CurOffset = -4;
 351:   unsigned Last = determineLastCalleeSave(CSI);
 352:   MachineFrameInfo &MFI = MF.getFrameInfo();
 353:   if (hasFP(MF)) {
 354:     // Create a fixed slot at for FP
 355:     int StackObj = MFI.CreateFixedSpillStackObject(4, CurOffset, true);
 356:     LLVM_DEBUG(dbgs() << "Creating fixed object (" << StackObj << ") for FP at "
 357:                       << CurOffset << "\n");
 358:     (void)StackObj;
 359:     CurOffset -= 4;
 360:   }
```

- EN: Function bodies or method definitions such as assignCalleeSavedSpillSlots contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: assignCalleeSavedSpillSlots 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 361-380

```cpp
 361:   if (MFI.hasCalls() || (UseSaveRestoreFunclet && Last > ARC::R14)) {
 362:     // Create a fixed slot for BLINK.
 363:     int StackObj  = MFI.CreateFixedSpillStackObject(4, CurOffset, true);
 364:     LLVM_DEBUG(dbgs() << "Creating fixed object (" << StackObj
 365:                       << ") for BLINK at " << CurOffset << "\n");
 366:     (void)StackObj;
 367:     CurOffset -= 4;
 368:   }
 369: 
 370:   // Create slots for last down to r13.
 371:   for (unsigned Which = Last; Which > ARC::R12; Which--) {
 372:     auto RegI = getSavedReg(CSI, Which);
 373:     if (RegI == CSI.end() || RegI->getFrameIdx() == 0) {
 374:       // Always create the stack slot.  If for some reason the register isn't in
 375:       // the save list, then don't worry about it.
 376:       int FI = MFI.CreateFixedSpillStackObject(4, CurOffset, true);
 377:       if (RegI != CSI.end())
 378:         RegI->setFrameIdx(FI);
 379:     } else
 380:       MFI.setObjectOffset(RegI->getFrameIdx(), CurOffset);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 381-400

```cpp
 381:     CurOffset -= 4;
 382:   }
 383:   for (auto &I : CSI) {
 384:     if (I.getReg() > ARC::R12)
 385:       continue;
 386:     if (I.getFrameIdx() == 0) {
 387:       I.setFrameIdx(MFI.CreateFixedSpillStackObject(4, CurOffset, true));
 388:       LLVM_DEBUG(dbgs() << "Creating fixed object (" << I.getFrameIdx()
 389:                         << ") for other register at " << CurOffset << "\n");
 390:     } else {
 391:       MFI.setObjectOffset(I.getFrameIdx(), CurOffset);
 392:       LLVM_DEBUG(dbgs() << "Updating fixed object (" << I.getFrameIdx()
 393:                         << ") for other register at " << CurOffset << "\n");
 394:     }
 395:     CurOffset -= 4;
 396:   }
 397:   return true;
 398: }
 399: 
 400: bool ARCFrameLowering::spillCalleeSavedRegisters(
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 401-420

```cpp
 401:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 402:     ArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 403:   LLVM_DEBUG(dbgs() << "Spill callee saved registers: "
 404:                     << MBB.getParent()->getName() << "\n");
 405:   // There are routines for saving at least 3 registers (r13 to r15, etc.)
 406:   unsigned Last = determineLastCalleeSave(CSI);
 407:   if (UseSaveRestoreFunclet && Last > ARC::R14) {
 408:     // Use setObjectOffset for these registers.
 409:     // Needs to be in or before processFunctionBeforeFrameFinalized.
 410:     // Or, do assignCalleeSaveSpillSlots?
 411:     // Will be handled in prolog.
 412:     return true;
 413:   }
 414:   return false;
 415: }
 416: 
 417: bool ARCFrameLowering::restoreCalleeSavedRegisters(
 418:     MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
 419:     MutableArrayRef<CalleeSavedInfo> CSI, const TargetRegisterInfo *TRI) const {
 420:   LLVM_DEBUG(dbgs() << "Restore callee saved registers: "
```

- EN: Function bodies or method definitions such as restoreCalleeSavedRegisters contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: restoreCalleeSavedRegisters 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 421-440

```cpp
 421:                     << MBB.getParent()->getName() << "\n");
 422:   // There are routines for saving at least 3 registers (r13 to r15, etc.)
 423:   unsigned Last = determineLastCalleeSave(CSI);
 424:   if (UseSaveRestoreFunclet && Last > ARC::R14) {
 425:     // Will be handled in epilog.
 426:     return true;
 427:   }
 428:   return false;
 429: }
 430: 
 431: // Adjust local variables that are 4-bytes or larger to 4-byte boundary
 432: void ARCFrameLowering::processFunctionBeforeFrameFinalized(
 433:     MachineFunction &MF, RegScavenger *RS) const {
 434:   const TargetRegisterInfo *RegInfo = MF.getSubtarget().getRegisterInfo();
 435:   LLVM_DEBUG(dbgs() << "Process function before frame finalized: "
 436:                     << MF.getName() << "\n");
 437:   MachineFrameInfo &MFI = MF.getFrameInfo();
 438:   LLVM_DEBUG(dbgs() << "Current stack size: " << MFI.getStackSize() << "\n");
 439:   const TargetRegisterClass *RC = &ARC::GPR32RegClass;
 440:   if (MFI.hasStackObjects()) {
```

- EN: Function bodies or method definitions such as processFunctionBeforeFrameFinalized contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: processFunctionBeforeFrameFinalized 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 441-460

```cpp
 441:     int RegScavFI = MFI.CreateSpillStackObject(RegInfo->getSpillSize(*RC),
 442:                                                RegInfo->getSpillAlign(*RC));
 443:     RS->addScavengingFrameIndex(RegScavFI);
 444:     LLVM_DEBUG(dbgs() << "Created scavenging index RegScavFI=" << RegScavFI
 445:                       << "\n");
 446:   }
 447: }
 448: 
 449: static void emitRegUpdate(MachineBasicBlock &MBB,
 450:                           MachineBasicBlock::iterator &MBBI, DebugLoc dl,
 451:                           unsigned Reg, int NumBytes, bool IsAdd,
 452:                           const ARCInstrInfo *TII) {
 453:   unsigned Opc;
 454:   if (isUInt<6>(NumBytes))
 455:     Opc = IsAdd ? ARC::ADD_rru6 : ARC::SUB_rru6;
 456:   else if (isInt<12>(NumBytes))
 457:     Opc = IsAdd ? ARC::ADD_rrs12 : ARC::SUB_rrs12;
 458:   else
 459:     Opc = IsAdd ? ARC::ADD_rrlimm : ARC::SUB_rrlimm;
 460: 
```

- EN: Function bodies or method definitions such as emitRegUpdate contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: emitRegUpdate 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 461-480

```cpp
 461:   BuildMI(MBB, MBBI, dl, TII->get(Opc), Reg)
 462:       .addReg(Reg, RegState::Kill)
 463:       .addImm(NumBytes);
 464: }
 465: 
 466: MachineBasicBlock::iterator ARCFrameLowering::eliminateCallFramePseudoInstr(
 467:     MachineFunction &MF, MachineBasicBlock &MBB,
 468:     MachineBasicBlock::iterator I) const {
 469:   LLVM_DEBUG(dbgs() << "EmitCallFramePseudo: " << MF.getName() << "\n");
 470:   const ARCInstrInfo *TII = MF.getSubtarget<ARCSubtarget>().getInstrInfo();
 471:   MachineInstr &Old = *I;
 472:   DebugLoc dl = Old.getDebugLoc();
 473:   unsigned Amt = Old.getOperand(0).getImm();
 474:   auto *AFI = MF.getInfo<ARCFunctionInfo>();
 475:   if (!hasFP(MF)) {
 476:     if (Amt > AFI->MaxCallStackReq && Old.getOpcode() == ARC::ADJCALLSTACKDOWN)
 477:       AFI->MaxCallStackReq = Amt;
 478:   } else {
 479:     if (Amt != 0) {
 480:       assert((Old.getOpcode() == ARC::ADJCALLSTACKDOWN ||
```

- EN: Function bodies or method definitions such as eliminateCallFramePseudoInstr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: eliminateCallFramePseudoInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

### Lines 481-497

```cpp
 481:               Old.getOpcode() == ARC::ADJCALLSTACKUP) &&
 482:              "Unknown Frame Pseudo.");
 483:       bool IsAdd = (Old.getOpcode() == ARC::ADJCALLSTACKUP);
 484:       emitRegUpdate(MBB, I, dl, ARC::SP, Amt, IsAdd, TII);
 485:     }
 486:   }
 487:   return MBB.erase(I);
 488: }
 489: 
 490: bool ARCFrameLowering::hasFPImpl(const MachineFunction &MF) const {
 491:   const TargetRegisterInfo *RegInfo = MF.getSubtarget().getRegisterInfo();
 492:   bool HasFP = MF.getTarget().Options.DisableFramePointerElim(MF) ||
 493:                MF.getFrameInfo().hasVarSizedObjects() ||
 494:                MF.getFrameInfo().isFrameAddressTaken() ||
 495:                RegInfo->hasStackRealignment(MF);
 496:   return HasFP;
 497: }
```

- EN: Function bodies or method definitions such as hasFPImpl contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, the code here contributes to stack-frame construction and prologue/epilogue decisions.
- 中文: hasFPImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这里的代码参与栈帧构建以及序言/尾声决策。

## Key Concepts / 关键概念

- Prologue and epilogue emission / 序言与尾声生成
- Stack object layout / 栈对象布局
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCFrameLowering.h`, `ARCMachineFunctionInfo.h`, `ARCSubtarget.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineModuleInfo.h`, `llvm/CodeGen/RegisterScavenging.h`, `llvm/CodeGen/TargetRegisterInfo.h`, `llvm/IR/Function.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `ARCFrameLowering.h`
