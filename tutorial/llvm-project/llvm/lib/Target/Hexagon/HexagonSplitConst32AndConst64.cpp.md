# HexagonSplitConst32AndConst64.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonSplitConst32AndConst64.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon splitting of large constants into legal forms.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //=== HexagonSplitConst32AndConst64.cpp - split CONST32/Const64 into HI/LO ===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // When the compiler is invoked with no small data, for instance, with the -G0
    10: // command line option, then all CONST* opcodes should be broken down into
    11: // appropriate LO and HI instructions. This splitting is done by this pass.
    12: // The only reason this is not done in the DAG lowering itself is that there
    13: // is no simple way of getting the register allocator to allot the same hard
    14: // register to the result of LO and HI instructions. This pass is always
    15: // scheduled after register allocation.
    16: //
    17: //===----------------------------------------------------------------------===//
    18: 
    19: #include "Hexagon.h"
    20: #include "HexagonSubtarget.h"
    21: #include "HexagonTargetMachine.h"
    22: #include "HexagonTargetObjectFile.h"
    23: #include "llvm/CodeGen/MachineFunctionPass.h"
    24: #include "llvm/CodeGen/MachineInstrBuilder.h"
    25: #include "llvm/CodeGen/Passes.h"
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as Hexagon.h, HexagonSubtarget.h, HexagonTargetMachine.h, HexagonTargetObjectFile.h, ... (7 total), establishing the LLVM/Hexagon APIs used below. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitConst32AndConst64, HexagonSubtarget, HexagonTargetMachine, HexagonTargetObjectFile, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 Hexagon.h, HexagonSubtarget.h, HexagonTargetMachine.h, HexagonTargetObjectFile.h, ... (7 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitConst32AndConst64, HexagonSubtarget, HexagonTargetMachine, HexagonTargetObjectFile，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: #include "llvm/CodeGen/TargetInstrInfo.h"
    27: #include "llvm/CodeGen/TargetRegisterInfo.h"
    28: 
    29: using namespace llvm;
    30: 
    31: #define DEBUG_TYPE "xfer"
    32: 
    33: namespace {
    34:   class HexagonSplitConst32AndConst64 : public MachineFunctionPass {
    35:   public:
    36:     static char ID;
    37:     HexagonSplitConst32AndConst64() : MachineFunctionPass(ID) {}
    38:     StringRef getPassName() const override {
    39:       return "Hexagon Split Const32s and Const64s";
    40:     }
    41:     bool runOnMachineFunction(MachineFunction &Fn) override;
    42:     MachineFunctionProperties getRequiredProperties() const override {
    43:       return MachineFunctionProperties().setNoVRegs();
    44:     }
    45:   };
    46: }
    47: 
    48: char HexagonSplitConst32AndConst64::ID = 0;
    49: 
    50: INITIALIZE_PASS(HexagonSplitConst32AndConst64, "split-const-for-sdata",
```
- EN: It imports headers such as llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetRegisterInfo.h, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonSplitConst32AndConst64, which carry the state or API of this component.
- CN: 这里引入了 llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetRegisterInfo.h 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonSplitConst32AndConst64 等类型，用来承载该组件的状态或接口。

### Lines 51-75 / 第 51-75 行

```cpp
    51:       "Hexagon Split Const32s and Const64s", false, false)
    52: 
    53: bool HexagonSplitConst32AndConst64::runOnMachineFunction(MachineFunction &Fn) {
    54:   auto &HST = Fn.getSubtarget<HexagonSubtarget>();
    55:   auto &HTM = static_cast<const HexagonTargetMachine&>(Fn.getTarget());
    56:   auto &TLOF = *HTM.getObjFileLowering();
    57:   if (HST.useSmallData() && TLOF.isSmallDataEnabled(HTM))
    58:     return false;
    59: 
    60:   const TargetInstrInfo *TII = HST.getInstrInfo();
    61:   const TargetRegisterInfo *TRI = HST.getRegisterInfo();
    62: 
    63:   // Loop over all of the basic blocks
    64:   for (MachineBasicBlock &B : Fn) {
    65:     for (MachineInstr &MI : llvm::make_early_inc_range(B)) {
    66:       unsigned Opc = MI.getOpcode();
    67: 
    68:       if (Opc == Hexagon::CONST32) {
    69:         Register DestReg = MI.getOperand(0).getReg();
    70:         uint64_t ImmValue = MI.getOperand(1).getImm();
    71:         const DebugLoc &DL = MI.getDebugLoc();
    72:         BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), DestReg)
    73:             .addImm(ImmValue);
    74:         B.erase(&MI);
    75:       } else if (Opc == Hexagon::CONST64) {
```
- EN: It declares or implements routines such as HexagonSplitConst32AndConst64::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getTarget, getObjFileLowering, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSplitConst32AndConst64, HexagonSubtarget, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonSplitConst32AndConst64::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getTarget, getObjFileLowering, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSplitConst32AndConst64, HexagonSubtarget, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:         Register DestReg = MI.getOperand(0).getReg();
    77:         int64_t ImmValue = MI.getOperand(1).getImm();
    78:         const DebugLoc &DL = MI.getDebugLoc();
    79:         Register DestLo = TRI->getSubReg(DestReg, Hexagon::isub_lo);
    80:         Register DestHi = TRI->getSubReg(DestReg, Hexagon::isub_hi);
    81: 
    82:         int32_t LowWord = (ImmValue & 0xFFFFFFFF);
    83:         int32_t HighWord = (ImmValue >> 32) & 0xFFFFFFFF;
    84: 
    85:         BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), DestLo)
    86:             .addImm(LowWord);
    87:         BuildMI(B, MI, DL, TII->get(Hexagon::A2_tfrsi), DestHi)
    88:             .addImm(HighWord);
    89:         B.erase(&MI);
    90:       }
    91:     }
    92:   }
    93: 
    94:   return true;
    95: }
    96: 
    97: 
    98: //===----------------------------------------------------------------------===//
    99: //                         Public Constructor Functions
   100: //===----------------------------------------------------------------------===//
```
- EN: It declares or implements routines such as getOperand, getDebugLoc, getSubReg, BuildMI, ... (5 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 getOperand, getDebugLoc, getSubReg, BuildMI, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-103 / 第 101-103 行

```cpp
   101: FunctionPass *llvm::createHexagonSplitConst32AndConst64() {
   102:   return new HexagonSplitConst32AndConst64();
   103: }
```
- EN: It declares or implements routines such as llvm::createHexagonSplitConst32AndConst64, HexagonSplitConst32AndConst64, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonSplitConst32AndConst64, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonSplitConst32AndConst64, HexagonSplitConst32AndConst64 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonSplitConst32AndConst64，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonSubtarget.h, HexagonTargetMachine.h, HexagonTargetObjectFile.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetRegisterInfo.h`
- Hexagon symbols / Hexagon 符号: `HexagonSplitConst32AndConst64, HexagonSubtarget, HexagonTargetMachine, HexagonTargetObjectFile`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
