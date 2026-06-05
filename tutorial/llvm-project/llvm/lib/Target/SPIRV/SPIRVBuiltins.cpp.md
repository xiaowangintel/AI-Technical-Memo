# SPIRVBuiltins.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVBuiltins.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements lowering builtin function calls and types using their demangled names and TableGen records.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-120
```cpp
  1: //===- SPIRVBuiltins.cpp - SPIR-V Built-in Functions ------------*- C++ -*-===//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8: //
  9: // This file implements lowering builtin function calls and types using their
 10: // demangled names and TableGen records.
 11: //
 12: //===----------------------------------------------------------------------===//
 13:
 14: #include "SPIRVBuiltins.h"
 15: #include "SPIRV.h"
 16: #include "SPIRVSubtarget.h"
 17: #include "SPIRVUtils.h"
 18: #include "llvm/ADT/StringExtras.h"
 19: #include "llvm/Analysis/ValueTracking.h"
 20: #include "llvm/IR/IntrinsicsSPIRV.h"
 21: #include <regex>
 22: #include <string>
 23: #include <tuple>
 24:
 25: #define DEBUG_TYPE "spirv-builtins"
 26:
 27: namespace llvm {
 28: namespace SPIRV {
 29: #define GET_BuiltinGroup_DECL
 30: #include "SPIRVGenTables.inc"
 31:
 32: struct DemangledBuiltin {
 33:   StringRef Name;
 34:   InstructionSet::InstructionSet Set;
 35:   BuiltinGroup Group;
 36:   uint8_t MinNumArgs;
 37:   uint8_t MaxNumArgs;
 38: };
 39:
 40: #define GET_DemangledBuiltins_DECL
 41: #define GET_DemangledBuiltins_IMPL
 42:
 43: struct IncomingCall {
 44:   const std::string BuiltinName;
 45:   const DemangledBuiltin *Builtin;
 46:
 47:   const Register ReturnRegister;
 48:   const SPIRVTypeInst ReturnType;
 49:   const SmallVectorImpl<Register> &Arguments;
 50:
 51:   IncomingCall(const std::string BuiltinName, const DemangledBuiltin *Builtin,
 52:                const Register ReturnRegister, SPIRVTypeInst ReturnType,
 53:                const SmallVectorImpl<Register> &Arguments)
 54:       : BuiltinName(std::move(BuiltinName)), Builtin(Builtin),
 55:         ReturnRegister(ReturnRegister), ReturnType(ReturnType),
 56:         Arguments(Arguments) {}
 57:
 58:   bool isSpirvOp() const { return BuiltinName.rfind("__spirv_", 0) == 0; }
 59: };
 60:
 61: struct NativeBuiltin {
 62:   StringRef Name;
 63:   InstructionSet::InstructionSet Set;
 64:   uint32_t Opcode;
 65: };
 66:
 67: #define GET_NativeBuiltins_DECL
 68: #define GET_NativeBuiltins_IMPL
 69:
 70: struct GroupBuiltin {
 71:   StringRef Name;
 72:   uint32_t Opcode;
 73:   uint32_t GroupOperation;
 74:   bool IsElect;
 75:   bool IsAllOrAny;
 76:   bool IsAllEqual;
 77:   bool IsBallot;
 78:   bool IsInverseBallot;
 79:   bool IsBallotBitExtract;
 80:   bool IsBallotFindBit;
 81:   bool IsLogical;
 82:   bool NoGroupOperation;
 83:   bool HasBoolArg;
 84: };
 85:
 86: #define GET_GroupBuiltins_DECL
 87: #define GET_GroupBuiltins_IMPL
 88:
 89: struct IntelSubgroupsBuiltin {
 90:   StringRef Name;
 91:   uint32_t Opcode;
 92:   bool IsBlock;
 93:   bool IsWrite;
 94:   bool IsMedia;
 95: };
 96:
 97: #define GET_IntelSubgroupsBuiltins_DECL
 98: #define GET_IntelSubgroupsBuiltins_IMPL
 99:
100: struct AtomicFloatingBuiltin {
101:   StringRef Name;
102:   uint32_t Opcode;
103: };
104:
105: #define GET_AtomicFloatingBuiltins_DECL
106: #define GET_AtomicFloatingBuiltins_IMPL
107: struct GroupUniformBuiltin {
108:   StringRef Name;
109:   uint32_t Opcode;
110:   bool IsLogical;
111: };
112:
113: #define GET_GroupUniformBuiltins_DECL
114: #define GET_GroupUniformBuiltins_IMPL
115:
116: struct GetBuiltin {
117:   StringRef Name;
118:   InstructionSet::InstructionSet Set;
119:   BuiltIn::BuiltIn Value;
120: };
```
- EN: This range defines or declares important types such as DemangledBuiltin, IncomingCall, Arguments, isSpirvOp, shaping the data model used by SPIRVBuiltins.cpp.
- CN: 这一段定义或声明了 DemangledBuiltin、IncomingCall、Arguments、isSpirvOp 等关键类型，构成 SPIRVBuiltins.cpp 使用的数据模型。

### Lines 121-240
```cpp
121:
122: using namespace BuiltIn;
123: #define GET_GetBuiltins_DECL
124: #define GET_GetBuiltins_IMPL
125:
126: struct ImageQueryBuiltin {
127:   StringRef Name;
128:   InstructionSet::InstructionSet Set;
129:   uint32_t Component;
130: };
131:
132: #define GET_ImageQueryBuiltins_DECL
133: #define GET_ImageQueryBuiltins_IMPL
134:
135: struct IntegerDotProductBuiltin {
136:   StringRef Name;
137:   uint32_t Opcode;
138:   bool IsSwapReq;
139: };
140:
141: #define GET_IntegerDotProductBuiltins_DECL
142: #define GET_IntegerDotProductBuiltins_IMPL
143:
144: struct ConvertBuiltin {
145:   StringRef Name;
146:   InstructionSet::InstructionSet Set;
147:   bool IsDestinationSigned;
148:   bool IsSaturated;
149:   bool IsRounded;
150:   bool IsBfloat16;
151:   bool IsTF32;
152:   FPRoundingMode::FPRoundingMode RoundingMode;
153: };
154:
155: struct VectorLoadStoreBuiltin {
156:   StringRef Name;
157:   InstructionSet::InstructionSet Set;
158:   uint32_t Number;
159:   uint32_t ElementCount;
160:   bool IsRounded;
161:   FPRoundingMode::FPRoundingMode RoundingMode;
162: };
163:
164: using namespace FPRoundingMode;
165: #define GET_ConvertBuiltins_DECL
166: #define GET_ConvertBuiltins_IMPL
167:
168: using namespace InstructionSet;
169: #define GET_VectorLoadStoreBuiltins_DECL
170: #define GET_VectorLoadStoreBuiltins_IMPL
171:
172: #define GET_CLMemoryScope_DECL
173: #define GET_CLSamplerAddressingMode_DECL
174: #define GET_CLMemoryFenceFlags_DECL
175: #define GET_ExtendedBuiltins_DECL
176: #include "SPIRVGenTables.inc"
177: } // namespace SPIRV
178:
179: //===----------------------------------------------------------------------===//
180: // Misc functions for looking up builtins and veryfying requirements using
181: // TableGen records
182: //===----------------------------------------------------------------------===//
183:
184: namespace SPIRV {
185: /// Parses the name part of the demangled builtin call.
186: std::string lookupBuiltinNameHelper(StringRef DemangledCall,
187:                                     FPDecorationId *DecorationId) {
188:   StringRef PassPrefix = "(anonymous namespace)::";
189:   StringRef SpvPrefix = "__spv::";
190:   std::string BuiltinName = DemangledCall.str();
191:
192:   // Check if the extracted name contains type information between angle
193:   // brackets. If so, the builtin is an instantiated template - needs to have
194:   // the information after angle brackets and return type removed.
195:   std::size_t Pos = BuiltinName.find(">(");
196:   if (Pos != std::string::npos) {
197:     BuiltinName = BuiltinName.substr(0, BuiltinName.rfind('<', Pos));
198:   } else {
199:     Pos = BuiltinName.find('(');
200:     if (Pos != std::string::npos)
201:       BuiltinName = BuiltinName.substr(0, Pos);
202:   }
203:   BuiltinName = BuiltinName.substr(BuiltinName.find_last_of(' ') + 1);
204:
205:   // Itanium Demangler result may have "(anonymous namespace)::" or "__spv::"
206:   // prefix.
207:   if (BuiltinName.find(PassPrefix) == 0)
208:     BuiltinName = BuiltinName.substr(PassPrefix.size());
209:   else if (BuiltinName.find(SpvPrefix) == 0)
210:     BuiltinName = BuiltinName.substr(SpvPrefix.size());
211:
212:   // Account for possible "__spirv_ocl_" prefix in SPIR-V friendly LLVM IR
213:   if (BuiltinName.rfind("__spirv_ocl_", 0) == 0)
214:     BuiltinName = BuiltinName.substr(12);
215:
216:   // Check if the extracted name begins with:
217:   // - "__spirv_ImageSampleExplicitLod"
218:   // - "__spirv_ImageRead"
219:   // - "__spirv_ImageWrite"
220:   // - "__spirv_ImageQuerySizeLod"
221:   // - "__spirv_UDotKHR"
222:   // - "__spirv_SDotKHR"
223:   // - "__spirv_SUDotKHR"
224:   // - "__spirv_SDotAccSatKHR"
225:   // - "__spirv_UDotAccSatKHR"
226:   // - "__spirv_SUDotAccSatKHR"
227:   // - "__spirv_ReadClockKHR"
228:   // - "__spirv_SubgroupBlockReadINTEL"
229:   // - "__spirv_SubgroupImageBlockReadINTEL"
230:   // - "__spirv_SubgroupImageMediaBlockReadINTEL"
231:   // - "__spirv_SubgroupImageMediaBlockWriteINTEL"
232:   // - "__spirv_Convert"
233:   // - "__spirv_Round"
234:   // - "__spirv_UConvert"
235:   // - "__spirv_SConvert"
236:   // - "__spirv_FConvert"
237:   // - "__spirv_SatConvert"
238:   // and maybe contains return type information at the end "_R<type>".
239:   // If so, extract the plain builtin name without the type information.
240:   static const std::regex SpvWithR(
```
- EN: This range defines or declares important types such as ImageQueryBuiltin, IntegerDotProductBuiltin, ConvertBuiltin, VectorLoadStoreBuiltin, shaping the data model used by SPIRVBuiltins.cpp.
- CN: 这一段定义或声明了 ImageQueryBuiltin、IntegerDotProductBuiltin、ConvertBuiltin、VectorLoadStoreBuiltin 等关键类型，构成 SPIRVBuiltins.cpp 使用的数据模型。

