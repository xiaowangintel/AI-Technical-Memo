# SPIRVLegalizeZeroSizeArrays.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVLegalizeZeroSizeArrays.cpp`
- Repository: `llvm-project`
- Purpose (EN): SPIR-V does not support zero-size arrays unless it is within a shader.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- SPIRVLegalizeZeroSizeArrays.cpp - Legalize zero-size arrays -------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // SPIR-V does not support zero-size arrays unless it is within a shader. This
10: // pass legalizes zero-size arrays ([0 x T]) in unsupported cases.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVLegalizeZeroSizeArrays.h"
15: #include "SPIRV.h"
16: #include "SPIRVTargetMachine.h"
17: #include "SPIRVUtils.h"
18: #include "llvm/ADT/DenseMap.h"
19: #include "llvm/ADT/SmallVector.h"
20: #include "llvm/IR/IRBuilder.h"
21: #include "llvm/IR/InstIterator.h"
22: #include "llvm/IR/InstVisitor.h"
23: #include "llvm/Pass.h"
24: #include "llvm/Support/Debug.h"
25:
26: #define DEBUG_TYPE "spirv-legalize-zero-size-arrays"
27:
28: using namespace llvm;
29:
30: namespace {
31:
32: bool hasZeroSizeArray(const Type *Ty) {
33:   if (const ArrayType *ArrTy = dyn_cast<ArrayType>(Ty)) {
34:     if (ArrTy->getNumElements() == 0)
35:       return true;
36:     return hasZeroSizeArray(ArrTy->getElementType());
37:   }
38:
39:   if (const StructType *StructTy = dyn_cast<StructType>(Ty)) {
40:     for (Type *ElemTy : StructTy->elements()) {
41:       if (hasZeroSizeArray(ElemTy))
42:         return true;
43:     }
44:   }
45:
46:   return false;
47: }
48:
49: bool shouldLegalizeInstType(const Type *Ty) {
50:   // This recursive function will always terminate because we only look inside
51:   // array types, and those can't be recursive.
52:   if (const ArrayType *ArrTy = dyn_cast_if_present<ArrayType>(Ty)) {
53:     return ArrTy->getNumElements() == 0 ||
54:            shouldLegalizeInstType(ArrTy->getElementType());
55:   }
56:   return false;
57: }
58:
59: class SPIRVLegalizeZeroSizeArraysImpl
60:     : public InstVisitor<SPIRVLegalizeZeroSizeArraysImpl> {
```
- EN: This range defines or declares important types such as hasZeroSizeArray, shouldLegalizeInstType, SPIRVLegalizeZeroSizeArraysImpl, shaping the data model used by SPIRVLegalizeZeroSizeArrays.cpp.
- CN: 这一段定义或声明了 hasZeroSizeArray、shouldLegalizeInstType、SPIRVLegalizeZeroSizeArraysImpl 等关键类型，构成 SPIRVLegalizeZeroSizeArrays.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:   friend class InstVisitor<SPIRVLegalizeZeroSizeArraysImpl>;
 62:
 63: public:
 64:   SPIRVLegalizeZeroSizeArraysImpl(const SPIRVTargetMachine &TM)
 65:       : InstVisitor(), TM(TM) {}
 66:   bool runOnModule(Module &M);
 67:
 68:   // TODO: Handle GEP, PHI.
 69:   void visitAllocaInst(AllocaInst &AI);
 70:   void visitLoadInst(LoadInst &LI);
 71:   void visitStoreInst(StoreInst &SI);
 72:   void visitSelectInst(SelectInst &Sel);
 73:   void visitExtractValueInst(ExtractValueInst &EVI);
 74:   void visitInsertValueInst(InsertValueInst &IVI);
 75:
 76: private:
 77:   Type *legalizeType(Type *Ty);
 78:   Constant *legalizeConstant(Constant *C);
 79:
 80:   const SPIRVTargetMachine &TM;
 81:   DenseMap<Type *, Type *> TypeMap;
 82:   DenseMap<GlobalVariable *, GlobalVariable *> GlobalMap;
 83:   SmallVector<Instruction *, 16> ToErase;
 84:   bool Modified = false;
 85: };
 86:
 87: class SPIRVLegalizeZeroSizeArraysLegacy : public ModulePass {
 88: public:
 89:   static char ID;
 90:   SPIRVLegalizeZeroSizeArraysLegacy(const SPIRVTargetMachine &TM)
 91:       : ModulePass(ID), TM(TM) {}
 92:   StringRef getPassName() const override {
 93:     return "SPIRV Legalize Zero-Size Arrays";
 94:   }
 95:   bool runOnModule(Module &M) override {
 96:     SPIRVLegalizeZeroSizeArraysImpl Impl(TM);
 97:     return Impl.runOnModule(M);
 98:   }
 99:
100: private:
101:   const SPIRVTargetMachine &TM;
102: };
103:
104: // Legalize a type. There are only two cases we need to care about:
105: // arrays and structs.
106: //
107: // For arrays, we just replace the entire array type with a ptr.
108: //
109: // For structs, we create a new type with any members containing
110: // nested arrays legalized.
111:
112: Type *SPIRVLegalizeZeroSizeArraysImpl::legalizeType(Type *Ty) {
113:   auto It = TypeMap.find(Ty);
114:   if (It != TypeMap.end())
115:     return It->second;
116:
117:   Type *LegalizedTy = Ty;
118:
119:   if (isa<ArrayType>(Ty)) {
120:     LegalizedTy = PointerType::get(
```
- EN: This range defines or declares important types such as InstVisitor, SPIRVLegalizeZeroSizeArraysImpl, runOnModule, visitAllocaInst, shaping the data model used by SPIRVLegalizeZeroSizeArrays.cpp.
- CN: 这一段定义或声明了 InstVisitor、SPIRVLegalizeZeroSizeArraysImpl、runOnModule、visitAllocaInst 等关键类型，构成 SPIRVLegalizeZeroSizeArrays.cpp 使用的数据模型。

### Lines 121-180
```cpp
121:         Ty->getContext(),
122:         storageClassToAddressSpace(SPIRV::StorageClass::Generic));
123:
124:   } else if (StructType *StructTy = dyn_cast<StructType>(Ty)) {
125:     SmallVector<Type *, 8> ElemTypes;
126:     bool Changed = false;
127:     for (Type *ElemTy : StructTy->elements()) {
128:       Type *LegalizedElemTy = legalizeType(ElemTy);
129:       ElemTypes.push_back(LegalizedElemTy);
130:       Changed |= LegalizedElemTy != ElemTy;
131:     }
132:     if (Changed) {
133:       LegalizedTy =
134:           StructTy->hasName()
135:               ? StructType::create(StructTy->getContext(), ElemTypes,
136:                                    (StructTy->getName() + ".legalized").str(),
137:                                    StructTy->isPacked())
138:               : StructType::get(StructTy->getContext(), ElemTypes,
139:                                 StructTy->isPacked());
140:     }
141:   }
142:
143:   TypeMap[Ty] = LegalizedTy;
144:   return LegalizedTy;
145: }
146:
147: Constant *SPIRVLegalizeZeroSizeArraysImpl::legalizeConstant(Constant *C) {
148:   if (!C || !hasZeroSizeArray(C->getType()))
149:     return C;
150:
151:   if (GlobalVariable *GV = dyn_cast<GlobalVariable>(C)) {
152:     if (GlobalVariable *NewGV = GlobalMap.lookup(GV))
153:       return NewGV;
154:     return C;
155:   }
156:
157:   Type *NewTy = legalizeType(C->getType());
158:   if (isa<UndefValue>(C))
159:     return PoisonValue::get(NewTy);
160:   if (isa<ConstantAggregateZero>(C))
161:     return Constant::getNullValue(NewTy);
162:   if (ConstantArray *CA = dyn_cast<ConstantArray>(C)) {
163:     SmallVector<Constant *, 8> Elems;
164:     for (Use &U : CA->operands())
165:       Elems.push_back(legalizeConstant(cast<Constant>(U)));
166:     return ConstantArray::get(cast<ArrayType>(NewTy), Elems);
167:   }
168:
169:   if (ConstantStruct *CS = dyn_cast<ConstantStruct>(C)) {
170:     SmallVector<Constant *, 8> Fields;
171:     for (Use &U : CS->operands())
172:       Fields.push_back(legalizeConstant(cast<Constant>(U)));
173:     return ConstantStruct::get(cast<StructType>(NewTy), Fields);
174:   }
175:
176:   if (ConstantExpr *CE = dyn_cast<ConstantExpr>(C)) {
177:     // Don't legalize GEP constant expressions, the backend deals with them
178:     // fine.
179:     if (CE->getOpcode() == Instruction::GetElementPtr)
180:       return CE;
```
- EN: This range implements operational logic in helpers such as storageClassToAddressSpace, legalizeType, push_back, hasName, translating backend policy into executable code.
- CN: 这一段实现了 storageClassToAddressSpace、legalizeType、push_back、hasName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:     SmallVector<Constant *, 4> Ops;
182:     bool Changed = false;
183:     for (Use &U : CE->operands()) {
184:       Constant *LegalizedOp = legalizeConstant(cast<Constant>(U));
185:       Ops.push_back(LegalizedOp);
186:       Changed |= LegalizedOp != cast<Constant>(U.get());
187:     }
188:     if (Changed)
189:       return CE->getWithOperands(Ops);
190:   }
191:
192:   return C;
193: }
194:
195: void SPIRVLegalizeZeroSizeArraysImpl::visitAllocaInst(AllocaInst &AI) {
196:   // Check if allocation size is known-zero
197:   const DataLayout &DL = AI.getModule()->getDataLayout();
198:   std::optional<TypeSize> Size = AI.getAllocationSize(DL);
199:   if (!Size || !Size->isZero())
200:     return;
201:
202:   // Allocate a byte instead of an empty alloca.
203:   IRBuilder<> Builder(&AI);
204:   AllocaInst *NewAI = Builder.CreateAlloca(Builder.getInt8Ty());
205:   NewAI->takeName(&AI);
206:   NewAI->setAlignment(AI.getAlign());
207:   NewAI->setDebugLoc(AI.getDebugLoc());
208:   AI.replaceAllUsesWith(NewAI);
209:   ToErase.push_back(&AI);
210:   Modified = true;
211: }
212:
213: void SPIRVLegalizeZeroSizeArraysImpl::visitLoadInst(LoadInst &LI) {
214:   if (!hasZeroSizeArray(LI.getType()))
215:     return;
216:
217:   // TODO: Handle structs containing zero-size arrays.
218:   ArrayType *ArrTy = dyn_cast<ArrayType>(LI.getType());
219:   if (shouldLegalizeInstType(ArrTy)) {
220:     LI.replaceAllUsesWith(PoisonValue::get(LI.getType()));
221:     ToErase.push_back(&LI);
222:     Modified = true;
223:   }
224: }
225:
226: void SPIRVLegalizeZeroSizeArraysImpl::visitStoreInst(StoreInst &SI) {
227:   Type *StoreTy = SI.getValueOperand()->getType();
228:
229:   // TODO: Handle structs containing zero-size arrays.
230:   ArrayType *ArrTy = dyn_cast<ArrayType>(StoreTy);
231:   if (shouldLegalizeInstType(ArrTy)) {
232:     ToErase.push_back(&SI);
233:     Modified = true;
234:   }
235: }
236:
237: void SPIRVLegalizeZeroSizeArraysImpl::visitSelectInst(SelectInst &Sel) {
238:   if (!hasZeroSizeArray(Sel.getType()))
239:     return;
240:
```
- EN: This range implements operational logic in helpers such as legalizeConstant, push_back, get, getWithOperands, translating backend policy into executable code.
- CN: 这一段实现了 legalizeConstant、push_back、get、getWithOperands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:   // TODO: Handle structs containing zero-size arrays.
242:   ArrayType *ArrTy = dyn_cast<ArrayType>(Sel.getType());
243:   if (shouldLegalizeInstType(ArrTy)) {
244:     Sel.replaceAllUsesWith(PoisonValue::get(Sel.getType()));
245:     ToErase.push_back(&Sel);
246:     Modified = true;
247:   }
248: }
249:
250: void SPIRVLegalizeZeroSizeArraysImpl::visitExtractValueInst(
251:     ExtractValueInst &EVI) {
252:   if (!hasZeroSizeArray(EVI.getAggregateOperand()->getType()))
253:     return;
254:
255:   // TODO: Handle structs containing zero-size arrays.
256:   ArrayType *ArrTy = dyn_cast<ArrayType>(EVI.getType());
257:   if (shouldLegalizeInstType(ArrTy)) {
258:     EVI.replaceAllUsesWith(PoisonValue::get(EVI.getType()));
259:     ToErase.push_back(&EVI);
260:     Modified = true;
261:   }
262: }
263:
264: void SPIRVLegalizeZeroSizeArraysImpl::visitInsertValueInst(
265:     InsertValueInst &IVI) {
266:   if (!hasZeroSizeArray(IVI.getAggregateOperand()->getType()))
267:     return;
268:
269:   // TODO: Handle structs containing zero-size arrays.
270:   ArrayType *ArrTy =
271:       dyn_cast<ArrayType>(IVI.getInsertedValueOperand()->getType());
272:   if (shouldLegalizeInstType(ArrTy)) {
273:     IVI.replaceAllUsesWith(IVI.getAggregateOperand());
274:     ToErase.push_back(&IVI);
275:     Modified = true;
276:   }
277: }
278:
279: bool SPIRVLegalizeZeroSizeArraysImpl::runOnModule(Module &M) {
280:   TypeMap.clear();
281:   GlobalMap.clear();
282:   ToErase.clear();
283:   Modified = false;
284:
285:   // Runtime arrays are allowed for shaders, so we don't need to do anything.
286:   if (TM.getSubtargetImpl()->isShader())
287:     return false;
288:   // 0-sized arrays are handled differently for AMDGCN flavoured SPIRV.
289:   if (M.getTargetTriple().getVendor() == Triple::VendorType::AMD)
290:     return false;
291:
292:   // First pass: create new globals (legalizing the initializer as needed) and
293:   // track mapping (don't erase old ones yet).
294:   SmallVector<GlobalVariable *, 8> OldGlobals;
295:   for (GlobalVariable &GV : M.globals()) {
296:     if (!hasZeroSizeArray(GV.getValueType()))
297:       continue;
298:
299:     Type *NewTy = legalizeType(GV.getValueType());
300:     Constant *LegalizedInitializer =
```
- EN: This range implements operational logic in helpers such as getType, replaceAllUsesWith, push_back, getInsertedValueOperand, translating backend policy into executable code.
- CN: 这一段实现了 getType、replaceAllUsesWith、push_back、getInsertedValueOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:         GV.hasInitializer() && !GV.hasAppendingLinkage()
302:             ? legalizeConstant(GV.getInitializer())
303:             : nullptr;
304:
305:     // The new global will have the same linkage type as the original,
306:     // except in the case that it is an llvm intrinsic global such as
307:     // llvm.global_ctors with appending linkage, in which case we need to change
308:     // the linkage as appending linkage is only allowed for arrays.
309:     GlobalValue::LinkageTypes NewLT =
310:         GV.hasAppendingLinkage()
311:             ? GlobalValue::LinkageTypes::ExternalWeakLinkage
312:             : GV.getLinkage();
313:
314:     // Use an empty name for now, we will update it in the
315:     // following step.
316:     GlobalVariable *NewGV = new GlobalVariable(
317:         M, NewTy, GV.isConstant(), NewLT, LegalizedInitializer,
318:         /*Name=*/"", &GV, GV.getThreadLocalMode(), GV.getAddressSpace(),
319:         GV.isExternallyInitialized());
320:     NewGV->copyAttributesFrom(&GV);
321:     NewGV->copyMetadata(&GV, 0);
322:     NewGV->setComdat(GV.getComdat());
323:     NewGV->setAlignment(GV.getAlign());
324:     GlobalMap[&GV] = NewGV;
325:     OldGlobals.push_back(&GV);
326:     Modified = true;
327:   }
328:
329:   // Second pass: replace uses, transfer names, and erase old globals.
330:   for (GlobalVariable *GV : OldGlobals) {
331:     GlobalVariable *NewGV = GlobalMap[GV];
332:     GV->replaceAllUsesWith(ConstantExpr::getBitCast(NewGV, GV->getType()));
333:     NewGV->takeName(GV);
334:     GV->eraseFromParent();
335:   }
336:
337:   for (Function &F : M)
338:     for (Instruction &I : instructions(F))
339:       visit(I);
340:
341:   for (Instruction *I : ToErase)
342:     I->eraseFromParent();
343:
344:   return Modified;
345: }
346:
347: } // namespace
348:
349: PreservedAnalyses SPIRVLegalizeZeroSizeArrays::run(Module &M,
350:                                                    ModuleAnalysisManager &AM) {
351:   SPIRVLegalizeZeroSizeArraysImpl Impl(TM);
352:   if (Impl.runOnModule(M))
353:     return PreservedAnalyses::none();
354:   return PreservedAnalyses::all();
355: }
356:
357: char SPIRVLegalizeZeroSizeArraysLegacy::ID = 0;
358:
359: INITIALIZE_PASS(SPIRVLegalizeZeroSizeArraysLegacy,
360:                 "spirv-legalize-zero-size-arrays",
```
- EN: This range implements operational logic in helpers such as hasInitializer, legalizeConstant, hasAppendingLinkage, getLinkage, translating backend policy into executable code.
- CN: 这一段实现了 hasInitializer、legalizeConstant、hasAppendingLinkage、getLinkage 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-366
```cpp
361:                 "Legalize SPIR-V zero-size arrays", false, false)
362:
363: ModulePass *
364: llvm::createSPIRVLegalizeZeroSizeArraysPass(const SPIRVTargetMachine &TM) {
365:   return new SPIRVLegalizeZeroSizeArraysLegacy(TM);
366: }
```
- EN: This range implements operational logic in helpers such as llvm::createSPIRVLegalizeZeroSizeArraysPass, SPIRVLegalizeZeroSizeArraysLegacy, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createSPIRVLegalizeZeroSizeArraysPass、SPIRVLegalizeZeroSizeArraysLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include hasZeroSizeArray, shouldLegalizeInstType, SPIRVLegalizeZeroSizeArraysImpl, InstVisitor, runOnModule, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 hasZeroSizeArray, shouldLegalizeInstType, SPIRVLegalizeZeroSizeArraysImpl, InstVisitor, runOnModule，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVLegalizeZeroSizeArrays.h`
  - `SPIRV.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/InstIterator.h`
  - `llvm/IR/InstVisitor.h`
  - `llvm/Pass.h`
  - `llvm/Support/Debug.h`
