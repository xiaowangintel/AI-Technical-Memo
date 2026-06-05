# NVPTXLowerAggrCopies.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXLowerAggrCopies.cpp`
- Repository: `llvm-project`
- Purpose (EN): Lower aggregate copies, memset, memcpy, memmov intrinsics into loops when the size is large or is not a compile-time constant.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- NVPTXLowerAggrCopies.cpp - ------------------------------*- C++ -*--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // \file
10: // Lower aggregate copies, memset, memcpy, memmov intrinsics into loops when
11: // the size is large or is not a compile-time constant.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #include "NVPTXLowerAggrCopies.h"
16: #include "NVPTX.h"
17: #include "llvm/Analysis/TargetTransformInfo.h"
18: #include "llvm/CodeGen/StackProtector.h"
19: #include "llvm/IR/Constants.h"
20: #include "llvm/IR/DataLayout.h"
21: #include "llvm/IR/Function.h"
22: #include "llvm/IR/Instructions.h"
23: #include "llvm/IR/IntrinsicInst.h"
24: #include "llvm/IR/Intrinsics.h"
25: #include "llvm/IR/LLVMContext.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
28: #include "llvm/Transforms/Utils/LowerMemIntrinsics.h"
29:
30: #define DEBUG_TYPE "nvptx"
31:
32: using namespace llvm;
33:
34: namespace {
35:
36: // actual analysis class, which is a functionpass
37: struct NVPTXLowerAggrCopies : public FunctionPass {
38:   static char ID;
39:
40:   NVPTXLowerAggrCopies() : FunctionPass(ID) {}
```
- EN: This range defines or declares important types such as NVPTXLowerAggrCopies, shaping the data model used by NVPTXLowerAggrCopies.cpp.
- CN: 这一段定义或声明了 NVPTXLowerAggrCopies 等关键类型，构成 NVPTXLowerAggrCopies.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:
42:   void getAnalysisUsage(AnalysisUsage &AU) const override {
43:     AU.addPreserved<StackProtector>();
44:     AU.addRequired<TargetTransformInfoWrapperPass>();
45:   }
46:
47:   bool runOnFunction(Function &F) override;
48:
49:   static const unsigned MaxAggrCopySize = 128;
50:
51:   StringRef getPassName() const override {
52:     return "Lower aggregate copies/intrinsics into loops";
53:   }
54: };
55:
56: char NVPTXLowerAggrCopies::ID = 0;
57:
58: bool NVPTXLowerAggrCopies::runOnFunction(Function &F) {
59:   SmallVector<LoadInst *, 4> AggrLoads;
60:   SmallVector<MemIntrinsic *, 4> MemCalls;
61:
62:   const DataLayout &DL = F.getDataLayout();
63:   LLVMContext &Context = F.getParent()->getContext();
64:   const TargetTransformInfo &TTI =
65:       getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
66:
67:   // Collect all aggregate loads and mem* calls.
68:   for (BasicBlock &BB : F) {
69:     for (Instruction &I : BB) {
70:       if (LoadInst *LI = dyn_cast<LoadInst>(&I)) {
71:         if (!LI->hasOneUse())
72:           continue;
73:
74:         if (DL.getTypeStoreSize(LI->getType()) < MaxAggrCopySize)
75:           continue;
76:
77:         if (StoreInst *SI = dyn_cast<StoreInst>(LI->user_back())) {
78:           if (SI->getOperand(0) != LI)
79:             continue;
80:           AggrLoads.push_back(LI);
```
- EN: This range implements operational logic in helpers such as getAnalysisUsage, runOnFunction, getPassName, NVPTXLowerAggrCopies::runOnFunction, translating backend policy into executable code.
- CN: 这一段实现了 getAnalysisUsage、runOnFunction、getPassName、NVPTXLowerAggrCopies::runOnFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:         }
 82:       } else if (MemIntrinsic *IntrCall = dyn_cast<MemIntrinsic>(&I)) {
 83:         // Convert intrinsic calls with variable size or with constant size
 84:         // larger than the MaxAggrCopySize threshold.
 85:         if (ConstantInt *LenCI = dyn_cast<ConstantInt>(IntrCall->getLength())) {
 86:           if (LenCI->getZExtValue() >= MaxAggrCopySize) {
 87:             MemCalls.push_back(IntrCall);
 88:           }
 89:         } else {
 90:           MemCalls.push_back(IntrCall);
 91:         }
 92:       }
 93:     }
 94:   }
 95:
 96:   if (AggrLoads.size() == 0 && MemCalls.size() == 0) {
 97:     return false;
 98:   }
 99:
