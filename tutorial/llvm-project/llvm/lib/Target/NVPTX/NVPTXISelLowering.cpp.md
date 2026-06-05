# NVPTXISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXISelLowering.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines the interfaces that NVPTX uses to lower LLVM code into a selection DAG.
- 目的（中文）: 该文件实现目标相关的指令选择逻辑，把 LLVM 或机器 IR 映射到后端可接受的表示。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-140
```cpp
  1: //===-- NVPTXISelLowering.cpp - NVPTX DAG Lowering Implementation ---------===//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8: //
  9: // This file defines the interfaces that NVPTX uses to lower LLVM code into a
 10: // selection DAG.
 11: //
 12: //===----------------------------------------------------------------------===//
 13:
 14: #include "NVPTXISelLowering.h"
 15: #include "MCTargetDesc/NVPTXBaseInfo.h"
 16: #include "NVPTX.h"
 17: #include "NVPTXISelDAGToDAG.h"
 18: #include "NVPTXSelectionDAGInfo.h"
 19: #include "NVPTXSubtarget.h"
 20: #include "NVPTXTargetMachine.h"
 21: #include "NVPTXTargetObjectFile.h"
 22: #include "NVPTXUtilities.h"
 23: #include "NVVMProperties.h"
 24: #include "llvm/ADT/APFloat.h"
 25: #include "llvm/ADT/APInt.h"
 26: #include "llvm/ADT/STLExtras.h"
 27: #include "llvm/ADT/SmallVector.h"
 28: #include "llvm/ADT/StringRef.h"
 29: #include "llvm/CodeGen/Analysis.h"
 30: #include "llvm/CodeGen/ISDOpcodes.h"
 31: #include "llvm/CodeGen/MachineFunction.h"
 32: #include "llvm/CodeGen/MachineJumpTableInfo.h"
 33: #include "llvm/CodeGen/MachineMemOperand.h"
 34: #include "llvm/CodeGen/SDPatternMatch.h"
 35: #include "llvm/CodeGen/SelectionDAG.h"
 36: #include "llvm/CodeGen/SelectionDAGNodes.h"
 37: #include "llvm/CodeGen/TargetCallingConv.h"
 38: #include "llvm/CodeGen/TargetLowering.h"
 39: #include "llvm/CodeGen/ValueTypes.h"
 40: #include "llvm/CodeGenTypes/MachineValueType.h"
 41: #include "llvm/IR/Argument.h"
 42: #include "llvm/IR/Attributes.h"
 43: #include "llvm/IR/Constants.h"
 44: #include "llvm/IR/DataLayout.h"
 45: #include "llvm/IR/DerivedTypes.h"
 46: #include "llvm/IR/DiagnosticInfo.h"
 47: #include "llvm/IR/FPEnv.h"
 48: #include "llvm/IR/Function.h"
 49: #include "llvm/IR/GlobalValue.h"
 50: #include "llvm/IR/IRBuilder.h"
 51: #include "llvm/IR/Instruction.h"
 52: #include "llvm/IR/Instructions.h"
 53: #include "llvm/IR/IntrinsicsNVPTX.h"
 54: #include "llvm/IR/Module.h"
 55: #include "llvm/IR/Type.h"
 56: #include "llvm/IR/Value.h"
 57: #include "llvm/Support/Alignment.h"
 58: #include "llvm/Support/AtomicOrdering.h"
 59: #include "llvm/Support/Casting.h"
 60: #include "llvm/Support/CodeGen.h"
 61: #include "llvm/Support/CommandLine.h"
 62: #include "llvm/Support/ErrorHandling.h"
 63: #include "llvm/Support/KnownBits.h"
 64: #include "llvm/Support/NVPTXAddrSpace.h"
 65: #include "llvm/Support/raw_ostream.h"
 66: #include "llvm/Target/TargetMachine.h"
 67: #include "llvm/Target/TargetOptions.h"
 68: #include <algorithm>
 69: #include <cassert>
 70: #include <cmath>
 71: #include <cstdint>
 72: #include <iterator>
 73: #include <optional>
 74: #include <string>
 75: #include <tuple>
 76: #include <utility>
 77: #include <vector>
 78:
 79: #define DEBUG_TYPE "nvptx-lower"
 80:
 81: using namespace llvm;
 82:
 83: static cl::opt<bool> sched4reg(
 84:     "nvptx-sched4reg",
 85:     cl::desc("NVPTX Specific: schedule for register pressue"), cl::init(false));
 86:
 87: static cl::opt<unsigned> FMAContractLevelOpt(
 88:     "nvptx-fma-level", cl::Hidden,
 89:     cl::desc("NVPTX Specific: FMA contraction (0: don't do it"
 90:              " 1: do it  2: do it aggressively"),
 91:     cl::init(2));
 92:
 93: static cl::opt<NVPTX::DivPrecisionLevel> UsePrecDivF32(
 94:     "nvptx-prec-divf32", cl::Hidden,
 95:     cl::desc(
 96:         "NVPTX Specific: Override the precision of the lowering for f32 fdiv"),
 97:     cl::values(
 98:         clEnumValN(NVPTX::DivPrecisionLevel::Approx, "0", "Use div.approx"),
 99:         clEnumValN(NVPTX::DivPrecisionLevel::Full, "1", "Use div.full"),
100:         clEnumValN(NVPTX::DivPrecisionLevel::IEEE754, "2",
101:                    "Use IEEE Compliant F32 div.rnd if available (default)"),
102:         clEnumValN(NVPTX::DivPrecisionLevel::IEEE754_NoFTZ, "3",
103:                    "Use IEEE Compliant F32 div.rnd if available, no FTZ")),
104:     cl::init(NVPTX::DivPrecisionLevel::IEEE754));
105:
106: static cl::opt<bool> UsePrecSqrtF32(
107:     "nvptx-prec-sqrtf32", cl::Hidden,
108:     cl::desc("NVPTX Specific: 0 use sqrt.approx, 1 use sqrt.rn."),
109:     cl::init(true));
110:
111: /// Whereas CUDA's implementation (see libdevice) uses ex2.approx for exp2(), it
112: /// does NOT use lg2.approx for log2, so this is disabled by default.
113: static cl::opt<bool> UseApproxLog2F32(
114:     "nvptx-approx-log2f32",
115:     cl::desc("NVPTX Specific: whether to use lg2.approx for log2"),
116:     cl::init(false));
117:
118: NVPTX::DivPrecisionLevel
119: NVPTXTargetLowering::getDivF32Level(const MachineFunction &MF,
120:                                     const SDNode &N) const {
121:   // If nvptx-prec-div32=N is used on the command-line, always honor it
122:   if (UsePrecDivF32.getNumOccurrences() > 0)
123:     return UsePrecDivF32;
124:
125:   const SDNodeFlags Flags = N.getFlags();
126:   if (Flags.hasApproximateFuncs())
127:     return NVPTX::DivPrecisionLevel::Approx;
128:
129:   return NVPTX::DivPrecisionLevel::IEEE754;
130: }
131:
132: bool NVPTXTargetLowering::usePrecSqrtF32(const SDNode *N) const {
133:   // If nvptx-prec-sqrtf32 is used on the command-line, always honor it
134:   if (UsePrecSqrtF32.getNumOccurrences() > 0)
135:     return UsePrecSqrtF32;
136:
137:   if (N) {
138:     const SDNodeFlags Flags = N->getFlags();
139:     if (Flags.hasApproximateFuncs())
140:       return false;
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 141-280
```cpp
141:   }
142:
143:   return true;
144: }
145:
146: bool NVPTXTargetLowering::useF32FTZ(const MachineFunction &MF) const {
147:   return MF.getDenormalMode(APFloat::IEEEsingle()).Output ==
148:          DenormalMode::PreserveSign;
149: }
150:
151: static bool IsPTXVectorType(MVT VT) {
152:   switch (VT.SimpleTy) {
153:   default:
154:     return false;
155:   case MVT::v2i1:
156:   case MVT::v4i1:
157:   case MVT::v2i8:
158:   case MVT::v4i8:
159:   case MVT::v8i8:  // <2 x i8x4>
160:   case MVT::v16i8: // <4 x i8x4>
161:   case MVT::v2i16:
162:   case MVT::v4i16:
163:   case MVT::v8i16: // <4 x i16x2>
164:   case MVT::v2i32:
165:   case MVT::v4i32:
166:   case MVT::v2i64:
167:   case MVT::v2f16:
168:   case MVT::v4f16:
169:   case MVT::v8f16: // <4 x f16x2>
170:   case MVT::v2bf16:
171:   case MVT::v4bf16:
172:   case MVT::v8bf16: // <4 x bf16x2>
173:   case MVT::v2f32:
174:   case MVT::v4f32:
175:   case MVT::v2f64:
176:   case MVT::v4i64:
177:   case MVT::v4f64:
178:   case MVT::v8i32:
179:   case MVT::v8f32:
180:   case MVT::v16f16:  // <8 x f16x2>
181:   case MVT::v16bf16: // <8 x bf16x2>
182:   case MVT::v16i16:  // <8 x i16x2>
183:   case MVT::v32i8:   // <8 x i8x4>
184:     return true;
185:   }
186: }
187:
188: // When legalizing vector loads/stores, this function is called, which does two
189: // things:
190: // 1. Determines Whether the vector is something we want to custom lower,
191: // std::nullopt is returned if we do not want to custom lower it.
192: // 2. If we do want to handle it, returns two parameters:
193: //    - unsigned int NumElts - The number of elements in the final vector
194: //    - EVT EltVT - The type of the elements in the final vector
195: static std::optional<std::pair<unsigned int, MVT>>
196: getVectorLoweringShape(EVT VectorEVT, const NVPTXSubtarget &STI,
197:                        unsigned AddressSpace) {
198:   const bool CanLowerTo256Bit = STI.has256BitVectorLoadStore(AddressSpace);
199:
200:   if (CanLowerTo256Bit && VectorEVT.isScalarInteger() &&
201:       VectorEVT.getSizeInBits() == 256)
202:     return {{4, MVT::i64}};
203:
204:   if (!VectorEVT.isSimple())
205:     return std::nullopt;
206:   const MVT VectorVT = VectorEVT.getSimpleVT();
207:
208:   if (!VectorVT.isVector()) {
209:     if (VectorVT == MVT::i128 || VectorVT == MVT::f128)
210:       return {{2, MVT::i64}};
211:     return std::nullopt;
212:   }
213:
214:   const MVT EltVT = VectorVT.getVectorElementType();
215:   const unsigned NumElts = VectorVT.getVectorNumElements();
216:
217:   // The size of the PTX virtual register that holds a packed type.
218:   unsigned PackRegSize;
219:
220:   // We only handle "native" vector sizes for now, e.g. <4 x double> is not
221:   // legal.  We can (and should) split that into 2 stores of <2 x double> here
222:   // but I'm leaving that as a TODO for now.
223:   switch (VectorVT.SimpleTy) {
224:   default:
225:     return std::nullopt;
226:
227:   case MVT::v4i64:
228:   case MVT::v4f64:
229:     // This is a "native" vector type iff the address space is global and the
230:     // target supports 256-bit loads/stores
231:     if (!CanLowerTo256Bit)
232:       return std::nullopt;
233:     [[fallthrough]];
234:   case MVT::v2i8:
235:   case MVT::v2i64:
236:   case MVT::v2f64:
237:     // This is a "native" vector type
238:     return std::pair(NumElts, EltVT);
239:
240:   case MVT::v16f16:  // <8 x f16x2>
241:   case MVT::v16bf16: // <8 x bf16x2>
242:   case MVT::v16i16:  // <8 x i16x2>
243:   case MVT::v32i8:   // <8 x i8x4>
244:     // This can be upsized into a "native" vector type iff the address space is
245:     // global and the target supports 256-bit loads/stores.
246:     if (!CanLowerTo256Bit)
247:       return std::nullopt;
248:     [[fallthrough]];
249:   case MVT::v2i16:  // <1 x i16x2>
250:   case MVT::v2f16:  // <1 x f16x2>
251:   case MVT::v2bf16: // <1 x bf16x2>
252:   case MVT::v4i8:   // <1 x i8x4>
253:   case MVT::v4i16:  // <2 x i16x2>
254:   case MVT::v4f16:  // <2 x f16x2>
255:   case MVT::v4bf16: // <2 x bf16x2>
256:   case MVT::v8i8:   // <2 x i8x4>
257:   case MVT::v8f16:  // <4 x f16x2>
258:   case MVT::v8bf16: // <4 x bf16x2>
259:   case MVT::v8i16:  // <4 x i16x2>
260:   case MVT::v16i8:  // <4 x i8x4>
261:     PackRegSize = 32;
262:     break;
263:
264:   case MVT::v8f32: // <4 x f32x2>
265:   case MVT::v8i32: // <4 x i32x2>
266:     // This is a "native" vector type iff the address space is global and the
267:     // target supports 256-bit loads/stores
268:     if (!CanLowerTo256Bit)
269:       return std::nullopt;
270:     [[fallthrough]];
271:   case MVT::v2f32: // <1 x f32x2>
272:   case MVT::v4f32: // <2 x f32x2>
273:   case MVT::v2i32: // <1 x i32x2>
274:   case MVT::v4i32: // <2 x i32x2>
275:     if (!STI.hasF32x2Instructions())
276:       return std::pair(NumElts, EltVT);
277:     PackRegSize = 64;
278:     break;
279:   }
280:
```
- EN: This range implements operational logic in helpers such as NVPTXTargetLowering::useF32FTZ, IsPTXVectorType, has256BitVectorLoadStore, getSizeInBits, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXTargetLowering::useF32FTZ、IsPTXVectorType、has256BitVectorLoadStore、getSizeInBits 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 281-420
```cpp
281:   // If we reach here, then we can pack 2 or more elements into a single 32-bit
282:   // or 64-bit PTX register and treat the vector as a new vector containing
283:   // packed elements.
284:
285:   // Number of elements to pack in one word.
286:   const unsigned NPerReg = PackRegSize / EltVT.getSizeInBits();
287:
288:   return std::pair(NumElts / NPerReg, MVT::getVectorVT(EltVT, NPerReg));
289: }
290:
291: /// ComputePTXValueVTs - For the given Type \p Ty, returns the set of primitive
292: /// legal-ish MVTs that compose it. Unlike ComputeValueVTs, this will legalize
293: /// the types as required by the calling convention (with special handling for
294: /// i8s).
295: /// NOTE: This is a band-aid for code that expects ComputeValueVTs to return the
296: /// same number of types as the Ins/Outs arrays in LowerFormalArguments,
297: /// LowerCall, and LowerReturn.
298: static void ComputePTXValueVTs(const TargetLowering &TLI, const DataLayout &DL,
299:                                LLVMContext &Ctx, CallingConv::ID CallConv,
300:                                Type *Ty, SmallVectorImpl<EVT> &ValueVTs,
301:                                SmallVectorImpl<uint64_t> &Offsets,
302:                                uint64_t StartingOffset = 0) {
303:   SmallVector<EVT, 16> TempVTs;
304:   SmallVector<uint64_t, 16> TempOffsets;
305:   ComputeValueVTs(TLI, DL, Ty, TempVTs, /*MemVTs=*/nullptr, &TempOffsets,
306:                   StartingOffset);
307:
308:   for (const auto [VT, Off] : zip(TempVTs, TempOffsets)) {
309:     MVT RegisterVT = TLI.getRegisterTypeForCallingConv(Ctx, CallConv, VT);
310:     unsigned NumRegs = TLI.getNumRegistersForCallingConv(Ctx, CallConv, VT);
311:
312:     // Since we actually can load/store b8, we need to ensure that we'll use
313:     // the original sized type for any i8s or i8 vectors.
314:     if (VT.getScalarType() == MVT::i8) {
315:       if (RegisterVT == MVT::i16)
316:         RegisterVT = MVT::i8;
317:       else if (RegisterVT == MVT::v2i16)
318:         RegisterVT = MVT::v2i8;
319:       else
320:         assert(RegisterVT == MVT::v4i8 &&
321:                "Expected v4i8, v2i16, or i16 for i8 RegisterVT");
322:     }
323:
324:     // TODO: This is horribly incorrect for cases where the vector elements are
325:     // not a multiple of bytes (ex i1) and legal or i8. However, this problem
326:     // has existed for as long as NVPTX has and no one has complained, so we'll
327:     // leave it for now.
328:     for (unsigned I : seq(NumRegs)) {
329:       ValueVTs.push_back(RegisterVT);
330:       Offsets.push_back(Off + I * RegisterVT.getStoreSize());
331:     }
332:   }
333: }
334:
335: // We return an EVT that can hold N VTs
336: // If the VT is a vector, the resulting EVT is a flat vector with the same
337: // element type as VT's element type.
338: static EVT getVectorizedVT(EVT VT, unsigned N, LLVMContext &C) {
339:   if (N == 1)
340:     return VT;
341:
342:   return VT.isVector() ? EVT::getVectorVT(C, VT.getScalarType(),
343:                                           VT.getVectorNumElements() * N)
344:                        : EVT::getVectorVT(C, VT, N);
345: }
346:
347: static SDValue getExtractVectorizedValue(SDValue V, unsigned I, EVT VT,
348:                                          const SDLoc &dl, SelectionDAG &DAG) {
349:   if (V.getValueType() == VT) {
350:     assert(I == 0 && "Index must be 0 for scalar value");
351:     return V;
352:   }
353:
354:   if (!VT.isVector())
355:     return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, VT, V,
356:                        DAG.getVectorIdxConstant(I, dl));
357:
358:   return DAG.getNode(
359:       ISD::EXTRACT_SUBVECTOR, dl, VT, V,
360:       DAG.getVectorIdxConstant(I * VT.getVectorNumElements(), dl));
361: }
362:
363: template <typename T>
364: static inline SDValue getBuildVectorizedValue(unsigned N, const SDLoc &dl,
365:                                               SelectionDAG &DAG, T GetElement) {
366:   if (N == 1)
367:     return GetElement(0);
368:
369:   SmallVector<SDValue, 8> Values;
370:   for (const unsigned I : llvm::seq(N)) {
371:     SDValue Val = GetElement(I);
372:     if (Val.getValueType().isVector())
373:       DAG.ExtractVectorElements(Val, Values);
374:     else
375:       Values.push_back(Val);
376:   }
377:
378:   EVT VT = EVT::getVectorVT(*DAG.getContext(), Values[0].getValueType(),
379:                             Values.size());
380:   return DAG.getBuildVector(VT, dl, Values);
381: }
382:
383: /// PromoteScalarIntegerPTX
384: /// Used to make sure the arguments/returns are suitable for passing
385: /// and promote them to a larger size if they're not.
386: ///
387: /// The promoted type is placed in \p PromoteVT if the function returns true.
388: static EVT promoteScalarIntegerPTX(const EVT VT) {
389:   if (VT.isScalarInteger()) {
390:     switch (PowerOf2Ceil(VT.getFixedSizeInBits())) {
391:     default:
392:       llvm_unreachable(
393:           "Promotion is not suitable for scalars of size larger than 64-bits");
394:     case 1:
395:       return MVT::i1;
396:     case 2:
397:     case 4:
398:     case 8:
399:       return MVT::i8;
400:     case 16:
401:       return MVT::i16;
402:     case 32:
403:       return MVT::i32;
404:     case 64:
405:       return MVT::i64;
406:     }
407:   }
408:   return VT;
409: }
410:
411: // Check whether we can merge loads/stores of some of the pieces of a
412: // flattened function parameter or return value into a single vector
413: // load/store.
414: //
415: // The flattened parameter is represented as a list of EVTs and
416: // offsets, and the whole structure is aligned to ParamAlignment. This
417: // function determines whether we can load/store pieces of the
418: // parameter starting at index Idx using a single vectorized op of
419: // size AccessSize. If so, it returns the number of param pieces
420: // covered by the vector op. Otherwise, it returns 1.
```
- EN: This range implements operational logic in helpers such as getSizeInBits, std::pair, getRegisterTypeForCallingConv, getNumRegistersForCallingConv, translating backend policy into executable code.
- CN: 这一段实现了 getSizeInBits、std::pair、getRegisterTypeForCallingConv、getNumRegistersForCallingConv 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-560
```cpp
421: template <typename T>
422: static unsigned canMergeParamLoadStoresStartingAt(
423:     unsigned Idx, uint32_t AccessSize, const SmallVectorImpl<EVT> &ValueVTs,
424:     const SmallVectorImpl<T> &Offsets, Align ParamAlignment) {
425:
426:   // Can't vectorize if param alignment is not sufficient.
427:   if (ParamAlignment < AccessSize)
428:     return 1;
429:   // Can't vectorize if offset is not aligned.
430:   if (Offsets[Idx] & (AccessSize - 1))
431:     return 1;
432:
433:   EVT EltVT = ValueVTs[Idx];
434:   unsigned EltSize = EltVT.getStoreSize();
435:
436:   // Element is too large to vectorize.
437:   if (EltSize >= AccessSize)
438:     return 1;
439:
440:   unsigned NumElts = AccessSize / EltSize;
441:   // Can't vectorize if AccessBytes if not a multiple of EltSize.
442:   if (AccessSize != EltSize * NumElts)
443:     return 1;
444:
445:   // We don't have enough elements to vectorize.
446:   if (Idx + NumElts > ValueVTs.size())
447:     return 1;
448:
449:   // PTX ISA can only deal with 2- and 4-element vector ops.
450:   if (NumElts != 4 && NumElts != 2)
451:     return 1;
452:
453:   for (unsigned j = Idx + 1; j < Idx + NumElts; ++j) {
454:     // Types do not match.
455:     if (ValueVTs[j] != EltVT)
456:       return 1;
457:
458:     // Elements are not contiguous.
459:     if (Offsets[j] - Offsets[j - 1] != EltSize)
460:       return 1;
461:   }
462:   // OK. We can vectorize ValueVTs[i..i+NumElts)
463:   return NumElts;
464: }
465:
466: // Computes whether and how we can vectorize the loads/stores of a
467: // flattened function parameter or return value.
468: //
469: // The flattened parameter is represented as the list of ValueVTs and
470: // Offsets, and is aligned to ParamAlignment bytes. We return a vector
471: // of the same size as ValueVTs indicating how each piece should be
472: // loaded/stored (i.e. as a scalar, or as part of a vector
473: // load/store).
474: template <typename T>
475: static SmallVector<unsigned, 16>
476: VectorizePTXValueVTs(const SmallVectorImpl<EVT> &ValueVTs,
477:                      const SmallVectorImpl<T> &Offsets, Align ParamAlignment,
478:                      bool IsVAArg = false) {
479:   // Set vector size to match ValueVTs and mark all elements as
480:   // scalars by default.
481:
482:   if (IsVAArg)
483:     return SmallVector<unsigned>(ValueVTs.size(), 1);
484:
485:   SmallVector<unsigned, 16> VectorInfo;
486:
487:   const auto GetNumElts = [&](unsigned I) -> unsigned {
488:     for (const unsigned AccessSize : {16, 8, 4, 2}) {
489:       const unsigned NumElts = canMergeParamLoadStoresStartingAt(
490:           I, AccessSize, ValueVTs, Offsets, ParamAlignment);
491:       assert((NumElts == 1 || NumElts == 2 || NumElts == 4) &&
492:              "Unexpected vectorization size");
493:       if (NumElts != 1)
494:         return NumElts;
495:     }
496:     return 1;
497:   };
498:
499:   // Check what we can vectorize using 128/64/32-bit accesses.
500:   for (unsigned I = 0, E = ValueVTs.size(); I != E;) {
501:     const unsigned NumElts = GetNumElts(I);
502:     VectorInfo.push_back(NumElts);
503:     I += NumElts;
504:   }
505:   assert(std::accumulate(VectorInfo.begin(), VectorInfo.end(), 0u) ==
506:          ValueVTs.size());
507:   return VectorInfo;
508: }
509:
510: // NVPTXTargetLowering Constructor.
511: NVPTXTargetLowering::NVPTXTargetLowering(const NVPTXTargetMachine &TM,
512:                                          const NVPTXSubtarget &STI)
513:     : TargetLowering(TM, STI), nvTM(&TM), STI(STI), GlobalUniqueCallSite(0) {
514:   // always lower memset, memcpy, and memmove intrinsics to load/store
515:   // instructions, rather
516:   // then generating calls to memset, mempcy or memmove.
517:   MaxStoresPerMemset = MaxStoresPerMemsetOptSize = (unsigned)0xFFFFFFFF;
518:   MaxStoresPerMemcpy = MaxStoresPerMemcpyOptSize = (unsigned) 0xFFFFFFFF;
519:   MaxStoresPerMemmove = MaxStoresPerMemmoveOptSize = (unsigned) 0xFFFFFFFF;
520:
521:   setBooleanContents(ZeroOrNegativeOneBooleanContent);
522:   setBooleanVectorContents(ZeroOrNegativeOneBooleanContent);
523:
524:   // Jump is Expensive. Don't create extra control flow for 'and', 'or'
525:   // condition branches.
526:   setJumpIsExpensive(true);
527:
528:   // Wide divides are _very_ slow. Try to reduce the width of the divide if
529:   // possible.
530:   addBypassSlowDiv(64, 32);
531:
532:   // By default, use the Source scheduling
533:   if (sched4reg)
534:     setSchedulingPreference(Sched::RegPressure);
535:   else
536:     setSchedulingPreference(Sched::Source);
537:
538:   auto setFP16OperationAction = [&](unsigned Op, MVT VT, LegalizeAction Action,
539:                                     LegalizeAction NoF16Action) {
540:     bool IsOpSupported = STI.allowFP16Math();
541:     switch (Op) {
542:     // Several FP16 instructions are available on sm_80 only.
543:     case ISD::FMINNUM:
544:     case ISD::FMAXNUM:
545:     case ISD::FMAXNUM_IEEE:
546:     case ISD::FMINNUM_IEEE:
547:     case ISD::FMAXIMUM:
548:     case ISD::FMINIMUM:
549:     case ISD::FMAXIMUMNUM:
550:     case ISD::FMINIMUMNUM:
551:       IsOpSupported &= STI.getSmVersion() >= 80 && STI.getPTXVersion() >= 70;
552:       break;
553:     case ISD::FEXP2:
554:       IsOpSupported &= STI.getSmVersion() >= 75 && STI.getPTXVersion() >= 70;
555:       break;
556:     }
557:     setOperationAction(Op, VT, IsOpSupported ? Action : NoF16Action);
558:   };
559:
560:   auto setBF16OperationAction = [&](unsigned Op, MVT VT, LegalizeAction Action,
```
- EN: This range implements operational logic in helpers such as getStoreSize, size, GetNumElts, push_back, translating backend policy into executable code.
- CN: 这一段实现了 getStoreSize、size、GetNumElts、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 561-700
```cpp
561:                                     LegalizeAction NoBF16Action) {
562:     bool IsOpSupported = STI.hasNativeBF16Support(Op);
563:     setOperationAction(
564:         Op, VT, IsOpSupported ? Action : NoBF16Action);
565:   };
566:
567:   auto setI16x2OperationAction = [&](unsigned Op, MVT VT, LegalizeAction Action,
568:                                      LegalizeAction NoI16x2Action) {
569:     bool IsOpSupported = false;
570:     // instructions are available on sm_90 only
571:     switch (Op) {
572:     case ISD::ADD:
573:     case ISD::SMAX:
574:     case ISD::SMIN:
575:     case ISD::UMIN:
576:     case ISD::UMAX:
577:       IsOpSupported = STI.getSmVersion() >= 90 && STI.getPTXVersion() >= 80;
578:       break;
579:     }
580:     setOperationAction(Op, VT, IsOpSupported ? Action : NoI16x2Action);
581:   };
582:
583:   addRegisterClass(MVT::i1, &NVPTX::B1RegClass);
584:   addRegisterClass(MVT::i16, &NVPTX::B16RegClass);
585:   addRegisterClass(MVT::v2i16, &NVPTX::B32RegClass);
586:   addRegisterClass(MVT::v4i8, &NVPTX::B32RegClass);
587:   addRegisterClass(MVT::i32, &NVPTX::B32RegClass);
588:   addRegisterClass(MVT::i64, &NVPTX::B64RegClass);
589:   addRegisterClass(MVT::f32, &NVPTX::B32RegClass);
590:   addRegisterClass(MVT::f64, &NVPTX::B64RegClass);
591:   addRegisterClass(MVT::f16, &NVPTX::B16RegClass);
592:   addRegisterClass(MVT::v2f16, &NVPTX::B32RegClass);
593:   addRegisterClass(MVT::bf16, &NVPTX::B16RegClass);
594:   addRegisterClass(MVT::v2bf16, &NVPTX::B32RegClass);
595:
596:   if (STI.hasF32x2Instructions()) {
597:     addRegisterClass(MVT::v2f32, &NVPTX::B64RegClass);
598:     addRegisterClass(MVT::v2i32, &NVPTX::B64RegClass);
599:   }
600:
601:   // Conversion to/from FP16/FP16x2 is always legal.
602:   setOperationAction(ISD::BUILD_VECTOR, MVT::v2f16, Custom);
603:   setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v2f16, Custom);
604:   setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v2f16, Expand);
605:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v2f16, Expand);
606:
607:   setOperationAction(ISD::READCYCLECOUNTER, MVT::i64, Legal);
608:   if (STI.getSmVersion() >= 30 && STI.getPTXVersion() > 31)
609:     setOperationAction(ISD::READSTEADYCOUNTER, MVT::i64, Legal);
610:
611:   setFP16OperationAction(ISD::SETCC, MVT::f16, Legal, Promote);
612:   setFP16OperationAction(ISD::SETCC, MVT::v2f16, Legal, Expand);
613:
614:   // Conversion to/from BFP16/BFP16x2 is always legal.
615:   setOperationAction(ISD::BUILD_VECTOR, MVT::v2bf16, Custom);
616:   setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v2bf16, Custom);
617:   setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v2bf16, Expand);
618:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v2bf16, Expand);
619:
620:   setBF16OperationAction(ISD::SETCC, MVT::v2bf16, Legal, Expand);
621:   setBF16OperationAction(ISD::SETCC, MVT::bf16, Legal, Promote);
622:   if (getOperationAction(ISD::SETCC, MVT::bf16) == Promote)
623:     AddPromotedToType(ISD::SETCC, MVT::bf16, MVT::f32);
624:
625:   // Conversion to/from i16/i16x2 is always legal.
626:   setOperationAction(ISD::BUILD_VECTOR, MVT::v2i16, Custom);
627:   setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v2i16, Custom);
628:   setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v2i16, Expand);
629:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v2i16, Expand);
630:
631:   setOperationAction(ISD::BUILD_VECTOR, MVT::v4i8, Custom);
632:   setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v4i8, Custom);
633:   setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v4i8, Custom);
634:   setOperationAction(ISD::VECTOR_SHUFFLE, MVT::v4i8, Custom);
635:
636:   // No support for these operations with v2f32/v2i32
637:   setOperationAction(ISD::INSERT_VECTOR_ELT, {MVT::v2f32, MVT::v2i32}, Expand);
638:   setOperationAction(ISD::VECTOR_SHUFFLE, {MVT::v2f32, MVT::v2i32}, Expand);
639:
640:   setOperationAction(ISD::TRUNCATE, MVT::v2i16, Expand);
641:   setOperationAction({ISD::ANY_EXTEND, ISD::ZERO_EXTEND, ISD::SIGN_EXTEND},
642:                      MVT::v2i32, Expand);
643:
644:   // Need custom lowering in case the index is dynamic.
645:   if (STI.hasF32x2Instructions())
646:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, {MVT::v2f32, MVT::v2i32},
647:                        Custom);
648:
649:   // Custom conversions to/from v2i8.
650:   setOperationAction(ISD::BITCAST, MVT::v2i8, Custom);
651:
652:   // Only logical ops can be done on v4i8/v2i32 directly, others must be done
653:   // elementwise.
654:   setOperationAction(
655:       {ISD::ABS,         ISD::ADD,        ISD::ADDC,        ISD::ADDE,
656:        ISD::BITREVERSE,  ISD::CTLZ,       ISD::CTPOP,       ISD::CTTZ,
657:        ISD::FP_TO_SINT,  ISD::FP_TO_UINT, ISD::FSHL,        ISD::FSHR,
658:        ISD::MUL,         ISD::MULHS,      ISD::MULHU,       ISD::PARITY,
659:        ISD::ROTL,        ISD::ROTR,       ISD::SADDO,       ISD::SADDO_CARRY,
660:        ISD::SADDSAT,     ISD::SDIV,       ISD::SDIVREM,     ISD::SELECT_CC,
661:        ISD::SETCC,       ISD::SHL,        ISD::SINT_TO_FP,  ISD::SMAX,
662:        ISD::SMIN,        ISD::SMULO,      ISD::SMUL_LOHI,   ISD::SRA,
663:        ISD::SREM,        ISD::SRL,        ISD::SSHLSAT,     ISD::SSUBO,
664:        ISD::SSUBO_CARRY, ISD::SSUBSAT,    ISD::SUB,         ISD::SUBC,
665:        ISD::SUBE,        ISD::UADDO,      ISD::UADDO_CARRY, ISD::UADDSAT,
666:        ISD::UDIV,        ISD::UDIVREM,    ISD::UINT_TO_FP,  ISD::UMAX,
667:        ISD::UMIN,        ISD::UMULO,      ISD::UMUL_LOHI,   ISD::UREM,
668:        ISD::USHLSAT,     ISD::USUBO,      ISD::USUBO_CARRY, ISD::VSELECT,
669:        ISD::USUBSAT},
670:       {MVT::v4i8, MVT::v2i32}, Expand);
671:
672:   // Operations not directly supported by NVPTX.
673:   for (MVT VT : {MVT::bf16, MVT::f16, MVT::v2bf16, MVT::v2f16, MVT::f32,
674:                  MVT::v2f32, MVT::f64, MVT::i1, MVT::i8, MVT::i16, MVT::v2i16,
675:                  MVT::v4i8, MVT::i32, MVT::v2i32, MVT::i64}) {
676:     setOperationAction(ISD::SELECT_CC, VT, Expand);
677:     setOperationAction(ISD::BR_CC, VT, Expand);
678:   }
679:
680:   // We don't want ops like FMINIMUM or UMAX to be lowered to SETCC+VSELECT.
681:   setOperationAction(ISD::VSELECT, {MVT::v2f32, MVT::v2i32}, Expand);
682:
683:   // Some SIGN_EXTEND_INREG can be done using cvt instruction.
684:   // For others we will expand to a SHL/SRA pair.
685:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i64, Legal);
686:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i32, Legal);
687:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i16, Legal);
688:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i8 , Legal);
689:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Expand);
690:   setOperationAction(ISD::SIGN_EXTEND_INREG, {MVT::v2i16, MVT::v2i32}, Expand);
691:
692:   setOperationAction(ISD::SHL_PARTS, MVT::i32  , Custom);
693:   setOperationAction(ISD::SRA_PARTS, MVT::i32  , Custom);
694:   setOperationAction(ISD::SRL_PARTS, MVT::i32  , Custom);
695:   setOperationAction(ISD::SHL_PARTS, MVT::i64  , Custom);
696:   setOperationAction(ISD::SRA_PARTS, MVT::i64  , Custom);
697:   setOperationAction(ISD::SRL_PARTS, MVT::i64  , Custom);
698:
699:   setOperationAction(ISD::BITREVERSE, MVT::i32, Legal);
700:   setOperationAction(ISD::BITREVERSE, MVT::i64, Legal);
```
- EN: This range implements operational logic in helpers such as hasNativeBF16Support, setOperationAction, addRegisterClass, setFP16OperationAction, translating backend policy into executable code.
- CN: 这一段实现了 hasNativeBF16Support、setOperationAction、addRegisterClass、setFP16OperationAction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 701-840
```cpp
701:
702:   setOperationAction({ISD::ROTL, ISD::ROTR},
703:                      {MVT::i8, MVT::i16, MVT::v2i16, MVT::i32, MVT::i64},
704:                      Expand);
705:
706:   if (STI.hasHWROT32()) {
707:     setOperationAction({ISD::FSHL, ISD::FSHR}, MVT::i32, Legal);
708:     setOperationAction({ISD::ROTL, ISD::ROTR, ISD::FSHL, ISD::FSHR}, MVT::i64,
709:                        Custom);
710:   }
711:
712:   setOperationAction(ISD::BR_JT, MVT::Other, STI.hasBrx() ? Legal : Expand);
713:   setOperationAction(ISD::BRIND, MVT::Other, Expand);
714:
715:   // We want to legalize constant related memmove and memcopy
716:   // intrinsics.
717:   setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::Other, Custom);
718:
719:   // FP extload/truncstore is not legal in PTX. We need to expand all these.
720:   for (auto FloatVTs :
721:        {MVT::fp_valuetypes(), MVT::fp_fixedlen_vector_valuetypes()}) {
722:     for (MVT ValVT : FloatVTs) {
723:       for (MVT MemVT : FloatVTs) {
724:         setLoadExtAction(ISD::EXTLOAD, ValVT, MemVT, Expand);
725:         setTruncStoreAction(ValVT, MemVT, Expand);
726:       }
727:     }
728:   }
729:
730:   // To improve CodeGen we'll legalize any-extend loads to zext loads. This is
731:   // how they'll be lowered in ISel anyway, and by doing this a little earlier
732:   // we allow for more DAG combine opportunities.
733:   for (auto IntVTs :
734:        {MVT::integer_valuetypes(), MVT::integer_fixedlen_vector_valuetypes()})
735:     for (MVT ValVT : IntVTs)
736:       for (MVT MemVT : IntVTs)
737:         if (isTypeLegal(ValVT))
738:           setLoadExtAction(ISD::EXTLOAD, ValVT, MemVT, Custom);
739:
740:   // PTX does not support load / store predicate registers
741:   setOperationAction({ISD::LOAD, ISD::STORE}, MVT::i1, Custom);
742:   for (MVT VT : MVT::integer_valuetypes()) {
743:     setLoadExtAction({ISD::SEXTLOAD, ISD::ZEXTLOAD, ISD::EXTLOAD}, VT, MVT::i1,
744:                      Promote);
745:     setTruncStoreAction(VT, MVT::i1, Expand);
746:   }
747:
748:   // Disable generations of extload/truncstore for v2i32/v2i16/v2i8. The generic
749:   // expansion for these nodes when they are unaligned is incorrect if the
750:   // type is a vector.
751:   //
752:   // TODO: Fix the generic expansion for these nodes found in
753:   //       TargetLowering::expandUnalignedLoad/Store.
754:   setLoadExtAction({ISD::EXTLOAD, ISD::SEXTLOAD, ISD::ZEXTLOAD}, MVT::v2i16,
755:                    MVT::v2i8, Expand);
756:   setLoadExtAction({ISD::EXTLOAD, ISD::SEXTLOAD, ISD::ZEXTLOAD}, MVT::v2i32,
757:                    {MVT::v2i8, MVT::v2i16}, Expand);
758:   setTruncStoreAction(MVT::v2i16, MVT::v2i8, Expand);
759:   setTruncStoreAction(MVT::v2i32, MVT::v2i16, Expand);
760:   setTruncStoreAction(MVT::v2i32, MVT::v2i8, Expand);
761:
762:   // Register custom handling for illegal type loads/stores. We'll try to custom
763:   // lower almost all illegal types and logic in the lowering will discard cases
764:   // we can't handle.
765:   setOperationAction({ISD::LOAD, ISD::STORE}, {MVT::i128, MVT::i256, MVT::f128},
766:                      Custom);
767:   for (MVT VT : MVT::fixedlen_vector_valuetypes())
768:     if (!isTypeLegal(VT) && VT.getStoreSizeInBits() <= 256)
769:       setOperationAction({ISD::STORE, ISD::LOAD, ISD::MSTORE, ISD::MLOAD}, VT,
770:                          Custom);
771:
772:   // Custom legalization for LDU intrinsics.
773:   // TODO: The logic to lower these is not very robust and we should rewrite it.
774:   //       Perhaps LDU should not be represented as an intrinsic at all.
775:   setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::i8, Custom);
776:   for (MVT VT : MVT::fixedlen_vector_valuetypes())
777:     if (IsPTXVectorType(VT))
778:       setOperationAction(ISD::INTRINSIC_W_CHAIN, VT, Custom);
779:
780:   setCondCodeAction({ISD::SETNE, ISD::SETEQ, ISD::SETUGE, ISD::SETULE,
781:                      ISD::SETUGT, ISD::SETULT, ISD::SETGT, ISD::SETLT,
782:                      ISD::SETGE, ISD::SETLE},
783:                     MVT::i1, Expand);
784:
785:   // This is legal in NVPTX
786:   setOperationAction(ISD::ConstantFP, MVT::f64, Legal);
787:   setOperationAction(ISD::ConstantFP, MVT::f32, Legal);
788:   setOperationAction(ISD::ConstantFP, MVT::f16, Legal);
789:   setOperationAction(ISD::ConstantFP, MVT::bf16, Legal);
790:
791:   setOperationAction(ISD::DYNAMIC_STACKALLOC, {MVT::i32, MVT::i64}, Custom);
792:   setOperationAction({ISD::STACKRESTORE, ISD::STACKSAVE}, MVT::Other, Custom);
793:
794:   // TRAP can be lowered to PTX trap
795:   setOperationAction(ISD::TRAP, MVT::Other, Legal);
796:   // DEBUGTRAP can be lowered to PTX brkpt
797:   setOperationAction(ISD::DEBUGTRAP, MVT::Other, Legal);
798:
799:   // Support varargs.
800:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
801:   setOperationAction(ISD::VAARG, MVT::Other, Custom);
802:   setOperationAction(ISD::VACOPY, MVT::Other, Expand);
803:   setOperationAction(ISD::VAEND, MVT::Other, Expand);
804:
805:   setOperationAction({ISD::SMIN, ISD::SMAX, ISD::UMIN, ISD::UMAX},
806:                      {MVT::i16, MVT::i32, MVT::i64}, Legal);
807:   // PTX abs.s is undefined for INT_MIN, so ISD::ABS (which requires
808:   // abs(INT_MIN) == INT_MIN) must be expanded. ABS_MIN_POISON matches
809:   // PTX abs semantics since INT_MIN input is poison/undefined.
810:   setOperationAction(ISD::ABS, {MVT::i16, MVT::i32, MVT::i64}, Expand);
811:   setOperationAction(ISD::ABS_MIN_POISON, {MVT::i16, MVT::i32, MVT::i64},
812:                      Legal);
813:
814:   setOperationAction({ISD::CTPOP, ISD::CTLZ, ISD::CTLZ_ZERO_POISON}, MVT::i16,
815:                      Promote);
816:   setOperationAction({ISD::CTPOP, ISD::CTLZ}, MVT::i32, Legal);
817:   setOperationAction({ISD::CTPOP, ISD::CTLZ}, MVT::i64, Custom);
818:
819:   setI16x2OperationAction(ISD::ABS_MIN_POISON, MVT::v2i16, Legal, Custom);
820:   setI16x2OperationAction(ISD::SMIN, MVT::v2i16, Legal, Custom);
821:   setI16x2OperationAction(ISD::SMAX, MVT::v2i16, Legal, Custom);
822:   setI16x2OperationAction(ISD::UMIN, MVT::v2i16, Legal, Custom);
823:   setI16x2OperationAction(ISD::UMAX, MVT::v2i16, Legal, Custom);
824:   setI16x2OperationAction(ISD::CTPOP, MVT::v2i16, Legal, Expand);
825:   setI16x2OperationAction(ISD::CTLZ, MVT::v2i16, Legal, Expand);
826:
827:   setI16x2OperationAction(ISD::ADD, MVT::v2i16, Legal, Custom);
828:   setI16x2OperationAction(ISD::SUB, MVT::v2i16, Legal, Custom);
829:   setI16x2OperationAction(ISD::MUL, MVT::v2i16, Legal, Custom);
830:   setI16x2OperationAction(ISD::SHL, MVT::v2i16, Legal, Custom);
831:   setI16x2OperationAction(ISD::SREM, MVT::v2i16, Legal, Custom);
832:   setI16x2OperationAction(ISD::UREM, MVT::v2i16, Legal, Custom);
833:
834:   // Other arithmetic and logic ops are unsupported.
835:   setOperationAction({ISD::SDIV, ISD::UDIV, ISD::SRA, ISD::SRL, ISD::MULHS,
836:                       ISD::MULHU, ISD::FP_TO_SINT, ISD::FP_TO_UINT,
837:                       ISD::SINT_TO_FP, ISD::UINT_TO_FP, ISD::SETCC},
838:                      {MVT::v2i16, MVT::v2i32}, Expand);
839:
840:   // v2i32 is not supported for any arithmetic operations
```
- EN: This range implements operational logic in helpers such as setOperationAction, MVT::fp_valuetypes, setLoadExtAction, setTruncStoreAction, translating backend policy into executable code.
- CN: 这一段实现了 setOperationAction、MVT::fp_valuetypes、setLoadExtAction、setTruncStoreAction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 841-980
```cpp
841:   setOperationAction({ISD::ABS, ISD::SMIN, ISD::SMAX, ISD::UMIN, ISD::UMAX,
842:                       ISD::CTPOP, ISD::CTLZ, ISD::ADD, ISD::SUB, ISD::MUL,
843:                       ISD::SHL, ISD::SRA, ISD::SRL, ISD::OR, ISD::AND, ISD::XOR,
844:                       ISD::SREM, ISD::UREM},
845:                      MVT::v2i32, Expand);
846:
847:   setOperationAction(ISD::ADDC, MVT::i32, Legal);
848:   setOperationAction(ISD::ADDE, MVT::i32, Legal);
849:   setOperationAction(ISD::SUBC, MVT::i32, Legal);
850:   setOperationAction(ISD::SUBE, MVT::i32, Legal);
851:   if (STI.getPTXVersion() >= 43) {
852:     setOperationAction(ISD::ADDC, MVT::i64, Legal);
853:     setOperationAction(ISD::ADDE, MVT::i64, Legal);
854:     setOperationAction(ISD::SUBC, MVT::i64, Legal);
855:     setOperationAction(ISD::SUBE, MVT::i64, Legal);
856:   }
857:
858:   setOperationAction(ISD::CTTZ, MVT::i16, Expand);
859:   setOperationAction(ISD::CTTZ, {MVT::v2i16, MVT::v2i32}, Expand);
860:   setOperationAction(ISD::CTTZ, MVT::i32, Expand);
861:   setOperationAction(ISD::CTTZ, MVT::i64, Expand);
862:
863:   // PTX does not directly support SELP of i1, so promote to i32 first
864:   setOperationAction(ISD::SELECT, MVT::i1, Custom);
865:
866:   // PTX cannot multiply two i64s in a single instruction.
867:   setOperationAction(ISD::SMUL_LOHI, MVT::i64, Expand);
868:   setOperationAction(ISD::UMUL_LOHI, MVT::i64, Expand);
869:
870:   // We have some custom DAG combine patterns for these nodes
871:   setTargetDAGCombine({ISD::ADD,
872:                        ISD::AND,
873:                        ISD::EXTRACT_VECTOR_ELT,
874:                        ISD::FADD,
875:                        ISD::FMAXNUM,
876:                        ISD::FMINNUM,
877:                        ISD::FMAXIMUM,
878:                        ISD::FMINIMUM,
879:                        ISD::FMAXIMUMNUM,
880:                        ISD::FMINIMUMNUM,
881:                        ISD::MUL,
882:                        ISD::SELECT,
883:                        ISD::SHL,
884:                        ISD::SREM,
885:                        ISD::UREM,
886:                        ISD::VSELECT,
887:                        ISD::BUILD_VECTOR,
888:                        ISD::ADDRSPACECAST,
889:                        ISD::LOAD,
890:                        ISD::STORE,
891:                        ISD::ZERO_EXTEND,
892:                        ISD::SIGN_EXTEND,
893:                        ISD::INTRINSIC_WO_CHAIN});
894:
895:   // If the vector operands require register coalescing, scalarize instead
896:   if (STI.hasF32x2Instructions())
897:     setTargetDAGCombine({ISD::FMA, ISD::FMUL, ISD::FSUB});
898:
899:   // setcc for f16x2 and bf16x2 needs special handling to prevent
900:   // legalizer's attempt to scalarize it due to v2i1 not being legal.
901:   if (STI.allowFP16Math() || STI.hasBF16Math())
902:     setTargetDAGCombine(ISD::SETCC);
903:
904:   // Vector reduction operations. These may be turned into shuffle or tree
905:   // reductions depending on what instructions are available for each type.
906:   for (MVT VT : MVT::fixedlen_vector_valuetypes()) {
907:     MVT EltVT = VT.getVectorElementType();
908:     if (EltVT == MVT::f32 || EltVT == MVT::f64) {
909:       setOperationAction({ISD::VECREDUCE_FMAX, ISD::VECREDUCE_FMIN,
910:                           ISD::VECREDUCE_FMAXIMUM, ISD::VECREDUCE_FMINIMUM},
911:                          VT, Custom);
912:     }
913:   }
914:
915:   // Promote fp16 arithmetic if fp16 hardware isn't available or the
916:   // user passed --nvptx-no-fp16-math. The flag is useful because,
917:   // although sm_53+ GPUs have some sort of FP16 support in
918:   // hardware, only sm_53 and sm_60 have full implementation. Others
919:   // only have token amount of hardware and are likely to run faster
920:   // by using fp32 units instead.
921:   for (const auto &Op : {ISD::FADD, ISD::FMUL, ISD::FSUB, ISD::FMA}) {
922:     setFP16OperationAction(Op, MVT::f16, Legal, Promote);
923:     setFP16OperationAction(Op, MVT::v2f16, Legal, Expand);
924:     setBF16OperationAction(Op, MVT::v2bf16, Legal, Expand);
925:     // bf16 must be promoted to f32.
926:     setBF16OperationAction(Op, MVT::bf16, Legal, Promote);
927:     if (getOperationAction(Op, MVT::bf16) == Promote)
928:       AddPromotedToType(Op, MVT::bf16, MVT::f32);
929:     setOperationAction(Op, MVT::v2f32,
930:                        STI.hasF32x2Instructions() ? Legal : Expand);
931:   }
932:
933:   // On SM80, we select add/mul/sub as fma to avoid promotion to float
934:   for (const auto &Op : {ISD::FADD, ISD::FMUL, ISD::FSUB}) {
935:     for (const auto &VT : {MVT::bf16, MVT::v2bf16}) {
936:       if (!STI.hasNativeBF16Support(Op) && STI.hasNativeBF16Support(ISD::FMA)) {
937:         setOperationAction(Op, VT, Custom);
938:       }
939:     }
940:   }
941:
942:   // f16/f16x2 neg was introduced in PTX 60, SM_53.
943:   const bool IsFP16FP16x2NegAvailable = STI.getSmVersion() >= 53 &&
944:                                         STI.getPTXVersion() >= 60 &&
945:                                         STI.allowFP16Math();
946:   for (const auto &VT : {MVT::f16, MVT::v2f16})
947:     setOperationAction(ISD::FNEG, VT,
948:                        IsFP16FP16x2NegAvailable ? Legal : Expand);
949:
950:   setBF16OperationAction(ISD::FNEG, MVT::bf16, Legal, Expand);
951:   setBF16OperationAction(ISD::FNEG, MVT::v2bf16, Legal, Expand);
952:   setOperationAction(ISD::FNEG, MVT::v2f32, Expand);
953:   // (would be) Library functions.
954:
955:   // These map to conversion instructions for scalar FP types.
956:   for (const auto &Op : {ISD::FCEIL, ISD::FFLOOR, ISD::FNEARBYINT, ISD::FRINT,
957:                          ISD::FROUNDEVEN, ISD::FTRUNC}) {
958:     setOperationAction(Op, MVT::f16, Legal);
959:     setOperationAction(Op, MVT::f32, Legal);
960:     setOperationAction(Op, MVT::f64, Legal);
961:     setOperationAction(Op, MVT::v2f16, Expand);
962:     setOperationAction(Op, MVT::v2bf16, Expand);
963:     setOperationAction(Op, MVT::v2f32, Expand);
964:     setBF16OperationAction(Op, MVT::bf16, Legal, Promote);
965:     if (getOperationAction(Op, MVT::bf16) == Promote)
966:       AddPromotedToType(Op, MVT::bf16, MVT::f32);
967:   }
968:
969:   if (STI.getSmVersion() < 80 || STI.getPTXVersion() < 71) {
970:     setOperationAction(ISD::BF16_TO_FP, MVT::f32, Expand);
971:   }
972:   if (STI.getSmVersion() < 90 || STI.getPTXVersion() < 78) {
973:     for (MVT VT : {MVT::bf16, MVT::f32, MVT::f64}) {
974:       setOperationAction(ISD::FP_EXTEND, VT, Custom);
975:       setOperationAction(ISD::FP_ROUND, VT, Custom);
976:     }
977:   }
978:
979:   // Expand v2f32 = fp_extend
980:   setOperationAction(ISD::FP_EXTEND, MVT::v2f32, Expand);
```
- EN: This range implements operational logic in helpers such as setOperationAction, setTargetDAGCombine, getVectorElementType, setFP16OperationAction, translating backend policy into executable code.
- CN: 这一段实现了 setOperationAction、setTargetDAGCombine、getVectorElementType、setFP16OperationAction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 981-1120
```cpp
 981:   // Expand v2[b]f16 = fp_round v2f32
 982:   setOperationAction(ISD::FP_ROUND, {MVT::v2bf16, MVT::v2f16}, Expand);
 983:
 984:   // sm_80 only has conversions between f32 and bf16. Custom lower all other
 985:   // bf16 conversions.
 986:   if (STI.getSmVersion() < 90 || STI.getPTXVersion() < 78) {
 987:     for (MVT VT : {MVT::i1, MVT::i16, MVT::i32, MVT::i64}) {
 988:       setOperationAction(
 989:           {ISD::SINT_TO_FP, ISD::UINT_TO_FP, ISD::FP_TO_SINT, ISD::FP_TO_UINT},
 990:           VT, Custom);
 991:     }
 992:     setOperationAction(
 993:         {ISD::SINT_TO_FP, ISD::UINT_TO_FP, ISD::FP_TO_SINT, ISD::FP_TO_UINT},
 994:         MVT::bf16, Custom);
 995:   }
 996:
 997:   setOperationAction(ISD::FROUND, MVT::f16, Promote);
 998:   setOperationAction(ISD::FROUND, MVT::v2f16, Expand);
 999:   setOperationAction(ISD::FROUND, MVT::v2bf16, Expand);
1000:   setOperationAction(ISD::FROUND, MVT::f32, Custom);
1001:   setOperationAction(ISD::FROUND, MVT::f64, Custom);
1002:   setOperationAction(ISD::FROUND, MVT::bf16, Promote);
1003:   AddPromotedToType(ISD::FROUND, MVT::bf16, MVT::f32);
1004:
1005:   // 'Expand' implements FCOPYSIGN without calling an external library.
1006:   setOperationAction(ISD::FCOPYSIGN, MVT::f16, Expand);
1007:   setOperationAction(ISD::FCOPYSIGN, MVT::v2f16, Expand);
1008:   setOperationAction(ISD::FCOPYSIGN, MVT::bf16, Expand);
1009:   setOperationAction(ISD::FCOPYSIGN, MVT::v2bf16, Expand);
1010:   setOperationAction(ISD::FCOPYSIGN, MVT::f32, Custom);
1011:   setOperationAction(ISD::FCOPYSIGN, MVT::f64, Custom);
1012:
1013:   // These map to corresponding instructions for f32/f64. f16 must be
1014:   // promoted to f32. v2f16 is expanded to f16, which is then promoted
1015:   // to f32.
1016:   for (const auto &Op :
1017:        {ISD::FDIV, ISD::FREM, ISD::FSQRT, ISD::FSIN, ISD::FCOS, ISD::FTANH}) {
1018:     setOperationAction(Op, MVT::f16, Promote);
1019:     setOperationAction(Op, MVT::f32, Legal);
1020:     // only div/rem/sqrt are legal for f64
1021:     if (Op == ISD::FDIV || Op == ISD::FREM || Op == ISD::FSQRT) {
1022:       setOperationAction(Op, MVT::f64, Legal);
1023:     }
1024:     setOperationAction(Op, {MVT::v2f16, MVT::v2bf16, MVT::v2f32}, Expand);
1025:     setOperationAction(Op, MVT::bf16, Promote);
1026:     AddPromotedToType(Op, MVT::bf16, MVT::f32);
1027:   }
1028:   setOperationAction(ISD::FREM, {MVT::f32, MVT::f64}, Custom);
1029:
1030:   setOperationAction(ISD::FABS, {MVT::f32, MVT::f64}, Legal);
1031:   setOperationAction(ISD::FABS, MVT::v2f32, Expand);
1032:   if (STI.getPTXVersion() >= 65) {
1033:     setFP16OperationAction(ISD::FABS, MVT::f16, Legal, Promote);
1034:     setFP16OperationAction(ISD::FABS, MVT::v2f16, Legal, Expand);
1035:   } else {
1036:     setOperationAction(ISD::FABS, MVT::f16, Promote);
1037:     setOperationAction(ISD::FABS, MVT::v2f16, Expand);
1038:   }
1039:   setBF16OperationAction(ISD::FABS, MVT::v2bf16, Legal, Expand);
1040:   setBF16OperationAction(ISD::FABS, MVT::bf16, Legal, Promote);
1041:   if (getOperationAction(ISD::FABS, MVT::bf16) == Promote)
1042:     AddPromotedToType(ISD::FABS, MVT::bf16, MVT::f32);
1043:
1044:   for (const auto &Op :
1045:        {ISD::FMINNUM, ISD::FMAXNUM, ISD::FMINIMUMNUM, ISD::FMAXIMUMNUM}) {
1046:     setOperationAction(Op, MVT::f32, Legal);
1047:     setOperationAction(Op, MVT::f64, Legal);
1048:     setFP16OperationAction(Op, MVT::f16, Legal, Promote);
1049:     setFP16OperationAction(Op, MVT::v2f16, Legal, Expand);
1050:     setBF16OperationAction(Op, MVT::v2bf16, Legal, Expand);
1051:     setBF16OperationAction(Op, MVT::bf16, Legal, Promote);
1052:     if (getOperationAction(Op, MVT::bf16) == Promote)
1053:       AddPromotedToType(Op, MVT::bf16, MVT::f32);
1054:     setOperationAction(Op, MVT::v2f32, Expand);
1055:   }
1056:   bool SupportsF32MinMaxNaN =
1057:       STI.getSmVersion() >= 80 && STI.getPTXVersion() >= 70;
1058:   for (const auto &Op : {ISD::FMINIMUM, ISD::FMAXIMUM}) {
1059:     setOperationAction(Op, MVT::f32, SupportsF32MinMaxNaN ? Legal : Expand);
1060:     setFP16OperationAction(Op, MVT::f16, Legal, Expand);
1061:     setFP16OperationAction(Op, MVT::v2f16, Legal, Expand);
1062:     setBF16OperationAction(Op, MVT::bf16, Legal, Expand);
1063:     setBF16OperationAction(Op, MVT::v2bf16, Legal, Expand);
1064:     setOperationAction(Op, MVT::v2f32, Expand);
1065:   }
1066:
1067:   // Custom lowering for inline asm with 128-bit operands
1068:   setOperationAction(ISD::CopyToReg, MVT::i128, Custom);
1069:   setOperationAction(ISD::CopyFromReg, MVT::i128, Custom);
1070:
1071:   // FEXP2 support:
1072:   // - f32
1073:   // - f16/f16x2 (sm_70+, PTX 7.0+)
1074:   // - bf16/bf16x2 (sm_90+, PTX 7.8+)
1075:   // When f16/bf16 types aren't supported, they are promoted/expanded to f32.
1076:   setOperationAction(ISD::FEXP2, MVT::f32, Legal);
1077:   setOperationAction(ISD::FEXP2, MVT::v2f32, Expand);
1078:   setFP16OperationAction(ISD::FEXP2, MVT::f16, Legal, Promote);
1079:   setFP16OperationAction(ISD::FEXP2, MVT::v2f16, Legal, Expand);
1080:   setBF16OperationAction(ISD::FEXP2, MVT::bf16, Legal, Promote);
1081:   setBF16OperationAction(ISD::FEXP2, MVT::v2bf16, Legal, Expand);
1082:
1083:   // FLOG2 supports f32 only
1084:   // f16/bf16 types aren't supported, but they are promoted/expanded to f32.
1085:   if (UseApproxLog2F32) {
1086:     setOperationAction(ISD::FLOG2, MVT::f32, Legal);
1087:     setOperationPromotedToType(ISD::FLOG2, MVT::f16, MVT::f32);
1088:     setOperationPromotedToType(ISD::FLOG2, MVT::bf16, MVT::f32);
1089:     setOperationAction(ISD::FLOG2, {MVT::v2f16, MVT::v2bf16, MVT::v2f32},
1090:                        Expand);
1091:   }
1092:
1093:   setOperationAction(ISD::ADDRSPACECAST, {MVT::i32, MVT::i64}, Custom);
1094:
1095:   setOperationAction(ISD::ATOMIC_LOAD_SUB, {MVT::i32, MVT::i64}, Expand);
1096:
1097:   // atom.b128 is legal in PTX but since we don't represent i128 as a legal
1098:   // type, we need to custom lower it.
1099:   setOperationAction({ISD::ATOMIC_CMP_SWAP, ISD::ATOMIC_SWAP}, MVT::i128,
1100:                      Custom);
1101:
1102:   // Now deduce the information based on the above mentioned
1103:   // actions
1104:   computeRegisterProperties(STI.getRegisterInfo());
1105:
1106:   // PTX support for 16-bit CAS is emulated. Only use 32+
1107:   setMinCmpXchgSizeInBits(STI.getMinCmpXchgSizeInBits());
1108:   setMaxAtomicSizeInBitsSupported(STI.hasAtomSwap128() ? 128 : 64);
1109:   setMaxDivRemBitWidthSupported(64);
1110:
1111:   // Custom lowering for tcgen05.ld vector operands
1112:   setOperationAction(ISD::INTRINSIC_W_CHAIN,
1113:                      {MVT::v2i32, MVT::v4i32, MVT::v8i32, MVT::v16i32,
1114:                       MVT::v32i32, MVT::v64i32, MVT::v128i32, MVT::v2f32,
1115:                       MVT::v4f32, MVT::v8f32, MVT::v16f32, MVT::v32f32,
1116:                       MVT::v64f32, MVT::v128f32},
1117:                      Custom);
1118:
1119:   // Custom lowering for tcgen05.st vector operands
1120:   setOperationAction(ISD::INTRINSIC_VOID,
```
- EN: This range implements operational logic in helpers such as setOperationAction, AddPromotedToType, setFP16OperationAction, setBF16OperationAction, translating backend policy into executable code.
- CN: 这一段实现了 setOperationAction、AddPromotedToType、setFP16OperationAction、setBF16OperationAction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1121-1260
```cpp
1121:                      {MVT::v2i32, MVT::v4i32, MVT::v8i32, MVT::v16i32,
1122:                       MVT::v32i32, MVT::v64i32, MVT::v128i32, MVT::Other},
1123:                      Custom);
1124:
1125:   // Enable custom lowering for the following:
1126:   //   * MVT::i128 - clusterlaunchcontrol
1127:   //   * MVT::i32 - prmt
1128:   //   * MVT::v4f32 - cvt_rs fp{4/6/8}x4 intrinsics
1129:   //   * MVT::Other - internal.addrspace.wrap
1130:   setOperationAction(ISD::INTRINSIC_WO_CHAIN,
1131:                      {MVT::i32, MVT::i128, MVT::v4f32, MVT::Other}, Custom);
1132:
1133:   // Custom lowering for bswap
1134:   setOperationAction(ISD::BSWAP, {MVT::i16, MVT::i32, MVT::i64, MVT::v2i16},
1135:                      Custom);
1136: }
1137:
1138: TargetLoweringBase::LegalizeTypeAction
1139: NVPTXTargetLowering::getPreferredVectorAction(MVT VT) const {
1140:   if (!VT.isScalableVector() && VT.getVectorNumElements() != 1 &&
1141:       VT.getScalarType() == MVT::i1)
1142:     return TypeSplitVector;
1143:   return TargetLoweringBase::getPreferredVectorAction(VT);
1144: }
1145:
1146: SDValue NVPTXTargetLowering::getSqrtEstimate(SDValue Operand, SelectionDAG &DAG,
1147:                                              int Enabled, int &ExtraSteps,
1148:                                              bool &UseOneConst,
1149:                                              bool Reciprocal) const {
1150:   if (!(Enabled == ReciprocalEstimate::Enabled ||
1151:         (Enabled == ReciprocalEstimate::Unspecified && !usePrecSqrtF32())))
1152:     return SDValue();
1153:
1154:   if (ExtraSteps == ReciprocalEstimate::Unspecified)
1155:     ExtraSteps = 0;
1156:
1157:   SDLoc DL(Operand);
1158:   EVT VT = Operand.getValueType();
1159:   bool Ftz = useF32FTZ(DAG.getMachineFunction());
1160:
1161:   auto MakeIntrinsicCall = [&](Intrinsic::ID IID) {
1162:     return DAG.getNode(ISD::INTRINSIC_WO_CHAIN, DL, VT,
1163:                        DAG.getConstant(IID, DL, MVT::i32), Operand);
1164:   };
1165:
1166:   // The sqrt and rsqrt refinement processes assume we always start out with an
1167:   // approximation of the rsqrt.  Therefore, if we're going to do any refinement
1168:   // (i.e. ExtraSteps > 0), we must return an rsqrt.  But if we're *not* doing
1169:   // any refinement, we must return a regular sqrt.
1170:   if (Reciprocal || ExtraSteps > 0) {
1171:     if (VT == MVT::f32)
1172:       return MakeIntrinsicCall(Ftz ? Intrinsic::nvvm_rsqrt_approx_ftz_f
1173:                                    : Intrinsic::nvvm_rsqrt_approx_f);
1174:     else if (VT == MVT::f64)
1175:       return MakeIntrinsicCall(Intrinsic::nvvm_rsqrt_approx_d);
1176:     else
1177:       return SDValue();
1178:   } else {
1179:     if (VT == MVT::f32)
1180:       return MakeIntrinsicCall(Ftz ? Intrinsic::nvvm_sqrt_approx_ftz_f
1181:                                    : Intrinsic::nvvm_sqrt_approx_f);
1182:     else {
1183:       // There's no sqrt.approx.f64 instruction, so we emit
1184:       // reciprocal(rsqrt(x)).  This is faster than
1185:       // select(x == 0, 0, x * rsqrt(x)).  (In fact, it's faster than plain
1186:       // x * rsqrt(x).)
1187:       return DAG.getNode(
1188:           ISD::INTRINSIC_WO_CHAIN, DL, VT,
1189:           DAG.getConstant(Intrinsic::nvvm_rcp_approx_ftz_d, DL, MVT::i32),
1190:           MakeIntrinsicCall(Intrinsic::nvvm_rsqrt_approx_d));
1191:     }
1192:   }
1193: }
1194:
1195: static Align getArgumentAlignment(const CallBase *CB, Type *Ty, unsigned Idx,
1196:                                   const DataLayout &DL);
1197:
1198: std::string NVPTXTargetLowering::getPrototype(
1199:     const DataLayout &DL, Type *RetTy, const ArgListTy &Args,
1200:     const SmallVectorImpl<ISD::OutputArg> &Outs,
1201:     std::optional<unsigned> FirstVAArg, const CallBase &CB,
1202:     unsigned UniqueCallSite) const {
1203:   auto PtrVT = getPointerTy(DL);
1204:
1205:   std::string Prototype;
1206:   raw_string_ostream O(Prototype);
1207:   O << "prototype_" << UniqueCallSite << " : .callprototype ";
1208:
1209:   if (RetTy->isVoidTy()) {
1210:     O << "()";
1211:   } else {
1212:     O << "(";
1213:     if (shouldPassAsArray(RetTy)) {
1214:       const Align RetAlign = getArgumentAlignment(&CB, RetTy, 0, DL);
1215:       O << ".param .align " << RetAlign.value() << " .b8 _["
1216:         << DL.getTypeAllocSize(RetTy) << "]";
1217:     } else if (RetTy->isFloatingPointTy() || RetTy->isIntegerTy()) {
1218:       unsigned size = 0;
1219:       if (auto *ITy = dyn_cast<IntegerType>(RetTy)) {
1220:         size = ITy->getBitWidth();
1221:       } else {
1222:         assert(RetTy->isFloatingPointTy() &&
1223:                "Floating point type expected here");
1224:         size = RetTy->getPrimitiveSizeInBits();
1225:       }
1226:       // PTX ABI requires all scalar return values to be at least 32
1227:       // bits in size.  fp16 normally uses .b16 as its storage type in
1228:       // PTX, so its size must be adjusted here, too.
1229:       size = promoteScalarArgumentSize(size);
1230:
1231:       O << ".param .b" << size << " _";
1232:     } else if (isa<PointerType>(RetTy)) {
1233:       O << ".param .b" << PtrVT.getSizeInBits() << " _";
1234:     } else {
1235:       llvm_unreachable("Unknown return type");
1236:     }
1237:     O << ") ";
1238:   }
1239:   O << "_ (";
1240:
1241:   bool first = true;
1242:
1243:   const unsigned NumArgs = FirstVAArg.value_or(Args.size());
1244:   auto AllOuts = ArrayRef(Outs);
1245:   for (const unsigned I : llvm::seq(NumArgs)) {
1246:     const auto ArgOuts =
1247:         AllOuts.take_while([I](auto O) { return O.OrigArgIndex == I; });
1248:     AllOuts = AllOuts.drop_front(ArgOuts.size());
1249:
1250:     Type *Ty = Args[I].Ty;
1251:     if (!first) {
1252:       O << ", ";
1253:     }
1254:     first = false;
1255:
1256:     if (ArgOuts[0].Flags.isByVal()) {
1257:       // Indirect calls need strict ABI alignment so we disable optimizations by
1258:       // not providing a function to optimize.
1259:       Type *ETy = Args[I].IndirectType;
1260:       Align InitialAlign = ArgOuts[0].Flags.getNonZeroByValAlign();
```
- EN: This range implements operational logic in helpers such as NVPTXTargetLowering::getPreferredVectorAction, getScalarType, TargetLoweringBase::getPreferredVectorAction, usePrecSqrtF32, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXTargetLowering::getPreferredVectorAction、getScalarType、TargetLoweringBase::getPreferredVectorAction、usePrecSqrtF32 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1261-1400
```cpp
1261:       Align ParamByValAlign =
1262:           getFunctionByValParamAlign(/*F=*/nullptr, ETy, InitialAlign, DL);
1263:
1264:       O << ".param .align " << ParamByValAlign.value() << " .b8 _["
1265:         << ArgOuts[0].Flags.getByValSize() << "]";
1266:     } else {
1267:       if (shouldPassAsArray(Ty)) {
1268:         Align ParamAlign =
1269:             getArgumentAlignment(&CB, Ty, I + AttributeList::FirstArgIndex, DL);
1270:         O << ".param .align " << ParamAlign.value() << " .b8 _["
1271:           << DL.getTypeAllocSize(Ty) << "]";
1272:         continue;
1273:       }
1274:       // i8 types in IR will be i16 types in SDAG
1275:       assert((getValueType(DL, Ty) == ArgOuts[0].VT ||
1276:               (getValueType(DL, Ty) == MVT::i8 && ArgOuts[0].VT == MVT::i16)) &&
1277:              "type mismatch between callee prototype and arguments");
1278:       // scalar type
1279:       unsigned sz = 0;
1280:       if (auto *ITy = dyn_cast<IntegerType>(Ty)) {
1281:         sz = promoteScalarArgumentSize(ITy->getBitWidth());
1282:       } else if (isa<PointerType>(Ty)) {
1283:         sz = PtrVT.getSizeInBits();
1284:       } else {
1285:         sz = Ty->getPrimitiveSizeInBits();
1286:       }
1287:       O << ".param .b" << sz << " _";
1288:     }
1289:   }
1290:
1291:   if (FirstVAArg)
1292:     O << (first ? "" : ",") << " .param .align "
1293:       << STI.getMaxRequiredAlignment() << " .b8 _[]";
1294:   O << ")";
1295:   if (shouldEmitPTXNoReturn(&CB, *nvTM))
1296:     O << " .noreturn";
1297:   O << ";";
1298:
1299:   return Prototype;
1300: }
1301:
1302: static Align getArgumentAlignment(const CallBase *CB, Type *Ty, unsigned Idx,
1303:                                   const DataLayout &DL) {
1304:   if (!CB) {
1305:     // CallSite is zero, fallback to ABI type alignment
1306:     return DL.getABITypeAlign(Ty);
1307:   }
1308:
1309:   const Function *DirectCallee = CB->getCalledFunction();
1310:
1311:   if (!DirectCallee) {
1312:     // We don't have a direct function symbol, but that may be because of
1313:     // constant cast instructions in the call.
1314:
1315:     // With bitcast'd call targets, the instruction will be the call
1316:     if (const auto *CI = dyn_cast<CallInst>(CB)) {
1317:       // Check if we have call alignment metadata
1318:       if (MaybeAlign StackAlign = getAlign(*CI, Idx))
1319:         return StackAlign.value();
1320:     }
1321:     DirectCallee = getMaybeBitcastedCallee(CB);
1322:   }
1323:
1324:   // Check for function alignment information if we found that the
1325:   // ultimate target is a Function
1326:   if (DirectCallee)
1327:     return getFunctionArgumentAlignment(DirectCallee, Ty, Idx, DL);
1328:
1329:   // Call is indirect, fall back to the ABI type alignment
1330:   return DL.getABITypeAlign(Ty);
1331: }
1332:
1333: static MachinePointerInfo refinePtrAS(SDValue &Ptr, SelectionDAG &DAG,
1334:                                       const DataLayout &DL,
1335:                                       const TargetLowering &TL) {
1336:   if (Ptr->getOpcode() == ISD::FrameIndex) {
1337:     auto Ty = TL.getPointerTy(DL, ADDRESS_SPACE_LOCAL);
1338:     Ptr = DAG.getAddrSpaceCast(SDLoc(), Ty, Ptr, ADDRESS_SPACE_GENERIC,
1339:                                ADDRESS_SPACE_LOCAL);
1340:
1341:     return MachinePointerInfo(ADDRESS_SPACE_LOCAL);
1342:   }
1343:
1344:   // Peel of an addrspacecast to generic and load directly from the specific
1345:   // address space.
1346:   if (Ptr->getOpcode() == ISD::ADDRSPACECAST) {
1347:     const auto *ASC = cast<AddrSpaceCastSDNode>(Ptr);
1348:     if (ASC->getDestAddressSpace() == ADDRESS_SPACE_GENERIC) {
1349:       Ptr = ASC->getOperand(0);
1350:       return MachinePointerInfo(ASC->getSrcAddressSpace());
1351:     }
1352:   }
1353:
1354:   return MachinePointerInfo();
1355: }
1356:
1357: static ISD::NodeType getExtOpcode(const ISD::ArgFlagsTy &Flags) {
1358:   if (Flags.isSExt())
1359:     return ISD::SIGN_EXTEND;
1360:   if (Flags.isZExt())
1361:     return ISD::ZERO_EXTEND;
1362:   return ISD::ANY_EXTEND;
1363: }
1364:
1365: static SDValue correctParamType(SDValue V, EVT ExpectedVT,
1366:                                 ISD::ArgFlagsTy Flags, SelectionDAG &DAG,
1367:                                 SDLoc dl) {
1368:   const EVT ActualVT = V.getValueType();
1369:   assert((ActualVT == ExpectedVT ||
1370:           (ExpectedVT.isInteger() && ActualVT.isInteger())) &&
1371:          "Non-integer argument type size mismatch");
1372:   if (ExpectedVT.bitsGT(ActualVT))
1373:     return DAG.getNode(getExtOpcode(Flags), dl, ExpectedVT, V);
1374:   if (ExpectedVT.bitsLT(ActualVT))
1375:     return DAG.getNode(ISD::TRUNCATE, dl, ExpectedVT, V);
1376:
1377:   return V;
1378: }
1379:
1380: SDValue NVPTXTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
1381:                                        SmallVectorImpl<SDValue> &InVals) const {
1382:
1383:   if (CLI.IsVarArg && (STI.getPTXVersion() < 60 || STI.getSmVersion() < 30))
1384:     report_fatal_error(
1385:         "Support for variadic functions (unsized array parameter) introduced "
1386:         "in PTX ISA version 6.0 and requires target sm_30.");
1387:
1388:   SelectionDAG &DAG = CLI.DAG;
1389:   SDLoc dl = CLI.DL;
1390:   const SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
1391:   SDValue Callee = CLI.Callee;
1392:   ArgListTy &Args = CLI.getArgs();
1393:   Type *RetTy = CLI.RetTy;
1394:   const CallBase *CB = CLI.CB;
1395:   const DataLayout &DL = DAG.getDataLayout();
1396:   LLVMContext &Ctx = *DAG.getContext();
1397:
1398:   const auto GetI32 = [&](const unsigned I) {
1399:     return DAG.getConstant(I, dl, MVT::i32);
1400:   };
```
- EN: This range implements operational logic in helpers such as getFunctionByValParamAlign, getArgumentAlignment, promoteScalarArgumentSize, getSizeInBits, translating backend policy into executable code.
- CN: 这一段实现了 getFunctionByValParamAlign、getArgumentAlignment、promoteScalarArgumentSize、getSizeInBits 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1401-1540
```cpp
1401:
1402:   const unsigned UniqueCallSite = GlobalUniqueCallSite++;
1403:   const SDValue CallChain = CLI.Chain;
1404:   const SDValue StartChain =
1405:       DAG.getCALLSEQ_START(CallChain, UniqueCallSite, 0, dl);
1406:   SDValue DeclareGlue = StartChain.getValue(1);
1407:
1408:   SmallVector<SDValue, 16> CallPrereqs{StartChain};
1409:
1410:   const auto MakeDeclareScalarParam = [&](SDValue Symbol, unsigned Size) {
1411:     // PTX ABI requires integral types to be at least 32 bits in size. FP16 is
1412:     // loaded/stored using i16, so it's handled here as well.
1413:     const unsigned SizeBits = promoteScalarArgumentSize(Size * 8);
1414:     SDValue Declare =
1415:         DAG.getNode(NVPTXISD::DeclareScalarParam, dl, {MVT::Other, MVT::Glue},
1416:                     {StartChain, Symbol, GetI32(SizeBits), DeclareGlue});
1417:     CallPrereqs.push_back(Declare);
1418:     DeclareGlue = Declare.getValue(1);
1419:     return Declare;
1420:   };
1421:
1422:   const auto MakeDeclareArrayParam = [&](SDValue Symbol, Align Align,
1423:                                          unsigned Size) {
1424:     SDValue Declare = DAG.getNode(
1425:         NVPTXISD::DeclareArrayParam, dl, {MVT::Other, MVT::Glue},
1426:         {StartChain, Symbol, GetI32(Align.value()), GetI32(Size), DeclareGlue});
1427:     CallPrereqs.push_back(Declare);
1428:     DeclareGlue = Declare.getValue(1);
1429:     return Declare;
1430:   };
1431:
1432:   // Variadic arguments.
1433:   //
1434:   // Normally, for each argument, we declare a param scalar or a param
1435:   // byte array in the .param space, and store the argument value to that
1436:   // param scalar or array starting at offset 0.
1437:   //
1438:   // In the case of the first variadic argument, we declare a vararg byte array
1439:   // with size 0. The exact size of this array isn't known at this point, so
1440:   // it'll be patched later. All the variadic arguments will be stored to this
1441:   // array at a certain offset (which gets tracked by 'VAOffset'). The offset is
1442:   // initially set to 0, so it can be used for non-variadic arguments (which use
1443:   // 0 offset) to simplify the code.
1444:   //
1445:   // After all vararg is processed, 'VAOffset' holds the size of the
1446:   // vararg byte array.
1447:   assert((CLI.IsVarArg || CLI.Args.size() == CLI.NumFixedArgs) &&
1448:          "Non-VarArg function with extra arguments");
1449:
1450:   const unsigned FirstVAArg = CLI.NumFixedArgs; // position of first variadic
1451:   unsigned VAOffset = 0; // current offset in the param array
1452:
1453:   const SDValue VADeclareParam =
1454:       CLI.Args.size() > FirstVAArg
1455:           ? MakeDeclareArrayParam(getCallParamSymbol(DAG, FirstVAArg, MVT::i32),
1456:                                   Align(STI.getMaxRequiredAlignment()), 0)
1457:           : SDValue();
1458:
1459:   // Args.size() and Outs.size() need not match.
1460:   // Outs.size() will be larger
1461:   //   * if there is an aggregate argument with multiple fields (each field
1462:   //     showing up separately in Outs)
1463:   //   * if there is a vector argument with more than typical vector-length
1464:   //     elements (generally if more than 4) where each vector element is
1465:   //     individually present in Outs.
1466:   // So a different index should be used for indexing into Outs/OutVals.
1467:   // See similar issue in LowerFormalArguments.
1468:   auto AllOuts = ArrayRef(CLI.Outs);
1469:   auto AllOutVals = ArrayRef(CLI.OutVals);
1470:   assert(AllOuts.size() == AllOutVals.size() &&
1471:          "Outs and OutVals must be the same size");
1472:   // Declare the .params or .reg need to pass values
1473:   // to the function
1474:   for (const auto E : llvm::enumerate(Args)) {
1475:     const auto ArgI = E.index();
1476:     const auto Arg = E.value();
1477:     const auto ArgOuts =
1478:         AllOuts.take_while([&](auto O) { return O.OrigArgIndex == ArgI; });
1479:     const auto ArgOutVals = AllOutVals.take_front(ArgOuts.size());
1480:     AllOuts = AllOuts.drop_front(ArgOuts.size());
1481:     AllOutVals = AllOutVals.drop_front(ArgOuts.size());
1482:
1483:     const bool IsVAArg = (ArgI >= FirstVAArg);
1484:     const bool IsByVal = Arg.IsByVal;
1485:
1486:     const SDValue ParamSymbol =
1487:         getCallParamSymbol(DAG, IsVAArg ? FirstVAArg : ArgI, MVT::i32);
1488:
1489:     assert((!IsByVal || Arg.IndirectType) &&
1490:            "byval arg must have indirect type");
1491:     Type *ETy = (IsByVal ? Arg.IndirectType : Arg.Ty);
1492:
1493:     const Align ArgAlign = [&]() {
1494:       if (IsByVal) {
1495:         // The ByValAlign in the Outs[OIdx].Flags is always set at this point,
1496:         // so we don't need to worry whether it's naturally aligned or not.
1497:         // See TargetLowering::LowerCallTo().
1498:         const Align InitialAlign = ArgOuts[0].Flags.getNonZeroByValAlign();
1499:         return getFunctionByValParamAlign(CB->getCalledFunction(), ETy,
1500:                                           InitialAlign, DL);
1501:       }
1502:       return getArgumentAlignment(CB, Arg.Ty, ArgI + 1, DL);
1503:     }();
1504:
1505:     const unsigned TySize = DL.getTypeAllocSize(ETy);
1506:     assert((!IsByVal || TySize == ArgOuts[0].Flags.getByValSize()) &&
1507:            "type size mismatch");
1508:
1509:     const SDValue ArgDeclare = [&]() {
1510:       if (IsVAArg)
1511:         return VADeclareParam;
1512:
1513:       if (IsByVal || shouldPassAsArray(Arg.Ty))
1514:         return MakeDeclareArrayParam(ParamSymbol, ArgAlign, TySize);
1515:
1516:       assert(ArgOuts.size() == 1 && "We must pass only one value as non-array");
1517:       assert((ArgOuts[0].VT.isInteger() || ArgOuts[0].VT.isFloatingPoint()) &&
1518:              "Only int and float types are supported as non-array arguments");
1519:
1520:       return MakeDeclareScalarParam(ParamSymbol, TySize);
1521:     }();
1522:
1523:     if (IsByVal) {
1524:       assert(ArgOutVals.size() == 1 && "We must pass only one value as byval");
1525:       SDValue SrcPtr = ArgOutVals[0];
1526:       const auto PointerInfo = refinePtrAS(SrcPtr, DAG, DL, *this);
1527:       const Align BaseSrcAlign = ArgOuts[0].Flags.getNonZeroByValAlign();
1528:
1529:       if (IsVAArg)
1530:         VAOffset = alignTo(VAOffset, ArgAlign);
1531:
1532:       SmallVector<EVT, 4> ValueVTs, MemVTs;
1533:       SmallVector<TypeSize, 4> Offsets;
1534:       ComputeValueVTs(*this, DL, ETy, ValueVTs, &MemVTs, &Offsets);
1535:
1536:       unsigned J = 0;
1537:       const auto VI = VectorizePTXValueVTs(MemVTs, Offsets, ArgAlign, IsVAArg);
1538:       for (const unsigned NumElts : VI) {
1539:         EVT LoadVT = getVectorizedVT(MemVTs[J], NumElts, Ctx);
1540:         Align SrcAlign = commonAlignment(BaseSrcAlign, Offsets[J]);
```
- EN: This range implements operational logic in helpers such as getCALLSEQ_START, getValue, promoteScalarArgumentSize, GetI32, translating backend policy into executable code.
- CN: 这一段实现了 getCALLSEQ_START、getValue、promoteScalarArgumentSize、GetI32 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1541-1680
```cpp
1541:         SDValue SrcAddr = DAG.getObjectPtrOffset(dl, SrcPtr, Offsets[J]);
1542:         SDValue SrcLoad =
1543:             DAG.getLoad(LoadVT, dl, CallChain, SrcAddr, PointerInfo, SrcAlign);
1544:
1545:         TypeSize ParamOffset = Offsets[J].getWithIncrement(VAOffset);
1546:         Align ParamAlign = commonAlignment(ArgAlign, ParamOffset);
1547:         SDValue ParamAddr =
1548:             DAG.getObjectPtrOffset(dl, ParamSymbol, ParamOffset);
1549:         SDValue StoreParam = DAG.getStore(
1550:             ArgDeclare, dl, SrcLoad, ParamAddr,
1551:             MachinePointerInfo(NVPTX::AddressSpace::DeviceParam), ParamAlign);
1552:         CallPrereqs.push_back(StoreParam);
1553:
1554:         J += NumElts;
1555:       }
1556:       if (IsVAArg)
1557:         VAOffset += TySize;
1558:     } else {
1559:       SmallVector<EVT, 16> VTs;
1560:       SmallVector<uint64_t, 16> Offsets;
1561:       ComputePTXValueVTs(*this, DL, Ctx, CLI.CallConv, Arg.Ty, VTs, Offsets,
1562:                          VAOffset);
1563:       assert(VTs.size() == Offsets.size() && "Size mismatch");
1564:       assert(VTs.size() == ArgOuts.size() && "Size mismatch");
1565:
1566:       // PTX Interoperability Guide 3.3(A): [Integer] Values shorter
1567:       // than 32-bits are sign extended or zero extended, depending on
1568:       // whether they are signed or unsigned types. This case applies
1569:       // only to scalar parameters and not to aggregate values.
1570:       const bool ExtendIntegerParam =
1571:           Arg.Ty->isIntegerTy() && DL.getTypeAllocSizeInBits(Arg.Ty) < 32;
1572:
1573:       const auto GetStoredValue = [&](const unsigned I) {
1574:         SDValue StVal = ArgOutVals[I];
1575:         assert(promoteScalarIntegerPTX(StVal.getValueType()) ==
1576:                    StVal.getValueType() &&
1577:                "OutVal type should always be legal");
1578:
1579:         const EVT VTI = promoteScalarIntegerPTX(VTs[I]);
1580:         const EVT StoreVT =
1581:             ExtendIntegerParam ? MVT::i32 : (VTI == MVT::i1 ? MVT::i8 : VTI);
1582:
1583:         return correctParamType(StVal, StoreVT, ArgOuts[I].Flags, DAG, dl);
1584:       };
1585:
1586:       unsigned J = 0;
1587:       const auto VI = VectorizePTXValueVTs(VTs, Offsets, ArgAlign, IsVAArg);
1588:       for (const unsigned NumElts : VI) {
1589:         const EVT EltVT = promoteScalarIntegerPTX(VTs[J]);
1590:
1591:         unsigned Offset;
1592:         if (IsVAArg) {
1593:           // TODO: We may need to support vector types that can be passed
1594:           // as scalars in variadic arguments.
1595:           assert(NumElts == 1 &&
1596:                  "Vectorization should be disabled for vaargs.");
1597:
1598:           // Align each part of the variadic argument to their type.
1599:           VAOffset = alignTo(VAOffset, DAG.getEVTAlign(EltVT));
1600:           Offset = VAOffset;
1601:
1602:           const EVT TheStoreType = ExtendIntegerParam ? MVT::i32 : EltVT;
1603:           VAOffset += DL.getTypeAllocSize(TheStoreType.getTypeForEVT(Ctx));
1604:         } else {
1605:           assert(VAOffset == 0 && "VAOffset must be 0 for non-VA args");
1606:           Offset = Offsets[J];
1607:         }
1608:
1609:         SDValue Ptr =
1610:             DAG.getObjectPtrOffset(dl, ParamSymbol, TypeSize::getFixed(Offset));
1611:
1612:         const MaybeAlign CurrentAlign = ExtendIntegerParam
1613:                                             ? MaybeAlign(std::nullopt)
1614:                                             : commonAlignment(ArgAlign, Offset);
1615:
1616:         SDValue Val =
1617:             getBuildVectorizedValue(NumElts, dl, DAG, [&](unsigned K) {
1618:               return GetStoredValue(J + K);
1619:             });
1620:
1621:         SDValue StoreParam = DAG.getStore(
1622:             ArgDeclare, dl, Val, Ptr,
1623:             MachinePointerInfo(NVPTX::AddressSpace::DeviceParam), CurrentAlign);
1624:         CallPrereqs.push_back(StoreParam);
1625:
1626:         J += NumElts;
1627:       }
1628:     }
1629:   }
1630:
1631:   // Handle Result
1632:   if (!Ins.empty()) {
1633:     const SDValue RetSymbol = DAG.getExternalSymbol("retval0", MVT::i32);
1634:     const unsigned ResultSize = DL.getTypeAllocSize(RetTy);
1635:     if (shouldPassAsArray(RetTy)) {
1636:       const Align RetAlign = getArgumentAlignment(CB, RetTy, 0, DL);
1637:       MakeDeclareArrayParam(RetSymbol, RetAlign, ResultSize);
1638:     } else {
1639:       MakeDeclareScalarParam(RetSymbol, ResultSize);
1640:     }
1641:   }
1642:
1643:   // Set the size of the vararg param byte array if the callee is a variadic
1644:   // function and the variadic part is not empty.
1645:   if (VADeclareParam) {
1646:     SDValue DeclareParamOps[] = {VADeclareParam.getOperand(0),
1647:                                  VADeclareParam.getOperand(1),
1648:                                  VADeclareParam.getOperand(2), GetI32(VAOffset),
1649:                                  VADeclareParam.getOperand(4)};
1650:     DAG.MorphNodeTo(VADeclareParam.getNode(), VADeclareParam.getOpcode(),
1651:                     VADeclareParam->getVTList(), DeclareParamOps);
1652:   }
1653:
1654:   const auto *Func = dyn_cast<GlobalAddressSDNode>(Callee.getNode());
1655:   const auto *CalleeF = Func ? dyn_cast<Function>(Func->getGlobal()) : nullptr;
1656:
1657:   // If the type of the callsite does not match that of the function, convert
1658:   // the callsite to an indirect call.
1659:   const bool ConvertToIndirectCall =
1660:       CalleeF && CB->getFunctionType() != CalleeF->getFunctionType();
1661:
1662:   // Both indirect calls and libcalls have nullptr Func. In order to distinguish
1663:   // between them we must rely on the call site value which is valid for
1664:   // indirect calls but is always null for libcalls.
1665:   const bool IsIndirectCall = (!Func && CB) || ConvertToIndirectCall;
1666:
1667:   if (isa<ExternalSymbolSDNode>(Callee)) {
1668:     Function* CalleeFunc = nullptr;
1669:
1670:     // Try to find the callee in the current module.
1671:     Callee = DAG.getSymbolFunctionGlobalAddress(Callee, &CalleeFunc);
1672:     assert(CalleeFunc != nullptr && "Libcall callee must be set.");
1673:
1674:     // Set the "libcall callee" attribute to indicate that the function
1675:     // must always have a declaration.
1676:     CalleeFunc->addFnAttr("nvptx-libcall-callee", "true");
1677:   }
1678:
1679:   if (IsIndirectCall) {
1680:     // This is indirect function call case : PTX requires a prototype of the
```
- EN: This range implements operational logic in helpers such as getObjectPtrOffset, getLoad, getWithIncrement, commonAlignment, translating backend policy into executable code.
- CN: 这一段实现了 getObjectPtrOffset、getLoad、getWithIncrement、commonAlignment 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1681-1820
```cpp
1681:     // form
1682:     // proto_0 : .callprototype(.param .b32 _) _ (.param .b32 _);
1683:     // to be emitted, and the label has to used as the last arg of call
1684:     // instruction.
1685:     // The prototype is embedded in a string and put as the operand for a
1686:     // CallPrototype SDNode which will print out to the value of the string.
1687:     const bool HasVAArgs = CLI.IsVarArg && (CLI.Args.size() > CLI.NumFixedArgs);
1688:     std::string Proto =
1689:         getPrototype(DL, RetTy, Args, CLI.Outs,
1690:                      HasVAArgs ? std::optional(FirstVAArg) : std::nullopt, *CB,
1691:                      UniqueCallSite);
1692:     const char *ProtoStr = nvTM->getStrPool().save(Proto).data();
1693:     const SDValue PrototypeDeclare = DAG.getNode(
1694:         NVPTXISD::CallPrototype, dl, MVT::Other,
1695:         {StartChain, DAG.getTargetExternalSymbol(ProtoStr, MVT::i32)});
1696:     CallPrereqs.push_back(PrototypeDeclare);
1697:   }
1698:
1699:   const bool IsUnknownIntrinsic =
1700:       CalleeF && CalleeF->isIntrinsic() &&
1701:       CalleeF->getIntrinsicID() == Intrinsic::not_intrinsic;
1702:   if (IsUnknownIntrinsic) {
1703:     DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
1704:         DAG.getMachineFunction().getFunction(),
1705:         "call to unknown intrinsic '" + CalleeF->getName() +
1706:             "' cannot be lowered by the NVPTX backend",
1707:         dl.getDebugLoc()));
1708:   }
1709:
1710:   const unsigned Proto = IsIndirectCall ? UniqueCallSite : 0;
1711:   const unsigned NumArgs =
1712:       std::min<unsigned>(CLI.NumFixedArgs + 1, Args.size());
1713:   /// CALL(Chain, IsConvergent, IsIndirectCall/IsUniform, NumReturns,
1714:   ///      NumParams, Callee, Proto)
1715:   const SDValue CallToken = DAG.getTokenFactor(dl, CallPrereqs);
1716:   const SDValue Call = DAG.getNode(
1717:       NVPTXISD::CALL, dl, MVT::Other,
1718:       {CallToken, GetI32(CLI.IsConvergent), GetI32(IsIndirectCall),
1719:        GetI32(Ins.empty() ? 0 : 1), GetI32(NumArgs), Callee, GetI32(Proto)});
1720:
1721:   SmallVector<SDValue, 16> LoadChains{Call};
1722:   SmallVector<SDValue, 16> ProxyRegOps;
1723:   if (!Ins.empty()) {
1724:     SmallVector<EVT, 16> VTs;
1725:     SmallVector<uint64_t, 16> Offsets;
1726:     ComputePTXValueVTs(*this, DL, Ctx, CLI.CallConv, RetTy, VTs, Offsets);
1727:     assert(VTs.size() == Ins.size() && "Bad value decomposition");
1728:
1729:     const Align RetAlign = getArgumentAlignment(CB, RetTy, 0, DL);
1730:     const SDValue RetSymbol = DAG.getExternalSymbol("retval0", MVT::i32);
1731:
1732:     // PTX Interoperability Guide 3.3(A): [Integer] Values shorter than
1733:     // 32-bits are sign extended or zero extended, depending on whether
1734:     // they are signed or unsigned types.
1735:     const bool ExtendIntegerRetVal =
1736:         RetTy->isIntegerTy() && DL.getTypeAllocSizeInBits(RetTy) < 32;
1737:
1738:     unsigned I = 0;
1739:     const auto VI = VectorizePTXValueVTs(VTs, Offsets, RetAlign);
1740:     for (const unsigned NumElts : VI) {
1741:       const MaybeAlign CurrentAlign =
1742:           ExtendIntegerRetVal ? MaybeAlign(std::nullopt)
1743:                               : commonAlignment(RetAlign, Offsets[I]);
1744:
1745:       const EVT VTI = promoteScalarIntegerPTX(VTs[I]);
1746:       const EVT LoadVT =
1747:           ExtendIntegerRetVal ? MVT::i32 : (VTI == MVT::i1 ? MVT::i8 : VTI);
1748:       const EVT VecVT = getVectorizedVT(LoadVT, NumElts, Ctx);
1749:       SDValue Ptr =
1750:           DAG.getObjectPtrOffset(dl, RetSymbol, TypeSize::getFixed(Offsets[I]));
1751:
1752:       SDValue R = DAG.getLoad(
1753:           VecVT, dl, Call, Ptr,
1754:           MachinePointerInfo(NVPTX::AddressSpace::DeviceParam), CurrentAlign);
1755:
1756:       LoadChains.push_back(R.getValue(1));
1757:       for (const unsigned J : llvm::seq(NumElts))
1758:         ProxyRegOps.push_back(getExtractVectorizedValue(R, J, LoadVT, dl, DAG));
1759:       I += NumElts;
1760:     }
1761:   }
1762:
1763:   const SDValue EndToken = DAG.getTokenFactor(dl, LoadChains);
1764:   const SDValue CallEnd = DAG.getCALLSEQ_END(EndToken, UniqueCallSite,
1765:                                              UniqueCallSite + 1, SDValue(), dl);
1766:
1767:   // Append ProxyReg instructions to the chain to make sure that `callseq_end`
1768:   // will not get lost. Otherwise, during libcalls expansion, the nodes can become
1769:   // dangling.
1770:   for (const auto [I, Reg] : llvm::enumerate(ProxyRegOps)) {
1771:     SDValue Proxy =
1772:         DAG.getNode(NVPTXISD::ProxyReg, dl, Reg.getValueType(), {CallEnd, Reg});
1773:     SDValue Ret = correctParamType(Proxy, Ins[I].VT, Ins[I].Flags, DAG, dl);
1774:     InVals.push_back(Ret);
1775:   }
1776:
1777:   // set IsTailCall to false for now, until we figure out how to express
1778:   // tail call optimization in PTX
1779:   CLI.IsTailCall = false;
1780:   return CallEnd;
1781: }
1782:
1783: SDValue NVPTXTargetLowering::LowerDYNAMIC_STACKALLOC(SDValue Op,
1784:                                                      SelectionDAG &DAG) const {
1785:
1786:   if (STI.getPTXVersion() < 73 || STI.getSmVersion() < 52) {
1787:     const Function &Fn = DAG.getMachineFunction().getFunction();
1788:
1789:     DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
1790:         Fn,
1791:         "Support for dynamic alloca introduced in PTX ISA version 7.3 and "
1792:         "requires target sm_52.",
1793:         SDLoc(Op).getDebugLoc()));
1794:     auto Ops = {DAG.getConstant(0, SDLoc(), Op.getValueType()),
1795:                 Op.getOperand(0)};
1796:     return DAG.getMergeValues(Ops, SDLoc());
1797:   }
1798:
1799:   SDLoc DL(Op.getNode());
1800:   SDValue Chain = Op.getOperand(0);
1801:   SDValue Size = Op.getOperand(1);
1802:   uint64_t Align = Op.getConstantOperandVal(2);
1803:
1804:   // The alignment on a ISD::DYNAMIC_STACKALLOC node may be 0 to indicate that
1805:   // the default stack alignment should be used.
1806:   if (Align == 0)
1807:     Align = DAG.getSubtarget().getFrameLowering()->getStackAlign().value();
1808:
1809:   // The size for ptx alloca instruction is 64-bit for m64 and 32-bit for m32.
1810:   const MVT LocalVT = getPointerTy(DAG.getDataLayout(), ADDRESS_SPACE_LOCAL);
1811:
1812:   SDValue Alloc =
1813:       DAG.getNode(NVPTXISD::DYNAMIC_STACKALLOC, DL, {LocalVT, MVT::Other},
1814:                   {Chain, DAG.getZExtOrTrunc(Size, DL, LocalVT),
1815:                    DAG.getTargetConstant(Align, DL, MVT::i32)});
1816:
1817:   SDValue ASC = DAG.getAddrSpaceCast(
1818:       DL, Op.getValueType(), Alloc, ADDRESS_SPACE_LOCAL, ADDRESS_SPACE_GENERIC);
1819:
1820:   return DAG.getMergeValues({ASC, SDValue(Alloc.getNode(), 1)}, DL);
```
- EN: This range implements operational logic in helpers such as size, getStrPool, getTargetExternalSymbol, push_back, translating backend policy into executable code.
- CN: 这一段实现了 size、getStrPool、getTargetExternalSymbol、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1821-1960
```cpp
1821: }
1822:
1823: SDValue NVPTXTargetLowering::LowerSTACKRESTORE(SDValue Op,
1824:                                                SelectionDAG &DAG) const {
1825:   SDLoc DL(Op.getNode());
1826:   if (STI.getPTXVersion() < 73 || STI.getSmVersion() < 52) {
1827:     const Function &Fn = DAG.getMachineFunction().getFunction();
1828:
1829:     DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
1830:         Fn,
1831:         "Support for stackrestore requires PTX ISA version >= 7.3 and target "
1832:         ">= sm_52.",
1833:         DL.getDebugLoc()));
1834:     return Op.getOperand(0);
1835:   }
1836:
1837:   const MVT LocalVT = getPointerTy(DAG.getDataLayout(), ADDRESS_SPACE_LOCAL);
1838:   SDValue Chain = Op.getOperand(0);
1839:   SDValue Ptr = Op.getOperand(1);
1840:   SDValue ASC = DAG.getAddrSpaceCast(DL, LocalVT, Ptr, ADDRESS_SPACE_GENERIC,
1841:                                      ADDRESS_SPACE_LOCAL);
1842:   return DAG.getNode(NVPTXISD::STACKRESTORE, DL, MVT::Other, {Chain, ASC});
1843: }
1844:
1845: SDValue NVPTXTargetLowering::LowerSTACKSAVE(SDValue Op,
1846:                                             SelectionDAG &DAG) const {
1847:   SDLoc DL(Op.getNode());
1848:   if (STI.getPTXVersion() < 73 || STI.getSmVersion() < 52) {
1849:     const Function &Fn = DAG.getMachineFunction().getFunction();
1850:
1851:     DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
1852:         Fn,
1853:         "Support for stacksave requires PTX ISA version >= 7.3 and target >= "
1854:         "sm_52.",
1855:         DL.getDebugLoc()));
1856:     auto Ops = {DAG.getConstant(0, DL, Op.getValueType()), Op.getOperand(0)};
1857:     return DAG.getMergeValues(Ops, DL);
1858:   }
1859:
1860:   const MVT LocalVT = getPointerTy(DAG.getDataLayout(), ADDRESS_SPACE_LOCAL);
1861:   SDValue Chain = Op.getOperand(0);
1862:   SDValue SS =
1863:       DAG.getNode(NVPTXISD::STACKSAVE, DL, {LocalVT, MVT::Other}, Chain);
1864:   SDValue ASC = DAG.getAddrSpaceCast(
1865:       DL, Op.getValueType(), SS, ADDRESS_SPACE_LOCAL, ADDRESS_SPACE_GENERIC);
1866:   return DAG.getMergeValues({ASC, SDValue(SS.getNode(), 1)}, DL);
1867: }
1868:
1869: // By default CONCAT_VECTORS is lowered by ExpandVectorBuildThroughStack()
1870: // (see LegalizeDAG.cpp). This is slow and uses local memory.
1871: // We use extract/insert/build vector just as what LegalizeOp() does in llvm 2.5
1872: SDValue
1873: NVPTXTargetLowering::LowerCONCAT_VECTORS(SDValue Op, SelectionDAG &DAG) const {
1874:   SDNode *Node = Op.getNode();
1875:   SDLoc dl(Node);
1876:   SmallVector<SDValue, 8> Ops;
1877:   unsigned NumOperands = Node->getNumOperands();
1878:   for (unsigned i = 0; i < NumOperands; ++i) {
1879:     SDValue SubOp = Node->getOperand(i);
1880:     EVT VVT = SubOp.getNode()->getValueType(0);
1881:     EVT EltVT = VVT.getVectorElementType();
1882:     unsigned NumSubElem = VVT.getVectorNumElements();
1883:     for (unsigned j = 0; j < NumSubElem; ++j) {
1884:       Ops.push_back(DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, EltVT, SubOp,
1885:                                 DAG.getIntPtrConstant(j, dl)));
1886:     }
1887:   }
1888:   return DAG.getBuildVector(Node->getValueType(0), dl, Ops);
1889: }
1890:
1891: static SDValue getPRMT(SDValue A, SDValue B, SDValue Selector, SDLoc DL,
1892:                        SelectionDAG &DAG,
1893:                        unsigned Mode = NVPTX::PTXPrmtMode::NONE) {
1894:   assert(A.getValueType() == MVT::i32 && B.getValueType() == MVT::i32 &&
1895:          Selector.getValueType() == MVT::i32 && "PRMT must have i32 operands");
1896:   return DAG.getNode(NVPTXISD::PRMT, DL, MVT::i32,
1897:                      {A, B, Selector, DAG.getConstant(Mode, DL, MVT::i32)});
1898: }
1899:
1900: static SDValue getPRMT(SDValue A, SDValue B, uint64_t Selector, SDLoc DL,
1901:                        SelectionDAG &DAG,
1902:                        unsigned Mode = NVPTX::PTXPrmtMode::NONE) {
1903:   return getPRMT(A, B, DAG.getConstant(Selector, DL, MVT::i32), DL, DAG, Mode);
1904: }
1905:
1906: /// Reduces the elements using the scalar operations provided. The operations
1907: /// are sorted descending in number of inputs they take. The flags on the
1908: /// original reduction operation will be propagated to each scalar operation.
1909: /// Nearby elements are grouped in tree reduction, unlike the shuffle reduction
1910: /// used in ExpandReductions and SelectionDAG.
1911: static SDValue buildTreeReduction(
1912:     const SmallVector<SDValue> &Elements, EVT EltTy,
1913:     ArrayRef<std::pair<unsigned /*NodeType*/, unsigned /*NumInputs*/>> Ops,
1914:     const SDLoc &DL, const SDNodeFlags Flags, SelectionDAG &DAG) {
1915:   // Build the reduction tree at each level, starting with all the elements.
1916:   SmallVector<SDValue> Level = Elements;
1917:
1918:   unsigned OpIdx = 0;
1919:   while (Level.size() > 1) {
1920:     // Try to reduce this level using the current operator.
1921:     const auto [Op, NumInputs] = Ops[OpIdx];
1922:
1923:     // Build the next level by partially reducing all elements.
1924:     SmallVector<SDValue> ReducedLevel;
1925:     unsigned I = 0, E = Level.size();
1926:     for (; I + NumInputs <= E; I += NumInputs) {
1927:       // Reduce elements in groups of [NumInputs], as much as possible.
1928:       ReducedLevel.push_back(DAG.getNode(
1929:           Op, DL, EltTy, ArrayRef<SDValue>(Level).slice(I, NumInputs), Flags));
1930:     }
1931:
1932:     if (I < E) {
1933:       // Handle leftover elements.
1934:
1935:       if (ReducedLevel.empty()) {
1936:         // We didn't reduce anything at this level. We need to pick a smaller
1937:         // operator.
1938:         ++OpIdx;
1939:         assert(OpIdx < Ops.size() && "no smaller operators for reduction");
1940:         continue;
1941:       }
1942:
1943:       // We reduced some things but there's still more left, meaning the
1944:       // operator's number of inputs doesn't evenly divide this level size. Move
1945:       // these elements to the next level.
1946:       for (; I < E; ++I)
1947:         ReducedLevel.push_back(Level[I]);
1948:     }
1949:
1950:     // Process the next level.
1951:     Level = ReducedLevel;
1952:   }
1953:
1954:   return *Level.begin();
1955: }
1956:
1957: // Get scalar reduction opcode
1958: static ISD::NodeType getScalarOpcodeForReduction(unsigned ReductionOpcode) {
1959:   switch (ReductionOpcode) {
1960:   case ISD::VECREDUCE_FMAX:
```
- EN: This range implements operational logic in helpers such as DL, getMachineFunction, getDebugLoc, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 DL、getMachineFunction、getDebugLoc、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1961-2100
```cpp
1961:     return ISD::FMAXNUM;
1962:   case ISD::VECREDUCE_FMIN:
1963:     return ISD::FMINNUM;
1964:   case ISD::VECREDUCE_FMAXIMUM:
1965:     return ISD::FMAXIMUM;
1966:   case ISD::VECREDUCE_FMINIMUM:
1967:     return ISD::FMINIMUM;
1968:   default:
1969:     llvm_unreachable("unhandled reduction opcode");
1970:   }
1971: }
1972:
1973: /// Get 3-input scalar reduction opcode
1974: static std::optional<unsigned>
1975: getScalar3OpcodeForReduction(unsigned ReductionOpcode) {
1976:   switch (ReductionOpcode) {
1977:   case ISD::VECREDUCE_FMAX:
1978:     return NVPTXISD::FMAXNUM3;
1979:   case ISD::VECREDUCE_FMIN:
1980:     return NVPTXISD::FMINNUM3;
1981:   case ISD::VECREDUCE_FMAXIMUM:
1982:     return NVPTXISD::FMAXIMUM3;
1983:   case ISD::VECREDUCE_FMINIMUM:
1984:     return NVPTXISD::FMINIMUM3;
1985:   default:
1986:     return std::nullopt;
1987:   }
1988: }
1989:
1990: /// Lower reductions to either a sequence of operations or a tree if
1991: /// reassociations are allowed. This method will use larger operations like
1992: /// max3/min3 when the target supports them.
1993: SDValue NVPTXTargetLowering::LowerVECREDUCE(SDValue Op,
1994:                                             SelectionDAG &DAG) const {
1995:   SDLoc DL(Op);
1996:   const SDNodeFlags Flags = Op->getFlags();
1997:   SDValue Vector = Op.getOperand(0);
1998:
1999:   const unsigned Opcode = Op->getOpcode();
2000:   const EVT EltTy = Vector.getValueType().getVectorElementType();
2001:
2002:   // Whether we can use 3-input min/max when expanding the reduction.
2003:   const bool CanUseMinMax3 =
2004:       EltTy == MVT::f32 && STI.getSmVersion() >= 100 &&
2005:       STI.getPTXVersion() >= 88 &&
2006:       (Opcode == ISD::VECREDUCE_FMAX || Opcode == ISD::VECREDUCE_FMIN ||
2007:        Opcode == ISD::VECREDUCE_FMAXIMUM || Opcode == ISD::VECREDUCE_FMINIMUM);
2008:
2009:   // A list of SDNode opcodes with equivalent semantics, sorted descending by
2010:   // number of inputs they take.
2011:   SmallVector<std::pair<unsigned /*Op*/, unsigned /*NumIn*/>, 2> ScalarOps;
2012:
2013:   if (auto Opcode3Elem = getScalar3OpcodeForReduction(Opcode);
2014:       CanUseMinMax3 && Opcode3Elem)
2015:     ScalarOps.push_back({*Opcode3Elem, 3});
2016:   ScalarOps.push_back({getScalarOpcodeForReduction(Opcode), 2});
2017:
2018:   SmallVector<SDValue> Elements;
2019:   DAG.ExtractVectorElements(Vector, Elements);
2020:
2021:   return buildTreeReduction(Elements, EltTy, ScalarOps, DL, Flags, DAG);
2022: }
2023:
2024: SDValue NVPTXTargetLowering::LowerBITCAST(SDValue Op, SelectionDAG &DAG) const {
2025:   // Handle bitcasting from v2i8 without hitting the default promotion
2026:   // strategy which goes through stack memory.
2027:   EVT FromVT = Op->getOperand(0)->getValueType(0);
2028:   if (FromVT != MVT::v2i8) {
2029:     return Op;
2030:   }
2031:
2032:   // Pack vector elements into i16 and bitcast to final type
2033:   SDLoc DL(Op);
2034:   SDValue Vec0 = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i8,
2035:                              Op->getOperand(0), DAG.getIntPtrConstant(0, DL));
2036:   SDValue Vec1 = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i8,
2037:                              Op->getOperand(0), DAG.getIntPtrConstant(1, DL));
2038:   SDValue Extend0 = DAG.getNode(ISD::ZERO_EXTEND, DL, MVT::i16, Vec0);
2039:   SDValue Extend1 = DAG.getNode(ISD::ZERO_EXTEND, DL, MVT::i16, Vec1);
2040:   SDValue Const8 = DAG.getConstant(8, DL, MVT::i16);
2041:   SDValue AsInt = DAG.getNode(
2042:       ISD::OR, DL, MVT::i16,
2043:       {Extend0, DAG.getNode(ISD::SHL, DL, MVT::i16, {Extend1, Const8})});
2044:   EVT ToVT = Op->getValueType(0);
2045:   return DAG.getBitcast(ToVT, AsInt);
2046: }
2047:
2048: // We can init constant f16x2/v2i16/v4i8 with a single .b32 move.  Normally it
2049: // would get lowered as two constant loads and vector-packing move.
2050: // Instead we want just a constant move:
2051: //        mov.b32         %r2, 0x40003C00
2052: SDValue NVPTXTargetLowering::LowerBUILD_VECTOR(SDValue Op,
2053:                                                SelectionDAG &DAG) const {
2054:   EVT VT = Op->getValueType(0);
2055:   if (!(NVPTX::isPackedVectorTy(VT) && VT.is32BitVector()))
2056:     return Op;
2057:   SDLoc DL(Op);
2058:
2059:   if (!llvm::all_of(Op->ops(), [](SDValue Operand) {
2060:         return Operand->isUndef() || isa<ConstantSDNode>(Operand) ||
2061:                isa<ConstantFPSDNode>(Operand);
2062:       })) {
2063:     if (VT != MVT::v4i8)
2064:       return Op;
2065:     // Lower non-const v4i8 vector as byte-wise constructed i32, which allows us
2066:     // to optimize calculation of constant parts.
2067:     auto GetPRMT = [&](const SDValue Left, const SDValue Right, bool Cast,
2068:                        uint64_t SelectionValue) -> SDValue {
2069:       SDValue L = Left;
2070:       SDValue R = Right;
2071:       if (Cast) {
2072:         L = DAG.getAnyExtOrTrunc(L, DL, MVT::i32);
2073:         R = DAG.getAnyExtOrTrunc(R, DL, MVT::i32);
2074:       }
2075:       return getPRMT(L, R, SelectionValue, DL, DAG);
2076:     };
2077:     auto PRMT__10 = GetPRMT(Op->getOperand(0), Op->getOperand(1), true, 0x3340);
2078:     auto PRMT__32 = GetPRMT(Op->getOperand(2), Op->getOperand(3), true, 0x3340);
2079:     auto PRMT3210 = GetPRMT(PRMT__10, PRMT__32, false, 0x5410);
2080:     return DAG.getBitcast(VT, PRMT3210);
2081:   }
2082:
2083:   // Get value or the Nth operand as an APInt(32). Undef values treated as 0.
2084:   auto GetOperand = [](SDValue Op, int N) -> APInt {
2085:     const SDValue &Operand = Op->getOperand(N);
2086:     EVT VT = Op->getValueType(0);
2087:     if (Operand->isUndef())
2088:       return APInt(32, 0);
2089:     APInt Value;
2090:     if (VT == MVT::v2f16 || VT == MVT::v2bf16)
2091:       Value = cast<ConstantFPSDNode>(Operand)->getValueAPF().bitcastToAPInt();
2092:     else if (VT == MVT::v2i16 || VT == MVT::v4i8)
2093:       Value = Operand->getAsAPIntVal();
2094:     else
2095:       llvm_unreachable("Unsupported type");
2096:     // i8 values are carried around as i16, so we need to zero out upper bits,
2097:     // so they do not get in the way of combining individual byte values
2098:     if (VT == MVT::v4i8)
2099:       Value = Value.trunc(8);
2100:     return Value.zext(32);
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, getScalar3OpcodeForReduction, DL, getFlags, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、getScalar3OpcodeForReduction、DL、getFlags 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2101-2240
```cpp
2101:   };
2102:
2103:   // Construct a 32-bit constant by shifting into place smaller values
2104:   // (elements of the vector type VT).
2105:   // For example, if VT has 2 elements, then N == 2:
2106:   //   ShiftAmount = 32 / N = 16
2107:   //   Value |= Op0 (b16) << 0
2108:   //   Value |= Op1 (b16) << 16
2109:   // If N == 4:
2110:   //   ShiftAmount = 32 / N = 8
2111:   //   Value |= Op0 (b8) << 0
2112:   //   Value |= Op1 (b8) << 8
2113:   //   Value |= Op2 (b8) << 16
2114:   //   Value |= Op3 (b8) << 24
2115:   // ...etc
2116:   APInt Value(32, 0);
2117:   const unsigned NumElements = VT.getVectorNumElements();
2118:   assert(32 % NumElements == 0 && "must evenly divide bit length");
2119:   const unsigned ShiftAmount = 32 / NumElements;
2120:   for (unsigned ElementNo : seq(NumElements))
2121:     Value |= GetOperand(Op, ElementNo).shl(ElementNo * ShiftAmount);
2122:   SDValue Const = DAG.getConstant(Value, DL, MVT::i32);
2123:   return DAG.getNode(ISD::BITCAST, DL, Op->getValueType(0), Const);
2124: }
2125:
2126: SDValue NVPTXTargetLowering::LowerEXTRACT_VECTOR_ELT(SDValue Op,
2127:                                                      SelectionDAG &DAG) const {
2128:   SDValue Index = Op->getOperand(1);
2129:   SDValue Vector = Op->getOperand(0);
2130:   SDLoc DL(Op);
2131:   EVT VectorVT = Vector.getValueType();
2132:
2133:   if (VectorVT == MVT::v4i8) {
2134:     SDValue Selector = DAG.getNode(ISD::OR, DL, MVT::i32,
2135:                                    DAG.getZExtOrTrunc(Index, DL, MVT::i32),
2136:                                    DAG.getConstant(0x7770, DL, MVT::i32));
2137:     SDValue PRMT = getPRMT(DAG.getBitcast(MVT::i32, Vector),
2138:                            DAG.getConstant(0, DL, MVT::i32), Selector, DL, DAG);
2139:     SDValue Ext = DAG.getAnyExtOrTrunc(PRMT, DL, Op->getValueType(0));
2140:     SDNodeFlags Flags;
2141:     Flags.setNoSignedWrap(Ext.getScalarValueSizeInBits() > 8);
2142:     Flags.setNoUnsignedWrap(Ext.getScalarValueSizeInBits() >= 8);
2143:     Ext->setFlags(Flags);
2144:     return Ext;
2145:   }
2146:
2147:   // Constant index will be matched by tablegen.
2148:   if (isa<ConstantSDNode>(Index.getNode()))
2149:     return Op;
2150:
2151:   // Extract individual elements and select one of them.
2152:   assert(NVPTX::isPackedVectorTy(VectorVT) &&
2153:          VectorVT.getVectorNumElements() == 2 && "Unexpected vector type.");
2154:   EVT EltVT = VectorVT.getVectorElementType();
2155:
2156:   SDLoc dl(Op.getNode());
2157:   SDValue E0 = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, EltVT, Vector,
2158:                            DAG.getIntPtrConstant(0, dl));
2159:   SDValue E1 = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, EltVT, Vector,
2160:                            DAG.getIntPtrConstant(1, dl));
2161:   return DAG.getSelectCC(dl, Index, DAG.getIntPtrConstant(0, dl), E0, E1,
2162:                          ISD::CondCode::SETEQ);
2163: }
2164:
2165: SDValue NVPTXTargetLowering::LowerINSERT_VECTOR_ELT(SDValue Op,
2166:                                                     SelectionDAG &DAG) const {
2167:   SDValue Vector = Op->getOperand(0);
2168:   EVT VectorVT = Vector.getValueType();
2169:
2170:   if (VectorVT != MVT::v4i8)
2171:     return Op;
2172:   SDLoc DL(Op);
2173:   SDValue Value = Op->getOperand(1);
2174:   if (Value->isUndef())
2175:     return Vector;
2176:
2177:   SDValue Index = Op->getOperand(2);
2178:
2179:   SDValue BFI =
2180:       DAG.getNode(NVPTXISD::BFI, DL, MVT::i32,
2181:                   {DAG.getZExtOrTrunc(Value, DL, MVT::i32), Vector,
2182:                    DAG.getNode(ISD::MUL, DL, MVT::i32,
2183:                                DAG.getZExtOrTrunc(Index, DL, MVT::i32),
2184:                                DAG.getConstant(8, DL, MVT::i32)),
2185:                    DAG.getConstant(8, DL, MVT::i32)});
2186:   return DAG.getNode(ISD::BITCAST, DL, Op->getValueType(0), BFI);
2187: }
2188:
2189: SDValue NVPTXTargetLowering::LowerVECTOR_SHUFFLE(SDValue Op,
2190:                                                  SelectionDAG &DAG) const {
2191:   SDValue V1 = Op.getOperand(0);
2192:   EVT VectorVT = V1.getValueType();
2193:   if (VectorVT != MVT::v4i8 || Op.getValueType() != MVT::v4i8)
2194:     return Op;
2195:
2196:   // Lower shuffle to PRMT instruction.
2197:   const ShuffleVectorSDNode *SVN = cast<ShuffleVectorSDNode>(Op.getNode());
2198:   SDValue V2 = Op.getOperand(1);
2199:   uint32_t Selector = 0;
2200:   for (auto I : llvm::enumerate(SVN->getMask())) {
2201:     if (I.value() != -1) // -1 is a placeholder for undef.
2202:       Selector |= (I.value() << (I.index() * 4));
2203:   }
2204:
2205:   SDLoc DL(Op);
2206:   SDValue PRMT = getPRMT(DAG.getBitcast(MVT::i32, V1),
2207:                          DAG.getBitcast(MVT::i32, V2), Selector, DL, DAG);
2208:   return DAG.getBitcast(Op.getValueType(), PRMT);
2209: }
2210: /// LowerShiftRightParts - Lower SRL_PARTS, SRA_PARTS, which
2211: /// 1) returns two i32 values and take a 2 x i32 value to shift plus a shift
2212: ///    amount, or
2213: /// 2) returns two i64 values and take a 2 x i64 value to shift plus a shift
2214: ///    amount.
2215: SDValue NVPTXTargetLowering::LowerShiftRightParts(SDValue Op,
2216:                                                   SelectionDAG &DAG) const {
2217:   assert(Op.getNumOperands() == 3 && "Not a double-shift!");
2218:   assert(Op.getOpcode() == ISD::SRA_PARTS || Op.getOpcode() == ISD::SRL_PARTS);
2219:
2220:   EVT VT = Op.getValueType();
2221:   unsigned VTBits = VT.getSizeInBits();
2222:   SDLoc dl(Op);
2223:   SDValue ShOpLo = Op.getOperand(0);
2224:   SDValue ShOpHi = Op.getOperand(1);
2225:   SDValue ShAmt  = Op.getOperand(2);
2226:   unsigned Opc = (Op.getOpcode() == ISD::SRA_PARTS) ? ISD::SRA : ISD::SRL;
2227:
2228:   if (VTBits == 32 && STI.getSmVersion() >= 35) {
2229:     // For 32bit and sm35, we can use the funnel shift 'shf' instruction.
2230:     // {dHi, dLo} = {aHi, aLo} >> Amt
2231:     //   dHi = aHi >> Amt
2232:     //   dLo = shf.r.clamp aLo, aHi, Amt
2233:
2234:     SDValue Hi = DAG.getNode(Opc, dl, VT, ShOpHi, ShAmt);
2235:     SDValue Lo =
2236:         DAG.getNode(NVPTXISD::FSHR_CLAMP, dl, VT, ShOpHi, ShOpLo, ShAmt);
2237:
2238:     SDValue Ops[2] = { Lo, Hi };
2239:     return DAG.getMergeValues(Ops, dl);
2240:   }
```
- EN: This range implements operational logic in helpers such as Value, getVectorNumElements, assert, GetOperand, translating backend policy into executable code.
- CN: 这一段实现了 Value、getVectorNumElements、assert、GetOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2241-2380
```cpp
2241:   else {
2242:     // {dHi, dLo} = {aHi, aLo} >> Amt
2243:     // - if (Amt>=size) then
2244:     //      dLo = aHi >> (Amt-size)
2245:     //      dHi = aHi >> Amt (this is either all 0 or all 1)
2246:     //   else
2247:     //      dLo = (aLo >>logic Amt) | (aHi << (size-Amt))
2248:     //      dHi = aHi >> Amt
2249:
2250:     SDValue RevShAmt = DAG.getNode(ISD::SUB, dl, MVT::i32,
2251:                                    DAG.getConstant(VTBits, dl, MVT::i32),
2252:                                    ShAmt);
2253:     SDValue Tmp1 = DAG.getNode(ISD::SRL, dl, VT, ShOpLo, ShAmt);
2254:     SDValue ExtraShAmt = DAG.getNode(ISD::SUB, dl, MVT::i32, ShAmt,
2255:                                      DAG.getConstant(VTBits, dl, MVT::i32));
2256:     SDValue Tmp2 = DAG.getNode(ISD::SHL, dl, VT, ShOpHi, RevShAmt);
2257:     SDValue FalseVal = DAG.getNode(ISD::OR, dl, VT, Tmp1, Tmp2);
2258:     SDValue TrueVal = DAG.getNode(Opc, dl, VT, ShOpHi, ExtraShAmt);
2259:
2260:     SDValue Cmp = DAG.getSetCC(dl, MVT::i1, ShAmt,
2261:                                DAG.getConstant(VTBits, dl, MVT::i32),
2262:                                ISD::SETGE);
2263:     SDValue Hi = DAG.getNode(Opc, dl, VT, ShOpHi, ShAmt);
2264:     SDValue Lo = DAG.getNode(ISD::SELECT, dl, VT, Cmp, TrueVal, FalseVal);
2265:
2266:     SDValue Ops[2] = { Lo, Hi };
2267:     return DAG.getMergeValues(Ops, dl);
2268:   }
2269: }
2270:
2271: /// LowerShiftLeftParts - Lower SHL_PARTS, which
2272: /// 1) returns two i32 values and take a 2 x i32 value to shift plus a shift
2273: ///    amount, or
2274: /// 2) returns two i64 values and take a 2 x i64 value to shift plus a shift
2275: ///    amount.
2276: SDValue NVPTXTargetLowering::LowerShiftLeftParts(SDValue Op,
2277:                                                  SelectionDAG &DAG) const {
2278:   assert(Op.getNumOperands() == 3 && "Not a double-shift!");
2279:   assert(Op.getOpcode() == ISD::SHL_PARTS);
2280:
2281:   EVT VT = Op.getValueType();
2282:   unsigned VTBits = VT.getSizeInBits();
2283:   SDLoc dl(Op);
2284:   SDValue ShOpLo = Op.getOperand(0);
2285:   SDValue ShOpHi = Op.getOperand(1);
2286:   SDValue ShAmt  = Op.getOperand(2);
2287:
2288:   if (VTBits == 32 && STI.getSmVersion() >= 35) {
2289:     // For 32bit and sm35, we can use the funnel shift 'shf' instruction.
2290:     // {dHi, dLo} = {aHi, aLo} << Amt
2291:     //   dHi = shf.l.clamp aLo, aHi, Amt
2292:     //   dLo = aLo << Amt
2293:
2294:     SDValue Hi =
2295:         DAG.getNode(NVPTXISD::FSHL_CLAMP, dl, VT, ShOpHi, ShOpLo, ShAmt);
2296:     SDValue Lo = DAG.getNode(ISD::SHL, dl, VT, ShOpLo, ShAmt);
2297:
2298:     SDValue Ops[2] = { Lo, Hi };
2299:     return DAG.getMergeValues(Ops, dl);
2300:   }
2301:   else {
2302:     // {dHi, dLo} = {aHi, aLo} << Amt
2303:     // - if (Amt>=size) then
2304:     //      dLo = aLo << Amt (all 0)
2305:     //      dLo = aLo << (Amt-size)
2306:     //   else
2307:     //      dLo = aLo << Amt
2308:     //      dHi = (aHi << Amt) | (aLo >> (size-Amt))
2309:
2310:     SDValue RevShAmt = DAG.getNode(ISD::SUB, dl, MVT::i32,
2311:                                    DAG.getConstant(VTBits, dl, MVT::i32),
2312:                                    ShAmt);
2313:     SDValue Tmp1 = DAG.getNode(ISD::SHL, dl, VT, ShOpHi, ShAmt);
2314:     SDValue ExtraShAmt = DAG.getNode(ISD::SUB, dl, MVT::i32, ShAmt,
2315:                                      DAG.getConstant(VTBits, dl, MVT::i32));
2316:     SDValue Tmp2 = DAG.getNode(ISD::SRL, dl, VT, ShOpLo, RevShAmt);
2317:     SDValue FalseVal = DAG.getNode(ISD::OR, dl, VT, Tmp1, Tmp2);
2318:     SDValue TrueVal = DAG.getNode(ISD::SHL, dl, VT, ShOpLo, ExtraShAmt);
2319:
2320:     SDValue Cmp = DAG.getSetCC(dl, MVT::i1, ShAmt,
2321:                                DAG.getConstant(VTBits, dl, MVT::i32),
2322:                                ISD::SETGE);
2323:     SDValue Lo = DAG.getNode(ISD::SHL, dl, VT, ShOpLo, ShAmt);
2324:     SDValue Hi = DAG.getNode(ISD::SELECT, dl, VT, Cmp, TrueVal, FalseVal);
2325:
2326:     SDValue Ops[2] = { Lo, Hi };
2327:     return DAG.getMergeValues(Ops, dl);
2328:   }
2329: }
2330:
2331: /// If the types match, convert the generic copysign to the NVPTXISD version,
2332: /// otherwise bail ensuring that mismatched cases are properly expaned.
2333: SDValue NVPTXTargetLowering::LowerFCOPYSIGN(SDValue Op,
2334:                                             SelectionDAG &DAG) const {
2335:   EVT VT = Op.getValueType();
2336:   SDLoc DL(Op);
2337:
2338:   SDValue In1 = Op.getOperand(0);
2339:   SDValue In2 = Op.getOperand(1);
2340:   EVT SrcVT = In2.getValueType();
2341:
2342:   if (!SrcVT.bitsEq(VT))
2343:     return SDValue();
2344:
2345:   return DAG.getNode(NVPTXISD::FCOPYSIGN, DL, VT, In1, In2);
2346: }
2347:
2348: SDValue NVPTXTargetLowering::LowerFROUND(SDValue Op, SelectionDAG &DAG) const {
2349:   EVT VT = Op.getValueType();
2350:
2351:   if (VT == MVT::f32)
2352:     return LowerFROUND32(Op, DAG);
2353:
2354:   if (VT == MVT::f64)
2355:     return LowerFROUND64(Op, DAG);
2356:
2357:   llvm_unreachable("unhandled type");
2358: }
2359:
2360: // This is the the rounding method used in CUDA libdevice in C like code:
2361: // float roundf(float A)
2362: // {
2363: //   float RoundedA = (float) (int) ( A > 0 ? (A + 0.5f) : (A - 0.5f));
2364: //   RoundedA = abs(A) > 0x1.0p23 ? A : RoundedA;
2365: //   return abs(A) < 0.5 ? (float)(int)A : RoundedA;
2366: // }
2367: SDValue NVPTXTargetLowering::LowerFROUND32(SDValue Op,
2368:                                            SelectionDAG &DAG) const {
2369:   SDLoc SL(Op);
2370:   SDValue A = Op.getOperand(0);
2371:   EVT VT = Op.getValueType();
2372:
2373:   SDValue AbsA = DAG.getNode(ISD::FABS, SL, VT, A);
2374:
2375:   // RoundedA = (float) (int) ( A > 0 ? (A + 0.5f) : (A - 0.5f))
2376:   SDValue Bitcast  = DAG.getNode(ISD::BITCAST, SL, MVT::i32, A);
2377:   const unsigned SignBitMask = 0x80000000;
2378:   SDValue Sign = DAG.getNode(ISD::AND, SL, MVT::i32, Bitcast,
2379:                              DAG.getConstant(SignBitMask, SL, MVT::i32));
2380:   const unsigned PointFiveInBits = 0x3F000000;
```
- EN: This range implements operational logic in helpers such as getNode, getConstant, getMergeValues, assert, translating backend policy into executable code.
- CN: 这一段实现了 getNode、getConstant、getMergeValues、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2381-2520
```cpp
2381:   SDValue PointFiveWithSignRaw =
2382:       DAG.getNode(ISD::OR, SL, MVT::i32, Sign,
2383:                   DAG.getConstant(PointFiveInBits, SL, MVT::i32));
2384:   SDValue PointFiveWithSign =
2385:       DAG.getNode(ISD::BITCAST, SL, VT, PointFiveWithSignRaw);
2386:   SDValue AdjustedA = DAG.getNode(ISD::FADD, SL, VT, A, PointFiveWithSign);
2387:   SDValue RoundedA = DAG.getNode(ISD::FTRUNC, SL, VT, AdjustedA);
2388:
2389:   // RoundedA = abs(A) > 0x1.0p23 ? A : RoundedA;
2390:   EVT SetCCVT = getSetCCResultType(DAG.getDataLayout(), *DAG.getContext(), VT);
2391:   SDValue IsLarge =
2392:       DAG.getSetCC(SL, SetCCVT, AbsA, DAG.getConstantFP(pow(2.0, 23.0), SL, VT),
2393:                    ISD::SETOGT);
2394:   RoundedA = DAG.getNode(ISD::SELECT, SL, VT, IsLarge, A, RoundedA);
2395:
2396:   // return abs(A) < 0.5 ? (float)(int)A : RoundedA;
2397:   SDValue IsSmall =DAG.getSetCC(SL, SetCCVT, AbsA,
2398:                                 DAG.getConstantFP(0.5, SL, VT), ISD::SETOLT);
2399:   SDValue RoundedAForSmallA = DAG.getNode(ISD::FTRUNC, SL, VT, A);
2400:   return DAG.getNode(ISD::SELECT, SL, VT, IsSmall, RoundedAForSmallA, RoundedA);
2401: }
2402:
2403: // The implementation of round(double) is similar to that of round(float) in
2404: // that they both separate the value range into three regions and use a method
2405: // specific to the region to round the values. However, round(double) first
2406: // calculates the round of the absolute value and then adds the sign back while
2407: // round(float) directly rounds the value with sign.
2408: SDValue NVPTXTargetLowering::LowerFROUND64(SDValue Op,
2409:                                            SelectionDAG &DAG) const {
2410:   SDLoc SL(Op);
2411:   SDValue A = Op.getOperand(0);
2412:   EVT VT = Op.getValueType();
2413:
2414:   SDValue AbsA = DAG.getNode(ISD::FABS, SL, VT, A);
2415:
2416:   // double RoundedA = (double) (int) (abs(A) + 0.5f);
2417:   SDValue AdjustedA = DAG.getNode(ISD::FADD, SL, VT, AbsA,
2418:                                   DAG.getConstantFP(0.5, SL, VT));
2419:   SDValue RoundedA = DAG.getNode(ISD::FTRUNC, SL, VT, AdjustedA);
2420:
2421:   // RoundedA = abs(A) < 0.5 ? (double)0 : RoundedA;
2422:   EVT SetCCVT = getSetCCResultType(DAG.getDataLayout(), *DAG.getContext(), VT);
2423:   SDValue IsSmall =DAG.getSetCC(SL, SetCCVT, AbsA,
2424:                                 DAG.getConstantFP(0.5, SL, VT), ISD::SETOLT);
2425:   RoundedA = DAG.getNode(ISD::SELECT, SL, VT, IsSmall,
2426:                          DAG.getConstantFP(0, SL, VT),
2427:                          RoundedA);
2428:
2429:   // Add sign to rounded_A
2430:   RoundedA = DAG.getNode(ISD::FCOPYSIGN, SL, VT, RoundedA, A);
2431:   DAG.getNode(ISD::FTRUNC, SL, VT, A);
2432:
2433:   // RoundedA = abs(A) > 0x1.0p52 ? A : RoundedA;
2434:   SDValue IsLarge =
2435:       DAG.getSetCC(SL, SetCCVT, AbsA, DAG.getConstantFP(pow(2.0, 52.0), SL, VT),
2436:                    ISD::SETOGT);
2437:   return DAG.getNode(ISD::SELECT, SL, VT, IsLarge, A, RoundedA);
2438: }
2439:
2440: static SDValue PromoteBinOpToF32(SDNode *N, SelectionDAG &DAG) {
2441:   EVT VT = N->getValueType(0);
2442:   EVT NVT = MVT::f32;
2443:   if (VT.isVector()) {
2444:     NVT = EVT::getVectorVT(*DAG.getContext(), NVT, VT.getVectorElementCount());
2445:   }
2446:   SDLoc DL(N);
2447:   SDValue Tmp0 = DAG.getFPExtendOrRound(N->getOperand(0), DL, NVT);
2448:   SDValue Tmp1 = DAG.getFPExtendOrRound(N->getOperand(1), DL, NVT);
2449:   SDValue Res = DAG.getNode(N->getOpcode(), DL, NVT, Tmp0, Tmp1, N->getFlags());
2450:   return DAG.getFPExtendOrRound(Res, DL, VT);
2451: }
2452:
2453: SDValue NVPTXTargetLowering::PromoteBinOpIfF32FTZ(SDValue Op,
2454:                                                   SelectionDAG &DAG) const {
2455:   if (useF32FTZ(DAG.getMachineFunction())) {
2456:     return PromoteBinOpToF32(Op.getNode(), DAG);
2457:   }
2458:   return Op;
2459: }
2460:
2461: SDValue NVPTXTargetLowering::LowerINT_TO_FP(SDValue Op,
2462:                                             SelectionDAG &DAG) const {
2463:   assert(STI.getSmVersion() < 90 || STI.getPTXVersion() < 78);
2464:
2465:   if (Op.getValueType() == MVT::bf16) {
2466:     SDLoc Loc(Op);
2467:     return DAG.getNode(
2468:         ISD::FP_ROUND, Loc, MVT::bf16,
2469:         DAG.getNode(Op.getOpcode(), Loc, MVT::f32, Op.getOperand(0)),
2470:         DAG.getIntPtrConstant(0, Loc, /*isTarget=*/true));
2471:   }
2472:
2473:   // Everything else is considered legal.
2474:   return Op;
2475: }
2476:
2477: SDValue NVPTXTargetLowering::LowerFP_TO_INT(SDValue Op,
2478:                                             SelectionDAG &DAG) const {
2479:   assert(STI.getSmVersion() < 90 || STI.getPTXVersion() < 78);
2480:
2481:   if (Op.getOperand(0).getValueType() == MVT::bf16) {
2482:     SDLoc Loc(Op);
2483:     return DAG.getNode(
2484:         Op.getOpcode(), Loc, Op.getValueType(),
2485:         DAG.getNode(ISD::FP_EXTEND, Loc, MVT::f32, Op.getOperand(0)));
2486:   }
2487:
2488:   // Everything else is considered legal.
2489:   return Op;
2490: }
2491:
2492: SDValue NVPTXTargetLowering::LowerFP_ROUND(SDValue Op,
2493:                                            SelectionDAG &DAG) const {
2494:   EVT NarrowVT = Op.getValueType();
2495:   SDValue Wide = Op.getOperand(0);
2496:   EVT WideVT = Wide.getValueType();
2497:   if (NarrowVT.getScalarType() == MVT::bf16) {
2498:     const TargetLowering *TLI = STI.getTargetLowering();
2499:     if (STI.getSmVersion() < 80 || STI.getPTXVersion() < 70) {
2500:       return TLI->expandFP_ROUND(Op.getNode(), DAG);
2501:     }
2502:     if (STI.getSmVersion() < 90 || STI.getPTXVersion() < 78) {
2503:       // This combination was the first to support f32 -> bf16.
2504:       if (STI.getSmVersion() >= 80 && STI.getPTXVersion() >= 70) {
2505:         if (WideVT.getScalarType() == MVT::f32) {
2506:           return Op;
2507:         }
2508:         if (WideVT.getScalarType() == MVT::f64) {
2509:           SDLoc Loc(Op);
2510:           // Round-inexact-to-odd f64 to f32, then do the final rounding using
2511:           // the hardware f32 -> bf16 instruction.
2512:           SDValue rod = TLI->expandRoundInexactToOdd(
2513:               WideVT.changeElementType(*DAG.getContext(), MVT::f32), Wide, Loc,
2514:               DAG);
2515:           return DAG.getFPExtendOrRound(rod, Loc, NarrowVT);
2516:         }
2517:       }
2518:       return TLI->expandFP_ROUND(Op.getNode(), DAG);
2519:     }
2520:   }
```
- EN: This range implements operational logic in helpers such as getConstant, getNode, getSetCCResultType, getConstantFP, translating backend policy into executable code.
- CN: 这一段实现了 getConstant、getNode、getSetCCResultType、getConstantFP 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2521-2660
```cpp
2521:
2522:   // Everything else is considered legal.
2523:   return Op;
2524: }
2525:
2526: SDValue NVPTXTargetLowering::LowerFP_EXTEND(SDValue Op,
2527:                                             SelectionDAG &DAG) const {
2528:   SDValue Narrow = Op.getOperand(0);
2529:   EVT NarrowVT = Narrow.getValueType();
2530:   EVT WideVT = Op.getValueType();
2531:   if (NarrowVT.getScalarType() == MVT::bf16) {
2532:     if (WideVT.getScalarType() == MVT::f32 &&
2533:         (STI.getSmVersion() < 80 || STI.getPTXVersion() < 71)) {
2534:       SDLoc Loc(Op);
2535:       return DAG.getNode(ISD::BF16_TO_FP, Loc, WideVT, Narrow);
2536:     }
2537:     if (WideVT.getScalarType() == MVT::f64 &&
2538:         (STI.getSmVersion() < 90 || STI.getPTXVersion() < 78)) {
2539:       EVT F32 = NarrowVT.changeElementType(*DAG.getContext(), MVT::f32);
2540:       SDLoc Loc(Op);
2541:       if (STI.getSmVersion() >= 80 && STI.getPTXVersion() >= 71) {
2542:         Op = DAG.getNode(ISD::FP_EXTEND, Loc, F32, Narrow);
2543:       } else {
2544:         Op = DAG.getNode(ISD::BF16_TO_FP, Loc, F32, Narrow);
2545:       }
2546:       return DAG.getNode(ISD::FP_EXTEND, Loc, WideVT, Op);
2547:     }
2548:   }
2549:
2550:   // Everything else is considered legal.
2551:   return Op;
2552: }
2553:
2554: static SDValue LowerVectorArith(SDValue Op, SelectionDAG &DAG) {
2555:   SDLoc DL(Op);
2556:   if (Op.getValueType() != MVT::v2i16)
2557:     return Op;
2558:   EVT EltVT = Op.getValueType().getVectorElementType();
2559:   SmallVector<SDValue> VecElements;
2560:   for (int I = 0, E = Op.getValueType().getVectorNumElements(); I < E; I++) {
2561:     SmallVector<SDValue> ScalarArgs;
2562:     llvm::transform(Op->ops(), std::back_inserter(ScalarArgs),
2563:                     [&](const SDUse &O) {
2564:                       return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT,
2565:                                          O.get(), DAG.getIntPtrConstant(I, DL));
2566:                     });
2567:     VecElements.push_back(DAG.getNode(Op.getOpcode(), DL, EltVT, ScalarArgs));
2568:   }
2569:   SDValue V =
2570:       DAG.getNode(ISD::BUILD_VECTOR, DL, Op.getValueType(), VecElements);
2571:   return V;
2572: }
2573:
2574: static SDValue lowerTcgen05St(SDValue Op, SelectionDAG &DAG,
2575:                               bool hasOffset = false) {
2576:   // skip lowering if the vector operand is already legalized
2577:   if (!Op->getOperand(hasOffset ? 4 : 3).getValueType().isVector())
2578:     return Op;
2579:
2580:   SDNode *N = Op.getNode();
2581:   SDLoc DL(N);
2582:   SmallVector<SDValue, 32> Ops;
2583:
2584:   // split the vector argument
2585:   for (size_t I = 0; I < N->getNumOperands(); I++) {
2586:     SDValue Val = N->getOperand(I);
2587:     EVT ValVT = Val.getValueType();
2588:     if (ValVT.isVector()) {
2589:       EVT EltVT = ValVT.getVectorElementType();
2590:       for (unsigned J = 0, NElts = ValVT.getVectorNumElements(); J < NElts; J++)
2591:         Ops.push_back(DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT, Val,
2592:                                   DAG.getIntPtrConstant(J, DL)));
2593:     } else
2594:       Ops.push_back(Val);
2595:   }
2596:
2597:   MemIntrinsicSDNode *MemSD = cast<MemIntrinsicSDNode>(N);
2598:   SDValue Tcgen05StNode =
2599:       DAG.getMemIntrinsicNode(ISD::INTRINSIC_VOID, DL, N->getVTList(), Ops,
2600:                               MemSD->getMemoryVT(), MemSD->getMemOperand());
2601:
2602:   return Tcgen05StNode;
2603: }
2604:
2605: static SDValue lowerBSWAP(SDValue Op, SelectionDAG &DAG) {
2606:   SDLoc DL(Op);
2607:   SDValue Src = Op.getOperand(0);
2608:   EVT VT = Op.getValueType();
2609:
2610:   switch (VT.getSimpleVT().SimpleTy) {
2611:   case MVT::i16: {
2612:     SDValue Extended = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i32, Src);
2613:     SDValue Swapped =
2614:         getPRMT(Extended, DAG.getConstant(0, DL, MVT::i32), 0x7701, DL, DAG);
2615:     return DAG.getNode(ISD::TRUNCATE, DL, MVT::i16, Swapped);
2616:   }
2617:   case MVT::i32: {
2618:     return getPRMT(Src, DAG.getConstant(0, DL, MVT::i32), 0x0123, DL, DAG);
2619:   }
2620:   case MVT::v2i16: {
2621:     SDValue Converted = DAG.getBitcast(MVT::i32, Src);
2622:     SDValue Swapped =
2623:         getPRMT(Converted, DAG.getConstant(0, DL, MVT::i32), 0x2301, DL, DAG);
2624:     return DAG.getNode(ISD::BITCAST, DL, MVT::v2i16, Swapped);
2625:   }
2626:   case MVT::i64: {
2627:     SDValue UnpackSrc =
2628:         DAG.getNode(NVPTXISD::UNPACK_VECTOR, DL, {MVT::i32, MVT::i32}, Src);
2629:     SDValue SwappedLow =
2630:         getPRMT(UnpackSrc.getValue(0), DAG.getConstant(0, DL, MVT::i32), 0x0123,
2631:                 DL, DAG);
2632:     SDValue SwappedHigh =
2633:         getPRMT(UnpackSrc.getValue(1), DAG.getConstant(0, DL, MVT::i32), 0x0123,
2634:                 DL, DAG);
2635:     return DAG.getNode(NVPTXISD::BUILD_VECTOR, DL, MVT::i64,
2636:                        {SwappedHigh, SwappedLow});
2637:   }
2638:   default:
2639:     llvm_unreachable("unsupported type for bswap");
2640:   }
2641: }
2642:
2643: static unsigned getTcgen05MMADisableOutputLane(unsigned IID) {
2644:   switch (IID) {
2645:   case Intrinsic::nvvm_tcgen05_mma_shared_disable_output_lane_cg1:
2646:     return NVPTXISD::TCGEN05_MMA_SHARED_DISABLE_OUTPUT_LANE_CG1;
2647:   case Intrinsic::nvvm_tcgen05_mma_shared_disable_output_lane_cg2:
2648:     return NVPTXISD::TCGEN05_MMA_SHARED_DISABLE_OUTPUT_LANE_CG2;
2649:   case Intrinsic::nvvm_tcgen05_mma_shared_scale_d_disable_output_lane_cg1:
2650:     return NVPTXISD::TCGEN05_MMA_SHARED_SCALE_D_DISABLE_OUTPUT_LANE_CG1;
2651:   case Intrinsic::nvvm_tcgen05_mma_shared_scale_d_disable_output_lane_cg2:
2652:     return NVPTXISD::TCGEN05_MMA_SHARED_SCALE_D_DISABLE_OUTPUT_LANE_CG2;
2653:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg1:
2654:     return NVPTXISD::TCGEN05_MMA_TENSOR_DISABLE_OUTPUT_LANE_CG1;
2655:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg2:
2656:     return NVPTXISD::TCGEN05_MMA_TENSOR_DISABLE_OUTPUT_LANE_CG2;
2657:   case Intrinsic::nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg1:
2658:     return NVPTXISD::TCGEN05_MMA_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG1;
2659:   case Intrinsic::nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg2:
2660:     return NVPTXISD::TCGEN05_MMA_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG2;
```
- EN: This range implements operational logic in helpers such as getOperand, getValueType, getSmVersion, Loc, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getValueType、getSmVersion、Loc 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2661-2800
```cpp
2661:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg1_ashift:
2662:     return NVPTXISD::TCGEN05_MMA_TENSOR_DISABLE_OUTPUT_LANE_CG1_ASHIFT;
2663:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg2_ashift:
2664:     return NVPTXISD::TCGEN05_MMA_TENSOR_DISABLE_OUTPUT_LANE_CG2_ASHIFT;
2665:   case Intrinsic::
2666:       nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg1_ashift:
2667:     return NVPTXISD::TCGEN05_MMA_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG1_ASHIFT;
2668:   case Intrinsic::
2669:       nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg2_ashift:
2670:     return NVPTXISD::TCGEN05_MMA_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG2_ASHIFT;
2671:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_disable_output_lane_cg1:
2672:     return NVPTXISD::TCGEN05_MMA_SP_SHARED_DISABLE_OUTPUT_LANE_CG1;
2673:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_disable_output_lane_cg2:
2674:     return NVPTXISD::TCGEN05_MMA_SP_SHARED_DISABLE_OUTPUT_LANE_CG2;
2675:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_scale_d_disable_output_lane_cg1:
2676:     return NVPTXISD::TCGEN05_MMA_SP_SHARED_SCALE_D_DISABLE_OUTPUT_LANE_CG1;
2677:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_scale_d_disable_output_lane_cg2:
2678:     return NVPTXISD::TCGEN05_MMA_SP_SHARED_SCALE_D_DISABLE_OUTPUT_LANE_CG2;
2679:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg1:
2680:     return NVPTXISD::TCGEN05_MMA_SP_TENSOR_DISABLE_OUTPUT_LANE_CG1;
2681:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg2:
2682:     return NVPTXISD::TCGEN05_MMA_SP_TENSOR_DISABLE_OUTPUT_LANE_CG2;
2683:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg1_ashift:
2684:     return NVPTXISD::TCGEN05_MMA_SP_TENSOR_DISABLE_OUTPUT_LANE_CG1_ASHIFT;
2685:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg2_ashift:
2686:     return NVPTXISD::TCGEN05_MMA_SP_TENSOR_DISABLE_OUTPUT_LANE_CG2_ASHIFT;
2687:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg1:
2688:     return NVPTXISD::TCGEN05_MMA_SP_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG1;
2689:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg2:
2690:     return NVPTXISD::TCGEN05_MMA_SP_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG2;
2691:   case Intrinsic::
2692:       nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg1_ashift:
2693:     return NVPTXISD::
2694:         TCGEN05_MMA_SP_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG1_ASHIFT;
2695:   case Intrinsic::
2696:       nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg2_ashift:
2697:     return NVPTXISD::
2698:         TCGEN05_MMA_SP_TENSOR_SCALE_D_DISABLE_OUTPUT_LANE_CG2_ASHIFT;
2699:   };
2700:   llvm_unreachable("unhandled tcgen05.mma.disable_output_lane intrinsic");
2701: }
2702:
2703: static SDValue LowerTcgen05MMADisableOutputLane(SDValue Op, SelectionDAG &DAG) {
2704:   SDNode *N = Op.getNode();
2705:   SDLoc DL(N);
2706:   unsigned IID = cast<ConstantSDNode>(N->getOperand(1))->getZExtValue();
2707:
2708:   SmallVector<SDValue, 16> Ops;
2709:   // split the vector argument
2710:   for (size_t I = 0; I < N->getNumOperands(); I++) {
2711:     if (I == 1)
2712:       continue; // skip IID
2713:     SDValue Val = N->getOperand(I);
2714:     EVT ValVT = Val.getValueType();
2715:     if (ValVT.isVector()) {
2716:       EVT EltVT = ValVT.getVectorElementType();
2717:       for (unsigned J = 0, NElts = ValVT.getVectorNumElements(); J < NElts; J++)
2718:         Ops.push_back(DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT, Val,
2719:                                   DAG.getIntPtrConstant(J, DL)));
2720:     } else
2721:       Ops.push_back(Val);
2722:   }
2723:
2724:   MemIntrinsicSDNode *MemSD = cast<MemIntrinsicSDNode>(N);
2725:   SDValue Tcgen05MMANode = DAG.getMemIntrinsicNode(
2726:       getTcgen05MMADisableOutputLane(IID), DL, N->getVTList(), Ops,
2727:       MemSD->getMemoryVT(), MemSD->getMemOperand());
2728:
2729:   return Tcgen05MMANode;
2730: }
2731:
2732: // Lower vector return type of tcgen05.ld intrinsics
2733: static std::optional<std::pair<SDValue, SDValue>>
2734: lowerTcgen05Ld(SDNode *N, SelectionDAG &DAG, bool HasOffset = false) {
2735:   SDLoc DL(N);
2736:   EVT ResVT = N->getValueType(0);
2737:   if (!ResVT.isVector())
2738:     return {}; // already legalized.
2739:
2740:   const unsigned NumElts = ResVT.getVectorNumElements();
2741:
2742:   // Create the return type of the instructions
2743:   SmallVector<EVT, 5> ListVTs;
2744:   for (unsigned i = 0; i < NumElts; ++i)
2745:     ListVTs.push_back(MVT::i32);
2746:
2747:   ListVTs.push_back(N->getValueType(1)); // Chain
2748:
2749:   SDVTList ResVTs = DAG.getVTList(ListVTs);
2750:
2751:   SmallVector<SDValue, 8> Ops{N->getOperand(0), N->getOperand(1),
2752:                               N->getOperand(2)};
2753:
2754:   if (HasOffset) {
2755:     Ops.push_back(N->getOperand(3)); // offset
2756:     Ops.push_back(N->getOperand(4)); // Pack flag
2757:   } else
2758:     Ops.push_back(N->getOperand(3)); // Pack flag
2759:
2760:   MemIntrinsicSDNode *MemSD = cast<MemIntrinsicSDNode>(N);
2761:   SDValue NewNode =
2762:       DAG.getMemIntrinsicNode(ISD::INTRINSIC_W_CHAIN, DL, ResVTs, Ops,
2763:                               MemSD->getMemoryVT(), MemSD->getMemOperand());
2764:
2765:   // split the vector result
2766:   SmallVector<SDValue, 4> ScalarRes;
2767:   for (unsigned i = 0; i < NumElts; ++i) {
2768:     SDValue Res = NewNode.getValue(i);
2769:     ScalarRes.push_back(Res);
2770:   }
2771:
2772:   SDValue Chain = NewNode.getValue(NumElts);
2773:   SDValue BuildVector = DAG.getNode(ISD::BUILD_VECTOR, DL, ResVT, ScalarRes);
2774:   return {{BuildVector, Chain}};
2775: }
2776:
2777: static SDValue reportInvalidTensormapReplaceUsage(SDValue Op, SelectionDAG &DAG,
2778:                                                   unsigned Val) {
2779:   SDNode *N = Op.getNode();
2780:   SDLoc DL(N);
2781:
2782:   const Function &Fn = DAG.getMachineFunction().getFunction();
2783:
2784:   unsigned AS = 0;
2785:   if (auto *MemN = dyn_cast<MemIntrinsicSDNode>(N))
2786:     AS = MemN->getAddressSpace();
2787:   Type *PtrTy = PointerType::get(*DAG.getContext(), AS);
2788:   Module *M = DAG.getMachineFunction().getFunction().getParent();
2789:
2790:   DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
2791:       Fn,
2792:       "Intrinsic " +
2793:           Intrinsic::getName(N->getConstantOperandVal(1), {PtrTy}, M) +
2794:           " with value " + Twine(Val) +
2795:           " is not supported on the given target.",
2796:       DL.getDebugLoc()));
2797:   return Op.getOperand(0);
2798: }
2799:
2800: static SDValue lowerTensormapReplaceElemtype(SDValue Op, SelectionDAG &DAG) {
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, LowerTcgen05MMADisableOutputLane, getNode, DL, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、LowerTcgen05MMADisableOutputLane、getNode、DL 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2801-2940
```cpp
2801:   SDNode *N = Op.getNode();
2802:   SDLoc DL(N);
2803:
2804:   // immediate argument representing elemtype
2805:   unsigned Val = N->getConstantOperandVal(3);
2806:
2807:   if (!DAG.getSubtarget<NVPTXSubtarget>().hasTensormapReplaceElemtypeSupport(
2808:           Val))
2809:     return reportInvalidTensormapReplaceUsage(Op, DAG, Val);
2810:
2811:   return Op;
2812: }
2813:
2814: static SDValue lowerTensormapReplaceSwizzleMode(SDValue Op, SelectionDAG &DAG) {
2815:   SDNode *N = Op.getNode();
2816:   SDLoc DL(N);
2817:
2818:   // immediate argument representing swizzle mode
2819:   unsigned Val = N->getConstantOperandVal(3);
2820:
2821:   if (!DAG.getSubtarget<NVPTXSubtarget>().hasTensormapReplaceSwizzleModeSupport(
2822:           Val))
2823:     return reportInvalidTensormapReplaceUsage(Op, DAG, Val);
2824:
2825:   return Op;
2826: }
2827:
2828: static SDValue lowerIntrinsicVoid(SDValue Op, SelectionDAG &DAG) {
2829:   SDNode *N = Op.getNode();
2830:   SDValue Intrin = N->getOperand(1);
2831:
2832:   // Get the intrinsic ID
2833:   unsigned IntrinNo = cast<ConstantSDNode>(Intrin.getNode())->getZExtValue();
2834:   switch (IntrinNo) {
2835:   default:
2836:     break;
2837:   case Intrinsic::nvvm_tcgen05_st_16x64b_x2:
2838:   case Intrinsic::nvvm_tcgen05_st_16x64b_x4:
2839:   case Intrinsic::nvvm_tcgen05_st_16x64b_x8:
2840:   case Intrinsic::nvvm_tcgen05_st_16x64b_x16:
2841:   case Intrinsic::nvvm_tcgen05_st_16x64b_x32:
2842:   case Intrinsic::nvvm_tcgen05_st_16x64b_x128:
2843:   case Intrinsic::nvvm_tcgen05_st_16x128b_x1:
2844:   case Intrinsic::nvvm_tcgen05_st_16x128b_x2:
2845:   case Intrinsic::nvvm_tcgen05_st_16x128b_x4:
2846:   case Intrinsic::nvvm_tcgen05_st_16x128b_x8:
2847:   case Intrinsic::nvvm_tcgen05_st_16x128b_x16:
2848:   case Intrinsic::nvvm_tcgen05_st_16x128b_x32:
2849:   case Intrinsic::nvvm_tcgen05_st_16x128b_x64:
2850:   case Intrinsic::nvvm_tcgen05_st_16x256b_x1:
2851:   case Intrinsic::nvvm_tcgen05_st_16x256b_x2:
2852:   case Intrinsic::nvvm_tcgen05_st_16x256b_x4:
2853:   case Intrinsic::nvvm_tcgen05_st_16x256b_x8:
2854:   case Intrinsic::nvvm_tcgen05_st_16x256b_x16:
2855:   case Intrinsic::nvvm_tcgen05_st_16x256b_x32:
2856:   case Intrinsic::nvvm_tcgen05_st_32x32b_x2:
2857:   case Intrinsic::nvvm_tcgen05_st_32x32b_x4:
2858:   case Intrinsic::nvvm_tcgen05_st_32x32b_x8:
2859:   case Intrinsic::nvvm_tcgen05_st_32x32b_x16:
2860:   case Intrinsic::nvvm_tcgen05_st_32x32b_x32:
2861:   case Intrinsic::nvvm_tcgen05_st_16x64b_x64:
2862:   case Intrinsic::nvvm_tcgen05_st_32x32b_x64:
2863:   case Intrinsic::nvvm_tcgen05_st_32x32b_x128:
2864:     return lowerTcgen05St(Op, DAG);
2865:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x2:
2866:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x4:
2867:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x8:
2868:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x16:
2869:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x32:
2870:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x64:
2871:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x128:
2872:     return lowerTcgen05St(Op, DAG, /* hasOffset */ true);
2873:   case Intrinsic::nvvm_tcgen05_mma_shared_disable_output_lane_cg1:
2874:   case Intrinsic::nvvm_tcgen05_mma_shared_disable_output_lane_cg2:
2875:   case Intrinsic::nvvm_tcgen05_mma_shared_scale_d_disable_output_lane_cg1:
2876:   case Intrinsic::nvvm_tcgen05_mma_shared_scale_d_disable_output_lane_cg2:
2877:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_disable_output_lane_cg1:
2878:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_disable_output_lane_cg2:
2879:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_scale_d_disable_output_lane_cg1:
2880:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_scale_d_disable_output_lane_cg2:
2881:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg1:
2882:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg2:
2883:   case Intrinsic::nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg1:
2884:   case Intrinsic::nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg2:
2885:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg1:
2886:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg2:
2887:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg1:
2888:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg2:
2889:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg1_ashift:
2890:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg2_ashift:
2891:   case Intrinsic::
2892:       nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg1_ashift:
2893:   case Intrinsic::
2894:       nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg2_ashift:
2895:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg1_ashift:
2896:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg2_ashift:
2897:   case Intrinsic::
2898:       nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg1_ashift:
2899:   case Intrinsic::
2900:       nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg2_ashift:
2901:     return LowerTcgen05MMADisableOutputLane(Op, DAG);
2902:   case Intrinsic::nvvm_tensormap_replace_elemtype:
2903:     return lowerTensormapReplaceElemtype(Op, DAG);
2904:   case Intrinsic::nvvm_tensormap_replace_swizzle_mode:
2905:     return lowerTensormapReplaceSwizzleMode(Op, DAG);
2906:   }
2907:   return Op;
2908: }
2909:
2910: static SDValue LowerClusterLaunchControlQueryCancel(SDValue Op,
2911:                                                     SelectionDAG &DAG) {
2912:
2913:   SDNode *N = Op.getNode();
2914:   if (N->getOperand(1).getValueType() != MVT::i128) {
2915:     // return, if the operand is already lowered
2916:     return SDValue();
2917:   }
2918:
2919:   unsigned IID =
2920:       cast<ConstantSDNode>(N->getOperand(0).getNode())->getZExtValue();
2921:   auto Opcode = [&]() {
2922:     switch (IID) {
2923:     case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_is_canceled:
2924:       return NVPTXISD::CLUSTERLAUNCHCONTROL_QUERY_CANCEL_IS_CANCELED;
2925:     case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_x:
2926:       return NVPTXISD::CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID_X;
2927:     case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_y:
2928:       return NVPTXISD::CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID_Y;
2929:     case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_z:
2930:       return NVPTXISD::CLUSTERLAUNCHCONTROL_QUERY_CANCEL_GET_FIRST_CTAID_Z;
2931:     default:
2932:       llvm_unreachable("unsupported/unhandled intrinsic");
2933:     }
2934:   }();
2935:
2936:   SDLoc DL(N);
2937:   SDValue TryCancelResponse = N->getOperand(1);
2938:   SDValue Cast = DAG.getNode(ISD::BITCAST, DL, MVT::v2i64, TryCancelResponse);
2939:   SDValue TryCancelResponse0 =
2940:       DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i64, Cast,
```
- EN: This range implements operational logic in helpers such as getNode, DL, getConstantOperandVal, reportInvalidTensormapReplaceUsage, translating backend policy into executable code.
- CN: 这一段实现了 getNode、DL、getConstantOperandVal、reportInvalidTensormapReplaceUsage 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2941-3080
```cpp
2941:                   DAG.getIntPtrConstant(0, DL));
2942:   SDValue TryCancelResponse1 =
2943:       DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i64, Cast,
2944:                   DAG.getIntPtrConstant(1, DL));
2945:
2946:   return DAG.getNode(Opcode, DL, N->getVTList(),
2947:                      {TryCancelResponse0, TryCancelResponse1});
2948: }
2949:
2950: static SDValue lowerCvtRSIntrinsics(SDValue Op, SelectionDAG &DAG) {
2951:   SDNode *N = Op.getNode();
2952:   SDLoc DL(N);
2953:   SDValue F32Vec = N->getOperand(1);
2954:   SDValue RBits = N->getOperand(2);
2955:
2956:   unsigned IntrinsicID = N->getConstantOperandVal(0);
2957:
2958:   // Extract the 4 float elements from the vector
2959:   SmallVector<SDValue, 6> Ops;
2960:   for (unsigned i = 0; i < 4; ++i)
2961:     Ops.push_back(DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::f32, F32Vec,
2962:                               DAG.getIntPtrConstant(i, DL)));
2963:
2964:   using NVPTX::PTXCvtMode::CvtMode;
2965:
2966:   auto [OpCode, RetTy, CvtModeFlag] =
2967:       [&]() -> std::tuple<unsigned, MVT::SimpleValueType, uint32_t> {
2968:     switch (IntrinsicID) {
2969:     case Intrinsic::nvvm_f32x4_to_e4m3x4_rs_relu_satfinite:
2970:       return {NVPTXISD::CVT_E4M3X4_F32X4_RS_SF, MVT::v4i8,
2971:               CvtMode::RS | CvtMode::RELU_FLAG};
2972:     case Intrinsic::nvvm_f32x4_to_e4m3x4_rs_satfinite:
2973:       return {NVPTXISD::CVT_E4M3X4_F32X4_RS_SF, MVT::v4i8, CvtMode::RS};
2974:     case Intrinsic::nvvm_f32x4_to_e5m2x4_rs_relu_satfinite:
2975:       return {NVPTXISD::CVT_E5M2X4_F32X4_RS_SF, MVT::v4i8,
2976:               CvtMode::RS | CvtMode::RELU_FLAG};
2977:     case Intrinsic::nvvm_f32x4_to_e5m2x4_rs_satfinite:
2978:       return {NVPTXISD::CVT_E5M2X4_F32X4_RS_SF, MVT::v4i8, CvtMode::RS};
2979:     case Intrinsic::nvvm_f32x4_to_e2m3x4_rs_relu_satfinite:
2980:       return {NVPTXISD::CVT_E2M3X4_F32X4_RS_SF, MVT::v4i8,
2981:               CvtMode::RS | CvtMode::RELU_FLAG};
2982:     case Intrinsic::nvvm_f32x4_to_e2m3x4_rs_satfinite:
2983:       return {NVPTXISD::CVT_E2M3X4_F32X4_RS_SF, MVT::v4i8, CvtMode::RS};
2984:     case Intrinsic::nvvm_f32x4_to_e3m2x4_rs_relu_satfinite:
2985:       return {NVPTXISD::CVT_E3M2X4_F32X4_RS_SF, MVT::v4i8,
2986:               CvtMode::RS | CvtMode::RELU_FLAG};
2987:     case Intrinsic::nvvm_f32x4_to_e3m2x4_rs_satfinite:
2988:       return {NVPTXISD::CVT_E3M2X4_F32X4_RS_SF, MVT::v4i8, CvtMode::RS};
2989:     case Intrinsic::nvvm_f32x4_to_e2m1x4_rs_relu_satfinite:
2990:       return {NVPTXISD::CVT_E2M1X4_F32X4_RS_SF, MVT::i16,
2991:               CvtMode::RS | CvtMode::RELU_FLAG};
2992:     case Intrinsic::nvvm_f32x4_to_e2m1x4_rs_satfinite:
2993:       return {NVPTXISD::CVT_E2M1X4_F32X4_RS_SF, MVT::i16, CvtMode::RS};
2994:     default:
2995:       llvm_unreachable("unsupported/unhandled intrinsic");
2996:     }
2997:   }();
2998:
2999:   Ops.push_back(RBits);
3000:   Ops.push_back(DAG.getConstant(CvtModeFlag, DL, MVT::i32));
3001:
3002:   return DAG.getNode(OpCode, DL, RetTy, Ops);
3003: }
3004:
3005: static SDValue lowerPrmtIntrinsic(SDValue Op, SelectionDAG &DAG) {
3006:   const unsigned Mode = [&]() {
3007:     switch (Op->getConstantOperandVal(0)) {
3008:     case Intrinsic::nvvm_prmt:
3009:       return NVPTX::PTXPrmtMode::NONE;
3010:     case Intrinsic::nvvm_prmt_b4e:
3011:       return NVPTX::PTXPrmtMode::B4E;
3012:     case Intrinsic::nvvm_prmt_ecl:
3013:       return NVPTX::PTXPrmtMode::ECL;
3014:     case Intrinsic::nvvm_prmt_ecr:
3015:       return NVPTX::PTXPrmtMode::ECR;
3016:     case Intrinsic::nvvm_prmt_f4e:
3017:       return NVPTX::PTXPrmtMode::F4E;
3018:     case Intrinsic::nvvm_prmt_rc16:
3019:       return NVPTX::PTXPrmtMode::RC16;
3020:     case Intrinsic::nvvm_prmt_rc8:
3021:       return NVPTX::PTXPrmtMode::RC8;
3022:     default:
3023:       llvm_unreachable("unsupported/unhandled intrinsic");
3024:     }
3025:   }();
3026:   SDLoc DL(Op);
3027:   SDValue A = Op->getOperand(1);
3028:   SDValue B = Op.getNumOperands() == 4 ? Op.getOperand(2)
3029:                                        : DAG.getConstant(0, DL, MVT::i32);
3030:   SDValue Selector = (Op->op_end() - 1)->get();
3031:   return getPRMT(A, B, Selector, DL, DAG, Mode);
3032: }
3033:
3034: #define TCGEN05_LD_RED_INTR(SHAPE, NUM, TYPE)                                  \
3035:   Intrinsic::nvvm_tcgen05_ld_red_##SHAPE##_x##NUM##_##TYPE
3036:
3037: #define TCGEN05_LD_RED_INST(SHAPE, NUM, TYPE)                                  \
3038:   NVPTXISD::TCGEN05_LD_RED_##SHAPE##_X##NUM##_##TYPE
3039:
3040: static unsigned getTcgen05LdRedID(Intrinsic::ID IID) {
3041:   switch (IID) {
3042:   case TCGEN05_LD_RED_INTR(32x32b, 2, f32):
3043:     return TCGEN05_LD_RED_INST(32x32b, 2, F32);
3044:   case TCGEN05_LD_RED_INTR(32x32b, 4, f32):
3045:     return TCGEN05_LD_RED_INST(32x32b, 4, F32);
3046:   case TCGEN05_LD_RED_INTR(32x32b, 8, f32):
3047:     return TCGEN05_LD_RED_INST(32x32b, 8, F32);
3048:   case TCGEN05_LD_RED_INTR(32x32b, 16, f32):
3049:     return TCGEN05_LD_RED_INST(32x32b, 16, F32);
3050:   case TCGEN05_LD_RED_INTR(32x32b, 32, f32):
3051:     return TCGEN05_LD_RED_INST(32x32b, 32, F32);
3052:   case TCGEN05_LD_RED_INTR(32x32b, 64, f32):
3053:     return TCGEN05_LD_RED_INST(32x32b, 64, F32);
3054:   case TCGEN05_LD_RED_INTR(32x32b, 128, f32):
3055:     return TCGEN05_LD_RED_INST(32x32b, 128, F32);
3056:   case TCGEN05_LD_RED_INTR(16x32bx2, 2, f32):
3057:     return TCGEN05_LD_RED_INST(16x32bx2, 2, F32);
3058:   case TCGEN05_LD_RED_INTR(16x32bx2, 4, f32):
3059:     return TCGEN05_LD_RED_INST(16x32bx2, 4, F32);
3060:   case TCGEN05_LD_RED_INTR(16x32bx2, 8, f32):
3061:     return TCGEN05_LD_RED_INST(16x32bx2, 8, F32);
3062:   case TCGEN05_LD_RED_INTR(16x32bx2, 16, f32):
3063:     return TCGEN05_LD_RED_INST(16x32bx2, 16, F32);
3064:   case TCGEN05_LD_RED_INTR(16x32bx2, 32, f32):
3065:     return TCGEN05_LD_RED_INST(16x32bx2, 32, F32);
3066:   case TCGEN05_LD_RED_INTR(16x32bx2, 64, f32):
3067:     return TCGEN05_LD_RED_INST(16x32bx2, 64, F32);
3068:   case TCGEN05_LD_RED_INTR(16x32bx2, 128, f32):
3069:     return TCGEN05_LD_RED_INST(16x32bx2, 128, F32);
3070:   case TCGEN05_LD_RED_INTR(32x32b, 2, i32):
3071:     return TCGEN05_LD_RED_INST(32x32b, 2, I32);
3072:   case TCGEN05_LD_RED_INTR(32x32b, 4, i32):
3073:     return TCGEN05_LD_RED_INST(32x32b, 4, I32);
3074:   case TCGEN05_LD_RED_INTR(32x32b, 8, i32):
3075:     return TCGEN05_LD_RED_INST(32x32b, 8, I32);
3076:   case TCGEN05_LD_RED_INTR(32x32b, 16, i32):
3077:     return TCGEN05_LD_RED_INST(32x32b, 16, I32);
3078:   case TCGEN05_LD_RED_INTR(32x32b, 32, i32):
3079:     return TCGEN05_LD_RED_INST(32x32b, 32, I32);
3080:   case TCGEN05_LD_RED_INTR(32x32b, 64, i32):
```
- EN: This range implements operational logic in helpers such as getIntPtrConstant, lowerCvtRSIntrinsics, getNode, DL, translating backend policy into executable code.
- CN: 这一段实现了 getIntPtrConstant、lowerCvtRSIntrinsics、getNode、DL 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3081-3220
```cpp
3081:     return TCGEN05_LD_RED_INST(32x32b, 64, I32);
3082:   case TCGEN05_LD_RED_INTR(32x32b, 128, i32):
3083:     return TCGEN05_LD_RED_INST(32x32b, 128, I32);
3084:   case TCGEN05_LD_RED_INTR(16x32bx2, 2, i32):
3085:     return TCGEN05_LD_RED_INST(16x32bx2, 2, I32);
3086:   case TCGEN05_LD_RED_INTR(16x32bx2, 4, i32):
3087:     return TCGEN05_LD_RED_INST(16x32bx2, 4, I32);
3088:   case TCGEN05_LD_RED_INTR(16x32bx2, 8, i32):
3089:     return TCGEN05_LD_RED_INST(16x32bx2, 8, I32);
3090:   case TCGEN05_LD_RED_INTR(16x32bx2, 16, i32):
3091:     return TCGEN05_LD_RED_INST(16x32bx2, 16, I32);
3092:   case TCGEN05_LD_RED_INTR(16x32bx2, 32, i32):
3093:     return TCGEN05_LD_RED_INST(16x32bx2, 32, I32);
3094:   case TCGEN05_LD_RED_INTR(16x32bx2, 64, i32):
3095:     return TCGEN05_LD_RED_INST(16x32bx2, 64, I32);
3096:   case TCGEN05_LD_RED_INTR(16x32bx2, 128, i32):
3097:     return TCGEN05_LD_RED_INST(16x32bx2, 128, I32);
3098:   default:
3099:     llvm_unreachable("Invalid tcgen05.ld.red intrinsic ID");
3100:   }
3101: }
3102:
3103: // Lower vector return type of tcgen05.ld intrinsics
3104: static std::optional<std::tuple<SDValue, SDValue, SDValue>>
3105: lowerTcgen05LdRed(SDNode *N, SelectionDAG &DAG) {
3106:   SDLoc DL(N);
3107:   EVT ResVT = N->getValueType(0);
3108:   if (!ResVT.isVector())
3109:     return {}; // already legalized.
3110:
3111:   const unsigned NumElts = ResVT.getVectorNumElements();
3112:
3113:   // Create the return type of the instructions
3114:   // +1 represents the reduction value
3115:   SmallVector<EVT, 132> ListVTs{
3116:       NumElts + 1,
3117:       ResVT.getVectorElementType().isFloatingPoint() ? MVT::f32 : MVT::i32};
3118:
3119:   ListVTs.push_back(MVT::Other); // Chain
3120:
3121:   SDVTList ResVTs = DAG.getVTList(ListVTs);
3122:
3123:   // Prepare the Operands
3124:   SmallVector<SDValue, 8> Ops{N->getOperand(0)}; // Chain
3125:
3126:   // skip IID at index 1
3127:   for (unsigned i = 2; i < N->getNumOperands(); i++)
3128:     Ops.push_back(N->getOperand(i));
3129:
3130:   unsigned IID = cast<ConstantSDNode>(N->getOperand(1))->getZExtValue();
3131:   MemIntrinsicSDNode *MemSD = cast<MemIntrinsicSDNode>(N);
3132:   SDValue NewNode =
3133:       DAG.getMemIntrinsicNode(getTcgen05LdRedID(IID), DL, ResVTs, Ops,
3134:                               MemSD->getMemoryVT(), MemSD->getMemOperand());
3135:
3136:   // Split vector result
3137:   SmallVector<SDValue, 132> ScalarRes;
3138:   for (unsigned i = 0; i < NumElts; ++i) {
3139:     SDValue Res = NewNode.getValue(i);
3140:     ScalarRes.push_back(Res);
3141:   }
3142:
3143:   SDValue BuildVector = DAG.getNode(ISD::BUILD_VECTOR, DL, ResVT, ScalarRes);
3144:   SDValue RedResult = NewNode.getValue(NumElts);
3145:   SDValue Chain = NewNode.getValue(NumElts + 1);
3146:   return {{BuildVector, RedResult, Chain}};
3147: }
3148:
3149: static SDValue lowerIntrinsicWChain(SDValue Op, SelectionDAG &DAG) {
3150:   switch (Op->getConstantOperandVal(1)) {
3151:   default:
3152:     return Op;
3153:
3154:   // These tcgen05 intrinsics return a v2i32, which is legal, so we have to
3155:   // lower them through LowerOperation() instead of ReplaceNodeResults().
3156:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x2:
3157:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x1:
3158:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x2:
3159:     if (auto Res = lowerTcgen05Ld(Op.getNode(), DAG))
3160:       return DAG.getMergeValues({Res->first, Res->second}, SDLoc(Op));
3161:     return SDValue();
3162:
3163:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x2:
3164:     if (auto Res = lowerTcgen05Ld(Op.getNode(), DAG, /*HasOffset=*/true))
3165:       return DAG.getMergeValues({Res->first, Res->second}, SDLoc(Op));
3166:     return SDValue();
3167:
3168:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x2_f32:
3169:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x2_i32:
3170:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x2_f32:
3171:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x2_i32:
3172:     if (auto Res = lowerTcgen05LdRed(Op.getNode(), DAG))
3173:       return DAG.getMergeValues(
3174:           {std::get<0>(*Res), std::get<1>(*Res), std::get<2>(*Res)}, SDLoc(Op));
3175:     return SDValue();
3176:   }
3177: }
3178:
3179: static SDValue lowerIntrinsicWOChain(SDValue Op, SelectionDAG &DAG) {
3180:   switch (Op->getConstantOperandVal(0)) {
3181:   default:
3182:     return Op;
3183:   case Intrinsic::nvvm_prmt:
3184:   case Intrinsic::nvvm_prmt_b4e:
3185:   case Intrinsic::nvvm_prmt_ecl:
3186:   case Intrinsic::nvvm_prmt_ecr:
3187:   case Intrinsic::nvvm_prmt_f4e:
3188:   case Intrinsic::nvvm_prmt_rc16:
3189:   case Intrinsic::nvvm_prmt_rc8:
3190:     return lowerPrmtIntrinsic(Op, DAG);
3191:   case Intrinsic::nvvm_internal_addrspace_wrap:
3192:     return Op.getOperand(1);
3193:   case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_is_canceled:
3194:   case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_x:
3195:   case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_y:
3196:   case Intrinsic::nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_z:
3197:     return LowerClusterLaunchControlQueryCancel(Op, DAG);
3198:   case Intrinsic::nvvm_f32x4_to_e4m3x4_rs_satfinite:
3199:   case Intrinsic::nvvm_f32x4_to_e4m3x4_rs_relu_satfinite:
3200:   case Intrinsic::nvvm_f32x4_to_e5m2x4_rs_satfinite:
3201:   case Intrinsic::nvvm_f32x4_to_e5m2x4_rs_relu_satfinite:
3202:   case Intrinsic::nvvm_f32x4_to_e2m3x4_rs_satfinite:
3203:   case Intrinsic::nvvm_f32x4_to_e2m3x4_rs_relu_satfinite:
3204:   case Intrinsic::nvvm_f32x4_to_e3m2x4_rs_satfinite:
3205:   case Intrinsic::nvvm_f32x4_to_e3m2x4_rs_relu_satfinite:
3206:   case Intrinsic::nvvm_f32x4_to_e2m1x4_rs_satfinite:
3207:   case Intrinsic::nvvm_f32x4_to_e2m1x4_rs_relu_satfinite:
3208:     return lowerCvtRSIntrinsics(Op, DAG);
3209:   }
3210: }
3211:
3212: // In PTX 64-bit CTLZ and CTPOP are supported, but they return a 32-bit value.
3213: // Lower these into a node returning the correct type which is zero-extended
3214: // back to the correct size.
3215: static SDValue lowerCTLZCTPOP(SDValue Op, SelectionDAG &DAG) {
3216:   SDValue V = Op->getOperand(0);
3217:   assert(V.getValueType() == MVT::i64 &&
3218:          "Unexpected CTLZ/CTPOP type to legalize");
3219:
3220:   SDLoc DL(Op);
```
- EN: This range implements operational logic in helpers such as TCGEN05_LD_RED_INST, llvm_unreachable, lowerTcgen05LdRed, DL, translating backend policy into executable code.
- CN: 这一段实现了 TCGEN05_LD_RED_INST、llvm_unreachable、lowerTcgen05LdRed、DL 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3221-3360
```cpp
3221:   SDValue CT = DAG.getNode(Op->getOpcode(), DL, MVT::i32, V);
3222:   return DAG.getNode(ISD::ZERO_EXTEND, DL, MVT::i64, CT, SDNodeFlags::NonNeg);
3223: }
3224:
3225: static SDValue expandFSH64(SDValue A, SDValue B, SDValue ShiftAmount, SDLoc DL,
3226:                            unsigned Opcode, SelectionDAG &DAG) {
3227:   assert(A.getValueType() == MVT::i64 && B.getValueType() == MVT::i64);
3228:
3229:   const auto *AmtConst = dyn_cast<ConstantSDNode>(ShiftAmount);
3230:   if (!AmtConst)
3231:     return SDValue();
3232:   const auto Amt = AmtConst->getZExtValue() & 63;
3233:
3234:   SDValue UnpackA =
3235:       DAG.getNode(NVPTXISD::UNPACK_VECTOR, DL, {MVT::i32, MVT::i32}, A);
3236:   SDValue UnpackB =
3237:       DAG.getNode(NVPTXISD::UNPACK_VECTOR, DL, {MVT::i32, MVT::i32}, B);
3238:
3239:   // Arch is Little endiain: 0 = low bits, 1 = high bits
3240:   SDValue ALo = UnpackA.getValue(0);
3241:   SDValue AHi = UnpackA.getValue(1);
3242:   SDValue BLo = UnpackB.getValue(0);
3243:   SDValue BHi = UnpackB.getValue(1);
3244:
3245:   // The bitfeild consists of { AHi : ALo : BHi : BLo }
3246:   //
3247:   // * FSHL, Amt <  32 - The window will contain { AHi : ALo : BHi }
3248:   // * FSHL, Amt >= 32 - The window will contain { ALo : BHi : BLo }
3249:   // * FSHR, Amt <  32 - The window will contain { ALo : BHi : BLo }
3250:   // * FSHR, Amt >= 32 - The window will contain { AHi : ALo : BHi }
3251:   //
3252:   // Note that Amt = 0 and Amt = 32 are special cases where 32-bit funnel shifts
3253:   // are not needed at all. Amt = 0 is a no-op producing either A or B depending
3254:   // on the direction. Amt = 32 can be implemented by a packing and unpacking
3255:   // move to select and arrange the 32bit values. For simplicity, these cases
3256:   // are not handled here explicitly and instead we rely on DAGCombiner to
3257:   // remove the no-op funnel shifts we insert.
3258:   auto [High, Mid, Low] = ((Opcode == ISD::FSHL) == (Amt < 32))
3259:                               ? std::make_tuple(AHi, ALo, BHi)
3260:                               : std::make_tuple(ALo, BHi, BLo);
3261:
3262:   SDValue NewAmt = DAG.getConstant(Amt & 31, DL, MVT::i32);
3263:   SDValue RHi = DAG.getNode(Opcode, DL, MVT::i32, {High, Mid, NewAmt});
3264:   SDValue RLo = DAG.getNode(Opcode, DL, MVT::i32, {Mid, Low, NewAmt});
3265:
3266:   return DAG.getNode(NVPTXISD::BUILD_VECTOR, DL, MVT::i64, {RLo, RHi});
3267: }
3268:
3269: static SDValue lowerFSH(SDValue Op, SelectionDAG &DAG) {
3270:   return expandFSH64(Op->getOperand(0), Op->getOperand(1), Op->getOperand(2),
3271:                      SDLoc(Op), Op->getOpcode(), DAG);
3272: }
3273:
3274: static SDValue lowerROT(SDValue Op, SelectionDAG &DAG) {
3275:   unsigned Opcode = Op->getOpcode() == ISD::ROTL ? ISD::FSHL : ISD::FSHR;
3276:   return expandFSH64(Op->getOperand(0), Op->getOperand(0), Op->getOperand(1),
3277:                      SDLoc(Op), Opcode, DAG);
3278: }
3279:
3280: static SDValue lowerFREM(SDValue Op, SelectionDAG &DAG) {
3281:   // Lower (frem x, y) into (sub x, (mul (ftrunc (div x, y)) y)),
3282:   // i.e. "poor man's fmod()". When y is infinite, x is returned. This matches
3283:   // the semantics of LLVM's frem.
3284:   SDLoc DL(Op);
3285:   SDValue X = Op->getOperand(0);
3286:   SDValue Y = Op->getOperand(1);
3287:   EVT Ty = Op.getValueType();
3288:   SDNodeFlags Flags = Op->getFlags();
3289:
3290:   SDValue Div = DAG.getNode(ISD::FDIV, DL, Ty, X, Y, Flags);
3291:   SDValue Trunc = DAG.getNode(ISD::FTRUNC, DL, Ty, Div, Flags);
3292:   SDValue Mul = DAG.getNode(ISD::FMUL, DL, Ty, Trunc, Y,
3293:                             Flags | SDNodeFlags::AllowContract);
3294:   SDValue Sub = DAG.getNode(ISD::FSUB, DL, Ty, X, Mul,
3295:                             Flags | SDNodeFlags::AllowContract);
3296:
3297:   if (Flags.hasNoInfs())
3298:     return Sub;
3299:
3300:   // If Y is infinite, return X
3301:   SDValue AbsY = DAG.getNode(ISD::FABS, DL, Ty, Y);
3302:   SDValue Inf =
3303:       DAG.getConstantFP(APFloat::getInf(Ty.getFltSemantics()), DL, Ty);
3304:   SDValue IsInf = DAG.getSetCC(DL, MVT::i1, AbsY, Inf, ISD::SETEQ);
3305:   return DAG.getSelect(DL, Ty, IsInf, X, Sub);
3306: }
3307:
3308: static SDValue lowerSELECT(SDValue Op, SelectionDAG &DAG) {
3309:   assert(Op.getValueType() == MVT::i1 && "Custom lowering enabled only for i1");
3310:
3311:   SDValue Cond = Op->getOperand(0);
3312:   SDValue TrueVal = Op->getOperand(1);
3313:   SDValue FalseVal = Op->getOperand(2);
3314:   SDLoc DL(Op);
3315:
3316:   // If both operands are truncated, we push the select through the truncates.
3317:   if (TrueVal.getOpcode() == ISD::TRUNCATE &&
3318:       FalseVal.getOpcode() == ISD::TRUNCATE) {
3319:     TrueVal = TrueVal.getOperand(0);
3320:     FalseVal = FalseVal.getOperand(0);
3321:
3322:     EVT VT = TrueVal.getSimpleValueType().bitsLE(FalseVal.getSimpleValueType())
3323:                  ? TrueVal.getValueType()
3324:                  : FalseVal.getValueType();
3325:     TrueVal = DAG.getAnyExtOrTrunc(TrueVal, DL, VT);
3326:     FalseVal = DAG.getAnyExtOrTrunc(FalseVal, DL, VT);
3327:     SDValue Select = DAG.getSelect(DL, VT, Cond, TrueVal, FalseVal);
3328:     return DAG.getNode(ISD::TRUNCATE, DL, MVT::i1, Select);
3329:   }
3330:
3331:   // Otherwise, expand the select into a series of logical operations. These
3332:   // often can be folded into other operations either by us or ptxas.
3333:   TrueVal = DAG.getFreeze(TrueVal);
3334:   FalseVal = DAG.getFreeze(FalseVal);
3335:   SDValue And1 = DAG.getNode(ISD::AND, DL, MVT::i1, Cond, TrueVal);
3336:   SDValue NotCond = DAG.getNOT(DL, Cond, MVT::i1);
3337:   SDValue And2 = DAG.getNode(ISD::AND, DL, MVT::i1, NotCond, FalseVal);
3338:   SDValue Or = DAG.getNode(ISD::OR, DL, MVT::i1, And1, And2);
3339:   return Or;
3340: }
3341:
3342: static SDValue lowerMSTORE(SDValue Op, SelectionDAG &DAG) {
3343:   SDNode *N = Op.getNode();
3344:
3345:   SDValue Chain = N->getOperand(0);
3346:   SDValue Val = N->getOperand(1);
3347:   SDValue BasePtr = N->getOperand(2);
3348:   SDValue Offset = N->getOperand(3);
3349:   SDValue Mask = N->getOperand(4);
3350:
3351:   SDLoc DL(N);
3352:   EVT ValVT = Val.getValueType();
3353:   MemSDNode *MemSD = cast<MemSDNode>(N);
3354:   assert(ValVT.isVector() && "Masked vector store must have vector type");
3355:   assert(MemSD->getAlign() >= DAG.getEVTAlign(ValVT) &&
3356:          "Unexpected alignment for masked store");
3357:
3358:   unsigned Opcode = 0;
3359:   switch (ValVT.getSimpleVT().SimpleTy) {
3360:   default:
```
- EN: This range implements operational logic in helpers such as getNode, assert, SDValue, getValue, translating backend policy into executable code.
- CN: 这一段实现了 getNode、assert、SDValue、getValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3361-3500
```cpp
3361:     llvm_unreachable("Unexpected masked vector store type");
3362:   case MVT::v4i64:
3363:   case MVT::v4f64: {
3364:     Opcode = NVPTXISD::StoreV4;
3365:     break;
3366:   }
3367:   case MVT::v8i32:
3368:   case MVT::v8f32: {
3369:     Opcode = NVPTXISD::StoreV8;
3370:     break;
3371:   }
3372:   }
3373:
3374:   SmallVector<SDValue, 8> Ops;
3375:
3376:   // Construct the new SDNode. First operand is the chain.
3377:   Ops.push_back(Chain);
3378:
3379:   // The next N operands are the values to store. Encode the mask into the
3380:   // values using the sentinel register 0 to represent a masked-off element.
3381:   assert(Mask.getValueType().isVector() &&
3382:          Mask.getValueType().getVectorElementType() == MVT::i1 &&
3383:          "Mask must be a vector of i1");
3384:   assert(Mask.getOpcode() == ISD::BUILD_VECTOR &&
3385:          "Mask expected to be a BUILD_VECTOR");
3386:   assert(Mask.getValueType().getVectorNumElements() ==
3387:              ValVT.getVectorNumElements() &&
3388:          "Mask size must be the same as the vector size");
3389:   for (auto [I, Op] : enumerate(Mask->ops())) {
3390:     // Mask elements must be constants.
3391:     if (Op.getNode()->getAsZExtVal() == 0) {
3392:       // Append a sentinel register 0 to the Ops vector to represent a masked
3393:       // off element, this will be handled in tablegen
3394:       Ops.push_back(DAG.getRegister(MCRegister::NoRegister,
3395:                                     ValVT.getVectorElementType()));
3396:     } else {
3397:       // Extract the element from the vector to store
3398:       SDValue ExtVal =
3399:           DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, ValVT.getVectorElementType(),
3400:                       Val, DAG.getIntPtrConstant(I, DL));
3401:       Ops.push_back(ExtVal);
3402:     }
3403:   }
3404:
3405:   // Next, the pointer operand.
3406:   Ops.push_back(BasePtr);
3407:
3408:   // Finally, the offset operand. We expect this to always be undef, and it will
3409:   // be ignored in lowering, but to mirror the handling of the other vector
3410:   // store instructions we include it in the new SDNode.
3411:   assert(Offset.getOpcode() == ISD::UNDEF &&
3412:          "Offset operand expected to be undef");
3413:   Ops.push_back(Offset);
3414:
3415:   SDValue NewSt =
3416:       DAG.getMemIntrinsicNode(Opcode, DL, DAG.getVTList(MVT::Other), Ops,
3417:                               MemSD->getMemoryVT(), MemSD->getMemOperand());
3418:
3419:   return NewSt;
3420: }
3421:
3422: SDValue
3423: NVPTXTargetLowering::LowerOperation(SDValue Op, SelectionDAG &DAG) const {
3424:   switch (Op.getOpcode()) {
3425:   case ISD::RETURNADDR:
3426:     return SDValue();
3427:   case ISD::FRAMEADDR:
3428:     return SDValue();
3429:   case ISD::ADDRSPACECAST:
3430:     return LowerADDRSPACECAST(Op, DAG);
3431:   case ISD::INTRINSIC_W_CHAIN:
3432:     return lowerIntrinsicWChain(Op, DAG);
3433:   case ISD::INTRINSIC_WO_CHAIN:
3434:     return lowerIntrinsicWOChain(Op, DAG);
3435:   case ISD::INTRINSIC_VOID:
3436:     return lowerIntrinsicVoid(Op, DAG);
3437:   case ISD::BUILD_VECTOR:
3438:     return LowerBUILD_VECTOR(Op, DAG);
3439:   case ISD::BITCAST:
3440:     return LowerBITCAST(Op, DAG);
3441:   case ISD::EXTRACT_SUBVECTOR:
3442:     return Op;
3443:   case ISD::EXTRACT_VECTOR_ELT:
3444:     return LowerEXTRACT_VECTOR_ELT(Op, DAG);
3445:   case ISD::INSERT_VECTOR_ELT:
3446:     return LowerINSERT_VECTOR_ELT(Op, DAG);
3447:   case ISD::VECTOR_SHUFFLE:
3448:     return LowerVECTOR_SHUFFLE(Op, DAG);
3449:   case ISD::CONCAT_VECTORS:
3450:     return LowerCONCAT_VECTORS(Op, DAG);
3451:   case ISD::VECREDUCE_FMAX:
3452:   case ISD::VECREDUCE_FMIN:
3453:   case ISD::VECREDUCE_FMAXIMUM:
3454:   case ISD::VECREDUCE_FMINIMUM:
3455:     return LowerVECREDUCE(Op, DAG);
3456:   case ISD::STORE:
3457:     return LowerSTORE(Op, DAG);
3458:   case ISD::MSTORE: {
3459:     assert(STI.has256BitVectorLoadStore(
3460:                cast<MemSDNode>(Op.getNode())->getAddressSpace()) &&
3461:            "Masked store vector not supported on subtarget.");
3462:     return lowerMSTORE(Op, DAG);
3463:   }
3464:   case ISD::LOAD:
3465:     return LowerLOAD(Op, DAG);
3466:   case ISD::MLOAD:
3467:     return LowerMLOAD(Op, DAG);
3468:   case ISD::SHL_PARTS:
3469:     return LowerShiftLeftParts(Op, DAG);
3470:   case ISD::SRA_PARTS:
3471:   case ISD::SRL_PARTS:
3472:     return LowerShiftRightParts(Op, DAG);
3473:   case ISD::SELECT:
3474:     return lowerSELECT(Op, DAG);
3475:   case ISD::FROUND:
3476:     return LowerFROUND(Op, DAG);
3477:   case ISD::FCOPYSIGN:
3478:     return LowerFCOPYSIGN(Op, DAG);
3479:   case ISD::SINT_TO_FP:
3480:   case ISD::UINT_TO_FP:
3481:     return LowerINT_TO_FP(Op, DAG);
3482:   case ISD::FP_TO_SINT:
3483:   case ISD::FP_TO_UINT:
3484:     return LowerFP_TO_INT(Op, DAG);
3485:   case ISD::FP_ROUND:
3486:     return LowerFP_ROUND(Op, DAG);
3487:   case ISD::FP_EXTEND:
3488:     return LowerFP_EXTEND(Op, DAG);
3489:   case ISD::VAARG:
3490:     return LowerVAARG(Op, DAG);
3491:   case ISD::VASTART:
3492:     return LowerVASTART(Op, DAG);
3493:   case ISD::FSHL:
3494:   case ISD::FSHR:
3495:     return lowerFSH(Op, DAG);
3496:   case ISD::ROTL:
3497:   case ISD::ROTR:
3498:     return lowerROT(Op, DAG);
3499:   case ISD::ABS:
3500:   case ISD::ABS_MIN_POISON:
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, push_back, getVectorElementType, getIntPtrConstant, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、push_back、getVectorElementType、getIntPtrConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3501-3640
```cpp
3501:   case ISD::SMIN:
3502:   case ISD::SMAX:
3503:   case ISD::UMIN:
3504:   case ISD::UMAX:
3505:   case ISD::ADD:
3506:   case ISD::SUB:
3507:   case ISD::MUL:
3508:   case ISD::SHL:
3509:   case ISD::SREM:
3510:   case ISD::UREM:
3511:     return LowerVectorArith(Op, DAG);
3512:   case ISD::DYNAMIC_STACKALLOC:
3513:     return LowerDYNAMIC_STACKALLOC(Op, DAG);
3514:   case ISD::STACKRESTORE:
3515:     return LowerSTACKRESTORE(Op, DAG);
3516:   case ISD::STACKSAVE:
3517:     return LowerSTACKSAVE(Op, DAG);
3518:   case ISD::CopyToReg:
3519:     return LowerCopyToReg_128(Op, DAG);
3520:   case ISD::FADD:
3521:   case ISD::FSUB:
3522:   case ISD::FMUL:
3523:     // Used only for bf16 on SM80, where we select fma for non-ftz operation
3524:     return PromoteBinOpIfF32FTZ(Op, DAG);
3525:   case ISD::CTPOP:
3526:   case ISD::CTLZ:
3527:     return lowerCTLZCTPOP(Op, DAG);
3528:   case ISD::FREM:
3529:     return lowerFREM(Op, DAG);
3530:   case ISD::BSWAP:
3531:     return lowerBSWAP(Op, DAG);
3532:   default:
3533:     llvm_unreachable("Custom lowering not defined for operation");
3534:   }
3535: }
3536:
3537: // This will prevent AsmPrinter from trying to print the jump tables itself.
3538: unsigned NVPTXTargetLowering::getJumpTableEncoding() const {
3539:   return MachineJumpTableInfo::EK_Inline;
3540: }
3541:
3542: SDValue NVPTXTargetLowering::LowerADDRSPACECAST(SDValue Op,
3543:                                                 SelectionDAG &DAG) const {
3544:   AddrSpaceCastSDNode *N = cast<AddrSpaceCastSDNode>(Op.getNode());
3545:   unsigned SrcAS = N->getSrcAddressSpace();
3546:   unsigned DestAS = N->getDestAddressSpace();
3547:   if (SrcAS != llvm::ADDRESS_SPACE_GENERIC &&
3548:       DestAS != llvm::ADDRESS_SPACE_GENERIC) {
3549:     // Shared and SharedCluster can be converted to each other through generic
3550:     // space
3551:     if ((SrcAS == llvm::ADDRESS_SPACE_SHARED &&
3552:          DestAS == llvm::ADDRESS_SPACE_SHARED_CLUSTER) ||
3553:         (SrcAS == llvm::ADDRESS_SPACE_SHARED_CLUSTER &&
3554:          DestAS == llvm::ADDRESS_SPACE_SHARED)) {
3555:       SDLoc DL(Op.getNode());
3556:       const MVT GenerictVT =
3557:           getPointerTy(DAG.getDataLayout(), ADDRESS_SPACE_GENERIC);
3558:       SDValue GenericConversion = DAG.getAddrSpaceCast(
3559:           DL, GenerictVT, Op.getOperand(0), SrcAS, ADDRESS_SPACE_GENERIC);
3560:       SDValue SharedClusterConversion =
3561:           DAG.getAddrSpaceCast(DL, Op.getValueType(), GenericConversion,
3562:                                ADDRESS_SPACE_GENERIC, DestAS);
3563:       return SharedClusterConversion;
3564:     }
3565:
3566:     return DAG.getUNDEF(Op.getValueType());
3567:   }
3568:
3569:   return Op;
3570: }
3571:
3572: // This function is almost a copy of SelectionDAG::expandVAArg().
3573: // The only diff is that this one produces loads from local address space.
3574: SDValue NVPTXTargetLowering::LowerVAARG(SDValue Op, SelectionDAG &DAG) const {
3575:   const TargetLowering *TLI = STI.getTargetLowering();
3576:   SDLoc DL(Op);
3577:
3578:   SDNode *Node = Op.getNode();
3579:   const Value *V = cast<SrcValueSDNode>(Node->getOperand(2))->getValue();
3580:   EVT VT = Node->getValueType(0);
3581:   auto *Ty = VT.getTypeForEVT(*DAG.getContext());
3582:   SDValue Tmp1 = Node->getOperand(0);
3583:   SDValue Tmp2 = Node->getOperand(1);
3584:   const MaybeAlign MA(Node->getConstantOperandVal(3));
3585:
3586:   SDValue VAListLoad = DAG.getLoad(TLI->getPointerTy(DAG.getDataLayout()), DL,
3587:                                    Tmp1, Tmp2, MachinePointerInfo(V));
3588:   SDValue VAList = VAListLoad;
3589:
3590:   if (MA && *MA > TLI->getMinStackArgumentAlignment()) {
3591:     VAList = DAG.getNode(
3592:         ISD::ADD, DL, VAList.getValueType(), VAList,
3593:         DAG.getConstant(MA->value() - 1, DL, VAList.getValueType()));
3594:
3595:     VAList = DAG.getNode(ISD::AND, DL, VAList.getValueType(), VAList,
3596:                          DAG.getSignedConstant(-(int64_t)MA->value(), DL,
3597:                                                VAList.getValueType()));
3598:   }
3599:
3600:   // Increment the pointer, VAList, to the next vaarg
3601:   Tmp1 = DAG.getNode(ISD::ADD, DL, VAList.getValueType(), VAList,
3602:                      DAG.getConstant(DAG.getDataLayout().getTypeAllocSize(Ty),
3603:                                      DL, VAList.getValueType()));
3604:
3605:   // Store the incremented VAList to the legalized pointer
3606:   Tmp1 = DAG.getStore(VAListLoad.getValue(1), DL, Tmp1, Tmp2,
3607:                       MachinePointerInfo(V));
3608:
3609:   const Value *SrcV = Constant::getNullValue(
3610:       PointerType::get(*DAG.getContext(), ADDRESS_SPACE_LOCAL));
3611:
3612:   // Load the actual argument out of the pointer VAList
3613:   return DAG.getLoad(VT, DL, Tmp1, VAList, MachinePointerInfo(SrcV));
3614: }
3615:
3616: SDValue NVPTXTargetLowering::LowerVASTART(SDValue Op, SelectionDAG &DAG) const {
3617:   const TargetLowering *TLI = STI.getTargetLowering();
3618:   SDLoc DL(Op);
3619:   EVT PtrVT = TLI->getPointerTy(DAG.getDataLayout());
3620:
3621:   // Store the address of unsized array <function>_vararg[] in the ap object.
3622:   SDValue VAReg = getParamSymbol(DAG, /* vararg */ -1, PtrVT);
3623:
3624:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
3625:   return DAG.getStore(Op.getOperand(0), DL, VAReg, Op.getOperand(1),
3626:                       MachinePointerInfo(SV));
3627: }
3628:
3629: static std::pair<MemSDNode *, uint32_t>
3630: convertMLOADToLoadWithUsedBytesMask(MemSDNode *N, SelectionDAG &DAG,
3631:                                     const NVPTXSubtarget &STI) {
3632:   SDValue Chain = N->getOperand(0);
3633:   SDValue BasePtr = N->getOperand(1);
3634:   SDValue Mask = N->getOperand(3);
3635:   [[maybe_unused]] SDValue Passthru = N->getOperand(4);
3636:
3637:   SDLoc DL(N);
3638:   EVT ResVT = N->getValueType(0);
3639:   assert(ResVT.isVector() && "Masked vector load must have vector type");
3640:   // While we only expect poison passthru vectors as an input to the backend,
```
- EN: This range implements operational logic in helpers such as LowerVectorArith, LowerDYNAMIC_STACKALLOC, LowerSTACKRESTORE, LowerSTACKSAVE, translating backend policy into executable code.
- CN: 这一段实现了 LowerVectorArith、LowerDYNAMIC_STACKALLOC、LowerSTACKRESTORE、LowerSTACKSAVE 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3641-3780
```cpp
3641:   // when the legalization framework splits a poison vector in half, it creates
3642:   // two undef vectors, so we can technically expect those too.
3643:   assert((Passthru.getOpcode() == ISD::POISON ||
3644:           Passthru.getOpcode() == ISD::UNDEF) &&
3645:          "Passthru operand expected to be poison or undef");
3646:
3647:   // Extract the mask and convert it to a uint32_t representing the used bytes
3648:   // of the entire vector load
3649:   uint32_t UsedBytesMask = 0;
3650:   uint32_t ElementSizeInBits = ResVT.getVectorElementType().getSizeInBits();
3651:   assert(ElementSizeInBits % 8 == 0 && "Unexpected element size");
3652:   uint32_t ElementSizeInBytes = ElementSizeInBits / 8;
3653:   uint32_t ElementMask = (1u << ElementSizeInBytes) - 1u;
3654:
3655:   for (SDValue Op : reverse(Mask->ops())) {
3656:     // We technically only want to do this shift for every
3657:     // iteration *but* the first, but in the first iteration UsedBytesMask is 0,
3658:     // so this shift is a no-op.
3659:     UsedBytesMask <<= ElementSizeInBytes;
3660:
3661:     // Mask elements must be constants.
3662:     if (Op->getAsZExtVal() != 0)
3663:       UsedBytesMask |= ElementMask;
3664:   }
3665:
3666:   assert(UsedBytesMask != 0 && UsedBytesMask != UINT32_MAX &&
3667:          "Unexpected masked load with elements masked all on or all off");
3668:
3669:   // Create a new load sd node to be handled normally by ReplaceLoadVector.
3670:   MemSDNode *NewLD = cast<MemSDNode>(
3671:       DAG.getLoad(ResVT, DL, Chain, BasePtr, N->getMemOperand()).getNode());
3672:
3673:   // If our subtarget does not support the used bytes mask pragma, "drop" the
3674:   // mask by setting it to UINT32_MAX
3675:   if (!STI.hasUsedBytesMaskPragma())
3676:     UsedBytesMask = UINT32_MAX;
3677:
3678:   return {NewLD, UsedBytesMask};
3679: }
3680:
3681: /// replaceLoadVector - Convert vector loads into multi-output scalar loads.
3682: static std::optional<std::pair<SDValue, SDValue>>
3683: replaceLoadVector(SDNode *N, SelectionDAG &DAG, const NVPTXSubtarget &STI) {
3684:   MemSDNode *LD = cast<MemSDNode>(N);
3685:   const EVT ResVT = LD->getValueType(0);
3686:   const EVT MemVT = LD->getMemoryVT();
3687:
3688:   // If we're doing sign/zero extension as part of the load, avoid lowering to
3689:   // a LoadV node. TODO: consider relaxing this restriction.
3690:   if (ResVT != MemVT)
3691:     return std::nullopt;
3692:
3693:   const auto NumEltsAndEltVT =
3694:       getVectorLoweringShape(ResVT, STI, LD->getAddressSpace());
3695:   if (!NumEltsAndEltVT)
3696:     return std::nullopt;
3697:   const auto [NumElts, EltVT] = NumEltsAndEltVT.value();
3698:
3699:   Align Alignment = LD->getAlign();
3700:   const auto &TD = DAG.getDataLayout();
3701:   Align PrefAlign = TD.getPrefTypeAlign(MemVT.getTypeForEVT(*DAG.getContext()));
3702:   if (Alignment < PrefAlign) {
3703:     // This load is not sufficiently aligned, so bail out and let this vector
3704:     // load be scalarized.  Note that we may still be able to emit smaller
3705:     // vector loads.  For example, if we are loading a <4 x float> with an
3706:     // alignment of 8, this check will fail but the legalizer will try again
3707:     // with 2 x <2 x float>, which will succeed with an alignment of 8.
3708:     return std::nullopt;
3709:   }
3710:
3711:   // If we have a masked load, convert it to a normal load now
3712:   std::optional<uint32_t> UsedBytesMask = std::nullopt;
3713:   if (LD->getOpcode() == ISD::MLOAD)
3714:     std::tie(LD, UsedBytesMask) =
3715:         convertMLOADToLoadWithUsedBytesMask(LD, DAG, STI);
3716:
3717:   // Since LoadV2 is a target node, we cannot rely on DAG type legalization.
3718:   // Therefore, we must ensure the type is legal.  For i1 and i8, we set the
3719:   // loaded type to i16 and propagate the "real" type as the memory type.
3720:   const MVT LoadEltVT = (EltVT.getSizeInBits() < 16) ? MVT::i16 : EltVT;
3721:
3722:   unsigned Opcode;
3723:   switch (NumElts) {
3724:   default:
3725:     return std::nullopt;
3726:   case 2:
3727:     Opcode = NVPTXISD::LoadV2;
3728:     break;
3729:   case 4:
3730:     Opcode = NVPTXISD::LoadV4;
3731:     break;
3732:   case 8:
3733:     Opcode = NVPTXISD::LoadV8;
3734:     break;
3735:   }
3736:   auto ListVTs = SmallVector<EVT, 9>(NumElts, LoadEltVT);
3737:   ListVTs.push_back(MVT::Other);
3738:   SDVTList LdResVTs = DAG.getVTList(ListVTs);
3739:
3740:   SDLoc DL(LD);
3741:
3742:   // Copy regular operands
3743:   SmallVector<SDValue, 8> OtherOps(LD->ops());
3744:
3745:   OtherOps.push_back(
3746:       DAG.getConstant(UsedBytesMask.value_or(UINT32_MAX), DL, MVT::i32));
3747:
3748:   // The select routine does not have access to the LoadSDNode instance, so
3749:   // pass along the extension information
3750:   OtherOps.push_back(
3751:       DAG.getIntPtrConstant(cast<LoadSDNode>(LD)->getExtensionType(), DL));
3752:
3753:   SDValue NewLD = DAG.getMemIntrinsicNode(Opcode, DL, LdResVTs, OtherOps, MemVT,
3754:                                           LD->getMemOperand());
3755:
3756:   SmallVector<SDValue> ScalarRes;
3757:   if (EltVT.isVector()) {
3758:     assert(EVT(EltVT.getVectorElementType()) == ResVT.getVectorElementType());
3759:     assert(NumElts * EltVT.getVectorNumElements() ==
3760:            ResVT.getVectorNumElements());
3761:     // Generate EXTRACT_VECTOR_ELTs to split v2[i,f,bf]16/v4i8 subvectors back
3762:     // into individual elements.
3763:     for (const unsigned I : llvm::seq(NumElts)) {
3764:       SDValue SubVector = NewLD.getValue(I);
3765:       DAG.ExtractVectorElements(SubVector, ScalarRes);
3766:     }
3767:   } else {
3768:     for (const unsigned I : llvm::seq(NumElts)) {
3769:       SDValue Res = NewLD.getValue(I);
3770:       if (LoadEltVT != EltVT)
3771:         Res = DAG.getNode(ISD::TRUNCATE, DL, EltVT, Res);
3772:       ScalarRes.push_back(Res);
3773:     }
3774:   }
3775:
3776:   SDValue LoadChain = NewLD.getValue(NumElts);
3777:
3778:   const MVT BuildVecVT =
3779:       MVT::getVectorVT(EltVT.getScalarType(), ScalarRes.size());
3780:   SDValue BuildVec = DAG.getBuildVector(BuildVecVT, DL, ScalarRes);
```
- EN: This range implements operational logic in helpers such as getVectorElementType, assert, getLoad, replaceLoadVector, translating backend policy into executable code.
- CN: 这一段实现了 getVectorElementType、assert、getLoad、replaceLoadVector 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3781-3920
```cpp
3781:   SDValue LoadValue = DAG.getBitcast(ResVT, BuildVec);
3782:
3783:   return {{LoadValue, LoadChain}};
3784: }
3785:
3786: static void replaceLoadVector(SDNode *N, SelectionDAG &DAG,
3787:                               SmallVectorImpl<SDValue> &Results,
3788:                               const NVPTXSubtarget &STI) {
3789:   if (auto Res = replaceLoadVector(N, DAG, STI))
3790:     Results.append({Res->first, Res->second});
3791: }
3792:
3793: static SDValue lowerLoadVector(SDNode *N, SelectionDAG &DAG,
3794:                                const NVPTXSubtarget &STI) {
3795:   if (auto Res = replaceLoadVector(N, DAG, STI))
3796:     return DAG.getMergeValues({Res->first, Res->second}, SDLoc(N));
3797:   return SDValue();
3798: }
3799:
3800: // v = ld i1* addr
3801: //   =>
3802: // v1 = ld i8* addr (-> i16)
3803: // v = trunc i16 to i1
3804: static SDValue lowerLOADi1(LoadSDNode *LD, SelectionDAG &DAG) {
3805:   SDLoc dl(LD);
3806:   assert(LD->getExtensionType() == ISD::NON_EXTLOAD);
3807:   assert(LD->getValueType(0) == MVT::i1 && "Custom lowering for i1 load only");
3808:   SDValue newLD = DAG.getExtLoad(ISD::ZEXTLOAD, dl, MVT::i16, LD->getChain(),
3809:                                  LD->getBasePtr(), LD->getPointerInfo(),
3810:                                  MVT::i8, LD->getAlign(),
3811:                                  LD->getMemOperand()->getFlags());
3812:   SDValue result = DAG.getNode(ISD::TRUNCATE, dl, MVT::i1, newLD);
3813:   // The legalizer (the caller) is expecting two values from the legalized
3814:   // load, so we build a MergeValues node for it. See ExpandUnalignedLoad()
3815:   // in LegalizeDAG.cpp which also uses MergeValues.
3816:   return DAG.getMergeValues({result, LD->getChain()}, dl);
3817: }
3818:
3819: SDValue NVPTXTargetLowering::LowerLOAD(SDValue Op, SelectionDAG &DAG) const {
3820:   LoadSDNode *LD = cast<LoadSDNode>(Op);
3821:
3822:   if (Op.getValueType() == MVT::i1)
3823:     return lowerLOADi1(LD, DAG);
3824:
3825:   // To improve CodeGen we'll legalize any-extend loads to zext loads. This is
3826:   // how they'll be lowered in ISel anyway, and by doing this a little earlier
3827:   // we allow for more DAG combine opportunities.
3828:   if (LD->getExtensionType() == ISD::EXTLOAD) {
3829:     assert(LD->getValueType(0).isInteger() && LD->getMemoryVT().isInteger() &&
3830:            "Unexpected fpext-load");
3831:     return DAG.getExtLoad(ISD::ZEXTLOAD, SDLoc(Op), Op.getValueType(),
3832:                           LD->getChain(), LD->getBasePtr(), LD->getMemoryVT(),
3833:                           LD->getMemOperand());
3834:   }
3835:
3836:   llvm_unreachable("Unexpected custom lowering for load");
3837: }
3838:
3839: SDValue NVPTXTargetLowering::LowerMLOAD(SDValue Op, SelectionDAG &DAG) const {
3840:   // v2f16/v2bf16/v2i16/v4i8 are legal, so we can't rely on legalizer to handle
3841:   // masked loads of these types and have to handle them here.
3842:   // v2f32 also needs to be handled here if the subtarget has f32x2
3843:   // instructions, making it legal.
3844:   //
3845:   // Note: misaligned masked loads should never reach this point
3846:   // because the override of isLegalMaskedLoad in NVPTXTargetTransformInfo.cpp
3847:   // will validate alignment. Therefore, we do not need to special case handle
3848:   // them here.
3849:   EVT VT = Op.getValueType();
3850:   if (NVPTX::isPackedVectorTy(VT)) {
3851:     auto Result = convertMLOADToLoadWithUsedBytesMask(
3852:         cast<MemSDNode>(Op.getNode()), DAG, STI);
3853:     MemSDNode *LD = std::get<0>(Result);
3854:     uint32_t UsedBytesMask = std::get<1>(Result);
3855:
3856:     SDLoc DL(LD);
3857:
3858:     // Copy regular operands
3859:     SmallVector<SDValue, 8> OtherOps(LD->ops());
3860:
3861:     OtherOps.push_back(DAG.getConstant(UsedBytesMask, DL, MVT::i32));
3862:
3863:     // We currently are not lowering extending loads, but pass the extension
3864:     // type anyway as later handling expects it.
3865:     OtherOps.push_back(
3866:         DAG.getIntPtrConstant(cast<LoadSDNode>(LD)->getExtensionType(), DL));
3867:     SDValue NewLD =
3868:         DAG.getMemIntrinsicNode(NVPTXISD::MLoad, DL, LD->getVTList(), OtherOps,
3869:                                 LD->getMemoryVT(), LD->getMemOperand());
3870:     return NewLD;
3871:   }
3872:   return SDValue();
3873: }
3874:
3875: static SDValue lowerSTOREVector(SDValue Op, SelectionDAG &DAG,
3876:                                 const NVPTXSubtarget &STI) {
3877:   MemSDNode *N = cast<MemSDNode>(Op.getNode());
3878:   SDValue Val = N->getOperand(1);
3879:   SDLoc DL(N);
3880:   const EVT ValVT = Val.getValueType();
3881:   const EVT MemVT = N->getMemoryVT();
3882:
3883:   // If we're truncating as part of the store, avoid lowering to a StoreV node.
3884:   // TODO: consider relaxing this restriction.
3885:   if (ValVT != MemVT)
3886:     return SDValue();
3887:
3888:   const auto NumEltsAndEltVT =
3889:       getVectorLoweringShape(ValVT, STI, N->getAddressSpace());
3890:   if (!NumEltsAndEltVT)
3891:     return SDValue();
3892:   const auto [NumElts, EltVT] = NumEltsAndEltVT.value();
3893:
3894:   const DataLayout &TD = DAG.getDataLayout();
3895:
3896:   Align Alignment = N->getAlign();
3897:   Align PrefAlign = TD.getPrefTypeAlign(ValVT.getTypeForEVT(*DAG.getContext()));
3898:   if (Alignment < PrefAlign) {
3899:     // This store is not sufficiently aligned, so bail out and let this vector
3900:     // store be scalarized.  Note that we may still be able to emit smaller
3901:     // vector stores.  For example, if we are storing a <4 x float> with an
3902:     // alignment of 8, this check will fail but the legalizer will try again
3903:     // with 2 x <2 x float>, which will succeed with an alignment of 8.
3904:     return SDValue();
3905:   }
3906:
3907:   unsigned Opcode;
3908:   switch (NumElts) {
3909:   default:
3910:     return SDValue();
3911:   case 2:
3912:     Opcode = NVPTXISD::StoreV2;
3913:     break;
3914:   case 4:
3915:     Opcode = NVPTXISD::StoreV4;
3916:     break;
3917:   case 8:
3918:     Opcode = NVPTXISD::StoreV8;
3919:     break;
3920:   }
```
- EN: This range implements operational logic in helpers such as getBitcast, append, getMergeValues, SDValue, translating backend policy into executable code.
- CN: 这一段实现了 getBitcast、append、getMergeValues、SDValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 3921-4060
```cpp
3921:
3922:   SmallVector<SDValue, 8> Ops;
3923:
3924:   // First is the chain
3925:   Ops.push_back(N->getOperand(0));
3926:
3927:   // Then the split values
3928:   if (EltVT.isVector()) {
3929:     assert(EVT(EltVT.getVectorElementType()) == ValVT.getVectorElementType());
3930:     assert(NumElts * EltVT.getVectorNumElements() ==
3931:            ValVT.getVectorNumElements());
3932:     // Combine individual elements into v2[i,f,bf]16/v4i8 subvectors to be
3933:     // stored as b32s
3934:     const unsigned NumEltsPerSubVector = EltVT.getVectorNumElements();
3935:     for (const unsigned I : llvm::seq(NumElts)) {
3936:       SmallVector<SDValue, 4> SubVectorElts;
3937:       DAG.ExtractVectorElements(Val, SubVectorElts, I * NumEltsPerSubVector,
3938:                                 NumEltsPerSubVector);
3939:       Ops.push_back(DAG.getBuildVector(EltVT, DL, SubVectorElts));
3940:     }
3941:   } else {
3942:     SDValue V = DAG.getBitcast(MVT::getVectorVT(EltVT, NumElts), Val);
3943:     for (const unsigned I : llvm::seq(NumElts)) {
3944:       SDValue ExtVal = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT, V,
3945:                                    DAG.getIntPtrConstant(I, DL));
3946:
3947:       // Since StoreV2 is a target node, we cannot rely on DAG type
3948:       // legalization. Therefore, we must ensure the type is legal.  For i1 and
3949:       // i8, we set the stored type to i16 and propagate the "real" type as the
3950:       // memory type.
3951:       if (EltVT.getSizeInBits() < 16)
3952:         ExtVal = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i16, ExtVal);
3953:       Ops.push_back(ExtVal);
3954:     }
3955:   }
3956:
3957:   // Then any remaining arguments
3958:   Ops.append(N->op_begin() + 2, N->op_end());
3959:
3960:   SDValue NewSt =
3961:       DAG.getMemIntrinsicNode(Opcode, DL, DAG.getVTList(MVT::Other), Ops,
3962:                               N->getMemoryVT(), N->getMemOperand());
3963:
3964:   // return DCI.CombineTo(N, NewSt, true);
3965:   return NewSt;
3966: }
3967:
3968: SDValue NVPTXTargetLowering::LowerSTORE(SDValue Op, SelectionDAG &DAG) const {
3969:   StoreSDNode *Store = cast<StoreSDNode>(Op);
3970:   EVT VT = Store->getMemoryVT();
3971:
3972:   if (VT == MVT::i1)
3973:     return LowerSTOREi1(Op, DAG);
3974:
3975:   // Lower store of any other vector type, including v2f32 as we want to break
3976:   // it apart since this is not a widely-supported type.
3977:   return lowerSTOREVector(Op, DAG, STI);
3978: }
3979:
3980: // st i1 v, addr
3981: //    =>
3982: // v1 = zxt v to i16
3983: // st.u8 i16, addr
3984: SDValue NVPTXTargetLowering::LowerSTOREi1(SDValue Op, SelectionDAG &DAG) const {
3985:   SDNode *Node = Op.getNode();
3986:   SDLoc dl(Node);
3987:   StoreSDNode *ST = cast<StoreSDNode>(Node);
3988:   SDValue Tmp1 = ST->getChain();
3989:   SDValue Tmp2 = ST->getBasePtr();
3990:   SDValue Tmp3 = ST->getValue();
3991:   assert(Tmp3.getValueType() == MVT::i1 && "Custom lowering for i1 store only");
3992:   Tmp3 = DAG.getNode(ISD::ZERO_EXTEND, dl, MVT::i16, Tmp3);
3993:   SDValue Result =
3994:       DAG.getTruncStore(Tmp1, dl, Tmp3, Tmp2, ST->getPointerInfo(), MVT::i8,
3995:                         ST->getAlign(), ST->getMemOperand()->getFlags());
3996:   return Result;
3997: }
3998:
3999: SDValue NVPTXTargetLowering::LowerCopyToReg_128(SDValue Op,
4000:                                                 SelectionDAG &DAG) const {
4001:   // Change the CopyToReg to take in two 64-bit operands instead of a 128-bit
4002:   // operand so that it can pass the legalization.
4003:
4004:   assert(Op.getOperand(1).getValueType() == MVT::i128 &&
4005:          "Custom lowering for 128-bit CopyToReg only");
4006:
4007:   SDNode *Node = Op.getNode();
4008:   SDLoc DL(Node);
4009:
4010:   SDValue Cast = DAG.getBitcast(MVT::v2i64, Op->getOperand(2));
4011:   SDValue Lo = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i64, Cast,
4012:                            DAG.getIntPtrConstant(0, DL));
4013:   SDValue Hi = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i64, Cast,
4014:                            DAG.getIntPtrConstant(1, DL));
4015:
4016:   SmallVector<SDValue, 5> NewOps(Op->getNumOperands() + 1);
4017:   SmallVector<EVT, 3> ResultsType(Node->values());
4018:
4019:   NewOps[0] = Op->getOperand(0); // Chain
4020:   NewOps[1] = Op->getOperand(1); // Dst Reg
4021:   NewOps[2] = Lo;                // Lower 64-bit
4022:   NewOps[3] = Hi;                // Higher 64-bit
4023:   if (Op.getNumOperands() == 4)
4024:     NewOps[4] = Op->getOperand(3); // Glue if exists
4025:
4026:   return DAG.getNode(ISD::CopyToReg, DL, ResultsType, NewOps);
4027: }
4028:
4029: unsigned NVPTXTargetLowering::getNumRegisters(
4030:     LLVMContext &Context, EVT VT,
4031:     std::optional<MVT> RegisterVT = std::nullopt) const {
4032:   if (VT == MVT::i128 && RegisterVT == MVT::i128)
4033:     return 1;
4034:   return TargetLoweringBase::getNumRegisters(Context, VT, RegisterVT);
4035: }
4036:
4037: bool NVPTXTargetLowering::splitValueIntoRegisterParts(
4038:     SelectionDAG &DAG, const SDLoc &DL, SDValue Val, SDValue *Parts,
4039:     unsigned NumParts, MVT PartVT, std::optional<CallingConv::ID> CC) const {
4040:   if (Val.getValueType() == MVT::i128 && NumParts == 1) {
4041:     Parts[0] = Val;
4042:     return true;
4043:   }
4044:   return false;
4045: }
4046:
4047: // This creates target external symbol for a function parameter.
4048: // Name of the symbol is composed from its index and the function name.
4049: // Negative index corresponds to special parameter (unsized array) used for
4050: // passing variable arguments.
4051: SDValue NVPTXTargetLowering::getParamSymbol(SelectionDAG &DAG, int I,
4052:                                             EVT T) const {
4053:   StringRef SavedStr = nvTM->getStrPool().save(
4054:       getParamName(&DAG.getMachineFunction().getFunction(), I));
4055:   return DAG.getExternalSymbol(SavedStr.data(), T);
4056: }
4057:
4058: SDValue NVPTXTargetLowering::getCallParamSymbol(SelectionDAG &DAG, int I,
4059:                                                 EVT T) const {
4060:   const StringRef SavedStr = nvTM->getStrPool().save("param" + Twine(I));
```
- EN: This range implements operational logic in helpers such as push_back, assert, getVectorNumElements, getBitcast, translating backend policy into executable code.
- CN: 这一段实现了 push_back、assert、getVectorNumElements、getBitcast 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4061-4200
```cpp
4061:   return DAG.getExternalSymbol(SavedStr.data(), T);
4062: }
4063:
4064: SDValue NVPTXTargetLowering::LowerFormalArguments(
4065:     SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
4066:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
4067:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
4068:   const DataLayout &DL = DAG.getDataLayout();
4069:   LLVMContext &Ctx = *DAG.getContext();
4070:   auto PtrVT = getPointerTy(DAG.getDataLayout());
4071:
4072:   const Function &F = DAG.getMachineFunction().getFunction();
4073:   const bool IsKernel = isKernelFunction(F);
4074:
4075:   SDValue Root = DAG.getRoot();
4076:   SmallVector<SDValue, 16> OutChains;
4077:
4078:   // argTypes.size() (or theArgs.size()) and Ins.size() need not match.
4079:   // Ins.size() will be larger
4080:   //   * if there is an aggregate argument with multiple fields (each field
4081:   //     showing up separately in Ins)
4082:   //   * if there is a vector argument with more than typical vector-length
4083:   //     elements (generally if more than 4) where each vector element is
4084:   //     individually present in Ins.
4085:   // So a different index should be used for indexing into Ins.
4086:   // See similar issue in LowerCall.
4087:
4088:   auto AllIns = ArrayRef(Ins);
4089:   for (const auto &Arg : F.args()) {
4090:     const auto ArgIns = AllIns.take_while(
4091:         [&](auto I) { return I.OrigArgIndex == Arg.getArgNo(); });
4092:     AllIns = AllIns.drop_front(ArgIns.size());
4093:
4094:     Type *Ty = Arg.getType();
4095:
4096:     if (ArgIns.empty())
4097:       report_fatal_error("Empty parameter types are not supported");
4098:
4099:     if (Arg.use_empty()) {
4100:       // argument is dead
4101:       for (const auto &In : ArgIns) {
4102:         assert(!In.Used && "Arg.use_empty() is true but Arg is used?");
4103:         InVals.push_back(DAG.getUNDEF(In.VT));
4104:       }
4105:       continue;
4106:     }
4107:
4108:     SDValue ArgSymbol = getParamSymbol(DAG, Arg.getArgNo(), PtrVT);
4109:
4110:     // In the following cases, assign a node order of "i+1"
4111:     // to newly created nodes. The SDNodes for params have to
4112:     // appear in the same order as their order of appearance
4113:     // in the original function. "i+1" holds that order.
4114:     if (Arg.hasByValAttr()) {
4115:       // Param has ByVal attribute
4116:       // Return MoveParam(param symbol).
4117:       // Ideally, the param symbol can be returned directly,
4118:       // but when SDNode builder decides to use it in a CopyToReg(),
4119:       // machine instruction fails because TargetExternalSymbol
4120:       // (not lowered) is target dependent, and CopyToReg assumes
4121:       // the source is lowered.
4122:       assert(ArgIns.size() == 1 && "ByVal argument must be a pointer");
4123:       const auto &ByvalIn = ArgIns[0];
4124:       assert(getValueType(DL, Ty) == ByvalIn.VT &&
4125:              "Ins type did not match function type");
4126:       assert(ByvalIn.VT == PtrVT && "ByVal argument must be a pointer");
4127:
4128:       SDValue P;
4129:       if (IsKernel) {
4130:         assert(isParamGridConstant(Arg) && "ByVal argument must be lowered to "
4131:                                            "grid_constant by NVPTXLowerArgs");
4132:         P = ArgSymbol;
4133:         P.getNode()->setIROrder(Arg.getArgNo() + 1);
4134:       } else {
4135:         P = DAG.getNode(NVPTXISD::MoveParam, dl, ByvalIn.VT, ArgSymbol);
4136:         P.getNode()->setIROrder(Arg.getArgNo() + 1);
4137:         P = DAG.getAddrSpaceCast(dl, ByvalIn.VT, P, ADDRESS_SPACE_LOCAL,
4138:                                  ADDRESS_SPACE_GENERIC);
4139:       }
4140:       InVals.push_back(P);
4141:     } else {
4142:       SmallVector<EVT, 16> VTs;
4143:       SmallVector<uint64_t, 16> Offsets;
4144:       ComputePTXValueVTs(*this, DL, Ctx, CallConv, Ty, VTs, Offsets);
4145:       assert(VTs.size() == ArgIns.size() && "Size mismatch");
4146:       assert(VTs.size() == Offsets.size() && "Size mismatch");
4147:
4148:       const Align ArgAlign = getFunctionArgumentAlignment(
4149:           &F, Ty, Arg.getArgNo() + AttributeList::FirstArgIndex, DL);
4150:
4151:       unsigned I = 0;
4152:       const auto VI = VectorizePTXValueVTs(VTs, Offsets, ArgAlign);
4153:       for (const unsigned NumElts : VI) {
4154:         // i1 is loaded/stored as i8
4155:         const EVT LoadVT = VTs[I] == MVT::i1 ? MVT::i8 : VTs[I];
4156:         const EVT VecVT = getVectorizedVT(LoadVT, NumElts, Ctx);
4157:
4158:         SDValue VecAddr = DAG.getObjectPtrOffset(
4159:             dl, ArgSymbol, TypeSize::getFixed(Offsets[I]));
4160:
4161:         const Align PartAlign = commonAlignment(ArgAlign, Offsets[I]);
4162:         const unsigned AS = IsKernel ? NVPTX::AddressSpace::EntryParam
4163:                                      : NVPTX::AddressSpace::DeviceParam;
4164:         SDValue P = DAG.getLoad(VecVT, dl, Root, VecAddr,
4165:                                 MachinePointerInfo(AS), PartAlign,
4166:                                 MachineMemOperand::MODereferenceable |
4167:                                     MachineMemOperand::MOInvariant);
4168:         P.getNode()->setIROrder(Arg.getArgNo() + 1);
4169:         for (const unsigned J : llvm::seq(NumElts)) {
4170:           SDValue Elt = getExtractVectorizedValue(P, J, LoadVT, dl, DAG);
4171:
4172:           Elt = correctParamType(Elt, ArgIns[I + J].VT, ArgIns[I + J].Flags,
4173:                                  DAG, dl);
4174:           InVals.push_back(Elt);
4175:         }
4176:         I += NumElts;
4177:       }
4178:     }
4179:   }
4180:
4181:   if (!OutChains.empty())
4182:     DAG.setRoot(DAG.getTokenFactor(dl, OutChains));
4183:
4184:   return Chain;
4185: }
4186:
4187: SDValue
4188: NVPTXTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
4189:                                  bool isVarArg,
4190:                                  const SmallVectorImpl<ISD::OutputArg> &Outs,
4191:                                  const SmallVectorImpl<SDValue> &OutVals,
4192:                                  const SDLoc &dl, SelectionDAG &DAG) const {
4193:   const Function &F = DAG.getMachineFunction().getFunction();
4194:   Type *RetTy = F.getReturnType();
4195:
4196:   if (RetTy->isVoidTy()) {
4197:     assert(OutVals.empty() && Outs.empty() && "Return value expected for void");
4198:     return DAG.getNode(NVPTXISD::RET_GLUE, dl, MVT::Other, Chain);
4199:   }
4200:
```
- EN: This range implements operational logic in helpers such as getExternalSymbol, getDataLayout, getContext, getPointerTy, translating backend policy into executable code.
- CN: 这一段实现了 getExternalSymbol、getDataLayout、getContext、getPointerTy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4201-4340
```cpp
4201:   const DataLayout &DL = DAG.getDataLayout();
4202:   LLVMContext &Ctx = *DAG.getContext();
4203:
4204:   const SDValue RetSymbol = DAG.getExternalSymbol("func_retval0", MVT::i32);
4205:   const auto RetAlign = getFunctionParamOptimizedAlign(&F, RetTy, DL);
4206:
4207:   // PTX Interoperability Guide 3.3(A): [Integer] Values shorter than
4208:   // 32-bits are sign extended or zero extended, depending on whether
4209:   // they are signed or unsigned types.
4210:   const bool ExtendIntegerRetVal =
4211:       RetTy->isIntegerTy() && DL.getTypeAllocSizeInBits(RetTy) < 32;
4212:
4213:   SmallVector<EVT, 16> VTs;
4214:   SmallVector<uint64_t, 16> Offsets;
4215:   ComputePTXValueVTs(*this, DL, Ctx, CallConv, RetTy, VTs, Offsets);
4216:   assert(VTs.size() == OutVals.size() && "Bad return value decomposition");
4217:
4218:   const auto GetRetVal = [&](unsigned I) -> SDValue {
4219:     SDValue RetVal = OutVals[I];
4220:     assert(promoteScalarIntegerPTX(RetVal.getValueType()) ==
4221:                RetVal.getValueType() &&
4222:            "OutVal type should always be legal");
4223:
4224:     const EVT VTI = promoteScalarIntegerPTX(VTs[I]);
4225:     const EVT StoreVT =
4226:         ExtendIntegerRetVal ? MVT::i32 : (VTI == MVT::i1 ? MVT::i8 : VTI);
4227:     return correctParamType(RetVal, StoreVT, Outs[I].Flags, DAG, dl);
4228:   };
4229:
4230:   unsigned I = 0;
4231:   const auto VI = VectorizePTXValueVTs(VTs, Offsets, RetAlign);
4232:   for (const unsigned NumElts : VI) {
4233:     const MaybeAlign CurrentAlign = ExtendIntegerRetVal
4234:                                         ? MaybeAlign(std::nullopt)
4235:                                         : commonAlignment(RetAlign, Offsets[I]);
4236:
4237:     SDValue Val = getBuildVectorizedValue(
4238:         NumElts, dl, DAG, [&](unsigned K) { return GetRetVal(I + K); });
4239:
4240:     SDValue Ptr =
4241:         DAG.getObjectPtrOffset(dl, RetSymbol, TypeSize::getFixed(Offsets[I]));
4242:
4243:     Chain = DAG.getStore(Chain, dl, Val, Ptr,
4244:                          MachinePointerInfo(NVPTX::AddressSpace::DeviceParam),
4245:                          CurrentAlign);
4246:
4247:     I += NumElts;
4248:   }
4249:
4250:   return DAG.getNode(NVPTXISD::RET_GLUE, dl, MVT::Other, Chain);
4251: }
4252:
4253: void NVPTXTargetLowering::LowerAsmOperandForConstraint(
4254:     SDValue Op, StringRef Constraint, std::vector<SDValue> &Ops,
4255:     SelectionDAG &DAG) const {
4256:   if (Constraint.size() > 1)
4257:     return;
4258:   TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
4259: }
4260:
4261: // llvm.ptx.memcpy.const and llvm.ptx.memmove.const need to be modeled as
4262: // TgtMemIntrinsic
4263: // because we need the information that is only available in the "Value" type
4264: // of destination
4265: // pointer. In particular, the address space information.
4266: void NVPTXTargetLowering::getTgtMemIntrinsic(
4267:     SmallVectorImpl<IntrinsicInfo> &Infos, const CallBase &I,
4268:     MachineFunction &MF, unsigned Intrinsic) const {
4269:   IntrinsicInfo Info;
4270:   switch (Intrinsic) {
4271:   default:
4272:     return;
4273:   case Intrinsic::nvvm_match_all_sync_i32p:
4274:   case Intrinsic::nvvm_match_all_sync_i64p:
4275:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4276:     // memVT is bogus. These intrinsics have IntrInaccessibleMemOnly attribute
4277:     // in order to model data exchange with other threads, but perform no real
4278:     // memory accesses.
4279:     Info.memVT = MVT::i1;
4280:
4281:     // Our result depends on both our and other thread's arguments.
4282:     Info.flags = MachineMemOperand::MOLoad | MachineMemOperand::MOStore;
4283:     Infos.push_back(Info);
4284:     return;
4285:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_f16_col:
4286:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_f16_row:
4287:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_f16_col_stride:
4288:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_f16_row_stride:
4289:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_f16_col:
4290:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_f16_row:
4291:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_f16_col_stride:
4292:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_f16_row_stride:
4293:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_f16_col:
4294:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_f16_row:
4295:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_f16_col_stride:
4296:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_f16_row_stride:
4297:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_f16_col:
4298:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_f16_row:
4299:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_f16_col_stride:
4300:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_f16_row_stride:
4301:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_f16_col:
4302:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_f16_row:
4303:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_f16_col_stride:
4304:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_f16_row_stride:
4305:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_f16_col:
4306:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_f16_row:
4307:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_f16_col_stride:
4308:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_f16_row_stride: {
4309:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4310:     Info.memVT = MVT::v8f16;
4311:     Info.ptrVal = I.getArgOperand(0);
4312:     Info.offset = 0;
4313:     Info.flags = MachineMemOperand::MOLoad;
4314:     Info.align = Align(16);
4315:     Infos.push_back(Info);
4316:     return;
4317:   }
4318:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_s8_col:
4319:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_s8_col_stride:
4320:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_u8_col_stride:
4321:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_u8_col:
4322:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_s8_row:
4323:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_s8_row_stride:
4324:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_u8_row_stride:
4325:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_u8_row:
4326:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_bf16_col:
4327:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_bf16_col_stride:
4328:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_bf16_row:
4329:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_bf16_row_stride:
4330:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_s8_col:
4331:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_s8_col_stride:
4332:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_u8_col_stride:
4333:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_u8_col:
4334:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_s8_row:
4335:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_s8_row_stride:
4336:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_u8_row_stride:
4337:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_u8_row:
4338:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_bf16_col:
4339:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_bf16_col_stride:
4340:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_bf16_row:
```
- EN: This range implements operational logic in helpers such as getDataLayout, getContext, getExternalSymbol, getFunctionParamOptimizedAlign, translating backend policy into executable code.
- CN: 这一段实现了 getDataLayout、getContext、getExternalSymbol、getFunctionParamOptimizedAlign 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4341-4480
```cpp
4341:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_bf16_row_stride: {
4342:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4343:     Info.memVT = MVT::v2i32;
4344:     Info.ptrVal = I.getArgOperand(0);
4345:     Info.offset = 0;
4346:     Info.flags = MachineMemOperand::MOLoad;
4347:     Info.align = Align(8);
4348:     Infos.push_back(Info);
4349:     return;
4350:   }
4351:
4352:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_s8_col:
4353:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_s8_col_stride:
4354:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_u8_col_stride:
4355:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_u8_col:
4356:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_s8_row:
4357:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_s8_row_stride:
4358:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_u8_row_stride:
4359:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_u8_row:
4360:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_bf16_col:
4361:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_bf16_col_stride:
4362:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_bf16_row:
4363:   case Intrinsic::nvvm_wmma_m16n16k16_load_a_bf16_row_stride:
4364:   case Intrinsic::nvvm_wmma_m16n16k8_load_a_tf32_col:
4365:   case Intrinsic::nvvm_wmma_m16n16k8_load_a_tf32_col_stride:
4366:   case Intrinsic::nvvm_wmma_m16n16k8_load_a_tf32_row:
4367:   case Intrinsic::nvvm_wmma_m16n16k8_load_a_tf32_row_stride:
4368:
4369:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_s8_col:
4370:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_s8_col_stride:
4371:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_u8_col_stride:
4372:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_u8_col:
4373:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_s8_row:
4374:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_s8_row_stride:
4375:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_u8_row_stride:
4376:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_u8_row:
4377:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_bf16_col:
4378:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_bf16_col_stride:
4379:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_bf16_row:
4380:   case Intrinsic::nvvm_wmma_m16n16k16_load_b_bf16_row_stride:
4381:   case Intrinsic::nvvm_wmma_m16n16k8_load_b_tf32_col:
4382:   case Intrinsic::nvvm_wmma_m16n16k8_load_b_tf32_col_stride:
4383:   case Intrinsic::nvvm_wmma_m16n16k8_load_b_tf32_row:
4384:   case Intrinsic::nvvm_wmma_m16n16k8_load_b_tf32_row_stride:
4385:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n8_x4_b16:
4386:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n8_x4_trans_b16:
4387:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m16n16_x2_trans_b8:
4388:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m16n16_x2_trans_b8x16_b4x16_p64:
4389:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m16n16_x2_trans_b8x16_b6x16_p32:
4390:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n16_x4_b8x16_b4x16_p64:
4391:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n16_x4_b8x16_b6x16_p32: {
4392:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4393:     Info.memVT = MVT::v4i32;
4394:     Info.ptrVal = I.getArgOperand(0);
4395:     Info.offset = 0;
4396:     Info.flags = MachineMemOperand::MOLoad;
4397:     Info.align = Align(16);
4398:     Infos.push_back(Info);
4399:     return;
4400:   }
4401:
4402:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_s8_col:
4403:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_s8_col_stride:
4404:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_u8_col_stride:
4405:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_u8_col:
4406:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_s8_row:
4407:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_s8_row_stride:
4408:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_u8_row_stride:
4409:   case Intrinsic::nvvm_wmma_m32n8k16_load_b_u8_row:
4410:
4411:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_s8_col:
4412:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_s8_col_stride:
4413:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_u8_col_stride:
4414:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_u8_col:
4415:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_s8_row:
4416:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_s8_row_stride:
4417:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_u8_row_stride:
4418:   case Intrinsic::nvvm_wmma_m8n32k16_load_a_u8_row:
4419:   case Intrinsic::nvvm_wmma_m8n8k128_load_a_b1_row:
4420:   case Intrinsic::nvvm_wmma_m8n8k128_load_a_b1_row_stride:
4421:   case Intrinsic::nvvm_wmma_m8n8k128_load_b_b1_col:
4422:   case Intrinsic::nvvm_wmma_m8n8k128_load_b_b1_col_stride:
4423:   case Intrinsic::nvvm_wmma_m8n8k32_load_a_s4_row:
4424:   case Intrinsic::nvvm_wmma_m8n8k32_load_a_s4_row_stride:
4425:   case Intrinsic::nvvm_wmma_m8n8k32_load_a_u4_row_stride:
4426:   case Intrinsic::nvvm_wmma_m8n8k32_load_a_u4_row:
4427:   case Intrinsic::nvvm_wmma_m8n8k32_load_b_s4_col:
4428:   case Intrinsic::nvvm_wmma_m8n8k32_load_b_s4_col_stride:
4429:   case Intrinsic::nvvm_wmma_m8n8k32_load_b_u4_col_stride:
4430:   case Intrinsic::nvvm_wmma_m8n8k32_load_b_u4_col:
4431:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n8_x1_b16:
4432:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n8_x1_trans_b16:
4433:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n16_x1_b8x16_b4x16_p64:
4434:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n16_x1_b8x16_b6x16_p32: {
4435:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4436:     Info.memVT = MVT::i32;
4437:     Info.ptrVal = I.getArgOperand(0);
4438:     Info.offset = 0;
4439:     Info.flags = MachineMemOperand::MOLoad;
4440:     Info.align = Align(4);
4441:     Infos.push_back(Info);
4442:     return;
4443:   }
4444:
4445:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f16_col:
4446:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f16_row:
4447:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f16_col_stride:
4448:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f16_row_stride:
4449:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f16_col:
4450:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f16_row:
4451:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f16_col_stride:
4452:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f16_row_stride:
4453:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f16_col:
4454:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f16_row:
4455:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f16_col_stride:
4456:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f16_row_stride: {
4457:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4458:     Info.memVT = MVT::v4f16;
4459:     Info.ptrVal = I.getArgOperand(0);
4460:     Info.offset = 0;
4461:     Info.flags = MachineMemOperand::MOLoad;
4462:     Info.align = Align(16);
4463:     Infos.push_back(Info);
4464:     return;
4465:   }
4466:
4467:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f32_col:
4468:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f32_row:
4469:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f32_col_stride:
4470:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_f32_row_stride:
4471:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f32_col:
4472:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f32_row:
4473:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f32_col_stride:
4474:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_f32_row_stride:
4475:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f32_col:
4476:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f32_row:
4477:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f32_col_stride:
4478:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_f32_row_stride:
4479:   case Intrinsic::nvvm_wmma_m16n16k8_load_c_f32_col:
4480:   case Intrinsic::nvvm_wmma_m16n16k8_load_c_f32_row:
```
- EN: This range implements operational logic in helpers such as getArgOperand, Align, push_back, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、Align、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4481-4620
```cpp
4481:   case Intrinsic::nvvm_wmma_m16n16k8_load_c_f32_col_stride:
4482:   case Intrinsic::nvvm_wmma_m16n16k8_load_c_f32_row_stride: {
4483:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4484:     Info.memVT = MVT::v8f32;
4485:     Info.ptrVal = I.getArgOperand(0);
4486:     Info.offset = 0;
4487:     Info.flags = MachineMemOperand::MOLoad;
4488:     Info.align = Align(16);
4489:     Infos.push_back(Info);
4490:     return;
4491:   }
4492:
4493:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_bf16_col:
4494:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_bf16_col_stride:
4495:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_bf16_row:
4496:   case Intrinsic::nvvm_wmma_m32n8k16_load_a_bf16_row_stride:
4497:
4498:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_bf16_col:
4499:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_bf16_col_stride:
4500:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_bf16_row:
4501:   case Intrinsic::nvvm_wmma_m8n32k16_load_b_bf16_row_stride:
4502:
4503:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_s32_col:
4504:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_s32_col_stride:
4505:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_s32_row:
4506:   case Intrinsic::nvvm_wmma_m16n16k16_load_c_s32_row_stride:
4507:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_s32_col:
4508:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_s32_col_stride:
4509:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_s32_row:
4510:   case Intrinsic::nvvm_wmma_m32n8k16_load_c_s32_row_stride:
4511:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_s32_col:
4512:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_s32_col_stride:
4513:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_s32_row:
4514:   case Intrinsic::nvvm_wmma_m8n32k16_load_c_s32_row_stride: {
4515:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4516:     Info.memVT = MVT::v8i32;
4517:     Info.ptrVal = I.getArgOperand(0);
4518:     Info.offset = 0;
4519:     Info.flags = MachineMemOperand::MOLoad;
4520:     Info.align = Align(16);
4521:     Infos.push_back(Info);
4522:     return;
4523:   }
4524:
4525:   case Intrinsic::nvvm_wmma_m8n8k128_load_c_s32_col:
4526:   case Intrinsic::nvvm_wmma_m8n8k128_load_c_s32_col_stride:
4527:   case Intrinsic::nvvm_wmma_m8n8k128_load_c_s32_row:
4528:   case Intrinsic::nvvm_wmma_m8n8k128_load_c_s32_row_stride:
4529:   case Intrinsic::nvvm_wmma_m8n8k32_load_c_s32_col:
4530:   case Intrinsic::nvvm_wmma_m8n8k32_load_c_s32_col_stride:
4531:   case Intrinsic::nvvm_wmma_m8n8k32_load_c_s32_row:
4532:   case Intrinsic::nvvm_wmma_m8n8k32_load_c_s32_row_stride:
4533:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n8_x2_b16:
4534:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n8_x2_trans_b16:
4535:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m16n16_x1_trans_b8:
4536:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m16n16_x1_trans_b8x16_b4x16_p64:
4537:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m16n16_x1_trans_b8x16_b6x16_p32:
4538:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n16_x2_b8x16_b4x16_p64:
4539:   case Intrinsic::nvvm_ldmatrix_sync_aligned_m8n16_x2_b8x16_b6x16_p32: {
4540:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4541:     Info.memVT = MVT::v2i32;
4542:     Info.ptrVal = I.getArgOperand(0);
4543:     Info.offset = 0;
4544:     Info.flags = MachineMemOperand::MOLoad;
4545:     Info.align = Align(8);
4546:     Infos.push_back(Info);
4547:     return;
4548:   }
4549:
4550:   case Intrinsic::nvvm_wmma_m8n8k4_load_a_f64_col:
4551:   case Intrinsic::nvvm_wmma_m8n8k4_load_a_f64_col_stride:
4552:   case Intrinsic::nvvm_wmma_m8n8k4_load_a_f64_row:
4553:   case Intrinsic::nvvm_wmma_m8n8k4_load_a_f64_row_stride:
4554:
4555:   case Intrinsic::nvvm_wmma_m8n8k4_load_b_f64_col:
4556:   case Intrinsic::nvvm_wmma_m8n8k4_load_b_f64_col_stride:
4557:   case Intrinsic::nvvm_wmma_m8n8k4_load_b_f64_row:
4558:   case Intrinsic::nvvm_wmma_m8n8k4_load_b_f64_row_stride: {
4559:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4560:     Info.memVT = MVT::f64;
4561:     Info.ptrVal = I.getArgOperand(0);
4562:     Info.offset = 0;
4563:     Info.flags = MachineMemOperand::MOLoad;
4564:     Info.align = Align(8);
4565:     Infos.push_back(Info);
4566:     return;
4567:   }
4568:
4569:   case Intrinsic::nvvm_wmma_m8n8k4_load_c_f64_col:
4570:   case Intrinsic::nvvm_wmma_m8n8k4_load_c_f64_col_stride:
4571:   case Intrinsic::nvvm_wmma_m8n8k4_load_c_f64_row:
4572:   case Intrinsic::nvvm_wmma_m8n8k4_load_c_f64_row_stride: {
4573:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4574:     Info.memVT = MVT::v2f64;
4575:     Info.ptrVal = I.getArgOperand(0);
4576:     Info.offset = 0;
4577:     Info.flags = MachineMemOperand::MOLoad;
4578:     Info.align = Align(16);
4579:     Infos.push_back(Info);
4580:     return;
4581:   }
4582:
4583:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f16_col:
4584:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f16_row:
4585:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f16_col_stride:
4586:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f16_row_stride:
4587:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f16_col:
4588:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f16_row:
4589:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f16_col_stride:
4590:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f16_row_stride:
4591:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f16_col:
4592:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f16_row:
4593:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f16_col_stride:
4594:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f16_row_stride: {
4595:     Info.opc = ISD::INTRINSIC_VOID;
4596:     Info.memVT = MVT::v4f16;
4597:     Info.ptrVal = I.getArgOperand(0);
4598:     Info.offset = 0;
4599:     Info.flags = MachineMemOperand::MOStore;
4600:     Info.align = Align(16);
4601:     Infos.push_back(Info);
4602:     return;
4603:   }
4604:
4605:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f32_col:
4606:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f32_row:
4607:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f32_col_stride:
4608:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_f32_row_stride:
4609:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f32_col:
4610:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f32_row:
4611:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f32_col_stride:
4612:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_f32_row_stride:
4613:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f32_col:
4614:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f32_row:
4615:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f32_col_stride:
4616:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_f32_row_stride:
4617:   case Intrinsic::nvvm_wmma_m16n16k8_store_d_f32_col:
4618:   case Intrinsic::nvvm_wmma_m16n16k8_store_d_f32_row:
4619:   case Intrinsic::nvvm_wmma_m16n16k8_store_d_f32_col_stride:
4620:   case Intrinsic::nvvm_wmma_m16n16k8_store_d_f32_row_stride: {
```
- EN: This range implements operational logic in helpers such as getArgOperand, Align, push_back, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、Align、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4621-4760
```cpp
4621:     Info.opc = ISD::INTRINSIC_VOID;
4622:     Info.memVT = MVT::v8f32;
4623:     Info.ptrVal = I.getArgOperand(0);
4624:     Info.offset = 0;
4625:     Info.flags = MachineMemOperand::MOStore;
4626:     Info.align = Align(16);
4627:     Infos.push_back(Info);
4628:     return;
4629:   }
4630:
4631:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_s32_col:
4632:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_s32_col_stride:
4633:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_s32_row:
4634:   case Intrinsic::nvvm_wmma_m16n16k16_store_d_s32_row_stride:
4635:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_s32_col:
4636:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_s32_col_stride:
4637:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_s32_row:
4638:   case Intrinsic::nvvm_wmma_m32n8k16_store_d_s32_row_stride:
4639:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_s32_col:
4640:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_s32_col_stride:
4641:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_s32_row:
4642:   case Intrinsic::nvvm_wmma_m8n32k16_store_d_s32_row_stride: {
4643:     Info.opc = ISD::INTRINSIC_VOID;
4644:     Info.memVT = MVT::v8i32;
4645:     Info.ptrVal = I.getArgOperand(0);
4646:     Info.offset = 0;
4647:     Info.flags = MachineMemOperand::MOStore;
4648:     Info.align = Align(16);
4649:     Infos.push_back(Info);
4650:     return;
4651:   }
4652:
4653:   case Intrinsic::nvvm_wmma_m8n8k128_store_d_s32_col:
4654:   case Intrinsic::nvvm_wmma_m8n8k128_store_d_s32_col_stride:
4655:   case Intrinsic::nvvm_wmma_m8n8k128_store_d_s32_row:
4656:   case Intrinsic::nvvm_wmma_m8n8k128_store_d_s32_row_stride:
4657:   case Intrinsic::nvvm_wmma_m8n8k32_store_d_s32_col:
4658:   case Intrinsic::nvvm_wmma_m8n8k32_store_d_s32_col_stride:
4659:   case Intrinsic::nvvm_wmma_m8n8k32_store_d_s32_row:
4660:   case Intrinsic::nvvm_wmma_m8n8k32_store_d_s32_row_stride:
4661:   case Intrinsic::nvvm_stmatrix_sync_aligned_m8n8_x2_b16:
4662:   case Intrinsic::nvvm_stmatrix_sync_aligned_m8n8_x2_trans_b16:
4663:   case Intrinsic::nvvm_stmatrix_sync_aligned_m16n8_x2_trans_b8: {
4664:     Info.opc = ISD::INTRINSIC_VOID;
4665:     Info.memVT = MVT::v2i32;
4666:     Info.ptrVal = I.getArgOperand(0);
4667:     Info.offset = 0;
4668:     Info.flags = MachineMemOperand::MOStore;
4669:     Info.align = Align(8);
4670:     Infos.push_back(Info);
4671:     return;
4672:   }
4673:
4674:   case Intrinsic::nvvm_wmma_m8n8k4_store_d_f64_col:
4675:   case Intrinsic::nvvm_wmma_m8n8k4_store_d_f64_col_stride:
4676:   case Intrinsic::nvvm_wmma_m8n8k4_store_d_f64_row:
4677:   case Intrinsic::nvvm_wmma_m8n8k4_store_d_f64_row_stride: {
4678:     Info.opc = ISD::INTRINSIC_VOID;
4679:     Info.memVT = MVT::v2f64;
4680:     Info.ptrVal = I.getArgOperand(0);
4681:     Info.offset = 0;
4682:     Info.flags = MachineMemOperand::MOStore;
4683:     Info.align = Align(16);
4684:     Infos.push_back(Info);
4685:     return;
4686:   }
4687:
4688:   case Intrinsic::nvvm_stmatrix_sync_aligned_m8n8_x1_b16:
4689:   case Intrinsic::nvvm_stmatrix_sync_aligned_m8n8_x1_trans_b16:
4690:   case Intrinsic::nvvm_stmatrix_sync_aligned_m16n8_x1_trans_b8: {
4691:     Info.opc = ISD::INTRINSIC_VOID;
4692:     Info.memVT = MVT::i32;
4693:     Info.ptrVal = I.getArgOperand(0);
4694:     Info.offset = 0;
4695:     Info.flags = MachineMemOperand::MOStore;
4696:     Info.align = Align(4);
4697:     Infos.push_back(Info);
4698:     return;
4699:   }
4700:
4701:   case Intrinsic::nvvm_stmatrix_sync_aligned_m8n8_x4_b16:
4702:   case Intrinsic::nvvm_stmatrix_sync_aligned_m8n8_x4_trans_b16:
4703:   case Intrinsic::nvvm_stmatrix_sync_aligned_m16n8_x4_trans_b8: {
4704:     Info.opc = ISD::INTRINSIC_VOID;
4705:     Info.memVT = MVT::v4i32;
4706:     Info.ptrVal = I.getArgOperand(0);
4707:     Info.offset = 0;
4708:     Info.flags = MachineMemOperand::MOStore;
4709:     Info.align = Align(16);
4710:     Infos.push_back(Info);
4711:     return;
4712:   }
4713:
4714:   case Intrinsic::nvvm_atomic_add_gen_f_cta:
4715:   case Intrinsic::nvvm_atomic_add_gen_f_sys:
4716:   case Intrinsic::nvvm_atomic_add_gen_i_cta:
4717:   case Intrinsic::nvvm_atomic_add_gen_i_sys:
4718:   case Intrinsic::nvvm_atomic_and_gen_i_cta:
4719:   case Intrinsic::nvvm_atomic_and_gen_i_sys:
4720:   case Intrinsic::nvvm_atomic_cas_gen_i_cta:
4721:   case Intrinsic::nvvm_atomic_cas_gen_i_sys:
4722:   case Intrinsic::nvvm_atomic_dec_gen_i_cta:
4723:   case Intrinsic::nvvm_atomic_dec_gen_i_sys:
4724:   case Intrinsic::nvvm_atomic_inc_gen_i_cta:
4725:   case Intrinsic::nvvm_atomic_inc_gen_i_sys:
4726:   case Intrinsic::nvvm_atomic_max_gen_i_cta:
4727:   case Intrinsic::nvvm_atomic_max_gen_i_sys:
4728:   case Intrinsic::nvvm_atomic_min_gen_i_cta:
4729:   case Intrinsic::nvvm_atomic_min_gen_i_sys:
4730:   case Intrinsic::nvvm_atomic_or_gen_i_cta:
4731:   case Intrinsic::nvvm_atomic_or_gen_i_sys:
4732:   case Intrinsic::nvvm_atomic_exch_gen_i_cta:
4733:   case Intrinsic::nvvm_atomic_exch_gen_i_sys:
4734:   case Intrinsic::nvvm_atomic_xor_gen_i_cta:
4735:   case Intrinsic::nvvm_atomic_xor_gen_i_sys: {
4736:     auto &DL = I.getDataLayout();
4737:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4738:     Info.memVT = getValueType(DL, I.getType());
4739:     Info.ptrVal = I.getArgOperand(0);
4740:     Info.offset = 0;
4741:     Info.flags = MachineMemOperand::MOLoad | MachineMemOperand::MOStore;
4742:     Info.align.reset();
4743:     Infos.push_back(Info);
4744:     return;
4745:   }
4746:
4747:   case Intrinsic::nvvm_prefetch_tensormap: {
4748:     auto &DL = I.getDataLayout();
4749:     Info.opc = ISD::INTRINSIC_VOID;
4750:     Info.memVT = getPointerTy(DL);
4751:     Info.ptrVal = I.getArgOperand(0);
4752:     Info.offset = 0;
4753:     Info.flags =
4754:         MachineMemOperand::MOLoad | MachineMemOperand::MODereferenceable;
4755:     Info.align.reset();
4756:     Infos.push_back(Info);
4757:     return;
4758:   }
4759:
4760:   case Intrinsic::nvvm_tensormap_replace_global_address:
```
- EN: This range implements operational logic in helpers such as getArgOperand, Align, push_back, getDataLayout, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、Align、push_back、getDataLayout 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4761-4900
```cpp
4761:   case Intrinsic::nvvm_tensormap_replace_global_stride: {
4762:     Info.opc = ISD::INTRINSIC_VOID;
4763:     Info.memVT = MVT::i64;
4764:     Info.ptrVal = I.getArgOperand(0);
4765:     Info.offset = 0;
4766:     Info.flags = MachineMemOperand::MOStore;
4767:     Info.align.reset();
4768:     Infos.push_back(Info);
4769:     return;
4770:   }
4771:
4772:   case Intrinsic::nvvm_tensormap_replace_rank:
4773:   case Intrinsic::nvvm_tensormap_replace_box_dim:
4774:   case Intrinsic::nvvm_tensormap_replace_global_dim:
4775:   case Intrinsic::nvvm_tensormap_replace_element_stride:
4776:   case Intrinsic::nvvm_tensormap_replace_elemtype:
4777:   case Intrinsic::nvvm_tensormap_replace_interleave_layout:
4778:   case Intrinsic::nvvm_tensormap_replace_swizzle_mode:
4779:   case Intrinsic::nvvm_tensormap_replace_swizzle_atomicity:
4780:   case Intrinsic::nvvm_tensormap_replace_fill_mode: {
4781:     Info.opc = ISD::INTRINSIC_VOID;
4782:     Info.memVT = MVT::i32;
4783:     Info.ptrVal = I.getArgOperand(0);
4784:     Info.offset = 0;
4785:     Info.flags = MachineMemOperand::MOStore;
4786:     Info.align.reset();
4787:     Infos.push_back(Info);
4788:     return;
4789:   }
4790:
4791:   case Intrinsic::nvvm_ldu_global_i:
4792:   case Intrinsic::nvvm_ldu_global_f:
4793:   case Intrinsic::nvvm_ldu_global_p: {
4794:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4795:     Info.memVT = getValueType(I.getDataLayout(), I.getType());
4796:     Info.ptrVal = I.getArgOperand(0);
4797:     Info.offset = 0;
4798:     Info.flags = MachineMemOperand::MOLoad;
4799:     Info.align = cast<ConstantInt>(I.getArgOperand(1))->getMaybeAlignValue();
4800:
4801:     Infos.push_back(Info);
4802:     return;
4803:   }
4804:   case Intrinsic::nvvm_tex_1d_v4f32_s32:
4805:   case Intrinsic::nvvm_tex_1d_v4f32_f32:
4806:   case Intrinsic::nvvm_tex_1d_level_v4f32_f32:
4807:   case Intrinsic::nvvm_tex_1d_grad_v4f32_f32:
4808:   case Intrinsic::nvvm_tex_1d_array_v4f32_s32:
4809:   case Intrinsic::nvvm_tex_1d_array_v4f32_f32:
4810:   case Intrinsic::nvvm_tex_1d_array_level_v4f32_f32:
4811:   case Intrinsic::nvvm_tex_1d_array_grad_v4f32_f32:
4812:   case Intrinsic::nvvm_tex_2d_v4f32_s32:
4813:   case Intrinsic::nvvm_tex_2d_v4f32_f32:
4814:   case Intrinsic::nvvm_tex_2d_level_v4f32_f32:
4815:   case Intrinsic::nvvm_tex_2d_grad_v4f32_f32:
4816:   case Intrinsic::nvvm_tex_2d_array_v4f32_s32:
4817:   case Intrinsic::nvvm_tex_2d_array_v4f32_f32:
4818:   case Intrinsic::nvvm_tex_2d_array_level_v4f32_f32:
4819:   case Intrinsic::nvvm_tex_2d_array_grad_v4f32_f32:
4820:   case Intrinsic::nvvm_tex_3d_v4f32_s32:
4821:   case Intrinsic::nvvm_tex_3d_v4f32_f32:
4822:   case Intrinsic::nvvm_tex_3d_level_v4f32_f32:
4823:   case Intrinsic::nvvm_tex_3d_grad_v4f32_f32:
4824:   case Intrinsic::nvvm_tex_cube_v4f32_f32:
4825:   case Intrinsic::nvvm_tex_cube_level_v4f32_f32:
4826:   case Intrinsic::nvvm_tex_cube_array_v4f32_f32:
4827:   case Intrinsic::nvvm_tex_cube_array_level_v4f32_f32:
4828:   case Intrinsic::nvvm_tld4_r_2d_v4f32_f32:
4829:   case Intrinsic::nvvm_tld4_g_2d_v4f32_f32:
4830:   case Intrinsic::nvvm_tld4_b_2d_v4f32_f32:
4831:   case Intrinsic::nvvm_tld4_a_2d_v4f32_f32:
4832:   case Intrinsic::nvvm_tex_unified_1d_v4f32_s32:
4833:   case Intrinsic::nvvm_tex_unified_1d_v4f32_f32:
4834:   case Intrinsic::nvvm_tex_unified_1d_level_v4f32_f32:
4835:   case Intrinsic::nvvm_tex_unified_1d_grad_v4f32_f32:
4836:   case Intrinsic::nvvm_tex_unified_1d_array_v4f32_s32:
4837:   case Intrinsic::nvvm_tex_unified_1d_array_v4f32_f32:
4838:   case Intrinsic::nvvm_tex_unified_1d_array_level_v4f32_f32:
4839:   case Intrinsic::nvvm_tex_unified_1d_array_grad_v4f32_f32:
4840:   case Intrinsic::nvvm_tex_unified_2d_v4f32_s32:
4841:   case Intrinsic::nvvm_tex_unified_2d_v4f32_f32:
4842:   case Intrinsic::nvvm_tex_unified_2d_level_v4f32_f32:
4843:   case Intrinsic::nvvm_tex_unified_2d_grad_v4f32_f32:
4844:   case Intrinsic::nvvm_tex_unified_2d_array_v4f32_s32:
4845:   case Intrinsic::nvvm_tex_unified_2d_array_v4f32_f32:
4846:   case Intrinsic::nvvm_tex_unified_2d_array_level_v4f32_f32:
4847:   case Intrinsic::nvvm_tex_unified_2d_array_grad_v4f32_f32:
4848:   case Intrinsic::nvvm_tex_unified_3d_v4f32_s32:
4849:   case Intrinsic::nvvm_tex_unified_3d_v4f32_f32:
4850:   case Intrinsic::nvvm_tex_unified_3d_level_v4f32_f32:
4851:   case Intrinsic::nvvm_tex_unified_3d_grad_v4f32_f32:
4852:   case Intrinsic::nvvm_tex_unified_cube_v4f32_f32:
4853:   case Intrinsic::nvvm_tex_unified_cube_level_v4f32_f32:
4854:   case Intrinsic::nvvm_tex_unified_cube_array_v4f32_f32:
4855:   case Intrinsic::nvvm_tex_unified_cube_array_level_v4f32_f32:
4856:   case Intrinsic::nvvm_tex_unified_cube_grad_v4f32_f32:
4857:   case Intrinsic::nvvm_tex_unified_cube_array_grad_v4f32_f32:
4858:   case Intrinsic::nvvm_tld4_unified_r_2d_v4f32_f32:
4859:   case Intrinsic::nvvm_tld4_unified_g_2d_v4f32_f32:
4860:   case Intrinsic::nvvm_tld4_unified_b_2d_v4f32_f32:
4861:   case Intrinsic::nvvm_tld4_unified_a_2d_v4f32_f32:
4862:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4863:     Info.memVT = MVT::v4f32;
4864:     Info.ptrVal = nullptr;
4865:     Info.offset = 0;
4866:     Info.flags = MachineMemOperand::MOLoad;
4867:     Info.align = Align(16);
4868:     Infos.push_back(Info);
4869:     return;
4870:
4871:   case Intrinsic::nvvm_tex_1d_v4s32_s32:
4872:   case Intrinsic::nvvm_tex_1d_v4s32_f32:
4873:   case Intrinsic::nvvm_tex_1d_level_v4s32_f32:
4874:   case Intrinsic::nvvm_tex_1d_grad_v4s32_f32:
4875:   case Intrinsic::nvvm_tex_1d_array_v4s32_s32:
4876:   case Intrinsic::nvvm_tex_1d_array_v4s32_f32:
4877:   case Intrinsic::nvvm_tex_1d_array_level_v4s32_f32:
4878:   case Intrinsic::nvvm_tex_1d_array_grad_v4s32_f32:
4879:   case Intrinsic::nvvm_tex_2d_v4s32_s32:
4880:   case Intrinsic::nvvm_tex_2d_v4s32_f32:
4881:   case Intrinsic::nvvm_tex_2d_level_v4s32_f32:
4882:   case Intrinsic::nvvm_tex_2d_grad_v4s32_f32:
4883:   case Intrinsic::nvvm_tex_2d_array_v4s32_s32:
4884:   case Intrinsic::nvvm_tex_2d_array_v4s32_f32:
4885:   case Intrinsic::nvvm_tex_2d_array_level_v4s32_f32:
4886:   case Intrinsic::nvvm_tex_2d_array_grad_v4s32_f32:
4887:   case Intrinsic::nvvm_tex_3d_v4s32_s32:
4888:   case Intrinsic::nvvm_tex_3d_v4s32_f32:
4889:   case Intrinsic::nvvm_tex_3d_level_v4s32_f32:
4890:   case Intrinsic::nvvm_tex_3d_grad_v4s32_f32:
4891:   case Intrinsic::nvvm_tex_cube_v4s32_f32:
4892:   case Intrinsic::nvvm_tex_cube_level_v4s32_f32:
4893:   case Intrinsic::nvvm_tex_cube_array_v4s32_f32:
4894:   case Intrinsic::nvvm_tex_cube_array_level_v4s32_f32:
4895:   case Intrinsic::nvvm_tex_cube_v4u32_f32:
4896:   case Intrinsic::nvvm_tex_cube_level_v4u32_f32:
4897:   case Intrinsic::nvvm_tex_cube_array_v4u32_f32:
4898:   case Intrinsic::nvvm_tex_cube_array_level_v4u32_f32:
4899:   case Intrinsic::nvvm_tex_1d_v4u32_s32:
4900:   case Intrinsic::nvvm_tex_1d_v4u32_f32:
```
- EN: This range implements operational logic in helpers such as getArgOperand, reset, push_back, getValueType, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、reset、push_back、getValueType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 4901-5040
```cpp
4901:   case Intrinsic::nvvm_tex_1d_level_v4u32_f32:
4902:   case Intrinsic::nvvm_tex_1d_grad_v4u32_f32:
4903:   case Intrinsic::nvvm_tex_1d_array_v4u32_s32:
4904:   case Intrinsic::nvvm_tex_1d_array_v4u32_f32:
4905:   case Intrinsic::nvvm_tex_1d_array_level_v4u32_f32:
4906:   case Intrinsic::nvvm_tex_1d_array_grad_v4u32_f32:
4907:   case Intrinsic::nvvm_tex_2d_v4u32_s32:
4908:   case Intrinsic::nvvm_tex_2d_v4u32_f32:
4909:   case Intrinsic::nvvm_tex_2d_level_v4u32_f32:
4910:   case Intrinsic::nvvm_tex_2d_grad_v4u32_f32:
4911:   case Intrinsic::nvvm_tex_2d_array_v4u32_s32:
4912:   case Intrinsic::nvvm_tex_2d_array_v4u32_f32:
4913:   case Intrinsic::nvvm_tex_2d_array_level_v4u32_f32:
4914:   case Intrinsic::nvvm_tex_2d_array_grad_v4u32_f32:
4915:   case Intrinsic::nvvm_tex_3d_v4u32_s32:
4916:   case Intrinsic::nvvm_tex_3d_v4u32_f32:
4917:   case Intrinsic::nvvm_tex_3d_level_v4u32_f32:
4918:   case Intrinsic::nvvm_tex_3d_grad_v4u32_f32:
4919:   case Intrinsic::nvvm_tld4_r_2d_v4s32_f32:
4920:   case Intrinsic::nvvm_tld4_g_2d_v4s32_f32:
4921:   case Intrinsic::nvvm_tld4_b_2d_v4s32_f32:
4922:   case Intrinsic::nvvm_tld4_a_2d_v4s32_f32:
4923:   case Intrinsic::nvvm_tld4_r_2d_v4u32_f32:
4924:   case Intrinsic::nvvm_tld4_g_2d_v4u32_f32:
4925:   case Intrinsic::nvvm_tld4_b_2d_v4u32_f32:
4926:   case Intrinsic::nvvm_tld4_a_2d_v4u32_f32:
4927:   case Intrinsic::nvvm_tex_unified_1d_v4s32_s32:
4928:   case Intrinsic::nvvm_tex_unified_1d_v4s32_f32:
4929:   case Intrinsic::nvvm_tex_unified_1d_level_v4s32_f32:
4930:   case Intrinsic::nvvm_tex_unified_1d_grad_v4s32_f32:
4931:   case Intrinsic::nvvm_tex_unified_1d_array_v4s32_s32:
4932:   case Intrinsic::nvvm_tex_unified_1d_array_v4s32_f32:
4933:   case Intrinsic::nvvm_tex_unified_1d_array_level_v4s32_f32:
4934:   case Intrinsic::nvvm_tex_unified_1d_array_grad_v4s32_f32:
4935:   case Intrinsic::nvvm_tex_unified_2d_v4s32_s32:
4936:   case Intrinsic::nvvm_tex_unified_2d_v4s32_f32:
4937:   case Intrinsic::nvvm_tex_unified_2d_level_v4s32_f32:
4938:   case Intrinsic::nvvm_tex_unified_2d_grad_v4s32_f32:
4939:   case Intrinsic::nvvm_tex_unified_2d_array_v4s32_s32:
4940:   case Intrinsic::nvvm_tex_unified_2d_array_v4s32_f32:
4941:   case Intrinsic::nvvm_tex_unified_2d_array_level_v4s32_f32:
4942:   case Intrinsic::nvvm_tex_unified_2d_array_grad_v4s32_f32:
4943:   case Intrinsic::nvvm_tex_unified_3d_v4s32_s32:
4944:   case Intrinsic::nvvm_tex_unified_3d_v4s32_f32:
4945:   case Intrinsic::nvvm_tex_unified_3d_level_v4s32_f32:
4946:   case Intrinsic::nvvm_tex_unified_3d_grad_v4s32_f32:
4947:   case Intrinsic::nvvm_tex_unified_1d_v4u32_s32:
4948:   case Intrinsic::nvvm_tex_unified_1d_v4u32_f32:
4949:   case Intrinsic::nvvm_tex_unified_1d_level_v4u32_f32:
4950:   case Intrinsic::nvvm_tex_unified_1d_grad_v4u32_f32:
4951:   case Intrinsic::nvvm_tex_unified_1d_array_v4u32_s32:
4952:   case Intrinsic::nvvm_tex_unified_1d_array_v4u32_f32:
4953:   case Intrinsic::nvvm_tex_unified_1d_array_level_v4u32_f32:
4954:   case Intrinsic::nvvm_tex_unified_1d_array_grad_v4u32_f32:
4955:   case Intrinsic::nvvm_tex_unified_2d_v4u32_s32:
4956:   case Intrinsic::nvvm_tex_unified_2d_v4u32_f32:
4957:   case Intrinsic::nvvm_tex_unified_2d_level_v4u32_f32:
4958:   case Intrinsic::nvvm_tex_unified_2d_grad_v4u32_f32:
4959:   case Intrinsic::nvvm_tex_unified_2d_array_v4u32_s32:
4960:   case Intrinsic::nvvm_tex_unified_2d_array_v4u32_f32:
4961:   case Intrinsic::nvvm_tex_unified_2d_array_level_v4u32_f32:
4962:   case Intrinsic::nvvm_tex_unified_2d_array_grad_v4u32_f32:
4963:   case Intrinsic::nvvm_tex_unified_3d_v4u32_s32:
4964:   case Intrinsic::nvvm_tex_unified_3d_v4u32_f32:
4965:   case Intrinsic::nvvm_tex_unified_3d_level_v4u32_f32:
4966:   case Intrinsic::nvvm_tex_unified_3d_grad_v4u32_f32:
4967:   case Intrinsic::nvvm_tex_unified_cube_v4s32_f32:
4968:   case Intrinsic::nvvm_tex_unified_cube_level_v4s32_f32:
4969:   case Intrinsic::nvvm_tex_unified_cube_array_v4s32_f32:
4970:   case Intrinsic::nvvm_tex_unified_cube_array_level_v4s32_f32:
4971:   case Intrinsic::nvvm_tex_unified_cube_v4u32_f32:
4972:   case Intrinsic::nvvm_tex_unified_cube_level_v4u32_f32:
4973:   case Intrinsic::nvvm_tex_unified_cube_array_v4u32_f32:
4974:   case Intrinsic::nvvm_tex_unified_cube_array_level_v4u32_f32:
4975:   case Intrinsic::nvvm_tex_unified_cube_grad_v4s32_f32:
4976:   case Intrinsic::nvvm_tex_unified_cube_grad_v4u32_f32:
4977:   case Intrinsic::nvvm_tex_unified_cube_array_grad_v4s32_f32:
4978:   case Intrinsic::nvvm_tex_unified_cube_array_grad_v4u32_f32:
4979:   case Intrinsic::nvvm_tld4_unified_r_2d_v4s32_f32:
4980:   case Intrinsic::nvvm_tld4_unified_g_2d_v4s32_f32:
4981:   case Intrinsic::nvvm_tld4_unified_b_2d_v4s32_f32:
4982:   case Intrinsic::nvvm_tld4_unified_a_2d_v4s32_f32:
4983:   case Intrinsic::nvvm_tld4_unified_r_2d_v4u32_f32:
4984:   case Intrinsic::nvvm_tld4_unified_g_2d_v4u32_f32:
4985:   case Intrinsic::nvvm_tld4_unified_b_2d_v4u32_f32:
4986:   case Intrinsic::nvvm_tld4_unified_a_2d_v4u32_f32:
4987:     Info.opc = ISD::INTRINSIC_W_CHAIN;
4988:     Info.memVT = MVT::v4i32;
4989:     Info.ptrVal = nullptr;
4990:     Info.offset = 0;
4991:     Info.flags = MachineMemOperand::MOLoad;
4992:     Info.align = Align(16);
4993:     Infos.push_back(Info);
4994:     return;
4995:
4996:   case Intrinsic::nvvm_suld_1d_i8_clamp:
4997:   case Intrinsic::nvvm_suld_1d_v2i8_clamp:
4998:   case Intrinsic::nvvm_suld_1d_v4i8_clamp:
4999:   case Intrinsic::nvvm_suld_1d_array_i8_clamp:
5000:   case Intrinsic::nvvm_suld_1d_array_v2i8_clamp:
5001:   case Intrinsic::nvvm_suld_1d_array_v4i8_clamp:
5002:   case Intrinsic::nvvm_suld_2d_i8_clamp:
5003:   case Intrinsic::nvvm_suld_2d_v2i8_clamp:
5004:   case Intrinsic::nvvm_suld_2d_v4i8_clamp:
5005:   case Intrinsic::nvvm_suld_2d_array_i8_clamp:
5006:   case Intrinsic::nvvm_suld_2d_array_v2i8_clamp:
5007:   case Intrinsic::nvvm_suld_2d_array_v4i8_clamp:
5008:   case Intrinsic::nvvm_suld_3d_i8_clamp:
5009:   case Intrinsic::nvvm_suld_3d_v2i8_clamp:
5010:   case Intrinsic::nvvm_suld_3d_v4i8_clamp:
5011:   case Intrinsic::nvvm_suld_1d_i8_trap:
5012:   case Intrinsic::nvvm_suld_1d_v2i8_trap:
5013:   case Intrinsic::nvvm_suld_1d_v4i8_trap:
5014:   case Intrinsic::nvvm_suld_1d_array_i8_trap:
5015:   case Intrinsic::nvvm_suld_1d_array_v2i8_trap:
5016:   case Intrinsic::nvvm_suld_1d_array_v4i8_trap:
5017:   case Intrinsic::nvvm_suld_2d_i8_trap:
5018:   case Intrinsic::nvvm_suld_2d_v2i8_trap:
5019:   case Intrinsic::nvvm_suld_2d_v4i8_trap:
5020:   case Intrinsic::nvvm_suld_2d_array_i8_trap:
5021:   case Intrinsic::nvvm_suld_2d_array_v2i8_trap:
5022:   case Intrinsic::nvvm_suld_2d_array_v4i8_trap:
5023:   case Intrinsic::nvvm_suld_3d_i8_trap:
5024:   case Intrinsic::nvvm_suld_3d_v2i8_trap:
5025:   case Intrinsic::nvvm_suld_3d_v4i8_trap:
5026:   case Intrinsic::nvvm_suld_1d_i8_zero:
5027:   case Intrinsic::nvvm_suld_1d_v2i8_zero:
5028:   case Intrinsic::nvvm_suld_1d_v4i8_zero:
5029:   case Intrinsic::nvvm_suld_1d_array_i8_zero:
5030:   case Intrinsic::nvvm_suld_1d_array_v2i8_zero:
5031:   case Intrinsic::nvvm_suld_1d_array_v4i8_zero:
5032:   case Intrinsic::nvvm_suld_2d_i8_zero:
5033:   case Intrinsic::nvvm_suld_2d_v2i8_zero:
5034:   case Intrinsic::nvvm_suld_2d_v4i8_zero:
5035:   case Intrinsic::nvvm_suld_2d_array_i8_zero:
5036:   case Intrinsic::nvvm_suld_2d_array_v2i8_zero:
5037:   case Intrinsic::nvvm_suld_2d_array_v4i8_zero:
5038:   case Intrinsic::nvvm_suld_3d_i8_zero:
5039:   case Intrinsic::nvvm_suld_3d_v2i8_zero:
5040:   case Intrinsic::nvvm_suld_3d_v4i8_zero:
```
- EN: This range implements operational logic in helpers such as Align, push_back, translating backend policy into executable code.
- CN: 这一段实现了 Align、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5041-5180
```cpp
5041:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5042:     Info.memVT = MVT::i8;
5043:     Info.ptrVal = nullptr;
5044:     Info.offset = 0;
5045:     Info.flags = MachineMemOperand::MOLoad;
5046:     Info.align = Align(16);
5047:     Infos.push_back(Info);
5048:     return;
5049:
5050:   case Intrinsic::nvvm_suld_1d_i16_clamp:
5051:   case Intrinsic::nvvm_suld_1d_v2i16_clamp:
5052:   case Intrinsic::nvvm_suld_1d_v4i16_clamp:
5053:   case Intrinsic::nvvm_suld_1d_array_i16_clamp:
5054:   case Intrinsic::nvvm_suld_1d_array_v2i16_clamp:
5055:   case Intrinsic::nvvm_suld_1d_array_v4i16_clamp:
5056:   case Intrinsic::nvvm_suld_2d_i16_clamp:
5057:   case Intrinsic::nvvm_suld_2d_v2i16_clamp:
5058:   case Intrinsic::nvvm_suld_2d_v4i16_clamp:
5059:   case Intrinsic::nvvm_suld_2d_array_i16_clamp:
5060:   case Intrinsic::nvvm_suld_2d_array_v2i16_clamp:
5061:   case Intrinsic::nvvm_suld_2d_array_v4i16_clamp:
5062:   case Intrinsic::nvvm_suld_3d_i16_clamp:
5063:   case Intrinsic::nvvm_suld_3d_v2i16_clamp:
5064:   case Intrinsic::nvvm_suld_3d_v4i16_clamp:
5065:   case Intrinsic::nvvm_suld_1d_i16_trap:
5066:   case Intrinsic::nvvm_suld_1d_v2i16_trap:
5067:   case Intrinsic::nvvm_suld_1d_v4i16_trap:
5068:   case Intrinsic::nvvm_suld_1d_array_i16_trap:
5069:   case Intrinsic::nvvm_suld_1d_array_v2i16_trap:
5070:   case Intrinsic::nvvm_suld_1d_array_v4i16_trap:
5071:   case Intrinsic::nvvm_suld_2d_i16_trap:
5072:   case Intrinsic::nvvm_suld_2d_v2i16_trap:
5073:   case Intrinsic::nvvm_suld_2d_v4i16_trap:
5074:   case Intrinsic::nvvm_suld_2d_array_i16_trap:
5075:   case Intrinsic::nvvm_suld_2d_array_v2i16_trap:
5076:   case Intrinsic::nvvm_suld_2d_array_v4i16_trap:
5077:   case Intrinsic::nvvm_suld_3d_i16_trap:
5078:   case Intrinsic::nvvm_suld_3d_v2i16_trap:
5079:   case Intrinsic::nvvm_suld_3d_v4i16_trap:
5080:   case Intrinsic::nvvm_suld_1d_i16_zero:
5081:   case Intrinsic::nvvm_suld_1d_v2i16_zero:
5082:   case Intrinsic::nvvm_suld_1d_v4i16_zero:
5083:   case Intrinsic::nvvm_suld_1d_array_i16_zero:
5084:   case Intrinsic::nvvm_suld_1d_array_v2i16_zero:
5085:   case Intrinsic::nvvm_suld_1d_array_v4i16_zero:
5086:   case Intrinsic::nvvm_suld_2d_i16_zero:
5087:   case Intrinsic::nvvm_suld_2d_v2i16_zero:
5088:   case Intrinsic::nvvm_suld_2d_v4i16_zero:
5089:   case Intrinsic::nvvm_suld_2d_array_i16_zero:
5090:   case Intrinsic::nvvm_suld_2d_array_v2i16_zero:
5091:   case Intrinsic::nvvm_suld_2d_array_v4i16_zero:
5092:   case Intrinsic::nvvm_suld_3d_i16_zero:
5093:   case Intrinsic::nvvm_suld_3d_v2i16_zero:
5094:   case Intrinsic::nvvm_suld_3d_v4i16_zero:
5095:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5096:     Info.memVT = MVT::i16;
5097:     Info.ptrVal = nullptr;
5098:     Info.offset = 0;
5099:     Info.flags = MachineMemOperand::MOLoad;
5100:     Info.align = Align(16);
5101:     Infos.push_back(Info);
5102:     return;
5103:
5104:   case Intrinsic::nvvm_suld_1d_i32_clamp:
5105:   case Intrinsic::nvvm_suld_1d_v2i32_clamp:
5106:   case Intrinsic::nvvm_suld_1d_v4i32_clamp:
5107:   case Intrinsic::nvvm_suld_1d_array_i32_clamp:
5108:   case Intrinsic::nvvm_suld_1d_array_v2i32_clamp:
5109:   case Intrinsic::nvvm_suld_1d_array_v4i32_clamp:
5110:   case Intrinsic::nvvm_suld_2d_i32_clamp:
5111:   case Intrinsic::nvvm_suld_2d_v2i32_clamp:
5112:   case Intrinsic::nvvm_suld_2d_v4i32_clamp:
5113:   case Intrinsic::nvvm_suld_2d_array_i32_clamp:
5114:   case Intrinsic::nvvm_suld_2d_array_v2i32_clamp:
5115:   case Intrinsic::nvvm_suld_2d_array_v4i32_clamp:
5116:   case Intrinsic::nvvm_suld_3d_i32_clamp:
5117:   case Intrinsic::nvvm_suld_3d_v2i32_clamp:
5118:   case Intrinsic::nvvm_suld_3d_v4i32_clamp:
5119:   case Intrinsic::nvvm_suld_1d_i32_trap:
5120:   case Intrinsic::nvvm_suld_1d_v2i32_trap:
5121:   case Intrinsic::nvvm_suld_1d_v4i32_trap:
5122:   case Intrinsic::nvvm_suld_1d_array_i32_trap:
5123:   case Intrinsic::nvvm_suld_1d_array_v2i32_trap:
5124:   case Intrinsic::nvvm_suld_1d_array_v4i32_trap:
5125:   case Intrinsic::nvvm_suld_2d_i32_trap:
5126:   case Intrinsic::nvvm_suld_2d_v2i32_trap:
5127:   case Intrinsic::nvvm_suld_2d_v4i32_trap:
5128:   case Intrinsic::nvvm_suld_2d_array_i32_trap:
5129:   case Intrinsic::nvvm_suld_2d_array_v2i32_trap:
5130:   case Intrinsic::nvvm_suld_2d_array_v4i32_trap:
5131:   case Intrinsic::nvvm_suld_3d_i32_trap:
5132:   case Intrinsic::nvvm_suld_3d_v2i32_trap:
5133:   case Intrinsic::nvvm_suld_3d_v4i32_trap:
5134:   case Intrinsic::nvvm_suld_1d_i32_zero:
5135:   case Intrinsic::nvvm_suld_1d_v2i32_zero:
5136:   case Intrinsic::nvvm_suld_1d_v4i32_zero:
5137:   case Intrinsic::nvvm_suld_1d_array_i32_zero:
5138:   case Intrinsic::nvvm_suld_1d_array_v2i32_zero:
5139:   case Intrinsic::nvvm_suld_1d_array_v4i32_zero:
5140:   case Intrinsic::nvvm_suld_2d_i32_zero:
5141:   case Intrinsic::nvvm_suld_2d_v2i32_zero:
5142:   case Intrinsic::nvvm_suld_2d_v4i32_zero:
5143:   case Intrinsic::nvvm_suld_2d_array_i32_zero:
5144:   case Intrinsic::nvvm_suld_2d_array_v2i32_zero:
5145:   case Intrinsic::nvvm_suld_2d_array_v4i32_zero:
5146:   case Intrinsic::nvvm_suld_3d_i32_zero:
5147:   case Intrinsic::nvvm_suld_3d_v2i32_zero:
5148:   case Intrinsic::nvvm_suld_3d_v4i32_zero:
5149:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5150:     Info.memVT = MVT::i32;
5151:     Info.ptrVal = nullptr;
5152:     Info.offset = 0;
5153:     Info.flags = MachineMemOperand::MOLoad;
5154:     Info.align = Align(16);
5155:     Infos.push_back(Info);
5156:     return;
5157:
5158:   case Intrinsic::nvvm_suld_1d_i64_clamp:
5159:   case Intrinsic::nvvm_suld_1d_v2i64_clamp:
5160:   case Intrinsic::nvvm_suld_1d_array_i64_clamp:
5161:   case Intrinsic::nvvm_suld_1d_array_v2i64_clamp:
5162:   case Intrinsic::nvvm_suld_2d_i64_clamp:
5163:   case Intrinsic::nvvm_suld_2d_v2i64_clamp:
5164:   case Intrinsic::nvvm_suld_2d_array_i64_clamp:
5165:   case Intrinsic::nvvm_suld_2d_array_v2i64_clamp:
5166:   case Intrinsic::nvvm_suld_3d_i64_clamp:
5167:   case Intrinsic::nvvm_suld_3d_v2i64_clamp:
5168:   case Intrinsic::nvvm_suld_1d_i64_trap:
5169:   case Intrinsic::nvvm_suld_1d_v2i64_trap:
5170:   case Intrinsic::nvvm_suld_1d_array_i64_trap:
5171:   case Intrinsic::nvvm_suld_1d_array_v2i64_trap:
5172:   case Intrinsic::nvvm_suld_2d_i64_trap:
5173:   case Intrinsic::nvvm_suld_2d_v2i64_trap:
5174:   case Intrinsic::nvvm_suld_2d_array_i64_trap:
5175:   case Intrinsic::nvvm_suld_2d_array_v2i64_trap:
5176:   case Intrinsic::nvvm_suld_3d_i64_trap:
5177:   case Intrinsic::nvvm_suld_3d_v2i64_trap:
5178:   case Intrinsic::nvvm_suld_1d_i64_zero:
5179:   case Intrinsic::nvvm_suld_1d_v2i64_zero:
5180:   case Intrinsic::nvvm_suld_1d_array_i64_zero:
```
- EN: This range implements operational logic in helpers such as Align, push_back, translating backend policy into executable code.
- CN: 这一段实现了 Align、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5181-5320
```cpp
5181:   case Intrinsic::nvvm_suld_1d_array_v2i64_zero:
5182:   case Intrinsic::nvvm_suld_2d_i64_zero:
5183:   case Intrinsic::nvvm_suld_2d_v2i64_zero:
5184:   case Intrinsic::nvvm_suld_2d_array_i64_zero:
5185:   case Intrinsic::nvvm_suld_2d_array_v2i64_zero:
5186:   case Intrinsic::nvvm_suld_3d_i64_zero:
5187:   case Intrinsic::nvvm_suld_3d_v2i64_zero:
5188:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5189:     Info.memVT = MVT::i64;
5190:     Info.ptrVal = nullptr;
5191:     Info.offset = 0;
5192:     Info.flags = MachineMemOperand::MOLoad;
5193:     Info.align = Align(16);
5194:     Infos.push_back(Info);
5195:     return;
5196:
5197:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x1:
5198:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x1:
5199:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x1: {
5200:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5201:     Info.memVT = MVT::v1i32;
5202:     Info.ptrVal = I.getArgOperand(0);
5203:     Info.offset = 0;
5204:     Info.flags = MachineMemOperand::MOLoad;
5205:     Info.align.reset();
5206:     Infos.push_back(Info);
5207:     return;
5208:   }
5209:
5210:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x2:
5211:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x1:
5212:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x2:
5213:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x2:
5214:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x2_i32:
5215:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x2_i32: {
5216:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5217:     Info.memVT = MVT::v2i32;
5218:     Info.ptrVal = I.getArgOperand(0);
5219:     Info.offset = 0;
5220:     Info.flags = MachineMemOperand::MOLoad;
5221:     Info.align.reset();
5222:     Infos.push_back(Info);
5223:     return;
5224:   }
5225:
5226:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x2_f32:
5227:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x2_f32: {
5228:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5229:     Info.memVT = MVT::v2f32;
5230:     Info.ptrVal = I.getArgOperand(0);
5231:     Info.offset = 0;
5232:     Info.flags = MachineMemOperand::MOLoad;
5233:     Info.align.reset();
5234:     Infos.push_back(Info);
5235:     return;
5236:   }
5237:
5238:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x4:
5239:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x2:
5240:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x4:
5241:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x1:
5242:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x4:
5243:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x4_i32:
5244:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x4_i32: {
5245:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5246:     Info.memVT = MVT::v4i32;
5247:     Info.ptrVal = I.getArgOperand(0);
5248:     Info.offset = 0;
5249:     Info.flags = MachineMemOperand::MOLoad;
5250:     Info.align.reset();
5251:     Infos.push_back(Info);
5252:     return;
5253:   }
5254:
5255:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x4_f32:
5256:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x4_f32: {
5257:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5258:     Info.memVT = MVT::v4f32;
5259:     Info.ptrVal = I.getArgOperand(0);
5260:     Info.offset = 0;
5261:     Info.flags = MachineMemOperand::MOLoad;
5262:     Info.align.reset();
5263:     Infos.push_back(Info);
5264:     return;
5265:   }
5266:
5267:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x8:
5268:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x4:
5269:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x2:
5270:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x8:
5271:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x8:
5272:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x8_i32:
5273:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x8_i32: {
5274:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5275:     Info.memVT = MVT::v8i32;
5276:     Info.ptrVal = I.getArgOperand(0);
5277:     Info.offset = 0;
5278:     Info.flags = MachineMemOperand::MOLoad;
5279:     Info.align.reset();
5280:     Infos.push_back(Info);
5281:     return;
5282:   }
5283:
5284:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x8_f32:
5285:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x8_f32: {
5286:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5287:     Info.memVT = MVT::v8f32;
5288:     Info.ptrVal = I.getArgOperand(0);
5289:     Info.offset = 0;
5290:     Info.flags = MachineMemOperand::MOLoad;
5291:     Info.align.reset();
5292:     Infos.push_back(Info);
5293:     return;
5294:   }
5295:
5296:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x16:
5297:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x8:
5298:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x4:
5299:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x16:
5300:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x16:
5301:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x16_i32:
5302:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x16_i32: {
5303:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5304:     Info.memVT = MVT::v16i32;
5305:     Info.ptrVal = I.getArgOperand(0);
5306:     Info.offset = 0;
5307:     Info.flags = MachineMemOperand::MOLoad;
5308:     Info.align.reset();
5309:     Infos.push_back(Info);
5310:     return;
5311:   }
5312:
5313:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x16_f32:
5314:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x16_f32: {
5315:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5316:     Info.memVT = MVT::v16f32;
5317:     Info.ptrVal = I.getArgOperand(0);
5318:     Info.offset = 0;
5319:     Info.flags = MachineMemOperand::MOLoad;
5320:     Info.align.reset();
```
- EN: This range implements operational logic in helpers such as Align, push_back, getArgOperand, reset, translating backend policy into executable code.
- CN: 这一段实现了 Align、push_back、getArgOperand、reset 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5321-5460
```cpp
5321:     Infos.push_back(Info);
5322:     return;
5323:   }
5324:
5325:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x32:
5326:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x16:
5327:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x8:
5328:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x32:
5329:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x32:
5330:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x32_i32:
5331:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x32_i32: {
5332:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5333:     Info.memVT = MVT::v32i32;
5334:     Info.ptrVal = I.getArgOperand(0);
5335:     Info.offset = 0;
5336:     Info.flags = MachineMemOperand::MOLoad;
5337:     Info.align.reset();
5338:     Infos.push_back(Info);
5339:     return;
5340:   }
5341:
5342:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x32_f32:
5343:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x32_f32: {
5344:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5345:     Info.memVT = MVT::v32f32;
5346:     Info.ptrVal = I.getArgOperand(0);
5347:     Info.offset = 0;
5348:     Info.flags = MachineMemOperand::MOLoad;
5349:     Info.align.reset();
5350:     Infos.push_back(Info);
5351:     return;
5352:   }
5353:
5354:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x64:
5355:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x32:
5356:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x16:
5357:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x64:
5358:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x64:
5359:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x64_i32:
5360:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x64_i32: {
5361:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5362:     Info.memVT = MVT::v64i32;
5363:     Info.ptrVal = I.getArgOperand(0);
5364:     Info.offset = 0;
5365:     Info.flags = MachineMemOperand::MOLoad;
5366:     Info.align.reset();
5367:     Infos.push_back(Info);
5368:     return;
5369:   }
5370:
5371:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x64_f32:
5372:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x64_f32: {
5373:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5374:     Info.memVT = MVT::v64f32;
5375:     Info.ptrVal = I.getArgOperand(0);
5376:     Info.offset = 0;
5377:     Info.flags = MachineMemOperand::MOLoad;
5378:     Info.align.reset();
5379:     Infos.push_back(Info);
5380:     return;
5381:   }
5382:
5383:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x128:
5384:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x64:
5385:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x32:
5386:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x128:
5387:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x128:
5388:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x128_i32:
5389:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x128_i32: {
5390:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5391:     Info.memVT = MVT::v128i32;
5392:     Info.ptrVal = I.getArgOperand(0);
5393:     Info.offset = 0;
5394:     Info.flags = MachineMemOperand::MOLoad;
5395:     Info.align.reset();
5396:     Infos.push_back(Info);
5397:     return;
5398:   }
5399:
5400:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x128_f32:
5401:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x128_f32: {
5402:     Info.opc = ISD::INTRINSIC_W_CHAIN;
5403:     Info.memVT = MVT::v128f32;
5404:     Info.ptrVal = I.getArgOperand(0);
5405:     Info.offset = 0;
5406:     Info.flags = MachineMemOperand::MOLoad;
5407:     Info.align.reset();
5408:     Infos.push_back(Info);
5409:     return;
5410:   }
5411:
5412:   case Intrinsic::nvvm_tcgen05_st_16x64b_x1:
5413:   case Intrinsic::nvvm_tcgen05_st_32x32b_x1:
5414:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x1: {
5415:     Info.opc = ISD::INTRINSIC_VOID;
5416:     Info.memVT = MVT::i32;
5417:     Info.ptrVal = I.getArgOperand(0);
5418:     Info.offset = 0;
5419:     Info.flags = MachineMemOperand::MOStore;
5420:     Info.align.reset();
5421:     Infos.push_back(Info);
5422:     return;
5423:   }
5424:
5425:   case Intrinsic::nvvm_tcgen05_st_16x64b_x2:
5426:   case Intrinsic::nvvm_tcgen05_st_16x128b_x1:
5427:   case Intrinsic::nvvm_tcgen05_st_32x32b_x2:
5428:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x2: {
5429:     Info.opc = ISD::INTRINSIC_VOID;
5430:     Info.memVT = MVT::v2i32;
5431:     Info.ptrVal = I.getArgOperand(0);
5432:     Info.offset = 0;
5433:     Info.flags = MachineMemOperand::MOStore;
5434:     Info.align.reset();
5435:     Infos.push_back(Info);
5436:     return;
5437:   }
5438:
5439:   case Intrinsic::nvvm_tcgen05_st_16x64b_x4:
5440:   case Intrinsic::nvvm_tcgen05_st_16x128b_x2:
5441:   case Intrinsic::nvvm_tcgen05_st_16x256b_x1:
5442:   case Intrinsic::nvvm_tcgen05_st_32x32b_x4:
5443:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x4: {
5444:     Info.opc = ISD::INTRINSIC_VOID;
5445:     Info.memVT = MVT::v4i32;
5446:     Info.ptrVal = I.getArgOperand(0);
5447:     Info.offset = 0;
5448:     Info.flags = MachineMemOperand::MOStore;
5449:     Info.align.reset();
5450:     Infos.push_back(Info);
5451:     return;
5452:   }
5453:
5454:   case Intrinsic::nvvm_tcgen05_st_16x64b_x8:
5455:   case Intrinsic::nvvm_tcgen05_st_16x128b_x4:
5456:   case Intrinsic::nvvm_tcgen05_st_16x256b_x2:
5457:   case Intrinsic::nvvm_tcgen05_st_32x32b_x8:
5458:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x8: {
5459:     Info.opc = ISD::INTRINSIC_VOID;
5460:     Info.memVT = MVT::v8i32;
```
- EN: This range implements operational logic in helpers such as push_back, getArgOperand, reset, translating backend policy into executable code.
- CN: 这一段实现了 push_back、getArgOperand、reset 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5461-5600
```cpp
5461:     Info.ptrVal = I.getArgOperand(0);
5462:     Info.offset = 0;
5463:     Info.flags = MachineMemOperand::MOStore;
5464:     Info.align.reset();
5465:     Infos.push_back(Info);
5466:     return;
5467:   }
5468:
5469:   case Intrinsic::nvvm_tcgen05_st_16x64b_x16:
5470:   case Intrinsic::nvvm_tcgen05_st_16x128b_x8:
5471:   case Intrinsic::nvvm_tcgen05_st_16x256b_x4:
5472:   case Intrinsic::nvvm_tcgen05_st_32x32b_x16:
5473:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x16: {
5474:     Info.opc = ISD::INTRINSIC_VOID;
5475:     Info.memVT = MVT::v16i32;
5476:     Info.ptrVal = I.getArgOperand(0);
5477:     Info.offset = 0;
5478:     Info.flags = MachineMemOperand::MOStore;
5479:     Info.align.reset();
5480:     Infos.push_back(Info);
5481:     return;
5482:   }
5483:
5484:   case Intrinsic::nvvm_tcgen05_st_16x64b_x32:
5485:   case Intrinsic::nvvm_tcgen05_st_16x128b_x16:
5486:   case Intrinsic::nvvm_tcgen05_st_16x256b_x8:
5487:   case Intrinsic::nvvm_tcgen05_st_32x32b_x32:
5488:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x32: {
5489:     Info.opc = ISD::INTRINSIC_VOID;
5490:     Info.memVT = MVT::v32i32;
5491:     Info.ptrVal = I.getArgOperand(0);
5492:     Info.offset = 0;
5493:     Info.flags = MachineMemOperand::MOStore;
5494:     Info.align.reset();
5495:     Infos.push_back(Info);
5496:     return;
5497:   }
5498:
5499:   case Intrinsic::nvvm_tcgen05_st_16x64b_x64:
5500:   case Intrinsic::nvvm_tcgen05_st_16x128b_x32:
5501:   case Intrinsic::nvvm_tcgen05_st_16x256b_x16:
5502:   case Intrinsic::nvvm_tcgen05_st_32x32b_x64:
5503:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x64: {
5504:     Info.opc = ISD::INTRINSIC_VOID;
5505:     Info.memVT = MVT::v64i32;
5506:     Info.ptrVal = I.getArgOperand(0);
5507:     Info.offset = 0;
5508:     Info.flags = MachineMemOperand::MOStore;
5509:     Info.align.reset();
5510:     Infos.push_back(Info);
5511:     return;
5512:   }
5513:
5514:   case Intrinsic::nvvm_tcgen05_st_16x64b_x128:
5515:   case Intrinsic::nvvm_tcgen05_st_16x128b_x64:
5516:   case Intrinsic::nvvm_tcgen05_st_16x256b_x32:
5517:   case Intrinsic::nvvm_tcgen05_st_32x32b_x128:
5518:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x128: {
5519:     Info.opc = ISD::INTRINSIC_VOID;
5520:     Info.memVT = MVT::v128i32;
5521:     Info.ptrVal = I.getArgOperand(0);
5522:     Info.offset = 0;
5523:     Info.flags = MachineMemOperand::MOStore;
5524:     Info.align.reset();
5525:     Infos.push_back(Info);
5526:     return;
5527:   }
5528:   case Intrinsic::nvvm_tcgen05_mma_shared_disable_output_lane_cg1:
5529:   case Intrinsic::nvvm_tcgen05_mma_shared_scale_d_disable_output_lane_cg1:
5530:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_disable_output_lane_cg1:
5531:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_scale_d_disable_output_lane_cg1:
5532:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg1:
5533:   case Intrinsic::nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg1:
5534:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg1_ashift:
5535:   case Intrinsic::
5536:       nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg1_ashift:
5537:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg1:
5538:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg1:
5539:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg1_ashift:
5540:   case Intrinsic::
5541:       nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg1_ashift: {
5542:     // We are reading and writing back to TMem
5543:     Info.opc = ISD::INTRINSIC_VOID;
5544:     Info.memVT = MVT::v4i32;
5545:     Info.ptrVal = I.getArgOperand(0);
5546:     Info.offset = 0;
5547:     Info.flags = MachineMemOperand::MOLoad | MachineMemOperand::MOStore;
5548:     Info.align = Align(16);
5549:     Infos.push_back(Info);
5550:     return;
5551:   }
5552:
5553:   case Intrinsic::nvvm_tcgen05_mma_shared_disable_output_lane_cg2:
5554:   case Intrinsic::nvvm_tcgen05_mma_shared_scale_d_disable_output_lane_cg2:
5555:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_disable_output_lane_cg2:
5556:   case Intrinsic::nvvm_tcgen05_mma_sp_shared_scale_d_disable_output_lane_cg2:
5557:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg2:
5558:   case Intrinsic::nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg2:
5559:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg2:
5560:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg2:
5561:   case Intrinsic::nvvm_tcgen05_mma_tensor_disable_output_lane_cg2_ashift:
5562:   case Intrinsic::
5563:       nvvm_tcgen05_mma_tensor_scale_d_disable_output_lane_cg2_ashift:
5564:   case Intrinsic::nvvm_tcgen05_mma_sp_tensor_disable_output_lane_cg2_ashift:
5565:   case Intrinsic::
5566:       nvvm_tcgen05_mma_sp_tensor_scale_d_disable_output_lane_cg2_ashift: {
5567:     // We are reading and writing back to TMem
5568:     Info.opc = ISD::INTRINSIC_VOID;
5569:     Info.memVT = MVT::v8i32;
5570:     Info.ptrVal = I.getArgOperand(0);
5571:     Info.offset = 0;
5572:     Info.flags = MachineMemOperand::MOLoad | MachineMemOperand::MOStore;
5573:     Info.align = Align(16);
5574:     Infos.push_back(Info);
5575:     return;
5576:   }
5577:   }
5578: }
5579:
5580: // Helper for getting a function parameter name. Name is composed from
5581: // its index and the function name. Negative index corresponds to special
5582: // parameter (unsized array) used for passing variable arguments.
5583: std::string NVPTXTargetLowering::getParamName(const Function *F,
5584:                                               int Idx) const {
5585:   std::string ParamName;
5586:   raw_string_ostream ParamStr(ParamName);
5587:
5588:   ParamStr << getTargetMachine().getSymbol(F)->getName();
5589:   if (Idx < 0)
5590:     ParamStr << "_vararg";
5591:   else
5592:     ParamStr << "_param_" << Idx;
5593:
5594:   return ParamName;
5595: }
5596:
5597: /// isLegalAddressingMode - Return true if the addressing mode represented
5598: /// by AM is legal for this target, for a load/store of the specified type.
5599: /// Used to guide target specific optimizations, like loop strength reduction
5600: /// (LoopStrengthReduce.cpp) and memory optimization for address mode
```
- EN: This range implements operational logic in helpers such as getArgOperand, reset, push_back, Align, translating backend policy into executable code.
- CN: 这一段实现了 getArgOperand、reset、push_back、Align 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5601-5740
```cpp
5601: /// (CodeGenPrepare.cpp)
5602: bool NVPTXTargetLowering::isLegalAddressingMode(const DataLayout &DL,
5603:                                                 const AddrMode &AM, Type *Ty,
5604:                                                 unsigned AS, Instruction *I) const {
5605:   // AddrMode - This represents an addressing mode of:
5606:   //    BaseGV + BaseOffs + BaseReg + Scale*ScaleReg
5607:   //
5608:   // The legal address modes are
5609:   // - [avar]
5610:   // - [areg]
5611:   // - [areg+immoff]
5612:   // - [immAddr]
5613:
5614:   // immoff must fit in a signed 32-bit int
5615:   if (!APInt(64, AM.BaseOffs).isSignedIntN(32))
5616:     return false;
5617:
5618:   if (AM.BaseGV)
5619:     return !AM.BaseOffs && !AM.HasBaseReg && !AM.Scale;
5620:
5621:   switch (AM.Scale) {
5622:   case 0: // "r", "r+i" or "i" is allowed
5623:     break;
5624:   case 1:
5625:     if (AM.HasBaseReg) // "r+r+i" or "r+r" is not allowed.
5626:       return false;
5627:     // Otherwise we have r+i.
5628:     break;
5629:   default:
5630:     // No scale > 1 is allowed
5631:     return false;
5632:   }
5633:   return true;
5634: }
5635:
5636: //===----------------------------------------------------------------------===//
5637: //                         NVPTX Inline Assembly Support
5638: //===----------------------------------------------------------------------===//
5639:
5640: /// getConstraintType - Given a constraint letter, return the type of
5641: /// constraint it is for this target.
5642: NVPTXTargetLowering::ConstraintType
5643: NVPTXTargetLowering::getConstraintType(StringRef Constraint) const {
5644:   if (Constraint.size() == 1) {
5645:     switch (Constraint[0]) {
5646:     default:
5647:       break;
5648:     case 'b':
5649:     case 'r':
5650:     case 'h':
5651:     case 'c':
5652:     case 'l':
5653:     case 'f':
5654:     case 'd':
5655:     case 'q':
5656:     case '0':
5657:     case 'N':
5658:       return C_RegisterClass;
5659:     }
5660:   }
5661:   return TargetLowering::getConstraintType(Constraint);
5662: }
5663:
5664: std::pair<unsigned, const TargetRegisterClass *>
5665: NVPTXTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
5666:                                                   StringRef Constraint,
5667:                                                   MVT VT) const {
5668:   if (Constraint.size() == 1) {
5669:     switch (Constraint[0]) {
5670:     case 'b':
5671:       return std::make_pair(0U, &NVPTX::B1RegClass);
5672:     case 'c':
5673:     case 'h':
5674:       return std::make_pair(0U, &NVPTX::B16RegClass);
5675:     case 'r':
5676:     case 'f':
5677:       return std::make_pair(0U, &NVPTX::B32RegClass);
5678:     case 'l':
5679:     case 'N':
5680:     case 'd':
5681:       return std::make_pair(0U, &NVPTX::B64RegClass);
5682:     case 'q': {
5683:       if (STI.getSmVersion() < 70)
5684:         report_fatal_error("Inline asm with 128 bit operands is only "
5685:                            "supported for sm_70 and higher!");
5686:       return std::make_pair(0U, &NVPTX::B128RegClass);
5687:     }
5688:     }
5689:   }
5690:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
5691: }
5692:
5693: //===----------------------------------------------------------------------===//
5694: //                         NVPTX DAG Combining
5695: //===----------------------------------------------------------------------===//
5696:
5697: bool NVPTXTargetLowering::allowFMA(MachineFunction &MF,
5698:                                    CodeGenOptLevel OptLevel) const {
5699:   // Always honor command-line argument
5700:   if (FMAContractLevelOpt.getNumOccurrences() > 0)
5701:     return FMAContractLevelOpt > 0;
5702:
5703:   // Do not contract if we're not optimizing the code.
5704:   if (OptLevel == CodeGenOptLevel::None)
5705:     return false;
5706:
5707:   // Honor TargetOptions flags that explicitly say fusion is okay.
5708:   if (MF.getTarget().Options.AllowFPOpFusion == FPOpFusion::Fast)
5709:     return true;
5710:
5711:   return false;
5712: }
5713:
5714: static bool isConstZero(const SDValue &Operand) {
5715:   const auto *Const = dyn_cast<ConstantSDNode>(Operand);
5716:   return Const && Const->getZExtValue() == 0;
5717: }
5718:
5719: /// PerformADDCombineWithOperands - Try DAG combinations for an ADD with
5720: /// operands N0 and N1.  This is a helper for PerformADDCombine that is
5721: /// called with the default operands, and if that fails, with commuted
5722: /// operands.
5723: static SDValue
5724: PerformADDCombineWithOperands(SDNode *N, SDValue N0, SDValue N1,
5725:                               TargetLowering::DAGCombinerInfo &DCI) {
5726:   EVT VT = N0.getValueType();
5727:
5728:   // Since integer multiply-add costs the same as integer multiply
5729:   // but is more costly than integer add, do the fusion only when
5730:   // the mul is only used in the add.
5731:   // TODO: this may not be true for later architectures, consider relaxing this
5732:   if (!N0.getNode()->hasOneUse())
5733:     return SDValue();
5734:
5735:   // fold (add (select cond, 0, (mul a, b)), c)
5736:   //   -> (select cond, c, (add (mul a, b), c))
5737:   //
5738:   if (N0.getOpcode() == ISD::SELECT) {
5739:     unsigned ZeroOpNum;
5740:     if (isConstZero(N0->getOperand(1)))
```
- EN: This range implements operational logic in helpers such as NVPTXTargetLowering::getConstraintType, TargetLowering::getConstraintType, std::make_pair, TargetLowering::getRegForInlineAsmConstraint, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXTargetLowering::getConstraintType、TargetLowering::getConstraintType、std::make_pair、TargetLowering::getRegForInlineAsmConstraint 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5741-5880
```cpp
5741:       ZeroOpNum = 1;
5742:     else if (isConstZero(N0->getOperand(2)))
5743:       ZeroOpNum = 2;
5744:     else
5745:       return SDValue();
5746:
5747:     SDValue M = N0->getOperand((ZeroOpNum == 1) ? 2 : 1);
5748:     if (M->getOpcode() != ISD::MUL || !M.getNode()->hasOneUse())
5749:       return SDValue();
5750:
5751:     SDLoc DL(N);
5752:     SDValue Mul =
5753:         DCI.DAG.getNode(ISD::MUL, DL, VT, M->getOperand(0), M->getOperand(1));
5754:     SDValue MAD = DCI.DAG.getNode(ISD::ADD, DL, VT, Mul, N1);
5755:     return DCI.DAG.getSelect(SDLoc(N), VT, N0->getOperand(0),
5756:                              ((ZeroOpNum == 1) ? N1 : MAD),
5757:                              ((ZeroOpNum == 1) ? MAD : N1));
5758:   }
5759:
5760:   return SDValue();
5761: }
5762:
5763: SDValue NVPTXTargetLowering::performFADDCombineWithOperands(
5764:     SDNode *N, SDValue N0, SDValue N1, TargetLowering::DAGCombinerInfo &DCI,
5765:     CodeGenOptLevel OptLevel) const {
5766:   EVT VT = N0.getValueType();
5767:   if (N0.getOpcode() == ISD::FMUL) {
5768:     if (!(allowFMA(DCI.DAG.getMachineFunction(), OptLevel) ||
5769:           (N->getFlags().hasAllowContract() &&
5770:            N0->getFlags().hasAllowContract())))
5771:       return SDValue();
5772:
5773:     // For floating point:
5774:     // Do the fusion only when the mul has less than 5 uses and all
5775:     // are add.
5776:     // The heuristic is that if a use is not an add, then that use
5777:     // cannot be fused into fma, therefore mul is still needed anyway.
5778:     // If there are more than 4 uses, even if they are all add, fusing
5779:     // them will increase register pressue.
5780:     //
5781:     int numUses = 0;
5782:     int nonAddCount = 0;
5783:     for (const SDNode *User : N0.getNode()->users()) {
5784:       numUses++;
5785:       if (User->getOpcode() != ISD::FADD)
5786:         ++nonAddCount;
5787:       if (numUses >= 5)
5788:         return SDValue();
5789:     }
5790:     if (nonAddCount) {
5791:       int orderNo = N->getIROrder();
5792:       int orderNo2 = N0.getNode()->getIROrder();
5793:       // simple heuristics here for considering potential register
5794:       // pressure, the logics here is that the differnce are used
5795:       // to measure the distance between def and use, the longer distance
5796:       // more likely cause register pressure.
5797:       if (orderNo - orderNo2 < 500)
5798:         return SDValue();
5799:
5800:       // Now, check if at least one of the FMUL's operands is live beyond the
5801:       // node N, which guarantees that the FMA will not increase register
5802:       // pressure at node N.
5803:       bool opIsLive = false;
5804:       const SDNode *left = N0.getOperand(0).getNode();
5805:       const SDNode *right = N0.getOperand(1).getNode();
5806:
5807:       if (isa<ConstantSDNode>(left) || isa<ConstantSDNode>(right))
5808:         opIsLive = true;
5809:
5810:       if (!opIsLive)
5811:         for (const SDNode *User : left->users()) {
5812:           int orderNo3 = User->getIROrder();
5813:           if (orderNo3 > orderNo) {
5814:             opIsLive = true;
5815:             break;
5816:           }
5817:         }
5818:
5819:       if (!opIsLive)
5820:         for (const SDNode *User : right->users()) {
5821:           int orderNo3 = User->getIROrder();
5822:           if (orderNo3 > orderNo) {
5823:             opIsLive = true;
5824:             break;
5825:           }
5826:         }
5827:
5828:       if (!opIsLive)
5829:         return SDValue();
5830:     }
5831:
5832:     return DCI.DAG.getNode(ISD::FMA, SDLoc(N), VT, N0.getOperand(0),
5833:                            N0.getOperand(1), N1);
5834:   }
5835:
5836:   return SDValue();
5837: }
5838:
5839: /// Fold unpacking movs into a load by increasing the number of return values.
5840: ///
5841: /// ex:
5842: /// L: v2f16,ch = load <p>
5843: /// a: f16 = extractelt L:0, 0
5844: /// b: f16 = extractelt L:0, 1
5845: /// use(a, b)
5846: ///
5847: /// ...is turned into...
5848: ///
5849: /// L: f16,f16,ch = LoadV2 <p>
5850: /// use(L:0, L:1)
5851: static SDValue
5852: combineUnpackingMovIntoLoad(SDNode *N, TargetLowering::DAGCombinerInfo &DCI) {
5853:   // Don't run this optimization before the legalizer
5854:   if (!DCI.isAfterLegalizeDAG())
5855:     return SDValue();
5856:
5857:   EVT ElementVT = N->getValueType(0);
5858:   // Avoid non-packed types and v4i8
5859:   if (!NVPTX::isPackedVectorTy(ElementVT) || ElementVT == MVT::v4i8)
5860:     return SDValue();
5861:
5862:   // Check whether all outputs are either used by an extractelt or are
5863:   // glue/chain nodes
5864:   if (!all_of(N->uses(), [&](SDUse &U) {
5865:         // Skip glue, chain nodes
5866:         if (U.getValueType() == MVT::Glue || U.getValueType() == MVT::Other)
5867:           return true;
5868:         if (U.getUser()->getOpcode() == ISD::EXTRACT_VECTOR_ELT) {
5869:           if (N->getOpcode() != ISD::LOAD)
5870:             return true;
5871:           // Since this is an ISD::LOAD, check all extractelts are used. If
5872:           // any are not used, we don't want to defeat another optimization that
5873:           // will narrow the load.
5874:           //
5875:           // For example:
5876:           //
5877:           // L: v2f16,ch = load <p>
5878:           // e0: f16 = extractelt L:0, 0
5879:           // e1: f16 = extractelt L:0, 1        <-- unused
5880:           // store e0
```
- EN: This range implements operational logic in helpers such as SDValue, getOperand, DL, getNode, translating backend policy into executable code.
- CN: 这一段实现了 SDValue、getOperand、DL、getNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 5881-6020
```cpp
5881:           //
5882:           // Can be optimized by DAGCombiner to:
5883:           //
5884:           // L: f16,ch = load <p>
5885:           // store L:0
5886:           return !U.getUser()->use_empty();
5887:         }
5888:
5889:         // Otherwise, this use prevents us from splitting a value.
5890:         return false;
5891:       }))
5892:     return SDValue();
5893:
5894:   auto *LD = cast<MemSDNode>(N);
5895:   SDLoc DL(LD);
5896:
5897:   // the new opcode after we double the number of operands
5898:   unsigned Opcode;
5899:   SmallVector<SDValue> Operands(LD->ops());
5900:   unsigned OldNumOutputs; // non-glue, non-chain outputs
5901:   switch (LD->getOpcode()) {
5902:   case ISD::LOAD:
5903:     OldNumOutputs = 1;
5904:     // Any packed type is legal, so the legalizer will not have lowered
5905:     // ISD::LOAD -> NVPTXISD::Load (unless it's under-aligned). We have to do it
5906:     // here.
5907:     Opcode = NVPTXISD::LoadV2;
5908:     // append a "full" used bytes mask operand right before the extension type
5909:     // operand, signifying that all bytes are used.
5910:     Operands.push_back(DCI.DAG.getConstant(UINT32_MAX, DL, MVT::i32));
5911:     Operands.push_back(DCI.DAG.getIntPtrConstant(
5912:         cast<LoadSDNode>(LD)->getExtensionType(), DL));
5913:     break;
5914:   case NVPTXISD::LoadV2:
5915:     OldNumOutputs = 2;
5916:     Opcode = NVPTXISD::LoadV4;
5917:     break;
5918:   case NVPTXISD::LoadV4:
5919:     // V8 is only supported for f32/i32. Don't forget, we're not changing the
5920:     // load size here. This is already a 256-bit load.
5921:     if (ElementVT != MVT::v2f32 && ElementVT != MVT::v2i32)
5922:       return SDValue();
5923:     OldNumOutputs = 4;
5924:     Opcode = NVPTXISD::LoadV8;
5925:     break;
5926:   case NVPTXISD::LoadV8:
5927:     // PTX doesn't support the next doubling of outputs
5928:     return SDValue();
5929:   }
5930:
5931:   // the non-glue, non-chain outputs in the new load
5932:   const unsigned NewNumOutputs = OldNumOutputs * 2;
5933:   SmallVector<EVT> NewVTs(NewNumOutputs, ElementVT.getVectorElementType());
5934:   // add remaining chain and glue values
5935:   NewVTs.append(LD->value_begin() + OldNumOutputs, LD->value_end());
5936:
5937:   // Create the new load
5938:   SDValue NewLoad = DCI.DAG.getMemIntrinsicNode(
5939:       Opcode, DL, DCI.DAG.getVTList(NewVTs), Operands, LD->getMemoryVT(),
5940:       LD->getMemOperand());
5941:
5942:   // Now we use a combination of BUILD_VECTORs and a MERGE_VALUES node to keep
5943:   // the outputs the same. These nodes will be optimized away in later
5944:   // DAGCombiner iterations.
5945:   SmallVector<SDValue> Results;
5946:   for (unsigned I : seq(OldNumOutputs))
5947:     Results.push_back(DCI.DAG.getBuildVector(
5948:         ElementVT, DL, {NewLoad.getValue(I * 2), NewLoad.getValue(I * 2 + 1)}));
5949:   // Add remaining chain and glue nodes
5950:   for (unsigned I : seq(NewLoad->getNumValues() - NewNumOutputs))
5951:     Results.push_back(NewLoad.getValue(NewNumOutputs + I));
5952:
5953:   return DCI.DAG.getMergeValues(Results, DL);
5954: }
5955:
5956: /// Fold packing movs into a store.
5957: ///
5958: /// ex:
5959: /// v1: v2f16 = BUILD_VECTOR a:f16, b:f16
5960: /// v2: v2f16 = BUILD_VECTOR c:f16, d:f16
5961: /// StoreV2 v1, v2
5962: ///
5963: /// ...is turned into...
5964: ///
5965: /// StoreV4 a, b, c, d
5966: static SDValue combinePackingMovIntoStore(SDNode *N,
5967:                                           TargetLowering::DAGCombinerInfo &DCI,
5968:                                           unsigned Front, unsigned Back) {
5969:   // We want to run this as late as possible since other optimizations may
5970:   // eliminate the BUILD_VECTORs.
5971:   if (!DCI.isAfterLegalizeDAG())
5972:     return SDValue();
5973:
5974:   // Get the type of the operands being stored.
5975:   EVT ElementVT = N->getOperand(Front).getValueType();
5976:
5977:   // Avoid non-packed types and v4i8
5978:   if (!NVPTX::isPackedVectorTy(ElementVT) || ElementVT == MVT::v4i8)
5979:     return SDValue();
5980:
5981:   auto *ST = cast<MemSDNode>(N);
5982:
5983:   // The new opcode after we double the number of operands.
5984:   unsigned Opcode;
5985:   switch (N->getOpcode()) {
5986:   case ISD::STORE:
5987:     // Any packed type is legal, so the legalizer will not have lowered
5988:     // ISD::STORE -> NVPTXISD::Store (unless it's under-aligned). We have to do
5989:     // it here.
5990:     Opcode = NVPTXISD::StoreV2;
5991:     break;
5992:   case NVPTXISD::StoreV2:
5993:     Opcode = NVPTXISD::StoreV4;
5994:     break;
5995:   case NVPTXISD::StoreV4:
5996:     // V8 is only supported for f32/i32. Don't forget, we're not changing the
5997:     // store size here. This is already a 256-bit store.
5998:     if (ElementVT != MVT::v2f32 && ElementVT != MVT::v2i32)
5999:       return SDValue();
6000:     Opcode = NVPTXISD::StoreV8;
6001:     break;
6002:   case NVPTXISD::StoreV8:
6003:     // PTX doesn't support the next doubling of operands
6004:     return SDValue();
6005:   default:
6006:     llvm_unreachable("Unhandled store opcode");
6007:   }
6008:
6009:   // Scan the operands and if they're all BUILD_VECTORs, we'll have gathered
6010:   // their elements.
6011:   SmallVector<SDValue, 4> Operands(N->ops().take_front(Front));
6012:   for (SDValue BV : N->ops().drop_front(Front).drop_back(Back)) {
6013:     if (BV.getOpcode() != ISD::BUILD_VECTOR)
6014:       return SDValue();
6015:
6016:     // If the operand has multiple uses, this optimization can increase register
6017:     // pressure.
6018:     if (!BV.hasOneUse())
6019:       return SDValue();
6020:
```
- EN: This range implements operational logic in helpers such as getUser, SDValue, DL, Operands, translating backend policy into executable code.
- CN: 这一段实现了 getUser、SDValue、DL、Operands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 6021-6160
```cpp
6021:     // DAGCombiner visits nodes bottom-up. Check the BUILD_VECTOR operands for
6022:     // any signs they may be folded by some other pattern or rule.
6023:     for (SDValue Op : BV->ops()) {
6024:       // Peek through bitcasts
6025:       if (Op.getOpcode() == ISD::BITCAST)
6026:         Op = Op.getOperand(0);
6027:
6028:       // This may be folded into a PRMT.
6029:       if (Op.getValueType() == MVT::i16 && Op.getOpcode() == ISD::TRUNCATE &&
6030:           Op->getOperand(0).getValueType() == MVT::i32)
6031:         return SDValue();
6032:
6033:       // This may be folded into cvt.bf16x2
6034:       if (Op.getOpcode() == ISD::FP_ROUND)
6035:         return SDValue();
6036:     }
6037:     Operands.append({BV.getOperand(0), BV.getOperand(1)});
6038:   }
6039:   Operands.append(N->op_end() - Back, N->op_end());
6040:
6041:   // Now we replace the store
6042:   return DCI.DAG.getMemIntrinsicNode(Opcode, SDLoc(N), N->getVTList(), Operands,
6043:                                      ST->getMemoryVT(), ST->getMemOperand());
6044: }
6045:
6046: static SDValue combineSTORE(SDNode *N, TargetLowering::DAGCombinerInfo &DCI,
6047:                             const NVPTXSubtarget &STI) {
6048:
6049:   if (DCI.isBeforeLegalize() && N->getOpcode() == ISD::STORE) {
6050:     // Here is our chance to custom lower a store with a non-simple type.
6051:     // Unfortunately, we can't do this in the legalizer because there is no
6052:     // way to setOperationAction for an non-simple type.
6053:     StoreSDNode *ST = cast<StoreSDNode>(N);
6054:     if (!ST->getValue().getValueType().isSimple())
6055:       return lowerSTOREVector(SDValue(ST, 0), DCI.DAG, STI);
6056:   }
6057:
6058:   return combinePackingMovIntoStore(N, DCI, 1, 2);
6059: }
6060:
6061: static SDValue combineLOAD(SDNode *N, TargetLowering::DAGCombinerInfo &DCI,
6062:                            const NVPTXSubtarget &STI) {
6063:   if (DCI.isBeforeLegalize() && N->getOpcode() == ISD::LOAD) {
6064:     // Here is our chance to custom lower a load with a non-simple type.
6065:     // Unfortunately, we can't do this in the legalizer because there is no
6066:     // way to setOperationAction for an non-simple type.
6067:     if (!N->getValueType(0).isSimple())
6068:       return lowerLoadVector(N, DCI.DAG, STI);
6069:   }
6070:
6071:   return combineUnpackingMovIntoLoad(N, DCI);
6072: }
6073:
6074: /// PerformADDCombine - Target-specific dag combine xforms for ISD::ADD.
6075: ///
6076: static SDValue PerformADDCombine(SDNode *N,
6077:                                  TargetLowering::DAGCombinerInfo &DCI,
6078:                                  CodeGenOptLevel OptLevel) {
6079:   if (OptLevel == CodeGenOptLevel::None)
6080:     return SDValue();
6081:
6082:   SDValue N0 = N->getOperand(0);
6083:   SDValue N1 = N->getOperand(1);
6084:
6085:   // Skip non-integer, non-scalar case
6086:   EVT VT = N0.getValueType();
6087:   if (VT.isVector() || VT != MVT::i32)
6088:     return SDValue();
6089:
6090:   // First try with the default operand order.
6091:   if (SDValue Result = PerformADDCombineWithOperands(N, N0, N1, DCI))
6092:     return Result;
6093:
6094:   // If that didn't work, try again with the operands commuted.
6095:   return PerformADDCombineWithOperands(N, N1, N0, DCI);
6096: }
6097:
6098: /// Check if a v2f32 BUILD_VECTOR provably packs values from non-adjacent
6099: /// register pairs (non-coalescable).
6100: static bool isNonCoalescableBuildVector(const SDValue &BV) {
6101:   if (BV.getOpcode() != ISD::BUILD_VECTOR || BV.getValueType() != MVT::v2f32)
6102:     return false;
6103:
6104:   SDValue Elt0 = BV.getOperand(0);
6105:   SDValue Elt1 = BV.getOperand(1);
6106:
6107:   bool IsExt0 = Elt0.getOpcode() == ISD::EXTRACT_VECTOR_ELT;
6108:   bool IsExt1 = Elt1.getOpcode() == ISD::EXTRACT_VECTOR_ELT;
6109:
6110:   // If neither element is an EXTRACT_VECTOR_ELT they are free-standing
6111:   // scalars and the register allocator can still place them side-by-side.
6112:   if (!IsExt0 && !IsExt1)
6113:     return false;
6114:
6115:   // If exactly one element is an EXTRACT_VECTOR_ELT, the other is a scalar
6116:   // that cannot generally occupy the adjacent register slot.
6117:   if (IsExt0 != IsExt1)
6118:     return true;
6119:
6120:   // At this point both sources are extracting from vectors. If they are from
6121:   // different vectors, then the BUILD_VECTOR is non-coalescable.
6122:   SDValue Src0 = Elt0.getOperand(0);
6123:   SDValue Src1 = Elt1.getOperand(0);
6124:   if (Src0 != Src1)
6125:     return true;
6126:
6127:   auto *Idx0 = dyn_cast<ConstantSDNode>(Elt0.getOperand(1));
6128:   auto *Idx1 = dyn_cast<ConstantSDNode>(Elt1.getOperand(1));
6129:   // If both indices are dynamic they will be lowered to
6130:   // loads and the vector will be spilled to local memory. The register
6131:   // allocator can easily place the results in adjacent registers.
6132:   if (!Idx0 && !Idx1)
6133:     return false;
6134:
6135:   // If one index is dynamic and the other is constant, the value from the
6136:   // constant load will result in an additional register to pair with the result
6137:   // from the dynamic load. We consider this non-coalescable.
6138:   if ((Idx0 && !Idx1) || (!Idx0 && Idx1))
6139:     return true;
6140:
6141:   // Both are constant, adjacent pairs are coalescable
6142:   return std::abs(Idx0->getSExtValue() - Idx1->getSExtValue()) != 1;
6143: }
6144:
6145: /// Return true if FMUL v2f32 node \p N may be scalarized to fold each lane's
6146: /// product into a scalar FMA.
6147: bool NVPTXTargetLowering::mayFoldFMULIntoFMA(SDNode *N, MachineFunction &MF,
6148:                                              CodeGenOptLevel OptLevel) const {
6149:   if (N->getOpcode() != ISD::FMUL || N->getValueType(0) != MVT::v2f32)
6150:     return false;
6151:   const bool GlobalFMA = allowFMA(MF, OptLevel);
6152:   if (!N->getFlags().hasAllowContract() && !GlobalFMA)
6153:     return false;
6154:
6155:   const SDNode *FirstFAdd = nullptr;
6156:   unsigned NumScalarFAdd = 0;
6157:
6158:   // Both lanes must feed unique FADDs
6159:   for (SDNode *EE : N->users()) {
6160:     if (NumScalarFAdd == 2)
```
- EN: This range implements operational logic in helpers such as getOperand, SDValue, append, getMemoryVT, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、SDValue、append、getMemoryVT 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 6161-6300
```cpp
6161:       return false;
6162:
6163:     if (EE->getOpcode() != ISD::EXTRACT_VECTOR_ELT || !EE->hasOneUse() ||
6164:         !isa<ConstantSDNode>(EE->getOperand(1)))
6165:       return false;
6166:
6167:     const SDNode *const FAdd = *EE->users().begin();
6168:     if (FAdd->getOpcode() != ISD::FADD ||
6169:         (!GlobalFMA && !FAdd->getFlags().hasAllowContract()))
6170:       return false;
6171:
6172:     if (!FirstFAdd)
6173:       FirstFAdd = FAdd;
6174:     else if (FAdd == FirstFAdd)
6175:       return false;
6176:
6177:     NumScalarFAdd++;
6178:   }
6179:
6180:   return NumScalarFAdd == 2;
6181: }
6182:
6183: /// Scalarize a v2f32 arithmetic node (FADD, FMUL, FSUB, FMA) when at least
6184: /// one operand is a BUILD_VECTOR that repacks values from non-adjacent register
6185: /// pairs.  Without this combine the BUILD_VECTOR forces allocation of a
6186: /// temporary 64-bit register, increasing register pressure.
6187: ///
6188: /// Example - before:
6189: ///   t0: v2f32,v2f32,ch = LoadV2 ...
6190: ///   t1: f32 = extract_vector_elt t0, 0
6191: ///   t2: f32 = extract_vector_elt t0:1, 0
6192: ///   t3: v2f32 = BUILD_VECTOR t1, t2       ;; non-coalescable repack
6193: ///   t4: v2f32 = fma t_a, t3, t_c
6194: ///
6195: /// After:
6196: ///   t0: v2f32,v2f32,ch = LoadV2 ...
6197: ///   t1: f32 = extract_vector_elt t0, 0
6198: ///   t2: f32 = extract_vector_elt t0:1, 0
6199: ///   a0: f32 = extract_vector_elt t_a, 0
6200: ///   a1: f32 = extract_vector_elt t_a, 1
6201: ///   c0: f32 = extract_vector_elt t_c, 0
6202: ///   c1: f32 = extract_vector_elt t_c, 1
6203: ///   r0: f32 = fma a0, t1, c0
6204: ///   r1: f32 = fma a1, t2, c1
6205: ///   t4: v2f32 = BUILD_VECTOR r0, r1
6206: ///
6207: /// Also scalarizes an FMUL when all output lanes feed into scalar FADDs
6208: /// to enable scalar FMA combining.
6209: SDValue NVPTXTargetLowering::performScalarizeV2F32Op(
6210:     SDNode *N, TargetLowering::DAGCombinerInfo &DCI,
6211:     CodeGenOptLevel OptLevel) const {
6212:   EVT VT = N->getValueType(0);
6213:   if (VT != MVT::v2f32)
6214:     return SDValue();
6215:
6216:   if (none_of(N->ops(), isNonCoalescableBuildVector) &&
6217:       !mayFoldFMULIntoFMA(N, DCI.DAG.getMachineFunction(), OptLevel))
6218:     return SDValue();
6219:
6220:   SelectionDAG &DAG = DCI.DAG;
6221:   SDLoc DL(N);
6222:   EVT EltVT = VT.getVectorElementType();
6223:   unsigned Opc = N->getOpcode();
6224:
6225:   // For each operand, get the scalar element at the given index: if the operand
6226:   // is a BUILD_VECTOR, grab the element directly; otherwise, emit an
6227:   // EXTRACT_VECTOR_ELT.
6228:   auto GetElement = [&](SDValue Op, unsigned Index) -> SDValue {
6229:     if (Op.getOpcode() == ISD::BUILD_VECTOR)
6230:       return Op.getOperand(Index);
6231:     return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT, Op,
6232:                        DAG.getVectorIdxConstant(Index, DL));
6233:   };
6234:
6235:   // Build scalar operand lists for element 0 and element 1.
6236:   SmallVector<SDValue, 3> Ops0, Ops1;
6237:   for (const SDValue &Op : N->ops()) {
6238:     Ops0.push_back(GetElement(Op, 0));
6239:     Ops1.push_back(GetElement(Op, 1));
6240:   }
6241:
6242:   SDValue Res0 = DAG.getNode(Opc, DL, EltVT, Ops0, N->getFlags());
6243:   SDValue Res1 = DAG.getNode(Opc, DL, EltVT, Ops1, N->getFlags());
6244:
6245:   return DAG.getNode(ISD::BUILD_VECTOR, DL, VT, Res0, Res1);
6246: }
6247:
6248: /// Target-specific dag combine xforms for ISD::FADD.
6249: SDValue
6250: NVPTXTargetLowering::performFADDCombine(SDNode *N,
6251:                                         TargetLowering::DAGCombinerInfo &DCI,
6252:                                         CodeGenOptLevel OptLevel) const {
6253:   if (SDValue Result = performScalarizeV2F32Op(N, DCI, OptLevel))
6254:     return Result;
6255:
6256:   SDValue N0 = N->getOperand(0);
6257:   SDValue N1 = N->getOperand(1);
6258:
6259:   EVT VT = N0.getValueType();
6260:   if (VT.isVector() || !(VT == MVT::f32 || VT == MVT::f64))
6261:     return SDValue();
6262:
6263:   // First try with the default operand order.
6264:   if (SDValue Result = performFADDCombineWithOperands(N, N0, N1, DCI, OptLevel))
6265:     return Result;
6266:
6267:   // If that didn't work, try again with the operands commuted.
6268:   return performFADDCombineWithOperands(N, N1, N0, DCI, OptLevel);
6269: }
6270:
6271: /// Get 3-input version of a 2-input min/max opcode
6272: static unsigned getMinMax3Opcode(unsigned MinMax2Opcode) {
6273:   switch (MinMax2Opcode) {
6274:   case ISD::FMAXNUM:
6275:   case ISD::FMAXIMUMNUM:
6276:     return NVPTXISD::FMAXNUM3;
6277:   case ISD::FMINNUM:
6278:   case ISD::FMINIMUMNUM:
6279:     return NVPTXISD::FMINNUM3;
6280:   case ISD::FMAXIMUM:
6281:     return NVPTXISD::FMAXIMUM3;
6282:   case ISD::FMINIMUM:
6283:     return NVPTXISD::FMINIMUM3;
6284:   default:
6285:     llvm_unreachable("Invalid 2-input min/max opcode");
6286:   }
6287: }
6288:
6289: /// PerformFMinMaxCombine - Combine (fmaxnum (fmaxnum a, b), c) into
6290: /// (fmaxnum3 a, b, c). Also covers other llvm min/max intrinsics.
6291: static SDValue PerformFMinMaxCombine(SDNode *N,
6292:                                      TargetLowering::DAGCombinerInfo &DCI,
6293:                                      unsigned PTXVersion, unsigned SmVersion) {
6294:
6295:   // 3-input min/max requires PTX 8.8+ and SM_100+, and only supports f32s
6296:   EVT VT = N->getValueType(0);
6297:   if (VT != MVT::f32 || PTXVersion < 88 || SmVersion < 100)
6298:     return SDValue();
6299:
6300:   SDValue Op0 = N->getOperand(0);
```
- EN: This range implements operational logic in helpers such as getOperand, users, getFlags, getValueType, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、users、getFlags、getValueType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 6301-6440
```cpp
6301:   SDValue Op1 = N->getOperand(1);
6302:   unsigned MinMaxOp2 = N->getOpcode();
6303:   unsigned MinMaxOp3 = getMinMax3Opcode(MinMaxOp2);
6304:
6305:   if (Op0.getOpcode() == MinMaxOp2 && Op0.hasOneUse()) {
6306:     // (maxnum (maxnum a, b), c) -> (maxnum3 a, b, c)
6307:     SDValue A = Op0.getOperand(0);
6308:     SDValue B = Op0.getOperand(1);
6309:     SDValue C = Op1;
6310:     return DCI.DAG.getNode(MinMaxOp3, SDLoc(N), VT, A, B, C, N->getFlags());
6311:   } else if (Op1.getOpcode() == MinMaxOp2 && Op1.hasOneUse()) {
6312:     // (maxnum a, (maxnum b, c)) -> (maxnum3 a, b, c)
6313:     SDValue A = Op0;
6314:     SDValue B = Op1.getOperand(0);
6315:     SDValue C = Op1.getOperand(1);
6316:     return DCI.DAG.getNode(MinMaxOp3, SDLoc(N), VT, A, B, C, N->getFlags());
6317:   }
6318:   return SDValue();
6319: }
6320:
6321: static SDValue PerformREMCombine(SDNode *N,
6322:                                  TargetLowering::DAGCombinerInfo &DCI,
6323:                                  CodeGenOptLevel OptLevel) {
6324:   assert(N->getOpcode() == ISD::SREM || N->getOpcode() == ISD::UREM);
6325:
6326:   // Don't do anything at less than -O2.
6327:   if (OptLevel < CodeGenOptLevel::Default)
6328:     return SDValue();
6329:
6330:   SelectionDAG &DAG = DCI.DAG;
6331:   SDLoc DL(N);
6332:   EVT VT = N->getValueType(0);
6333:   bool IsSigned = N->getOpcode() == ISD::SREM;
6334:   unsigned DivOpc = IsSigned ? ISD::SDIV : ISD::UDIV;
6335:
6336:   const SDValue &Num = N->getOperand(0);
6337:   const SDValue &Den = N->getOperand(1);
6338:
6339:   for (const SDNode *U : Num->users()) {
6340:     if (U->getOpcode() == DivOpc && U->getOperand(0) == Num &&
6341:         U->getOperand(1) == Den) {
6342:       // Num % Den -> Num - (Num / Den) * Den
6343:       return DAG.getNode(ISD::SUB, DL, VT, Num,
6344:                          DAG.getNode(ISD::MUL, DL, VT,
6345:                                      DAG.getNode(DivOpc, DL, VT, Num, Den),
6346:                                      Den));
6347:     }
6348:   }
6349:   return SDValue();
6350: }
6351:
6352: // (sign_extend|zero_extend (mul|shl) x, y) -> (mul.wide x, y)
6353: static SDValue combineMulWide(SDNode *N, TargetLowering::DAGCombinerInfo &DCI,
6354:                               CodeGenOptLevel OptLevel) {
6355:   if (OptLevel == CodeGenOptLevel::None)
6356:     return SDValue();
6357:
6358:   SDValue Op = N->getOperand(0);
6359:   if (!Op.hasOneUse())
6360:     return SDValue();
6361:   EVT ToVT = N->getValueType(0);
6362:   EVT FromVT = Op.getValueType();
6363:   if (!((ToVT == MVT::i32 && FromVT == MVT::i16) ||
6364:         (ToVT == MVT::i64 && FromVT == MVT::i32)))
6365:     return SDValue();
6366:   if (!(Op.getOpcode() == ISD::MUL ||
6367:         (Op.getOpcode() == ISD::SHL && isa<ConstantSDNode>(Op.getOperand(1)))))
6368:     return SDValue();
6369:
6370:   SDLoc DL(N);
6371:   unsigned ExtOpcode = N->getOpcode();
6372:   unsigned Opcode = 0;
6373:   if (ExtOpcode == ISD::SIGN_EXTEND && Op->getFlags().hasNoSignedWrap())
6374:     Opcode = NVPTXISD::MUL_WIDE_SIGNED;
6375:   else if (ExtOpcode == ISD::ZERO_EXTEND && Op->getFlags().hasNoUnsignedWrap())
6376:     Opcode = NVPTXISD::MUL_WIDE_UNSIGNED;
6377:   else
6378:     return SDValue();
6379:   SDValue RHS = Op.getOperand(1);
6380:   if (Op.getOpcode() == ISD::SHL) {
6381:     const auto ShiftAmt = Op.getConstantOperandVal(1);
6382:     const auto MulVal = APInt(FromVT.getSizeInBits(), 1) << ShiftAmt;
6383:     RHS = DCI.DAG.getConstant(MulVal, DL, FromVT);
6384:   }
6385:   return DCI.DAG.getNode(Opcode, DL, ToVT, Op.getOperand(0), RHS);
6386: }
6387:
6388: enum OperandSignedness {
6389:   Signed = 0,
6390:   Unsigned,
6391:   Unknown
6392: };
6393:
6394: /// IsMulWideOperandDemotable - Checks if the provided DAG node is an operand
6395: /// that can be demoted to \p OptSize bits without loss of information. The
6396: /// signedness of the operand, if determinable, is placed in \p S.
6397: static bool IsMulWideOperandDemotable(SDValue Op,
6398:                                       unsigned OptSize,
6399:                                       OperandSignedness &S) {
6400:   S = Unknown;
6401:
6402:   if (Op.getOpcode() == ISD::SIGN_EXTEND ||
6403:       Op.getOpcode() == ISD::SIGN_EXTEND_INREG) {
6404:     EVT OrigVT = Op.getOperand(0).getValueType();
6405:     if (OrigVT.getFixedSizeInBits() <= OptSize) {
6406:       S = Signed;
6407:       return true;
6408:     }
6409:   } else if (Op.getOpcode() == ISD::ZERO_EXTEND) {
6410:     EVT OrigVT = Op.getOperand(0).getValueType();
6411:     if (OrigVT.getFixedSizeInBits() <= OptSize) {
6412:       S = Unsigned;
6413:       return true;
6414:     }
6415:   }
6416:
6417:   return false;
6418: }
6419:
6420: /// AreMulWideOperandsDemotable - Checks if the given LHS and RHS operands can
6421: /// be demoted to \p OptSize bits without loss of information. If the operands
6422: /// contain a constant, it should appear as the RHS operand. The signedness of
6423: /// the operands is placed in \p IsSigned.
6424: static bool AreMulWideOperandsDemotable(SDValue LHS, SDValue RHS,
6425:                                         unsigned OptSize,
6426:                                         bool &IsSigned) {
6427:   OperandSignedness LHSSign;
6428:
6429:   // The LHS operand must be a demotable op
6430:   if (!IsMulWideOperandDemotable(LHS, OptSize, LHSSign))
6431:     return false;
6432:
6433:   // We should have been able to determine the signedness from the LHS
6434:   if (LHSSign == Unknown)
6435:     return false;
6436:
6437:   IsSigned = (LHSSign == Signed);
6438:
6439:   // The RHS can be a demotable op or a constant
6440:   if (ConstantSDNode *CI = dyn_cast<ConstantSDNode>(RHS)) {
```
- EN: This range defines or declares important types such as getOperand, getOpcode, getMinMax3Opcode, getNode, shaping the data model used by NVPTXISelLowering.cpp.
- CN: 这一段定义或声明了 getOperand、getOpcode、getMinMax3Opcode、getNode 等关键类型，构成 NVPTXISelLowering.cpp 使用的数据模型。

### Lines 6441-6580
```cpp
6441:     const APInt &Val = CI->getAPIntValue();
6442:     if (LHSSign == Unsigned) {
6443:       return Val.isIntN(OptSize);
6444:     } else {
6445:       return Val.isSignedIntN(OptSize);
6446:     }
6447:   } else {
6448:     OperandSignedness RHSSign;
6449:     if (!IsMulWideOperandDemotable(RHS, OptSize, RHSSign))
6450:       return false;
6451:
6452:     return LHSSign == RHSSign;
6453:   }
6454: }
6455:
6456: /// TryMULWIDECombine - Attempt to replace a multiply of M bits with a multiply
6457: /// of M/2 bits that produces an M-bit result (i.e. mul.wide). This transform
6458: /// works on both multiply DAG nodes and SHL DAG nodes with a constant shift
6459: /// amount.
6460: static SDValue TryMULWIDECombine(SDNode *N,
6461:                                  TargetLowering::DAGCombinerInfo &DCI) {
6462:   EVT MulType = N->getValueType(0);
6463:   if (MulType != MVT::i32 && MulType != MVT::i64) {
6464:     return SDValue();
6465:   }
6466:
6467:   SDLoc DL(N);
6468:   unsigned OptSize = MulType.getSizeInBits() >> 1;
6469:   SDValue LHS = N->getOperand(0);
6470:   SDValue RHS = N->getOperand(1);
6471:
6472:   // Canonicalize the multiply so the constant (if any) is on the right
6473:   if (N->getOpcode() == ISD::MUL) {
6474:     if (isa<ConstantSDNode>(LHS)) {
6475:       std::swap(LHS, RHS);
6476:     }
6477:   }
6478:
6479:   // If we have a SHL, determine the actual multiply amount
6480:   if (N->getOpcode() == ISD::SHL) {
6481:     ConstantSDNode *ShlRHS = dyn_cast<ConstantSDNode>(RHS);
6482:     if (!ShlRHS) {
6483:       return SDValue();
6484:     }
6485:
6486:     APInt ShiftAmt = ShlRHS->getAPIntValue();
6487:     unsigned BitWidth = MulType.getSizeInBits();
6488:     if (ShiftAmt.sge(0) && ShiftAmt.slt(BitWidth)) {
6489:       APInt MulVal = APInt(BitWidth, 1) << ShiftAmt;
6490:       RHS = DCI.DAG.getConstant(MulVal, DL, MulType);
6491:     } else {
6492:       return SDValue();
6493:     }
6494:   }
6495:
6496:   bool Signed;
6497:   // Verify that our operands are demotable
6498:   if (!AreMulWideOperandsDemotable(LHS, RHS, OptSize, Signed)) {
6499:     return SDValue();
6500:   }
6501:
6502:   EVT DemotedVT;
6503:   if (MulType == MVT::i32) {
6504:     DemotedVT = MVT::i16;
6505:   } else {
6506:     DemotedVT = MVT::i32;
6507:   }
6508:
6509:   // Truncate the operands to the correct size. Note that these are just for
6510:   // type consistency and will (likely) be eliminated in later phases.
6511:   SDValue TruncLHS =
6512:     DCI.DAG.getNode(ISD::TRUNCATE, DL, DemotedVT, LHS);
6513:   SDValue TruncRHS =
6514:     DCI.DAG.getNode(ISD::TRUNCATE, DL, DemotedVT, RHS);
6515:
6516:   unsigned Opc;
6517:   if (Signed) {
6518:     Opc = NVPTXISD::MUL_WIDE_SIGNED;
6519:   } else {
6520:     Opc = NVPTXISD::MUL_WIDE_UNSIGNED;
6521:   }
6522:
6523:   return DCI.DAG.getNode(Opc, DL, MulType, TruncLHS, TruncRHS);
6524: }
6525:
6526: static bool isConstOne(const SDValue &Operand) {
6527:   const auto *Const = dyn_cast<ConstantSDNode>(Operand);
6528:   return Const && Const->getZExtValue() == 1;
6529: }
6530:
6531: static SDValue matchMADConstOnePattern(SDValue Add) {
6532:   if (Add->getOpcode() != ISD::ADD)
6533:     return SDValue();
6534:
6535:   if (isConstOne(Add->getOperand(0)))
6536:     return Add->getOperand(1);
6537:
6538:   if (isConstOne(Add->getOperand(1)))
6539:     return Add->getOperand(0);
6540:
6541:   return SDValue();
6542: }
6543:
6544: static SDValue combineMADConstOne(SDValue X, SDValue Add, EVT VT, SDLoc DL,
6545:                                   TargetLowering::DAGCombinerInfo &DCI) {
6546:
6547:   if (SDValue Y = matchMADConstOnePattern(Add)) {
6548:     SDValue Mul = DCI.DAG.getNode(ISD::MUL, DL, VT, X, Y);
6549:     return DCI.DAG.getNode(ISD::ADD, DL, VT, Mul, X);
6550:   }
6551:
6552:   return SDValue();
6553: }
6554:
6555: static SDValue combineMulSelectConstOne(SDValue X, SDValue Select, EVT VT,
6556:                                         SDLoc DL,
6557:                                         TargetLowering::DAGCombinerInfo &DCI) {
6558:   if (Select->getOpcode() != ISD::SELECT)
6559:     return SDValue();
6560:
6561:   SDValue Cond = Select->getOperand(0);
6562:
6563:   unsigned ConstOpNo;
6564:   if (isConstOne(Select->getOperand(1)))
6565:     ConstOpNo = 1;
6566:   else if (isConstOne(Select->getOperand(2)))
6567:     ConstOpNo = 2;
6568:   else
6569:     return SDValue();
6570:
6571:   SDValue Y = Select->getOperand((ConstOpNo == 1) ? 2 : 1);
6572:
6573:   // Do not combine if the resulting sequence is not obviously profitable.
6574:   if (!matchMADConstOnePattern(Y))
6575:     return SDValue();
6576:
6577:   SDValue NewMul = DCI.DAG.getNode(ISD::MUL, DL, VT, X, Y);
6578:
6579:   return DCI.DAG.getNode(ISD::SELECT, DL, VT, Cond,
6580:                          (ConstOpNo == 1) ? X : NewMul,
```
- EN: This range implements operational logic in helpers such as getAPIntValue, isIntN, isSignedIntN, getValueType, translating backend policy into executable code.
- CN: 这一段实现了 getAPIntValue、isIntN、isSignedIntN、getValueType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 6581-6720
```cpp
6581:                          (ConstOpNo == 1) ? NewMul : X);
6582: }
6583:
6584: static SDValue
6585: PerformMULCombineWithOperands(SDNode *N, SDValue N0, SDValue N1,
6586:                               TargetLowering::DAGCombinerInfo &DCI) {
6587:
6588:   EVT VT = N0.getValueType();
6589:   if (VT.isVector())
6590:     return SDValue();
6591:
6592:   if (VT != MVT::i16 && VT != MVT::i32 && VT != MVT::i64)
6593:     return SDValue();
6594:
6595:   SDLoc DL(N);
6596:
6597:   // (mul x, (add y, 1)) -> (add (mul x, y), x)
6598:   if (SDValue Res = combineMADConstOne(N0, N1, VT, DL, DCI))
6599:     return Res;
6600:   if (SDValue Res = combineMADConstOne(N1, N0, VT, DL, DCI))
6601:     return Res;
6602:
6603:   // (mul x, (select y, 1)) -> (select (mul x, y), x)
6604:   if (SDValue Res = combineMulSelectConstOne(N0, N1, VT, DL, DCI))
6605:     return Res;
6606:   if (SDValue Res = combineMulSelectConstOne(N1, N0, VT, DL, DCI))
6607:     return Res;
6608:
6609:   return SDValue();
6610: }
6611:
6612: /// PerformMULCombine - Runs PTX-specific DAG combine patterns on MUL nodes.
6613: static SDValue PerformMULCombine(SDNode *N,
6614:                                  TargetLowering::DAGCombinerInfo &DCI,
6615:                                  CodeGenOptLevel OptLevel) {
6616:   if (OptLevel == CodeGenOptLevel::None)
6617:     return SDValue();
6618:
6619:   if (SDValue Ret = TryMULWIDECombine(N, DCI))
6620:     return Ret;
6621:
6622:   SDValue N0 = N->getOperand(0);
6623:   SDValue N1 = N->getOperand(1);
6624:   return PerformMULCombineWithOperands(N, N0, N1, DCI);
6625: }
6626:
6627: /// PerformSHLCombine - Runs PTX-specific DAG combine patterns on SHL nodes.
6628: static SDValue PerformSHLCombine(SDNode *N,
6629:                                  TargetLowering::DAGCombinerInfo &DCI,
6630:                                  CodeGenOptLevel OptLevel) {
6631:   if (OptLevel > CodeGenOptLevel::None) {
6632:     // Try mul.wide combining at OptLevel > 0
6633:     if (SDValue Ret = TryMULWIDECombine(N, DCI))
6634:       return Ret;
6635:   }
6636:
6637:   return SDValue();
6638: }
6639:
6640: static SDValue PerformSETCCCombine(SDNode *N,
6641:                                    TargetLowering::DAGCombinerInfo &DCI,
6642:                                    unsigned int SmVersion) {
6643:   EVT CCType = N->getValueType(0);
6644:   SDValue A = N->getOperand(0);
6645:   SDValue B = N->getOperand(1);
6646:
6647:   EVT AType = A.getValueType();
6648:   if (!(CCType == MVT::v2i1 && (AType == MVT::v2f16 || AType == MVT::v2bf16)))
6649:     return SDValue();
6650:
6651:   if (A.getValueType() == MVT::v2bf16 && SmVersion < 90)
6652:     return SDValue();
6653:
6654:   SDLoc DL(N);
6655:   // setp.f16x2 returns two scalar predicates, which we need to
6656:   // convert back to v2i1. The returned result will be scalarized by
6657:   // the legalizer, but the comparison will remain a single vector
6658:   // instruction.
6659:   SDValue CCNode = DCI.DAG.getNode(
6660:       A.getValueType() == MVT::v2f16 ? NVPTXISD::SETP_F16X2
6661:                                      : NVPTXISD::SETP_BF16X2,
6662:       DL, DCI.DAG.getVTList(MVT::i1, MVT::i1), {A, B, N->getOperand(2)});
6663:   return DCI.DAG.getNode(ISD::BUILD_VECTOR, DL, CCType, CCNode.getValue(0),
6664:                          CCNode.getValue(1));
6665: }
6666:
6667: static SDValue PerformEXTRACTCombine(SDNode *N,
6668:                                      TargetLowering::DAGCombinerInfo &DCI) {
6669:   SDValue Vector = peekThroughFreeze(N->getOperand(0));
6670:   SDLoc DL(N);
6671:   EVT VectorVT = Vector.getValueType();
6672:   if (Vector->getOpcode() == ISD::LOAD && VectorVT.isSimple() &&
6673:       IsPTXVectorType(VectorVT.getSimpleVT()))
6674:     return SDValue(); // Native vector loads already combine nicely w/
6675:                       // extract_vector_elt.
6676:   // Don't mess with singletons or packed types (v2*32, v2*16, v4i8 and v8i8),
6677:   // we already handle them OK.
6678:   if (VectorVT.getVectorNumElements() == 1 ||
6679:       NVPTX::isPackedVectorTy(VectorVT) || VectorVT == MVT::v8i8)
6680:     return SDValue();
6681:
6682:   // Don't mess with undef values as sra may be simplified to 0, not undef.
6683:   if (Vector->isUndef() || ISD::allOperandsUndef(Vector.getNode()))
6684:     return SDValue();
6685:
6686:   uint64_t VectorBits = VectorVT.getSizeInBits();
6687:   // We only handle the types we can extract in-register.
6688:   if (!(VectorBits == 16 || VectorBits == 32 || VectorBits == 64))
6689:     return SDValue();
6690:
6691:   ConstantSDNode *Index = dyn_cast<ConstantSDNode>(N->getOperand(1));
6692:   // Index == 0 is handled by generic DAG combiner.
6693:   if (!Index || Index->getZExtValue() == 0)
6694:     return SDValue();
6695:
6696:   MVT IVT = MVT::getIntegerVT(VectorBits);
6697:   EVT EltVT = VectorVT.getVectorElementType();
6698:   EVT EltIVT = EltVT.changeTypeToInteger();
6699:   uint64_t EltBits = EltVT.getScalarSizeInBits();
6700:
6701:   SDValue Result = DCI.DAG.getNode(
6702:       ISD::TRUNCATE, DL, EltIVT,
6703:       DCI.DAG.getNode(
6704:           ISD::SRA, DL, IVT, DCI.DAG.getNode(ISD::BITCAST, DL, IVT, Vector),
6705:           DCI.DAG.getConstant(Index->getZExtValue() * EltBits, DL, IVT)));
6706:
6707:   // If element has non-integer type, bitcast it back to the expected type.
6708:   if (EltVT != EltIVT)
6709:     Result = DCI.DAG.getNode(ISD::BITCAST, DL, EltVT, Result);
6710:   // Past legalizer, we may need to extent i8 -> i16 to match the register type.
6711:   if (EltVT != N->getValueType(0))
6712:     Result = DCI.DAG.getNode(ISD::ANY_EXTEND, DL, N->getValueType(0), Result);
6713:
6714:   return Result;
6715: }
6716:
6717: /// Transform patterns like:
6718: ///   (select (ugt shift_amt, BitWidth-1), 0, (srl/shl x, shift_amt))
6719: ///   (select (ult shift_amt, BitWidth), (srl/shl x, shift_amt), 0)
6720: /// Into:
```
- EN: This range implements operational logic in helpers such as getValueType, SDValue, DL, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getValueType、SDValue、DL、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 6721-6860
```cpp
6721: ///   (NVPTXISD::SRL_CLAMP x, shift_amt) or (NVPTXISD::SHL_CLAMP x, shift_amt)
6722: ///
6723: /// These patterns arise from C/C++ code like `shift >= 32 ? 0 : x >> shift`
6724: /// which guards against undefined behavior. PTX shr/shl instructions clamp
6725: /// shift amounts >= BitWidth to produce 0 for logical shifts, making the
6726: /// guard redundant.
6727: ///
6728: /// Note: We only handle SRL and SHL, not SRA, because arithmetic right
6729: /// shifts could produce 0 or -1 when shift >= BitWidth.
6730: /// Note: We don't handle uge or ule. These don't appear because of
6731: /// canonicalization.
6732: static SDValue PerformSELECTShiftCombine(SDNode *N,
6733:                                          TargetLowering::DAGCombinerInfo &DCI) {
6734:   if (!DCI.isAfterLegalizeDAG())
6735:     return SDValue();
6736:
6737:   using namespace SDPatternMatch;
6738:   unsigned BitWidth = N->getValueType(0).getSizeInBits();
6739:   SDValue ShiftAmt, ShiftOp;
6740:
6741:   // Match logical shifts where the shift amount in the guard matches the shift
6742:   // amount in the operation.
6743:   auto LogicalShift =
6744:       m_AllOf(m_Value(ShiftOp),
6745:               m_AnyOf(m_Srl(m_Value(), m_TruncOrSelf(m_Deferred(ShiftAmt))),
6746:                       m_Shl(m_Value(), m_TruncOrSelf(m_Deferred(ShiftAmt)))));
6747:
6748:   // shift_amt > BitWidth-1 ? 0 : shift_op
6749:   bool MatchedUGT =
6750:       sd_match(N, m_Select(m_SetCC(m_Value(ShiftAmt),
6751:                                    m_SpecificInt(APInt(BitWidth, BitWidth - 1)),
6752:                                    m_SpecificCondCode(ISD::SETUGT)),
6753:                            m_Zero(), LogicalShift));
6754:   // shift_amt < BitWidth ? shift_op : 0
6755:   bool MatchedULT =
6756:       !MatchedUGT &&
6757:       sd_match(N, m_Select(m_SetCC(m_Value(ShiftAmt),
6758:                                    m_SpecificInt(APInt(BitWidth, BitWidth)),
6759:                                    m_SpecificCondCode(ISD::SETULT)),
6760:                            LogicalShift, m_Zero()));
6761:
6762:   if (!MatchedUGT && !MatchedULT)
6763:     return SDValue();
6764:
6765:   // Return a clamp shift operation, which has the same semantics as PTX shift.
6766:   unsigned ClampOpc = ShiftOp.getOpcode() == ISD::SRL ? NVPTXISD::SRL_CLAMP
6767:                                                       : NVPTXISD::SHL_CLAMP;
6768:   return DCI.DAG.getNode(ClampOpc, SDLoc(N), ShiftOp.getValueType(),
6769:                          ShiftOp.getOperand(0), ShiftOp.getOperand(1));
6770: }
6771:
6772: static SDValue PerformVSELECTCombine(SDNode *N,
6773:                                      TargetLowering::DAGCombinerInfo &DCI) {
6774:   SDValue VA = N->getOperand(1);
6775:   EVT VectorVT = VA.getValueType();
6776:   if (VectorVT != MVT::v4i8)
6777:     return SDValue();
6778:
6779:   // We need to split vselect into individual per-element operations Because we
6780:   // use BFE/BFI instruction for byte extraction/insertion, we do end up with
6781:   // 32-bit values, so we may as well do comparison as i32 to avoid conversions
6782:   // to/from i16 normally used for i8 values.
6783:   SmallVector<SDValue, 4> E;
6784:   SDLoc DL(N);
6785:   SDValue VCond = N->getOperand(0);
6786:   SDValue VB = N->getOperand(2);
6787:   for (int I = 0; I < 4; ++I) {
6788:     SDValue C = DCI.DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i1, VCond,
6789:                                 DCI.DAG.getConstant(I, DL, MVT::i32));
6790:     SDValue EA = DCI.DAG.getAnyExtOrTrunc(
6791:         DCI.DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i8, VA,
6792:                         DCI.DAG.getConstant(I, DL, MVT::i32)),
6793:         DL, MVT::i32);
6794:     SDValue EB = DCI.DAG.getAnyExtOrTrunc(
6795:         DCI.DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i8, VB,
6796:                         DCI.DAG.getConstant(I, DL, MVT::i32)),
6797:         DL, MVT::i32);
6798:     E.push_back(DCI.DAG.getAnyExtOrTrunc(
6799:         DCI.DAG.getNode(ISD::SELECT, DL, MVT::i32, C, EA, EB), DL, MVT::i8));
6800:   }
6801:   return DCI.DAG.getNode(ISD::BUILD_VECTOR, DL, MVT::v4i8, E);
6802: }
6803:
6804: static SDValue
6805: PerformBUILD_VECTORCombine(SDNode *N, TargetLowering::DAGCombinerInfo &DCI) {
6806:   auto VT = N->getValueType(0);
6807:   if (!DCI.isAfterLegalizeDAG() ||
6808:       // only process v2*16 types
6809:       !(NVPTX::isPackedVectorTy(VT) && VT.is32BitVector() &&
6810:         VT.getVectorNumElements() == 2))
6811:     return SDValue();
6812:
6813:   auto Op0 = N->getOperand(0);
6814:   auto Op1 = N->getOperand(1);
6815:
6816:   // Start out by assuming we want to take the lower 2 bytes of each i32
6817:   // operand.
6818:   uint64_t Op0Bytes = 0x10;
6819:   uint64_t Op1Bytes = 0x54;
6820:
6821:   std::pair<SDValue *, uint64_t *> OpData[2] = {{&Op0, &Op0Bytes},
6822:                                                 {&Op1, &Op1Bytes}};
6823:
6824:   // Check that each operand is an i16, truncated from an i32 operand. We'll
6825:   // select individual bytes from those original operands. Optionally, fold in a
6826:   // shift right of that original operand.
6827:   for (auto &[Op, OpBytes] : OpData) {
6828:     // Eat up any bitcast
6829:     if (Op->getOpcode() == ISD::BITCAST)
6830:       *Op = Op->getOperand(0);
6831:
6832:     if (!(Op->getValueType() == MVT::i16 && Op->getOpcode() == ISD::TRUNCATE &&
6833:           Op->getOperand(0).getValueType() == MVT::i32))
6834:       return SDValue();
6835:
6836:     // If the truncate has multiple uses, this optimization can increase
6837:     // register pressure
6838:     if (!Op->hasOneUse())
6839:       return SDValue();
6840:
6841:     *Op = Op->getOperand(0);
6842:
6843:     // Optionally, fold in a shift-right of the original operand and let permute
6844:     // pick the two higher bytes of the original value directly.
6845:     if (Op->getOpcode() == ISD::SRL && isa<ConstantSDNode>(Op->getOperand(1))) {
6846:       if (cast<ConstantSDNode>(Op->getOperand(1))->getZExtValue() == 16) {
6847:         // Shift the PRMT byte selector to pick upper bytes from each respective
6848:         // value, instead of the lower ones: 0x10 -> 0x32, 0x54 -> 0x76
6849:         assert((*OpBytes == 0x10 || *OpBytes == 0x54) &&
6850:                "PRMT selector values out of range");
6851:         *OpBytes += 0x22;
6852:         *Op = Op->getOperand(0);
6853:       }
6854:     }
6855:   }
6856:
6857:   SDLoc DL(N);
6858:   auto &DAG = DCI.DAG;
6859:
6860:   auto PRMT =
```
- EN: This range implements operational logic in helpers such as SDValue, getValueType, m_Shl, m_Zero, translating backend policy into executable code.
- CN: 这一段实现了 SDValue、getValueType、m_Shl、m_Zero 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 6861-7000
```cpp
6861:       getPRMT(DAG.getBitcast(MVT::i32, Op0), DAG.getBitcast(MVT::i32, Op1),
6862:               (Op1Bytes << 8) | Op0Bytes, DL, DAG);
6863:   return DAG.getBitcast(VT, PRMT);
6864: }
6865:
6866: static SDValue combineADDRSPACECAST(SDNode *N,
6867:                                     TargetLowering::DAGCombinerInfo &DCI) {
6868:   auto *ASCN1 = cast<AddrSpaceCastSDNode>(N);
6869:
6870:   if (auto *ASCN2 = dyn_cast<AddrSpaceCastSDNode>(ASCN1->getOperand(0))) {
6871:     assert(ASCN2->getDestAddressSpace() == ASCN1->getSrcAddressSpace());
6872:
6873:     // Fold asc[B -> A](asc[A -> B](x)) -> x
6874:     if (ASCN1->getDestAddressSpace() == ASCN2->getSrcAddressSpace())
6875:       return ASCN2->getOperand(0);
6876:   }
6877:
6878:   return SDValue();
6879: }
6880:
6881: // Given a constant selector value and a prmt mode, return the selector value
6882: // normalized to the generic prmt mode. See the PTX ISA documentation for more
6883: // details:
6884: // https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#data-movement-and-conversion-instructions-prmt
6885: static APInt getPRMTSelector(const APInt &Selector, unsigned Mode) {
6886:   assert(Selector.getBitWidth() == 32 && "PRMT must have i32 operands");
6887:
6888:   if (Mode == NVPTX::PTXPrmtMode::NONE)
6889:     return Selector;
6890:
6891:   const unsigned V = Selector.trunc(2).getZExtValue();
6892:
6893:   const auto GetSelector = [](unsigned S0, unsigned S1, unsigned S2,
6894:                               unsigned S3) {
6895:     return APInt(32, S0 | (S1 << 4) | (S2 << 8) | (S3 << 12));
6896:   };
6897:
6898:   switch (Mode) {
6899:   case NVPTX::PTXPrmtMode::F4E:
6900:     return GetSelector(V, V + 1, V + 2, V + 3);
6901:   case NVPTX::PTXPrmtMode::B4E:
6902:     return GetSelector(V, (V - 1) & 7, (V - 2) & 7, (V - 3) & 7);
6903:   case NVPTX::PTXPrmtMode::RC8:
6904:     return GetSelector(V, V, V, V);
6905:   case NVPTX::PTXPrmtMode::ECL:
6906:     return GetSelector(V, std::max(V, 1U), std::max(V, 2U), 3U);
6907:   case NVPTX::PTXPrmtMode::ECR:
6908:     return GetSelector(0, std::min(V, 1U), std::min(V, 2U), V);
6909:   case NVPTX::PTXPrmtMode::RC16: {
6910:     unsigned V1 = (V & 1) << 1;
6911:     return GetSelector(V1, V1 + 1, V1, V1 + 1);
6912:   }
6913:   default:
6914:     llvm_unreachable("Invalid PRMT mode");
6915:   }
6916: }
6917:
6918: static APInt computePRMT(APInt A, APInt B, APInt Selector, unsigned Mode) {
6919:   assert(A.getBitWidth() == 32 && B.getBitWidth() == 32 &&
6920:          Selector.getBitWidth() == 32 && "PRMT must have i32 operands");
6921:   // {b, a} = {{b7, b6, b5, b4}, {b3, b2, b1, b0}}
6922:   APInt BitField = B.concat(A);
6923:   APInt SelectorVal = getPRMTSelector(Selector, Mode);
6924:   APInt Result(32, 0);
6925:   for (unsigned I : llvm::seq(4U)) {
6926:     APInt Sel = SelectorVal.extractBits(4, I * 4);
6927:     unsigned Idx = Sel.getLoBits(3).getZExtValue();
6928:     unsigned Sign = Sel.getHiBits(1).getZExtValue();
6929:     APInt Byte = BitField.extractBits(8, Idx * 8);
6930:     if (Sign)
6931:       Byte = Byte.ashr(8);
6932:     Result.insertBits(Byte, I * 8);
6933:   }
6934:   return Result;
6935: }
6936:
6937: static SDValue combinePRMT(SDNode *N, TargetLowering::DAGCombinerInfo &DCI,
6938:                            CodeGenOptLevel OptLevel) {
6939:   if (OptLevel == CodeGenOptLevel::None)
6940:     return SDValue();
6941:
6942:   // Constant fold PRMT
6943:   if (isa<ConstantSDNode>(N->getOperand(0)) &&
6944:       isa<ConstantSDNode>(N->getOperand(1)) &&
6945:       isa<ConstantSDNode>(N->getOperand(2)))
6946:     return DCI.DAG.getConstant(computePRMT(N->getConstantOperandAPInt(0),
6947:                                            N->getConstantOperandAPInt(1),
6948:                                            N->getConstantOperandAPInt(2),
6949:                                            N->getConstantOperandVal(3)),
6950:                                SDLoc(N), N->getValueType(0));
6951:   return SDValue();
6952: }
6953:
6954: // During call lowering we wrap the return values in a ProxyReg node which
6955: // depend on the chain value produced by the completed call. This ensures that
6956: // the full call is emitted in cases where libcalls are used to legalize
6957: // operations. To improve the functioning of other DAG combines we pull all
6958: // operations we can through one of these nodes, ensuring that the ProxyReg
6959: // directly wraps a load. That is:
6960: //
6961: //  (ProxyReg (zext (load retval0)))  =>  (zext (ProxyReg (load retval0)))
6962: //
6963: static SDValue sinkProxyReg(SDValue R, SDValue Chain,
6964:                             TargetLowering::DAGCombinerInfo &DCI) {
6965:   switch (R.getOpcode()) {
6966:   case ISD::TRUNCATE:
6967:   case ISD::ANY_EXTEND:
6968:   case ISD::SIGN_EXTEND:
6969:   case ISD::ZERO_EXTEND:
6970:   case ISD::BITCAST: {
6971:     if (SDValue V = sinkProxyReg(R.getOperand(0), Chain, DCI))
6972:       return DCI.DAG.getNode(R.getOpcode(), SDLoc(R), R.getValueType(), V);
6973:     return SDValue();
6974:   }
6975:   case ISD::SHL:
6976:   case ISD::SRL:
6977:   case ISD::SRA:
6978:   case ISD::OR: {
6979:     if (SDValue A = sinkProxyReg(R.getOperand(0), Chain, DCI))
6980:       if (SDValue B = sinkProxyReg(R.getOperand(1), Chain, DCI))
6981:         return DCI.DAG.getNode(R.getOpcode(), SDLoc(R), R.getValueType(), A, B);
6982:     return SDValue();
6983:   }
6984:   case ISD::Constant:
6985:     return R;
6986:   case ISD::LOAD:
6987:   case NVPTXISD::LoadV2:
6988:   case NVPTXISD::LoadV4: {
6989:     return DCI.DAG.getNode(NVPTXISD::ProxyReg, SDLoc(R), R.getValueType(),
6990:                            {Chain, R});
6991:   }
6992:   case ISD::BUILD_VECTOR: {
6993:     if (DCI.isBeforeLegalize())
6994:       return SDValue();
6995:
6996:     SmallVector<SDValue, 16> Ops;
6997:     for (auto &Op : R->ops()) {
6998:       SDValue V = sinkProxyReg(Op, Chain, DCI);
6999:       if (!V)
7000:         return SDValue();
```
- EN: This range implements operational logic in helpers such as getBitcast, assert, getOperand, SDValue, translating backend policy into executable code.
- CN: 这一段实现了 getBitcast、assert、getOperand、SDValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7001-7140
```cpp
7001:       Ops.push_back(V);
7002:     }
7003:     return DCI.DAG.getNode(ISD::BUILD_VECTOR, SDLoc(R), R.getValueType(), Ops);
7004:   }
7005:   case ISD::EXTRACT_VECTOR_ELT: {
7006:     if (DCI.isBeforeLegalize())
7007:       return SDValue();
7008:
7009:     if (SDValue V = sinkProxyReg(R.getOperand(0), Chain, DCI))
7010:       return DCI.DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(R),
7011:                              R.getValueType(), V, R.getOperand(1));
7012:     return SDValue();
7013:   }
7014:   default:
7015:     return SDValue();
7016:   }
7017: }
7018:
7019: static unsigned getF16SubOpc(Intrinsic::ID AddIntrinsicID) {
7020:   switch (AddIntrinsicID) {
7021:   default:
7022:     break;
7023:   case Intrinsic::nvvm_add_rn_sat_f16:
7024:   case Intrinsic::nvvm_add_rn_sat_v2f16:
7025:     return NVPTXISD::SUB_RN_SAT;
7026:   case Intrinsic::nvvm_add_rn_ftz_sat_f16:
7027:   case Intrinsic::nvvm_add_rn_ftz_sat_v2f16:
7028:     return NVPTXISD::SUB_RN_FTZ_SAT;
7029:   }
7030:   llvm_unreachable("Invalid F16 add intrinsic");
7031: }
7032:
7033: static SDValue combineF16AddWithNeg(SDNode *N, SelectionDAG &DAG,
7034:                                     Intrinsic::ID AddIntrinsicID) {
7035:   SDValue Op1 = N->getOperand(1);
7036:   SDValue Op2 = N->getOperand(2);
7037:
7038:   SDValue SubOp1, SubOp2;
7039:
7040:   if (Op1.getOpcode() == ISD::FNEG) {
7041:     SubOp1 = Op2;
7042:     SubOp2 = Op1.getOperand(0);
7043:   } else if (Op2.getOpcode() == ISD::FNEG) {
7044:     SubOp1 = Op1;
7045:     SubOp2 = Op2.getOperand(0);
7046:   } else {
7047:     return SDValue();
7048:   }
7049:
7050:   SDLoc DL(N);
7051:   return DAG.getNode(getF16SubOpc(AddIntrinsicID), DL, N->getValueType(0),
7052:                      SubOp1, SubOp2);
7053: }
7054:
7055: static SDValue combineIntrinsicWOChain(SDNode *N,
7056:                                        TargetLowering::DAGCombinerInfo &DCI,
7057:                                        const NVPTXSubtarget &STI) {
7058:   unsigned IID = N->getConstantOperandVal(0);
7059:
7060:   switch (IID) {
7061:   default:
7062:     break;
7063:   case Intrinsic::nvvm_add_rn_sat_f16:
7064:   case Intrinsic::nvvm_add_rn_ftz_sat_f16:
7065:   case Intrinsic::nvvm_add_rn_sat_v2f16:
7066:   case Intrinsic::nvvm_add_rn_ftz_sat_v2f16:
7067:     return combineF16AddWithNeg(N, DCI.DAG, IID);
7068:   }
7069:   return SDValue();
7070: }
7071:
7072: static SDValue combineProxyReg(SDNode *N,
7073:                                TargetLowering::DAGCombinerInfo &DCI) {
7074:
7075:   SDValue Chain = N->getOperand(0);
7076:   SDValue Reg = N->getOperand(1);
7077:
7078:   // If the ProxyReg is not wrapping a load, try to pull the operations through
7079:   // the ProxyReg.
7080:   if (Reg.getOpcode() != ISD::LOAD) {
7081:     if (SDValue V = sinkProxyReg(Reg, Chain, DCI))
7082:       return V;
7083:   }
7084:
7085:   return SDValue();
7086: }
7087:
7088: SDValue NVPTXTargetLowering::PerformDAGCombine(SDNode *N,
7089:                                                DAGCombinerInfo &DCI) const {
7090:   CodeGenOptLevel OptLevel = getTargetMachine().getOptLevel();
7091:   switch (N->getOpcode()) {
7092:   default:
7093:     break;
7094:   case ISD::ADD:
7095:     return PerformADDCombine(N, DCI, OptLevel);
7096:   case ISD::ADDRSPACECAST:
7097:     return combineADDRSPACECAST(N, DCI);
7098:   case ISD::SIGN_EXTEND:
7099:   case ISD::ZERO_EXTEND:
7100:     return combineMulWide(N, DCI, OptLevel);
7101:   case ISD::BUILD_VECTOR:
7102:     return PerformBUILD_VECTORCombine(N, DCI);
7103:   case ISD::EXTRACT_VECTOR_ELT:
7104:     return PerformEXTRACTCombine(N, DCI);
7105:   case ISD::FADD:
7106:     return performFADDCombine(N, DCI, OptLevel);
7107:   case ISD::FMA:
7108:   case ISD::FMUL:
7109:   case ISD::FSUB:
7110:     return performScalarizeV2F32Op(N, DCI, OptLevel);
7111:   case ISD::FMAXNUM:
7112:   case ISD::FMINNUM:
7113:   case ISD::FMAXIMUM:
7114:   case ISD::FMINIMUM:
7115:   case ISD::FMAXIMUMNUM:
7116:   case ISD::FMINIMUMNUM:
7117:     return PerformFMinMaxCombine(N, DCI, STI.getPTXVersion(),
7118:                                  STI.getSmVersion());
7119:   case ISD::LOAD:
7120:   case NVPTXISD::LoadV2:
7121:   case NVPTXISD::LoadV4:
7122:     return combineLOAD(N, DCI, STI);
7123:   case ISD::MUL:
7124:     return PerformMULCombine(N, DCI, OptLevel);
7125:   case NVPTXISD::PRMT:
7126:     return combinePRMT(N, DCI, OptLevel);
7127:   case NVPTXISD::ProxyReg:
7128:     return combineProxyReg(N, DCI);
7129:   case ISD::SETCC:
7130:     return PerformSETCCCombine(N, DCI, STI.getSmVersion());
7131:   case ISD::SHL:
7132:     return PerformSHLCombine(N, DCI, OptLevel);
7133:   case ISD::SREM:
7134:   case ISD::UREM:
7135:     return PerformREMCombine(N, DCI, OptLevel);
7136:   case ISD::STORE:
7137:   case NVPTXISD::StoreV2:
7138:   case NVPTXISD::StoreV4:
7139:     return combineSTORE(N, DCI, STI);
7140:   case ISD::SELECT:
```
- EN: This range implements operational logic in helpers such as push_back, getNode, SDValue, getValueType, translating backend policy into executable code.
- CN: 这一段实现了 push_back、getNode、SDValue、getValueType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7141-7280
```cpp
7141:     return PerformSELECTShiftCombine(N, DCI);
7142:   case ISD::VSELECT:
7143:     return PerformVSELECTCombine(N, DCI);
7144:   case ISD::INTRINSIC_WO_CHAIN:
7145:     return combineIntrinsicWOChain(N, DCI, STI);
7146:   }
7147:   return SDValue();
7148: }
7149:
7150: static void ReplaceBITCAST(SDNode *Node, SelectionDAG &DAG,
7151:                            SmallVectorImpl<SDValue> &Results) {
7152:   // Handle bitcasting to v2i8 without hitting the default promotion
7153:   // strategy which goes through stack memory.
7154:   SDValue Op(Node, 0);
7155:   EVT ToVT = Op->getValueType(0);
7156:   if (ToVT != MVT::v2i8) {
7157:     return;
7158:   }
7159:
7160:   // Bitcast to i16 and unpack elements into a vector
7161:   SDLoc DL(Node);
7162:   SDValue AsInt = DAG.getBitcast(MVT::i16, Op->getOperand(0));
7163:   SDValue Vec0 = DAG.getNode(ISD::TRUNCATE, DL, MVT::i8, AsInt);
7164:   SDValue Const8 = DAG.getConstant(8, DL, MVT::i16);
7165:   SDValue Vec1 =
7166:       DAG.getNode(ISD::TRUNCATE, DL, MVT::i8,
7167:                   DAG.getNode(ISD::SRL, DL, MVT::i16, {AsInt, Const8}));
7168:   Results.push_back(
7169:       DAG.getNode(ISD::BUILD_VECTOR, DL, MVT::v2i8, {Vec0, Vec1}));
7170: }
7171:
7172: static void ReplaceINTRINSIC_W_CHAIN(SDNode *N, SelectionDAG &DAG,
7173:                                      SmallVectorImpl<SDValue> &Results) {
7174:   SDValue Chain = N->getOperand(0);
7175:   SDValue Intrin = N->getOperand(1);
7176:   SDLoc DL(N);
7177:
7178:   // Get the intrinsic ID
7179:   unsigned IntrinNo = Intrin.getNode()->getAsZExtVal();
7180:   switch (IntrinNo) {
7181:   default:
7182:     return;
7183:   case Intrinsic::nvvm_ldu_global_i:
7184:   case Intrinsic::nvvm_ldu_global_f:
7185:   case Intrinsic::nvvm_ldu_global_p: {
7186:     EVT ResVT = N->getValueType(0);
7187:
7188:     if (ResVT.isVector()) {
7189:       // Vector LDG/LDU
7190:
7191:       unsigned NumElts = ResVT.getVectorNumElements();
7192:       EVT EltVT = ResVT.getVectorElementType();
7193:
7194:       // Since LDU/LDG are target nodes, we cannot rely on DAG type
7195:       // legalization.
7196:       // Therefore, we must ensure the type is legal.  For i1 and i8, we set the
7197:       // loaded type to i16 and propagate the "real" type as the memory type.
7198:       bool NeedTrunc = false;
7199:       if (EltVT.getSizeInBits() < 16) {
7200:         EltVT = MVT::i16;
7201:         NeedTrunc = true;
7202:       }
7203:
7204:       unsigned Opcode = 0;
7205:       SDVTList LdResVTs;
7206:
7207:       switch (NumElts) {
7208:       default:
7209:         return;
7210:       case 2:
7211:         Opcode = NVPTXISD::LDUV2;
7212:         LdResVTs = DAG.getVTList(EltVT, EltVT, MVT::Other);
7213:         break;
7214:       case 4: {
7215:         Opcode = NVPTXISD::LDUV4;
7216:         EVT ListVTs[] = { EltVT, EltVT, EltVT, EltVT, MVT::Other };
7217:         LdResVTs = DAG.getVTList(ListVTs);
7218:         break;
7219:       }
7220:       }
7221:
7222:       SmallVector<SDValue, 8> OtherOps;
7223:
7224:       // Copy regular operands
7225:
7226:       OtherOps.push_back(Chain); // Chain
7227:                                  // Skip operand 1 (intrinsic ID)
7228:       // Others
7229:       OtherOps.append(N->op_begin() + 2, N->op_end());
7230:
7231:       MemIntrinsicSDNode *MemSD = cast<MemIntrinsicSDNode>(N);
7232:
7233:       SDValue NewLD = DAG.getMemIntrinsicNode(Opcode, DL, LdResVTs, OtherOps,
7234:                                               MemSD->getMemoryVT(),
7235:                                               MemSD->getMemOperand());
7236:
7237:       SmallVector<SDValue, 4> ScalarRes;
7238:
7239:       for (unsigned i = 0; i < NumElts; ++i) {
7240:         SDValue Res = NewLD.getValue(i);
7241:         if (NeedTrunc)
7242:           Res =
7243:               DAG.getNode(ISD::TRUNCATE, DL, ResVT.getVectorElementType(), Res);
7244:         ScalarRes.push_back(Res);
7245:       }
7246:
7247:       SDValue LoadChain = NewLD.getValue(NumElts);
7248:
7249:       SDValue BuildVec =
7250:           DAG.getBuildVector(ResVT, DL, ScalarRes);
7251:
7252:       Results.push_back(BuildVec);
7253:       Results.push_back(LoadChain);
7254:     } else {
7255:       // i8 LDG/LDU
7256:       assert(ResVT.isSimple() && ResVT.getSimpleVT().SimpleTy == MVT::i8 &&
7257:              "Custom handling of non-i8 ldu/ldg?");
7258:
7259:       // Just copy all operands as-is
7260:       SmallVector<SDValue, 4> Ops(N->ops());
7261:
7262:       // Force output to i16
7263:       SDVTList LdResVTs = DAG.getVTList(MVT::i16, MVT::Other);
7264:
7265:       MemIntrinsicSDNode *MemSD = cast<MemIntrinsicSDNode>(N);
7266:
7267:       // We make sure the memory type is i8, which will be used during isel
7268:       // to select the proper instruction.
7269:       SDValue NewLD =
7270:           DAG.getMemIntrinsicNode(ISD::INTRINSIC_W_CHAIN, DL, LdResVTs, Ops,
7271:                                   MVT::i8, MemSD->getMemOperand());
7272:
7273:       Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, MVT::i8,
7274:                                     NewLD.getValue(0)));
7275:       Results.push_back(NewLD.getValue(1));
7276:     }
7277:     return;
7278:   }
7279:
7280:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x4:
```
- EN: This range implements operational logic in helpers such as PerformSELECTShiftCombine, PerformVSELECTCombine, combineIntrinsicWOChain, SDValue, translating backend policy into executable code.
- CN: 这一段实现了 PerformSELECTShiftCombine、PerformVSELECTCombine、combineIntrinsicWOChain、SDValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7281-7420
```cpp
7281:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x8:
7282:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x16:
7283:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x32:
7284:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x64:
7285:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x128:
7286:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x4:
7287:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x8:
7288:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x16:
7289:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x32:
7290:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x64:
7291:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x128:
7292:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x2:
7293:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x4:
7294:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x8:
7295:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x16:
7296:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x32:
7297:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x64:
7298:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x1:
7299:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x2:
7300:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x4:
7301:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x8:
7302:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x16:
7303:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x32:
7304:     if (auto Res = lowerTcgen05Ld(N, DAG)) {
7305:       Results.push_back(Res->first);
7306:       Results.push_back(Res->second);
7307:     }
7308:     return;
7309:
7310:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x4:
7311:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x8:
7312:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x16:
7313:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x32:
7314:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x64:
7315:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x128:
7316:     if (auto Res = lowerTcgen05Ld(N, DAG, /*HasOffset=*/true)) {
7317:       Results.push_back(Res->first);
7318:       Results.push_back(Res->second);
7319:     }
7320:     return;
7321:
7322:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x8_i32:
7323:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x8_f32:
7324:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x64_i32:
7325:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x64_f32:
7326:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x4_i32:
7327:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x4_f32:
7328:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x32_i32:
7329:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x32_f32:
7330:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x16_i32:
7331:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x16_f32:
7332:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x128_i32:
7333:   case Intrinsic::nvvm_tcgen05_ld_red_32x32b_x128_f32:
7334:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x8_i32:
7335:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x8_f32:
7336:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x64_i32:
7337:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x64_f32:
7338:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x4_i32:
7339:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x4_f32:
7340:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x32_i32:
7341:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x32_f32:
7342:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x16_i32:
7343:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x16_f32:
7344:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x128_i32:
7345:   case Intrinsic::nvvm_tcgen05_ld_red_16x32bx2_x128_f32:
7346:     if (auto Res = lowerTcgen05LdRed(N, DAG)) {
7347:       Results.push_back(std::get<0>(*Res));
7348:       Results.push_back(std::get<1>(*Res));
7349:       Results.push_back(std::get<2>(*Res));
7350:     }
7351:     return;
7352:   }
7353: }
7354:
7355: static void ReplaceCopyFromReg_128(SDNode *N, SelectionDAG &DAG,
7356:                                    SmallVectorImpl<SDValue> &Results) {
7357:   // Change the CopyFromReg to output 2 64-bit results instead of a 128-bit
7358:   // result so that it can pass the legalization
7359:   SDLoc DL(N);
7360:   SDValue Chain = N->getOperand(0);
7361:   SDValue Reg = N->getOperand(1);
7362:   SDValue Glue = N->getOperand(2);
7363:
7364:   assert(Reg.getValueType() == MVT::i128 &&
7365:          "Custom lowering for CopyFromReg with 128-bit reg only");
7366:   SmallVector<EVT, 4> ResultsType = {MVT::i64, MVT::i64, N->getValueType(1),
7367:                                      N->getValueType(2)};
7368:   SmallVector<SDValue, 3> NewOps = {Chain, Reg, Glue};
7369:
7370:   SDValue NewValue = DAG.getNode(ISD::CopyFromReg, DL, ResultsType, NewOps);
7371:   SDValue Pair = DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i128,
7372:                              {NewValue.getValue(0), NewValue.getValue(1)});
7373:
7374:   Results.push_back(Pair);
7375:   Results.push_back(NewValue.getValue(2));
7376:   Results.push_back(NewValue.getValue(3));
7377: }
7378:
7379: static void replaceProxyReg(SDNode *N, SelectionDAG &DAG,
7380:                             const TargetLowering &TLI,
7381:                             SmallVectorImpl<SDValue> &Results) {
7382:   SDValue Chain = N->getOperand(0);
7383:   SDValue Reg = N->getOperand(1);
7384:
7385:   MVT VT = TLI.getRegisterType(*DAG.getContext(), Reg.getValueType());
7386:
7387:   SDValue NewReg = DAG.getAnyExtOrTrunc(Reg, SDLoc(N), VT);
7388:   SDValue NewProxy =
7389:       DAG.getNode(NVPTXISD::ProxyReg, SDLoc(N), VT, {Chain, NewReg});
7390:   SDValue Res = DAG.getAnyExtOrTrunc(NewProxy, SDLoc(N), N->getValueType(0));
7391:
7392:   Results.push_back(Res);
7393: }
7394:
7395: static void replaceAtomicSwap128(SDNode *N, SelectionDAG &DAG,
7396:                                  const NVPTXSubtarget &STI,
7397:                                  SmallVectorImpl<SDValue> &Results) {
7398:   assert(N->getValueType(0) == MVT::i128 &&
7399:          "Custom lowering for atomic128 only supports i128");
7400:
7401:   AtomicSDNode *AN = cast<AtomicSDNode>(N);
7402:   SDLoc dl(N);
7403:
7404:   if (!STI.hasAtomSwap128()) {
7405:     DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
7406:         DAG.getMachineFunction().getFunction(),
7407:         "Support for b128 atomics introduced in PTX ISA version 8.3 and "
7408:         "requires target sm_90.",
7409:         dl.getDebugLoc()));
7410:
7411:     Results.push_back(DAG.getUNDEF(MVT::i128));
7412:     Results.push_back(AN->getOperand(0)); // Chain
7413:     return;
7414:   }
7415:
7416:   SmallVector<SDValue, 6> Ops;
7417:   Ops.push_back(AN->getOperand(0)); // Chain
7418:   Ops.push_back(AN->getOperand(1)); // Ptr
7419:   for (const auto &Op : AN->ops().drop_front(2)) {
7420:     // Low part
```
- EN: This range implements operational logic in helpers such as push_back, DL, getOperand, getNode, translating backend policy into executable code.
- CN: 这一段实现了 push_back、DL、getOperand、getNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7421-7560
```cpp
7421:     Ops.push_back(DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i64, Op,
7422:                               DAG.getIntPtrConstant(0, dl)));
7423:     // High part
7424:     Ops.push_back(DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i64, Op,
7425:                               DAG.getIntPtrConstant(1, dl)));
7426:   }
7427:   unsigned Opcode = N->getOpcode() == ISD::ATOMIC_SWAP
7428:                         ? NVPTXISD::ATOMIC_SWAP_B128
7429:                         : NVPTXISD::ATOMIC_CMP_SWAP_B128;
7430:   SDVTList Tys = DAG.getVTList(MVT::i64, MVT::i64, MVT::Other);
7431:   SDValue Result = DAG.getMemIntrinsicNode(Opcode, dl, Tys, Ops, MVT::i128,
7432:                                            AN->getMemOperand());
7433:   Results.push_back(DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i128,
7434:                                 {Result.getValue(0), Result.getValue(1)}));
7435:   Results.push_back(Result.getValue(2));
7436: }
7437:
7438: void NVPTXTargetLowering::ReplaceNodeResults(
7439:     SDNode *N, SmallVectorImpl<SDValue> &Results, SelectionDAG &DAG) const {
7440:   switch (N->getOpcode()) {
7441:   default:
7442:     report_fatal_error("Unhandled custom legalization");
7443:   case ISD::BITCAST:
7444:     ReplaceBITCAST(N, DAG, Results);
7445:     return;
7446:   case ISD::LOAD:
7447:   case ISD::MLOAD:
7448:     replaceLoadVector(N, DAG, Results, STI);
7449:     return;
7450:   case ISD::INTRINSIC_W_CHAIN:
7451:     ReplaceINTRINSIC_W_CHAIN(N, DAG, Results);
7452:     return;
7453:   case ISD::CopyFromReg:
7454:     ReplaceCopyFromReg_128(N, DAG, Results);
7455:     return;
7456:   case NVPTXISD::ProxyReg:
7457:     replaceProxyReg(N, DAG, *this, Results);
7458:     return;
7459:   case ISD::ATOMIC_CMP_SWAP:
7460:   case ISD::ATOMIC_SWAP:
7461:     replaceAtomicSwap128(N, DAG, STI, Results);
7462:     return;
7463:   }
7464: }
7465:
7466: NVPTXTargetLowering::AtomicExpansionKind
7467: NVPTXTargetLowering::shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const {
7468:   Type *Ty = AI->getValOperand()->getType();
7469:
7470:   if (AI->isFloatingPointOperation()) {
7471:     if (AI->getOperation() == AtomicRMWInst::BinOp::FAdd) {
7472:       if (Ty->isHalfTy() && STI.getSmVersion() >= 70 &&
7473:           STI.getPTXVersion() >= 63)
7474:         return AtomicExpansionKind::None;
7475:       if (Ty->isBFloatTy() && STI.getSmVersion() >= 90 &&
7476:           STI.getPTXVersion() >= 78)
7477:         return AtomicExpansionKind::None;
7478:       if (Ty->isFloatTy())
7479:         return AtomicExpansionKind::None;
7480:       if (Ty->isDoubleTy() && STI.hasAtomAddF64())
7481:         return AtomicExpansionKind::None;
7482:     }
7483:     return AtomicExpansionKind::CmpXChg;
7484:   }
7485:
7486:   assert(Ty->isIntegerTy() && "Ty should be integer at this point");
7487:   const unsigned BitWidth = cast<IntegerType>(Ty)->getBitWidth();
7488:
7489:   switch (AI->getOperation()) {
7490:   default:
7491:     return AtomicExpansionKind::CmpXChg;
7492:   case AtomicRMWInst::BinOp::Xchg:
7493:     if (BitWidth == 128)
7494:       return AtomicExpansionKind::None;
7495:     [[fallthrough]];
7496:   case AtomicRMWInst::BinOp::And:
7497:   case AtomicRMWInst::BinOp::Or:
7498:   case AtomicRMWInst::BinOp::Xor:
7499:     switch (BitWidth) {
7500:     case 8:
7501:     case 16:
7502:       return AtomicExpansionKind::CmpXChg;
7503:     case 32:
7504:       return AtomicExpansionKind::None;
7505:     case 64:
7506:       if (STI.hasAtomBitwise64())
7507:         return AtomicExpansionKind::None;
7508:       return AtomicExpansionKind::CmpXChg;
7509:     case 128:
7510:       return AtomicExpansionKind::CmpXChg;
7511:     default:
7512:       llvm_unreachable("unsupported width encountered");
7513:     }
7514:   case AtomicRMWInst::BinOp::Add:
7515:   case AtomicRMWInst::BinOp::Sub:
7516:   case AtomicRMWInst::BinOp::Max:
7517:   case AtomicRMWInst::BinOp::Min:
7518:   case AtomicRMWInst::BinOp::UMax:
7519:   case AtomicRMWInst::BinOp::UMin:
7520:     switch (BitWidth) {
7521:     case 8:
7522:     case 16:
7523:       return AtomicExpansionKind::CmpXChg;
7524:     case 32:
7525:       return AtomicExpansionKind::None;
7526:     case 64:
7527:       if (STI.hasAtomMinMax64())
7528:         return AtomicExpansionKind::None;
7529:       return AtomicExpansionKind::CmpXChg;
7530:     case 128:
7531:       return AtomicExpansionKind::CmpXChg;
7532:     default:
7533:       llvm_unreachable("unsupported width encountered");
7534:     }
7535:   case AtomicRMWInst::BinOp::UIncWrap:
7536:   case AtomicRMWInst::BinOp::UDecWrap:
7537:     switch (BitWidth) {
7538:     case 32:
7539:       return AtomicExpansionKind::None;
7540:     case 8:
7541:     case 16:
7542:     case 64:
7543:     case 128:
7544:       return AtomicExpansionKind::CmpXChg;
7545:     default:
7546:       llvm_unreachable("unsupported width encountered");
7547:     }
7548:   }
7549:
7550:   return AtomicExpansionKind::CmpXChg;
7551: }
7552:
7553: bool NVPTXTargetLowering::shouldInsertFencesForAtomic(
7554:     const Instruction *I) const {
7555:   // This function returns true iff the operation is emulated using a CAS-loop,
7556:   // or if it has the memory order seq_cst (which is not natively supported in
7557:   // the PTX `atom` instruction).
7558:   //
7559:   // atomicrmw and cmpxchg instructions not efficiently supported by PTX
7560:   // are lowered to CAS emulation loops that preserve their memory order,
```
- EN: This range implements operational logic in helpers such as getIntPtrConstant, getVTList, getMemOperand, getValue, translating backend policy into executable code.
- CN: 这一段实现了 getIntPtrConstant、getVTList、getMemOperand、getValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7561-7700
```cpp
7561:   // syncscope, and volatile semantics. For PTX, it is more efficient to use
7562:   // atom.cas.relaxed.sco instructions within the loop, and fences before and
7563:   // after the loop to restore order.
7564:   //
7565:   // Atomic instructions efficiently supported by PTX are lowered to
7566:   // `atom.<op>.<sem>.<scope` instruction with their corresponding memory order
7567:   // and scope. Since PTX does not support seq_cst, we emulate it by lowering to
7568:   // a fence.sc followed by an atom according to the PTX atomics ABI
7569:   // https://docs.nvidia.com/cuda/ptx-writers-guide-to-interoperability/atomic-abi.html
7570:   if (auto *CI = dyn_cast<AtomicCmpXchgInst>(I))
7571:     return (cast<IntegerType>(CI->getCompareOperand()->getType())
7572:                 ->getBitWidth() < STI.getMinCmpXchgSizeInBits()) ||
7573:            CI->getMergedOrdering() == AtomicOrdering::SequentiallyConsistent;
7574:   if (auto *RI = dyn_cast<AtomicRMWInst>(I))
7575:     return shouldExpandAtomicRMWInIR(RI) == AtomicExpansionKind::CmpXChg ||
7576:            RI->getOrdering() == AtomicOrdering::SequentiallyConsistent;
7577:   return false;
7578: }
7579:
7580: AtomicOrdering NVPTXTargetLowering::atomicOperationOrderAfterFenceSplit(
7581:     const Instruction *I) const {
7582:   // If the operation is emulated by a CAS-loop, we lower the instruction to
7583:   // atom.<op>.relaxed, since AtomicExpandPass will insert fences for enforcing
7584:   // the correct memory ordering around the CAS loop.
7585:   //
7586:   // When the operation is not emulated, but the memory order is seq_cst,
7587:   // we must lower to "fence.sc.<scope>; atom.<op>.acquire.<scope>;" to conform
7588:   // to the PTX atomics ABI.
7589:   // https://docs.nvidia.com/cuda/ptx-writers-guide-to-interoperability/atomic-abi.html
7590:   // For such cases, emitLeadingFence() will separately insert the leading
7591:   // "fence.sc.<scope>;". Here, we only set the memory order to acquire.
7592:   //
7593:   // Otherwise, the operation is not emulated, and the memory order is not
7594:   // seq_cst.  In this case, the LLVM memory order is natively supported by the
7595:   // PTX `atom` instruction, and we just lower to the corresponding
7596:   // `atom.<op>.relaxed|acquire|release|acq_rel". For such cases, this function
7597:   // will NOT be called.
7598:   // prerequisite: shouldInsertFencesForAtomic() should have returned `true` for
7599:   // I before its memory order was modified.
7600:   if (auto *CI = dyn_cast<AtomicCmpXchgInst>(I);
7601:       CI && CI->getMergedOrdering() == AtomicOrdering::SequentiallyConsistent &&
7602:       cast<IntegerType>(CI->getCompareOperand()->getType())->getBitWidth() >=
7603:           STI.getMinCmpXchgSizeInBits())
7604:     return AtomicOrdering::Acquire;
7605:   else if (auto *RI = dyn_cast<AtomicRMWInst>(I);
7606:            RI && RI->getOrdering() == AtomicOrdering::SequentiallyConsistent &&
7607:            shouldExpandAtomicRMWInIR(RI) == AtomicExpansionKind::None)
7608:     return AtomicOrdering::Acquire;
7609:
7610:   return AtomicOrdering::Monotonic;
7611: }
7612:
7613: Instruction *NVPTXTargetLowering::emitLeadingFence(IRBuilderBase &Builder,
7614:                                                    Instruction *Inst,
7615:                                                    AtomicOrdering Ord) const {
7616:   // prerequisite: shouldInsertFencesForAtomic() should have returned `true` for
7617:   // `Inst` before its memory order was modified. We cannot enforce this with an
7618:   // assert, because AtomicExpandPass will have modified the memory order
7619:   // between the initial call to shouldInsertFencesForAtomic() and the call to
7620:   // this function.
7621:   if (!isa<AtomicCmpXchgInst>(Inst) && !isa<AtomicRMWInst>(Inst))
7622:     return TargetLoweringBase::emitLeadingFence(Builder, Inst, Ord);
7623:
7624:   // Specialize for cmpxchg and atomicrmw
7625:   auto SSID = getAtomicSyncScopeID(Inst);
7626:   assert(SSID.has_value() && "Expected an atomic operation");
7627:
7628:   if (isReleaseOrStronger(Ord))
7629:     return Builder.CreateFence(Ord == AtomicOrdering::SequentiallyConsistent
7630:                                    ? AtomicOrdering::SequentiallyConsistent
7631:                                    : AtomicOrdering::Release,
7632:                                SSID.value());
7633:
7634:   return nullptr;
7635: }
7636:
7637: Instruction *NVPTXTargetLowering::emitTrailingFence(IRBuilderBase &Builder,
7638:                                                     Instruction *Inst,
7639:                                                     AtomicOrdering Ord) const {
7640:   // prerequisite: shouldInsertFencesForAtomic() should have returned `true` for
7641:   // `Inst` before its memory order was modified. See `emitLeadingFence` for why
7642:   // this cannot be enforced with an assert.  Specialize for cmpxchg and
7643:   // atomicrmw
7644:   auto *CI = dyn_cast<AtomicCmpXchgInst>(Inst);
7645:   auto *RI = dyn_cast<AtomicRMWInst>(Inst);
7646:   if (!CI && !RI)
7647:     return TargetLoweringBase::emitTrailingFence(Builder, Inst, Ord);
7648:
7649:   auto SSID = getAtomicSyncScopeID(Inst);
7650:   assert(SSID.has_value() && "Expected an atomic operation");
7651:
7652:   bool IsEmulated =
7653:       CI ? cast<IntegerType>(CI->getCompareOperand()->getType())
7654:                    ->getBitWidth() < STI.getMinCmpXchgSizeInBits()
7655:          : shouldExpandAtomicRMWInIR(RI) == AtomicExpansionKind::CmpXChg;
7656:
7657:   if (isAcquireOrStronger(Ord) && IsEmulated)
7658:     return Builder.CreateFence(AtomicOrdering::Acquire, SSID.value());
7659:
7660:   return nullptr;
7661: }
7662:
7663: // Rather than default to SINT when both UINT and SINT are custom, we only
7664: // change the opcode when UINT is not legal and SINT is. UINT is preferred when
7665: // both are custom since unsigned CVT instructions can lead to slightly better
7666: // SASS code with fewer instructions.
7667: unsigned NVPTXTargetLowering::getPreferredFPToIntOpcode(unsigned Op, EVT FromVT,
7668:                                                         EVT ToVT) const {
7669:   if (isOperationLegal(Op, ToVT))
7670:     return Op;
7671:   switch (Op) {
7672:   case ISD::FP_TO_UINT:
7673:     if (isOperationLegal(ISD::FP_TO_SINT, ToVT))
7674:       return ISD::FP_TO_SINT;
7675:     break;
7676:   case ISD::STRICT_FP_TO_UINT:
7677:     if (isOperationLegal(ISD::STRICT_FP_TO_SINT, ToVT))
7678:       return ISD::STRICT_FP_TO_SINT;
7679:     break;
7680:   case ISD::VP_FP_TO_UINT:
7681:     if (isOperationLegal(ISD::VP_FP_TO_SINT, ToVT))
7682:       return ISD::VP_FP_TO_SINT;
7683:     break;
7684:   default:
7685:     break;
7686:   }
7687:   return Op;
7688: }
7689:
7690: // Pin NVPTXTargetObjectFile's vtables to this file.
7691: NVPTXTargetObjectFile::~NVPTXTargetObjectFile() = default;
7692:
7693: MCSection *NVPTXTargetObjectFile::SelectSectionForGlobal(
7694:     const GlobalObject *GO, SectionKind Kind, const TargetMachine &TM) const {
7695:   return getDataSection();
7696: }
7697:
7698: static void computeKnownBitsForPRMT(const SDValue Op, KnownBits &Known,
7699:                                     const SelectionDAG &DAG, unsigned Depth) {
7700:   SDValue A = Op.getOperand(0);
```
- EN: This range implements operational logic in helpers such as getMinCmpXchgSizeInBits, shouldExpandAtomicRMWInIR, TargetLoweringBase::emitLeadingFence, getAtomicSyncScopeID, translating backend policy into executable code.
- CN: 这一段实现了 getMinCmpXchgSizeInBits、shouldExpandAtomicRMWInIR、TargetLoweringBase::emitLeadingFence、getAtomicSyncScopeID 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7701-7840
```cpp
7701:   SDValue B = Op.getOperand(1);
7702:   ConstantSDNode *Selector = dyn_cast<ConstantSDNode>(Op.getOperand(2));
7703:   unsigned Mode = Op.getConstantOperandVal(3);
7704:
7705:   if (!Selector)
7706:     return;
7707:
7708:   KnownBits AKnown = DAG.computeKnownBits(A, Depth);
7709:   KnownBits BKnown = DAG.computeKnownBits(B, Depth);
7710:
7711:   // {b, a} = {{b7, b6, b5, b4}, {b3, b2, b1, b0}}
7712:   assert(AKnown.getBitWidth() == 32 && BKnown.getBitWidth() == 32 &&
7713:          "PRMT must have i32 operands");
7714:   assert(Known.getBitWidth() == 32 && "PRMT must have i32 result");
7715:   KnownBits BitField = BKnown.concat(AKnown);
7716:
7717:   APInt SelectorVal = getPRMTSelector(Selector->getAPIntValue(), Mode);
7718:   for (unsigned I : llvm::seq(4)) {
7719:     APInt Sel = SelectorVal.extractBits(4, I * 4);
7720:     unsigned Idx = Sel.getLoBits(3).getZExtValue();
7721:     unsigned Sign = Sel.getHiBits(1).getZExtValue();
7722:     KnownBits Byte = BitField.extractBits(8, Idx * 8);
7723:     if (Sign)
7724:       Byte = KnownBits::ashr(Byte, KnownBits::makeConstant(APInt(8, 7)));
7725:     Known.insertBits(Byte, I * 8);
7726:   }
7727: }
7728:
7729: static void computeKnownBitsForLoadV(const SDValue Op, KnownBits &Known) {
7730:   MemSDNode *LD = cast<MemSDNode>(Op);
7731:
7732:   // We can't do anything without knowing the sign bit.
7733:   auto ExtType = LD->getConstantOperandVal(LD->getNumOperands() - 1);
7734:   if (ExtType == ISD::SEXTLOAD)
7735:     return;
7736:
7737:   // ExtLoading to vector types is weird and may not work well with known bits.
7738:   auto DestVT = LD->getValueType(0);
7739:   if (DestVT.isVector())
7740:     return;
7741:
7742:   assert(Known.getBitWidth() == DestVT.getSizeInBits());
7743:   auto ElementBitWidth = NVPTXDAGToDAGISel::getFromTypeWidthForLoad(LD);
7744:   Known.Zero.setHighBits(Known.getBitWidth() - ElementBitWidth);
7745: }
7746:
7747: void NVPTXTargetLowering::computeKnownBitsForTargetNode(
7748:     const SDValue Op, KnownBits &Known, const APInt &DemandedElts,
7749:     const SelectionDAG &DAG, unsigned Depth) const {
7750:   Known.resetAll();
7751:
7752:   switch (Op.getOpcode()) {
7753:   case NVPTXISD::PRMT:
7754:     computeKnownBitsForPRMT(Op, Known, DAG, Depth);
7755:     break;
7756:   case NVPTXISD::LoadV2:
7757:   case NVPTXISD::LoadV4:
7758:   case NVPTXISD::LoadV8:
7759:     computeKnownBitsForLoadV(Op, Known);
7760:     break;
7761:   default:
7762:     break;
7763:   }
7764: }
7765:
7766: static std::pair<APInt, APInt> getPRMTDemandedBits(const APInt &SelectorVal,
7767:                                                    const APInt &DemandedBits) {
7768:   APInt DemandedLHS = APInt(32, 0);
7769:   APInt DemandedRHS = APInt(32, 0);
7770:
7771:   for (unsigned I : llvm::seq(4)) {
7772:     if (DemandedBits.extractBits(8, I * 8).isZero())
7773:       continue;
7774:
7775:     APInt Sel = SelectorVal.extractBits(4, I * 4);
7776:     unsigned Idx = Sel.getLoBits(3).getZExtValue();
7777:     unsigned Sign = Sel.getHiBits(1).getZExtValue();
7778:
7779:     APInt &Src = Idx < 4 ? DemandedLHS : DemandedRHS;
7780:     unsigned ByteStart = (Idx % 4) * 8;
7781:     if (Sign)
7782:       Src.setBit(ByteStart + 7);
7783:     else
7784:       Src.setBits(ByteStart, ByteStart + 8);
7785:   }
7786:
7787:   return {DemandedLHS, DemandedRHS};
7788: }
7789:
7790: // Replace undef with 0 as this is easier for other optimizations such as
7791: // known bits.
7792: static SDValue canonicalizePRMTInput(SDValue Op, SelectionDAG &DAG) {
7793:   if (!Op)
7794:     return SDValue();
7795:   if (Op.isUndef())
7796:     return DAG.getConstant(0, SDLoc(), MVT::i32);
7797:   return Op;
7798: }
7799:
7800: static SDValue simplifyDemandedBitsForPRMT(SDValue PRMT,
7801:                                            const APInt &DemandedBits,
7802:                                            SelectionDAG &DAG,
7803:                                            const TargetLowering &TLI,
7804:                                            unsigned Depth) {
7805:   assert(PRMT.getOpcode() == NVPTXISD::PRMT);
7806:   SDValue Op0 = PRMT.getOperand(0);
7807:   SDValue Op1 = PRMT.getOperand(1);
7808:   auto *SelectorConst = dyn_cast<ConstantSDNode>(PRMT.getOperand(2));
7809:   if (!SelectorConst)
7810:     return SDValue();
7811:
7812:   unsigned Mode = PRMT.getConstantOperandVal(3);
7813:   const APInt Selector = getPRMTSelector(SelectorConst->getAPIntValue(), Mode);
7814:
7815:   // Try to simplify the PRMT to one of the inputs if the used bytes are all
7816:   // from the same input in the correct order.
7817:   const unsigned LeadingBytes = DemandedBits.countLeadingZeros() / 8;
7818:   const unsigned SelBits = (4 - LeadingBytes) * 4;
7819:   if (Selector.getLoBits(SelBits) == APInt(32, 0x3210).getLoBits(SelBits))
7820:     return Op0;
7821:   if (Selector.getLoBits(SelBits) == APInt(32, 0x7654).getLoBits(SelBits))
7822:     return Op1;
7823:
7824:   auto [DemandedLHS, DemandedRHS] = getPRMTDemandedBits(Selector, DemandedBits);
7825:
7826:   // Attempt to avoid multi-use ops if we don't need anything from them.
7827:   SDValue DemandedOp0 =
7828:       TLI.SimplifyMultipleUseDemandedBits(Op0, DemandedLHS, DAG, Depth + 1);
7829:   SDValue DemandedOp1 =
7830:       TLI.SimplifyMultipleUseDemandedBits(Op1, DemandedRHS, DAG, Depth + 1);
7831:
7832:   DemandedOp0 = canonicalizePRMTInput(DemandedOp0, DAG);
7833:   DemandedOp1 = canonicalizePRMTInput(DemandedOp1, DAG);
7834:   if ((DemandedOp0 && DemandedOp0 != Op0) ||
7835:       (DemandedOp1 && DemandedOp1 != Op1)) {
7836:     Op0 = DemandedOp0 ? DemandedOp0 : Op0;
7837:     Op1 = DemandedOp1 ? DemandedOp1 : Op1;
7838:     return getPRMT(Op0, Op1, Selector.getZExtValue(), SDLoc(PRMT), DAG);
7839:   }
7840:
```
- EN: This range implements operational logic in helpers such as getOperand, getConstantOperandVal, computeKnownBits, assert, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getConstantOperandVal、computeKnownBits、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 7841-7863
```cpp
7841:   return SDValue();
7842: }
7843:
7844: bool NVPTXTargetLowering::SimplifyDemandedBitsForTargetNode(
7845:     SDValue Op, const APInt &DemandedBits, const APInt &DemandedElts,
7846:     KnownBits &Known, TargetLoweringOpt &TLO, unsigned Depth) const {
7847:   Known.resetAll();
7848:
7849:   switch (Op.getOpcode()) {
7850:   case NVPTXISD::PRMT:
7851:     if (SDValue Result = simplifyDemandedBitsForPRMT(Op, DemandedBits, TLO.DAG,
7852:                                                      *this, Depth)) {
7853:       TLO.CombineTo(Op, Result);
7854:       return true;
7855:     }
7856:     break;
7857:   default:
7858:     break;
7859:   }
7860:
7861:   computeKnownBitsForTargetNode(Op, Known, DemandedElts, TLO.DAG, Depth);
7862:   return false;
7863: }
```
- EN: This range implements operational logic in helpers such as SDValue, resetAll, CombineTo, computeKnownBitsForTargetNode, translating backend policy into executable code.
- CN: 这一段实现了 SDValue、resetAll、CombineTo、computeKnownBitsForTargetNode 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Instruction selection maps generic LLVM operations onto target-specific machine instructions.
  - CN: 指令选择负责把通用 LLVM 操作映射为目标相关的机器指令。
- EN: Lowering turns higher-level IR constructs into forms accepted by the target pipeline.
  - CN: 降级过程会把更高层的 IR 构造转换成目标流水线可接受的形式。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include cl::desc, cl::init, getFlags, NVPTXTargetLowering::usePrecSqrtF32, NVPTXTargetLowering::useF32FTZ, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::desc, cl::init, getFlags, NVPTXTargetLowering::usePrecSqrtF32, NVPTXTargetLowering::useF32FTZ，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXISelLowering.h`
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `NVPTX.h`
  - `NVPTXISelDAGToDAG.h`
  - `NVPTXSelectionDAGInfo.h`
  - `NVPTXSubtarget.h`
  - `NVPTXTargetMachine.h`
  - `NVPTXTargetObjectFile.h`
  - `NVPTXUtilities.h`
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APFloat.h`
  - `llvm/ADT/APInt.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/CodeGen/Analysis.h`
  - `llvm/CodeGen/ISDOpcodes.h`
  - `llvm/CodeGen/MachineFunction.h`
  - `llvm/CodeGen/MachineJumpTableInfo.h`
  - `llvm/CodeGen/MachineMemOperand.h`
  - `llvm/CodeGen/SDPatternMatch.h`
  - `llvm/CodeGen/SelectionDAG.h`
  - `llvm/CodeGen/SelectionDAGNodes.h`
  - `llvm/CodeGen/TargetCallingConv.h`
  - `llvm/CodeGen/TargetLowering.h`
  - `llvm/CodeGen/ValueTypes.h`
- System/standard headers / 系统或标准头文件:
  - `algorithm`
  - `cassert`
  - `cmath`
  - `cstdint`
  - `iterator`
  - `optional`
  - `string`
  - `tuple`
