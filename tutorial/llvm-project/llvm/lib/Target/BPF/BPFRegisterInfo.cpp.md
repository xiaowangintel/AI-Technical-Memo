# BPFRegisterInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFRegisterInfo.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file contains the BPF implementation of the TargetRegisterInfo class.
- 目的（中文）: 定义目标寄存器属性、被调用者保存规则以及寄存器分配辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFRegisterInfo.cpp - BPF Register Information ----------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the BPF implementation of the TargetRegisterInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "BPFRegisterInfo.h"
  14: #include "BPFSubtarget.h"
  15: #include "llvm/CodeGen/MachineFrameInfo.h"
  16: #include "llvm/CodeGen/MachineFunction.h"
  17: #include "llvm/CodeGen/MachineInstrBuilder.h"
  18: #include "llvm/CodeGen/RegisterScavenging.h"
  19: #include "llvm/CodeGen/TargetFrameLowering.h"
  20: #include "llvm/CodeGen/TargetInstrInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 21-40

```cpp
  21: #include "llvm/IR/DiagnosticInfo.h"
  22: #include "llvm/Support/CommandLine.h"
  23: #include "llvm/Support/ErrorHandling.h"
  24: 
  25: #define GET_REGINFO_TARGET_DESC
  26: #include "BPFGenRegisterInfo.inc"
  27: using namespace llvm;
  28: 
  29: static cl::opt<int>
  30:     BPFStackSizeOption("bpf-stack-size",
  31:                        cl::desc("Specify the BPF stack size limit"),
  32:                        cl::init(512));
  33: 
  34: BPFRegisterInfo::BPFRegisterInfo()
  35:     : BPFGenRegisterInfo(BPF::R0) {}
  36: 
  37: const MCPhysReg *
  38: BPFRegisterInfo::getCalleeSavedRegs(const MachineFunction *MF) const {
  39:   return CSR_SaveList;
  40: }
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as BPFGenRegisterInfo, getCalleeSavedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 BPFGenRegisterInfo, getCalleeSavedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 41-60

```cpp
  41: 
  42: const uint32_t *
  43: BPFRegisterInfo::getCallPreservedMask(const MachineFunction &MF,
  44:                                       CallingConv::ID CC) const {
  45:   switch (CC) {
  46:   default:
  47:     return CSR_RegMask;
  48:   case CallingConv::PreserveAll:
  49:     return CSR_PreserveAll_RegMask;
  50:   }
  51: }
  52: 
  53: BitVector BPFRegisterInfo::getReservedRegs(const MachineFunction &MF) const {
  54:   BitVector Reserved(getNumRegs());
  55:   markSuperRegs(Reserved, BPF::W10); // [W|R]10 is read only frame pointer
  56:   markSuperRegs(Reserved, BPF::W11); // [W|R]11 is pseudo stack pointer
  57:   return Reserved;
  58: }
  59: 
  60: static void WarnSize(int Offset, MachineFunction &MF, DebugLoc& DL,
```

- EN: Function bodies or method definitions such as getCallPreservedMask, getReservedRegs contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getCallPreservedMask, getReservedRegs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 61-80

```cpp
  61:                      MachineBasicBlock& MBB) {
  62:   if (Offset <= -BPFStackSizeOption) {
  63:     if (!DL)
  64:       /* try harder to get some debug loc */
  65:       for (auto &I : MBB)
  66:         if (I.getDebugLoc()) {
  67:           DL = I.getDebugLoc();
  68:           break;
  69:         }
  70: 
  71:     const Function &F = MF.getFunction();
  72:     F.getContext().diagnose(DiagnosticInfoUnsupported(
  73:         F,
  74:         "Looks like the BPF stack limit is exceeded. "
  75:         "Please move large on stack variables into BPF per-cpu array map. For "
  76:         "non-kernel uses, the stack can be increased using -mllvm "
  77:         "-bpf-stack-size.\n",
  78:         DL));
  79:   }
  80: }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 81-100

