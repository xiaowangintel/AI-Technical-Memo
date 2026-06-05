# HexagonCFGOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonCFGOptimizer.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon control-flow graph cleanup and simplification.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonCFGOptimizer.cpp - CFG optimizations ------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    11: #include "llvm/CodeGen/MachineBasicBlock.h"
    12: #include "llvm/CodeGen/MachineFunction.h"
    13: #include "llvm/CodeGen/MachineFunctionPass.h"
    14: #include "llvm/CodeGen/MachineInstr.h"
    15: #include "llvm/CodeGen/MachineOperand.h"
    16: #include "llvm/CodeGen/TargetInstrInfo.h"
    17: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    18: #include "llvm/Pass.h"
    19: #include "llvm/Support/ErrorHandling.h"
    20: #include <cassert>
    21: #include <vector>
    22: 
    23: using namespace llvm;
    24: 
    25: #define DEBUG_TYPE "hexagon_cfg"
    26: 
    27: namespace {
    28: 
    29: class HexagonCFGOptimizer : public MachineFunctionPass {
    30: private:
    31:   void InvertAndChangeJumpTarget(MachineInstr &, MachineBasicBlock *);
    32:   bool isOnFallThroughPath(MachineBasicBlock *MBB);
    33: 
    34: public:
    35:   static char ID;
    36: 
    37:   HexagonCFGOptimizer() : MachineFunctionPass(ID) {}
    38: 
    39:   StringRef getPassName() const override { return "Hexagon CFG Optimizer"; }
    40:   bool runOnMachineFunction(MachineFunction &Fn) override;
    41: 
    42:   MachineFunctionProperties getRequiredProperties() const override {
    43:     return MachineFunctionProperties().setNoVRegs();
    44:   }
    45: };
    46: 
    47: } // end anonymous namespace
    48: 
    49: char HexagonCFGOptimizer::ID = 0;
    50: 
