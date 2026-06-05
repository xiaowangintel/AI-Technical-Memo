# HexagonBranchRelaxation.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonBranchRelaxation.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon branch-range relaxation and rewriting.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===--- HexagonBranchRelaxation.cpp - Identify and relax long jumps ------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: #include "HexagonInstrInfo.h"
    11: #include "HexagonSubtarget.h"
    12: #include "llvm/ADT/DenseMap.h"
    13: #include "llvm/ADT/SmallVector.h"
    14: #include "llvm/ADT/StringRef.h"
    15: #include "llvm/CodeGen/MachineBasicBlock.h"
    16: #include "llvm/CodeGen/MachineFunction.h"
    17: #include "llvm/CodeGen/MachineFunctionPass.h"
    18: #include "llvm/CodeGen/MachineInstr.h"
    19: #include "llvm/CodeGen/MachineOperand.h"
    20: #include "llvm/CodeGen/Passes.h"
    21: #include "llvm/Pass.h"
    22: #include "llvm/Support/CommandLine.h"
    23: #include "llvm/Support/Debug.h"
    24: #include "llvm/Support/raw_ostream.h"
    25: #include <cassert>
    26: #include <cstdint>
    27: #include <cstdlib>
    28: #include <iterator>
    29: 
    30: #define DEBUG_TYPE "hexagon-brelax"
    31: 
    32: using namespace llvm;
    33: 
    34: // Since we have no exact knowledge of code layout, allow some safety buffer
    35: // for jump target. This is measured in bytes.
    36: static cl::opt<uint32_t>
    37:     BranchRelaxSafetyBuffer("branch-relax-safety-buffer", cl::init(200),
    38:                             cl::Hidden, cl::desc("safety buffer size"));
    39: 
    40: namespace {
    41: 
    42:   struct HexagonBranchRelaxation : public MachineFunctionPass {
    43:   public:
    44:     static char ID;
    45: 
    46:     HexagonBranchRelaxation() : MachineFunctionPass(ID) {}
    47: 
    48:     bool runOnMachineFunction(MachineFunction &MF) override;
    49: 
    50:     StringRef getPassName() const override {
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseMap.h, ... (20 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonBranchRelaxation, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseMap.h, ... (20 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonBranchRelaxation 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:       return "Hexagon Branch Relaxation";
    52:     }
    53: 
    54:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    55:       AU.setPreservesCFG();
    56:       MachineFunctionPass::getAnalysisUsage(AU);
    57:     }
    58: 
    59:   private:
    60:     const HexagonInstrInfo *HII;
    61:     const HexagonRegisterInfo *HRI;
    62: 
    63:     bool relaxBranches(MachineFunction &MF);
    64:     void computeOffset(MachineFunction &MF,
    65:           DenseMap<MachineBasicBlock*, unsigned> &BlockToInstOffset);
    66:     bool reGenerateBranch(MachineFunction &MF,
    67:           DenseMap<MachineBasicBlock*, unsigned> &BlockToInstOffset);
    68:     bool isJumpOutOfRange(MachineInstr &MI,
    69:           DenseMap<MachineBasicBlock*, unsigned> &BlockToInstOffset);
    70:   };
    71: 
    72:   char HexagonBranchRelaxation::ID = 0;
    73: 
    74: } // end anonymous namespace
    75: 
    76: INITIALIZE_PASS(HexagonBranchRelaxation, "hexagon-brelax",
    77:                 "Hexagon Branch Relaxation", false, false)
    78: 
    79: FunctionPass *llvm::createHexagonBranchRelaxation() {
    80:   return new HexagonBranchRelaxation();
    81: }
    82: 
    83: bool HexagonBranchRelaxation::runOnMachineFunction(MachineFunction &MF) {
    84:   LLVM_DEBUG(dbgs() << "****** Hexagon Branch Relaxation ******\n");
    85: 
    86:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
    87:   HII = HST.getInstrInfo();
    88:   HRI = HST.getRegisterInfo();
    89: 
    90:   bool Changed = false;
    91:   Changed = relaxBranches(MF);
    92:   return Changed;
    93: }
    94: 
    95: void HexagonBranchRelaxation::computeOffset(MachineFunction &MF,
    96:       DenseMap<MachineBasicBlock*, unsigned> &OffsetMap) {
    97:   // offset of the current instruction from the start.
    98:   unsigned InstOffset = 0;
    99:   for (auto &B : MF) {
   100:     if (B.getAlignment() != Align(1)) {
```
- EN: It opens namespaces (INITIALIZE_PASS) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as getAnalysisUsage, setPreservesCFG, MachineFunctionPass::getAnalysisUsage, relaxBranches, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonRegisterInfo, HexagonBranchRelaxation, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（INITIALIZE_PASS），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 getAnalysisUsage, setPreservesCFG, MachineFunctionPass::getAnalysisUsage, relaxBranches, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonRegisterInfo, HexagonBranchRelaxation, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:       // Although we don't know the exact layout of the final code, we need
   102:       // to account for alignment padding somehow. This heuristic pads each
   103:       // aligned basic block according to the alignment value.
   104:       InstOffset = alignTo(InstOffset, B.getAlignment());
   105:     }
   106:     OffsetMap[&B] = InstOffset;
   107:     for (auto &MI : B.instrs()) {
   108:       InstOffset += HII->getSize(MI);
   109:       // Assume that all extendable branches will be extended.
   110:       if (MI.isBranch() && HII->isExtendable(MI))
   111:         InstOffset += HEXAGON_INSTR_SIZE;
   112:     }
   113:   }
   114: }
   115: 
   116: /// relaxBranches - For Hexagon, if the jump target/loop label is too far from
   117: /// the jump/loop instruction then, we need to make sure that we have constant
   118: /// extenders set for jumps and loops.
   119: 
   120: /// There are six iterations in this phase. It's self explanatory below.
   121: bool HexagonBranchRelaxation::relaxBranches(MachineFunction &MF) {
   122:   // Compute the offset of each basic block
   123:   // offset of the current instruction from the start.
   124:   // map for each instruction to the beginning of the function
   125:   DenseMap<MachineBasicBlock*, unsigned> BlockToInstOffset;
   126:   computeOffset(MF, BlockToInstOffset);
   127: 
   128:   return reGenerateBranch(MF, BlockToInstOffset);
   129: }
   130: 
   131: /// Check if a given instruction is:
   132: /// - a jump to a distant target
   133: /// - that exceeds its immediate range
   134: /// If both conditions are true, it requires constant extension.
   135: bool HexagonBranchRelaxation::isJumpOutOfRange(MachineInstr &MI,
   136:       DenseMap<MachineBasicBlock*, unsigned> &BlockToInstOffset) {
   137:   MachineBasicBlock &B = *MI.getParent();
   138:   auto FirstTerm = B.getFirstInstrTerminator();
   139:   if (FirstTerm == B.instr_end())
   140:     return false;
   141: 
   142:   if (HII->isExtended(MI))
   143:     return false;
   144: 
   145:   unsigned InstOffset = BlockToInstOffset[&B];
   146:   unsigned Distance = 0;
   147: 
   148:   // To save time, estimate exact position of a branch instruction
   149:   // as one at the end of the MBB.
   150:   // Number of instructions times typical instruction size.
```
- EN: It declares or implements routines such as alignTo, getSize, HexagonBranchRelaxation::relaxBranches, computeOffset, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBranchRelaxation, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 alignTo, getSize, HexagonBranchRelaxation::relaxBranches, computeOffset, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBranchRelaxation，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   InstOffset += HII->nonDbgBBSize(&B) * HEXAGON_INSTR_SIZE;
   152: 
   153:   MachineBasicBlock *TBB = nullptr, *FBB = nullptr;
   154:   SmallVector<MachineOperand, 4> Cond;
   155: 
   156:   // Try to analyze this branch.
   157:   if (HII->analyzeBranch(B, TBB, FBB, Cond, false)) {
   158:     // Could not analyze it. See if this is something we can recognize.
   159:     // If it is a NVJ, it should always have its target in
   160:     // a fixed location.
   161:     if (HII->isNewValueJump(*FirstTerm))
   162:       TBB = FirstTerm->getOperand(HII->getCExtOpNum(*FirstTerm)).getMBB();
   163:   }
   164:   if (TBB && &MI == &*FirstTerm) {
   165:     Distance = std::abs((long long)InstOffset - BlockToInstOffset[TBB])
   166:                 + BranchRelaxSafetyBuffer;
   167:     return !HII->isJumpWithinBranchRange(*FirstTerm, Distance);
   168:   }
   169:   if (FBB) {
   170:     // Look for second terminator.
   171:     auto SecondTerm = std::next(FirstTerm);
   172:     assert(SecondTerm != B.instr_end() &&
   173:           (SecondTerm->isBranch() || SecondTerm->isCall()) &&
   174:           "Bad second terminator");
   175:     if (&MI != &*SecondTerm)
   176:       return false;
   177:     // Analyze the second branch in the BB.
   178:     Distance = std::abs((long long)InstOffset - BlockToInstOffset[FBB])
   179:                 + BranchRelaxSafetyBuffer;
   180:     return !HII->isJumpWithinBranchRange(*SecondTerm, Distance);
   181:   }
   182:   return false;
   183: }
   184: 
   185: bool HexagonBranchRelaxation::reGenerateBranch(MachineFunction &MF,
   186:       DenseMap<MachineBasicBlock*, unsigned> &BlockToInstOffset) {
   187:   bool Changed = false;
   188: 
   189:   for (auto &B : MF) {
   190:     for (auto &MI : B) {
   191:       if (!MI.isBranch() || !isJumpOutOfRange(MI, BlockToInstOffset))
   192:         continue;
   193:       LLVM_DEBUG(dbgs() << "Long distance jump. isExtendable("
   194:                         << HII->isExtendable(MI) << ") isConstExtended("
   195:                         << HII->isConstExtended(MI) << ") " << MI);
   196: 
   197:       // Since we have not merged HW loops relaxation into
   198:       // this code (yet), soften our approach for the moment.
   199:       if (!HII->isExtendable(MI) && !HII->isExtended(MI)) {
   200:         LLVM_DEBUG(dbgs() << "\tUnderimplemented relax branch instruction.\n");
```
- EN: It declares or implements routines such as isJumpWithinBranchRange, std::next, assert, HexagonBranchRelaxation::reGenerateBranch, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonBranchRelaxation, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isJumpWithinBranchRange, std::next, assert, HexagonBranchRelaxation::reGenerateBranch, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonBranchRelaxation，说明了它与同级后端组件的连接关系。

### Lines 201-216 / 第 201-216 行

```cpp
   201:       } else {
   202:         // Find which operand is expandable.
   203:         int ExtOpNum = HII->getCExtOpNum(MI);
   204:         MachineOperand &MO = MI.getOperand(ExtOpNum);
   205:         // This need to be something we understand. So far we assume all
   206:         // branches have only MBB address as expandable field.
   207:         // If it changes, this will need to be expanded.
   208:         assert(MO.isMBB() && "Branch with unknown expandable field type");
   209:         // Mark given operand as extended.
   210:         MO.addTargetFlag(HexagonII::HMOTF_ConstExtended);
   211:         Changed = true;
   212:       }
   213:     }
   214:   }
   215:   return Changed;
   216: }
```
- EN: It declares or implements routines such as getCExtOpNum, getOperand, assert, addTargetFlag, translating Hexagon-specific policy into reusable code paths. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getCExtOpNum, getOperand, assert, addTargetFlag 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/DenseMap.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, ... (20 total)`
- Hexagon symbols / Hexagon 符号: `HexagonBranchRelaxation, HexagonInstrInfo, HexagonSubtarget, HexagonRegisterInfo, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
