# ARCRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCRegisterInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the ARC implementation of the MRegisterInfo class.
- 目的（中文）: 定义目标寄存器属性、被调用者保存规则以及寄存器分配辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCRegisterInfo.cpp - ARC Register Information -----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the ARC implementation of the MRegisterInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "ARCRegisterInfo.h"
  14: #include "ARC.h"
  15: #include "ARCInstrInfo.h"
  16: #include "ARCMachineFunctionInfo.h"
  17: #include "ARCSubtarget.h"
  18: #include "llvm/ADT/BitVector.h"
  19: #include "llvm/CodeGen/MachineFrameInfo.h"
  20: #include "llvm/CodeGen/MachineFunction.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineInstrBuilder.h"
  22: #include "llvm/CodeGen/MachineModuleInfo.h"
  23: #include "llvm/CodeGen/MachineRegisterInfo.h"
  24: #include "llvm/CodeGen/RegisterScavenging.h"
  25: #include "llvm/CodeGen/TargetFrameLowering.h"
  26: #include "llvm/IR/Function.h"
  27: #include "llvm/Support/Debug.h"
  28: #include "llvm/Target/TargetMachine.h"
  29: #include "llvm/Target/TargetOptions.h"
  30: 
  31: using namespace llvm;
  32: 
  33: #define DEBUG_TYPE "arc-reg-info"
  34: 
  35: #define GET_REGINFO_TARGET_DESC
  36: #include "ARCGenRegisterInfo.inc"
  37: 
  38: static void replaceFrameIndex(MachineBasicBlock::iterator II,
  39:                               const ARCInstrInfo &TII, unsigned Reg,
  40:                               unsigned FrameReg, int Offset, int StackSize,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 41-60

```cpp
  41:                               int ObjSize, RegScavenger *RS, int SPAdj) {
  42:   assert(RS && "Need register scavenger.");
  43:   MachineInstr &MI = *II;
  44:   MachineBasicBlock &MBB = *MI.getParent();
  45:   DebugLoc DL = MI.getDebugLoc();
  46:   unsigned BaseReg = FrameReg;
  47:   RegState KillState = {};
  48:   if (MI.getOpcode() == ARC::LD_rs9 && (Offset >= 256 || Offset < -256)) {
  49:     // Loads can always be reached with LD_rlimm.
  50:     BuildMI(MBB, II, DL, TII.get(ARC::LD_rlimm), Reg)
  51:         .addReg(BaseReg)
  52:         .addImm(Offset)
  53:         .addMemOperand(*MI.memoperands_begin());
  54:     MBB.erase(II);
  55:     return;
  56:   }
  57: 
  58:   if (MI.getOpcode() != ARC::GETFI && (Offset >= 256 || Offset < -256)) {
  59:     // We need to use a scratch register to reach the far-away frame indexes.
  60:     BaseReg = RS->FindUnusedReg(&ARC::GPR32RegClass);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 61-80

```cpp
  61:     if (!BaseReg) {
  62:       // We can be sure that the scavenged-register slot is within the range
  63:       // of the load offset.
  64:       const TargetRegisterInfo *TRI =
  65:           MBB.getParent()->getSubtarget().getRegisterInfo();
  66:       BaseReg =
  67:           RS->scavengeRegisterBackwards(ARC::GPR32RegClass, II, false, SPAdj);
  68:       assert(BaseReg && "Register scavenging failed.");
  69:       LLVM_DEBUG(dbgs() << "Scavenged register " << printReg(BaseReg, TRI)
  70:                         << " for FrameReg=" << printReg(FrameReg, TRI)
  71:                         << "+Offset=" << Offset << "\n");
  72:       (void)TRI;
  73:       RS->setRegUsed(BaseReg);
  74:     }
  75:     unsigned AddOpc = isUInt<6>(Offset) ? ARC::ADD_rru6 : ARC::ADD_rrlimm;
  76:     BuildMI(MBB, II, DL, TII.get(AddOpc))
  77:         .addReg(BaseReg, RegState::Define)
  78:         .addReg(FrameReg)
  79:         .addImm(Offset);
  80:     Offset = 0;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 81-100

```cpp
  81:     KillState = RegState::Kill;
  82:   }
  83:   switch (MI.getOpcode()) {
  84:   case ARC::LD_rs9:
  85:     assert((Offset % 4 == 0) && "LD needs 4 byte alignment.");
  86:     [[fallthrough]];
  87:   case ARC::LDH_rs9:
  88:   case ARC::LDH_X_rs9:
  89:     assert((Offset % 2 == 0) && "LDH needs 2 byte alignment.");
  90:     [[fallthrough]];
  91:   case ARC::LDB_rs9:
  92:   case ARC::LDB_X_rs9:
  93:     LLVM_DEBUG(dbgs() << "Building LDFI\n");
  94:     BuildMI(MBB, II, DL, TII.get(MI.getOpcode()), Reg)
  95:         .addReg(BaseReg, KillState)
  96:         .addImm(Offset)
  97:         .addMemOperand(*MI.memoperands_begin());
  98:     break;
  99:   case ARC::ST_rs9:
 100:     assert((Offset % 4 == 0) && "ST needs 4 byte alignment.");
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 101-120

```cpp
 101:     [[fallthrough]];
 102:   case ARC::STH_rs9:
 103:     assert((Offset % 2 == 0) && "STH needs 2 byte alignment.");
 104:     [[fallthrough]];
 105:   case ARC::STB_rs9:
 106:     LLVM_DEBUG(dbgs() << "Building STFI\n");
 107:     BuildMI(MBB, II, DL, TII.get(MI.getOpcode()))
 108:         .addReg(Reg, getKillRegState(MI.getOperand(0).isKill()))
 109:         .addReg(BaseReg, KillState)
 110:         .addImm(Offset)
 111:         .addMemOperand(*MI.memoperands_begin());
 112:     break;
 113:   case ARC::GETFI:
 114:     LLVM_DEBUG(dbgs() << "Building GETFI\n");
 115:     BuildMI(MBB, II, DL,
 116:             TII.get(isUInt<6>(Offset) ? ARC::ADD_rru6 : ARC::ADD_rrlimm))
 117:         .addReg(Reg, RegState::Define)
 118:         .addReg(FrameReg)
 119:         .addImm(Offset);
 120:     break;
```

- EN: At the backend level, this range manages register properties and allocation-related rules.
- 中文: 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 121-140

```cpp
 121:   default:
 122:     llvm_unreachable("Unhandled opcode.");
 123:   }
 124: 
 125:   // Erase old instruction.
 126:   MBB.erase(II);
 127: }
 128: 
 129: ARCRegisterInfo::ARCRegisterInfo(const ARCSubtarget &ST)
 130:     : ARCGenRegisterInfo(ARC::BLINK), ST(ST) {}
 131: 
 132: bool ARCRegisterInfo::needsFrameMoves(const MachineFunction &MF) {
 133:   return MF.needsFrameMoves();
 134: }
 135: 
 136: const MCPhysReg *
 137: ARCRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
 138:   return CSR_ARC_SaveList;
 139: }
 140: 
