# AVRRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRRegisterInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the AVR implementation of the TargetRegisterInfo class.
- 目的（中文）: 定义目标寄存器属性、被调用者保存规则以及寄存器分配辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRRegisterInfo.cpp - AVR Register Information --------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the AVR implementation of the TargetRegisterInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "AVRRegisterInfo.h"
  14: 
  15: #include "llvm/ADT/BitVector.h"
  16: #include "llvm/CodeGen/MachineFrameInfo.h"
  17: #include "llvm/CodeGen/MachineFunction.h"
  18: #include "llvm/CodeGen/MachineInstrBuilder.h"
  19: #include "llvm/CodeGen/TargetFrameLowering.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 21-40

```cpp
  21: #include "AVR.h"
  22: #include "AVRInstrInfo.h"
  23: #include "AVRMachineFunctionInfo.h"
  24: #include "AVRTargetMachine.h"
  25: #include "MCTargetDesc/AVRMCTargetDesc.h"
  26: 
  27: #define GET_REGINFO_TARGET_DESC
  28: #include "AVRGenRegisterInfo.inc"
  29: 
  30: namespace llvm {
  31: 
  32: AVRRegisterInfo::AVRRegisterInfo() : AVRGenRegisterInfo(0) {}
  33: 
  34: const uint16_t *
  35: AVRRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  36:   const AVRMachineFunctionInfo *AFI = MF->getInfo<AVRMachineFunctionInfo>();
  37:   const AVRSubtarget &STI = MF->getSubtarget<AVRSubtarget>();
  38:   if (STI.hasTinyEncoding())
  39:     return AFI->isInterruptOrSignalHandler() ? CSR_InterruptsTiny_SaveList
  40:                                              : CSR_NormalTiny_SaveList;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as getCalleeSavedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 getCalleeSavedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 41-60

```cpp
  41:   else
  42:     return AFI->isInterruptOrSignalHandler() ? CSR_Interrupts_SaveList
  43:                                              : CSR_Normal_SaveList;
  44: }
  45: 
  46: const uint32_t *
  47: AVRRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
  48:                                       CallingConv::ID CC) const {
  49:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
  50:   return STI.hasTinyEncoding() ? CSR_NormalTiny_RegMask : CSR_Normal_RegMask;
  51: }
  52: 
  53: BitVector AVRRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  54:   BitVector Reserved(getNumRegs());
  55: 
  56:   // Reserve the intermediate result registers r1 and r2
  57:   // The result of instructions like 'mul' is always stored here.
  58:   // R0/R1/R1R0 are always reserved on both avr and avrtiny.
  59:   Reserved.set(AVR::R0);
  60:   Reserved.set(AVR::R1);
```

- EN: Function bodies or method definitions such as getCallPreservedMask, getReservedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getCallPreservedMask, getReservedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 61-80

```cpp
  61:   Reserved.set(AVR::R1R0);
  62: 
  63:   // Reserve the stack pointer.
  64:   Reserved.set(AVR::SPL);
  65:   Reserved.set(AVR::SPH);
  66:   Reserved.set(AVR::SP);
  67: 
  68:   // Reserve R2~R17 only on avrtiny.
  69:   if (MF.getSubtarget<AVRSubtarget>().hasTinyEncoding()) {
  70:     // Reserve 8-bit registers R2~R15, Rtmp(R16) and Zero(R17).
  71:     for (unsigned Reg = AVR::R2; Reg <= AVR::R17; Reg++)
  72:       Reserved.set(Reg);
  73:     // Reserve 16-bit registers R3R2~R18R17.
  74:     for (unsigned Reg = AVR::R3R2; Reg <= AVR::R18R17; Reg++)
  75:       Reserved.set(Reg);
  76:   }
  77: 
  78:   // We tenatively reserve the frame pointer register r29:r28 because the
  79:   // function may require one, but we cannot tell until register allocation
  80:   // is complete, which can be too late.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 81-100

