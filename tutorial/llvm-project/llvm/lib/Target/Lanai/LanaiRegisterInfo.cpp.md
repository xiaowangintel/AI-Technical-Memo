# LanaiRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiRegisterInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the Lanai implementation of the TargetRegisterInfo class.
- 目的（中文）: 定义目标寄存器属性、被调用者保存规则以及寄存器分配辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiRegisterInfo.cpp - Lanai Register Information ------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the Lanai implementation of the TargetRegisterInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LanaiRegisterInfo.h"
  14: #include "LanaiAluCode.h"
  15: #include "LanaiCondCode.h"
  16: #include "LanaiFrameLowering.h"
  17: #include "LanaiInstrInfo.h"
  18: #include "llvm/ADT/BitVector.h"
  19: #include "llvm/CodeGen/MachineFrameInfo.h"
  20: #include "llvm/CodeGen/MachineFunction.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineInstrBuilder.h"
  22: #include "llvm/CodeGen/RegisterScavenging.h"
  23: #include "llvm/CodeGen/TargetFrameLowering.h"
  24: #include "llvm/CodeGen/TargetInstrInfo.h"
  25: #include "llvm/Support/ErrorHandling.h"
  26: 
  27: #define GET_REGINFO_TARGET_DESC
  28: #include "LanaiGenRegisterInfo.inc"
  29: 
  30: using namespace llvm;
  31: 
  32: LanaiRegisterInfo::LanaiRegisterInfo() : LanaiGenRegisterInfo(Lanai::RCA) {}
  33: 
  34: const uint16_t *
  35: LanaiRegisterInfo::getCalleeSavedRegs(const MachineFunction * /*MF*/) const {
  36:   return CSR_SaveList;
  37: }
  38: 
  39: BitVector LanaiRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  40:   BitVector Reserved(getNumRegs());
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as getCalleeSavedRegs, getReservedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 getCalleeSavedRegs, getReservedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 41-60

```cpp
  41: 
  42:   Reserved.set(Lanai::R0);
  43:   Reserved.set(Lanai::R1);
  44:   Reserved.set(Lanai::PC);
  45:   Reserved.set(Lanai::R2);
  46:   Reserved.set(Lanai::SP);
  47:   Reserved.set(Lanai::R4);
  48:   Reserved.set(Lanai::FP);
  49:   Reserved.set(Lanai::R5);
  50:   Reserved.set(Lanai::RR1);
  51:   Reserved.set(Lanai::R10);
  52:   Reserved.set(Lanai::RR2);
  53:   Reserved.set(Lanai::R11);
  54:   Reserved.set(Lanai::RCA);
  55:   Reserved.set(Lanai::R15);
  56:   if (hasBasePointer(MF))
  57:     Reserved.set(getBaseRegister());
  58:   return Reserved;
  59: }
  60: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 61-80

```cpp
  61: bool LanaiRegisterInfo::requiresRegisterScavenging(
  62:     const MachineFunction & /*MF*/) const {
  63:   return true;
  64: }
  65: 
  66: static bool isALUArithLoOpcode(unsigned Opcode) {
  67:   switch (Opcode) {
  68:   case Lanai::ADD_I_LO:
  69:   case Lanai::SUB_I_LO:
  70:   case Lanai::ADD_F_I_LO:
  71:   case Lanai::SUB_F_I_LO:
  72:   case Lanai::ADDC_I_LO:
  73:   case Lanai::SUBB_I_LO:
  74:   case Lanai::ADDC_F_I_LO:
  75:   case Lanai::SUBB_F_I_LO:
  76:     return true;
  77:   default:
  78:     return false;
  79:   }
  80: }
