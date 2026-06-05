# LoongArchRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchRegisterInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target register information for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责定义或实现目标寄存器信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===- LoongArchRegisterInfo.cpp - LoongArch Register Information -*- C++ -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the LoongArch implementation of the TargetRegisterInfo
  10: // class.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArchRegisterInfo.h"
  15: #include "LoongArch.h"
  16: #include "LoongArchInstrInfo.h"
  17: #include "LoongArchSubtarget.h"
  18: #include "MCTargetDesc/LoongArchBaseInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchRegisterInfo.h`, `LoongArch.h`, `LoongArchInstrInfo.h`, `LoongArchSubtarget.h`, `LoongArchBaseInfo.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchRegisterInfo.h`, `LoongArch.h`, `LoongArchInstrInfo.h`, `LoongArchSubtarget.h`, `LoongArchBaseInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  20: #include "llvm/CodeGen/MachineFrameInfo.h"
  21: #include "llvm/CodeGen/MachineFunction.h"
  22: #include "llvm/CodeGen/MachineInstrBuilder.h"
  23: #include "llvm/CodeGen/RegisterScavenging.h"
  24: #include "llvm/CodeGen/TargetFrameLowering.h"
  25: #include "llvm/CodeGen/TargetInstrInfo.h"
  26: #include "llvm/Support/ErrorHandling.h"
  27: 
  28: using namespace llvm;
  29: 
  30: #define GET_REGINFO_TARGET_DESC
  31: #include "LoongArchGenRegisterInfo.inc"
  32: 
  33: LoongArchRegisterInfo::LoongArchRegisterInfo(unsigned HwMode)
  34:     : LoongArchGenRegisterInfo(LoongArch::R1, /*DwarfFlavour*/ 0,
  35:                                /*EHFlavor*/ 0,
  36:                                /*PC*/ 0, HwMode) {}
```
- **EN**: It imports dependencies such as `LoongArchMCTargetDesc.h`, `MachineFrameInfo.h`, `MachineFunction.h`, `MachineInstrBuilder.h`, `RegisterScavenging.h`, `TargetFrameLowering.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchGenRegisterInfo`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchMCTargetDesc.h`, `MachineFrameInfo.h`, `MachineFunction.h`, `MachineInstrBuilder.h`, `RegisterScavenging.h`, `TargetFrameLowering.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchGenRegisterInfo` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38: const MCPhysReg *
  39: LoongArchRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  40:   auto &Subtarget = MF->getSubtarget<LoongArchSubtarget>();
  41:   auto CC = MF->getFunction().getCallingConv();
  42: 
  43:   if (CC == CallingConv::GHC)
  44:     return CSR_NoRegs_SaveList;
  45:   if (CC == CallingConv::PreserveNone)
  46:     return CSR_NoneRegs_SaveList;
  47:   if (CC == CallingConv::PreserveMost)
  48:     return CSR_MostRegs_SaveList;
  49:   switch (Subtarget.getTargetABI()) {
  50:   default:
  51:     llvm_unreachable("Unrecognized ABI");
  52:   case LoongArchABI::ABI_ILP32S:
  53:   case LoongArchABI::ABI_LP64S:
  54:     return CSR_ILP32S_LP64S_SaveList;
```
- **EN**: The range implements or declares functions including `LoongArchRegisterInfo::getCalleeSavedRegs`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchRegisterInfo::getCalleeSavedRegs` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   case LoongArchABI::ABI_ILP32F:
  56:   case LoongArchABI::ABI_LP64F:
  57:     return CSR_ILP32F_LP64F_SaveList;
  58:   case LoongArchABI::ABI_ILP32D:
  59:   case LoongArchABI::ABI_LP64D:
  60:     return CSR_ILP32D_LP64D_SaveList;
  61:   }
  62: }
  63: 
  64: const uint32_t *
  65: LoongArchRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
  66:                                             CallingConv::ID CC) const {
  67:   auto &Subtarget = MF.getSubtarget<LoongArchSubtarget>();
  68: 
  69:   if (CC == CallingConv::GHC)
  70:     return CSR_NoRegs_RegMask;
  71:   if (CC == CallingConv::PreserveNone)
  72:     return CSR_NoneRegs_RegMask;
```
- **EN**: The range implements or declares functions including `LoongArchRegisterInfo::getCallPreservedMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchRegisterInfo::getCallPreservedMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   if (CC == CallingConv::PreserveMost)
  74:     return CSR_MostRegs_RegMask;
  75:   switch (Subtarget.getTargetABI()) {
  76:   default:
  77:     llvm_unreachable("Unrecognized ABI");
  78:   case LoongArchABI::ABI_ILP32S:
  79:   case LoongArchABI::ABI_LP64S:
  80:     return CSR_ILP32S_LP64S_RegMask;
  81:   case LoongArchABI::ABI_ILP32F:
  82:   case LoongArchABI::ABI_LP64F:
  83:     return CSR_ILP32F_LP64F_RegMask;
  84:   case LoongArchABI::ABI_ILP32D:
  85:   case LoongArchABI::ABI_LP64D:
  86:     return CSR_ILP32D_LP64D_RegMask;
  87:   }
  88: }
  89: 
  90: const uint32_t *LoongArchRegisterInfo::getNoPreservedMask() const {
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   return CSR_NoRegs_RegMask;
  92: }
  93: 
  94: BitVector
  95: LoongArchRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  96:   const LoongArchFrameLowering *TFI = getFrameLowering(MF);
  97:   BitVector Reserved(getNumRegs());
  98: 
  99:   // Use markSuperRegs to ensure any register aliases are also reserved
 100:   markSuperRegs(Reserved, LoongArch::R0);  // zero
 101:   markSuperRegs(Reserved, LoongArch::R2);  // tp
 102:   markSuperRegs(Reserved, LoongArch::R3);  // sp
 103:   markSuperRegs(Reserved, LoongArch::R21); // non-allocatable
 104:   if (TFI->hasFP(MF))
 105:     markSuperRegs(Reserved, LoongArch::R22); // fp
 106:   // Reserve the base register if we need to realign the stack and allocate
 107:   // variable-sized objects at runtime.
 108:   if (TFI->hasBP(MF))
