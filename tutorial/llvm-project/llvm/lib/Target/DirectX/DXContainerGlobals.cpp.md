# DXContainerGlobals.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXContainerGlobals.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXContainerGlobals support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXContainerGlobals.cpp - DXContainer global generator pass ---------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // DXContainerGlobalsPass implementation.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "DXILRootSignature.h"
14: #include "DXILShaderFlags.h"
15: #include "DirectX.h"
16: #include "llvm/ADT/SmallVector.h"
17: #include "llvm/ADT/StringExtras.h"
18: #include "llvm/ADT/StringRef.h"
19: #include "llvm/Analysis/DXILMetadataAnalysis.h"
20: #include "llvm/Analysis/DXILResource.h"
21: #include "llvm/BinaryFormat/DXContainer.h"
22: #include "llvm/CodeGen/Passes.h"
23: #include "llvm/IR/Constants.h"
24: #include "llvm/IR/Module.h"
25: #include "llvm/InitializePasses.h"
26: #include "llvm/MC/DXContainerPSVInfo.h"
27: #include "llvm/Pass.h"
28: #include "llvm/Support/MD5.h"
29: #include "llvm/TargetParser/Triple.h"
30: #include "llvm/Transforms/Utils/ModuleUtils.h"
31: #include <cstdint>
32:
33: using namespace llvm;
34: using namespace llvm::dxil;
35: using namespace llvm::mcdxbc;
36:
37: namespace {
38: class DXContainerGlobals : public llvm::ModulePass {
39:
40:   GlobalVariable *buildContainerGlobal(Module &M, Constant *Content,
41:                                        StringRef Name, StringRef SectionName);
42:   GlobalVariable *getFeatureFlags(Module &M);
43:   GlobalVariable *computeShaderHash(Module &M);
44:   GlobalVariable *buildSignature(Module &M, Signature &Sig, StringRef Name,
45:                                  StringRef SectionName);
46:   void addSignature(Module &M, SmallVector<GlobalValue *> &Globals);
47:   void addRootSignature(Module &M, SmallVector<GlobalValue *> &Globals);
48:   void addResourcesForPSV(Module &M, PSVRuntimeInfo &PSV);
49:   void addPipelineStateValidationInfo(Module &M,
50:                                       SmallVector<GlobalValue *> &Globals);
51:
52: public:
53:   static char ID; // Pass identification, replacement for typeid
54:   DXContainerGlobals() : ModulePass(ID) {}
55:
56:   StringRef getPassName() const override {
57:     return "DXContainer Global Emitter";
58:   }
59:
60:   bool runOnModule(Module &M) override;
```
- EN: This range defines or declares important types such as DXContainerGlobals, getFeatureFlags, computeShaderHash, addSignature, shaping the data model used by DXContainerGlobals.cpp.
- CN: 这一段定义或声明了 DXContainerGlobals、getFeatureFlags、computeShaderHash、addSignature 等关键类型，构成 DXContainerGlobals.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:
 62:   void getAnalysisUsage(AnalysisUsage &AU) const override {
 63:     AU.setPreservesAll();
 64:     AU.addRequired<ShaderFlagsAnalysisWrapper>();
 65:     AU.addRequired<RootSignatureAnalysisWrapper>();
 66:     AU.addRequired<DXILMetadataAnalysisWrapperPass>();
 67:     AU.addRequired<DXILResourceTypeWrapperPass>();
 68:     AU.addRequired<DXILResourceWrapperPass>();
 69:   }
 70: };
 71:
 72: } // namespace
 73:
 74: bool DXContainerGlobals::runOnModule(Module &M) {
 75:   llvm::SmallVector<GlobalValue *> Globals;
 76:   Globals.push_back(getFeatureFlags(M));
 77:   Globals.push_back(computeShaderHash(M));
 78:   addSignature(M, Globals);
 79:   addRootSignature(M, Globals);
 80:   addPipelineStateValidationInfo(M, Globals);
 81:   appendToCompilerUsed(M, Globals);
 82:   return true;
 83: }
 84:
 85: GlobalVariable *DXContainerGlobals::getFeatureFlags(Module &M) {
 86:   uint64_t CombinedFeatureFlags = getAnalysis<ShaderFlagsAnalysisWrapper>()
 87:                                       .getShaderFlags()
 88:                                       .getCombinedFlags()
 89:                                       .getFeatureFlags();
 90:
 91:   Constant *FeatureFlagsConstant =
 92:       ConstantInt::get(M.getContext(), APInt(64, CombinedFeatureFlags));
 93:   return buildContainerGlobal(M, FeatureFlagsConstant, "dx.sfi0", "SFI0");
 94: }
 95:
 96: GlobalVariable *DXContainerGlobals::computeShaderHash(Module &M) {
 97:   auto *DXILConstant =
 98:       cast<ConstantDataArray>(M.getNamedGlobal("dx.dxil")->getInitializer());
 99:   MD5 Digest;
100:   Digest.update(DXILConstant->getRawDataValues());
101:   MD5::MD5Result Result = Digest.final();
102:
103:   dxbc::ShaderHash HashData = {0, {0}};
104:   // The Hash's IncludesSource flag gets set whenever the hashed shader includes
105:   // debug information.
106:   if (!M.debug_compile_units().empty())
107:     HashData.Flags = static_cast<uint32_t>(dxbc::HashFlags::IncludesSource);
108:
109:   memcpy(reinterpret_cast<void *>(&HashData.Digest), Result.data(), 16);
110:   if (sys::IsBigEndianHost)
111:     HashData.swapBytes();
112:   StringRef Data(reinterpret_cast<char *>(&HashData), sizeof(dxbc::ShaderHash));
113:
114:   Constant *ModuleConstant =
115:       ConstantDataArray::get(M.getContext(), arrayRefFromStringRef(Data));
116:   return buildContainerGlobal(M, ModuleConstant, "dx.hash", "HASH");
117: }
118:
119: GlobalVariable *DXContainerGlobals::buildContainerGlobal(
120:     Module &M, Constant *Content, StringRef Name, StringRef SectionName) {
```
- EN: This range implements operational logic in helpers such as getAnalysisUsage, setPreservesAll, DXContainerGlobals::runOnModule, push_back, translating backend policy into executable code.
- CN: 这一段实现了 getAnalysisUsage、setPreservesAll、DXContainerGlobals::runOnModule、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   auto *GV = new llvm::GlobalVariable(
122:       M, Content->getType(), true, GlobalValue::PrivateLinkage, Content, Name);
123:   GV->setSection(SectionName);
124:   GV->setAlignment(Align(4));
125:   return GV;
126: }
127:
128: GlobalVariable *DXContainerGlobals::buildSignature(Module &M, Signature &Sig,
129:                                                    StringRef Name,
130:                                                    StringRef SectionName) {
131:   SmallString<256> Data;
132:   raw_svector_ostream OS(Data);
133:   Sig.write(OS);
134:   Constant *Constant =
135:       ConstantDataArray::getString(M.getContext(), Data, /*AddNull*/ false);
136:   return buildContainerGlobal(M, Constant, Name, SectionName);
137: }
138:
139: void DXContainerGlobals::addSignature(Module &M,
140:                                       SmallVector<GlobalValue *> &Globals) {
141:   // FIXME: support graphics shader.
142:   //  see issue https://github.com/llvm/llvm-project/issues/90504.
143:
144:   Signature InputSig;
145:   Globals.emplace_back(buildSignature(M, InputSig, "dx.isg1", "ISG1"));
146:
147:   Signature OutputSig;
148:   Globals.emplace_back(buildSignature(M, OutputSig, "dx.osg1", "OSG1"));
149: }
150:
151: void DXContainerGlobals::addRootSignature(Module &M,
152:                                           SmallVector<GlobalValue *> &Globals) {
153:
154:   dxil::ModuleMetadataInfo &MMI =
155:       getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
156:
157:   // Root Signature in Library don't compile to DXContainer.
158:   if (MMI.ShaderProfile == llvm::Triple::Library)
159:     return;
160:
161:   auto &RSA = getAnalysis<RootSignatureAnalysisWrapper>().getRSInfo();
162:   const Function *EntryFunction = nullptr;
163:
164:   if (MMI.ShaderProfile != llvm::Triple::RootSignature) {
165:     assert(MMI.EntryPropertyVec.size() == 1);
166:     EntryFunction = MMI.EntryPropertyVec[0].Entry;
167:   }
168:
169:   const mcdxbc::RootSignatureDesc *RS = RSA.getDescForFunction(EntryFunction);
170:   if (!RS)
171:     return;
172:
173:   SmallString<256> Data;
174:   raw_svector_ostream OS(Data);
175:
176:   RS->write(OS);
177:
178:   Constant *Constant =
179:       ConstantDataArray::getString(M.getContext(), Data, /*AddNull*/ false);
180:   Globals.emplace_back(buildContainerGlobal(M, Constant, "dx.rts0", "RTS0"));
```
- EN: This range implements operational logic in helpers such as getType, setSection, setAlignment, OS, translating backend policy into executable code.
- CN: 这一段实现了 getType、setSection、setAlignment、OS 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181: }
182:
183: void DXContainerGlobals::addResourcesForPSV(Module &M, PSVRuntimeInfo &PSV) {
184:   const DXILResourceMap &DRM =
185:       getAnalysis<DXILResourceWrapperPass>().getResourceMap();
186:   DXILResourceTypeMap &DRTM =
187:       getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
188:
189:   auto MakeBinding =
190:       [](const dxil::ResourceInfo::ResourceBinding &Binding,
191:          const dxbc::PSV::ResourceType Type, const dxil::ResourceKind Kind,
192:          const dxbc::PSV::ResourceFlags Flags = dxbc::PSV::ResourceFlags()) {
193:         dxbc::PSV::v2::ResourceBindInfo BindInfo;
194:         BindInfo.Type = Type;
195:         BindInfo.LowerBound = Binding.LowerBound;
196:         assert(
197:             (Binding.Size == 0 ||
198:              (uint64_t)Binding.LowerBound + Binding.Size - 1 <= UINT32_MAX) &&
199:             "Resource range is too large");
200:         BindInfo.UpperBound = (Binding.Size == 0)
201:                                   ? UINT32_MAX
202:                                   : Binding.LowerBound + Binding.Size - 1;
203:         BindInfo.Space = Binding.Space;
204:         BindInfo.Kind = static_cast<dxbc::PSV::ResourceKind>(Kind);
205:         BindInfo.Flags = Flags;
206:         return BindInfo;
207:       };
208:
209:   for (const dxil::ResourceInfo &RI : DRM.cbuffers()) {
210:     const dxil::ResourceInfo::ResourceBinding &Binding = RI.getBinding();
211:     PSV.Resources.push_back(MakeBinding(Binding, dxbc::PSV::ResourceType::CBV,
212:                                         dxil::ResourceKind::CBuffer));
213:   }
214:   for (const dxil::ResourceInfo &RI : DRM.samplers()) {
215:     const dxil::ResourceInfo::ResourceBinding &Binding = RI.getBinding();
216:     PSV.Resources.push_back(MakeBinding(Binding,
217:                                         dxbc::PSV::ResourceType::Sampler,
218:                                         dxil::ResourceKind::Sampler));
219:   }
220:   for (const dxil::ResourceInfo &RI : DRM.srvs()) {
221:     const dxil::ResourceInfo::ResourceBinding &Binding = RI.getBinding();
222:
223:     dxil::ResourceTypeInfo &TypeInfo = DRTM[RI.getHandleTy()];
224:     dxbc::PSV::ResourceType ResType;
225:     if (TypeInfo.isStruct())
226:       ResType = dxbc::PSV::ResourceType::SRVStructured;
227:     else if (TypeInfo.isTyped())
228:       ResType = dxbc::PSV::ResourceType::SRVTyped;
229:     else
230:       ResType = dxbc::PSV::ResourceType::SRVRaw;
231:
232:     PSV.Resources.push_back(
233:         MakeBinding(Binding, ResType, TypeInfo.getResourceKind()));
234:   }
235:   for (const dxil::ResourceInfo &RI : DRM.uavs()) {
236:     const dxil::ResourceInfo::ResourceBinding &Binding = RI.getBinding();
237:
238:     dxil::ResourceTypeInfo &TypeInfo = DRTM[RI.getHandleTy()];
239:     dxbc::PSV::ResourceType ResType;
240:     if (RI.hasCounter())
```
- EN: This range implements operational logic in helpers such as DXContainerGlobals::addResourcesForPSV, getResourceMap, getResourceTypeMap, dxbc::PSV::ResourceFlags, translating backend policy into executable code.
- CN: 这一段实现了 DXContainerGlobals::addResourcesForPSV、getResourceMap、getResourceTypeMap、dxbc::PSV::ResourceFlags 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:       ResType = dxbc::PSV::ResourceType::UAVStructuredWithCounter;
242:     else if (TypeInfo.isStruct())
243:       ResType = dxbc::PSV::ResourceType::UAVStructured;
244:     else if (TypeInfo.isTyped())
245:       ResType = dxbc::PSV::ResourceType::UAVTyped;
246:     else
247:       ResType = dxbc::PSV::ResourceType::UAVRaw;
248:
249:     dxbc::PSV::ResourceFlags Flags;
250:     // TODO: Add support for dxbc::PSV::ResourceFlag::UsedByAtomic64, tracking
251:     // with https://github.com/llvm/llvm-project/issues/104392
252:     Flags.Flags = 0u;
253:
254:     PSV.Resources.push_back(
255:         MakeBinding(Binding, ResType, TypeInfo.getResourceKind(), Flags));
256:   }
257: }
258:
259: void DXContainerGlobals::addPipelineStateValidationInfo(
260:     Module &M, SmallVector<GlobalValue *> &Globals) {
261:   SmallString<256> Data;
262:   raw_svector_ostream OS(Data);
263:   PSVRuntimeInfo PSV;
264:   PSV.BaseData.MinimumWaveLaneCount = 0;
265:   PSV.BaseData.MaximumWaveLaneCount = std::numeric_limits<uint32_t>::max();
266:
267:   dxil::ModuleMetadataInfo &MMI =
268:       getAnalysis<DXILMetadataAnalysisWrapperPass>().getModuleMetadata();
269:   assert(MMI.EntryPropertyVec.size() == 1 ||
270:          MMI.ShaderProfile == Triple::Library ||
271:          MMI.ShaderProfile == Triple::RootSignature);
272:   PSV.BaseData.ShaderStage =
273:       static_cast<uint8_t>(MMI.ShaderProfile - Triple::Pixel);
274:
275:   addResourcesForPSV(M, PSV);
276:
277:   // Hardcoded values here to unblock loading the shader into D3D.
278:   //
279:   // TODO: Lots more stuff to do here!
280:   //
281:   // See issue https://github.com/llvm/llvm-project/issues/96674.
282:   switch (MMI.ShaderProfile) {
283:   case Triple::Compute:
284:     PSV.BaseData.NumThreadsX = MMI.EntryPropertyVec[0].NumThreadsX;
285:     PSV.BaseData.NumThreadsY = MMI.EntryPropertyVec[0].NumThreadsY;
286:     PSV.BaseData.NumThreadsZ = MMI.EntryPropertyVec[0].NumThreadsZ;
287:     if (MMI.EntryPropertyVec[0].WaveSizeMin) {
288:       PSV.BaseData.MinimumWaveLaneCount = MMI.EntryPropertyVec[0].WaveSizeMin;
289:       PSV.BaseData.MaximumWaveLaneCount =
290:           MMI.EntryPropertyVec[0].WaveSizeMax
291:               ? MMI.EntryPropertyVec[0].WaveSizeMax
292:               : MMI.EntryPropertyVec[0].WaveSizeMin;
293:     }
294:     break;
295:   default:
296:     break;
297:   }
298:
299:   if (MMI.ShaderProfile != Triple::Library &&
300:       MMI.ShaderProfile != Triple::RootSignature)
```
- EN: This range implements operational logic in helpers such as MakeBinding, OS, max, getModuleMetadata, translating backend policy into executable code.
- CN: 这一段实现了 MakeBinding、OS、max、getModuleMetadata 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-322
```cpp
301:     PSV.EntryName = MMI.EntryPropertyVec[0].Entry->getName();
302:
303:   PSV.finalize(MMI.ShaderProfile);
304:   PSV.write(OS);
305:   Constant *Constant =
306:       ConstantDataArray::getString(M.getContext(), Data, /*AddNull*/ false);
307:   Globals.emplace_back(buildContainerGlobal(M, Constant, "dx.psv0", "PSV0"));
308: }
309:
310: char DXContainerGlobals::ID = 0;
311: INITIALIZE_PASS_BEGIN(DXContainerGlobals, "dxil-globals",
312:                       "DXContainer Global Emitter", false, true)
313: INITIALIZE_PASS_DEPENDENCY(ShaderFlagsAnalysisWrapper)
314: INITIALIZE_PASS_DEPENDENCY(DXILMetadataAnalysisWrapperPass)
315: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
316: INITIALIZE_PASS_DEPENDENCY(DXILResourceWrapperPass)
317: INITIALIZE_PASS_END(DXContainerGlobals, "dxil-globals",
318:                     "DXContainer Global Emitter", false, true)
319:
320: ModulePass *llvm::createDXContainerGlobalsPass() {
321:   return new DXContainerGlobals();
322: }
```
- EN: This range implements operational logic in helpers such as getName, finalize, write, ConstantDataArray::getString, translating backend policy into executable code.
- CN: 这一段实现了 getName、finalize、write、ConstantDataArray::getString 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include DXContainerGlobals, getFeatureFlags, computeShaderHash, addSignature, addRootSignature, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 DXContainerGlobals, getFeatureFlags, computeShaderHash, addSignature, addRootSignature，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILRootSignature.h`
  - `DXILShaderFlags.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringExtras.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/BinaryFormat/DXContainer.h`
  - `llvm/CodeGen/Passes.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
  - `llvm/MC/DXContainerPSVInfo.h`
  - `llvm/Pass.h`
  - `llvm/Support/MD5.h`
  - `llvm/TargetParser/Triple.h`
  - `llvm/Transforms/Utils/ModuleUtils.h`
- System/standard headers / 系统或标准头文件:
  - `cstdint`
