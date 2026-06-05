# SPIRVLegalizeImplicitBinding.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVLegalizeImplicitBinding.cpp`
- Repository: `llvm-project`
- Purpose (EN): This pass legalizes the @llvm.spv.resource.handlefromimplicitbinding intrinsic by replacing it with a call to @llvm.spv.resource.handlefrombinding.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- SPIRVLegalizeImplicitBinding.cpp - Legalize implicit bindings ----*- C++
 2: //-*-===//
 3: //
 4: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 5: // See https://llvm.org/LICENSE.txt for license information.
 6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 7: //
 8: //===----------------------------------------------------------------------===//
 9: //
10: // This pass legalizes the @llvm.spv.resource.handlefromimplicitbinding
11: // intrinsic by replacing it with a call to
12: // @llvm.spv.resource.handlefrombinding.
13: //
14: //===----------------------------------------------------------------------===//
15:
16: #include "SPIRVLegalizeImplicitBinding.h"
17: #include "SPIRV.h"
18: #include "llvm/ADT/BitVector.h"
19: #include "llvm/ADT/SmallVector.h"
20: #include "llvm/IR/IRBuilder.h"
21: #include "llvm/IR/InstVisitor.h"
22: #include "llvm/IR/Intrinsics.h"
23: #include "llvm/IR/IntrinsicsSPIRV.h"
24: #include "llvm/IR/Module.h"
25: #include "llvm/Pass.h"
26: #include <algorithm>
27: #include <vector>
28:
29: using namespace llvm;
30:
31: namespace {
32: class SPIRVLegalizeImplicitBindingImpl {
33: public:
34:   bool runOnModule(Module &M);
35:
36: private:
37:   void collectBindingInfo(Module &M);
38:   uint32_t getAndReserveFirstUnusedBinding(uint32_t DescSet);
39:   void replaceImplicitBindingCalls(Module &M);
40:   void replaceResourceHandleCall(Module &M, CallInst *OldCI,
```
- EN: This range defines or declares important types such as SPIRVLegalizeImplicitBindingImpl, runOnModule, collectBindingInfo, getAndReserveFirstUnusedBinding, shaping the data model used by SPIRVLegalizeImplicitBinding.cpp.
- CN: 这一段定义或声明了 SPIRVLegalizeImplicitBindingImpl、runOnModule、collectBindingInfo、getAndReserveFirstUnusedBinding 等关键类型，构成 SPIRVLegalizeImplicitBinding.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:                                  uint32_t NewBinding);
42:   void replaceCounterHandleCall(Module &M, CallInst *OldCI,
43:                                 uint32_t NewBinding);
44:   void verifyUniqueOrderIdPerResource(SmallVectorImpl<CallInst *> &Calls);
45:
46:   // A map from descriptor set to a bit vector of used binding numbers.
47:   std::vector<BitVector> UsedBindings;
48:   // A list of all implicit binding calls, to be sorted by order ID.
49:   SmallVector<CallInst *, 16> ImplicitBindingCalls;
50: };
51:
52: class SPIRVLegalizeImplicitBindingLegacy : public ModulePass {
53: public:
54:   static char ID;
55:   SPIRVLegalizeImplicitBindingLegacy() : ModulePass(ID) {}
56:   StringRef getPassName() const override {
57:     return "SPIRV Legalize Implicit Binding";
58:   }
59:   bool runOnModule(Module &M) override {
60:     return SPIRVLegalizeImplicitBindingImpl().runOnModule(M);
61:   }
62: };
63:
64: struct BindingInfoCollector : public InstVisitor<BindingInfoCollector> {
65:   std::vector<BitVector> &UsedBindings;
66:   SmallVector<CallInst *, 16> &ImplicitBindingCalls;
67:
68:   BindingInfoCollector(std::vector<BitVector> &UsedBindings,
69:                        SmallVector<CallInst *, 16> &ImplicitBindingCalls)
70:       : UsedBindings(UsedBindings), ImplicitBindingCalls(ImplicitBindingCalls) {
71:   }
72:
73:   void addBinding(uint32_t DescSet, uint32_t Binding) {
74:     if (UsedBindings.size() <= DescSet) {
75:       UsedBindings.resize(DescSet + 1);
76:       UsedBindings[DescSet].resize(64);
77:     }
78:     if (UsedBindings[DescSet].size() <= Binding) {
79:       UsedBindings[DescSet].resize(2 * Binding + 1);
80:     }
```
- EN: This range defines or declares important types such as verifyUniqueOrderIdPerResource, SPIRVLegalizeImplicitBindingLegacy, getPassName, runOnModule, shaping the data model used by SPIRVLegalizeImplicitBinding.cpp.
- CN: 这一段定义或声明了 verifyUniqueOrderIdPerResource、SPIRVLegalizeImplicitBindingLegacy、getPassName、runOnModule 等关键类型，构成 SPIRVLegalizeImplicitBinding.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:     UsedBindings[DescSet].set(Binding);
 82:   }
 83:
 84:   void visitCallInst(CallInst &CI) {
 85:     if (CI.getIntrinsicID() == Intrinsic::spv_resource_handlefrombinding) {
 86:       const uint32_t DescSet =
 87:           cast<ConstantInt>(CI.getArgOperand(0))->getZExtValue();
 88:       const uint32_t Binding =
 89:           cast<ConstantInt>(CI.getArgOperand(1))->getZExtValue();
 90:       addBinding(DescSet, Binding);
 91:     } else if (CI.getIntrinsicID() ==
 92:                Intrinsic::spv_resource_handlefromimplicitbinding) {
 93:       ImplicitBindingCalls.push_back(&CI);
 94:     } else if (CI.getIntrinsicID() ==
 95:                Intrinsic::spv_resource_counterhandlefrombinding) {
 96:       const uint32_t DescSet =
 97:           cast<ConstantInt>(CI.getArgOperand(2))->getZExtValue();
 98:       const uint32_t Binding =
 99:           cast<ConstantInt>(CI.getArgOperand(1))->getZExtValue();
100:       addBinding(DescSet, Binding);
101:     } else if (CI.getIntrinsicID() ==
102:                Intrinsic::spv_resource_counterhandlefromimplicitbinding) {
103:       ImplicitBindingCalls.push_back(&CI);
104:     }
105:   }
106: };
107:
108: static uint32_t getOrderId(const CallInst *CI) {
109:   uint32_t OrderIdArgIdx = 0;
110:   switch (CI->getIntrinsicID()) {
111:   case Intrinsic::spv_resource_handlefromimplicitbinding:
112:     OrderIdArgIdx = 0;
113:     break;
114:   case Intrinsic::spv_resource_counterhandlefromimplicitbinding:
115:     OrderIdArgIdx = 1;
116:     break;
117:   default:
118:     llvm_unreachable("CallInst is not an implicit binding intrinsic");
119:   }
120:   return cast<ConstantInt>(CI->getArgOperand(OrderIdArgIdx))->getZExtValue();
```
- EN: This range implements operational logic in helpers such as set, visitCallInst, getArgOperand, addBinding, translating backend policy into executable code.
- CN: 这一段实现了 set、visitCallInst、getArgOperand、addBinding 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121: }
122:
123: static uint32_t getDescSet(const CallInst *CI) {
124:   uint32_t DescSetArgIdx;
125:   switch (CI->getIntrinsicID()) {
126:   case Intrinsic::spv_resource_handlefromimplicitbinding:
127:   case Intrinsic::spv_resource_handlefrombinding:
128:     DescSetArgIdx = 1;
129:     break;
130:   case Intrinsic::spv_resource_counterhandlefromimplicitbinding:
131:   case Intrinsic::spv_resource_counterhandlefrombinding:
132:     DescSetArgIdx = 2;
133:     break;
134:   default:
135:     llvm_unreachable("CallInst is not an implicit binding intrinsic");
136:   }
137:   return cast<ConstantInt>(CI->getArgOperand(DescSetArgIdx))->getZExtValue();
138: }
139:
140: void SPIRVLegalizeImplicitBindingImpl::collectBindingInfo(Module &M) {
141:   BindingInfoCollector InfoCollector(UsedBindings, ImplicitBindingCalls);
142:   InfoCollector.visit(M);
143:
144:   // Sort the collected calls by their order ID.
145:   std::sort(ImplicitBindingCalls.begin(), ImplicitBindingCalls.end(),
146:             [](const CallInst *A, const CallInst *B) {
147:               return getOrderId(A) < getOrderId(B);
148:             });
149: }
150:
151: void SPIRVLegalizeImplicitBindingImpl::verifyUniqueOrderIdPerResource(
152:     SmallVectorImpl<CallInst *> &Calls) {
153:   // Check that the order Id is unique per resource.
154:   for (uint32_t i = 1; i < Calls.size(); ++i) {
155:     const uint32_t OrderA = getOrderId(Calls[i - 1]);
156:     const uint32_t OrderB = getOrderId(Calls[i]);
157:     if (OrderA == OrderB) {
158:       const uint32_t DescSetA = getDescSet(Calls[i - 1]);
159:       const uint32_t DescSetB = getDescSet(Calls[i]);
160:       if (DescSetA != DescSetB) {
```
- EN: This range implements operational logic in helpers such as getDescSet, llvm_unreachable, getArgOperand, SPIRVLegalizeImplicitBindingImpl::collectBindingInfo, translating backend policy into executable code.
- CN: 这一段实现了 getDescSet、llvm_unreachable、getArgOperand、SPIRVLegalizeImplicitBindingImpl::collectBindingInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:         report_fatal_error("Implicit binding calls with the same order ID must "
162:                            "have the same descriptor set");
163:       }
164:     }
165:   }
166: }
167:
168: uint32_t SPIRVLegalizeImplicitBindingImpl::getAndReserveFirstUnusedBinding(
169:     uint32_t DescSet) {
170:   if (UsedBindings.size() <= DescSet) {
171:     UsedBindings.resize(DescSet + 1);
172:     UsedBindings[DescSet].resize(64);
173:   }
174:
175:   int NewBinding = UsedBindings[DescSet].find_first_unset();
176:   if (NewBinding == -1) {
177:     NewBinding = UsedBindings[DescSet].size();
178:     UsedBindings[DescSet].resize(2 * NewBinding + 1);
179:   }
180:
181:   UsedBindings[DescSet].set(NewBinding);
182:   return NewBinding;
183: }
184:
185: void SPIRVLegalizeImplicitBindingImpl::replaceImplicitBindingCalls(Module &M) {
186:   uint32_t lastOrderId = -1;
187:   uint32_t lastBindingNumber = -1;
188:
189:   for (CallInst *OldCI : ImplicitBindingCalls) {
190:     const uint32_t OrderId = getOrderId(OldCI);
191:     uint32_t BindingNumber;
192:     if (OrderId == lastOrderId) {
193:       BindingNumber = lastBindingNumber;
194:     } else {
195:       const uint32_t DescSet = getDescSet(OldCI);
196:       BindingNumber = getAndReserveFirstUnusedBinding(DescSet);
197:     }
198:
199:     if (OldCI->getIntrinsicID() ==
200:         Intrinsic::spv_resource_handlefromimplicitbinding) {
```
- EN: This range implements operational logic in helpers such as resize, find_first_unset, size, set, translating backend policy into executable code.
- CN: 这一段实现了 resize、find_first_unset、size、set 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:       replaceResourceHandleCall(M, OldCI, BindingNumber);
202:     } else {
203:       assert(OldCI->getIntrinsicID() ==
204:                  Intrinsic::spv_resource_counterhandlefromimplicitbinding &&
205:              "Unexpected implicit binding intrinsic");
206:       replaceCounterHandleCall(M, OldCI, BindingNumber);
207:     }
208:     lastOrderId = OrderId;
209:     lastBindingNumber = BindingNumber;
210:   }
211: }
212:
213: bool SPIRVLegalizeImplicitBindingImpl::runOnModule(Module &M) {
214:   collectBindingInfo(M);
215:   if (ImplicitBindingCalls.empty()) {
216:     return false;
217:   }
218:   verifyUniqueOrderIdPerResource(ImplicitBindingCalls);
219:
220:   replaceImplicitBindingCalls(M);
221:   return true;
222: }
223: } // namespace
224:
225: PreservedAnalyses SPIRVLegalizeImplicitBinding::run(Module &M,
226:                                                     ModuleAnalysisManager &AM) {
227:   return SPIRVLegalizeImplicitBindingImpl().runOnModule(M)
228:              ? PreservedAnalyses::none()
229:              : PreservedAnalyses::all();
230: }
231:
232: char SPIRVLegalizeImplicitBindingLegacy::ID = 0;
233:
234: INITIALIZE_PASS(SPIRVLegalizeImplicitBindingLegacy,
235:                 "legalize-spirv-implicit-binding",
236:                 "Legalize SPIR-V implicit bindings", false, false)
237:
238: ModulePass *llvm::createSPIRVLegalizeImplicitBindingPass() {
239:   return new SPIRVLegalizeImplicitBindingLegacy();
240: }
```
- EN: This range implements operational logic in helpers such as replaceResourceHandleCall, replaceCounterHandleCall, SPIRVLegalizeImplicitBindingImpl::runOnModule, collectBindingInfo, translating backend policy into executable code.
- CN: 这一段实现了 replaceResourceHandleCall、replaceCounterHandleCall、SPIRVLegalizeImplicitBindingImpl::runOnModule、collectBindingInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-280
```cpp
241:
242: void SPIRVLegalizeImplicitBindingImpl::replaceResourceHandleCall(
243:     Module &M, CallInst *OldCI, uint32_t NewBinding) {
244:   IRBuilder<> Builder(OldCI);
245:   const uint32_t DescSet =
246:       cast<ConstantInt>(OldCI->getArgOperand(1))->getZExtValue();
247:
248:   SmallVector<Value *, 8> Args;
249:   Args.push_back(Builder.getInt32(DescSet));
250:   Args.push_back(Builder.getInt32(NewBinding));
251:
252:   // Copy the remaining arguments from the old call.
253:   for (uint32_t i = 2; i < OldCI->arg_size(); ++i) {
254:     Args.push_back(OldCI->getArgOperand(i));
255:   }
256:
257:   Function *NewFunc = Intrinsic::getOrInsertDeclaration(
258:       &M, Intrinsic::spv_resource_handlefrombinding, OldCI->getType());
259:   CallInst *NewCI = Builder.CreateCall(NewFunc, Args);
260:   NewCI->setCallingConv(OldCI->getCallingConv());
261:
262:   OldCI->replaceAllUsesWith(NewCI);
263:   OldCI->eraseFromParent();
264: }
265:
266: void SPIRVLegalizeImplicitBindingImpl::replaceCounterHandleCall(
267:     Module &M, CallInst *OldCI, uint32_t NewBinding) {
268:   IRBuilder<> Builder(OldCI);
269:   const uint32_t DescSet =
270:       cast<ConstantInt>(OldCI->getArgOperand(2))->getZExtValue();
271:
272:   SmallVector<Value *, 8> Args;
273:   Args.push_back(OldCI->getArgOperand(0));
274:   Args.push_back(Builder.getInt32(NewBinding));
275:   Args.push_back(Builder.getInt32(DescSet));
276:
277:   Type *Tys[] = {OldCI->getType(), OldCI->getArgOperand(0)->getType()};
278:   Function *NewFunc = Intrinsic::getOrInsertDeclaration(
279:       &M, Intrinsic::spv_resource_counterhandlefrombinding, Tys);
280:   CallInst *NewCI = Builder.CreateCall(NewFunc, Args);
```
- EN: This range implements operational logic in helpers such as Builder, getArgOperand, push_back, arg_size, translating backend policy into executable code.
- CN: 这一段实现了 Builder、getArgOperand、push_back、arg_size 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 281-285
```cpp
281:   NewCI->setCallingConv(OldCI->getCallingConv());
282:
283:   OldCI->replaceAllUsesWith(NewCI);
284:   OldCI->eraseFromParent();
285: }
```
- EN: This range implements operational logic in helpers such as setCallingConv, replaceAllUsesWith, eraseFromParent, translating backend policy into executable code.
- CN: 这一段实现了 setCallingConv、replaceAllUsesWith、eraseFromParent 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVLegalizeImplicitBindingImpl, runOnModule, collectBindingInfo, getAndReserveFirstUnusedBinding, replaceImplicitBindingCalls, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVLegalizeImplicitBindingImpl, runOnModule, collectBindingInfo, getAndReserveFirstUnusedBinding, replaceImplicitBindingCalls，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVLegalizeImplicitBinding.h`
  - `SPIRV.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/BitVector.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstVisitor.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/IR/Module.h`
  - `llvm/Pass.h`
- System/standard headers / 系统或标准头文件:
  - `algorithm`
  - `vector`
