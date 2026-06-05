# SPIRVGlobalRegistry.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVGlobalRegistry.h`
- Repository: `llvm-project`
- Purpose (EN): SPIRVGlobalRegistry is used to maintain rich type information required for SPIR-V even after lowering from LLVM IR to GMIR.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVGlobalRegistry.h - SPIR-V Global Registry ----------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // SPIRVGlobalRegistry is used to maintain rich type information required for
10: // SPIR-V even after lowering from LLVM IR to GMIR. It can convert an llvm::Type
11: // into an OpTypeXXX instruction, and map it to a virtual register. Also it
12: // builds and supports consistency of constants and global variables.
13: //
14: //===----------------------------------------------------------------------===//
15:
16: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVGLOBALREGISTRY_H
17: #define LLVM_LIB_TARGET_SPIRV_SPIRVGLOBALREGISTRY_H
18:
19: #include "MCTargetDesc/SPIRVBaseInfo.h"
20: #include "SPIRVIRMapping.h"
21: #include "SPIRVInstrInfo.h"
22: #include "SPIRVTypeInst.h"
23: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
24: #include "llvm/IR/Constant.h"
25: #include "llvm/IR/TypedPointerType.h"
26:
27: namespace llvm {
28: class SPIRVSubtarget;
29:
30: using StructOffsetDecorator = std::function<void(Register)>;
31:
32: class SPIRVGlobalRegistry : public SPIRVIRMapping {
33:   // Registers holding values which have types associated with them.
34:   // Initialized upon VReg definition in IRTranslator.
35:   // Do not confuse this with DuplicatesTracker as DT maps Type* to <MF, Reg>
36:   // where Reg = OpType...
37:   // while VRegToTypeMap tracks SPIR-V type assigned to other regs (i.e. not
38:   // type-declaring ones).
39:   DenseMap<const MachineFunction *, DenseMap<Register, SPIRVTypeInst>>
40:       VRegToTypeMap;
41:
42:   DenseMap<SPIRVTypeInst, const Type *> SPIRVToLLVMType;
43:
44:   // map a Function to its definition (as a machine instruction operand)
45:   DenseMap<const Function *, const MachineOperand *> FunctionToInstr;
46:   DenseMap<const MachineInstr *, const Function *> FunctionToInstrRev;
47:   // map function pointer (as a machine instruction operand) to the used
48:   // Function
49:   DenseMap<const MachineOperand *, const Function *> InstrToFunction;
50:   // Maps Functions to their calls (in a form of the machine instruction,
51:   // OpFunctionCall) that happened before the definition is available
52:   DenseMap<const Function *, SmallPtrSet<MachineInstr *, 8>> ForwardCalls;
53:   // map a Function to its original return type before the clone function was
54:   // created during substitution of aggregate arguments
55:   // (see `SPIRVPrepareFunctions::removeAggregateTypesFromSignature()`)
56:   DenseMap<Value *, Type *> MutatedAggRet;
57:   // map an instruction to its value's attributes (type, name)
58:   DenseMap<MachineInstr *, std::pair<Type *, std::string>> ValueAttrs;
59:
60:   SmallPtrSet<const Type *, 4> TypesInProcessing;
```
- EN: This range defines or declares important types such as SPIRVSubtarget, SPIRVGlobalRegistry, shaping the data model used by SPIRVGlobalRegistry.h.
- CN: 这一段定义或声明了 SPIRVSubtarget、SPIRVGlobalRegistry 等关键类型，构成 SPIRVGlobalRegistry.h 使用的数据模型。

### Lines 61-120
```cpp
 61:   DenseMap<const Type *, SPIRVTypeInst> ForwardPointerTypes;
 62:
 63:   // Stores for each function the last inserted SPIR-V Type.
 64:   // See: SPIRVGlobalRegistry::createOpType.
 65:   DenseMap<const MachineFunction *, MachineInstr *> LastInsertedTypeMap;
 66:
 67:   // if a function returns a pointer, this is to map it into TypedPointerType
 68:   DenseMap<const Function *, TypedPointerType *> FunResPointerTypes;
 69:
 70:   // Current target's datalayout.
 71:   DataLayout DL;
 72:
 73:   // Holds the maximum ID we have in the module.
 74:   unsigned Bound;
 75:
 76:   // Maps values associated with untyped pointers into deduced element types of
 77:   // untyped pointers.
 78:   DenseMap<Value *, Type *> DeducedElTys;
 79:   // Maps composite values to deduced types where untyped pointers are replaced
 80:   // with typed ones.
 81:   DenseMap<Value *, Type *> DeducedNestedTys;
 82:   // Maps values to "assign type" calls, thus being a registry of created
 83:   // Intrinsic::spv_assign_ptr_type instructions.
 84:   DenseMap<Value *, CallInst *> AssignPtrTypeInstr;
 85:
 86:   // Maps OpVariable and OpFunction-related v-regs to its LLVM IR definition.
 87:   DenseMap<std::pair<const MachineFunction *, Register>, const Value *> Reg2GO;
 88:
 89:   // map of aliasing decorations to aliasing metadata
 90:   std::unordered_map<const MDNode *, MachineInstr *> AliasInstMDMap;
 91:
 92:   // Add a new OpTypeXXX instruction without checking for duplicates.
 93:   SPIRVTypeInst createSPIRVType(const Type *Type, MachineIRBuilder &MIRBuilder,
 94:                                 SPIRV::AccessQualifier::AccessQualifier AQ,
 95:                                 bool ExplicitLayoutRequired, bool EmitIR);
 96:   SPIRVTypeInst
 97:   findSPIRVType(const Type *Ty, MachineIRBuilder &MIRBuilder,
 98:                 SPIRV::AccessQualifier::AccessQualifier accessQual,
 99:                 bool ExplicitLayoutRequired, bool EmitIR);
100:   SPIRVTypeInst
101:   restOfCreateSPIRVType(const Type *Type, MachineIRBuilder &MIRBuilder,
102:                         SPIRV::AccessQualifier::AccessQualifier AccessQual,
103:                         bool ExplicitLayoutRequired, bool EmitIR);
104:
105:   // Internal function creating the Types/Constants at the correct position
106:   // in the function by tweaking the passed "MIRBuilder" insertion point and
107:   // restoring it to the correct position. "Op" should be the function creating
108:   // the specific operation you need, and should return the newly created
109:   // instruction.
110:   const MachineInstr *createConstOrTypeAtFunctionEntry(
111:       MachineIRBuilder &MIRBuilder,
112:       std::function<MachineInstr *(MachineIRBuilder &)> Op);
113:
114: public:
115:   SPIRVGlobalRegistry(DataLayout DL);
116:
117:   MachineFunction *CurMF;
118:
119:   void setBound(unsigned V) { Bound = V; }
120:   unsigned getBound() { return Bound; }
```
- EN: This range declares interfaces or inline helpers such as SPIRVGlobalRegistry, setBound, getBound, defining how other backend pieces interact with this header.
- CN: 这一段声明了 SPIRVGlobalRegistry、setBound、getBound 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 121-180
```cpp
121:
122:   void addGlobalObject(const Value *V, const MachineFunction *MF, Register R) {
123:     Reg2GO[std::make_pair(MF, R)] = V;
124:   }
125:   const Value *getGlobalObject(const MachineFunction *MF, Register R) {
126:     auto It = Reg2GO.find(std::make_pair(MF, R));
127:     return It == Reg2GO.end() ? nullptr : It->second;
128:   }
129:
130:   // Add a record to the map of function return pointer types.
131:   void addReturnType(const Function *ArgF, TypedPointerType *DerivedTy) {
132:     FunResPointerTypes[ArgF] = DerivedTy;
133:   }
134:   // Find a record in the map of function return pointer types.
135:   const TypedPointerType *findReturnType(const Function *ArgF) {
136:     auto It = FunResPointerTypes.find(ArgF);
137:     return It == FunResPointerTypes.end() ? nullptr : It->second;
138:   }
139:
140:   // A registry of "assign type" records:
141:   // - Add a record.
142:   void addAssignPtrTypeInstr(Value *Val, CallInst *AssignPtrTyCI) {
143:     AssignPtrTypeInstr[Val] = AssignPtrTyCI;
144:   }
145:   // - Find a record.
146:   CallInst *findAssignPtrTypeInstr(const Value *Val) {
147:     auto It = AssignPtrTypeInstr.find(Val);
148:     return It == AssignPtrTypeInstr.end() ? nullptr : It->second;
149:   }
150:   // - Find a record and update its key or add a new record, if found.
151:   void updateIfExistAssignPtrTypeInstr(Value *OldVal, Value *NewVal,
152:                                        bool DeleteOld) {
153:     if (CallInst *CI = findAssignPtrTypeInstr(OldVal)) {
154:       if (DeleteOld)
155:         AssignPtrTypeInstr.erase(OldVal);
156:       AssignPtrTypeInstr[NewVal] = CI;
157:     }
158:   }
159:
160:   // A registry of mutated values
161:   // (see `SPIRVPrepareFunctions::removeAggregateTypesFromSignature()`):
162:   // - Add a record.
163:   void addMutated(Value *Val, Type *Ty) { MutatedAggRet[Val] = Ty; }
164:   // - Find a record.
165:   Type *findMutated(const Value *Val) {
166:     auto It = MutatedAggRet.find(Val);
167:     return It == MutatedAggRet.end() ? nullptr : It->second;
168:   }
169:
170:   // A registry of value's attributes (type, name)
171:   // - Add a record.
172:   void addValueAttrs(MachineInstr *Key, std::pair<Type *, std::string> Val) {
173:     ValueAttrs[Key] = Val;
174:   }
175:   // - Find a record.
176:   bool findValueAttrs(const MachineInstr *Key, Type *&Ty, StringRef &Name) {
177:     auto It = ValueAttrs.find(Key);
178:     if (It == ValueAttrs.end())
179:       return false;
180:     Ty = It->second.first;
```
- EN: This range declares interfaces or inline helpers such as addGlobalObject, getGlobalObject, find, addReturnType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 addGlobalObject、getGlobalObject、find、addReturnType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 181-240
```cpp
181:     Name = It->second.second;
182:     return true;
183:   }
184:
185:   // Deduced element types of untyped pointers and composites:
186:   // - Add a record to the map of deduced element types.
187:   void addDeducedElementType(Value *Val, Type *Ty) { DeducedElTys[Val] = Ty; }
188:   // - Find a record in the map of deduced element types.
189:   Type *findDeducedElementType(const Value *Val) {
190:     auto It = DeducedElTys.find(Val);
191:     return It == DeducedElTys.end() ? nullptr : It->second;
192:   }
193:   // - Find a record and update its key or add a new record, if found.
194:   void updateIfExistDeducedElementType(Value *OldVal, Value *NewVal,
195:                                        bool DeleteOld) {
196:     if (Type *Ty = findDeducedElementType(OldVal)) {
197:       if (DeleteOld)
198:         DeducedElTys.erase(OldVal);
199:       DeducedElTys[NewVal] = Ty;
200:     }
201:   }
202:   // - Add a record to the map of deduced composite types.
203:   void addDeducedCompositeType(Value *Val, Type *Ty) {
204:     DeducedNestedTys[Val] = Ty;
205:   }
206:   // - Find a record in the map of deduced composite types.
207:   Type *findDeducedCompositeType(const Value *Val) {
208:     auto It = DeducedNestedTys.find(Val);
209:     return It == DeducedNestedTys.end() ? nullptr : It->second;
210:   }
211:   // - Find a type of the given Global value
212:   Type *getDeducedGlobalValueType(const GlobalValue *Global) {
213:     // we may know element type if it was deduced earlier
214:     Type *ElementTy = findDeducedElementType(Global);
215:     if (!ElementTy) {
216:       // or we may know element type if it's associated with a composite
217:       // value
218:       if (Value *GlobalElem =
219:               Global->getNumOperands() > 0 ? Global->getOperand(0) : nullptr)
220:         ElementTy = findDeducedCompositeType(GlobalElem);
221:       else if (const Function *Fn = dyn_cast<Function>(Global))
222:         ElementTy = SPIRV::getOriginalFunctionType(*Fn);
223:     }
224:     return ElementTy ? ElementTy : Global->getValueType();
225:   }
226:
227:   // Map a machine operand that represents a use of a function via function
228:   // pointer to a machine operand that represents the function definition.
229:   // Return either the register or invalid value, because we have no context for
230:   // a good diagnostic message in case of unexpectedly missing references.
231:   const MachineOperand *getFunctionDefinitionByUse(const MachineOperand *Use) {
232:     auto ResF = InstrToFunction.find(Use);
233:     if (ResF == InstrToFunction.end())
234:       return nullptr;
235:     auto ResReg = FunctionToInstr.find(ResF->second);
236:     return ResReg == FunctionToInstr.end() ? nullptr : ResReg->second;
237:   }
238:
239:   // Map a Function to a machine instruction that represents the function
240:   // definition.
```
- EN: This range declares interfaces or inline helpers such as addDeducedElementType, findDeducedElementType, find, erase, defining how other backend pieces interact with this header.
- CN: 这一段声明了 addDeducedElementType、findDeducedElementType、find、erase 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 241-300
```cpp
241:   const MachineInstr *getFunctionDefinition(const Function *F) {
242:     if (!F)
243:       return nullptr;
244:     auto MOIt = FunctionToInstr.find(F);
245:     return MOIt == FunctionToInstr.end() ? nullptr : MOIt->second->getParent();
246:   }
247:
248:   // Map a Function to a machine instruction that represents the function
249:   // definition.
250:   const Function *getFunctionByDefinition(const MachineInstr *MI) {
251:     if (!MI)
252:       return nullptr;
253:     auto FIt = FunctionToInstrRev.find(MI);
254:     return FIt == FunctionToInstrRev.end() ? nullptr : FIt->second;
255:   }
256:
257:   // map function pointer (as a machine instruction operand) to the used
258:   // Function
259:   void recordFunctionPointer(const MachineOperand *MO, const Function *F) {
260:     InstrToFunction[MO] = F;
261:   }
262:
263:   // map a Function to its definition (as a machine instruction)
264:   void recordFunctionDefinition(const Function *F, const MachineOperand *MO) {
265:     FunctionToInstr[F] = MO;
266:     FunctionToInstrRev[MO->getParent()] = F;
267:   }
268:
269:   // Return true if any OpConstantFunctionPointerINTEL were generated
270:   bool hasConstFunPtr() { return !InstrToFunction.empty(); }
271:
272:   // Add a record about forward function call.
273:   void addForwardCall(const Function *F, MachineInstr *MI) {
274:     ForwardCalls[F].insert(MI);
275:   }
276:
277:   // Map a Function to the vector of machine instructions that represents
278:   // forward function calls or to nullptr if not found.
279:   SmallPtrSet<MachineInstr *, 8> *getForwardCalls(const Function *F) {
280:     auto It = ForwardCalls.find(F);
281:     return It == ForwardCalls.end() ? nullptr : &It->second;
282:   }
283:
284:   // Get or create a SPIR-V type corresponding the given LLVM IR type,
285:   // and map it to the given VReg.
286:   SPIRVTypeInst assignTypeToVReg(const Type *Type, Register VReg,
287:                                  MachineIRBuilder &MIRBuilder,
288:                                  SPIRV::AccessQualifier::AccessQualifier AQ,
289:                                  bool EmitIR);
290:   SPIRVTypeInst assignIntTypeToVReg(unsigned BitWidth, Register VReg,
291:                                     MachineInstr &I, const SPIRVInstrInfo &TII);
292:   SPIRVTypeInst assignFloatTypeToVReg(unsigned BitWidth, Register VReg,
293:                                       MachineInstr &I,
294:                                       const SPIRVInstrInfo &TII);
295:   SPIRVTypeInst assignVectTypeToVReg(SPIRVTypeInst BaseType,
296:                                      unsigned NumElements, Register VReg,
297:                                      MachineInstr &I,
298:                                      const SPIRVInstrInfo &TII);
299:
300:   // In cases where the SPIR-V type is already known, this function can be
```
- EN: This range declares interfaces or inline helpers such as getFunctionDefinition, find, end, getFunctionByDefinition, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getFunctionDefinition、find、end、getFunctionByDefinition 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 301-360
```cpp
301:   // used to map it to the given VReg.
302:   void assignSPIRVTypeToVReg(SPIRVTypeInst Type, Register VReg,
303:                              const MachineFunction &MF);
304:
305:   // Either generate a new OpTypeXXX instruction or return an existing one
306:   // corresponding to the given LLVM IR type.
307:   // EmitIR controls if we emit GMIR or SPV constants (e.g. for array sizes)
308:   // because this method may be called from InstructionSelector and we don't
309:   // want to emit extra IR instructions there.
310:   SPIRVTypeInst getOrCreateSPIRVType(const Type *Type, MachineInstr &I,
311:                                      SPIRV::AccessQualifier::AccessQualifier AQ,
312:                                      bool EmitIR) {
313:     MachineIRBuilder MIRBuilder(I);
314:     return getOrCreateSPIRVType(Type, MIRBuilder, AQ, EmitIR);
315:   }
316:
317:   SPIRVTypeInst getOrCreateSPIRVType(const Type *Type,
318:                                      MachineIRBuilder &MIRBuilder,
319:                                      SPIRV::AccessQualifier::AccessQualifier AQ,
320:                                      bool EmitIR) {
321:     return getOrCreateSPIRVType(Type, MIRBuilder, AQ, false, EmitIR);
322:   }
323:
324:   const Type *getTypeForSPIRVType(SPIRVTypeInst Ty) const {
325:     auto Res = SPIRVToLLVMType.find(Ty);
326:     assert(Res != SPIRVToLLVMType.end());
327:     return Res->second;
328:   }
329:
330:   // Return a pointee's type, or nullptr otherwise.
331:   SPIRVTypeInst getPointeeType(SPIRVTypeInst PtrType);
332:   // Return a pointee's type op code, or 0 otherwise.
333:   unsigned getPointeeTypeOp(Register PtrReg);
334:
335:   // Either generate a new OpTypeXXX instruction or return an existing one
336:   // corresponding to the given string containing the name of the builtin type.
337:   // Return nullptr if unable to recognize SPIRV type name from `TypeStr`.
338:   SPIRVTypeInst getOrCreateSPIRVTypeByName(
339:       StringRef TypeStr, MachineIRBuilder &MIRBuilder, bool EmitIR,
340:       SPIRV::StorageClass::StorageClass SC = SPIRV::StorageClass::Function,
341:       SPIRV::AccessQualifier::AccessQualifier AQ =
342:           SPIRV::AccessQualifier::ReadWrite);
343:
344:   // Return the SPIR-V type instruction corresponding to the given VReg, or
345:   // nullptr if no such type instruction exists. The second argument MF
346:   // allows to search for the association in a context of the machine functions
347:   // than the current one, without switching between different "current" machine
348:   // functions.
349:   SPIRVTypeInst getSPIRVTypeForVReg(Register VReg,
350:                                     const MachineFunction *MF = nullptr) const;
351:
352:   // Return the result type of the instruction defining the register.
353:   SPIRVTypeInst getResultType(Register VReg, MachineFunction *MF = nullptr);
354:
355:   // Whether the given VReg has a SPIR-V type mapped to it yet.
356:   bool hasSPIRVTypeForVReg(Register VReg) const {
357:     return getSPIRVTypeForVReg(VReg) != nullptr;
358:   }
359:
360:   // Return the VReg holding the result of the given OpTypeXXX instruction.
```
- EN: This range declares interfaces or inline helpers such as MIRBuilder, getOrCreateSPIRVType, getTypeForSPIRVType, find, defining how other backend pieces interact with this header.
- CN: 这一段声明了 MIRBuilder、getOrCreateSPIRVType、getTypeForSPIRVType、find 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 361-420
```cpp
361:   Register getSPIRVTypeID(SPIRVTypeInst SpirvType) const;
362:
363:   // Return previous value of the current machine function
364:   MachineFunction *setCurrentFunc(MachineFunction &MF) {
365:     MachineFunction *Ret = CurMF;
366:     CurMF = &MF;
367:     return Ret;
368:   }
369:
370:   // Return true if the type is an aggregate type.
371:   bool isAggregateType(SPIRVTypeInst Type) const {
372:     return Type && (Type->getOpcode() == SPIRV::OpTypeStruct ||
373:                     Type->getOpcode() == SPIRV::OpTypeArray);
374:   }
375:
376:   // Whether the given VReg has an OpTypeXXX instruction mapped to it with the
377:   // given opcode (e.g. OpTypeFloat).
378:   bool isScalarOfType(Register VReg, unsigned TypeOpcode) const;
379:
380:   // Return true if the given VReg's assigned SPIR-V type is either a scalar
381:   // matching the given opcode, or a vector with an element type matching that
382:   // opcode (e.g. OpTypeBool, or OpTypeVector %x 4, where %x is OpTypeBool).
383:   bool isScalarOrVectorOfType(Register VReg, unsigned TypeOpcode) const;
384:
385:   // Returns true if `Type` is a resource type. This could be an image type
386:   // or a struct for a buffer decorated with the block decoration.
387:   bool isResourceType(SPIRVTypeInst Type) const;
388:
389:   // Return number of elements in a vector if the argument is associated with
390:   // a vector type. Return 1 for a scalar type, and 0 for a missing type.
391:   unsigned getScalarOrVectorComponentCount(Register VReg) const;
392:   unsigned getScalarOrVectorComponentCount(SPIRVTypeInst Type) const;
393:
394:   // Return the component type in a vector if the argument is associated with
395:   // a vector type. Returns the argument itself for other types, and nullptr
396:   // for a missing type.
397:   SPIRVTypeInst getScalarOrVectorComponentType(SPIRVTypeInst Type) const;
398:
399:   // For vectors or scalars of booleans, integers and floats, return the scalar
400:   // type's bitwidth. Otherwise calls llvm_unreachable().
401:   unsigned getScalarOrVectorBitWidth(SPIRVTypeInst Type) const;
402:
403:   // For vectors or scalars of integers and floats, return total bitwidth of the
404:   // argument. Otherwise returns 0.
405:   unsigned getNumScalarOrVectorTotalBitWidth(SPIRVTypeInst Type) const;
406:
407:   // Returns either pointer to integer type, that may be a type of vector
408:   // elements or an original type, or nullptr if the argument is niether
409:   // an integer scalar, nor an integer vector
410:   SPIRVTypeInst retrieveScalarOrVectorIntType(SPIRVTypeInst Type) const;
411:
412:   // For integer vectors or scalars, return whether the integers are signed.
413:   bool isScalarOrVectorSigned(SPIRVTypeInst Type) const;
414:
415:   // Gets the storage class of the pointer type assigned to this vreg.
416:   SPIRV::StorageClass::StorageClass getPointerStorageClass(Register VReg) const;
417:   SPIRV::StorageClass::StorageClass
418:   getPointerStorageClass(SPIRVTypeInst Type) const;
419:
420:   // Return the number of bits SPIR-V pointers and size_t variables require.
```
- EN: This range declares interfaces or inline helpers such as getSPIRVTypeID, setCurrentFunc, isAggregateType, getOpcode, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getSPIRVTypeID、setCurrentFunc、isAggregateType、getOpcode 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 421-480
```cpp
421:   unsigned getPointerSize() const {
422:     return DL.getPointerSizeInBits(/* AS = */ 0);
423:   }
424:
425:   // Returns true if two types are defined and are compatible in a sense of
426:   // OpBitcast instruction
427:   bool isBitcastCompatible(SPIRVTypeInst Type1, SPIRVTypeInst Type2) const;
428:
429:   // Informs about removal of the machine instruction and invalidates data
430:   // structures referring this instruction.
431:   void invalidateMachineInstr(MachineInstr *MI);
432:
433: private:
434:   SPIRVTypeInst getOpTypeBool(MachineIRBuilder &MIRBuilder);
435:
436:   const Type *adjustIntTypeByWidth(const Type *Ty) const;
437:   unsigned adjustOpTypeIntWidth(unsigned Width) const;
438:
439:   SPIRVTypeInst getOrCreateSPIRVType(const Type *Type,
440:                                      MachineIRBuilder &MIRBuilder,
441:                                      SPIRV::AccessQualifier::AccessQualifier AQ,
442:                                      bool ExplicitLayoutRequired, bool EmitIR);
443:
444:   SPIRVTypeInst getOpTypeInt(unsigned Width, MachineIRBuilder &MIRBuilder,
445:                              bool IsSigned = false);
446:
447:   SPIRVTypeInst getOpTypeFloat(uint32_t Width, MachineIRBuilder &MIRBuilder);
448:
449:   SPIRVTypeInst getOpTypeFloat(uint32_t Width, MachineIRBuilder &MIRBuilder,
450:                                SPIRV::FPEncoding::FPEncoding FPEncode);
451:
452:   SPIRVTypeInst getOpTypeVoid(MachineIRBuilder &MIRBuilder);
453:
454:   SPIRVTypeInst getOpTypeVector(uint32_t NumElems, SPIRVTypeInst ElemType,
455:                                 MachineIRBuilder &MIRBuilder);
456:
457:   SPIRVTypeInst getOpTypeArray(uint32_t NumElems, SPIRVTypeInst ElemType,
458:                                MachineIRBuilder &MIRBuilder,
459:                                bool ExplicitLayoutRequired, bool EmitIR);
460:
461:   SPIRVTypeInst getOpTypeOpaque(const StructType *Ty,
462:                                 MachineIRBuilder &MIRBuilder);
463:
464:   SPIRVTypeInst getOpTypeStruct(const StructType *Ty,
465:                                 MachineIRBuilder &MIRBuilder,
466:                                 SPIRV::AccessQualifier::AccessQualifier AccQual,
467:                                 StructOffsetDecorator Decorator, bool EmitIR);
468:
469:   SPIRVTypeInst getOpTypePointer(SPIRV::StorageClass::StorageClass SC,
470:                                  SPIRVTypeInst ElemType,
471:                                  MachineIRBuilder &MIRBuilder, Register Reg);
472:
473:   SPIRVTypeInst getOpTypeForwardPointer(SPIRV::StorageClass::StorageClass SC,
474:                                         MachineIRBuilder &MIRBuilder);
475:
476:   SPIRVTypeInst
477:   getOpTypeFunction(const FunctionType *Ty, SPIRVTypeInst RetType,
478:                     const SmallVectorImpl<SPIRVTypeInst> &ArgTypes,
479:                     MachineIRBuilder &MIRBuilder);
480:
```
- EN: This range declares interfaces or inline helpers such as getPointerSize, getPointerSizeInBits, isBitcastCompatible, invalidateMachineInstr, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getPointerSize、getPointerSizeInBits、isBitcastCompatible、invalidateMachineInstr 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 481-540
```cpp
481:   SPIRVTypeInst
482:   getOrCreateSpecialType(const Type *Ty, MachineIRBuilder &MIRBuilder,
483:                          SPIRV::AccessQualifier::AccessQualifier AccQual);
484:
485:   SPIRVTypeInst finishCreatingSPIRVType(const Type *LLVMTy,
486:                                         SPIRVTypeInst SpirvType);
487:   Register getOrCreateBaseRegister(Constant *Val, MachineInstr &I,
488:                                    SPIRVTypeInst SpvType,
489:                                    const SPIRVInstrInfo &TII, unsigned BitWidth,
490:                                    bool ZeroAsNull);
491:   Register getOrCreateCompositeOrNull(Constant *Val, MachineInstr &I,
492:                                       SPIRVTypeInst SpvType,
493:                                       const SPIRVInstrInfo &TII, Constant *CA,
494:                                       unsigned BitWidth, unsigned ElemCnt,
495:                                       bool ZeroAsNull = true);
496:
497:   Register getOrCreateIntCompositeOrNull(uint64_t Val,
498:                                          MachineIRBuilder &MIRBuilder,
499:                                          SPIRVTypeInst SpvType, bool EmitIR,
500:                                          Constant *CA, unsigned BitWidth,
501:                                          unsigned ElemCnt);
502:
503:   // Returns a pointer to a SPIR-V pointer type with the given base type and
504:   // storage class. It is the responsibility of the caller to make sure the
505:   // decorations on the base type are valid for the given storage class. For
506:   // example, it has the correct offset and stride decorations.
507:   SPIRVTypeInst
508:   getOrCreateSPIRVPointerTypeInternal(SPIRVTypeInst BaseType,
509:                                       MachineIRBuilder &MIRBuilder,
510:                                       SPIRV::StorageClass::StorageClass SC);
511:
512:   void addStructOffsetDecorations(Register Reg, StructType *Ty,
513:                                   MachineIRBuilder &MIRBuilder);
514:   void addArrayStrideDecorations(Register Reg, Type *ElementType,
515:                                  MachineIRBuilder &MIRBuilder);
516:   bool hasBlockDecoration(SPIRVTypeInst Type) const;
517:
518:   SPIRVTypeInst
519:   getOrCreateOpTypeImage(MachineIRBuilder &MIRBuilder,
520:                          SPIRVTypeInst SampledType, SPIRV::Dim::Dim Dim,
521:                          uint32_t Depth, uint32_t Arrayed,
522:                          uint32_t Multisampled, uint32_t Sampled,
523:                          SPIRV::ImageFormat::ImageFormat ImageFormat,
524:                          SPIRV::AccessQualifier::AccessQualifier AccQual);
525:
526: public:
527:   Register buildConstantInt(uint64_t Val, MachineIRBuilder &MIRBuilder,
528:                             SPIRVTypeInst SpvType, bool EmitIR,
529:                             bool ZeroAsNull = true);
530:   Register getOrCreateConstInt(uint64_t Val, MachineInstr &I,
531:                                SPIRVTypeInst SpvType, const SPIRVInstrInfo &TII,
532:                                bool ZeroAsNull = true);
533:   Register getOrCreateConstInt(const APInt &Val, MachineInstr &I,
534:                                SPIRVTypeInst SpvType, const SPIRVInstrInfo &TII,
535:                                bool ZeroAsNull = true);
536:   Register createConstInt(const ConstantInt *CI, MachineInstr &I,
537:                           SPIRVTypeInst SpvType, const SPIRVInstrInfo &TII,
538:                           bool ZeroAsNull);
539:   Register getOrCreateConstFP(APFloat Val, MachineInstr &I,
540:                               SPIRVTypeInst SpvType, const SPIRVInstrInfo &TII,
```
- EN: This range declares interfaces or inline helpers such as hasBlockDecoration, defining how other backend pieces interact with this header.
- CN: 这一段声明了 hasBlockDecoration 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 541-600
```cpp
541:                               bool ZeroAsNull = true);
542:   Register createConstFP(const ConstantFP *CF, MachineInstr &I,
543:                          SPIRVTypeInst SpvType, const SPIRVInstrInfo &TII,
544:                          bool ZeroAsNull);
545:   Register buildConstantFP(APFloat Val, MachineIRBuilder &MIRBuilder,
546:                            SPIRVTypeInst SpvType = nullptr);
547:
548:   Register getOrCreateConstVector(uint64_t Val, MachineInstr &I,
549:                                   SPIRVTypeInst SpvType,
550:                                   const SPIRVInstrInfo &TII,
551:                                   bool ZeroAsNull = true);
552:   Register getOrCreateConstVector(const APInt &Val, MachineInstr &I,
553:                                   SPIRVTypeInst SpvType,
554:                                   const SPIRVInstrInfo &TII,
555:                                   bool ZeroAsNull = true);
556:   Register getOrCreateConstVector(APFloat Val, MachineInstr &I,
557:                                   SPIRVTypeInst SpvType,
558:                                   const SPIRVInstrInfo &TII,
559:                                   bool ZeroAsNull = true);
560:   Register getOrCreateConstIntArray(uint64_t Val, size_t Num, MachineInstr &I,
561:                                     SPIRVTypeInst SpvType,
562:                                     const SPIRVInstrInfo &TII);
563:   Register getOrCreateConsIntVector(uint64_t Val, MachineIRBuilder &MIRBuilder,
564:                                     SPIRVTypeInst SpvType, bool EmitIR);
565:   Register getOrCreateConstNullPtr(MachineIRBuilder &MIRBuilder,
566:                                    SPIRVTypeInst SpvType);
567:   Register buildConstantSampler(Register Res, unsigned AddrMode, unsigned Param,
568:                                 unsigned FilerMode,
569:                                 MachineIRBuilder &MIRBuilder);
570:   Register getOrCreateUndef(MachineInstr &I, SPIRVTypeInst SpvType,
571:                             const SPIRVInstrInfo &TII);
572:   Register buildGlobalVariable(
573:       Register Reg, SPIRVTypeInst BaseType, StringRef Name,
574:       const GlobalValue *GV, SPIRV::StorageClass::StorageClass Storage,
575:       const MachineInstr *Init, bool IsConst,
576:       const std::optional<SPIRV::LinkageType::LinkageType> &LinkageType,
577:       MachineIRBuilder &MIRBuilder, bool IsInstSelector);
578:   Register getOrCreateGlobalVariableWithBinding(SPIRVTypeInst VarType,
579:                                                 uint32_t Set, uint32_t Binding,
580:                                                 StringRef Name,
581:                                                 MachineIRBuilder &MIRBuilder);
582:
583:   // Convenient helpers for getting types with check for duplicates.
584:   SPIRVTypeInst getOrCreateSPIRVIntegerType(unsigned BitWidth,
585:                                             MachineIRBuilder &MIRBuilder);
586:   SPIRVTypeInst getOrCreateSPIRVIntegerType(unsigned BitWidth, MachineInstr &I,
587:                                             const SPIRVInstrInfo &TII);
588:   SPIRVTypeInst getOrCreateSPIRVType(unsigned BitWidth, MachineInstr &I,
589:                                      const SPIRVInstrInfo &TII,
590:                                      unsigned SPIRVOPcode, Type *LLVMTy);
591:   SPIRVTypeInst getOrCreateSPIRVFloatType(unsigned BitWidth, MachineInstr &I,
592:                                           const SPIRVInstrInfo &TII);
593:   SPIRVTypeInst getOrCreateSPIRVBoolType(MachineIRBuilder &MIRBuilder,
594:                                          bool EmitIR);
595:   SPIRVTypeInst getOrCreateSPIRVBoolType(MachineInstr &I,
596:                                          const SPIRVInstrInfo &TII);
597:   SPIRVTypeInst getOrCreateSPIRVVectorType(SPIRVTypeInst BaseType,
598:                                            unsigned NumElements,
599:                                            MachineIRBuilder &MIRBuilder,
600:                                            bool EmitIR);
```
- EN: This range declares interfaces or inline helpers such as backend logic, defining how other backend pieces interact with this header.
- CN: 这一段声明了 后端逻辑 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 601-660
```cpp
601:   SPIRVTypeInst getOrCreateSPIRVVectorType(SPIRVTypeInst BaseType,
602:                                            unsigned NumElements,
603:                                            MachineInstr &I,
604:                                            const SPIRVInstrInfo &TII);
605:
606:   // Returns a pointer to a SPIR-V pointer type with the given base type and
607:   // storage class. The base type will be translated to a SPIR-V type, and the
608:   // appropriate layout decorations will be added to the base type.
609:   SPIRVTypeInst
610:   getOrCreateSPIRVPointerType(const Type *BaseType,
611:                               MachineIRBuilder &MIRBuilder,
612:                               SPIRV::StorageClass::StorageClass SC);
613:   SPIRVTypeInst
614:   getOrCreateSPIRVPointerType(const Type *BaseType, MachineInstr &I,
615:                               SPIRV::StorageClass::StorageClass SC);
616:
617:   // Returns a pointer to a SPIR-V pointer type with the given base type and
618:   // storage class. It is the responsibility of the caller to make sure the
619:   // decorations on the base type are valid for the given storage class. For
620:   // example, it has the correct offset and stride decorations.
621:   SPIRVTypeInst
622:   getOrCreateSPIRVPointerType(SPIRVTypeInst BaseType,
623:                               MachineIRBuilder &MIRBuilder,
624:                               SPIRV::StorageClass::StorageClass SC);
625:
626:   // Returns a pointer to a SPIR-V pointer type that is the same as `PtrType`
627:   // except the stroage class has been changed to `SC`. It is the responsibility
628:   // of the caller to be sure that the original and new storage class have the
629:   // same layout requirements.
630:   SPIRVTypeInst changePointerStorageClass(SPIRVTypeInst PtrType,
631:                                           SPIRV::StorageClass::StorageClass SC,
632:                                           MachineInstr &I);
633:
634:   SPIRVTypeInst
635:   getOrCreateVulkanBufferType(MachineIRBuilder &MIRBuilder, Type *ElemType,
636:                               SPIRV::StorageClass::StorageClass SC,
637:                               bool IsWritable, bool EmitIr = false);
638:
639:   SPIRVTypeInst getOrCreatePaddingType(MachineIRBuilder &MIRBuilder);
640:
641:   SPIRVTypeInst getOrCreateVulkanPushConstantType(MachineIRBuilder &MIRBuilder,
642:                                                   Type *ElemType);
643:
644:   SPIRVTypeInst getOrCreateLayoutType(MachineIRBuilder &MIRBuilder,
645:                                       const TargetExtType *T,
646:                                       bool EmitIr = false);
647:
648:   SPIRVTypeInst
649:   getImageType(const TargetExtType *ExtensionType,
650:                const SPIRV::AccessQualifier::AccessQualifier Qualifier,
651:                MachineIRBuilder &MIRBuilder);
652:
653:   SPIRVTypeInst getOrCreateOpTypeSampler(MachineIRBuilder &MIRBuilder);
654:
655:   SPIRVTypeInst getOrCreateOpTypeSampledImage(SPIRVTypeInst ImageType,
656:                                               MachineIRBuilder &MIRBuilder);
657:   SPIRVTypeInst getOrCreateOpTypeCoopMatr(MachineIRBuilder &MIRBuilder,
658:                                           const TargetExtType *ExtensionType,
659:                                           SPIRVTypeInst ElemType,
660:                                           uint32_t Scope, uint32_t Rows,
```
- EN: This range declares interfaces or inline helpers such as getOrCreatePaddingType, getOrCreateOpTypeSampler, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getOrCreatePaddingType、getOrCreateOpTypeSampler 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 661-696
```cpp
661:                                           uint32_t Columns, uint32_t Use,
662:                                           bool EmitIR);
663:   SPIRVTypeInst
664:   getOrCreateOpTypePipe(MachineIRBuilder &MIRBuilder,
665:                         SPIRV::AccessQualifier::AccessQualifier AccQual);
666:   SPIRVTypeInst getOrCreateOpTypeDeviceEvent(MachineIRBuilder &MIRBuilder);
667:   SPIRVTypeInst getOrCreateOpTypeFunctionWithArgs(
668:       const Type *Ty, SPIRVTypeInst RetType,
669:       const SmallVectorImpl<SPIRVTypeInst> &ArgTypes,
670:       MachineIRBuilder &MIRBuilder);
671:   SPIRVTypeInst getOrCreateOpTypeByOpcode(const Type *Ty,
672:                                           MachineIRBuilder &MIRBuilder,
673:                                           unsigned Opcode);
674:
675:   SPIRVTypeInst getOrCreateUnknownType(const Type *Ty,
676:                                        MachineIRBuilder &MIRBuilder,
677:                                        unsigned Opcode,
678:                                        const ArrayRef<MCOperand> Operands);
679:
680:   const TargetRegisterClass *getRegClass(SPIRVTypeInst SpvType) const;
681:   LLT getRegType(SPIRVTypeInst SpvType) const;
682:
683:   MachineInstr *getOrAddMemAliasingINTELInst(MachineIRBuilder &MIRBuilder,
684:                                              const MDNode *AliasingListMD);
685:   void buildMemAliasingOpDecorate(Register Reg, MachineIRBuilder &MIRBuilder,
686:                                   uint32_t Dec, const MDNode *GVarMD);
687:   // Replace all uses of a |Old| with |New| updates the global registry type
688:   // mappings.
689:   void replaceAllUsesWith(Value *Old, Value *New, bool DeleteOld = true);
690:
691:   void buildAssignType(IRBuilder<> &B, Type *Ty, Value *Arg);
692:   void buildAssignPtr(IRBuilder<> &B, Type *ElemTy, Value *Arg);
693:   void updateAssignType(CallInst *AssignCI, Value *Arg, Value *OfType);
694: };
695: } // end namespace llvm
696: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVGLOBALREGISTRY_H
```
- EN: This range declares interfaces or inline helpers such as getOrCreateOpTypeDeviceEvent, getRegClass, getRegType, replaceAllUsesWith, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getOrCreateOpTypeDeviceEvent、getRegClass、getRegType、replaceAllUsesWith 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVSubtarget, SPIRVGlobalRegistry, setBound, getBound, addGlobalObject, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVSubtarget, SPIRVGlobalRegistry, setBound, getBound, addGlobalObject，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRVIRMapping.h`
  - `SPIRVInstrInfo.h`
  - `SPIRVTypeInst.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
  - `llvm/IR/Constant.h`
  - `llvm/IR/TypedPointerType.h`
