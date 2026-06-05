# SPIRVCallLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVCallLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the lowering of LLVM calls to machine code calls for GlobalISel.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===--- SPIRVCallLowering.cpp - Call lowering ------------------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the lowering of LLVM calls to machine code calls for
10: // GlobalISel.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "SPIRVCallLowering.h"
15: #include "MCTargetDesc/SPIRVBaseInfo.h"
16: #include "SPIRV.h"
17: #include "SPIRVBuiltins.h"
18: #include "SPIRVGlobalRegistry.h"
19: #include "SPIRVISelLowering.h"
20: #include "SPIRVMetadata.h"
21: #include "SPIRVRegisterInfo.h"
22: #include "SPIRVSubtarget.h"
23: #include "SPIRVUtils.h"
24: #include "llvm/CodeGen/FunctionLoweringInfo.h"
25: #include "llvm/IR/IntrinsicInst.h"
26: #include "llvm/IR/IntrinsicsSPIRV.h"
27: #include "llvm/Support/ModRef.h"
28:
29: using namespace llvm;
30:
31: SPIRVCallLowering::SPIRVCallLowering(const SPIRVTargetLowering &TLI,
32:                                      SPIRVGlobalRegistry *GR)
33:     : CallLowering(&TLI), GR(GR) {}
34:
35: bool SPIRVCallLowering::lowerReturn(MachineIRBuilder &MIRBuilder,
36:                                     const Value *Val, ArrayRef<Register> VRegs,
37:                                     FunctionLoweringInfo &FLI,
38:                                     Register SwiftErrorVReg) const {
39:   // Ignore if called from the internal service function
40:   if (MIRBuilder.getMF()
41:           .getFunction()
42:           .getFnAttribute(SPIRV_BACKEND_SERVICE_FUN_NAME)
43:           .isValid())
44:     return true;
45:
46:   // Currently all return types should use a single register.
47:   // TODO: handle the case of multiple registers.
48:   if (VRegs.size() > 1)
49:     return false;
50:
51:   if (Val) {
52:     const auto &STI = MIRBuilder.getMF().getSubtarget();
53:     MIRBuilder.buildInstr(SPIRV::OpReturnValue)
54:         .addUse(VRegs[0])
55:         .constrainAllUses(MIRBuilder.getTII(), *STI.getRegisterInfo(),
56:                           *STI.getRegBankInfo());
57:     return true;
58:   }
59:   MIRBuilder.buildInstr(SPIRV::OpReturn);
60:   return true;
```
- EN: This range implements operational logic in helpers such as CallLowering, getFunction, getFnAttribute, isValid, translating backend policy into executable code.
- CN: 这一段实现了 CallLowering、getFunction、getFnAttribute、isValid 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61: }
 62:
 63: // Based on the LLVM function attributes, get a SPIR-V FunctionControl.
 64: static uint32_t getFunctionControl(const Function &F,
 65:                                    const SPIRVSubtarget *ST) {
 66:   MemoryEffects MemEffects = F.getMemoryEffects();
 67:
 68:   uint32_t FuncControl = static_cast<uint32_t>(SPIRV::FunctionControl::None);
 69:
 70:   if (F.hasFnAttribute(Attribute::AttrKind::NoInline))
 71:     FuncControl |= static_cast<uint32_t>(SPIRV::FunctionControl::DontInline);
 72:   else if (F.hasFnAttribute(Attribute::AttrKind::AlwaysInline))
 73:     FuncControl |= static_cast<uint32_t>(SPIRV::FunctionControl::Inline);
 74:
 75:   if (MemEffects.doesNotAccessMemory())
 76:     FuncControl |= static_cast<uint32_t>(SPIRV::FunctionControl::Pure);
 77:   else if (MemEffects.onlyReadsMemory())
 78:     FuncControl |= static_cast<uint32_t>(SPIRV::FunctionControl::Const);
 79:
 80:   if (ST->canUseExtension(SPIRV::Extension::SPV_INTEL_optnone) ||
 81:       ST->canUseExtension(SPIRV::Extension::SPV_EXT_optnone))
 82:     if (F.hasFnAttribute(Attribute::OptimizeNone))
 83:       FuncControl |= static_cast<uint32_t>(SPIRV::FunctionControl::OptNoneEXT);
 84:
 85:   return FuncControl;
 86: }
 87:
 88: static ConstantInt *getConstInt(MDNode *MD, unsigned NumOp) {
 89:   if (MD->getNumOperands() > NumOp) {
 90:     auto *CMeta = dyn_cast<ConstantAsMetadata>(MD->getOperand(NumOp));
 91:     if (CMeta)
 92:       return dyn_cast<ConstantInt>(CMeta->getValue());
 93:   }
 94:   return nullptr;
 95: }
 96:
 97: // If the function has pointer arguments, we are forced to re-create this
 98: // function type from the very beginning, changing PointerType by
 99: // TypedPointerType for each pointer argument. Otherwise, the same `Type*`
100: // potentially corresponds to different SPIR-V function type, effectively
101: // invalidating logic behind global registry and duplicates tracker.
102: static FunctionType *
103: fixFunctionTypeIfPtrArgs(SPIRVGlobalRegistry *GR, const Function &F,
104:                          FunctionType *FTy, SPIRVTypeInst SRetTy,
105:                          const SmallVector<SPIRVTypeInst, 4> &SArgTys) {
106:   bool hasArgPtrs = false;
107:   for (auto &Arg : F.args()) {
108:     // check if it's an instance of a non-typed PointerType
109:     if (Arg.getType()->isPointerTy()) {
110:       hasArgPtrs = true;
111:       break;
112:     }
113:   }
114:   if (!hasArgPtrs) {
115:     Type *RetTy = FTy->getReturnType();
116:     // check if it's an instance of a non-typed PointerType
117:     if (!RetTy->isPointerTy())
118:       return FTy;
119:   }
120:
```
- EN: This range implements operational logic in helpers such as getMemoryEffects, canUseExtension, getConstInt, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getMemoryEffects、canUseExtension、getConstInt、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   // re-create function type, using TypedPointerType instead of PointerType to
122:   // properly trace argument types
123:   const Type *RetTy = GR->getTypeForSPIRVType(SRetTy);
124:   SmallVector<Type *, 4> ArgTys;
125:   for (auto SArgTy : SArgTys)
126:     ArgTys.push_back(const_cast<Type *>(GR->getTypeForSPIRVType(SArgTy)));
127:   return FunctionType::get(const_cast<Type *>(RetTy), ArgTys, false);
128: }
129:
130: static SPIRV::AccessQualifier::AccessQualifier
131: getArgAccessQual(const Function &F, unsigned ArgIdx) {
132:   if (F.getCallingConv() != CallingConv::SPIR_KERNEL)
133:     return SPIRV::AccessQualifier::ReadWrite;
134:
135:   MDString *ArgAttribute = getOCLKernelArgAccessQual(F, ArgIdx);
136:   if (!ArgAttribute)
137:     return SPIRV::AccessQualifier::ReadWrite;
138:
139:   if (ArgAttribute->getString() == "read_only")
140:     return SPIRV::AccessQualifier::ReadOnly;
141:   if (ArgAttribute->getString() == "write_only")
142:     return SPIRV::AccessQualifier::WriteOnly;
143:   return SPIRV::AccessQualifier::ReadWrite;
144: }
145:
146: static std::vector<SPIRV::Decoration::Decoration>
147: getKernelArgTypeQual(const Function &F, unsigned ArgIdx) {
148:   MDString *ArgAttribute = getOCLKernelArgTypeQual(F, ArgIdx);
149:   if (ArgAttribute && ArgAttribute->getString() == "volatile")
150:     return {SPIRV::Decoration::Volatile};
151:   return {};
152: }
153:
154: static SPIRVTypeInst getArgSPIRVType(const Function &F, unsigned ArgIdx,
155:                                      SPIRVGlobalRegistry *GR,
156:                                      MachineIRBuilder &MIRBuilder,
157:                                      const SPIRVSubtarget &ST) {
158:   // Read argument's access qualifier from metadata or default.
159:   SPIRV::AccessQualifier::AccessQualifier ArgAccessQual =
160:       getArgAccessQual(F, ArgIdx);
161:
162:   Type *OriginalArgType =
163:       SPIRV::getOriginalFunctionType(F)->getParamType(ArgIdx);
164:
165:   // If OriginalArgType is non-pointer, use the OriginalArgType (the type cannot
166:   // be legally reassigned later).
167:   if (!isPointerTy(OriginalArgType))
168:     return GR->getOrCreateSPIRVType(OriginalArgType, MIRBuilder, ArgAccessQual,
169:                                     true);
170:
171:   Argument *Arg = F.getArg(ArgIdx);
172:   Type *ArgType = Arg->getType();
173:   if (isTypedPointerTy(ArgType)) {
174:     return GR->getOrCreateSPIRVPointerType(
175:         cast<TypedPointerType>(ArgType)->getElementType(), MIRBuilder,
176:         addressSpaceToStorageClass(getPointerAddressSpace(ArgType), ST));
177:   }
178:
179:   // In case OriginalArgType is of untyped pointer type, there are three
180:   // possibilities:
```
- EN: This range implements operational logic in helpers such as getTypeForSPIRVType, push_back, FunctionType::get, getArgAccessQual, translating backend policy into executable code.
- CN: 这一段实现了 getTypeForSPIRVType、push_back、FunctionType::get、getArgAccessQual 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   // 1) This is a pointer of an LLVM IR element type, passed byval/byref.
182:   // 2) This is an OpenCL/SPIR-V builtin type if there is spv_assign_type
183:   //    intrinsic assigning a TargetExtType.
184:   // 3) This is a pointer, try to retrieve pointer element type from a
185:   // spv_assign_ptr_type intrinsic or otherwise use default pointer element
186:   // type.
187:   if (hasPointeeTypeAttr(Arg)) {
188:     return GR->getOrCreateSPIRVPointerType(
189:         getPointeeTypeByAttr(Arg), MIRBuilder,
190:         addressSpaceToStorageClass(getPointerAddressSpace(ArgType), ST));
191:   }
192:
193:   for (auto User : Arg->users()) {
194:     auto *II = dyn_cast<IntrinsicInst>(User);
195:     // Check if this is spv_assign_type assigning OpenCL/SPIR-V builtin type.
196:     if (II && II->getIntrinsicID() == Intrinsic::spv_assign_type) {
197:       MetadataAsValue *VMD = cast<MetadataAsValue>(II->getOperand(1));
198:       Type *BuiltinType =
199:           cast<ConstantAsMetadata>(VMD->getMetadata())->getType();
200:       assert(BuiltinType->isTargetExtTy() && "Expected TargetExtType");
201:       return GR->getOrCreateSPIRVType(BuiltinType, MIRBuilder, ArgAccessQual,
202:                                       true);
203:     }
204:
205:     // Check if this is spv_assign_ptr_type assigning pointer element type.
206:     if (!II || II->getIntrinsicID() != Intrinsic::spv_assign_ptr_type)
207:       continue;
208:
209:     MetadataAsValue *VMD = cast<MetadataAsValue>(II->getOperand(1));
210:     Type *ElementTy =
211:         toTypedPointer(cast<ConstantAsMetadata>(VMD->getMetadata())->getType());
212:     return GR->getOrCreateSPIRVPointerType(
213:         ElementTy, MIRBuilder,
214:         addressSpaceToStorageClass(
215:             cast<ConstantInt>(II->getOperand(2))->getZExtValue(), ST));
216:   }
217:
218:   // Replace PointerType with TypedPointerType to be able to map SPIR-V types to
219:   // LLVM types in a consistent manner
220:   return GR->getOrCreateSPIRVType(toTypedPointer(OriginalArgType), MIRBuilder,
221:                                   ArgAccessQual, true);
222: }
223:
224: static SPIRV::ExecutionModel::ExecutionModel
225: getExecutionModel(const SPIRVSubtarget &STI, const Function &F) {
226:   assert(STI.getEnv() != SPIRVSubtarget::Unknown &&
227:          "Environment must be resolved before lowering entry points.");
228:
229:   if (STI.isKernel())
230:     return SPIRV::ExecutionModel::Kernel;
231:
232:   auto attribute = F.getFnAttribute("hlsl.shader");
233:   if (!attribute.isValid()) {
234:     report_fatal_error(
235:         "This entry point lacks mandatory hlsl.shader attribute.");
236:   }
237:
238:   const auto value = attribute.getValueAsString();
239:   if (value == "compute")
240:     return SPIRV::ExecutionModel::GLCompute;
```
- EN: This range implements operational logic in helpers such as addressSpaceToStorageClass, getOperand, getMetadata, assert, translating backend policy into executable code.
- CN: 这一段实现了 addressSpaceToStorageClass、getOperand、getMetadata、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:   if (value == "vertex")
242:     return SPIRV::ExecutionModel::Vertex;
243:   if (value == "pixel")
244:     return SPIRV::ExecutionModel::Fragment;
245:
246:   report_fatal_error("This HLSL entry point is not supported by this backend.");
247: }
248:
249: bool SPIRVCallLowering::lowerFormalArguments(MachineIRBuilder &MIRBuilder,
250:                                              const Function &F,
251:                                              ArrayRef<ArrayRef<Register>> VRegs,
252:                                              FunctionLoweringInfo &FLI) const {
253:   // Discard the internal service function
254:   if (F.getFnAttribute(SPIRV_BACKEND_SERVICE_FUN_NAME).isValid())
255:     return true;
256:
257:   assert(GR && "Must initialize the SPIRV type registry before lowering args.");
258:   GR->setCurrentFunc(MIRBuilder.getMF());
259:
260:   // Get access to information about available extensions
261:   const SPIRVSubtarget *ST =
262:       static_cast<const SPIRVSubtarget *>(&MIRBuilder.getMF().getSubtarget());
263:
264:   // Assign types and names to all args, and store their types for later.
265:   SmallVector<SPIRVTypeInst, 4> ArgTypeVRegs;
266:   if (VRegs.size() > 0) {
267:     unsigned i = 0;
268:     for (const auto &Arg : F.args()) {
269:       // Currently formal args should use single registers.
270:       // TODO: handle the case of multiple registers.
271:       if (VRegs[i].size() > 1)
272:         return false;
273:       SPIRVTypeInst SpirvTy = getArgSPIRVType(F, i, GR, MIRBuilder, *ST);
274:       GR->assignSPIRVTypeToVReg(SpirvTy, VRegs[i][0], MIRBuilder.getMF());
275:       ArgTypeVRegs.push_back(SpirvTy);
276:
277:       if (Arg.hasName())
278:         buildOpName(VRegs[i][0], Arg.getName(), MIRBuilder);
279:       if (isPointerTyOrWrapper(Arg.getType())) {
280:         auto DerefBytes = static_cast<unsigned>(Arg.getDereferenceableBytes());
281:         if (DerefBytes != 0)
282:           buildOpDecorate(VRegs[i][0], MIRBuilder,
283:                           SPIRV::Decoration::MaxByteOffset, {DerefBytes});
284:       }
285:       if (Arg.hasAttribute(Attribute::Alignment) && !ST->isShader()) {
286:         auto Alignment = static_cast<unsigned>(
287:             Arg.getAttribute(Attribute::Alignment).getValueAsInt());
288:         buildOpDecorate(VRegs[i][0], MIRBuilder, SPIRV::Decoration::Alignment,
289:                         {Alignment});
290:       }
291:       if (!ST->isShader()) {
292:         if (Arg.hasAttribute(Attribute::ReadOnly)) {
293:           auto Attr =
294:               static_cast<unsigned>(SPIRV::FunctionParameterAttribute::NoWrite);
295:           buildOpDecorate(VRegs[i][0], MIRBuilder,
296:                           SPIRV::Decoration::FuncParamAttr, {Attr});
297:         }
298:         if (Arg.hasAttribute(Attribute::ZExt)) {
299:           auto Attr =
300:               static_cast<unsigned>(SPIRV::FunctionParameterAttribute::Zext);
```
- EN: This range implements operational logic in helpers such as report_fatal_error, assert, setCurrentFunc, getMF, translating backend policy into executable code.
- CN: 这一段实现了 report_fatal_error、assert、setCurrentFunc、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:           buildOpDecorate(VRegs[i][0], MIRBuilder,
302:                           SPIRV::Decoration::FuncParamAttr, {Attr});
303:         }
304:         if (Arg.hasAttribute(Attribute::NoAlias)) {
305:           auto Attr =
306:               static_cast<unsigned>(SPIRV::FunctionParameterAttribute::NoAlias);
307:           buildOpDecorate(VRegs[i][0], MIRBuilder,
308:                           SPIRV::Decoration::FuncParamAttr, {Attr});
309:         }
310:         // TODO: the AMDGPU BE only supports ByRef argument passing, thus for
311:         //       AMDGCN flavoured SPIRV we CodeGen for ByRef, but lower it to
312:         //       ByVal, handling the impedance mismatch during reverse
313:         //       translation from SPIRV to LLVM IR; the vendor check should be
314:         //       removed once / if SPIRV adds ByRef support.
315:         if (Arg.hasAttribute(Attribute::ByVal) ||
316:             (Arg.hasAttribute(Attribute::ByRef) &&
317:              F.getParent()->getTargetTriple().getVendor() ==
318:                  Triple::VendorType::AMD)) {
319:           auto Attr =
320:               static_cast<unsigned>(SPIRV::FunctionParameterAttribute::ByVal);
321:           buildOpDecorate(VRegs[i][0], MIRBuilder,
322:                           SPIRV::Decoration::FuncParamAttr, {Attr});
323:         }
324:         if (Arg.hasAttribute(Attribute::StructRet)) {
325:           auto Attr =
326:               static_cast<unsigned>(SPIRV::FunctionParameterAttribute::Sret);
327:           buildOpDecorate(VRegs[i][0], MIRBuilder,
328:                           SPIRV::Decoration::FuncParamAttr, {Attr});
329:         }
330:       }
331:
332:       if (F.getCallingConv() == CallingConv::SPIR_KERNEL) {
333:         std::vector<SPIRV::Decoration::Decoration> ArgTypeQualDecs =
334:             getKernelArgTypeQual(F, i);
335:         for (SPIRV::Decoration::Decoration Decoration : ArgTypeQualDecs)
336:           buildOpDecorate(VRegs[i][0], MIRBuilder, Decoration, {});
337:       }
338:
339:       MDNode *Node = F.getMetadata("spirv.ParameterDecorations");
340:       if (Node && i < Node->getNumOperands() &&
341:           isa<MDNode>(Node->getOperand(i))) {
342:         MDNode *MD = cast<MDNode>(Node->getOperand(i));
343:         for (const MDOperand &MDOp : MD->operands()) {
344:           MDNode *MD2 = dyn_cast<MDNode>(MDOp);
345:           assert(MD2 && "Metadata operand is expected");
346:           ConstantInt *Const = getConstInt(MD2, 0);
347:           assert(Const && "MDOperand should be ConstantInt");
348:           auto Dec =
349:               static_cast<SPIRV::Decoration::Decoration>(Const->getZExtValue());
350:           std::vector<uint32_t> DecVec;
351:           for (unsigned j = 1; j < MD2->getNumOperands(); j++) {
352:             ConstantInt *Const = getConstInt(MD2, j);
353:             assert(Const && "MDOperand should be ConstantInt");
354:             DecVec.push_back(static_cast<uint32_t>(Const->getZExtValue()));
355:           }
356:           buildOpDecorate(VRegs[i][0], MIRBuilder, Dec, DecVec);
357:         }
358:       }
359:       ++i;
360:     }
```
- EN: This range implements operational logic in helpers such as getKernelArgTypeQual, buildOpDecorate, getMetadata, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getKernelArgTypeQual、buildOpDecorate、getMetadata、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   }
362:
363:   auto MRI = MIRBuilder.getMRI();
364:   Register FuncVReg = MRI->createGenericVirtualRegister(LLT::scalar(64));
365:   MRI->setRegClass(FuncVReg, &SPIRV::iIDRegClass);
366:   FunctionType *FTy = SPIRV::getOriginalFunctionType(F);
367:   Type *FRetTy = FTy->getReturnType();
368:   if (isUntypedPointerTy(FRetTy)) {
369:     if (Type *FRetElemTy = GR->findDeducedElementType(&F)) {
370:       TypedPointerType *DerivedTy = TypedPointerType::get(
371:           toTypedPointer(FRetElemTy), getPointerAddressSpace(FRetTy));
372:       GR->addReturnType(&F, DerivedTy);
373:       FRetTy = DerivedTy;
374:     }
375:   }
376:   SPIRVTypeInst RetTy = GR->getOrCreateSPIRVType(
377:       FRetTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
378:   FTy = fixFunctionTypeIfPtrArgs(GR, F, FTy, RetTy, ArgTypeVRegs);
379:   SPIRVTypeInst FuncTy = GR->getOrCreateOpTypeFunctionWithArgs(
380:       FTy, RetTy, ArgTypeVRegs, MIRBuilder);
381:   uint32_t FuncControl = getFunctionControl(F, ST);
382:
383:   // Add OpFunction instruction
384:   MachineInstrBuilder MB = MIRBuilder.buildInstr(SPIRV::OpFunction)
385:                                .addDef(FuncVReg)
386:                                .addUse(GR->getSPIRVTypeID(RetTy))
387:                                .addImm(FuncControl)
388:                                .addUse(GR->getSPIRVTypeID(FuncTy));
389:   GR->recordFunctionDefinition(&F, &MB.getInstr()->getOperand(0));
390:   GR->addGlobalObject(&F, &MIRBuilder.getMF(), FuncVReg);
391:   if (F.isDeclaration())
392:     GR->add(&F, MB);
393:
394:   // Add OpFunctionParameter instructions
395:   int i = 0;
396:   for (const auto &Arg : F.args()) {
397:     assert(VRegs[i].size() == 1 && "Formal arg has multiple vregs");
398:     Register ArgReg = VRegs[i][0];
399:     MRI->setRegClass(ArgReg, GR->getRegClass(ArgTypeVRegs[i]));
400:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpFunctionParameter)
401:                    .addDef(ArgReg)
402:                    .addUse(GR->getSPIRVTypeID(ArgTypeVRegs[i]));
403:     if (F.isDeclaration())
404:       GR->add(&Arg, MIB);
405:     GR->addGlobalObject(&Arg, &MIRBuilder.getMF(), ArgReg);
406:     i++;
407:   }
408:   // Name the function.
409:   if (F.hasName())
410:     buildOpName(FuncVReg, F.getName(), MIRBuilder);
411:
412:   // Handle entry points and function linkage.
413:   if (isEntryPoint(F)) {
414:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpEntryPoint)
415:                    .addImm(static_cast<uint32_t>(getExecutionModel(*ST, F)))
416:                    .addUse(FuncVReg);
417:     addStringImm(F.getName(), MIB);
418:   } else if (const auto LnkTy = getSpirvLinkageTypeFor(*ST, F)) {
419:     buildOpDecorate(FuncVReg, MIRBuilder, SPIRV::Decoration::LinkageAttributes,
420:                     {static_cast<uint32_t>(*LnkTy)}, F.getName());
```
- EN: This range implements operational logic in helpers such as getMRI, createGenericVirtualRegister, setRegClass, SPIRV::getOriginalFunctionType, translating backend policy into executable code.
- CN: 这一段实现了 getMRI、createGenericVirtualRegister、setRegClass、SPIRV::getOriginalFunctionType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:   }
422:
423:   // Handle function pointers decoration
424:   bool hasFunctionPointers =
425:       ST->canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers);
426:   if (hasFunctionPointers) {
427:     if (F.hasFnAttribute("referenced-indirectly")) {
428:       assert((F.getCallingConv() != CallingConv::SPIR_KERNEL) &&
429:              "Unexpected 'referenced-indirectly' attribute of the kernel "
430:              "function");
431:       buildOpDecorate(FuncVReg, MIRBuilder,
432:                       SPIRV::Decoration::ReferencedIndirectlyINTEL, {});
433:     }
434:   }
435:
436:   return true;
437: }
438:
439: // TODO:
440: // - add a topological sort of IndirectCalls to ensure the best types knowledge
441: // - we may need to fix function formal parameter types if they are opaque
442: //   pointers used as function pointers in these indirect calls
443: // - defaulting to StorageClass::Function in the absence of the
444: //   SPV_INTEL_function_pointers extension seems wrong, as that might not be
445: //   able to hold a full width pointer to function, and it also does not model
446: //   the semantics of a pointer to function in a generic fashion.
447: void SPIRVCallLowering::produceIndirectPtrType(
448:     MachineIRBuilder &MIRBuilder,
449:     const SPIRVCallLowering::SPIRVIndirectCall &IC) const {
450:   // Create indirect call data type if any
451:   MachineFunction &MF = MIRBuilder.getMF();
452:   const SPIRVSubtarget &ST = MF.getSubtarget<SPIRVSubtarget>();
453:   SPIRVTypeInst SpirvRetTy = GR->getOrCreateSPIRVType(
454:       IC.RetTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
455:   SmallVector<SPIRVTypeInst, 4> SpirvArgTypes;
456:   for (size_t i = 0; i < IC.ArgTys.size(); ++i) {
457:     SPIRVTypeInst SPIRVTy = GR->getOrCreateSPIRVType(
458:         IC.ArgTys[i], MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
459:     SpirvArgTypes.push_back(SPIRVTy);
460:     if (!GR->getSPIRVTypeForVReg(IC.ArgRegs[i]))
461:       GR->assignSPIRVTypeToVReg(SPIRVTy, IC.ArgRegs[i], MF);
462:   }
463:   // SPIR-V function type:
464:   FunctionType *FTy =
465:       FunctionType::get(const_cast<Type *>(IC.RetTy), IC.ArgTys, false);
466:   SPIRVTypeInst SpirvFuncTy = GR->getOrCreateOpTypeFunctionWithArgs(
467:       FTy, SpirvRetTy, SpirvArgTypes, MIRBuilder);
468:   // SPIR-V pointer to function type:
469:   auto SC = ST.canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers)
470:                 ? SPIRV::StorageClass::CodeSectionINTEL
471:                 : SPIRV::StorageClass::Function;
472:   SPIRVTypeInst IndirectFuncPtrTy =
473:       GR->getOrCreateSPIRVPointerType(SpirvFuncTy, MIRBuilder, SC);
474:   // Correct the Callee type
475:   GR->assignSPIRVTypeToVReg(IndirectFuncPtrTy, IC.Callee, MF);
476: }
477:
478: bool SPIRVCallLowering::lowerCall(MachineIRBuilder &MIRBuilder,
479:                                   CallLoweringInfo &Info) const {
480:   // Currently call returns should have single vregs.
```
- EN: This range implements operational logic in helpers such as canUseExtension, getMF, size, push_back, translating backend policy into executable code.
- CN: 这一段实现了 canUseExtension、getMF、size、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:   // TODO: handle the case of multiple registers.
482:   if (Info.OrigRet.Regs.size() > 1)
483:     return false;
484:   MachineFunction &MF = MIRBuilder.getMF();
485:   GR->setCurrentFunc(MF);
486:   const Function *CF = nullptr;
487:   std::string DemangledName;
488:   const Type *OrigRetTy = Info.OrigRet.Ty;
489:
490:   // Emit a regular OpFunctionCall. If it's an externally declared function,
491:   // be sure to emit its type and function declaration here. It will be hoisted
492:   // globally later.
493:   if (Info.Callee.isGlobal()) {
494:     std::string FuncName = Info.Callee.getGlobal()->getName().str();
495:     DemangledName = getOclOrSpirvBuiltinDemangledName(FuncName);
496:     CF = dyn_cast_or_null<const Function>(Info.Callee.getGlobal());
497:     // TODO: support constexpr casts and indirect calls.
498:     if (CF == nullptr)
499:       return false;
500:
501:     FunctionType *FTy = SPIRV::getOriginalFunctionType(*CF);
502:     OrigRetTy = FTy->getReturnType();
503:     if (isUntypedPointerTy(OrigRetTy)) {
504:       if (auto *DerivedRetTy = GR->findReturnType(CF))
505:         OrigRetTy = DerivedRetTy;
506:     }
507:   }
508:
509:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
510:   Register ResVReg =
511:       Info.OrigRet.Regs.empty() ? Register(0) : Info.OrigRet.Regs[0];
512:   const auto *ST = static_cast<const SPIRVSubtarget *>(&MF.getSubtarget());
513:
514:   bool isFunctionDecl = CF && CF->isDeclaration();
515:   if (isFunctionDecl && !DemangledName.empty()) {
516:     if (ResVReg.isValid()) {
517:       if (!GR->getSPIRVTypeForVReg(ResVReg)) {
518:         const Type *RetTy = OrigRetTy;
519:         if (auto *PtrRetTy = dyn_cast<PointerType>(OrigRetTy)) {
520:           const Value *OrigValue = Info.OrigRet.OrigValue;
521:           if (!OrigValue)
522:             OrigValue = Info.CB;
523:           if (OrigValue)
524:             if (Type *ElemTy = GR->findDeducedElementType(OrigValue))
525:               RetTy =
526:                   TypedPointerType::get(ElemTy, PtrRetTy->getAddressSpace());
527:         }
528:         setRegClassType(ResVReg, RetTy, GR, MIRBuilder,
529:                         SPIRV::AccessQualifier::ReadWrite, true);
530:       }
531:     } else {
532:       ResVReg = createVirtualRegister(OrigRetTy, GR, MIRBuilder,
533:                                       SPIRV::AccessQualifier::ReadWrite, true);
534:     }
535:     SmallVector<Register, 8> ArgVRegs;
536:     for (auto Arg : Info.OrigArgs) {
537:       assert(Arg.Regs.size() == 1 && "Call arg has multiple VRegs");
538:       Register ArgReg = Arg.Regs[0];
539:       ArgVRegs.push_back(ArgReg);
540:       SPIRVTypeInst SpvType = GR->getSPIRVTypeForVReg(ArgReg);
```
- EN: This range implements operational logic in helpers such as getMF, setCurrentFunc, getGlobal, getOclOrSpirvBuiltinDemangledName, translating backend policy into executable code.
- CN: 这一段实现了 getMF、setCurrentFunc、getGlobal、getOclOrSpirvBuiltinDemangledName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:       if (!SpvType) {
542:         Type *ArgTy = nullptr;
543:         if (auto *PtrArgTy = dyn_cast<PointerType>(Arg.Ty)) {
544:           // If Arg.Ty is an untyped pointer (i.e., ptr [addrspace(...)]) and we
545:           // don't have access to original value in LLVM IR or info about
546:           // deduced pointee type, then we should wait with setting the type for
547:           // the virtual register until pre-legalizer step when we access
548:           // @llvm.spv.assign.ptr.type.p...(...)'s info.
549:           if (Arg.OrigValue)
550:             if (Type *ElemTy = GR->findDeducedElementType(Arg.OrigValue))
551:               ArgTy =
552:                   TypedPointerType::get(ElemTy, PtrArgTy->getAddressSpace());
553:         } else {
554:           ArgTy = Arg.Ty;
555:         }
556:         if (ArgTy) {
557:           SpvType = GR->getOrCreateSPIRVType(
558:               ArgTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
559:           GR->assignSPIRVTypeToVReg(SpvType, ArgReg, MF);
560:         }
561:       }
562:       if (!MRI->getRegClassOrNull(ArgReg)) {
563:         // Either we have SpvType created, or Arg.Ty is an untyped pointer and
564:         // we know its virtual register's class and type even if we don't know
565:         // pointee type.
566:         MRI->setRegClass(ArgReg, SpvType ? GR->getRegClass(SpvType)
567:                                          : &SPIRV::pIDRegClass);
568:         MRI->setType(
569:             ArgReg,
570:             SpvType ? GR->getRegType(SpvType)
571:                     : LLT::pointer(cast<PointerType>(Arg.Ty)->getAddressSpace(),
572:                                    GR->getPointerSize()));
573:       }
574:     }
575:     if (auto Res = SPIRV::lowerBuiltin(
576:             DemangledName, ST->getPreferredInstructionSet(), MIRBuilder,
577:             ResVReg, OrigRetTy, ArgVRegs, GR, *Info.CB))
578:       return *Res;
579:   }
580:
581:   if (isFunctionDecl && !GR->find(CF, &MF).isValid()) {
582:     // Emit the type info and forward function declaration to the first MBB
583:     // to ensure VReg definition dependencies are valid across all MBBs.
584:     MachineIRBuilder FirstBlockBuilder;
585:     FirstBlockBuilder.setMF(MF);
586:     FirstBlockBuilder.setMBB(*MF.getBlockNumbered(0));
587:
588:     SmallVector<ArrayRef<Register>, 8> VRegArgs;
589:     SmallVector<SmallVector<Register, 1>, 8> ToInsert;
590:     for (const Argument &Arg : CF->args()) {
591:       if (MIRBuilder.getDataLayout().getTypeStoreSize(Arg.getType()).isZero())
592:         continue; // Don't handle zero sized types.
593:       Register Reg = MRI->createGenericVirtualRegister(LLT::scalar(64));
594:       MRI->setRegClass(Reg, &SPIRV::iIDRegClass);
595:       ToInsert.push_back({Reg});
596:       VRegArgs.push_back(ToInsert.back());
597:     }
598:     // TODO: Reuse FunctionLoweringInfo
599:     FunctionLoweringInfo FuncInfo;
600:     lowerFormalArguments(FirstBlockBuilder, *CF, VRegArgs, FuncInfo);
```
- EN: This range implements operational logic in helpers such as TypedPointerType::get, assignSPIRVTypeToVReg, setRegClass, getRegType, translating backend policy into executable code.
- CN: 这一段实现了 TypedPointerType::get、assignSPIRVTypeToVReg、setRegClass、getRegType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-660
```cpp
601:   }
602:
603:   // Ignore the call if it's called from the internal service function
604:   if (MIRBuilder.getMF()
605:           .getFunction()
606:           .getFnAttribute(SPIRV_BACKEND_SERVICE_FUN_NAME)
607:           .isValid()) {
608:     // insert a no-op
609:     MIRBuilder.buildTrap();
610:     return true;
611:   }
612:
613:   unsigned CallOp;
614:   if (Info.CB->isIndirectCall()) {
615:     if (!ST->canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers))
616:       report_fatal_error("An indirect call is encountered but SPIR-V without "
617:                          "extensions does not support it",
618:                          false);
619:     // Set instruction operation according to SPV_INTEL_function_pointers
620:     CallOp = SPIRV::OpFunctionPointerCallINTEL;
621:     // Collect information about the indirect call to create correct types.
622:     Register CalleeReg = Info.Callee.getReg();
623:     if (CalleeReg.isValid()) {
624:       SPIRVCallLowering::SPIRVIndirectCall IndirectCall;
625:       IndirectCall.Callee = CalleeReg;
626:       FunctionType *FTy = SPIRV::getOriginalFunctionType(*Info.CB);
627:       IndirectCall.RetTy = OrigRetTy = FTy->getReturnType();
628:       assert(FTy->getNumParams() == Info.OrigArgs.size() &&
629:              "Function types mismatch");
630:       for (unsigned I = 0; I != Info.OrigArgs.size(); ++I) {
631:         assert(Info.OrigArgs[I].Regs.size() == 1 &&
632:                "Call arg has multiple VRegs");
633:         IndirectCall.ArgTys.push_back(FTy->getParamType(I));
634:         IndirectCall.ArgRegs.push_back(Info.OrigArgs[I].Regs[0]);
635:       }
636:       produceIndirectPtrType(MIRBuilder, IndirectCall);
637:     }
638:   } else {
639:     // Emit a regular OpFunctionCall
640:     CallOp = SPIRV::OpFunctionCall;
641:   }
642:
643:   // Make sure there's a valid return reg, even for functions returning void.
644:   if (!ResVReg.isValid())
645:     ResVReg = MIRBuilder.getMRI()->createVirtualRegister(&SPIRV::iIDRegClass);
646:   SPIRVTypeInst RetType = GR->assignTypeToVReg(
647:       OrigRetTy, ResVReg, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
648:
649:   // Emit the call instruction and its args.
650:   auto MIB = MIRBuilder.buildInstr(CallOp)
651:                  .addDef(ResVReg)
652:                  .addUse(GR->getSPIRVTypeID(RetType))
653:                  .add(Info.Callee);
654:
655:   for (const auto &Arg : Info.OrigArgs) {
656:     // Currently call args should have single vregs.
657:     if (Arg.Regs.size() > 1)
658:       return false;
659:     MIB.addUse(Arg.Regs[0]);
660:   }
```
- EN: This range implements operational logic in helpers such as getFunction, getFnAttribute, isValid, buildTrap, translating backend policy into executable code.
- CN: 这一段实现了 getFunction、getFnAttribute、isValid、buildTrap 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 661-678
```cpp
661:
662:   if (ST->canUseExtension(SPIRV::Extension::SPV_INTEL_memory_access_aliasing)) {
663:     // Process aliasing metadata.
664:     const CallBase *CI = Info.CB;
665:     if (CI && CI->hasMetadata()) {
666:       if (MDNode *MD = CI->getMetadata(LLVMContext::MD_alias_scope))
667:         GR->buildMemAliasingOpDecorate(ResVReg, MIRBuilder,
668:                                        SPIRV::Decoration::AliasScopeINTEL, MD);
669:       if (MDNode *MD = CI->getMetadata(LLVMContext::MD_noalias))
670:         GR->buildMemAliasingOpDecorate(ResVReg, MIRBuilder,
671:                                        SPIRV::Decoration::NoAliasINTEL, MD);
672:     }
673:   }
674:
675:   MIB.constrainAllUses(MIRBuilder.getTII(), *ST->getRegisterInfo(),
676:                        *ST->getRegBankInfo());
677:   return true;
678: }
```
- EN: This range implements operational logic in helpers such as backend logic, translating backend policy into executable code.
- CN: 这一段实现了 后端逻辑 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include CallLowering, getFunction, getFnAttribute, isValid, getMF, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 CallLowering, getFunction, getFnAttribute, isValid, getMF，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVCallLowering.h`
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRV.h`
  - `SPIRVBuiltins.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVISelLowering.h`
  - `SPIRVMetadata.h`
  - `SPIRVRegisterInfo.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/FunctionLoweringInfo.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/Support/ModRef.h`
