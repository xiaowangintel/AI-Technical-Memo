# CSKYRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYRegisterInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the CSKY implementation of the TargetRegisterInfo class.
- 目的（中文）: 定义目标寄存器属性、被调用者保存规则以及寄存器分配辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYRegisterInfo.h - CSKY Register Information Impl ---*- C++ -*---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the CSKY implementation of the TargetRegisterInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYRegisterInfo.h"
  14: #include "CSKY.h"
  15: #include "CSKYSubtarget.h"
  16: #include "llvm/CodeGen/MachineFrameInfo.h"
  17: #include "llvm/CodeGen/MachineFunction.h"
  18: #include "llvm/CodeGen/RegisterScavenging.h"
  19: #include "llvm/MC/MCContext.h"
  20: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 21-40

```cpp
  21: #define GET_REGINFO_TARGET_DESC
  22: #include "CSKYGenRegisterInfo.inc"
  23: 
  24: using namespace llvm;
  25: 
  26: CSKYRegisterInfo::CSKYRegisterInfo()
  27:     : CSKYGenRegisterInfo(CSKY::R15, 0, 0, 0) {}
  28: 
  29: const uint32_t *
  30: CSKYRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
  31:                                        CallingConv::ID Id) const {
  32:   const CSKYSubtarget &STI = MF.getSubtarget<CSKYSubtarget>();
  33:   if (STI.hasFPUv2DoubleFloat() || STI.hasFPUv3DoubleFloat())
  34:     return CSR_GPR_FPR64_RegMask;
  35:   if (STI.hasFPUv2SingleFloat() || STI.hasFPUv3SingleFloat())
  36:     return CSR_GPR_FPR32_RegMask;
  37:   return CSR_I32_RegMask;
  38: }
  39: 
  40: Register CSKYRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as CSKYGenRegisterInfo, getCallPreservedMask, getFrameRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 CSKYGenRegisterInfo, getCallPreservedMask, getFrameRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 41-60

```cpp
  41:   const TargetFrameLowering *TFI = getFrameLowering(MF);
  42:   return TFI->hasFP(MF) ? CSKY::R8 : CSKY::R14;
  43: }
  44: 
  45: BitVector CSKYRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  46:   const CSKYFrameLowering *TFI = getFrameLowering(MF);
  47:   const CSKYSubtarget &STI = MF.getSubtarget<CSKYSubtarget>();
  48:   BitVector Reserved(getNumRegs());
  49: 
  50:   // Reserve the base register if we need to allocate
  51:   // variable-sized objects at runtime.
  52:   if (TFI->hasBP(MF))
  53:     markSuperRegs(Reserved, CSKY::R7); // bp
  54: 
  55:   if (TFI->hasFP(MF))
  56:     markSuperRegs(Reserved, CSKY::R8); // fp
  57: 
  58:   if (!STI.hasE2()) {
  59:     for (unsigned i = 0; i < 6; i++)
  60:       markSuperRegs(Reserved, CSKY::R8 + i); // R8 - R13
```

- EN: Function bodies or method definitions such as getReservedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getReservedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 61-80

```cpp
  61:   }
  62: 
  63:   markSuperRegs(Reserved, CSKY::R14); // sp
  64:   markSuperRegs(Reserved, CSKY::R15); // lr
  65: 
  66:   if (!STI.hasHighRegisters()) {
  67:     for (unsigned i = 0; i < 10; i++)
  68:       markSuperRegs(Reserved, CSKY::R16 + i); // R16 - R25
  69:   }
  70: 
  71:   markSuperRegs(Reserved, CSKY::R26);
  72:   markSuperRegs(Reserved, CSKY::R27);
  73:   markSuperRegs(Reserved, CSKY::R28); // gp
  74:   markSuperRegs(Reserved, CSKY::R29);
  75:   markSuperRegs(Reserved, CSKY::R30);
  76:   markSuperRegs(Reserved, CSKY::R31); // tp
  77: 
  78:   assert(checkAllSuperRegsMarked(Reserved));
  79:   return Reserved;
  80: }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 81-100

```cpp
  81: 
  82: const uint32_t *CSKYRegisterInfo::getNoPreservedMask() const {
  83:   return CSR_NoRegs_RegMask;
  84: }
  85: 
  86: const MCPhysReg *
  87: CSKYRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  88:   const CSKYSubtarget &STI = MF->getSubtarget<CSKYSubtarget>();
  89:   if (MF->getFunction().hasFnAttribute("interrupt")) {
  90:     if (STI.hasFPUv3DoubleFloat())
  91:       return CSR_GPR_FPR64v3_ISR_SaveList;
  92:     if (STI.hasFPUv3SingleFloat())
  93:       return CSR_GPR_FPR32v3_ISR_SaveList;
  94:     if (STI.hasFPUv2DoubleFloat())
  95:       return CSR_GPR_FPR64_ISR_SaveList;
  96:     if (STI.hasFPUv2SingleFloat())
  97:       return CSR_GPR_FPR32_ISR_SaveList;
  98:     return CSR_GPR_ISR_SaveList;
  99:   }
 100: 
```

