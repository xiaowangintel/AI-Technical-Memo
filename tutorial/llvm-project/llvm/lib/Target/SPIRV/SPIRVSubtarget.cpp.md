# SPIRVSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVSubtarget.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the SPIR-V specific subclass of TargetSubtargetInfo.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVSubtarget.cpp - SPIR-V Subtarget Information ------*- C++ -*--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file implements the SPIR-V specific subclass of TargetSubtargetInfo.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "SPIRVSubtarget.h"
14:
15: #include "MCTargetDesc/SPIRVBaseInfo.h"
16: #include "SPIRV.h"
17: #include "SPIRVCommandLine.h"
18: #include "SPIRVGlobalRegistry.h"
19: #include "SPIRVLegalizerInfo.h"
20: #include "SPIRVRegisterBankInfo.h"
21: #include "SPIRVTargetMachine.h"
22:
23: #include "llvm/TargetParser/Host.h"
24:
25: using namespace llvm;
26:
27: #define DEBUG_TYPE "spirv-subtarget"
28:
29: #define GET_SUBTARGETINFO_TARGET_DESC
30: #define GET_SUBTARGETINFO_CTOR
31: #include "SPIRVGenSubtargetInfo.inc"
32:
33: static cl::opt<bool>
34:     SPVTranslatorCompat("translator-compatibility-mode",
35:                         cl::desc("SPIR-V Translator compatibility mode"),
36:                         cl::Optional, cl::init(false));
37:
38: static cl::opt<ExtensionSet, false, SPIRVExtensionsParser>
39:     Extensions("spirv-ext",
40:                cl::desc("Specify list of enabled SPIR-V extensions"));
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 41-80
```cpp
41:
42: // Provides access to the cl::opt<...> `Extensions` variable from outside of the
43: // module.
44: void SPIRVSubtarget::addExtensionsToClOpt(const ExtensionSet &AllowList) {
45:   Extensions.insert(AllowList.begin(), AllowList.end());
46: }
47:
48: // Compare version numbers, but allow 0 to mean unspecified.
49: static bool isAtLeastVer(VersionTuple Target, VersionTuple VerToCompareTo) {
50:   return Target.empty() || Target >= VerToCompareTo;
51: }
52:
53: SPIRVSubtarget::SPIRVSubtarget(const Triple &TT, const std::string &CPU,
54:                                const std::string &FS,
55:                                const SPIRVTargetMachine &TM)
56:     : SPIRVGenSubtargetInfo(TT, CPU, /*TuneCPU=*/CPU, FS),
57:       PointerSize(TM.getPointerSizeInBits(/* AS= */ 0)),
58:       InstrInfo(initSubtargetDependencies(CPU, FS)), FrameLowering(*this),
59:       TLInfo(TM, *this), TargetTriple(TT) {
60:   switch (TT.getSubArch()) {
61:   case Triple::SPIRVSubArch_v10:
62:     SPIRVVersion = VersionTuple(1, 0);
63:     break;
64:   case Triple::SPIRVSubArch_v11:
65:     SPIRVVersion = VersionTuple(1, 1);
66:     break;
67:   case Triple::SPIRVSubArch_v12:
68:     SPIRVVersion = VersionTuple(1, 2);
69:     break;
70:   case Triple::SPIRVSubArch_v13:
71:     SPIRVVersion = VersionTuple(1, 3);
72:     break;
73:   case Triple::SPIRVSubArch_v14:
74:     SPIRVVersion = VersionTuple(1, 4);
75:     break;
76:   case Triple::SPIRVSubArch_v15:
77:     SPIRVVersion = VersionTuple(1, 5);
78:     break;
79:   case Triple::SPIRVSubArch_v16:
80:     SPIRVVersion = VersionTuple(1, 6);
```
- EN: This range implements operational logic in helpers such as SPIRVSubtarget::addExtensionsToClOpt, insert, isAtLeastVer, TLInfo, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVSubtarget::addExtensionsToClOpt、insert、isAtLeastVer、TLInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 81-120
```cpp
 81:     break;
 82:   default:
 83:     if (TT.getVendor() == Triple::AMD)
 84:       SPIRVVersion = VersionTuple(1, 6);
 85:     else
 86:       SPIRVVersion = VersionTuple(1, 4);
 87:   }
 88:   OpenCLVersion = VersionTuple(2, 2);
 89:
 90:   // Set the environment based on the target triple.
 91:   if (TargetTriple.getOS() == Triple::Vulkan)
 92:     Env = Shader;
 93:   else if (TargetTriple.getOS() == Triple::OpenCL ||
 94:            TargetTriple.getVendor() == Triple::AMD)
 95:     Env = Kernel;
 96:   else
 97:     Env = Unknown;
 98:
 99:   // Set the default extensions based on the target triple.
100:   if (TargetTriple.getVendor() == Triple::Intel) {
101:     Extensions.insert(SPIRV::Extension::SPV_INTEL_function_pointers);
102:     Extensions.insert(
103:         SPIRV::Extension::SPV_EXT_relaxed_printf_string_address_space);
104:   }
105:   if (TargetTriple.getVendor() == Triple::AMD)
106:     Extensions = SPIRVExtensionsParser::getValidExtensions(TargetTriple);
107:
108:   // The order of initialization is important.
109:   initAvailableExtensions(Extensions);
110:   initAvailableExtInstSets();
111:
112:   GR = std::make_unique<SPIRVGlobalRegistry>(TM.createDataLayout());
113:   CallLoweringInfo = std::make_unique<SPIRVCallLowering>(TLInfo, GR.get());
114:   InlineAsmInfo = std::make_unique<SPIRVInlineAsmLowering>(TLInfo);
115:   Legalizer = std::make_unique<SPIRVLegalizerInfo>(*this);
116:   RegBankInfo = std::make_unique<SPIRVRegisterBankInfo>();
117:   InstSelector.reset(createSPIRVInstructionSelector(TM, *this, *RegBankInfo));
118: }
119:
120: SPIRVSubtarget &SPIRVSubtarget::initSubtargetDependencies(StringRef CPU,
```
- EN: This range implements operational logic in helpers such as VersionTuple, getVendor, insert, SPIRVExtensionsParser::getValidExtensions, translating backend policy into executable code.
- CN: 这一段实现了 VersionTuple、getVendor、insert、SPIRVExtensionsParser::getValidExtensions 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:                                                           StringRef FS) {
122:   ParseSubtargetFeatures(CPU, /*TuneCPU=*/CPU, FS);
123:   return *this;
124: }
125:
126: bool SPIRVSubtarget::canUseExtension(SPIRV::Extension::Extension E) const {
127:   return AvailableExtensions.contains(E);
128: }
129:
130: bool SPIRVSubtarget::canUseExtInstSet(
131:     SPIRV::InstructionSet::InstructionSet E) const {
132:   return AvailableExtInstSets.contains(E);
133: }
134:
135: SPIRV::InstructionSet::InstructionSet
136: SPIRVSubtarget::getPreferredInstructionSet() const {
137:   if (isShader())
138:     return SPIRV::InstructionSet::GLSL_std_450;
139:   else
140:     return SPIRV::InstructionSet::OpenCL_std;
141: }
142:
143: bool SPIRVSubtarget::isAtLeastSPIRVVer(VersionTuple VerToCompareTo) const {
144:   return isAtLeastVer(SPIRVVersion, VerToCompareTo);
145: }
146:
147: bool SPIRVSubtarget::isAtLeastOpenCLVer(VersionTuple VerToCompareTo) const {
148:   if (isShader())
149:     return false;
150:   return isAtLeastVer(OpenCLVersion, VerToCompareTo);
151: }
152:
153: // If the SPIR-V version is >= 1.4 we can call OpPtrEqual and OpPtrNotEqual.
154: // In SPIR-V Translator compatibility mode this feature is not available.
155: bool SPIRVSubtarget::canDirectlyComparePointers() const {
156:   return !SPVTranslatorCompat && isAtLeastVer(SPIRVVersion, VersionTuple(1, 4));
157: }
158:
159: void SPIRVSubtarget::accountForAMDShaderTrinaryMinmax() {
160:   if (canUseExtension(
```
- EN: This range implements operational logic in helpers such as ParseSubtargetFeatures, SPIRVSubtarget::canUseExtension, contains, SPIRVSubtarget::getPreferredInstructionSet, translating backend policy into executable code.
- CN: 这一段实现了 ParseSubtargetFeatures、SPIRVSubtarget::canUseExtension、contains、SPIRVSubtarget::getPreferredInstructionSet 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:           SPIRV::Extension::SPV_AMD_shader_trinary_minmax_extension)) {
162:     AvailableExtInstSets.insert(
163:         SPIRV::InstructionSet::SPV_AMD_shader_trinary_minmax);
164:   }
165: }
166:
167: // TODO: use command line args for this rather than just defaults.
168: // Must have called initAvailableExtensions first.
169: void SPIRVSubtarget::initAvailableExtInstSets() {
170:   AvailableExtInstSets.clear();
171:   if (isShader())
172:     AvailableExtInstSets.insert(SPIRV::InstructionSet::GLSL_std_450);
173:   else
174:     AvailableExtInstSets.insert(SPIRV::InstructionSet::OpenCL_std);
175:
176:   // Handle extended instruction sets from extensions.
177:   accountForAMDShaderTrinaryMinmax();
178: }
179:
180: void SPIRVSubtarget::setEnv(SPIRVEnvType E) {
181:   if (E == Unknown)
182:     report_fatal_error("Unknown environment is not allowed.");
183:   if (Env != Unknown && Env != E)
184:     report_fatal_error("Environment is already set to a different value.");
185:   if (Env == E)
186:     return;
187:
188:   Env = E;
189:
190:   // Reinitialize Env-dependent state aka ExtInstSet and legalizer info.
191:   initAvailableExtInstSets();
192:   Legalizer = std::make_unique<SPIRVLegalizerInfo>(*this);
193: }
194:
195: void SPIRVSubtarget::resolveEnvFromModule(const Module &M) {
196:   *GR = SPIRVGlobalRegistry(M.getDataLayout());
197:
198:   if (Env != Unknown) {
199:     assert(!(isKernel() && any_of(M,
200:                                   [](const Function &F) {
```
- EN: This range implements operational logic in helpers such as SPIRVSubtarget::initAvailableExtInstSets, clear, insert, accountForAMDShaderTrinaryMinmax, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVSubtarget::initAvailableExtInstSets、clear、insert、accountForAMDShaderTrinaryMinmax 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:                                     return F.hasFnAttribute("hlsl.shader");
202:                                   })) &&
203:            "Module has hlsl.shader attributes but environment is Kernel");
204:     return;
205:   }
206:
207:   bool HasShaderAttr = false;
208:   for (const Function &F : M) {
209:     if (F.hasFnAttribute("hlsl.shader")) {
210:       HasShaderAttr = true;
211:       break;
212:     }
213:   }
214:
215:   if (!HasShaderAttr) {
216:     if (auto *MemModel = M.getNamedMetadata("spirv.MemoryModel")) {
217:       if (MemModel->getNumOperands() == 0)
218:         report_fatal_error("Invalid spirv.MemoryModel metadata");
219:       auto *MemMD = MemModel->getOperand(0);
220:       if (MemMD->getNumOperands() < 2)
221:         report_fatal_error("Invalid spirv.MemoryModel operand");
222:       unsigned MemModelVal =
223:           mdconst::extract<ConstantInt>(MemMD->getOperand(1))->getZExtValue();
224:       switch (MemModelVal) {
225:       case SPIRV::MemoryModel::Simple:
226:       case SPIRV::MemoryModel::GLSL450:
227:         HasShaderAttr = true;
228:         break;
229:       case SPIRV::MemoryModel::VulkanKHR:
230:         HasShaderAttr = true;
231:         AvailableExtensions.insert(
232:             SPIRV::Extension::SPV_KHR_vulkan_memory_model);
233:         break;
234:       case SPIRV::MemoryModel::OpenCL:
235:         break;
236:       default:
237:         report_fatal_error(
238:             "Unknown memory model in spirv.MemoryModel metadata");
239:       }
240:     }
```
- EN: This range implements operational logic in helpers such as hasFnAttribute, report_fatal_error, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 hasFnAttribute、report_fatal_error、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-259
```cpp
241:   }
242:
243:   setEnv(HasShaderAttr ? Shader : Kernel);
244: }
245:
246: // Set available extensions after SPIRVSubtarget is created.
247: void SPIRVSubtarget::initAvailableExtensions(
248:     const ExtensionSet &AllowedExtIds) {
249:   AvailableExtensions.clear();
250:   const ExtensionSet &ValidExtensions =
251:       SPIRVExtensionsParser::getValidExtensions(TargetTriple);
252:
253:   for (const auto &Ext : AllowedExtIds) {
254:     if (ValidExtensions.count(Ext))
255:       AvailableExtensions.insert(Ext);
256:   }
257:
258:   accountForAMDShaderTrinaryMinmax();
259: }
```
- EN: This range implements operational logic in helpers such as setEnv, clear, SPIRVExtensionsParser::getValidExtensions, insert, translating backend policy into executable code.
- CN: 这一段实现了 setEnv、clear、SPIRVExtensionsParser::getValidExtensions、insert 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Subtarget objects record per-environment or per-CPU capabilities that specialize backend behavior.
  - CN: Subtarget 对象记录按环境或 CPU 细分的能力，用于特化后端行为。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include cl::init, cl::desc, SPIRVSubtarget::addExtensionsToClOpt, insert, isAtLeastVer, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, cl::desc, SPIRVSubtarget::addExtensionsToClOpt, insert, isAtLeastVer，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVSubtarget.h`
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `SPIRV.h`
  - `SPIRVCommandLine.h`
  - `SPIRVGlobalRegistry.h`
  - `SPIRVLegalizerInfo.h`
  - `SPIRVRegisterBankInfo.h`
  - `SPIRVTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/TargetParser/Host.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenSubtargetInfo.inc`
