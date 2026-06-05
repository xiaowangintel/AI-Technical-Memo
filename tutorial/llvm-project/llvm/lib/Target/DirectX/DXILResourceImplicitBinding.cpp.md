# DXILResourceImplicitBinding.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILResourceImplicitBinding.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILResourceImplicitBinding support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILResourceImplicitBinding.cpp -----------------------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILResourceImplicitBinding.h"
10: #include "DirectX.h"
11: #include "llvm/ADT/APInt.h"
12: #include "llvm/ADT/STLExtras.h"
13: #include "llvm/Analysis/DXILResource.h"
14: #include "llvm/IR/Analysis.h"
15: #include "llvm/IR/Constants.h"
16: #include "llvm/IR/DiagnosticInfo.h"
17: #include "llvm/IR/Function.h"
18: #include "llvm/IR/IRBuilder.h"
19: #include "llvm/IR/Instructions.h"
20: #include "llvm/IR/IntrinsicsDirectX.h"
21: #include "llvm/IR/Module.h"
22: #include "llvm/InitializePasses.h"
23: #include <cstdint>
24:
25: #define DEBUG_TYPE "dxil-resource-implicit-binding"
26:
27: using namespace llvm;
28: using namespace llvm::dxil;
29:
30: namespace {
31:
32: static void diagnoseImplicitBindingNotFound(CallInst *ImplBindingCall) {
33:   Function *F = ImplBindingCall->getFunction();
34:   LLVMContext &Context = F->getParent()->getContext();
35:   // FIXME: include the name of the resource in the error message
36:   // (llvm/llvm-project#137868)
37:   Context.diagnose(
38:       DiagnosticInfoGenericWithLoc("resource cannot be allocated", *F,
39:                                    ImplBindingCall->getDebugLoc(), DS_Error));
40: }
```
- EN: This range implements operational logic in helpers such as diagnoseImplicitBindingNotFound, getFunction, getParent, getDebugLoc, translating backend policy into executable code.
- CN: 这一段实现了 diagnoseImplicitBindingNotFound、getFunction、getParent、getDebugLoc 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:
42: static bool assignBindings(Module &M, DXILResourceBindingInfo &DRBI,
43:                            DXILResourceTypeMap &DRTM) {
44:   struct ImplicitBindingCall {
45:     int OrderID;
46:     CallInst *Call;
47:     ImplicitBindingCall(int OrderID, CallInst *Call)
48:         : OrderID(OrderID), Call(Call) {}
49:   };
50:   SmallVector<ImplicitBindingCall> Calls;
51:   SmallVector<Function *> FunctionsToMaybeRemove;
52:
53:   // collect all of the llvm.dx.resource.handlefromImplicitbinding calls
54:   for (Function &F : M.functions()) {
55:     if (!F.isDeclaration())
56:       continue;
57:
58:     if (F.getIntrinsicID() != Intrinsic::dx_resource_handlefromimplicitbinding)
59:       continue;
60:
61:     for (User *U : F.users()) {
62:       if (CallInst *CI = dyn_cast<CallInst>(U)) {
63:         int OrderID = cast<ConstantInt>(CI->getArgOperand(0))->getZExtValue();
64:         Calls.emplace_back(OrderID, CI);
65:       }
66:     }
67:     FunctionsToMaybeRemove.emplace_back(&F);
68:   }
69:
70:   // sort all the collected implicit bindings by OrderID
71:   llvm::stable_sort(
72:       Calls, [](auto &LHS, auto &RHS) { return LHS.OrderID < RHS.OrderID; });
73:
74:   // iterate over sorted calls, find binding for each new OrderID and replace
75:   // each call with dx_resource_handlefrombinding using the new binding
76:   int LastOrderID = -1;
77:   llvm::TargetExtType *HandleTy = nullptr;
78:   ConstantInt *RegSlotOp = nullptr;
79:   bool AllBindingsAssigned = true;
80:   bool Changed = false;
```
- EN: This range defines or declares important types such as ImplicitBindingCall, OrderID, getArgOperand, emplace_back, shaping the data model used by DXILResourceImplicitBinding.cpp.
- CN: 这一段定义或声明了 ImplicitBindingCall、OrderID、getArgOperand、emplace_back 等关键类型，构成 DXILResourceImplicitBinding.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:
 82:   for (ImplicitBindingCall &IB : Calls) {
 83:     IRBuilder<> Builder(IB.Call);
 84:
 85:     if (IB.OrderID != LastOrderID) {
 86:       LastOrderID = IB.OrderID;
 87:       HandleTy = cast<TargetExtType>(IB.Call->getType());
 88:       ResourceTypeInfo &RTI = DRTM[HandleTy];
 89:
 90:       uint32_t Space =
 91:           cast<ConstantInt>(IB.Call->getArgOperand(1))->getZExtValue();
 92:       int32_t Size =
 93:           cast<ConstantInt>(IB.Call->getArgOperand(2))->getZExtValue();
 94:
 95:       std::optional<uint32_t> RegSlot =
 96:           DRBI.findAvailableBinding(RTI.getResourceClass(), Space, Size);
 97:       if (!RegSlot) {
 98:         diagnoseImplicitBindingNotFound(IB.Call);
 99:         AllBindingsAssigned = false;
100:         continue;
101:       }
102:       RegSlotOp = ConstantInt::get(Builder.getInt32Ty(), RegSlot.value());
103:     }
104:
105:     if (!RegSlotOp)
106:       continue;
107:
108:     auto *NewCall = Builder.CreateIntrinsic(
109:         HandleTy, Intrinsic::dx_resource_handlefrombinding,
110:         {IB.Call->getOperand(1),   /* space */
111:          RegSlotOp,                /* register slot */
112:          IB.Call->getOperand(2),   /* size */
113:          IB.Call->getOperand(3),   /* index */
114:          IB.Call->getOperand(4)}); /* name */
115:     IB.Call->replaceAllUsesWith(NewCall);
116:     IB.Call->eraseFromParent();
117:     Changed = true;
118:   }
119:
120:   for (Function *F : FunctionsToMaybeRemove) {
```
- EN: This range implements operational logic in helpers such as Builder, getType, getArgOperand, findAvailableBinding, translating backend policy into executable code.
- CN: 这一段实现了 Builder、getType、getArgOperand、findAvailableBinding 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:     if (F->user_empty()) {
122:       F->eraseFromParent();
123:       Changed = true;
124:     }
125:   }
126:
127:   DRBI.setHasImplicitBinding(!AllBindingsAssigned);
128:   return Changed;
129: }
130:
131: } // end anonymous namespace
132:
133: PreservedAnalyses DXILResourceImplicitBinding::run(Module &M,
134:                                                    ModuleAnalysisManager &AM) {
135:
136:   DXILResourceBindingInfo &DRBI = AM.getResult<DXILResourceBindingAnalysis>(M);
137:   DXILResourceTypeMap &DRTM = AM.getResult<DXILResourceTypeAnalysis>(M);
138:
139:   if (!DRBI.hasImplicitBinding())
140:     return PreservedAnalyses::all();
141:
142:   if (!assignBindings(M, DRBI, DRTM))
143:     return PreservedAnalyses::all();
144:
145:   PreservedAnalyses PA;
146:   PA.preserve<DXILResourceBindingAnalysis>();
147:   PA.preserve<DXILResourceTypeAnalysis>();
148:   return PA;
149: }
150:
151: namespace {
152:
153: class DXILResourceImplicitBindingLegacy : public ModulePass {
154: public:
155:   DXILResourceImplicitBindingLegacy() : ModulePass(ID) {}
156:
157:   bool runOnModule(Module &M) override {
158:     DXILResourceTypeMap &DRTM =
159:         getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
160:     DXILResourceBindingInfo &DRBI =
```
- EN: This range defines or declares important types such as eraseFromParent, setHasImplicitBinding, PreservedAnalyses::all, DXILResourceImplicitBindingLegacy, shaping the data model used by DXILResourceImplicitBinding.cpp.
- CN: 这一段定义或声明了 eraseFromParent、setHasImplicitBinding、PreservedAnalyses::all、DXILResourceImplicitBindingLegacy 等关键类型，构成 DXILResourceImplicitBinding.cpp 使用的数据模型。

### Lines 161-189
```cpp
161:         getAnalysis<DXILResourceBindingWrapperPass>().getBindingInfo();
162:
163:     if (DRBI.hasImplicitBinding())
164:       return assignBindings(M, DRBI, DRTM);
165:     return false;
166:   }
167:
168:   static char ID; // Pass identification.
169:   void getAnalysisUsage(llvm::AnalysisUsage &AU) const override {
170:     AU.addRequired<DXILResourceTypeWrapperPass>();
171:     AU.addRequired<DXILResourceBindingWrapperPass>();
172:     AU.addPreserved<DXILResourceTypeWrapperPass>();
173:     AU.addPreserved<DXILResourceBindingWrapperPass>();
174:   }
175: };
176:
177: char DXILResourceImplicitBindingLegacy::ID = 0;
178: } // end anonymous namespace
179:
180: INITIALIZE_PASS_BEGIN(DXILResourceImplicitBindingLegacy, DEBUG_TYPE,
181:                       "DXIL Resource Implicit Binding", false, false)
182: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
183: INITIALIZE_PASS_DEPENDENCY(DXILResourceBindingWrapperPass)
184: INITIALIZE_PASS_END(DXILResourceImplicitBindingLegacy, DEBUG_TYPE,
185:                     "DXIL Resource Implicit Binding", false, false)
186:
187: ModulePass *llvm::createDXILResourceImplicitBindingLegacyPass() {
188:   return new DXILResourceImplicitBindingLegacy();
189: }
```
- EN: This range implements operational logic in helpers such as getBindingInfo, assignBindings, getAnalysisUsage, INITIALIZE_PASS_DEPENDENCY, translating backend policy into executable code.
- CN: 这一段实现了 getBindingInfo、assignBindings、getAnalysisUsage、INITIALIZE_PASS_DEPENDENCY 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include diagnoseImplicitBindingNotFound, getFunction, getParent, getDebugLoc, ImplicitBindingCall, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 diagnoseImplicitBindingNotFound, getFunction, getParent, getDebugLoc, ImplicitBindingCall，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILResourceImplicitBinding.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APInt.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/IR/Analysis.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
- System/standard headers / 系统或标准头文件:
  - `cstdint`
