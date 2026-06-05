# DXILPrettyPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILPrettyPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): DXILPrettyPrinter support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===- DXILPrettyPrinter.cpp - Print resources for textual DXIL -----------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8:
 9: #include "DXILPrettyPrinter.h"
10: #include "DirectX.h"
11: #include "llvm/ADT/StringRef.h"
12: #include "llvm/Analysis/DXILResource.h"
13: #include "llvm/IR/PassManager.h"
14: #include "llvm/InitializePasses.h"
15: #include "llvm/Pass.h"
16: #include "llvm/Support/FormatAdapters.h"
17: #include "llvm/Support/FormatVariadic.h"
18: #include "llvm/Support/raw_ostream.h"
19:
20: using namespace llvm;
21:
22: static StringRef getRCName(dxil::ResourceClass RC) {
23:   switch (RC) {
24:   case dxil::ResourceClass::SRV:
25:     return "texture";
26:   case dxil::ResourceClass::UAV:
27:     return "UAV";
28:   case dxil::ResourceClass::CBuffer:
29:     return "cbuffer";
30:   case dxil::ResourceClass::Sampler:
31:     return "sampler";
32:   }
33:   llvm_unreachable("covered switch");
34: }
35:
36: static StringRef getRCPrefix(dxil::ResourceClass RC) {
37:   switch (RC) {
38:   case dxil::ResourceClass::SRV:
39:     return "t";
40:   case dxil::ResourceClass::UAV:
41:     return "u";
42:   case dxil::ResourceClass::CBuffer:
43:     return "cb";
44:   case dxil::ResourceClass::Sampler:
45:     return "s";
46:   }
47:   llvm_unreachable("covered switch");
48: }
49:
50: static StringRef getFormatName(const dxil::ResourceTypeInfo &RI) {
51:   if (RI.isTyped()) {
52:     switch (RI.getTyped().DXILStorageTy) {
53:     case dxil::ElementType::I1:
54:       return "i1";
55:     case dxil::ElementType::I16:
56:       return "i16";
57:     case dxil::ElementType::U16:
58:       return "u16";
59:     case dxil::ElementType::I32:
60:       return "i32";
```
- EN: This range implements operational logic in helpers such as getRCName, llvm_unreachable, getRCPrefix, getFormatName, translating backend policy into executable code.
- CN: 这一段实现了 getRCName、llvm_unreachable、getRCPrefix、getFormatName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:     case dxil::ElementType::U32:
 62:       return "u32";
 63:     case dxil::ElementType::I64:
 64:       return "i64";
 65:     case dxil::ElementType::U64:
 66:       return "u64";
 67:     case dxil::ElementType::F16:
 68:       return "f16";
 69:     case dxil::ElementType::F32:
 70:       return "f32";
 71:     case dxil::ElementType::F64:
 72:       return "f64";
 73:     case dxil::ElementType::SNormF16:
 74:       return "snorm_f16";
 75:     case dxil::ElementType::UNormF16:
 76:       return "unorm_f16";
 77:     case dxil::ElementType::SNormF32:
 78:       return "snorm_f32";
 79:     case dxil::ElementType::UNormF32:
 80:       return "unorm_f32";
 81:     case dxil::ElementType::SNormF64:
 82:       return "snorm_f64";
 83:     case dxil::ElementType::UNormF64:
 84:       return "unorm_f64";
 85:     case dxil::ElementType::PackedS8x32:
 86:       return "p32i8";
 87:     case dxil::ElementType::PackedU8x32:
 88:       return "p32u8";
 89:     case dxil::ElementType::Invalid:
 90:       llvm_unreachable("Invalid ElementType");
 91:     }
 92:     llvm_unreachable("Unhandled ElementType");
 93:   } else if (RI.isStruct())
 94:     return "struct";
 95:   else if (RI.isCBuffer() || RI.isSampler())
 96:     return "NA";
 97:   return "byte";
 98: }
 99:
100: static StringRef getTextureDimName(dxil::ResourceKind RK) {
101:   switch (RK) {
102:   case dxil::ResourceKind::Texture1D:
103:     return "1d";
104:   case dxil::ResourceKind::Texture2D:
105:     return "2d";
106:   case dxil::ResourceKind::Texture3D:
107:     return "3d";
108:   case dxil::ResourceKind::TextureCube:
109:     return "cube";
110:   case dxil::ResourceKind::Texture1DArray:
111:     return "1darray";
112:   case dxil::ResourceKind::Texture2DArray:
113:     return "2darray";
114:   case dxil::ResourceKind::TextureCubeArray:
115:     return "cubearray";
116:   case dxil::ResourceKind::TBuffer:
117:     return "tbuffer";
118:   case dxil::ResourceKind::FeedbackTexture2D:
119:     return "fbtex2d";
120:   case dxil::ResourceKind::FeedbackTexture2DArray:
```
- EN: This range defines or declares important types such as llvm_unreachable, getTextureDimName, shaping the data model used by DXILPrettyPrinter.cpp.
- CN: 这一段定义或声明了 llvm_unreachable、getTextureDimName 等关键类型，构成 DXILPrettyPrinter.cpp 使用的数据模型。

### Lines 121-180
```cpp
121:     return "fbtex2darray";
122:   case dxil::ResourceKind::Texture2DMS:
123:     return "2dMS";
124:   case dxil::ResourceKind::Texture2DMSArray:
125:     return "2darrayMS";
126:   case dxil::ResourceKind::Invalid:
127:   case dxil::ResourceKind::NumEntries:
128:   case dxil::ResourceKind::CBuffer:
129:   case dxil::ResourceKind::RawBuffer:
130:   case dxil::ResourceKind::Sampler:
131:   case dxil::ResourceKind::StructuredBuffer:
132:   case dxil::ResourceKind::TypedBuffer:
133:   case dxil::ResourceKind::RTAccelerationStructure:
134:     llvm_unreachable("Invalid ResourceKind for texture");
135:   }
136:   llvm_unreachable("Unhandled ResourceKind");
137: }
138:
139: namespace {
140: struct FormatResourceDimension
141:     : public llvm::FormatAdapter<const dxil::ResourceTypeInfo &> {
142:   FormatResourceDimension(const dxil::ResourceTypeInfo &RI, bool HasCounter)
143:       : llvm::FormatAdapter<const dxil::ResourceTypeInfo &>(RI),
144:         HasCounter(HasCounter) {}
145:
146:   bool HasCounter;
147:
148:   void format(llvm::raw_ostream &OS, StringRef Style) override {
149:     dxil::ResourceKind RK = Item.getResourceKind();
150:     switch (RK) {
151:     default: {
152:       OS << getTextureDimName(RK);
153:       if (Item.isMultiSample())
154:         OS << Item.getMultiSampleCount();
155:       break;
156:     }
157:     case dxil::ResourceKind::RawBuffer:
158:     case dxil::ResourceKind::StructuredBuffer:
159:       if (!Item.isUAV())
160:         OS << "r/o";
161:       else if (HasCounter)
162:         OS << "r/w+cnt";
163:       else
164:         OS << "r/w";
165:       break;
166:     case dxil::ResourceKind::TypedBuffer:
167:       OS << "buf";
168:       break;
169:     case dxil::ResourceKind::CBuffer:
170:       OS << "NA";
171:       break;
172:     case dxil::ResourceKind::RTAccelerationStructure:
173:       // TODO: dxc would print "ras" here. Can/should this happen?
174:       llvm_unreachable("RTAccelerationStructure printing is not implemented");
175:     }
176:   }
177: };
178:
179: struct FormatBindingID
180:     : public llvm::FormatAdapter<const dxil::ResourceInfo &> {
```
- EN: This range defines or declares important types such as llvm_unreachable, FormatResourceDimension, HasCounter, format, shaping the data model used by DXILPrettyPrinter.cpp.
- CN: 这一段定义或声明了 llvm_unreachable、FormatResourceDimension、HasCounter、format 等关键类型，构成 DXILPrettyPrinter.cpp 使用的数据模型。

### Lines 181-240
```cpp
181:   dxil::ResourceClass RC;
182:
183:   explicit FormatBindingID(const dxil::ResourceInfo &RI,
184:                            const dxil::ResourceTypeInfo &RTI)
185:       : llvm::FormatAdapter<const dxil::ResourceInfo &>(RI),
186:         RC(RTI.getResourceClass()) {}
187:
188:   void format(llvm::raw_ostream &OS, StringRef Style) override {
189:     OS << getRCPrefix(RC).upper() << Item.getBinding().RecordID;
190:   }
191: };
192:
193: struct FormatBindingLocation
194:     : public llvm::FormatAdapter<const dxil::ResourceInfo &> {
195:   dxil::ResourceClass RC;
196:
197:   explicit FormatBindingLocation(const dxil::ResourceInfo &RI,
198:                                  const dxil::ResourceTypeInfo &RTI)
199:       : llvm::FormatAdapter<const dxil::ResourceInfo &>(RI),
200:         RC(RTI.getResourceClass()) {}
201:
202:   void format(llvm::raw_ostream &OS, StringRef Style) override {
203:     const auto &Binding = Item.getBinding();
204:     OS << getRCPrefix(RC) << Binding.LowerBound;
205:     if (Binding.Space)
206:       OS << ",space" << Binding.Space;
207:   }
208: };
209:
210: struct FormatBindingSize
211:     : public llvm::FormatAdapter<const dxil::ResourceInfo &> {
212:   explicit FormatBindingSize(const dxil::ResourceInfo &RI)
213:       : llvm::FormatAdapter<const dxil::ResourceInfo &>(RI) {}
214:
215:   void format(llvm::raw_ostream &OS, StringRef Style) override {
216:     uint32_t Size = Item.getBinding().Size;
217:     if (Size == 0)
218:       OS << "unbounded";
219:     else
220:       OS << Size;
221:   }
222: };
223:
224: } // namespace
225:
226: static void prettyPrintResources(raw_ostream &OS, const DXILResourceMap &DRM,
227:                                  DXILResourceTypeMap &DRTM) {
228:   // Column widths are arbitrary but match the widths DXC uses.
229:   OS << ";\n; Resource Bindings:\n;\n";
230:   OS << formatv("; {0,-30} {1,10} {2,7} {3,11} {4,7} {5,14} {6,9}\n", "Name",
231:                 "Type", "Format", "Dim", "ID", "HLSL Bind", "Count");
232:   OS << formatv(
233:       "; {0,-+30} {1,-+10} {2,-+7} {3,-+11} {4,-+7} {5,-+14} {6,-+9}\n", "", "",
234:       "", "", "", "", "");
235:
236:   // TODO: Do we want to sort these by binding or something like that?
237:   for (const dxil::ResourceInfo &RI : DRM) {
238:     const dxil::ResourceTypeInfo &RTI = DRTM[RI.getHandleTy()];
239:
240:     dxil::ResourceClass RC = RTI.getResourceClass();
```
- EN: This range defines or declares important types such as RC, format, FormatBindingLocation, getBinding, shaping the data model used by DXILPrettyPrinter.cpp.
- CN: 这一段定义或声明了 RC、format、FormatBindingLocation、getBinding 等关键类型，构成 DXILPrettyPrinter.cpp 使用的数据模型。

### Lines 241-300
```cpp
241:     StringRef Name(RI.getName());
242:     StringRef Type(getRCName(RC));
243:     StringRef Format(getFormatName(RTI));
244:     FormatResourceDimension Dim(RTI, RI.hasCounter());
245:     FormatBindingID ID(RI, RTI);
246:     FormatBindingLocation Bind(RI, RTI);
247:     FormatBindingSize Count(RI);
248:     OS << formatv("; {0,-30} {1,10} {2,7} {3,11} {4,7} {5,14} {6,9}\n", Name,
249:                   Type, Format, Dim, ID, Bind, Count);
250:   }
251:   OS << ";\n";
252: }
253:
254: PreservedAnalyses DXILPrettyPrinterPass::run(Module &M,
255:                                              ModuleAnalysisManager &MAM) {
256:   const DXILResourceMap &DRM = MAM.getResult<DXILResourceAnalysis>(M);
257:   DXILResourceTypeMap &DRTM = MAM.getResult<DXILResourceTypeAnalysis>(M);
258:   prettyPrintResources(OS, DRM, DRTM);
259:   return PreservedAnalyses::all();
260: }
261:
262: namespace {
263: class DXILPrettyPrinterLegacy : public llvm::ModulePass {
264:   raw_ostream &OS; // raw_ostream to print to.
265:
266: public:
267:   static char ID;
268:
269:   explicit DXILPrettyPrinterLegacy(raw_ostream &O) : ModulePass(ID), OS(O) {}
270:
271:   StringRef getPassName() const override {
272:     return "DXIL Metadata Pretty Printer";
273:   }
274:
275:   bool runOnModule(Module &M) override;
276:   void getAnalysisUsage(AnalysisUsage &AU) const override {
277:     AU.setPreservesAll();
278:     AU.addRequired<DXILResourceTypeWrapperPass>();
279:     AU.addRequired<DXILResourceWrapperPass>();
280:   }
281: };
282: } // namespace
283:
284: char DXILPrettyPrinterLegacy::ID = 0;
285: INITIALIZE_PASS_BEGIN(DXILPrettyPrinterLegacy, "dxil-pretty-printer",
286:                       "DXIL Metadata Pretty Printer", true, true)
287: INITIALIZE_PASS_DEPENDENCY(DXILResourceTypeWrapperPass)
288: INITIALIZE_PASS_DEPENDENCY(DXILResourceWrapperPass)
289: INITIALIZE_PASS_END(DXILPrettyPrinterLegacy, "dxil-pretty-printer",
290:                     "DXIL Metadata Pretty Printer", true, true)
291:
292: bool DXILPrettyPrinterLegacy::runOnModule(Module &M) {
293:   const DXILResourceMap &DRM =
294:       getAnalysis<DXILResourceWrapperPass>().getResourceMap();
295:   DXILResourceTypeMap &DRTM =
296:       getAnalysis<DXILResourceTypeWrapperPass>().getResourceTypeMap();
297:   prettyPrintResources(OS, DRM, DRTM);
298:   return false;
299: }
300:
```
- EN: This range defines or declares important types such as Name, Type, Format, Dim, shaping the data model used by DXILPrettyPrinter.cpp.
- CN: 这一段定义或声明了 Name、Type、Format、Dim 等关键类型，构成 DXILPrettyPrinter.cpp 使用的数据模型。

### Lines 301-303
```cpp
301: ModulePass *llvm::createDXILPrettyPrinterLegacyPass(raw_ostream &OS) {
302:   return new DXILPrettyPrinterLegacy(OS);
303: }
```
- EN: This range implements operational logic in helpers such as llvm::createDXILPrettyPrinterLegacyPass, DXILPrettyPrinterLegacy, translating backend policy into executable code.
- CN: 这一段实现了 llvm::createDXILPrettyPrinterLegacyPass、DXILPrettyPrinterLegacy 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include getRCName, llvm_unreachable, getRCPrefix, getFormatName, getTextureDimName, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getRCName, llvm_unreachable, getRCPrefix, getFormatName, getTextureDimName，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILPrettyPrinter.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/StringRef.h`
  - `llvm/Analysis/DXILResource.h`
  - `llvm/IR/PassManager.h`
  - `llvm/InitializePasses.h`
  - `llvm/Pass.h`
  - `llvm/Support/FormatAdapters.h`
  - `llvm/Support/FormatVariadic.h`
  - `llvm/Support/raw_ostream.h`
