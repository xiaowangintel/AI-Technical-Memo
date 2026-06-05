# PointerTypeAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DirectXIRPasses/PointerTypeAnalysis.cpp`
- Repository: `llvm-project`
- Purpose (EN): Target DirectX PointerTypeAnalisis.cpp - PointerType analysis ------===.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- Target/DirectX/PointerTypeAnalisis.cpp - PointerType analysis ------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Analysis pass to assign types to opaque pointers.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "PointerTypeAnalysis.h"
14: #include "llvm/IR/Constants.h"
15: #include "llvm/IR/GlobalVariable.h"
16: #include "llvm/IR/Instructions.h"
17: #include "llvm/IR/Module.h"
18: #include "llvm/IR/Operator.h"
19:
20: using namespace llvm;
21: using namespace llvm::dxil;
22:
23: namespace {
24:
25: Type *classifyFunctionType(const Function &F, PointerTypeMap &Map);
26:
27: // Classifies the type of the value passed in by walking the value's users to
28: // find a typed instruction to materialize a type from.
29: Type *classifyPointerType(const Value *V, PointerTypeMap &Map) {
30:   assert(V->getType()->isPointerTy() &&
31:          "classifyPointerType called with non-pointer");
32:
33:   // A CallInst will trigger this case, and we want to classify its Function
34:   // operand as a Function rather than a generic Value.
35:   if (const Function *F = dyn_cast<Function>(V))
36:     return classifyFunctionType(*F, Map);
37:
38:   // There can potentially be dead constants hanging off of the globals we do
39:   // not want to deal with. So we remove them here.
40:   if (const GlobalVariable *GV = dyn_cast<GlobalVariable>(V))
```
- EN: This range implements operational logic in helpers such as classifyFunctionType, classifyPointerType, translating backend policy into executable code.
- CN: 这一段实现了 classifyFunctionType、classifyPointerType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:     GV->removeDeadConstantUsers();
42:
43:   auto It = Map.find(V);
44:   if (It != Map.end())
45:     return It->second;
46:
47:   Type *PointeeTy = nullptr;
48:   if (auto *GEP = dyn_cast<GEPOperator>(V)) {
49:     if (!GEP->getResultElementType()->isPointerTy())
50:       PointeeTy = GEP->getResultElementType();
51:   } else if (auto *Inst = dyn_cast<AllocaInst>(V)) {
52:     PointeeTy = Inst->getAllocatedType();
53:   } else if (auto *GV = dyn_cast<GlobalVariable>(V)) {
54:     PointeeTy = GV->getValueType();
55:   }
56:
57:   for (const auto *User : V->users()) {
58:     Type *NewPointeeTy = nullptr;
59:     if (const auto *Inst = dyn_cast<LoadInst>(User)) {
60:       NewPointeeTy = Inst->getType();
61:     } else if (const auto *Inst = dyn_cast<StoreInst>(User)) {
62:       NewPointeeTy = Inst->getValueOperand()->getType();
63:       // When store value is ptr type, cannot get more type info.
64:       if (NewPointeeTy->isPointerTy())
65:         continue;
66:     } else if (const auto *GEP = dyn_cast<GEPOperator>(User)) {
67:       NewPointeeTy = GEP->getSourceElementType();
68:     }
69:     if (NewPointeeTy) {
70:       // HLSL doesn't support pointers, so it is unlikely to get more than one
71:       // or two levels of indirection in the IR. Because of this, recursion is
72:       // pretty safe.
73:       if (NewPointeeTy->isPointerTy()) {
74:         PointeeTy = classifyPointerType(User, Map);
75:         break;
76:       }
77:       if (!PointeeTy)
78:         PointeeTy = NewPointeeTy;
79:       else if (PointeeTy != NewPointeeTy)
80:         PointeeTy = Type::getInt8Ty(V->getContext());
```
- EN: This range implements operational logic in helpers such as removeDeadConstantUsers, find, getResultElementType, getAllocatedType, translating backend policy into executable code.
- CN: 这一段实现了 removeDeadConstantUsers、find、getResultElementType、getAllocatedType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     }
 82:   }
 83:   // If we were unable to determine the pointee type, set to i8
 84:   // If we were able to determine the pointee type as ptr, set to i8*
 85:   if (!PointeeTy)
 86:     PointeeTy = Type::getInt8Ty(V->getContext());
 87:   if (PointeeTy->isPointerTy())
 88:     PointeeTy = TypedPointerType::get(Type::getInt8Ty(V->getContext()),
 89:                                       PointeeTy->getPointerAddressSpace());
 90:   auto *TypedPtrTy =
 91:       TypedPointerType::get(PointeeTy, V->getType()->getPointerAddressSpace());
 92:
 93:   Map[V] = TypedPtrTy;
 94:   return TypedPtrTy;
 95: }
 96:
 97: // This function constructs a function type accepting typed pointers. It only
 98: // handles function arguments and return types, and assigns the function type to
 99: // the function's value in the type map.