```cpp
  81: 
  82: bool BPFRegisterInfo::eliminateFrameIndex(MachineBasicBlock::iterator II,
  83:                                           int SPAdj, unsigned FIOperandNum,
  84:                                           RegScavenger *RS) const {
  85:   assert(SPAdj == 0 && "Unexpected");
  86: 
  87:   unsigned i = 0;
  88:   MachineInstr &MI = *II;
  89:   MachineBasicBlock &MBB = *MI.getParent();
  90:   MachineFunction &MF = *MBB.getParent();
  91:   DebugLoc DL = MI.getDebugLoc();
  92: 
  93:   while (!MI.getOperand(i).isFI()) {
  94:     ++i;
  95:     assert(i < MI.getNumOperands() && "Instr doesn't have FrameIndex operand!");
  96:   }
  97: 
  98:   Register FrameReg = getFrameRegister(MF);
  99:   int FrameIndex = MI.getOperand(i).getIndex();
 100:   const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
```

- EN: Function bodies or method definitions such as eliminateFrameIndex contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this range manages register properties and allocation-related rules.
- 中文: eliminateFrameIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 101-120

```cpp
 101: 
 102:   if (MI.getOpcode() == BPF::MOV_rr) {
 103:     int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex);
 104: 
 105:     WarnSize(Offset, MF, DL, MBB);
 106:     MI.getOperand(i).ChangeToRegister(FrameReg, false);
 107:     Register reg = MI.getOperand(i - 1).getReg();
 108:     BuildMI(MBB, ++II, DL, TII.get(BPF::ADD_ri), reg)
 109:         .addReg(reg)
 110:         .addImm(Offset);
 111:     return false;
 112:   }
 113: 
 114:   int Offset = MF.getFrameInfo().getObjectOffset(FrameIndex) +
 115:                MI.getOperand(i + 1).getImm();
 116: 
 117:   if (!isInt<32>(Offset))
 118:     llvm_unreachable("bug in frame offset");
 119: 
 120:   WarnSize(Offset, MF, DL, MBB);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 121-140

```cpp
 121: 
 122:   if (MI.getOpcode() == BPF::FI_ri) {
 123:     // architecture does not really support FI_ri, replace it with
 124:     //    MOV_rr <target_reg>, frame_reg
 125:     //    ADD_ri <target_reg>, imm
 126:     Register reg = MI.getOperand(i - 1).getReg();
 127: 
 128:     BuildMI(MBB, ++II, DL, TII.get(BPF::MOV_rr), reg)
 129:         .addReg(FrameReg);
 130:     BuildMI(MBB, II, DL, TII.get(BPF::ADD_ri), reg)
 131:         .addReg(reg)
 132:         .addImm(Offset);
 133: 
 134:     // Remove FI_ri instruction
 135:     MI.eraseFromParent();
 136:   } else {
 137:     MI.getOperand(i).ChangeToRegister(FrameReg, false);
 138:     MI.getOperand(i + 1).ChangeToImmediate(Offset);
 139:   }
 140:   return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this range manages register properties and allocation-related rules.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

### Lines 141-145

```cpp
 141: }
 142: 
 143: Register BPFRegisterInfo::getFrameRegister(const MachineFunction &MF) const {
 144:   return BPF::R10;
 145: }
```

- EN: Function bodies or method definitions such as getFrameRegister contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this range manages register properties and allocation-related rules.
- 中文: getFrameRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段管理寄存器属性及与分配相关的规则。

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

- Direct includes / 直接包含: `BPFRegisterInfo.h`, `BPFSubtarget.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/RegisterScavenging.h`, `llvm/CodeGen/TargetFrameLowering.h`, `llvm/CodeGen/TargetInstrInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Generated or companion files / 生成或配套文件: `BPFGenRegisterInfo.inc`
- Local companions / 本地配套文件: `BPFRegisterInfo.h`, `BPFRegisterInfo.td`
