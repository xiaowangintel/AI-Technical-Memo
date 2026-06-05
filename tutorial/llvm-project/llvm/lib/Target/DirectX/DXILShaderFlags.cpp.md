# DXILShaderFlags.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILShaderFlags.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains helper objects and APIs for working with DXIL Shader Flags.
- 目的（中文）: 该文件提供 LLVM 目标后端所需的辅助逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILShaderFlags.cpp - DXIL Shader Flags helper objects -------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains helper objects and APIs for working with DXIL
10: ///       Shader Flags.
11: ///
12: //===----------------------------------------------------------------------===//
13:
14: #include "DXILShaderFlags.h"
15: #include "DirectX.h"
16: #include "llvm/ADT/SCCIterator.h"
17: #include "llvm/ADT/SmallVector.h"
18: #include "llvm/Analysis/CallGraph.h"
19: #include "llvm/Analysis/DXILResource.h"
20: #include "llvm/IR/Attributes.h"
21: #include "llvm/IR/DiagnosticInfo.h"
22: #include "llvm/IR/Instruction.h"
23: #include "llvm/IR/Instructions.h"
24: #include "llvm/IR/IntrinsicInst.h"
25: #include "llvm/IR/Intrinsics.h"
26: #include "llvm/IR/IntrinsicsDirectX.h"
27: #include "llvm/IR/Module.h"
28: #include "llvm/InitializePasses.h"
29: #include "llvm/Support/FormatVariadic.h"
30: #include "llvm/Support/raw_ostream.h"
31:
32: using namespace llvm;
33: using namespace llvm::dxil;
34:
35: static bool hasUAVsAtEveryStage(const DXILResourceMap &DRM,
36:                                 const ModuleMetadataInfo &MMDI) {
37:   if (DRM.uavs().empty())
38:     return false;
39:
40:   switch (MMDI.ShaderProfile) {
41:   default:
42:     return false;
43:   case Triple::EnvironmentType::Compute:
44:   case Triple::EnvironmentType::Pixel:
45:     return false;
46:   case Triple::EnvironmentType::Vertex:
47:   case Triple::EnvironmentType::Geometry:
48:   case Triple::EnvironmentType::Hull:
49:   case Triple::EnvironmentType::Domain:
50:     return true;
51:   case Triple::EnvironmentType::Library:
52:   case Triple::EnvironmentType::RayGeneration:
53:   case Triple::EnvironmentType::Intersection:
54:   case Triple::EnvironmentType::AnyHit:
55:   case Triple::EnvironmentType::ClosestHit:
56:   case Triple::EnvironmentType::Miss:
57:   case Triple::EnvironmentType::Callable:
58:   case Triple::EnvironmentType::Mesh:
59:   case Triple::EnvironmentType::Amplification:
60:     return MMDI.ValidatorVersion < VersionTuple(1, 8);
```
- EN: This range implements operational logic in helpers such as VersionTuple, translating backend policy into executable code.
- CN: 这一段实现了 VersionTuple 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:   }
 62: }
 63:
 64: static bool checkWaveOps(Intrinsic::ID IID) {
 65:   // Currently unsupported intrinsics
 66:   // case Intrinsic::dx_wave_getlanecount:
 67:   // case Intrinsic::dx_wave_readfirst:
 68:   // case Intrinsic::dx_wave_reduce.and:
 69:   // case Intrinsic::dx_wave_reduce.or:
 70:   // case Intrinsic::dx_wave_reduce.xor:
 71:   // case Intrinsic::dx_wave_prefixop:
 72:   // case Intrinsic::dx_quad.readat:
 73:   // case Intrinsic::dx_quad.readacrossy:
 74:   // case Intrinsic::dx_quad.readacrossdiagonal:
 75:   // case Intrinsic::dx_wave_prefixballot:
 76:   // case Intrinsic::dx_wave_match:
 77:   // case Intrinsic::dx_wavemulti.*:
 78:   // case Intrinsic::dx_wavemulti.ballot:
 79:   // case Intrinsic::dx_quad.vote:
 80:   switch (IID) {
 81:   default:
 82:     return false;
 83:   case Intrinsic::dx_wave_is_first_lane:
 84:   case Intrinsic::dx_wave_getlaneindex:
 85:   case Intrinsic::dx_wave_any:
 86:   case Intrinsic::dx_wave_all_equal:
 87:   case Intrinsic::dx_wave_all:
 88:   case Intrinsic::dx_wave_readlane:
 89:   case Intrinsic::dx_wave_active_countbits:
 90:   case Intrinsic::dx_wave_ballot:
 91:   case Intrinsic::dx_wave_prefix_bit_count:
 92:   // Wave Active Op Variants
 93:   case Intrinsic::dx_wave_reduce_or:
 94:   case Intrinsic::dx_wave_reduce_xor:
 95:   case Intrinsic::dx_wave_reduce_and:
 96:   case Intrinsic::dx_wave_reduce_sum:
 97:   case Intrinsic::dx_wave_reduce_usum:
 98:   case Intrinsic::dx_wave_product:
 99:   case Intrinsic::dx_wave_uproduct:
100:   case Intrinsic::dx_wave_reduce_max:
101:   case Intrinsic::dx_wave_reduce_umax:
102:   case Intrinsic::dx_wave_reduce_min:
103:   case Intrinsic::dx_wave_reduce_umin:
104:     // Wave Prefix Op Variants
105:   case Intrinsic::dx_wave_prefix_sum:
106:   case Intrinsic::dx_wave_prefix_usum:
107:   case Intrinsic::dx_wave_prefix_product:
108:   case Intrinsic::dx_wave_prefix_uproduct:
109:     // Quad Op Variants
110:   case Intrinsic::dx_quad_read_across_x:
111:   case Intrinsic::dx_quad_read_across_y:
112:     return true;
113:   }
114: }
115:
116: static bool checkDoubleExtensionOps(Intrinsic::ID IID) {
117:   switch (IID) {
118:   default:
119:     return false;
120:   case Intrinsic::fma:
```
- EN: This range implements operational logic in helpers such as checkWaveOps, checkDoubleExtensionOps, translating backend policy into executable code.
- CN: 这一段实现了 checkWaveOps、checkDoubleExtensionOps 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:     return true;
122:   }
123: }
124:
125: static bool isOptimizationDisabled(const Module &M) {
126:   const StringRef Key = "dx.disable_optimizations";
127:   if (auto *Flag = mdconst::extract_or_null<ConstantInt>(M.getModuleFlag(Key)))
128:     return Flag->getValue().getBoolValue();
129:   return false;
130: }
131:
132: // Checks to see if the status bit from a load with status
133: // instruction is ever extracted. If it is, the module needs
134: // to have the TiledResources shader flag set.
135: bool checkIfStatusIsExtracted(const IntrinsicInst &II) {
136:   [[maybe_unused]] Intrinsic::ID IID = II.getIntrinsicID();
137:   assert(IID == Intrinsic::dx_resource_load_typedbuffer ||
138:          IID == Intrinsic::dx_resource_load_rawbuffer &&
139:              "unexpected intrinsic ID");
140:   for (const User *U : II.users()) {
141:     if (const ExtractValueInst *EVI = dyn_cast<ExtractValueInst>(U)) {
142:       // Resource load operations return a {result, status} pair.
143:       // Check if we extract the status
144:       if (EVI->getNumIndices() == 1 && EVI->getIndices()[0] == 1)
145:         return true;
146:     }
147:   }
148:
149:   return false;
150: }
151:
152: /// Update the shader flags mask based on the given instruction.
153: /// \param CSF Shader flags mask to update.
154: /// \param I Instruction to check.
155: void ModuleShaderFlags::updateFunctionFlags(ComputedShaderFlags &CSF,
156:                                             const Instruction &I,
157:                                             DXILResourceTypeMap &DRTM,
158:                                             const ModuleMetadataInfo &MMDI) {
159:   if (!CSF.Doubles)
160:     CSF.Doubles = I.getType()->getScalarType()->isDoubleTy();
161:
162:   if (!CSF.Doubles) {
163:     for (const Value *Op : I.operands()) {
164:       if (Op->getType()->getScalarType()->isDoubleTy()) {
165:         CSF.Doubles = true;
166:         break;
167:       }
168:     }
169:   }
170:
171:   if (CSF.Doubles) {
172:     switch (I.getOpcode()) {
173:     case Instruction::FDiv:
174:     case Instruction::UIToFP:
175:     case Instruction::SIToFP:
176:     case Instruction::FPToUI:
177:     case Instruction::FPToSI:
178:       CSF.DX11_1_DoubleExtensions = true;
179:       break;
180:     }
```
- EN: This range implements operational logic in helpers such as isOptimizationDisabled, getValue, checkIfStatusIsExtracted, getIntrinsicID, translating backend policy into executable code.
- CN: 这一段实现了 isOptimizationDisabled、getValue、checkIfStatusIsExtracted、getIntrinsicID 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   }
182:
183:   if (!CSF.LowPrecisionPresent)
184:     CSF.LowPrecisionPresent = I.getType()->getScalarType()->isIntegerTy(16) ||
185:                               I.getType()->getScalarType()->isHalfTy();
186:
187:   if (!CSF.LowPrecisionPresent) {
188:     for (const Value *Op : I.operands()) {
189:       if (Op->getType()->getScalarType()->isIntegerTy(16) ||
190:           Op->getType()->getScalarType()->isHalfTy()) {
191:         CSF.LowPrecisionPresent = true;
192:         break;
193:       }
194:     }
195:   }
196:
197:   if (CSF.LowPrecisionPresent) {
198:     if (CSF.NativeLowPrecisionMode)
199:       CSF.NativeLowPrecision = true;
200:     else
201:       CSF.MinimumPrecision = true;
202:   }
203:
204:   if (!CSF.Int64Ops)
205:     CSF.Int64Ops = I.getType()->getScalarType()->isIntegerTy(64);
206:
207:   if (!CSF.Int64Ops && !isa<LifetimeIntrinsic>(&I)) {
208:     for (const Value *Op : I.operands()) {
209:       if (Op->getType()->getScalarType()->isIntegerTy(64)) {
210:         CSF.Int64Ops = true;
211:         break;
212:       }
213:     }
214:   }
215:
216:   if (const auto *II = dyn_cast<IntrinsicInst>(&I)) {
217:     switch (II->getIntrinsicID()) {
218:     default:
219:       break;
220:     case Intrinsic::dx_resource_handlefrombinding: {
221:       dxil::ResourceTypeInfo &RTI = DRTM[cast<TargetExtType>(II->getType())];
222:
223:       // Set ResMayNotAlias if DXIL validator version >= 1.8 and the function
224:       // uses UAVs
225:       if (!CSF.ResMayNotAlias && CanSetResMayNotAlias &&
226:           MMDI.ValidatorVersion >= VersionTuple(1, 8) && RTI.isUAV())
227:         CSF.ResMayNotAlias = true;
228:
229:       switch (RTI.getResourceKind()) {
230:       case dxil::ResourceKind::StructuredBuffer:
231:       case dxil::ResourceKind::RawBuffer:
232:         CSF.EnableRawAndStructuredBuffers = true;
233:         break;
234:       default:
235:         break;
236:       }
237:       break;
238:     }
239:     case Intrinsic::dx_resource_load_typedbuffer: {
240:       dxil::ResourceTypeInfo &RTI =
```
- EN: This range implements operational logic in helpers such as getType, VersionTuple, translating backend policy into executable code.
- CN: 这一段实现了 getType、VersionTuple 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:           DRTM[cast<TargetExtType>(II->getArgOperand(0)->getType())];
242:       if (RTI.isTyped() && RTI.isUAV())
243:         CSF.TypedUAVLoadAdditionalFormats |= RTI.getTyped().ElementCount > 1;
244:       if (!CSF.TiledResources && checkIfStatusIsExtracted(*II))
245:         CSF.TiledResources = true;
246:       break;
247:     }
248:     case Intrinsic::dx_resource_load_rawbuffer: {
249:       if (!CSF.TiledResources && checkIfStatusIsExtracted(*II))
250:         CSF.TiledResources = true;
251:       break;
252:     }
253:     }
254:   }
255:   // Handle call instructions
256:   if (auto *CI = dyn_cast<CallInst>(&I)) {
257:     const Function *CF = CI->getCalledFunction();
258:     // Merge-in shader flags mask of the called function in the current module
259:     if (FunctionFlags.contains(CF))
260:       CSF.merge(FunctionFlags[CF]);
261:
262:     CSF.DX11_1_DoubleExtensions |=
263:         checkDoubleExtensionOps(CI->getIntrinsicID());
264:     CSF.WaveOps |= checkWaveOps(CI->getIntrinsicID());
265:   }
266: }
267:
268: /// Set shader flags that apply to all functions within the module
269: ComputedShaderFlags
270: ModuleShaderFlags::gatherGlobalModuleFlags(const Module &M,
271:                                            const DXILResourceMap &DRM,
272:                                            const ModuleMetadataInfo &MMDI) {
273:
274:   ComputedShaderFlags CSF;
275:
276:   CSF.DisableOptimizations = isOptimizationDisabled(M);
277:
278:   CSF.UAVsAtEveryStage = hasUAVsAtEveryStage(DRM, MMDI);
279:
280:   // Set the Max64UAVs flag if the number of UAVs is > 8
281:   uint32_t NumUAVs = 0;
282:   for (auto &UAV : DRM.uavs())
283:     if (MMDI.ValidatorVersion < VersionTuple(1, 6)) {
284:       NumUAVs++;
285:     } else { // MMDI.ValidatorVersion >= VersionTuple(1, 6)
286:       uint32_t Size = UAV.getBinding().Size;
287:       uint32_t NewNum = NumUAVs + (Size == 0 ? ~0U : Size);
288:       if (NewNum < NumUAVs)
289:         NewNum = ~0U;
290:       NumUAVs = NewNum;
291:     }
292:   if (NumUAVs > 8)
293:     CSF.Max64UAVs = true;
294:
295:   // Set the module flag that enables native low-precision execution mode.
296:   // NativeLowPrecisionMode can only be set when the command line option
297:   // -enable-16bit-types is provided. This is indicated by the dx.nativelowprec
298:   // module flag being set
299:   // This flag is needed even if the module does not use 16-bit types because a
300:   // corresponding debug module may include 16-bit types, and tools that use the
```
- EN: This range implements operational logic in helpers such as getCalledFunction, merge, checkDoubleExtensionOps, checkWaveOps, translating backend policy into executable code.
- CN: 这一段实现了 getCalledFunction、merge、checkDoubleExtensionOps、checkWaveOps 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:   // debug module may expect it to have the same flags as the original
302:   if (auto *NativeLowPrec = mdconst::extract_or_null<ConstantInt>(
303:           M.getModuleFlag("dx.nativelowprec")))
304:     if (MMDI.ShaderModelVersion >= VersionTuple(6, 2))
305:       CSF.NativeLowPrecisionMode = NativeLowPrec->getValue().getBoolValue();
306:
307:   // Set ResMayNotAlias to true if DXIL validator version < 1.8 and there
308:   // are UAVs present globally.
309:   if (CanSetResMayNotAlias && MMDI.ValidatorVersion < VersionTuple(1, 8))
310:     CSF.ResMayNotAlias = !DRM.uavs().empty();
311:
312:   // The command line option -all-resources-bound will set the
313:   // dx.allresourcesbound module flag to 1
314:   if (auto *AllResourcesBound = mdconst::extract_or_null<ConstantInt>(
315:           M.getModuleFlag("dx.allresourcesbound")))
316:     if (AllResourcesBound->getValue().getBoolValue())
317:       CSF.AllResourcesBound = true;
318:
319:   return CSF;
320: }
321:
322: /// Construct ModuleShaderFlags for module Module M
323: void ModuleShaderFlags::initialize(Module &M, DXILResourceTypeMap &DRTM,
324:                                    const DXILResourceMap &DRM,
325:                                    const ModuleMetadataInfo &MMDI) {
326:
327:   CanSetResMayNotAlias = MMDI.DXILVersion >= VersionTuple(1, 7);
328:   // The command line option -res-may-alias will set the dx.resmayalias module
329:   // flag to 1, thereby disabling the ability to set the ResMayNotAlias flag
330:   if (auto *ResMayAlias = mdconst::extract_or_null<ConstantInt>(
331:           M.getModuleFlag("dx.resmayalias")))
332:     if (ResMayAlias->getValue().getBoolValue())
333:       CanSetResMayNotAlias = false;
334:
335:   ComputedShaderFlags GlobalSFMask = gatherGlobalModuleFlags(M, DRM, MMDI);
336:
337:   CallGraph CG(M);
338:
339:   // Compute Shader Flags Mask for all functions using post-order visit of SCC
340:   // of the call graph.
341:   for (scc_iterator<CallGraph *> SCCI = scc_begin(&CG); !SCCI.isAtEnd();
342:        ++SCCI) {
343:     const std::vector<CallGraphNode *> &CurSCC = *SCCI;
344:
345:     // Union of shader masks of all functions in CurSCC
346:     ComputedShaderFlags SCCSF;
347:     // List of functions in CurSCC that are neither external nor declarations
348:     // and hence whose flags are collected
349:     SmallVector<Function *> CurSCCFuncs;
350:     for (CallGraphNode *CGN : CurSCC) {
351:       Function *F = CGN->getFunction();
352:       if (!F)
353:         continue;
354:
355:       if (F->isDeclaration()) {
356:         assert(!F->getName().starts_with("dx.op.") &&
357:                "DXIL Shader Flag analysis should not be run post-lowering.");
358:         continue;
359:       }
360:
```
- EN: This range implements operational logic in helpers such as getModuleFlag, getValue, uavs, VersionTuple, translating backend policy into executable code.
- CN: 这一段实现了 getModuleFlag、getValue、uavs、VersionTuple 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:       ComputedShaderFlags CSF = GlobalSFMask;
362:       for (const auto &BB : *F)
363:         for (const auto &I : BB)
364:           updateFunctionFlags(CSF, I, DRTM, MMDI);
365:       // Update combined shader flags mask for all functions in this SCC
366:       SCCSF.merge(CSF);
367:
368:       CurSCCFuncs.push_back(F);
369:     }
370:
371:     // Update combined shader flags mask for all functions of the module
372:     CombinedSFMask.merge(SCCSF);
373:
374:     // Shader flags mask of each of the functions in an SCC of the call graph is
375:     // the union of all functions in the SCC. Update shader flags masks of
376:     // functions in CurSCC accordingly. This is trivially true if SCC contains
377:     // one function.
378:     for (Function *F : CurSCCFuncs)
379:       // Merge SCCSF with that of F
380:       FunctionFlags[F].merge(SCCSF);
381:   }
382: }
383:
384: void ComputedShaderFlags::print(raw_ostream &OS) const {
385:   uint64_t FlagVal = (uint64_t) * this;
386:   OS << formatv("; Shader Flags Value: {0:x8}\n;\n", FlagVal);
387:   if (FlagVal == 0)
388:     return;
389:   OS << "; Note: shader requires additional functionality:\n";
390: #define SHADER_FEATURE_FLAG(FeatureBit, DxilModuleNum, FlagName, Str)          \
391:   if (FlagName)                                                                \
392:     (OS << ";").indent(7) << Str << "\n";
393: #include "llvm/BinaryFormat/DXContainerConstants.def"
394:   OS << "; Note: extra DXIL module flags:\n";
395: #define DXIL_MODULE_FLAG(DxilModuleBit, FlagName, Str)                         \
396:   if (FlagName)                                                                \
397:     (OS << ";").indent(7) << Str << "\n";
398: #include "llvm/BinaryFormat/DXContainerConstants.def"
399:   OS << ";\n";
400: }
401:
402: /// Return the shader flags mask of the specified function Func.
403: const ComputedShaderFlags &
404: ModuleShaderFlags::getFunctionFlags(const Function *Func) const {
405:   auto Iter = FunctionFlags.find(Func);
406:   assert((Iter != FunctionFlags.end() && Iter->first == Func) &&
407:          "Get Shader Flags : No Shader Flags Mask exists for function");
408:   return Iter->second;
409: }
410:
411: //===----------------------------------------------------------------------===//
412: // ShaderFlagsAnalysis and ShaderFlagsAnalysisPrinterPass
413:
414: // Provide an explicit template instantiation for the static ID.
415: AnalysisKey ShaderFlagsAnalysis::Key;
416:
417: ModuleShaderFlags ShaderFlagsAnalysis::run(Module &M,
418:                                            ModuleAnalysisManager &AM) {
419:   DXILResourceTypeMap &DRTM = AM.getResult<DXILResourceTypeAnalysis>(M);
420:   DXILResourceMap &DRM = AM.getResult<DXILResourceAnalysis>(M);
```
- EN: This range implements operational logic in helpers such as updateFunctionFlags, merge, push_back, ComputedShaderFlags::print, translating backend policy into executable code.
- CN: 这一段实现了 updateFunctionFlags、merge、push_back、ComputedShaderFlags::print 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-477
```cpp
421:   const ModuleMetadataInfo MMDI = AM.getResult<DXILMetadataAnalysis>(M);
422:
423:   ModuleShaderFlags MSFI;
424:   MSFI.initialize(M, DRTM, DRM, MMDI);
425:
426:   return MSFI;
427: }
428:
429: PreservedAnalyses ShaderFlagsAnalysisPrinter::run(Module &M,
430:                                                   ModuleAnalysisManager &AM) {
431:   const ModuleShaderFlags &FlagsInfo = AM.getResult<ShaderFlagsAnalysis>(M);
432:   // Print description of combined shader flags for all module functions
433:   OS << "; Combined Shader Flags for Module\n";
434:   FlagsInfo.getCombinedFlags().print(OS);
435:   // Print shader flags mask for each of the module functions
436:   OS << "; Shader Flags for Module Functions\n";
437:   for (const auto &F : M.getFunctionList()) {
438:     if (F.isDeclaration())
439:       continue;
440:     const ComputedShaderFlags &SFMask = FlagsInfo.getFunctionFlags(&F);
441:     OS << formatv("; Function {0} : {1:x8}\n;\n", F.getName(),
442:                   (uint64_t)(SFMask));
443:   }
444:
445:   return PreservedAnalyses::all();
446: }
447:
448: //===----------------------------------------------------------------------===//
449: // ShaderFlagsAnalysis and ShaderFlagsAnalysisPrinterPass
450:
451: bool ShaderFlagsAnalysisWrapper::runOnModule(Module &M) {
452:   DXILResourceTypeMap &DRTM =
453:       getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
454:   DXILResourceMap &DRM =
455:       getAnalysis<DXILResourceWrapperPass>().getResourceMap();
456:   const ModuleMetadataInfo MMDI =
457:       getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
458:
459:   MSFI.initialize(M, DRTM, DRM, MMDI);
460:   return false;
461: }
462:
463: void ShaderFlagsAnalysisWrapper::getAnalysisUsage(AnalysisUsage &AU) const {
464:   AU.setPreservesAll();
465:   AU.addRequiredTransitive<DXILResourceTypeWrapperPass>();
466:   AU.addRequiredTransitive<DXILResourceWrapperPass>();
467:   AU.addRequired<DXILMetadataAnalysisWrapperPass>();
468: }
469:
470: char ShaderFlagsAnalysisWrapper::ID = 0;
471:
472: INITIALIZE_PASS_BEGIN(ShaderFlagsAnalysisWrapper, "dx-shader-flag-analysis",
473:                       "DXIL Shader Flag Analysis", true, true)
474: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
475: INITIALIZE_PASS_DEPENDENCY(DXILMetadataAnalysisWrapperPass)
476: INITIALIZE_PASS_END(ShaderFlagsAnalysisWrapper, "dx-shader-flag-analysis",
477:                     "DXIL Shader Flag Analysis", true, true)
```
- EN: This range implements operational logic in helpers such as initialize, getCombinedFlags, getFunctionFlags, PreservedAnalyses::all, translating backend policy into executable code.
- CN: 这一段实现了 initialize、getCombinedFlags、getFunctionFlags、PreservedAnalyses::all 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include VersionTuple, checkWaveOps, checkDoubleExtensionOps, isOptimizationDisabled, getValue, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 VersionTuple, checkWaveOps, checkDoubleExtensionOps, isOptimizationDisabled, getValue，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILShaderFlags.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SCCIterator.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/Analysis/CallGraph.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/IR/Attributes.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Instruction.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicInst.h`
  - `llvm/IR/Intrinsics.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
  - `llvm/Support/FormatVariadic.h`
  - `llvm/Support/raw_ostream.h`
  - `llvm/BinaryFormat/DXContainerConstants.def`
