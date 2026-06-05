# NVPTXLowerAlloca.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXLowerAlloca.cpp`
- Repository: `llvm-project`
- Purpose (EN): For all alloca instructions, and add a pair of cast to local address for each of them.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- NVPTXLowerAlloca.cpp - Make alloca to use local memory =====--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // For all alloca instructions, and add a pair of cast to local address for
10: // each of them. For example,
11: //
12: //   %A = alloca i32
13: //   store i32 0, i32* %A ; emits st.u32
14: //
15: // will be transformed to
16: //
17: //   %A = alloca i32
18: //   %Local = addrspacecast i32* %A to i32 addrspace(5)*
19: //   %Generic = addrspacecast i32 addrspace(5)* %A to i32*
20: //   store i32 0, i32 addrspace(5)* %Generic ; emits st.local.u32
21: //
22: // And we will rely on NVPTXInferAddressSpaces to combine the last two
23: // instructions.
24: //
25: //===----------------------------------------------------------------------===//
26:
27: #include "MCTargetDesc/NVPTXBaseInfo.h"
28: #include "NVPTX.h"
29: #include "llvm/IR/Function.h"
30: #include "llvm/IR/Instructions.h"
31: #include "llvm/IR/Type.h"
32: #include "llvm/Pass.h"
33:
34: using namespace llvm;
35:
36: namespace {
37: class NVPTXLowerAlloca : public FunctionPass {
38:   bool runOnFunction(Function &F) override;
39:
40: public:
```
- EN: This range defines or declares important types such as NVPTXLowerAlloca, runOnFunction, shaping the data model used by NVPTXLowerAlloca.cpp.
- CN: 这一段定义或声明了 NVPTXLowerAlloca、runOnFunction 等关键类型，构成 NVPTXLowerAlloca.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:   static char ID; // Pass identification, replacement for typeid
42:   NVPTXLowerAlloca() : FunctionPass(ID) {}
43:   StringRef getPassName() const override {
44:     return "convert address space of alloca'ed memory to local";
45:   }
46: };
47: } // namespace
48:
49: char NVPTXLowerAlloca::ID = 1;
50:
51: INITIALIZE_PASS(NVPTXLowerAlloca, "nvptx-lower-alloca", "Lower Alloca", false,
52:                 false)
53:
54: // =============================================================================
55: // Main function for this pass.
56: // =============================================================================
57: bool NVPTXLowerAlloca::runOnFunction(Function &F) {
58:   if (skipFunction(F))
59:     return false;
60:
61:   bool Changed = false;
62:   for (auto &BB : F)
63:     for (auto &I : BB) {
64:       if (auto allocaInst = dyn_cast<AllocaInst>(&I)) {
65:         Changed = true;
66:
67:         unsigned AllocAddrSpace = allocaInst->getAddressSpace();
68:         assert((AllocAddrSpace == ADDRESS_SPACE_GENERIC ||
69:                 AllocAddrSpace == ADDRESS_SPACE_LOCAL) &&
70:                "AllocaInst can only be in Generic or Local address space for "
71:                "NVPTX.");
72:
73:         Instruction *AllocaInLocalAS = allocaInst;
74:
75:         // We need to make sure that LLVM has info that alloca needs to go to
76:         // ADDRESS_SPACE_LOCAL for InferAddressSpace pass.
77:         //
78:         // For allocas in ADDRESS_SPACE_GENERIC, we add addrspacecast to
79:         // ADDRESS_SPACE_LOCAL and back to ADDRESS_SPACE_GENERIC, so that
80:         // the alloca's users still use a generic pointer to operate on.
```
- EN: This range implements operational logic in helpers such as NVPTXLowerAlloca, getPassName, NVPTXLowerAlloca::runOnFunction, getAddressSpace, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXLowerAlloca、getPassName、NVPTXLowerAlloca::runOnFunction、getAddressSpace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:         //
 82:         // For allocas already in ADDRESS_SPACE_LOCAL, we just need
 83:         // addrspacecast to ADDRESS_SPACE_GENERIC.
 84:         if (AllocAddrSpace == ADDRESS_SPACE_GENERIC) {
 85:           auto ASCastToLocalAS = new AddrSpaceCastInst(
 86:               allocaInst,
 87:               PointerType::get(allocaInst->getContext(), ADDRESS_SPACE_LOCAL),
 88:               "");
 89:           ASCastToLocalAS->insertAfter(allocaInst->getIterator());
 90:           AllocaInLocalAS = ASCastToLocalAS;
 91:         }
 92:
 93:         auto AllocaInGenericAS = new AddrSpaceCastInst(
 94:             AllocaInLocalAS,
 95:             PointerType::get(allocaInst->getContext(), ADDRESS_SPACE_GENERIC),
 96:             "");
 97:         AllocaInGenericAS->insertAfter(AllocaInLocalAS->getIterator());
 98:
 99:         for (Use &AllocaUse : llvm::make_early_inc_range(allocaInst->uses())) {
100:           // Check Load, Store, GEP, and BitCast Uses on alloca and make them
101:           // use the converted generic address, in order to expose non-generic
102:           // addrspacecast to NVPTXInferAddressSpaces. For other types
103:           // of instructions this is unnecessary and may introduce redundant
104:           // address cast.
105:           auto LI = dyn_cast<LoadInst>(AllocaUse.getUser());
106:           if (LI && LI->getPointerOperand() == allocaInst &&
107:               !LI->isVolatile()) {
108:             LI->setOperand(LI->getPointerOperandIndex(), AllocaInGenericAS);
109:             continue;
110:           }
111:           auto SI = dyn_cast<StoreInst>(AllocaUse.getUser());
112:           if (SI && SI->getPointerOperand() == allocaInst &&
113:               !SI->isVolatile()) {
114:             SI->setOperand(SI->getPointerOperandIndex(), AllocaInGenericAS);
115:             continue;
116:           }
117:           auto GI = dyn_cast<GetElementPtrInst>(AllocaUse.getUser());
118:           if (GI && GI->getPointerOperand() == allocaInst) {
119:             GI->setOperand(GI->getPointerOperandIndex(), AllocaInGenericAS);
120:             continue;
```
- EN: This range implements operational logic in helpers such as insertAfter, getUser, isVolatile, setOperand, translating backend policy into executable code.
- CN: 这一段实现了 insertAfter、getUser、isVolatile、setOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-135
```cpp
121:           }
122:           auto BI = dyn_cast<BitCastInst>(AllocaUse.getUser());
123:           if (BI && BI->getOperand(0) == allocaInst) {
124:             BI->setOperand(0, AllocaInGenericAS);
125:             continue;
126:           }
127:         }
128:       }
129:     }
130:   return Changed;
131: }
132:
133: FunctionPass *llvm::createNVPTXLowerAllocaPass() {
134:   return new NVPTXLowerAlloca();
135: }
```
- EN: This range implements operational logic in helpers such as getUser, setOperand, llvm::createNVPTXLowerAllocaPass, NVPTXLowerAlloca, translating backend policy into executable code.
- CN: 这一段实现了 getUser、setOperand、llvm::createNVPTXLowerAllocaPass、NVPTXLowerAlloca 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXLowerAlloca, runOnFunction, getPassName, NVPTXLowerAlloca::runOnFunction, getAddressSpace, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXLowerAlloca, runOnFunction, getPassName, NVPTXLowerAlloca::runOnFunction, getAddressSpace，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Function.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Type.h`
  - `llvm/Pass.h`
