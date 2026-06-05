# NVPTXGenericToNVVM.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXGenericToNVVM.cpp`
- Repository: `llvm-project`
- Purpose (EN): GenericToNVVM.cpp - Convert generic module to NVVM module - C++ -*-===.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- GenericToNVVM.cpp - Convert generic module to NVVM module - C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // Convert generic global variables into either .global or .const access based
10: // on the variable's "constant" qualifier.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "MCTargetDesc/NVPTXBaseInfo.h"
15: #include "NVPTX.h"
16: #include "NVVMProperties.h"
17: #include "llvm/CodeGen/ValueTypes.h"
18: #include "llvm/IR/Constants.h"
19: #include "llvm/IR/DerivedTypes.h"
20: #include "llvm/IR/IRBuilder.h"
21: #include "llvm/IR/Instructions.h"
22: #include "llvm/IR/Intrinsics.h"
23: #include "llvm/IR/LegacyPassManager.h"
24: #include "llvm/IR/Module.h"
25: #include "llvm/IR/Operator.h"
26: #include "llvm/IR/ValueMap.h"
27: #include "llvm/Transforms/Utils/ValueMapper.h"
28:
29: using namespace llvm;
30:
31: namespace {
32: class GenericToNVVM {
33: public:
34:   bool runOnModule(Module &M);
35:
36: private:
37:   Value *remapConstant(Module *M, Function *F, Constant *C,
38:                        IRBuilder<> &Builder);
39:   Value *remapConstantVectorOrConstantAggregate(Module *M, Function *F,
40:                                                 Constant *C,
```
- EN: This range defines or declares important types such as GenericToNVVM, runOnModule, shaping the data model used by NVPTXGenericToNVVM.cpp.
- CN: 这一段定义或声明了 GenericToNVVM、runOnModule 等关键类型，构成 NVPTXGenericToNVVM.cpp 使用的数据模型。

### Lines 41-80
```cpp
41:                                                 IRBuilder<> &Builder);
42:   Value *remapConstantExpr(Module *M, Function *F, ConstantExpr *C,
43:                            IRBuilder<> &Builder);
44:
45:   typedef ValueMap<GlobalVariable *, GlobalVariable *> GVMapTy;
46:   typedef ValueMap<Constant *, Value *> ConstantToValueMapTy;
47:   GVMapTy GVMap;
48:   ConstantToValueMapTy ConstantToValueMap;
49: };
50: } // end namespace
51:
52: bool GenericToNVVM::runOnModule(Module &M) {
53:   // Create a clone of each global variable that has the default address space.
54:   // The clone is created with the global address space  specifier, and the pair
55:   // of original global variable and its clone is placed in the GVMap for later
56:   // use.
57:
58:   for (GlobalVariable &GV : llvm::make_early_inc_range(M.globals())) {
59:     if (GV.getType()->getAddressSpace() == llvm::ADDRESS_SPACE_GENERIC &&
60:         llvm::getPTXOpaqueType(GV) == llvm::PTXOpaqueType::None &&
61:         !GV.getName().starts_with("llvm.")) {
62:       GlobalVariable *NewGV = new GlobalVariable(
63:           M, GV.getValueType(), GV.isConstant(), GV.getLinkage(),
64:           GV.hasInitializer() ? GV.getInitializer() : nullptr, "", &GV,
65:           GV.getThreadLocalMode(), llvm::ADDRESS_SPACE_GLOBAL);
66:       NewGV->copyAttributesFrom(&GV);
67:       NewGV->copyMetadata(&GV, /*Offset=*/0);
68:       GVMap[&GV] = NewGV;
69:     }
70:   }
71:
72:   // Return immediately, if every global variable has a specific address space
73:   // specifier.
74:   if (GVMap.empty()) {
75:     return false;
76:   }
77:
78:   // Walk through the instructions in function defitinions, and replace any use
79:   // of original global variables in GVMap with a use of the corresponding
80:   // copies in GVMap.  If necessary, promote constants to instructions.
```
- EN: This range implements operational logic in helpers such as GenericToNVVM::runOnModule, getName, getThreadLocalMode, copyAttributesFrom, translating backend policy into executable code.
- CN: 这一段实现了 GenericToNVVM::runOnModule、getName、getThreadLocalMode、copyAttributesFrom 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   for (Function &F : M) {
 82:     if (F.isDeclaration()) {
 83:       continue;
 84:     }
 85:     IRBuilder<> Builder(&*F.getEntryBlock().getFirstNonPHIOrDbg());
 86:     for (BasicBlock &BB : F) {
 87:       for (Instruction &II : BB) {
 88:         for (unsigned i = 0, e = II.getNumOperands(); i < e; ++i) {
 89:           Value *Operand = II.getOperand(i);
 90:           if (isa<Constant>(Operand)) {
 91:             II.setOperand(
 92:                 i, remapConstant(&M, &F, cast<Constant>(Operand), Builder));
 93:           }
 94:         }
 95:       }
 96:     }
 97:     ConstantToValueMap.clear();
 98:   }
 99:
100:   // Copy GVMap over to a standard value map.
101:   ValueToValueMapTy VM;
102:   for (auto I = GVMap.begin(), E = GVMap.end(); I != E; ++I)
103:     VM[I->first] = I->second;
104:
105:   // Walk through the global variable  initializers, and replace any use of
106:   // original global variables in GVMap with a use of the corresponding copies
107:   // in GVMap.  The copies need to be bitcast to the original global variable
108:   // types, as we cannot use cvta in global variable initializers.
109:   for (GVMapTy::iterator I = GVMap.begin(), E = GVMap.end(); I != E;) {
110:     GlobalVariable *GV = I->first;
111:     GlobalVariable *NewGV = I->second;
112:
113:     // Remove GV from the map so that it can be RAUWed.  Note that
114:     // DenseMap::erase() won't invalidate any iterators but this one.
115:     auto Next = std::next(I);
116:     GVMap.erase(I);
117:     I = Next;
118:
119:     Constant *BitCastNewGV = ConstantExpr::getPointerCast(NewGV, GV->getType());
120:     // At this point, the remaining uses of GV should be found only in global
```
- EN: This range implements operational logic in helpers such as Builder, getOperand, remapConstant, clear, translating backend policy into executable code.
- CN: 这一段实现了 Builder、getOperand、remapConstant、clear 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:     // variable initializers, as other uses have been already been removed
122:     // while walking through the instructions in function definitions.
123:     GV->replaceAllUsesWith(BitCastNewGV);
124:     std::string Name = std::string(GV->getName());
125:     GV->eraseFromParent();
126:     NewGV->setName(Name);
127:   }
128:   assert(GVMap.empty() && "Expected it to be empty by now");
129:
130:   return true;
131: }
132:
133: Value *GenericToNVVM::remapConstant(Module *M, Function *F, Constant *C,
134:                                     IRBuilder<> &Builder) {
135:   // If the constant C has been converted already in the given function  F, just
136:   // return the converted value.
137:   ConstantToValueMapTy::iterator CTII = ConstantToValueMap.find(C);
138:   if (CTII != ConstantToValueMap.end()) {
139:     return CTII->second;
140:   }
141:
142:   Value *NewValue = C;
143:   if (isa<GlobalVariable>(C)) {
144:     // If the constant C is a global variable and is found in GVMap, substitute
145:     //
146:     //   addrspacecast GVMap[C] to addrspace(0)
147:     //
148:     // for our use of C.
149:     GVMapTy::iterator I = GVMap.find(cast<GlobalVariable>(C));
150:     if (I != GVMap.end()) {
151:       GlobalVariable *GV = I->second;
152:       NewValue = Builder.CreateAddrSpaceCast(
153:           GV, PointerType::get(GV->getContext(), llvm::ADDRESS_SPACE_GENERIC));
154:     }
155:   } else if (isa<ConstantAggregate>(C)) {
156:     // If any element in the constant vector or aggregate C is or uses a global
157:     // variable in GVMap, the constant C needs to be reconstructed, using a set
158:     // of instructions.
159:     NewValue = remapConstantVectorOrConstantAggregate(M, F, C, Builder);
160:   } else if (isa<ConstantExpr>(C)) {
```
- EN: This range implements operational logic in helpers such as replaceAllUsesWith, std::string, eraseFromParent, setName, translating backend policy into executable code.
- CN: 这一段实现了 replaceAllUsesWith、std::string、eraseFromParent、setName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:     // If any operand in the constant expression C is or uses a global variable
162:     // in GVMap, the constant expression C needs to be reconstructed, using a
163:     // set of instructions.
164:     NewValue = remapConstantExpr(M, F, cast<ConstantExpr>(C), Builder);
165:   }
166:
167:   ConstantToValueMap[C] = NewValue;
168:   return NewValue;
169: }
170:
171: Value *GenericToNVVM::remapConstantVectorOrConstantAggregate(
172:     Module *M, Function *F, Constant *C, IRBuilder<> &Builder) {
173:   bool OperandChanged = false;
174:   SmallVector<Value *, 4> NewOperands;
175:   unsigned NumOperands = C->getNumOperands();
176:
177:   // Check if any element is or uses a global variable in  GVMap, and thus
178:   // converted to another value.
179:   for (unsigned i = 0; i < NumOperands; ++i) {
180:     Value *Operand = C->getOperand(i);
181:     Value *NewOperand = remapConstant(M, F, cast<Constant>(Operand), Builder);
182:     OperandChanged |= Operand != NewOperand;
183:     NewOperands.push_back(NewOperand);
184:   }
185:
186:   // If none of the elements has been modified, return C as it is.
187:   if (!OperandChanged) {
188:     return C;
189:   }
190:
191:   // If any of the elements has been  modified, construct the equivalent
192:   // vector or aggregate value with a set instructions and the converted
193:   // elements.
194:   Value *NewValue = PoisonValue::get(C->getType());
195:   if (isa<ConstantVector>(C)) {
196:     for (unsigned i = 0; i < NumOperands; ++i) {
197:       Value *Idx = ConstantInt::get(Type::getInt32Ty(M->getContext()), i);
198:       NewValue = Builder.CreateInsertElement(NewValue, NewOperands[i], Idx);
199:     }
200:   } else {
```
- EN: This range implements operational logic in helpers such as remapConstantExpr, getNumOperands, getOperand, remapConstant, translating backend policy into executable code.
- CN: 这一段实现了 remapConstantExpr、getNumOperands、getOperand、remapConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:     for (unsigned i = 0; i < NumOperands; ++i) {
202:       NewValue =
203:           Builder.CreateInsertValue(NewValue, NewOperands[i], ArrayRef(i));
204:     }
205:   }
206:
207:   return NewValue;
208: }
209:
210: Value *GenericToNVVM::remapConstantExpr(Module *M, Function *F, ConstantExpr *C,
211:                                         IRBuilder<> &Builder) {
212:   bool OperandChanged = false;
213:   SmallVector<Value *, 4> NewOperands;
214:   unsigned NumOperands = C->getNumOperands();
215:
216:   // Check if any operand is or uses a global variable in  GVMap, and thus
217:   // converted to another value.
218:   for (unsigned i = 0; i < NumOperands; ++i) {
219:     Value *Operand = C->getOperand(i);
220:     Value *NewOperand = remapConstant(M, F, cast<Constant>(Operand), Builder);
221:     OperandChanged |= Operand != NewOperand;
222:     NewOperands.push_back(NewOperand);
223:   }
224:
225:   // If none of the operands has been modified, return C as it is.
226:   if (!OperandChanged) {
227:     return C;
228:   }
229:
230:   // If any of the operands has been modified, construct the instruction with
231:   // the converted operands.
232:   unsigned Opcode = C->getOpcode();
233:   switch (Opcode) {
234:   case Instruction::ExtractElement:
235:     // ExtractElementConstantExpr
236:     return Builder.CreateExtractElement(NewOperands[0], NewOperands[1]);
237:   case Instruction::InsertElement:
238:     // InsertElementConstantExpr
239:     return Builder.CreateInsertElement(NewOperands[0], NewOperands[1],
240:                                        NewOperands[2]);
```
- EN: This range implements operational logic in helpers such as CreateInsertValue, getNumOperands, getOperand, remapConstant, translating backend policy into executable code.
- CN: 这一段实现了 CreateInsertValue、getNumOperands、getOperand、remapConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-280
```cpp
241:   case Instruction::ShuffleVector:
242:     // ShuffleVector
243:     return Builder.CreateShuffleVector(NewOperands[0], NewOperands[1],
244:                                        NewOperands[2]);
245:   case Instruction::GetElementPtr:
246:     // GetElementPtrConstantExpr
247:     return Builder.CreateGEP(cast<GEPOperator>(C)->getSourceElementType(),
248:                              NewOperands[0],
249:                              ArrayRef(&NewOperands[1], NumOperands - 1), "",
250:                              cast<GEPOperator>(C)->isInBounds());
251:   case Instruction::Select:
252:     // SelectConstantExpr
253:     return Builder.CreateSelect(NewOperands[0], NewOperands[1], NewOperands[2]);
254:   default:
255:     // BinaryConstantExpr
256:     if (Instruction::isBinaryOp(Opcode)) {
257:       return Builder.CreateBinOp(Instruction::BinaryOps(C->getOpcode()),
258:                                  NewOperands[0], NewOperands[1]);
259:     }
260:     // UnaryConstantExpr
261:     if (Instruction::isCast(Opcode)) {
262:       return Builder.CreateCast(Instruction::CastOps(C->getOpcode()),
263:                                 NewOperands[0], C->getType());
264:     }
265:     llvm_unreachable("GenericToNVVM encountered an unsupported ConstantExpr");
266:   }
267: }
268:
269: namespace {
270: class GenericToNVVMLegacyPass : public ModulePass {
271: public:
272:   static char ID;
273:
274:   GenericToNVVMLegacyPass() : ModulePass(ID) {}
275:
276:   bool runOnModule(Module &M) override;
277: };
278: } // namespace
279:
280: char GenericToNVVMLegacyPass::ID = 0;
```
- EN: This range defines or declares important types such as isInBounds, CreateSelect, getType, llvm_unreachable, shaping the data model used by NVPTXGenericToNVVM.cpp.
- CN: 这一段定义或声明了 isInBounds、CreateSelect、getType、llvm_unreachable 等关键类型，构成 NVPTXGenericToNVVM.cpp 使用的数据模型。

### Lines 281-298
```cpp
281:
282: ModulePass *llvm::createGenericToNVVMLegacyPass() {
283:   return new GenericToNVVMLegacyPass();
284: }
285:
286: INITIALIZE_PASS(
287:     GenericToNVVMLegacyPass, "generic-to-nvvm",
288:     "Ensure that the global variables are in the global address space", false,
289:     false)
290:
291: bool GenericToNVVMLegacyPass::runOnModule(Module &M) {
292:   return GenericToNVVM().runOnModule(M);
293: }
294:
295: PreservedAnalyses GenericToNVVMPass::run(Module &M, ModuleAnalysisManager &AM) {
296:   return GenericToNVVM().runOnModule(M) ? PreservedAnalyses::none()
297:                                         : PreservedAnalyses::all();
298: }
```
- EN: This range implements operational logic in helpers such as llvm::createGenericToNVVMLegacyPass, GenericToNVVMLegacyPass, GenericToNVVMLegacyPass::runOnModule, GenericToNVVM, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createGenericToNVVMLegacyPass、GenericToNVVMLegacyPass、GenericToNVVMLegacyPass::runOnModule、GenericToNVVM 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include GenericToNVVM, runOnModule, GenericToNVVM::runOnModule, getName, getThreadLocalMode, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 GenericToNVVM, runOnModule, GenericToNVVM::runOnModule, getName, getThreadLocalMode，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `NVPTX.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/ValueTypes.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DerivedTypes.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/LegacyPassManager.h`
  - `llvm/IR/Module.h`
  - `llvm/IR/Operator.h`
  - `llvm/IR/ValueMap.h`
  - `llvm/Transforms/Utils/ValueMapper.h`
