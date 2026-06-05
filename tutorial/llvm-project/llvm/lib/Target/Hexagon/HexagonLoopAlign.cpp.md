# HexagonLoopAlign.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonLoopAlign.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon loop alignment decisions.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及循环优化。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===----- HexagonLoopAlign.cpp - Generate loop alignment directives  -----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // Inspect a basic block and if its single basic block loop with a small
     9: // number of instructions, set the prefLoopAlignment to 32 bytes (5).
    10: //===----------------------------------------------------------------------===//
    11: 
    12: #include "Hexagon.h"
    13: #include "HexagonTargetMachine.h"
    14: #include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
    15: #include "llvm/CodeGen/MachineBranchProbabilityInfo.h"
    16: #include "llvm/Support/Debug.h"
    17: 
    18: #define DEBUG_TYPE "hexagon-loop-align"
    19: 
    20: using namespace llvm;
    21: 
    22: static cl::opt<bool>
    23:     DisableLoopAlign("disable-hexagon-loop-align", cl::Hidden,
    24:                      cl::desc("Disable Hexagon loop alignment pass"));
    25: 
    26: static cl::opt<uint32_t> HVXLoopAlignLimitUB(
    27:     "hexagon-hvx-loop-align-limit-ub", cl::Hidden, cl::init(16),
    28:     cl::desc("Set hexagon hvx loop upper bound align limit"));
    29: 
    30: static cl::opt<uint32_t> TinyLoopAlignLimitUB(
    31:     "hexagon-tiny-loop-align-limit-ub", cl::Hidden, cl::init(16),
    32:     cl::desc("Set hexagon tiny-core loop upper bound align limit"));
    33: 
    34: static cl::opt<uint32_t>
    35:     LoopAlignLimitUB("hexagon-loop-align-limit-ub", cl::Hidden, cl::init(8),
    36:                      cl::desc("Set hexagon loop upper bound align limit"));
    37: 
    38: static cl::opt<uint32_t>
    39:     LoopAlignLimitLB("hexagon-loop-align-limit-lb", cl::Hidden, cl::init(4),
    40:                      cl::desc("Set hexagon loop lower bound align limit"));
    41: 
    42: static cl::opt<uint32_t>
    43:     LoopBndlAlignLimit("hexagon-loop-bundle-align-limit", cl::Hidden,
    44:                        cl::init(4),
    45:                        cl::desc("Set hexagon loop align bundle limit"));
    46: 
    47: static cl::opt<uint32_t> TinyLoopBndlAlignLimit(
    48:     "hexagon-tiny-loop-bundle-align-limit", cl::Hidden, cl::init(8),
    49:     cl::desc("Set hexagon tiny-core loop align bundle limit"));
    50: 
