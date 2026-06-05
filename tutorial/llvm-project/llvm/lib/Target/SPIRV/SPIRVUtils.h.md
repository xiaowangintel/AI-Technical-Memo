# SPIRVUtils.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVUtils.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains miscellaneous utility functions.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===--- SPIRVUtils.h ---- SPIR-V Utility Functions -------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains miscellaneous utility functions.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #ifndef LLVM_LIB_TARGET_SPIRV_SPIRVUTILS_H
14: #define LLVM_LIB_TARGET_SPIRV_SPIRVUTILS_H
15:
16: #include "MCTargetDesc/SPIRVBaseInfo.h"
17: #include "llvm/Analysis/LoopInfo.h"
18: #include "llvm/CodeGen/MachineBasicBlock.h"
19: #include "llvm/IR/Dominators.h"
20: #include "llvm/IR/GlobalVariable.h"
21: #include "llvm/IR/IRBuilder.h"
22: #include "llvm/IR/TypedPointerType.h"
23: #include <queue>
24: #include <set>
25: #include <string>
26: #include <unordered_map>
27: #include <unordered_set>
28:
29: #include "SPIRVTypeInst.h"
30:
31: namespace llvm {
32: class MCInst;
33: class MachineFunction;
34: class MachineInstrBuilder;
35: class MachineIRBuilder;
36: class MachineRegisterInfo;
37: class Register;
38: class StringRef;
39: class SPIRVInstrInfo;
40: class SPIRVSubtarget;
41: class SPIRVGlobalRegistry;
42:
43: // This class implements a partial ordering visitor, which visits a cyclic graph
44: // in natural topological-like ordering. Topological ordering is not defined for
45: // directed graphs with cycles, so this assumes cycles are a single node, and
46: // ignores back-edges. The cycle is visited from the entry in the same
47: // topological-like ordering.
48: //
49: // Note: this visitor REQUIRES a reducible graph.
50: //
51: // This means once we visit a node, we know all the possible ancestors have been
52: // visited.
53: //
54: // clang-format off
55: //
56: // Given this graph:
57: //
58: //     ,-> B -\
59: // A -+        +---> D ----> E -> F -> G -> H
60: //     `-> C -/      ^                 |
```
- EN: This range defines or declares important types such as MCInst, MachineFunction, MachineInstrBuilder, MachineIRBuilder, shaping the data model used by SPIRVUtils.h.
- CN: 这一段定义或声明了 MCInst、MachineFunction、MachineInstrBuilder、MachineIRBuilder 等关键类型，构成 SPIRVUtils.h 使用的数据模型。

### Lines 61-120
```cpp
 61: //                   +-----------------+
 62: //
 63: // Visit order is:
 64: //  A, [B, C in any order], D, E, F, G, H
 65: //
 66: // clang-format on
 67: //
 68: // Changing the function CFG between the construction of the visitor and
 69: // visiting is undefined. The visitor can be reused, but if the CFG is updated,
 70: // the visitor must be rebuilt.
 71: class PartialOrderingVisitor {
 72:   DomTreeBuilder::BBDomTree DT;
 73:   LoopInfo LI;
 74:
 75:   std::unordered_set<BasicBlock *> Queued = {};
 76:   std::queue<BasicBlock *> ToVisit = {};
 77:
 78:   struct OrderInfo {
 79:     size_t Rank;
 80:     size_t TraversalIndex;
 81:   };
 82:
 83:   using BlockToOrderInfoMap = std::unordered_map<BasicBlock *, OrderInfo>;
 84:   BlockToOrderInfoMap BlockToOrder;
 85:   std::vector<BasicBlock *> Order = {};
 86:
 87:   // Get all basic-blocks reachable from Start.
 88:   std::unordered_set<BasicBlock *> getReachableFrom(BasicBlock *Start);
 89:
 90:   // Internal function used to determine the partial ordering.
 91:   // Visits |BB| with the current rank being |Rank|.
 92:   size_t visit(BasicBlock *BB, size_t Rank);
 93:
 94:   bool CanBeVisited(BasicBlock *BB) const;
 95:
 96: public:
 97:   size_t GetNodeRank(BasicBlock *BB) const;
 98:
 99:   // Build the visitor to operate on the function F.
100:   PartialOrderingVisitor(Function &F);
101:
102:   // Returns true is |LHS| comes before |RHS| in the partial ordering.
103:   // If |LHS| and |RHS| have the same rank, the traversal order determines the
104:   // order (order is stable).
105:   bool compare(const BasicBlock *LHS, const BasicBlock *RHS) const;
106:
107:   // Visit the function starting from the basic block |Start|, and calling |Op|
108:   // on each visited BB. This traversal ignores back-edges, meaning this won't
109:   // visit a node to which |Start| is not an ancestor.
110:   // If Op returns |true|, the visitor continues. If |Op| returns false, the
111:   // visitor will stop at that rank. This means if 2 nodes share the same rank,
112:   // and Op returns false when visiting the first, the second will be visited
113:   // afterwards. But none of their successors will.
114:   void partialOrderVisit(BasicBlock &Start,
115:                          std::function<bool(BasicBlock *)> Op);
116: };
117:
118: namespace SPIRV {
119: struct FPFastMathDefaultInfo {
120:   const Type *Ty = nullptr;
```
- EN: This range defines or declares important types such as PartialOrderingVisitor, OrderInfo, getReachableFrom, visit, shaping the data model used by SPIRVUtils.h.
- CN: 这一段定义或声明了 PartialOrderingVisitor、OrderInfo、getReachableFrom、visit 等关键类型，构成 SPIRVUtils.h 使用的数据模型。

### Lines 121-180
```cpp
121:   unsigned FastMathFlags = 0;
122:   // When SPV_KHR_float_controls2 ContractionOff and SignzeroInfNanPreserve are
123:   // deprecated, and we replace them with FPFastMathDefault appropriate flags
124:   // instead. However, we have no guarantee about the order in which we will
125:   // process execution modes. Therefore it could happen that we first process
126:   // ContractionOff, setting AllowContraction bit to 0, and then we process
127:   // FPFastMathDefault enabling AllowContraction bit, effectively invalidating
128:   // ContractionOff. Because of that, it's best to keep separate bits for the
129:   // different execution modes, and we will try and combine them later when we
130:   // emit OpExecutionMode instructions.
131:   bool ContractionOff = false;
132:   bool SignedZeroInfNanPreserve = false;
133:   bool FPFastMathDefault = false;
134:
135:   FPFastMathDefaultInfo() = default;
136:   FPFastMathDefaultInfo(const Type *Ty, unsigned FastMathFlags)
137:       : Ty(Ty), FastMathFlags(FastMathFlags) {}
138:   bool operator==(const FPFastMathDefaultInfo &Other) const {
139:     return Ty == Other.Ty && FastMathFlags == Other.FastMathFlags &&
140:            ContractionOff == Other.ContractionOff &&
141:            SignedZeroInfNanPreserve == Other.SignedZeroInfNanPreserve &&
142:            FPFastMathDefault == Other.FPFastMathDefault;
143:   }
144: };
145:
146: struct FPFastMathDefaultInfoVector
147:     : public SmallVector<SPIRV::FPFastMathDefaultInfo, 3> {
148:   static size_t computeFPFastMathDefaultInfoVecIndex(size_t BitWidth) {
149:     switch (BitWidth) {
150:     case 16: // half
151:       return 0;
152:     case 32: // float
153:       return 1;
154:     case 64: // double
155:       return 2;
156:     default:
157:       report_fatal_error("Expected BitWidth to be 16, 32, 64", false);
158:     }
159:     llvm_unreachable(
160:         "Unreachable code in computeFPFastMathDefaultInfoVecIndex");
161:   }
162: };
163:
164: // This code restores function args/retvalue types for composite cases
165: // because the final types should still be aggregate whereas they're i32
166: // during the translation to cope with aggregate flattening etc.
167: FunctionType *getOriginalFunctionType(const Function &F);
168: FunctionType *getOriginalFunctionType(const CallBase &CB);
169: // This handles retrieving the original ASM constraints, which we had to spoof
170: // into having a single output.
171: StringRef getOriginalAsmConstraints(const CallBase &CB);
172: } // namespace SPIRV
173:
174: // Add the given string as a series of integer operand, inserting null
175: // terminators and padding to make sure the operands all have 32-bit
176: // little-endian words.
177: void addStringImm(const StringRef &Str, MCInst &Inst);
178: void addStringImm(const StringRef &Str, MachineInstrBuilder &MIB);
179: void addStringImm(const StringRef &Str, IRBuilder<> &B,
180:                   std::vector<Value *> &Args);
```
- EN: This range defines or declares important types such as FPFastMathDefaultInfo, Ty, FPFastMathDefaultInfoVector, computeFPFastMathDefaultInfoVecIndex, shaping the data model used by SPIRVUtils.h.
- CN: 这一段定义或声明了 FPFastMathDefaultInfo、Ty、FPFastMathDefaultInfoVector、computeFPFastMathDefaultInfoVecIndex 等关键类型，构成 SPIRVUtils.h 使用的数据模型。

### Lines 181-240
```cpp
181:
182: // Read the series of integer operands back as a null-terminated string using
183: // the reverse of the logic in addStringImm.
184: std::string getStringImm(const MachineInstr &MI, unsigned StartIndex);
185:
186: // Returns the string constant that the register refers to. It is assumed that
187: // Reg is a global value that contains a string.
188: std::string getStringValueFromReg(Register Reg, MachineRegisterInfo &MRI);
189:
190: // Add the given numerical immediate to MIB.
191: void addNumImm(const APInt &Imm, MachineInstrBuilder &MIB);
192:
193: // Add an OpName instruction for the given target register.
194: void buildOpName(Register Target, const StringRef &Name,
195:                  MachineIRBuilder &MIRBuilder);
196: void buildOpName(Register Target, const StringRef &Name, MachineInstr &I,
197:                  const SPIRVInstrInfo &TII);
198:
199: // Add an OpDecorate instruction for the given Reg.
200: void buildOpDecorate(Register Reg, MachineIRBuilder &MIRBuilder,
201:                      SPIRV::Decoration::Decoration Dec,
202:                      const std::vector<uint32_t> &DecArgs,
203:                      StringRef StrImm = "");
204: void buildOpDecorate(Register Reg, MachineInstr &I, const SPIRVInstrInfo &TII,
205:                      SPIRV::Decoration::Decoration Dec,
206:                      const std::vector<uint32_t> &DecArgs,
207:                      StringRef StrImm = "");
208:
209: // Add an OpDecorate instruction for the given Reg.
210: void buildOpMemberDecorate(Register Reg, MachineIRBuilder &MIRBuilder,
211:                            SPIRV::Decoration::Decoration Dec, uint32_t Member,
212:                            const std::vector<uint32_t> &DecArgs,
213:                            StringRef StrImm = "");
214: void buildOpMemberDecorate(Register Reg, MachineInstr &I,
215:                            const SPIRVInstrInfo &TII,
216:                            SPIRV::Decoration::Decoration Dec, uint32_t Member,
217:                            const std::vector<uint32_t> &DecArgs,
218:                            StringRef StrImm = "");
219:
220: // Add an OpDecorate instruction by "spirv.Decorations" metadata node.
221: void buildOpSpirvDecorations(Register Reg, MachineIRBuilder &MIRBuilder,
222:                              const MDNode *GVarMD, const SPIRVSubtarget &ST);
223:
224: // Return a valid position for the OpVariable instruction inside a function,
225: // i.e., at the beginning of the first block of the function.
226: MachineBasicBlock::iterator getOpVariableMBBIt(MachineFunction &MF);
227:
228: // Return a valid position for the instruction at the end of the block before
229: // terminators and debug instructions.
230: MachineBasicBlock::iterator getInsertPtValidEnd(MachineBasicBlock *MBB);
231:
232: // Returns true if a pointer to the storage class can be casted to/from a
233: // pointer to the Generic storage class.
234: constexpr bool isGenericCastablePtr(SPIRV::StorageClass::StorageClass SC) {
235:   switch (SC) {
236:   case SPIRV::StorageClass::Workgroup:
237:   case SPIRV::StorageClass::CrossWorkgroup:
238:   case SPIRV::StorageClass::Function:
239:     return true;
240:   default:
```
- EN: This range declares interfaces or inline helpers such as getStringImm, getStringValueFromReg, addNumImm, getOpVariableMBBIt, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getStringImm、getStringValueFromReg、addNumImm、getOpVariableMBBIt 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 241-300
```cpp
241:     return false;
242:   }
243: }
244:
245: // Convert a SPIR-V storage class to the corresponding LLVM IR address space.
246: // TODO: maybe the following two functions should be handled in the subtarget
247: // to allow for different OpenCL vs Vulkan handling.
248: constexpr unsigned
249: storageClassToAddressSpace(SPIRV::StorageClass::StorageClass SC) {
250:   switch (SC) {
251:   case SPIRV::StorageClass::Function:
252:     return 0;
253:   case SPIRV::StorageClass::CrossWorkgroup:
254:     return 1;
255:   case SPIRV::StorageClass::UniformConstant:
256:     return 2;
257:   case SPIRV::StorageClass::Workgroup:
258:     return 3;
259:   case SPIRV::StorageClass::Generic:
260:     return 4;
261:   case SPIRV::StorageClass::DeviceOnlyINTEL:
262:     return 5;
263:   case SPIRV::StorageClass::HostOnlyINTEL:
264:     return 6;
265:   case SPIRV::StorageClass::Input:
266:     return 7;
267:   case SPIRV::StorageClass::Output:
268:     return 8;
269:   case SPIRV::StorageClass::CodeSectionINTEL:
270:     return 9;
271:   case SPIRV::StorageClass::Private:
272:     return 10;
273:   case SPIRV::StorageClass::StorageBuffer:
274:     return 11;
275:   case SPIRV::StorageClass::Uniform:
276:     return 12;
277:   case SPIRV::StorageClass::PushConstant:
278:     return 13;
279:   default:
280:     report_fatal_error("Unable to get address space id");
281:   }
282: }
283:
284: // Convert an LLVM IR address space to a SPIR-V storage class.
285: SPIRV::StorageClass::StorageClass
286: addressSpaceToStorageClass(unsigned AddrSpace, const SPIRVSubtarget &STI);
287:
288: SPIRV::MemorySemantics::MemorySemantics
289: getMemSemanticsForStorageClass(SPIRV::StorageClass::StorageClass SC);
290:
291: SPIRV::MemorySemantics::MemorySemantics getMemSemantics(AtomicOrdering Ord);
292:
293: SPIRV::Scope::Scope getMemScope(LLVMContext &Ctx, SyncScope::ID Id);
294:
295: // Find def instruction for the given ConstReg, walking through
296: // spv_track_constant and ASSIGN_TYPE instructions. Updates ConstReg by def
297: // of OpConstant instruction.
298: MachineInstr *getDefInstrMaybeConstant(Register &ConstReg,
299:                                        const MachineRegisterInfo *MRI);
300:
```
- EN: This range declares interfaces or inline helpers such as storageClassToAddressSpace, report_fatal_error, addressSpaceToStorageClass, getMemSemanticsForStorageClass, defining how other backend pieces interact with this header.
- CN: 这一段声明了 storageClassToAddressSpace、report_fatal_error、addressSpaceToStorageClass、getMemSemanticsForStorageClass 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 301-360
```cpp
301: // Get constant integer value of the given ConstReg.
302: uint64_t getIConstVal(Register ConstReg, const MachineRegisterInfo *MRI);
303:
304: // Get constant integer value of the given ConstReg, sign-extended.
305: int64_t getIConstValSext(Register ConstReg, const MachineRegisterInfo *MRI);
306:
307: // Check if MI is a SPIR-V specific intrinsic call.
308: bool isSpvIntrinsic(const MachineInstr &MI, Intrinsic::ID IntrinsicID);
309: // Check if it's a SPIR-V specific intrinsic call.
310: bool isSpvIntrinsic(const Value *Arg);
311:
312: // Get type of i-th operand of the metadata node.
313: Type *getMDOperandAsType(const MDNode *N, unsigned I);
314:
315: // If OpenCL or SPIR-V builtin function name is recognized, return a demangled
316: // name, otherwise return an empty string.
317: std::string getOclOrSpirvBuiltinDemangledName(StringRef Name);
318:
319: // Check if a string contains a builtin prefix.
320: bool hasBuiltinTypePrefix(StringRef Name);
321:
322: // Check if given LLVM type is a special opaque builtin type.
323: bool isSpecialOpaqueType(const Type *Ty);
324:
325: // Check if the function is an SPIR-V entry point
326: bool isEntryPoint(const Function &F);
327:
328: // Parse basic scalar type name, substring TypeName, and return LLVM type.
329: Type *parseBasicTypeName(StringRef &TypeName, LLVMContext &Ctx);
330:
331: // Sort blocks in a partial ordering, so each block is after all its
332: // dominators. This should match both the SPIR-V and the MIR requirements.
333: // Returns true if the function was changed.
334: bool sortBlocks(Function &F);
335:
336: // Check for peeled array structs and recursively reconstitute them. In HLSL
337: // CBuffers, arrays may have padding between the elements, but not after the
338: // last element. To represent this in LLVM IR an array [N x T] will be
339: // represented as {[N-1 x {T, spirv.Padding}], T}. The function
340: // matchPeeledArrayPattern recognizes this pattern retrieving the type {T,
341: // spirv.Padding}, and the size N.
342: bool matchPeeledArrayPattern(const StructType *Ty, Type *&OriginalElementType,
343:                              uint64_t &TotalSize);
344:
345: // This function will turn the type {[N-1 x {T, spirv.Padding}], T} back into
346: // [N x {T, spirv.Padding}]. So it can be translated into SPIR-V. The offset
347: // decorations will be such that there will be no padding after the array when
348: // relevant.
349: Type *reconstitutePeeledArrayType(Type *Ty);
350:
351: inline bool hasInitializer(const GlobalVariable *GV) {
352:   if (!GV->hasInitializer())
353:     return false;
354:   if (const auto *Init = GV->getInitializer(); isa<UndefValue>(Init))
355:     return GV->isConstant() && Init->getType()->isAggregateType();
356:   return true;
357: }
358:
359: // True if this is an instance of TypedPointerType.
360: inline bool isTypedPointerTy(const Type *T) {
```
- EN: This range declares interfaces or inline helpers such as getIConstVal, getIConstValSext, isSpvIntrinsic, getMDOperandAsType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getIConstVal、getIConstValSext、isSpvIntrinsic、getMDOperandAsType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 361-420
```cpp
361:   return T && T->getTypeID() == Type::TypedPointerTyID;
362: }
363:
364: // True if this is an instance of PointerType.
365: inline bool isUntypedPointerTy(const Type *T) {
366:   return T && T->getTypeID() == Type::PointerTyID;
367: }
368:
369: // True if this is an instance of PointerType or TypedPointerType.
370: inline bool isPointerTy(const Type *T) {
371:   return isUntypedPointerTy(T) || isTypedPointerTy(T);
372: }
373:
374: // Get the address space of this pointer or pointer vector type for instances of
375: // PointerType or TypedPointerType.
376: inline unsigned getPointerAddressSpace(const Type *T) {
377:   Type *SubT = T->getScalarType();
378:   return SubT->getTypeID() == Type::PointerTyID
379:              ? cast<PointerType>(SubT)->getAddressSpace()
380:              : cast<TypedPointerType>(SubT)->getAddressSpace();
381: }
382:
383: // Return true if the Argument is decorated with a pointee type
384: inline bool hasPointeeTypeAttr(Argument *Arg) {
385:   return Arg->hasByValAttr() || Arg->hasByRefAttr() || Arg->hasStructRetAttr();
386: }
387:
388: // Return the pointee type of the argument or nullptr otherwise
389: inline Type *getPointeeTypeByAttr(Argument *Arg) {
390:   if (Arg->hasByValAttr())
391:     return Arg->getParamByValType();
392:   if (Arg->hasStructRetAttr())
393:     return Arg->getParamStructRetType();
394:   if (Arg->hasByRefAttr())
395:     return Arg->getParamByRefType();
396:   return nullptr;
397: }
398:
399: #define TYPED_PTR_TARGET_EXT_NAME "spirv.$TypedPointerType"
400: inline Type *getTypedPointerWrapper(Type *ElemTy, unsigned AS) {
401:   return TargetExtType::get(ElemTy->getContext(), TYPED_PTR_TARGET_EXT_NAME,
402:                             {ElemTy}, {AS});
403: }
404:
405: inline bool isTypedPointerWrapper(const TargetExtType *ExtTy) {
406:   return ExtTy->getName() == TYPED_PTR_TARGET_EXT_NAME &&
407:          ExtTy->getNumIntParameters() == 1 &&
408:          ExtTy->getNumTypeParameters() == 1;
409: }
410:
411: // True if this is an instance of PointerType or TypedPointerType.
412: inline bool isPointerTyOrWrapper(const Type *Ty) {
413:   if (auto *ExtTy = dyn_cast<TargetExtType>(Ty))
414:     return isTypedPointerWrapper(ExtTy);
415:   return isPointerTy(Ty);
416: }
417:
418: inline Type *applyWrappers(Type *Ty) {
419:   if (auto *ExtTy = dyn_cast<TargetExtType>(Ty)) {
420:     if (isTypedPointerWrapper(ExtTy))
```
- EN: This range declares interfaces or inline helpers such as isUntypedPointerTy, isPointerTy, getPointerAddressSpace, getScalarType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 isUntypedPointerTy、isPointerTy、getPointerAddressSpace、getScalarType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 421-480
```cpp
421:       return TypedPointerType::get(applyWrappers(ExtTy->getTypeParameter(0)),
422:                                    ExtTy->getIntParameter(0));
423:   } else if (auto *VecTy = dyn_cast<VectorType>(Ty)) {
424:     Type *ElemTy = VecTy->getElementType();
425:     Type *NewElemTy = ElemTy->isTargetExtTy() ? applyWrappers(ElemTy) : ElemTy;
426:     if (NewElemTy != ElemTy)
427:       return VectorType::get(NewElemTy, VecTy->getElementCount());
428:   }
429:   return Ty;
430: }
431:
432: inline Type *getPointeeType(const Type *Ty) {
433:   if (Ty) {
434:     if (auto PType = dyn_cast<TypedPointerType>(Ty))
435:       return PType->getElementType();
436:     else if (auto *ExtTy = dyn_cast<TargetExtType>(Ty))
437:       if (isTypedPointerWrapper(ExtTy))
438:         return ExtTy->getTypeParameter(0);
439:   }
440:   return nullptr;
441: }
442:
443: inline bool isUntypedEquivalentToTyExt(Type *Ty1, Type *Ty2) {
444:   if (!isUntypedPointerTy(Ty1) || !Ty2)
445:     return false;
446:   if (auto *ExtTy = dyn_cast<TargetExtType>(Ty2))
447:     if (isTypedPointerWrapper(ExtTy) &&
448:         ExtTy->getTypeParameter(0) ==
449:             IntegerType::getInt8Ty(Ty1->getContext()) &&
450:         ExtTy->getIntParameter(0) == cast<PointerType>(Ty1)->getAddressSpace())
451:       return true;
452:   return false;
453: }
454:
455: inline bool isEquivalentTypes(Type *Ty1, Type *Ty2) {
456:   return isUntypedEquivalentToTyExt(Ty1, Ty2) ||
457:          isUntypedEquivalentToTyExt(Ty2, Ty1);
458: }
459:
460: inline Type *toTypedPointer(Type *Ty) {
461:   if (Type *NewTy = applyWrappers(Ty); NewTy != Ty)
462:     return NewTy;
463:   return isUntypedPointerTy(Ty)
464:              ? TypedPointerType::get(IntegerType::getInt8Ty(Ty->getContext()),
465:                                      getPointerAddressSpace(Ty))
466:              : Ty;
467: }
468:
469: inline Type *toTypedFunPointer(FunctionType *FTy) {
470:   Type *OrigRetTy = FTy->getReturnType();
471:   Type *RetTy = toTypedPointer(OrigRetTy);
472:   bool IsUntypedPtr = false;
473:   for (Type *PTy : FTy->params()) {
474:     if (isUntypedPointerTy(PTy)) {
475:       IsUntypedPtr = true;
476:       break;
477:     }
478:   }
479:   if (!IsUntypedPtr && RetTy == OrigRetTy)
480:     return FTy;
```
- EN: This range declares interfaces or inline helpers such as getIntParameter, getElementType, VectorType::get, getPointeeType, defining how other backend pieces interact with this header.
- CN: 这一段声明了 getIntParameter、getElementType、VectorType::get、getPointeeType 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 481-540
```cpp
481:   SmallVector<Type *> ParamTys;
482:   for (Type *PTy : FTy->params())
483:     ParamTys.push_back(toTypedPointer(PTy));
484:   return FunctionType::get(RetTy, ParamTys, FTy->isVarArg());
485: }
486:
487: inline const Type *unifyPtrType(const Type *Ty) {
488:   if (auto FTy = dyn_cast<FunctionType>(Ty))
489:     return toTypedFunPointer(const_cast<FunctionType *>(FTy));
490:   return toTypedPointer(const_cast<Type *>(Ty));
491: }
492:
493: inline bool isVector1(Type *Ty) {
494:   auto *FVTy = dyn_cast<FixedVectorType>(Ty);
495:   return FVTy && FVTy->getNumElements() == 1;
496: }
497:
498: // Modify an LLVM type to conform with future transformations in IRTranslator.
499: // At the moment use cases comprise only a <1 x Type> vector. To extend when/if
500: // needed.
501: inline Type *normalizeType(Type *Ty) {
502:   auto *FVTy = dyn_cast<FixedVectorType>(Ty);
503:   if (!FVTy || FVTy->getNumElements() != 1)
504:     return Ty;
505:   // If it's a <1 x Type> vector type, replace it by the element type, because
506:   // it's not a legal vector type in LLT and IRTranslator will represent it as
507:   // the scalar eventually.
508:   return normalizeType(FVTy->getElementType());
509: }
510:
511: inline PoisonValue *getNormalizedPoisonValue(Type *Ty) {
512:   return PoisonValue::get(normalizeType(Ty));
513: }
514:
515: inline MetadataAsValue *buildMD(Value *Arg) {
516:   LLVMContext &Ctx = Arg->getContext();
517:   return MetadataAsValue::get(
518:       Ctx, MDNode::get(Ctx, ValueAsMetadata::getConstant(Arg)));
519: }
520:
521: CallInst *buildIntrWithMD(Intrinsic::ID IntrID, ArrayRef<Type *> Types,
522:                           Value *Arg, Value *Arg2, ArrayRef<Constant *> Imms,
523:                           IRBuilder<> &B);
524:
525: MachineInstr *getVRegDef(MachineRegisterInfo &MRI, Register Reg);
526:
527: #define SPIRV_BACKEND_SERVICE_FUN_NAME "__spirv_backend_service_fun"
528: bool getVacantFunctionName(Module &M, std::string &Name);
529:
530: void setRegClassType(Register Reg, const Type *Ty, SPIRVGlobalRegistry *GR,
531:                      MachineIRBuilder &MIRBuilder,
532:                      SPIRV::AccessQualifier::AccessQualifier AccessQual,
533:                      bool EmitIR, bool Force = false);
534: void setRegClassType(Register Reg, SPIRVTypeInst SpvType,
535:                      SPIRVGlobalRegistry *GR, MachineRegisterInfo *MRI,
536:                      const MachineFunction &MF, bool Force = false);
537: Register createVirtualRegister(SPIRVTypeInst SpvType, SPIRVGlobalRegistry *GR,
538:                                MachineRegisterInfo *MRI,
539:                                const MachineFunction &MF);
540: Register createVirtualRegister(SPIRVTypeInst SpvType, SPIRVGlobalRegistry *GR,
```
- EN: This range declares interfaces or inline helpers such as push_back, FunctionType::get, unifyPtrType, toTypedFunPointer, defining how other backend pieces interact with this header.
- CN: 这一段声明了 push_back、FunctionType::get、unifyPtrType、toTypedFunPointer 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

### Lines 541-589
```cpp
541:                                MachineIRBuilder &MIRBuilder);
542: Register createVirtualRegister(
543:     const Type *Ty, SPIRVGlobalRegistry *GR, MachineIRBuilder &MIRBuilder,
544:     SPIRV::AccessQualifier::AccessQualifier AccessQual, bool EmitIR);
545:
546: // Return true if there is an opaque pointer type nested in the argument.
547: bool isNestedPointer(const Type *Ty);
548:
549: enum FPDecorationId { NONE, RTE, RTZ, RTP, RTN, SAT };
550:
551: inline FPDecorationId demangledPostfixToDecorationId(const std::string &S) {
552:   static std::unordered_map<std::string, FPDecorationId> Mapping = {
553:       {"rte", FPDecorationId::RTE},
554:       {"rtz", FPDecorationId::RTZ},
555:       {"rtp", FPDecorationId::RTP},
556:       {"rtn", FPDecorationId::RTN},
557:       {"sat", FPDecorationId::SAT}};
558:   auto It = Mapping.find(S);
559:   return It == Mapping.end() ? FPDecorationId::NONE : It->second;
560: }
561:
562: SmallVector<MachineInstr *, 4>
563: createContinuedInstructions(MachineIRBuilder &MIRBuilder, unsigned Opcode,
564:                             unsigned MinWC, unsigned ContinuedOpcode,
565:                             ArrayRef<Register> Args, Register ReturnRegister,
566:                             Register TypeID);
567:
568: // Instruction selection directed by type folding.
569: const std::set<unsigned> &getTypeFoldingSupportedOpcodes();
570: bool isTypeFoldingSupported(unsigned Opcode);
571:
572: // Get loop controls from llvm.loop. metadata.
573: SmallVector<unsigned, 1> getSpirvLoopControlOperandsFromLoopMetadata(Loop *L);
574: SmallVector<unsigned, 1>
575: getSpirvLoopControlOperandsFromLoopMetadata(MDNode *LoopMD);
576:
577: // Traversing [g]MIR accounting for pseudo-instructions.
578: MachineInstr *passCopy(MachineInstr *Def, const MachineRegisterInfo *MRI);
579: MachineInstr *getDef(const MachineOperand &MO, const MachineRegisterInfo *MRI);
580: MachineInstr *getImm(const MachineOperand &MO, const MachineRegisterInfo *MRI);
581: int64_t foldImm(const MachineOperand &MO, const MachineRegisterInfo *MRI);
582: unsigned getArrayComponentCount(const MachineRegisterInfo *MRI,
583:                                 const MachineInstr *ResType);
584:
585: std::optional<SPIRV::LinkageType::LinkageType>
586: getSpirvLinkageTypeFor(const SPIRVSubtarget &ST, const GlobalValue &GV);
587: Function *getOrCreateBackendServiceFunction(Module &M);
588: } // namespace llvm
589: #endif // LLVM_LIB_TARGET_SPIRV_SPIRVUTILS_H
```
- EN: This range defines or declares important types such as isNestedPointer, FPDecorationId, demangledPostfixToDecorationId, find, shaping the data model used by SPIRVUtils.h.
- CN: 这一段定义或声明了 isNestedPointer、FPDecorationId、demangledPostfixToDecorationId、find 等关键类型，构成 SPIRVUtils.h 使用的数据模型。

## Key Concepts / 关键概念

- EN: Utility files package reusable helpers so other backend components can stay focused on core compilation steps.
  - CN: 工具类文件封装可复用辅助逻辑，使其他后端组件能够聚焦核心编译步骤。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include MCInst, MachineFunction, MachineInstrBuilder, MachineIRBuilder, MachineRegisterInfo, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 MCInst, MachineFunction, MachineInstrBuilder, MachineIRBuilder, MachineRegisterInfo，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRVTypeInst.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/Analysis/LoopInfo.h`
  - `llvm/CodeGen/MachineBasicBlock.h`
  - `llvm/IR/Dominators.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/TypedPointerType.h`
- System/standard headers / 系统或标准头文件:
  - `queue`
  - `set`
  - `string`
  - `unordered_map`
  - `unordered_set`
