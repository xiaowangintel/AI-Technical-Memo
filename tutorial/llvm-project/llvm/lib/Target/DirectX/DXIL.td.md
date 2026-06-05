# DXIL.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXIL.td`
- Repository: `llvm-project`
- Purpose (EN): This is a target description file for DXIL operations.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```tablegen
 1: //- DXIL.td - Describe DXIL operation -------------------------*- tablegen -*-//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file
10: /// This is a target description file for DXIL operations.
11: ///
12: //===----------------------------------------------------------------------===//
13:
14: include "llvm/IR/Intrinsics.td"
15:
16: // Abstract class to represent major and minor version values
17: class Version<int major, int minor> {
18:   int Major = major;
19:   int Minor = minor;
20: }
21:
22: // Valid DXIL Version records
23: foreach i = 0...8 in {
24:   def DXIL1_ #i : Version<1, i>;
25: }
26:
27: class DXILOpParamType {
28:   int isOverload = 0;
29: }
30:
31: let isOverload = 1 in {
32:   def OverloadTy : DXILOpParamType;
33: }
34: def VoidTy : DXILOpParamType;
35: def Int1Ty : DXILOpParamType;
36: def Int8Ty : DXILOpParamType;
37: def Int16Ty : DXILOpParamType;
38: def Int32Ty : DXILOpParamType;
39: def Int64Ty : DXILOpParamType;
40: def HalfTy : DXILOpParamType;
41: def FloatTy : DXILOpParamType;
42: def DoubleTy : DXILOpParamType;
43: def ResRetHalfTy : DXILOpParamType;
44: def ResRetFloatTy : DXILOpParamType;
45: def ResRetDoubleTy : DXILOpParamType;
46: def ResRetInt16Ty : DXILOpParamType;
47: def ResRetInt32Ty : DXILOpParamType;
48: def ResRetInt64Ty : DXILOpParamType;
49: def CBufRetHalfTy : DXILOpParamType;
50: def CBufRetFloatTy : DXILOpParamType;
51: def CBufRetDoubleTy : DXILOpParamType;
52: def CBufRetInt16Ty : DXILOpParamType;
53: def CBufRetInt32Ty : DXILOpParamType;
54: def CBufRetInt64Ty : DXILOpParamType;
55: def HandleTy : DXILOpParamType;
56: def ResBindTy : DXILOpParamType;
57: def ResPropsTy : DXILOpParamType;
58: def SplitDoubleTy : DXILOpParamType;
59: def BinaryWithCarryTy : DXILOpParamType;
60: def DimensionsTy : DXILOpParamType;
61: def Fouri32s : DXILOpParamType;
62:
63: class DXILOpClass;
64:
65: defset list<DXILOpClass> OpClasses = {
66:   def acceptHitAndEndSearch : DXILOpClass;
67:   def allocateNodeOutputRecords : DXILOpClass;
68:   def allocateRayQuery : DXILOpClass;
69:   def annotateHandle : DXILOpClass;
70:   def annotateNodeHandle : DXILOpClass;
71:   def annotateNodeRecordHandle : DXILOpClass;
72:   def atomicBinOp : DXILOpClass;
73:   def atomicCompareExchange : DXILOpClass;
74:   def attributeAtVertex : DXILOpClass;
75:   def barrier : DXILOpClass;
76:   def barrierByMemoryHandle : DXILOpClass;
77:   def barrierByMemoryType : DXILOpClass;
78:   def barrierByNodeRecordHandle : DXILOpClass;
79:   def binary : DXILOpClass;
80:   def binaryWithCarryOrBorrow : DXILOpClass;
81:   def binaryWithTwoOuts : DXILOpClass;
82:   def bitcastF16toI16 : DXILOpClass;
83:   def bitcastF32toI32 : DXILOpClass;
84:   def bitcastF64toI64 : DXILOpClass;
85:   def bitcastI16toF16 : DXILOpClass;
86:   def bitcastI32toF32 : DXILOpClass;
87:   def bitcastI64toF64 : DXILOpClass;
88:   def bufferLoad : DXILOpClass;
89:   def bufferStore : DXILOpClass;
90:   def bufferUpdateCounter : DXILOpClass;
```
- EN: This range uses TableGen DSL to describe records such as Version, DXIL1_, DXILOpParamType, OverloadTy; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Version、DXIL1_、DXILOpParamType、OverloadTy 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 91-180
```tablegen
 91:   def calculateLOD : DXILOpClass;
 92:   def callShader : DXILOpClass;
 93:   def cbufferLoad : DXILOpClass;
 94:   def cbufferLoadLegacy : DXILOpClass;
 95:   def checkAccessFullyMapped : DXILOpClass;
 96:   def coverage : DXILOpClass;
 97:   def createHandle : DXILOpClass;
 98:   def createHandleForLib : DXILOpClass;
 99:   def createHandleFromBinding : DXILOpClass;
100:   def createHandleFromHeap : DXILOpClass;
101:   def createNodeInputRecordHandle : DXILOpClass;
102:   def createNodeOutputHandle : DXILOpClass;
103:   def cutStream : DXILOpClass;
104:   def cycleCounterLegacy : DXILOpClass;
105:   def discard : DXILOpClass;
106:   def dispatchMesh : DXILOpClass;
107:   def dispatchRaysDimensions : DXILOpClass;
108:   def dispatchRaysIndex : DXILOpClass;
109:   def domainLocation : DXILOpClass;
110:   def dot2 : DXILOpClass;
111:   def dot2AddHalf : DXILOpClass;
112:   def dot3 : DXILOpClass;
113:   def dot4 : DXILOpClass;
114:   def dot4AddPacked : DXILOpClass;
115:   def emitIndices : DXILOpClass;
116:   def emitStream : DXILOpClass;
117:   def emitThenCutStream : DXILOpClass;
118:   def evalCentroid : DXILOpClass;
119:   def evalSampleIndex : DXILOpClass;
120:   def evalSnapped : DXILOpClass;
121:   def finishedCrossGroupSharing : DXILOpClass;
122:   def flattenedThreadIdInGroup : DXILOpClass;
123:   def geometryIndex : DXILOpClass;
124:   def getDimensions : DXILOpClass;
125:   def getInputRecordCount : DXILOpClass;
126:   def getMeshPayload : DXILOpClass;
127:   def getNodeRecordPtr : DXILOpClass;
128:   def getRemainingRecursionLevels : DXILOpClass;
129:   def groupId : DXILOpClass;
130:   def gsInstanceID : DXILOpClass;
131:   def hitKind : DXILOpClass;
132:   def ignoreHit : DXILOpClass;
133:   def incrementOutputCount : DXILOpClass;
134:   def indexNodeHandle : DXILOpClass;
135:   def innerCoverage : DXILOpClass;
136:   def instanceID : DXILOpClass;
137:   def instanceIndex : DXILOpClass;
138:   def isHelperLane : DXILOpClass;
139:   def isSpecialFloat : DXILOpClass;
140:   def legacyDoubleToFloat : DXILOpClass;
141:   def legacyDoubleToSInt32 : DXILOpClass;
142:   def legacyDoubleToUInt32 : DXILOpClass;
143:   def legacyF16ToF32 : DXILOpClass;
144:   def legacyF32ToF16 : DXILOpClass;
145:   def loadInput : DXILOpClass;
146:   def loadOutputControlPoint : DXILOpClass;
147:   def loadPatchConstant : DXILOpClass;
148:   def makeDouble : DXILOpClass;
149:   def minPrecXRegLoad : DXILOpClass;
150:   def minPrecXRegStore : DXILOpClass;
151:   def nodeOutputIsValid : DXILOpClass;
152:   def objectRayDirection : DXILOpClass;
153:   def objectRayOrigin : DXILOpClass;
154:   def objectToWorld : DXILOpClass;
155:   def outputComplete : DXILOpClass;
156:   def outputControlPointID : DXILOpClass;
157:   def pack4x8 : DXILOpClass;
158:   def primitiveID : DXILOpClass;
159:   def primitiveIndex : DXILOpClass;
160:   def quadOp : DXILOpClass;
161:   def quadReadLaneAt : DXILOpClass;
162:   def quadVote : DXILOpClass;
163:   def quaternary : DXILOpClass;
164:   def rawBufferLoad : DXILOpClass;
165:   def rawBufferStore : DXILOpClass;
166:   def rayFlags : DXILOpClass;
167:   def rayQuery_Abort : DXILOpClass;
168:   def rayQuery_CommitNonOpaqueTriangleHit : DXILOpClass;
169:   def rayQuery_CommitProceduralPrimitiveHit : DXILOpClass;
170:   def rayQuery_Proceed : DXILOpClass;
171:   def rayQuery_StateMatrix : DXILOpClass;
172:   def rayQuery_StateScalar : DXILOpClass;
173:   def rayQuery_StateVector : DXILOpClass;
174:   def rayQuery_TraceRayInline : DXILOpClass;
175:   def rayTCurrent : DXILOpClass;
176:   def rayTMin : DXILOpClass;
177:   def renderTargetGetSampleCount : DXILOpClass;
178:   def renderTargetGetSamplePosition : DXILOpClass;
179:   def reportHit : DXILOpClass;
180:   def sample : DXILOpClass;
```
- EN: This range uses TableGen DSL to describe records such as calculateLOD, callShader, cbufferLoad, cbufferLoadLegacy; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 calculateLOD、callShader、cbufferLoad、cbufferLoadLegacy 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 181-270
```tablegen
181:   def sampleBias : DXILOpClass;
182:   def sampleCmp : DXILOpClass;
183:   def sampleCmpBias : DXILOpClass;
184:   def sampleCmpGrad : DXILOpClass;
185:   def sampleCmpLevel : DXILOpClass;
186:   def sampleCmpLevelZero : DXILOpClass;
187:   def sampleGrad : DXILOpClass;
188:   def sampleIndex : DXILOpClass;
189:   def sampleLevel : DXILOpClass;
190:   def setMeshOutputCounts : DXILOpClass;
191:   def splitDouble : DXILOpClass;
192:   def startInstanceLocation : DXILOpClass;
193:   def startVertexLocation : DXILOpClass;
194:   def storeOutput : DXILOpClass;
195:   def storePatchConstant : DXILOpClass;
196:   def storePrimitiveOutput : DXILOpClass;
197:   def storeVertexOutput : DXILOpClass;
198:   def tempRegLoad : DXILOpClass;
199:   def tempRegStore : DXILOpClass;
200:   def tertiary : DXILOpClass;
201:   def texture2DMSGetSamplePosition : DXILOpClass;
202:   def textureGather : DXILOpClass;
203:   def textureGatherCmp : DXILOpClass;
204:   def textureGatherRaw : DXILOpClass;
205:   def textureLoad : DXILOpClass;
206:   def textureStore : DXILOpClass;
207:   def textureStoreSample : DXILOpClass;
208:   def threadId : DXILOpClass;
209:   def threadIdInGroup : DXILOpClass;
210:   def traceRay : DXILOpClass;
211:   def unary : DXILOpClass;
212:   def unaryBits : DXILOpClass;
213:   def unpack4x8 : DXILOpClass;
214:   def viewID : DXILOpClass;
215:   def waveActiveAllEqual : DXILOpClass;
216:   def waveActiveBit : DXILOpClass;
217:   def waveActiveOp : DXILOpClass;
218:   def waveAllOp : DXILOpClass;
219:   def waveAllTrue : DXILOpClass;
220:   def waveAnyTrue : DXILOpClass;
221:   def waveActiveBallot : DXILOpClass;
222:   def waveGetLaneCount : DXILOpClass;
223:   def waveGetLaneIndex : DXILOpClass;
224:   def waveIsFirstLane : DXILOpClass;
225:   def waveMatch : DXILOpClass;
226:   def waveMatrix_Accumulate : DXILOpClass;
227:   def waveMatrix_Annotate : DXILOpClass;
228:   def waveMatrix_Depth : DXILOpClass;
229:   def waveMatrix_Fill : DXILOpClass;
230:   def waveMatrix_LoadGroupShared : DXILOpClass;
231:   def waveMatrix_LoadRawBuf : DXILOpClass;
232:   def waveMatrix_Multiply : DXILOpClass;
233:   def waveMatrix_ScalarOp : DXILOpClass;
234:   def waveMatrix_StoreGroupShared : DXILOpClass;
235:   def waveMatrix_StoreRawBuf : DXILOpClass;
236:   def waveMultiPrefixBitCount : DXILOpClass;
237:   def waveMultiPrefixOp : DXILOpClass;
238:   def wavePrefixOp : DXILOpClass;
239:   def waveReadLaneAt : DXILOpClass;
240:   def waveReadLaneFirst : DXILOpClass;
241:   def worldRayDirection : DXILOpClass;
242:   def worldRayOrigin : DXILOpClass;
243:   def worldToObject : DXILOpClass;
244:   def writeSamplerFeedback : DXILOpClass;
245:   def writeSamplerFeedbackBias : DXILOpClass;
246:   def writeSamplerFeedbackGrad : DXILOpClass;
247:   def writeSamplerFeedbackLevel : DXILOpClass;
248:
249:   // This is a sentinel definition. Hence placed at the end here and
250:   // not as part of the above alphabetically sorted valid definitions.
251:   // It is never used to construct the name of DXIL Op call name.
252:   // Additionally it is capitalized unlike all the others.
253:   def UnknownOpClass : DXILOpClass;
254: }
255:
256: class DXILShaderStage;
257:
258: def compute : DXILShaderStage;
259: def domain : DXILShaderStage;
260: def hull : DXILShaderStage;
261: def pixel : DXILShaderStage;
262: def vertex : DXILShaderStage;
263: def geometry : DXILShaderStage;
264: def library : DXILShaderStage;
265: def amplification : DXILShaderStage;
266: def mesh : DXILShaderStage;
267: def node : DXILShaderStage;
268: def raygeneration : DXILShaderStage;
269: def intersection : DXILShaderStage;
270: def anyhit : DXILShaderStage;
```
- EN: This range uses TableGen DSL to describe records such as sampleBias, sampleCmp, sampleCmpBias, sampleCmpGrad; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 sampleBias、sampleCmp、sampleCmpBias、sampleCmpGrad 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 271-360
```tablegen
271: def closesthit : DXILShaderStage;
272: def callable : DXILShaderStage;
273: def miss : DXILShaderStage;
274:
275: // Pseudo-stages
276: // Denote DXIL Op to be supported in all stages
277: def all_stages : DXILShaderStage;
278: // Denote support for DXIL Op to have been removed
279: def removed : DXILShaderStage;
280:
281: // DXIL Op attributes
282:
283: // A function attribute denotes that there is a corresponding LLVM function
284: // attribute that will be set when building the DXIL op. The mapping is defined
285: // by setDXILAttributes in DXILOpBuilder.cpp
286: class DXILAttribute;
287:
288: def ReadNone : DXILAttribute;
289: def ReadOnly : DXILAttribute;
290: def NoDuplicate : DXILAttribute;
291: def NoReturn : DXILAttribute;
292:
293: class Overloads<Version ver, list<DXILOpParamType> ols> {
294:   Version dxil_version = ver;
295:   list<DXILOpParamType> overload_types = ols;
296: }
297:
298: class Stages<Version ver, list<DXILShaderStage> st> {
299:   Version dxil_version = ver;
300:   list<DXILShaderStage> shader_stages = st;
301: }
302:
303: class Attributes<Version ver = DXIL1_0, list<DXILAttribute> attrs> {
304:   Version dxil_version = ver;
305:   list<DXILAttribute> fn_attrs = attrs;
306: }
307:
308: defvar BarrierMode_DeviceMemoryBarrier = 2;
309: defvar BarrierMode_DeviceMemoryBarrierWithGroupSync = 3;
310: defvar BarrierMode_GroupMemoryBarrier = 8;
311: defvar BarrierMode_GroupMemoryBarrierWithGroupSync = 9;
312: defvar BarrierMode_AllMemoryBarrier = 10;
313: defvar BarrierMode_AllMemoryBarrierWithGroupSync = 11;
314:
315: defvar WaveOpKind_Sum = 0;
316: defvar WaveOpKind_Product = 1;
317: defvar WaveOpKind_Min = 2;
318: defvar WaveOpKind_Max = 3;
319:
320: defvar WaveBitOpKind_And = 0;
321: defvar WaveBitOpKind_Or = 1;
322: defvar WaveBitOpKind_Xor = 2;
323:
324: defvar SignedOpKind_Signed = 0;
325: defvar SignedOpKind_Unsigned = 1;
326:
327: defvar QuadOpKind_ReadAcrossX = 0;
328: defvar QuadOpKind_ReadAcrossY = 1;
329: defvar QuadOpKind_ReadAcrossDiagonal = 2;
330:
331: // Intrinsic arg selection
332: class IntrinArgSelectType;
333: def IntrinArgSelect_Index : IntrinArgSelectType;
334: def IntrinArgSelect_I8 : IntrinArgSelectType;
335: def IntrinArgSelect_I32 : IntrinArgSelectType;
336:
337: class IntrinArgSelect<IntrinArgSelectType type_, int value_> {
338:   IntrinArgSelectType type = type_;
339:   int value = value_;
340: }
341:
342: class IntrinArgIndex<int index> : IntrinArgSelect<IntrinArgSelect_Index, index>;
343: class IntrinArgI8<int value> : IntrinArgSelect<IntrinArgSelect_I8, value>;
344: class IntrinArgI32<int value> : IntrinArgSelect<IntrinArgSelect_I32, value>;
345:
346: // Select which intrinsic to lower from for a DXILOp.
347: // If the intrinsic is the only argument given to IntrinSelect, then the
348: // arguments of the intrinsic will be copied in the same order. Example:
349: //   let intrinsics = [
350: //     IntrinSelect<int_dx_my_intrinsic>,
351: //     IntrinSelect<int_dx_my_intrinsic2>,
352: //   ]
353: //=========================================================================================
354: // Using IntrinArgIndex<>, arguments of the intrinsic can be copied to the DXIL
355: // OP in specific order:
356: //   let intrinsics = [
357: //     IntrinSelect<int_dx_my_intrinsic,
358: //       [IntrinArgIndex<2>, IntrinArgIndex<1>, IntrinArgIndex<0>> ]
359: //     >,
360: //   ]
```
- EN: This range uses TableGen DSL to describe records such as closesthit, callable, miss, all_stages; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 closesthit、callable、miss、all_stages 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 361-450
```tablegen
361: //=========================================================================================
362: // Using IntrinArgI8<> and IntrinArgI32<>, integer constants can be added
363: // directly to the dxil op. This can be used in conjunction with
364: // IntrinArgIndex:
365: //   let intrinsics = [
366: //     IntrinSelect<int_dx_wave_active_usum,
367: //       [ IntrinArgIndex<0>, IntrinArgI8<0>, IntrinArgI8<1> ]
368: //     >,
369: //     IntrinSelect<int_dx_wave_reduce_sum,
370: //       [ IntrinArgIndex<0>, IntrinArgI8<0>, IntrinArgI8<0> ]
371: //     >,
372: //   ]
373: //
374: class IntrinSelect<Intrinsic intrinsic_,
375:                    list<IntrinArgSelect> arg_selects_ = []> {
376:   Intrinsic intrinsic = intrinsic_;
377:   list<IntrinArgSelect> arg_selects = arg_selects_;
378: }
379:
380: // Abstraction DXIL Operation
381: class DXILOp<int opcode, DXILOpClass opclass> {
382:   // A short description of the operation
383:   string Doc = "";
384:
385:   // Opcode of DXIL Operation
386:   int OpCode = opcode;
387:
388:   // Class of DXIL Operation.
389:   DXILOpClass OpClass = opclass;
390:
391:   // LLVM Intrinsics DXIL Operation maps from
392:   list<IntrinSelect> intrinsics = [];
393:
394:   // Result type of the op
395:   DXILOpParamType result;
396:
397:   // List of argument types of the op. Default to 0 arguments.
398:   list<DXILOpParamType> arguments = [];
399:
400:   // List of valid overload types predicated by DXIL version
401:   list<Overloads> overloads = [];
402:
403:   // List of valid shader stages predicated by DXIL version
404:   list<Stages> stages;
405:
406:   // Versioned attributes of operation
407:   list<Attributes> attributes = [];
408: }
409:
410: // Concrete definitions of DXIL Operations
411: //
412: // This are sorted by ascending value of the DXIL Opcodes
413:
414: def Abs : DXILOp<6, unary> {
415:   let Doc = "Returns the absolute value of the input.";
416:   let intrinsics = [IntrinSelect<int_fabs>];
417:   let arguments = [OverloadTy];
418:   let result = OverloadTy;
419:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy]>];
420:   let stages = [Stages<DXIL1_0, [all_stages]>];
421:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
422: }
423:
424: def Saturate : DXILOp<7, unary> {
425:   let Doc = "Clamps a single or double precision floating point value to "
426:             "[0.0f...1.0f].";
427:   let intrinsics = [IntrinSelect<int_dx_saturate>];
428:   let arguments = [OverloadTy];
429:   let result = OverloadTy;
430:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy]>];
431:   let stages = [Stages<DXIL1_0, [all_stages]>];
432:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
433: }
434:
435: def IsNaN : DXILOp<8, isSpecialFloat> {
436:   let Doc = "Determines if the specified value is NaN.";
437:   let intrinsics = [IntrinSelect<int_dx_isnan>];
438:   let arguments = [OverloadTy];
439:   let result = Int1Ty;
440:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
441:   let stages = [Stages<DXIL1_0, [all_stages]>];
442:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
443: }
444:
445: def IsInf : DXILOp<9, isSpecialFloat> {
446:   let Doc = "Determines if the specified value is infinite.";
447:   let intrinsics = [IntrinSelect<int_dx_isinf>];
448:   let arguments = [OverloadTy];
449:   let result = Int1Ty;
450:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
```
- EN: This range uses TableGen DSL to describe records such as IntrinSelect, DXILOp, Abs, Saturate; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 IntrinSelect、DXILOp、Abs、Saturate 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 451-540
```tablegen
451:   let stages = [Stages<DXIL1_0, [all_stages]>];
452:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
453: }
454:
455: def IsFinite : DXILOp<10, isSpecialFloat> {
456:   let Doc = "Determines if the specified value is finite.";
457:   let arguments = [OverloadTy];
458:   let result = Int1Ty;
459:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
460:   let stages = [Stages<DXIL1_0, [all_stages]>];
461:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
462: }
463:
464: def IsNormal : DXILOp<11, isSpecialFloat> {
465:   let Doc = "Determines if the specified value is normal.";
466:   let arguments = [OverloadTy];
467:   let result = Int1Ty;
468:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
469:   let stages = [Stages<DXIL1_0, [all_stages]>];
470:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
471: }
472:
473: def Cos : DXILOp<12, unary> {
474:   let Doc = "Returns cosine(theta) for theta in radians.";
475:   let intrinsics = [IntrinSelect<int_cos>];
476:   let arguments = [OverloadTy];
477:   let result = OverloadTy;
478:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
479:   let stages = [Stages<DXIL1_0, [all_stages]>];
480:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
481: }
482:
483: def Sin : DXILOp<13, unary> {
484:   let Doc = "Returns sine(theta) for theta in radians.";
485:   let intrinsics = [IntrinSelect<int_sin>];
486:   let arguments = [OverloadTy];
487:   let result = OverloadTy;
488:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
489:   let stages = [Stages<DXIL1_0, [all_stages]>];
490:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
491: }
492:
493: def Tan : DXILOp<14, unary> {
494:   let Doc = "Returns tangent(theta) for theta in radians.";
495:   let intrinsics = [IntrinSelect<int_tan>];
496:   let arguments = [OverloadTy];
497:   let result = OverloadTy;
498:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
499:   let stages = [Stages<DXIL1_0, [all_stages]>];
500:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
501: }
502:
503: def ACos : DXILOp<15, unary> {
504:   let Doc = "Returns the arccosine of the specified value.";
505:   let intrinsics = [IntrinSelect<int_acos>];
506:   let arguments = [OverloadTy];
507:   let result = OverloadTy;
508:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
509:   let stages = [Stages<DXIL1_0, [all_stages]>];
510:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
511: }
512:
513: def ASin : DXILOp<16, unary> {
514:   let Doc = "Returns the arcsine of the specified value.";
515:   let intrinsics = [IntrinSelect<int_asin>];
516:   let arguments = [OverloadTy];
517:   let result = OverloadTy;
518:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
519:   let stages = [Stages<DXIL1_0, [all_stages]>];
520:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
521: }
522:
523: def ATan : DXILOp<17, unary> {
524:   let Doc = "Returns the arctangent of the specified value.";
525:   let intrinsics = [IntrinSelect<int_atan>];
526:   let arguments = [OverloadTy];
527:   let result = OverloadTy;
528:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
529:   let stages = [Stages<DXIL1_0, [all_stages]>];
530:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
531: }
532:
533: def HCos : DXILOp<18, unary> {
534:   let Doc = "Returns the hyperbolic cosine of the specified value.";
535:   let intrinsics = [IntrinSelect<int_cosh>];
536:   let arguments = [OverloadTy];
537:   let result = OverloadTy;
538:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
539:   let stages = [Stages<DXIL1_0, [all_stages]>];
540:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
```
- EN: This range uses TableGen DSL to describe records such as IsFinite, IsNormal, Cos, Sin; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 IsFinite、IsNormal、Cos、Sin 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 541-630
```tablegen
541: }
542:
543: def HSin : DXILOp<19, unary> {
544:   let Doc = "Returns the hyperbolic sine of the specified value.";
545:   let intrinsics = [IntrinSelect<int_sinh>];
546:   let arguments = [OverloadTy];
547:   let result = OverloadTy;
548:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
549:   let stages = [Stages<DXIL1_0, [all_stages]>];
550:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
551: }
552:
553: def HTan : DXILOp<20, unary> {
554:   let Doc = "Returns the hyperbolic tan of the specified value.";
555:   let intrinsics = [IntrinSelect<int_tanh>];
556:   let arguments = [OverloadTy];
557:   let result = OverloadTy;
558:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
559:   let stages = [Stages<DXIL1_0, [all_stages]>];
560:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
561: }
562:
563: def Exp2 : DXILOp<21, unary> {
564:   let Doc = "Returns the base 2 exponential, or 2**x, of the specified value. "
565:             "exp2(x) = 2**x.";
566:   let intrinsics = [IntrinSelect<int_exp2>];
567:   let arguments = [OverloadTy];
568:   let result = OverloadTy;
569:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
570:   let stages = [Stages<DXIL1_0, [all_stages]>];
571:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
572: }
573:
574: def Frac : DXILOp<22, unary> {
575:   let Doc = "Returns a fraction from 0 to 1 that represents the decimal part "
576:             "of the input.";
577:   let intrinsics = [IntrinSelect<int_dx_frac>];
578:   let arguments = [OverloadTy];
579:   let result = OverloadTy;
580:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
581:   let stages = [Stages<DXIL1_0, [all_stages]>];
582:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
583: }
584:
585: def Log2 : DXILOp<23, unary> {
586:   let Doc = "Returns the base-2 logarithm of the specified value.";
587:   let intrinsics = [IntrinSelect<int_log2>];
588:   let arguments = [OverloadTy];
589:   let result = OverloadTy;
590:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
591:   let stages = [Stages<DXIL1_0, [all_stages]>];
592:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
593: }
594:
595: def Sqrt : DXILOp<24, unary> {
596:   let Doc = "Returns the square root of the specified floating-point value, "
597:             "per component.";
598:   let intrinsics = [IntrinSelect<int_sqrt>];
599:   let arguments = [OverloadTy];
600:   let result = OverloadTy;
601:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
602:   let stages = [Stages<DXIL1_0, [all_stages]>];
603:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
604: }
605:
606: def RSqrt : DXILOp<25, unary> {
607:   let Doc = "Returns the reciprocal of the square root of the specified value. "
608:             "rsqrt(x) = 1 / sqrt(x).";
609:   let intrinsics = [IntrinSelect<int_dx_rsqrt>];
610:   let arguments = [OverloadTy];
611:   let result = OverloadTy;
612:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
613:   let stages = [Stages<DXIL1_0, [all_stages]>];
614:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
615: }
616:
617: def Round : DXILOp<26, unary> {
618:   let Doc = "Returns the input rounded to the nearest integer within a "
619:             "floating-point type.";
620:   let intrinsics = [IntrinSelect<int_roundeven>];
621:   let arguments = [OverloadTy];
622:   let result = OverloadTy;
623:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
624:   let stages = [Stages<DXIL1_0, [all_stages]>];
625:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
626: }
627:
628: def Floor : DXILOp<27, unary> {
629:   let Doc =
630:       "Returns the largest integer that is less than or equal to the input.";
```
- EN: This range uses TableGen DSL to describe records such as HSin, HTan, Exp2, Frac; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 HSin、HTan、Exp2、Frac 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 631-720
```tablegen
631:   let intrinsics = [IntrinSelect<int_floor>];
632:   let arguments = [OverloadTy];
633:   let result = OverloadTy;
634:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
635:   let stages = [Stages<DXIL1_0, [all_stages]>];
636:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
637: }
638:
639: def Ceil : DXILOp<28, unary> {
640:   let Doc = "Returns the smallest integer that is greater than or equal to the "
641:             "input.";
642:   let intrinsics = [IntrinSelect<int_ceil>];
643:   let arguments = [OverloadTy];
644:   let result = OverloadTy;
645:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
646:   let stages = [Stages<DXIL1_0, [all_stages]>];
647:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
648: }
649:
650: def Trunc : DXILOp<29, unary> {
651:   let Doc = "Returns the specified value truncated to the integer component.";
652:   let intrinsics = [IntrinSelect<int_trunc>];
653:   let arguments = [OverloadTy];
654:   let result = OverloadTy;
655:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
656:   let stages = [Stages<DXIL1_0, [all_stages]>];
657:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
658: }
659:
660: def Rbits : DXILOp<30, unary> {
661:   let Doc = "Returns the specified value with its bits reversed.";
662:   let intrinsics = [IntrinSelect<int_bitreverse>];
663:   let arguments = [OverloadTy];
664:   let result = OverloadTy;
665:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
666:   let stages = [Stages<DXIL1_0, [all_stages]>];
667:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
668: }
669:
670: def CountBits : DXILOp<31, unaryBits> {
671:   let Doc = "Returns the number of 1 bits in the specified value.";
672:   let arguments = [OverloadTy];
673:   let result = Int32Ty;
674:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
675:   let stages = [Stages<DXIL1_0, [all_stages]>];
676:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
677: }
678:
679: def FirstbitLo : DXILOp<32, unaryBits> {
680:   let Doc = "Returns the location of the first set bit starting from "
681:             "the lowest order bit and working upward.";
682:   let intrinsics = [IntrinSelect<int_dx_firstbitlow>];
683:   let arguments = [OverloadTy];
684:   let result = Int32Ty;
685:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
686:   let stages = [Stages<DXIL1_0, [all_stages]>];
687:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
688: }
689:
690: def FirstbitHi : DXILOp<33, unaryBits> {
691:   let Doc = "Returns the location of the first set bit starting from "
692:             "the highest order bit and working downward.";
693:   let intrinsics = [IntrinSelect<int_dx_firstbituhigh>];
694:   let arguments = [OverloadTy];
695:   let result = Int32Ty;
696:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
697:   let stages = [Stages<DXIL1_0, [all_stages]>];
698:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
699: }
700:
701: def FirstbitSHi : DXILOp<34, unaryBits> {
702:   let Doc = "Returns the location of the first set bit from "
703:             "the highest order bit based on the sign.";
704:   let intrinsics = [IntrinSelect<int_dx_firstbitshigh>];
705:   let arguments = [OverloadTy];
706:   let result = Int32Ty;
707:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
708:   let stages = [Stages<DXIL1_0, [all_stages]>];
709:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
710: }
711:
712: def FMax : DXILOp<35, binary> {
713:   let Doc = "Float maximum. FMax(a,b) = a > b ? a : b";
714:   let intrinsics = [IntrinSelect<int_maxnum>];
715:   let arguments = [OverloadTy, OverloadTy];
716:   let result = OverloadTy;
717:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy]>];
718:   let stages = [Stages<DXIL1_0, [all_stages]>];
719:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
720: }
```
- EN: This range uses TableGen DSL to describe records such as Ceil, Trunc, Rbits, CountBits; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Ceil、Trunc、Rbits、CountBits 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 721-810
```tablegen
721:
722: def FMin : DXILOp<36, binary> {
723:   let Doc = "Float minimum. FMin(a,b) = a < b ? a : b";
724:   let intrinsics = [IntrinSelect<int_minnum>];
725:   let arguments = [OverloadTy, OverloadTy];
726:   let result = OverloadTy;
727:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy]>];
728:   let stages = [Stages<DXIL1_0, [all_stages]>];
729:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
730: }
731:
732: def SMax : DXILOp<37, binary> {
733:   let Doc = "Signed integer maximum. SMax(a,b) = a > b ? a : b";
734:   let intrinsics = [IntrinSelect<int_smax>];
735:   let arguments = [OverloadTy, OverloadTy];
736:   let result = OverloadTy;
737:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
738:   let stages = [Stages<DXIL1_0, [all_stages]>];
739:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
740: }
741:
742: def SMin : DXILOp<38, binary> {
743:   let Doc = "Signed integer minimum. SMin(a,b) = a < b ? a : b";
744:   let intrinsics = [IntrinSelect<int_smin>];
745:   let arguments = [OverloadTy, OverloadTy];
746:   let result = OverloadTy;
747:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
748:   let stages = [Stages<DXIL1_0, [all_stages]>];
749:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
750: }
751:
752: def UMax : DXILOp<39, binary> {
753:   let Doc = "Unsigned integer maximum. UMax(a,b) = a > b ? a : b";
754:   let intrinsics = [IntrinSelect<int_umax>];
755:   let arguments = [OverloadTy, OverloadTy];
756:   let result = OverloadTy;
757:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
758:   let stages = [Stages<DXIL1_0, [all_stages]>];
759:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
760: }
761:
762: def UMin : DXILOp<40, binary> {
763:   let Doc = "Unsigned integer minimum. UMin(a,b) = a < b ? a : b";
764:   let intrinsics = [IntrinSelect<int_umin>];
765:   let arguments = [OverloadTy, OverloadTy];
766:   let result = OverloadTy;
767:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
768:   let stages = [Stages<DXIL1_0, [all_stages]>];
769:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
770: }
771:
772: def UAddc : DXILOp<44, binaryWithCarryOrBorrow > {
773:   let Doc = "unsigned add of 32-bit operand with the carry";
774:   let intrinsics = [IntrinSelect<int_uadd_with_overflow>];
775:   let arguments = [OverloadTy, OverloadTy];
776:   let result = BinaryWithCarryTy;
777:   let overloads = [Overloads<DXIL1_0, [Int32Ty]>];
778:   let stages = [Stages<DXIL1_0, [all_stages]>];
779:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
780: }
781:
782: def FMad : DXILOp<46, tertiary> {
783:   let Doc = "Floating point arithmetic multiply/add operation. fmad(m,a,b) = m "
784:             "* a + b.";
785:   let intrinsics = [IntrinSelect<int_fmuladd>];
786:   let arguments = [OverloadTy, OverloadTy, OverloadTy];
787:   let result = OverloadTy;
788:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy]>];
789:   let stages = [Stages<DXIL1_0, [all_stages]>];
790:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
791: }
792:
793: def Fma : DXILOp<47, tertiary> {
794:   let Doc = "Double-precision fused multiply-add. fma(a,b,c) = a * b + c.";
795:   let intrinsics = [IntrinSelect<int_fma>];
796:   let arguments = [OverloadTy, OverloadTy, OverloadTy];
797:   let result = OverloadTy;
798:   let overloads = [Overloads<DXIL1_0, [DoubleTy]>];
799:   let stages = [Stages<DXIL1_0, [all_stages]>];
800:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
801: }
802:
803: def IMad : DXILOp<48, tertiary> {
804:   let Doc = "Signed integer arithmetic multiply/add operation. imad(m,a,b) = m "
805:             "* a + b.";
806:   let intrinsics = [IntrinSelect<int_dx_imad>];
807:   let arguments = [OverloadTy, OverloadTy, OverloadTy];
808:   let result = OverloadTy;
809:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
810:   let stages = [Stages<DXIL1_0, [all_stages]>];
```
- EN: This range uses TableGen DSL to describe records such as FMin, SMax, SMin, UMax; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 FMin、SMax、SMin、UMax 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 811-900
```tablegen
811:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
812: }
813:
814: def UMad : DXILOp<49, tertiary> {
815:   let Doc = "Unsigned integer arithmetic multiply/add operation. umad(m,a, = m "
816:             "* a + b.";
817:   let intrinsics = [IntrinSelect<int_dx_umad>];
818:   let arguments = [OverloadTy, OverloadTy, OverloadTy];
819:   let result = OverloadTy;
820:   let overloads = [Overloads<DXIL1_0, [Int16Ty, Int32Ty, Int64Ty]>];
821:   let stages = [Stages<DXIL1_0, [all_stages]>];
822:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
823: }
824:
825: def Dot2 : DXILOp<54, dot2> {
826:   let Doc = "dot product of two float vectors Dot(a,b) = a[0]*b[0] + ... + "
827:             "a[n]*b[n] where n is 0 to 1 inclusive";
828:   let intrinsics = [IntrinSelect<int_dx_dot2>];
829:   let arguments = !listsplat(OverloadTy, 4);
830:   let result = OverloadTy;
831:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
832:   let stages = [Stages<DXIL1_0, [all_stages]>];
833:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
834: }
835:
836: def Dot3 : DXILOp<55, dot3> {
837:   let Doc = "dot product of two float vectors Dot(a,b) = a[0]*b[0] + ... + "
838:             "a[n]*b[n] where n is 0 to 2 inclusive";
839:   let intrinsics = [IntrinSelect<int_dx_dot3>];
840:   let arguments = !listsplat(OverloadTy, 6);
841:   let result = OverloadTy;
842:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
843:   let stages = [Stages<DXIL1_0, [all_stages]>];
844:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
845: }
846:
847: def Dot4 : DXILOp<56, dot4> {
848:   let Doc = "dot product of two float vectors Dot(a,b) = a[0]*b[0] + ... + "
849:             "a[n]*b[n] where n is 0 to 3 inclusive";
850:   let intrinsics = [IntrinSelect<int_dx_dot4>];
851:   let arguments = !listsplat(OverloadTy, 8);
852:   let result = OverloadTy;
853:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
854:   let stages = [Stages<DXIL1_0, [all_stages]>];
855:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
856: }
857:
858: def CreateHandle : DXILOp<57, createHandle> {
859:   let Doc = "creates the handle to a resource";
860:   // ResourceClass, RangeID, Index, NonUniform
861:   let arguments = [Int8Ty, Int32Ty, Int32Ty, Int1Ty];
862:   let result = HandleTy;
863:   let stages = [Stages<DXIL1_0, [all_stages]>, Stages<DXIL1_6, [removed]>];
864:   // NOTE: The ReadOnly attribute was set for consistency with DXC. However, it
865:   // seems like ReadNone may more appropiately describe it. So noted to
866:   // consider a change in the future
867:   let attributes = [Attributes<DXIL1_0, [ReadOnly]>];
868: }
869:
870: def CBufferLoadLegacy : DXILOp<59, cbufferLoadLegacy> {
871:   let Doc = "loads a value from a constant buffer resource";
872:   // Handle, Index
873:   let arguments = [HandleTy, Int32Ty];
874:   let result = OverloadTy;
875:   let overloads = [Overloads<DXIL1_0, [
876:     CBufRetHalfTy, CBufRetFloatTy, CBufRetDoubleTy, CBufRetInt16Ty,
877:     CBufRetInt32Ty, CBufRetInt64Ty
878:   ]>];
879:   let stages = [Stages<DXIL1_0, [all_stages]>];
880:   let attributes = [Attributes<DXIL1_0, [ReadOnly]>];
881: }
882:
883: def TextureLoad : DXILOp<66, textureLoad> {
884:   let Doc = "reads from a texture resource";
885:   // Handle, MipLevelOrSampleCount, Coord0, Coord1, Coord2, Offset0, Offset1, Offset2
886:   let arguments = [HandleTy, Int32Ty, Int32Ty, Int32Ty, Int32Ty, Int32Ty,
887:                    Int32Ty, Int32Ty];
888:   let result = OverloadTy;
889:   let overloads =
890:       [Overloads<DXIL1_0,
891:                  [ResRetHalfTy, ResRetFloatTy, ResRetInt16Ty, ResRetInt32Ty]>];
892:   let stages = [Stages<DXIL1_0, [all_stages]>];
893:   let attributes = [Attributes<DXIL1_0, [ReadOnly]>];
894: }
895:
896: def BufferLoad : DXILOp<68, bufferLoad> {
897:   let Doc = "reads from a TypedBuffer";
898:   // Handle, Coord0, Coord1
899:   let arguments = [HandleTy, Int32Ty, Int32Ty];
900:   let result = OverloadTy;
```
- EN: This range uses TableGen DSL to describe records such as UMad, Dot2, Dot3, Dot4; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 UMad、Dot2、Dot3、Dot4 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 901-990
```tablegen
901:   let overloads =
902:       [Overloads<DXIL1_0,
903:                  [ResRetHalfTy, ResRetFloatTy, ResRetInt16Ty, ResRetInt32Ty]>];
904:   let stages = [Stages<DXIL1_0, [all_stages]>];
905:   let attributes = [Attributes<DXIL1_0, [ReadOnly]>];
906: }
907:
908: def BufferStore : DXILOp<69, bufferStore> {
909:   let Doc = "writes to an RWTypedBuffer";
910:   // Handle, Coord0, Coord1, Val0, Val1, Val2, Val3, Mask
911:   let arguments = [
912:     HandleTy, Int32Ty, Int32Ty, OverloadTy, OverloadTy, OverloadTy, OverloadTy,
913:     Int8Ty
914:   ];
915:   let result = VoidTy;
916:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy, Int16Ty, Int32Ty]>];
917:   let stages = [Stages<DXIL1_0, [all_stages]>];
918: }
919:
920: def UpdateCounter : DXILOp<70, bufferUpdateCounter> {
921:   let Doc = "increments/decrements a buffer counter";
922:   let arguments = [HandleTy, Int8Ty];
923:   let result = Int32Ty;
924:   let stages = [Stages<DXIL1_0, [all_stages]>];
925: }
926:
927: def CheckAccessFullyMapped : DXILOp<71, checkAccessFullyMapped> {
928:   let Doc = "checks whether a Sample, Gather, or Load operation "
929:             "accessed mapped tiles in a tiled resource";
930:   let arguments = [OverloadTy];
931:   let result = Int1Ty;
932:   let overloads = [Overloads<DXIL1_0, [Int32Ty]>];
933:   let stages = [Stages<DXIL1_0, [all_stages]>];
934:   let attributes = [Attributes<DXIL1_0, [ReadOnly]>];
935: }
936:
937: def GetDimensions : DXILOp<72, getDimensions> {
938:   let Doc = "gets the dimensions of a buffer or texture";
939:   let arguments = [HandleTy, Int32Ty];
940:   let result = DimensionsTy;
941:   let stages = [Stages<DXIL1_0, [all_stages]>];
942: }
943:
944: def Barrier : DXILOp<80, barrier> {
945:   let Doc = "inserts a memory barrier in the shader";
946:   let intrinsics = [
947:     IntrinSelect<int_dx_all_memory_barrier,
948:                  [IntrinArgI32<BarrierMode_AllMemoryBarrier>]>,
949:     IntrinSelect<int_dx_all_memory_barrier_with_group_sync,
950:                  [IntrinArgI32<BarrierMode_AllMemoryBarrierWithGroupSync>]>,
951:     IntrinSelect<int_dx_device_memory_barrier,
952:                  [IntrinArgI32<BarrierMode_DeviceMemoryBarrier>]>,
953:     IntrinSelect<int_dx_device_memory_barrier_with_group_sync,
954:                  [IntrinArgI32<BarrierMode_DeviceMemoryBarrierWithGroupSync>]>,
955:     IntrinSelect<int_dx_group_memory_barrier,
956:                  [IntrinArgI32<BarrierMode_GroupMemoryBarrier>]>,
957:     IntrinSelect<int_dx_group_memory_barrier_with_group_sync,
958:                  [IntrinArgI32<BarrierMode_GroupMemoryBarrierWithGroupSync>]>,
959:   ];
960:
961:   let arguments = [Int32Ty];
962:   let result = VoidTy;
963:   let stages = [Stages<DXIL1_0, [compute, library]>];
964:   let attributes = [Attributes<DXIL1_0, []>];
965: }
966:
967: def Discard : DXILOp<82, discard> {
968:   let Doc = "discard the current pixel";
969:   let intrinsics = [IntrinSelect<int_dx_discard>];
970:   let arguments = [Int1Ty];
971:   let result = VoidTy;
972:   let stages = [Stages<DXIL1_0, [pixel]>];
973: }
974:
975: def DerivCoarseX : DXILOp<83, unary> {
976:   let Doc = "computes the rate of change per stamp in x direction";
977:   let intrinsics = [IntrinSelect<int_dx_ddx_coarse>];
978:   let arguments = [OverloadTy];
979:   let result = OverloadTy;
980:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
981:   let stages = [Stages<DXIL1_0, [library, pixel]>];
982: }
983:
984: def DerivCoarseY : DXILOp<84, unary> {
985:   let Doc = "computes the rate of change per stamp in y direction";
986:   let intrinsics = [IntrinSelect<int_dx_ddy_coarse>];
987:   let arguments = [OverloadTy];
988:   let result = OverloadTy;
989:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
990:   let stages = [Stages<DXIL1_0, [library, pixel]>];
```
- EN: This range uses TableGen DSL to describe records such as BufferStore, UpdateCounter, CheckAccessFullyMapped, GetDimensions; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 BufferStore、UpdateCounter、CheckAccessFullyMapped、GetDimensions 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 991-1080
```tablegen
 991: }
 992:
 993: def DerivFineX : DXILOp<85, unary> {
 994:   let Doc = "computes the rate of change per stamp in x direction";
 995:   let intrinsics = [IntrinSelect<int_dx_ddx_fine>];
 996:   let arguments = [OverloadTy];
 997:   let result = OverloadTy;
 998:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
 999:   let stages = [Stages<DXIL1_0, [library, pixel]>];
1000: }
1001:
1002: def DerivFineY : DXILOp<86, unary> {
1003:   let Doc = "computes the rate of change per stamp in y direction";
1004:   let intrinsics = [IntrinSelect<int_dx_ddy_fine>];
1005:   let arguments = [OverloadTy];
1006:   let result = OverloadTy;
1007:   let overloads = [Overloads<DXIL1_0, [HalfTy, FloatTy]>];
1008:   let stages = [Stages<DXIL1_0, [library, pixel]>];
1009: }
1010:
1011: def ThreadId : DXILOp<93, threadId> {
1012:   let Doc = "Reads the thread ID";
1013:   let intrinsics = [IntrinSelect<int_dx_thread_id>];
1014:   let arguments = [OverloadTy];
1015:   let result = OverloadTy;
1016:   let overloads = [Overloads<DXIL1_0, [Int32Ty]>];
1017:   let stages = [Stages<DXIL1_0, [compute, mesh, amplification, node]>];
1018:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1019: }
1020:
1021: def GroupId : DXILOp<94, groupId> {
1022:   let Doc = "Reads the group ID (SV_GroupID)";
1023:   let intrinsics = [IntrinSelect<int_dx_group_id>];
1024:   let arguments = [OverloadTy];
1025:   let result = OverloadTy;
1026:   let overloads = [Overloads<DXIL1_0, [Int32Ty]>];
1027:   let stages = [Stages<DXIL1_0, [compute, mesh, amplification, node]>];
1028:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1029: }
1030:
1031: def ThreadIdInGroup : DXILOp<95, threadIdInGroup> {
1032:   let Doc = "Reads the thread ID within the group  (SV_GroupThreadID)";
1033:   let intrinsics = [IntrinSelect<int_dx_thread_id_in_group>];
1034:   let arguments = [OverloadTy];
1035:   let result = OverloadTy;
1036:   let overloads = [Overloads<DXIL1_0, [Int32Ty]>];
1037:   let stages = [Stages<DXIL1_0, [compute, mesh, amplification, node]>];
1038:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1039: }
1040:
1041: def FlattenedThreadIdInGroup : DXILOp<96, flattenedThreadIdInGroup> {
1042:   let Doc = "Provides a flattened index for a given thread within a given "
1043:             "group (SV_GroupIndex)";
1044:   let intrinsics = [IntrinSelect<int_dx_flattened_thread_id_in_group>];
1045:   let result = OverloadTy;
1046:   let overloads = [Overloads<DXIL1_0, [Int32Ty]>];
1047:   let stages = [Stages<DXIL1_0, [compute, mesh, amplification, node]>];
1048:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1049: }
1050:
1051: def MakeDouble : DXILOp<101, makeDouble> {
1052:   let Doc = "creates a double value";
1053:   let intrinsics = [IntrinSelect<int_dx_asdouble>];
1054:   let arguments = [Int32Ty, Int32Ty];
1055:   let result = OverloadTy;
1056:   let overloads = [Overloads<DXIL1_0, [DoubleTy]>];
1057:   let stages = [Stages<DXIL1_0, [all_stages]>];
1058:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1059: }
1060:
1061: def SplitDouble : DXILOp<102, splitDouble> {
1062:   let Doc = "Splits a double into 2 uints";
1063:   let intrinsics = [IntrinSelect<int_dx_splitdouble>];
1064:   let arguments = [OverloadTy];
1065:   let result = SplitDoubleTy;
1066:   let overloads = [Overloads<DXIL1_0, [DoubleTy]>];
1067:   let stages = [Stages<DXIL1_0, [all_stages]>];
1068:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1069: }
1070:
1071: def WaveIsFirstLane : DXILOp<110, waveIsFirstLane> {
1072:   let Doc = "returns 1 for the first lane in the wave";
1073:   let intrinsics = [IntrinSelect<int_dx_wave_is_first_lane>];
1074:   let arguments = [];
1075:   let result = Int1Ty;
1076:   let stages = [Stages<DXIL1_0, [all_stages]>];
1077: }
1078:
1079: def WaveGetLaneIndex : DXILOp<111, waveGetLaneIndex> {
1080:   let Doc = "returns the index of the current lane in the wave";
```
- EN: This range uses TableGen DSL to describe records such as DerivFineX, DerivFineY, ThreadId, GroupId; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DerivFineX、DerivFineY、ThreadId、GroupId 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1081-1170
```tablegen
1081:   let intrinsics = [IntrinSelect<int_dx_wave_getlaneindex>];
1082:   let arguments = [];
1083:   let result = Int32Ty;
1084:   let stages = [Stages<DXIL1_0, [all_stages]>];
1085:   let attributes = [Attributes<DXIL1_0, [ReadOnly]>];
1086: }
1087:
1088: def WaveActiveAnyTrue : DXILOp<113, waveAnyTrue> {
1089:   let Doc = "returns true if the expression is true in any of the active lanes "
1090:             "in the current wave";
1091:   let intrinsics = [IntrinSelect<int_dx_wave_any>];
1092:   let arguments = [Int1Ty];
1093:   let result = Int1Ty;
1094:   let stages = [Stages<DXIL1_0, [all_stages]>];
1095: }
1096:
1097: def WaveActiveAllTrue : DXILOp<114, waveAllTrue> {
1098:   let Doc = "returns true if the expression is true in all of the active lanes "
1099:             "in the current wave";
1100:   let intrinsics = [IntrinSelect<int_dx_wave_all>];
1101:   let arguments = [Int1Ty];
1102:   let result = Int1Ty;
1103:   let stages = [Stages<DXIL1_0, [all_stages]>];
1104: }
1105:
1106: def WaveActiveAllEqual : DXILOp<115, waveActiveAllEqual> {
1107:   let Doc = "returns true for each scalar element of the expression if the "
1108:             "expression is equal in all of the active lanes in the current wave";
1109:   let intrinsics = [IntrinSelect<int_dx_wave_all_equal>];
1110:   let arguments = [OverloadTy];
1111:   let result = Int1Ty;
1112:   let stages = [Stages<DXIL1_0, [all_stages]>];
1113: }
1114:
1115: def WaveActiveBallot : DXILOp<116, waveActiveBallot> {
1116:   let Doc = "returns uint4 containing a bitmask of the evaluation of the boolean expression for all active lanes in the current wave.";
1117:   let intrinsics = [IntrinSelect<int_dx_wave_ballot>];
1118:   let arguments = [Int1Ty];
1119:   let result = Fouri32s;
1120:   let stages = [Stages<DXIL1_0, [all_stages]>];
1121: }
1122:
1123: def WaveReadLaneAt : DXILOp<117, waveReadLaneAt> {
1124:   let Doc = "returns the value from the specified lane";
1125:   let intrinsics = [IntrinSelect<int_dx_wave_readlane>];
1126:   let arguments = [OverloadTy, Int32Ty];
1127:   let result = OverloadTy;
1128:   let overloads = [Overloads<
1129:       DXIL1_0, [HalfTy, FloatTy, DoubleTy, Int1Ty, Int16Ty, Int32Ty, Int64Ty]>];
1130:   let stages = [Stages<DXIL1_0, [all_stages]>];
1131: }
1132:
1133: def WaveActiveOp : DXILOp<119, waveActiveOp> {
1134:   let Doc = "returns the result of the operation across waves";
1135:   let intrinsics = [
1136:     IntrinSelect<int_dx_wave_reduce_sum,
1137:                  [
1138:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Sum>,
1139:                    IntrinArgI8<SignedOpKind_Signed>
1140:                  ]>,
1141:     IntrinSelect<int_dx_wave_reduce_usum,
1142:                  [
1143:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Sum>,
1144:                    IntrinArgI8<SignedOpKind_Unsigned>
1145:                  ]>,
1146:     IntrinSelect<int_dx_wave_product,
1147:                  [
1148:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Product>,
1149:                    IntrinArgI8<SignedOpKind_Signed>
1150:                  ]>,
1151:     IntrinSelect<int_dx_wave_uproduct,
1152:                  [
1153:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Product>,
1154:                    IntrinArgI8<SignedOpKind_Unsigned>
1155:                  ]>,
1156:     IntrinSelect<int_dx_wave_reduce_max,
1157:                  [
1158:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Max>,
1159:                    IntrinArgI8<SignedOpKind_Signed>
1160:                  ]>,
1161:     IntrinSelect<int_dx_wave_reduce_umax,
1162:                  [
1163:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Max>,
1164:                    IntrinArgI8<SignedOpKind_Unsigned>
1165:                  ]>,
1166:     IntrinSelect<int_dx_wave_reduce_min,
1167:                  [
1168:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Min>,
1169:                    IntrinArgI8<SignedOpKind_Signed>
1170:                  ]>,
```
- EN: This range uses TableGen DSL to describe records such as WaveActiveAnyTrue, WaveActiveAllTrue, WaveActiveAllEqual, WaveActiveBallot; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 WaveActiveAnyTrue、WaveActiveAllTrue、WaveActiveAllEqual、WaveActiveBallot 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1171-1260
```tablegen
1171:     IntrinSelect<int_dx_wave_reduce_umin,
1172:                  [
1173:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Min>,
1174:                    IntrinArgI8<SignedOpKind_Unsigned>
1175:                  ]>,
1176:   ];
1177:
1178:   let arguments = [OverloadTy, Int8Ty, Int8Ty];
1179:   let result = OverloadTy;
1180:   let overloads = [
1181:     Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy, Int16Ty, Int32Ty, Int64Ty]>
1182:   ];
1183:   let stages = [Stages<DXIL1_0, [all_stages]>];
1184:   let attributes = [Attributes<DXIL1_0, []>];
1185: }
1186:
1187: def WaveActiveBit : DXILOp<120, waveActiveBit> {
1188:   let Doc = "returns the result of the operation across waves";
1189:   let intrinsics = [
1190:     IntrinSelect<int_dx_wave_reduce_or,
1191:                  [
1192:                    IntrinArgIndex<0>, IntrinArgI8<WaveBitOpKind_Or>,
1193:                  ]>,
1194:     IntrinSelect<int_dx_wave_reduce_xor,
1195:                  [
1196:                    IntrinArgIndex<0>, IntrinArgI8<WaveBitOpKind_Xor>,
1197:                  ]>,
1198:     IntrinSelect<int_dx_wave_reduce_and,
1199:                  [
1200:                    IntrinArgIndex<0>, IntrinArgI8<WaveBitOpKind_And>,
1201:                  ]>
1202:   ];
1203:
1204:   let arguments = [OverloadTy, Int8Ty];
1205:   let result = OverloadTy;
1206:   let overloads = [
1207:     Overloads<DXIL1_0, [Int32Ty, Int64Ty]>
1208:   ];
1209:   let stages = [Stages<DXIL1_0, [all_stages]>];
1210:   let attributes = [Attributes<DXIL1_0, []>];
1211: }
1212:
1213: def WavePrefixOp : DXILOp<121, wavePrefixOp> {
1214:   let Doc = "returns partial result of the computation in the corresponding lane";
1215:   let intrinsics = [
1216:     IntrinSelect<int_dx_wave_prefix_sum,
1217:                  [
1218:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Sum>,
1219:                    IntrinArgI8<SignedOpKind_Signed>
1220:                  ]>,
1221:     IntrinSelect<int_dx_wave_prefix_usum,
1222:                  [
1223:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Sum>,
1224:                    IntrinArgI8<SignedOpKind_Unsigned>
1225:                  ]>,
1226:     IntrinSelect<int_dx_wave_prefix_product,
1227:                  [
1228:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Product>,
1229:                    IntrinArgI8<SignedOpKind_Signed>
1230:                  ]>,
1231:     IntrinSelect<int_dx_wave_prefix_uproduct,
1232:                  [
1233:                    IntrinArgIndex<0>, IntrinArgI8<WaveOpKind_Product>,
1234:                    IntrinArgI8<SignedOpKind_Unsigned>
1235:                  ]>,
1236:   ];
1237:
1238:   let arguments = [OverloadTy, Int8Ty, Int8Ty];
1239:   let result = OverloadTy;
1240:   let overloads = [
1241:     Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy, Int16Ty, Int32Ty, Int64Ty]>
1242:   ];
1243:   let stages = [Stages<DXIL1_0, [all_stages]>];
1244:   let attributes = [Attributes<DXIL1_0, []>];
1245: }
1246:
1247: def QuadOp : DXILOp<123, quadOp> {
1248:   let Doc = "returns the value from another lane within the quad by swapping values in a direction";
1249:   let intrinsics = [
1250:     IntrinSelect<int_dx_quad_read_across_x,
1251:                  [
1252:                    IntrinArgIndex<0>, IntrinArgI8<QuadOpKind_ReadAcrossX>
1253:                  ]>,
1254:     IntrinSelect<int_dx_quad_read_across_y,
1255:                  [
1256:                    IntrinArgIndex<0>, IntrinArgI8<QuadOpKind_ReadAcrossY>
1257:                  ]>,
1258:   ];
1259:
1260:   let arguments = [OverloadTy, Int8Ty];
```
- EN: This range uses TableGen DSL to describe records such as WaveActiveBit, WavePrefixOp, QuadOp; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 WaveActiveBit、WavePrefixOp、QuadOp 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1261-1350
```tablegen
1261:   let result = OverloadTy;
1262:   let overloads = [
1263:     Overloads<DXIL1_0, [HalfTy, FloatTy, DoubleTy, Int16Ty, Int32Ty, Int64Ty]>
1264:   ];
1265:   let stages = [Stages<DXIL1_0, [all_stages]>];
1266:   let attributes = [Attributes<DXIL1_0, []>];
1267: }
1268:
1269: def WavePrefixBitCount : DXILOp<136, wavePrefixOp> {
1270:   let Doc = "returns the count of bits of Expr set to 1 on prior lanes";
1271:   let intrinsics = [IntrinSelect<int_dx_wave_prefix_bit_count>];
1272:   let arguments = [Int1Ty];
1273:   let result = Int32Ty;
1274:   let stages = [Stages<DXIL1_0, [all_stages]>];
1275:   let attributes = [Attributes<DXIL1_0, []>];
1276: }
1277:
1278: def LegacyF32ToF16 : DXILOp<130, legacyF32ToF16> {
1279:   let Doc = "converts the float stored in the first element of the float2 "
1280:             "to a half and stores it in the low-half of the output uint";
1281:   let intrinsics = [IntrinSelect<int_dx_legacyf32tof16>];
1282:   let arguments = [FloatTy];
1283:   let result = Int32Ty;
1284:   let stages = [Stages<DXIL1_0, [all_stages]>];
1285: }
1286:
1287: def LegacyF16ToF32 : DXILOp<131, legacyF16ToF32> {
1288:   let Doc = "returns the half stored in the low-half of the uint converted "
1289:             "to a float";
1290:   let intrinsics = [IntrinSelect<int_dx_legacyf16tof32>];
1291:   let arguments = [Int32Ty];
1292:   let result = FloatTy;
1293:   let stages = [Stages<DXIL1_0, [all_stages]>];
1294: }
1295:
1296: def WaveAllBitCount : DXILOp<135, waveAllOp> {
1297:   let Doc = "returns the count of bits set to 1 across the wave";
1298:   let intrinsics = [IntrinSelect<int_dx_wave_active_countbits>];
1299:   let arguments = [Int1Ty];
1300:   let result = Int32Ty;
1301:   let stages = [Stages<DXIL1_0, [all_stages]>];
1302: }
1303:
1304: def RawBufferLoad : DXILOp<139, rawBufferLoad> {
1305:   let Doc = "reads from a raw buffer and structured buffer";
1306:   // Handle, Coord0, Coord1, Mask, Alignment
1307:   let arguments = [HandleTy, Int32Ty, Int32Ty, Int8Ty, Int32Ty];
1308:   let result = OverloadTy;
1309:   let overloads = [
1310:     Overloads<DXIL1_2,
1311:               [ResRetHalfTy, ResRetFloatTy, ResRetInt16Ty, ResRetInt32Ty]>,
1312:     Overloads<DXIL1_3,
1313:               [
1314:                 ResRetHalfTy, ResRetFloatTy, ResRetDoubleTy, ResRetInt16Ty,
1315:                 ResRetInt32Ty, ResRetInt64Ty
1316:               ]>
1317:   ];
1318:   let stages = [Stages<DXIL1_2, [all_stages]>];
1319: }
1320:
1321: def RawBufferStore : DXILOp<140, rawBufferStore> {
1322:   let Doc = "writes to a RWByteAddressBuffer or RWStructuredBuffer";
1323:   // Handle, Coord0, Coord1, Val0, Val1, Val2, Val3, Mask, Alignment
1324:   let arguments = [
1325:     HandleTy, Int32Ty, Int32Ty, OverloadTy, OverloadTy, OverloadTy, OverloadTy,
1326:     Int8Ty, Int32Ty
1327:   ];
1328:   let result = VoidTy;
1329:   let overloads = [
1330:     Overloads<DXIL1_2,
1331:               [ResRetHalfTy, ResRetFloatTy, ResRetInt16Ty, ResRetInt32Ty]>,
1332:     Overloads<DXIL1_3,
1333:               [
1334:                 ResRetHalfTy, ResRetFloatTy, ResRetDoubleTy, ResRetInt16Ty,
1335:                 ResRetInt32Ty, ResRetInt64Ty
1336:               ]>
1337:   ];
1338:   let stages = [Stages<DXIL1_2, [all_stages]>];
1339: }
1340:
1341: def Dot2AddHalf : DXILOp<162, dot2AddHalf> {
1342:   let Doc = "2D half dot product with accumulate to float";
1343:   let intrinsics = [IntrinSelect<int_dx_dot2add>];
1344:   let arguments = [OverloadTy, HalfTy, HalfTy, HalfTy, HalfTy];
1345:   let result = OverloadTy;
1346:   let overloads = [Overloads<DXIL1_4, [FloatTy]>];
1347:   let stages = [Stages<DXIL1_4, [all_stages]>];
1348:   let attributes = [Attributes<DXIL1_4, [ReadNone]>];
1349: }
1350:
```
- EN: This range uses TableGen DSL to describe records such as WavePrefixBitCount, LegacyF32ToF16, LegacyF16ToF32, WaveAllBitCount; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 WavePrefixBitCount、LegacyF32ToF16、LegacyF16ToF32、WaveAllBitCount 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1351-1387
```tablegen
1351: def Dot4AddI8Packed : DXILOp<163, dot4AddPacked> {
1352:   let Doc = "signed dot product of 4 x i8 vectors packed into i32, with "
1353:             "accumulate to i32";
1354:   let intrinsics = [IntrinSelect<int_dx_dot4add_i8packed>];
1355:   let arguments = [Int32Ty, Int32Ty, Int32Ty];
1356:   let result = OverloadTy;
1357:   let overloads = [Overloads<DXIL1_4, [Int32Ty]>];
1358:   let stages = [Stages<DXIL1_4, [all_stages]>];
1359:   let attributes = [Attributes<DXIL1_4, [ReadNone]>];
1360: }
1361:
1362: def Dot4AddU8Packed : DXILOp<164, dot4AddPacked> {
1363:   let Doc = "unsigned dot product of 4 x i8 vectors packed into i32, with "
1364:             "accumulate to i32";
1365:   let intrinsics = [IntrinSelect<int_dx_dot4add_u8packed>];
1366:   let arguments = [Int32Ty, Int32Ty, Int32Ty];
1367:   let result = OverloadTy;
1368:   let overloads = [Overloads<DXIL1_4, [Int32Ty]>];
1369:   let stages = [Stages<DXIL1_4, [all_stages]>];
1370:   let attributes = [Attributes<DXIL1_4, [ReadNone]>];
1371: }
1372:
1373: def AnnotateHandle : DXILOp<216, annotateHandle> {
1374:   let Doc = "annotate handle with resource properties";
1375:   let arguments = [HandleTy, ResPropsTy];
1376:   let result = HandleTy;
1377:   let stages = [Stages<DXIL1_6, [all_stages]>];
1378:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1379: }
1380:
1381: def CreateHandleFromBinding : DXILOp<217, createHandleFromBinding> {
1382:   let Doc = "create resource handle from binding";
1383:   let arguments = [ResBindTy, Int32Ty, Int1Ty];
1384:   let result = HandleTy;
1385:   let stages = [Stages<DXIL1_6, [all_stages]>];
1386:   let attributes = [Attributes<DXIL1_0, [ReadNone]>];
1387: }
```
- EN: This range uses TableGen DSL to describe records such as Dot4AddI8Packed, Dot4AddU8Packed, AnnotateHandle, CreateHandleFromBinding; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 Dot4AddI8Packed、Dot4AddU8Packed、AnnotateHandle、CreateHandleFromBinding 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include Version, DXIL1_, DXILOpParamType, OverloadTy, VoidTy, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 Version, DXIL1_, DXILOpParamType, OverloadTy, VoidTy，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/IR/Intrinsics.td`
- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