```
- EN: It imports headers such as Hexagon.h, HexagonTargetMachine.h, llvm/CodeGen/MachineBlockFrequencyInfo.h, llvm/CodeGen/MachineBranchProbabilityInfo.h, ... (5 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as DisableLoopAlign, HVXLoopAlignLimitUB, TinyLoopAlignLimitUB, LoopAlignLimitUB, ... (7 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 Hexagon.h, HexagonTargetMachine.h, llvm/CodeGen/MachineBlockFrequencyInfo.h, llvm/CodeGen/MachineBranchProbabilityInfo.h, ... (5 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 DisableLoopAlign, HVXLoopAlignLimitUB, TinyLoopAlignLimitUB, LoopAlignLimitUB, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51: static cl::opt<uint32_t>
    52:     LoopEdgeThreshold("hexagon-loop-edge-threshold", cl::Hidden, cl::init(7500),
    53:                       cl::desc("Set hexagon loop align edge threshold"));
    54: 
    55: namespace {
    56: 
    57: class HexagonLoopAlign : public MachineFunctionPass {
    58:   const HexagonSubtarget *HST = nullptr;
    59:   const TargetMachine *HTM = nullptr;
    60:   const HexagonInstrInfo *HII = nullptr;
    61: 
    62: public:
    63:   static char ID;
    64:   HexagonLoopAlign() : MachineFunctionPass(ID) {}
    65:   bool shouldBalignLoop(MachineBasicBlock &BB, bool AboveThres);
    66:   bool isSingleLoop(MachineBasicBlock &MBB);
    67:   bool attemptToBalignSmallLoop(MachineFunction &MF, MachineBasicBlock &MBB);
    68: 
    69:   void getAnalysisUsage(AnalysisUsage &AU) const override {
    70:     AU.addRequired<MachineBranchProbabilityInfoWrapperPass>();
    71:     AU.addRequired<MachineBlockFrequencyInfoWrapperPass>();
    72:     MachineFunctionPass::getAnalysisUsage(AU);
    73:   }
    74: 
    75:   StringRef getPassName() const override { return "Hexagon LoopAlign pass"; }
    76:   bool runOnMachineFunction(MachineFunction &MF) override;
    77: };
    78: 
    79: char HexagonLoopAlign::ID = 0;
    80: 
    81: bool HexagonLoopAlign::shouldBalignLoop(MachineBasicBlock &BB,
    82:                                         bool AboveThres) {
    83:   bool isVec = false;
    84:   unsigned InstCnt = 0;
    85:   unsigned BndlCnt = 0;
    86: 
    87:   for (MachineBasicBlock::instr_iterator II = BB.instr_begin(),
    88:                                          IE = BB.instr_end();
    89:        II != IE; ++II) {
    90: 
    91:     // End if the instruction is endloop.
    92:     if (HII->isEndLoopN(II->getOpcode()))
    93:       break;
    94:     // Count the number of bundles.
    95:     if (II->isBundle()) {
    96:       BndlCnt++;
    97:       continue;
    98:     }
    99:     // Skip over debug instructions.
   100:     if (II->isDebugInstr())
```
- EN: It declares types such as HexagonLoopAlign, which carry the state or API of this component. It defines declarative TableGen records like HexagonLoopAlign; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as LoopEdgeThreshold, HexagonLoopAlign, shouldBalignLoop, isSingleLoop, ... (12 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里声明了 HexagonLoopAlign 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonLoopAlign 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 LoopEdgeThreshold, HexagonLoopAlign, shouldBalignLoop, isSingleLoop, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 101-150 / 第 101-150 行

```cpp
   101:       continue;
   102:     // Check if there are any HVX instructions in loop.
   103:     isVec |= HII->isHVXVec(*II);
   104:     // Count the number of instructions.
   105:     InstCnt++;
   106:   }
   107: 
   108:   LLVM_DEBUG({
   109:     dbgs() << "Bundle Count : " << BndlCnt << "\n";
   110:     dbgs() << "Instruction Count : " << InstCnt << "\n";
   111:   });
   112: 
   113:   unsigned LimitUB = 0;
   114:   unsigned LimitBndl = LoopBndlAlignLimit;
   115:   // The conditions in the order of priority.
   116:   if (HST->isTinyCore()) {
   117:     LimitUB = TinyLoopAlignLimitUB;
   118:     LimitBndl = TinyLoopBndlAlignLimit;
   119:   } else if (isVec)
   120:     LimitUB = HVXLoopAlignLimitUB;
   121:   else if (AboveThres)
   122:     LimitUB = LoopAlignLimitUB;
   123: 
   124:   // if the upper bound is not set to a value, implies we didn't meet
   125:   // the criteria.
   126:   if (LimitUB == 0)
   127:     return false;
   128: 
   129:   return InstCnt >= LoopAlignLimitLB && InstCnt <= LimitUB &&
   130:          BndlCnt <= LimitBndl;
   131: }
   132: 
   133: bool HexagonLoopAlign::isSingleLoop(MachineBasicBlock &MBB) {
   134:   int Succs = MBB.succ_size();
   135:   return (MBB.isSuccessor(&MBB) && (Succs == 2));
   136: }
   137: 
   138: bool HexagonLoopAlign::attemptToBalignSmallLoop(MachineFunction &MF,
   139:                                                 MachineBasicBlock &MBB) {
   140:   if (!isSingleLoop(MBB))
   141:     return false;
   142: 
   143:   const MachineBranchProbabilityInfo *MBPI =
   144:       &getAnalysis<MachineBranchProbabilityInfoWrapperPass>().getMBPI();
   145:   const MachineBlockFrequencyInfo *MBFI =
   146:       &getAnalysis<MachineBlockFrequencyInfoWrapperPass>().getMBFI();
   147: 
   148:   // Compute frequency of back edge,
   149:   BlockFrequency BlockFreq = MBFI->getBlockFreq(&MBB);
   150:   BranchProbability BrProb = MBPI->getEdgeProbability(&MBB, &MBB);
```
- EN: It declares or implements routines such as isHVXVec, HexagonLoopAlign::isSingleLoop, succ_size, HexagonLoopAlign::attemptToBalignSmallLoop, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLoopAlign, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isHVXVec, HexagonLoopAlign::isSingleLoop, succ_size, HexagonLoopAlign::attemptToBalignSmallLoop, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLoopAlign，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   BlockFrequency EdgeFreq = BlockFreq * BrProb;
   152:   LLVM_DEBUG({
   153:     dbgs() << "Loop Align Pass:\n";
   154:     dbgs() << "\tedge with freq(" << EdgeFreq.getFrequency() << ")\n";
   155:   });
   156: 
   157:   bool AboveThres = EdgeFreq.getFrequency() > LoopEdgeThreshold;
   158:   if (shouldBalignLoop(MBB, AboveThres)) {
   159:     // We found a loop, change its alignment to be 32 (5).
   160:     MBB.setAlignment(llvm::Align(1 << 5));
   161:     return true;
   162:   }
   163:   return false;
   164: }
   165: 
   166: // Inspect each basic block, and if its a single BB loop, see if it
   167: // meets the criteria for increasing alignment to 32.
   168: 
   169: bool HexagonLoopAlign::runOnMachineFunction(MachineFunction &MF) {
   170: 
   171:   HST = &MF.getSubtarget<HexagonSubtarget>();
   172:   HII = HST->getInstrInfo();
   173:   HTM = &MF.getTarget();
   174: 
   175:   if (skipFunction(MF.getFunction()))
   176:     return false;
   177:   if (DisableLoopAlign)
   178:     return false;
   179: 
   180:   // This optimization is performed at
   181:   // i) -O2 and above, and  when the loop has a HVX instruction.
   182:   // ii) -O3
   183:   if (HST->useHVXOps()) {
   184:     if (HTM->getOptLevel() < CodeGenOptLevel::Default)
   185:       return false;
   186:   } else {
   187:     if (HTM->getOptLevel() < CodeGenOptLevel::Aggressive)
   188:       return false;
   189:   }
   190: 
   191:   bool Changed = false;
   192:   for (MachineFunction::iterator MBBi = MF.begin(), MBBe = MF.end();
   193:        MBBi != MBBe; ++MBBi) {
   194:     MachineBasicBlock &MBB = *MBBi;
   195:     Changed |= attemptToBalignSmallLoop(MF, MBB);
   196:   }
   197:   return Changed;
   198: }
   199: 
   200: } // namespace
