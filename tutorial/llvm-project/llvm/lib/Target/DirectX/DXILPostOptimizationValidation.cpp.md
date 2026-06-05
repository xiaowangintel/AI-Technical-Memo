# DXILPostOptimizationValidation.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILPostOptimizationValidation.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILPostOptimizationValidation support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILPostOptimizationValidation.cpp - Opt DXIL validation ----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILPostOptimizationValidation.h"
10: #include "DXILRootSignature.h"
11: #include "DXILShaderFlags.h"
12: #include "DirectX.h"
13: #include "llvm/ADT/SmallString.h"
14: #include "llvm/Analysis/DXILMetadataAnalysis.h"
15: #include "llvm/Analysis/DXILResource.h"
16: #include "llvm/IR/DiagnosticInfo.h"
17: #include "llvm/IR/Instructions.h"
18: #include "llvm/IR/IntrinsicsDirectX.h"
19: #include "llvm/IR/Module.h"
20: #include "llvm/InitializePasses.h"
21: #include "llvm/Support/DXILABI.h"
22:
23: #define DEBUG_TYPE "dxil-post-optimization-validation"
24:
25: using namespace llvm;
26: using namespace llvm::dxil;
27:
28: static ResourceClass toResourceClass(dxbc::RootParameterType Type) {
29:   using namespace dxbc;
30:   switch (Type) {
31:   case RootParameterType::Constants32Bit:
32:     return ResourceClass::CBuffer;
33:   case RootParameterType::SRV:
34:     return ResourceClass::SRV;
35:   case RootParameterType::UAV:
36:     return ResourceClass::UAV;
37:   case RootParameterType::CBV:
38:     return ResourceClass::CBuffer;
39:   case dxbc::RootParameterType::DescriptorTable:
40:     llvm_unreachable("DescriptorTable is not convertible to ResourceClass");
41:   }
42:   llvm_unreachable("Unknown RootParameterType");
43: }
44:
45: static void reportInvalidDirection(Module &M, DXILResourceMap &DRM) {
46:   for (const auto &UAV : DRM.uavs()) {
47:     if (UAV.CounterDirection != ResourceCounterDirection::Invalid)
48:       continue;
49:
50:     CallInst *ResourceHandle = nullptr;
51:     for (CallInst *MaybeHandle : DRM.calls()) {
52:       if (*DRM.find(MaybeHandle) == UAV) {
53:         ResourceHandle = MaybeHandle;
54:         break;
55:       }
56:     }
57:
58:     StringRef Message = "RWStructuredBuffers may increment or decrement their "
59:                         "counters, but not both.";
60:     for (const auto &U : ResourceHandle->users()) {
```
- EN: This range implements operational logic in helpers such as toResourceClass, llvm_unreachable, reportInvalidDirection, translating backend policy into executable code.
- CN: 这一段实现了 toResourceClass、llvm_unreachable、reportInvalidDirection 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:       const CallInst *CI = dyn_cast<CallInst>(U);
 62:       if (!CI && CI->getIntrinsicID() != Intrinsic::dx_resource_updatecounter)
 63:         continue;
 64:
 65:       M.getContext().diagnose(DiagnosticInfoGenericWithLoc(
 66:           Message, *CI->getFunction(), CI->getDebugLoc()));
 67:     }
 68:   }
 69: }
 70:
 71: static void reportOverlappingError(Module &M, ResourceInfo R1,
 72:                                    ResourceInfo R2) {
 73:   SmallString<128> Message;
 74:   raw_svector_ostream OS(Message);
 75:   OS << "resource " << R1.getName() << " at register "
 76:      << R1.getBinding().LowerBound << " overlaps with resource " << R2.getName()
 77:      << " at register " << R2.getBinding().LowerBound << " in space "
 78:      << R2.getBinding().Space;
 79:   M.getContext().diagnose(DiagnosticInfoGeneric(Message));
 80: }
 81:
 82: static void reportOverlappingBinding(Module &M, DXILResourceMap &DRM) {
 83:   [[maybe_unused]] bool ErrorFound = false;
 84:   for (const auto &ResList :
 85:        {DRM.srvs(), DRM.uavs(), DRM.cbuffers(), DRM.samplers()}) {
 86:     if (ResList.empty())
 87:       continue;
 88:     const ResourceInfo *PrevRI = &*ResList.begin();
 89:     for (auto *I = ResList.begin() + 1; I != ResList.end(); ++I) {
 90:       const ResourceInfo *CurrentRI = &*I;
 91:       const ResourceInfo *RI = CurrentRI;
 92:       while (RI != ResList.end() &&
 93:              PrevRI->getBinding().overlapsWith(RI->getBinding())) {
 94:         reportOverlappingError(M, *PrevRI, *RI);
 95:         ErrorFound = true;
 96:         RI++;
 97:       }
 98:       PrevRI = CurrentRI;
 99:     }
100:   }
101:   assert(ErrorFound && "this function should be called only when if "
102:                        "DXILResourceBindingInfo::hasOverlapingBinding() is "
103:                        "true, yet no overlapping binding was found");
104: }
105:
106: static void reportInvalidHandleTyError(Module &M, ResourceClass RC,
107:                                        ResourceInfo::ResourceBinding Binding) {
108:   SmallString<160> Message;
109:   raw_svector_ostream OS(Message);
110:   StringRef RCName = getResourceClassName(RC);
111:   OS << RCName << " at register " << Binding.LowerBound << " and space "
112:      << Binding.Space << " is bound to a texture or typed buffer. " << RCName
113:      << " root descriptors can only be Raw or Structured buffers.";
114:   M.getContext().diagnose(DiagnosticInfoGeneric(Message));
115: }
116:
117: static void reportOverlappingRegisters(Module &M, const llvm::hlsl::Binding &R1,
118:                                        const llvm::hlsl::Binding &R2) {
119:   SmallString<128> Message;
120:
```
- EN: This range implements operational logic in helpers such as getFunction, OS, getBinding, getContext, translating backend policy into executable code.
- CN: 这一段实现了 getFunction、OS、getBinding、getContext 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   raw_svector_ostream OS(Message);
122:   OS << "resource " << getResourceClassName(R1.RC) << " (space=" << R1.Space
123:      << ", registers=[" << R1.LowerBound << ", " << R1.UpperBound
124:      << "]) overlaps with resource " << getResourceClassName(R2.RC)
125:      << " (space=" << R2.Space << ", registers=[" << R2.LowerBound << ", "
126:      << R2.UpperBound << "])";
127:   M.getContext().diagnose(DiagnosticInfoGeneric(Message));
128: }
129:
130: static void
131: reportRegNotBound(Module &M, ResourceClass Class,
132:                   const llvm::dxil::ResourceInfo::ResourceBinding &Unbound) {
133:   SmallString<128> Message;
134:   raw_svector_ostream OS(Message);
135:   OS << getResourceClassName(Class) << " register " << Unbound.LowerBound
136:      << " in space " << Unbound.Space
137:      << " does not have a binding in the Root Signature";
138:   M.getContext().diagnose(DiagnosticInfoGeneric(Message));
139: }
140:
141: static dxbc::ShaderVisibility
142: tripleToVisibility(llvm::Triple::EnvironmentType ET) {
143:   switch (ET) {
144:   case Triple::Pixel:
145:     return dxbc::ShaderVisibility::Pixel;
146:   case Triple::Vertex:
147:     return dxbc::ShaderVisibility::Vertex;
148:   case Triple::Geometry:
149:     return dxbc::ShaderVisibility::Geometry;
150:   case Triple::Hull:
151:     return dxbc::ShaderVisibility::Hull;
152:   case Triple::Domain:
153:     return dxbc::ShaderVisibility::Domain;
154:   case Triple::Mesh:
155:     return dxbc::ShaderVisibility::Mesh;
156:   case Triple::Compute:
157:     return dxbc::ShaderVisibility::All;
158:   default:
159:     llvm_unreachable("Invalid triple to shader stage conversion");
160:   }
161: }
162:
163: static void reportIfDeniedShaderStageAccess(Module &M,
164:                                             const dxbc::RootFlags &Flags,
165:                                             const dxbc::RootFlags &Mask) {
166:   if ((Flags & Mask) != Mask)
167:     return;
168:
169:   SmallString<128> Message;
170:   raw_svector_ostream OS(Message);
171:   OS << "Shader has root bindings but root signature uses a DENY flag to "
172:         "disallow root binding access to the shader stage.";
173:   M.getContext().diagnose(DiagnosticInfoGeneric(Message));
174: }
175:
176: static std::optional<dxbc::RootFlags>
177: getEnvironmentDenyFlagMask(Triple::EnvironmentType ShaderProfile) {
178:   switch (ShaderProfile) {
179:   case Triple::Pixel:
180:     return dxbc::RootFlags::DenyPixelShaderRootAccess;
```
- EN: This range implements operational logic in helpers such as OS, getResourceClassName, getContext, tripleToVisibility, translating backend policy into executable code.
- CN: 这一段实现了 OS、getResourceClassName、getContext、tripleToVisibility 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:   case Triple::Vertex:
182:     return dxbc::RootFlags::DenyVertexShaderRootAccess;
183:   case Triple::Geometry:
184:     return dxbc::RootFlags::DenyGeometryShaderRootAccess;
185:   case Triple::Hull:
186:     return dxbc::RootFlags::DenyHullShaderRootAccess;
187:   case Triple::Domain:
188:     return dxbc::RootFlags::DenyDomainShaderRootAccess;
189:   case Triple::Mesh:
190:     return dxbc::RootFlags::DenyMeshShaderRootAccess;
191:   case Triple::Amplification:
192:     return dxbc::RootFlags::DenyAmplificationShaderRootAccess;
193:   default:
194:     return std::nullopt;
195:   }
196: }
197:
198: static void validateRootSignature(Module &M,
199:                                   const mcdxbc::RootSignatureDesc &RSD,
200:                                   dxil::ModuleMetadataInfo &MMI,
201:                                   DXILResourceMap &DRM,
202:                                   DXILResourceTypeMap &DRTM) {
203:
204:   hlsl::BindingInfoBuilder Builder;
205:   dxbc::ShaderVisibility Visibility = tripleToVisibility(MMI.ShaderProfile);
206:
207:   for (const mcdxbc::RootParameterInfo &ParamInfo : RSD.ParametersContainer) {
208:     dxbc::ShaderVisibility ParamVisibility =
209:         dxbc::ShaderVisibility(ParamInfo.Visibility);
210:     if (ParamVisibility != dxbc::ShaderVisibility::All &&
211:         ParamVisibility != Visibility)
212:       continue;
213:     dxbc::RootParameterType ParamType = dxbc::RootParameterType(ParamInfo.Type);
214:     switch (ParamType) {
215:     case dxbc::RootParameterType::Constants32Bit: {
216:       mcdxbc::RootConstants Const =
217:           RSD.ParametersContainer.getConstant(ParamInfo.Location);
218:       Builder.trackBinding(dxil::ResourceClass::CBuffer, Const.RegisterSpace,
219:                            Const.ShaderRegister, Const.ShaderRegister,
220:                            &ParamInfo);
221:       break;
222:     }
223:
224:     case dxbc::RootParameterType::SRV:
225:     case dxbc::RootParameterType::UAV:
226:     case dxbc::RootParameterType::CBV: {
227:       mcdxbc::RootDescriptor Desc =
228:           RSD.ParametersContainer.getRootDescriptor(ParamInfo.Location);
229:       Builder.trackBinding(toResourceClass(ParamInfo.Type), Desc.RegisterSpace,
230:                            Desc.ShaderRegister, Desc.ShaderRegister,
231:                            &ParamInfo);
232:
233:       break;
234:     }
235:     case dxbc::RootParameterType::DescriptorTable: {
236:       const mcdxbc::DescriptorTable &Table =
237:           RSD.ParametersContainer.getDescriptorTable(ParamInfo.Location);
238:
239:       for (const mcdxbc::DescriptorRange &Range : Table.Ranges) {
240:         uint32_t UpperBound =
```
- EN: This range implements operational logic in helpers such as tripleToVisibility, dxbc::ShaderVisibility, dxbc::RootParameterType, getConstant, translating backend policy into executable code.
- CN: 这一段实现了 tripleToVisibility、dxbc::ShaderVisibility、dxbc::RootParameterType、getConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:             Range.NumDescriptors == ~0U
242:                 ? Range.BaseShaderRegister
243:                 : Range.BaseShaderRegister + Range.NumDescriptors - 1;
244:         Builder.trackBinding(Range.RangeType, Range.RegisterSpace,
245:                              Range.BaseShaderRegister, UpperBound, &ParamInfo);
246:       }
247:       break;
248:     }
249:     }
250:   }
251:
252:   for (const mcdxbc::StaticSampler &S : RSD.StaticSamplers)
253:     Builder.trackBinding(dxil::ResourceClass::Sampler, S.RegisterSpace,
254:                          S.ShaderRegister, S.ShaderRegister, &S);
255:
256:   Builder.calculateBindingInfo(
257:       [&M](const llvm::hlsl::BindingInfoBuilder &Builder,
258:            const llvm::hlsl::Binding &ReportedBinding) {
259:         const llvm::hlsl::Binding &Overlaping =
260:             Builder.findOverlapping(ReportedBinding);
261:         reportOverlappingRegisters(M, ReportedBinding, Overlaping);
262:       });
263:
264:   const hlsl::BoundRegs &BoundRegs = Builder.takeBoundRegs();
265:   bool HasBindings = false;
266:   for (const ResourceInfo &RI : DRM) {
267:     const ResourceInfo::ResourceBinding &Binding = RI.getBinding();
268:     const dxil::ResourceTypeInfo &RTI = DRTM[RI.getHandleTy()];
269:     dxil::ResourceClass RC = RTI.getResourceClass();
270:     dxil::ResourceKind RK = RTI.getResourceKind();
271:
272:     const llvm::hlsl::Binding *Reg =
273:         BoundRegs.findBoundReg(RC, Binding.Space, Binding.LowerBound,
274:                                Binding.LowerBound + Binding.Size - 1);
275:
276:     if (!Reg) {
277:       reportRegNotBound(M, RC, Binding);
278:       continue;
279:     }
280:
281:     const auto *ParamInfo =
282:         static_cast<const mcdxbc::RootParameterInfo *>(Reg->Cookie);
283:
284:     bool IsSRVOrUAV = RC == ResourceClass::SRV || RC == ResourceClass::UAV;
285:     bool IsDescriptorTable =
286:         ParamInfo->Type == dxbc::RootParameterType::DescriptorTable;
287:     bool IsRawOrStructuredBuffer =
288:         RK != ResourceKind::RawBuffer && RK != ResourceKind::StructuredBuffer;
289:     if (IsSRVOrUAV && !IsDescriptorTable && IsRawOrStructuredBuffer) {
290:       reportInvalidHandleTyError(M, RC, Binding);
291:       continue;
292:     }
293:
294:     HasBindings = true;
295:   }
296:
297:   if (!HasBindings)
298:     return;
299:
300:   if (std::optional<dxbc::RootFlags> Mask =
```
- EN: This range implements operational logic in helpers such as findOverlapping, reportOverlappingRegisters, takeBoundRegs, getBinding, translating backend policy into executable code.
- CN: 这一段实现了 findOverlapping、reportOverlappingRegisters、takeBoundRegs、getBinding 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:           getEnvironmentDenyFlagMask(MMI.ShaderProfile))
302:     reportIfDeniedShaderStageAccess(M, dxbc::RootFlags(RSD.Flags), *Mask);
303: }
304:
305: static mcdxbc::RootSignatureDesc *
306: getRootSignature(RootSignatureBindingInfo &RSBI,
307:                  dxil::ModuleMetadataInfo &MMI) {
308:   if (MMI.EntryPropertyVec.size() == 0)
309:     return nullptr;
310:   return RSBI.getDescForFunction(MMI.EntryPropertyVec[0].Entry);
311: }
312:
313: static void reportErrors(Module &M, DXILResourceMap &DRM,
314:                          DXILResourceBindingInfo &DRBI,
315:                          RootSignatureBindingInfo &RSBI,
316:                          dxil::ModuleMetadataInfo &MMI,
317:                          DXILResourceTypeMap &DRTM) {
318:   if (DRM.hasInvalidCounterDirection())
319:     reportInvalidDirection(M, DRM);
320:
321:   if (DRBI.hasOverlappingBinding())
322:     reportOverlappingBinding(M, DRM);
323:
324:   assert(!DRBI.hasImplicitBinding() && "implicit bindings should be handled in "
325:                                        "DXILResourceImplicitBinding pass");
326:
327:   if (mcdxbc::RootSignatureDesc *RSD = getRootSignature(RSBI, MMI))
328:     validateRootSignature(M, *RSD, MMI, DRM, DRTM);
329: }
330:
331: PreservedAnalyses
332: DXILPostOptimizationValidation::run(Module &M, ModuleAnalysisManager &MAM) {
333:   DXILResourceMap &DRM = MAM.getResult<DXILResourceAnalysis>(M);
334:   DXILResourceBindingInfo &DRBI = MAM.getResult<DXILResourceBindingAnalysis>(M);
335:   RootSignatureBindingInfo &RSBI = MAM.getResult<RootSignatureAnalysis>(M);
336:   ModuleMetadataInfo &MMI = MAM.getResult<DXILMetadataAnalysis>(M);
337:   DXILResourceTypeMap &DRTM = MAM.getResult<DXILResourceTypeAnalysis>(M);
338:
339:   reportErrors(M, DRM, DRBI, RSBI, MMI, DRTM);
340:   return PreservedAnalyses::all();
341: }
342:
343: namespace {
344: class DXILPostOptimizationValidationLegacy : public ModulePass {
345: public:
346:   bool runOnModule(Module &M) override {
347:     DXILResourceMap &DRM =
348:         getAnalysis<DXILResourceWrapperPass>().getResourceMap();
349:     DXILResourceBindingInfo &DRBI =
350:         getAnalysis<DXILResourceBindingWrapperPass>().getBindingInfo();
351:     RootSignatureBindingInfo &RSBI =
352:         getAnalysis<RootSignatureAnalysisWrapper>().getRSInfo();
353:     dxil::ModuleMetadataInfo &MMI =
354:         getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
355:     DXILResourceTypeMap &DRTM =
356:         getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
357:
358:     reportErrors(M, DRM, DRBI, RSBI, MMI, DRTM);
359:     return false;
360:   }
```
- EN: This range defines or declares important types such as getEnvironmentDenyFlagMask, reportIfDeniedShaderStageAccess, getDescForFunction, reportInvalidDirection, shaping the data model used by DXILPostOptimizationValidation.cpp.
- CN: 这一段定义或声明了 getEnvironmentDenyFlagMask、reportIfDeniedShaderStageAccess、getDescForFunction、reportInvalidDirection 等关键类型，构成 DXILPostOptimizationValidation.cpp 使用的数据模型。

### Lines 361-396
```cpp
361:   StringRef getPassName() const override {
362:     return "DXIL Post Optimization Validation";
363:   }
364:   DXILPostOptimizationValidationLegacy() : ModulePass(ID) {}
365:
366:   static char ID; // Pass identification.
367:   void getAnalysisUsage(llvm::AnalysisUsage &AU) const override {
368:     AU.addRequired<DXILResourceWrapperPass>();
369:     AU.addRequired<DXILResourceBindingWrapperPass>();
370:     AU.addRequired<DXILMetadataAnalysisWrapperPass>();
371:     AU.addRequired<RootSignatureAnalysisWrapper>();
372:     AU.addRequired<DXILResourceTypeWrapperPass>();
373:     AU.addPreserved<DXILResourceWrapperPass>();
374:     AU.addPreserved<DXILResourceBindingWrapperPass>();
375:     AU.addPreserved<DXILMetadataAnalysisWrapperPass>();
376:     AU.addPreserved<ShaderFlagsAnalysisWrapper>();
377:     AU.addPreserved<RootSignatureAnalysisWrapper>();
378:   }
379: };
380: char DXILPostOptimizationValidationLegacy::ID = 0;
381: } // end anonymous namespace
382:
383: INITIALIZE_PASS_BEGIN(DXILPostOptimizationValidationLegacy, DEBUG_TYPE,
384:                       "DXIL Post Optimization Validation", false, false)
385: INITIALIZE_PASS_DEPENDENCY(DXILResourceBindingWrapperPass)
386: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
387: INITIALIZE_PASS_DEPENDENCY(DXILResourceWrapperPass)
388: INITIALIZE_PASS_DEPENDENCY(DXILMetadataAnalysisWrapperPass)
389: INITIALIZE_PASS_DEPENDENCY(RootSignatureAnalysisWrapper)
390: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
391: INITIALIZE_PASS_END(DXILPostOptimizationValidationLegacy, DEBUG_TYPE,
392:                     "DXIL Post Optimization Validation", false, false)
393:
394: ModulePass *llvm::createDXILPostOptimizationValidationLegacyPass() {
395:   return new DXILPostOptimizationValidationLegacy();
396: }
```
- EN: This range implements operational logic in helpers such as getPassName, DXILPostOptimizationValidationLegacy, getAnalysisUsage, INITIALIZE_PASS_DEPENDENCY, translating backend policy into executable code.
- CN: 这一段实现了 getPassName、DXILPostOptimizationValidationLegacy、getAnalysisUsage、INITIALIZE_PASS_DEPENDENCY 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include toResourceClass, llvm_unreachable, reportInvalidDirection, getFunction, OS, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 toResourceClass, llvm_unreachable, reportInvalidDirection, getFunction, OS，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILPostOptimizationValidation.h`
  - `DXILRootSignature.h`
  - `DXILShaderFlags.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallString.h`
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicsDirectX.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
  - `llvm/Support/DXILABI.h`