```
- **EN**: The range implements or declares functions including `LoongArchRegisterInfo::getReservedRegs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchRegisterInfo::getReservedRegs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109:     markSuperRegs(Reserved, LoongArchABI::getBPReg()); // bp
 110: 
 111:   assert(checkAllSuperRegsMarked(Reserved));
 112:   return Reserved;
 113: }
 114: 
 115: Register
 116: LoongArchRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
 117:   const TargetFrameLowering *TFI = getFrameLowering(MF);
 118:   return TFI->hasFP(MF) ? LoongArch::R22 : LoongArch::R3;
 119: }
 120: 
 121: bool LoongArchRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
 122:                                                 int SPAdj,
 123:                                                 unsigned FIOperandNum,
 124:                                                 RegScavenger *RS) const {
 125:   // TODO: this implementation is a temporary placeholder which does just
 126:   // enough to allow other aspects of code generation to be tested.
```
- **EN**: The range implements or declares functions including `LoongArchRegisterInfo::getFrameRegister`, `LoongArchRegisterInfo::eliminateFrameIndex`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchRegisterInfo::getFrameRegister`, `LoongArchRegisterInfo::eliminateFrameIndex` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 127-144 / 第 127-144 行
```cpp
 127: 
 128:   assert(SPAdj == 0 && "Unexpected non-zero SPAdj value");
 129: 
 130:   MachineInstr &MI = *II;
 131:   assert(MI.getOperand(FIOperandNum + 1).isImm() &&
 132:          "Unexpected FI-consuming insn");
 133: 
 134:   MachineBasicBlock &MBB = *MI.getParent();
 135:   MachineFunction &MF = *MI.getParent()->getParent();
 136:   MachineRegisterInfo &MRI = MF.getRegInfo();
 137:   const LoongArchSubtarget &STI = MF.getSubtarget<LoongArchSubtarget>();
 138:   const LoongArchInstrInfo *TII = STI.getInstrInfo();
 139:   const TargetFrameLowering *TFI = MF.getSubtarget().getFrameLowering();
 140:   DebugLoc DL = MI.getDebugLoc();
 141:   bool IsLA64 = STI.is64Bit();
 142:   unsigned MIOpc = MI.getOpcode();
 143: 
 144:   int FrameIndex = MI.getOperand(FIOperandNum).getIndex();
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   Register FrameReg;
 146:   StackOffset Offset =
 147:       TFI->getFrameIndexReference(MF, FrameIndex, FrameReg) +
 148:       StackOffset::getFixed(MI.getOperand(FIOperandNum + 1).getImm());
 149: 
 150:   bool FrameRegIsKill = false;
 151: 
 152:   int FixedOffset = Offset.getFixed();
 153:   bool OffsetLegal = true;
 154: 
 155:   // Handle offsets that exceed the immediate range of the instruction.
 156:   switch (MIOpc) {
 157:   case LoongArch::VSTELM_B:
 158:   case LoongArch::XVSTELM_B:
 159:     OffsetLegal = isInt<8>(FixedOffset);
 160:     break;
 161:   case LoongArch::VSTELM_H:
 162:   case LoongArch::XVSTELM_H:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 163-180 / 第 163-180 行
```cpp
 163:     OffsetLegal = isShiftedInt<8, 1>(FixedOffset);
 164:     break;
 165:   case LoongArch::VSTELM_W:
 166:   case LoongArch::XVSTELM_W:
 167:     OffsetLegal = isShiftedInt<8, 2>(FixedOffset);
 168:     break;
 169:   case LoongArch::VSTELM_D:
 170:   case LoongArch::XVSTELM_D:
 171:     OffsetLegal = isShiftedInt<8, 3>(FixedOffset);
 172:     break;
 173:   }
 174: 
 175:   if (!OffsetLegal && isInt<12>(FixedOffset)) {
 176:     unsigned Addi = IsLA64 ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 177: 
 178:     // The offset fits in si12 but is not legal for the instruction,
 179:     // so use only one scratch register instead.
 180:     Register ScratchReg = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:     BuildMI(MBB, II, DL, TII->get(Addi), ScratchReg)
 182:         .addReg(FrameReg)
 183:         .addImm(FixedOffset);
 184:     Offset = StackOffset::getFixed(0);
 185:     FrameReg = ScratchReg;
 186:     FrameRegIsKill = true;
 187:   }
 188: 
 189:   if (!isInt<12>(FixedOffset)) {
 190:     unsigned Addi = IsLA64 ? LoongArch::ADDI_D : LoongArch::ADDI_W;
 191:     unsigned Add = IsLA64 ? LoongArch::ADD_D : LoongArch::ADD_W;
 192: 
 193:     // The offset won't fit in an immediate, so use a scratch register instead.
 194:     // Modify Offset and FrameReg appropriately.
 195:     Register ScratchReg = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
 196:     TII->movImm(MBB, II, DL, ScratchReg, Offset.getFixed());
 197:     if (MIOpc == Addi) {
 198:       BuildMI(MBB, II, DL, TII->get(Add), MI.getOperand(0).getReg())
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199:           .addReg(FrameReg)
 200:           .addReg(ScratchReg, RegState::Kill);
 201:       MI.eraseFromParent();
 202:       return true;
 203:     }
 204:     BuildMI(MBB, II, DL, TII->get(Add), ScratchReg)
 205:         .addReg(FrameReg)
 206:         .addReg(ScratchReg, RegState::Kill);
 207:     Offset = StackOffset::getFixed(0);
 208:     FrameReg = ScratchReg;
 209:     FrameRegIsKill = true;
 210:   }
 211: 
 212:   // Spill CFRs.
 213:   if (MIOpc == LoongArch::PseudoST_CFR) {
 214:     Register ScratchReg = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
 215:     BuildMI(MBB, II, DL, TII->get(LoongArch::MOVCF2GR), ScratchReg)
 216:         .add(MI.getOperand(0));
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 217-234 / 第 217-234 行
```cpp
 217:     BuildMI(MBB, II, DL, TII->get(IsLA64 ? LoongArch::ST_D : LoongArch::ST_W))
 218:         .addReg(ScratchReg, RegState::Kill)
 219:         .addReg(FrameReg)
 220:         .addImm(Offset.getFixed());
 221:     MI.eraseFromParent();
 222:     return true;
 223:   }
 224: 
 225:   // Reload CFRs.
 226:   if (MIOpc == LoongArch::PseudoLD_CFR) {
 227:     Register ScratchReg = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
 228:     BuildMI(MBB, II, DL, TII->get(IsLA64 ? LoongArch::LD_D : LoongArch::LD_W),
 229:             ScratchReg)
 230:         .addReg(FrameReg)
 231:         .addImm(Offset.getFixed());
 232:     BuildMI(MBB, II, DL, TII->get(LoongArch::MOVGR2CF))
 233:         .add(MI.getOperand(0))
 234:         .addReg(ScratchReg, RegState::Kill);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:     MI.eraseFromParent();
 236:     return true;
 237:   }
 238: 
 239:   MI.getOperand(FIOperandNum)
 240:       .ChangeToRegister(FrameReg, false, false, FrameRegIsKill);
 241:   MI.getOperand(FIOperandNum + 1).ChangeToImmediate(Offset.getFixed());
 242:   return false;
 243: }
 244: 
 245: bool LoongArchRegisterInfo::canRealignStack(const MachineFunction &MF) const {
 246:   if (!TargetRegisterInfo::canRealignStack(MF))
 247:     return false;
 248: 
 249:   const MachineRegisterInfo *MRI = &MF.getRegInfo();
 250:   const LoongArchFrameLowering *TFI = getFrameLowering(MF);
 251: 
 252:   // Stack realignment requires a frame pointer.  If we already started
```
- **EN**: The range implements or declares functions including `LoongArchRegisterInfo::canRealignStack`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchRegisterInfo::canRealignStack` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-265 / 第 253-265 行
```cpp
 253:   // register allocation with frame pointer elimination, it is too late now.
 254:   if (!MRI->canReserveReg(LoongArch::R22))
 255:     return false;
 256: 
 257:   // We may also need a base pointer if there are dynamic allocas or stack
 258:   // pointer adjustments around calls.
 259:   if (TFI->hasReservedCallFrame(MF))
 260:     return true;
 261: 
 262:   // A base pointer is required and allowed.  Check that it isn't too late to
 263:   // reserve it.
 264:   return MRI->canReserveReg(LoongArchABI::getBPReg());
 265: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `LoongArchRegisterInfo.h`
- `LoongArch.h`
- `LoongArchInstrInfo.h`
- `LoongArchSubtarget.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/RegisterScavenging.h`
- `llvm/CodeGen/TargetFrameLowering.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `llvm/Support/ErrorHandling.h`
- `LoongArchGenRegisterInfo.inc`
