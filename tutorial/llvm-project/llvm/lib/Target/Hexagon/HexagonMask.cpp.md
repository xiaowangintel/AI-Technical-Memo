# HexagonMask.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonMask.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon mask-oriented rewriting and analysis.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===-- HexagonMask.cpp - replace const ext tfri with mask ------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: //===----------------------------------------------------------------------===//
    10: 
    11: #include "Hexagon.h"
    12: #include "HexagonSubtarget.h"
    13: #include "llvm/ADT/Statistic.h"
    14: #include "llvm/CodeGen/MachineFunction.h"
    15: #include "llvm/CodeGen/MachineFunctionPass.h"
    16: #include "llvm/CodeGen/MachineInstrBuilder.h"
    17: #include "llvm/CodeGen/Passes.h"
    18: #include "llvm/IR/Function.h"
    19: #include "llvm/Support/MathExtras.h"
    20: #include "llvm/Target/TargetMachine.h"
    21: 
    22: #define DEBUG_TYPE "mask"
    23: 
    24: using namespace llvm;
    25: 
```
- EN: It imports headers such as Hexagon.h, HexagonSubtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunction.h, ... (10 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 Hexagon.h, HexagonSubtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunction.h, ... (10 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 26-50 / 第 26-50 行

```cpp
    26: namespace {
    27: class HexagonMask : public MachineFunctionPass {
    28: public:
    29:   static char ID;
    30:   HexagonMask() : MachineFunctionPass(ID) {}
    31: 
    32:   StringRef getPassName() const override {
    33:     return "Hexagon replace const ext tfri with mask";
    34:   }
    35:   bool runOnMachineFunction(MachineFunction &MF) override;
    36: 
    37: private:
    38:   const HexagonInstrInfo *HII;
    39:   void replaceConstExtTransferImmWithMask(MachineFunction &MF);
    40: };
    41: } // end anonymous namespace
    42: 
    43: char HexagonMask::ID = 0;
    44: 
    45: void HexagonMask::replaceConstExtTransferImmWithMask(MachineFunction &MF) {
    46:   for (auto &MBB : MF) {
    47:     for (auto &MI : llvm::make_early_inc_range(MBB)) {
    48:       if (MI.getOpcode() != Hexagon::A2_tfrsi)
    49:         continue;
    50: 
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonMask, which carry the state or API of this component. It defines declarative TableGen records like HexagonMask; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as HexagonMask, getPassName, runOnMachineFunction, replaceConstExtTransferImmWithMask, ... (5 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonMask 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonMask 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 HexagonMask, getPassName, runOnMachineFunction, replaceConstExtTransferImmWithMask, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-75 / 第 51-75 行

```cpp
    51:       const MachineOperand &Op0 = MI.getOperand(0);
    52:       const MachineOperand &Op1 = MI.getOperand(1);
    53:       if (!Op1.isImm())
    54:         continue;
    55:       int32_t V = Op1.getImm();
    56:       if (isInt<16>(V))
    57:         continue;
    58: 
    59:       unsigned Idx, Len;
    60:       if (!isShiftedMask_32(V, Idx, Len))
    61:         continue;
    62:       if (!isUInt<5>(Idx) || !isUInt<5>(Len))
    63:         continue;
    64: 
    65:       BuildMI(MBB, MI, MI.getDebugLoc(), HII->get(Hexagon::S2_mask),
    66:               Op0.getReg())
    67:           .addImm(Len)
    68:           .addImm(Idx);
    69:       MBB.erase(MI);
    70:     }
    71:   }
    72: }
    73: 
    74: bool HexagonMask::runOnMachineFunction(MachineFunction &MF) {
    75:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
```
- EN: It declares or implements routines such as getOperand, getImm, BuildMI, erase, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMask, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, getImm, BuildMI, erase, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMask, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 76-99 / 第 76-99 行

```cpp
    76:   HII = HST.getInstrInfo();
    77:   const Function &F = MF.getFunction();
    78: 
    79:   if (!F.hasOptSize())
    80:     return false;
    81:   // Mask instruction is available only from v66
    82:   if (!HST.hasV66Ops())
    83:     return false;
    84:   // The mask instruction available in v66 can be used to generate values in
    85:   // registers using 2 immediates Eg. to form 0x07fffffc in R0, you would write
    86:   // "R0 = mask(#25,#2)" Since it is a single-word instruction, it takes less
    87:   // code size than a constant-extended transfer at Os
    88:   replaceConstExtTransferImmWithMask(MF);
    89: 
    90:   return true;
    91: }
    92: 
    93: //===----------------------------------------------------------------------===//
    94: //                         Public Constructor Functions
    95: //===----------------------------------------------------------------------===//
    96: 
    97: INITIALIZE_PASS(HexagonMask, "hexagon-mask", "Hexagon mask", false, false)
    98: 
    99: FunctionPass *llvm::createHexagonMask() { return new HexagonMask(); }
```
- EN: It declares or implements routines such as getInstrInfo, getFunction, mask, INITIALIZE_PASS, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMask, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getInstrInfo, getFunction, mask, INITIALIZE_PASS, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMask，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonSubtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/Passes.h, llvm/IR/Function.h, llvm/Support/MathExtras.h, llvm/Target/TargetMachine.h`
- Hexagon symbols / Hexagon 符号: `HexagonMask, HexagonSubtarget, HexagonInstrInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
