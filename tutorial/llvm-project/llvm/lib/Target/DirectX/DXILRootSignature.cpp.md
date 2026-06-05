# DXILRootSignature.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/DirectX/DXILRootSignature.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains helper objects and APIs for working with DXIL Root Signatures.
- 目的（中文）: 该文件提供 LLVM 目标后端所需的辅助逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===- DXILRootSignature.cpp - DXIL Root Signature helper objects -------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: ///
 9: /// \file This file contains helper objects and APIs for working with DXIL
10: ///       Root Signatures.
11: ///
12: //===----------------------------------------------------------------------===//
13: #include "DXILRootSignature.h"
14: #include "DirectX.h"
15: #include "llvm/ADT/StringSwitch.h"
16: #include "llvm/ADT/Twine.h"
17: #include "llvm/Analysis/DXILMetadataAnalysis.h"
18: #include "llvm/BinaryFormat/DXContainer.h"
19: #include "llvm/Frontend/HLSL/RootSignatureMetadata.h"
20: #include "llvm/Frontend/HLSL/RootSignatureValidations.h"
21: #include "llvm/IR/Constants.h"
22: #include "llvm/IR/DiagnosticInfo.h"
23: #include "llvm/IR/Function.h"
24: #include "llvm/IR/LLVMContext.h"
25: #include "llvm/IR/Metadata.h"
26: #include "llvm/IR/Module.h"
27: #include "llvm/InitializePasses.h"
28: #include "llvm/MC/DXContainerRootSignature.h"
29: #include "llvm/Pass.h"
30: #include "llvm/Support/Error.h"
31: #include "llvm/Support/ErrorHandling.h"
32: #include "llvm/Support/ScopedPrinter.h"
33: #include "llvm/Support/raw_ostream.h"
34: #include <cstdint>
35:
36: using namespace llvm;
37: using namespace llvm::dxil;
38:
39: static std::optional<uint32_t> extractMdIntValue(MDNode *Node,
40:                                                  unsigned int OpId) {
```
- EN: This range implements operational logic in helpers such as backend logic, translating backend policy into executable code.
- CN: 这一段实现了 后端逻辑 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 41-80
```cpp
41:   if (auto *CI =
42:           mdconst::dyn_extract<ConstantInt>(Node->getOperand(OpId).get()))
43:     return CI->getZExtValue();
44:   return std::nullopt;
45: }
46:
47: static bool reportError(LLVMContext *Ctx, Twine Message,
48:                         DiagnosticSeverity Severity = DS_Error) {
49:   Ctx->diagnose(DiagnosticInfoGeneric(Message, Severity));
50:   return true;
51: }
52:
53: static SmallDenseMap<const Function *, mcdxbc::RootSignatureDesc>
54: analyzeModule(Module &M) {
55:
56:   /** Root Signature are specified as following in the metadata:
57:
58:     !dx.rootsignatures = !{!2} ; list of function/root signature pairs
59:     !2 = !{ ptr @main, !3 } ; function, root signature
60:     !3 = !{ !4, !5, !6, !7 } ; list of root signature elements
61:
62:     So for each MDNode inside dx.rootsignatures NamedMDNode
63:     (the Root parameter of this function), the parsing process needs
64:     to loop through each of its operands and process the function,
65:     signature pair.
66:  */
67:
68:   LLVMContext *Ctx = &M.getContext();
69:
70:   SmallDenseMap<const Function *, mcdxbc::RootSignatureDesc> RSDMap;
71:
72:   NamedMDNode *RootSignatureNode = M.getNamedMetadata("dx.rootsignatures");
73:   if (RootSignatureNode == nullptr)
74:     return RSDMap;
75:
76:   bool AllowNullFunctions = false;
77:   if (M.getTargetTriple().getEnvironment() ==
78:       Triple::EnvironmentType::RootSignature) {
79:     assert(RootSignatureNode->getNumOperands() == 1);
80:     AllowNullFunctions = true;
```
- EN: This range implements operational logic in helpers such as getOperand, getZExtValue, diagnose, analyzeModule, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getZExtValue、diagnose、analyzeModule 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:   }
 82:
 83:   for (const auto &RSDefNode : RootSignatureNode->operands()) {
 84:     if (RSDefNode->getNumOperands() != 3) {
 85:       reportError(Ctx, "Invalid Root Signature metadata - expected function, "
 86:                        "signature, and version.");
 87:       continue;
 88:     }
 89:
 90:     // Function was pruned during compilation.
 91:     Function *F = nullptr;
 92:
 93:     if (!AllowNullFunctions) {
 94:       const MDOperand &FunctionPointerMdNode = RSDefNode->getOperand(0);
 95:       if (FunctionPointerMdNode == nullptr) {
 96:         reportError(
 97:             Ctx, "Function associated with Root Signature definition is null.");
 98:         continue;
 99:       }
100:
101:       ValueAsMetadata *VAM =
102:           llvm::dyn_cast<ValueAsMetadata>(FunctionPointerMdNode.get());
103:       if (VAM == nullptr) {
104:         reportError(Ctx, "First element of root signature is not a Value");
105:         continue;
106:       }
107:
108:       F = dyn_cast<Function>(VAM->getValue());
109:       if (F == nullptr) {
110:         reportError(Ctx, "First element of root signature is not a Function");
111:         continue;
112:       }
113:     }
114:
115:     Metadata *RootElementListOperand = RSDefNode->getOperand(1).get();
116:
117:     if (RootElementListOperand == nullptr) {
118:       reportError(Ctx, "Root Element mdnode is null.");
119:       continue;
120:     }
```
- EN: This range implements operational logic in helpers such as getOperand, get, reportError, getValue, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、get、reportError、getValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:
122:     MDNode *RootElementListNode = dyn_cast<MDNode>(RootElementListOperand);
123:     if (RootElementListNode == nullptr) {
124:       reportError(Ctx, "Root Element is not a metadata node.");
125:       continue;
126:     }
127:     std::optional<uint32_t> V = extractMdIntValue(RSDefNode, 2);
128:     if (!V.has_value()) {
129:       reportError(Ctx, "Invalid RSDefNode value, expected constant int");
130:       continue;
131:     }
132:
133:     if (!hlsl::rootsig::verifyVersion(*V)) {
134:       reportError(Ctx, "Invalid Root Signature Version: " + Twine(*V));
135:       continue;
136:     }
137:
138:     llvm::hlsl::rootsig::MetadataParser MDParser(RootElementListNode);
139:     llvm::Expected<mcdxbc::RootSignatureDesc> RSDOrErr =
140:         MDParser.ParseRootSignature(V.value());
141:
142:     if (!RSDOrErr) {
143:       handleAllErrors(RSDOrErr.takeError(), [&](ErrorInfoBase &EIB) {
144:         Ctx->emitError(EIB.message());
145:       });
146:       continue;
147:     }
148:
149:     auto &RSD = *RSDOrErr;
150:
151:     // Clang emits the root signature data in dxcontainer following a specific
152:     // sequence. First the header, then the root parameters. So the header
153:     // offset will always equal to the header size.
154:     RSD.RootParameterOffset = sizeof(dxbc::RTS0::v1::RootSignatureHeader);
155:
156:     // static sampler offset is calculated when writting dxcontainer.
157:     RSD.StaticSamplersOffset = 0u;
158:
159:     RSDMap.insert(std::make_pair(F, RSD));
160:   }
```
- EN: This range implements operational logic in helpers such as reportError, extractMdIntValue, MDParser, ParseRootSignature, translating backend policy into executable code.
- CN: 这一段实现了 reportError、extractMdIntValue、MDParser、ParseRootSignature 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:
162:   return RSDMap;
163: }
164:
165: AnalysisKey RootSignatureAnalysis::Key;
166:
167: RootSignatureAnalysis::Result
168: RootSignatureAnalysis::run(Module &M, ModuleAnalysisManager &AM) {
169:   return RootSignatureBindingInfo(analyzeModule(M));
170: }
171:
172: //===----------------------------------------------------------------------===//
173:
174: PreservedAnalyses RootSignatureAnalysisPrinter::run(Module &M,
175:                                                     ModuleAnalysisManager &AM) {
176:
177:   RootSignatureBindingInfo &RSDMap = AM.getResult<RootSignatureAnalysis>(M);
178:   if (RSDMap.empty())
179:     return PreservedAnalyses::all();
180:
181:   OS << "Root Signature Definitions"
182:      << "\n";
183:   for (const Function &F : M) {
184:     auto It = RSDMap.find(&F);
185:     if (It == RSDMap.end())
186:       continue;
187:     const auto &RS = It->second;
188:     OS << "Definition for '" << F.getName() << "':\n";
189:     // start root signature header
190:     OS << "Flags: " << format_hex(RS.Flags, 8) << "\n"
191:        << "Version: " << RS.Version << "\n"
192:        << "RootParametersOffset: " << RS.RootParameterOffset << "\n"
193:        << "NumParameters: " << RS.ParametersContainer.size() << "\n";
194:     for (size_t I = 0; I < RS.ParametersContainer.size(); I++) {
195:       const mcdxbc::RootParameterInfo &Info = RS.ParametersContainer.getInfo(I);
196:
197:       OS << "- Parameter Type: "
198:          << enumToStringRef(Info.Type, dxbc::getRootParameterTypes()) << "\n"
199:          << "  Shader Visibility: "
200:          << enumToStringRef(Info.Visibility, dxbc::getShaderVisibility())
```
- EN: This range implements operational logic in helpers such as RootSignatureAnalysis::run, RootSignatureBindingInfo, PreservedAnalyses::all, find, translating backend policy into executable code.
- CN: 这一段实现了 RootSignatureAnalysis::run、RootSignatureBindingInfo、PreservedAnalyses::all、find 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:          << "\n";
202:       switch (Info.Type) {
203:       case dxbc::RootParameterType::Constants32Bit: {
204:         const mcdxbc::RootConstants &Constants =
205:             RS.ParametersContainer.getConstant(Info.Location);
206:         OS << "  Register Space: " << Constants.RegisterSpace << "\n"
207:            << "  Shader Register: " << Constants.ShaderRegister << "\n"
208:            << "  Num 32 Bit Values: " << Constants.Num32BitValues << "\n";
209:         break;
210:       }
211:       case dxbc::RootParameterType::CBV:
212:       case dxbc::RootParameterType::UAV:
213:       case dxbc::RootParameterType::SRV: {
214:         const mcdxbc::RootDescriptor &Descriptor =
215:             RS.ParametersContainer.getRootDescriptor(Info.Location);
216:         OS << "  Register Space: " << Descriptor.RegisterSpace << "\n"
217:            << "  Shader Register: " << Descriptor.ShaderRegister << "\n";
218:         if (RS.Version > 1)
219:           OS << "  Flags: " << Descriptor.Flags << "\n";
220:         break;
221:       }
222:       case dxbc::RootParameterType::DescriptorTable: {
223:         const mcdxbc::DescriptorTable &Table =
224:             RS.ParametersContainer.getDescriptorTable(Info.Location);
225:         OS << "  NumRanges: " << Table.Ranges.size() << "\n";
226:
227:         for (const mcdxbc::DescriptorRange &Range : Table) {
228:           OS << "  - Range Type: "
229:              << dxil::getResourceClassName(Range.RangeType) << "\n"
230:              << "    Register Space: " << Range.RegisterSpace << "\n"
231:              << "    Base Shader Register: " << Range.BaseShaderRegister << "\n"
232:              << "    Num Descriptors: " << Range.NumDescriptors << "\n"
233:              << "    Offset In Descriptors From Table Start: "
234:              << Range.OffsetInDescriptorsFromTableStart << "\n";
235:           if (RS.Version > 1)
236:             OS << "    Flags: " << Range.Flags << "\n";
237:         }
238:         break;
239:       }
240:       }
```
- EN: This range implements operational logic in helpers such as getConstant, getRootDescriptor, getDescriptorTable, translating backend policy into executable code.
- CN: 这一段实现了 getConstant、getRootDescriptor、getDescriptorTable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-267
```cpp
241:     }
242:     OS << "NumStaticSamplers: " << 0 << "\n";
243:     OS << "StaticSamplersOffset: " << RS.StaticSamplersOffset << "\n";
244:   }
245:   return PreservedAnalyses::all();
246: }
247:
248: //===----------------------------------------------------------------------===//
249: bool RootSignatureAnalysisWrapper::runOnModule(Module &M) {
250:   FuncToRsMap = std::make_unique<RootSignatureBindingInfo>(
251:       RootSignatureBindingInfo(analyzeModule(M)));
252:   return false;
253: }
254:
255: void RootSignatureAnalysisWrapper::getAnalysisUsage(AnalysisUsage &AU) const {
256:   AU.setPreservesAll();
257:   AU.addPreserved<DXILMetadataAnalysisWrapperPass>();
258: }
259:
260: char RootSignatureAnalysisWrapper::ID = 0;
261:
262: INITIALIZE_PASS_BEGIN(RootSignatureAnalysisWrapper,
263:                       "dxil-root-signature-analysis",
264:                       "DXIL Root Signature Analysis", true, true)
265: INITIALIZE_PASS_END(RootSignatureAnalysisWrapper,
266:                     "dxil-root-signature-analysis",
267:                     "DXIL Root Signature Analysis", true, true)
```
- EN: This range implements operational logic in helpers such as PreservedAnalyses::all, RootSignatureAnalysisWrapper::runOnModule, RootSignatureBindingInfo, RootSignatureAnalysisWrapper::getAnalysisUsage, translating backend policy into executable code.
- CN: 这一段实现了 PreservedAnalyses::all、RootSignatureAnalysisWrapper::runOnModule、RootSignatureBindingInfo、RootSignatureAnalysisWrapper::getAnalysisUsage 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: DirectX/DXIL backend code models HLSL/DXIL semantics, including resources, shader stages, and validation-friendly IR forms.
  - CN: DirectX/DXIL 后端代码建模了 HLSL/DXIL 语义，包括资源、着色器阶段以及便于验证的 IR 形式。
- EN: Key symbols in this file include getOperand, getZExtValue, diagnose, analyzeModule, getContext, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getOperand, getZExtValue, diagnose, analyzeModule, getContext，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `DXILRootSignature.h`
  - `DirectX.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/StringSwitch.h`
  - `llvm/ADT/Twine.h`
  - `llvm/Analysis/DXILMetadataAnalysis.h`
  - `llvm/BinaryFormat/DXContainer.h`
  - `llvm/Frontend/HLSL/RootSignatureMetadata.h`
  - `llvm/Frontend/HLSL/RootSignatureValidations.h`
  - `llvm/IR/Constants.h`
  - `llvm/IR/DiagnosticInfo.h`
  - `llvm/IR/Function.h`
  - `llvm/IR/LLVMContext.h`
  - `llvm/IR/Metadata.h`
  - `llvm/IR/Module.h`
  - `llvm/InitializePasses.h`
  - `llvm/MC/DXContainerRootSignature.h`
  - `llvm/Pass.h`
  - `llvm/Support/Error.h`
- System/standard headers / 系统或标准头文件:
  - `cstdint`