```

- EN: Function bodies or method definitions such as requiresRegisterScavenging, isALUArithLoOpcode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range manages register properties and allocation-related rules.
- 中文: requiresRegisterScavenging, isALUArithLoOpcode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 81-100

```cpp
  81: 
  82: static unsigned getOppositeALULoOpcode(unsigned Opcode) {
  83:   switch (Opcode) {
  84:   case Lanai::ADD_I_LO:
  85:     return Lanai::SUB_I_LO;
  86:   case Lanai::SUB_I_LO:
  87:     return Lanai::ADD_I_LO;
  88:   case Lanai::ADD_F_I_LO:
  89:     return Lanai::SUB_F_I_LO;
  90:   case Lanai::SUB_F_I_LO:
  91:     return Lanai::ADD_F_I_LO;
  92:   case Lanai::ADDC_I_LO:
  93:     return Lanai::SUBB_I_LO;
  94:   case Lanai::SUBB_I_LO:
  95:     return Lanai::ADDC_I_LO;
  96:   case Lanai::ADDC_F_I_LO:
  97:     return Lanai::SUBB_F_I_LO;
  98:   case Lanai::SUBB_F_I_LO:
  99:     return Lanai::ADDC_F_I_LO;
 100:   default:
```

- EN: Function bodies or method definitions such as getOppositeALULoOpcode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getOppositeALULoOpcode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 101-120

```cpp
 101:     llvm_unreachable("Invalid ALU lo opcode");
 102:   }
 103: }
 104: 
 105: static unsigned getRRMOpcodeVariant(unsigned Opcode) {
 106:   switch (Opcode) {
 107:   case Lanai::LDBs_RI:
 108:     return Lanai::LDBs_RR;
 109:   case Lanai::LDBz_RI:
 110:     return Lanai::LDBz_RR;
 111:   case Lanai::LDHs_RI:
 112:     return Lanai::LDHs_RR;
 113:   case Lanai::LDHz_RI:
 114:     return Lanai::LDHz_RR;
 115:   case Lanai::LDW_RI:
 116:     return Lanai::LDW_RR;
 117:   case Lanai::STB_RI:
 118:     return Lanai::STB_RR;
 119:   case Lanai::STH_RI:
 120:     return Lanai::STH_RR;
```

- EN: Function bodies or method definitions such as getRRMOpcodeVariant contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getRRMOpcodeVariant 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 121-140

```cpp
 121:   case Lanai::SW_RI:
 122:     return Lanai::SW_RR;
 123:   default:
 124:     llvm_unreachable("Opcode has no RRM variant");
 125:   }
 126: }
 127: 
 128: bool LanaiRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
 129:                                             int SPAdj, unsigned FIOperandNum,
 130:                                             RegScavenger *RS) const {
 131:   assert(SPAdj == 0 && "Unexpected");
 132: 
 133:   MachineInstr &MI = *II;
 134:   MachineFunction &MF = *MI.getParent()->getParent();
 135:   const TargetInstrInfo *TII = MF.getSubtarget().getInstrInfo();
 136:   const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
 137:   bool HasFP = TFI->hasFP(MF);
 138:   DebugLoc DL = MI.getDebugLoc();
 139: 
 140:   int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
```

- EN: Function bodies or method definitions such as eliminateFrameIndex contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: eliminateFrameIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 141-160

```cpp
 141: 
 142:   int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex) +
 143:                MI.getOperand(FIOperandNum + 1).getImm();
 144: 
 145:   // Addressable stack objects are addressed using neg. offsets from fp
 146:   // or pos. offsets from sp/basepointer
 147:   if (!HasFP || (hasStackRealignment(MF) && FrameIndex >= 0))
 148:     Offset += MF.getFrameInfo().getStackSize();
 149: 
 150:   Register FrameReg = getFrameRegister(MF);
 151:   if (FrameIndex >= 0) {
 152:     if (hasBasePointer(MF))
 153:       FrameReg = getBaseRegister();
 154:     else if (hasStackRealignment(MF))
 155:       FrameReg = Lanai::SP;
 156:   }
 157: 
 158:   // Replace frame index with a frame pointer reference.
 159:   // If the offset is small enough to fit in the immediate field, directly
 160:   // encode it.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 161-180

