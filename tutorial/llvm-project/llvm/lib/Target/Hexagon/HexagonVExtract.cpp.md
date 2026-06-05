# HexagonVExtract.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonVExtract.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon vector extract lowering.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```cpp
     1: //===- HexagonVExtract.cpp ------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // This pass will replace multiple occurrences of V6_extractw from the same
     9: // vector register with a combination of a vector store and scalar loads.
    10: //===----------------------------------------------------------------------===//
    11: 
    12: #include "Hexagon.h"
    13: #include "HexagonInstrInfo.h"
    14: #include "HexagonMachineFunctionInfo.h"
    15: #include "HexagonRegisterInfo.h"
    16: #include "HexagonSubtarget.h"
    17: #include "llvm/ADT/SmallVector.h"
    18: #include "llvm/Pass.h"
    19: #include "llvm/CodeGen/MachineBasicBlock.h"
    20: #include "llvm/CodeGen/MachineFunction.h"
    21: #include "llvm/CodeGen/MachineFunctionPass.h"
    22: #include "llvm/CodeGen/MachineInstrBuilder.h"
    23: #include "llvm/CodeGen/MachineRegisterInfo.h"
    24: #include "llvm/Support/CommandLine.h"
    25: 
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, ... (13 total), establishing the LLVM/Hexagon APIs used below. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVExtract, HexagonInstrInfo, HexagonMachineFunctionInfo, HexagonRegisterInfo, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, ... (13 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVExtract, HexagonInstrInfo, HexagonMachineFunctionInfo, HexagonRegisterInfo, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```cpp
    26: #include <map>
    27: 
    28: using namespace llvm;
    29: 
    30: static cl::opt<unsigned> VExtractThreshold(
    31:     "hexagon-vextract-threshold", cl::Hidden, cl::init(1),
    32:     cl::desc("Threshold for triggering vextract replacement"));
    33: 
    34: namespace {
    35:   class HexagonVExtract : public MachineFunctionPass {
    36:   public:
    37:     static char ID;
    38:     HexagonVExtract() : MachineFunctionPass(ID) {}
    39: 
    40:     StringRef getPassName() const override {
    41:       return "Hexagon optimize vextract";
    42:     }
    43:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    44:       MachineFunctionPass::getAnalysisUsage(AU);
    45:     }
    46:     bool runOnMachineFunction(MachineFunction &MF) override;
    47: 
    48:   private:
    49:     const HexagonSubtarget *HST = nullptr;
    50:     const HexagonInstrInfo *HII = nullptr;
```
- EN: It imports headers such as map, establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonVExtract, which carry the state or API of this component. It defines declarative TableGen records like HexagonVExtract; these records are consumed by TableGen instead of executed directly.
- CN: 这里引入了 map 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonVExtract 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonVExtract 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 51-75 / 第 51-75 行

```cpp
    51: 
    52:     unsigned genElemLoad(MachineInstr *ExtI, unsigned BaseR,
    53:                          MachineRegisterInfo &MRI);
    54:   };
    55: 
    56:   char HexagonVExtract::ID = 0;
    57: }
    58: 
    59: INITIALIZE_PASS(HexagonVExtract, "hexagon-vextract",
    60:   "Hexagon optimize vextract", false, false)
    61: 
    62: unsigned HexagonVExtract::genElemLoad(MachineInstr *ExtI, unsigned BaseR,
    63:                                       MachineRegisterInfo &MRI) {
    64:   MachineBasicBlock &ExtB = *ExtI->getParent();
    65:   DebugLoc DL = ExtI->getDebugLoc();
    66:   Register ElemR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
    67: 
    68:   Register ExtIdxR = ExtI->getOperand(2).getReg();
    69:   unsigned ExtIdxS = ExtI->getOperand(2).getSubReg();
    70: 
    71:   // Simplified check for a compile-time constant value of ExtIdxR.
    72:   if (ExtIdxS == 0) {
    73:     MachineInstr *DI = MRI.getVRegDef(ExtIdxR);
    74:     if (DI->getOpcode() == Hexagon::A2_tfrsi) {
    75:       unsigned V = DI->getOperand(1).getImm();
```
- EN: It declares or implements routines such as genElemLoad, INITIALIZE_PASS, getParent, getDebugLoc, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonVExtract, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 genElemLoad, INITIALIZE_PASS, getParent, getDebugLoc, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonVExtract，说明了它与同级后端组件的连接关系。

### Lines 76-100 / 第 76-100 行

```cpp
    76:       V &= (HST->getVectorLength()-1) & -4u;
    77: 
    78:       BuildMI(ExtB, ExtI, DL, HII->get(Hexagon::L2_loadri_io), ElemR)
    79:         .addReg(BaseR)
    80:         .addImm(V);
    81:       return ElemR;
    82:     }
    83:   }
    84: 
    85:   Register IdxR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
    86:   BuildMI(ExtB, ExtI, DL, HII->get(Hexagon::A2_andir), IdxR)
    87:     .add(ExtI->getOperand(2))
    88:     .addImm(-4);
    89:   BuildMI(ExtB, ExtI, DL, HII->get(Hexagon::L4_loadri_rr), ElemR)
    90:     .addReg(BaseR)
    91:     .addReg(IdxR)
    92:     .addImm(0);
    93:   return ElemR;
    94: }
    95: 
    96: bool HexagonVExtract::runOnMachineFunction(MachineFunction &MF) {
    97:   HST = &MF.getSubtarget<HexagonSubtarget>();
    98:   HII = HST->getInstrInfo();
    99:   const auto &HRI = *HST->getRegisterInfo();
   100:   MachineRegisterInfo &MRI = MF.getRegInfo();
```
- EN: It declares or implements routines such as BuildMI, createVirtualRegister, HexagonVExtract::runOnMachineFunction, getSubtarget<HexagonSubtarget>, ... (7 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonVExtract, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 BuildMI, createVirtualRegister, HexagonVExtract::runOnMachineFunction, getSubtarget<HexagonSubtarget>, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonVExtract, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 101-125 / 第 101-125 行

```cpp
   101:   MachineFrameInfo &MFI = MF.getFrameInfo();
   102:   Register AR =
   103:       MF.getInfo<HexagonMachineFunctionInfo>()->getStackAlignBaseReg();
   104:   std::map<unsigned, SmallVector<MachineInstr *, 4>> VExtractMap;
   105:   bool Changed = false;
   106: 
   107:   for (MachineBasicBlock &MBB : MF) {
   108:     for (MachineInstr &MI : MBB) {
   109:       unsigned Opc = MI.getOpcode();
   110:       if (Opc != Hexagon::V6_extractw)
   111:         continue;
   112:       Register VecR = MI.getOperand(1).getReg();
   113:       VExtractMap[VecR].push_back(&MI);
   114:     }
   115:   }
   116: 
   117:   auto EmitAddr = [&] (MachineBasicBlock &BB, MachineBasicBlock::iterator At,
   118:                        DebugLoc dl, int FI, unsigned Offset) {
   119:     Register AddrR = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   120:     unsigned FiOpc = AR != 0 ? Hexagon::PS_fia : Hexagon::PS_fi;
   121:     auto MIB = BuildMI(BB, At, dl, HII->get(FiOpc), AddrR);
   122:     if (AR)
   123:       MIB.addReg(AR);
   124:     MIB.addFrameIndex(FI).addImm(Offset);
   125:     return AddrR;
```
- EN: It declares or implements routines such as getFrameInfo, getInfo<HexagonMachineFunctionInfo>, getOpcode, getOperand, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMachineFunctionInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getFrameInfo, getInfo<HexagonMachineFunctionInfo>, getOpcode, getOperand, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMachineFunctionInfo，说明了它与同级后端组件的连接关系。

### Lines 126-150 / 第 126-150 行

```cpp
   126:   };
   127: 
   128:   MaybeAlign MaxAlign;
   129:   for (auto &P : VExtractMap) {
   130:     unsigned VecR = P.first;
   131:     if (P.second.size() <= VExtractThreshold)
   132:       continue;
   133: 
   134:     const auto &VecRC = *MRI.getRegClass(VecR);
   135:     Align Alignment = HRI.getSpillAlign(VecRC);
   136:     MaxAlign = std::max(MaxAlign.valueOrOne(), Alignment);
   137:     // Make sure this is not a spill slot: spill slots cannot be aligned
   138:     // if there are variable-sized objects on the stack. They must be
   139:     // accessible via FP (which is not aligned), because SP is unknown,
   140:     // and AP may not be available at the location of the load/store.
   141:     int FI = MFI.CreateStackObject(HRI.getSpillSize(VecRC), Alignment,
   142:                                    /*isSpillSlot*/ false);
   143: 
   144:     MachineInstr *DefI = MRI.getVRegDef(VecR);
   145:     MachineBasicBlock::iterator At = std::next(DefI->getIterator());
   146:     MachineBasicBlock &DefB = *DefI->getParent();
   147:     unsigned StoreOpc = VecRC.getID() == Hexagon::HvxVRRegClassID
   148:                           ? Hexagon::V6_vS32b_ai
   149:                           : Hexagon::PS_vstorerw_ai;
   150:     Register AddrR = EmitAddr(DefB, At, DefI->getDebugLoc(), FI, 0);
```
- EN: It declares or implements routines such as getRegClass, getSpillAlign, std::max, FP, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getRegClass, getSpillAlign, std::max, FP, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-175 / 第 151-175 行

```cpp
   151:     BuildMI(DefB, At, DefI->getDebugLoc(), HII->get(StoreOpc))
   152:       .addReg(AddrR)
   153:       .addImm(0)
   154:       .addReg(VecR);
   155: 
   156:     unsigned VecSize = HRI.getRegSizeInBits(VecRC) / 8;
   157: 
   158:     for (MachineInstr *ExtI : P.second) {
   159:       assert(ExtI->getOpcode() == Hexagon::V6_extractw);
   160:       unsigned SR = ExtI->getOperand(1).getSubReg();
   161:       assert(ExtI->getOperand(1).getReg() == VecR);
   162: 
   163:       MachineBasicBlock &ExtB = *ExtI->getParent();
   164:       Register BaseR = EmitAddr(ExtB, ExtI, ExtI->getDebugLoc(), FI,
   165:                                 SR == 0 ? 0 : VecSize/2);
   166: 
   167:       unsigned ElemR = genElemLoad(ExtI, BaseR, MRI);
   168:       Register ExtR = ExtI->getOperand(0).getReg();
   169:       MRI.replaceRegWith(ExtR, ElemR);
   170:       ExtB.erase(ExtI);
   171:       Changed = true;
   172:     }
   173:   }
   174: 
   175:   if (AR && MaxAlign) {
```
- EN: It declares or implements routines such as BuildMI, assert, getOperand, getParent, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 BuildMI, assert, getOperand, getParent, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 176-189 / 第 176-189 行

```cpp
   176:     // Update the required stack alignment.
   177:     MachineInstr *AlignaI = MRI.getVRegDef(AR);
   178:     assert(AlignaI->getOpcode() == Hexagon::PS_aligna);
   179:     MachineOperand &Op = AlignaI->getOperand(1);
   180:     if (*MaxAlign > Op.getImm())
   181:       Op.setImm(MaxAlign->value());
   182:   }
   183: 
   184:   return Changed;
   185: }
   186: 
   187: FunctionPass *llvm::createHexagonVExtract() {
   188:   return new HexagonVExtract();
   189: }
```
- EN: It declares or implements routines such as getVRegDef, assert, getOperand, llvm::createHexagonVExtract, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonVExtract, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getVRegDef, assert, getOperand, llvm::createHexagonVExtract, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonVExtract，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制
- ABI and stack-frame lowering / ABI 与栈帧下沉

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonMachineFunctionInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SmallVector.h, llvm/Pass.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, ... (14 total)`
- Hexagon symbols / Hexagon 符号: `HexagonVExtract, HexagonInstrInfo, HexagonMachineFunctionInfo, HexagonRegisterInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