```

- EN: Function bodies or method definitions such as ARCGenRegisterInfo, needsFrameMoves, getCalleeSavedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: ARCGenRegisterInfo, needsFrameMoves, getCalleeSavedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 141-160

```cpp
 141: BitVector ARCRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
 142:   BitVector Reserved(getNumRegs());
 143: 
 144:   Reserved.set(ARC::ILINK);
 145:   Reserved.set(ARC::SP);
 146:   Reserved.set(ARC::GP);
 147:   Reserved.set(ARC::R25);
 148:   Reserved.set(ARC::BLINK);
 149:   Reserved.set(ARC::FP);
 150: 
 151:   return Reserved;
 152: }
 153: 
 154: bool ARCRegisterInfo::requiresRegisterScavenging(
 155:     const MachineFunction &MF) const {
 156:   return true;
 157: }
 158: 
 159: bool ARCRegisterInfo::useFPForScavengingIndex(const MachineFunction &MF) const {
 160:   return true;
```

- EN: Function bodies or method definitions such as getReservedRegs, requiresRegisterScavenging, useFPForScavengingIndex contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getReservedRegs, requiresRegisterScavenging, useFPForScavengingIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 161-180

```cpp
 161: }
 162: 
 163: bool ARCRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
 164:                                           int SPAdj, unsigned FIOperandNum,
 165:                                           RegScavenger *RS) const {
 166:   assert(SPAdj == 0 && "Unexpected");
 167:   MachineInstr &MI = *II;
 168:   MachineOperand &FrameOp = MI.getOperand(FIOperandNum);
 169:   int FrameIndex = FrameOp.getIndex();
 170: 
 171:   MachineFunction &MF = *MI.getParent()->getParent();
 172:   const ARCInstrInfo &TII = *MF.getSubtarget<ARCSubtarget>().getInstrInfo();
 173:   const ARCFrameLowering *TFI = getFrameLowering(MF);
 174:   int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex);
 175:   int ObjSize = MF.getFrameInfo().getObjectSize(FrameIndex);
 176:   int StackSize = MF.getFrameInfo().getStackSize();
 177:   int LocalFrameSize = MF.getFrameInfo().getLocalFrameSize();
 178: 
 179:   LLVM_DEBUG(dbgs() << "\nFunction         : " << MF.getName() << "\n");
 180:   LLVM_DEBUG(dbgs() << "<--------->\n");
