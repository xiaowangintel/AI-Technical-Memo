# HexagonGenMemAbsolute.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGenMemAbsolute.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon creation of absolute-memory forms.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===--- HexagonGenMemAbsolute.cpp - Generate Load/Store Set Absolute ---===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // This pass traverses through all the basic blocks in a function and converts
    10: // an indexed load/store with offset "0" to a absolute-set load/store
    11: // instruction as long as the use of the register in the new instruction
    12: // dominates the rest of the uses and there are more than 2 uses.
    13: 
    14: #include "HexagonTargetMachine.h"
    15: #include "llvm/ADT/Statistic.h"
    16: #include "llvm/CodeGen/MachineDominators.h"
    17: #include "llvm/CodeGen/MachineFunctionPass.h"
    18: #include "llvm/CodeGen/MachineInstrBuilder.h"
    19: #include "llvm/CodeGen/MachineRegisterInfo.h"
    20: #include "llvm/CodeGen/Passes.h"
    21: #include "llvm/CodeGen/TargetInstrInfo.h"
    22: #include "llvm/Support/Debug.h"
    23: #include "llvm/Support/raw_ostream.h"
    24: 
    25: #define DEBUG_TYPE "hexagon-abs"
    26: 
    27: using namespace llvm;
    28: 
    29: STATISTIC(HexagonNumLoadAbsConversions,
    30:           "Number of Load instructions converted to absolute-set form");
    31: STATISTIC(HexagonNumStoreAbsConversions,
    32:           "Number of Store instructions converted to absolute-set form");
    33: 
    34: namespace {
    35: 
    36: class HexagonGenMemAbsolute : public MachineFunctionPass {
    37:   const HexagonInstrInfo *TII = nullptr;
    38:   MachineRegisterInfo *MRI = nullptr;
    39:   const TargetRegisterInfo *TRI = nullptr;
    40: 
    41: public:
    42:   static char ID;
    43:   HexagonGenMemAbsolute() : MachineFunctionPass(ID) {}
    44: 
    45:   StringRef getPassName() const override {
    46:     return "Hexagon Generate Load/Store Set Absolute Address Instruction";
    47:   }
    48: 
    49:   void getAnalysisUsage(AnalysisUsage &AU) const override {
    50:     MachineFunctionPass::getAnalysisUsage(AU);
```
- EN: It imports headers such as HexagonTargetMachine.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunctionPass.h, ... (10 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonGenMemAbsolute, which carry the state or API of this component.
- CN: 这里引入了 HexagonTargetMachine.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunctionPass.h, ... (10 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonGenMemAbsolute 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:     AU.addRequired<MachineDominatorTreeWrapperPass>();
    52:     AU.addPreserved<MachineDominatorTreeWrapperPass>();
    53:   }
    54: 
    55:   bool runOnMachineFunction(MachineFunction &Fn) override;
    56: 
    57: private:
    58:   static bool isValidIndexedLoad(int &Opcode, int &NewOpcode);
    59:   static bool isValidIndexedStore(int &Opcode, int &NewOpcode);
    60: };
    61: } // namespace
    62: 
    63: char HexagonGenMemAbsolute::ID = 0;
    64: 
    65: INITIALIZE_PASS(HexagonGenMemAbsolute, "hexagon-gen-load-absolute",
    66:                 "Hexagon Generate Load/Store Set Absolute Address Instruction",
    67:                 false, false)
    68: 
    69: bool HexagonGenMemAbsolute::runOnMachineFunction(MachineFunction &Fn) {
    70:   if (skipFunction(Fn.getFunction()))
    71:     return false;
    72: 
    73:   TII = Fn.getSubtarget<HexagonSubtarget>().getInstrInfo();
    74:   MRI = &Fn.getRegInfo();
    75:   TRI = Fn.getRegInfo().getTargetRegisterInfo();
    76: 
    77:   MachineDominatorTree &MDT =
    78:       getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
    79: 
    80:   // Loop over all of the basic blocks
    81:   for (MachineBasicBlock &MBB : Fn) {
    82:     // Traverse the basic block
    83:     for (MachineBasicBlock::iterator MII = MBB.begin(); MII != MBB.end();
    84:          ++MII) {
    85:       MachineInstr *MI = &*MII;
    86:       int Opc = MI->getOpcode();
    87:       if (Opc != Hexagon::CONST32 && Opc != Hexagon::A2_tfrsi)
    88:         continue;
    89: 
    90:       const MachineOperand &MO = MI->getOperand(0);
    91:       if (!MO.isReg() || !MO.isDef())
    92:         continue;
    93: 
    94:       unsigned DstReg = MO.getReg();
    95:       if (MRI->use_nodbg_empty(DstReg))
    96:         continue;
    97: 
    98:       typedef MachineRegisterInfo::use_nodbg_iterator use_iterator;
    99:       use_iterator NextUseMI = MRI->use_nodbg_begin(DstReg);
   100: 
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as addRequired<MachineDominatorTreeWrapperPass>, addPreserved<MachineDominatorTreeWrapperPass>, runOnMachineFunction, isValidIndexedLoad, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenMemAbsolute, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 addRequired<MachineDominatorTreeWrapperPass>, addPreserved<MachineDominatorTreeWrapperPass>, runOnMachineFunction, isValidIndexedLoad, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenMemAbsolute, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:       MachineInstr *NextMI = NextUseMI->getParent();
   102:       int NextOpc = NextMI->getOpcode();
   103:       int NewOpc;
   104:       bool IsLoad = isValidIndexedLoad(NextOpc, NewOpc);
   105: 
   106:       if (!IsLoad && !isValidIndexedStore(NextOpc, NewOpc))
   107:         continue;
   108: 
   109:       // Base and Offset positions for load and store instructions
   110:       // Load R(dest), R(base), Imm -> R(dest) = mem(R(base) + Imm)
   111:       // Store R(base), Imm, R (src) -> mem(R(base) + Imm) = R(src)
   112:       unsigned BaseRegPos, ImmPos, RegPos;
   113:       if (!TII->getBaseAndOffsetPosition(*NextMI, BaseRegPos, ImmPos))
   114:         continue;
   115:       RegPos = IsLoad ? 0 : 2;
   116: 
   117:       bool IsGlobal = MI->getOperand(1).isGlobal();
   118:       if (!MI->getOperand(1).isImm() && !IsGlobal)
   119:         continue;
   120: 
   121:       const MachineOperand *BaseOp = nullptr;
   122:       int64_t Offset;
   123:       bool Scalable;
   124:       TII->getMemOperandWithOffset(*NextMI, BaseOp, Offset, Scalable, TRI);
   125: 
   126:       // Ensure BaseOp is non-null and register type.
   127:       if (!BaseOp || !BaseOp->isReg())
   128:         continue;
   129: 
   130:       if (Scalable)
   131:         continue;
   132: 
   133:       unsigned BaseReg = BaseOp->getReg();
   134:       if ((DstReg != BaseReg) || (Offset != 0))
   135:         continue;
   136: 
   137:       const MachineOperand &MO0 = NextMI->getOperand(RegPos);
   138: 
   139:       if (!MO0.isReg())
   140:         continue;
   141: 
   142:       unsigned LoadStoreReg = MO0.getReg();
   143: 
   144:       // Store: Bail out if the src and base are same (def and use on same
   145:       // register).
   146:       if (LoadStoreReg == BaseReg)
   147:         continue;
   148: 
   149:       // Insert the absolute-set instruction "I" only if the use of the
   150:       // BaseReg in "I" dominates the rest of the uses of BaseReg and if
```
- EN: It declares or implements routines such as getParent, getOpcode, isValidIndexedLoad, getOperand, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getParent, getOpcode, isValidIndexedLoad, getOperand, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:       // there are more than 2 uses of this BaseReg.
   152:       bool Dominates = true;
   153:       unsigned Counter = 0;
   154:       for (use_iterator I = NextUseMI, E = MRI->use_nodbg_end(); I != E; ++I) {
   155:         Counter++;
   156:         if (!MDT.dominates(NextMI, I->getParent()))
   157:           Dominates = false;
   158:       }
   159: 
   160:       if ((!Dominates) || (Counter < 3))
   161:         continue;
   162: 
   163:       // If we reach here, we have met all the conditions required for the
   164:       // replacement of the absolute instruction.
   165:       LLVM_DEBUG({
   166:         dbgs() << "Found a pair of instructions for absolute-set "
   167:                << (IsLoad ? "load" : "store") << "\n";
   168:         dbgs() << *MI;
   169:         dbgs() << *NextMI;
   170:       });
   171:       MachineBasicBlock *ParentBlock = NextMI->getParent();
   172:       MachineInstrBuilder MIB;
   173:       if (IsLoad) { // Insert absolute-set load instruction
   174:         ++HexagonNumLoadAbsConversions;
   175:         MIB = BuildMI(*ParentBlock, NextMI, NextMI->getDebugLoc(),
   176:                       TII->get(NewOpc), LoadStoreReg)
   177:                   .addReg(DstReg, RegState::Define);
   178:       } else { // Insert absolute-set store instruction
   179:         ++HexagonNumStoreAbsConversions;
   180:         MIB = BuildMI(*ParentBlock, NextMI, NextMI->getDebugLoc(),
   181:                       TII->get(NewOpc), DstReg);
   182:       }
   183: 
   184:       MachineOperand ImmOperand = MI->getOperand(1);
   185:       if (IsGlobal)
   186:         MIB.addGlobalAddress(ImmOperand.getGlobal(), ImmOperand.getOffset(),
   187:                              ImmOperand.getTargetFlags());
   188:       else
   189:         MIB.addImm(ImmOperand.getImm());
   190: 
   191:       if (IsLoad)
   192:         MIB->getOperand(0).setSubReg(MO0.getSubReg());
   193:       else
   194:         MIB.addReg(LoadStoreReg, {}, MO0.getSubReg());
   195: 
   196:       LLVM_DEBUG(dbgs() << "Replaced with " << *MIB << "\n");
   197:       // Erase the instructions that got replaced.
   198:       MII = MBB.erase(MI);
   199:       --MII;
   200:       NextMI->getParent()->erase(NextMI);
```
- EN: It declares or implements routines such as getParent, BuildMI, getOperand, addImm, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonNumLoadAbsConversions, HexagonNumStoreAbsConversions, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getParent, BuildMI, getOperand, addImm, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonNumLoadAbsConversions, HexagonNumStoreAbsConversions，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:     }
   202:   }
   203: 
   204:   return true;
   205: }
   206: 
   207: bool HexagonGenMemAbsolute::isValidIndexedLoad(int &Opc, int &NewOpc) {
   208: 
   209:   bool Result = true;
   210:   switch (Opc) {
   211:   case Hexagon::L2_loadrb_io:
   212:     NewOpc = Hexagon::L4_loadrb_ap;
   213:     break;
   214:   case Hexagon::L2_loadrh_io:
   215:     NewOpc = Hexagon::L4_loadrh_ap;
   216:     break;
   217:   case Hexagon::L2_loadri_io:
   218:     NewOpc = Hexagon::L4_loadri_ap;
   219:     break;
   220:   case Hexagon::L2_loadrd_io:
   221:     NewOpc = Hexagon::L4_loadrd_ap;
   222:     break;
   223:   case Hexagon::L2_loadruh_io:
   224:     NewOpc = Hexagon::L4_loadruh_ap;
   225:     break;
   226:   case Hexagon::L2_loadrub_io:
   227:     NewOpc = Hexagon::L4_loadrub_ap;
   228:     break;
   229:   default:
   230:     Result = false;
   231:   }
   232: 
   233:   return Result;
   234: }
   235: 
   236: bool HexagonGenMemAbsolute::isValidIndexedStore(int &Opc, int &NewOpc) {
   237: 
   238:   bool Result = true;
   239:   switch (Opc) {
   240:   case Hexagon::S2_storerd_io:
   241:     NewOpc = Hexagon::S4_storerd_ap;
   242:     break;
   243:   case Hexagon::S2_storeri_io:
   244:     NewOpc = Hexagon::S4_storeri_ap;
   245:     break;
   246:   case Hexagon::S2_storerh_io:
   247:     NewOpc = Hexagon::S4_storerh_ap;
   248:     break;
   249:   case Hexagon::S2_storerb_io:
   250:     NewOpc = Hexagon::S4_storerb_ap;
```
- EN: It declares or implements routines such as HexagonGenMemAbsolute::isValidIndexedLoad, HexagonGenMemAbsolute::isValidIndexedStore, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenMemAbsolute, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGenMemAbsolute::isValidIndexedLoad, HexagonGenMemAbsolute::isValidIndexedStore 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenMemAbsolute，说明了它与同级后端组件的连接关系。

### Lines 251-265 / 第 251-265 行

```cpp
   251:     break;
   252:   default:
   253:     Result = false;
   254:   }
   255: 
   256:   return Result;
   257: }
   258: 
   259: //===----------------------------------------------------------------------===//
   260: //                         Public Constructor Functions
   261: //===----------------------------------------------------------------------===//
   262: 
   263: FunctionPass *llvm::createHexagonGenMemAbsolute() {
   264:   return new HexagonGenMemAbsolute();
   265: }
```
- EN: It declares or implements routines such as llvm::createHexagonGenMemAbsolute, HexagonGenMemAbsolute, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonGenMemAbsolute, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonGenMemAbsolute, HexagonGenMemAbsolute 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonGenMemAbsolute，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- loop-aware code generation / 循环相关代码生成
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonTargetMachine.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h, llvm/Support/Debug.h, llvm/Support/raw_ostream.h`
- Hexagon symbols / Hexagon 符号: `HexagonGenMemAbsolute, HexagonTargetMachine, HexagonNumLoadAbsConversions, HexagonNumStoreAbsConversions, HexagonInstrInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
