# SPIRVBaseInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/MCTargetDesc/SPIRVBaseInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains the implementation for helper mnemonic lookup functions, versioning capabilities extensions getters for symbolic named operands used in various SPIR-V instructions.
- 目的（中文）: 该文件实现 LLVM 目标后端中的相关逻辑。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-40
```cpp
 1: //===-- SPIRVBaseInfo.cpp - Top level SPIRV definitions ---------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains the implementation for helper mnemonic lookup functions,
10: // versioning/capabilities/extensions getters for symbolic/named operands used
11: // in various SPIR-V instructions.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #include "SPIRVBaseInfo.h"
16: #include "llvm/ADT/ArrayRef.h"
17: #include "llvm/ADT/StringRef.h"
18:
19: namespace llvm {
20: namespace SPIRV {
21: struct SymbolicOperand {
22:   OperandCategory::OperandCategory Category;
23:   uint32_t Value;
24:   StringRef Mnemonic;
25:   uint32_t MinVersion;
26:   uint32_t MaxVersion;
27: };
28:
29: struct ExtensionEntry {
30:   OperandCategory::OperandCategory Category;
31:   uint32_t Value;
32:   Extension::Extension ReqExtension;
33: };
34:
35: struct CapabilityEntry {
36:   OperandCategory::OperandCategory Category;
37:   uint32_t Value;
38:   Capability::Capability ReqCapability;
39: };
40:
```
- EN: This range defines or declares important types such as SymbolicOperand, ExtensionEntry, CapabilityEntry, shaping the data model used by SPIRVBaseInfo.cpp.
- CN: 这一段定义或声明了 SymbolicOperand、ExtensionEntry、CapabilityEntry 等关键类型，构成 SPIRVBaseInfo.cpp 使用的数据模型。

### Lines 41-80
```cpp
41: struct EnvironmentEntry {
42:   OperandCategory::OperandCategory Category;
43:   uint32_t Value;
44:   Environment::Environment AllowedEnvironment;
45: };
46:
47: using namespace OperandCategory;
48: using namespace Extension;
49: using namespace Environment;
50: using namespace Capability;
51: using namespace InstructionSet;
52: #define GET_SymbolicOperands_DECL
53: #define GET_SymbolicOperands_IMPL
54: #define GET_ExtensionEntries_DECL
55: #define GET_ExtensionEntries_IMPL
56: #define GET_CapabilityEntries_DECL
57: #define GET_CapabilityEntries_IMPL
58: #define GET_EnvironmentEntries_DECL
59: #define GET_EnvironmentEntries_IMPL
60: #define GET_ExtendedBuiltins_DECL
61: #define GET_ExtendedBuiltins_IMPL
62: #include "SPIRVGenTables.inc"
63: } // namespace SPIRV
64:
65: std::string
66: getSymbolicOperandMnemonic(SPIRV::OperandCategory::OperandCategory Category,
67:                            int32_t Value) {
68:   const SPIRV::SymbolicOperand *Lookup =
69:       SPIRV::lookupSymbolicOperandByCategoryAndValue(Category, Value);
70:   // Value that encodes just one enum value.
71:   if (Lookup)
72:     return Lookup->Mnemonic.str();
73:   if (Category != SPIRV::OperandCategory::ImageOperandOperand &&
74:       Category != SPIRV::OperandCategory::FPFastMathModeOperand &&
75:       Category != SPIRV::OperandCategory::SelectionControlOperand &&
76:       Category != SPIRV::OperandCategory::LoopControlOperand &&
77:       Category != SPIRV::OperandCategory::FunctionControlOperand &&
78:       Category != SPIRV::OperandCategory::MemorySemanticsOperand &&
79:       Category != SPIRV::OperandCategory::MemoryOperandOperand &&
80:       Category != SPIRV::OperandCategory::KernelProfilingInfoOperand &&
```
- EN: This range defines or declares important types such as EnvironmentEntry, SPIRV::lookupSymbolicOperandByCategoryAndValue, str, shaping the data model used by SPIRVBaseInfo.cpp.
- CN: 这一段定义或声明了 EnvironmentEntry、SPIRV::lookupSymbolicOperandByCategoryAndValue、str 等关键类型，构成 SPIRVBaseInfo.cpp 使用的数据模型。

