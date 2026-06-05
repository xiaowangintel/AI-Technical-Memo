# NVPTXForwardParams.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXForwardParams.cpp`
- Repository: `llvm-project`
- Purpose (EN): PTX supports 2 methods of accessing device function parameters:.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //- NVPTXForwardParams.cpp - NVPTX Forward Device Params Removing Local Copy -//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // PTX supports 2 methods of accessing device function parameters:
10: //
11: //   - "simple" case: If a parameters is only loaded, and all loads can address
12: //     the parameter via a constant offset, then the parameter may be loaded via
13: //     the ".param" address space. This case is not possible if the parameters
14: //     is stored to or has it's address taken. This method is preferable when
15: //     possible. Ex:
16: //
17: //            ld.param.u32    %r1, [foo_param_1];
18: //            ld.param.u32    %r2, [foo_param_1+4];
19: //
20: //   - "move param" case: For more complex cases the address of the param may be
21: //     placed in a register via a "mov" instruction. This "mov" also implicitly
22: //     moves the param to the ".local" address space and allows for it to be
23: //     written to. This essentially defers the responsibilty of the byval copy
24: //     to the PTX calling convention.
25: //
26: //            mov.b64         %rd1, foo_param_0;
27: //            st.local.u32    [%rd1], 42;
28: //            add.u64         %rd3, %rd1, %rd2;
29: //            ld.local.u32    %r2, [%rd3];
30: //
31: // In NVPTXLowerArgs and SelectionDAG, we pessimistically assume that all
32: // parameters will use the "move param" case and the local address space. This
33: // pass is responsible for switching to the "simple" case when possible, as it
34: // is more efficient.
35: //
36: // We do this by simply traversing uses of the param "mov" instructions an
37: // trivially checking if they are all loads.
38: //
39: //===----------------------------------------------------------------------===//
40:
```
- EN: This range contains banner comments or file-level notes that explain the role of the file and establish context for the backend component.
- CN: 这一段包含文件头注释或说明文字，用来交代该后端组件的职责与上下文。

### Lines 41-80
```cpp
41: #include "NVPTX.h"
42: #include "llvm/ADT/SmallVector.h"
43: #include "llvm/CodeGen/MachineFunctionPass.h"
44: #include "llvm/CodeGen/MachineInstr.h"
45: #include "llvm/CodeGen/MachineOperand.h"
46: #include "llvm/CodeGen/MachineRegisterInfo.h"
47: #include "llvm/CodeGen/TargetRegisterInfo.h"
48: #include "llvm/Support/ErrorHandling.h"
49:
50: using namespace llvm;
51:
52: static bool traverseMoveUse(MachineInstr &U, const MachineRegisterInfo &MRI,
53:                             SmallVectorImpl<MachineInstr *> &RemoveList,
54:                             SmallVectorImpl<MachineInstr *> &LoadInsts) {
55:   switch (U.getOpcode()) {
56:   case NVPTX::LD_i16:
57:   case NVPTX::LD_i32:
58:   case NVPTX::LD_i64:
59:   case NVPTX::LDV_i16_v2:
60:   case NVPTX::LDV_i16_v4:
61:   case NVPTX::LDV_i32_v2:
62:   case NVPTX::LDV_i32_v4:
63:   case NVPTX::LDV_i64_v2:
64:   case NVPTX::LDV_i64_v4: {
65:     LoadInsts.push_back(&U);
66:     return true;
67:   }
68:   case NVPTX::cvta_local:
69:   case NVPTX::cvta_local_64:
70:   case NVPTX::cvta_to_local:
71:   case NVPTX::cvta_to_local_64: {
72:     for (auto &U2 : MRI.use_instructions(U.operands_begin()->getReg()))
73:       if (!traverseMoveUse(U2, MRI, RemoveList, LoadInsts))
74:         return false;
75:
76:     RemoveList.push_back(&U);
77:     return true;
78:   }
79:   default:
80:     return false;
```
- EN: This range implements operational logic in helpers such as push_back, translating backend policy into executable code.
- CN: 这一段实现了 push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   }
 82: }
 83:
 84: static bool eliminateMove(MachineInstr &Mov, const MachineRegisterInfo &MRI,
 85:                           SmallVectorImpl<MachineInstr *> &RemoveList) {
 86:   SmallVector<MachineInstr *, 16> MaybeRemoveList;
 87:   SmallVector<MachineInstr *, 16> LoadInsts;
 88:
 89:   for (auto &U : MRI.use_instructions(Mov.operands_begin()->getReg()))
 90:     if (!traverseMoveUse(U, MRI, MaybeRemoveList, LoadInsts))
 91:       return false;
 92:
 93:   RemoveList.append(MaybeRemoveList);
 94:   RemoveList.push_back(&Mov);
 95:
 96:   const MachineOperand *ParamSymbol = Mov.uses().begin();
 97:   assert(ParamSymbol->isSymbol());
 98:
 99:   constexpr unsigned LDInstBasePtrOpIdx = 6;
100:   constexpr unsigned LDInstAddrSpaceOpIdx = 2;
101:   for (auto *LI : LoadInsts) {
102:     (LI->uses().begin() + LDInstBasePtrOpIdx)
103:         ->ChangeToES(ParamSymbol->getSymbolName());
104:     (LI->uses().begin() + LDInstAddrSpaceOpIdx)
105:         ->ChangeToImmediate(NVPTX::AddressSpace::DeviceParam);
106:   }
107:   return true;
108: }
109:
110: static bool forwardDeviceParams(MachineFunction &MF) {
111:   const auto &MRI = MF.getRegInfo();
112:
113:   bool Changed = false;
114:   SmallVector<MachineInstr *, 16> RemoveList;
115:   for (auto &MI : make_early_inc_range(*MF.begin()))
116:     if (MI.getOpcode() == NVPTX::MOV32_PARAM ||
117:         MI.getOpcode() == NVPTX::MOV64_PARAM)
118:       Changed |= eliminateMove(MI, MRI, RemoveList);
119:
120:   for (auto *MI : RemoveList)
```
- EN: This range implements operational logic in helpers such as append, push_back, uses, assert, translating backend policy into executable code.
- CN: 这一段实现了 append、push_back、uses、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-154
```cpp
121:     MI->eraseFromParent();
122:
123:   return Changed;
124: }
125:
126: /// ----------------------------------------------------------------------------
127: ///                       Pass (Manager) Boilerplate
128: /// ----------------------------------------------------------------------------
129:
130: namespace {
131: struct NVPTXForwardParamsPass : public MachineFunctionPass {
132:   static char ID;
133:   NVPTXForwardParamsPass() : MachineFunctionPass(ID) {}
134:
135:   bool runOnMachineFunction(MachineFunction &MF) override;
136:
137:   void getAnalysisUsage(AnalysisUsage &AU) const override {
138:     MachineFunctionPass::getAnalysisUsage(AU);
139:   }
140: };
141: } // namespace
142:
143: char NVPTXForwardParamsPass::ID = 0;
144:
145: INITIALIZE_PASS(NVPTXForwardParamsPass, "nvptx-forward-params",
146:                 "NVPTX Forward Params", false, false)
147:
148: bool NVPTXForwardParamsPass::runOnMachineFunction(MachineFunction &MF) {
149:   return forwardDeviceParams(MF);
150: }
151:
152: MachineFunctionPass *llvm::createNVPTXForwardParamsPass() {
153:   return new NVPTXForwardParamsPass();
154: }
```
- EN: This range defines or declares important types such as eraseFromParent, NVPTXForwardParamsPass, runOnMachineFunction, getAnalysisUsage, shaping the data model used by NVPTXForwardParams.cpp.
- CN: 这一段定义或声明了 eraseFromParent、NVPTXForwardParamsPass、runOnMachineFunction、getAnalysisUsage 等关键类型，构成 NVPTXForwardParams.cpp 使用的数据模型。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include push_back, append, uses, assert, ChangeToES, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 push_back, append, uses, assert, ChangeToES，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/CodeGen/MachineFunctionPass.h`
  - `llvm/CodeGen/MachineInstr.h`
  - `llvm/CodeGen/MachineOperand.h`
  - `llvm/CodeGen/MachineRegisterInfo.h`
  - `llvm/CodeGen/TargetRegisterInfo.h`
  - `llvm/Support/ErrorHandling.h`
