# HexagonVectorPrint.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVectorPrint.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon vector-oriented debug printing or diagnostics.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonVectorPrint.cpp - Generate vector printing instructions -----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This pass adds the capability to generate pseudo vector/predicate register
    10: // printing instructions. These pseudo instructions should be used with the
    11: // simulator, NEVER on hardware.
    12: //
    13: //===----------------------------------------------------------------------===//
    14: 
    15: #include "Hexagon.h"
    16: #include "HexagonInstrInfo.h"
    17: #include "HexagonSubtarget.h"
    18: #include "llvm/ADT/StringRef.h"
    19: #include "llvm/CodeGen/MachineBasicBlock.h"
    20: #include "llvm/CodeGen/MachineFunction.h"
    21: #include "llvm/CodeGen/MachineFunctionPass.h"
    22: #include "llvm/CodeGen/MachineInstr.h"
    23: #include "llvm/CodeGen/MachineInstrBuilder.h"
    24: #include "llvm/CodeGen/MachineOperand.h"
    25: #include "llvm/CodeGen/TargetOpcodes.h"
    26: #include "llvm/IR/DebugLoc.h"
    27: #include "llvm/IR/InlineAsm.h"
    28: #include "llvm/Pass.h"
    29: #include "llvm/Support/CommandLine.h"
    30: #include "llvm/Support/Debug.h"
    31: #include "llvm/Support/ErrorHandling.h"
    32: #include "llvm/Support/raw_ostream.h"
    33: #include <string>
    34: #include <vector>
    35: 
    36: using namespace llvm;
    37: 
    38: #define DEBUG_TYPE "hexagon-vector-print"
    39: 
    40: static cl::opt<bool>
    41:     TraceHexVectorStoresOnly("trace-hex-vector-stores-only", cl::Hidden,
    42:                              cl::desc("Enables tracing of vector stores"));
    43: 
    44: namespace {
    45: 
    46: class HexagonVectorPrint : public MachineFunctionPass {
    47:   const HexagonSubtarget *QST = nullptr;
    48:   const HexagonInstrInfo *QII = nullptr;
    49:   const HexagonRegisterInfo *QRI = nullptr;
    50: 
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/StringRef.h, ... (20 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonVectorPrint, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/StringRef.h, ... (20 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonVectorPrint 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51: public:
    52:   static char ID;
    53: 
    54:   HexagonVectorPrint() : MachineFunctionPass(ID) {}
    55: 
    56:   StringRef getPassName() const override { return "Hexagon VectorPrint pass"; }
    57: 
    58:   bool runOnMachineFunction(MachineFunction &Fn) override;
    59: };
    60: 
    61: } // end anonymous namespace
    62: 
    63: char HexagonVectorPrint::ID = 0;
    64: 
    65: static bool isVecReg(unsigned Reg) {
    66:   return (Reg >= Hexagon::V0 && Reg <= Hexagon::V31) ||
    67:          (Reg >= Hexagon::W0 && Reg <= Hexagon::W15) ||
    68:          (Reg >= Hexagon::WR0 && Reg <= Hexagon::WR15) ||
    69:          (Reg >= Hexagon::Q0 && Reg <= Hexagon::Q3);
    70: }
    71: 
    72: static std::string getStringReg(unsigned R) {
    73:   if (R >= Hexagon::V0 && R <= Hexagon::V31) {
    74:     static const char* S[] = { "20", "21", "22", "23", "24", "25", "26", "27",
    75:                         "28", "29", "2a", "2b", "2c", "2d", "2e", "2f",
    76:                         "30", "31", "32", "33", "34", "35", "36", "37",
    77:                         "38", "39", "3a", "3b", "3c", "3d", "3e", "3f"};
    78:     return S[R-Hexagon::V0];
    79:   }
    80:   if (R >= Hexagon::Q0 && R <= Hexagon::Q3) {
    81:     static const char* S[] = { "00", "01", "02", "03"};
    82:     return S[R-Hexagon::Q0];
    83: 
    84:   }
    85:   llvm_unreachable("valid vreg");
    86: }
    87: 
    88: static void addAsmInstr(MachineBasicBlock *MBB, unsigned Reg,
    89:                         MachineBasicBlock::instr_iterator I,
    90:                         const DebugLoc &DL, const HexagonInstrInfo *QII,
    91:                         MachineFunction &Fn) {
    92:   std::string VDescStr = ".long 0x1dffe0" + getStringReg(Reg);
    93:   const char *cstr = Fn.createExternalSymbolName(VDescStr);
    94:   unsigned ExtraInfo = InlineAsm::Extra_HasSideEffects;
    95:   BuildMI(*MBB, I, DL, QII->get(TargetOpcode::INLINEASM))
    96:     .addExternalSymbol(cstr)
    97:     .addImm(ExtraInfo);
    98: }
    99: 
   100: static bool getInstrVecReg(const MachineInstr &MI, unsigned &Reg) {
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonVectorPrint, getPassName, runOnMachineFunction, isVecReg, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorPrint, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonVectorPrint, getPassName, runOnMachineFunction, isVecReg, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorPrint, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   if (MI.getNumOperands() < 1) return false;
   102:   // Vec load or compute.
   103:   if (MI.getOperand(0).isReg() && MI.getOperand(0).isDef()) {
   104:     Reg = MI.getOperand(0).getReg();
   105:     if (isVecReg(Reg))
   106:       return !TraceHexVectorStoresOnly;
   107:   }
   108:   // Vec store.
   109:   if (MI.mayStore() && MI.getNumOperands() >= 3 && MI.getOperand(2).isReg()) {
   110:     Reg = MI.getOperand(2).getReg();
   111:     if (isVecReg(Reg))
   112:       return true;
   113:   }
   114:   // Vec store post increment.
   115:   if (MI.mayStore() && MI.getNumOperands() >= 4 && MI.getOperand(3).isReg()) {
   116:     Reg = MI.getOperand(3).getReg();
   117:     if (isVecReg(Reg))
   118:       return true;
   119:   }
   120:   return false;
   121: }
   122: 
   123: bool HexagonVectorPrint::runOnMachineFunction(MachineFunction &Fn) {
   124:   bool Changed = false;
   125:   QST = &Fn.getSubtarget<HexagonSubtarget>();
   126:   QRI = QST->getRegisterInfo();
   127:   QII = QST->getInstrInfo();
   128:   std::vector<MachineInstr *> VecPrintList;
   129:   for (auto &MBB : Fn)
   130:     for (auto &MI : MBB) {
   131:       if (MI.isBundle()) {
   132:         MachineBasicBlock::instr_iterator MII = MI.getIterator();
   133:         for (++MII; MII != MBB.instr_end() && MII->isInsideBundle(); ++MII) {
   134:           if (MII->getNumOperands() < 1)
   135:             continue;
   136:           unsigned Reg = 0;
   137:           if (getInstrVecReg(*MII, Reg)) {
   138:             VecPrintList.push_back((&*MII));
   139:             LLVM_DEBUG(dbgs() << "Found vector reg inside bundle \n";
   140:                        MII->dump());
   141:           }
   142:         }
   143:       } else {
   144:         unsigned Reg = 0;
   145:         if (getInstrVecReg(MI, Reg)) {
   146:           VecPrintList.push_back(&MI);
   147:           LLVM_DEBUG(dbgs() << "Found vector reg \n"; MI.dump());
   148:         }
   149:       }
   150:     }
```
- EN: It declares or implements routines such as getOperand, HexagonVectorPrint::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getRegisterInfo, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorPrint, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, HexagonVectorPrint::runOnMachineFunction, getSubtarget<HexagonSubtarget>, getRegisterInfo, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorPrint, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152:   Changed = !VecPrintList.empty();
   153:   if (!Changed)
   154:     return Changed;
   155: 
   156:   for (auto *I : VecPrintList) {
   157:     DebugLoc DL = I->getDebugLoc();
   158:     MachineBasicBlock *MBB = I->getParent();
   159:     LLVM_DEBUG(dbgs() << "Evaluating V MI\n"; I->dump());
   160:     unsigned Reg = 0;
   161:     if (!getInstrVecReg(*I, Reg))
   162:       llvm_unreachable("Need a vector reg");
   163:     MachineBasicBlock::instr_iterator MII = I->getIterator();
   164:     if (I->isInsideBundle()) {
   165:       LLVM_DEBUG(dbgs() << "add to end of bundle\n"; I->dump());
   166:       while (MBB->instr_end() != MII && MII->isInsideBundle())
   167:         MII++;
   168:     } else {
   169:       LLVM_DEBUG(dbgs() << "add after instruction\n"; I->dump());
   170:       MII++;
   171:     }
   172:     if (MBB->instr_end() == MII)
   173:       continue;
   174: 
   175:     if (Reg >= Hexagon::V0 && Reg <= Hexagon::V31) {
   176:       LLVM_DEBUG(dbgs() << "adding dump for V" << Reg - Hexagon::V0 << '\n');
   177:       addAsmInstr(MBB, Reg, MII, DL, QII, Fn);
   178:     } else if (Reg >= Hexagon::W0 && Reg <= Hexagon::W15) {
   179:       LLVM_DEBUG(dbgs() << "adding dump for W" << Reg - Hexagon::W0 << '\n');
   180:       addAsmInstr(MBB, Hexagon::V0 + (Reg - Hexagon::W0) * 2 + 1,
   181:                   MII, DL, QII, Fn);
   182:       addAsmInstr(MBB, Hexagon::V0 + (Reg - Hexagon::W0) * 2,
   183:                    MII, DL, QII, Fn);
   184:     } else if (Reg >= Hexagon::Q0 && Reg <= Hexagon::Q3) {
   185:       LLVM_DEBUG(dbgs() << "adding dump for Q" << Reg - Hexagon::Q0 << '\n');
   186:       addAsmInstr(MBB, Reg, MII, DL, QII, Fn);
   187:     } else
   188:       llvm_unreachable("Bad Vector reg");
   189:   }
   190:   return Changed;
   191: }
   192: 
   193: //===----------------------------------------------------------------------===//
   194: //                         Public Constructor Functions
   195: //===----------------------------------------------------------------------===//
   196: INITIALIZE_PASS(HexagonVectorPrint, "hexagon-vector-print",
   197:   "Hexagon VectorPrint pass", false, false)
   198: 
   199: FunctionPass *llvm::createHexagonVectorPrint() {
   200:   return new HexagonVectorPrint();
```
- EN: It declares or implements routines such as empty, getDebugLoc, getParent, dump, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVectorPrint, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 empty, getDebugLoc, getParent, dump, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVectorPrint，说明了它与同级后端组件的连接关系。

### Lines 201-201 / 第 201-201 行

```cpp
   201: }
```
- EN: This range contains executable implementation details for a Hexagon backend subsystem.
- CN: 这一段包含了某个 Hexagon 后端子系统的可执行实现细节。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineOperand.h, ... (20 total)`
- Hexagon symbols / Hexagon 符号: `HexagonVectorPrint, HexagonInstrInfo, HexagonSubtarget, HexagonRegisterInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