100: Type *classifyFunctionType(const Function &F, PointerTypeMap &Map) {
101:   auto It = Map.find(&F);
102:   if (It != Map.end())
103:     return It->second;
104:
105:   SmallVector<Type *, 8> NewArgs;
106:   Type *RetTy = F.getReturnType();
107:   LLVMContext &Ctx = F.getContext();
108:   if (RetTy->isPointerTy()) {
109:     RetTy = nullptr;
110:     for (const auto &B : F) {
111:       const auto *RetInst = dyn_cast_or_null<ReturnInst>(B.getTerminator());
112:       if (!RetInst)
113:         continue;
114:
115:       Type *NewRetTy = classifyPointerType(RetInst->getReturnValue(), Map);
116:       if (!RetTy)
117:         RetTy = NewRetTy;
118:       else if (RetTy != NewRetTy)
119:         RetTy = TypedPointerType::get(
120:             Type::getInt8Ty(Ctx), F.getReturnType()->getPointerAddressSpace());
```
- EN: This range implements operational logic in helpers such as Type::getInt8Ty, getPointerAddressSpace, TypedPointerType::get, classifyFunctionType, translating backend policy into executable code.
- CN: 这一段实现了 Type::getInt8Ty、getPointerAddressSpace、TypedPointerType::get、classifyFunctionType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:     }
122:     // For function decl.
123:     if (!RetTy)
124:       RetTy = TypedPointerType::get(
125:           Type::getInt8Ty(Ctx), F.getReturnType()->getPointerAddressSpace());
126:   }
127:   for (auto &A : F.args()) {
128:     Type *ArgTy = A.getType();
129:     if (ArgTy->isPointerTy())
130:       ArgTy = classifyPointerType(&A, Map);
131:     NewArgs.push_back(ArgTy);
132:   }
133:   auto *TypedPtrTy =
134:       TypedPointerType::get(FunctionType::get(RetTy, NewArgs, false), 0);
135:   Map[&F] = TypedPtrTy;
136:   return TypedPtrTy;
137: }
138: } // anonymous namespace
139:
140: static Type *classifyConstantWithOpaquePtr(const Constant *C,
141:                                            PointerTypeMap &Map) {
142:   // FIXME: support ConstantPointerNull which could map to more than one
143:   // TypedPointerType.
144:   // See https://github.com/llvm/llvm-project/issues/57942.
145:   if (isa<ConstantPointerNull>(C))
146:     return TypedPointerType::get(Type::getInt8Ty(C->getContext()),
147:                                  C->getType()->getPointerAddressSpace());
148:
149:   // Skip ConstantData which cannot have opaque ptr.
150:   if (isa<ConstantData>(C))
151:     return C->getType();
152:
153:   auto It = Map.find(C);
154:   if (It != Map.end())
155:     return It->second;
156:
157:   if (const auto *F = dyn_cast<Function>(C))
158:     return classifyFunctionType(*F, Map);
159:
160:   Type *Ty = C->getType();
```
- EN: This range implements operational logic in helpers such as Type::getInt8Ty, getType, classifyPointerType, push_back, translating backend policy into executable code.
- CN: 这一段实现了 Type::getInt8Ty、getType、classifyPointerType、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:   Type *TargetTy = nullptr;
162:   if (auto *CS = dyn_cast<ConstantStruct>(C)) {
163:     SmallVector<Type *> EltTys;
164:     for (unsigned int I = 0; I < CS->getNumOperands(); ++I) {
165:       const Constant *Elt = C->getAggregateElement(I);
166:       Type *EltTy = classifyConstantWithOpaquePtr(Elt, Map);
167:       EltTys.emplace_back(EltTy);
168:     }
169:     TargetTy = StructType::get(C->getContext(), EltTys);
170:   } else if (auto *CA = dyn_cast<ConstantAggregate>(C)) {
171:
172:     Type *TargetEltTy = nullptr;
173:     for (auto &Elt : CA->operands()) {
174:       Type *EltTy = classifyConstantWithOpaquePtr(cast<Constant>(&Elt), Map);
175:       assert(TargetEltTy == EltTy || TargetEltTy == nullptr);
176:       TargetEltTy = EltTy;
177:     }
178:
179:     if (auto *AT = dyn_cast<ArrayType>(Ty)) {
180:       TargetTy = ArrayType::get(TargetEltTy, AT->getNumElements());
181:     } else {
182:       // Not struct, not array, must be vector here.
183:       auto *VT = cast<VectorType>(Ty);
184:       TargetTy = VectorType::get(TargetEltTy, VT);
185:     }
186:   }
187:   // Must have a target ty when map.
188:   assert(TargetTy && "PointerTypeAnalyisis failed to identify target type");
189:
190:   // Same type, no need to map.
191:   if (TargetTy == Ty)
192:     return Ty;
193:
194:   Map[C] = TargetTy;
195:   return TargetTy;
196: }
197:
198: static void classifyGlobalCtorPointerType(const GlobalVariable &GV,
199:                                           PointerTypeMap &Map) {
200:   const auto *CA = dyn_cast<ConstantArray>(GV.getInitializer());
```
- EN: This range implements operational logic in helpers such as getNumOperands, getAggregateElement, classifyConstantWithOpaquePtr, emplace_back, translating backend policy into executable code.
- CN: 这一段实现了 getNumOperands、getAggregateElement、classifyConstantWithOpaquePtr、emplace_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-238
```cpp
201:   if (!CA) {
202:     // An empty global_ctors will be a zeroinitializer, so just skip it.
203:     assert(isa<ConstantAggregateZero>(GV.getInitializer()) &&
204:            "global_ctors should be a ConstantArray or ConstantAggregateZero");
205:     return;
206:   }
207:   // Type for global ctor should be array of { i32, void ()*, i8* }.
208:   Type *CtorArrayTy = classifyConstantWithOpaquePtr(CA, Map);
209:
210:   // Map the global type.
211:   Map[&GV] = TypedPointerType::get(CtorArrayTy,
212:                                    GV.getType()->getPointerAddressSpace());
213: }
214:
215: PointerTypeMap PointerTypeAnalysis::run(const Module &M) {
216:   PointerTypeMap Map;
217:   for (auto &G : M.globals()) {
218:     if (G.getType()->isPointerTy())
219:       classifyPointerType(&G, Map);
220:     if (G.getName() == "llvm.global_ctors")
221:       classifyGlobalCtorPointerType(G, Map);
222:   }
223:
224:   for (auto &F : M) {
225:     classifyFunctionType(F, Map);
226:
227:     for (const auto &B : F) {
228:       for (const auto &I : B) {
229:         if (I.getType()->isPointerTy())
230:           classifyPointerType(&I, Map);
231:         for (const auto &O : I.operands())
232:           if (O.get()->getType()->isPointerTy())
233:             classifyPointerType(O.get(), Map);
234:       }
235:     }
236:   }
237:   return Map;
238: }
```
- EN: This range implements operational logic in helpers such as classifyConstantWithOpaquePtr, getType, PointerTypeAnalysis::run, classifyPointerType, translating backend policy into executable code.
- CN: 这一段实现了 classifyConstantWithOpaquePtr、getType、PointerTypeAnalysis::run、classifyPointerType 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include classifyFunctionType, classifyPointerType, removeDeadConstantUsers, find, getResultElementType, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 classifyFunctionType, classifyPointerType, removeDeadConstantUsers, find, getResultElementType，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `PointerTypeAnalysis.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Constants.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/Operator.h`
