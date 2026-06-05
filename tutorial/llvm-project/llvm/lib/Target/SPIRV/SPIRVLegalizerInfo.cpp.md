# SPIRVLegalizerInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVLegalizerInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the targeting of the Machinelegalizer class for SPIR-V.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===- SPIRVLegalizerInfo.cpp --- SPIR-V Legalization Rules ------*- C++ -*-==//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the targeting of the Machinelegalizer class for SPIR-V.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "SPIRVLegalizerInfo.h"
14: #include "SPIRV.h"
15: #include "SPIRVGlobalRegistry.h"
16: #include "SPIRVSubtarget.h"
17: #include "SPIRVUtils.h"
18: #include "llvm/CodeGen/GlobalISel/GenericMachineInstrs.h"
19: #include "llvm/CodeGen/GlobalISel/LegalizerHelper.h"
20: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
21: #include "llvm/CodeGen/MachineInstr.h"
22: #include "llvm/CodeGen/MachineRegisterInfo.h"
23: #include "llvm/CodeGen/TargetOpcodes.h"
24: #include "llvm/IR/IntrinsicsSPIRV.h"
25: #include "llvm/Support/Debug.h"
26: #include "llvm/Support/MathExtras.h"
27:
28: using namespace llvm;
29: using namespace llvm::LegalizeActions;
30: using namespace llvm::LegalityPredicates;
31:
32: #define DEBUG_TYPE "spirv-legalizer"
33:
34: LegalityPredicate typeOfExtendedScalars(unsigned TypeIdx, bool IsExtendedInts) {
35:   return [IsExtendedInts, TypeIdx](const LegalityQuery &Query) {
36:     const LLT Ty = Query.Types[TypeIdx];
37:     return IsExtendedInts && Ty.isValid() && Ty.isScalar();
38:   };
39: }
40:
41: SPIRVLegalizerInfo::SPIRVLegalizerInfo(const SPIRVSubtarget &ST) {
42:   using namespace TargetOpcode;
43:
44:   this->ST = &ST;
45:   GR = ST.getSPIRVGlobalRegistry();
46:
47:   const LLT s1 = LLT::scalar(1);
48:   const LLT s8 = LLT::scalar(8);
49:   const LLT s16 = LLT::scalar(16);
50:   const LLT s32 = LLT::scalar(32);
51:   const LLT s64 = LLT::scalar(64);
52:   const LLT s128 = LLT::scalar(128);
53:
54:   const LLT v16s64 = LLT::fixed_vector(16, 64);
55:   const LLT v16s32 = LLT::fixed_vector(16, 32);
56:   const LLT v16s16 = LLT::fixed_vector(16, 16);
57:   const LLT v16s8 = LLT::fixed_vector(16, 8);
58:   const LLT v16s1 = LLT::fixed_vector(16, 1);
59:
60:   const LLT v8s64 = LLT::fixed_vector(8, 64);
61:   const LLT v8s32 = LLT::fixed_vector(8, 32);
62:   const LLT v8s16 = LLT::fixed_vector(8, 16);
63:   const LLT v8s8 = LLT::fixed_vector(8, 8);
64:   const LLT v8s1 = LLT::fixed_vector(8, 1);
65:
66:   const LLT v4s64 = LLT::fixed_vector(4, 64);
67:   const LLT v4s32 = LLT::fixed_vector(4, 32);
68:   const LLT v4s16 = LLT::fixed_vector(4, 16);
69:   const LLT v4s8 = LLT::fixed_vector(4, 8);
70:   const LLT v4s1 = LLT::fixed_vector(4, 1);
71:
72:   const LLT v3s64 = LLT::fixed_vector(3, 64);
73:   const LLT v3s32 = LLT::fixed_vector(3, 32);
74:   const LLT v3s16 = LLT::fixed_vector(3, 16);
75:   const LLT v3s8 = LLT::fixed_vector(3, 8);
76:   const LLT v3s1 = LLT::fixed_vector(3, 1);
77:
78:   const LLT v2s64 = LLT::fixed_vector(2, 64);
79:   const LLT v2s32 = LLT::fixed_vector(2, 32);
80:   const LLT v2s16 = LLT::fixed_vector(2, 16);
81:   const LLT v2s8 = LLT::fixed_vector(2, 8);
82:   const LLT v2s1 = LLT::fixed_vector(2, 1);
83:
84:   const unsigned PSize = ST.getPointerSize();
85:   const LLT p0 = LLT::pointer(0, PSize); // Function
86:   const LLT p1 = LLT::pointer(1, PSize); // CrossWorkgroup
87:   const LLT p2 = LLT::pointer(2, PSize); // UniformConstant
88:   const LLT p3 = LLT::pointer(3, PSize); // Workgroup
89:   const LLT p4 = LLT::pointer(4, PSize); // Generic
90:   const LLT p5 =
```
- EN: This range implements operational logic in helpers such as typeOfExtendedScalars, isValid, SPIRVLegalizerInfo::SPIRVLegalizerInfo, getSPIRVGlobalRegistry, translating backend policy into executable code.
- CN: 这一段实现了 typeOfExtendedScalars、isValid、SPIRVLegalizerInfo::SPIRVLegalizerInfo、getSPIRVGlobalRegistry 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 91-180
```cpp
 91:       LLT::pointer(5, PSize); // Input, SPV_INTEL_usm_storage_classes (Device)
 92:   const LLT p6 = LLT::pointer(6, PSize); // SPV_INTEL_usm_storage_classes (Host)
 93:   const LLT p7 = LLT::pointer(7, PSize); // Input
 94:   const LLT p8 = LLT::pointer(8, PSize); // Output
 95:   const LLT p9 =
 96:       LLT::pointer(9, PSize); // CodeSectionINTEL, SPV_INTEL_function_pointers
 97:   const LLT p10 = LLT::pointer(10, PSize); // Private
 98:   const LLT p11 = LLT::pointer(11, PSize); // StorageBuffer
 99:   const LLT p12 = LLT::pointer(12, PSize); // Uniform
100:   const LLT p13 = LLT::pointer(13, PSize); // PushConstant
101:
102:   // TODO: remove copy-pasting here by using concatenation in some way.
103:   auto allPtrsScalarsAndVectors = {
104:       p0,    p1,    p2,    p3,    p4,    p5,     p6,     p7,    p8,
105:       p9,    p10,   p11,   p12,   p13,   s1,     s8,     s16,   s32,
106:       s64,   v2s1,  v2s8,  v2s16, v2s32, v2s64,  v3s1,   v3s8,  v3s16,
107:       v3s32, v3s64, v4s1,  v4s8,  v4s16, v4s32,  v4s64,  v8s1,  v8s8,
108:       v8s16, v8s32, v8s64, v16s1, v16s8, v16s16, v16s32, v16s64};
109:
110:   auto allVectors = {v2s1,  v2s8,   v2s16,  v2s32, v2s64, v3s1,  v3s8,
111:                      v3s16, v3s32,  v3s64,  v4s1,  v4s8,  v4s16, v4s32,
112:                      v4s64, v8s1,   v8s8,   v8s16, v8s32, v8s64, v16s1,
113:                      v16s8, v16s16, v16s32, v16s64};
114:
115:   auto allShaderVectors = {v2s1, v2s8, v2s16, v2s32, v2s64,
116:                            v3s1, v3s8, v3s16, v3s32, v3s64,
117:                            v4s1, v4s8, v4s16, v4s32, v4s64};
118:
119:   auto allScalars = {s1, s8, s16, s32, s64};
120:
121:   auto allScalarsAndVectors = {
122:       s1,    s8,    s16,   s32,   s64,    s128,   v2s1,  v2s8,
123:       v2s16, v2s32, v2s64, v3s1,  v3s8,   v3s16,  v3s32, v3s64,
124:       v4s1,  v4s8,  v4s16, v4s32, v4s64,  v8s1,   v8s8,  v8s16,
125:       v8s32, v8s64, v16s1, v16s8, v16s16, v16s32, v16s64};
126:
127:   auto allIntScalarsAndVectors = {
128:       s8,    s16,   s32,   s64,   s128,   v2s8,   v2s16, v2s32, v2s64,
129:       v3s8,  v3s16, v3s32, v3s64, v4s8,   v4s16,  v4s32, v4s64, v8s8,
130:       v8s16, v8s32, v8s64, v16s8, v16s16, v16s32, v16s64};
131:
132:   auto allBoolScalarsAndVectors = {s1, v2s1, v3s1, v4s1, v8s1, v16s1};
133:
134:   auto allIntScalars = {s8, s16, s32, s64, s128};
135:
136:   auto allFloatScalarsAndF16Vector2AndVector4s = {s16, s32, s64, v2s16, v4s16};
137:
138:   auto allFloatScalars = {s16, s32, s64};
139:
140:   auto allFloatScalarsAndVectors = {
141:       s16,   s32,   s64,   v2s16, v2s32, v2s64, v3s16,  v3s32,  v3s64,
142:       v4s16, v4s32, v4s64, v8s16, v8s32, v8s64, v16s16, v16s32, v16s64};
143:
144:   auto allShaderFloatVectors = {v2s16, v2s32, v2s64, v3s16, v3s32,
145:                                 v3s64, v4s16, v4s32, v4s64};
146:
147:   auto allFloatVectors = {v2s16, v2s32, v2s64,  v3s16,  v3s32,
148:                           v3s64, v4s16, v4s32,  v4s64,  v8s16,
149:                           v8s32, v8s64, v16s16, v16s32, v16s64};
150:
151:   auto &allowedFloatVectorTypes =
152:       ST.isShader() ? allShaderFloatVectors : allFloatVectors;
153:
154:   auto allFloatAndIntScalarsAndPtrs = {s8, s16, s32, s64, p0,  p1,
155:                                        p2, p3,  p4,  p5,  p6,  p7,
156:                                        p8, p9,  p10, p11, p12, p13};
157:
158:   auto allPtrs = {p0, p1, p2, p3, p4, p5, p6, p7, p8, p9, p10, p11, p12, p13};
159:
160:   auto &allowedVectorTypes = ST.isShader() ? allShaderVectors : allVectors;
161:
162:   bool IsExtendedInts =
163:       ST.canUseExtension(
164:           SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers) ||
165:       ST.canUseExtension(SPIRV::Extension::SPV_KHR_bit_instructions) ||
166:       ST.canUseExtension(SPIRV::Extension::SPV_INTEL_int4);
167:   auto extendedScalarsAndVectors =
168:       [IsExtendedInts](const LegalityQuery &Query) {
169:         const LLT Ty = Query.Types[0];
170:         return IsExtendedInts && Ty.isValid() && !Ty.isPointerOrPointerVector();
171:       };
172:   auto extendedScalarsAndVectorsProduct = [IsExtendedInts](
173:                                               const LegalityQuery &Query) {
174:     const LLT Ty1 = Query.Types[0], Ty2 = Query.Types[1];
175:     return IsExtendedInts && Ty1.isValid() && Ty2.isValid() &&
176:            !Ty1.isPointerOrPointerVector() && !Ty2.isPointerOrPointerVector();
177:   };
178:   auto extendedPtrsScalarsAndVectors =
179:       [IsExtendedInts](const LegalityQuery &Query) {
180:         const LLT Ty = Query.Types[0];
```
- EN: This range implements operational logic in helpers such as LLT::pointer, canUseExtension, isValid, isPointerOrPointerVector, translating backend policy into executable code.
- CN: 这一段实现了 LLT::pointer、canUseExtension、isValid、isPointerOrPointerVector 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:         return IsExtendedInts && Ty.isValid();
182:       };
183:
184:   // The universal validation rules in the SPIR-V specification state that
185:   // vector sizes are typically limited to 2, 3, or 4. However, larger vector
186:   // sizes (8 and 16) are enabled when the Kernel capability is present. For
187:   // shader execution models, vector sizes are strictly limited to 4. In
188:   // non-shader contexts, vector sizes of 8 and 16 are also permitted, but
189:   // arbitrary sizes (e.g., 6 or 11) are not.
190:   uint32_t MaxVectorSize = ST.isShader() ? 4 : 16;
191:   LLVM_DEBUG(dbgs() << "MaxVectorSize: " << MaxVectorSize << "\n");
192:
193:   for (auto Opc : getTypeFoldingSupportedOpcodes()) {
194:     switch (Opc) {
195:     case G_EXTRACT_VECTOR_ELT:
196:     case G_UREM:
197:     case G_SREM:
198:     case G_UDIV:
199:     case G_SDIV:
200:     case G_FREM:
201:       break;
202:     default:
203:       getActionDefinitionsBuilder(Opc)
204:           .customFor(allScalars)
205:           .customFor(allowedVectorTypes)
206:           .moreElementsToNextPow2(0)
207:           .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
208:                            LegalizeMutations::changeElementCountTo(
209:                                0, ElementCount::getFixed(MaxVectorSize)))
210:           .custom();
211:       break;
212:     }
213:   }
214:
215:   getActionDefinitionsBuilder({G_UREM, G_SREM, G_SDIV, G_UDIV, G_FREM})
216:       .customFor(allScalars)
217:       .customFor(allowedVectorTypes)
218:       .scalarizeIf(numElementsNotPow2(0), 0)
219:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
220:                        LegalizeMutations::changeElementCountTo(
221:                            0, ElementCount::getFixed(MaxVectorSize)))
222:       .custom();
223:
224:   getActionDefinitionsBuilder({G_FMA, G_STRICT_FMA})
225:       .legalFor(allScalars)
226:       .legalFor(allowedVectorTypes)
227:       .moreElementsToNextPow2(0)
228:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
229:                        LegalizeMutations::changeElementCountTo(
230:                            0, ElementCount::getFixed(MaxVectorSize)))
231:       .alwaysLegal();
232:
233:   getActionDefinitionsBuilder(G_INTRINSIC_W_SIDE_EFFECTS).custom();
234:
235:   getActionDefinitionsBuilder(G_SHUFFLE_VECTOR)
236:       .legalForCartesianProduct(allowedVectorTypes, allowedVectorTypes)
237:       .moreElementsToNextPow2(0)
238:       .lowerIf(vectorElementCountIsGreaterThan(0, MaxVectorSize))
239:       .moreElementsToNextPow2(1)
240:       .lowerIf(vectorElementCountIsGreaterThan(1, MaxVectorSize));
241:
242:   getActionDefinitionsBuilder(G_EXTRACT_VECTOR_ELT)
243:       .moreElementsToNextPow2(1)
244:       .fewerElementsIf(vectorElementCountIsGreaterThan(1, MaxVectorSize),
245:                        LegalizeMutations::changeElementCountTo(
246:                            1, ElementCount::getFixed(MaxVectorSize)))
247:       .custom();
248:
249:   getActionDefinitionsBuilder(G_INSERT_VECTOR_ELT)
250:       .moreElementsToNextPow2(0)
251:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
252:                        LegalizeMutations::changeElementCountTo(
253:                            0, ElementCount::getFixed(MaxVectorSize)))
254:       .custom();
255:
256:   // Illegal G_UNMERGE_VALUES instructions should be handled
257:   // during the combine phase.
258:   getActionDefinitionsBuilder(G_BUILD_VECTOR)
259:       .legalIf(vectorElementCountIsLessThanOrEqualTo(0, MaxVectorSize));
260:
261:   // When entering the legalizer, there should be no G_BITCAST instructions.
262:   // They should all be calls to the `spv_bitcast` intrinsic. The call to
263:   // the intrinsic will be converted to a G_BITCAST during legalization if
264:   // the vectors are not legal. After using the rules to legalize a G_BITCAST,
265:   // we turn it back into a call to the intrinsic with a custom rule to avoid
266:   // potential machine verifier failures.
267:   getActionDefinitionsBuilder(G_BITCAST)
268:       .moreElementsToNextPow2(0)
269:       .moreElementsToNextPow2(1)
270:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
```
- EN: This range implements operational logic in helpers such as isValid, LLVM_DEBUG, getActionDefinitionsBuilder, customFor, translating backend policy into executable code.
- CN: 这一段实现了 isValid、LLVM_DEBUG、getActionDefinitionsBuilder、customFor 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:                        LegalizeMutations::changeElementCountTo(
272:                            0, ElementCount::getFixed(MaxVectorSize)))
273:       .lowerIf(vectorElementCountIsGreaterThan(1, MaxVectorSize))
274:       .custom();
275:
276:   // If the result is still illegal, the combiner should be able to remove it.
277:   getActionDefinitionsBuilder(G_CONCAT_VECTORS)
278:       .legalForCartesianProduct(allowedVectorTypes, allowedVectorTypes);
279:
280:   getActionDefinitionsBuilder(G_SPLAT_VECTOR)
281:       .legalFor(allowedVectorTypes)
282:       .moreElementsToNextPow2(0)
283:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
284:                        LegalizeMutations::changeElementSizeTo(0, MaxVectorSize))
285:       .alwaysLegal();
286:
287:   // Vector Reduction Operations
288:   getActionDefinitionsBuilder(
289:       {G_VECREDUCE_SMIN, G_VECREDUCE_SMAX, G_VECREDUCE_UMIN, G_VECREDUCE_UMAX,
290:        G_VECREDUCE_ADD, G_VECREDUCE_MUL, G_VECREDUCE_FMUL, G_VECREDUCE_FMIN,
291:        G_VECREDUCE_FMAX, G_VECREDUCE_FMINIMUM, G_VECREDUCE_FMAXIMUM,
292:        G_VECREDUCE_OR, G_VECREDUCE_AND, G_VECREDUCE_XOR})
293:       .legalFor(allowedVectorTypes)
294:       .scalarize(1)
295:       .lower();
296:
297:   getActionDefinitionsBuilder({G_VECREDUCE_SEQ_FADD, G_VECREDUCE_SEQ_FMUL})
298:       .scalarize(2)
299:       .lower();
300:
301:   // Illegal G_UNMERGE_VALUES instructions should be handled
302:   // during the combine phase.
303:   getActionDefinitionsBuilder(G_UNMERGE_VALUES)
304:       .legalIf(vectorElementCountIsLessThanOrEqualTo(1, MaxVectorSize));
305:
306:   getActionDefinitionsBuilder({G_MEMCPY, G_MEMMOVE})
307:       .unsupportedIf(LegalityPredicates::any(typeIs(0, p9), typeIs(1, p9)))
308:       .legalIf(all(typeInSet(0, allPtrs), typeInSet(1, allPtrs)));
309:
310:   getActionDefinitionsBuilder(G_MEMSET)
311:       .unsupportedIf(typeIs(0, p9))
312:       .legalIf(all(typeInSet(0, allPtrs), typeInSet(1, allIntScalars)));
313:
314:   getActionDefinitionsBuilder(G_ADDRSPACE_CAST)
315:       .unsupportedIf(
316:           LegalityPredicates::any(all(typeIs(0, p9), typeIsNot(1, p9)),
317:                                   all(typeIsNot(0, p9), typeIs(1, p9))))
318:       .legalForCartesianProduct(allPtrs, allPtrs);
319:
320:   // Should we be legalizing bad scalar sizes like s5 here instead
321:   // of handling them in the instruction selector?
322:   getActionDefinitionsBuilder({G_LOAD, G_STORE})
323:       .unsupportedIf(typeIs(1, p9))
324:       .legalForCartesianProduct(allowedVectorTypes, allPtrs)
325:       .legalForCartesianProduct(allPtrs, allPtrs)
326:       .legalIf(isScalar(0))
327:       .custom();
328:
329:   getActionDefinitionsBuilder({G_SMIN, G_SMAX, G_UMIN, G_UMAX, G_ABS,
330:                                G_BITREVERSE, G_SADDSAT, G_UADDSAT, G_SSUBSAT,
331:                                G_USUBSAT, G_SCMP, G_UCMP})
332:       .legalFor(allIntScalarsAndVectors)
333:       .legalIf(extendedScalarsAndVectors);
334:
335:   getActionDefinitionsBuilder(G_STRICT_FLDEXP)
336:       .legalForCartesianProduct(allFloatScalarsAndVectors, allIntScalars);
337:
338:   getActionDefinitionsBuilder({G_FPTOSI, G_FPTOUI})
339:       .legalForCartesianProduct(allIntScalarsAndVectors,
340:                                 allFloatScalarsAndVectors);
341:
342:   getActionDefinitionsBuilder({G_FPTOSI_SAT, G_FPTOUI_SAT})
343:       .legalForCartesianProduct(allIntScalarsAndVectors,
344:                                 allFloatScalarsAndVectors);
345:
346:   getActionDefinitionsBuilder({G_SITOFP, G_UITOFP})
347:       .legalForCartesianProduct(allFloatScalarsAndVectors,
348:                                 allScalarsAndVectors);
349:
350:   getActionDefinitionsBuilder(G_CTPOP)
351:       .legalForCartesianProduct(allIntScalarsAndVectors)
352:       .legalIf(extendedScalarsAndVectorsProduct);
353:
354:   // Extensions.
355:   getActionDefinitionsBuilder({G_TRUNC, G_ZEXT, G_SEXT, G_ANYEXT})
356:       .legalForCartesianProduct(allScalarsAndVectors)
357:       .legalIf(extendedScalarsAndVectorsProduct);
358:
359:   getActionDefinitionsBuilder(G_PHI)
360:       .legalFor(allPtrsScalarsAndVectors)
```
- EN: This range implements operational logic in helpers such as ElementCount::getFixed, lowerIf, custom, getActionDefinitionsBuilder, translating backend policy into executable code.
- CN: 这一段实现了 ElementCount::getFixed、lowerIf、custom、getActionDefinitionsBuilder 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:       .legalIf(extendedPtrsScalarsAndVectors);
362:
363:   getActionDefinitionsBuilder(G_BITCAST).legalIf(
364:       all(typeInSet(0, allPtrsScalarsAndVectors),
365:           typeInSet(1, allPtrsScalarsAndVectors)));
366:
367:   getActionDefinitionsBuilder({G_IMPLICIT_DEF, G_FREEZE})
368:       .legalFor({s1, s128})
369:       .legalFor(allFloatAndIntScalarsAndPtrs)
370:       .legalFor(allowedVectorTypes)
371:       .legalIf([](const LegalityQuery &Query) {
372:         return Query.Types[0].isPointerVector();
373:       })
374:       .moreElementsToNextPow2(0)
375:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
376:                        LegalizeMutations::changeElementCountTo(
377:                            0, ElementCount::getFixed(MaxVectorSize)));
378:
379:   getActionDefinitionsBuilder({G_STACKSAVE, G_STACKRESTORE}).alwaysLegal();
380:
381:   getActionDefinitionsBuilder(G_INTTOPTR)
382:       .legalForCartesianProduct(allPtrs, allIntScalars)
383:       .legalIf(
384:           all(typeInSet(0, allPtrs), typeOfExtendedScalars(1, IsExtendedInts)))
385:       .legalIf([](const LegalityQuery &Query) {
386:         const LLT DstTy = Query.Types[0];
387:         const LLT SrcTy = Query.Types[1];
388:         return DstTy.isPointerVector() && SrcTy.isVector() &&
389:                !SrcTy.isPointer() &&
390:                DstTy.getNumElements() == SrcTy.getNumElements();
391:       });
392:   getActionDefinitionsBuilder(G_PTRTOINT)
393:       .legalForCartesianProduct(allIntScalars, allPtrs)
394:       .legalIf(
395:           all(typeOfExtendedScalars(0, IsExtendedInts), typeInSet(1, allPtrs)))
396:       .legalIf([](const LegalityQuery &Query) {
397:         const LLT DstTy = Query.Types[0];
398:         const LLT SrcTy = Query.Types[1];
399:         return SrcTy.isPointerVector() && DstTy.isVector() &&
400:                !DstTy.isPointer() &&
401:                DstTy.getNumElements() == SrcTy.getNumElements();
402:       });
403:   getActionDefinitionsBuilder(G_PTR_ADD)
404:       .legalForCartesianProduct(allPtrs, allIntScalars)
405:       .legalIf(
406:           all(typeInSet(0, allPtrs), typeOfExtendedScalars(1, IsExtendedInts)));
407:
408:   // ST.canDirectlyComparePointers() for pointer args is supported in
409:   // legalizeCustom().
410:   getActionDefinitionsBuilder(G_ICMP)
411:       .unsupportedIf(LegalityPredicates::any(
412:           all(typeIs(0, p9), typeInSet(1, allPtrs), typeIsNot(1, p9)),
413:           all(typeInSet(0, allPtrs), typeIsNot(0, p9), typeIs(1, p9))))
414:       .legalIf([IsExtendedInts](const LegalityQuery &Query) {
415:         const LLT Ty = Query.Types[1];
416:         return IsExtendedInts && Ty.isValid() && !Ty.isPointerOrPointerVector();
417:       })
418:       .customIf(all(typeInSet(0, allBoolScalarsAndVectors),
419:                     typeInSet(1, allPtrsScalarsAndVectors)));
420:
421:   getActionDefinitionsBuilder(G_FCMP).legalIf(
422:       all(typeInSet(0, allBoolScalarsAndVectors),
423:           typeInSet(1, allFloatScalarsAndVectors)));
424:
425:   getActionDefinitionsBuilder({G_ATOMICRMW_OR, G_ATOMICRMW_ADD, G_ATOMICRMW_AND,
426:                                G_ATOMICRMW_MAX, G_ATOMICRMW_MIN,
427:                                G_ATOMICRMW_SUB, G_ATOMICRMW_XOR,
428:                                G_ATOMICRMW_UMAX, G_ATOMICRMW_UMIN})
429:       .legalForCartesianProduct(allIntScalars, allPtrs);
430:
431:   getActionDefinitionsBuilder(
432:       {G_ATOMICRMW_FADD, G_ATOMICRMW_FSUB, G_ATOMICRMW_FMIN, G_ATOMICRMW_FMAX})
433:       .legalForCartesianProduct(allFloatScalarsAndF16Vector2AndVector4s,
434:                                 allPtrs);
435:
436:   getActionDefinitionsBuilder(G_ATOMICRMW_XCHG)
437:       .legalForCartesianProduct(allFloatAndIntScalarsAndPtrs, allPtrs);
438:
439:   getActionDefinitionsBuilder(G_ATOMIC_CMPXCHG_WITH_SUCCESS).lower();
440:   // TODO: add proper legalization rules.
441:   getActionDefinitionsBuilder(G_ATOMIC_CMPXCHG).alwaysLegal();
442:
443:   getActionDefinitionsBuilder({G_UADDO, G_USUBO, G_UMULO, G_SMULO})
444:       .alwaysLegal();
445:
446:   getActionDefinitionsBuilder({G_SADDO, G_SSUBO}).lower();
447:
448:   getActionDefinitionsBuilder({G_LROUND, G_LLROUND})
449:       .legalForCartesianProduct(allFloatScalarsAndVectors,
450:                                 allIntScalarsAndVectors);
```
- EN: This range implements operational logic in helpers such as legalIf, typeInSet, getActionDefinitionsBuilder, legalFor, translating backend policy into executable code.
- CN: 这一段实现了 legalIf、typeInSet、getActionDefinitionsBuilder、legalFor 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:
452:   // FP conversions.
453:   getActionDefinitionsBuilder({G_FPTRUNC, G_FPEXT})
454:       .legalForCartesianProduct(allFloatScalarsAndVectors);
455:
456:   // Pointer-handling.
457:   getActionDefinitionsBuilder(G_FRAME_INDEX).legalFor({p0});
458:
459:   getActionDefinitionsBuilder(G_GLOBAL_VALUE).legalFor(allPtrs);
460:
461:   // Control-flow. In some cases (e.g. constants) s1 may be promoted to s32.
462:   getActionDefinitionsBuilder(G_BR).alwaysLegal();
463:   getActionDefinitionsBuilder(G_BRCOND).legalFor({s1, s32});
464:
465:   getActionDefinitionsBuilder(G_FFREXP).legalForCartesianProduct(
466:       allFloatScalarsAndVectors, {s32, v2s32, v3s32, v4s32, v8s32, v16s32});
467:
468:   // TODO: Review the target OpenCL and GLSL Extended Instruction Set specs to
469:   // tighten these requirements. Many of these math functions are only legal on
470:   // specific bitwidths, so they are not selectable for
471:   // allFloatScalarsAndVectors.
472:   // clang-format off
473:   getActionDefinitionsBuilder({G_STRICT_FSQRT,
474:                                G_FPOW,
475:                                G_FEXP,
476:                                G_FMODF,
477:                                G_FSINCOS,
478:                                G_FEXP2,
479:                                G_FEXP10,
480:                                G_FLOG,
481:                                G_FLOG2,
482:                                G_FLOG10,
483:                                G_FABS,
484:                                G_FMINNUM,
485:                                G_FMAXNUM,
486:                                G_FCEIL,
487:                                G_FCOS,
488:                                G_FSIN,
489:                                G_FTAN,
490:                                G_FACOS,
491:                                G_FASIN,
492:                                G_FATAN,
493:                                G_FATAN2,
494:                                G_FCOSH,
495:                                G_FSINH,
496:                                G_FTANH,
497:                                G_FSQRT,
498:                                G_FFLOOR,
499:                                G_FRINT,
500:                                G_FNEARBYINT,
501:                                G_INTRINSIC_ROUND,
502:                                G_INTRINSIC_TRUNC,
503:                                G_FMINIMUM,
504:                                G_FMAXIMUM,
505:                                G_INTRINSIC_ROUNDEVEN})
506:       .legalFor(allFloatScalars)
507:       .legalFor(allowedFloatVectorTypes)
508:       .moreElementsToNextPow2(0)
509:       .fewerElementsIf(vectorElementCountIsGreaterThan(0, MaxVectorSize),
510:                        LegalizeMutations::changeElementCountTo(
511:                            0, ElementCount::getFixed(MaxVectorSize)));
512:   // clang-format on
513:
514:   getActionDefinitionsBuilder(G_FCOPYSIGN)
515:       .legalForCartesianProduct(allFloatScalarsAndVectors,
516:                                 allFloatScalarsAndVectors);
517:
518:   getActionDefinitionsBuilder(G_FPOWI).legalForCartesianProduct(
519:       allFloatScalarsAndVectors, allIntScalarsAndVectors);
520:
521:   if (ST.canUseExtInstSet(SPIRV::InstructionSet::OpenCL_std)) {
522:     getActionDefinitionsBuilder(
523:         {G_CTTZ, G_CTTZ_ZERO_POISON, G_CTLZ, G_CTLZ_ZERO_POISON})
524:         .legalForCartesianProduct(allIntScalarsAndVectors,
525:                                   allIntScalarsAndVectors);
526:
527:     // Struct return types become a single scalar, so cannot easily legalize.
528:     getActionDefinitionsBuilder({G_SMULH, G_UMULH}).alwaysLegal();
529:   }
530:
531:   getActionDefinitionsBuilder(G_IS_FPCLASS).custom();
532:
533:   getActionDefinitionsBuilder({G_INTRINSIC, G_INTRINSIC_CONVERGENT,
534:                                G_INTRINSIC_CONVERGENT_W_SIDE_EFFECTS})
535:       .alwaysLegal();
536:   getActionDefinitionsBuilder(G_FENCE).alwaysLegal();
537:   getActionDefinitionsBuilder({G_TRAP, G_DEBUGTRAP, G_UBSANTRAP}).alwaysLegal();
538:
539:   getLegacyLegalizerInfo().computeTables();
540:   verify(*ST.getInstrInfo());
```
- EN: This range implements operational logic in helpers such as getActionDefinitionsBuilder, legalForCartesianProduct, legalFor, moreElementsToNextPow2, translating backend policy into executable code.
- CN: 这一段实现了 getActionDefinitionsBuilder、legalForCartesianProduct、legalFor、moreElementsToNextPow2 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541: }
542:
543: static bool legalizeExtractVectorElt(LegalizerHelper &Helper, MachineInstr &MI,
544:                                      SPIRVGlobalRegistry *GR) {
545:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
546:   Register DstReg = MI.getOperand(0).getReg();
547:   Register SrcReg = MI.getOperand(1).getReg();
548:   Register IdxReg = MI.getOperand(2).getReg();
549:
550:   MIRBuilder
551:       .buildIntrinsic(Intrinsic::spv_extractelt, ArrayRef<Register>{DstReg})
552:       .addUse(SrcReg)
553:       .addUse(IdxReg);
554:   MI.eraseFromParent();
555:   return true;
556: }
557:
558: static bool legalizeInsertVectorElt(LegalizerHelper &Helper, MachineInstr &MI,
559:                                     SPIRVGlobalRegistry *GR) {
560:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
561:   Register DstReg = MI.getOperand(0).getReg();
562:   Register SrcReg = MI.getOperand(1).getReg();
563:   Register ValReg = MI.getOperand(2).getReg();
564:   Register IdxReg = MI.getOperand(3).getReg();
565:
566:   MIRBuilder
567:       .buildIntrinsic(Intrinsic::spv_insertelt, ArrayRef<Register>{DstReg})
568:       .addUse(SrcReg)
569:       .addUse(ValReg)
570:       .addUse(IdxReg);
571:   MI.eraseFromParent();
572:   return true;
573: }
574:
575: static Register convertPtrToInt(Register Reg, LLT ConvTy, SPIRVTypeInst SpvType,
576:                                 LegalizerHelper &Helper,
577:                                 MachineRegisterInfo &MRI,
578:                                 SPIRVGlobalRegistry *GR) {
579:   Register ConvReg = MRI.createGenericVirtualRegister(ConvTy);
580:   MRI.setRegClass(ConvReg, GR->getRegClass(SpvType));
581:   GR->assignSPIRVTypeToVReg(SpvType, ConvReg, Helper.MIRBuilder.getMF());
582:   Helper.MIRBuilder.buildInstr(TargetOpcode::G_PTRTOINT)
583:       .addDef(ConvReg)
584:       .addUse(Reg);
585:   return ConvReg;
586: }
587:
588: static bool needsVectorLegalization(const LLT &Ty, const SPIRVSubtarget &ST) {
589:   if (!Ty.isVector())
590:     return false;
591:   unsigned NumElements = Ty.getNumElements();
592:   unsigned MaxVectorSize = ST.isShader() ? 4 : 16;
593:   return (NumElements > 4 && !isPowerOf2_32(NumElements)) ||
594:          NumElements > MaxVectorSize;
595: }
596:
597: static bool legalizeLoad(LegalizerHelper &Helper, MachineInstr &MI,
598:                          SPIRVGlobalRegistry *GR) {
599:   MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
600:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
601:   Register DstReg = MI.getOperand(0).getReg();
602:   Register PtrReg = MI.getOperand(1).getReg();
603:   LLT DstTy = MRI.getType(DstReg);
604:
605:   if (!DstTy.isVector())
606:     return true;
607:
608:   const SPIRVSubtarget &ST = MI.getMF()->getSubtarget<SPIRVSubtarget>();
609:   if (!needsVectorLegalization(DstTy, ST))
610:     return true;
611:
612:   SmallVector<Register, 8> SplitRegs;
613:   LLT EltTy = DstTy.getElementType();
614:   unsigned NumElts = DstTy.getNumElements();
615:
616:   LLT PtrTy = MRI.getType(PtrReg);
617:   auto Zero = MIRBuilder.buildConstant(LLT::scalar(32), 0);
618:
619:   for (unsigned i = 0; i < NumElts; ++i) {
620:     auto Idx = MIRBuilder.buildConstant(LLT::scalar(32), i);
621:     Register EltPtr = MRI.createGenericVirtualRegister(PtrTy);
622:
623:     MIRBuilder.buildIntrinsic(Intrinsic::spv_gep, ArrayRef<Register>{EltPtr})
624:         .addImm(1) // InBounds
625:         .addUse(PtrReg)
626:         .addUse(Zero.getReg(0))
627:         .addUse(Idx.getReg(0));
628:
629:     MachinePointerInfo EltPtrInfo;
630:     Align EltAlign = Align(1);
```
- EN: This range implements operational logic in helpers such as getOperand, buildIntrinsic, addUse, eraseFromParent, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、buildIntrinsic、addUse、eraseFromParent 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:     if (!MI.memoperands_empty()) {
632:       MachineMemOperand *MMO = *MI.memoperands_begin();
633:       EltPtrInfo =
634:           MMO->getPointerInfo().getWithOffset(i * EltTy.getSizeInBytes());
635:       EltAlign = commonAlignment(MMO->getAlign(), i * EltTy.getSizeInBytes());
636:     }
637:
638:     Register EltReg = MRI.createGenericVirtualRegister(EltTy);
639:     MIRBuilder.buildLoad(EltReg, EltPtr, EltPtrInfo, EltAlign);
640:     SplitRegs.push_back(EltReg);
641:   }
642:
643:   MIRBuilder.buildBuildVector(DstReg, SplitRegs);
644:   MI.eraseFromParent();
645:   return true;
646: }
647:
648: static bool legalizeStore(LegalizerHelper &Helper, MachineInstr &MI,
649:                           SPIRVGlobalRegistry *GR) {
650:   MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
651:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
652:   Register ValReg = MI.getOperand(0).getReg();
653:   Register PtrReg = MI.getOperand(1).getReg();
654:   LLT ValTy = MRI.getType(ValReg);
655:
656:   assert(ValTy.isVector() && "Expected vector store");
657:
658:   SmallVector<Register, 8> SplitRegs;
659:   LLT EltTy = ValTy.getElementType();
660:   unsigned NumElts = ValTy.getNumElements();
661:
662:   for (unsigned i = 0; i < NumElts; ++i)
663:     SplitRegs.push_back(MRI.createGenericVirtualRegister(EltTy));
664:
665:   MIRBuilder.buildUnmerge(SplitRegs, ValReg);
666:
667:   LLT PtrTy = MRI.getType(PtrReg);
668:   auto Zero = MIRBuilder.buildConstant(LLT::scalar(32), 0);
669:
670:   for (unsigned i = 0; i < NumElts; ++i) {
671:     auto Idx = MIRBuilder.buildConstant(LLT::scalar(32), i);
672:     Register EltPtr = MRI.createGenericVirtualRegister(PtrTy);
673:
674:     MIRBuilder.buildIntrinsic(Intrinsic::spv_gep, ArrayRef<Register>{EltPtr})
675:         .addImm(1) // InBounds
676:         .addUse(PtrReg)
677:         .addUse(Zero.getReg(0))
678:         .addUse(Idx.getReg(0));
679:
680:     MachinePointerInfo EltPtrInfo;
681:     Align EltAlign = Align(1);
682:     if (!MI.memoperands_empty()) {
683:       MachineMemOperand *MMO = *MI.memoperands_begin();
684:       EltPtrInfo =
685:           MMO->getPointerInfo().getWithOffset(i * EltTy.getSizeInBytes());
686:       EltAlign = commonAlignment(MMO->getAlign(), i * EltTy.getSizeInBytes());
687:     }
688:
689:     MIRBuilder.buildStore(SplitRegs[i], EltPtr, EltPtrInfo, EltAlign);
690:   }
691:
692:   MI.eraseFromParent();
693:   return true;
694: }
695:
696: bool SPIRVLegalizerInfo::legalizeCustom(
697:     LegalizerHelper &Helper, MachineInstr &MI,
698:     LostDebugLocObserver &LocObserver) const {
699:   MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
700:   switch (MI.getOpcode()) {
701:   default:
702:     // TODO: implement legalization for other opcodes.
703:     return true;
704:   case TargetOpcode::G_BITCAST:
705:     return legalizeBitcast(Helper, MI);
706:   case TargetOpcode::G_EXTRACT_VECTOR_ELT:
707:     return legalizeExtractVectorElt(Helper, MI, GR);
708:   case TargetOpcode::G_INSERT_VECTOR_ELT:
709:     return legalizeInsertVectorElt(Helper, MI, GR);
710:   case TargetOpcode::G_INTRINSIC:
711:   case TargetOpcode::G_INTRINSIC_W_SIDE_EFFECTS:
712:     return legalizeIntrinsic(Helper, MI);
713:   case TargetOpcode::G_IS_FPCLASS:
714:     return legalizeIsFPClass(Helper, MI, LocObserver);
715:   case TargetOpcode::G_ICMP: {
716:     auto &Op0 = MI.getOperand(2);
717:     auto &Op1 = MI.getOperand(3);
718:     Register Reg0 = Op0.getReg();
719:     Register Reg1 = Op1.getReg();
720:     CmpInst::Predicate Cond =
```
- EN: This range implements operational logic in helpers such as memoperands_begin, getPointerInfo, commonAlignment, createGenericVirtualRegister, translating backend policy into executable code.
- CN: 这一段实现了 memoperands_begin、getPointerInfo、commonAlignment、createGenericVirtualRegister 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:         static_cast<CmpInst::Predicate>(MI.getOperand(1).getPredicate());
722:     if ((!ST->canDirectlyComparePointers() ||
723:          (Cond != CmpInst::ICMP_EQ && Cond != CmpInst::ICMP_NE)) &&
724:         MRI.getType(Reg0).isPointer() && MRI.getType(Reg1).isPointer()) {
725:       LLT ConvT = LLT::scalar(ST->getPointerSize());
726:       Type *LLVMTy = IntegerType::get(MI.getMF()->getFunction().getContext(),
727:                                       ST->getPointerSize());
728:       SPIRVTypeInst SpirvTy = GR->getOrCreateSPIRVType(
729:           LLVMTy, Helper.MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
730:       Op0.setReg(convertPtrToInt(Reg0, ConvT, SpirvTy, Helper, MRI, GR));
731:       Op1.setReg(convertPtrToInt(Reg1, ConvT, SpirvTy, Helper, MRI, GR));
732:     }
733:     return true;
734:   }
735:   case TargetOpcode::G_LOAD:
736:     return legalizeLoad(Helper, MI, GR);
737:   case TargetOpcode::G_STORE:
738:     return legalizeStore(Helper, MI, GR);
739:   }
740: }
741:
742: static MachineInstrBuilder
743: createStackTemporaryForVector(LegalizerHelper &Helper, SPIRVGlobalRegistry *GR,
744:                               Register SrcReg, LLT SrcTy,
745:                               MachinePointerInfo &PtrInfo, Align &VecAlign) {
746:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
747:   MachineRegisterInfo &MRI = *MIRBuilder.getMRI();
748:
749:   VecAlign = Helper.getStackTemporaryAlignment(SrcTy);
750:   auto StackTemp = Helper.createStackTemporary(
751:       TypeSize::getFixed(SrcTy.getSizeInBytes()), VecAlign, PtrInfo);
752:
753:   // Set the type of StackTemp to a pointer to an array of the element type.
754:   SPIRVTypeInst SpvSrcTy = GR->getSPIRVTypeForVReg(SrcReg);
755:   SPIRVTypeInst EltSpvTy = GR->getScalarOrVectorComponentType(SpvSrcTy);
756:   const Type *LLVMEltTy = GR->getTypeForSPIRVType(EltSpvTy);
757:   const Type *LLVMArrTy =
758:       ArrayType::get(const_cast<Type *>(LLVMEltTy), SrcTy.getNumElements());
759:   SPIRVTypeInst ArrSpvTy = GR->getOrCreateSPIRVType(
760:       LLVMArrTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite, true);
761:   SPIRVTypeInst PtrToArrSpvTy = GR->getOrCreateSPIRVPointerType(
762:       ArrSpvTy, MIRBuilder, SPIRV::StorageClass::Function);
763:
764:   Register StackReg = StackTemp.getReg(0);
765:   MRI.setRegClass(StackReg, GR->getRegClass(PtrToArrSpvTy));
766:   GR->assignSPIRVTypeToVReg(PtrToArrSpvTy, StackReg, MIRBuilder.getMF());
767:
768:   return StackTemp;
769: }
770:
771: static bool legalizeSpvBitcast(LegalizerHelper &Helper, MachineInstr &MI,
772:                                SPIRVGlobalRegistry *GR) {
773:   LLVM_DEBUG(dbgs() << "Found a bitcast instruction\n");
774:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
775:   MachineRegisterInfo &MRI = *MIRBuilder.getMRI();
776:   const SPIRVSubtarget &ST = MI.getMF()->getSubtarget<SPIRVSubtarget>();
777:
778:   Register DstReg = MI.getOperand(0).getReg();
779:   Register SrcReg = MI.getOperand(2).getReg();
780:   LLT DstTy = MRI.getType(DstReg);
781:   LLT SrcTy = MRI.getType(SrcReg);
782:
783:   // If an spv_bitcast needs to be legalized, we convert it to G_BITCAST to
784:   // allow using the generic legalization rules.
785:   if (needsVectorLegalization(DstTy, ST) ||
786:       needsVectorLegalization(SrcTy, ST)) {
787:     LLVM_DEBUG(dbgs() << "Replacing with a G_BITCAST\n");
788:     MIRBuilder.buildBitcast(DstReg, SrcReg);
789:     MI.eraseFromParent();
790:   }
791:   return true;
792: }
793:
794: static bool legalizeSpvInsertElt(LegalizerHelper &Helper, MachineInstr &MI,
795:                                  SPIRVGlobalRegistry *GR) {
796:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
797:   MachineRegisterInfo &MRI = *MIRBuilder.getMRI();
798:   const SPIRVSubtarget &ST = MI.getMF()->getSubtarget<SPIRVSubtarget>();
799:
800:   Register DstReg = MI.getOperand(0).getReg();
801:   LLT DstTy = MRI.getType(DstReg);
802:
803:   if (needsVectorLegalization(DstTy, ST)) {
804:     Register SrcReg = MI.getOperand(2).getReg();
805:     Register ValReg = MI.getOperand(3).getReg();
806:     LLT SrcTy = MRI.getType(SrcReg);
807:     MachineOperand &IdxOperand = MI.getOperand(4);
808:
809:     if (getImm(IdxOperand, &MRI)) {
810:       uint64_t IdxVal = foldImm(IdxOperand, &MRI);
```
- EN: This range implements operational logic in helpers such as getOperand, getType, LLT::scalar, getPointerSize, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getType、LLT::scalar、getPointerSize 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:       if (IdxVal < SrcTy.getNumElements()) {
812:         SmallVector<Register, 8> Regs;
813:         SPIRVTypeInst ElementType =
814:             GR->getScalarOrVectorComponentType(GR->getSPIRVTypeForVReg(DstReg));
815:         LLT ElementLLTTy = GR->getRegType(ElementType);
816:         for (unsigned I = 0, E = SrcTy.getNumElements(); I < E; ++I) {
817:           Register Reg = MRI.createGenericVirtualRegister(ElementLLTTy);
818:           MRI.setRegClass(Reg, GR->getRegClass(ElementType));
819:           GR->assignSPIRVTypeToVReg(ElementType, Reg, *MI.getMF());
820:           Regs.push_back(Reg);
821:         }
822:         MIRBuilder.buildUnmerge(Regs, SrcReg);
823:         Regs[IdxVal] = ValReg;
824:         MIRBuilder.buildBuildVector(DstReg, Regs);
825:         MI.eraseFromParent();
826:         return true;
827:       }
828:     }
829:
830:     LLT EltTy = SrcTy.getElementType();
831:     Align VecAlign;
832:     MachinePointerInfo PtrInfo;
833:     auto StackTemp = createStackTemporaryForVector(Helper, GR, SrcReg, SrcTy,
834:                                                    PtrInfo, VecAlign);
835:
836:     MIRBuilder.buildStore(SrcReg, StackTemp, PtrInfo, VecAlign);
837:
838:     Register IdxReg = IdxOperand.getReg();
839:     LLT PtrTy = MRI.getType(StackTemp.getReg(0));
840:     Register EltPtr = MRI.createGenericVirtualRegister(PtrTy);
841:     auto Zero = MIRBuilder.buildConstant(LLT::scalar(32), 0);
842:
843:     MIRBuilder.buildIntrinsic(Intrinsic::spv_gep, ArrayRef<Register>{EltPtr})
844:         .addImm(1) // InBounds
845:         .addUse(StackTemp.getReg(0))
846:         .addUse(Zero.getReg(0))
847:         .addUse(IdxReg);
848:
849:     MachinePointerInfo EltPtrInfo = MachinePointerInfo(PtrTy.getAddressSpace());
850:     Align EltAlign = Helper.getStackTemporaryAlignment(EltTy);
851:     MIRBuilder.buildStore(ValReg, EltPtr, EltPtrInfo, EltAlign);
852:
853:     MIRBuilder.buildLoad(DstReg, StackTemp, PtrInfo, VecAlign);
854:     MI.eraseFromParent();
855:     return true;
856:   }
857:   return true;
858: }
859:
860: static bool legalizeSpvExtractElt(LegalizerHelper &Helper, MachineInstr &MI,
861:                                   SPIRVGlobalRegistry *GR) {
862:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
863:   MachineRegisterInfo &MRI = *MIRBuilder.getMRI();
864:   const SPIRVSubtarget &ST = MI.getMF()->getSubtarget<SPIRVSubtarget>();
865:
866:   Register SrcReg = MI.getOperand(2).getReg();
867:   LLT SrcTy = MRI.getType(SrcReg);
868:
869:   if (needsVectorLegalization(SrcTy, ST)) {
870:     Register DstReg = MI.getOperand(0).getReg();
871:     MachineOperand &IdxOperand = MI.getOperand(3);
872:
873:     if (getImm(IdxOperand, &MRI)) {
874:       uint64_t IdxVal = foldImm(IdxOperand, &MRI);
875:       if (IdxVal < SrcTy.getNumElements()) {
876:         LLT DstTy = MRI.getType(DstReg);
877:         SmallVector<Register, 8> Regs;
878:         SPIRVTypeInst DstSpvTy = GR->getSPIRVTypeForVReg(DstReg);
879:         for (unsigned I = 0, E = SrcTy.getNumElements(); I < E; ++I) {
880:           if (I == IdxVal) {
881:             Regs.push_back(DstReg);
882:           } else {
883:             Register Reg = MRI.createGenericVirtualRegister(DstTy);
884:             MRI.setRegClass(Reg, GR->getRegClass(DstSpvTy));
885:             GR->assignSPIRVTypeToVReg(DstSpvTy, Reg, *MI.getMF());
886:             Regs.push_back(Reg);
887:           }
888:         }
889:         MIRBuilder.buildUnmerge(Regs, SrcReg);
890:         MI.eraseFromParent();
891:         return true;
892:       }
893:     }
894:
895:     LLT EltTy = SrcTy.getElementType();
896:     Align VecAlign;
897:     MachinePointerInfo PtrInfo;
898:     auto StackTemp = createStackTemporaryForVector(Helper, GR, SrcReg, SrcTy,
899:                                                    PtrInfo, VecAlign);
900:
```
- EN: This range implements operational logic in helpers such as getScalarOrVectorComponentType, getRegType, createGenericVirtualRegister, setRegClass, translating backend policy into executable code.
- CN: 这一段实现了 getScalarOrVectorComponentType、getRegType、createGenericVirtualRegister、setRegClass 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:     MIRBuilder.buildStore(SrcReg, StackTemp, PtrInfo, VecAlign);
902:
903:     Register IdxReg = IdxOperand.getReg();
904:     LLT PtrTy = MRI.getType(StackTemp.getReg(0));
905:     Register EltPtr = MRI.createGenericVirtualRegister(PtrTy);
906:     auto Zero = MIRBuilder.buildConstant(LLT::scalar(32), 0);
907:
908:     MIRBuilder.buildIntrinsic(Intrinsic::spv_gep, ArrayRef<Register>{EltPtr})
909:         .addImm(1) // InBounds
910:         .addUse(StackTemp.getReg(0))
911:         .addUse(Zero.getReg(0))
912:         .addUse(IdxReg);
913:
914:     MachinePointerInfo EltPtrInfo = MachinePointerInfo(PtrTy.getAddressSpace());
915:     Align EltAlign = Helper.getStackTemporaryAlignment(EltTy);
916:     MIRBuilder.buildLoad(DstReg, EltPtr, EltPtrInfo, EltAlign);
917:
918:     MI.eraseFromParent();
919:     return true;
920:   }
921:   return true;
922: }
923:
924: static bool legalizeSpvConstComposite(LegalizerHelper &Helper, MachineInstr &MI,
925:                                       SPIRVGlobalRegistry *GR) {
926:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
927:   MachineRegisterInfo &MRI = *MIRBuilder.getMRI();
928:   const SPIRVSubtarget &ST = MI.getMF()->getSubtarget<SPIRVSubtarget>();
929:
930:   Register DstReg = MI.getOperand(0).getReg();
931:   LLT DstTy = MRI.getType(DstReg);
932:
933:   if (!needsVectorLegalization(DstTy, ST))
934:     return true;
935:
936:   SmallVector<Register, 8> SrcRegs;
937:   if (MI.getNumOperands() == 2) {
938:     // The "null" case: no values are attached.
939:     LLT EltTy = DstTy.getElementType();
940:     auto Zero = MIRBuilder.buildConstant(EltTy, 0);
941:     SPIRVTypeInst SpvDstTy = GR->getSPIRVTypeForVReg(DstReg);
942:     SPIRVTypeInst SpvEltTy = GR->getScalarOrVectorComponentType(SpvDstTy);
943:     GR->assignSPIRVTypeToVReg(SpvEltTy, Zero.getReg(0), MIRBuilder.getMF());
944:     for (unsigned i = 0; i < DstTy.getNumElements(); ++i)
945:       SrcRegs.push_back(Zero.getReg(0));
946:   } else {
947:     for (unsigned i = 2; i < MI.getNumOperands(); ++i) {
948:       SrcRegs.push_back(MI.getOperand(i).getReg());
949:     }
950:   }
951:   MIRBuilder.buildBuildVector(DstReg, SrcRegs);
952:   MI.eraseFromParent();
953:   return true;
954: }
955:
956: bool SPIRVLegalizerInfo::legalizeIntrinsic(LegalizerHelper &Helper,
957:                                            MachineInstr &MI) const {
958:   LLVM_DEBUG(dbgs() << "legalizeIntrinsic: " << MI);
959:   auto IntrinsicID = cast<GIntrinsic>(MI).getIntrinsicID();
960:   switch (IntrinsicID) {
961:   case Intrinsic::spv_bitcast:
962:     return legalizeSpvBitcast(Helper, MI, GR);
963:   case Intrinsic::spv_insertelt:
964:     return legalizeSpvInsertElt(Helper, MI, GR);
965:   case Intrinsic::spv_extractelt:
966:     return legalizeSpvExtractElt(Helper, MI, GR);
967:   case Intrinsic::spv_const_composite:
968:     return legalizeSpvConstComposite(Helper, MI, GR);
969:   }
970:   return true;
971: }
972:
973: bool SPIRVLegalizerInfo::legalizeBitcast(LegalizerHelper &Helper,
974:                                          MachineInstr &MI) const {
975:   // Once the G_BITCAST is using vectors that are allowed, we turn it back into
976:   // an spv_bitcast to avoid verifier problems when the register types are the
977:   // same for the source and the result. Note that the SPIR-V types associated
978:   // with the bitcast can be different even if the register types are the same.
979:   MachineIRBuilder &MIRBuilder = Helper.MIRBuilder;
980:   Register DstReg = MI.getOperand(0).getReg();
981:   Register SrcReg = MI.getOperand(1).getReg();
982:   SmallVector<Register, 1> DstRegs = {DstReg};
983:   MIRBuilder.buildIntrinsic(Intrinsic::spv_bitcast, DstRegs).addUse(SrcReg);
984:   MI.eraseFromParent();
985:   return true;
986: }
987:
988: // Note this code was copied from LegalizerHelper::lowerISFPCLASS and adjusted
989: // to ensure that all instructions created during the lowering have SPIR-V types
990: // assigned to them.
```
- EN: This range implements operational logic in helpers such as buildStore, getReg, getType, createGenericVirtualRegister, translating backend policy into executable code.
- CN: 这一段实现了 buildStore、getReg、getType、createGenericVirtualRegister 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991: bool SPIRVLegalizerInfo::legalizeIsFPClass(
 992:     LegalizerHelper &Helper, MachineInstr &MI,
 993:     LostDebugLocObserver &LocObserver) const {
 994:   auto [DstReg, DstTy, SrcReg, SrcTy] = MI.getFirst2RegLLTs();
 995:   FPClassTest Mask = static_cast<FPClassTest>(MI.getOperand(2).getImm());
 996:
 997:   auto &MIRBuilder = Helper.MIRBuilder;
 998:   auto &MF = MIRBuilder.getMF();
 999:   MachineRegisterInfo &MRI = MF.getRegInfo();
1000:
1001:   Type *LLVMDstTy =
1002:       IntegerType::get(MIRBuilder.getContext(), DstTy.getScalarSizeInBits());
1003:   if (DstTy.isVector())
1004:     LLVMDstTy = VectorType::get(LLVMDstTy, DstTy.getElementCount());
1005:   SPIRVTypeInst SPIRVDstTy = GR->getOrCreateSPIRVType(
1006:       LLVMDstTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite,
1007:       /*EmitIR*/ true);
1008:
1009:   unsigned BitSize = SrcTy.getScalarSizeInBits();
1010:   const fltSemantics &Semantics = getFltSemanticForLLT(SrcTy.getScalarType());
1011:
1012:   LLT IntTy = LLT::scalar(BitSize);
1013:   Type *LLVMIntTy = IntegerType::get(MIRBuilder.getContext(), BitSize);
1014:   if (SrcTy.isVector()) {
1015:     IntTy = LLT::vector(SrcTy.getElementCount(), IntTy);
1016:     LLVMIntTy = VectorType::get(LLVMIntTy, SrcTy.getElementCount());
1017:   }
1018:   SPIRVTypeInst SPIRVIntTy = GR->getOrCreateSPIRVType(
1019:       LLVMIntTy, MIRBuilder, SPIRV::AccessQualifier::ReadWrite,
1020:       /*EmitIR*/ true);
1021:
1022:   // Clang doesn't support capture of structured bindings:
1023:   LLT DstTyCopy = DstTy;
1024:   const auto assignSPIRVTy = [&](MachineInstrBuilder &&MI) {
1025:     // Assign this MI's (assumed only) destination to one of the two types we
1026:     // expect: either the G_IS_FPCLASS's destination type, or the integer type
1027:     // bitcast from the source type.
1028:     LLT MITy = MRI.getType(MI.getReg(0));
1029:     assert((MITy == IntTy || MITy == DstTyCopy) &&
1030:            "Unexpected LLT type while lowering G_IS_FPCLASS");
1031:     SPIRVTypeInst SPVTy = MITy == IntTy ? SPIRVIntTy : SPIRVDstTy;
1032:     GR->assignSPIRVTypeToVReg(SPVTy, MI.getReg(0), MF);
1033:     return MI;
1034:   };
1035:
1036:   // Helper to build and assign a constant in one go
1037:   const auto buildSPIRVConstant = [&](LLT Ty, auto &&C) -> MachineInstrBuilder {
1038:     if (!Ty.isFixedVector())
1039:       return assignSPIRVTy(MIRBuilder.buildConstant(Ty, C));
1040:     auto ScalarC = MIRBuilder.buildConstant(Ty.getScalarType(), C);
1041:     assert((Ty == IntTy || Ty == DstTyCopy) &&
1042:            "Unexpected LLT type while lowering constant for G_IS_FPCLASS");
1043:     SPIRVTypeInst VecEltTy = GR->getOrCreateSPIRVType(
1044:         (Ty == IntTy ? LLVMIntTy : LLVMDstTy)->getScalarType(), MIRBuilder,
1045:         SPIRV::AccessQualifier::ReadWrite,
1046:         /*EmitIR*/ true);
1047:     GR->assignSPIRVTypeToVReg(VecEltTy, ScalarC.getReg(0), MF);
1048:     return assignSPIRVTy(MIRBuilder.buildSplatBuildVector(Ty, ScalarC));
1049:   };
1050:
1051:   if (Mask == fcNone) {
1052:     MIRBuilder.buildCopy(DstReg, buildSPIRVConstant(DstTy, 0));
1053:     MI.eraseFromParent();
1054:     return true;
1055:   }
1056:   if (Mask == fcAllFlags) {
1057:     MIRBuilder.buildCopy(DstReg, buildSPIRVConstant(DstTy, 1));
1058:     MI.eraseFromParent();
1059:     return true;
1060:   }
1061:
1062:   // Note that rather than creating a COPY here (between a floating-point and
1063:   // integer type of the same size) we create a SPIR-V bitcast immediately. We
1064:   // can't create a G_BITCAST because the LLTs are the same, and we can't seem
1065:   // to correctly lower COPYs to SPIR-V bitcasts at this moment.
1066:   Register ResVReg = MRI.createGenericVirtualRegister(IntTy);
1067:   MRI.setRegClass(ResVReg, GR->getRegClass(SPIRVIntTy));
1068:   GR->assignSPIRVTypeToVReg(SPIRVIntTy, ResVReg, Helper.MIRBuilder.getMF());
1069:   auto AsInt = MIRBuilder.buildInstr(SPIRV::OpBitcast)
1070:                    .addDef(ResVReg)
1071:                    .addUse(GR->getSPIRVTypeID(SPIRVIntTy))
1072:                    .addUse(SrcReg);
1073:   AsInt = assignSPIRVTy(std::move(AsInt));
1074:
1075:   // Various masks.
1076:   APInt SignBit = APInt::getSignMask(BitSize);
1077:   APInt ValueMask = APInt::getSignedMaxValue(BitSize);     // All bits but sign.
1078:   APInt Inf = APFloat::getInf(Semantics).bitcastToAPInt(); // Exp and int bit.
1079:   APInt ExpMask = Inf;
1080:   APInt AllOneMantissa = APFloat::getLargest(Semantics).bitcastToAPInt() & ~Inf;
```
- EN: This range implements operational logic in helpers such as getFirst2RegLLTs, getOperand, getMF, getRegInfo, translating backend policy into executable code.
- CN: 这一段实现了 getFirst2RegLLTs、getOperand、getMF、getRegInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:   APInt QNaNBitMask =
1082:       APInt::getOneBitSet(BitSize, AllOneMantissa.getActiveBits() - 1);
1083:   APInt InversionMask = APInt::getAllOnes(DstTy.getScalarSizeInBits());
1084:
1085:   auto SignBitC = buildSPIRVConstant(IntTy, SignBit);
1086:   auto ValueMaskC = buildSPIRVConstant(IntTy, ValueMask);
1087:   auto InfC = buildSPIRVConstant(IntTy, Inf);
1088:   auto ExpMaskC = buildSPIRVConstant(IntTy, ExpMask);
1089:   auto ZeroC = buildSPIRVConstant(IntTy, 0);
1090:
1091:   auto Abs = assignSPIRVTy(MIRBuilder.buildAnd(IntTy, AsInt, ValueMaskC));
1092:   auto Sign = assignSPIRVTy(
1093:       MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_NE, DstTy, AsInt, Abs));
1094:
1095:   auto Res = buildSPIRVConstant(DstTy, 0);
1096:
1097:   const auto appendToRes = [&](MachineInstrBuilder &&ToAppend) {
1098:     Res = assignSPIRVTy(
1099:         MIRBuilder.buildOr(DstTyCopy, Res, assignSPIRVTy(std::move(ToAppend))));
1100:   };
1101:
1102:   // Tests that involve more than one class should be processed first.
1103:   if ((Mask & fcFinite) == fcFinite) {
1104:     // finite(V) ==> abs(V) u< exp_mask
1105:     appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_ULT, DstTy, Abs,
1106:                                      ExpMaskC));
1107:     Mask &= ~fcFinite;
1108:   } else if ((Mask & fcFinite) == fcPosFinite) {
1109:     // finite(V) && V > 0 ==> V u< exp_mask
1110:     appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_ULT, DstTy, AsInt,
1111:                                      ExpMaskC));
1112:     Mask &= ~fcPosFinite;
1113:   } else if ((Mask & fcFinite) == fcNegFinite) {
1114:     // finite(V) && V < 0 ==> abs(V) u< exp_mask && signbit == 1
1115:     auto Cmp = assignSPIRVTy(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_ULT,
1116:                                                   DstTy, Abs, ExpMaskC));
1117:     appendToRes(MIRBuilder.buildAnd(DstTy, Cmp, Sign));
1118:     Mask &= ~fcNegFinite;
1119:   }
1120:
1121:   if (FPClassTest PartialCheck = Mask & (fcZero | fcSubnormal)) {
1122:     // fcZero | fcSubnormal => test all exponent bits are 0
1123:     // TODO: Handle sign bit specific cases
1124:     // TODO: Handle inverted case
1125:     if (PartialCheck == (fcZero | fcSubnormal)) {
1126:       auto ExpBits = assignSPIRVTy(MIRBuilder.buildAnd(IntTy, AsInt, ExpMaskC));
1127:       appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy,
1128:                                        ExpBits, ZeroC));
1129:       Mask &= ~PartialCheck;
1130:     }
1131:   }
1132:
1133:   // Check for individual classes.
1134:   if (FPClassTest PartialCheck = Mask & fcZero) {
1135:     if (PartialCheck == fcPosZero)
1136:       appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy,
1137:                                        AsInt, ZeroC));
1138:     else if (PartialCheck == fcZero)
1139:       appendToRes(
1140:           MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy, Abs, ZeroC));
1141:     else // fcNegZero
1142:       appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy,
1143:                                        AsInt, SignBitC));
1144:   }
1145:
1146:   if (FPClassTest PartialCheck = Mask & fcSubnormal) {
1147:     // issubnormal(V) ==> unsigned(abs(V) - 1) u< (all mantissa bits set)
1148:     // issubnormal(V) && V>0 ==> unsigned(V - 1) u< (all mantissa bits set)
1149:     auto V = (PartialCheck == fcPosSubnormal) ? AsInt : Abs;
1150:     auto OneC = buildSPIRVConstant(IntTy, 1);
1151:     auto VMinusOne = MIRBuilder.buildSub(IntTy, V, OneC);
1152:     auto SubnormalRes = assignSPIRVTy(
1153:         MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_ULT, DstTy, VMinusOne,
1154:                              buildSPIRVConstant(IntTy, AllOneMantissa)));
1155:     if (PartialCheck == fcNegSubnormal)
1156:       SubnormalRes = MIRBuilder.buildAnd(DstTy, SubnormalRes, Sign);
1157:     appendToRes(std::move(SubnormalRes));
1158:   }
1159:
1160:   if (FPClassTest PartialCheck = Mask & fcInf) {
1161:     if (PartialCheck == fcPosInf)
1162:       appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy,
1163:                                        AsInt, InfC));
1164:     else if (PartialCheck == fcInf)
1165:       appendToRes(
1166:           MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy, Abs, InfC));
1167:     else { // fcNegInf
1168:       APInt NegInf = APFloat::getInf(Semantics, true).bitcastToAPInt();
1169:       auto NegInfC = buildSPIRVConstant(IntTy, NegInf);
1170:       appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_EQ, DstTy,
```
- EN: This range implements operational logic in helpers such as APInt::getOneBitSet, APInt::getAllOnes, buildSPIRVConstant, assignSPIRVTy, translating backend policy into executable code.
- CN: 这一段实现了 APInt::getOneBitSet、APInt::getAllOnes、buildSPIRVConstant、assignSPIRVTy 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1220
```cpp
1171:                                        AsInt, NegInfC));
1172:     }
1173:   }
1174:
1175:   if (FPClassTest PartialCheck = Mask & fcNan) {
1176:     auto InfWithQnanBitC =
1177:         buildSPIRVConstant(IntTy, std::move(Inf) | QNaNBitMask);
1178:     if (PartialCheck == fcNan) {
1179:       // isnan(V) ==> abs(V) u> int(inf)
1180:       appendToRes(
1181:           MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_UGT, DstTy, Abs, InfC));
1182:     } else if (PartialCheck == fcQNan) {
1183:       // isquiet(V) ==> abs(V) u>= (unsigned(Inf) | quiet_bit)
1184:       appendToRes(MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_UGE, DstTy, Abs,
1185:                                        InfWithQnanBitC));
1186:     } else { // fcSNan
1187:       // issignaling(V) ==> abs(V) u> unsigned(Inf) &&
1188:       //                    abs(V) u< (unsigned(Inf) | quiet_bit)
1189:       auto IsNan = assignSPIRVTy(
1190:           MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_UGT, DstTy, Abs, InfC));
1191:       auto IsNotQnan = assignSPIRVTy(MIRBuilder.buildICmp(
1192:           CmpInst::Predicate::ICMP_ULT, DstTy, Abs, InfWithQnanBitC));
1193:       appendToRes(MIRBuilder.buildAnd(DstTy, IsNan, IsNotQnan));
1194:     }
1195:   }
1196:
1197:   if (FPClassTest PartialCheck = Mask & fcNormal) {
1198:     // isnormal(V) ==> (0 u< exp u< max_exp) ==> (unsigned(exp-1) u<
1199:     // (max_exp-1))
1200:     APInt ExpLSB = ExpMask & ~(ExpMask.shl(1));
1201:     auto ExpMinusOne = assignSPIRVTy(
1202:         MIRBuilder.buildSub(IntTy, Abs, buildSPIRVConstant(IntTy, ExpLSB)));
1203:     APInt MaxExpMinusOne = std::move(ExpMask) - ExpLSB;
1204:     auto NormalRes = assignSPIRVTy(
1205:         MIRBuilder.buildICmp(CmpInst::Predicate::ICMP_ULT, DstTy, ExpMinusOne,
1206:                              buildSPIRVConstant(IntTy, MaxExpMinusOne)));
1207:     if (PartialCheck == fcNegNormal)
1208:       NormalRes = MIRBuilder.buildAnd(DstTy, NormalRes, Sign);
1209:     else if (PartialCheck == fcPosNormal) {
1210:       auto PosSign = assignSPIRVTy(MIRBuilder.buildXor(
1211:           DstTy, Sign, buildSPIRVConstant(DstTy, InversionMask)));
1212:       NormalRes = MIRBuilder.buildAnd(DstTy, NormalRes, PosSign);
1213:     }
1214:     appendToRes(std::move(NormalRes));
1215:   }
1216:
1217:   MIRBuilder.buildCopy(DstReg, Res);
1218:   MI.eraseFromParent();
1219:   return true;
1220: }
```
- EN: This range implements operational logic in helpers such as buildSPIRVConstant, buildICmp, appendToRes, ~, translating backend policy into executable code.
- CN: 这一段实现了 buildSPIRVConstant、buildICmp、appendToRes、~ 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Legalization logic rewrites unsupported operations into forms that the target can handle.
  - CN: 合法化逻辑会把目标不支持的操作重写为可处理的形式。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include typeOfExtendedScalars, isValid, SPIRVLegalizerInfo::SPIRVLegalizerInfo, getSPIRVGlobalRegistry, LLT::scalar, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 typeOfExtendedScalars, isValid, SPIRVLegalizerInfo::SPIRVLegalizerInfo, getSPIRVGlobalRegistry, LLT::scalar，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVLegalizerInfo.h`
  - `SPIRV.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/CodeGen/GlobalISel/GenericMachineInstrs.h`
  - `llvm/CodeGen/GlobalISel/LegalizerHelper.h`
  - `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
  - `llvm/CodeGen/MachineInstr.h`
  - `llvm/CodeGen/MachineRegisterInfo.h`
  - `llvm/CodeGen/TargetOpcodes.h`
  - `llvm/IR/IntrinsicsSPIRV.h`
  - `llvm/Support/Debug.h`
  - `llvm/Support/MathExtras.h`
