# NVPTXProxyRegErasure.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXProxyRegErasure.cpp`
- Repository: `llvm-project`
- Purpose (EN): The pass is needed to remove ProxyReg instructions and restore related registers.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVPTXProxyRegErasure.cpp - NVPTX Proxy Register Instruction Erasure -==//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // The pass is needed to remove ProxyReg instructions and restore related
10: // registers. The instructions were needed at instruction selection stage to
11: // make sure that callseq_end nodes won't be removed as "dead nodes". This can
12: // happen when we expand instructions into libcalls and the call site doesn't
13: // care about the libcall chain. Call site cares about data flow only, and the
14: // latest data flow node happens to be before callseq_end. Therefore the node
15: // becomes dangling and "dead". The ProxyReg acts like an additional data flow
16: // node *after* the callseq_end in the chain and ensures that everything will be
17: // preserved.
18: //
19: //===----------------------------------------------------------------------===//
20:
21: #include "NVPTX.h"
22: #include "llvm/CodeGen/MachineFunctionPass.h"
23: #include "llvm/CodeGen/TargetRegisterInfo.h"
24:
25: using namespace llvm;
26:
27: namespace {
28:
29: struct NVPTXProxyRegErasure : public MachineFunctionPass {
30:   static char ID;
31:   NVPTXProxyRegErasure() : MachineFunctionPass(ID) {}
32:
33:   bool runOnMachineFunction(MachineFunction &MF) override;
34:
35:   StringRef getPassName() const override {
36:     return "NVPTX Proxy Register Instruction Erasure";
37:   }
38:
39:   void getAnalysisUsage(AnalysisUsage &AU) const override {
40:     MachineFunctionPass::getAnalysisUsage(AU);
```
- EN: This range defines or declares important types such as NVPTXProxyRegErasure, runOnMachineFunction, getPassName, getAnalysisUsage, shaping the data model used by NVPTXProxyRegErasure.cpp.
- CN: 这一段定义或声明了 NVPTXProxyRegErasure、runOnMachineFunction、getPassName、getAnalysisUsage 等关键类型，构成 NVPTXProxyRegErasure.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:   }
42: };
43:
44: } // namespace
45:
46: char NVPTXProxyRegErasure::ID = 0;
47:
48: INITIALIZE_PASS(NVPTXProxyRegErasure, "nvptx-proxyreg-erasure",
49:                 "NVPTX ProxyReg Erasure", false, false)
50:
51: bool NVPTXProxyRegErasure::runOnMachineFunction(MachineFunction &MF) {
52:   SmallVector<MachineInstr *, 16> RemoveList;
53:
54:   // ProxyReg instructions forward a register as another: `%dst = mov.iN %src`.
55:   // Bulk RAUW the `%dst` registers in two passes over the machine function.
56:   DenseMap<Register, Register> RAUWBatch;
57:
58:   for (auto &BB : MF) {
59:     for (auto &MI : BB) {
60:       switch (MI.getOpcode()) {
61:       case NVPTX::ProxyRegB1:
62:       case NVPTX::ProxyRegB16:
63:       case NVPTX::ProxyRegB32:
64:       case NVPTX::ProxyRegB64: {
65:         auto &InOp = *MI.uses().begin();
66:         auto &OutOp = *MI.defs().begin();
67:         assert(InOp.isReg() && "ProxyReg input should be a register.");
68:         assert(OutOp.isReg() && "ProxyReg output should be a register.");
69:         RemoveList.push_back(&MI);
70:         Register replacement = InOp.getReg();
71:         // Check if the replacement itself has been replaced.
72:         if (auto it = RAUWBatch.find(replacement); it != RAUWBatch.end())
73:           replacement = it->second;
74:         RAUWBatch.try_emplace(OutOp.getReg(), replacement);
75:         break;
76:       }
77:       }
78:     }
79:   }
80:
```
- EN: This range implements operational logic in helpers such as NVPTXProxyRegErasure::runOnMachineFunction, uses, defs, assert, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXProxyRegErasure::runOnMachineFunction、uses、defs、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-108
```cpp
 81:   // If there were no proxy instructions, exit early.
 82:   if (RemoveList.empty())
 83:     return false;
 84:
 85:   // Erase the proxy instructions first.
 86:   for (auto *MI : RemoveList) {
 87:     MI->eraseFromParent();
 88:   }
 89:
 90:   // Now go replace the registers.
 91:   for (auto &BB : MF) {
 92:     for (auto &MI : BB) {
 93:       for (auto &Op : MI.uses()) {
 94:         if (!Op.isReg())
 95:           continue;
 96:         auto it = RAUWBatch.find(Op.getReg());
 97:         if (it != RAUWBatch.end())
 98:           Op.setReg(it->second);
 99:       }
100:     }
101:   }
102:
103:   return true;
104: }
105:
106: MachineFunctionPass *llvm::createNVPTXProxyRegErasurePass() {
107:   return new NVPTXProxyRegErasure();
108: }
```
- EN: This range implements operational logic in helpers such as eraseFromParent, find, setReg, llvm::createNVPTXProxyRegErasurePass, translating backend policy into executable code.
- CN: 这一段实现了 eraseFromParent、find、setReg、llvm::createNVPTXProxyRegErasurePass 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXProxyRegErasure, runOnMachineFunction, getPassName, getAnalysisUsage, MachineFunctionPass::getAnalysisUsage, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXProxyRegErasure, runOnMachineFunction, getPassName, getAnalysisUsage, MachineFunctionPass::getAnalysisUsage，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/MachineFunctionPass.h`
  - `llvm/CodeGen/TargetRegisterInfo.h`