### Lines 81-120
```cpp
 81:       Category != SPIRV::OperandCategory::SpecConstantOpOperandsOperand)
 82:     return "UNKNOWN";
 83:   // Value that encodes many enum values (one bit per enum value).
 84:   std::string Name;
 85:   std::string Separator;
 86:   const SPIRV::SymbolicOperand *EnumValueInCategory =
 87:       SPIRV::lookupSymbolicOperandByCategory(Category);
 88:
 89:   auto TableEnd = ArrayRef(SPIRV::SymbolicOperands).end();
 90:   while (EnumValueInCategory && EnumValueInCategory->Category == Category) {
 91:     if ((EnumValueInCategory->Value != 0) &&
 92:         (Value & EnumValueInCategory->Value)) {
 93:       Name += Separator + EnumValueInCategory->Mnemonic.str();
 94:       Separator = "|";
 95:     }
 96:     if (++EnumValueInCategory == TableEnd)
 97:       break;
 98:   }
 99:
100:   return Name;
101: }
102:
103: VersionTuple
104: getSymbolicOperandMinVersion(SPIRV::OperandCategory::OperandCategory Category,
105:                              uint32_t Value) {
106:   const SPIRV::SymbolicOperand *Lookup =
107:       SPIRV::lookupSymbolicOperandByCategoryAndValue(Category, Value);
108:
109:   if (Lookup)
110:     return VersionTuple(Lookup->MinVersion / 10, Lookup->MinVersion % 10);
111:
112:   return VersionTuple(0);
113: }
114:
115: VersionTuple
116: getSymbolicOperandMaxVersion(SPIRV::OperandCategory::OperandCategory Category,
117:                              uint32_t Value) {
118:   const SPIRV::SymbolicOperand *Lookup =
119:       SPIRV::lookupSymbolicOperandByCategoryAndValue(Category, Value);
120:
```
- EN: This range implements operational logic in helpers such as SPIRV::lookupSymbolicOperandByCategory, ArrayRef, str, SPIRV::lookupSymbolicOperandByCategoryAndValue, translating backend policy into executable code.
- CN: 这一段实现了 SPIRV::lookupSymbolicOperandByCategory、ArrayRef、str、SPIRV::lookupSymbolicOperandByCategoryAndValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-160
```cpp
121:   if (Lookup)
122:     return VersionTuple(Lookup->MaxVersion / 10, Lookup->MaxVersion % 10);
123:
124:   return VersionTuple();
125: }
126:
127: CapabilityList
128: getSymbolicOperandCapabilities(SPIRV::OperandCategory::OperandCategory Category,
129:                                uint32_t Value) {
130:   CapabilityList Capabilities;
131:   const SPIRV::CapabilityEntry *Capability =
132:       SPIRV::lookupCapabilityByCategoryAndValue(Category, Value);
133:   auto TableEnd = ArrayRef(SPIRV::CapabilityEntries).end();
134:   while (Capability && Capability->Category == Category &&
135:          Capability->Value == Value) {
136:     Capabilities.push_back(
137:         static_cast<SPIRV::Capability::Capability>(Capability->ReqCapability));
138:     if (++Capability == TableEnd)
139:       break;
140:   }
141:
142:   return Capabilities;
143: }
144:
145: EnvironmentList getSymbolicOperandAllowedEnvironments(
146:     SPIRV::OperandCategory::OperandCategory Category, uint32_t Value) {
147:   EnvironmentList Environments;
148:   const SPIRV::EnvironmentEntry *Environment =
149:       SPIRV::lookupEnvironmentByCategoryAndValue(Category, Value);
150:   auto TableEnd = ArrayRef(SPIRV::EnvironmentEntries).end();
151:   while (Environment && Environment->Category == Category &&
152:          Environment->Value == Value) {
153:     Environments.push_back(static_cast<SPIRV::Environment::Environment>(
154:         Environment->AllowedEnvironment));
155:     if (++Environment == TableEnd)
156:       break;
157:   }
158:
159:   return Environments;
160: }
```
- EN: This range implements operational logic in helpers such as VersionTuple, SPIRV::lookupCapabilityByCategoryAndValue, ArrayRef, SPIRV::lookupEnvironmentByCategoryAndValue, translating backend policy into executable code.
- CN: 这一段实现了 VersionTuple、SPIRV::lookupCapabilityByCategoryAndValue、ArrayRef、SPIRV::lookupEnvironmentByCategoryAndValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 161-200
```cpp
161:
162: CapabilityList
163: getCapabilitiesEnabledByExtension(SPIRV::Extension::Extension Extension) {
164:   const SPIRV::ExtensionEntry *Entry =
165:       SPIRV::lookupSymbolicOperandsEnabledByExtension(
166:           Extension, SPIRV::OperandCategory::CapabilityOperand);
167:
168:   CapabilityList Capabilities;
169:   auto TableEnd = ArrayRef(SPIRV::ExtensionEntries).end();
170:   while (Entry &&
171:          Entry->Category == SPIRV::OperandCategory::CapabilityOperand) {
172:     // Some capabilities' codes might go not in order.
173:     if (Entry->ReqExtension == Extension)
174:       Capabilities.push_back(
175:           static_cast<SPIRV::Capability::Capability>(Entry->Value));
176:     if (++Entry == TableEnd)
177:       break;
178:   }
179:
180:   return Capabilities;
181: }
182:
183: ExtensionList
184: getSymbolicOperandExtensions(SPIRV::OperandCategory::OperandCategory Category,
185:                              uint32_t Value) {
186:   const SPIRV::ExtensionEntry *Extension =
187:       SPIRV::lookupExtensionByCategoryAndValue(Category, Value);
188:
189:   ExtensionList Extensions;
190:   auto TableEnd = ArrayRef(SPIRV::ExtensionEntries).end();
191:   while (Extension && Extension->Category == Category &&
192:          Extension->Value == Value) {
193:     Extensions.push_back(
194:         static_cast<SPIRV::Extension::Extension>(Extension->ReqExtension));
195:     if (++Extension == TableEnd)
196:       break;
197:   }
198:
199:   return Extensions;
200: }
```
- EN: This range implements operational logic in helpers such as getCapabilitiesEnabledByExtension, ArrayRef, SPIRV::lookupExtensionByCategoryAndValue, translating backend policy into executable code.
- CN: 这一段实现了 getCapabilitiesEnabledByExtension、ArrayRef、SPIRV::lookupExtensionByCategoryAndValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 201-240
```cpp
201:
202: std::string getLinkStringForBuiltIn(SPIRV::BuiltIn::BuiltIn BuiltInValue) {
203:   const SPIRV::SymbolicOperand *Lookup =
204:       SPIRV::lookupSymbolicOperandByCategoryAndValue(
205:           SPIRV::OperandCategory::BuiltInOperand, BuiltInValue);
206:
207:   if (Lookup)
208:     return "__spirv_BuiltIn" + Lookup->Mnemonic.str();
209:   return "UNKNOWN_BUILTIN";
210: }
211:
212: bool getSpirvBuiltInIdByName(llvm::StringRef Name,
213:                              SPIRV::BuiltIn::BuiltIn &BI) {
214:   const std::string Prefix = "__spirv_BuiltIn";
215:   if (!Name.starts_with(Prefix))
216:     return false;
217:
218:   const SPIRV::SymbolicOperand *Lookup =
219:       SPIRV::lookupSymbolicOperandByCategoryAndMnemonic(
220:           SPIRV::OperandCategory::BuiltInOperand,
221:           Name.drop_front(Prefix.length()));
222:
223:   if (!Lookup)
224:     return false;
225:
226:   BI = static_cast<SPIRV::BuiltIn::BuiltIn>(Lookup->Value);
227:   return true;
228: }
229:
230: std::string getExtInstSetName(SPIRV::InstructionSet::InstructionSet Set) {
231:   switch (Set) {
232:   case SPIRV::InstructionSet::OpenCL_std:
233:     return "OpenCL.std";
234:   case SPIRV::InstructionSet::GLSL_std_450:
235:     return "GLSL.std.450";
236:   case SPIRV::InstructionSet::NonSemantic_Shader_DebugInfo_100:
237:     return "NonSemantic.Shader.DebugInfo.100";
238:   case SPIRV::InstructionSet::SPV_AMD_shader_trinary_minmax:
239:     return "SPV_AMD_shader_trinary_minmax";
240:   }
```
- EN: This range implements operational logic in helpers such as getLinkStringForBuiltIn, str, drop_front, getExtInstSetName, translating backend policy into executable code.
- CN: 这一段实现了 getLinkStringForBuiltIn、str、drop_front、getExtInstSetName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-265
```cpp
241:   return "UNKNOWN_EXT_INST_SET";
242: }
243:
244: SPIRV::InstructionSet::InstructionSet
245: getExtInstSetFromString(std::string SetName) {
246:   for (auto Set :
247:        {SPIRV::InstructionSet::GLSL_std_450, SPIRV::InstructionSet::OpenCL_std,
248:         SPIRV::InstructionSet::NonSemantic_Shader_DebugInfo_100}) {
249:     if (SetName == getExtInstSetName(Set))
250:       return Set;
251:   }
252:   llvm_unreachable("UNKNOWN_EXT_INST_SET");
253: }
254:
255: std::string getExtInstName(SPIRV::InstructionSet::InstructionSet Set,
256:                            uint32_t InstructionNumber) {
257:   const SPIRV::ExtendedBuiltin *Lookup =
258:       SPIRV::lookupExtendedBuiltinBySetAndNumber(Set, InstructionNumber);
259:
260:   if (!Lookup)
261:     return "UNKNOWN_EXT_INST";
262:
263:   return Lookup->Name.str();
264: }
265: } // namespace llvm
```
- EN: This range implements operational logic in helpers such as getExtInstSetFromString, llvm_unreachable, SPIRV::lookupExtendedBuiltinBySetAndNumber, str, translating backend policy into executable code.
- CN: 这一段实现了 getExtInstSetFromString、llvm_unreachable、SPIRV::lookupExtendedBuiltinBySetAndNumber、str 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SymbolicOperand, ExtensionEntry, CapabilityEntry, EnvironmentEntry, SPIRV::lookupSymbolicOperandByCategoryAndValue, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SymbolicOperand, ExtensionEntry, CapabilityEntry, EnvironmentEntry, SPIRV::lookupSymbolicOperandByCategoryAndValue，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVBaseInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/ArrayRef.h`
  - `llvm/ADT/StringRef.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenTables.inc`