```cpp
  81:   //
  82:   // Instead we just unconditionally reserve the Y register.
  83:   //
  84:   // TODO: Write a pass to enumerate functions which reserved the Y register
  85:   //       but didn't end up needing a frame pointer. In these, we can
  86:   //       convert one or two of the spills inside to use the Y register.
  87:   Reserved.set(AVR::R28);
  88:   Reserved.set(AVR::R29);
  89:   Reserved.set(AVR::R29R28);
  90: 
  91:   return Reserved;
  92: }
  93: 
  94: const TargetRegisterClass *
  95: AVRRegisterInfo::getLargestLegalSuperClass(const TargetRegisterClass *RC,
  96:                                            const MachineFunction &MF) const {
  97:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
  98:   if (TRI->isTypeLegalForClass(*RC, MVT::i16)) {
  99:     return &AVR::DREGSRegClass;
 100:   }
```

- EN: Function bodies or method definitions such as getLargestLegalSuperClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getLargestLegalSuperClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 101-120

```cpp
 101: 
 102:   if (TRI->isTypeLegalForClass(*RC, MVT::i8)) {
 103:     return &AVR::GPR8RegClass;
 104:   }
 105: 
 106:   llvm_unreachable("Invalid register size");
 107: }
 108: 
 109: /// Fold a frame offset shared between two add instructions into a single one.
 110: static void foldFrameOffset(MachineBasicBlock::iterator &II, int &Offset,
 111:                             Register DstReg) {
 112:   MachineInstr &MI = *II;
 113:   int Opcode = MI.getOpcode();
 114: 
 115:   // Don't bother trying if the next instruction is not an add or a sub.
 116:   if ((Opcode != AVR::SUBIWRdK) && (Opcode != AVR::ADIWRdK)) {
 117:     return;
 118:   }
 119: 
 120:   // Check that DstReg matches with next instruction, otherwise the instruction
```

- EN: Function bodies or method definitions such as foldFrameOffset contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: foldFrameOffset 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 121-140