```cpp
 161:   // Otherwise scavenge a register and encode it into a MOVHI, OR_I_LO sequence.
 162:   if ((isSPLSOpcode(MI.getOpcode()) && !isInt<10>(Offset)) ||
 163:       !isInt<16>(Offset)) {
 164:     assert(RS && "Register scavenging must be on");
 165:     Register Reg = RS->FindUnusedReg(&Lanai::GPRRegClass);
 166:     if (!Reg)
 167:       Reg = RS->scavengeRegisterBackwards(Lanai::GPRRegClass, II, false, SPAdj);
 168:     assert(Reg && "Register scavenger failed");
 169: 
 170:     bool HasNegOffset = false;
 171:     // ALU ops have unsigned immediate values. If the Offset is negative, we
 172:     // negate it here and reverse the opcode later.
 173:     if (Offset < 0) {
 174:       HasNegOffset = true;
 175:       Offset = -Offset;
 176:     }
 177: 
 178:     if (!isInt<16>(Offset)) {
 179:       // Reg = hi(offset) | lo(offset)
 180:       BuildMI(*MI.getParent(), II, DL, TII->get(Lanai::MOVHI), Reg)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 181-200

```cpp
 181:           .addImm(static_cast<uint32_t>(Offset) >> 16);
 182:       BuildMI(*MI.getParent(), II, DL, TII->get(Lanai::OR_I_LO), Reg)
 183:           .addReg(Reg)
 184:           .addImm(Offset & 0xffffU);
 185:     } else {
 186:       // Reg = mov(offset)
 187:       BuildMI(*MI.getParent(), II, DL, TII->get(Lanai::ADD_I_LO), Reg)
 188:           .addImm(0)
 189:           .addImm(Offset);
 190:     }
 191:     // Reg = FrameReg OP Reg
 192:     if (MI.getOpcode() == Lanai::ADD_I_LO) {
 193:       BuildMI(*MI.getParent(), II, DL,
 194:               HasNegOffset ? TII->get(Lanai::SUB_R) : TII->get(Lanai::ADD_R),
 195:               MI.getOperand(0).getReg())
 196:           .addReg(FrameReg)
 197:           .addReg(Reg)
 198:           .addImm(LPCC::ICC_T);
 199:       MI.eraseFromParent();
 200:       return true;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 201-220

```cpp
 201:     }
 202:     if (isSPLSOpcode(MI.getOpcode()) || isRMOpcode(MI.getOpcode())) {
 203:       MI.setDesc(TII->get(getRRMOpcodeVariant(MI.getOpcode())));
 204:       if (HasNegOffset) {
 205:         // Change the ALU op (operand 3) from LPAC::ADD (the default) to
 206:         // LPAC::SUB with the already negated offset.
 207:         assert((MI.getOperand(3).getImm() == LPAC::ADD) &&
 208:                "Unexpected ALU op in RRM instruction");
 209:         MI.getOperand(3).setImm(LPAC::SUB);
 210:       }
 211:     } else
 212:       llvm_unreachable("Unexpected opcode in frame index operation");
 213: 
 214:     MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, /*isDef=*/false);
 215:     MI.getOperand(FIOperandNum + 1)
 216:         .ChangeToRegister(Reg, /*isDef=*/false, /*isImp=*/false,
 217:                           /*isKill=*/true);
 218:     return false;
 219:   }
 220: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 221-240

```cpp
 221:   // ALU arithmetic ops take unsigned immediates. If the offset is negative,
 222:   // we replace the instruction with one that inverts the opcode and negates
 223:   // the immediate.
 224:   if ((Offset < 0) && isALUArithLoOpcode(MI.getOpcode())) {
 225:     unsigned NewOpcode = getOppositeALULoOpcode(MI.getOpcode());
 226:     // We know this is an ALU op, so we know the operands are as follows:
 227:     // 0: destination register
 228:     // 1: source register (frame register)
 229:     // 2: immediate
 230:     BuildMI(*MI.getParent(), II, DL, TII->get(NewOpcode),
 231:             MI.getOperand(0).getReg())
 232:         .addReg(FrameReg)
 233:         .addImm(-Offset);
 234:     MI.eraseFromParent();
 235:     return true;
 236:   }
 237: 
 238:   MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, /*isDef=*/false);
 239:   MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
 240:   return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 241-260

```cpp
 241: }
 242: 
 243: bool LanaiRegisterInfo::hasBasePointer(const MachineFunction &MF) const {
 244:   const MachineFrameInfo &MFI = MF.getFrameInfo();
 245:   // When we need stack realignment and there are dynamic allocas, we can't
 246:   // reference off of the stack pointer, so we reserve a base pointer.
 247:   if (hasStackRealignment(MF) && MFI.hasVarSizedObjects())
 248:     return true;
 249: 
 250:   return false;
 251: }
 252: 
 253: unsigned LanaiRegisterInfo::getRARegister() const { return Lanai::RCA; }
 254: 
 255: Register
 256: LanaiRegisterInfo::getFrameRegister(const MachineFunction & /*MF*/) const {
 257:   return Lanai::FP;
 258: }
 259: 
 260: Register LanaiRegisterInfo::getBaseRegister() const { return Lanai::R14; }
```

- EN: Function bodies or method definitions such as hasBasePointer, getRARegister, getFrameRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: hasBasePointer, getRARegister, getFrameRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 261-266

```cpp
 261: 
 262: const uint32_t *
 263: LanaiRegisterInfo::getCallPreservedMask(const MachineFunction & /*MF*/,
 264:                                         CallingConv::ID /*CC*/) const {
 265:   return CSR_RegMask;
 266: }
```

- EN: Function bodies or method definitions such as getCallPreservedMask contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getCallPreservedMask 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

## Key Concepts / 关键概念

- Callee-saved handling / 被调用者保存处理
- Register allocation support / 寄存器分配支持
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Instruction semantics helpers / 指令语义辅助逻辑
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiRegisterInfo.h`, `LanaiAluCode.h`, `LanaiCondCode.h`, `LanaiFrameLowering.h`, `LanaiInstrInfo.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `LanaiGenRegisterInfo.inc`
- Local companions / 本地配套文件: `LanaiRegisterInfo.h`, `LanaiRegisterInfo.td`
