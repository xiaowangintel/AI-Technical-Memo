# DXILTranslateMetadata.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILTranslateMetadata.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILTranslateMetadata support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILTranslateMetadata.cpp - Pass to emit DXIL metadata -------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILTranslateMetadata.h"
10: #include "DXILRootSignature.h"
11: #include "DXILShaderFlags.h"
12: #include "DirectX.h"
13: #include "llvm/ADT/STLExtras.h"
14: #include "llvm/ADT/SmallVector.h"
15: #include "llvm/ADT/Twine.h"
16: #include "llvm/Analysis/DXILMetadataAnalysis.h"
17: #include "llvm/Analysis/DXILResource.h"
18: #include "llvm/IR/BasicBlock.h"
19: #include "llvm/IR/Constants.h"
20: #include "llvm/IR/DiagnosticInfo.h"
21: #include "llvm/IR/DiagnosticPrinter.h"
22: #include "llvm/IR/Function.h"
23: #include "llvm/IR/IRBuilder.h"
24: #include "llvm/IR/LLVMContext.h"
25: #include "llvm/IR/MDBuilder.h"
26: #include "llvm/IR/Metadata.h"
27: #include "llvm/IR/Module.h"
28: #include "llvm/InitializePasses.h"
29: #include "llvm/Pass.h"
30: #include "llvm/Support/ErrorHandling.h"
31: #include "llvm/Support/VersionTuple.h"
32: #include "llvm/TargetParser/Triple.h"
33: #include <cstdint>
34:
35: using namespace llvm;
36: using namespace llvm::dxil;
37:
38: namespace {
39:
40: /// A simple wrapper of DiagnosticInfo that generates module-level diagnostic
41: /// for the DXILValidateMetadata pass
42: class DiagnosticInfoValidateMD : public DiagnosticInfo {
43: private:
44:   const Twine &Msg;
45:   const Module &Mod;
46:
47: public:
48:   /// \p M is the module for which the diagnostic is being emitted. \p Msg is
49:   /// the message to show. Note that this class does not copy this message, so
50:   /// this reference must be valid for the whole life time of the diagnostic.
51:   DiagnosticInfoValidateMD(const Module &M,
52:                            const Twine &Msg LLVM_LIFETIME_BOUND,
53:                            DiagnosticSeverity Severity = DS_Error)
54:       : DiagnosticInfo(DK_Unsupported, Severity), Msg(Msg), Mod(M) {}
55:
56:   void print(DiagnosticPrinter &DP) const override {
57:     DP << Mod.getName() << ": " << Msg << '\n';
58:   }
59: };
60:
```
- EN: This range defines or declares important types such as DiagnosticInfoValidateMD, DiagnosticInfo, print, shaping the data model used by DXILTranslateMetadata.cpp.
- CN: 这一段定义或声明了 DiagnosticInfoValidateMD、DiagnosticInfo、print 等关键类型，构成 DXILTranslateMetadata.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61: static void reportError(Module &M, Twine Message,
 62:                         DiagnosticSeverity Severity = DS_Error) {
 63:   M.getContext().diagnose(DiagnosticInfoValidateMD(M, Message, Severity));
 64: }
 65:
 66: static void reportLoopError(Module &M, Twine Message,
 67:                             DiagnosticSeverity Severity = DS_Error) {
 68:   reportError(M, Twine("Invalid \"llvm.loop\" metadata: ") + Message, Severity);
 69: }
 70:
 71: enum class EntryPropsTag {
 72:   ShaderFlags = 0,
 73:   GSState,
 74:   DSState,
 75:   HSState,
 76:   NumThreads,
 77:   AutoBindingSpace,
 78:   RayPayloadSize,
 79:   RayAttribSize,
 80:   ShaderKind,
 81:   MSState,
 82:   ASStateTag,
 83:   WaveSize,
 84:   EntryRootSig,
 85:   WaveRange = 23,
 86: };
 87:
 88: } // namespace
 89:
 90: static NamedMDNode *emitResourceMetadata(Module &M, DXILResourceMap &DRM,
 91:                                          DXILResourceTypeMap &DRTM) {
 92:   LLVMContext &Context = M.getContext();
 93:
 94:   for (ResourceInfo &RI : DRM)
 95:     if (!RI.hasSymbol())
 96:       RI.createSymbol(M,
 97:                       DRTM[RI.getHandleTy()].createElementStruct(RI.getName()));
 98:
 99:   SmallVector<Metadata *> SRVs, UAVs, CBufs, Smps;
100:   for (const ResourceInfo &RI : DRM.srvs())
101:     SRVs.push_back(RI.getAsMetadata(M, DRTM[RI.getHandleTy()]));
102:   for (const ResourceInfo &RI : DRM.uavs())
103:     UAVs.push_back(RI.getAsMetadata(M, DRTM[RI.getHandleTy()]));
104:   for (const ResourceInfo &RI : DRM.cbuffers())
105:     CBufs.push_back(RI.getAsMetadata(M, DRTM[RI.getHandleTy()]));
106:   for (const ResourceInfo &RI : DRM.samplers())
107:     Smps.push_back(RI.getAsMetadata(M, DRTM[RI.getHandleTy()]));
108:
109:   Metadata *SRVMD = SRVs.empty() ? nullptr : MDNode::get(Context, SRVs);
110:   Metadata *UAVMD = UAVs.empty() ? nullptr : MDNode::get(Context, UAVs);
111:   Metadata *CBufMD = CBufs.empty() ? nullptr : MDNode::get(Context, CBufs);
112:   Metadata *SmpMD = Smps.empty() ? nullptr : MDNode::get(Context, Smps);
113:
114:   if (DRM.empty())
115:     return nullptr;
116:
117:   NamedMDNode *ResourceMD = M.getOrInsertNamedMetadata("dx.resources");
118:   ResourceMD->addOperand(
119:       MDNode::get(M.getContext(), {SRVMD, UAVMD, CBufMD, SmpMD}));
120:
```
- EN: This range defines or declares important types such as getContext, reportError, EntryPropsTag, getHandleTy, shaping the data model used by DXILTranslateMetadata.cpp.
- CN: 这一段定义或声明了 getContext、reportError、EntryPropsTag、getHandleTy 等关键类型，构成 DXILTranslateMetadata.cpp 使用的数据模型。

### Lines 121-180
```cpp
121:   return ResourceMD;
122: }
123:
124: static StringRef getShortShaderStage(Triple::EnvironmentType Env) {
125:   switch (Env) {
126:   case Triple::Pixel:
127:     return "ps";
128:   case Triple::Vertex:
129:     return "vs";
130:   case Triple::Geometry:
131:     return "gs";
132:   case Triple::Hull:
133:     return "hs";
134:   case Triple::Domain:
135:     return "ds";
136:   case Triple::Compute:
137:     return "cs";
138:   case Triple::Library:
139:     return "lib";
140:   case Triple::Mesh:
141:     return "ms";
142:   case Triple::Amplification:
143:     return "as";
144:   case Triple::RootSignature:
145:     return "rootsig";
146:   default:
147:     break;
148:   }
149:   llvm_unreachable("Unsupported environment for DXIL generation.");
150: }
151:
152: static uint32_t getShaderStage(Triple::EnvironmentType Env) {
153:   return (uint32_t)Env - (uint32_t)llvm::Triple::Pixel;
154: }
155:
156: static SmallVector<Metadata *>
157: getTagValueAsMetadata(EntryPropsTag Tag, uint64_t Value, LLVMContext &Ctx) {
158:   SmallVector<Metadata *> MDVals;
159:   MDVals.emplace_back(ConstantAsMetadata::get(
160:       ConstantInt::get(Type::getInt32Ty(Ctx), static_cast<int>(Tag))));
161:   switch (Tag) {
162:   case EntryPropsTag::ShaderFlags:
163:     MDVals.emplace_back(ConstantAsMetadata::get(
164:         ConstantInt::get(Type::getInt64Ty(Ctx), Value)));
165:     break;
166:   case EntryPropsTag::ShaderKind:
167:     MDVals.emplace_back(ConstantAsMetadata::get(
168:         ConstantInt::get(Type::getInt32Ty(Ctx), Value)));
169:     break;
170:   case EntryPropsTag::GSState:
171:   case EntryPropsTag::DSState:
172:   case EntryPropsTag::HSState:
173:   case EntryPropsTag::NumThreads:
174:   case EntryPropsTag::AutoBindingSpace:
175:   case EntryPropsTag::RayPayloadSize:
176:   case EntryPropsTag::RayAttribSize:
177:   case EntryPropsTag::MSState:
178:   case EntryPropsTag::ASStateTag:
179:   case EntryPropsTag::WaveSize:
180:   case EntryPropsTag::EntryRootSig:
```
- EN: This range implements operational logic in helpers such as getShortShaderStage, llvm_unreachable, getShaderStage, getTagValueAsMetadata, translating backend policy into executable code.
- CN: 这一段实现了 getShortShaderStage、llvm_unreachable、getShaderStage、getTagValueAsMetadata 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   case EntryPropsTag::WaveRange:
182:     llvm_unreachable("NYI: Unhandled entry property tag");
183:   }
184:   return MDVals;
185: }
186:
187: static MDTuple *getEntryPropAsMetadata(Module &M, const EntryProperties &EP,
188:                                        uint64_t EntryShaderFlags,
189:                                        const ModuleMetadataInfo &MMDI) {
190:   SmallVector<Metadata *> MDVals;
191:   LLVMContext &Ctx = EP.Entry->getContext();
192:   if (EntryShaderFlags != 0)
193:     MDVals.append(getTagValueAsMetadata(EntryPropsTag::ShaderFlags,
194:                                         EntryShaderFlags, Ctx));
195:
196:   if (EP.Entry != nullptr) {
197:     // FIXME: support more props.
198:     // See https://github.com/llvm/llvm-project/issues/57948.
199:     // Add shader kind for lib entries.
200:     if (MMDI.ShaderProfile == Triple::EnvironmentType::Library &&
201:         EP.ShaderStage != Triple::EnvironmentType::Library)
202:       MDVals.append(getTagValueAsMetadata(EntryPropsTag::ShaderKind,
203:                                           getShaderStage(EP.ShaderStage), Ctx));
204:
205:     if (EP.ShaderStage == Triple::EnvironmentType::Compute) {
206:       // Handle mandatory "hlsl.numthreads"
207:       MDVals.emplace_back(ConstantAsMetadata::get(ConstantInt::get(
208:           Type::getInt32Ty(Ctx), static_cast<int>(EntryPropsTag::NumThreads))));
209:       Metadata *NumThreadVals[] = {ConstantAsMetadata::get(ConstantInt::get(
210:                                        Type::getInt32Ty(Ctx), EP.NumThreadsX)),
211:                                    ConstantAsMetadata::get(ConstantInt::get(
212:                                        Type::getInt32Ty(Ctx), EP.NumThreadsY)),
213:                                    ConstantAsMetadata::get(ConstantInt::get(
214:                                        Type::getInt32Ty(Ctx), EP.NumThreadsZ))};
215:       MDVals.emplace_back(MDNode::get(Ctx, NumThreadVals));
216:
217:       // Handle optional "hlsl.wavesize". The fields are optionally represented
218:       // if they are non-zero.
219:       if (EP.WaveSizeMin != 0) {
220:         bool IsWaveRange = VersionTuple(6, 8) <= MMDI.ShaderModelVersion;
221:         bool IsWaveSize =
222:             !IsWaveRange && VersionTuple(6, 6) <= MMDI.ShaderModelVersion;
223:
224:         if (!IsWaveRange && !IsWaveSize) {
225:           reportError(M, "Shader model 6.6 or greater is required to specify "
226:                          "the \"hlsl.wavesize\" function attribute");
227:           return nullptr;
228:         }
229:
230:         // A range is being specified if EP.WaveSizeMax != 0
231:         if (EP.WaveSizeMax && !IsWaveRange) {
232:           reportError(
233:               M, "Shader model 6.8 or greater is required to specify "
234:                  "wave size range values of the \"hlsl.wavesize\" function "
235:                  "attribute");
236:           return nullptr;
237:         }
238:
239:         EntryPropsTag Tag =
240:             IsWaveSize ? EntryPropsTag::WaveSize : EntryPropsTag::WaveRange;
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, getContext, getShaderStage, Type::getInt32Ty, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、getContext、getShaderStage、Type::getInt32Ty 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:         MDVals.emplace_back(ConstantAsMetadata::get(
242:             ConstantInt::get(Type::getInt32Ty(Ctx), static_cast<int>(Tag))));
243:
244:         SmallVector<Metadata *> WaveSizeVals = {ConstantAsMetadata::get(
245:             ConstantInt::get(Type::getInt32Ty(Ctx), EP.WaveSizeMin))};
246:         if (IsWaveRange) {
247:           WaveSizeVals.push_back(ConstantAsMetadata::get(
248:               ConstantInt::get(Type::getInt32Ty(Ctx), EP.WaveSizeMax)));
249:           WaveSizeVals.push_back(ConstantAsMetadata::get(
250:               ConstantInt::get(Type::getInt32Ty(Ctx), EP.WaveSizePref)));
251:         }
252:
253:         MDVals.emplace_back(MDNode::get(Ctx, WaveSizeVals));
254:       }
255:     }
256:   }
257:
258:   if (MDVals.empty())
259:     return nullptr;
260:   return MDNode::get(Ctx, MDVals);
261: }
262:
263: static MDTuple *constructEntryMetadata(const Function *EntryFn,
264:                                        MDTuple *Signatures, MDNode *Resources,
265:                                        MDTuple *Properties, LLVMContext &Ctx) {
266:   // Each entry point metadata record specifies:
267:   //  * reference to the entry point function global symbol
268:   //  * unmangled name
269:   //  * list of signatures
270:   //  * list of resources
271:   //  * list of tag-value pairs of shader capabilities and other properties
272:   Metadata *MDVals[5];
273:   MDVals[0] =
274:       EntryFn ? ValueAsMetadata::get(const_cast<Function *>(EntryFn)) : nullptr;
275:   MDVals[1] = MDString::get(Ctx, EntryFn ? EntryFn->getName() : "");
276:   MDVals[2] = Signatures;
277:   MDVals[3] = Resources;
278:   MDVals[4] = Properties;
279:   return MDNode::get(Ctx, MDVals);
280: }
281:
282: static MDTuple *emitEntryMD(Module &M, const EntryProperties &EP,
283:                             MDTuple *Signatures, MDNode *MDResources,
284:                             const uint64_t EntryShaderFlags,
285:                             const ModuleMetadataInfo &MMDI) {
286:   MDTuple *Properties = getEntryPropAsMetadata(M, EP, EntryShaderFlags, MMDI);
287:   return constructEntryMetadata(EP.Entry, Signatures, MDResources, Properties,
288:                                 EP.Entry->getContext());
289: }
290:
291: static void emitValidatorVersionMD(Module &M, const ModuleMetadataInfo &MMDI) {
292:   if (MMDI.ValidatorVersion.empty())
293:     return;
294:
295:   LLVMContext &Ctx = M.getContext();
296:   IRBuilder<> IRB(Ctx);
297:   Metadata *MDVals[2];
298:   MDVals[0] =
299:       ConstantAsMetadata::get(IRB.getInt32(MMDI.ValidatorVersion.getMajor()));
300:   MDVals[1] = ConstantAsMetadata::get(
```
- EN: This range implements operational logic in helpers such as ConstantInt::get, emplace_back, MDNode::get, MDString::get, translating backend policy into executable code.
- CN: 这一段实现了 ConstantInt::get、emplace_back、MDNode::get、MDString::get 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:       IRB.getInt32(MMDI.ValidatorVersion.getMinor().value_or(0)));
302:   NamedMDNode *ValVerNode = M.getOrInsertNamedMetadata("dx.valver");
303:   // Set validator version obtained from DXIL Metadata Analysis pass
304:   ValVerNode->clearOperands();
305:   ValVerNode->addOperand(MDNode::get(Ctx, MDVals));
306: }
307:
308: static void emitShaderModelVersionMD(Module &M,
309:                                      const ModuleMetadataInfo &MMDI) {
310:   LLVMContext &Ctx = M.getContext();
311:   IRBuilder<> IRB(Ctx);
312:   Metadata *SMVals[3];
313:   VersionTuple SM = MMDI.ShaderModelVersion;
314:   SMVals[0] = MDString::get(Ctx, getShortShaderStage(MMDI.ShaderProfile));
315:   SMVals[1] = ConstantAsMetadata::get(IRB.getInt32(SM.getMajor()));
316:   SMVals[2] = ConstantAsMetadata::get(IRB.getInt32(SM.getMinor().value_or(0)));
317:   NamedMDNode *SMMDNode = M.getOrInsertNamedMetadata("dx.shaderModel");
318:   SMMDNode->addOperand(MDNode::get(Ctx, SMVals));
319: }
320:
321: static void emitDXILVersionTupleMD(Module &M, const ModuleMetadataInfo &MMDI) {
322:   LLVMContext &Ctx = M.getContext();
323:   IRBuilder<> IRB(Ctx);
324:   VersionTuple DXILVer = MMDI.DXILVersion;
325:   Metadata *DXILVals[2];
326:   DXILVals[0] = ConstantAsMetadata::get(IRB.getInt32(DXILVer.getMajor()));
327:   DXILVals[1] =
328:       ConstantAsMetadata::get(IRB.getInt32(DXILVer.getMinor().value_or(0)));
329:   NamedMDNode *DXILVerMDNode = M.getOrInsertNamedMetadata("dx.version");
330:   DXILVerMDNode->addOperand(MDNode::get(Ctx, DXILVals));
331: }
332:
333: static MDTuple *emitTopLevelLibraryNode(Module &M, MDNode *RMD,
334:                                         uint64_t ShaderFlags) {
335:   LLVMContext &Ctx = M.getContext();
336:   MDTuple *Properties = nullptr;
337:   if (ShaderFlags != 0) {
338:     SmallVector<Metadata *> MDVals;
339:     MDVals.append(
340:         getTagValueAsMetadata(EntryPropsTag::ShaderFlags, ShaderFlags, Ctx));
341:     Properties = MDNode::get(Ctx, MDVals);
342:   }
343:   // Library has an entry metadata with resource table metadata and all other
344:   // MDNodes as null.
345:   return constructEntryMetadata(nullptr, nullptr, RMD, Properties, Ctx);
346: }
347:
348: static void translateBranchMetadata(Module &M, Instruction *BBTerminatorInst) {
349:   MDNode *HlslControlFlowMD =
350:       BBTerminatorInst->getMetadata("hlsl.controlflow.hint");
351:
352:   if (!HlslControlFlowMD)
353:     return;
354:
355:   assert(HlslControlFlowMD->getNumOperands() == 2 &&
356:          "invalid operands for hlsl.controlflow.hint");
357:
358:   MDBuilder MDHelper(M.getContext());
359:
360:   llvm::Metadata *HintsStr = MDHelper.createString("dx.controlflow.hints");
```
- EN: This range implements operational logic in helpers such as getInt32, getOrInsertNamedMetadata, clearOperands, addOperand, translating backend policy into executable code.
- CN: 这一段实现了 getInt32、getOrInsertNamedMetadata、clearOperands、addOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:   llvm::Metadata *HintsValue = MDHelper.createConstant(
362:       mdconst::extract<ConstantInt>(HlslControlFlowMD->getOperand(1)));
363:
364:   MDNode *MDNode = llvm::MDNode::get(M.getContext(), {HintsStr, HintsValue});
365:
366:   BBTerminatorInst->setMetadata("dx.controlflow.hints", MDNode);
367:   BBTerminatorInst->setMetadata("hlsl.controlflow.hint", nullptr);
368: }
369:
370: // Determines if the metadata node will be compatible with DXIL's loop metadata
371: // representation.
372: //
373: // Reports an error for compatible metadata that is ill-formed.
374: static bool isLoopMDCompatible(Module &M, Metadata *MD) {
375:   // DXIL only accepts the following loop hints:
376:   std::array<StringLiteral, 3> ValidHintNames = {"llvm.loop.unroll.count",
377:                                                  "llvm.loop.unroll.disable",
378:                                                  "llvm.loop.unroll.full"};
379:
380:   MDNode *HintMD = dyn_cast<MDNode>(MD);
381:   if (!HintMD || HintMD->getNumOperands() == 0)
382:     return false;
383:
384:   auto *HintStr = dyn_cast<MDString>(HintMD->getOperand(0));
385:   if (!HintStr)
386:     return false;
387:
388:   if (!llvm::is_contained(ValidHintNames, HintStr->getString()))
389:     return false;
390:
391:   auto ValidCountNode = [](MDNode *CountMD) -> bool {
392:     if (CountMD->getNumOperands() == 2)
393:       if (auto *Count = dyn_cast<ConstantAsMetadata>(CountMD->getOperand(1)))
394:         if (isa<ConstantInt>(Count->getValue()))
395:           return true;
396:     return false;
397:   };
398:
399:   if (HintStr->getString() == "llvm.loop.unroll.count") {
400:     if (!ValidCountNode(HintMD)) {
401:       reportLoopError(M, "\"llvm.loop.unroll.count\" must have 2 operands and "
402:                          "the second must be a constant integer");
403:       return false;
404:     }
405:   } else if (HintMD->getNumOperands() != 1) {
406:     reportLoopError(
407:         M, "\"llvm.loop.unroll.disable\" and \"llvm.loop.unroll.full\" "
408:            "must be provided as a single operand");
409:     return false;
410:   }
411:
412:   return true;
413: }
414:
415: static void translateLoopMetadata(Module &M, Instruction *I, MDNode *BaseMD) {
416:   // A distinct node has the self-referential form: !0 = !{ !0, ... }
417:   auto IsDistinctNode = [](MDNode *Node) -> bool {
418:     return Node && Node->getNumOperands() != 0 && Node == Node->getOperand(0);
419:   };
420:
```
- EN: This range implements operational logic in helpers such as getOperand, llvm::MDNode::get, setMetadata, isLoopMDCompatible, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、llvm::MDNode::get、setMetadata、isLoopMDCompatible 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:   // Set metadata to null to remove empty/ill-formed metadata from instruction
422:   if (BaseMD->getNumOperands() == 0 || !IsDistinctNode(BaseMD))
423:     return I->setMetadata("llvm.loop", nullptr);
424:
425:   // It is valid to have a chain of self-refential loop metadata nodes, as
426:   // below. We will collapse these into just one when we reconstruct the
427:   // metadata.
428:   //
429:   // Eg:
430:   // !0 = !{!0, !1}
431:   // !1 = !{!1, !2}
432:   // !2 = !{!"llvm.loop.unroll.disable"}
433:   //
434:   // So, traverse down a potential self-referential chain
435:   while (1 < BaseMD->getNumOperands() &&
436:          IsDistinctNode(dyn_cast<MDNode>(BaseMD->getOperand(1))))
437:     BaseMD = dyn_cast<MDNode>(BaseMD->getOperand(1));
438:
439:   // To reconstruct a distinct node we create a temporary node that we will
440:   // then update to create a self-reference.
441:   llvm::TempMDTuple TempNode = llvm::MDNode::getTemporary(M.getContext(), {});
442:   SmallVector<Metadata *> CompatibleOperands = {TempNode.get()};
443:
444:   // Iterate and reconstruct the metadata nodes that contains any hints,
445:   // stripping any unrecognized metadata.
446:   ArrayRef<MDOperand> Operands = BaseMD->operands();
447:   for (auto &Op : Operands.drop_front())
448:     if (isLoopMDCompatible(M, Op.get()))
449:       CompatibleOperands.push_back(Op.get());
450:
451:   if (2 < CompatibleOperands.size())
452:     reportLoopError(M, "Provided conflicting hints");
453:
454:   MDNode *CompatibleLoopMD = MDNode::get(M.getContext(), CompatibleOperands);
455:   TempNode->replaceAllUsesWith(CompatibleLoopMD);
456:
457:   I->setMetadata("llvm.loop", CompatibleLoopMD);
458: }
459:
460: using InstructionMDList = std::array<unsigned, 7>;
461:
462: static InstructionMDList getCompatibleInstructionMDs(llvm::Module &M) {
463:   return {
464:       M.getMDKindID("dx.nonuniform"),    M.getMDKindID("dx.controlflow.hints"),
465:       M.getMDKindID("dx.precise"),       llvm::LLVMContext::MD_range,
466:       llvm::LLVMContext::MD_alias_scope, llvm::LLVMContext::MD_noalias,
467:       M.getMDKindID("llvm.loop")};
468: }
469:
470: static void translateInstructionMetadata(Module &M) {
471:   // construct allowlist of valid metadata node kinds
472:   InstructionMDList DXILCompatibleMDs = getCompatibleInstructionMDs(M);
473:   unsigned char MDLoopKind = M.getContext().getMDKindID("llvm.loop");
474:
475:   for (Function &F : M) {
476:     for (BasicBlock &BB : F) {
477:       // This needs to be done first so that "hlsl.controlflow.hints" isn't
478:       // removed in the allow-list below
479:       if (auto *I = BB.getTerminator())
480:         translateBranchMetadata(M, I);
```
- EN: This range implements operational logic in helpers such as setMetadata, IsDistinctNode, getOperand, llvm::MDNode::getTemporary, translating backend policy into executable code.
- CN: 这一段实现了 setMetadata、IsDistinctNode、getOperand、llvm::MDNode::getTemporary 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:
482:       for (auto &I : make_early_inc_range(BB)) {
483:         if (isa<UncondBrInst, CondBrInst>(I))
484:           if (MDNode *LoopMD = I.getMetadata(MDLoopKind))
485:             translateLoopMetadata(M, &I, LoopMD);
486:         I.dropUnknownNonDebugMetadata(DXILCompatibleMDs);
487:       }
488:     }
489:   }
490: }
491:
492: static void cleanModuleFlags(Module &M) {
493:   NamedMDNode *MDFlags = M.getModuleFlagsMetadata();
494:   if (!MDFlags)
495:     return;
496:
497:   SmallVector<llvm::Module::ModuleFlagEntry> FlagEntries;
498:   M.getModuleFlagsMetadata(FlagEntries);
499:   bool Updated = false;
500:   for (auto &Flag : FlagEntries) {
501:     // llvm 3.7 only supports behavior up to AppendUnique.
502:     if (Flag.Behavior <= Module::ModFlagBehavior::AppendUnique)
503:       continue;
504:     Flag.Behavior = Module::ModFlagBehavior::Warning;
505:     Updated = true;
506:   }
507:
508:   if (!Updated)
509:     return;
510:
511:   MDFlags->eraseFromParent();
512:
513:   for (auto &Flag : FlagEntries)
514:     M.addModuleFlag(Flag.Behavior, Flag.Key->getString(), Flag.Val);
515: }
516:
517: using GlobalMDList = std::array<StringLiteral, 7>;
518:
519: // The following are compatible with DXIL but not emit with clang, they can
520: // be added when applicable:
521: // dx.typeAnnotations, dx.viewIDState, dx.dxrPayloadAnnotations
522: static GlobalMDList CompatibleNamedModuleMDs = {
523:     "llvm.ident",     "llvm.module.flags", "dx.resources",   "dx.valver",
524:     "dx.shaderModel", "dx.version",        "dx.entryPoints",
525: };
526:
527: static void translateGlobalMetadata(Module &M, DXILResourceMap &DRM,
528:                                     DXILResourceTypeMap &DRTM,
529:                                     const ModuleShaderFlags &ShaderFlags,
530:                                     const ModuleMetadataInfo &MMDI) {
531:   LLVMContext &Ctx = M.getContext();
532:   IRBuilder<> IRB(Ctx);
533:   SmallVector<MDNode *> EntryFnMDNodes;
534:
535:   emitValidatorVersionMD(M, MMDI);
536:   emitShaderModelVersionMD(M, MMDI);
537:   emitDXILVersionTupleMD(M, MMDI);
538:   NamedMDNode *NamedResourceMD = emitResourceMetadata(M, DRM, DRTM);
539:   auto *ResourceMD =
540:       (NamedResourceMD != nullptr) ? NamedResourceMD->getOperand(0) : nullptr;
```
- EN: This range implements operational logic in helpers such as translateLoopMetadata, dropUnknownNonDebugMetadata, cleanModuleFlags, getModuleFlagsMetadata, translating backend policy into executable code.
- CN: 这一段实现了 translateLoopMetadata、dropUnknownNonDebugMetadata、cleanModuleFlags、getModuleFlagsMetadata 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:   // FIXME: Add support to construct Signatures
542:   // See https://github.com/llvm/llvm-project/issues/57928
543:   MDTuple *Signatures = nullptr;
544:
545:   if (MMDI.ShaderProfile == Triple::EnvironmentType::Library) {
546:     // Get the combined shader flag mask of all functions in the library to be
547:     // used as shader flags mask value associated with top-level library entry
548:     // metadata.
549:     uint64_t CombinedMask = ShaderFlags.getCombinedFlags();
550:     EntryFnMDNodes.emplace_back(
551:         emitTopLevelLibraryNode(M, ResourceMD, CombinedMask));
552:   } else if (1 < MMDI.EntryPropertyVec.size())
553:     reportError(M, "Non-library shader: One and only one entry expected");
554:
555:   for (const EntryProperties &EntryProp : MMDI.EntryPropertyVec) {
556:     uint64_t EntryShaderFlags = 0;
557:     if (MMDI.ShaderProfile != Triple::EnvironmentType::Library) {
558:       EntryShaderFlags = ShaderFlags.getFunctionFlags(EntryProp.Entry);
559:       if (EntryProp.ShaderStage != MMDI.ShaderProfile)
560:         reportError(
561:             M, "Shader stage '" +
562:                    Twine(getShortShaderStage(EntryProp.ShaderStage)) +
563:                    "' for entry '" + Twine(EntryProp.Entry->getName()) +
564:                    "' different from specified target profile '" +
565:                    Twine(Triple::getEnvironmentTypeName(MMDI.ShaderProfile) +
566:                          "'"));
567:     }
568:     EntryFnMDNodes.emplace_back(emitEntryMD(
569:         M, EntryProp, Signatures, ResourceMD, EntryShaderFlags, MMDI));
570:   }
571:
572:   NamedMDNode *EntryPointsNamedMD =
573:       M.getOrInsertNamedMetadata("dx.entryPoints");
574:   for (auto *Entry : EntryFnMDNodes)
575:     EntryPointsNamedMD->addOperand(Entry);
576:
577:   cleanModuleFlags(M);
578:
579:   // Finally, strip all module metadata that is not explicitly specified in the
580:   // allow-list
581:   SmallVector<NamedMDNode *> ToStrip;
582:
583:   for (NamedMDNode &NamedMD : M.named_metadata())
584:     if (!NamedMD.getName().starts_with("llvm.dbg.") &&
585:         !llvm::is_contained(CompatibleNamedModuleMDs, NamedMD.getName()))
586:       ToStrip.push_back(&NamedMD);
587:
588:   for (NamedMDNode *NamedMD : ToStrip)
589:     NamedMD->eraseFromParent();
590: }
591:
592: PreservedAnalyses DXILTranslateMetadata::run(Module &M,
593:                                              ModuleAnalysisManager &MAM) {
594:   DXILResourceMap &DRM = MAM.getResult<DXILResourceAnalysis>(M);
595:   DXILResourceTypeMap &DRTM = MAM.getResult<DXILResourceTypeAnalysis>(M);
596:   const ModuleShaderFlags &ShaderFlags = MAM.getResult<ShaderFlagsAnalysis>(M);
597:   const dxil::ModuleMetadataInfo MMDI = MAM.getResult<DXILMetadataAnalysis>(M);
598:
599:   translateGlobalMetadata(M, DRM, DRTM, ShaderFlags, MMDI);
600:   translateInstructionMetadata(M);
```
- EN: This range implements operational logic in helpers such as getCombinedFlags, emitTopLevelLibraryNode, reportError, getFunctionFlags, translating backend policy into executable code.
- CN: 这一段实现了 getCombinedFlags、emitTopLevelLibraryNode、reportError、getFunctionFlags 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-647
```cpp
601:
602:   return PreservedAnalyses::all();
603: }
604:
605: void DXILTranslateMetadataLegacy::getAnalysisUsage(AnalysisUsage &AU) const {
606:   AU.addRequired<DXILResourceTypeWrapperPass>();
607:   AU.addRequired<DXILResourceWrapperPass>();
608:   AU.addRequired<ShaderFlagsAnalysisWrapper>();
609:   AU.addRequired<DXILMetadataAnalysisWrapperPass>();
610:   AU.addRequired<RootSignatureAnalysisWrapper>();
611:
612:   AU.addPreserved<DXILMetadataAnalysisWrapperPass>();
613:   AU.addPreserved<DXILResourceBindingWrapperPass>();
614:   AU.addPreserved<DXILResourceWrapperPass>();
615:   AU.addPreserved<RootSignatureAnalysisWrapper>();
616:   AU.addPreserved<ShaderFlagsAnalysisWrapper>();
617: }
618:
619: bool DXILTranslateMetadataLegacy::runOnModule(Module &M) {
620:   DXILResourceMap &DRM =
621:       getAnalysis<DXILResourceWrapperPass>().getResourceMap();
622:   DXILResourceTypeMap &DRTM =
623:       getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
624:   const ModuleShaderFlags &ShaderFlags =
625:       getAnalysis<ShaderFlagsAnalysisWrapper>().getShaderFlags();
626:   dxil::ModuleMetadataInfo MMDI =
627:       getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
628:
629:   translateGlobalMetadata(M, DRM, DRTM, ShaderFlags, MMDI);
630:   translateInstructionMetadata(M);
631:   return true;
632: }
633:
634: char DXILTranslateMetadataLegacy::ID = 0;
635:
636: ModulePass *llvm::createDXILTranslateMetadataLegacyPass() {
637:   return new DXILTranslateMetadataLegacy();
638: }
639:
640: INITIALIZE_PASS_BEGIN(DXILTranslateMetadataLegacy, "dxil-translate-metadata",
641:                       "DXIL Translate Metadata", false, false)
642: INITIALIZE_PASS_DEPENDENCY(DXILResourceWrapperPass)
643: INITIALIZE_PASS_DEPENDENCY(ShaderFlagsAnalysisWrapper)
644: INITIALIZE_PASS_DEPENDENCY(RootSignatureAnalysisWrapper)
645: INITIALIZE_PASS_DEPENDENCY(DXILMetadataAnalysisWrapperPass)
646: INITIALIZE_PASS_END(DXILTranslateMetadataLegacy, "dxil-translate-metadata",
647:                     "DXIL Translate Metadata", false, false)
```
- EN: This range implements operational logic in helpers such as PreservedAnalyses::all, DXILTranslateMetadataLegacy::getAnalysisUsage, DXILTranslateMetadataLegacy::runOnModule, getResourceMap, translating backend policy into executable code.
- CN: 这一段实现了 PreservedAnalyses::all、DXILTranslateMetadataLegacy::getAnalysisUsage、DXILTranslateMetadataLegacy::runOnModule、getResourceMap 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include DiagnosticInfoValidateMD, DiagnosticInfo, print, getContext, reportError, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DiagnosticInfoValidateMD, DiagnosticInfo, print, getContext, reportError，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILTranslateMetadata.h`
  - `DXILRootSignature.h`
  - `DXILShaderFlags.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/Twine.h`
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/IR/BasicBlock.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/DiagnosticPrinter.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/IRBuilder.h`
  - `llvm/IR/LLVMContext.h`
  - `llvm/IR/MDBuilder.h`
  - `llvm/IR/Metadata.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
- System/standard headers / 系统或标准头文件:
  - `cstdint`
