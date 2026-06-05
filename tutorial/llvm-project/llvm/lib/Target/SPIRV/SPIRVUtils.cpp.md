# SPIRVUtils.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVUtils.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains miscellaneous utility functions.
- 目的（中文）: 该文件提供 LLVM 目标后端所需的辅助逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===--- SPIRVUtils.cpp ---- SPIR-V Utility Functions -----------*- C++ -*-===//
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
13: #include "SPIRVUtils.h"
14: #include "MCTargetDesc/SPIRVBaseInfo.h"
15: #include "SPIRV.h"
16: #include "SPIRVGlobalRegistry.h"
17: #include "SPIRVInstrInfo.h"
18: #include "SPIRVSubtarget.h"
19: #include "llvm/ADT/StringRef.h"
20: #include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
21: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
22: #include "llvm/CodeGen/MachineInstr.h"
23: #include "llvm/CodeGen/MachineInstrBuilder.h"
24: #include "llvm/Demangle/Demangle.h"
25: #include "llvm/IR/IntrinsicInst.h"
26: #include "llvm/IR/IntrinsicsSPIRV.h"
27: #include <queue>
28: #include <vector>
29:
30: namespace llvm {
31: namespace SPIRV {
32: // This code restores function args/retvalue types for composite cases
33: // because the final types should still be aggregate whereas they're i32
34: // during the translation to cope with aggregate flattening etc.
35: // TODO: should these just return nullptr when there's no metadata?
36: static FunctionType *extractFunctionTypeFromMetadata(NamedMDNode *NMD,
37:                                                      FunctionType *FTy,
38:                                                      StringRef Name) {
39:   if (!NMD)
40:     return FTy;
41:
42:   constexpr auto getConstInt = [](MDNode *MD, unsigned OpId) -> ConstantInt * {
43:     if (MD->getNumOperands() <= OpId)
44:       return nullptr;
45:     if (auto *CMeta = dyn_cast<ConstantAsMetadata>(MD->getOperand(OpId)))
46:       return dyn_cast<ConstantInt>(CMeta->getValue());
47:     return nullptr;
48:   };
49:
50:   auto It = find_if(NMD->operands(), [Name](MDNode *N) {
51:     if (auto *MDS = dyn_cast_or_null<MDString>(N->getOperand(0)))
52:       return MDS->getString() == Name;
53:     return false;
54:   });
55:
56:   if (It == NMD->op_end())
57:     return FTy;
58:
59:   Type *RetTy = FTy->getReturnType();
60:   SmallVector<Type *, 4> PTys(FTy->params());
61:
62:   for (unsigned I = 1; I != (*It)->getNumOperands(); ++I) {
63:     MDNode *MD = dyn_cast<MDNode>((*It)->getOperand(I));
64:     assert(MD && "MDNode operand is expected");
65:
66:     if (auto *Const = getConstInt(MD, 0)) {
67:       auto *CMeta = dyn_cast<ConstantAsMetadata>(MD->getOperand(1));
68:       assert(CMeta && "ConstantAsMetadata operand is expected");
69:       assert(Const->getSExtValue() >= -1);
70:       // Currently -1 indicates return value, greater values mean
71:       // argument numbers.
72:       if (Const->getSExtValue() == -1)
73:         RetTy = CMeta->getType();
74:       else
75:         PTys[Const->getSExtValue()] = CMeta->getType();
76:     }
77:   }
78:
79:   return FunctionType::get(RetTy, PTys, FTy->isVarArg());
80: }
81:
82: static StringRef extractAsmConstraintsFromMetadata(NamedMDNode *NMD,
83:                                                    StringRef Constraints,
84:                                                    StringRef Name) {
85:   // TODO: unify the extractors.
86:   if (!NMD)
87:     return Constraints;
88:
89:   auto It = find_if(NMD->operands(), [Name](MDNode *N) {
90:     if (auto *MDS = dyn_cast_or_null<MDString>(N->getOperand(0)))
```
- EN: This range implements operational logic in helpers such as getValue, find_if, getReturnType, PTys, translating backend policy into executable code.
- CN: 这一段实现了 getValue、find_if、getReturnType、PTys 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 91-180
```cpp
 91:       return MDS->getString() == Name;
 92:     return false;
 93:   });
 94:
 95:   if (It == NMD->op_end())
 96:     return Constraints;
 97:
 98:   // By convention, the constraints string is stored in the final MD operand.
 99:   MDNode *MD = dyn_cast<MDNode>((*It)->getOperand((*It)->getNumOperands() - 1));
100:   assert(MD && "MDNode operand is expected");
101:
102:   if (auto *MDS = dyn_cast<MDString>(MD->getOperand(0)))
103:     Constraints = MDS->getString();
104:
105:   return Constraints;
106: }
107:
108: FunctionType *getOriginalFunctionType(const Function &F) {
109:   return extractFunctionTypeFromMetadata(
110:       F.getParent()->getNamedMetadata("spv.cloned_funcs"), F.getFunctionType(),
111:       F.getName());
112: }
113:
114: FunctionType *getOriginalFunctionType(const CallBase &CB) {
115:   return extractFunctionTypeFromMetadata(
116:       CB.getModule()->getNamedMetadata("spv.mutated_callsites"),
117:       CB.getFunctionType(), CB.getName());
118: }
119:
120: StringRef getOriginalAsmConstraints(const CallBase &CB) {
121:   return extractAsmConstraintsFromMetadata(
122:       CB.getModule()->getNamedMetadata("spv.mutated_callsites"),
123:       cast<InlineAsm>(CB.getCalledOperand())->getConstraintString(),
124:       CB.getName());
125: }
126: } // Namespace SPIRV
127:
128: // The following functions are used to add these string literals as a series of
129: // 32-bit integer operands with the correct format, and unpack them if necessary
130: // when making string comparisons in compiler passes.
131: // SPIR-V requires null-terminated UTF-8 strings padded to 32-bit alignment.
132: static uint32_t convertCharsToWord(const StringRef &Str, unsigned i) {
133:   uint32_t Word = 0u; // Build up this 32-bit word from 4 8-bit chars.
134:   for (unsigned WordIndex = 0; WordIndex < 4; ++WordIndex) {
135:     unsigned StrIndex = i + WordIndex;
136:     uint8_t CharToAdd = 0;       // Initilize char as padding/null.
137:     if (StrIndex < Str.size()) { // If it's within the string, get a real char.
138:       CharToAdd = Str[StrIndex];
139:     }
140:     Word |= (CharToAdd << (WordIndex * 8));
141:   }
142:   return Word;
143: }
144:
145: // Get length including padding and null terminator.
146: static size_t getPaddedLen(const StringRef &Str) {
147:   return (Str.size() + 4) & ~3;
148: }
149:
150: void addStringImm(const StringRef &Str, MCInst &Inst) {
151:   const size_t PaddedLen = getPaddedLen(Str);
152:   for (unsigned i = 0; i < PaddedLen; i += 4) {
153:     // Add an operand for the 32-bits of chars or padding.
154:     Inst.addOperand(MCOperand::createImm(convertCharsToWord(Str, i)));
155:   }
156: }
157:
158: void addStringImm(const StringRef &Str, MachineInstrBuilder &MIB) {
159:   const size_t PaddedLen = getPaddedLen(Str);
160:   for (unsigned i = 0; i < PaddedLen; i += 4) {
161:     // Add an operand for the 32-bits of chars or padding.
162:     MIB.addImm(convertCharsToWord(Str, i));
163:   }
164: }
165:
166: void addStringImm(const StringRef &Str, IRBuilder<> &B,
167:                   std::vector<Value *> &Args) {
168:   const size_t PaddedLen = getPaddedLen(Str);
169:   for (unsigned i = 0; i < PaddedLen; i += 4) {
170:     // Add a vector element for the 32-bits of chars or padding.
171:     Args.push_back(B.getInt32(convertCharsToWord(Str, i)));
172:   }
173: }
174:
175: std::string getStringImm(const MachineInstr &MI, unsigned StartIndex) {
176:   return getSPIRVStringOperand(MI, StartIndex);
177: }
178:
179: std::string getStringValueFromReg(Register Reg, MachineRegisterInfo &MRI) {
180:   MachineInstr *Def = getVRegDef(MRI, Reg);
```
- EN: This range implements operational logic in helpers such as getOperand, assert, getString, getOriginalFunctionType, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、assert、getString、getOriginalFunctionType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:   assert(Def && Def->getOpcode() == TargetOpcode::G_GLOBAL_VALUE &&
182:          "Expected G_GLOBAL_VALUE");
183:   const GlobalValue *GV = Def->getOperand(1).getGlobal();
184:   Value *V = GV->getOperand(0);
185:   const ConstantDataArray *CDA = cast<ConstantDataArray>(V);
186:   return CDA->getAsCString().str();
187: }
188:
189: void addNumImm(const APInt &Imm, MachineInstrBuilder &MIB) {
190:   const auto Bitwidth = Imm.getBitWidth();
191:   if (Bitwidth == 1)
192:     return; // Already handled
193:   else if (Bitwidth <= 32) {
194:     MIB.addImm(Imm.getZExtValue());
195:     // Asm Printer needs this info to print floating-type correctly
196:     if (Bitwidth == 16)
197:       MIB.getInstr()->setAsmPrinterFlag(SPIRV::ASM_PRINTER_WIDTH16);
198:     return;
199:   } else if (Bitwidth <= 64) {
200:     uint64_t FullImm = Imm.getZExtValue();
201:     uint32_t LowBits = FullImm & 0xffffffff;
202:     uint32_t HighBits = (FullImm >> 32) & 0xffffffff;
203:     MIB.addImm(LowBits).addImm(HighBits);
204:     // Asm Printer needs this info to print 64-bit operands correctly
205:     MIB.getInstr()->setAsmPrinterFlag(SPIRV::ASM_PRINTER_WIDTH64);
206:     return;
207:   } else {
208:     // Emit ceil(Bitwidth / 32) words to conform SPIR-V spec.
209:     unsigned NumWords = (Bitwidth + 31) / 32;
210:     for (unsigned I = 0; I < NumWords; ++I) {
211:       unsigned LimbIdx = I / 2;
212:       unsigned LimbShift = (I % 2) * 32;
213:       uint32_t Word = (Imm.getRawData()[LimbIdx] >> LimbShift) & 0xffffffff;
214:       MIB.addImm(Word);
215:     }
216:     return;
217:   }
218: }
219:
220: void buildOpName(Register Target, const StringRef &Name,
221:                  MachineIRBuilder &MIRBuilder) {
222:   if (!Name.empty()) {
223:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpName).addUse(Target);
224:     addStringImm(Name, MIB);
225:   }
226: }
227:
228: void buildOpName(Register Target, const StringRef &Name, MachineInstr &I,
229:                  const SPIRVInstrInfo &TII) {
230:   if (!Name.empty()) {
231:     auto MIB =
232:         BuildMI(*I.getParent(), I, I.getDebugLoc(), TII.get(SPIRV::OpName))
233:             .addUse(Target);
234:     addStringImm(Name, MIB);
235:   }
236: }
237:
238: static void finishBuildOpDecorate(MachineInstrBuilder &MIB,
239:                                   const std::vector<uint32_t> &DecArgs,
240:                                   StringRef StrImm) {
241:   if (!StrImm.empty())
242:     addStringImm(StrImm, MIB);
243:   for (const auto &DecArg : DecArgs)
244:     MIB.addImm(DecArg);
245: }
246:
247: void buildOpDecorate(Register Reg, MachineIRBuilder &MIRBuilder,
248:                      SPIRV::Decoration::Decoration Dec,
249:                      const std::vector<uint32_t> &DecArgs, StringRef StrImm) {
250:   auto MIB = MIRBuilder.buildInstr(SPIRV::OpDecorate)
251:                  .addUse(Reg)
252:                  .addImm(static_cast<uint32_t>(Dec));
253:   finishBuildOpDecorate(MIB, DecArgs, StrImm);
254: }
255:
256: void buildOpDecorate(Register Reg, MachineInstr &I, const SPIRVInstrInfo &TII,
257:                      SPIRV::Decoration::Decoration Dec,
258:                      const std::vector<uint32_t> &DecArgs, StringRef StrImm) {
259:   MachineBasicBlock &MBB = *I.getParent();
260:   auto MIB = BuildMI(MBB, I, I.getDebugLoc(), TII.get(SPIRV::OpDecorate))
261:                  .addUse(Reg)
262:                  .addImm(static_cast<uint32_t>(Dec));
263:   finishBuildOpDecorate(MIB, DecArgs, StrImm);
264: }
265:
266: void buildOpMemberDecorate(Register Reg, MachineIRBuilder &MIRBuilder,
267:                            SPIRV::Decoration::Decoration Dec, uint32_t Member,
268:                            const std::vector<uint32_t> &DecArgs,
269:                            StringRef StrImm) {
270:   auto MIB = MIRBuilder.buildInstr(SPIRV::OpMemberDecorate)
```
- EN: This range implements operational logic in helpers such as getOperand, getAsCString, addNumImm, getBitWidth, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getAsCString、addNumImm、getBitWidth 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:                  .addUse(Reg)
272:                  .addImm(Member)
273:                  .addImm(static_cast<uint32_t>(Dec));
274:   finishBuildOpDecorate(MIB, DecArgs, StrImm);
275: }
276:
277: void buildOpMemberDecorate(Register Reg, MachineInstr &I,
278:                            const SPIRVInstrInfo &TII,
279:                            SPIRV::Decoration::Decoration Dec, uint32_t Member,
280:                            const std::vector<uint32_t> &DecArgs,
281:                            StringRef StrImm) {
282:   MachineBasicBlock &MBB = *I.getParent();
283:   auto MIB = BuildMI(MBB, I, I.getDebugLoc(), TII.get(SPIRV::OpMemberDecorate))
284:                  .addUse(Reg)
285:                  .addImm(Member)
286:                  .addImm(static_cast<uint32_t>(Dec));
287:   finishBuildOpDecorate(MIB, DecArgs, StrImm);
288: }
289:
290: void buildOpSpirvDecorations(Register Reg, MachineIRBuilder &MIRBuilder,
291:                              const MDNode *GVarMD, const SPIRVSubtarget &ST) {
292:   for (unsigned I = 0, E = GVarMD->getNumOperands(); I != E; ++I) {
293:     auto *OpMD = dyn_cast<MDNode>(GVarMD->getOperand(I));
294:     if (!OpMD)
295:       report_fatal_error("Invalid decoration");
296:     if (OpMD->getNumOperands() == 0)
297:       report_fatal_error("Expect operand(s) of the decoration");
298:     ConstantInt *DecorationId =
299:         mdconst::dyn_extract<ConstantInt>(OpMD->getOperand(0));
300:     if (!DecorationId)
301:       report_fatal_error("Expect SPIR-V <Decoration> operand to be the first "
302:                          "element of the decoration");
303:
304:     // The goal of `spirv.Decorations` metadata is to provide a way to
305:     // represent SPIR-V entities that do not map to LLVM in an obvious way.
306:     // FP flags do have obvious matches between LLVM IR and SPIR-V.
307:     // Additionally, we have no guarantee at this point that the flags passed
308:     // through the decoration are not violated already in the optimizer passes.
309:     // Therefore, we simply ignore FP flags, including NoContraction, and
310:     // FPFastMathMode.
311:     if (DecorationId->getZExtValue() ==
312:             static_cast<uint32_t>(SPIRV::Decoration::NoContraction) ||
313:         DecorationId->getZExtValue() ==
314:             static_cast<uint32_t>(SPIRV::Decoration::FPFastMathMode)) {
315:       continue; // Ignored.
316:     }
317:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpDecorate)
318:                    .addUse(Reg)
319:                    .addImm(static_cast<uint32_t>(DecorationId->getZExtValue()));
320:     for (unsigned OpI = 1, OpE = OpMD->getNumOperands(); OpI != OpE; ++OpI) {
321:       if (ConstantInt *OpV =
322:               mdconst::dyn_extract<ConstantInt>(OpMD->getOperand(OpI)))
323:         MIB.addImm(static_cast<uint32_t>(OpV->getZExtValue()));
324:       else if (MDString *OpV = dyn_cast<MDString>(OpMD->getOperand(OpI)))
325:         addStringImm(OpV->getString(), MIB);
326:       else
327:         report_fatal_error("Unexpected operand of the decoration");
328:     }
329:   }
330: }
331:
332: MachineBasicBlock::iterator getOpVariableMBBIt(MachineFunction &MF) {
333:   MachineBasicBlock &MBB = MF.front();
334:   // Find the position to insert the OpVariable instruction.
335:   // We will insert it after the last OpFunctionParameter, if any, or
336:   // after OpFunction otherwise.
337:   auto IsPreamble = [](const MachineInstr &MI) {
338:     switch (MI.getOpcode()) {
339:     case SPIRV::OpFunction:
340:     case SPIRV::OpFunctionParameter:
341:     case SPIRV::OpLabel:
342:     case SPIRV::ASSIGN_TYPE:
343:       return true;
344:     default:
345:       return false;
346:     }
347:   };
348:   MachineBasicBlock::iterator VarPos = MBB.SkipPHIsAndLabels(MBB.begin());
349:   while (VarPos != MBB.end() && VarPos->getOpcode() != SPIRV::OpFunction)
350:     ++VarPos;
351:   // Advance past the preamble.
352:   while (VarPos != MBB.end() && IsPreamble(*VarPos))
353:     ++VarPos;
354:   return VarPos;
355: }
356:
357: MachineBasicBlock::iterator getInsertPtValidEnd(MachineBasicBlock *MBB) {
358:   MachineBasicBlock::iterator I = MBB->end();
359:   if (I == MBB->begin())
360:     return I;
```
- EN: This range implements operational logic in helpers such as addUse, addImm, finishBuildOpDecorate, getParent, translating backend policy into executable code.
- CN: 这一段实现了 addUse、addImm、finishBuildOpDecorate、getParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:   --I;
362:   while (I->isTerminator() || I->isDebugValue()) {
363:     if (I == MBB->begin())
364:       break;
365:     --I;
366:   }
367:   return I;
368: }
369:
370: SPIRV::StorageClass::StorageClass
371: addressSpaceToStorageClass(unsigned AddrSpace, const SPIRVSubtarget &STI) {
372:   switch (AddrSpace) {
373:   case 0:
374:     return SPIRV::StorageClass::Function;
375:   case 1:
376:     return SPIRV::StorageClass::CrossWorkgroup;
377:   case 2:
378:     return SPIRV::StorageClass::UniformConstant;
379:   case 3:
380:     return SPIRV::StorageClass::Workgroup;
381:   case 4:
382:     return SPIRV::StorageClass::Generic;
383:   case 5:
384:     return STI.canUseExtension(SPIRV::Extension::SPV_INTEL_usm_storage_classes)
385:                ? SPIRV::StorageClass::DeviceOnlyINTEL
386:                : SPIRV::StorageClass::CrossWorkgroup;
387:   case 6:
388:     return STI.canUseExtension(SPIRV::Extension::SPV_INTEL_usm_storage_classes)
389:                ? SPIRV::StorageClass::HostOnlyINTEL
390:                : SPIRV::StorageClass::CrossWorkgroup;
391:   case 7:
392:     return SPIRV::StorageClass::Input;
393:   case 8:
394:     return SPIRV::StorageClass::Output;
395:   case 9:
396:     return SPIRV::StorageClass::CodeSectionINTEL;
397:   case 10:
398:     return SPIRV::StorageClass::Private;
399:   case 11:
400:     return SPIRV::StorageClass::StorageBuffer;
401:   case 12:
402:     return SPIRV::StorageClass::Uniform;
403:   case 13:
404:     return SPIRV::StorageClass::PushConstant;
405:   default:
406:     report_fatal_error("Unknown address space");
407:   }
408: }
409:
410: SPIRV::MemorySemantics::MemorySemantics
411: getMemSemanticsForStorageClass(SPIRV::StorageClass::StorageClass SC) {
412:   switch (SC) {
413:   case SPIRV::StorageClass::StorageBuffer:
414:   case SPIRV::StorageClass::Uniform:
415:     return SPIRV::MemorySemantics::UniformMemory;
416:   case SPIRV::StorageClass::Workgroup:
417:     return SPIRV::MemorySemantics::WorkgroupMemory;
418:   case SPIRV::StorageClass::CrossWorkgroup:
419:     return SPIRV::MemorySemantics::CrossWorkgroupMemory;
420:   case SPIRV::StorageClass::AtomicCounter:
421:     return SPIRV::MemorySemantics::AtomicCounterMemory;
422:   case SPIRV::StorageClass::Image:
423:     return SPIRV::MemorySemantics::ImageMemory;
424:   default:
425:     return SPIRV::MemorySemantics::None;
426:   }
427: }
428:
429: SPIRV::MemorySemantics::MemorySemantics getMemSemantics(AtomicOrdering Ord) {
430:   switch (Ord) {
431:   case AtomicOrdering::Acquire:
432:     return SPIRV::MemorySemantics::Acquire;
433:   case AtomicOrdering::Release:
434:     return SPIRV::MemorySemantics::Release;
435:   case AtomicOrdering::AcquireRelease:
436:     return SPIRV::MemorySemantics::AcquireRelease;
437:   case AtomicOrdering::SequentiallyConsistent:
438:     return SPIRV::MemorySemantics::SequentiallyConsistent;
439:   case AtomicOrdering::Unordered:
440:   case AtomicOrdering::Monotonic:
441:   case AtomicOrdering::NotAtomic:
442:     return SPIRV::MemorySemantics::None;
443:   }
444:   llvm_unreachable(nullptr);
445: }
446:
447: SPIRV::Scope::Scope getMemScope(LLVMContext &Ctx, SyncScope::ID Id) {
448:   // Named by
449:   // https://registry.khronos.org/SPIR-V/specs/unified1/SPIRV.html#_scope_id.
450:   // We don't need aliases for Invocation and CrossDevice, as we already have
```
- EN: This range implements operational logic in helpers such as addressSpaceToStorageClass, canUseExtension, report_fatal_error, getMemSemanticsForStorageClass, translating backend policy into executable code.
- CN: 这一段实现了 addressSpaceToStorageClass、canUseExtension、report_fatal_error、getMemSemanticsForStorageClass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:   // them covered by "singlethread" and "" strings respectively (see
452:   // implementation of LLVMContext::LLVMContext()).
453:   static const llvm::SyncScope::ID SubGroup =
454:       Ctx.getOrInsertSyncScopeID("subgroup");
455:   static const llvm::SyncScope::ID WorkGroup =
456:       Ctx.getOrInsertSyncScopeID("workgroup");
457:   static const llvm::SyncScope::ID Device =
458:       Ctx.getOrInsertSyncScopeID("device");
459:
460:   if (Id == llvm::SyncScope::SingleThread)
461:     return SPIRV::Scope::Invocation;
462:   else if (Id == llvm::SyncScope::System)
463:     return SPIRV::Scope::CrossDevice;
464:   else if (Id == SubGroup)
465:     return SPIRV::Scope::Subgroup;
466:   else if (Id == WorkGroup)
467:     return SPIRV::Scope::Workgroup;
468:   else if (Id == Device)
469:     return SPIRV::Scope::Device;
470:   return SPIRV::Scope::CrossDevice;
471: }
472:
473: MachineInstr *getDefInstrMaybeConstant(Register &ConstReg,
474:                                        const MachineRegisterInfo *MRI) {
475:   MachineInstr *MI = MRI->getVRegDef(ConstReg);
476:   MachineInstr *ConstInstr =
477:       MI->getOpcode() == SPIRV::G_TRUNC || MI->getOpcode() == SPIRV::G_ZEXT
478:           ? MRI->getVRegDef(MI->getOperand(1).getReg())
479:           : MI;
480:   if (auto *GI = dyn_cast<GIntrinsic>(ConstInstr)) {
481:     if (GI->is(Intrinsic::spv_track_constant)) {
482:       ConstReg = ConstInstr->getOperand(2).getReg();
483:       return MRI->getVRegDef(ConstReg);
484:     }
485:   } else if (ConstInstr->getOpcode() == SPIRV::ASSIGN_TYPE) {
486:     ConstReg = ConstInstr->getOperand(1).getReg();
487:     return MRI->getVRegDef(ConstReg);
488:   } else if (ConstInstr->getOpcode() == TargetOpcode::G_CONSTANT ||
489:              ConstInstr->getOpcode() == TargetOpcode::G_FCONSTANT) {
490:     ConstReg = ConstInstr->getOperand(0).getReg();
491:     return ConstInstr;
492:   }
493:   return MRI->getVRegDef(ConstReg);
494: }
495:
496: uint64_t getIConstVal(Register ConstReg, const MachineRegisterInfo *MRI) {
497:   const MachineInstr *MI = getDefInstrMaybeConstant(ConstReg, MRI);
498:   assert(MI && MI->getOpcode() == TargetOpcode::G_CONSTANT);
499:   return MI->getOperand(1).getCImm()->getValue().getZExtValue();
500: }
501:
502: int64_t getIConstValSext(Register ConstReg, const MachineRegisterInfo *MRI) {
503:   const MachineInstr *MI = getDefInstrMaybeConstant(ConstReg, MRI);
504:   assert(MI && MI->getOpcode() == TargetOpcode::G_CONSTANT);
505:   return MI->getOperand(1).getCImm()->getSExtValue();
506: }
507:
508: bool isSpvIntrinsic(const MachineInstr &MI, Intrinsic::ID IntrinsicID) {
509:   if (const auto *GI = dyn_cast<GIntrinsic>(&MI))
510:     return GI->is(IntrinsicID);
511:   return false;
512: }
513:
514: Type *getMDOperandAsType(const MDNode *N, unsigned I) {
515:   Type *ElementTy = cast<ValueAsMetadata>(N->getOperand(I))->getType();
516:   return toTypedPointer(ElementTy);
517: }
518:
519: // The set of names is borrowed from the SPIR-V translator.
520: // TODO: may be implemented in SPIRVBuiltins.td.
521: static bool isPipeOrAddressSpaceCastBI(const StringRef MangledName) {
522:   return MangledName == "write_pipe_2" || MangledName == "read_pipe_2" ||
523:          MangledName == "write_pipe_2_bl" || MangledName == "read_pipe_2_bl" ||
524:          MangledName == "write_pipe_4" || MangledName == "read_pipe_4" ||
525:          MangledName == "reserve_write_pipe" ||
526:          MangledName == "reserve_read_pipe" ||
527:          MangledName == "commit_write_pipe" ||
528:          MangledName == "commit_read_pipe" ||
529:          MangledName == "work_group_reserve_write_pipe" ||
530:          MangledName == "work_group_reserve_read_pipe" ||
531:          MangledName == "work_group_commit_write_pipe" ||
532:          MangledName == "work_group_commit_read_pipe" ||
533:          MangledName == "get_pipe_num_packets_ro" ||
534:          MangledName == "get_pipe_max_packets_ro" ||
535:          MangledName == "get_pipe_num_packets_wo" ||
536:          MangledName == "get_pipe_max_packets_wo" ||
537:          MangledName == "sub_group_reserve_write_pipe" ||
538:          MangledName == "sub_group_reserve_read_pipe" ||
539:          MangledName == "sub_group_commit_write_pipe" ||
540:          MangledName == "sub_group_commit_read_pipe" ||
```
- EN: This range implements operational logic in helpers such as getOrInsertSyncScopeID, getVRegDef, getOperand, getOpcode, translating backend policy into executable code.
- CN: 这一段实现了 getOrInsertSyncScopeID、getVRegDef、getOperand、getOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:          MangledName == "to_global" || MangledName == "to_local" ||
542:          MangledName == "to_private";
543: }
544:
545: static bool isEnqueueKernelBI(const StringRef MangledName) {
546:   return MangledName == "__enqueue_kernel_basic" ||
547:          MangledName == "__enqueue_kernel_basic_events" ||
548:          MangledName == "__enqueue_kernel_varargs" ||
549:          MangledName == "__enqueue_kernel_events_varargs";
550: }
551:
552: static bool isKernelQueryBI(const StringRef MangledName) {
553:   return MangledName == "__get_kernel_work_group_size_impl" ||
554:          MangledName == "__get_kernel_sub_group_count_for_ndrange_impl" ||
555:          MangledName == "__get_kernel_max_sub_group_size_for_ndrange_impl" ||
556:          MangledName == "__get_kernel_preferred_work_group_size_multiple_impl";
557: }
558:
559: static bool isNonMangledOCLBuiltin(StringRef Name) {
560:   if (!Name.starts_with("__"))
561:     return false;
562:
563:   return isEnqueueKernelBI(Name) || isKernelQueryBI(Name) ||
564:          isPipeOrAddressSpaceCastBI(Name.drop_front(2)) ||
565:          Name == "__translate_sampler_initializer";
566: }
567:
568: std::string getOclOrSpirvBuiltinDemangledName(StringRef Name) {
569:   bool IsNonMangledOCL = isNonMangledOCLBuiltin(Name);
570:   bool IsNonMangledSPIRV = Name.starts_with("__spirv_");
571:   bool IsNonMangledHLSL = Name.starts_with("__hlsl_");
572:   bool IsMangled = Name.starts_with("_Z");
573:
574:   // Otherwise use simple demangling to return the function name.
575:   if (IsNonMangledOCL || IsNonMangledSPIRV || IsNonMangledHLSL || !IsMangled)
576:     return Name.str();
577:
578:   // Try to use the itanium demangler.
579:   if (char *DemangledName = itaniumDemangle(Name.data())) {
580:     std::string Result = DemangledName;
581:     free(DemangledName);
582:     return Result;
583:   }
584:
585:   // Autocheck C++, maybe need to do explicit check of the source language.
586:   // OpenCL C++ built-ins are declared in cl namespace.
587:   // TODO: consider using 'St' abbriviation for cl namespace mangling.
588:   // Similar to ::std:: in C++.
589:   size_t Start, Len = 0;
590:   size_t DemangledNameLenStart = 2;
591:   if (Name.starts_with("_ZN")) {
592:     // Skip CV and ref qualifiers.
593:     size_t NameSpaceStart = Name.find_first_not_of("rVKRO", 3);
594:     // All built-ins are in the ::cl:: namespace.
595:     if (Name.substr(NameSpaceStart, 11) != "2cl7__spirv")
596:       return std::string();
597:     DemangledNameLenStart = NameSpaceStart + 11;
598:   }
599:   Start = Name.find_first_not_of("0123456789", DemangledNameLenStart);
600:   [[maybe_unused]] bool Error =
601:       Name.substr(DemangledNameLenStart, Start - DemangledNameLenStart)
602:           .getAsInteger(10, Len);
603:   assert(!Error && "Failed to parse demangled name length");
604:   return Name.substr(Start, Len).str();
605: }
606:
607: bool hasBuiltinTypePrefix(StringRef Name) {
608:   if (Name.starts_with("opencl.") || Name.starts_with("ocl_") ||
609:       Name.starts_with("spirv."))
610:     return true;
611:   return false;
612: }
613:
614: bool isSpecialOpaqueType(const Type *Ty) {
615:   if (const TargetExtType *ExtTy = dyn_cast<TargetExtType>(Ty))
616:     return isTypedPointerWrapper(ExtTy)
617:                ? false
618:                : hasBuiltinTypePrefix(ExtTy->getName());
619:
620:   return false;
621: }
622:
623: bool isEntryPoint(const Function &F) {
624:   // OpenCL handling: any function with the SPIR_KERNEL
625:   // calling convention will be a potential entry point.
626:   if (F.getCallingConv() == CallingConv::SPIR_KERNEL)
627:     return true;
628:
629:   // HLSL handling: special attribute are emitted from the
630:   // front-end.
```
- EN: This range implements operational logic in helpers such as isEnqueueKernelBI, isKernelQueryBI, isNonMangledOCLBuiltin, getOclOrSpirvBuiltinDemangledName, translating backend policy into executable code.
- CN: 这一段实现了 isEnqueueKernelBI、isKernelQueryBI、isNonMangledOCLBuiltin、getOclOrSpirvBuiltinDemangledName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:   if (F.getFnAttribute("hlsl.shader").isValid())
632:     return true;
633:
634:   return false;
635: }
636:
637: Type *parseBasicTypeName(StringRef &TypeName, LLVMContext &Ctx) {
638:   TypeName.consume_front("atomic_");
639:   if (TypeName.consume_front("void"))
640:     return Type::getVoidTy(Ctx);
641:   else if (TypeName.consume_front("bool") || TypeName.consume_front("_Bool"))
642:     return Type::getIntNTy(Ctx, 1);
643:   else if (TypeName.consume_front("char") ||
644:            TypeName.consume_front("signed char") ||
645:            TypeName.consume_front("unsigned char") ||
646:            TypeName.consume_front("uchar"))
647:     return Type::getInt8Ty(Ctx);
648:   else if (TypeName.consume_front("short") ||
649:            TypeName.consume_front("signed short") ||
650:            TypeName.consume_front("unsigned short") ||
651:            TypeName.consume_front("ushort"))
652:     return Type::getInt16Ty(Ctx);
653:   else if (TypeName.consume_front("int") ||
654:            TypeName.consume_front("signed int") ||
655:            TypeName.consume_front("unsigned int") ||
656:            TypeName.consume_front("uint"))
657:     return Type::getInt32Ty(Ctx);
658:   else if (TypeName.consume_front("long") ||
659:            TypeName.consume_front("signed long") ||
660:            TypeName.consume_front("unsigned long") ||
661:            TypeName.consume_front("ulong"))
662:     return Type::getInt64Ty(Ctx);
663:   else if (TypeName.consume_front("half") ||
664:            TypeName.consume_front("_Float16") ||
665:            TypeName.consume_front("__fp16"))
666:     return Type::getHalfTy(Ctx);
667:   else if (TypeName.consume_front("float"))
668:     return Type::getFloatTy(Ctx);
669:   else if (TypeName.consume_front("double"))
670:     return Type::getDoubleTy(Ctx);
671:
672:   // Unable to recognize SPIRV type name
673:   return nullptr;
674: }
675:
676: std::unordered_set<BasicBlock *>
677: PartialOrderingVisitor::getReachableFrom(BasicBlock *Start) {
678:   std::queue<BasicBlock *> ToVisit;
679:   ToVisit.push(Start);
680:
681:   std::unordered_set<BasicBlock *> Output;
682:   while (ToVisit.size() != 0) {
683:     BasicBlock *BB = ToVisit.front();
684:     ToVisit.pop();
685:
686:     if (Output.count(BB) != 0)
687:       continue;
688:     Output.insert(BB);
689:
690:     for (BasicBlock *Successor : successors(BB)) {
691:       if (DT.dominates(Successor, BB))
692:         continue;
693:       ToVisit.push(Successor);
694:     }
695:   }
696:
697:   return Output;
698: }
699:
700: bool PartialOrderingVisitor::CanBeVisited(BasicBlock *BB) const {
701:   for (BasicBlock *P : predecessors(BB)) {
702:     // Ignore back-edges.
703:     if (DT.dominates(BB, P))
704:       continue;
705:
706:     // One of the predecessor hasn't been visited. Not ready yet.
707:     if (BlockToOrder.count(P) == 0)
708:       return false;
709:
710:     // If the block is a loop exit, the loop must be finished before
711:     // we can continue.
712:     Loop *L = LI.getLoopFor(P);
713:     if (L == nullptr || L->contains(BB))
714:       continue;
715:
716:     // SPIR-V requires a single back-edge. And the backend first
717:     // step transforms loops into the simplified format. If we have
718:     // more than 1 back-edge, something is wrong.
719:     assert(L->getNumBackEdges() <= 1);
720:
```
- EN: This range implements operational logic in helpers such as parseBasicTypeName, consume_front, Type::getVoidTy, Type::getIntNTy, translating backend policy into executable code.
- CN: 这一段实现了 parseBasicTypeName、consume_front、Type::getVoidTy、Type::getIntNTy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:     // If the loop has no latch, loop's rank won't matter, so we can
722:     // proceed.
723:     BasicBlock *Latch = L->getLoopLatch();
724:     assert(Latch);
725:     if (Latch == nullptr)
726:       continue;
727:
728:     // The latch is not ready yet, let's wait.
729:     if (BlockToOrder.count(Latch) == 0)
730:       return false;
731:   }
732:
733:   return true;
734: }
735:
736: size_t PartialOrderingVisitor::GetNodeRank(BasicBlock *BB) const {
737:   auto It = BlockToOrder.find(BB);
738:   if (It != BlockToOrder.end())
739:     return It->second.Rank;
740:
741:   size_t result = 0;
742:   for (BasicBlock *P : predecessors(BB)) {
743:     // Ignore back-edges.
744:     if (DT.dominates(BB, P))
745:       continue;
746:
747:     auto Iterator = BlockToOrder.end();
748:     Loop *L = LI.getLoopFor(P);
749:     BasicBlock *Latch = L ? L->getLoopLatch() : nullptr;
750:
751:     // If the predecessor is either outside a loop, or part of
752:     // the same loop, simply take its rank + 1.
753:     if (L == nullptr || L->contains(BB) || Latch == nullptr) {
754:       Iterator = BlockToOrder.find(P);
755:     } else {
756:       // Otherwise, take the loop's rank (highest rank in the loop) as base.
757:       // Since loops have a single latch, highest rank is easy to find.
758:       // If the loop has no latch, then it doesn't matter.
759:       Iterator = BlockToOrder.find(Latch);
760:     }
761:
762:     assert(Iterator != BlockToOrder.end());
763:     result = std::max(result, Iterator->second.Rank + 1);
764:   }
765:
766:   return result;
767: }
768:
769: size_t PartialOrderingVisitor::visit(BasicBlock *BB, size_t Unused) {
770:   ToVisit.push(BB);
771:   Queued.insert(BB);
772:
773:   size_t QueueIndex = 0;
774:   while (ToVisit.size() != 0) {
775:     BasicBlock *BB = ToVisit.front();
776:     ToVisit.pop();
777:
778:     if (!CanBeVisited(BB)) {
779:       ToVisit.push(BB);
780:       if (QueueIndex >= ToVisit.size())
781:         llvm::report_fatal_error(
782:             "No valid candidate in the queue. Is the graph reducible?");
783:       QueueIndex++;
784:       continue;
785:     }
786:
787:     QueueIndex = 0;
788:     size_t Rank = GetNodeRank(BB);
789:     OrderInfo Info = {Rank, BlockToOrder.size()};
790:     BlockToOrder.emplace(BB, Info);
791:
792:     for (BasicBlock *S : successors(BB)) {
793:       if (Queued.count(S) != 0)
794:         continue;
795:       ToVisit.push(S);
796:       Queued.insert(S);
797:     }
798:   }
799:
800:   return 0;
801: }
802:
803: PartialOrderingVisitor::PartialOrderingVisitor(Function &F) {
804:   DT.recalculate(F);
805:   LI = LoopInfo(DT);
806:
807:   visit(&*F.begin(), 0);
808:
809:   Order.reserve(F.size());
810:   for (auto &[BB, Info] : BlockToOrder)
```
- EN: This range implements operational logic in helpers such as getLoopLatch, assert, PartialOrderingVisitor::GetNodeRank, find, translating backend policy into executable code.
- CN: 这一段实现了 getLoopLatch、assert、PartialOrderingVisitor::GetNodeRank、find 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:     Order.emplace_back(BB);
812:
813:   std::sort(Order.begin(), Order.end(), [&](const auto &LHS, const auto &RHS) {
814:     return compare(LHS, RHS);
815:   });
816: }
817:
818: bool PartialOrderingVisitor::compare(const BasicBlock *LHS,
819:                                      const BasicBlock *RHS) const {
820:   const OrderInfo &InfoLHS = BlockToOrder.at(const_cast<BasicBlock *>(LHS));
821:   const OrderInfo &InfoRHS = BlockToOrder.at(const_cast<BasicBlock *>(RHS));
822:   if (InfoLHS.Rank != InfoRHS.Rank)
823:     return InfoLHS.Rank < InfoRHS.Rank;
824:   return InfoLHS.TraversalIndex < InfoRHS.TraversalIndex;
825: }
826:
827: void PartialOrderingVisitor::partialOrderVisit(
828:     BasicBlock &Start, std::function<bool(BasicBlock *)> Op) {
829:   std::unordered_set<BasicBlock *> Reachable = getReachableFrom(&Start);
830:   assert(BlockToOrder.count(&Start) != 0);
831:
832:   // Skipping blocks with a rank inferior to |Start|'s rank.
833:   auto It = Order.begin();
834:   while (It != Order.end() && *It != &Start)
835:     ++It;
836:
837:   // This is unexpected. Worst case |Start| is the last block,
838:   // so It should point to the last block, not past-end.
839:   assert(It != Order.end());
840:
841:   // By default, there is no rank limit. Setting it to the maximum value.
842:   std::optional<size_t> EndRank = std::nullopt;
843:   for (; It != Order.end(); ++It) {
844:     if (EndRank.has_value() && BlockToOrder[*It].Rank > *EndRank)
845:       break;
846:
847:     if (Reachable.count(*It) == 0) {
848:       continue;
849:     }
850:
851:     if (!Op(*It)) {
852:       EndRank = BlockToOrder[*It].Rank;
853:     }
854:   }
855: }
856:
857: bool sortBlocks(Function &F) {
858:   if (F.size() == 0)
859:     return false;
860:
861:   bool Modified = false;
862:   std::vector<BasicBlock *> Order;
863:   Order.reserve(F.size());
864:
865:   ReversePostOrderTraversal<Function *> RPOT(&F);
866:   llvm::append_range(Order, RPOT);
867:
868:   assert(&*F.begin() == Order[0]);
869:   BasicBlock *LastBlock = &*F.begin();
870:   for (BasicBlock *BB : Order) {
871:     if (BB != LastBlock && &*LastBlock->getNextNode() != BB) {
872:       Modified = true;
873:       BB->moveAfter(LastBlock);
874:     }
875:     LastBlock = BB;
876:   }
877:
878:   return Modified;
879: }
880:
881: MachineInstr *getVRegDef(MachineRegisterInfo &MRI, Register Reg) {
882:   MachineInstr *MaybeDef = MRI.getVRegDef(Reg);
883:   if (MaybeDef && MaybeDef->getOpcode() == SPIRV::ASSIGN_TYPE)
884:     MaybeDef = MRI.getVRegDef(MaybeDef->getOperand(1).getReg());
885:   return MaybeDef;
886: }
887:
888: bool getVacantFunctionName(Module &M, std::string &Name) {
889:   // It's a bit of paranoia, but still we don't want to have even a chance that
890:   // the loop will work for too long.
891:   constexpr unsigned MaxIters = 1024;
892:   for (unsigned I = 0; I < MaxIters; ++I) {
893:     std::string OrdName = Name + Twine(I).str();
894:     if (!M.getFunction(OrdName)) {
895:       Name = std::move(OrdName);
896:       return true;
897:     }
898:   }
899:   return false;
900: }
```
- EN: This range implements operational logic in helpers such as emplace_back, std::sort, compare, at, translating backend policy into executable code.
- CN: 这一段实现了 emplace_back、std::sort、compare、at 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:
902: // Assign SPIR-V type to the register. If the register has no valid assigned
903: // class, set register LLT type and class according to the SPIR-V type.
904: void setRegClassType(Register Reg, SPIRVTypeInst SpvType,
905:                      SPIRVGlobalRegistry *GR, MachineRegisterInfo *MRI,
906:                      const MachineFunction &MF, bool Force) {
907:   GR->assignSPIRVTypeToVReg(SpvType, Reg, MF);
908:   if (!MRI->getRegClassOrNull(Reg) || Force) {
909:     MRI->setRegClass(Reg, GR->getRegClass(SpvType));
910:     LLT RegType = GR->getRegType(SpvType);
911:     if (Force || !MRI->getType(Reg).isValid())
912:       MRI->setType(Reg, RegType);
913:   }
914: }
915:
916: // Create a SPIR-V type, assign SPIR-V type to the register. If the register has
917: // no valid assigned class, set register LLT type and class according to the
918: // SPIR-V type.
919: void setRegClassType(Register Reg, const Type *Ty, SPIRVGlobalRegistry *GR,
920:                      MachineIRBuilder &MIRBuilder,
921:                      SPIRV::AccessQualifier::AccessQualifier AccessQual,
922:                      bool EmitIR, bool Force) {
923:   setRegClassType(Reg,
924:                   GR->getOrCreateSPIRVType(Ty, MIRBuilder, AccessQual, EmitIR),
925:                   GR, MIRBuilder.getMRI(), MIRBuilder.getMF(), Force);
926: }
927:
928: // Create a virtual register and assign SPIR-V type to the register. Set
929: // register LLT type and class according to the SPIR-V type.
930: Register createVirtualRegister(SPIRVTypeInst SpvType, SPIRVGlobalRegistry *GR,
931:                                MachineRegisterInfo *MRI,
932:                                const MachineFunction &MF) {
933:   Register Reg = MRI->createVirtualRegister(GR->getRegClass(SpvType));
934:   MRI->setType(Reg, GR->getRegType(SpvType));
935:   GR->assignSPIRVTypeToVReg(SpvType, Reg, MF);
936:   return Reg;
937: }
938:
939: // Create a virtual register and assign SPIR-V type to the register. Set
940: // register LLT type and class according to the SPIR-V type.
941: Register createVirtualRegister(SPIRVTypeInst SpvType, SPIRVGlobalRegistry *GR,
942:                                MachineIRBuilder &MIRBuilder) {
943:   return createVirtualRegister(SpvType, GR, MIRBuilder.getMRI(),
944:                                MIRBuilder.getMF());
945: }
946:
947: // Create a SPIR-V type, virtual register and assign SPIR-V type to the
948: // register. Set register LLT type and class according to the SPIR-V type.
949: Register createVirtualRegister(
950:     const Type *Ty, SPIRVGlobalRegistry *GR, MachineIRBuilder &MIRBuilder,
951:     SPIRV::AccessQualifier::AccessQualifier AccessQual, bool EmitIR) {
952:   return createVirtualRegister(
953:       GR->getOrCreateSPIRVType(Ty, MIRBuilder, AccessQual, EmitIR), GR,
954:       MIRBuilder);
955: }
956:
957: CallInst *buildIntrWithMD(Intrinsic::ID IntrID, ArrayRef<Type *> Types,
958:                           Value *Arg, Value *Arg2, ArrayRef<Constant *> Imms,
959:                           IRBuilder<> &B) {
960:   SmallVector<Value *, 4> Args;
961:   Args.push_back(Arg2);
962:   Args.push_back(buildMD(Arg));
963:   llvm::append_range(Args, Imms);
964:   return B.CreateIntrinsic(IntrID, {Types}, Args);
965: }
966:
967: // Return true if there is an opaque pointer type nested in the argument.
968: bool isNestedPointer(const Type *Ty) {
969:   if (Ty->isPtrOrPtrVectorTy())
970:     return true;
971:   if (const FunctionType *RefTy = dyn_cast<FunctionType>(Ty)) {
972:     if (isNestedPointer(RefTy->getReturnType()))
973:       return true;
974:     for (const Type *ArgTy : RefTy->params())
975:       if (isNestedPointer(ArgTy))
976:         return true;
977:     return false;
978:   }
979:   if (const ArrayType *RefTy = dyn_cast<ArrayType>(Ty))
980:     return isNestedPointer(RefTy->getElementType());
981:   return false;
982: }
983:
984: bool isSpvIntrinsic(const Value *Arg) {
985:   if (const auto *II = dyn_cast<IntrinsicInst>(Arg))
986:     if (Function *F = II->getCalledFunction())
987:       if (F->getName().starts_with("llvm.spv."))
988:         return true;
989:   return false;
990: }
```
- EN: This range implements operational logic in helpers such as assignSPIRVTypeToVReg, setRegClass, getRegType, setType, translating backend policy into executable code.
- CN: 这一段实现了 assignSPIRVTypeToVReg、setRegClass、getRegType、setType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:
 992: // Function to create continued instructions for SPV_INTEL_long_composites
 993: // extension
 994: SmallVector<MachineInstr *, 4>
 995: createContinuedInstructions(MachineIRBuilder &MIRBuilder, unsigned Opcode,
 996:                             unsigned MinWC, unsigned ContinuedOpcode,
 997:                             ArrayRef<Register> Args, Register ReturnRegister,
 998:                             Register TypeID) {
 999:
1000:   SmallVector<MachineInstr *, 4> Instructions;
1001:   constexpr unsigned MaxWordCount = UINT16_MAX;
1002:   const size_t NumElements = Args.size();
1003:   size_t MaxNumElements = MaxWordCount - MinWC;
1004:   size_t SPIRVStructNumElements = NumElements;
1005:
1006:   if (NumElements > MaxNumElements) {
1007:     // Do adjustments for continued instructions which always had only one
1008:     // minumum word count.
1009:     SPIRVStructNumElements = MaxNumElements;
1010:     MaxNumElements = MaxWordCount - 1;
1011:   }
1012:
1013:   auto MIB =
1014:       MIRBuilder.buildInstr(Opcode).addDef(ReturnRegister).addUse(TypeID);
1015:
1016:   for (size_t I = 0; I < SPIRVStructNumElements; ++I)
1017:     MIB.addUse(Args[I]);
1018:
1019:   Instructions.push_back(MIB.getInstr());
1020:
1021:   for (size_t I = SPIRVStructNumElements; I < NumElements;
1022:        I += MaxNumElements) {
1023:     auto MIB = MIRBuilder.buildInstr(ContinuedOpcode);
1024:     for (size_t J = I; J < std::min(I + MaxNumElements, NumElements); ++J)
1025:       MIB.addUse(Args[J]);
1026:     Instructions.push_back(MIB.getInstr());
1027:   }
1028:   return Instructions;
1029: }
1030:
1031: SmallVector<unsigned, 1>
1032: getSpirvLoopControlOperandsFromLoopMetadata(MDNode *LoopMD) {
1033:   unsigned LC = SPIRV::LoopControl::None;
1034:   // Currently used only to store PartialCount value. Later when other
1035:   // LoopControls are added - this map should be sorted before making
1036:   // them loop_merge operands to satisfy 3.23. Loop Control requirements.
1037:   std::vector<std::pair<unsigned, unsigned>> MaskToValueMap;
1038:   if (findOptionMDForLoopID(LoopMD, "llvm.loop.unroll.disable")) {
1039:     LC |= SPIRV::LoopControl::DontUnroll;
1040:   } else {
1041:     if (findOptionMDForLoopID(LoopMD, "llvm.loop.unroll.enable") ||
1042:         findOptionMDForLoopID(LoopMD, "llvm.loop.unroll.full")) {
1043:       LC |= SPIRV::LoopControl::Unroll;
1044:     }
1045:     if (MDNode *CountMD =
1046:             findOptionMDForLoopID(LoopMD, "llvm.loop.unroll.count")) {
1047:       if (auto *CI =
1048:               mdconst::extract_or_null<ConstantInt>(CountMD->getOperand(1))) {
1049:         unsigned Count = CI->getZExtValue();
1050:         if (Count != 1) {
1051:           LC |= SPIRV::LoopControl::PartialCount;
1052:           MaskToValueMap.emplace_back(
1053:               std::make_pair(SPIRV::LoopControl::PartialCount, Count));
1054:         }
1055:       }
1056:     }
1057:   }
1058:   SmallVector<unsigned, 1> Result = {LC};
1059:   for (auto &[Mask, Val] : MaskToValueMap)
1060:     Result.push_back(Val);
1061:   return Result;
1062: }
1063:
1064: SmallVector<unsigned, 1> getSpirvLoopControlOperandsFromLoopMetadata(Loop *L) {
1065:   return getSpirvLoopControlOperandsFromLoopMetadata(L->getLoopID());
1066: }
1067:
1068: const std::set<unsigned> &getTypeFoldingSupportedOpcodes() {
1069:   // clang-format off
1070:   static const std::set<unsigned> TypeFoldingSupportingOpcs = {
1071:     TargetOpcode::G_ADD,
1072:     TargetOpcode::G_FADD,
1073:     TargetOpcode::G_STRICT_FADD,
1074:     TargetOpcode::G_SUB,
1075:     TargetOpcode::G_FSUB,
1076:     TargetOpcode::G_STRICT_FSUB,
1077:     TargetOpcode::G_MUL,
1078:     TargetOpcode::G_FMUL,
1079:     TargetOpcode::G_STRICT_FMUL,
1080:     TargetOpcode::G_SDIV,
```
- EN: This range implements operational logic in helpers such as size, buildInstr, addUse, push_back, translating backend policy into executable code.
- CN: 这一段实现了 size、buildInstr、addUse、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:     TargetOpcode::G_UDIV,
1082:     TargetOpcode::G_FDIV,
1083:     TargetOpcode::G_STRICT_FDIV,
1084:     TargetOpcode::G_SREM,
1085:     TargetOpcode::G_UREM,
1086:     TargetOpcode::G_FREM,
1087:     TargetOpcode::G_STRICT_FREM,
1088:     TargetOpcode::G_FNEG,
1089:     TargetOpcode::G_CONSTANT,
1090:     TargetOpcode::G_FCONSTANT,
1091:     TargetOpcode::G_AND,
1092:     TargetOpcode::G_OR,
1093:     TargetOpcode::G_XOR,
1094:     TargetOpcode::G_SHL,
1095:     TargetOpcode::G_ASHR,
1096:     TargetOpcode::G_LSHR,
1097:     TargetOpcode::G_SELECT,
1098:     TargetOpcode::G_EXTRACT_VECTOR_ELT,
1099:   };
1100:   // clang-format on
1101:   return TypeFoldingSupportingOpcs;
1102: }
1103:
1104: bool isTypeFoldingSupported(unsigned Opcode) {
1105:   return getTypeFoldingSupportedOpcodes().count(Opcode) > 0;
1106: }
1107:
1108: // Traversing [g]MIR accounting for pseudo-instructions.
1109: MachineInstr *passCopy(MachineInstr *Def, const MachineRegisterInfo *MRI) {
1110:   return (Def->getOpcode() == SPIRV::ASSIGN_TYPE ||
1111:           Def->getOpcode() == TargetOpcode::COPY)
1112:              ? MRI->getVRegDef(Def->getOperand(1).getReg())
1113:              : Def;
1114: }
1115:
1116: MachineInstr *getDef(const MachineOperand &MO, const MachineRegisterInfo *MRI) {
1117:   if (MachineInstr *Def = MRI->getVRegDef(MO.getReg()))
1118:     return passCopy(Def, MRI);
1119:   return nullptr;
1120: }
1121:
1122: MachineInstr *getImm(const MachineOperand &MO, const MachineRegisterInfo *MRI) {
1123:   if (MachineInstr *Def = getDef(MO, MRI)) {
1124:     if (Def->getOpcode() == TargetOpcode::G_CONSTANT ||
1125:         Def->getOpcode() == SPIRV::OpConstantI)
1126:       return Def;
1127:   }
1128:   return nullptr;
1129: }
1130:
1131: int64_t foldImm(const MachineOperand &MO, const MachineRegisterInfo *MRI) {
1132:   if (MachineInstr *Def = getImm(MO, MRI)) {
1133:     if (Def->getOpcode() == SPIRV::OpConstantI)
1134:       return Def->getOperand(2).getImm();
1135:     if (Def->getOpcode() == TargetOpcode::G_CONSTANT)
1136:       return Def->getOperand(1).getCImm()->getZExtValue();
1137:   }
1138:   llvm_unreachable("Unexpected integer constant pattern");
1139: }
1140:
1141: unsigned getArrayComponentCount(const MachineRegisterInfo *MRI,
1142:                                 const MachineInstr *ResType) {
1143:   return foldImm(ResType->getOperand(2), MRI);
1144: }
1145:
1146: bool matchPeeledArrayPattern(const StructType *Ty, Type *&OriginalElementType,
1147:                              uint64_t &TotalSize) {
1148:   // An array of N padded structs is represented as {[N-1 x <{T, pad}>], T}.
1149:   if (Ty->getStructNumElements() != 2)
1150:     return false;
1151:
1152:   Type *FirstElement = Ty->getStructElementType(0);
1153:   Type *SecondElement = Ty->getStructElementType(1);
1154:
1155:   if (!FirstElement->isArrayTy())
1156:     return false;
1157:
1158:   Type *ArrayElementType = FirstElement->getArrayElementType();
1159:   if (!ArrayElementType->isStructTy() ||
1160:       ArrayElementType->getStructNumElements() != 2)
1161:     return false;
1162:
1163:   Type *T_in_struct = ArrayElementType->getStructElementType(0);
1164:   if (T_in_struct != SecondElement)
1165:     return false;
1166:
1167:   auto *Padding_in_struct =
1168:       dyn_cast<TargetExtType>(ArrayElementType->getStructElementType(1));
1169:   if (!Padding_in_struct || Padding_in_struct->getName() != "spirv.Padding")
1170:     return false;
```
- EN: This range implements operational logic in helpers such as isTypeFoldingSupported, passCopy, getOpcode, getVRegDef, translating backend policy into executable code.
- CN: 这一段实现了 isTypeFoldingSupported、passCopy、getOpcode、getVRegDef 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1260
```cpp
1171:
1172:   const uint64_t ArraySize = FirstElement->getArrayNumElements();
1173:   TotalSize = ArraySize + 1;
1174:   OriginalElementType = ArrayElementType;
1175:   return true;
1176: }
1177:
1178: Type *reconstitutePeeledArrayType(Type *Ty) {
1179:   if (!Ty->isStructTy())
1180:     return Ty;
1181:
1182:   auto *STy = cast<StructType>(Ty);
1183:   Type *OriginalElementType = nullptr;
1184:   uint64_t TotalSize = 0;
1185:   if (matchPeeledArrayPattern(STy, OriginalElementType, TotalSize)) {
1186:     Type *ResultTy = ArrayType::get(
1187:         reconstitutePeeledArrayType(OriginalElementType), TotalSize);
1188:     return ResultTy;
1189:   }
1190:
1191:   SmallVector<Type *, 4> NewElementTypes;
1192:   bool Changed = false;
1193:   for (Type *ElementTy : STy->elements()) {
1194:     Type *NewElementTy = reconstitutePeeledArrayType(ElementTy);
1195:     if (NewElementTy != ElementTy)
1196:       Changed = true;
1197:     NewElementTypes.push_back(NewElementTy);
1198:   }
1199:
1200:   if (!Changed)
1201:     return Ty;
1202:
1203:   Type *ResultTy;
1204:   if (STy->isLiteral())
1205:     ResultTy =
1206:         StructType::get(STy->getContext(), NewElementTypes, STy->isPacked());
1207:   else {
1208:     auto *NewTy = StructType::create(STy->getContext(), STy->getName());
1209:     NewTy->setBody(NewElementTypes, STy->isPacked());
1210:     ResultTy = NewTy;
1211:   }
1212:   return ResultTy;
1213: }
1214:
1215: std::optional<SPIRV::LinkageType::LinkageType>
1216: getSpirvLinkageTypeFor(const SPIRVSubtarget &ST, const GlobalValue &GV) {
1217:   if (GV.hasLocalLinkage())
1218:     return std::nullopt;
1219:
1220:   if (GV.isDeclarationForLinker()) {
1221:     // Interface variables must not get Import linkage.
1222:     if (const auto *GVar = dyn_cast<GlobalVariable>(&GV)) {
1223:       auto SC = addressSpaceToStorageClass(GVar->getAddressSpace(), ST);
1224:       if (SC == SPIRV::StorageClass::Input ||
1225:           SC == SPIRV::StorageClass::Output ||
1226:           SC == SPIRV::StorageClass::PushConstant)
1227:         return std::nullopt;
1228:     }
1229:     return SPIRV::LinkageType::Import;
1230:   }
1231:
1232:   if (GV.hasHiddenVisibility())
1233:     return std::nullopt;
1234:
1235:   if (GV.hasLinkOnceODRLinkage() &&
1236:       ST.canUseExtension(SPIRV::Extension::SPV_KHR_linkonce_odr))
1237:     return SPIRV::LinkageType::LinkOnceODR;
1238:
1239:   if (GV.hasWeakLinkage() &&
1240:       ST.canUseExtension(SPIRV::Extension::SPV_AMD_weak_linkage))
1241:     return SPIRV::LinkageType::WeakAMD;
1242:
1243:   return SPIRV::LinkageType::Export;
1244: }
1245:
1246: Function *getOrCreateBackendServiceFunction(Module &M) {
1247:   std::string ServiceFunName = SPIRV_BACKEND_SERVICE_FUN_NAME;
1248:   if (!getVacantFunctionName(M, ServiceFunName))
1249:     report_fatal_error(
1250:         "cannot allocate a name for the internal service function");
1251:   if (Function *SF = M.getFunction(ServiceFunName)) {
1252:     if (SF->getInstructionCount() > 0)
1253:       report_fatal_error(
1254:           "Unexpected combination of global variables and function pointers");
1255:     return SF;
1256:   }
1257:   Function *SF = Function::Create(
1258:       FunctionType::get(Type::getVoidTy(M.getContext()), {}, false),
1259:       GlobalValue::PrivateLinkage, ServiceFunName, M);
1260:   SF->addFnAttr(SPIRV_BACKEND_SERVICE_FUN_NAME, "");
```
- EN: This range implements operational logic in helpers such as getArrayNumElements, reconstitutePeeledArrayType, push_back, StructType::get, translating backend policy into executable code.
- CN: 这一段实现了 getArrayNumElements、reconstitutePeeledArrayType、push_back、StructType::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1261-1264
```cpp
1261:   return SF;
1262: }
1263:
1264: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as backend logic, translating backend policy into executable code.
- CN: 这一段实现了 后端逻辑 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Utility files package reusable helpers so other backend components can stay focused on core compilation steps.
  - CN: 工具类文件封装可复用辅助逻辑，使其他后端组件能够聚焦核心编译步骤。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include getValue, find_if, getReturnType, PTys, getNumOperands, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getValue, find_if, getReturnType, PTys, getNumOperands，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVUtils.h`
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRV.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVInstrInfo.h`
  - `SPIRVSubtarget.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/StringRef.h`
  - `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`
  - `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
  - `llvm/CodeGen/MachineInstr.h`
  - `llvm/CodeGen/MachineInstrBuilder.h`
  - `llvm/Demangle/Demangle.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
- System/standard headers / 系统或标准头文件:
  - `queue`
  - `vector`
