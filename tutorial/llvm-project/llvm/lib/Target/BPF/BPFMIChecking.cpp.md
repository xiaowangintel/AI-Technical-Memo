# BPFMIChecking.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFMIChecking.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-------------- BPFMIChecking.cpp - MI Checking Legality -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass performs checking to signal errors for certain illegal usages at
  10: // MachineInstruction layer. Specially, the result of XADD{32,64} insn should
  11: // not be used. The pass is done at the PreEmit pass right before the
  12: // machine code is emitted at which point the register liveness information
  13: // is still available.
  14: //
  15: //===----------------------------------------------------------------------===//
  16: 
  17: #include "BPF.h"
  18: #include "BPFTargetMachine.h"
  19: #include "llvm/CodeGen/MachineFunctionPass.h"
  20: #include "llvm/IR/DiagnosticInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/Support/Debug.h"
  22: 
  23: using namespace llvm;
  24: 
  25: #define DEBUG_TYPE "bpf-mi-checking"
  26: 
  27: namespace {
  28: 
  29: struct BPFMIPreEmitChecking : public MachineFunctionPass {
  30: 
  31:   static char ID;
  32:   MachineFunction *MF;
  33:   const TargetRegisterInfo *TRI;
  34: 
  35:   BPFMIPreEmitChecking() : MachineFunctionPass(ID) {}
  36: 
  37: private:
  38:   // Initialize class variables.
  39:   void initialize(MachineFunction &MFParm);
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as BPFMIPreEmitChecking, variables, which organize the target-specific behavior exposed by the file.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 BPFMIPreEmitChecking, variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41:   void processAtomicInsts();
  42: 
  43: public:
  44:   // Main entry point for this pass.
  45:   bool runOnMachineFunction(MachineFunction &MF) override {
  46:     if (!skipFunction(MF.getFunction())) {
  47:       initialize(MF);
  48:       processAtomicInsts();
  49:     }
  50:     return false;
  51:   }
  52: };
  53: 
  54: // Initialize class variables.
  55: void BPFMIPreEmitChecking::initialize(MachineFunction &MFParm) {
  56:   MF = &MFParm;
  57:   TRI = MF->getSubtarget<BPFSubtarget>().getRegisterInfo();
  58:   LLVM_DEBUG(dbgs() << "*** BPF PreEmit checking pass ***\n\n");
  59: }
  60: 
```

- EN: This chunk introduces interfaces or data structures such as variables, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as initialize contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。 initialize 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 61-80

```cpp
  61: // Make sure all Defs of XADD are dead, meaning any result of XADD insn is not
  62: // used.
  63: //
  64: // NOTE: BPF backend hasn't enabled sub-register liveness track, so when the
  65: // source and destination operands of XADD are GPR32, there is no sub-register
  66: // dead info. If we rely on the generic MachineInstr::allDefsAreDead, then we
  67: // will raise false alarm on GPR32 Def.
  68: //
  69: // To support GPR32 Def, ideally we could just enable sub-registr liveness track
  70: // on BPF backend, then allDefsAreDead could work on GPR32 Def. This requires
  71: // implementing TargetSubtargetInfo::enableSubRegLiveness on BPF.
  72: //
  73: // However, sub-register liveness tracking module inside LLVM is actually
  74: // designed for the situation where one register could be split into more than
  75: // one sub-registers for which case each sub-register could have their own
  76: // liveness and kill one of them doesn't kill others. So, tracking liveness for
  77: // each make sense.
  78: //
  79: // For BPF, each 64-bit register could only have one 32-bit sub-register. This
  80: // is exactly the case which LLVM think brings no benefits for doing
```

- EN: This range continues the implementation of the backend component described by BPFMIChecking.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81: // sub-register tracking, because the live range of sub-register must always
  82: // equal to its parent register, therefore liveness tracking is disabled even
  83: // the back-end has implemented enableSubRegLiveness. The detailed information
  84: // is at r232695:
  85: //
  86: //   Author: Matthias Braun <matze@braunis.de>
  87: //   Date:   Thu Mar 19 00:21:58 2015 +0000
  88: //   Do not track subregister liveness when it brings no benefits
  89: //
  90: // Hence, for BPF, we enhance MachineInstr::allDefsAreDead. Given the solo
  91: // sub-register always has the same liveness as its parent register, LLVM is
  92: // already attaching a implicit 64-bit register Def whenever the there is
  93: // a sub-register Def. The liveness of the implicit 64-bit Def is available.
  94: // For example, for "lock *(u32 *)(r0 + 4) += w9", the MachineOperand info could
  95: // be:
  96: //
  97: //   $w9 = XADDW32 killed $r0, 4, $w9(tied-def 0),
  98: //                        implicit killed $r9, implicit-def dead $r9
  99: //
 100: // Even though w9 is not marked as Dead, the parent register r9 is marked as
```

- EN: This range continues the implementation of the backend component described by BPFMIChecking.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 101-120

```cpp
 101: // Dead correctly, and it is safe to use such information or our purpose.
 102: static bool hasLiveDefs(const MachineInstr &MI, const TargetRegisterInfo *TRI) {
 103:   const MCRegisterClass *GPR64RegClass =
 104:       &BPFMCRegisterClasses[BPF::GPRRegClassID];
 105:   std::vector<unsigned> GPR32LiveDefs;
 106:   std::vector<unsigned> GPR64DeadDefs;
 107: 
 108:   for (const MachineOperand &MO : MI.operands()) {
 109:     bool RegIsGPR64;
 110: 
 111:     if (!MO.isReg() || MO.isUse())
 112:       continue;
 113: 
 114:     RegIsGPR64 = GPR64RegClass->contains(MO.getReg());
 115:     if (!MO.isDead()) {
 116:       // It is a GPR64 live Def, we are sure it is live.
 117:       if (RegIsGPR64)
 118:         return true;
 119:       // It is a GPR32 live Def, we are unsure whether it is really dead due to
 120:       // no sub-register liveness tracking. Push it to vector for deferred
```

- EN: Function bodies or method definitions such as hasLiveDefs contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: hasLiveDefs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 121-140

```cpp
 121:       // check.
 122:       GPR32LiveDefs.push_back(MO.getReg());
 123:       continue;
 124:     }
 125: 
 126:     // Record any GPR64 dead Def as some unmarked GPR32 could be alias of its
 127:     // low 32-bit.
 128:     if (RegIsGPR64)
 129:       GPR64DeadDefs.push_back(MO.getReg());
 130:   }
 131: 
 132:   // No GPR32 live Def, safe to return false.
 133:   if (GPR32LiveDefs.empty())
 134:     return false;
 135: 
 136:   // No GPR64 dead Def, so all those GPR32 live Def can't have alias, therefore
 137:   // must be truely live, safe to return true.
 138:   if (GPR64DeadDefs.empty())
 139:     return true;
 140: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:   // Otherwise, return true if any aliased SuperReg of GPR32 is not dead.
 142:   for (auto I : GPR32LiveDefs)
 143:     for (MCPhysReg SR : TRI->superregs(I))
 144:       if (!llvm::is_contained(GPR64DeadDefs, SR))
 145:         return true;
 146: 
 147:   return false;
 148: }
 149: 
 150: void BPFMIPreEmitChecking::processAtomicInsts() {
 151:   if (MF->getSubtarget<BPFSubtarget>().getHasJmp32())
 152:     return;
 153: 
 154:   // Only check for cpu version 1 and 2.
 155:   for (MachineBasicBlock &MBB : *MF) {
 156:     for (MachineInstr &MI : MBB) {
 157:       if (MI.getOpcode() != BPF::XADDW && MI.getOpcode() != BPF::XADDD)
 158:         continue;
 159: 
 160:       LLVM_DEBUG(MI.dump());
```

- EN: Function bodies or method definitions such as processAtomicInsts contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: processAtomicInsts 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-179

```cpp
 161:       if (hasLiveDefs(MI, TRI)) {
 162:         const DebugLoc &DL = MI.getDebugLoc();
 163:         const Function &F = MF->getFunction();
 164:         F.getContext().diagnose(DiagnosticInfoUnsupported{
 165:             F, "Invalid usage of the XADD return value", DL});
 166:       }
 167:     }
 168:   }
 169: }
 170: 
 171: } // namespace
 172: 
 173: INITIALIZE_PASS(BPFMIPreEmitChecking, "bpf-mi-pemit-checking",
 174:                 "BPF PreEmit Checking", false, false)
 175: 
 176: char BPFMIPreEmitChecking::ID = 0;
 177: FunctionPass *llvm::createBPFMIPreEmitCheckingPass() {
 178:   return new BPFMIPreEmitChecking();
 179: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFTargetMachine.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/Support/Debug.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