100:   //
101:   // Do the transformation of an aggr load/copy/set to a loop
102:   //
103:   for (LoadInst *LI : AggrLoads) {
104:     auto *SI = cast<StoreInst>(*LI->user_begin());
105:     Value *SrcAddr = LI->getOperand(0);
106:     Value *DstAddr = SI->getOperand(1);
107:     unsigned NumLoads = DL.getTypeStoreSize(LI->getType());
108:     ConstantInt *CopyLen =
109:         ConstantInt::get(Type::getInt32Ty(Context), NumLoads);
110:
111:     createMemCpyLoopKnownSize(/* ConvertedInst */ SI,
112:                               /* SrcAddr */ SrcAddr, /* DstAddr */ DstAddr,
113:                               /* CopyLen */ CopyLen,
114:                               /* SrcAlign */ LI->getAlign(),
115:                               /* DestAlign */ SI->getAlign(),
116:                               /* SrcIsVolatile */ LI->isVolatile(),
117:                               /* DstIsVolatile */ SI->isVolatile(),
118:                               /* CanOverlap */ true, TTI);
119:
120:     SI->eraseFromParent();
```
- EN: This range implements operational logic in helpers such as push_back, user_begin, getOperand, getTypeStoreSize, translating backend policy into executable code.
- CN: 这一段实现了 push_back、user_begin、getOperand、getTypeStoreSize 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-147
```cpp
121:     LI->eraseFromParent();
122:   }
123:
124:   // Transform mem* intrinsic calls.
125:   for (MemIntrinsic *MemCall : MemCalls) {
126:     if (MemCpyInst *Memcpy = dyn_cast<MemCpyInst>(MemCall)) {
127:       expandMemCpyAsLoop(Memcpy, TTI);
128:     } else if (MemMoveInst *Memmove = dyn_cast<MemMoveInst>(MemCall)) {
129:       expandMemMoveAsLoop(Memmove, TTI);
130:     } else if (MemSetInst *Memset = dyn_cast<MemSetInst>(MemCall)) {
131:       expandMemSetAsLoop(Memset, TTI);
132:     }
133:     MemCall->eraseFromParent();
134:   }
135:
136:   return true;
137: }
138:
139: } // namespace
140:
141: INITIALIZE_PASS(NVPTXLowerAggrCopies, "nvptx-lower-aggr-copies",
142:                 "Lower aggregate copies, and llvm.mem* intrinsics into loops",
143:                 false, false)
144:
145: FunctionPass *llvm::createLowerAggrCopies() {
146:   return new NVPTXLowerAggrCopies();
147: }
```
- EN: This range implements operational logic in helpers such as eraseFromParent, expandMemCpyAsLoop, expandMemMoveAsLoop, expandMemSetAsLoop, translating backend policy into executable code.
- CN: 这一段实现了 eraseFromParent、expandMemCpyAsLoop、expandMemMoveAsLoop、expandMemSetAsLoop 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include NVPTXLowerAggrCopies, getAnalysisUsage, runOnFunction, getPassName, NVPTXLowerAggrCopies::runOnFunction, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 NVPTXLowerAggrCopies, getAnalysisUsage, runOnFunction, getPassName, NVPTXLowerAggrCopies::runOnFunction，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXLowerAggrCopies.h`
  - `NVPTX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/TargetTransformInfo.h`
  - `llvm/CodeGen/StackProtector.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DataLayout.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/LLVMContext.h`
  - `llvm/IR/Module.h`
  - `llvm/Transforms/Utils/BasicBlockUtils.h`
  - `llvm/Transforms/Utils/LowerMemIntrinsics.h`