```

- EN: Function bodies or method definitions such as eliminateFrameIndex contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: eliminateFrameIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 181-200

```cpp
 181:   LLVM_DEBUG(dbgs() << MI << "\n");
 182:   LLVM_DEBUG(dbgs() << "FrameIndex         : " << FrameIndex << "\n");
 183:   LLVM_DEBUG(dbgs() << "ObjSize            : " << ObjSize << "\n");
 184:   LLVM_DEBUG(dbgs() << "FrameOffset        : " << Offset << "\n");
 185:   LLVM_DEBUG(dbgs() << "StackSize          : " << StackSize << "\n");
 186:   LLVM_DEBUG(dbgs() << "LocalFrameSize     : " << LocalFrameSize << "\n");
 187:   (void)LocalFrameSize;
 188: 
 189:   // Special handling of DBG_VALUE instructions.
 190:   if (MI.isDebugValue()) {
 191:     Register FrameReg = getFrameRegister(MF);
 192:     MI.getOperand(FIOperandNum).ChangeToRegister(FrameReg, false /*isDef*/);
 193:     MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset);
 194:     return false;
 195:   }
 196: 
 197:   // fold constant into offset.
 198:   Offset += MI.getOperand(FIOperandNum + 1).getImm();
 199: 
 200:   // TODO: assert based on the load type:
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 201-220

```cpp
 201:   // ldb needs no alignment,
 202:   // ldh needs 2 byte alignment
 203:   // ld needs 4 byte alignment
 204:   LLVM_DEBUG(dbgs() << "Offset             : " << Offset << "\n"
 205:                     << "<--------->\n");
 206: 
 207:   Register Reg = MI.getOperand(0).getReg();
 208:   assert(ARC::GPR32RegClass.contains(Reg) && "Unexpected register operand");
 209: 
 210:   if (!TFI->hasFP(MF)) {
 211:     Offset = StackSize + Offset;
 212:     if (FrameIndex >= 0)
 213:       assert((Offset >= 0 && Offset < StackSize) && "SP Offset not in bounds.");
 214:   } else {
 215:     if (FrameIndex >= 0) {
 216:       assert((Offset < 0 && -Offset <= StackSize) &&
 217:              "FP Offset not in bounds.");
 218:     }
 219:   }
 220:   replaceFrameIndex(II, TII, Reg, getFrameRegister(MF), Offset, StackSize,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 221-234

```cpp
 221:                     ObjSize, RS, SPAdj);
 222:   return true;                  
 223: }
 224: 
 225: Register ARCRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
 226:   const ARCFrameLowering *TFI = getFrameLowering(MF);
 227:   return TFI->hasFP(MF) ? ARC::FP : ARC::SP;
 228: }
 229: 
 230: const uint32_t *
 231: ARCRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
 232:                                       CallingConv::ID CC) const {
 233:   return CSR_ARC_RegMask;
 234: }
```

- EN: Function bodies or method definitions such as getFrameRegister, getCallPreservedMask contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getFrameRegister, getCallPreservedMask 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

## Key Concepts / 关键概念

- Callee-saved handling / 被调用者保存处理
- Register allocation support / 寄存器分配支持
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Stack frame lowering / 栈帧降级

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCRegisterInfo.h`, `ARC.h`, `ARCInstrInfo.h`, `ARCMachineFunctionInfo.h`, `ARCSubtarget.h`, `llvm/ADT/BitVector.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `ARCGenRegisterInfo.inc`
- Local companions / 本地配套文件: `ARCRegisterInfo.h`, `ARCRegisterInfo.td`