```
- EN: It imports headers such as Hexagon.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, ... (13 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonCFGOptimizer, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, ... (13 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonCFGOptimizer 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51: static bool IsConditionalBranch(int Opc) {
    52:   switch (Opc) {
    53:     case Hexagon::J2_jumpt:
    54:     case Hexagon::J2_jumptpt:
    55:     case Hexagon::J2_jumpf:
    56:     case Hexagon::J2_jumpfpt:
    57:     case Hexagon::J2_jumptnew:
    58:     case Hexagon::J2_jumpfnew:
    59:     case Hexagon::J2_jumptnewpt:
    60:     case Hexagon::J2_jumpfnewpt:
    61:       return true;
    62:   }
    63:   return false;
    64: }
    65: 
    66: static bool IsUnconditionalJump(int Opc) {
    67:   return (Opc == Hexagon::J2_jump);
    68: }
    69: 
    70: void HexagonCFGOptimizer::InvertAndChangeJumpTarget(
    71:     MachineInstr &MI, MachineBasicBlock *NewTarget) {
    72:   const TargetInstrInfo *TII =
    73:       MI.getParent()->getParent()->getSubtarget().getInstrInfo();
    74:   int NewOpcode = 0;
    75:   switch (MI.getOpcode()) {
    76:   case Hexagon::J2_jumpt:
    77:     NewOpcode = Hexagon::J2_jumpf;
    78:     break;
    79:   case Hexagon::J2_jumpf:
    80:     NewOpcode = Hexagon::J2_jumpt;
    81:     break;
    82:   case Hexagon::J2_jumptnewpt:
    83:     NewOpcode = Hexagon::J2_jumpfnewpt;
    84:     break;
    85:   case Hexagon::J2_jumpfnewpt:
    86:     NewOpcode = Hexagon::J2_jumptnewpt;
    87:     break;
    88:   default:
    89:     llvm_unreachable("Cannot handle this case");
    90:   }
    91: 
    92:   MI.setDesc(TII->get(NewOpcode));
    93:   MI.getOperand(1).setMBB(NewTarget);
    94: }
    95: 
    96: bool HexagonCFGOptimizer::isOnFallThroughPath(MachineBasicBlock *MBB) {
    97:   if (MBB->canFallThrough())
    98:     return true;
    99:   for (MachineBasicBlock *PB : MBB->predecessors())
   100:     if (PB->isLayoutSuccessor(MBB) && PB->canFallThrough())
```
- EN: It declares or implements routines such as IsConditionalBranch, IsUnconditionalJump, HexagonCFGOptimizer::InvertAndChangeJumpTarget, getParent, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCFGOptimizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 IsConditionalBranch, IsUnconditionalJump, HexagonCFGOptimizer::InvertAndChangeJumpTarget, getParent, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCFGOptimizer，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:       return true;
   102:   return false;
   103: }
   104: 
   105: bool HexagonCFGOptimizer::runOnMachineFunction(MachineFunction &Fn) {
   106:   if (skipFunction(Fn.getFunction()))
   107:     return false;
   108: 
   109:   // Loop over all of the basic blocks.
   110:   for (MachineBasicBlock &MBB : Fn) {
   111:     // Traverse the basic block.
   112:     MachineBasicBlock::iterator MII = MBB.getFirstTerminator();
   113:     if (MII != MBB.end()) {
   114:       MachineInstr &MI = *MII;
   115:       int Opc = MI.getOpcode();
   116:       if (IsConditionalBranch(Opc)) {
   117:         // (Case 1) Transform the code if the following condition occurs:
   118:         //   BB1: if (p0) jump BB3
   119:         //   ...falls-through to BB2 ...
   120:         //   BB2: jump BB4
   121:         //   ...next block in layout is BB3...
   122:         //   BB3: ...
   123:         //
   124:         //  Transform this to:
   125:         //  BB1: if (!p0) jump BB4
   126:         //  Remove BB2
   127:         //  BB3: ...
   128:         //
   129:         // (Case 2) A variation occurs when BB3 contains a JMP to BB4:
   130:         //   BB1: if (p0) jump BB3
   131:         //   ...falls-through to BB2 ...
   132:         //   BB2: jump BB4
   133:         //   ...other basic blocks ...
   134:         //   BB4:
   135:         //   ...not a fall-thru
   136:         //   BB3: ...
   137:         //     jump BB4
   138:         //
   139:         // Transform this to:
   140:         //   BB1: if (!p0) jump BB4
   141:         //   Remove BB2
   142:         //   BB3: ...
   143:         //   BB4: ...
   144:         unsigned NumSuccs = MBB.succ_size();
   145:         MachineBasicBlock::succ_iterator SI = MBB.succ_begin();
   146:         MachineBasicBlock* FirstSucc = *SI;
   147:         MachineBasicBlock* SecondSucc = *(++SI);
   148:         MachineBasicBlock* LayoutSucc = nullptr;
   149:         MachineBasicBlock* JumpAroundTarget = nullptr;
   150: 
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as HexagonCFGOptimizer::runOnMachineFunction, getFirstTerminator, getOpcode, succ_begin, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCFGOptimizer, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 HexagonCFGOptimizer::runOnMachineFunction, getFirstTerminator, getOpcode, succ_begin 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCFGOptimizer，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:         if (MBB.isLayoutSuccessor(FirstSucc)) {
   152:           LayoutSucc = FirstSucc;
   153:           JumpAroundTarget = SecondSucc;
   154:         } else if (MBB.isLayoutSuccessor(SecondSucc)) {
   155:           LayoutSucc = SecondSucc;
   156:           JumpAroundTarget = FirstSucc;
   157:         } else {
   158:           // Odd case...cannot handle.
   159:         }
   160: 
   161:         // The target of the unconditional branch must be JumpAroundTarget.
   162:         // TODO: If not, we should not invert the unconditional branch.
   163:         MachineBasicBlock* CondBranchTarget = nullptr;
   164:         if (MI.getOpcode() == Hexagon::J2_jumpt ||
   165:             MI.getOpcode() == Hexagon::J2_jumpf) {
   166:           CondBranchTarget = MI.getOperand(1).getMBB();
   167:         }
   168: 
   169:         if (!LayoutSucc || (CondBranchTarget != JumpAroundTarget)) {
   170:           continue;
   171:         }
   172: 
   173:         if ((NumSuccs == 2) && LayoutSucc && (LayoutSucc->pred_size() == 1)) {
   174:           // Ensure that BB2 has one instruction -- an unconditional jump.
   175:           if ((LayoutSucc->size() == 1) &&
   176:               IsUnconditionalJump(LayoutSucc->front().getOpcode())) {
   177:             assert(JumpAroundTarget && "jump target is needed to process second basic block");
   178:             MachineBasicBlock* UncondTarget =
   179:               LayoutSucc->front().getOperand(0).getMBB();
   180:             // Check if the layout successor of BB2 is BB3.
   181:             bool case1 = LayoutSucc->isLayoutSuccessor(JumpAroundTarget);
   182:             bool case2 = JumpAroundTarget->isSuccessor(UncondTarget) &&
   183:               !JumpAroundTarget->empty() &&
   184:               IsUnconditionalJump(JumpAroundTarget->back().getOpcode()) &&
   185:               JumpAroundTarget->pred_size() == 1 &&
   186:               JumpAroundTarget->succ_size() == 1;
   187: 
   188:             if (case1 || case2) {
   189:               InvertAndChangeJumpTarget(MI, UncondTarget);
   190:               MBB.replaceSuccessor(JumpAroundTarget, UncondTarget);
   191: 
   192:               // Remove the unconditional branch in LayoutSucc.
   193:               LayoutSucc->erase(LayoutSucc->begin());
   194:               LayoutSucc->replaceSuccessor(UncondTarget, JumpAroundTarget);
   195: 
   196:               // This code performs the conversion for case 2, which moves
   197:               // the block to the fall-thru case (BB3 in the code above).
   198:               if (case2 && !case1) {
   199:                 JumpAroundTarget->moveAfter(LayoutSucc);
   200:                 // only move a block if it doesn't have a fall-thru. otherwise
```
- EN: It declares or implements routines such as getOperand, assert, front, isLayoutSuccessor, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getOperand, assert, front, isLayoutSuccessor, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-236 / 第 201-236 行

```cpp
   201:                 // the CFG will be incorrect.
   202:                 if (!isOnFallThroughPath(UncondTarget))
   203:                   UncondTarget->moveAfter(JumpAroundTarget);
   204:               }
   205: 
   206:               // Correct live-in information. Is used by post-RA scheduler
   207:               // The live-in to LayoutSucc is now all values live-in to
   208:               // JumpAroundTarget.
   209:               std::vector<MachineBasicBlock::RegisterMaskPair> OrigLiveIn(
   210:                   LayoutSucc->livein_begin(), LayoutSucc->livein_end());
   211:               std::vector<MachineBasicBlock::RegisterMaskPair> NewLiveIn(
   212:                   JumpAroundTarget->livein_begin(),
   213:                   JumpAroundTarget->livein_end());
   214:               for (const auto &OrigLI : OrigLiveIn)
   215:                 LayoutSucc->removeLiveIn(OrigLI.PhysReg);
   216:               for (const auto &NewLI : NewLiveIn)
   217:                 LayoutSucc->addLiveIn(NewLI);
   218:             }
   219:           }
   220:         }
   221:       }
   222:     }
   223:   }
   224:   return true;
   225: }
   226: 
   227: //===----------------------------------------------------------------------===//
   228: //                         Public Constructor Functions
   229: //===----------------------------------------------------------------------===//
   230: 
   231: INITIALIZE_PASS(HexagonCFGOptimizer, "hexagon-cfg", "Hexagon CFG Optimizer",
   232:                 false, false)
   233: 
   234: FunctionPass *llvm::createHexagonCFGOptimizer() {
   235:   return new HexagonCFGOptimizer();
   236: }
```
- EN: It declares or implements routines such as OrigLiveIn, NewLiveIn, INITIALIZE_PASS, HexagonCFGOptimizer, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCFGOptimizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 OrigLiveIn, NewLiveIn, INITIALIZE_PASS, HexagonCFGOptimizer 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCFGOptimizer，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetSubtargetInfo.h, llvm/Pass.h, ... (13 total)`
- Hexagon symbols / Hexagon 符号: `HexagonCFGOptimizer, HexagonMCTargetDesc`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