- EN: Function bodies or method definitions such as getCalleeSavedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getCalleeSavedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 101-120

```cpp
 101:   if (STI.hasFPUv2DoubleFloat() || STI.hasFPUv3DoubleFloat())
 102:     return CSR_GPR_FPR64_SaveList;
 103:   if (STI.hasFPUv2SingleFloat() || STI.hasFPUv3SingleFloat())
 104:     return CSR_GPR_FPR32_SaveList;
 105:   return CSR_I32_SaveList;
 106: }
 107: 
 108: static bool IsLegalOffset(const CSKYInstrInfo *TII, MachineInstr *MI,
 109:                           int &Offset) {
 110:   const MCInstrDesc &Desc = MI->getDesc();
 111:   unsigned AddrMode = (Desc.TSFlags & CSKYII::AddrModeMask);
 112:   unsigned i = 0;
 113:   for (; !MI->getOperand(i).isFI(); ++i) {
 114:     assert(i + 1 < MI->getNumOperands() &&
 115:            "Instr doesn't have FrameIndex operand!");
 116:   }
 117: 
 118:   if (MI->getOpcode() == CSKY::ADDI32) {
 119:     if (!isUInt<12>(std::abs(Offset) - 1))
 120:       return false;
```

- EN: Function bodies or method definitions such as IsLegalOffset contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: IsLegalOffset 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 121-140

```cpp
 121:     if (Offset < 0) {
 122:       MI->setDesc(TII->get(CSKY::SUBI32));
 123:       Offset = -Offset;
 124:     }
 125: 
 126:     return true;
 127:   }
 128: 
 129:   if (MI->getOpcode() == CSKY::ADDI16XZ)
 130:     return false;
 131: 
 132:   if (Offset < 0)
 133:     return false;
 134: 
 135:   unsigned NumBits = 0;
 136:   unsigned Scale = 1;
 137:   switch (AddrMode) {
 138:   case CSKYII::AddrMode32B:
 139:     Scale = 1;
 140:     NumBits = 12;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 141-160

```cpp
 141:     break;
 142:   case CSKYII::AddrMode32H:
 143:     Scale = 2;
 144:     NumBits = 12;
 145:     break;
 146:   case CSKYII::AddrMode32WD:
 147:     Scale = 4;
 148:     NumBits = 12;
 149:     break;
 150:   case CSKYII::AddrMode16B:
 151:     Scale = 1;
 152:     NumBits = 5;
 153:     break;
 154:   case CSKYII::AddrMode16H:
 155:     Scale = 2;
 156:     NumBits = 5;
 157:     break;
 158:   case CSKYII::AddrMode16W:
 159:     Scale = 4;
 160:     NumBits = 5;
```

- EN: At the backend level, this range manages register properties and allocation-related rules.
- 中文: 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 161-180

```cpp
 161:     break;
 162:   case CSKYII::AddrMode32SDF:
 163:     Scale = 4;
 164:     NumBits = 8;
 165:     break;
 166:   default:
 167:     llvm_unreachable("Unsupported addressing mode!");
 168:   }
 169: 
 170:   // Cannot encode offset.
 171:   if ((Offset & (Scale - 1)) != 0)
 172:     return false;
 173: 
 174:   unsigned Mask = (1 << NumBits) - 1;
 175:   if ((unsigned)Offset <= Mask * Scale)
 176:     return true;
 177: 
 178:   // Offset out of range.
 179:   return false;
 180: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 181-200

```cpp
 181: 
 182: bool CSKYRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
 183:                                            int SPAdj, unsigned FIOperandNum,
 184:                                            RegScavenger *RS) const {
 185:   assert(SPAdj == 0 && "Unexpected non-zero SPAdj value");
 186: 
 187:   MachineInstr *MI = &*II;
 188:   MachineBasicBlock &MBB = *MI->getParent();
 189:   MachineFunction &MF = *MI->getParent()->getParent();
 190:   MachineRegisterInfo &MRI = MF.getRegInfo();
 191:   const CSKYInstrInfo *TII = MF.getSubtarget<CSKYSubtarget>().getInstrInfo();
 192:   DebugLoc DL = MI->getDebugLoc();
 193:   const CSKYSubtarget &STI = MF.getSubtarget<CSKYSubtarget>();
 194: 
 195:   switch (MI->getOpcode()) {
 196:   default:
 197:     break;
 198:   case CSKY::RESTORE_CARRY: {
 199:     Register NewReg = STI.hasE2()
 200:                           ? MRI.createVirtualRegister(&CSKY::GPRRegClass)
```

- EN: Function bodies or method definitions such as eliminateFrameIndex contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range manages register properties and allocation-related rules.
- 中文: eliminateFrameIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 201-220

