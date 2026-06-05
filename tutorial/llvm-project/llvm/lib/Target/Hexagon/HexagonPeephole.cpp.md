# HexagonPeephole.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonPeephole.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon peephole optimizations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===-- HexagonPeephole.cpp - Hexagon Peephole Optimizations --------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: // This peephole pass optimizes in the following cases.
     8: // 1. Optimizes redundant sign extends for the following case
     9: //    Transform the following pattern
    10: //    %170 = SXTW %166
    11: //    ...
    12: //    %176 = COPY %170:isub_lo
    13: //
    14: //    Into
    15: //    %176 = COPY %166
    16: //
    17: //  2. Optimizes redundant negation of predicates.
    18: //     %15 = CMPGTrr %6, %2
    19: //     ...
    20: //     %16 = NOT_p killed %15
    21: //     ...
    22: //     JMP_c killed %16, <%bb.1>, implicit dead %pc
    23: //
    24: //     Into
    25: //     %15 = CMPGTrr %6, %2;
    26: //     ...
    27: //     JMP_cNot killed %15, <%bb.1>, implicit dead %pc;
    28: //
    29: // Note: The peephole pass makes the instrucstions like
    30: // %170 = SXTW %166 or %16 = NOT_p killed %15
    31: // redundant and relies on some form of dead removal instructions, like
    32: // DCE or DIE to actually eliminate them.
    33: 
    34: //===----------------------------------------------------------------------===//
    35: 
    36: #include "Hexagon.h"
    37: #include "HexagonTargetMachine.h"
    38: #include "llvm/ADT/DenseMap.h"
    39: #include "llvm/ADT/Statistic.h"
    40: #include "llvm/CodeGen/MachineFunction.h"
    41: #include "llvm/CodeGen/MachineFunctionPass.h"
    42: #include "llvm/CodeGen/MachineInstrBuilder.h"
    43: #include "llvm/CodeGen/MachineRegisterInfo.h"
    44: #include "llvm/CodeGen/Passes.h"
    45: #include "llvm/CodeGen/TargetInstrInfo.h"
    46: #include "llvm/CodeGen/TargetRegisterInfo.h"
    47: #include "llvm/Pass.h"
    48: #include "llvm/Support/CommandLine.h"
    49: #include "llvm/Target/TargetMachine.h"
    50: 
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as Hexagon.h, HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/ADT/Statistic.h, ... (14 total), establishing the LLVM/Hexagon APIs used below. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPeephole, HexagonTargetMachine, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 Hexagon.h, HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/ADT/Statistic.h, ... (14 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPeephole, HexagonTargetMachine，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```cpp
    51: using namespace llvm;
    52: 
    53: #define DEBUG_TYPE "hexagon-peephole"
    54: 
    55: static cl::opt<bool>
    56:     DisableHexagonPeephole("disable-hexagon-peephole", cl::Hidden,
    57:                            cl::desc("Disable Peephole Optimization"));
    58: 
    59: static cl::opt<bool> DisablePNotP("disable-hexagon-pnotp", cl::Hidden,
    60:                                   cl::desc("Disable Optimization of PNotP"));
    61: 
    62: static cl::opt<bool>
    63:     DisableOptSZExt("disable-hexagon-optszext", cl::Hidden, cl::init(true),
    64:                     cl::desc("Disable Optimization of Sign/Zero Extends"));
    65: 
    66: static cl::opt<bool>
    67:     DisableOptExtTo64("disable-hexagon-opt-ext-to-64", cl::Hidden,
    68:                       cl::init(true),
    69:                       cl::desc("Disable Optimization of extensions to i64."));
    70: 
    71: namespace {
    72:   struct HexagonPeephole : public MachineFunctionPass {
    73:     const HexagonInstrInfo    *QII;
    74:     const HexagonRegisterInfo *QRI;
    75:     const MachineRegisterInfo *MRI;
    76: 
    77:   public:
    78:     static char ID;
    79:     HexagonPeephole() : MachineFunctionPass(ID) {}
    80: 
    81:     bool runOnMachineFunction(MachineFunction &MF) override;
    82: 
    83:     StringRef getPassName() const override {
    84:       return "Hexagon optimize redundant zero and size extends";
    85:     }
    86: 
    87:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    88:       MachineFunctionPass::getAnalysisUsage(AU);
    89:     }
    90:   };
    91: }
    92: 
    93: char HexagonPeephole::ID = 0;
    94: 
    95: INITIALIZE_PASS(HexagonPeephole, "hexagon-peephole", "Hexagon Peephole",
    96:                 false, false)
    97: 
    98: bool HexagonPeephole::runOnMachineFunction(MachineFunction &MF) {
    99:   if (skipFunction(MF.getFunction()))
   100:     return false;
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonPeephole, which carry the state or API of this component. It declares or implements routines such as DisableHexagonPeephole, DisablePNotP, DisableOptSZExt, DisableOptExtTo64, ... (10 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonPeephole 等类型，用来承载该组件的状态或接口。 这里声明或实现了 DisableHexagonPeephole, DisablePNotP, DisableOptSZExt, DisableOptExtTo64, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102:   QII = static_cast<const HexagonInstrInfo *>(MF.getSubtarget().getInstrInfo());
   103:   QRI = MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   104:   MRI = &MF.getRegInfo();
   105: 
   106:   DenseMap<unsigned, unsigned> PeepholeMap;
   107:   DenseMap<unsigned, std::pair<unsigned, unsigned> > PeepholeDoubleRegsMap;
   108: 
   109:   if (DisableHexagonPeephole) return false;
   110: 
   111:   // Loop over all of the basic blocks.
   112:   for (MachineBasicBlock &MBB : MF) {
   113:     PeepholeMap.clear();
   114:     PeepholeDoubleRegsMap.clear();
   115: 
   116:     // Traverse the basic block.
   117:     for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
   118:       // Look for sign extends:
   119:       // %170 = SXTW %166
   120:       if (!DisableOptSZExt && MI.getOpcode() == Hexagon::A2_sxtw) {
   121:         assert(MI.getNumOperands() == 2);
   122:         MachineOperand &Dst = MI.getOperand(0);
   123:         MachineOperand &Src = MI.getOperand(1);
   124:         Register DstReg = Dst.getReg();
   125:         Register SrcReg = Src.getReg();
   126:         // Just handle virtual registers.
   127:         if (DstReg.isVirtual() && SrcReg.isVirtual()) {
   128:           // Map the following:
   129:           // %170 = SXTW %166
   130:           // PeepholeMap[170] = %166
   131:           PeepholeMap[DstReg] = SrcReg;
   132:         }
   133:       }
   134: 
   135:       // Look for  %170 = COMBINE_ir_V4 (0, %169)
   136:       // %170:DoublRegs, %169:IntRegs
   137:       if (!DisableOptExtTo64 && MI.getOpcode() == Hexagon::A4_combineir) {
   138:         assert(MI.getNumOperands() == 3);
   139:         MachineOperand &Dst = MI.getOperand(0);
   140:         MachineOperand &Src1 = MI.getOperand(1);
   141:         MachineOperand &Src2 = MI.getOperand(2);
   142:         if (Src1.getImm() != 0)
   143:           continue;
   144:         Register DstReg = Dst.getReg();
   145:         Register SrcReg = Src2.getReg();
   146:         PeepholeMap[DstReg] = SrcReg;
   147:       }
   148: 
   149:       // Look for this sequence below
   150:       // %DoubleReg1 = LSRd_ri %DoubleReg0, 32
```
- EN: It declares or implements routines such as getSubtarget, getSubtarget<HexagonSubtarget>, getRegInfo, clear, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSubtarget, getSubtarget<HexagonSubtarget>, getRegInfo, clear, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:       // %IntReg = COPY %DoubleReg1:isub_lo.
   152:       // and convert into
   153:       // %IntReg = COPY %DoubleReg0:isub_hi.
   154:       if (MI.getOpcode() == Hexagon::S2_lsr_i_p) {
   155:         assert(MI.getNumOperands() == 3);
   156:         MachineOperand &Dst = MI.getOperand(0);
   157:         MachineOperand &Src1 = MI.getOperand(1);
   158:         MachineOperand &Src2 = MI.getOperand(2);
   159:         if (Src2.getImm() != 32)
   160:           continue;
   161:         Register DstReg = Dst.getReg();
   162:         Register SrcReg = Src1.getReg();
   163:         PeepholeDoubleRegsMap[DstReg] =
   164:           std::make_pair(*&SrcReg, Hexagon::isub_hi);
   165:       }
   166: 
   167:       // Look for P=NOT(P).
   168:       if (!DisablePNotP && MI.getOpcode() == Hexagon::C2_not) {
   169:         assert(MI.getNumOperands() == 2);
   170:         MachineOperand &Dst = MI.getOperand(0);
   171:         MachineOperand &Src = MI.getOperand(1);
   172:         Register DstReg = Dst.getReg();
   173:         Register SrcReg = Src.getReg();
   174:         // Just handle virtual registers.
   175:         if (DstReg.isVirtual() && SrcReg.isVirtual()) {
   176:           // Map the following:
   177:           // %170 = NOT_xx %166
   178:           // PeepholeMap[170] = %166
   179:           PeepholeMap[DstReg] = SrcReg;
   180:         }
   181:       }
   182: 
   183:       // Look for copy:
   184:       // %176 = COPY %170:isub_lo
   185:       if (!DisableOptSZExt && MI.isCopy()) {
   186:         assert(MI.getNumOperands() == 2);
   187:         MachineOperand &Dst = MI.getOperand(0);
   188:         MachineOperand &Src = MI.getOperand(1);
   189: 
   190:         // Make sure we are copying the lower 32 bits.
   191:         if (Src.getSubReg() != Hexagon::isub_lo)
   192:           continue;
   193: 
   194:         Register DstReg = Dst.getReg();
   195:         Register SrcReg = Src.getReg();
   196:         if (DstReg.isVirtual() && SrcReg.isVirtual()) {
   197:           // Try to find in the map.
   198:           if (unsigned PeepholeSrc = PeepholeMap.lookup(SrcReg)) {
   199:             // Change the 1st operand.
   200:             MI.removeOperand(1);
```
- EN: It declares or implements routines such as assert, getOperand, getReg, std::make_pair, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, getOperand, getReg, std::make_pair, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-250 / 第 201-250 行

```cpp
   201:             MI.addOperand(MachineOperand::CreateReg(PeepholeSrc, false));
   202:           } else  {
   203:             DenseMap<unsigned, std::pair<unsigned, unsigned> >::iterator DI =
   204:               PeepholeDoubleRegsMap.find(SrcReg);
   205:             if (DI != PeepholeDoubleRegsMap.end()) {
   206:               std::pair<unsigned,unsigned> PeepholeSrc = DI->second;
   207:               MI.removeOperand(1);
   208:               MI.addOperand(MachineOperand::CreateReg(
   209:                   PeepholeSrc.first, false /*isDef*/, false /*isImp*/,
   210:                   false /*isKill*/, false /*isDead*/, false /*isUndef*/,
   211:                   false /*isEarlyClobber*/, PeepholeSrc.second));
   212:             }
   213:           }
   214:         }
   215:       }
   216: 
   217:       // Look for Predicated instructions.
   218:       if (!DisablePNotP) {
   219:         bool Done = false;
   220:         if (QII->isPredicated(MI)) {
   221:           MachineOperand &Op0 = MI.getOperand(0);
   222:           Register Reg0 = Op0.getReg();
   223:           const TargetRegisterClass *RC0 = MRI->getRegClass(Reg0);
   224:           if (RC0->getID() == Hexagon::PredRegsRegClassID) {
   225:             // Handle instructions that have a prediate register in op0
   226:             // (most cases of predicable instructions).
   227:             if (Reg0.isVirtual()) {
   228:               // Try to find in the map.
   229:               if (unsigned PeepholeSrc = PeepholeMap.lookup(Reg0)) {
   230:                 // Change the 1st operand and, flip the opcode.
   231:                 MI.getOperand(0).setReg(PeepholeSrc);
   232:                 MRI->clearKillFlags(PeepholeSrc);
   233:                 int NewOp = QII->getInvertedPredicatedOpcode(MI.getOpcode());
   234:                 MI.setDesc(QII->get(NewOp));
   235:                 Done = true;
   236:               }
   237:             }
   238:           }
   239:         }
   240: 
   241:         if (!Done) {
   242:           // Handle special instructions.
   243:           unsigned Op = MI.getOpcode();
   244:           unsigned NewOp = 0;
   245:           unsigned PR = 1, S1 = 2, S2 = 3;   // Operand indices.
   246: 
   247:           switch (Op) {
   248:             case Hexagon::C2_mux:
   249:             case Hexagon::C2_muxii:
   250:               NewOp = Op;
```
- EN: It declares or implements routines such as addOperand, find, removeOperand, getOperand, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 addOperand, find, removeOperand, getOperand, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 251-282 / 第 251-282 行

```cpp
   251:               break;
   252:             case Hexagon::C2_muxri:
   253:               NewOp = Hexagon::C2_muxir;
   254:               break;
   255:             case Hexagon::C2_muxir:
   256:               NewOp = Hexagon::C2_muxri;
   257:               break;
   258:           }
   259:           if (NewOp) {
   260:             Register PSrc = MI.getOperand(PR).getReg();
   261:             if (unsigned POrig = PeepholeMap.lookup(PSrc)) {
   262:               BuildMI(MBB, MI.getIterator(), MI.getDebugLoc(), QII->get(NewOp),
   263:                       MI.getOperand(0).getReg())
   264:                   .addReg(POrig)
   265:                   .add(MI.getOperand(S2))
   266:                   .add(MI.getOperand(S1));
   267:               MRI->clearKillFlags(POrig);
   268:               MI.eraseFromParent();
   269:             }
   270:           } // if (NewOp)
   271:         } // if (!Done)
   272: 
   273:       } // if (!DisablePNotP)
   274: 
   275:     } // Instruction
   276:   } // Basic Block
   277:   return true;
   278: }
   279: 
   280: FunctionPass *llvm::createHexagonPeephole() {
   281:   return new HexagonPeephole();
   282: }
```
- EN: It declares or implements routines such as getOperand, BuildMI, clearKillFlags, eraseFromParent, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPeephole, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, BuildMI, clearKillFlags, eraseFromParent, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPeephole，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonTargetMachine.h, llvm/ADT/DenseMap.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h, ... (14 total)`
- Hexagon symbols / Hexagon 符号: `HexagonPeephole, HexagonTargetMachine, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
