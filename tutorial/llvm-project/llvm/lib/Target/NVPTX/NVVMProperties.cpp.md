# NVVMProperties.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVVMProperties.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains NVVM attribute and metadata query utilities.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- NVVMProperties.cpp - NVVM annotation utilities ---------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains NVVM attribute and metadata query utilities.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVVMProperties.h"
14: #include "llvm/ADT/ArrayRef.h"
15: #include "llvm/ADT/STLExtras.h"
16: #include "llvm/ADT/StringRef.h"
17: #include "llvm/IR/Argument.h"
18: #include "llvm/IR/Constants.h"
19: #include "llvm/IR/Function.h"
20: #include "llvm/IR/GlobalValue.h"
21: #include "llvm/IR/GlobalVariable.h"
22: #include "llvm/IR/Instructions.h"
23: #include "llvm/IR/Metadata.h"
24: #include "llvm/IR/Module.h"
25: #include "llvm/Support/ModRef.h"
26: #include "llvm/Support/Mutex.h"
27: #include "llvm/Support/NVVMAttributes.h"
28: #include <functional>
29: #include <map>
30: #include <mutex>
31: #include <numeric>
32: #include <string>
33: #include <vector>
34:
35: namespace llvm {
36:
37: namespace {
38: using AnnotationValues = std::map<std::string, std::vector<unsigned>>;
39: using AnnotationMap = std::map<const GlobalValue *, AnnotationValues>;
40:
41: struct AnnotationCache {
42:   sys::Mutex Lock;
43:   std::map<const Module *, AnnotationMap> Cache;
44: };
45:
46: AnnotationCache &getAnnotationCache() {
47:   static AnnotationCache AC;
48:   return AC;
49: }
50: } // namespace
51:
52: // TODO: Replace these legacy nvvm.annotations metadata names with proper
53: // function/parameter attributes (like the NVVMAttr:: constants).
54: namespace NVVMMetadata {
55: constexpr StringLiteral Texture("texture");
56: constexpr StringLiteral Surface("surface");
57: constexpr StringLiteral Sampler("sampler");
58: constexpr StringLiteral ReadOnlyImage("rdoimage");
59: constexpr StringLiteral WriteOnlyImage("wroimage");
60: constexpr StringLiteral ReadWriteImage("rdwrimage");
```
- EN: This range defines or declares important types such as AnnotationCache, getAnnotationCache, Texture, Surface, shaping the data model used by NVVMProperties.cpp.
- CN: 这一段定义或声明了 AnnotationCache、getAnnotationCache、Texture、Surface 等关键类型，构成 NVVMProperties.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61: constexpr StringLiteral Managed("managed");
 62: } // namespace NVVMMetadata
 63:
 64: void clearAnnotationCache(const Module *Mod) {
 65:   auto &AC = getAnnotationCache();
 66:   std::lock_guard<sys::Mutex> Guard(AC.Lock);
 67:   AC.Cache.erase(Mod);
 68: }
 69:
 70: static void cacheAnnotationFromMD(const MDNode *MetadataNode,
 71:                                   AnnotationValues &RetVal) {
 72:   auto &AC = getAnnotationCache();
 73:   std::lock_guard<sys::Mutex> Guard(AC.Lock);
 74:   assert(MetadataNode && "Invalid mdnode for annotation");
 75:   assert((MetadataNode->getNumOperands() % 2) == 1 &&
 76:          "Invalid number of operands");
 77:   // start index = 1, to skip the global variable key
 78:   // increment = 2, to skip the value for each property-value pairs
 79:   for (unsigned I = 1, E = MetadataNode->getNumOperands(); I != E; I += 2) {
 80:     const MDString *Prop = dyn_cast<MDString>(MetadataNode->getOperand(I));
 81:     assert(Prop && "Annotation property not a string");
 82:     std::string Key = Prop->getString().str();
 83:
 84:     if (ConstantInt *Val = mdconst::dyn_extract<ConstantInt>(
 85:             MetadataNode->getOperand(I + 1))) {
 86:       RetVal[Key].push_back(Val->getZExtValue());
 87:     } else {
 88:       llvm_unreachable("Value operand not a constant int");
 89:     }
 90:   }
 91: }
 92:
 93: static void cacheAnnotationFromMD(const Module *M, const GlobalValue *GV) {
 94:   auto &AC = getAnnotationCache();
 95:   std::lock_guard<sys::Mutex> Guard(AC.Lock);
 96:   NamedMDNode *NMD = M->getNamedMetadata("nvvm.annotations");
 97:   if (!NMD)
 98:     return;
 99:
100:   AnnotationValues Tmp;
101:   for (unsigned I = 0, E = NMD->getNumOperands(); I != E; ++I) {
102:     const MDNode *Elem = NMD->getOperand(I);
103:     GlobalValue *Entity =
104:         mdconst::dyn_extract_or_null<GlobalValue>(Elem->getOperand(0));
105:     // entity may be null due to DCE
106:     if (!Entity || Entity != GV)
107:       continue;
108:
109:     cacheAnnotationFromMD(Elem, Tmp);
110:   }
111:
112:   if (Tmp.empty())
113:     return;
114:
115:   AC.Cache[M][GV] = std::move(Tmp);
116: }
117:
118: static std::optional<unsigned> findOneNVVMAnnotation(const GlobalValue *GV,
119:                                                      StringRef Prop) {
120:   auto &AC = getAnnotationCache();
```
- EN: This range implements operational logic in helpers such as Managed, clearAnnotationCache, getAnnotationCache, Guard, translating backend policy into executable code.
- CN: 这一段实现了 Managed、clearAnnotationCache、getAnnotationCache、Guard 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   std::lock_guard<sys::Mutex> Guard(AC.Lock);
122:   const Module *M = GV->getParent();
123:   auto ACIt = AC.Cache.find(M);
124:   if (ACIt == AC.Cache.end())
125:     cacheAnnotationFromMD(M, GV);
126:   else if (ACIt->second.find(GV) == ACIt->second.end())
127:     cacheAnnotationFromMD(M, GV);
128:
129:   auto &KVP = AC.Cache[M][GV];
130:   auto It = KVP.find(Prop.str());
131:   if (It == KVP.end())
132:     return std::nullopt;
133:   return It->second[0];
134: }
135:
136: static bool findAllNVVMAnnotation(const GlobalValue *GV, StringRef Prop,
137:                                   std::vector<unsigned> &RetVal) {
138:   auto &AC = getAnnotationCache();
139:   std::lock_guard<sys::Mutex> Guard(AC.Lock);
140:   const Module *M = GV->getParent();
141:   auto ACIt = AC.Cache.find(M);
142:   if (ACIt == AC.Cache.end())
143:     cacheAnnotationFromMD(M, GV);
144:   else if (ACIt->second.find(GV) == ACIt->second.end())
145:     cacheAnnotationFromMD(M, GV);
146:
147:   auto &KVP = AC.Cache[M][GV];
148:   auto It = KVP.find(Prop.str());
149:   if (It == KVP.end())
150:     return false;
151:   RetVal = It->second;
152:   return true;
153: }
154:
155: static bool globalHasNVVMAnnotation(const Value &V, StringRef Prop) {
156:   if (const auto *GV = dyn_cast<GlobalValue>(&V))
157:     if (const auto Annot = findOneNVVMAnnotation(GV, Prop)) {
158:       assert((*Annot == 1) && "Unexpected annotation on a symbol");
159:       return true;
160:     }
161:
162:   return false;
163: }
164:
165: static bool argHasNVVMAnnotation(const Value &Val, StringRef Annotation) {
166:   if (const auto *Arg = dyn_cast<Argument>(&Val)) {
167:     std::vector<unsigned> Annot;
168:     if (findAllNVVMAnnotation(Arg->getParent(), Annotation, Annot) &&
169:         is_contained(Annot, Arg->getArgNo()))
170:       return true;
171:   }
172:   return false;
173: }
174:
175: static std::optional<unsigned> getFnAttrParsedInt(const Function &F,
176:                                                   StringRef Attr) {
177:   return F.hasFnAttribute(Attr)
178:              ? std::optional(F.getFnAttributeAsParsedInteger(Attr))
179:              : std::nullopt;
180: }
```
- EN: This range implements operational logic in helpers such as Guard, getParent, find, cacheAnnotationFromMD, translating backend policy into executable code.
- CN: 这一段实现了 Guard、getParent、find、cacheAnnotationFromMD 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:
182: static SmallVector<unsigned, 3> getFnAttrParsedVector(const Function &F,
183:                                                       StringRef Attr) {
184:   SmallVector<unsigned, 3> V;
185:   auto &Ctx = F.getContext();
186:
187:   if (F.hasFnAttribute(Attr)) {
188:     // We expect the attribute value to be of the form "x[,y[,z]]", where x, y,
189:     // and z are unsigned values.
190:     StringRef S = F.getFnAttribute(Attr).getValueAsString();
191:     for (unsigned I = 0; I < 3 && !S.empty(); I++) {
192:       auto [First, Rest] = S.split(",");
193:       unsigned IntVal;
194:       if (First.trim().getAsInteger(0, IntVal))
195:         Ctx.emitError("can't parse integer attribute " + First + " in " + Attr);
196:
197:       V.push_back(IntVal);
198:       S = Rest;
199:     }
200:   }
201:   return V;
202: }
203:
204: static std::optional<uint64_t> getVectorProduct(ArrayRef<unsigned> V) {
205:   if (V.empty())
206:     return std::nullopt;
207:
208:   return std::accumulate(V.begin(), V.end(), uint64_t(1),
209:                          std::multiplies<uint64_t>{});
210: }
211:
212: PTXOpaqueType getPTXOpaqueType(const GlobalVariable &GV) {
213:   if (findOneNVVMAnnotation(&GV, NVVMMetadata::Texture))
214:     return PTXOpaqueType::Texture;
215:   if (findOneNVVMAnnotation(&GV, NVVMMetadata::Surface))
216:     return PTXOpaqueType::Surface;
217:   if (findOneNVVMAnnotation(&GV, NVVMMetadata::Sampler))
218:     return PTXOpaqueType::Sampler;
219:   return PTXOpaqueType::None;
220: }
221:
222: PTXOpaqueType getPTXOpaqueType(const Argument &Arg) {
223:   if (argHasNVVMAnnotation(Arg, NVVMMetadata::Sampler))
224:     return PTXOpaqueType::Sampler;
225:   if (argHasNVVMAnnotation(Arg, NVVMMetadata::ReadOnlyImage))
226:     return PTXOpaqueType::Texture;
227:   if (argHasNVVMAnnotation(Arg, NVVMMetadata::WriteOnlyImage) ||
228:       argHasNVVMAnnotation(Arg, NVVMMetadata::ReadWriteImage))
229:     return PTXOpaqueType::Surface;
230:   return PTXOpaqueType::None;
231: }
232:
233: PTXOpaqueType getPTXOpaqueType(const Value &V) {
234:   if (const auto *GV = dyn_cast<GlobalVariable>(&V))
235:     return getPTXOpaqueType(*GV);
236:   if (const auto *Arg = dyn_cast<Argument>(&V))
237:     return getPTXOpaqueType(*Arg);
238:   return PTXOpaqueType::None;
239: }
240:
```
- EN: This range implements operational logic in helpers such as getContext, getFnAttribute, empty, split, translating backend policy into executable code.
- CN: 这一段实现了 getContext、getFnAttribute、empty、split 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241: bool isManaged(const Value &V) {
242:   return globalHasNVVMAnnotation(V, NVVMMetadata::Managed);
243: }
244:
245: SmallVector<unsigned, 3> getMaxNTID(const Function &F) {
246:   return getFnAttrParsedVector(F, NVVMAttr::MaxNTID);
247: }
248:
249: SmallVector<unsigned, 3> getReqNTID(const Function &F) {
250:   return getFnAttrParsedVector(F, NVVMAttr::ReqNTID);
251: }
252:
253: SmallVector<unsigned, 3> getClusterDim(const Function &F) {
254:   return getFnAttrParsedVector(F, NVVMAttr::ClusterDim);
255: }
256:
257: std::optional<uint64_t> getOverallMaxNTID(const Function &F) {
258:   // Note: The semantics here are a bit strange. The PTX ISA states the
259:   // following (11.4.2. Performance-Tuning Directives: .maxntid):
260:   //
261:   //  Note that this directive guarantees that the total number of threads does
262:   //  not exceed the maximum, but does not guarantee that the limit in any
263:   //  particular dimension is not exceeded.
264:   return getVectorProduct(getMaxNTID(F));
265: }
266:
267: std::optional<uint64_t> getOverallReqNTID(const Function &F) {
268:   // Note: The semantics here are a bit strange. See getOverallMaxNTID.
269:   return getVectorProduct(getReqNTID(F));
270: }
271:
272: std::optional<uint64_t> getOverallClusterRank(const Function &F) {
273:   // maxclusterrank and cluster_dim are mutually exclusive.
274:   if (const auto ClusterRank = getMaxClusterRank(F))
275:     return ClusterRank;
276:
277:   // Note: The semantics here are a bit strange. See getOverallMaxNTID.
278:   return getVectorProduct(getClusterDim(F));
279: }
280:
281: std::optional<unsigned> getMaxClusterRank(const Function &F) {
282:   return getFnAttrParsedInt(F, NVVMAttr::MaxClusterRank);
283: }
284:
285: std::optional<unsigned> getMinCTASm(const Function &F) {
286:   return getFnAttrParsedInt(F, NVVMAttr::MinCTASm);
287: }
288:
289: std::optional<unsigned> getMaxNReg(const Function &F) {
290:   return getFnAttrParsedInt(F, NVVMAttr::MaxNReg);
291: }
292:
293: bool hasBlocksAreClusters(const Function &F) {
294:   return F.hasFnAttribute(NVVMAttr::BlocksAreClusters);
295: }
296:
297: bool isParamGridConstant(const Argument &Arg) {
298:   assert(isKernelFunction(*Arg.getParent()) &&
299:          "only kernel arguments can be grid_constant");
300:
```
- EN: This range implements operational logic in helpers such as isManaged, globalHasNVVMAnnotation, getMaxNTID, getFnAttrParsedVector, translating backend policy into executable code.
- CN: 这一段实现了 isManaged、globalHasNVVMAnnotation、getMaxNTID、getFnAttrParsedVector 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-342
```cpp
301:   if (!Arg.hasByValAttr())
302:     return false;
303:
304:   // Lowering an argument as a grid_constant violates the byval semantics (and
305:   // the C++ API) by reusing the same memory location for the argument across
306:   // multiple threads. If an argument doesn't read memory and its address is not
307:   // captured (its address is not compared with any value), then the tweak of
308:   // the C++ API and byval semantics is unobservable by the program and we can
309:   // lower the arg as a grid_constant.
310:   if (Arg.onlyReadsMemory()) {
311:     const auto CI = Arg.getAttributes().getCaptureInfo();
312:     if (!capturesAddress(CI) && !capturesFullProvenance(CI))
313:       return true;
314:   }
315:
316:   // "grid_constant" counts argument indices starting from 1
317:   return Arg.hasAttribute(NVVMAttr::GridConstant);
318: }
319:
320: MaybeAlign getAlign(const CallInst &I, unsigned Index) {
321:   // First check the alignstack metadata.
322:   if (MaybeAlign StackAlign =
323:           I.getAttributes().getAttributes(Index).getStackAlignment())
324:     return StackAlign;
325:
326:   // If that is missing, check the legacy nvvm metadata.
327:   if (MDNode *AlignNode = I.getMetadata("callalign")) {
328:     for (int I = 0, N = AlignNode->getNumOperands(); I < N; I++) {
329:       if (const auto *CI =
330:               mdconst::dyn_extract<ConstantInt>(AlignNode->getOperand(I))) {
331:         unsigned V = CI->getZExtValue();
332:         if ((V >> 16) == Index)
333:           return Align(V & 0xFFFF);
334:         if ((V >> 16) > Index)
335:           return std::nullopt;
336:       }
337:     }
338:   }
339:   return std::nullopt;
340: }
341:
342: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as getAttributes, hasAttribute, getAlign, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getAttributes、hasAttribute、getAlign、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include AnnotationCache, getAnnotationCache, Texture, Surface, Sampler, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 AnnotationCache, getAnnotationCache, Texture, Surface, Sampler，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVVMProperties.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/ArrayRef.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/IR/Argument.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/GlobalValue.h`
  - `llvm/IR/GlobalVariable.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/Metadata.h`
  - `llvm/IR/Module.h`
  - `llvm/Support/ModRef.h`
  - `llvm/Support/Mutex.h`
  - `llvm/Support/NVVMAttributes.h`
- System/standard headers / 系统或标准头文件:
  - `functional`
  - `map`
  - `mutex`
  - `numeric`
  - `string`
  - `vector`
