# HexagonCopyHoisting.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonCopyHoisting.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon copy hoisting across Hexagon code regions.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===--------- HexagonCopyHoisting.cpp - Hexagon Copy Hoisting  ----------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // The purpose of this pass is to move the copy instructions that are
     9: // present in all the successor of a basic block (BB) to the end of BB.
    10: //===----------------------------------------------------------------------===//
    11: 
    12: #include "HexagonTargetMachine.h"
    13: #include "llvm/ADT/DenseMap.h"
    14: #include "llvm/ADT/PostOrderIterator.h"
    15: #include "llvm/ADT/StringRef.h"
    16: #include "llvm/ADT/Twine.h"
    17: #include "llvm/CodeGen/LiveInterval.h"
    18: #include "llvm/CodeGen/LiveIntervals.h"
    19: #include "llvm/CodeGen/MachineDominators.h"
    20: #include "llvm/CodeGen/MachineRegisterInfo.h"
    21: #include "llvm/Support/CommandLine.h"
    22: #include "llvm/Support/Debug.h"
    23: 
    24: #define DEBUG_TYPE "CopyHoist"
    25: 
    26: using namespace llvm;
    27: 
    28: static cl::opt<std::string> CPHoistFn("cphoistfn", cl::Hidden, cl::desc(""),
    29:                                       cl::init(""));
    30: 
    31: namespace {
    32: 
    33: class HexagonCopyHoisting : public MachineFunctionPass {
    34: 
    35: public:
    36:   static char ID;
    37:   HexagonCopyHoisting() : MachineFunctionPass(ID) {}
    38: 
    39:   StringRef getPassName() const override { return "Hexagon Copy Hoisting"; }
    40: 
    41:   void getAnalysisUsage(AnalysisUsage &AU) const override {
    42:     AU.addRequired<SlotIndexesWrapperPass>();
    43:     AU.addRequired<LiveIntervalsWrapperPass>();
    44:     AU.addPreserved<SlotIndexesWrapperPass>();
    45:     AU.addPreserved<LiveIntervalsWrapperPass>();
    46:     AU.addRequired<MachineDominatorTreeWrapperPass>();
    47:     AU.addPreserved<MachineDominatorTreeWrapperPass>();
    48:     MachineFunctionPass::getAnalysisUsage(AU);
    49:   }
    50: 
```
- EN: It imports headers such as HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/ADT/PostOrderIterator.h, llvm/ADT/StringRef.h, ... (11 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonCopyHoisting, which carry the state or API of this component.
- CN: 这里引入了 HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/ADT/PostOrderIterator.h, llvm/ADT/StringRef.h, ... (11 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonCopyHoisting 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   bool runOnMachineFunction(MachineFunction &Fn) override;
    52:   void collectCopyInst();
    53:   void addMItoCopyList(MachineInstr *MI);
    54:   bool analyzeCopy(MachineBasicBlock *BB);
    55:   bool isSafetoMove(MachineInstr *CandMI);
    56:   void moveCopyInstr(MachineBasicBlock *DestBB,
    57:                      std::pair<Register, Register> Key, MachineInstr *MI);
    58: 
    59:   MachineFunction *MFN = nullptr;
    60:   MachineRegisterInfo *MRI = nullptr;
    61:   std::vector<DenseMap<std::pair<Register, Register>, MachineInstr *>>
    62:       CopyMIList;
    63: };
    64: 
    65: } // namespace
    66: 
    67: char HexagonCopyHoisting::ID = 0;
    68: 
    69: namespace llvm {
    70: char &HexagonCopyHoistingID = HexagonCopyHoisting::ID;
    71: } // namespace llvm
    72: 
    73: bool HexagonCopyHoisting::runOnMachineFunction(MachineFunction &Fn) {
    74: 
    75:   if ((CPHoistFn != "") && (CPHoistFn != Fn.getFunction().getName()))
    76:     return false;
    77: 
    78:   MFN = &Fn;
    79:   MRI = &Fn.getRegInfo();
    80: 
    81:   LLVM_DEBUG(dbgs() << "\nCopy Hoisting:" << "\'" << Fn.getName() << "\'\n");
    82: 
    83:   CopyMIList.clear();
    84:   CopyMIList.resize(Fn.getNumBlockIDs());
    85: 
    86:   // Traverse through all basic blocks and collect copy instructions.
    87:   collectCopyInst();
    88: 
    89:   // Traverse through the basic blocks again and move the COPY instructions
    90:   // that are present in all the successors of BB to BB.
    91:   bool Changed = false;
    92:   for (MachineBasicBlock *BB : post_order(&Fn)) {
    93:     if (!BB->empty()) {
    94:       if (BB->pred_size() != 1)
    95:         continue;
    96:       auto &BBCopyInst = CopyMIList[BB->getNumber()];
    97:       if (BBCopyInst.size() > 0)
    98:         Changed |= analyzeCopy(*BB->pred_begin());
    99:     }
   100:   }
```
- EN: It opens namespaces (char, llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as runOnMachineFunction, collectCopyInst, addMItoCopyList, analyzeCopy, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCopyHoisting, HexagonCopyHoistingID, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char, llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 runOnMachineFunction, collectCopyInst, addMItoCopyList, analyzeCopy, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCopyHoisting, HexagonCopyHoistingID，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   // Re-compute liveness
   102:   if (Changed) {
   103:     LiveIntervals &LIS = getAnalysis<LiveIntervalsWrapperPass>().getLIS();
   104:     SlotIndexes *SI = LIS.getSlotIndexes();
   105:     SI->reanalyze(Fn);
   106:     LIS.reanalyze(Fn);
   107:   }
   108:   return Changed;
   109: }
   110: 
   111: //===----------------------------------------------------------------------===//
   112: // Save all COPY instructions for each basic block in CopyMIList vector.
   113: //===----------------------------------------------------------------------===//
   114: void HexagonCopyHoisting::collectCopyInst() {
   115:   for (MachineBasicBlock &BB : *MFN) {
   116: #ifndef NDEBUG
   117:     auto &BBCopyInst = CopyMIList[BB.getNumber()];
   118:     LLVM_DEBUG(dbgs() << "Visiting BB#" << BB.getNumber() << ":\n");
   119: #endif
   120: 
   121:     for (MachineInstr &MI : BB) {
   122:       if (MI.getOpcode() == TargetOpcode::COPY)
   123:         addMItoCopyList(&MI);
   124:     }
   125:     LLVM_DEBUG(dbgs() << "\tNumber of copies: " << BBCopyInst.size() << "\n");
   126:   }
   127: }
   128: 
   129: void HexagonCopyHoisting::addMItoCopyList(MachineInstr *MI) {
   130:   unsigned BBNum = MI->getParent()->getNumber();
   131:   auto &BBCopyInst = CopyMIList[BBNum];
   132:   Register DstReg = MI->getOperand(0).getReg();
   133:   Register SrcReg = MI->getOperand(1).getReg();
   134: 
   135:   if (!Register::isVirtualRegister(DstReg) ||
   136:       !Register::isVirtualRegister(SrcReg) ||
   137:       MRI->getRegClass(DstReg) != &Hexagon::IntRegsRegClass ||
   138:       MRI->getRegClass(SrcReg) != &Hexagon::IntRegsRegClass)
   139:     return;
   140: 
   141:   BBCopyInst.insert(std::pair(std::pair(SrcReg, DstReg), MI));
   142: #ifndef NDEBUG
   143:   LLVM_DEBUG(dbgs() << "\tAdding Copy Instr to the list: " << MI << "\n");
   144:   for (auto II : BBCopyInst) {
   145:     MachineInstr *TempMI = II.getSecond();
   146:     LLVM_DEBUG(dbgs() << "\tIn the list: " << TempMI << "\n");
   147:   }
   148: #endif
   149: }
   150: 
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getAnalysis<LiveIntervalsWrapperPass>, getSlotIndexes, reanalyze, HexagonCopyHoisting::collectCopyInst, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCopyHoisting, showing how the code connects to sibling backend components.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getAnalysis<LiveIntervalsWrapperPass>, getSlotIndexes, reanalyze, HexagonCopyHoisting::collectCopyInst, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCopyHoisting，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: //===----------------------------------------------------------------------===//
   152: // Look at the COPY instructions of all the successors of BB. If the same
   153: // instruction is present in every successor and can be safely moved,
   154: // pull it into BB.
   155: //===----------------------------------------------------------------------===//
   156: bool HexagonCopyHoisting::analyzeCopy(MachineBasicBlock *BB) {
   157: 
   158:   bool Changed = false;
   159:   if (BB->succ_size() < 2)
   160:     return false;
   161: 
   162:   for (MachineBasicBlock *SB : BB->successors()) {
   163:     if (SB->pred_size() != 1 || SB->isEHPad() || SB->hasAddressTaken())
   164:       return false;
   165:   }
   166: 
   167:   MachineBasicBlock *SBB1 = *BB->succ_begin();
   168:   auto &BBCopyInst1 = CopyMIList[SBB1->getNumber()];
   169: 
   170:   for (auto II : BBCopyInst1) {
   171:     std::pair<Register, Register> Key = II.getFirst();
   172:     MachineInstr *MI = II.getSecond();
   173:     bool IsSafetoMove = true;
   174:     for (MachineBasicBlock *SuccBB : BB->successors()) {
   175:       auto &SuccBBCopyInst = CopyMIList[SuccBB->getNumber()];
   176:       auto It = SuccBBCopyInst.find(Key);
   177:       if (It == SuccBBCopyInst.end()) {
   178:         // Same copy not present in this successor
   179:         IsSafetoMove = false;
   180:         break;
   181:       }
   182:       // If present, make sure that it's safe to pull this copy instruction
   183:       // into the predecessor.
   184:       MachineInstr *SuccMI = It->second;
   185:       if (!isSafetoMove(SuccMI)) {
   186:         IsSafetoMove = false;
   187:         break;
   188:       }
   189:     }
   190:     // If we have come this far, this copy instruction can be safely
   191:     // moved to the predecessor basic block.
   192:     if (IsSafetoMove) {
   193:       LLVM_DEBUG(dbgs() << "\t\t Moving instr to BB#" << BB->getNumber() << ": "
   194:                         << MI << "\n");
   195:       moveCopyInstr(BB, Key, MI);
   196:       // Add my into BB copyMI list.
   197:       Changed = true;
   198:     }
   199:   }
   200: 
```
- EN: It declares or implements routines such as HexagonCopyHoisting::analyzeCopy, succ_begin, getFirst, getSecond, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCopyHoisting, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonCopyHoisting::analyzeCopy, succ_begin, getFirst, getSecond, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCopyHoisting，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201: #ifndef NDEBUG
   202:   auto &BBCopyInst = CopyMIList[BB->getNumber()];
   203:   for (auto II : BBCopyInst) {
   204:     MachineInstr *TempMI = II.getSecond();
   205:     LLVM_DEBUG(dbgs() << "\tIn the list: " << TempMI << "\n");
   206:   }
   207: #endif
   208:   return Changed;
   209: }
   210: 
   211: bool HexagonCopyHoisting::isSafetoMove(MachineInstr *CandMI) {
   212:   // Make sure that it's safe to move this 'copy' instruction to the predecessor
   213:   // basic block.
   214:   assert(CandMI->getOperand(0).isReg() && CandMI->getOperand(1).isReg());
   215:   Register DefR = CandMI->getOperand(0).getReg();
   216:   Register UseR = CandMI->getOperand(1).getReg();
   217: 
   218:   MachineBasicBlock *BB = CandMI->getParent();
   219:   // There should not be a def/use of DefR between the start of BB and CandMI.
   220:   MachineBasicBlock::iterator MII, MIE;
   221:   for (MII = BB->begin(), MIE = CandMI; MII != MIE; ++MII) {
   222:     MachineInstr *OtherMI = &*MII;
   223:     for (const MachineOperand &Mo : OtherMI->operands())
   224:       if (Mo.isReg() && Mo.getReg() == DefR)
   225:         return false;
   226:   }
   227:   // There should not be a def of UseR between the start of BB and CandMI.
   228:   for (MII = BB->begin(), MIE = CandMI; MII != MIE; ++MII) {
   229:     MachineInstr *OtherMI = &*MII;
   230:     for (const MachineOperand &Mo : OtherMI->operands())
   231:       if (Mo.isReg() && Mo.isDef() && Mo.getReg() == UseR)
   232:         return false;
   233:   }
   234:   return true;
   235: }
   236: 
   237: void HexagonCopyHoisting::moveCopyInstr(MachineBasicBlock *DestBB,
   238:                                         std::pair<Register, Register> Key,
   239:                                         MachineInstr *MI) {
   240:   MachineBasicBlock::iterator FirstTI = DestBB->getFirstTerminator();
   241:   assert(FirstTI != DestBB->end());
   242: 
   243:   DestBB->splice(FirstTI, MI->getParent(), MI);
   244: 
   245:   addMItoCopyList(MI);
   246:   for (MachineBasicBlock *SuccBB : drop_begin(DestBB->successors())) {
   247:     auto &BBCopyInst = CopyMIList[SuccBB->getNumber()];
   248:     MachineInstr *SuccMI = BBCopyInst[Key];
   249:     SuccMI->eraseFromParent();
   250:     BBCopyInst.erase(Key);
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getSecond, LLVM_DEBUG, HexagonCopyHoisting::isSafetoMove, assert, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getSecond, LLVM_DEBUG, HexagonCopyHoisting::isSafetoMove, assert, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 251-263 / 第 251-263 行

```cpp
   251:   }
   252: }
   253: 
   254: //===----------------------------------------------------------------------===//
   255: //                         Public Constructor Functions
   256: //===----------------------------------------------------------------------===//
   257: 
   258: INITIALIZE_PASS(HexagonCopyHoisting, "hexagon-move-phicopy",
   259:                 "Hexagon move phi copy", false, false)
   260: 
   261: FunctionPass *llvm::createHexagonCopyHoisting() {
   262:   return new HexagonCopyHoisting();
   263: }
```
- EN: It declares or implements routines such as INITIALIZE_PASS, HexagonCopyHoisting, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonCopyHoisting, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 INITIALIZE_PASS, HexagonCopyHoisting 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonCopyHoisting，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/ADT/PostOrderIterator.h, llvm/ADT/StringRef.h, llvm/ADT/Twine.h, llvm/CodeGen/LiveInterval.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/Support/CommandLine.h, ... (11 total)`
- Hexagon symbols / Hexagon 符号: `HexagonCopyHoisting, HexagonTargetMachine, HexagonCopyHoistingID`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