```cpp
 121:   // is not related to stack address manipulation.
 122:   if (DstReg != MI.getOperand(0).getReg()) {
 123:     return;
 124:   }
 125: 
 126:   // Add the offset in the next instruction to our offset.
 127:   switch (Opcode) {
 128:   case AVR::SUBIWRdK:
 129:     Offset += -MI.getOperand(2).getImm();
 130:     break;
 131:   case AVR::ADIWRdK:
 132:     Offset += MI.getOperand(2).getImm();
 133:     break;
 134:   }
 135: 
 136:   // Finally remove the instruction.
 137:   II++;
 138:   MI.eraseFromParent();
 139: }
 140: 
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 141-160

```cpp
 141: bool AVRRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
 142:                                           int SPAdj, unsigned FIOperandNum,
 143:                                           RegScavenger *RS) const {
 144:   assert(SPAdj == 0 && "Unexpected SPAdj value");
 145: 
 146:   MachineInstr &MI = *II;
 147:   DebugLoc dl = MI.getDebugLoc();
 148:   MachineBasicBlock &MBB = *MI.getParent();
 149:   const MachineFunction &MF = *MBB.getParent();
 150:   const AVRTargetMachine &TM = (const AVRTargetMachine &)MF.getTarget();
 151:   const TargetInstrInfo &TII = *TM.getSubtargetImpl()->getInstrInfo();
 152:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 153:   const TargetFrameLowering *TFI = TM.getSubtargetImpl()->getFrameLowering();
 154:   const AVRSubtarget &STI = MF.getSubtarget<AVRSubtarget>();
 155:   int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
 156:   int Offset = MFI.getObjectOffset(FrameIndex);
 157: 
 158:   // Add one to the offset because SP points to an empty slot.
 159:   Offset += MFI.getStackSize() - TFI->getOffsetOfLocalArea() + 1;
 160:   // Fold incoming offset.
```

- EN: Function bodies or method definitions such as eliminateFrameIndex contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: eliminateFrameIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 161-180

```cpp
 161:   Offset += MI.getOperand(FIOperandNum + 1).getImm();
 162: 
 163:   // This is actually "load effective address" of the stack slot
 164:   // instruction. We have only two-address instructions, thus we need to
 165:   // expand it into move + add.
 166:   if (MI.getOpcode() == AVR::FRMIDX) {
 167:     Register DstReg = MI.getOperand(0).getReg();
 168:     assert(DstReg != AVR::R29R28 && "Dest reg cannot be the frame pointer");
 169: 
 170:     // Copy the frame pointer.
 171:     if (STI.hasMOVW()) {
 172:       BuildMI(MBB, MI, dl, TII.get(AVR::MOVWRdRr), DstReg).addReg(AVR::R29R28);
 173:     } else {
 174:       Register DstLoReg, DstHiReg;
 175:       splitReg(DstReg, DstLoReg, DstHiReg);
 176:       BuildMI(MBB, MI, dl, TII.get(AVR::MOVRdRr), DstLoReg).addReg(AVR::R28);
 177:       BuildMI(MBB, MI, dl, TII.get(AVR::MOVRdRr), DstHiReg).addReg(AVR::R29);
 178:     }
 179: 
 180:     assert(Offset > 0 && "Invalid offset");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 181-200

```cpp
 181: 
 182:     // We need to materialize the offset via an add instruction.
 183:     unsigned Opcode;
 184: 
 185:     II++; // Skip over the FRMIDX instruction.
 186: 
 187:     // Generally, to load a frame address two add instructions are emitted that
 188:     // could get folded into a single one:
 189:     //  movw    r31:r30, r29:r28
 190:     //  adiw    r31:r30, 29
 191:     //  adiw    r31:r30, 16
 192:     // to:
 193:     //  movw    r31:r30, r29:r28
 194:     //  adiw    r31:r30, 45
 195:     if (II != MBB.end())
 196:       foldFrameOffset(II, Offset, DstReg);
 197: 
 198:     // Select the best opcode based on DstReg and the offset size.
 199:     switch (DstReg) {
 200:     case AVR::R25R24:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 201-220

```cpp
 201:     case AVR::R27R26:
 202:     case AVR::R31R30: {
 203:       if (isUInt<6>(Offset) && STI.hasADDSUBIW()) {
 204:         Opcode = AVR::ADIWRdK;
 205:         break;
 206:       }
 207:       [[fallthrough]];
 208:     }
 209:     default: {
 210:       // This opcode will get expanded into a pair of subi/sbci.
 211:       Opcode = AVR::SUBIWRdK;
 212:       Offset = -Offset;
 213:       break;
 214:     }
 215:     }
 216: 
 217:     MachineInstr *New = BuildMI(MBB, II, dl, TII.get(Opcode), DstReg)
 218:                             .addReg(DstReg, RegState::Kill)
 219:                             .addImm(Offset);
 220:     New->getOperand(3).setIsDead();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 221-240

```cpp
 221: 
 222:     MI.eraseFromParent(); // remove FRMIDX
 223: 
 224:     return false;
 225:   }
 226: 
 227:   // On most AVRs, we can use an offset up to 62 for load/store with
 228:   // displacement (63 for byte values, 62 for word values). However, the
 229:   // "reduced tiny" cores don't support load/store with displacement. So for
 230:   // them, we force an offset of 0 meaning that any positive offset will require
 231:   // adjusting the frame pointer.
 232:   int MaxOffset = STI.hasTinyEncoding() ? 0 : 62;
 233: 
 234:   // If the offset is too big we have to adjust and restore the frame pointer
 235:   // to materialize a valid load/store with displacement.
 236:   //: TODO: consider using only one adiw/sbiw chain for more than one frame
 237:   //: index
 238:   if (Offset > MaxOffset) {
 239:     unsigned AddOpc = AVR::ADIWRdK, SubOpc = AVR::SBIWRdK;
 240:     int AddOffset = Offset - MaxOffset;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 241-260

```cpp
 241: 
 242:     // For huge offsets where adiw/sbiw cannot be used use a pair of subi/sbci.
 243:     if ((Offset - MaxOffset) > 63 || !STI.hasADDSUBIW()) {
 244:       AddOpc = AVR::SUBIWRdK;
 245:       SubOpc = AVR::SUBIWRdK;
 246:       AddOffset = -AddOffset;
 247:     }
 248: 
 249:     // It is possible that the spiller places this frame instruction in between
 250:     // a compare and branch, invalidating the contents of SREG set by the
 251:     // compare instruction because of the add/sub pairs. Conservatively save and
 252:     // restore SREG before and after each add/sub pair.
 253:     BuildMI(MBB, II, dl, TII.get(AVR::INRdA), STI.getTmpRegister())
 254:         .addImm(STI.getIORegSREG());
 255: 
 256:     MachineInstr *New = BuildMI(MBB, II, dl, TII.get(AddOpc), AVR::R29R28)
 257:                             .addReg(AVR::R29R28, RegState::Kill)
 258:                             .addImm(AddOffset);
 259:     New->getOperand(3).setIsDead();
 260: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 261-280

```cpp
 261:     // Restore SREG.
 262:     BuildMI(MBB, std::next(II), dl, TII.get(AVR::OUTARr))
 263:         .addImm(STI.getIORegSREG())
 264:         .addReg(STI.getTmpRegister(), RegState::Kill);
 265: 
 266:     // No need to set SREG as dead here otherwise if the next instruction is a
 267:     // cond branch it will be using a dead register.
 268:     BuildMI(MBB, std::next(II), dl, TII.get(SubOpc), AVR::R29R28)
 269:         .addReg(AVR::R29R28, RegState::Kill)
 270:         .addImm(Offset - MaxOffset);
 271: 
 272:     Offset = MaxOffset;
 273:   }
 274: 
 275:   MI.getOperand(FIOperandNum).ChangeToRegister(AVR::R29R28, false);
 276:   assert(isUInt<6>(Offset) && "Offset is out of range");
 277:   MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
 278:   return false;
 279: }
 280: 
```

- EN: At the backend level, this range manages register properties and allocation-related rules.
- 中文: 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 281-300

```cpp
 281: Register AVRRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
 282:   const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
 283:   if (TFI->hasFP(MF)) {
 284:     // The Y pointer register
 285:     return AVR::R28;
 286:   }
 287: 
 288:   return AVR::SP;
 289: }
 290: 
 291: const TargetRegisterClass *
 292: AVRRegisterInfo::getPointerRegClass(unsigned Kind) const {
 293:   // FIXME: Currently we're using avr-gcc as reference, so we restrict
 294:   // ptrs to Y and Z regs. Though avr-gcc has buggy implementation
 295:   // of memory constraint, so we can fix it and bit avr-gcc here ;-)
 296:   return &AVR::PTRDISPREGSRegClass;
 297: }
 298: 
 299: void AVRRegisterInfo::splitReg(Register Reg, Register &LoReg,
 300:                                Register &HiReg) const {
```

- EN: Function bodies or method definitions such as getFrameRegister, getPointerRegClass, splitReg contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getFrameRegister, getPointerRegClass, splitReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 301-319

```cpp
 301:   assert(AVR::DREGSRegClass.contains(Reg) && "can only split 16-bit registers");
 302: 
 303:   LoReg = getSubReg(Reg, AVR::sub_lo);
 304:   HiReg = getSubReg(Reg, AVR::sub_hi);
 305: }
 306: 
 307: bool AVRRegisterInfo::shouldCoalesce(
 308:     MachineInstr *MI, const TargetRegisterClass *SrcRC, unsigned SubReg,
 309:     const TargetRegisterClass *DstRC, unsigned DstSubReg,
 310:     const TargetRegisterClass *NewRC, LiveIntervals &LIS) const {
 311:   if (this->getRegClass(AVR::PTRDISPREGSRegClassID)->hasSubClassEq(NewRC)) {
 312:     return false;
 313:   }
 314: 
 315:   return TargetRegisterInfo::shouldCoalesce(MI, SrcRC, SubReg, DstRC, DstSubReg,
 316:                                             NewRC, LIS);
 317: }
 318: 
 319: } // end of namespace llvm
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as shouldCoalesce contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 shouldCoalesce 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

## Key Concepts / 关键概念

- Callee-saved handling / 被调用者保存处理
- Register allocation support / 寄存器分配支持
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRRegisterInfo.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/TargetFrameLowering.h`, `AVR.h`, `AVRInstrInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `AVRGenRegisterInfo.inc`
- Local companions / 本地配套文件: `AVRRegisterInfo.h`, `AVRRegisterInfo.td`