```
- EN: It declares or implements routines such as setAlignment, HexagonLoopAlign::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getInstrInfo, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonLoopAlign, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setAlignment, HexagonLoopAlign::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getInstrInfo, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonLoopAlign, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 201-209 / 第 201-209 行

```cpp
   201: 
   202: INITIALIZE_PASS(HexagonLoopAlign, "hexagon-loop-align",
   203:                 "Hexagon LoopAlign pass", false, false)
   204: 
   205: //===----------------------------------------------------------------------===//
   206: //                         Public Constructor Functions
   207: //===----------------------------------------------------------------------===//
   208: 
   209: FunctionPass *llvm::createHexagonLoopAlign() { return new HexagonLoopAlign(); }
```
- EN: It declares or implements routines such as INITIALIZE_PASS, HexagonLoopAlign, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonLoopAlign, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 INITIALIZE_PASS, HexagonLoopAlign 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonLoopAlign，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- loop-aware code generation / 循环相关代码生成
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonTargetMachine.h, llvm/CodeGen/MachineBlockFrequencyInfo.h, llvm/CodeGen/MachineBranchProbabilityInfo.h, llvm/Support/Debug.h`
- Hexagon symbols / Hexagon 符号: `HexagonLoopAlign, HexagonTargetMachine, HexagonSubtarget, HexagonInstrInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
