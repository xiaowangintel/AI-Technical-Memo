# HexagonFixupHwLoops.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonFixupHwLoops.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon post-processing and repair of hardware-loop instructions.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及指令语义与选择。 重点涉及循环优化。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===---- HexagonFixupHwLoops.cpp - Fixup HW loops too far from LOOPn. ----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: // The loop start address in the LOOPn instruction is encoded as a distance
     8: // from the LOOPn instruction itself. If the start address is too far from
     9: // the LOOPn instruction, the instruction needs to use a constant extender.
    10: // This pass will identify and convert such LOOPn instructions to a proper
    11: // form.
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "Hexagon.h"
    15: #include "HexagonTargetMachine.h"
    16: #include "llvm/ADT/DenseMap.h"
    17: #include "llvm/CodeGen/MachineFunction.h"
    18: #include "llvm/CodeGen/MachineFunctionPass.h"
    19: #include "llvm/CodeGen/MachineInstrBuilder.h"
    20: #include "llvm/CodeGen/Passes.h"
    21: #include "llvm/CodeGen/TargetInstrInfo.h"
    22: #include "llvm/Support/MathExtras.h"
    23: #include "llvm/Pass.h"
    24: 
    25: using namespace llvm;
```
- EN: It imports headers such as Hexagon.h, HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/CodeGen/MachineFunction.h, ... (10 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFixupHwLoops, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这里引入了 Hexagon.h, HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/CodeGen/MachineFunction.h, ... (10 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFixupHwLoops, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: 
    27: static cl::opt<unsigned> MaxLoopRange(
    28:     "hexagon-loop-range", cl::Hidden, cl::init(200),
    29:     cl::desc("Restrict range of loopN instructions (testing only)"));
    30: 
    31: namespace {
    32:   struct HexagonFixupHwLoops : public MachineFunctionPass {
    33:   public:
    34:     static char ID;
    35: 
    36:     HexagonFixupHwLoops() : MachineFunctionPass(ID) {}
    37: 
    38:     bool runOnMachineFunction(MachineFunction &MF) override;
    39: 
    40:     MachineFunctionProperties getRequiredProperties() const override {
    41:       return MachineFunctionProperties().setNoVRegs();
    42:     }
    43: 
    44:     StringRef getPassName() const override {
    45:       return "Hexagon Hardware Loop Fixup";
    46:     }
    47: 
    48:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    49:       AU.setPreservesCFG();
    50:       MachineFunctionPass::getAnalysisUsage(AU);
```
- EN: It declares types such as HexagonFixupHwLoops, which carry the state or API of this component. It declares or implements routines such as MaxLoopRange, HexagonFixupHwLoops, runOnMachineFunction, getRequiredProperties, ... (9 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior. Notable Hexagon symbols referenced here include HexagonFixupHwLoops, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonFixupHwLoops 等类型，用来承载该组件的状态或接口。 这里声明或实现了 MaxLoopRange, HexagonFixupHwLoops, runOnMachineFunction, getRequiredProperties, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。 这里引用的重要 Hexagon 符号包括 HexagonFixupHwLoops，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```cpp
    51:     }
    52: 
    53:   private:
    54:     /// Check the offset between each loop instruction and
    55:     /// the loop basic block to determine if we can use the LOOP instruction
    56:     /// or if we need to set the LC/SA registers explicitly.
    57:     bool fixupLoopInstrs(MachineFunction &MF);
    58: 
    59:     /// Replace loop instruction with the constant extended
    60:     /// version if the loop label is too far from the loop instruction.
    61:     void useExtLoopInstr(MachineFunction &MF,
    62:                          MachineBasicBlock::iterator &MII);
    63:   };
    64: 
    65:   char HexagonFixupHwLoops::ID = 0;
    66: }
    67: 
    68: INITIALIZE_PASS(HexagonFixupHwLoops, "hwloopsfixup",
    69:                 "Hexagon Hardware Loops Fixup", false, false)
    70: 
    71: FunctionPass *llvm::createHexagonFixupHwLoops() {
    72:   return new HexagonFixupHwLoops();
    73: }
    74: 
    75: /// Returns true if the instruction is a hardware loop instruction.
```
- EN: It declares or implements routines such as fixupLoopInstrs, useExtLoopInstr, INITIALIZE_PASS, HexagonFixupHwLoops, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFixupHwLoops, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 fixupLoopInstrs, useExtLoopInstr, INITIALIZE_PASS, HexagonFixupHwLoops 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFixupHwLoops，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76: static bool isHardwareLoop(const MachineInstr &MI) {
    77:   return MI.getOpcode() == Hexagon::J2_loop0r ||
    78:          MI.getOpcode() == Hexagon::J2_loop0i ||
    79:          MI.getOpcode() == Hexagon::J2_loop1r ||
    80:          MI.getOpcode() == Hexagon::J2_loop1i;
    81: }
    82: 
    83: bool HexagonFixupHwLoops::runOnMachineFunction(MachineFunction &MF) {
    84:   if (skipFunction(MF.getFunction()))
    85:     return false;
    86:   return fixupLoopInstrs(MF);
    87: }
    88: 
    89: /// For Hexagon, if the loop label is to far from the
    90: /// loop instruction then we need to set the LC0 and SA0 registers
    91: /// explicitly instead of using LOOP(start,count).  This function
    92: /// checks the distance, and generates register assignments if needed.
    93: ///
    94: /// This function makes two passes over the basic blocks.  The first
    95: /// pass computes the offset of the basic block from the start.
    96: /// The second pass checks all the loop instructions.
    97: bool HexagonFixupHwLoops::fixupLoopInstrs(MachineFunction &MF) {
    98: 
    99:   // Offset of the current instruction from the start.
   100:   unsigned InstOffset = 0;
```
- EN: It declares or implements routines such as isHardwareLoop, HexagonFixupHwLoops::runOnMachineFunction, fixupLoopInstrs, LOOP, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFixupHwLoops, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isHardwareLoop, HexagonFixupHwLoops::runOnMachineFunction, fixupLoopInstrs, LOOP 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFixupHwLoops，说明了它与同级后端组件的连接关系。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   // Map for each basic block to it's first instruction.
   102:   DenseMap<const MachineBasicBlock *, unsigned> BlockToInstOffset;
   103: 
   104:   const HexagonInstrInfo *HII =
   105:       static_cast<const HexagonInstrInfo *>(MF.getSubtarget().getInstrInfo());
   106: 
   107:   // First pass - compute the offset of each basic block.
   108:   for (const MachineBasicBlock &MBB : MF) {
   109:     if (MBB.getAlignment() != Align(1)) {
   110:       // Although we don't know the exact layout of the final code, we need
   111:       // to account for alignment padding somehow. This heuristic pads each
   112:       // aligned basic block according to the alignment value.
   113:       InstOffset = alignTo(InstOffset, MBB.getAlignment());
   114:     }
   115: 
   116:     BlockToInstOffset[&MBB] = InstOffset;
   117:     for (const MachineInstr &MI : MBB)
   118:       InstOffset += HII->getSize(MI);
   119:   }
   120: 
   121:   // Second pass - check each loop instruction to see if it needs to be
   122:   // converted.
   123:   bool Changed = false;
   124:   for (MachineBasicBlock &MBB : MF) {
   125:     InstOffset = BlockToInstOffset[&MBB];
```
- EN: It declares or implements routines such as getSubtarget, alignTo, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget, alignTo 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```cpp
   126: 
   127:     // Loop over all the instructions.
   128:     MachineBasicBlock::iterator MII = MBB.begin();
   129:     MachineBasicBlock::iterator MIE = MBB.end();
   130:     while (MII != MIE) {
   131:       unsigned InstSize = HII->getSize(*MII);
   132:       if (MII->isMetaInstruction()) {
   133:         ++MII;
   134:         continue;
   135:       }
   136:       if (isHardwareLoop(*MII)) {
   137:         assert(MII->getOperand(0).isMBB() &&
   138:                "Expect a basic block as loop operand");
   139:         MachineBasicBlock *TargetBB = MII->getOperand(0).getMBB();
   140:         unsigned Diff = AbsoluteDifference(InstOffset,
   141:                                            BlockToInstOffset[TargetBB]);
   142:         if (Diff > MaxLoopRange) {
   143:           useExtLoopInstr(MF, MII);
   144:           MII = MBB.erase(MII);
   145:           Changed = true;
   146:         } else {
   147:           ++MII;
   148:         }
   149:       } else {
   150:         ++MII;
```
- EN: It declares or implements routines such as begin, end, getSize, assert, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 begin, end, getSize, assert, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 151-175 / 第 151-175 行

```cpp
   151:       }
   152:       InstOffset += InstSize;
   153:     }
   154:   }
   155: 
   156:   return Changed;
   157: }
   158: 
   159: /// Replace loop instructions with the constant extended version.
   160: void HexagonFixupHwLoops::useExtLoopInstr(MachineFunction &MF,
   161:                                           MachineBasicBlock::iterator &MII) {
   162:   const TargetInstrInfo *TII = MF.getSubtarget().getInstrInfo();
   163:   MachineBasicBlock *MBB = MII->getParent();
   164:   DebugLoc DL = MII->getDebugLoc();
   165:   MachineInstrBuilder MIB;
   166:   unsigned newOp;
   167:   switch (MII->getOpcode()) {
   168:   case Hexagon::J2_loop0r:
   169:     newOp = Hexagon::J2_loop0rext;
   170:     break;
   171:   case Hexagon::J2_loop0i:
   172:     newOp = Hexagon::J2_loop0iext;
   173:     break;
   174:   case Hexagon::J2_loop1r:
   175:     newOp = Hexagon::J2_loop1rext;
```
- EN: It declares or implements routines such as HexagonFixupHwLoops::useExtLoopInstr, getSubtarget, getParent, getDebugLoc, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonFixupHwLoops, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonFixupHwLoops::useExtLoopInstr, getSubtarget, getParent, getDebugLoc 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonFixupHwLoops，说明了它与同级后端组件的连接关系。

### Lines 176-187 / 第 176-187 行

```cpp
   176:     break;
   177:   case Hexagon::J2_loop1i:
   178:     newOp = Hexagon::J2_loop1iext;
   179:     break;
   180:   default:
   181:     llvm_unreachable("Invalid Hardware Loop Instruction.");
   182:   }
   183:   MIB = BuildMI(*MBB, MII, DL, TII->get(newOp));
   184: 
   185:   for (unsigned i = 0; i < MII->getNumOperands(); ++i)
   186:     MIB.add(MII->getOperand(i));
   187: }
```
- EN: It declares or implements routines such as llvm_unreachable, BuildMI, getNumOperands, add, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 llvm_unreachable, BuildMI, getNumOperands, add 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h, llvm/Support/MathExtras.h, llvm/Pass.h`
- Hexagon symbols / Hexagon 符号: `HexagonFixupHwLoops, HexagonTargetMachine, HexagonInstrInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