```cpp
 201:                           : MRI.createVirtualRegister(&CSKY::mGPRRegClass);
 202: 
 203:     auto *Temp = BuildMI(MBB, II, DL, TII->get(CSKY::LD32W), NewReg)
 204:                      .add(MI->getOperand(1))
 205:                      .add(MI->getOperand(2))
 206:                      .getInstr();
 207: 
 208:     BuildMI(MBB, II, DL, TII->get(STI.hasE2() ? CSKY::BTSTI32 : CSKY::BTSTI16),
 209:             MI->getOperand(0).getReg())
 210:         .addReg(NewReg, getKillRegState(true))
 211:         .addImm(0);
 212: 
 213:     MI = Temp;
 214: 
 215:     MBB.erase(II);
 216:     break;
 217:   }
 218:   case CSKY::SPILL_CARRY: {
 219:     Register NewReg;
 220:     if (STI.hasE2()) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 221-240

```cpp
 221:       NewReg = MRI.createVirtualRegister(&CSKY::GPRRegClass);
 222:       BuildMI(MBB, II, DL, TII->get(CSKY::MVC32), NewReg)
 223:           .add(MI->getOperand(0));
 224:     } else {
 225:       NewReg = MRI.createVirtualRegister(&CSKY::mGPRRegClass);
 226:       BuildMI(MBB, II, DL, TII->get(CSKY::MOVI16), NewReg).addImm(0);
 227:       BuildMI(MBB, II, DL, TII->get(CSKY::ADDC16))
 228:           .addReg(NewReg, RegState::Define)
 229:           .addReg(MI->getOperand(0).getReg(), RegState::Define)
 230:           .addReg(NewReg, getKillRegState(true))
 231:           .addReg(NewReg, getKillRegState(true))
 232:           .addReg(MI->getOperand(0).getReg());
 233: 
 234:       BuildMI(MBB, II, DL, TII->get(CSKY::BTSTI16), MI->getOperand(0).getReg())
 235:           .addReg(NewReg)
 236:           .addImm(0);
 237:     }
 238: 
 239:     MI = BuildMI(MBB, II, DL, TII->get(CSKY::ST32W))
 240:              .addReg(NewReg, getKillRegState(true))
```

- EN: At the backend level, this range manages register properties and allocation-related rules.
- 中文: 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 241-260

```cpp
 241:              .add(MI->getOperand(1))
 242:              .add(MI->getOperand(2))
 243:              .getInstr();
 244: 
 245:     MBB.erase(II);
 246: 
 247:     break;
 248:   }
 249:   }
 250: 
 251:   int FrameIndex = MI->getOperand(FIOperandNum).getIndex();
 252:   Register FrameReg;
 253:   int Offset = getFrameLowering(MF)
 254:                    ->getFrameIndexReference(MF, FrameIndex, FrameReg)
 255:                    .getFixed() +
 256:                MI->getOperand(FIOperandNum + 1).getImm();
 257: 
 258:   if (!isInt<32>(Offset))
 259:     report_fatal_error(
 260:         "Frame offsets outside of the signed 32-bit range not supported");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 261-280

```cpp
 261: 
 262:   bool FrameRegIsKill = false;
 263:   MachineBasicBlock::iterator NewII(MI);
 264:   if (!IsLegalOffset(TII, MI, Offset)) {
 265:     assert(isInt<32>(Offset) && "Int32 expected");
 266:     // The offset won't fit in an immediate, so use a scratch register instead
 267:     // Modify Offset and FrameReg appropriately
 268:     Register ScratchReg = TII->movImm(MBB, NewII, DL, Offset);
 269:     BuildMI(MBB, NewII, DL,
 270:             TII->get(STI.hasE2() ? CSKY::ADDU32 : CSKY::ADDU16XZ), ScratchReg)
 271:         .addReg(ScratchReg, RegState::Kill)
 272:         .addReg(FrameReg);
 273: 
 274:     Offset = 0;
 275:     FrameReg = ScratchReg;
 276:     FrameRegIsKill = true;
 277:   }
 278: 
 279:   if (Offset == 0 &&
 280:       (MI->getOpcode() == CSKY::ADDI32 || MI->getOpcode() == CSKY::ADDI16XZ)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 281-291

```cpp
 281:     MI->setDesc(TII->get(TargetOpcode::COPY));
 282:     MI->getOperand(FIOperandNum)
 283:         .ChangeToRegister(FrameReg, false, false, FrameRegIsKill);
 284:     MI->removeOperand(FIOperandNum + 1);
 285:   } else {
 286:     MI->getOperand(FIOperandNum)
 287:         .ChangeToRegister(FrameReg, false, false, FrameRegIsKill);
 288:     MI->getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
 289:   }
 290:   return false;
 291: }
```

- EN: At the backend level, this range manages register properties and allocation-related rules.
- 中文: 在后端层面，这一段管理寄存器属性及与分配相关的规则。

## Key Concepts / 关键概念

- Callee-saved handling / 被调用者保存处理
- Register allocation support / 寄存器分配支持
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- MC instruction representation / MC 指令表示
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYRegisterInfo.h`, `CSKY.h`, `CSKYSubtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/RegisterScavenging.h`, `llvm/MC/MCContext.h`, `CSKYGenRegisterInfo.inc`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, LLVM MC
- Generated or companion files / 生成或配套文件: `CSKYGenRegisterInfo.inc`
- Local companions / 本地配套文件: `CSKYRegisterInfo.h`, `CSKYRegisterInfo.td`