### Lines 241-360
```cpp
241:       "(__spirv_(ImageSampleExplicitLod|ImageRead|ImageWrite|ImageQuerySizeLod|"
242:       "UDotKHR|"
243:       "SDotKHR|SUDotKHR|SDotAccSatKHR|UDotAccSatKHR|SUDotAccSatKHR|"
244:       "ReadClockKHR|SubgroupBlockReadINTEL|SubgroupImageBlockReadINTEL|"
245:       "SubgroupImageMediaBlockReadINTEL|SubgroupImageMediaBlockWriteINTEL|"
246:       "Convert|Round|"
247:       "UConvert|SConvert|FConvert|SatConvert)[^_]*)(_R[^_]*_?(\\w+)?.*)?");
248:   std::smatch Match;
249:   if (std::regex_match(BuiltinName, Match, SpvWithR) && Match.size() > 1) {
250:     std::ssub_match SubMatch;
251:     if (DecorationId && Match.size() > 3) {
252:       SubMatch = Match[4];
253:       *DecorationId = demangledPostfixToDecorationId(SubMatch.str());
254:     }
255:     SubMatch = Match[1];
256:     BuiltinName = SubMatch.str();
257:   }
258:
259:   return BuiltinName;
260: }
261: } // namespace SPIRV
262:
263: /// Looks up the demangled builtin call in the SPIRVBuiltins.td records using
264: /// the provided \p DemangledCall and specified \p Set.
265: ///
266: /// The lookup follows the following algorithm, returning the first successful
267: /// match:
268: /// 1. Search with the plain demangled name (expecting a 1:1 match).
269: /// 2. Search with the prefix before or suffix after the demangled name
270: /// signyfying the type of the first argument.
271: ///
272: /// \returns Wrapper around the demangled call and found builtin definition.
273: static std::unique_ptr<const SPIRV::IncomingCall>
274: lookupBuiltin(StringRef DemangledCall,
275:               SPIRV::InstructionSet::InstructionSet Set,
276:               Register ReturnRegister, SPIRVTypeInst ReturnType,
277:               const SmallVectorImpl<Register> &Arguments) {
278:   std::string BuiltinName = SPIRV::lookupBuiltinNameHelper(DemangledCall);
279:
280:   SmallVector<StringRef, 10> BuiltinArgumentTypes;
281:   StringRef BuiltinArgs =
282:       DemangledCall.slice(DemangledCall.find('(') + 1, DemangledCall.find(')'));
283:   BuiltinArgs.split(BuiltinArgumentTypes, ',', -1, false);
284:
285:   // Look up the builtin in the defined set. Start with the plain demangled
286:   // name, expecting a 1:1 match in the defined builtin set.
287:   const SPIRV::DemangledBuiltin *Builtin;
288:   if ((Builtin = SPIRV::lookupBuiltin(BuiltinName, Set)))
289:     return std::make_unique<SPIRV::IncomingCall>(
290:         BuiltinName, Builtin, ReturnRegister, ReturnType, Arguments);
291:
292:   // If the initial look up was unsuccessful and the demangled call takes at
293:   // least 1 argument, add a prefix or suffix signifying the type of the first
294:   // argument and repeat the search.
295:   if (BuiltinArgumentTypes.size() >= 1) {
296:     char FirstArgumentType = BuiltinArgumentTypes[0][0];
297:     // Prefix to be added to the builtin's name for lookup.
298:     // For example, OpenCL "abs" taking an unsigned value has a prefix "u_".
299:     std::string Prefix;
300:
301:     switch (FirstArgumentType) {
302:     // Unsigned:
303:     case 'u':
304:       if (Set == SPIRV::InstructionSet::OpenCL_std)
305:         Prefix = "u_";
306:       else if (Set == SPIRV::InstructionSet::GLSL_std_450)
307:         Prefix = "u";
308:       break;
309:     // Signed:
310:     case 'c':
311:     case 's':
312:     case 'i':
313:     case 'l':
314:       if (Set == SPIRV::InstructionSet::OpenCL_std)
315:         Prefix = "s_";
316:       else if (Set == SPIRV::InstructionSet::GLSL_std_450)
317:         Prefix = "s";
318:       break;
319:     // Floating-point:
320:     case 'f':
321:     case 'd':
322:     case 'h':
323:       if (Set == SPIRV::InstructionSet::OpenCL_std ||
324:           Set == SPIRV::InstructionSet::GLSL_std_450)
325:         Prefix = "f";
326:       break;
327:     }
328:
329:     // If argument-type name prefix was added, look up the builtin again.
330:     if (!Prefix.empty() &&
331:         (Builtin = SPIRV::lookupBuiltin(Prefix + BuiltinName, Set)))
332:       return std::make_unique<SPIRV::IncomingCall>(
333:           BuiltinName, Builtin, ReturnRegister, ReturnType, Arguments);
334:
335:     // If lookup with a prefix failed, find a suffix to be added to the
336:     // builtin's name for lookup. For example, OpenCL "group_reduce_max" taking
337:     // an unsigned value has a suffix "u".
338:     std::string Suffix;
339:
340:     switch (FirstArgumentType) {
341:     // Unsigned:
342:     case 'u':
343:       Suffix = "u";
344:       break;
345:     // Signed:
346:     case 'c':
347:     case 's':
348:     case 'i':
349:     case 'l':
350:       Suffix = "s";
351:       break;
352:     // Floating-point:
353:     case 'f':
354:     case 'd':
355:     case 'h':
356:       Suffix = "f";
357:       break;
358:     }
359:
360:     // If argument-type name suffix was added, look up the builtin again.
```
- EN: This range implements operational logic in helpers such as str, SPIRV::lookupBuiltinNameHelper, slice, split, translating backend policy into executable code.
- CN: 这一段实现了 str、SPIRV::lookupBuiltinNameHelper、slice、split 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-480
```cpp
361:     if (!Suffix.empty() &&
362:         (Builtin = SPIRV::lookupBuiltin(BuiltinName + Suffix, Set)))
363:       return std::make_unique<SPIRV::IncomingCall>(
364:           BuiltinName, Builtin, ReturnRegister, ReturnType, Arguments);
365:   }
366:
367:   // No builtin with such name was found in the set.
368:   return nullptr;
369: }
370:
371: static MachineInstr *getBlockStructInstr(Register ParamReg,
372:                                          MachineRegisterInfo *MRI) {
373:   // We expect the following sequence of instructions:
374:   //   %0:_(pN) = G_INTRINSIC_W_SIDE_EFFECTS intrinsic(@llvm.spv.alloca)
375:   //   or       = G_GLOBAL_VALUE @block_literal_global
376:   //   %1:_(pN) = G_INTRINSIC_W_SIDE_EFFECTS intrinsic(@llvm.spv.bitcast), %0
377:   //   %2:_(p4) = G_ADDRSPACE_CAST %1:_(pN)
378:   MachineInstr *MI = MRI->getUniqueVRegDef(ParamReg);
379:   assert(MI->getOpcode() == TargetOpcode::G_ADDRSPACE_CAST &&
380:          MI->getOperand(1).isReg());
381:   Register BitcastReg = MI->getOperand(1).getReg();
382:   MachineInstr *BitcastMI = MRI->getUniqueVRegDef(BitcastReg);
383:   assert(isSpvIntrinsic(*BitcastMI, Intrinsic::spv_bitcast) &&
384:          BitcastMI->getOperand(2).isReg());
385:   Register ValueReg = BitcastMI->getOperand(2).getReg();
386:   MachineInstr *ValueMI = MRI->getUniqueVRegDef(ValueReg);
387:   return ValueMI;
388: }
389:
390: // Return an integer constant corresponding to the given register and
391: // defined in spv_track_constant.
392: // TODO: maybe unify with prelegalizer pass.
393: static unsigned getConstFromIntrinsic(Register Reg, MachineRegisterInfo *MRI) {
394:   MachineInstr *DefMI = MRI->getUniqueVRegDef(Reg);
395:   assert(DefMI->getOpcode() == TargetOpcode::G_CONSTANT &&
396:          DefMI->getOperand(1).isCImm());
397:   return DefMI->getOperand(1).getCImm()->getValue().getZExtValue();
398: }
399:
400: // Return type of the instruction result from spv_assign_type intrinsic.
401: // TODO: maybe unify with prelegalizer pass.
402: static const Type *getMachineInstrType(MachineInstr *MI) {
403:   MachineInstr *NextMI = MI->getNextNode();
404:   if (!NextMI)
405:     return nullptr;
406:   if (isSpvIntrinsic(*NextMI, Intrinsic::spv_assign_name))
407:     if ((NextMI = NextMI->getNextNode()) == nullptr)
408:       return nullptr;
409:   Register ValueReg = MI->getOperand(0).getReg();
410:   if ((!isSpvIntrinsic(*NextMI, Intrinsic::spv_assign_type) &&
411:        !isSpvIntrinsic(*NextMI, Intrinsic::spv_assign_ptr_type)) ||
412:       NextMI->getOperand(1).getReg() != ValueReg)
413:     return nullptr;
414:   Type *Ty = getMDOperandAsType(NextMI->getOperand(2).getMetadata(), 0);
415:   assert(Ty && "Type is expected");
416:   return Ty;
417: }
418:
419: static const Type *getBlockStructType(Register ParamReg,
420:                                       MachineRegisterInfo *MRI) {
421:   // In principle, this information should be passed to us from Clang via
422:   // an elementtype attribute. However, said attribute requires that
423:   // the function call be an intrinsic, which is not. Instead, we rely on being
424:   // able to trace this to the declaration of a variable: OpenCL C specification
425:   // section 6.12.5 should guarantee that we can do this.
426:   MachineInstr *MI = getBlockStructInstr(ParamReg, MRI);
427:   if (MI->getOpcode() == TargetOpcode::G_GLOBAL_VALUE)
428:     return MI->getOperand(1).getGlobal()->getType();
429:   assert(isSpvIntrinsic(*MI, Intrinsic::spv_alloca) &&
430:          "Blocks in OpenCL C must be traceable to allocation site");
431:   return getMachineInstrType(MI);
432: }
433:
434: //===----------------------------------------------------------------------===//
435: // Helper functions for building misc instructions
436: //===----------------------------------------------------------------------===//
437:
438: /// Helper function building either a resulting scalar or vector bool register
439: /// depending on the expected \p ResultType.
440: ///
441: /// \returns Tuple of the resulting register and its type.
442: static std::tuple<Register, SPIRVTypeInst>
443: buildBoolRegister(MachineIRBuilder &MIRBuilder, SPIRVTypeInst ResultType,
444:                   SPIRVGlobalRegistry *GR) {
445:   LLT Type;
446:   SPIRVTypeInst BoolType = GR->getOrCreateSPIRVBoolType(MIRBuilder, true);
447:
448:   if (ResultType->getOpcode() == SPIRV::OpTypeVector) {
449:     unsigned VectorElements = GR->getScalarOrVectorComponentCount(ResultType);
450:     BoolType = GR->getOrCreateSPIRVVectorType(BoolType, VectorElements,
451:                                               MIRBuilder, true);
452:     const FixedVectorType *LLVMVectorType =
453:         cast<FixedVectorType>(GR->getTypeForSPIRVType(BoolType));
454:     Type = LLT::vector(LLVMVectorType->getElementCount(), 1);
455:   } else {
456:     Type = LLT::scalar(1);
457:   }
458:
459:   Register ResultRegister =
460:       MIRBuilder.getMRI()->createGenericVirtualRegister(Type);
461:   MIRBuilder.getMRI()->setRegClass(ResultRegister, GR->getRegClass(ResultType));
462:   GR->assignSPIRVTypeToVReg(BoolType, ResultRegister, MIRBuilder.getMF());
463:   return std::make_tuple(ResultRegister, BoolType);
464: }
465:
466: /// Helper function for building either a vector or scalar select instruction
467: /// depending on the expected \p ResultType.
468: static bool buildSelectInst(MachineIRBuilder &MIRBuilder,
469:                             Register ReturnRegister, Register SourceRegister,
470:                             SPIRVTypeInst ReturnType, SPIRVGlobalRegistry *GR) {
471:   Register TrueConst, FalseConst;
472:
473:   if (ReturnType->getOpcode() == SPIRV::OpTypeVector) {
474:     unsigned Bits = GR->getScalarOrVectorBitWidth(ReturnType);
475:     uint64_t AllOnes = APInt::getAllOnes(Bits).getZExtValue();
476:     TrueConst =
477:         GR->getOrCreateConsIntVector(AllOnes, MIRBuilder, ReturnType, true);
478:     FalseConst = GR->getOrCreateConsIntVector(0, MIRBuilder, ReturnType, true);
479:   } else {
480:     TrueConst = GR->buildConstantInt(1, MIRBuilder, ReturnType, true);
```
- EN: This range implements operational logic in helpers such as SPIRV::lookupBuiltin, getUniqueVRegDef, getOperand, getConstFromIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 SPIRV::lookupBuiltin、getUniqueVRegDef、getOperand、getConstFromIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-600
```cpp
481:     FalseConst = GR->buildConstantInt(0, MIRBuilder, ReturnType, true);
482:   }
483:
484:   return MIRBuilder.buildSelect(ReturnRegister, SourceRegister, TrueConst,
485:                                 FalseConst);
486: }
487:
488: /// Helper function for building a load instruction loading into the
489: /// \p DestinationReg.
490: static Register buildLoadInst(SPIRVTypeInst BaseType, Register PtrRegister,
491:                               MachineIRBuilder &MIRBuilder,
492:                               SPIRVGlobalRegistry *GR, LLT LowLevelType,
493:                               Register DestinationReg = Register(0)) {
494:   if (!DestinationReg.isValid())
495:     DestinationReg = createVirtualRegister(BaseType, GR, MIRBuilder);
496:   // TODO: consider using correct address space and alignment (p0 is canonical
497:   // type for selection though).
498:   MachinePointerInfo PtrInfo = MachinePointerInfo();
499:   MIRBuilder.buildLoad(DestinationReg, PtrRegister, PtrInfo, Align());
500:   return DestinationReg;
501: }
502:
503: /// Helper function for building a load instruction for loading a builtin global
504: /// variable of \p BuiltinValue value.
505: static Register buildBuiltinVariableLoad(
506:     MachineIRBuilder &MIRBuilder, SPIRVTypeInst VariableType,
507:     SPIRVGlobalRegistry *GR, SPIRV::BuiltIn::BuiltIn BuiltinValue, LLT LLType,
508:     Register Reg = Register(0), bool isConst = true,
509:     const std::optional<SPIRV::LinkageType::LinkageType> &LinkageTy = {
510:         SPIRV::LinkageType::Import}) {
511:   Register NewRegister =
512:       MIRBuilder.getMRI()->createVirtualRegister(&SPIRV::pIDRegClass);
513:   MIRBuilder.getMRI()->setType(
514:       NewRegister,
515:       LLT::pointer(storageClassToAddressSpace(SPIRV::StorageClass::Function),
516:                    GR->getPointerSize()));
517:   SPIRVTypeInst PtrType = GR->getOrCreateSPIRVPointerType(
518:       VariableType, MIRBuilder, SPIRV::StorageClass::Input);
519:   GR->assignSPIRVTypeToVReg(PtrType, NewRegister, MIRBuilder.getMF());
520:
521:   // Set up the global OpVariable with the necessary builtin decorations.
522:   Register Variable = GR->buildGlobalVariable(
523:       NewRegister, PtrType, getLinkStringForBuiltIn(BuiltinValue), nullptr,
524:       SPIRV::StorageClass::Input, nullptr, /* isConst= */ isConst, LinkageTy,
525:       MIRBuilder, false);
526:
527:   // Load the value from the global variable.
528:   Register LoadedRegister =
529:       buildLoadInst(VariableType, Variable, MIRBuilder, GR, LLType, Reg);
530:   MIRBuilder.getMRI()->setType(LoadedRegister, LLType);
531:   return LoadedRegister;
532: }
533:
534: /// Helper external function for assigning a SPIRV type to a register, ensuring
535: /// the register class and type are set in MRI. Defined in
536: /// SPIRVPreLegalizer.cpp.
537: extern void updateRegType(Register Reg, Type *Ty, SPIRVTypeInst SpirvTy,
538:                           SPIRVGlobalRegistry *GR, MachineIRBuilder &MIB,
539:                           MachineRegisterInfo &MRI);
540:
541: // TODO: Move to TableGen.
542: static SPIRV::MemorySemantics::MemorySemantics
543: getSPIRVMemSemantics(std::memory_order MemOrder) {
544:   switch (MemOrder) {
545:   case std::memory_order_relaxed:
546:     return SPIRV::MemorySemantics::None;
547:   case std::memory_order_acquire:
548:     return SPIRV::MemorySemantics::Acquire;
549:   case std::memory_order_release:
550:     return SPIRV::MemorySemantics::Release;
551:   case std::memory_order_acq_rel:
552:     return SPIRV::MemorySemantics::AcquireRelease;
553:   case std::memory_order_seq_cst:
554:     return SPIRV::MemorySemantics::SequentiallyConsistent;
555:   default:
556:     report_fatal_error("Unknown CL memory scope");
557:   }
558: }
559:
560: static SPIRV::Scope::Scope getSPIRVScope(SPIRV::CLMemoryScope ClScope) {
561:   switch (ClScope) {
562:   case SPIRV::CLMemoryScope::memory_scope_work_item:
563:     return SPIRV::Scope::Invocation;
564:   case SPIRV::CLMemoryScope::memory_scope_work_group:
565:     return SPIRV::Scope::Workgroup;
566:   case SPIRV::CLMemoryScope::memory_scope_device:
567:     return SPIRV::Scope::Device;
568:   case SPIRV::CLMemoryScope::memory_scope_all_svm_devices:
569:     return SPIRV::Scope::CrossDevice;
570:   case SPIRV::CLMemoryScope::memory_scope_sub_group:
571:     return SPIRV::Scope::Subgroup;
572:   }
573:   report_fatal_error("Unknown CL memory scope");
574: }
575:
576: static Register buildConstantIntReg32(uint64_t Val,
577:                                       MachineIRBuilder &MIRBuilder,
578:                                       SPIRVGlobalRegistry *GR) {
579:   return GR->buildConstantInt(
580:       Val, MIRBuilder, GR->getOrCreateSPIRVIntegerType(32, MIRBuilder), true);
581: }
582:
583: static Register buildScopeReg(Register CLScopeRegister,
584:                               SPIRV::Scope::Scope Scope,
585:                               MachineIRBuilder &MIRBuilder,
586:                               SPIRVGlobalRegistry *GR,
587:                               MachineRegisterInfo *MRI) {
588:   if (CLScopeRegister.isValid()) {
589:     auto CLScope =
590:         static_cast<SPIRV::CLMemoryScope>(getIConstVal(CLScopeRegister, MRI));
591:     Scope = getSPIRVScope(CLScope);
592:
593:     if (CLScope == static_cast<unsigned>(Scope)) {
594:       MRI->setRegClass(CLScopeRegister, &SPIRV::iIDRegClass);
595:       return CLScopeRegister;
596:     }
597:   }
598:   return buildConstantIntReg32(Scope, MIRBuilder, GR);
599: }
600:
```
- EN: This range implements operational logic in helpers such as buildConstantInt, Register, createVirtualRegister, MachinePointerInfo, translating backend policy into executable code.
- CN: 这一段实现了 buildConstantInt、Register、createVirtualRegister、MachinePointerInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-720
```cpp
601: static void setRegClassIfNull(Register Reg, MachineRegisterInfo *MRI,
602:                               SPIRVGlobalRegistry *GR) {
603:   if (MRI->getRegClassOrNull(Reg))
604:     return;
605:   SPIRVTypeInst SpvType = GR->getSPIRVTypeForVReg(Reg);
606:   MRI->setRegClass(Reg,
607:                    SpvType ? GR->getRegClass(SpvType) : &SPIRV::iIDRegClass);
608: }
609:
610: static Register buildMemSemanticsReg(Register SemanticsRegister,
611:                                      Register PtrRegister, unsigned &Semantics,
612:                                      MachineIRBuilder &MIRBuilder,
613:                                      SPIRVGlobalRegistry *GR) {
614:   if (SemanticsRegister.isValid()) {
615:     MachineRegisterInfo *MRI = MIRBuilder.getMRI();
616:     std::memory_order Order =
617:         static_cast<std::memory_order>(getIConstVal(SemanticsRegister, MRI));
618:     Semantics =
619:         getSPIRVMemSemantics(Order) |
620:         getMemSemanticsForStorageClass(GR->getPointerStorageClass(PtrRegister));
621:     if (static_cast<unsigned>(Order) == Semantics) {
622:       MRI->setRegClass(SemanticsRegister, &SPIRV::iIDRegClass);
623:       return SemanticsRegister;
624:     }
625:   }
626:   return buildConstantIntReg32(Semantics, MIRBuilder, GR);
627: }
628:
629: static bool buildOpFromWrapper(MachineIRBuilder &MIRBuilder, unsigned Opcode,
630:                                const SPIRV::IncomingCall *Call,
631:                                Register TypeReg,
632:                                ArrayRef<uint32_t> ImmArgs = {}) {
633:   auto MIB = MIRBuilder.buildInstr(Opcode);
634:   if (TypeReg.isValid())
635:     MIB.addDef(Call->ReturnRegister).addUse(TypeReg);
636:   unsigned Sz = Call->Arguments.size() - ImmArgs.size();
637:   for (unsigned i = 0; i < Sz; ++i)
638:     MIB.addUse(Call->Arguments[i]);
639:   for (uint32_t ImmArg : ImmArgs)
640:     MIB.addImm(ImmArg);
641:   return true;
642: }
643:
644: /// Helper function for translating atomic init to OpStore.
645: static bool buildAtomicInitInst(const SPIRV::IncomingCall *Call,
646:                                 MachineIRBuilder &MIRBuilder) {
647:   if (Call->isSpirvOp())
648:     return buildOpFromWrapper(MIRBuilder, SPIRV::OpStore, Call, Register(0));
649:
650:   assert(Call->Arguments.size() == 2 &&
651:          "Need 2 arguments for atomic init translation");
652:   MIRBuilder.buildInstr(SPIRV::OpStore)
653:       .addUse(Call->Arguments[0])
654:       .addUse(Call->Arguments[1]);
655:   return true;
656: }
657:
658: /// Helper function for building an atomic load instruction.
659: static bool buildAtomicLoadInst(const SPIRV::IncomingCall *Call,
660:                                 MachineIRBuilder &MIRBuilder,
661:                                 SPIRVGlobalRegistry *GR) {
662:   Register TypeReg = GR->getSPIRVTypeID(Call->ReturnType);
663:   if (Call->isSpirvOp())
664:     return buildOpFromWrapper(MIRBuilder, SPIRV::OpAtomicLoad, Call, TypeReg);
665:
666:   Register PtrRegister = Call->Arguments[0];
667:   // TODO: if true insert call to __translate_ocl_memory_sccope before
668:   // OpAtomicLoad and the function implementation. We can use Translator's
669:   // output for transcoding/atomic_explicit_arguments.cl as an example.
670:   Register ScopeRegister =
671:       Call->Arguments.size() > 1
672:           ? Call->Arguments[1]
673:           : buildConstantIntReg32(SPIRV::Scope::Device, MIRBuilder, GR);
674:   Register MemSemanticsReg;
675:   if (Call->Arguments.size() > 2) {
676:     // TODO: Insert call to __translate_ocl_memory_order before OpAtomicLoad.
677:     MemSemanticsReg = Call->Arguments[2];
678:   } else {
679:     int Semantics =
680:         SPIRV::MemorySemantics::SequentiallyConsistent |
681:         getMemSemanticsForStorageClass(GR->getPointerStorageClass(PtrRegister));
682:     MemSemanticsReg = buildConstantIntReg32(Semantics, MIRBuilder, GR);
683:   }
684:
685:   MIRBuilder.buildInstr(SPIRV::OpAtomicLoad)
686:       .addDef(Call->ReturnRegister)
687:       .addUse(TypeReg)
688:       .addUse(PtrRegister)
689:       .addUse(ScopeRegister)
690:       .addUse(MemSemanticsReg);
691:   return true;
692: }
693:
694: /// Helper function for building an atomic store instruction.
695: static bool buildAtomicStoreInst(const SPIRV::IncomingCall *Call,
696:                                  MachineIRBuilder &MIRBuilder,
697:                                  SPIRVGlobalRegistry *GR) {
698:   if (Call->isSpirvOp())
699:     return buildOpFromWrapper(MIRBuilder, SPIRV::OpAtomicStore, Call,
700:                               Register(0));
701:
702:   Register ScopeRegister =
703:       buildConstantIntReg32(SPIRV::Scope::Device, MIRBuilder, GR);
704:   Register PtrRegister = Call->Arguments[0];
705:   int Semantics =
706:       SPIRV::MemorySemantics::SequentiallyConsistent |
707:       getMemSemanticsForStorageClass(GR->getPointerStorageClass(PtrRegister));
708:   Register MemSemanticsReg = buildConstantIntReg32(Semantics, MIRBuilder, GR);
709:   MIRBuilder.buildInstr(SPIRV::OpAtomicStore)
710:       .addUse(PtrRegister)
711:       .addUse(ScopeRegister)
712:       .addUse(MemSemanticsReg)
713:       .addUse(Call->Arguments[1]);
714:   return true;
715: }
716:
717: /// Helper function for building an atomic compare-exchange instruction.
718: static bool buildAtomicCompareExchangeInst(
719:     const SPIRV::IncomingCall *Call, const SPIRV::DemangledBuiltin *Builtin,
720:     unsigned Opcode, MachineIRBuilder &MIRBuilder, SPIRVGlobalRegistry *GR) {
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeForVReg, getRegClass, getMRI, getIConstVal, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeForVReg、getRegClass、getMRI、getIConstVal 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-840
```cpp
721:   if (Call->isSpirvOp())
722:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
723:                               GR->getSPIRVTypeID(Call->ReturnType));
724:
725:   bool IsCmpxchg = Call->Builtin->Name.contains("cmpxchg");
726:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
727:
728:   Register ObjectPtr = Call->Arguments[0];   // Pointer (volatile A *object.)
729:   Register ExpectedArg = Call->Arguments[1]; // Comparator (C* expected).
730:   Register Desired = Call->Arguments[2];     // Value (C Desired).
731:   SPIRVTypeInst SpvDesiredTy = GR->getSPIRVTypeForVReg(Desired);
732:   LLT DesiredLLT = MRI->getType(Desired);
733:
734:   assert(GR->getSPIRVTypeForVReg(ObjectPtr)->getOpcode() ==
735:          SPIRV::OpTypePointer);
736:   unsigned ExpectedType = GR->getSPIRVTypeForVReg(ExpectedArg)->getOpcode();
737:   (void)ExpectedType;
738:   assert(IsCmpxchg ? ExpectedType == SPIRV::OpTypeInt
739:                    : ExpectedType == SPIRV::OpTypePointer);
740:   assert(GR->isScalarOfType(Desired, SPIRV::OpTypeInt));
741:
742:   SPIRVTypeInst SpvObjectPtrTy = GR->getSPIRVTypeForVReg(ObjectPtr);
743:   assert(SpvObjectPtrTy->getOperand(2).isReg() && "SPIRV type is expected");
744:   auto StorageClass = static_cast<SPIRV::StorageClass::StorageClass>(
745:       SpvObjectPtrTy->getOperand(1).getImm());
746:   auto MemSemStorage = getMemSemanticsForStorageClass(StorageClass);
747:
748:   Register MemSemEqualReg;
749:   Register MemSemUnequalReg;
750:   uint64_t MemSemEqual =
751:       IsCmpxchg
752:           ? SPIRV::MemorySemantics::None
753:           : SPIRV::MemorySemantics::SequentiallyConsistent | MemSemStorage;
754:   uint64_t MemSemUnequal =
755:       IsCmpxchg
756:           ? SPIRV::MemorySemantics::None
757:           : SPIRV::MemorySemantics::SequentiallyConsistent | MemSemStorage;
758:   if (Call->Arguments.size() >= 4) {
759:     assert(Call->Arguments.size() >= 5 &&
760:            "Need 5+ args for explicit atomic cmpxchg");
761:     auto MemOrdEq =
762:         static_cast<std::memory_order>(getIConstVal(Call->Arguments[3], MRI));
763:     auto MemOrdNeq =
764:         static_cast<std::memory_order>(getIConstVal(Call->Arguments[4], MRI));
765:     MemSemEqual = getSPIRVMemSemantics(MemOrdEq) | MemSemStorage;
766:     MemSemUnequal = getSPIRVMemSemantics(MemOrdNeq) | MemSemStorage;
767:     if (static_cast<unsigned>(MemOrdEq) == MemSemEqual)
768:       MemSemEqualReg = Call->Arguments[3];
769:     if (static_cast<unsigned>(MemOrdNeq) == MemSemEqual)
770:       MemSemUnequalReg = Call->Arguments[4];
771:   }
772:   if (!MemSemEqualReg.isValid())
773:     MemSemEqualReg = buildConstantIntReg32(MemSemEqual, MIRBuilder, GR);
774:   if (!MemSemUnequalReg.isValid())
775:     MemSemUnequalReg = buildConstantIntReg32(MemSemUnequal, MIRBuilder, GR);
776:
777:   Register ScopeReg;
778:   auto Scope = IsCmpxchg ? SPIRV::Scope::Workgroup : SPIRV::Scope::Device;
779:   if (Call->Arguments.size() >= 6) {
780:     assert(Call->Arguments.size() == 6 &&
781:            "Extra args for explicit atomic cmpxchg");
782:     auto ClScope = static_cast<SPIRV::CLMemoryScope>(
783:         getIConstVal(Call->Arguments[5], MRI));
784:     Scope = getSPIRVScope(ClScope);
785:     if (ClScope == static_cast<unsigned>(Scope))
786:       ScopeReg = Call->Arguments[5];
787:   }
788:   if (!ScopeReg.isValid())
789:     ScopeReg = buildConstantIntReg32(Scope, MIRBuilder, GR);
790:
791:   Register Expected = IsCmpxchg
792:                           ? ExpectedArg
793:                           : buildLoadInst(SpvDesiredTy, ExpectedArg, MIRBuilder,
794:                                           GR, LLT::scalar(64));
795:   MRI->setType(Expected, DesiredLLT);
796:   Register Tmp = !IsCmpxchg ? MRI->createGenericVirtualRegister(DesiredLLT)
797:                             : Call->ReturnRegister;
798:   if (!MRI->getRegClassOrNull(Tmp))
799:     MRI->setRegClass(Tmp, GR->getRegClass(SpvDesiredTy));
800:   GR->assignSPIRVTypeToVReg(SpvDesiredTy, Tmp, MIRBuilder.getMF());
801:
802:   MIRBuilder.buildInstr(Opcode)
803:       .addDef(Tmp)
804:       .addUse(GR->getSPIRVTypeID(SpvDesiredTy))
805:       .addUse(ObjectPtr)
806:       .addUse(ScopeReg)
807:       .addUse(MemSemEqualReg)
808:       .addUse(MemSemUnequalReg)
809:       .addUse(Desired)
810:       .addUse(Expected);
811:   if (!IsCmpxchg) {
812:     MIRBuilder.buildInstr(SPIRV::OpStore).addUse(ExpectedArg).addUse(Tmp);
813:     MIRBuilder.buildICmp(CmpInst::ICMP_EQ, Call->ReturnRegister, Tmp, Expected);
814:   }
815:   return true;
816: }
817:
818: /// Helper function for building atomic instructions.
819: static bool buildAtomicRMWInst(const SPIRV::IncomingCall *Call, unsigned Opcode,
820:                                MachineIRBuilder &MIRBuilder,
821:                                SPIRVGlobalRegistry *GR) {
822:   if (Call->isSpirvOp())
823:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
824:                               GR->getSPIRVTypeID(Call->ReturnType));
825:
826:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
827:   Register ScopeRegister =
828:       Call->Arguments.size() >= 4 ? Call->Arguments[3] : Register();
829:
830:   assert(Call->Arguments.size() <= 4 &&
831:          "Too many args for explicit atomic RMW");
832:   ScopeRegister = buildScopeReg(ScopeRegister, SPIRV::Scope::Workgroup,
833:                                 MIRBuilder, GR, MRI);
834:
835:   Register PtrRegister = Call->Arguments[0];
836:   unsigned Semantics = SPIRV::MemorySemantics::None;
837:   Register MemSemanticsReg =
838:       Call->Arguments.size() >= 3 ? Call->Arguments[2] : Register();
839:   MemSemanticsReg = buildMemSemanticsReg(MemSemanticsReg, PtrRegister,
840:                                          Semantics, MIRBuilder, GR);
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeID, contains, getMRI, Pointer, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeID、contains、getMRI、Pointer 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 841-960
```cpp
841:   Register ValueReg = Call->Arguments[1];
842:   Register ValueTypeReg = GR->getSPIRVTypeID(Call->ReturnType);
843:   // support cl_ext_float_atomics
844:   if (Call->ReturnType->getOpcode() == SPIRV::OpTypeFloat) {
845:     if (Opcode == SPIRV::OpAtomicIAdd) {
846:       Opcode = SPIRV::OpAtomicFAddEXT;
847:     } else if (Opcode == SPIRV::OpAtomicISub) {
848:       // Translate OpAtomicISub applied to a floating type argument to
849:       // OpAtomicFAddEXT with the negative value operand
850:       Opcode = SPIRV::OpAtomicFAddEXT;
851:       Register NegValueReg =
852:           MRI->createGenericVirtualRegister(MRI->getType(ValueReg));
853:       MRI->setRegClass(NegValueReg, GR->getRegClass(Call->ReturnType));
854:       GR->assignSPIRVTypeToVReg(Call->ReturnType, NegValueReg,
855:                                 MIRBuilder.getMF());
856:       MIRBuilder.buildInstr(TargetOpcode::G_FNEG)
857:           .addDef(NegValueReg)
858:           .addUse(ValueReg);
859:       updateRegType(NegValueReg, nullptr, Call->ReturnType, GR, MIRBuilder,
860:                     MIRBuilder.getMF().getRegInfo());
861:       ValueReg = NegValueReg;
862:     }
863:   }
864:   MIRBuilder.buildInstr(Opcode)
865:       .addDef(Call->ReturnRegister)
866:       .addUse(ValueTypeReg)
867:       .addUse(PtrRegister)
868:       .addUse(ScopeRegister)
869:       .addUse(MemSemanticsReg)
870:       .addUse(ValueReg);
871:   return true;
872: }
873:
874: /// Helper function for building an atomic floating-type instruction.
875: static bool buildAtomicFloatingRMWInst(const SPIRV::IncomingCall *Call,
876:                                        unsigned Opcode,
877:                                        MachineIRBuilder &MIRBuilder,
878:                                        SPIRVGlobalRegistry *GR) {
879:   assert(Call->Arguments.size() == 4 &&
880:          "Wrong number of atomic floating-type builtin");
881:   Register PtrReg = Call->Arguments[0];
882:   Register ScopeReg = Call->Arguments[1];
883:   Register MemSemanticsReg = Call->Arguments[2];
884:   Register ValueReg = Call->Arguments[3];
885:   MIRBuilder.buildInstr(Opcode)
886:       .addDef(Call->ReturnRegister)
887:       .addUse(GR->getSPIRVTypeID(Call->ReturnType))
888:       .addUse(PtrReg)
889:       .addUse(ScopeReg)
890:       .addUse(MemSemanticsReg)
891:       .addUse(ValueReg);
892:   return true;
893: }
894:
895: /// Helper function for building atomic flag instructions (e.g.
896: /// OpAtomicFlagTestAndSet).
897: static bool buildAtomicFlagInst(const SPIRV::IncomingCall *Call,
898:                                 unsigned Opcode, MachineIRBuilder &MIRBuilder,
899:                                 SPIRVGlobalRegistry *GR) {
900:   bool IsSet = Opcode == SPIRV::OpAtomicFlagTestAndSet;
901:   Register TypeReg = GR->getSPIRVTypeID(Call->ReturnType);
902:   if (Call->isSpirvOp())
903:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
904:                               IsSet ? TypeReg : Register(0));
905:
906:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
907:   Register PtrRegister = Call->Arguments[0];
908:   unsigned Semantics = SPIRV::MemorySemantics::SequentiallyConsistent;
909:   Register MemSemanticsReg =
910:       Call->Arguments.size() >= 2 ? Call->Arguments[1] : Register();
911:   MemSemanticsReg = buildMemSemanticsReg(MemSemanticsReg, PtrRegister,
912:                                          Semantics, MIRBuilder, GR);
913:
914:   assert((Opcode != SPIRV::OpAtomicFlagClear ||
915:           (Semantics != SPIRV::MemorySemantics::Acquire &&
916:            Semantics != SPIRV::MemorySemantics::AcquireRelease)) &&
917:          "Invalid memory order argument!");
918:
919:   Register ScopeRegister =
920:       Call->Arguments.size() >= 3 ? Call->Arguments[2] : Register();
921:   ScopeRegister =
922:       buildScopeReg(ScopeRegister, SPIRV::Scope::Device, MIRBuilder, GR, MRI);
923:
924:   auto MIB = MIRBuilder.buildInstr(Opcode);
925:   if (IsSet)
926:     MIB.addDef(Call->ReturnRegister).addUse(TypeReg);
927:
928:   MIB.addUse(PtrRegister).addUse(ScopeRegister).addUse(MemSemanticsReg);
929:   return true;
930: }
931:
932: /// Helper function for building barriers, i.e., memory/control ordering
933: /// operations.
934: static bool buildBarrierInst(const SPIRV::IncomingCall *Call, unsigned Opcode,
935:                              MachineIRBuilder &MIRBuilder,
936:                              SPIRVGlobalRegistry *GR) {
937:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
938:   const auto *ST =
939:       static_cast<const SPIRVSubtarget *>(&MIRBuilder.getMF().getSubtarget());
940:   if ((Opcode == SPIRV::OpControlBarrierArriveINTEL ||
941:        Opcode == SPIRV::OpControlBarrierWaitINTEL) &&
942:       !ST->canUseExtension(SPIRV::Extension::SPV_INTEL_split_barrier)) {
943:     std::string DiagMsg = std::string(Builtin->Name) +
944:                           ": the builtin requires the following SPIR-V "
945:                           "extension: SPV_INTEL_split_barrier";
946:     report_fatal_error(DiagMsg.c_str(), false);
947:   }
948:
949:   if (Call->isSpirvOp())
950:     return buildOpFromWrapper(MIRBuilder, Opcode, Call, Register(0));
951:
952:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
953:   unsigned MemFlags = getIConstVal(Call->Arguments[0], MRI);
954:   unsigned MemSemantics = SPIRV::MemorySemantics::None;
955:
956:   if (MemFlags & SPIRV::CLK_LOCAL_MEM_FENCE)
957:     MemSemantics |= SPIRV::MemorySemantics::WorkgroupMemory;
958:
959:   if (MemFlags & SPIRV::CLK_GLOBAL_MEM_FENCE)
960:     MemSemantics |= SPIRV::MemorySemantics::CrossWorkgroupMemory;
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeID, createGenericVirtualRegister, setRegClass, getMF, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeID、createGenericVirtualRegister、setRegClass、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 961-1080
```cpp
 961:
 962:   if (MemFlags & SPIRV::CLK_IMAGE_MEM_FENCE)
 963:     MemSemantics |= SPIRV::MemorySemantics::ImageMemory;
 964:
 965:   if (Opcode == SPIRV::OpMemoryBarrier)
 966:     MemSemantics = getSPIRVMemSemantics(static_cast<std::memory_order>(
 967:                        getIConstVal(Call->Arguments[1], MRI))) |
 968:                    MemSemantics;
 969:   else if (Opcode == SPIRV::OpControlBarrierArriveINTEL)
 970:     MemSemantics |= SPIRV::MemorySemantics::Release;
 971:   else if (Opcode == SPIRV::OpControlBarrierWaitINTEL)
 972:     MemSemantics |= SPIRV::MemorySemantics::Acquire;
 973:   else
 974:     MemSemantics |= SPIRV::MemorySemantics::SequentiallyConsistent;
 975:
 976:   Register MemSemanticsReg =
 977:       MemFlags == MemSemantics
 978:           ? Call->Arguments[0]
 979:           : buildConstantIntReg32(MemSemantics, MIRBuilder, GR);
 980:   Register ScopeReg;
 981:   SPIRV::Scope::Scope Scope = SPIRV::Scope::Workgroup;
 982:   SPIRV::Scope::Scope MemScope = Scope;
 983:   if (Call->Arguments.size() >= 2) {
 984:     assert(
 985:         ((Opcode != SPIRV::OpMemoryBarrier && Call->Arguments.size() == 2) ||
 986:          (Opcode == SPIRV::OpMemoryBarrier && Call->Arguments.size() == 3)) &&
 987:         "Extra args for explicitly scoped barrier");
 988:     Register ScopeArg = (Opcode == SPIRV::OpMemoryBarrier) ? Call->Arguments[2]
 989:                                                            : Call->Arguments[1];
 990:     SPIRV::CLMemoryScope CLScope =
 991:         static_cast<SPIRV::CLMemoryScope>(getIConstVal(ScopeArg, MRI));
 992:     MemScope = getSPIRVScope(CLScope);
 993:     if (!(MemFlags & SPIRV::CLK_LOCAL_MEM_FENCE) ||
 994:         (Opcode == SPIRV::OpMemoryBarrier))
 995:       Scope = MemScope;
 996:     if (CLScope == static_cast<unsigned>(Scope))
 997:       ScopeReg = Call->Arguments[1];
 998:   }
 999:
