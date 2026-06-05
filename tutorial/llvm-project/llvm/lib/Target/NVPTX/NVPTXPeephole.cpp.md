# NVPTXPeephole.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXPeephole.cpp`
- Repository: `llvm-project`
- Purpose (EN): In NVPTX, NVPTXFrameLowering will emit following instruction at the beginning of a MachineFunction.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXPeephole.cpp - NVPTX Peephole Optimiztions -------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // In NVPTX, NVPTXFrameLowering will emit following instruction at the beginning
10: // of a MachineFunction.
11: //
12: //   mov %SPL, %depot
13: //   cvta.local %SP, %SPL
14: //
15: // Because Frame Index is a generic address and alloca can only return generic
16: // pointer, without this pass the instructions producing alloca'ed address will
17: // be based on %SP. NVPTXLowerAlloca tends to help replace store and load on
18: // this address with their .local versions, but this may introduce a lot of
19: // cvta.to.local instructions. Performance can be improved if we avoid casting
20: // address back and forth and directly calculate local address based on %SPL.
21: // This peephole pass optimizes these cases, for example
22: //
23: // It will transform the following pattern
24: //    %0 = LEA_ADDRi64 %VRFrame64, 4
25: //    %1 = cvta_to_local_64 %0
26: //
27: // into
28: //    %1 = LEA_ADDRi64 %VRFrameLocal64, 4
29: //
30: // %VRFrameLocal64 is the virtual register name of %SPL
31: //
32: //===----------------------------------------------------------------------===//
33:
34: #include "NVPTX.h"
35: #include "NVPTXRegisterInfo.h"
36: #include "NVPTXSubtarget.h"
37: #include "llvm/CodeGen/MachineFunctionPass.h"
38: #include "llvm/CodeGen/MachineInstrBuilder.h"
39: #include "llvm/CodeGen/MachineRegisterInfo.h"
40: #include "llvm/CodeGen/TargetInstrInfo.h"
```
- EN: This range pulls in local backend headers and core LLVM infrastructure needed by the rest of the file.
- CN: 这一段引入本地后端头文件以及后续实现所依赖的 LLVM 基础设施。

### Lines 41-80
```cpp
41: #include "llvm/CodeGen/TargetRegisterInfo.h"
42:
43: using namespace llvm;
44:
45: #define DEBUG_TYPE "nvptx-peephole"
46:
47: namespace {
48: struct NVPTXPeephole : public MachineFunctionPass {
49:  public:
50:   static char ID;
51:   NVPTXPeephole() : MachineFunctionPass(ID) {}
52:
53:   bool runOnMachineFunction(MachineFunction &MF) override;
54:
55:   StringRef getPassName() const override {
56:     return "NVPTX optimize redundant cvta.to.local instruction";
57:   }
58:
59:   void getAnalysisUsage(AnalysisUsage &AU) const override {
60:     MachineFunctionPass::getAnalysisUsage(AU);
61:   }
62: };
63: }
64:
65: char NVPTXPeephole::ID = 0;
66:
67: INITIALIZE_PASS(NVPTXPeephole, "nvptx-peephole", "NVPTX Peephole", false, false)
68:
69: static bool isCVTAToLocalCombinationCandidate(MachineInstr &Root) {
70:   auto &MBB = *Root.getParent();
71:   auto &MF = *MBB.getParent();
72:   // Check current instruction is cvta.to.local
73:   if (Root.getOpcode() != NVPTX::cvta_to_local_64 &&
74:       Root.getOpcode() != NVPTX::cvta_to_local)
75:     return false;
76:
77:   auto &Op = Root.getOperand(1);
78:   const auto &MRI = MF.getRegInfo();
79:   MachineInstr *GenericAddrDef = nullptr;
80:   if (Op.isReg() && Op.getReg().isVirtual()) {
```
- EN: This range defines or declares important types such as NVPTXPeephole, runOnMachineFunction, getPassName, getAnalysisUsage, shaping the data model used by NVPTXPeephole.cpp.
- CN: 这一段定义或声明了 NVPTXPeephole、runOnMachineFunction、getPassName、getAnalysisUsage 等关键类型，构成 NVPTXPeephole.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:     GenericAddrDef = MRI.getUniqueVRegDef(Op.getReg());
 82:   }
 83:
 84:   // Check the register operand is uniquely defined by LEA_ADDRi instruction
 85:   if (!GenericAddrDef || GenericAddrDef->getParent() != &MBB ||
 86:       (GenericAddrDef->getOpcode() != NVPTX::LEA_ADDRi64 &&
 87:        GenericAddrDef->getOpcode() != NVPTX::LEA_ADDRi)) {
 88:     return false;
 89:   }
 90:
 91:   const NVPTXRegisterInfo *NRI =
 92:       MF.getSubtarget<NVPTXSubtarget>().getRegisterInfo();
 93:
 94:   // Check the LEA_ADDRi operand is Frame index
 95:   auto &BaseAddrOp = GenericAddrDef->getOperand(1);
 96:   if (BaseAddrOp.isReg() && BaseAddrOp.getReg() == NRI->getFrameRegister(MF)) {
 97:     return true;
 98:   }
 99:
100:   return false;
101: }
102:
103: static void CombineCVTAToLocal(MachineInstr &Root) {
104:   auto &MBB = *Root.getParent();
105:   auto &MF = *MBB.getParent();
106:   const auto &MRI = MF.getRegInfo();
107:   const TargetInstrInfo *TII = MF.getSubtarget().getInstrInfo();
108:   auto &Prev = *MRI.getUniqueVRegDef(Root.getOperand(1).getReg());
109:
110:   const NVPTXRegisterInfo *NRI =
111:       MF.getSubtarget<NVPTXSubtarget>().getRegisterInfo();
112:
113:   MachineInstrBuilder MIB =
114:       BuildMI(MF, Root.getDebugLoc(), TII->get(Prev.getOpcode()),
115:               Root.getOperand(0).getReg())
116:           .addReg(NRI->getFrameLocalRegister(MF))
117:           .add(Prev.getOperand(2));
118:
119:   MBB.insert((MachineBasicBlock::iterator)&Root, MIB);
120:
```
- EN: This range implements operational logic in helpers such as getUniqueVRegDef, getOpcode, getRegisterInfo, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getUniqueVRegDef、getOpcode、getRegisterInfo、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:   // Check if MRI has only one non dbg use, which is Root
122:   if (MRI.hasOneNonDBGUse(Prev.getOperand(0).getReg())) {
123:     Prev.eraseFromParent();
124:   }
125:   Root.eraseFromParent();
126: }
127:
128: bool NVPTXPeephole::runOnMachineFunction(MachineFunction &MF) {
129:   if (skipFunction(MF.getFunction()))
130:     return false;
131:
132:   bool Changed = false;
133:   // Loop over all of the basic blocks.
134:   for (auto &MBB : MF) {
135:     // Traverse the basic block.
136:     auto BlockIter = MBB.begin();
137:
138:     while (BlockIter != MBB.end()) {
139:       auto &MI = *BlockIter++;
140:       if (isCVTAToLocalCombinationCandidate(MI)) {
141:         CombineCVTAToLocal(MI);
142:         Changed = true;
143:       }
144:     }  // Instruction
145:   }    // Basic Block
146:
147:   const NVPTXRegisterInfo *NRI =
148:       MF.getSubtarget<NVPTXSubtarget>().getRegisterInfo();
149:
150:   // Remove unnecessary %VRFrame = cvta.local %VRFrameLocal
151:   const auto &MRI = MF.getRegInfo();
152:   if (MRI.use_empty(NRI->getFrameRegister(MF))) {
153:     if (auto MI = MRI.getUniqueVRegDef(NRI->getFrameRegister(MF))) {
154:       MI->eraseFromParent();
155:     }
156:   }
157:
158:   return Changed;
159: }
160:
```
- EN: This range implements operational logic in helpers such as eraseFromParent, NVPTXPeephole::runOnMachineFunction, begin, CombineCVTAToLocal, translating backend policy into executable code.
- CN: 这一段实现了 eraseFromParent、NVPTXPeephole::runOnMachineFunction、begin、CombineCVTAToLocal 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-161
```cpp
161: MachineFunctionPass *llvm::createNVPTXPeephole() { return new NVPTXPeephole(); }
```
- EN: This range implements operational logic in helpers such as llvm::createNVPTXPeephole, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createNVPTXPeephole 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXPeephole, runOnMachineFunction, getPassName, getAnalysisUsage, MachineFunctionPass::getAnalysisUsage, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXPeephole, runOnMachineFunction, getPassName, getAnalysisUsage, MachineFunctionPass::getAnalysisUsage，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
  - `NVPTXRegisterInfo.h`
  - `NVPTXSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineFunctionPass.h`
  - `llvm/CodeGen/MachineInstrBuilder.h`
  - `llvm/CodeGen/MachineRegisterInfo.h`
  - `llvm/CodeGen/TargetInstrInfo.h`
  - `llvm/CodeGen/TargetRegisterInfo.h`