1000:   if (!ScopeReg.isValid())
1001:     ScopeReg = buildConstantIntReg32(Scope, MIRBuilder, GR);
1002:
1003:   auto MIB = MIRBuilder.buildInstr(Opcode).addUse(ScopeReg);
1004:   if (Opcode != SPIRV::OpMemoryBarrier)
1005:     MIB.addUse(buildConstantIntReg32(MemScope, MIRBuilder, GR));
1006:   MIB.addUse(MemSemanticsReg);
1007:   return true;
1008: }
1009:
1010: /// Helper function for building extended bit operations.
1011: static bool buildExtendedBitOpsInst(const SPIRV::IncomingCall *Call,
1012:                                     unsigned Opcode,
1013:                                     MachineIRBuilder &MIRBuilder,
1014:                                     SPIRVGlobalRegistry *GR) {
1015:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1016:   const auto *ST =
1017:       static_cast<const SPIRVSubtarget *>(&MIRBuilder.getMF().getSubtarget());
1018:   if ((Opcode == SPIRV::OpBitFieldInsert ||
1019:        Opcode == SPIRV::OpBitFieldSExtract ||
1020:        Opcode == SPIRV::OpBitFieldUExtract || Opcode == SPIRV::OpBitReverse) &&
1021:       !ST->canUseExtension(SPIRV::Extension::SPV_KHR_bit_instructions)) {
1022:     std::string DiagMsg = std::string(Builtin->Name) +
1023:                           ": the builtin requires the following SPIR-V "
1024:                           "extension: SPV_KHR_bit_instructions";
1025:     report_fatal_error(DiagMsg.c_str(), false);
1026:   }
1027:
1028:   // Generate SPIRV instruction accordingly.
1029:   if (Call->isSpirvOp())
1030:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
1031:                               GR->getSPIRVTypeID(Call->ReturnType));
1032:
1033:   auto MIB = MIRBuilder.buildInstr(Opcode)
1034:                  .addDef(Call->ReturnRegister)
1035:                  .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1036:   for (unsigned i = 0; i < Call->Arguments.size(); ++i)
1037:     MIB.addUse(Call->Arguments[i]);
1038:
1039:   return true;
1040: }
1041:
1042: /// Helper function for building Intel's bindless image instructions.
1043: static bool buildBindlessImageINTELInst(const SPIRV::IncomingCall *Call,
1044:                                         unsigned Opcode,
1045:                                         MachineIRBuilder &MIRBuilder,
1046:                                         SPIRVGlobalRegistry *GR) {
1047:   // Generate SPIRV instruction accordingly.
1048:   if (Call->isSpirvOp())
1049:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
1050:                               GR->getSPIRVTypeID(Call->ReturnType));
1051:
1052:   MIRBuilder.buildInstr(Opcode)
1053:       .addDef(Call->ReturnRegister)
1054:       .addUse(GR->getSPIRVTypeID(Call->ReturnType))
1055:       .addUse(Call->Arguments[0]);
1056:
1057:   return true;
1058: }
1059:
1060: /// Helper function for building Intel's OpBitwiseFunctionINTEL instruction.
1061: static bool buildTernaryBitwiseFunctionINTELInst(
1062:     const SPIRV::IncomingCall *Call, unsigned Opcode,
1063:     MachineIRBuilder &MIRBuilder, SPIRVGlobalRegistry *GR) {
1064:   // Generate SPIRV instruction accordingly.
1065:   if (Call->isSpirvOp())
1066:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
1067:                               GR->getSPIRVTypeID(Call->ReturnType));
1068:
1069:   auto MIB = MIRBuilder.buildInstr(Opcode)
1070:                  .addDef(Call->ReturnRegister)
1071:                  .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1072:   for (unsigned i = 0; i < Call->Arguments.size(); ++i)
1073:     MIB.addUse(Call->Arguments[i]);
1074:
1075:   return true;
1076: }
1077:
1078: static bool buildImageChannelDataTypeInst(const SPIRV::IncomingCall *Call,
1079:                                           unsigned Opcode,
1080:                                           MachineIRBuilder &MIRBuilder,
```
- EN: This range implements operational logic in helpers such as buildConstantIntReg32, getIConstVal, getSPIRVScope, buildInstr, translating backend policy into executable code.
- CN: 这一段实现了 buildConstantIntReg32、getIConstVal、getSPIRVScope、buildInstr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1200
```cpp
1081:                                           SPIRVGlobalRegistry *GR) {
1082:   if (Call->isSpirvOp())
1083:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
1084:                               GR->getSPIRVTypeID(Call->ReturnType));
1085:
1086:   auto MIB = MIRBuilder.buildInstr(Opcode)
1087:                  .addDef(Call->ReturnRegister)
1088:                  .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1089:   for (unsigned i = 0; i < Call->Arguments.size(); ++i)
1090:     MIB.addUse(Call->Arguments[i]);
1091:
1092:   return true;
1093: }
1094:
1095: /// Helper function for building Intel's 2d block io instructions.
1096: static bool build2DBlockIOINTELInst(const SPIRV::IncomingCall *Call,
1097:                                     unsigned Opcode,
1098:                                     MachineIRBuilder &MIRBuilder,
1099:                                     SPIRVGlobalRegistry *GR) {
1100:   // Generate SPIRV instruction accordingly.
1101:   if (Call->isSpirvOp())
1102:     return buildOpFromWrapper(MIRBuilder, Opcode, Call, Register(0));
1103:
1104:   auto MIB = MIRBuilder.buildInstr(Opcode)
1105:                  .addDef(Call->ReturnRegister)
1106:                  .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1107:   for (unsigned i = 0; i < Call->Arguments.size(); ++i)
1108:     MIB.addUse(Call->Arguments[i]);
1109:
1110:   return true;
1111: }
1112:
1113: static bool buildPipeInst(const SPIRV::IncomingCall *Call, unsigned Opcode,
1114:                           unsigned Scope, MachineIRBuilder &MIRBuilder,
1115:                           SPIRVGlobalRegistry *GR) {
1116:   switch (Opcode) {
1117:   case SPIRV::OpCommitReadPipe:
1118:   case SPIRV::OpCommitWritePipe:
1119:     return buildOpFromWrapper(MIRBuilder, Opcode, Call, Register(0));
1120:   case SPIRV::OpGroupCommitReadPipe:
1121:   case SPIRV::OpGroupCommitWritePipe:
1122:   case SPIRV::OpGroupReserveReadPipePackets:
1123:   case SPIRV::OpGroupReserveWritePipePackets: {
1124:     Register ScopeConstReg =
1125:         MIRBuilder.buildConstant(LLT::scalar(32), Scope).getReg(0);
1126:     MachineRegisterInfo *MRI = MIRBuilder.getMRI();
1127:     MRI->setRegClass(ScopeConstReg, &SPIRV::iIDRegClass);
1128:     MachineInstrBuilder MIB;
1129:     MIB = MIRBuilder.buildInstr(Opcode);
1130:     // Add Return register and type.
1131:     if (Opcode == SPIRV::OpGroupReserveReadPipePackets ||
1132:         Opcode == SPIRV::OpGroupReserveWritePipePackets)
1133:       MIB.addDef(Call->ReturnRegister)
1134:           .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1135:
1136:     MIB.addUse(ScopeConstReg);
1137:     for (unsigned int i = 0; i < Call->Arguments.size(); ++i)
1138:       MIB.addUse(Call->Arguments[i]);
1139:
1140:     return true;
1141:   }
1142:   default:
1143:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
1144:                               GR->getSPIRVTypeID(Call->ReturnType));
1145:   }
1146: }
1147:
1148: static unsigned getNumComponentsForDim(SPIRV::Dim::Dim dim) {
1149:   switch (dim) {
1150:   case SPIRV::Dim::DIM_1D:
1151:   case SPIRV::Dim::DIM_Buffer:
1152:     return 1;
1153:   case SPIRV::Dim::DIM_2D:
1154:   case SPIRV::Dim::DIM_Cube:
1155:   case SPIRV::Dim::DIM_Rect:
1156:     return 2;
1157:   case SPIRV::Dim::DIM_3D:
1158:     return 3;
1159:   default:
1160:     report_fatal_error("Cannot get num components for given Dim");
1161:   }
1162: }
1163:
1164: /// Helper function for obtaining the number of size components.
1165: static unsigned getNumSizeComponents(SPIRVTypeInst imgType) {
1166:   assert(imgType->getOpcode() == SPIRV::OpTypeImage);
1167:   auto dim = static_cast<SPIRV::Dim::Dim>(imgType->getOperand(2).getImm());
1168:   unsigned numComps = getNumComponentsForDim(dim);
1169:   bool arrayed = imgType->getOperand(4).getImm() == 1;
1170:   return arrayed ? numComps + 1 : numComps;
1171: }
1172:
1173: static bool builtinMayNeedPromotionToVec(uint32_t BuiltinNumber) {
1174:   switch (BuiltinNumber) {
1175:   case SPIRV::OpenCLExtInst::s_min:
1176:   case SPIRV::OpenCLExtInst::u_min:
1177:   case SPIRV::OpenCLExtInst::s_max:
1178:   case SPIRV::OpenCLExtInst::u_max:
1179:   case SPIRV::OpenCLExtInst::fmax:
1180:   case SPIRV::OpenCLExtInst::fmin:
1181:   case SPIRV::OpenCLExtInst::fmax_common:
1182:   case SPIRV::OpenCLExtInst::fmin_common:
1183:   case SPIRV::OpenCLExtInst::s_clamp:
1184:   case SPIRV::OpenCLExtInst::fclamp:
1185:   case SPIRV::OpenCLExtInst::u_clamp:
1186:   case SPIRV::OpenCLExtInst::mix:
1187:   case SPIRV::OpenCLExtInst::step:
1188:   case SPIRV::OpenCLExtInst::smoothstep:
1189:   case SPIRV::OpenCLExtInst::ldexp:
1190:   case SPIRV::OpenCLExtInst::pown:
1191:   case SPIRV::OpenCLExtInst::rootn:
1192:     return true;
1193:   default:
1194:     break;
1195:   }
1196:   return false;
1197: }
1198:
1199: //===----------------------------------------------------------------------===//
1200: // Implementation functions for each builtin group
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeID, buildInstr, addDef, addUse, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeID、buildInstr、addDef、addUse 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1201-1320
```cpp
1201: //===----------------------------------------------------------------------===//
1202:
1203: static SmallVector<Register>
1204: getBuiltinCallArguments(const SPIRV::IncomingCall *Call, uint32_t BuiltinNumber,
1205:                         MachineIRBuilder &MIRBuilder, SPIRVGlobalRegistry *GR) {
1206:
1207:   Register ReturnTypeId = GR->getSPIRVTypeID(Call->ReturnType);
1208:   unsigned ResultElementCount =
1209:       GR->getScalarOrVectorComponentCount(ReturnTypeId);
1210:   bool MayNeedPromotionToVec =
1211:       builtinMayNeedPromotionToVec(BuiltinNumber) && ResultElementCount > 1;
1212:
1213:   if (!MayNeedPromotionToVec)
1214:     return {Call->Arguments.begin(), Call->Arguments.end()};
1215:
1216:   SmallVector<Register> Arguments;
1217:   for (Register Argument : Call->Arguments) {
1218:     Register VecArg = Argument;
1219:     SPIRVTypeInst ArgumentType = GR->getSPIRVTypeForVReg(Argument);
1220:     if (GR->getScalarOrVectorComponentCount(ArgumentType) == 1 &&
1221:         ArgumentType != Call->ReturnType) {
1222:       SPIRVTypeInst VecType = GR->getOrCreateSPIRVVectorType(
1223:           ArgumentType, ResultElementCount, MIRBuilder, /*EmitIR=*/true);
1224:       VecArg = createVirtualRegister(VecType, GR, MIRBuilder);
1225:       Register VecTypeId = GR->getSPIRVTypeID(VecType);
1226:       auto VecSplat = MIRBuilder.buildInstr(SPIRV::OpCompositeConstruct)
1227:                           .addDef(VecArg)
1228:                           .addUse(VecTypeId);
1229:       for (unsigned I = 0; I != ResultElementCount; ++I)
1230:         VecSplat.addUse(Argument);
1231:     }
1232:     Arguments.push_back(VecArg);
1233:   }
1234:   return Arguments;
1235: }
1236:
1237: static bool generateExtInst(const SPIRV::IncomingCall *Call,
1238:                             MachineIRBuilder &MIRBuilder,
1239:                             SPIRVGlobalRegistry *GR, const CallBase &CB) {
1240:   // Lookup the extended instruction number in the TableGen records.
1241:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1242:   uint32_t Number =
1243:       SPIRV::lookupExtendedBuiltin(Builtin->Name, Builtin->Set)->Number;
1244:   // fmin_common and fmax_common are now deprecated, and we should use fmin and
1245:   // fmax with NotInf and NotNaN flags instead. Keep original number to add
1246:   // later the NoNans and NoInfs flags.
1247:   uint32_t OrigNumber = Number;
1248:   const SPIRVSubtarget &ST =
1249:       cast<SPIRVSubtarget>(MIRBuilder.getMF().getSubtarget());
1250:   if (ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2) &&
1251:       (Number == SPIRV::OpenCLExtInst::fmin_common ||
1252:        Number == SPIRV::OpenCLExtInst::fmax_common)) {
1253:     Number = (Number == SPIRV::OpenCLExtInst::fmin_common)
1254:                  ? SPIRV::OpenCLExtInst::fmin
1255:                  : SPIRV::OpenCLExtInst::fmax;
1256:   }
1257:
1258:   Register ReturnTypeId = GR->getSPIRVTypeID(Call->ReturnType);
1259:   SmallVector<Register> Arguments =
1260:       getBuiltinCallArguments(Call, Number, MIRBuilder, GR);
1261:
1262:   MachineInstrBuilder MIB;
1263:   if (ST.canUseExtension(SPIRV::Extension::SPV_KHR_fma) &&
1264:       Number == SPIRV::OpenCLExtInst::fma) {
1265:     // Use the SPIR-V fma instruction instead of the OpenCL extended
1266:     // instruction if the extension is available.
1267:     MIB = MIRBuilder.buildInstr(SPIRV::OpFmaKHR)
1268:               .addDef(Call->ReturnRegister)
1269:               .addUse(ReturnTypeId);
1270:   } else {
1271:     // Build extended instruction.
1272:     MIB = MIRBuilder.buildInstr(SPIRV::OpExtInst)
1273:               .addDef(Call->ReturnRegister)
1274:               .addUse(ReturnTypeId)
1275:               .addImm(static_cast<uint32_t>(SPIRV::InstructionSet::OpenCL_std))
1276:               .addImm(Number);
1277:   }
1278:
1279:   for (Register Argument : Arguments)
1280:     MIB.addUse(Argument);
1281:
1282:   MIB.getInstr()->copyIRFlags(CB);
1283:   if (OrigNumber == SPIRV::OpenCLExtInst::fmin_common ||
1284:       OrigNumber == SPIRV::OpenCLExtInst::fmax_common) {
1285:     // Add NoNans and NoInfs flags to fmin/fmax instruction.
1286:     MIB.getInstr()->setFlag(MachineInstr::MIFlag::FmNoNans);
1287:     MIB.getInstr()->setFlag(MachineInstr::MIFlag::FmNoInfs);
1288:   }
1289:
1290:   // Derive fast-math flags from nofpclass attributes on the called function.
1291:   // FPFastMathMode decoration is valid on ExtInst in Kernel environments
1292:   // (SPIR-V core) or with SPV_KHR_float_controls2 for any environment.
1293:   if (ST.isKernel() ||
1294:       ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2)) {
1295:     if (const Function *F = CB.getCalledFunction()) {
1296:       bool AddNoNan = CB.getRetNoFPClass() & fcNan;
1297:       bool AddNoInf = CB.getRetNoFPClass() & fcInf;
1298:       FunctionType *FTy = F->getFunctionType();
1299:       for (unsigned I = 0, E = FTy->getNumParams();
1300:            I != E && (AddNoNan || AddNoInf); ++I) {
1301:         if (!FTy->getParamType(I)->isFloatingPointTy())
1302:           continue;
1303:         FPClassTest ArgTest = CB.getParamNoFPClass(I);
1304:         AddNoNan = AddNoNan && ArgTest & fcNan;
1305:         AddNoInf = AddNoInf && ArgTest & fcInf;
1306:       }
1307:       if (AddNoNan)
1308:         MIB.getInstr()->setFlag(MachineInstr::MIFlag::FmNoNans);
1309:       if (AddNoInf)
1310:         MIB.getInstr()->setFlag(MachineInstr::MIFlag::FmNoInfs);
1311:     }
1312:   }
1313:
1314:   return true;
1315: }
1316:
1317: static bool generateRelationalInst(const SPIRV::IncomingCall *Call,
1318:                                    MachineIRBuilder &MIRBuilder,
1319:                                    SPIRVGlobalRegistry *GR) {
1320:   // Lookup the instruction opcode in the TableGen records.
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeID, getScalarOrVectorComponentCount, getSPIRVTypeForVReg, createVirtualRegister, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeID、getScalarOrVectorComponentCount、getSPIRVTypeForVReg、createVirtualRegister 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1321-1440
```cpp
1321:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1322:   unsigned Opcode =
1323:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
1324:
1325:   Register CompareRegister;
1326:   SPIRVTypeInst RelationType = nullptr;
1327:   std::tie(CompareRegister, RelationType) =
1328:       buildBoolRegister(MIRBuilder, Call->ReturnType, GR);
1329:
1330:   // OpAny/OpAll require a boolean vector input, but OpenCL any()/all()
1331:   // builtins receive integer vectors. Convert via OpINotEqual against zero.
1332:   SmallVector<Register> Arguments(Call->Arguments.begin(),
1333:                                   Call->Arguments.end());
1334:   if ((Opcode == SPIRV::OpAny || Opcode == SPIRV::OpAll) &&
1335:       !GR->isScalarOrVectorOfType(Arguments[0], SPIRV::OpTypeBool)) {
1336:     SPIRVTypeInst ArgType = GR->getSPIRVTypeForVReg(Arguments[0]);
1337:     unsigned NumElts = GR->getScalarOrVectorComponentCount(ArgType);
1338:     SPIRVTypeInst BoolVecTy = GR->getOrCreateSPIRVVectorType(
1339:         GR->getOrCreateSPIRVBoolType(MIRBuilder, /*EmitIR=*/true), NumElts,
1340:         MIRBuilder, /*EmitIR=*/true);
1341:     Register ZeroReg =
1342:         GR->getOrCreateConsIntVector(uint64_t(0), MIRBuilder, ArgType,
1343:                                      /*EmitIR=*/true);
1344:     Register BoolVecReg = createVirtualRegister(BoolVecTy, GR, MIRBuilder);
1345:     MIRBuilder.buildInstr(SPIRV::OpINotEqual)
1346:         .addDef(BoolVecReg)
1347:         .addUse(GR->getSPIRVTypeID(BoolVecTy))
1348:         .addUse(Arguments[0])
1349:         .addUse(ZeroReg);
1350:     Arguments[0] = BoolVecReg;
1351:   }
1352:
1353:   // Build relational instruction.
1354:   auto MIB = MIRBuilder.buildInstr(Opcode)
1355:                  .addDef(CompareRegister)
1356:                  .addUse(GR->getSPIRVTypeID(RelationType));
1357:
1358:   for (auto Argument : Arguments)
1359:     MIB.addUse(Argument);
1360:
1361:   // Build select instruction.
1362:   return buildSelectInst(MIRBuilder, Call->ReturnRegister, CompareRegister,
1363:                          Call->ReturnType, GR);
1364: }
1365:
1366: static bool generateGroupInst(const SPIRV::IncomingCall *Call,
1367:                               MachineIRBuilder &MIRBuilder,
1368:                               SPIRVGlobalRegistry *GR) {
1369:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1370:   const SPIRV::GroupBuiltin *GroupBuiltin =
1371:       SPIRV::lookupGroupBuiltin(Builtin->Name);
1372:
1373:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
1374:   if (Call->isSpirvOp()) {
1375:     if (GroupBuiltin->NoGroupOperation) {
1376:       SmallVector<uint32_t, 1> ImmArgs;
1377:       if (GroupBuiltin->Opcode ==
1378:               SPIRV::OpSubgroupMatrixMultiplyAccumulateINTEL &&
1379:           Call->Arguments.size() > 4)
1380:         ImmArgs.push_back(getConstFromIntrinsic(Call->Arguments[4], MRI));
1381:       return buildOpFromWrapper(MIRBuilder, GroupBuiltin->Opcode, Call,
1382:                                 GR->getSPIRVTypeID(Call->ReturnType), ImmArgs);
1383:     }
1384:
1385:     // Group Operation is a literal
1386:     Register GroupOpReg = Call->Arguments[1];
1387:     const MachineInstr *MI = getDefInstrMaybeConstant(GroupOpReg, MRI);
1388:     if (!MI || MI->getOpcode() != TargetOpcode::G_CONSTANT)
1389:       report_fatal_error(
1390:           "Group Operation parameter must be an integer constant");
1391:     uint64_t GrpOp = MI->getOperand(1).getCImm()->getValue().getZExtValue();
1392:     Register ScopeReg = Call->Arguments[0];
1393:     auto MIB = MIRBuilder.buildInstr(GroupBuiltin->Opcode)
1394:                    .addDef(Call->ReturnRegister)
1395:                    .addUse(GR->getSPIRVTypeID(Call->ReturnType))
1396:                    .addUse(ScopeReg)
1397:                    .addImm(GrpOp);
1398:     for (unsigned i = 2; i < Call->Arguments.size(); ++i)
1399:       MIB.addUse(Call->Arguments[i]);
1400:     return true;
1401:   }
1402:
1403:   Register Arg0;
1404:   if (GroupBuiltin->HasBoolArg) {
1405:     SPIRVTypeInst BoolType = GR->getOrCreateSPIRVBoolType(MIRBuilder, true);
1406:     Register BoolReg = Call->Arguments[0];
1407:     SPIRVTypeInst BoolRegType = GR->getSPIRVTypeForVReg(BoolReg);
1408:     if (!BoolRegType)
1409:       report_fatal_error("Can't find a register's type definition");
1410:     MachineInstr *ArgInstruction = getDefInstrMaybeConstant(BoolReg, MRI);
1411:     if (ArgInstruction->getOpcode() == TargetOpcode::G_CONSTANT) {
1412:       if (BoolRegType->getOpcode() != SPIRV::OpTypeBool)
1413:         Arg0 = GR->buildConstantInt(getIConstVal(BoolReg, MRI), MIRBuilder,
1414:                                     BoolType, true);
1415:     } else {
1416:       if (BoolRegType->getOpcode() == SPIRV::OpTypeInt) {
1417:         Arg0 = MRI->createGenericVirtualRegister(LLT::scalar(1));
1418:         MRI->setRegClass(Arg0, &SPIRV::iIDRegClass);
1419:         GR->assignSPIRVTypeToVReg(BoolType, Arg0, MIRBuilder.getMF());
1420:         MIRBuilder.buildICmp(
1421:             CmpInst::ICMP_NE, Arg0, BoolReg,
1422:             GR->buildConstantInt(0, MIRBuilder, BoolRegType, true));
1423:         updateRegType(Arg0, nullptr, BoolType, GR, MIRBuilder,
1424:                       MIRBuilder.getMF().getRegInfo());
1425:       } else if (BoolRegType->getOpcode() != SPIRV::OpTypeBool) {
1426:         report_fatal_error("Expect a boolean argument");
1427:       }
1428:       // if BoolReg is a boolean register, we don't need to do anything
1429:     }
1430:   }
1431:
1432:   Register GroupResultRegister = Call->ReturnRegister;
1433:   SPIRVTypeInst GroupResultType = Call->ReturnType;
1434:
1435:   // TODO: maybe we need to check whether the result type is already boolean
1436:   // and in this case do not insert select instruction.
1437:   const bool HasBoolReturnTy =
1438:       GroupBuiltin->IsElect || GroupBuiltin->IsAllOrAny ||
1439:       GroupBuiltin->IsAllEqual || GroupBuiltin->IsLogical ||
1440:       GroupBuiltin->IsInverseBallot || GroupBuiltin->IsBallotBitExtract;
```
- EN: This range implements operational logic in helpers such as buildBoolRegister, end, isScalarOrVectorOfType, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 buildBoolRegister、end、isScalarOrVectorOfType、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1441-1560
```cpp
1441:
1442:   if (HasBoolReturnTy)
1443:     std::tie(GroupResultRegister, GroupResultType) =
1444:         buildBoolRegister(MIRBuilder, Call->ReturnType, GR);
1445:
1446:   auto Scope = Builtin->Name.starts_with("sub_group") ? SPIRV::Scope::Subgroup
1447:                                                       : SPIRV::Scope::Workgroup;
1448:   Register ScopeRegister = buildConstantIntReg32(Scope, MIRBuilder, GR);
1449:
1450:   Register VecReg;
1451:   if (GroupBuiltin->Opcode == SPIRV::OpGroupBroadcast &&
1452:       Call->Arguments.size() > 2) {
1453:     // For OpGroupBroadcast "LocalId must be an integer datatype. It must be a
1454:     // scalar, a vector with 2 components, or a vector with 3 components.",
1455:     // meaning that we must create a vector from the function arguments if
1456:     // it's a work_group_broadcast(val, local_id_x, local_id_y) or
1457:     // work_group_broadcast(val, local_id_x, local_id_y, local_id_z) call.
1458:     Register ElemReg = Call->Arguments[1];
1459:     SPIRVTypeInst ElemType = GR->getSPIRVTypeForVReg(ElemReg);
1460:     if (!ElemType || ElemType->getOpcode() != SPIRV::OpTypeInt)
1461:       report_fatal_error("Expect an integer <LocalId> argument");
1462:     unsigned VecLen = Call->Arguments.size() - 1;
1463:     VecReg = MRI->createGenericVirtualRegister(
1464:         LLT::fixed_vector(VecLen, MRI->getType(ElemReg)));
1465:     MRI->setRegClass(VecReg, &SPIRV::viIDRegClass);
1466:     SPIRVTypeInst VecType =
1467:         GR->getOrCreateSPIRVVectorType(ElemType, VecLen, MIRBuilder, true);
1468:     GR->assignSPIRVTypeToVReg(VecType, VecReg, MIRBuilder.getMF());
1469:     auto MIB =
1470:         MIRBuilder.buildInstr(TargetOpcode::G_BUILD_VECTOR).addDef(VecReg);
1471:     for (unsigned i = 1; i < Call->Arguments.size(); i++) {
1472:       MIB.addUse(Call->Arguments[i]);
1473:       setRegClassIfNull(Call->Arguments[i], MRI, GR);
1474:     }
1475:     updateRegType(VecReg, nullptr, VecType, GR, MIRBuilder,
1476:                   MIRBuilder.getMF().getRegInfo());
1477:   }
1478:
1479:   // Build work/sub group instruction.
1480:   auto MIB = MIRBuilder.buildInstr(GroupBuiltin->Opcode)
1481:                  .addDef(GroupResultRegister)
1482:                  .addUse(GR->getSPIRVTypeID(GroupResultType))
1483:                  .addUse(ScopeRegister);
1484:
1485:   if (!GroupBuiltin->NoGroupOperation)
1486:     MIB.addImm(GroupBuiltin->GroupOperation);
1487:   if (Call->Arguments.size() > 0) {
1488:     MIB.addUse(Arg0.isValid() ? Arg0 : Call->Arguments[0]);
1489:     setRegClassIfNull(Call->Arguments[0], MRI, GR);
1490:     if (VecReg.isValid())
1491:       MIB.addUse(VecReg);
1492:     else
1493:       for (unsigned i = 1; i < Call->Arguments.size(); i++)
1494:         MIB.addUse(Call->Arguments[i]);
1495:   }
1496:
1497:   // Build select instruction.
1498:   if (HasBoolReturnTy)
1499:     buildSelectInst(MIRBuilder, Call->ReturnRegister, GroupResultRegister,
1500:                     Call->ReturnType, GR);
1501:   return true;
1502: }
1503:
1504: static bool generateIntelSubgroupsInst(const SPIRV::IncomingCall *Call,
1505:                                        MachineIRBuilder &MIRBuilder,
1506:                                        SPIRVGlobalRegistry *GR) {
1507:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1508:   MachineFunction &MF = MIRBuilder.getMF();
1509:   const auto *ST = static_cast<const SPIRVSubtarget *>(&MF.getSubtarget());
1510:   const SPIRV::IntelSubgroupsBuiltin *IntelSubgroups =
1511:       SPIRV::lookupIntelSubgroupsBuiltin(Builtin->Name);
1512:
1513:   if (IntelSubgroups->IsMedia &&
1514:       !ST->canUseExtension(SPIRV::Extension::SPV_INTEL_media_block_io)) {
1515:     std::string DiagMsg = std::string(Builtin->Name) +
1516:                           ": the builtin requires the following SPIR-V "
1517:                           "extension: SPV_INTEL_media_block_io";
1518:     report_fatal_error(DiagMsg.c_str(), false);
1519:   } else if (!IntelSubgroups->IsMedia &&
1520:              !ST->canUseExtension(SPIRV::Extension::SPV_INTEL_subgroups)) {
1521:     std::string DiagMsg = std::string(Builtin->Name) +
1522:                           ": the builtin requires the following SPIR-V "
1523:                           "extension: SPV_INTEL_subgroups";
1524:     report_fatal_error(DiagMsg.c_str(), false);
1525:   }
1526:
1527:   uint32_t OpCode = IntelSubgroups->Opcode;
1528:   if (Call->isSpirvOp()) {
1529:     bool IsSet = OpCode != SPIRV::OpSubgroupBlockWriteINTEL &&
1530:                  OpCode != SPIRV::OpSubgroupImageBlockWriteINTEL &&
1531:                  OpCode != SPIRV::OpSubgroupImageMediaBlockWriteINTEL;
1532:     return buildOpFromWrapper(MIRBuilder, OpCode, Call,
1533:                               IsSet ? GR->getSPIRVTypeID(Call->ReturnType)
1534:                                     : Register(0));
1535:   }
1536:
1537:   if (IntelSubgroups->IsBlock) {
1538:     // Minimal number or arguments set in TableGen records is 1
1539:     if (SPIRVTypeInst Arg0Type = GR->getSPIRVTypeForVReg(Call->Arguments[0])) {
1540:       if (Arg0Type->getOpcode() == SPIRV::OpTypeImage) {
1541:         // TODO: add required validation from the specification:
1542:         // "'Image' must be an object whose type is OpTypeImage with a 'Sampled'
1543:         // operand of 0 or 2. If the 'Sampled' operand is 2, then some
1544:         // dimensions require a capability."
1545:         switch (OpCode) {
1546:         case SPIRV::OpSubgroupBlockReadINTEL:
1547:           OpCode = SPIRV::OpSubgroupImageBlockReadINTEL;
1548:           break;
1549:         case SPIRV::OpSubgroupBlockWriteINTEL:
1550:           OpCode = SPIRV::OpSubgroupImageBlockWriteINTEL;
1551:           break;
1552:         }
1553:       }
1554:     }
1555:   }
1556:
1557:   // TODO: opaque pointers types should be eventually resolved in such a way
1558:   // that validation of block read is enabled with respect to the following
1559:   // specification requirement:
1560:   // "'Result Type' may be a scalar or vector type, and its component type must
```
- EN: This range implements operational logic in helpers such as buildBoolRegister, buildConstantIntReg32, size, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 buildBoolRegister、buildConstantIntReg32、size、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1561-1680
```cpp
1561:   // be equal to the type pointed to by 'Ptr'."
1562:   // For example, function parameter type should not be default i8 pointer, but
1563:   // depend on the result type of the instruction where it is used as a pointer
1564:   // argument of OpSubgroupBlockReadINTEL
1565:
1566:   // Build Intel subgroups instruction
1567:   MachineInstrBuilder MIB =
1568:       IntelSubgroups->IsWrite
1569:           ? MIRBuilder.buildInstr(OpCode)
1570:           : MIRBuilder.buildInstr(OpCode)
1571:                 .addDef(Call->ReturnRegister)
1572:                 .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1573:   for (size_t i = 0; i < Call->Arguments.size(); ++i)
1574:     MIB.addUse(Call->Arguments[i]);
1575:   return true;
1576: }
1577:
1578: static bool generateGroupUniformInst(const SPIRV::IncomingCall *Call,
1579:                                      MachineIRBuilder &MIRBuilder,
1580:                                      SPIRVGlobalRegistry *GR) {
1581:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1582:   MachineFunction &MF = MIRBuilder.getMF();
1583:   const auto *ST = static_cast<const SPIRVSubtarget *>(&MF.getSubtarget());
1584:   if (!ST->canUseExtension(
1585:           SPIRV::Extension::SPV_KHR_uniform_group_instructions)) {
1586:     std::string DiagMsg = std::string(Builtin->Name) +
1587:                           ": the builtin requires the following SPIR-V "
1588:                           "extension: SPV_KHR_uniform_group_instructions";
1589:     report_fatal_error(DiagMsg.c_str(), false);
1590:   }
1591:   const SPIRV::GroupUniformBuiltin *GroupUniform =
1592:       SPIRV::lookupGroupUniformBuiltin(Builtin->Name);
1593:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
1594:
1595:   Register GroupResultReg = Call->ReturnRegister;
1596:   Register ScopeReg = Call->Arguments[0];
1597:   Register ValueReg = Call->Arguments[2];
1598:
1599:   // Group Operation
1600:   Register ConstGroupOpReg = Call->Arguments[1];
1601:   const MachineInstr *Const = getDefInstrMaybeConstant(ConstGroupOpReg, MRI);
1602:   if (!Const || Const->getOpcode() != TargetOpcode::G_CONSTANT)
1603:     report_fatal_error(
1604:         "expect a constant group operation for a uniform group instruction",
1605:         false);
1606:   const MachineOperand &ConstOperand = Const->getOperand(1);
1607:   if (!ConstOperand.isCImm())
1608:     report_fatal_error("uniform group instructions: group operation must be an "
1609:                        "integer constant",
1610:                        false);
1611:
1612:   auto MIB = MIRBuilder.buildInstr(GroupUniform->Opcode)
1613:                  .addDef(GroupResultReg)
1614:                  .addUse(GR->getSPIRVTypeID(Call->ReturnType))
1615:                  .addUse(ScopeReg);
1616:   addNumImm(ConstOperand.getCImm()->getValue(), MIB);
1617:   MIB.addUse(ValueReg);
1618:
1619:   return true;
1620: }
1621:
1622: static bool generateKernelClockInst(const SPIRV::IncomingCall *Call,
1623:                                     MachineIRBuilder &MIRBuilder,
1624:                                     SPIRVGlobalRegistry *GR) {
1625:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1626:   MachineFunction &MF = MIRBuilder.getMF();
1627:   const auto *ST = static_cast<const SPIRVSubtarget *>(&MF.getSubtarget());
1628:   if (!ST->canUseExtension(SPIRV::Extension::SPV_KHR_shader_clock)) {
1629:     std::string DiagMsg = std::string(Builtin->Name) +
1630:                           ": the builtin requires the following SPIR-V "
1631:                           "extension: SPV_KHR_shader_clock";
1632:     report_fatal_error(DiagMsg.c_str(), false);
1633:   }
1634:
1635:   Register ResultReg = Call->ReturnRegister;
1636:
1637:   if (Builtin->Name == "__spirv_ReadClockKHR") {
1638:     MIRBuilder.buildInstr(SPIRV::OpReadClockKHR)
1639:         .addDef(ResultReg)
1640:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
1641:         .addUse(Call->Arguments[0]);
1642:   } else {
1643:     // Deduce the `Scope` operand from the builtin function name.
1644:     SPIRV::Scope::Scope ScopeArg =
1645:         StringSwitch<SPIRV::Scope::Scope>(Builtin->Name)
1646:             .EndsWith("device", SPIRV::Scope::Scope::Device)
1647:             .EndsWith("work_group", SPIRV::Scope::Scope::Workgroup)
1648:             .EndsWith("sub_group", SPIRV::Scope::Scope::Subgroup);
1649:     Register ScopeReg = buildConstantIntReg32(ScopeArg, MIRBuilder, GR);
1650:
1651:     MIRBuilder.buildInstr(SPIRV::OpReadClockKHR)
1652:         .addDef(ResultReg)
1653:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
1654:         .addUse(ScopeReg);
1655:   }
1656:
1657:   return true;
1658: }
1659:
1660: // These queries ask for a single size_t result for a given dimension index,
1661: // e.g. size_t get_global_id(uint dimindex). In SPIR-V, the builtins
1662: // corresponding to these values are all vec3 types, so we need to extract the
1663: // correct index or return DefaultValue (0 or 1 depending on the query). We also
1664: // handle extending or truncating in case size_t does not match the expected
1665: // result type's bitwidth.
1666: //
1667: // For a constant index >= 3 we generate:
1668: //  %res = OpConstant %SizeT DefaultValue
1669: //
1670: // For other indices we generate:
1671: //  %g = OpVariable %ptr_V3_SizeT Input
1672: //  OpDecorate %g BuiltIn XXX
1673: //  OpDecorate %g LinkageAttributes "__spirv_BuiltInXXX"
1674: //  OpDecorate %g Constant
1675: //  %loadedVec = OpLoad %V3_SizeT %g
1676: //
1677: //  Then, if the index is constant < 3, we generate:
1678: //    %res = OpCompositeExtract %SizeT %loadedVec idx
1679: //  If the index is dynamic, we generate:
1680: //    %tmp = OpVectorExtractDynamic %SizeT %loadedVec %idx
```
- EN: This range implements operational logic in helpers such as buildInstr, addDef, addUse, size, translating backend policy into executable code.
- CN: 这一段实现了 buildInstr、addDef、addUse、size 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1681-1800
```cpp
1681: //    %cmp = OpULessThan %bool %idx %const_3
1682: //    %res = OpSelect %SizeT %cmp %tmp %const_<DefaultValue>
1683: //
1684: //  If the bitwidth of %res does not match the expected return type, we add an
1685: //  extend or truncate.
1686: static bool genWorkgroupQuery(const SPIRV::IncomingCall *Call,
1687:                               MachineIRBuilder &MIRBuilder,
1688:                               SPIRVGlobalRegistry *GR,
1689:                               SPIRV::BuiltIn::BuiltIn BuiltinValue,
1690:                               uint64_t DefaultValue) {
1691:   Register IndexRegister = Call->Arguments[0];
1692:   const unsigned ResultWidth = Call->ReturnType->getOperand(1).getImm();
1693:   const unsigned PointerSize = GR->getPointerSize();
1694:   const SPIRVTypeInst PointerSizeType =
1695:       GR->getOrCreateSPIRVIntegerType(PointerSize, MIRBuilder);
1696:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
1697:   auto IndexInstruction = getDefInstrMaybeConstant(IndexRegister, MRI);
1698:
1699:   // Set up the final register to do truncation or extension on at the end.
1700:   Register ToTruncate = Call->ReturnRegister;
1701:
1702:   // If the index is constant, we can statically determine if it is in range.
1703:   bool IsConstantIndex =
1704:       IndexInstruction->getOpcode() == TargetOpcode::G_CONSTANT;
1705:
1706:   // If it's out of range (max dimension is 3), we can just return the constant
1707:   // default value (0 or 1 depending on which query function).
1708:   if (IsConstantIndex && getIConstVal(IndexRegister, MRI) >= 3) {
1709:     Register DefaultReg = Call->ReturnRegister;
1710:     if (PointerSize != ResultWidth) {
1711:       DefaultReg = MRI->createGenericVirtualRegister(LLT::scalar(PointerSize));
1712:       MRI->setRegClass(DefaultReg, &SPIRV::iIDRegClass);
1713:       GR->assignSPIRVTypeToVReg(PointerSizeType, DefaultReg,
1714:                                 MIRBuilder.getMF());
1715:       ToTruncate = DefaultReg;
1716:     }
1717:     auto NewRegister =
1718:         GR->buildConstantInt(DefaultValue, MIRBuilder, PointerSizeType, true);
1719:     MIRBuilder.buildCopy(DefaultReg, NewRegister);
1720:   } else { // If it could be in range, we need to load from the given builtin.
1721:     auto Vec3Ty =
1722:         GR->getOrCreateSPIRVVectorType(PointerSizeType, 3, MIRBuilder, true);
1723:     Register LoadedVector =
1724:         buildBuiltinVariableLoad(MIRBuilder, Vec3Ty, GR, BuiltinValue,
1725:                                  LLT::fixed_vector(3, PointerSize));
1726:     // Set up the vreg to extract the result to (possibly a new temporary one).
1727:     Register Extracted = Call->ReturnRegister;
1728:     if (!IsConstantIndex || PointerSize != ResultWidth) {
1729:       Extracted = MRI->createGenericVirtualRegister(LLT::scalar(PointerSize));
1730:       MRI->setRegClass(Extracted, &SPIRV::iIDRegClass);
1731:       GR->assignSPIRVTypeToVReg(PointerSizeType, Extracted, MIRBuilder.getMF());
1732:     }
1733:     // Use Intrinsic::spv_extractelt so dynamic vs static extraction is
1734:     // handled later: extr = spv_extractelt LoadedVector, IndexRegister.
1735:     MachineInstrBuilder ExtractInst = MIRBuilder.buildIntrinsic(
1736:         Intrinsic::spv_extractelt, ArrayRef<Register>{Extracted}, true, false);
1737:     ExtractInst.addUse(LoadedVector).addUse(IndexRegister);
1738:
1739:     // If the index is dynamic, need check if it's < 3, and then use a select.
1740:     if (!IsConstantIndex) {
1741:       updateRegType(Extracted, nullptr, PointerSizeType, GR, MIRBuilder, *MRI);
1742:
1743:       auto IndexType = GR->getSPIRVTypeForVReg(IndexRegister);
1744:       auto BoolType = GR->getOrCreateSPIRVBoolType(MIRBuilder, true);
1745:
1746:       Register CompareRegister =
1747:           MRI->createGenericVirtualRegister(LLT::scalar(1));
1748:       MRI->setRegClass(CompareRegister, &SPIRV::iIDRegClass);
1749:       GR->assignSPIRVTypeToVReg(BoolType, CompareRegister, MIRBuilder.getMF());
1750:
1751:       // Use G_ICMP to check if idxVReg < 3.
1752:       MIRBuilder.buildICmp(
1753:           CmpInst::ICMP_ULT, CompareRegister, IndexRegister,
1754:           GR->buildConstantInt(3, MIRBuilder, IndexType, true));
1755:
1756:       // Get constant for the default value (0 or 1 depending on which
1757:       // function).
1758:       Register DefaultRegister =
1759:           GR->buildConstantInt(DefaultValue, MIRBuilder, PointerSizeType, true);
1760:
1761:       // Get a register for the selection result (possibly a new temporary one).
1762:       Register SelectionResult = Call->ReturnRegister;
1763:       if (PointerSize != ResultWidth) {
1764:         SelectionResult =
1765:             MRI->createGenericVirtualRegister(LLT::scalar(PointerSize));
1766:         MRI->setRegClass(SelectionResult, &SPIRV::iIDRegClass);
1767:         GR->assignSPIRVTypeToVReg(PointerSizeType, SelectionResult,
1768:                                   MIRBuilder.getMF());
1769:       }
1770:       // Create the final G_SELECT to return the extracted value or the default.
1771:       MIRBuilder.buildSelect(SelectionResult, CompareRegister, Extracted,
1772:                              DefaultRegister);
1773:       ToTruncate = SelectionResult;
1774:     } else {
1775:       ToTruncate = Extracted;
1776:     }
1777:   }
1778:   // Alter the result's bitwidth if it does not match the SizeT value extracted.
1779:   if (PointerSize != ResultWidth)
1780:     MIRBuilder.buildZExtOrTrunc(Call->ReturnRegister, ToTruncate);
1781:   return true;
1782: }
1783:
1784: static bool generateBuiltinVar(const SPIRV::IncomingCall *Call,
1785:                                MachineIRBuilder &MIRBuilder,
1786:                                SPIRVGlobalRegistry *GR) {
1787:   // Lookup the builtin variable record.
1788:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1789:   SPIRV::BuiltIn::BuiltIn Value =
1790:       SPIRV::lookupGetBuiltin(Builtin->Name, Builtin->Set)->Value;
1791:
1792:   if (Value == SPIRV::BuiltIn::GlobalInvocationId)
1793:     return genWorkgroupQuery(Call, MIRBuilder, GR, Value, 0);
1794:
1795:   // Build a load instruction for the builtin variable.
1796:   unsigned BitWidth = GR->getScalarOrVectorBitWidth(Call->ReturnType);
1797:   LLT LLType;
1798:   if (Call->ReturnType->getOpcode() == SPIRV::OpTypeVector)
1799:     LLType = LLT::fixed_vector(
1800:         GR->getScalarOrVectorComponentCount(Call->ReturnType), BitWidth);
```
- EN: This range implements operational logic in helpers such as getOperand, getPointerSize, getOrCreateSPIRVIntegerType, getMRI, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getPointerSize、getOrCreateSPIRVIntegerType、getMRI 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1801-1920
```cpp
1801:   else
1802:     LLType = LLT::scalar(BitWidth);
1803:
1804:   return buildBuiltinVariableLoad(MIRBuilder, Call->ReturnType, GR, Value,
1805:                                   LLType, Call->ReturnRegister);
1806: }
1807:
1808: static bool generateAtomicInst(const SPIRV::IncomingCall *Call,
1809:                                MachineIRBuilder &MIRBuilder,
1810:                                SPIRVGlobalRegistry *GR) {
1811:   // Lookup the instruction opcode in the TableGen records.
1812:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1813:   unsigned Opcode =
1814:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
1815:
1816:   switch (Opcode) {
1817:   case SPIRV::OpStore:
1818:     return buildAtomicInitInst(Call, MIRBuilder);
1819:   case SPIRV::OpAtomicLoad:
1820:     return buildAtomicLoadInst(Call, MIRBuilder, GR);
1821:   case SPIRV::OpAtomicStore:
1822:     return buildAtomicStoreInst(Call, MIRBuilder, GR);
1823:   case SPIRV::OpAtomicCompareExchange:
1824:   case SPIRV::OpAtomicCompareExchangeWeak:
1825:     return buildAtomicCompareExchangeInst(Call, Builtin, Opcode, MIRBuilder,
1826:                                           GR);
1827:   case SPIRV::OpAtomicIAdd:
1828:   case SPIRV::OpAtomicISub:
1829:   case SPIRV::OpAtomicOr:
1830:   case SPIRV::OpAtomicXor:
1831:   case SPIRV::OpAtomicAnd:
1832:   case SPIRV::OpAtomicExchange:
1833:   case SPIRV::OpAtomicSMax:
1834:   case SPIRV::OpAtomicSMin:
1835:   case SPIRV::OpAtomicUMax:
1836:   case SPIRV::OpAtomicUMin:
1837:     return buildAtomicRMWInst(Call, Opcode, MIRBuilder, GR);
1838:   case SPIRV::OpMemoryBarrier:
1839:     return buildBarrierInst(Call, SPIRV::OpMemoryBarrier, MIRBuilder, GR);
1840:   case SPIRV::OpAtomicFlagTestAndSet:
1841:   case SPIRV::OpAtomicFlagClear:
1842:     return buildAtomicFlagInst(Call, Opcode, MIRBuilder, GR);
1843:   default:
1844:     if (Call->isSpirvOp())
1845:       return buildOpFromWrapper(MIRBuilder, Opcode, Call,
1846:                                 GR->getSPIRVTypeID(Call->ReturnType));
1847:     return false;
1848:   }
1849: }
1850:
1851: static bool generateAtomicFloatingInst(const SPIRV::IncomingCall *Call,
1852:                                        MachineIRBuilder &MIRBuilder,
1853:                                        SPIRVGlobalRegistry *GR) {
1854:   // Lookup the instruction opcode in the TableGen records.
1855:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1856:   unsigned Opcode = SPIRV::lookupAtomicFloatingBuiltin(Builtin->Name)->Opcode;
1857:
1858:   switch (Opcode) {
1859:   case SPIRV::OpAtomicFAddEXT:
1860:   case SPIRV::OpAtomicFMinEXT:
1861:   case SPIRV::OpAtomicFMaxEXT:
1862:     return buildAtomicFloatingRMWInst(Call, Opcode, MIRBuilder, GR);
1863:   default:
1864:     return false;
1865:   }
1866: }
1867:
1868: static bool generateBarrierInst(const SPIRV::IncomingCall *Call,
1869:                                 MachineIRBuilder &MIRBuilder,
1870:                                 SPIRVGlobalRegistry *GR) {
1871:   // Lookup the instruction opcode in the TableGen records.
1872:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1873:   unsigned Opcode =
1874:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
1875:
1876:   return buildBarrierInst(Call, Opcode, MIRBuilder, GR);
1877: }
1878:
1879: static bool generateCastToPtrInst(const SPIRV::IncomingCall *Call,
1880:                                   MachineIRBuilder &MIRBuilder,
1881:                                   SPIRVGlobalRegistry *GR) {
1882:   // Lookup the instruction opcode in the TableGen records.
1883:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1884:   unsigned Opcode =
1885:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
1886:
1887:   if (Opcode == SPIRV::OpGenericCastToPtrExplicit) {
1888:     SPIRV::StorageClass::StorageClass ResSC =
1889:         GR->getPointerStorageClass(Call->ReturnRegister);
1890:     if (!isGenericCastablePtr(ResSC))
1891:       return false;
1892:
1893:     MIRBuilder.buildInstr(Opcode)
1894:         .addDef(Call->ReturnRegister)
1895:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
1896:         .addUse(Call->Arguments[0])
1897:         .addImm(ResSC);
1898:   } else {
1899:     MIRBuilder.buildInstr(TargetOpcode::G_ADDRSPACE_CAST)
1900:         .addDef(Call->ReturnRegister)
1901:         .addUse(Call->Arguments[0]);
1902:   }
1903:   return true;
1904: }
1905:
1906: static bool generateDotOrFMulInst(const StringRef DemangledCall,
1907:                                   const SPIRV::IncomingCall *Call,
1908:                                   MachineIRBuilder &MIRBuilder,
1909:                                   SPIRVGlobalRegistry *GR) {
1910:   if (Call->isSpirvOp())
1911:     return buildOpFromWrapper(MIRBuilder, SPIRV::OpDot, Call,
1912:                               GR->getSPIRVTypeID(Call->ReturnType));
1913:
1914:   bool IsVec = GR->getSPIRVTypeForVReg(Call->Arguments[0])->getOpcode() ==
1915:                SPIRV::OpTypeVector;
1916:   // Use OpDot only in case of vector args and OpFMul in case of scalar args.
1917:   uint32_t OC = IsVec ? SPIRV::OpDot : SPIRV::OpFMulS;
1918:   bool IsSwapReq = false;
1919:
1920:   const auto *ST =
```
- EN: This range implements operational logic in helpers such as LLT::scalar, buildAtomicInitInst, buildAtomicLoadInst, buildAtomicStoreInst, translating backend policy into executable code.
- CN: 这一段实现了 LLT::scalar、buildAtomicInitInst、buildAtomicLoadInst、buildAtomicStoreInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1921-2040
```cpp
1921:       static_cast<const SPIRVSubtarget *>(&MIRBuilder.getMF().getSubtarget());
1922:   if (GR->isScalarOrVectorOfType(Call->ReturnRegister, SPIRV::OpTypeInt) &&
1923:       (ST->canUseExtension(SPIRV::Extension::SPV_KHR_integer_dot_product) ||
1924:        ST->isAtLeastSPIRVVer(VersionTuple(1, 6)))) {
1925:     const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1926:     const SPIRV::IntegerDotProductBuiltin *IntDot =
1927:         SPIRV::lookupIntegerDotProductBuiltin(Builtin->Name);
1928:     if (IntDot) {
1929:       OC = IntDot->Opcode;
1930:       IsSwapReq = IntDot->IsSwapReq;
1931:     } else if (IsVec) {
1932:       // Handling "dot" and "dot_acc_sat" builtins which use vectors of
1933:       // integers.
1934:       LLVMContext &Ctx = MIRBuilder.getContext();
1935:       SmallVector<StringRef, 10> TypeStrs;
1936:       SPIRV::parseBuiltinTypeStr(TypeStrs, DemangledCall, Ctx);
1937:       bool IsFirstSigned = TypeStrs[0].trim()[0] != 'u';
1938:       bool IsSecondSigned = TypeStrs[1].trim()[0] != 'u';
1939:
1940:       if (Call->BuiltinName == "dot") {
1941:         if (IsFirstSigned && IsSecondSigned)
1942:           OC = SPIRV::OpSDot;
1943:         else if (!IsFirstSigned && !IsSecondSigned)
1944:           OC = SPIRV::OpUDot;
1945:         else {
1946:           OC = SPIRV::OpSUDot;
1947:           if (!IsFirstSigned)
1948:             IsSwapReq = true;
1949:         }
1950:       } else if (Call->BuiltinName == "dot_acc_sat") {
1951:         if (IsFirstSigned && IsSecondSigned)
1952:           OC = SPIRV::OpSDotAccSat;
1953:         else if (!IsFirstSigned && !IsSecondSigned)
1954:           OC = SPIRV::OpUDotAccSat;
1955:         else {
1956:           OC = SPIRV::OpSUDotAccSat;
1957:           if (!IsFirstSigned)
1958:             IsSwapReq = true;
1959:         }
1960:       }
1961:     }
1962:   }
1963:
1964:   MachineInstrBuilder MIB = MIRBuilder.buildInstr(OC)
1965:                                 .addDef(Call->ReturnRegister)
1966:                                 .addUse(GR->getSPIRVTypeID(Call->ReturnType));
1967:
1968:   if (IsSwapReq) {
1969:     MIB.addUse(Call->Arguments[1]);
1970:     MIB.addUse(Call->Arguments[0]);
1971:     // needed for dot_acc_sat* builtins
1972:     for (size_t i = 2; i < Call->Arguments.size(); ++i)
1973:       MIB.addUse(Call->Arguments[i]);
1974:   } else {
1975:     for (size_t i = 0; i < Call->Arguments.size(); ++i)
1976:       MIB.addUse(Call->Arguments[i]);
1977:   }
1978:
1979:   // Add Packed Vector Format for Integer dot product builtins if arguments are
1980:   // scalar
1981:   if (!IsVec && OC != SPIRV::OpFMulS)
1982:     MIB.addImm(SPIRV::PackedVectorFormat4x8Bit);
1983:
1984:   return true;
1985: }
1986:
1987: static bool generateWaveInst(const SPIRV::IncomingCall *Call,
1988:                              MachineIRBuilder &MIRBuilder,
1989:                              SPIRVGlobalRegistry *GR) {
1990:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
1991:   SPIRV::BuiltIn::BuiltIn Value =
1992:       SPIRV::lookupGetBuiltin(Builtin->Name, Builtin->Set)->Value;
1993:
1994:   // For now, we only support a single Wave intrinsic with a single return type.
1995:   assert(Call->ReturnType->getOpcode() == SPIRV::OpTypeInt);
1996:   LLT LLType = LLT::scalar(GR->getScalarOrVectorBitWidth(Call->ReturnType));
1997:
1998:   return buildBuiltinVariableLoad(
1999:       MIRBuilder, Call->ReturnType, GR, Value, LLType, Call->ReturnRegister,
2000:       /* isConst= */ false, /* LinkageType= */ std::nullopt);
2001: }
2002:
2003: // Build a SPIR-V instruction with struct return via sret pointer:
2004: //     Res = Opcode RetType Op1 Op2
2005: //     OpStore SRetReg Res
2006: static void buildSRetInst(unsigned Opcode, Register SRetReg, Register Op1Reg,
2007:                           Register Op2Reg, SPIRVTypeInst RetType,
2008:                           MachineIRBuilder &MIRBuilder,
2009:                           SPIRVGlobalRegistry *GR) {
2010:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2011:   Register ResReg = MRI->createVirtualRegister(&SPIRV::iIDRegClass);
2012:   if (const TargetRegisterClass *DstRC = MRI->getRegClassOrNull(Op1Reg)) {
2013:     MRI->setRegClass(ResReg, DstRC);
2014:     MRI->setType(ResReg, MRI->getType(Op1Reg));
2015:   }
2016:   GR->assignSPIRVTypeToVReg(RetType, ResReg, MIRBuilder.getMF());
2017:   MIRBuilder.buildInstr(Opcode)
2018:       .addDef(ResReg)
2019:       .addUse(GR->getSPIRVTypeID(RetType))
2020:       .addUse(Op1Reg)
2021:       .addUse(Op2Reg);
2022:   MIRBuilder.buildInstr(SPIRV::OpStore).addUse(SRetReg).addUse(ResReg);
2023: }
2024:
2025: // We expect a builtin
2026: //     Name(ptr sret([RetType]) %result, Type %operand1, Type %operand1)
2027: // where %result is a pointer to where the result of the builtin execution
2028: // is to be stored, and generate the following instructions:
2029: //     Res = Opcode RetType Operand1 Operand1
2030: //     OpStore RetVariable Res
2031: static bool generateICarryBorrowInst(const SPIRV::IncomingCall *Call,
2032:                                      MachineIRBuilder &MIRBuilder,
2033:                                      SPIRVGlobalRegistry *GR) {
2034:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2035:   unsigned Opcode =
2036:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2037:
2038:   Register SRetReg = Call->Arguments[0];
2039:   SPIRVTypeInst PtrRetType = GR->getSPIRVTypeForVReg(SRetReg);
2040:   SPIRVTypeInst RetType = GR->getPointeeType(PtrRetType);
```
- EN: This range implements operational logic in helpers such as getMF, isAtLeastSPIRVVer, SPIRV::lookupIntegerDotProductBuiltin, getContext, translating backend policy into executable code.
- CN: 这一段实现了 getMF、isAtLeastSPIRVVer、SPIRV::lookupIntegerDotProductBuiltin、getContext 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2041-2160
```cpp
2041:   if (!RetType)
2042:     report_fatal_error("The first parameter must be a pointer");
2043:   if (RetType->getOpcode() != SPIRV::OpTypeStruct)
2044:     report_fatal_error("Expected struct type result for the arithmetic with "
2045:                        "overflow builtins");
2046:
2047:   SPIRVTypeInst OpType1 = GR->getSPIRVTypeForVReg(Call->Arguments[1]);
2048:   SPIRVTypeInst OpType2 = GR->getSPIRVTypeForVReg(Call->Arguments[2]);
2049:   if (!OpType1 || !OpType2 || OpType1 != OpType2)
2050:     report_fatal_error("Operands must have the same type");
2051:   if (OpType1->getOpcode() == SPIRV::OpTypeVector)
2052:     switch (Opcode) {
2053:     case SPIRV::OpIAddCarryS:
2054:       Opcode = SPIRV::OpIAddCarryV;
2055:       break;
2056:     case SPIRV::OpISubBorrowS:
2057:       Opcode = SPIRV::OpISubBorrowV;
2058:       break;
2059:     }
2060:
2061:   buildSRetInst(Opcode, SRetReg, Call->Arguments[1], Call->Arguments[2],
2062:                 RetType, MIRBuilder, GR);
2063:   return true;
2064: }
2065:
2066: // We expect a builtin in one of two forms:
2067: //
2068: //  (1) sret convention (3 arguments):
2069: //     void Name(ptr sret([RetType]) %result, Type %operand1, Type %operand2)
2070: //     =>  Res = Opcode RetType Operand1 Operand2
2071: //         OpStore %result Res
2072: //
2073: //  (2) direct return convention (2 arguments):
2074: //     RetType Name(Type %operand1, Type %operand2)
2075: //     =>  Res = Opcode RetType Operand1 Operand2
2076: //
2077: // RetType is a struct with two members of the same type as the operands.
2078: static bool generateMulExtendedInst(const SPIRV::IncomingCall *Call,
2079:                                     MachineIRBuilder &MIRBuilder,
2080:                                     SPIRVGlobalRegistry *GR) {
2081:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2082:   unsigned Opcode =
2083:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2084:   assert((Opcode == SPIRV::OpUMulExtended || Opcode == SPIRV::OpSMulExtended) &&
2085:          "Expected OpUMulExtended or OpSMulExtended");
2086:
2087:   const bool IsSret =
2088:       !Call->ReturnType || Call->ReturnType->getOpcode() == SPIRV::OpTypeVoid;
2089:   Register Op1Reg = IsSret ? Call->Arguments[1] : Call->Arguments[0];
2090:   Register Op2Reg = IsSret ? Call->Arguments[2] : Call->Arguments[1];
2091:
2092:   SPIRVTypeInst RetType = nullptr;
2093:   if (IsSret) {
2094:     Register SRetReg = Call->Arguments[0];
2095:     SPIRVTypeInst PtrRetType = GR->getSPIRVTypeForVReg(SRetReg);
2096:     RetType = GR->getPointeeType(PtrRetType);
2097:     if (!RetType)
2098:       report_fatal_error("The first parameter must be a pointer");
2099:   } else {
2100:     RetType = Call->ReturnType;
2101:   }
2102:
2103:   if (!RetType || RetType->getOpcode() != SPIRV::OpTypeStruct)
2104:     report_fatal_error("Expected struct type result for the extended "
2105:                        "multiplication builtins");
2106:   if (RetType->getNumOperands() != 3)
2107:     report_fatal_error("Expected struct with exactly two members for the "
2108:                        "extended multiplication builtins");
2109:   SPIRVTypeInst Member0Type =
2110:       GR->getSPIRVTypeForVReg(RetType->getOperand(1).getReg());
2111:   SPIRVTypeInst Member1Type =
2112:       GR->getSPIRVTypeForVReg(RetType->getOperand(2).getReg());
2113:   if (!Member0Type || !Member1Type || Member0Type != Member1Type)
2114:     report_fatal_error("Both struct members must be the same type");
2115:
2116:   SPIRVTypeInst OpType1 = GR->getSPIRVTypeForVReg(Op1Reg);
2117:   SPIRVTypeInst OpType2 = GR->getSPIRVTypeForVReg(Op2Reg);
2118:   if (!OpType1 || !OpType2 || OpType1 != OpType2)
2119:     report_fatal_error("Operands must have the same type");
2120:   if (OpType1 != Member0Type)
2121:     report_fatal_error("Operand type must match the struct member type");
2122:
2123:   if (IsSret) {
2124:     buildSRetInst(Opcode, Call->Arguments[0], Op1Reg, Op2Reg, RetType,
2125:                   MIRBuilder, GR);
2126:   } else {
2127:     MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2128:     Register ResReg = Call->ReturnRegister;
2129:     if (const TargetRegisterClass *DstRC = MRI->getRegClassOrNull(Op1Reg)) {
2130:       MRI->setRegClass(ResReg, DstRC);
2131:     }
2132:     GR->assignSPIRVTypeToVReg(RetType, ResReg, MIRBuilder.getMF());
2133:     MIRBuilder.buildInstr(Opcode)
2134:         .addDef(ResReg)
2135:         .addUse(GR->getSPIRVTypeID(RetType))
2136:         .addUse(Op1Reg)
2137:         .addUse(Op2Reg);
2138:   }
2139:   return true;
2140: }
2141:
2142: static bool generateArithmeticInst(const SPIRV::IncomingCall *Call,
2143:                                    MachineIRBuilder &MIRBuilder,
2144:                                    SPIRVGlobalRegistry *GR) {
2145:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2146:   unsigned Opcode =
2147:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2148:
2149:   auto MIB = MIRBuilder.buildInstr(Opcode)
2150:                  .addDef(Call->ReturnRegister)
2151:                  .addUse(GR->getSPIRVTypeID(Call->ReturnType));
2152:   for (Register Arg : Call->Arguments)
2153:     MIB.addUse(Arg);
2154:   return true;
2155: }
2156:
2157: static bool generateGetQueryInst(const SPIRV::IncomingCall *Call,
2158:                                  MachineIRBuilder &MIRBuilder,
2159:                                  SPIRVGlobalRegistry *GR) {
2160:   // Lookup the builtin record.
```
- EN: This range defines or declares important types such as report_fatal_error, type, getSPIRVTypeForVReg, getPointeeType, shaping the data model used by SPIRVBuiltins.cpp.
- CN: 这一段定义或声明了 report_fatal_error、type、getSPIRVTypeForVReg、getPointeeType 等关键类型，构成 SPIRVBuiltins.cpp 使用的数据模型。

### Lines 2161-2280
```cpp
2161:   SPIRV::BuiltIn::BuiltIn Value =
2162:       SPIRV::lookupGetBuiltin(Call->Builtin->Name, Call->Builtin->Set)->Value;
2163:   const bool IsDefaultOne = (Value == SPIRV::BuiltIn::GlobalSize ||
2164:                              Value == SPIRV::BuiltIn::NumWorkgroups ||
2165:                              Value == SPIRV::BuiltIn::WorkgroupSize ||
2166:                              Value == SPIRV::BuiltIn::EnqueuedWorkgroupSize);
2167:   return genWorkgroupQuery(Call, MIRBuilder, GR, Value, IsDefaultOne ? 1 : 0);
2168: }
2169:
2170: static bool generateImageSizeQueryInst(const SPIRV::IncomingCall *Call,
2171:                                        MachineIRBuilder &MIRBuilder,
2172:                                        SPIRVGlobalRegistry *GR) {
2173:   // Lookup the image size query component number in the TableGen records.
2174:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2175:   uint32_t Component =
2176:       SPIRV::lookupImageQueryBuiltin(Builtin->Name, Builtin->Set)->Component;
2177:   // Query result may either be a vector or a scalar. If return type is not a
2178:   // vector, expect only a single size component. Otherwise get the number of
2179:   // expected components.
2180:   unsigned NumExpectedRetComponents =
2181:       GR->getScalarOrVectorComponentCount(Call->ReturnType);
2182:   // Get the actual number of query result/size components.
2183:   SPIRVTypeInst ImgType = GR->getSPIRVTypeForVReg(Call->Arguments[0]);
2184:   unsigned NumActualRetComponents = getNumSizeComponents(ImgType);
2185:   Register QueryResult = Call->ReturnRegister;
2186:   SPIRVTypeInst QueryResultType = Call->ReturnType;
2187:   if (NumExpectedRetComponents != NumActualRetComponents) {
2188:     unsigned Bitwidth = Call->ReturnType->getOpcode() == SPIRV::OpTypeInt
2189:                             ? Call->ReturnType->getOperand(1).getImm()
2190:                             : 32;
2191:     QueryResult = MIRBuilder.getMRI()->createGenericVirtualRegister(
2192:         LLT::fixed_vector(NumActualRetComponents, Bitwidth));
2193:     MIRBuilder.getMRI()->setRegClass(QueryResult, &SPIRV::viIDRegClass);
2194:     SPIRVTypeInst IntTy = GR->getOrCreateSPIRVIntegerType(Bitwidth, MIRBuilder);
2195:     QueryResultType = GR->getOrCreateSPIRVVectorType(
2196:         IntTy, NumActualRetComponents, MIRBuilder, true);
2197:     GR->assignSPIRVTypeToVReg(QueryResultType, QueryResult, MIRBuilder.getMF());
2198:   }
2199:   bool IsDimBuf = ImgType->getOperand(2).getImm() == SPIRV::Dim::DIM_Buffer;
2200:   bool IsMultisampled = ImgType->getOperand(5).getImm() != 0;
2201:   bool UseQuerySize = IsDimBuf || IsMultisampled;
2202:   unsigned Opcode =
2203:       UseQuerySize ? SPIRV::OpImageQuerySize : SPIRV::OpImageQuerySizeLod;
2204:   auto MIB = MIRBuilder.buildInstr(Opcode)
2205:                  .addDef(QueryResult)
2206:                  .addUse(GR->getSPIRVTypeID(QueryResultType))
2207:                  .addUse(Call->Arguments[0]);
2208:   if (!UseQuerySize)
2209:     MIB.addUse(buildConstantIntReg32(0, MIRBuilder, GR)); // Lod id.
2210:   if (NumExpectedRetComponents == NumActualRetComponents)
2211:     return true;
2212:   if (NumExpectedRetComponents == 1) {
2213:     // Only 1 component is expected, build OpCompositeExtract instruction.
2214:     unsigned ExtractedComposite =
2215:         Component == 3 ? NumActualRetComponents - 1 : Component;
2216:     assert(ExtractedComposite < NumActualRetComponents &&
2217:            "Invalid composite index!");
2218:     Register TypeReg = GR->getSPIRVTypeID(Call->ReturnType);
2219:     SPIRVTypeInst NewType = nullptr;
2220:     if (QueryResultType->getOpcode() == SPIRV::OpTypeVector) {
2221:       NewType = GR->getScalarOrVectorComponentType(QueryResultType);
2222:       Register NewTypeReg = GR->getSPIRVTypeID(NewType);
2223:       if (TypeReg != NewTypeReg)
2224:         TypeReg = NewTypeReg;
2225:       else
2226:         NewType = nullptr;
2227:     }
2228:     MIRBuilder.buildInstr(SPIRV::OpCompositeExtract)
2229:         .addDef(Call->ReturnRegister)
2230:         .addUse(TypeReg)
2231:         .addUse(QueryResult)
2232:         .addImm(ExtractedComposite);
2233:     if (NewType)
2234:       updateRegType(Call->ReturnRegister, nullptr, NewType, GR, MIRBuilder,
2235:                     MIRBuilder.getMF().getRegInfo());
2236:   } else {
2237:     // More than 1 component is expected, fill a new vector.
2238:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpVectorShuffle)
2239:                    .addDef(Call->ReturnRegister)
2240:                    .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2241:                    .addUse(QueryResult)
2242:                    .addUse(QueryResult);
2243:     for (unsigned i = 0; i < NumExpectedRetComponents; ++i)
2244:       MIB.addImm(i < NumActualRetComponents ? i : 0xffffffff);
2245:   }
2246:   return true;
2247: }
2248:
2249: static bool generateImageMiscQueryInst(const SPIRV::IncomingCall *Call,
2250:                                        MachineIRBuilder &MIRBuilder,
2251:                                        SPIRVGlobalRegistry *GR) {
2252:   assert(Call->ReturnType->getOpcode() == SPIRV::OpTypeInt &&
2253:          "Image samples query result must be of int type!");
2254:
2255:   // Lookup the instruction opcode in the TableGen records.
2256:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2257:   unsigned Opcode =
2258:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2259:
2260:   Register Image = Call->Arguments[0];
2261:   SPIRV::Dim::Dim ImageDimensionality = static_cast<SPIRV::Dim::Dim>(
2262:       GR->getSPIRVTypeForVReg(Image)->getOperand(2).getImm());
2263:   (void)ImageDimensionality;
2264:
2265:   switch (Opcode) {
2266:   case SPIRV::OpImageQuerySamples:
2267:     assert(ImageDimensionality == SPIRV::Dim::DIM_2D &&
2268:            "Image must be of 2D dimensionality");
2269:     break;
2270:   case SPIRV::OpImageQueryLevels:
2271:     assert((ImageDimensionality == SPIRV::Dim::DIM_1D ||
2272:             ImageDimensionality == SPIRV::Dim::DIM_2D ||
2273:             ImageDimensionality == SPIRV::Dim::DIM_3D ||
2274:             ImageDimensionality == SPIRV::Dim::DIM_Cube) &&
2275:            "Image must be of 1D/2D/3D/Cube dimensionality");
2276:     break;
2277:   }
2278:
2279:   MIRBuilder.buildInstr(Opcode)
2280:       .addDef(Call->ReturnRegister)
```
- EN: This range implements operational logic in helpers such as genWorkgroupQuery, getScalarOrVectorComponentCount, getSPIRVTypeForVReg, getNumSizeComponents, translating backend policy into executable code.
- CN: 这一段实现了 genWorkgroupQuery、getScalarOrVectorComponentCount、getSPIRVTypeForVReg、getNumSizeComponents 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2281-2400
```cpp
2281:       .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2282:       .addUse(Image);
2283:   return true;
2284: }
2285:
2286: // TODO: Move to TableGen.
2287: static SPIRV::SamplerAddressingMode::SamplerAddressingMode
2288: getSamplerAddressingModeFromBitmask(unsigned Bitmask) {
2289:   switch (Bitmask & SPIRV::CLK_ADDRESS_MODE_MASK) {
2290:   case SPIRV::CLK_ADDRESS_CLAMP:
2291:     return SPIRV::SamplerAddressingMode::Clamp;
2292:   case SPIRV::CLK_ADDRESS_CLAMP_TO_EDGE:
2293:     return SPIRV::SamplerAddressingMode::ClampToEdge;
2294:   case SPIRV::CLK_ADDRESS_REPEAT:
2295:     return SPIRV::SamplerAddressingMode::Repeat;
2296:   case SPIRV::CLK_ADDRESS_MIRRORED_REPEAT:
2297:     return SPIRV::SamplerAddressingMode::RepeatMirrored;
2298:   case SPIRV::CLK_ADDRESS_NONE:
2299:     return SPIRV::SamplerAddressingMode::None;
2300:   default:
2301:     report_fatal_error("Unknown CL address mode");
2302:   }
2303: }
2304:
2305: static unsigned getSamplerParamFromBitmask(unsigned Bitmask) {
2306:   return (Bitmask & SPIRV::CLK_NORMALIZED_COORDS_TRUE) ? 1 : 0;
2307: }
2308:
2309: static SPIRV::SamplerFilterMode::SamplerFilterMode
2310: getSamplerFilterModeFromBitmask(unsigned Bitmask) {
2311:   if (Bitmask & SPIRV::CLK_FILTER_LINEAR)
2312:     return SPIRV::SamplerFilterMode::Linear;
2313:   if (Bitmask & SPIRV::CLK_FILTER_NEAREST)
2314:     return SPIRV::SamplerFilterMode::Nearest;
2315:   return SPIRV::SamplerFilterMode::Nearest;
2316: }
2317:
2318: static bool generateReadImageInst(const StringRef DemangledCall,
2319:                                   const SPIRV::IncomingCall *Call,
2320:                                   MachineIRBuilder &MIRBuilder,
2321:                                   SPIRVGlobalRegistry *GR) {
2322:   if (Call->isSpirvOp())
2323:     return buildOpFromWrapper(MIRBuilder, SPIRV::OpImageRead, Call,
2324:                               GR->getSPIRVTypeID(Call->ReturnType));
2325:   Register Image = Call->Arguments[0];
2326:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2327:   bool HasOclSampler = DemangledCall.contains_insensitive("ocl_sampler");
2328:   bool HasMsaa = DemangledCall.contains_insensitive("msaa");
2329:   if (HasOclSampler) {
2330:     Register Sampler = Call->Arguments[1];
2331:
2332:     if (!GR->isScalarOfType(Sampler, SPIRV::OpTypeSampler) &&
2333:         getDefInstrMaybeConstant(Sampler, MRI)->getOperand(1).isCImm()) {
2334:       uint64_t SamplerMask = getIConstVal(Sampler, MRI);
2335:       Sampler = GR->buildConstantSampler(
2336:           Register(), getSamplerAddressingModeFromBitmask(SamplerMask),
2337:           getSamplerParamFromBitmask(SamplerMask),
2338:           getSamplerFilterModeFromBitmask(SamplerMask), MIRBuilder);
2339:     }
2340:     SPIRVTypeInst ImageType = GR->getSPIRVTypeForVReg(Image);
2341:     SPIRVTypeInst SampledImageType =
2342:         GR->getOrCreateOpTypeSampledImage(ImageType, MIRBuilder);
2343:     Register SampledImage = MRI->createVirtualRegister(&SPIRV::iIDRegClass);
2344:
2345:     MIRBuilder.buildInstr(SPIRV::OpSampledImage)
2346:         .addDef(SampledImage)
2347:         .addUse(GR->getSPIRVTypeID(SampledImageType))
2348:         .addUse(Image)
2349:         .addUse(Sampler);
2350:
2351:     Register Lod = GR->buildConstantFP(APFloat::getZero(APFloat::IEEEsingle()),
2352:                                        MIRBuilder);
2353:
2354:     if (Call->ReturnType->getOpcode() != SPIRV::OpTypeVector) {
2355:       SPIRVTypeInst TempType =
2356:           GR->getOrCreateSPIRVVectorType(Call->ReturnType, 4, MIRBuilder, true);
2357:       Register TempRegister =
2358:           MRI->createGenericVirtualRegister(GR->getRegType(TempType));
2359:       MRI->setRegClass(TempRegister, GR->getRegClass(TempType));
2360:       GR->assignSPIRVTypeToVReg(TempType, TempRegister, MIRBuilder.getMF());
2361:       MIRBuilder.buildInstr(SPIRV::OpImageSampleExplicitLod)
2362:           .addDef(TempRegister)
2363:           .addUse(GR->getSPIRVTypeID(TempType))
2364:           .addUse(SampledImage)
2365:           .addUse(Call->Arguments[2]) // Coordinate.
2366:           .addImm(SPIRV::ImageOperand::Lod)
2367:           .addUse(Lod);
2368:       MIRBuilder.buildInstr(SPIRV::OpCompositeExtract)
2369:           .addDef(Call->ReturnRegister)
2370:           .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2371:           .addUse(TempRegister)
2372:           .addImm(0);
2373:     } else {
2374:       MIRBuilder.buildInstr(SPIRV::OpImageSampleExplicitLod)
2375:           .addDef(Call->ReturnRegister)
2376:           .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2377:           .addUse(SampledImage)
2378:           .addUse(Call->Arguments[2]) // Coordinate.
2379:           .addImm(SPIRV::ImageOperand::Lod)
2380:           .addUse(Lod);
2381:     }
2382:   } else if (HasMsaa) {
2383:     MIRBuilder.buildInstr(SPIRV::OpImageRead)
2384:         .addDef(Call->ReturnRegister)
2385:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2386:         .addUse(Image)
2387:         .addUse(Call->Arguments[1]) // Coordinate.
2388:         .addImm(SPIRV::ImageOperand::Sample)
2389:         .addUse(Call->Arguments[2]);
2390:   } else {
2391:     MIRBuilder.buildInstr(SPIRV::OpImageRead)
2392:         .addDef(Call->ReturnRegister)
2393:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2394:         .addUse(Image)
2395:         .addUse(Call->Arguments[1]); // Coordinate.
2396:   }
2397:   return true;
2398: }
2399:
2400: static bool generateWriteImageInst(const SPIRV::IncomingCall *Call,
```
- EN: This range implements operational logic in helpers such as addUse, getSamplerAddressingModeFromBitmask, report_fatal_error, getSamplerParamFromBitmask, translating backend policy into executable code.
- CN: 这一段实现了 addUse、getSamplerAddressingModeFromBitmask、report_fatal_error、getSamplerParamFromBitmask 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2401-2520
```cpp
2401:                                    MachineIRBuilder &MIRBuilder,
2402:                                    SPIRVGlobalRegistry *GR) {
2403:   if (Call->isSpirvOp())
2404:     return buildOpFromWrapper(MIRBuilder, SPIRV::OpImageWrite, Call,
2405:                               Register(0));
2406:   MIRBuilder.buildInstr(SPIRV::OpImageWrite)
2407:       .addUse(Call->Arguments[0])  // Image.
2408:       .addUse(Call->Arguments[1])  // Coordinate.
2409:       .addUse(Call->Arguments[2]); // Texel.
2410:   return true;
2411: }
2412:
2413: static bool generateSampleImageInst(const StringRef DemangledCall,
2414:                                     const SPIRV::IncomingCall *Call,
2415:                                     MachineIRBuilder &MIRBuilder,
2416:                                     SPIRVGlobalRegistry *GR) {
2417:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2418:   if (Call->Builtin->Name.contains_insensitive(
2419:           "__translate_sampler_initializer")) {
2420:     // Build sampler literal.
2421:     uint64_t Bitmask = getIConstVal(Call->Arguments[0], MRI);
2422:     Register Sampler = GR->buildConstantSampler(
2423:         Call->ReturnRegister, getSamplerAddressingModeFromBitmask(Bitmask),
2424:         getSamplerParamFromBitmask(Bitmask),
2425:         getSamplerFilterModeFromBitmask(Bitmask), MIRBuilder);
2426:     return Sampler.isValid();
2427:   } else if (Call->Builtin->Name.contains_insensitive("__spirv_SampledImage")) {
2428:     // Create OpSampledImage.
2429:     Register Image = Call->Arguments[0];
2430:     SPIRVTypeInst ImageType = GR->getSPIRVTypeForVReg(Image);
2431:     SPIRVTypeInst SampledImageType =
2432:         GR->getOrCreateOpTypeSampledImage(ImageType, MIRBuilder);
2433:     Register SampledImage =
2434:         Call->ReturnRegister.isValid()
2435:             ? Call->ReturnRegister
2436:             : MRI->createVirtualRegister(&SPIRV::iIDRegClass);
2437:     MIRBuilder.buildInstr(SPIRV::OpSampledImage)
2438:         .addDef(SampledImage)
2439:         .addUse(GR->getSPIRVTypeID(SampledImageType))
2440:         .addUse(Image)
2441:         .addUse(Call->Arguments[1]); // Sampler.
2442:     return true;
2443:   } else if (Call->Builtin->Name.contains_insensitive(
2444:                  "__spirv_ImageSampleExplicitLod")) {
2445:     // Sample an image using an explicit level of detail.
2446:     std::string ReturnType = DemangledCall.str();
2447:     if (DemangledCall.contains("_R")) {
2448:       ReturnType = ReturnType.substr(ReturnType.find("_R") + 2);
2449:       ReturnType = ReturnType.substr(0, ReturnType.find('('));
2450:     }
2451:     SPIRVTypeInst Type = Call->ReturnType
2452:                              ? Call->ReturnType
2453:                              : SPIRVTypeInst(GR->getOrCreateSPIRVTypeByName(
2454:                                    ReturnType, MIRBuilder, true));
2455:     if (!Type) {
2456:       std::string DiagMsg =
2457:           "Unable to recognize SPIRV type name: " + ReturnType;
2458:       report_fatal_error(DiagMsg.c_str());
2459:     }
2460:     MIRBuilder.buildInstr(SPIRV::OpImageSampleExplicitLod)
2461:         .addDef(Call->ReturnRegister)
2462:         .addUse(GR->getSPIRVTypeID(Type))
2463:         .addUse(Call->Arguments[0]) // Image.
2464:         .addUse(Call->Arguments[1]) // Coordinate.
2465:         .addImm(SPIRV::ImageOperand::Lod)
2466:         .addUse(Call->Arguments[3]);
2467:     return true;
2468:   }
2469:   return false;
2470: }
2471:
2472: static bool generateSelectInst(const SPIRV::IncomingCall *Call,
2473:                                MachineIRBuilder &MIRBuilder) {
2474:   const MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2475:   LLT ResTy = MRI->getType(Call->ReturnRegister);
2476:   LLT CondTy = MRI->getType(Call->Arguments[0]);
2477:   if (!ResTy.isVector() && CondTy.isVector())
2478:     report_fatal_error("OpSelect with a scalar result requires a scalar "
2479:                        "boolean condition");
2480:   MIRBuilder.buildSelect(Call->ReturnRegister, Call->Arguments[0],
2481:                          Call->Arguments[1], Call->Arguments[2]);
2482:   return true;
2483: }
2484:
2485: static bool generateConstructInst(const SPIRV::IncomingCall *Call,
2486:                                   MachineIRBuilder &MIRBuilder,
2487:                                   SPIRVGlobalRegistry *GR) {
2488:   createContinuedInstructions(MIRBuilder, SPIRV::OpCompositeConstruct, 3,
2489:                               SPIRV::OpCompositeConstructContinuedINTEL,
2490:                               Call->Arguments, Call->ReturnRegister,
2491:                               GR->getSPIRVTypeID(Call->ReturnType));
2492:   return true;
2493: }
2494:
2495: static bool generateCoopMatrInst(const SPIRV::IncomingCall *Call,
2496:                                  MachineIRBuilder &MIRBuilder,
2497:                                  SPIRVGlobalRegistry *GR) {
2498:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2499:   unsigned Opcode =
2500:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2501:   bool IsSet = Opcode != SPIRV::OpCooperativeMatrixStoreKHR &&
2502:                Opcode != SPIRV::OpCooperativeMatrixStoreCheckedINTEL &&
2503:                Opcode != SPIRV::OpCooperativeMatrixPrefetchINTEL;
2504:   unsigned ArgSz = Call->Arguments.size();
2505:   unsigned LiteralIdx = 0;
2506:   switch (Opcode) {
2507:   // Memory operand is optional and is literal.
2508:   case SPIRV::OpCooperativeMatrixLoadKHR:
2509:     LiteralIdx = ArgSz > 3 ? 3 : 0;
2510:     break;
2511:   case SPIRV::OpCooperativeMatrixStoreKHR:
2512:     LiteralIdx = ArgSz > 4 ? 4 : 0;
2513:     break;
2514:   case SPIRV::OpCooperativeMatrixLoadCheckedINTEL:
2515:     LiteralIdx = ArgSz > 7 ? 7 : 0;
2516:     break;
2517:   case SPIRV::OpCooperativeMatrixStoreCheckedINTEL:
2518:     LiteralIdx = ArgSz > 8 ? 8 : 0;
2519:     break;
2520:   // Cooperative Matrix Operands operand is optional and is literal.
```
- EN: This range implements operational logic in helpers such as Register, buildInstr, addUse, getMRI, translating backend policy into executable code.
- CN: 这一段实现了 Register、buildInstr、addUse、getMRI 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2521-2640
```cpp
2521:   case SPIRV::OpCooperativeMatrixMulAddKHR:
2522:     LiteralIdx = ArgSz > 3 ? 3 : 0;
2523:     break;
2524:   };
2525:
2526:   SmallVector<uint32_t, 1> ImmArgs;
2527:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2528:   if (Opcode == SPIRV::OpCooperativeMatrixPrefetchINTEL) {
2529:     const uint32_t CacheLevel = getConstFromIntrinsic(Call->Arguments[3], MRI);
2530:     auto MIB = MIRBuilder.buildInstr(SPIRV::OpCooperativeMatrixPrefetchINTEL)
2531:                    .addUse(Call->Arguments[0])  // pointer
2532:                    .addUse(Call->Arguments[1])  // rows
2533:                    .addUse(Call->Arguments[2])  // columns
2534:                    .addImm(CacheLevel)          // cache level
2535:                    .addUse(Call->Arguments[4]); // memory layout
2536:     if (ArgSz > 5)
2537:       MIB.addUse(Call->Arguments[5]); // stride
2538:     if (ArgSz > 6) {
2539:       const uint32_t MemOp = getConstFromIntrinsic(Call->Arguments[6], MRI);
2540:       MIB.addImm(MemOp); // memory operand
2541:     }
2542:     return true;
2543:   }
2544:   if (LiteralIdx > 0)
2545:     ImmArgs.push_back(getConstFromIntrinsic(Call->Arguments[LiteralIdx], MRI));
2546:   Register TypeReg = GR->getSPIRVTypeID(Call->ReturnType);
2547:   if (Opcode == SPIRV::OpCooperativeMatrixLengthKHR) {
2548:     SPIRVTypeInst CoopMatrType = GR->getSPIRVTypeForVReg(Call->Arguments[0]);
2549:     if (!CoopMatrType)
2550:       report_fatal_error("Can't find a register's type definition");
2551:     MIRBuilder.buildInstr(Opcode)
2552:         .addDef(Call->ReturnRegister)
2553:         .addUse(TypeReg)
2554:         .addUse(CoopMatrType->getOperand(0).getReg());
2555:     return true;
2556:   }
2557:   return buildOpFromWrapper(MIRBuilder, Opcode, Call,
2558:                             IsSet ? TypeReg : Register(0), ImmArgs);
2559: }
2560:
2561: static bool generateSpecConstantInst(const SPIRV::IncomingCall *Call,
2562:                                      MachineIRBuilder &MIRBuilder,
2563:                                      SPIRVGlobalRegistry *GR) {
2564:   // Lookup the instruction opcode in the TableGen records.
2565:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2566:   unsigned Opcode =
2567:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2568:   const MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2569:
2570:   switch (Opcode) {
2571:   case SPIRV::OpSpecConstant: {
2572:     // Determine the constant MI.
2573:     Register ConstRegister = Call->Arguments[1];
2574:     const MachineInstr *Const = getDefInstrMaybeConstant(ConstRegister, MRI);
2575:     assert(Const &&
2576:            (Const->getOpcode() == TargetOpcode::G_CONSTANT ||
2577:             Const->getOpcode() == TargetOpcode::G_FCONSTANT) &&
2578:            "Argument should be either an int or floating-point constant");
2579:     // Determine the opcode and built the OpSpec MI.
2580:     const MachineOperand &ConstOperand = Const->getOperand(1);
2581:     if (Call->ReturnType->getOpcode() == SPIRV::OpTypeBool) {
2582:       assert(ConstOperand.isCImm() && "Int constant operand is expected");
2583:       Opcode = ConstOperand.getCImm()->getValue().getZExtValue()
2584:                    ? SPIRV::OpSpecConstantTrue
2585:                    : SPIRV::OpSpecConstantFalse;
2586:     }
2587:     auto MIB = MIRBuilder.buildInstr(Opcode)
2588:                    .addDef(Call->ReturnRegister)
2589:                    .addUse(GR->getSPIRVTypeID(Call->ReturnType));
2590:
2591:     if (Call->ReturnType->getOpcode() != SPIRV::OpTypeBool) {
2592:       if (Const->getOpcode() == TargetOpcode::G_CONSTANT)
2593:         addNumImm(ConstOperand.getCImm()->getValue(), MIB);
2594:       else
2595:         addNumImm(ConstOperand.getFPImm()->getValueAPF().bitcastToAPInt(), MIB);
2596:     }
2597:     // Build the SpecID decoration.
2598:     unsigned SpecId =
2599:         static_cast<unsigned>(getIConstVal(Call->Arguments[0], MRI));
2600:     buildOpDecorate(Call->ReturnRegister, MIRBuilder, SPIRV::Decoration::SpecId,
2601:                     {SpecId});
2602:     return true;
2603:   }
2604:   case SPIRV::OpSpecConstantComposite: {
2605:     createContinuedInstructions(MIRBuilder, Opcode, 3,
2606:                                 SPIRV::OpSpecConstantCompositeContinuedINTEL,
2607:                                 Call->Arguments, Call->ReturnRegister,
2608:                                 GR->getSPIRVTypeID(Call->ReturnType));
2609:     return true;
2610:   }
2611:   default:
2612:     return false;
2613:   }
2614: }
2615:
2616: static bool generateExtendedBitOpsInst(const SPIRV::IncomingCall *Call,
2617:                                        MachineIRBuilder &MIRBuilder,
2618:                                        SPIRVGlobalRegistry *GR) {
2619:   // Lookup the instruction opcode in the TableGen records.
2620:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2621:   unsigned Opcode =
2622:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2623:
2624:   return buildExtendedBitOpsInst(Call, Opcode, MIRBuilder, GR);
2625: }
2626:
2627: static bool generateBindlessImageINTELInst(const SPIRV::IncomingCall *Call,
2628:                                            MachineIRBuilder &MIRBuilder,
2629:                                            SPIRVGlobalRegistry *GR) {
2630:   // Lookup the instruction opcode in the TableGen records.
2631:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2632:   unsigned Opcode =
2633:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2634:
2635:   return buildBindlessImageINTELInst(Call, Opcode, MIRBuilder, GR);
2636: }
2637:
2638: static bool generateBlockingPipesInst(const SPIRV::IncomingCall *Call,
2639:                                       MachineIRBuilder &MIRBuilder,
2640:                                       SPIRVGlobalRegistry *GR) {
```
- EN: This range implements operational logic in helpers such as getMRI, getConstFromIntrinsic, buildInstr, addUse, translating backend policy into executable code.
- CN: 这一段实现了 getMRI、getConstFromIntrinsic、buildInstr、addUse 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2641-2760
```cpp
2641:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2642:   unsigned Opcode =
2643:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2644:   return buildOpFromWrapper(MIRBuilder, Opcode, Call, Register(0));
2645: }
2646:
2647: static bool buildAPFixedPointInst(const SPIRV::IncomingCall *Call,
2648:                                   unsigned Opcode, MachineIRBuilder &MIRBuilder,
2649:                                   SPIRVGlobalRegistry *GR) {
2650:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2651:   SmallVector<uint32_t, 1> ImmArgs;
2652:   Register InputReg = Call->Arguments[0];
2653:   const Type *RetTy = GR->getTypeForSPIRVType(Call->ReturnType);
2654:   bool IsSRet = RetTy->isVoidTy();
2655:
2656:   if (IsSRet) {
2657:     const LLT ValTy = MRI->getType(InputReg);
2658:     Register ActualRetValReg = MRI->createGenericVirtualRegister(ValTy);
2659:     SPIRVTypeInst InstructionType =
2660:         GR->getPointeeType(GR->getSPIRVTypeForVReg(InputReg));
2661:     InputReg = Call->Arguments[1];
2662:     auto InputType = GR->getTypeForSPIRVType(GR->getSPIRVTypeForVReg(InputReg));
2663:     Register PtrInputReg;
2664:     if (InputType->getTypeID() == llvm::Type::TypeID::TypedPointerTyID) {
2665:       LLT InputLLT = MRI->getType(InputReg);
2666:       PtrInputReg = MRI->createGenericVirtualRegister(InputLLT);
2667:       SPIRVTypeInst PtrType =
2668:           GR->getPointeeType(GR->getSPIRVTypeForVReg(InputReg));
2669:       MachineMemOperand *MMO1 = MIRBuilder.getMF().getMachineMemOperand(
2670:           MachinePointerInfo(), MachineMemOperand::MOLoad,
2671:           InputLLT.getSizeInBytes(), Align(4));
2672:       MIRBuilder.buildLoad(PtrInputReg, InputReg, *MMO1);
2673:       MRI->setRegClass(PtrInputReg, &SPIRV::iIDRegClass);
2674:       GR->assignSPIRVTypeToVReg(PtrType, PtrInputReg, MIRBuilder.getMF());
2675:     }
2676:
2677:     for (unsigned index = 2; index < 7; index++) {
2678:       ImmArgs.push_back(getConstFromIntrinsic(Call->Arguments[index], MRI));
2679:     }
2680:
2681:     // Emit the instruction
2682:     auto MIB = MIRBuilder.buildInstr(Opcode)
2683:                    .addDef(ActualRetValReg)
2684:                    .addUse(GR->getSPIRVTypeID(InstructionType));
2685:     if (PtrInputReg)
2686:       MIB.addUse(PtrInputReg);
2687:     else
2688:       MIB.addUse(InputReg);
2689:
2690:     for (uint32_t Imm : ImmArgs)
2691:       MIB.addImm(Imm);
2692:     unsigned Size = ValTy.getSizeInBytes();
2693:     // Store result to the pointer passed in Arg[0]
2694:     MachineMemOperand *MMO = MIRBuilder.getMF().getMachineMemOperand(
2695:         MachinePointerInfo(), MachineMemOperand::MOStore, Size, Align(4));
2696:     MRI->setRegClass(ActualRetValReg, &SPIRV::pIDRegClass);
2697:     MIRBuilder.buildStore(ActualRetValReg, Call->Arguments[0], *MMO);
2698:     return true;
2699:   } else {
2700:     for (unsigned index = 1; index < 6; index++)
2701:       ImmArgs.push_back(getConstFromIntrinsic(Call->Arguments[index], MRI));
2702:
2703:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
2704:                               GR->getSPIRVTypeID(Call->ReturnType), ImmArgs);
2705:   }
2706: }
2707:
2708: static bool generateAPFixedPointInst(const SPIRV::IncomingCall *Call,
2709:                                      MachineIRBuilder &MIRBuilder,
2710:                                      SPIRVGlobalRegistry *GR) {
2711:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2712:   unsigned Opcode =
2713:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2714:
2715:   return buildAPFixedPointInst(Call, Opcode, MIRBuilder, GR);
2716: }
2717:
2718: static bool
2719: generateTernaryBitwiseFunctionINTELInst(const SPIRV::IncomingCall *Call,
2720:                                         MachineIRBuilder &MIRBuilder,
2721:                                         SPIRVGlobalRegistry *GR) {
2722:   // Lookup the instruction opcode in the TableGen records.
2723:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2724:   unsigned Opcode =
2725:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2726:
2727:   return buildTernaryBitwiseFunctionINTELInst(Call, Opcode, MIRBuilder, GR);
2728: }
2729:
2730: static bool generateImageChannelDataTypeInst(const SPIRV::IncomingCall *Call,
2731:                                              MachineIRBuilder &MIRBuilder,
2732:                                              SPIRVGlobalRegistry *GR) {
2733:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2734:   unsigned Opcode =
2735:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2736:
2737:   return buildImageChannelDataTypeInst(Call, Opcode, MIRBuilder, GR);
2738: }
2739:
2740: static bool generate2DBlockIOINTELInst(const SPIRV::IncomingCall *Call,
2741:                                        MachineIRBuilder &MIRBuilder,
2742:                                        SPIRVGlobalRegistry *GR) {
2743:   // Lookup the instruction opcode in the TableGen records.
2744:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2745:   unsigned Opcode =
2746:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2747:
2748:   return build2DBlockIOINTELInst(Call, Opcode, MIRBuilder, GR);
2749: }
2750:
2751: static bool generatePipeInst(const SPIRV::IncomingCall *Call,
2752:                              MachineIRBuilder &MIRBuilder,
2753:                              SPIRVGlobalRegistry *GR) {
2754:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2755:   unsigned Opcode =
2756:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2757:
2758:   unsigned Scope = SPIRV::Scope::Workgroup;
2759:   if (Builtin->Name.contains("sub_group"))
2760:     Scope = SPIRV::Scope::Subgroup;
```
- EN: This range implements operational logic in helpers such as buildOpFromWrapper, getMRI, getTypeForSPIRVType, isVoidTy, translating backend policy into executable code.
- CN: 这一段实现了 buildOpFromWrapper、getMRI、getTypeForSPIRVType、isVoidTy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2761-2880
```cpp
2761:
2762:   return buildPipeInst(Call, Opcode, Scope, MIRBuilder, GR);
2763: }
2764:
2765: static bool generatePredicatedLoadStoreInst(const SPIRV::IncomingCall *Call,
2766:                                             MachineIRBuilder &MIRBuilder,
2767:                                             SPIRVGlobalRegistry *GR) {
2768:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
2769:   unsigned Opcode =
2770:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
2771:
2772:   bool IsSet = Opcode != SPIRV::OpPredicatedStoreINTEL;
2773:   unsigned ArgSz = Call->Arguments.size();
2774:   SmallVector<uint32_t, 1> ImmArgs;
2775:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2776:   // Memory operand is optional and is literal.
2777:   if (ArgSz > 3)
2778:     ImmArgs.push_back(
2779:         getConstFromIntrinsic(Call->Arguments[/*Literal index*/ 3], MRI));
2780:
2781:   Register TypeReg = GR->getSPIRVTypeID(Call->ReturnType);
2782:   return buildOpFromWrapper(MIRBuilder, Opcode, Call,
2783:                             IsSet ? TypeReg : Register(0), ImmArgs);
2784: }
2785:
2786: static bool buildNDRange(const SPIRV::IncomingCall *Call,
2787:                          MachineIRBuilder &MIRBuilder,
2788:                          SPIRVGlobalRegistry *GR) {
2789:   // The OpenCL ndrange_*D functions are overloaded and support 1D, 2D, and 3D
2790:   // variants, accepting 1 to 3 arguments:
2791:   //   (global_work_size)
2792:   //   (global_work_size, local_work_size)
2793:   //   (global_work_offset, global_work_size, local_work_size)
2794:   // Note: When all three arguments are provided, they are reordered compared
2795:   // to the one- or two-argument form.
2796:   //
2797:   // The function may return data through an sret argument at position 0 (with
2798:   // a void function return type). When present, all other argument indices are
2799:   // adjusted accordingly.
2800:   //
2801:   // SPIR-V's OpBuildNDRange requires all three arguments (GlobalWorkSize,
2802:   // LocalWorkSize, GlobalWorkOffset). For 1D kernels, the values are scalars;
2803:   // for 2D/3D kernels, they are arrays of 2 or 3 elements. Missing arguments
2804:   // default to zero.
2805:   //
2806:   // Calculate argument indices based on the number of arguments and presence
2807:   // of sret:
2808:   const unsigned NumCallArgs = Call->Arguments.size();
2809:   const unsigned MaxCallArgs = Call->Builtin->MaxNumArgs;
2810:   const unsigned IncorrectArgIdx = MaxCallArgs + 1;
2811:
2812:   const Type *RetTy = GR->getTypeForSPIRVType(Call->ReturnType);
2813:   bool HasSRetArg = RetTy->isVoidTy();
2814:
2815:   const unsigned SRetArgIdx = HasSRetArg ? 0 : IncorrectArgIdx;
2816:   const unsigned ArgBase = HasSRetArg ? 1 : 0;
2817:   const unsigned MaxNDRangeArgs = 3;
2818:   const unsigned NumNDRangeArgs = NumCallArgs - ArgBase;
2819:
2820:   const unsigned GlobalWorkSizeArgIdx =
2821:       NumNDRangeArgs < MaxNDRangeArgs ? ArgBase : ArgBase + 1;
2822:   const unsigned LocalWorkSizeArgIdx =
2823:       (NumNDRangeArgs == 1)
2824:           ? IncorrectArgIdx
2825:           : (NumNDRangeArgs == MaxNDRangeArgs ? ArgBase + 2 : ArgBase + 1);
2826:   const unsigned GlobalWorkOffsetArgIdx =
2827:       NumNDRangeArgs == MaxNDRangeArgs ? ArgBase : IncorrectArgIdx;
2828:
2829:   // Each nd_range field is an array of <Dimension> integers matching the
2830:   // address model width (32 or 64 bits).
2831:   const unsigned AddressModelBits = GR->getPointerSize();
2832:   assert(AddressModelBits == 64 || AddressModelBits == 32);
2833:
2834:   // The dimension is encoded in the function name as "ndrange_XD" where X is
2835:   // 1, 2, or 3.
2836:   unsigned Dimension = 0;
2837:   Call->Builtin->Name.substr(8, 1).getAsInteger(10, Dimension);
2838:   assert(Dimension <= 3 && Dimension >= 1);
2839:
2840:   // Determine the work size type based on the dimension. For missing arguments,
2841:   // create a zero constant of the appropriate type.
2842:   MachineFunction &MF = MIRBuilder.getMF();
2843:   SPIRVTypeInst SpvFieldTy;
2844:   Register ConstZero;
2845:   if (Dimension == 1) {
2846:     SpvFieldTy = GR->getSPIRVTypeForVReg(Call->Arguments[GlobalWorkSizeArgIdx]);
2847:     assert(SpvFieldTy && SpvFieldTy->getOpcode() == SPIRV::OpTypeInt &&
2848:            "Expected scalar integer type");
2849:
2850:     if (NumNDRangeArgs < MaxNDRangeArgs)
2851:       ConstZero = GR->buildConstantInt(0, MIRBuilder, SpvFieldTy, true);
2852:   } else {
2853:     Type *BaseTy =
2854:         IntegerType::get(MF.getFunction().getContext(), AddressModelBits);
2855:     Type *FieldTy = ArrayType::get(BaseTy, Dimension);
2856:     SpvFieldTy = GR->getOrCreateSPIRVType(
2857:         FieldTy, MIRBuilder, SPIRV::AccessQualifier::ReadOnly, true);
2858:
2859:     if (NumNDRangeArgs < MaxNDRangeArgs) {
2860:       auto InsertIt = MIRBuilder.getInsertPt();
2861:       MachineBasicBlock &MBB = MIRBuilder.getMBB();
2862:       MachineInstr &InsertMI = (InsertIt != MBB.end()) ? *InsertIt : MBB.back();
2863:       const SPIRVSubtarget &ST = cast<SPIRVSubtarget>(MF.getSubtarget());
2864:       ConstZero = GR->getOrCreateConstIntArray(0, Dimension, InsertMI,
2865:                                                SpvFieldTy, *ST.getInstrInfo());
2866:     }
2867:   }
2868:
2869:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2870:
2871:   auto CreateDataRegister = [&](unsigned Idx) -> Register {
2872:     Register Reg = (Idx == IncorrectArgIdx) ? ConstZero : Call->Arguments[Idx];
2873:
2874:     if (GR->getSPIRVTypeForVReg(Reg) == SpvFieldTy) {
2875:       // Already has the correct type.
2876:       return Reg;
2877:     }
2878:
2879:     assert(GR->getSPIRVTypeForVReg(Reg)->getOpcode() == SPIRV::OpTypePointer &&
2880:            "Only pointer types are supported for loading values");
```
- EN: This range implements operational logic in helpers such as buildPipeInst, size, getMRI, getConstFromIntrinsic, translating backend policy into executable code.
- CN: 这一段实现了 buildPipeInst、size、getMRI、getConstFromIntrinsic 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2881-3000
```cpp
2881:
2882:     Register Ptr = Reg;
2883:
2884:     Reg = MRI->createVirtualRegister(&SPIRV::iIDRegClass);
2885:     GR->assignSPIRVTypeToVReg(SpvFieldTy, Reg, MF);
2886:
2887:     MIRBuilder.buildInstr(SPIRV::OpLoad)
2888:         .addDef(Reg)
2889:         .addUse(GR->getSPIRVTypeID(SpvFieldTy))
2890:         .addUse(Ptr);
2891:     return Reg;
2892:   };
2893:
2894:   Register GlobalWorkSize = CreateDataRegister(GlobalWorkSizeArgIdx);
2895:   Register LocalWorkSize = CreateDataRegister(LocalWorkSizeArgIdx);
2896:   Register GlobalWorkOffset = CreateDataRegister(GlobalWorkOffsetArgIdx);
2897:
2898:   if (!HasSRetArg) {
2899:     return MIRBuilder.buildInstr(SPIRV::OpBuildNDRange)
2900:         .addDef(Call->ReturnRegister)
2901:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
2902:         .addUse(GlobalWorkSize)
2903:         .addUse(LocalWorkSize)
2904:         .addUse(GlobalWorkOffset);
2905:   }
2906:
2907:   // When sret is used, store nd_range struct through the pointer in the first
2908:   // argument.
2909:   Register SRetReg = Call->Arguments[SRetArgIdx];
2910:   SPIRVTypeInst SRetPtrType = GR->getSPIRVTypeForVReg(SRetReg);
2911:   SPIRVTypeInst SRetType = GR->getPointeeType(SRetPtrType);
2912:
2913:   Register TmpReg = MRI->createVirtualRegister(&SPIRV::iIDRegClass);
2914:   GR->assignSPIRVTypeToVReg(SRetType, TmpReg, MF);
2915:
2916:   MIRBuilder.buildInstr(SPIRV::OpBuildNDRange)
2917:       .addDef(TmpReg)
2918:       .addUse(GR->getSPIRVTypeID(SRetType))
2919:       .addUse(GlobalWorkSize)
2920:       .addUse(LocalWorkSize)
2921:       .addUse(GlobalWorkOffset);
2922:   return MIRBuilder.buildInstr(SPIRV::OpStore)
2923:       .addUse(Call->Arguments[SRetArgIdx])
2924:       .addUse(TmpReg);
2925: }
2926:
2927: // TODO: maybe move to the global register.
2928: static SPIRVTypeInst
2929: getOrCreateSPIRVDeviceEventPointer(MachineIRBuilder &MIRBuilder,
2930:                                    SPIRVGlobalRegistry *GR) {
2931:   LLVMContext &Context = MIRBuilder.getMF().getFunction().getContext();
2932:   unsigned SC1 = storageClassToAddressSpace(SPIRV::StorageClass::Generic);
2933:   Type *PtrType = PointerType::get(Context, SC1);
2934:   return GR->getOrCreateSPIRVType(PtrType, MIRBuilder,
2935:                                   SPIRV::AccessQualifier::ReadWrite, true);
2936: }
2937:
2938: static bool buildEnqueueKernel(const SPIRV::IncomingCall *Call,
2939:                                MachineIRBuilder &MIRBuilder,
2940:                                SPIRVGlobalRegistry *GR) {
2941:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
2942:   const DataLayout &DL = MIRBuilder.getDataLayout();
2943:   bool IsSpirvOp = Call->isSpirvOp();
2944:   bool HasEvents = Call->Builtin->Name.contains("events") || IsSpirvOp;
2945:   const SPIRVTypeInst Int32Ty = GR->getOrCreateSPIRVIntegerType(32, MIRBuilder);
2946:
2947:   // Make vararg instructions before OpEnqueueKernel.
2948:   // Local sizes arguments: Sizes of block invoke arguments. Clang generates
2949:   // local size operands as an array, so we need to unpack them.
2950:   SmallVector<Register, 16> LocalSizes;
2951:   if (Call->Builtin->Name.contains("_varargs") || IsSpirvOp) {
2952:     const unsigned LocalSizeArrayIdx = HasEvents ? 9 : 6;
2953:     Register GepReg = Call->Arguments[LocalSizeArrayIdx];
2954:     MachineInstr *GepMI = MRI->getUniqueVRegDef(GepReg);
2955:     assert(isSpvIntrinsic(*GepMI, Intrinsic::spv_gep) &&
2956:            GepMI->getOperand(3).isReg());
2957:     Register ArrayReg = GepMI->getOperand(3).getReg();
2958:     MachineInstr *ArrayMI = MRI->getUniqueVRegDef(ArrayReg);
2959:     const Type *LocalSizeTy = getMachineInstrType(ArrayMI);
2960:     assert(LocalSizeTy && "Local size type is expected");
2961:     const uint64_t LocalSizeNum =
2962:         cast<ArrayType>(LocalSizeTy)->getNumElements();
2963:     unsigned SC = storageClassToAddressSpace(SPIRV::StorageClass::Generic);
2964:     const LLT LLType = LLT::pointer(SC, GR->getPointerSize());
2965:     const SPIRVTypeInst PointerSizeTy = GR->getOrCreateSPIRVPointerType(
2966:         Int32Ty, MIRBuilder, SPIRV::StorageClass::Function);
2967:     for (unsigned I = 0; I < LocalSizeNum; ++I) {
2968:       Register Reg = MRI->createVirtualRegister(&SPIRV::pIDRegClass);
2969:       MRI->setType(Reg, LLType);
2970:       GR->assignSPIRVTypeToVReg(PointerSizeTy, Reg, MIRBuilder.getMF());
2971:       auto GEPInst = MIRBuilder.buildIntrinsic(
2972:           Intrinsic::spv_gep, ArrayRef<Register>{Reg}, true, false);
2973:       GEPInst
2974:           .addImm(GepMI->getOperand(2).getImm())            // In bound.
2975:           .addUse(ArrayMI->getOperand(0).getReg())          // Alloca.
2976:           .addUse(buildConstantIntReg32(0, MIRBuilder, GR)) // Indices.
2977:           .addUse(buildConstantIntReg32(I, MIRBuilder, GR));
2978:       LocalSizes.push_back(Reg);
2979:     }
2980:   }
2981:
2982:   // SPIRV OpEnqueueKernel instruction has 10+ arguments.
2983:   auto MIB = MIRBuilder.buildInstr(SPIRV::OpEnqueueKernel)
2984:                  .addDef(Call->ReturnRegister)
2985:                  .addUse(GR->getSPIRVTypeID(Int32Ty));
2986:
2987:   // Copy all arguments before block invoke function pointer.
2988:   const unsigned BlockFIdx = HasEvents ? 6 : 3;
2989:   for (unsigned i = 0; i < BlockFIdx; i++)
2990:     MIB.addUse(Call->Arguments[i]);
2991:
2992:   // If there are no event arguments in the original call, add dummy ones.
2993:   if (!HasEvents) {
2994:     MIB.addUse(buildConstantIntReg32(0, MIRBuilder, GR)); // Dummy num events.
2995:     Register NullPtr = GR->getOrCreateConstNullPtr(
2996:         MIRBuilder, getOrCreateSPIRVDeviceEventPointer(MIRBuilder, GR));
2997:     MIB.addUse(NullPtr); // Dummy wait events.
2998:     MIB.addUse(NullPtr); // Dummy ret event.
2999:   }
3000:
```
- EN: This range implements operational logic in helpers such as createVirtualRegister, assignSPIRVTypeToVReg, buildInstr, addDef, translating backend policy into executable code.
- CN: 这一段实现了 createVirtualRegister、assignSPIRVTypeToVReg、buildInstr、addDef 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3001-3120
```cpp
3001:   MachineInstr *BlockMI = getBlockStructInstr(Call->Arguments[BlockFIdx], MRI);
3002:   assert(BlockMI->getOpcode() == TargetOpcode::G_GLOBAL_VALUE);
3003:   // Invoke: Pointer to invoke function.
3004:   MIB.addGlobalAddress(BlockMI->getOperand(1).getGlobal());
3005:
3006:   Register BlockLiteralReg = Call->Arguments[BlockFIdx + 1];
3007:   // Param: Pointer to block literal.
3008:   MIB.addUse(BlockLiteralReg);
3009:
3010:   Type *PType = const_cast<Type *>(getBlockStructType(BlockLiteralReg, MRI));
3011:   // TODO: these numbers should be obtained from block literal structure.
3012:   // Param Size: Size of block literal structure.
3013:   MIB.addUse(buildConstantIntReg32(DL.getTypeStoreSize(PType), MIRBuilder, GR));
3014:   // Param Aligment: Aligment of block literal structure.
3015:   MIB.addUse(buildConstantIntReg32(DL.getPrefTypeAlign(PType).value(),
3016:                                    MIRBuilder, GR));
3017:
3018:   for (unsigned i = 0; i < LocalSizes.size(); i++)
3019:     MIB.addUse(LocalSizes[i]);
3020:   return true;
3021: }
3022:
3023: static bool generateEnqueueInst(const SPIRV::IncomingCall *Call,
3024:                                 MachineIRBuilder &MIRBuilder,
3025:                                 SPIRVGlobalRegistry *GR) {
3026:   // Lookup the instruction opcode in the TableGen records.
3027:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
3028:   unsigned Opcode =
3029:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
3030:
3031:   switch (Opcode) {
3032:   case SPIRV::OpRetainEvent:
3033:   case SPIRV::OpReleaseEvent:
3034:     return MIRBuilder.buildInstr(Opcode).addUse(Call->Arguments[0]);
3035:   case SPIRV::OpCreateUserEvent:
3036:   case SPIRV::OpGetDefaultQueue:
3037:     return MIRBuilder.buildInstr(Opcode)
3038:         .addDef(Call->ReturnRegister)
3039:         .addUse(GR->getSPIRVTypeID(Call->ReturnType));
3040:   case SPIRV::OpIsValidEvent:
3041:     return MIRBuilder.buildInstr(Opcode)
3042:         .addDef(Call->ReturnRegister)
3043:         .addUse(GR->getSPIRVTypeID(Call->ReturnType))
3044:         .addUse(Call->Arguments[0]);
3045:   case SPIRV::OpSetUserEventStatus:
3046:     return MIRBuilder.buildInstr(Opcode)
3047:         .addUse(Call->Arguments[0])
3048:         .addUse(Call->Arguments[1]);
3049:   case SPIRV::OpCaptureEventProfilingInfo:
3050:     return MIRBuilder.buildInstr(Opcode)
3051:         .addUse(Call->Arguments[0])
3052:         .addUse(Call->Arguments[1])
3053:         .addUse(Call->Arguments[2]);
3054:   case SPIRV::OpBuildNDRange:
3055:     return buildNDRange(Call, MIRBuilder, GR);
3056:   case SPIRV::OpEnqueueKernel:
3057:     return buildEnqueueKernel(Call, MIRBuilder, GR);
3058:   default:
3059:     return false;
3060:   }
3061: }
3062:
3063: static bool generateAsyncCopy(const SPIRV::IncomingCall *Call,
3064:                               MachineIRBuilder &MIRBuilder,
3065:                               SPIRVGlobalRegistry *GR) {
3066:   // Lookup the instruction opcode in the TableGen records.
3067:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
3068:   unsigned Opcode =
3069:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
3070:
3071:   bool IsSet = Opcode == SPIRV::OpGroupAsyncCopy;
3072:   Register TypeReg = GR->getSPIRVTypeID(Call->ReturnType);
3073:   if (Call->isSpirvOp())
3074:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
3075:                               IsSet ? TypeReg : Register(0));
3076:
3077:   auto Scope = buildConstantIntReg32(SPIRV::Scope::Workgroup, MIRBuilder, GR);
3078:
3079:   switch (Opcode) {
3080:   case SPIRV::OpGroupAsyncCopy: {
3081:     SPIRVTypeInst NewType =
3082:         Call->ReturnType->getOpcode() == SPIRV::OpTypeEvent
3083:             ? nullptr
3084:             : GR->getOrCreateSPIRVTypeByName("spirv.Event", MIRBuilder, true);
3085:     Register TypeReg = GR->getSPIRVTypeID(NewType ? NewType : Call->ReturnType);
3086:     unsigned NumArgs = Call->Arguments.size();
3087:     Register EventReg = Call->Arguments[NumArgs - 1];
3088:     bool Res = MIRBuilder.buildInstr(Opcode)
3089:                    .addDef(Call->ReturnRegister)
3090:                    .addUse(TypeReg)
3091:                    .addUse(Scope)
3092:                    .addUse(Call->Arguments[0])
3093:                    .addUse(Call->Arguments[1])
3094:                    .addUse(Call->Arguments[2])
3095:                    .addUse(Call->Arguments.size() > 4
3096:                                ? Call->Arguments[3]
3097:                                : buildConstantIntReg32(1, MIRBuilder, GR))
3098:                    .addUse(EventReg);
3099:     if (NewType)
3100:       updateRegType(Call->ReturnRegister, nullptr, NewType, GR, MIRBuilder,
3101:                     MIRBuilder.getMF().getRegInfo());
3102:     return Res;
3103:   }
3104:   case SPIRV::OpGroupWaitEvents:
3105:     return MIRBuilder.buildInstr(Opcode)
3106:         .addUse(Scope)
3107:         .addUse(Call->Arguments[0])
3108:         .addUse(Call->Arguments[1]);
3109:   default:
3110:     return false;
3111:   }
3112: }
3113:
3114: static bool generateConvertInst(const StringRef DemangledCall,
3115:                                 const SPIRV::IncomingCall *Call,
3116:                                 MachineIRBuilder &MIRBuilder,
3117:                                 SPIRVGlobalRegistry *GR) {
3118:   // Lookup the conversion builtin in the TableGen records.
3119:   const SPIRV::ConvertBuiltin *Builtin =
3120:       SPIRV::lookupConvertBuiltin(Call->Builtin->Name, Call->Builtin->Set);
```
- EN: This range implements operational logic in helpers such as getBlockStructInstr, assert, addGlobalAddress, addUse, translating backend policy into executable code.
- CN: 这一段实现了 getBlockStructInstr、assert、addGlobalAddress、addUse 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3121-3240
```cpp
3121:
3122:   if (!Builtin && Call->isSpirvOp()) {
3123:     const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
3124:     unsigned Opcode =
3125:         SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
3126:     return buildOpFromWrapper(MIRBuilder, Opcode, Call,
3127:                               GR->getSPIRVTypeID(Call->ReturnType));
3128:   }
3129:
3130:   assert(Builtin && "Conversion builtin not found.");
3131:   if (Builtin->IsSaturated)
3132:     buildOpDecorate(Call->ReturnRegister, MIRBuilder,
3133:                     SPIRV::Decoration::SaturatedConversion, {});
3134:
3135:   if (Builtin->IsRounded) {
3136:     bool AnyTypeIsFloat =
3137:         GR->isScalarOrVectorOfType(Call->ReturnRegister, SPIRV::OpTypeFloat) ||
3138:         GR->isScalarOrVectorOfType(Call->Arguments[0], SPIRV::OpTypeFloat);
3139:
3140:     // Rounding mode decorations are only valid for floating point types.
3141:     // Conversion builtins from integer to integer are equivalent to their
3142:     // non-rounded counterparts.
3143:     if (AnyTypeIsFloat) {
3144:       buildOpDecorate(Call->ReturnRegister, MIRBuilder,
3145:                       SPIRV::Decoration::FPRoundingMode,
3146:                       {(unsigned)Builtin->RoundingMode});
3147:     }
3148:   }
3149:
3150:   std::string NeedExtMsg;              // no errors if empty
3151:   bool IsRightComponentsNumber = true; // check if input/output accepts vectors
3152:   unsigned Opcode = SPIRV::OpNop;
3153:   if (GR->isScalarOrVectorOfType(Call->Arguments[0], SPIRV::OpTypeInt)) {
3154:     // Int -> ...
3155:     if (GR->isScalarOrVectorOfType(Call->ReturnRegister, SPIRV::OpTypeInt)) {
3156:       // Int -> Int
3157:       if (Builtin->IsSaturated)
3158:         Opcode = Builtin->IsDestinationSigned ? SPIRV::OpSatConvertUToS
3159:                                               : SPIRV::OpSatConvertSToU;
3160:       else
3161:         Opcode = Builtin->IsDestinationSigned ? SPIRV::OpUConvert
3162:                                               : SPIRV::OpSConvert;
3163:     } else if (GR->isScalarOrVectorOfType(Call->ReturnRegister,
3164:                                           SPIRV::OpTypeFloat)) {
3165:       // Int -> Float
3166:       if (Builtin->IsBfloat16) {
3167:         const auto *ST = static_cast<const SPIRVSubtarget *>(
3168:             &MIRBuilder.getMF().getSubtarget());
3169:         if (!ST->canUseExtension(
3170:                 SPIRV::Extension::SPV_INTEL_bfloat16_conversion))
3171:           NeedExtMsg = "SPV_INTEL_bfloat16_conversion";
3172:         IsRightComponentsNumber =
3173:             GR->getScalarOrVectorComponentCount(Call->Arguments[0]) ==
3174:             GR->getScalarOrVectorComponentCount(Call->ReturnRegister);
3175:         Opcode = SPIRV::OpConvertBF16ToFINTEL;
3176:       } else {
3177:         bool IsSourceSigned =
3178:             DemangledCall[DemangledCall.find_first_of('(') + 1] != 'u';
3179:         Opcode = IsSourceSigned ? SPIRV::OpConvertSToF : SPIRV::OpConvertUToF;
3180:       }
3181:     }
3182:   } else if (GR->isScalarOrVectorOfType(Call->Arguments[0],
3183:                                         SPIRV::OpTypeFloat)) {
3184:     // Float -> ...
3185:     if (GR->isScalarOrVectorOfType(Call->ReturnRegister, SPIRV::OpTypeInt)) {
3186:       // Float -> Int
3187:       if (Builtin->IsBfloat16) {
3188:         const auto *ST = static_cast<const SPIRVSubtarget *>(
3189:             &MIRBuilder.getMF().getSubtarget());
3190:         if (!ST->canUseExtension(
3191:                 SPIRV::Extension::SPV_INTEL_bfloat16_conversion))
3192:           NeedExtMsg = "SPV_INTEL_bfloat16_conversion";
3193:         IsRightComponentsNumber =
3194:             GR->getScalarOrVectorComponentCount(Call->Arguments[0]) ==
3195:             GR->getScalarOrVectorComponentCount(Call->ReturnRegister);
3196:         Opcode = SPIRV::OpConvertFToBF16INTEL;
3197:       } else {
3198:         Opcode = Builtin->IsDestinationSigned ? SPIRV::OpConvertFToS
3199:                                               : SPIRV::OpConvertFToU;
3200:       }
3201:     } else if (GR->isScalarOrVectorOfType(Call->ReturnRegister,
3202:                                           SPIRV::OpTypeFloat)) {
3203:       if (Builtin->IsTF32) {
3204:         const auto *ST = static_cast<const SPIRVSubtarget *>(
3205:             &MIRBuilder.getMF().getSubtarget());
3206:         if (!ST->canUseExtension(
3207:                 SPIRV::Extension::SPV_INTEL_tensor_float32_conversion))
3208:           NeedExtMsg = "SPV_INTEL_tensor_float32_conversion";
3209:         IsRightComponentsNumber =
3210:             GR->getScalarOrVectorComponentCount(Call->Arguments[0]) ==
3211:             GR->getScalarOrVectorComponentCount(Call->ReturnRegister);
3212:         Opcode = SPIRV::OpRoundFToTF32INTEL;
3213:       } else {
3214:         // Float -> Float
3215:         Opcode = SPIRV::OpFConvert;
3216:       }
3217:     }
3218:   }
3219:
3220:   if (!NeedExtMsg.empty()) {
3221:     std::string DiagMsg = std::string(Builtin->Name) +
3222:                           ": the builtin requires the following SPIR-V "
3223:                           "extension: " +
3224:                           NeedExtMsg;
3225:     report_fatal_error(DiagMsg.c_str(), false);
3226:   }
3227:   if (!IsRightComponentsNumber) {
3228:     std::string DiagMsg =
3229:         std::string(Builtin->Name) +
3230:         ": result and argument must have the same number of components";
3231:     report_fatal_error(DiagMsg.c_str(), false);
3232:   }
3233:   assert(Opcode != SPIRV::OpNop &&
3234:          "Conversion between the types not implemented!");
3235:
3236:   MIRBuilder.buildInstr(Opcode)
3237:       .addDef(Call->ReturnRegister)
3238:       .addUse(GR->getSPIRVTypeID(Call->ReturnType))
3239:       .addUse(Call->Arguments[0]);
3240:   return true;
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeID, assert, isScalarOrVectorOfType, getMF, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeID、assert、isScalarOrVectorOfType、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3241-3360
```cpp
3241: }
3242:
3243: static bool generateVectorLoadStoreInst(const SPIRV::IncomingCall *Call,
3244:                                         MachineIRBuilder &MIRBuilder,
3245:                                         SPIRVGlobalRegistry *GR) {
3246:   // Lookup the vector load/store builtin in the TableGen records.
3247:   const SPIRV::VectorLoadStoreBuiltin *Builtin =
3248:       SPIRV::lookupVectorLoadStoreBuiltin(Call->Builtin->Name,
3249:                                           Call->Builtin->Set);
3250:   // Build extended instruction.
3251:   auto MIB =
3252:       MIRBuilder.buildInstr(SPIRV::OpExtInst)
3253:           .addDef(Call->ReturnRegister)
3254:           .addUse(GR->getSPIRVTypeID(Call->ReturnType))
3255:           .addImm(static_cast<uint32_t>(SPIRV::InstructionSet::OpenCL_std))
3256:           .addImm(Builtin->Number);
3257:   for (auto Argument : Call->Arguments)
3258:     MIB.addUse(Argument);
3259:   if (Builtin->Name.contains("load") && Builtin->ElementCount > 1)
3260:     MIB.addImm(Builtin->ElementCount);
3261:
3262:   // Rounding mode should be passed as a last argument in the MI for builtins
3263:   // like "vstorea_halfn_r".
3264:   if (Builtin->IsRounded)
3265:     MIB.addImm(static_cast<uint32_t>(Builtin->RoundingMode));
3266:   return true;
3267: }
3268:
3269: static bool generateAFPInst(const SPIRV::IncomingCall *Call,
3270:                             MachineIRBuilder &MIRBuilder,
3271:                             SPIRVGlobalRegistry *GR) {
3272:   const auto *Builtin = Call->Builtin;
3273:   auto *MRI = MIRBuilder.getMRI();
3274:   unsigned Opcode =
3275:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
3276:   const Type *RetTy = GR->getTypeForSPIRVType(Call->ReturnType);
3277:   bool IsVoid = RetTy->isVoidTy();
3278:   auto MIB = MIRBuilder.buildInstr(Opcode);
3279:   Register DestReg;
3280:   if (IsVoid) {
3281:     LLT PtrTy = MRI->getType(Call->Arguments[0]);
3282:     DestReg = MRI->createGenericVirtualRegister(PtrTy);
3283:     MRI->setRegClass(DestReg, &SPIRV::pIDRegClass);
3284:     SPIRVTypeInst PointeeTy =
3285:         GR->getPointeeType(GR->getSPIRVTypeForVReg(Call->Arguments[0]));
3286:     MIB.addDef(DestReg);
3287:     MIB.addUse(GR->getSPIRVTypeID(PointeeTy));
3288:   } else {
3289:     MIB.addDef(Call->ReturnRegister);
3290:     MIB.addUse(GR->getSPIRVTypeID(Call->ReturnType));
3291:   }
3292:   for (unsigned i = IsVoid ? 1 : 0; i < Call->Arguments.size(); ++i) {
3293:     Register Arg = Call->Arguments[i];
3294:     MachineInstr *DefMI = MRI->getUniqueVRegDef(Arg);
3295:     if (DefMI->getOpcode() == TargetOpcode::G_CONSTANT &&
3296:         DefMI->getOperand(1).isCImm()) {
3297:       MIB.addImm(getConstFromIntrinsic(Arg, MRI));
3298:     } else {
3299:       MIB.addUse(Arg);
3300:     }
3301:   }
3302:   if (IsVoid) {
3303:     LLT PtrTy = MRI->getType(Call->Arguments[0]);
3304:     MachineMemOperand *MMO = MIRBuilder.getMF().getMachineMemOperand(
3305:         MachinePointerInfo(), MachineMemOperand::MOStore,
3306:         PtrTy.getSizeInBytes(), Align(4));
3307:     MIRBuilder.buildStore(DestReg, Call->Arguments[0], *MMO);
3308:   }
3309:   return true;
3310: }
3311:
3312: static bool generateLoadStoreInst(const SPIRV::IncomingCall *Call,
3313:                                   MachineIRBuilder &MIRBuilder,
3314:                                   SPIRVGlobalRegistry *GR) {
3315:   // Lookup the instruction opcode in the TableGen records.
3316:   const SPIRV::DemangledBuiltin *Builtin = Call->Builtin;
3317:   unsigned Opcode =
3318:       SPIRV::lookupNativeBuiltin(Builtin->Name, Builtin->Set)->Opcode;
3319:   bool IsLoad = Opcode == SPIRV::OpLoad;
3320:   // Build the instruction.
3321:   auto MIB = MIRBuilder.buildInstr(Opcode);
3322:   if (IsLoad) {
3323:     MIB.addDef(Call->ReturnRegister);
3324:     MIB.addUse(GR->getSPIRVTypeID(Call->ReturnType));
3325:   }
3326:   // Add a pointer to the value to load/store.
3327:   MIB.addUse(Call->Arguments[0]);
3328:   MachineRegisterInfo *MRI = MIRBuilder.getMRI();
3329:   // Add a value to store.
3330:   if (!IsLoad)
3331:     MIB.addUse(Call->Arguments[1]);
3332:   // Add optional memory attributes and an alignment.
3333:   unsigned NumArgs = Call->Arguments.size();
3334:   if ((IsLoad && NumArgs >= 2) || NumArgs >= 3)
3335:     MIB.addImm(getConstFromIntrinsic(Call->Arguments[IsLoad ? 1 : 2], MRI));
3336:   if ((IsLoad && NumArgs >= 3) || NumArgs >= 4)
3337:     MIB.addImm(getConstFromIntrinsic(Call->Arguments[IsLoad ? 2 : 3], MRI));
3338:   return true;
3339: }
3340:
3341: namespace SPIRV {
3342: // Try to find a builtin function attributes by a demangled function name and
3343: // return a tuple <builtin group, op code, ext instruction number>, or a special
3344: // tuple value <-1, 0, 0> if the builtin function is not found.
3345: // Not all builtin functions are supported, only those with a ready-to-use op
3346: // code or instruction number defined in TableGen.
3347: // TODO: consider a major rework of mapping demangled calls into a builtin
3348: // functions to unify search and decrease number of individual cases.
3349: std::tuple<int, unsigned, unsigned>
3350: mapBuiltinToOpcode(const StringRef DemangledCall,
3351:                    SPIRV::InstructionSet::InstructionSet Set) {
3352:   Register Reg;
3353:   SmallVector<Register> Args;
3354:   std::unique_ptr<const IncomingCall> Call =
3355:       lookupBuiltin(DemangledCall, Set, Reg, nullptr, Args);
3356:   if (!Call)
3357:     return std::make_tuple(-1, 0, 0);
3358:
3359:   switch (Call->Builtin->Group) {
3360:   case SPIRV::Relational:
```
- EN: This range implements operational logic in helpers such as buildInstr, addDef, addUse, addImm, translating backend policy into executable code.
- CN: 这一段实现了 buildInstr、addDef、addUse、addImm 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3361-3480
```cpp
3361:   case SPIRV::Atomic:
3362:   case SPIRV::Barrier:
3363:   case SPIRV::CastToPtr:
3364:   case SPIRV::ImageMiscQuery:
3365:   case SPIRV::SpecConstant:
3366:   case SPIRV::Enqueue:
3367:   case SPIRV::AsyncCopy:
3368:   case SPIRV::LoadStore:
3369:   case SPIRV::CoopMatr:
3370:   case SPIRV::Arithmetic:
3371:     if (const auto *R =
3372:             SPIRV::lookupNativeBuiltin(Call->Builtin->Name, Call->Builtin->Set))
3373:       return std::make_tuple(Call->Builtin->Group, R->Opcode, 0);
3374:     break;
3375:   case SPIRV::Extended:
3376:     if (const auto *R = SPIRV::lookupExtendedBuiltin(Call->Builtin->Name,
3377:                                                      Call->Builtin->Set))
3378:       return std::make_tuple(Call->Builtin->Group, 0, R->Number);
3379:     break;
3380:   case SPIRV::VectorLoadStore:
3381:     if (const auto *R = SPIRV::lookupVectorLoadStoreBuiltin(Call->Builtin->Name,
3382:                                                             Call->Builtin->Set))
3383:       return std::make_tuple(SPIRV::Extended, 0, R->Number);
3384:     break;
3385:   case SPIRV::Group:
3386:     if (const auto *R = SPIRV::lookupGroupBuiltin(Call->Builtin->Name))
3387:       return std::make_tuple(Call->Builtin->Group, R->Opcode, 0);
3388:     break;
3389:   case SPIRV::AtomicFloating:
3390:     if (const auto *R = SPIRV::lookupAtomicFloatingBuiltin(Call->Builtin->Name))
3391:       return std::make_tuple(Call->Builtin->Group, R->Opcode, 0);
3392:     break;
3393:   case SPIRV::IntelSubgroups:
3394:     if (const auto *R = SPIRV::lookupIntelSubgroupsBuiltin(Call->Builtin->Name))
3395:       return std::make_tuple(Call->Builtin->Group, R->Opcode, 0);
3396:     break;
3397:   case SPIRV::GroupUniform:
3398:     if (const auto *R = SPIRV::lookupGroupUniformBuiltin(Call->Builtin->Name))
3399:       return std::make_tuple(Call->Builtin->Group, R->Opcode, 0);
3400:     break;
3401:   case SPIRV::IntegerDot:
3402:     if (const auto *R =
3403:             SPIRV::lookupIntegerDotProductBuiltin(Call->Builtin->Name))
3404:       return std::make_tuple(Call->Builtin->Group, R->Opcode, 0);
3405:     break;
3406:   case SPIRV::WriteImage:
3407:     return std::make_tuple(Call->Builtin->Group, SPIRV::OpImageWrite, 0);
3408:   case SPIRV::Select:
3409:     return std::make_tuple(Call->Builtin->Group, TargetOpcode::G_SELECT, 0);
3410:   case SPIRV::Construct:
3411:     return std::make_tuple(Call->Builtin->Group, SPIRV::OpCompositeConstruct,
3412:                            0);
3413:   case SPIRV::KernelClock:
3414:     return std::make_tuple(Call->Builtin->Group, SPIRV::OpReadClockKHR, 0);
3415:   default:
3416:     return std::make_tuple(-1, 0, 0);
3417:   }
3418:   return std::make_tuple(-1, 0, 0);
3419: }
3420:
3421: std::optional<bool> lowerBuiltin(const StringRef DemangledCall,
3422:                                  SPIRV::InstructionSet::InstructionSet Set,
3423:                                  MachineIRBuilder &MIRBuilder,
3424:                                  const Register OrigRet, const Type *OrigRetTy,
3425:                                  const SmallVectorImpl<Register> &Args,
3426:                                  SPIRVGlobalRegistry *GR, const CallBase &CB) {
3427:   LLVM_DEBUG(dbgs() << "Lowering builtin call: " << DemangledCall << "\n");
3428:
3429:   // Lookup the builtin in the TableGen records.
3430:   SPIRVTypeInst SpvType = GR->getSPIRVTypeForVReg(OrigRet);
3431:   assert(SpvType && "Inconsistent return register: expected valid type info");
3432:   std::unique_ptr<const IncomingCall> Call =
3433:       lookupBuiltin(DemangledCall, Set, OrigRet, SpvType, Args);
3434:
3435:   if (!Call) {
3436:     LLVM_DEBUG(dbgs() << "Builtin record was not found!\n");
3437:     return std::nullopt;
3438:   }
3439:
3440:   // Check if the provided args meet the builtin requirements. If not, treat
3441:   // the call as a regular function call rather than crashing.
3442:   if (Args.size() < Call->Builtin->MinNumArgs) {
3443:     LLVM_DEBUG(dbgs() << "Too few arguments for builtin " << DemangledCall
3444:                       << ": expected at least " << Call->Builtin->MinNumArgs
3445:                       << ", got " << Args.size()
3446:                       << "; treating as a normal function\n");
3447:     return std::nullopt;
3448:   }
3449:   if (Call->Builtin->MaxNumArgs && Args.size() > Call->Builtin->MaxNumArgs) {
3450:     LLVM_DEBUG(dbgs() << "Too many arguments for builtin " << DemangledCall
3451:                       << ": expected at most " << Call->Builtin->MaxNumArgs
3452:                       << ", got " << Args.size()
3453:                       << "; treating as a normal function\n");
3454:     return std::nullopt;
3455:   }
3456:
3457:   // Match the builtin with implementation based on the grouping.
3458:   switch (Call->Builtin->Group) {
3459:   case SPIRV::Extended:
3460:     return generateExtInst(Call.get(), MIRBuilder, GR, CB);
3461:   case SPIRV::Relational:
3462:     return generateRelationalInst(Call.get(), MIRBuilder, GR);
3463:   case SPIRV::Group:
3464:     return generateGroupInst(Call.get(), MIRBuilder, GR);
3465:   case SPIRV::Variable:
3466:     return generateBuiltinVar(Call.get(), MIRBuilder, GR);
3467:   case SPIRV::Atomic:
3468:     return generateAtomicInst(Call.get(), MIRBuilder, GR);
3469:   case SPIRV::AtomicFloating:
3470:     return generateAtomicFloatingInst(Call.get(), MIRBuilder, GR);
3471:   case SPIRV::Barrier:
3472:     return generateBarrierInst(Call.get(), MIRBuilder, GR);
3473:   case SPIRV::CastToPtr:
3474:     return generateCastToPtrInst(Call.get(), MIRBuilder, GR);
3475:   case SPIRV::Dot:
3476:   case SPIRV::IntegerDot:
3477:     return generateDotOrFMulInst(DemangledCall, Call.get(), MIRBuilder, GR);
3478:   case SPIRV::Wave:
3479:     return generateWaveInst(Call.get(), MIRBuilder, GR);
3480:   case SPIRV::ICarryBorrow:
```
- EN: This range implements operational logic in helpers such as SPIRV::lookupNativeBuiltin, std::make_tuple, SPIRV::lookupIntegerDotProductBuiltin, LLVM_DEBUG, translating backend policy into executable code.
- CN: 这一段实现了 SPIRV::lookupNativeBuiltin、std::make_tuple、SPIRV::lookupIntegerDotProductBuiltin、LLVM_DEBUG 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3481-3600
```cpp
3481:     return generateICarryBorrowInst(Call.get(), MIRBuilder, GR);
3482:   case SPIRV::MulExtended:
3483:     return generateMulExtendedInst(Call.get(), MIRBuilder, GR);
3484:   case SPIRV::Arithmetic:
3485:     return generateArithmeticInst(Call.get(), MIRBuilder, GR);
3486:   case SPIRV::GetQuery:
3487:     return generateGetQueryInst(Call.get(), MIRBuilder, GR);
3488:   case SPIRV::ImageSizeQuery:
3489:     return generateImageSizeQueryInst(Call.get(), MIRBuilder, GR);
3490:   case SPIRV::ImageMiscQuery:
3491:     return generateImageMiscQueryInst(Call.get(), MIRBuilder, GR);
3492:   case SPIRV::ReadImage:
3493:     return generateReadImageInst(DemangledCall, Call.get(), MIRBuilder, GR);
3494:   case SPIRV::WriteImage:
3495:     return generateWriteImageInst(Call.get(), MIRBuilder, GR);
3496:   case SPIRV::SampleImage:
3497:     return generateSampleImageInst(DemangledCall, Call.get(), MIRBuilder, GR);
3498:   case SPIRV::Select:
3499:     return generateSelectInst(Call.get(), MIRBuilder);
3500:   case SPIRV::Construct:
3501:     return generateConstructInst(Call.get(), MIRBuilder, GR);
3502:   case SPIRV::SpecConstant:
3503:     return generateSpecConstantInst(Call.get(), MIRBuilder, GR);
3504:   case SPIRV::Enqueue:
3505:     return generateEnqueueInst(Call.get(), MIRBuilder, GR);
3506:   case SPIRV::AsyncCopy:
3507:     return generateAsyncCopy(Call.get(), MIRBuilder, GR);
3508:   case SPIRV::Convert:
3509:     return generateConvertInst(DemangledCall, Call.get(), MIRBuilder, GR);
3510:   case SPIRV::VectorLoadStore:
3511:     return generateVectorLoadStoreInst(Call.get(), MIRBuilder, GR);
3512:   case SPIRV::LoadStore:
3513:     return generateLoadStoreInst(Call.get(), MIRBuilder, GR);
3514:   case SPIRV::IntelSubgroups:
3515:     return generateIntelSubgroupsInst(Call.get(), MIRBuilder, GR);
3516:   case SPIRV::GroupUniform:
3517:     return generateGroupUniformInst(Call.get(), MIRBuilder, GR);
3518:   case SPIRV::KernelClock:
3519:     return generateKernelClockInst(Call.get(), MIRBuilder, GR);
3520:   case SPIRV::CoopMatr:
3521:     return generateCoopMatrInst(Call.get(), MIRBuilder, GR);
3522:   case SPIRV::ExtendedBitOps:
3523:     return generateExtendedBitOpsInst(Call.get(), MIRBuilder, GR);
3524:   case SPIRV::BindlessINTEL:
3525:     return generateBindlessImageINTELInst(Call.get(), MIRBuilder, GR);
3526:   case SPIRV::TernaryBitwiseINTEL:
3527:     return generateTernaryBitwiseFunctionINTELInst(Call.get(), MIRBuilder, GR);
3528:   case SPIRV::Block2DLoadStore:
3529:     return generate2DBlockIOINTELInst(Call.get(), MIRBuilder, GR);
3530:   case SPIRV::Pipe:
3531:     return generatePipeInst(Call.get(), MIRBuilder, GR);
3532:   case SPIRV::PredicatedLoadStore:
3533:     return generatePredicatedLoadStoreInst(Call.get(), MIRBuilder, GR);
3534:   case SPIRV::BlockingPipes:
3535:     return generateBlockingPipesInst(Call.get(), MIRBuilder, GR);
3536:   case SPIRV::ArbitraryPrecisionFixedPoint:
3537:     return generateAPFixedPointInst(Call.get(), MIRBuilder, GR);
3538:   case SPIRV::ImageChannelDataTypes:
3539:     return generateImageChannelDataTypeInst(Call.get(), MIRBuilder, GR);
3540:   case SPIRV::ArbitraryFloatingPoint:
3541:     return generateAFPInst(Call.get(), MIRBuilder, GR);
3542:   }
3543:   return false;
3544: }
3545:
3546: Type *parseBuiltinCallArgumentType(StringRef TypeStr, LLVMContext &Ctx) {
3547:   // Parse strings representing OpenCL builtin types.
3548:   if (hasBuiltinTypePrefix(TypeStr)) {
3549:     // OpenCL builtin types in demangled call strings have the following format:
3550:     // e.g. ocl_image2d_ro
3551:     [[maybe_unused]] bool IsOCLBuiltinType = TypeStr.consume_front("ocl_");
3552:     assert(IsOCLBuiltinType && "Invalid OpenCL builtin prefix");
3553:
3554:     // Check if this is pointer to a builtin type and not just pointer
3555:     // representing a builtin type. In case it is a pointer to builtin type,
3556:     // this will require additional handling in the method calling
3557:     // parseBuiltinCallArgumentBaseType(...) as this function only retrieves the
3558:     // base types.
3559:     if (TypeStr.ends_with("*"))
3560:       TypeStr = TypeStr.slice(0, TypeStr.find_first_of(" *"));
3561:
3562:     return parseBuiltinTypeNameToTargetExtType("opencl." + TypeStr.str() + "_t",
3563:                                                Ctx);
3564:   }
3565:
3566:   // Parse type name in either "typeN" or "type vector[N]" format, where
3567:   // N is the number of elements of the vector.
3568:   Type *BaseType;
3569:   unsigned VecElts = 0;
3570:
3571:   BaseType = parseBasicTypeName(TypeStr, Ctx);
3572:   if (!BaseType)
3573:     // Unable to recognize SPIRV type name.
3574:     return nullptr;
3575:
3576:   // Handle "typeN*" or "type vector[N]*".
3577:   TypeStr.consume_back("*");
3578:
3579:   if (TypeStr.consume_front(" vector["))
3580:     TypeStr = TypeStr.substr(0, TypeStr.find(']'));
3581:
3582:   TypeStr.getAsInteger(10, VecElts);
3583:   if (VecElts > 0)
3584:     BaseType = VectorType::get(
3585:         BaseType->isVoidTy() ? Type::getInt8Ty(Ctx) : BaseType, VecElts, false);
3586:
3587:   return BaseType;
3588: }
3589:
3590: bool parseBuiltinTypeStr(SmallVector<StringRef, 10> &BuiltinArgsTypeStrs,
3591:                          const StringRef DemangledCall, LLVMContext &Ctx) {
3592:   auto Pos1 = DemangledCall.find('(');
3593:   if (Pos1 == StringRef::npos)
3594:     return false;
3595:   auto Pos2 = DemangledCall.find(')');
3596:   if (Pos2 == StringRef::npos || Pos1 > Pos2)
3597:     return false;
3598:   DemangledCall.slice(Pos1 + 1, Pos2)
3599:       .split(BuiltinArgsTypeStrs, ',', -1, false);
3600:   return true;
```
- EN: This range implements operational logic in helpers such as generateICarryBorrowInst, generateMulExtendedInst, generateArithmeticInst, generateGetQueryInst, translating backend policy into executable code.
- CN: 这一段实现了 generateICarryBorrowInst、generateMulExtendedInst、generateArithmeticInst、generateGetQueryInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3601-3720
```cpp
3601: }
3602:
3603: Type *parseBuiltinCallArgumentBaseType(const StringRef DemangledCall,
3604:                                        unsigned ArgIdx, LLVMContext &Ctx) {
3605:   SmallVector<StringRef, 10> BuiltinArgsTypeStrs;
3606:   parseBuiltinTypeStr(BuiltinArgsTypeStrs, DemangledCall, Ctx);
3607:   if (ArgIdx >= BuiltinArgsTypeStrs.size())
3608:     return nullptr;
3609:   StringRef TypeStr = BuiltinArgsTypeStrs[ArgIdx].trim();
3610:   return parseBuiltinCallArgumentType(TypeStr, Ctx);
3611: }
3612:
3613: struct BuiltinType {
3614:   StringRef Name;
3615:   uint32_t Opcode;
3616: };
3617:
3618: #define GET_BuiltinTypes_DECL
3619: #define GET_BuiltinTypes_IMPL
3620:
3621: struct OpenCLType {
3622:   StringRef Name;
3623:   StringRef SpirvTypeLiteral;
3624: };
3625:
3626: #define GET_OpenCLTypes_DECL
3627: #define GET_OpenCLTypes_IMPL
3628:
3629: #include "SPIRVGenTables.inc"
3630: } // namespace SPIRV
3631:
3632: //===----------------------------------------------------------------------===//
3633: // Misc functions for parsing builtin types.
3634: //===----------------------------------------------------------------------===//
3635:
3636: static Type *parseTypeString(const StringRef Name, LLVMContext &Context) {
3637:   if (Name.starts_with("void"))
3638:     return Type::getVoidTy(Context);
3639:   else if (Name.starts_with("int") || Name.starts_with("uint"))
3640:     return Type::getInt32Ty(Context);
3641:   else if (Name.starts_with("float"))
3642:     return Type::getFloatTy(Context);
3643:   else if (Name.starts_with("half"))
3644:     return Type::getHalfTy(Context);
3645:   report_fatal_error("Unable to recognize type!");
3646: }
3647:
3648: //===----------------------------------------------------------------------===//
3649: // Implementation functions for builtin types.
3650: //===----------------------------------------------------------------------===//
3651:
3652: static SPIRVTypeInst
3653: getNonParameterizedType(const TargetExtType *ExtensionType,
3654:                         const SPIRV::BuiltinType *TypeRecord,
3655:                         MachineIRBuilder &MIRBuilder, SPIRVGlobalRegistry *GR) {
3656:   unsigned Opcode = TypeRecord->Opcode;
3657:   // Create or get an existing type from GlobalRegistry.
3658:   return GR->getOrCreateOpTypeByOpcode(ExtensionType, MIRBuilder, Opcode);
3659: }
3660:
3661: static SPIRVTypeInst getSamplerType(MachineIRBuilder &MIRBuilder,
3662:                                     SPIRVGlobalRegistry *GR) {
3663:   // Create or get an existing type from GlobalRegistry.
3664:   return GR->getOrCreateOpTypeSampler(MIRBuilder);
3665: }
3666:
3667: static SPIRVTypeInst getPipeType(const TargetExtType *ExtensionType,
3668:                                  MachineIRBuilder &MIRBuilder,
3669:                                  SPIRVGlobalRegistry *GR) {
3670:   assert(ExtensionType->getNumIntParameters() == 1 &&
3671:          "Invalid number of parameters for SPIR-V pipe builtin!");
3672:   // Create or get an existing type from GlobalRegistry.
3673:   return GR->getOrCreateOpTypePipe(MIRBuilder,
3674:                                    SPIRV::AccessQualifier::AccessQualifier(
3675:                                        ExtensionType->getIntParameter(0)));
3676: }
3677:
3678: static SPIRVTypeInst getCoopMatrType(const TargetExtType *ExtensionType,
3679:                                      MachineIRBuilder &MIRBuilder,
3680:                                      SPIRVGlobalRegistry *GR) {
3681:   assert(ExtensionType->getNumIntParameters() == 4 &&
3682:          "Invalid number of parameters for SPIR-V coop matrices builtin!");
3683:   assert(ExtensionType->getNumTypeParameters() == 1 &&
3684:          "SPIR-V coop matrices builtin type must have a type parameter!");
3685:   SPIRVTypeInst ElemType =
3686:       GR->getOrCreateSPIRVType(ExtensionType->getTypeParameter(0), MIRBuilder,
3687:                                SPIRV::AccessQualifier::ReadWrite, true);
3688:   // Create or get an existing type from GlobalRegistry.
3689:   return GR->getOrCreateOpTypeCoopMatr(
3690:       MIRBuilder, ExtensionType, ElemType, ExtensionType->getIntParameter(0),
3691:       ExtensionType->getIntParameter(1), ExtensionType->getIntParameter(2),
3692:       ExtensionType->getIntParameter(3), true);
3693: }
3694:
3695: static SPIRVTypeInst getSampledImageType(const TargetExtType *OpaqueType,
3696:                                          MachineIRBuilder &MIRBuilder,
3697:                                          SPIRVGlobalRegistry *GR) {
3698:   SPIRVTypeInst OpaqueImageType = GR->getImageType(
3699:       OpaqueType, SPIRV::AccessQualifier::ReadOnly, MIRBuilder);
3700:   // Create or get an existing type from GlobalRegistry.
3701:   return GR->getOrCreateOpTypeSampledImage(OpaqueImageType, MIRBuilder);
3702: }
3703:
3704: static SPIRVTypeInst getInlineSpirvType(const TargetExtType *ExtensionType,
3705:                                         MachineIRBuilder &MIRBuilder,
3706:                                         SPIRVGlobalRegistry *GR) {
3707:   assert(ExtensionType->getNumIntParameters() == 3 &&
3708:          "Inline SPIR-V type builtin takes an opcode, size, and alignment "
3709:          "parameter");
3710:   auto Opcode = ExtensionType->getIntParameter(0);
3711:
3712:   SmallVector<MCOperand> Operands;
3713:   for (Type *Param : ExtensionType->type_params()) {
3714:     if (const TargetExtType *ParamEType = dyn_cast<TargetExtType>(Param)) {
3715:       if (ParamEType->getName() == "spirv.IntegralConstant") {
3716:         assert(ParamEType->getNumTypeParameters() == 1 &&
3717:                "Inline SPIR-V integral constant builtin must have a type "
3718:                "parameter");
3719:         assert(ParamEType->getNumIntParameters() == 1 &&
3720:                "Inline SPIR-V integral constant builtin must have a "
```
- EN: This range defines or declares important types such as parseBuiltinTypeStr, trim, parseBuiltinCallArgumentType, BuiltinType, shaping the data model used by SPIRVBuiltins.cpp.
- CN: 这一段定义或声明了 parseBuiltinTypeStr、trim、parseBuiltinCallArgumentType、BuiltinType 等关键类型，构成 SPIRVBuiltins.cpp 使用的数据模型。

### Lines 3721-3840
```cpp
3721:                "value parameter");
3722:
3723:         auto OperandValue = ParamEType->getIntParameter(0);
3724:         auto *OperandType = ParamEType->getTypeParameter(0);
3725:
3726:         SPIRVTypeInst OperandSPIRVType = GR->getOrCreateSPIRVType(
3727:             OperandType, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
3728:
3729:         Operands.push_back(MCOperand::createReg(GR->buildConstantInt(
3730:             OperandValue, MIRBuilder, OperandSPIRVType, true)));
3731:         continue;
3732:       } else if (ParamEType->getName() == "spirv.Literal") {
3733:         assert(ParamEType->getNumTypeParameters() == 0 &&
3734:                "Inline SPIR-V literal builtin does not take type "
3735:                "parameters");
3736:         assert(ParamEType->getNumIntParameters() == 1 &&
3737:                "Inline SPIR-V literal builtin must have an integer "
3738:                "parameter");
3739:
3740:         auto OperandValue = ParamEType->getIntParameter(0);
3741:
3742:         Operands.push_back(MCOperand::createImm(OperandValue));
3743:         continue;
3744:       }
3745:     }
3746:     SPIRVTypeInst TypeOperand = GR->getOrCreateSPIRVType(
3747:         Param, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
3748:     Operands.push_back(MCOperand::createReg(GR->getSPIRVTypeID(TypeOperand)));
3749:   }
3750:
3751:   return GR->getOrCreateUnknownType(ExtensionType, MIRBuilder, Opcode,
3752:                                     Operands);
3753: }
3754:
3755: static SPIRVTypeInst getVulkanBufferType(const TargetExtType *ExtensionType,
3756:                                          MachineIRBuilder &MIRBuilder,
3757:                                          SPIRVGlobalRegistry *GR) {
3758:   assert(ExtensionType->getNumTypeParameters() == 1 &&
3759:          "Vulkan buffers have exactly one type for the type of the buffer.");
3760:   assert(ExtensionType->getNumIntParameters() == 2 &&
3761:          "Vulkan buffer have 2 integer parameters: storage class and is "
3762:          "writable.");
3763:
3764:   auto *T = ExtensionType->getTypeParameter(0);
3765:   auto SC = static_cast<SPIRV::StorageClass::StorageClass>(
3766:       ExtensionType->getIntParameter(0));
3767:   bool IsWritable = ExtensionType->getIntParameter(1);
3768:   return GR->getOrCreateVulkanBufferType(MIRBuilder, T, SC, IsWritable);
3769: }
3770:
3771: static SPIRVTypeInst
3772: getVulkanPushConstantType(const TargetExtType *ExtensionType,
3773:                           MachineIRBuilder &MIRBuilder,
3774:                           SPIRVGlobalRegistry *GR) {
3775:   assert(ExtensionType->getNumTypeParameters() == 1 &&
3776:          "Vulkan push constants have exactly one type as argument.");
3777:   auto *T = ExtensionType->getTypeParameter(0);
3778:   return GR->getOrCreateVulkanPushConstantType(MIRBuilder, T);
3779: }
3780:
3781: static SPIRVTypeInst getLayoutType(const TargetExtType *ExtensionType,
3782:                                    MachineIRBuilder &MIRBuilder,
3783:                                    SPIRVGlobalRegistry *GR) {
3784:   return GR->getOrCreateLayoutType(MIRBuilder, ExtensionType);
3785: }
3786:
3787: namespace SPIRV {
3788: TargetExtType *parseBuiltinTypeNameToTargetExtType(std::string TypeName,
3789:                                                    LLVMContext &Context) {
3790:   StringRef NameWithParameters = TypeName;
3791:
3792:   // Pointers-to-opaque-structs representing OpenCL types are first translated
3793:   // to equivalent SPIR-V types. OpenCL builtin type names should have the
3794:   // following format: e.g. %opencl.event_t
3795:   if (NameWithParameters.starts_with("opencl.")) {
3796:     const SPIRV::OpenCLType *OCLTypeRecord =
3797:         SPIRV::lookupOpenCLType(NameWithParameters);
3798:     if (!OCLTypeRecord)
3799:       report_fatal_error("Missing TableGen record for OpenCL type: " +
3800:                          NameWithParameters);
3801:     NameWithParameters = OCLTypeRecord->SpirvTypeLiteral;
3802:     // Continue with the SPIR-V builtin type...
3803:   }
3804:
3805:   // Names of the opaque structs representing a SPIR-V builtins without
3806:   // parameters should have the following format: e.g. %spirv.Event
3807:   assert(NameWithParameters.starts_with("spirv.") &&
3808:          "Unknown builtin opaque type!");
3809:
3810:   // Parameterized SPIR-V builtins names follow this format:
3811:   // e.g. %spirv.Image._void_1_0_0_0_0_0_0, %spirv.Pipe._0
3812:   if (!NameWithParameters.contains('_'))
3813:     return TargetExtType::get(Context, NameWithParameters);
3814:
3815:   SmallVector<StringRef> Parameters;
3816:   unsigned BaseNameLength = NameWithParameters.find('_') - 1;
3817:   SplitString(NameWithParameters.substr(BaseNameLength + 1), Parameters, "_");
3818:
3819:   SmallVector<Type *, 1> TypeParameters;
3820:   bool HasTypeParameter = !isDigit(Parameters[0][0]);
3821:   if (HasTypeParameter)
3822:     TypeParameters.push_back(parseTypeString(Parameters[0], Context));
3823:   SmallVector<unsigned> IntParameters;
3824:   for (unsigned i = HasTypeParameter ? 1 : 0; i < Parameters.size(); i++) {
3825:     unsigned IntParameter = 0;
3826:     bool ValidLiteral = !Parameters[i].getAsInteger(10, IntParameter);
3827:     (void)ValidLiteral;
3828:     assert(ValidLiteral &&
3829:            "Invalid format of SPIR-V builtin parameter literal!");
3830:     IntParameters.push_back(IntParameter);
3831:   }
3832:   return TargetExtType::get(Context,
3833:                             NameWithParameters.substr(0, BaseNameLength),
3834:                             TypeParameters, IntParameters);
3835: }
3836:
3837: SPIRVTypeInst
3838: lowerBuiltinType(const Type *OpaqueType,
3839:                  SPIRV::AccessQualifier::AccessQualifier AccessQual,
3840:                  MachineIRBuilder &MIRBuilder, SPIRVGlobalRegistry *GR) {
```
- EN: This range defines or declares important types such as getIntParameter, getTypeParameter, push_back, and, shaping the data model used by SPIRVBuiltins.cpp.
- CN: 这一段定义或声明了 getIntParameter、getTypeParameter、push_back、and 等关键类型，构成 SPIRVBuiltins.cpp 使用的数据模型。

### Lines 3841-3916
```cpp
3841:   // In LLVM IR, SPIR-V and OpenCL builtin types are represented as either
3842:   // target(...) target extension types or pointers-to-opaque-structs. The
3843:   // approach relying on structs is deprecated and works only in the non-opaque
3844:   // pointer mode (-opaque-pointers=0).
3845:   // In order to maintain compatibility with LLVM IR generated by older versions
3846:   // of Clang and LLVM/SPIR-V Translator, the pointers-to-opaque-structs are
3847:   // "translated" to target extension types. This translation is temporary and
3848:   // will be removed in the future release of LLVM.
3849:   const TargetExtType *BuiltinType = dyn_cast<TargetExtType>(OpaqueType);
3850:   if (!BuiltinType)
3851:     BuiltinType = parseBuiltinTypeNameToTargetExtType(
3852:         OpaqueType->getStructName().str(), MIRBuilder.getContext());
3853:
3854:   unsigned NumStartingVRegs = MIRBuilder.getMRI()->getNumVirtRegs();
3855:
3856:   const StringRef Name = BuiltinType->getName();
3857:   LLVM_DEBUG(dbgs() << "Lowering builtin type: " << Name << "\n");
3858:
3859:   SPIRVTypeInst TargetType = nullptr;
3860:   if (Name == "spirv.Type") {
3861:     TargetType = getInlineSpirvType(BuiltinType, MIRBuilder, GR);
3862:   } else if (Name == "spirv.VulkanBuffer") {
3863:     TargetType = getVulkanBufferType(BuiltinType, MIRBuilder, GR);
3864:   } else if (Name == "spirv.Padding") {
3865:     TargetType = GR->getOrCreatePaddingType(MIRBuilder);
3866:   } else if (Name == "spirv.PushConstant") {
3867:     TargetType = getVulkanPushConstantType(BuiltinType, MIRBuilder, GR);
3868:   } else if (Name == "spirv.Layout") {
3869:     TargetType = getLayoutType(BuiltinType, MIRBuilder, GR);
3870:   } else {
3871:     // Lookup the demangled builtin type in the TableGen records.
3872:     const SPIRV::BuiltinType *TypeRecord = SPIRV::lookupBuiltinType(Name);
3873:     if (!TypeRecord)
3874:       report_fatal_error("Missing TableGen record for builtin type: " + Name);
3875:
3876:     // "Lower" the BuiltinType into TargetType. The following get<...>Type
3877:     // methods use the implementation details from TableGen records or
3878:     // TargetExtType parameters to either create a new OpType<...> machine
3879:     // instruction or get an existing equivalent SPIRV type from
3880:     // GlobalRegistry.
3881:
3882:     switch (TypeRecord->Opcode) {
3883:     case SPIRV::OpTypeImage:
3884:       TargetType = GR->getImageType(BuiltinType, AccessQual, MIRBuilder);
3885:       break;
3886:     case SPIRV::OpTypePipe:
3887:       TargetType = getPipeType(BuiltinType, MIRBuilder, GR);
3888:       break;
3889:     case SPIRV::OpTypeDeviceEvent:
3890:       TargetType = GR->getOrCreateOpTypeDeviceEvent(MIRBuilder);
3891:       break;
3892:     case SPIRV::OpTypeSampler:
3893:       TargetType = getSamplerType(MIRBuilder, GR);
3894:       break;
3895:     case SPIRV::OpTypeSampledImage:
3896:       TargetType = getSampledImageType(BuiltinType, MIRBuilder, GR);
3897:       break;
3898:     case SPIRV::OpTypeCooperativeMatrixKHR:
3899:       TargetType = getCoopMatrType(BuiltinType, MIRBuilder, GR);
3900:       break;
3901:     default:
3902:       TargetType =
3903:           getNonParameterizedType(BuiltinType, TypeRecord, MIRBuilder, GR);
3904:       break;
3905:     }
3906:   }
3907:
3908:   // Emit OpName instruction if a new OpType<...> instruction was added
3909:   // (equivalent type was not found in GlobalRegistry).
3910:   if (NumStartingVRegs < MIRBuilder.getMRI()->getNumVirtRegs())
3911:     buildOpName(GR->getSPIRVTypeID(TargetType), Name, MIRBuilder);
3912:
3913:   return TargetType;
3914: }
3915: } // namespace SPIRV
3916: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as getStructName, getMRI, getName, LLVM_DEBUG, translating backend policy into executable code.
- CN: 这一段实现了 getStructName、getMRI、getName、LLVM_DEBUG 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include DemangledBuiltin, IncomingCall, Arguments, isSpirvOp, NativeBuiltin, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DemangledBuiltin, IncomingCall, Arguments, isSpirvOp, NativeBuiltin，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVBuiltins.h`
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/StringExtras.h`
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
- System/standard headers / 系统或标准头文件:
  - `regex`
  - `string`
  - `tuple`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
