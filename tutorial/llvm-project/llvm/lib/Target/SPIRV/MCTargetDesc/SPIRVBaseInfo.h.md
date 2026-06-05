# SPIRVBaseInfo.h — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/MCTargetDesc/SPIRVBaseInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains TableGen generated enum definitions, mnemonic lookup functions, versioning capabilities extensions getters for symbolic named operands for various SPIR-V instructions.
- 目的（中文）: 该文件声明 LLVM 目标后端使用的数据结构、接口或辅助类型。
- Language: C++ header

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVBaseInfo.h - Top level SPIRV definitions -----------*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains TableGen generated enum definitions, mnemonic lookup
10: // functions, versioning/capabilities/extensions getters for symbolic/named
11: // operands for various SPIR-V instructions.
12: //
13: //===----------------------------------------------------------------------===//
14:
15: #ifndef LLVM_LIB_TARGET_SPIRV_MCTARGETDESC_SPIRVBASEINFO_H
16: #define LLVM_LIB_TARGET_SPIRV_MCTARGETDESC_SPIRVBASEINFO_H
17:
18: #include "llvm/ADT/DenseSet.h"
19: #include "llvm/ADT/SmallVector.h"
20: #include "llvm/ADT/StringRef.h"
21: #include "llvm/Support/VersionTuple.h"
22: #include <string>
23:
24: namespace llvm {
25: namespace SPIRV {
26: namespace OperandCategory {
27: #define GET_OperandCategory_DECL
28: #include "SPIRVGenTables.inc"
29: } // namespace OperandCategory
30:
31: namespace Extension {
32: #define GET_Extension_DECL
33: #include "SPIRVGenTables.inc"
34: } // namespace Extension
35:
36: namespace Capability {
37: #define GET_Capability_DECL
38: #include "SPIRVGenTables.inc"
39: } // namespace Capability
40:
41: namespace Environment {
42: #define GET_Environment_DECL
43: #include "SPIRVGenTables.inc"
44: } // namespace Environment
45:
46: namespace SourceLanguage {
47: #define GET_SourceLanguage_DECL
48: #include "SPIRVGenTables.inc"
49: } // namespace SourceLanguage
50:
51: namespace AddressingModel {
52: #define GET_AddressingModel_DECL
53: #include "SPIRVGenTables.inc"
54: } // namespace AddressingModel
55:
56: namespace ExecutionModel {
57: #define GET_ExecutionModel_DECL
58: #include "SPIRVGenTables.inc"
59: } // namespace ExecutionModel
60:
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 61-120
```cpp
 61: namespace MemoryModel {
 62: #define GET_MemoryModel_DECL
 63: #include "SPIRVGenTables.inc"
 64: } // namespace MemoryModel
 65:
 66: namespace MatrixMultiplyAccumulateOperands {
 67: #define GET_MatrixMultiplyAccumulateOperands_DECL
 68: #include "SPIRVGenTables.inc"
 69: } // namespace MatrixMultiplyAccumulateOperands
 70:
 71: namespace ExecutionMode {
 72: #define GET_ExecutionMode_DECL
 73: #include "SPIRVGenTables.inc"
 74: } // namespace ExecutionMode
 75:
 76: namespace StorageClass {
 77: #define GET_StorageClass_DECL
 78: #include "SPIRVGenTables.inc"
 79: } // namespace StorageClass
 80:
 81: namespace Dim {
 82: #define GET_Dim_DECL
 83: #include "SPIRVGenTables.inc"
 84: } // namespace Dim
 85:
 86: namespace SamplerAddressingMode {
 87: #define GET_SamplerAddressingMode_DECL
 88: #include "SPIRVGenTables.inc"
 89: } // namespace SamplerAddressingMode
 90:
 91: namespace SamplerFilterMode {
 92: #define GET_SamplerFilterMode_DECL
 93: #include "SPIRVGenTables.inc"
 94: } // namespace SamplerFilterMode
 95:
 96: namespace ImageFormat {
 97: #define GET_ImageFormat_DECL
 98: #include "SPIRVGenTables.inc"
 99: } // namespace ImageFormat
100:
101: namespace ImageChannelOrder {
102: #define GET_ImageChannelOrder_DECL
103: #include "SPIRVGenTables.inc"
104: } // namespace ImageChannelOrder
105:
106: namespace ImageChannelDataType {
107: #define GET_ImageChannelDataType_DECL
108: #include "SPIRVGenTables.inc"
109: } // namespace ImageChannelDataType
110:
111: namespace ImageOperand {
112: #define GET_ImageOperand_DECL
113: #include "SPIRVGenTables.inc"
114: } // namespace ImageOperand
115:
116: namespace FPFastMathMode {
117: #define GET_FPFastMathMode_DECL
118: #include "SPIRVGenTables.inc"
119: } // namespace FPFastMathMode
120:
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 121-180
```cpp
121: namespace FPRoundingMode {
122: #define GET_FPRoundingMode_DECL
123: #include "SPIRVGenTables.inc"
124: } // namespace FPRoundingMode
125:
126: namespace LinkageType {
127: #define GET_LinkageType_DECL
128: #include "SPIRVGenTables.inc"
129: } // namespace LinkageType
130:
131: namespace AccessQualifier {
132: #define GET_AccessQualifier_DECL
133: #include "SPIRVGenTables.inc"
134: } // namespace AccessQualifier
135:
136: namespace FunctionParameterAttribute {
137: #define GET_FunctionParameterAttribute_DECL
138: #include "SPIRVGenTables.inc"
139: } // namespace FunctionParameterAttribute
140:
141: namespace Decoration {
142: #define GET_Decoration_DECL
143: #include "SPIRVGenTables.inc"
144: } // namespace Decoration
145:
146: namespace BuiltIn {
147: #define GET_BuiltIn_DECL
148: #include "SPIRVGenTables.inc"
149: } // namespace BuiltIn
150:
151: namespace SelectionControl {
152: #define GET_SelectionControl_DECL
153: #include "SPIRVGenTables.inc"
154: } // namespace SelectionControl
155:
156: namespace LoopControl {
157: #define GET_LoopControl_DECL
158: #include "SPIRVGenTables.inc"
159: } // namespace LoopControl
160:
161: namespace FunctionControl {
162: #define GET_FunctionControl_DECL
163: #include "SPIRVGenTables.inc"
164: } // namespace FunctionControl
165:
166: namespace MemorySemantics {
167: #define GET_MemorySemantics_DECL
168: #include "SPIRVGenTables.inc"
169: } // namespace MemorySemantics
170:
171: namespace MemoryOperand {
172: #define GET_MemoryOperand_DECL
173: #include "SPIRVGenTables.inc"
174: } // namespace MemoryOperand
175:
176: namespace Scope {
177: #define GET_Scope_DECL
178: #include "SPIRVGenTables.inc"
179: } // namespace Scope
180:
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 181-240
```cpp
181: namespace GroupOperation {
182: #define GET_GroupOperation_DECL
183: #include "SPIRVGenTables.inc"
184: } // namespace GroupOperation
185:
186: namespace KernelEnqueueFlags {
187: #define GET_KernelEnqueueFlags_DECL
188: #include "SPIRVGenTables.inc"
189: } // namespace KernelEnqueueFlags
190:
191: namespace KernelProfilingInfo {
192: #define GET_KernelProfilingInfo_DECL
193: #include "SPIRVGenTables.inc"
194: } // namespace KernelProfilingInfo
195:
196: namespace InstructionSet {
197: #define GET_InstructionSet_DECL
198: #include "SPIRVGenTables.inc"
199: } // namespace InstructionSet
200:
201: namespace OpenCLExtInst {
202: #define GET_OpenCLExtInst_DECL
203: #include "SPIRVGenTables.inc"
204: } // namespace OpenCLExtInst
205:
206: namespace GLSLExtInst {
207: #define GET_GLSLExtInst_DECL
208: #include "SPIRVGenTables.inc"
209: } // namespace GLSLExtInst
210:
211: namespace NonSemanticExtInst {
212: #define GET_NonSemanticExtInst_DECL
213: #include "SPIRVGenTables.inc"
214: } // namespace NonSemanticExtInst
215:
216: namespace Opcode {
217: #define GET_Opcode_DECL
218: #include "SPIRVGenTables.inc"
219: } // namespace Opcode
220:
221: namespace CooperativeMatrixLayout {
222: #define GET_CooperativeMatrixLayout_DECL
223: #include "SPIRVGenTables.inc"
224: } // namespace CooperativeMatrixLayout
225:
226: namespace CooperativeMatrixOperands {
227: #define GET_CooperativeMatrixOperands_DECL
228: #include "SPIRVGenTables.inc"
229: } // namespace CooperativeMatrixOperands
230:
231: namespace SpecConstantOpOperands {
232: #define GET_SpecConstantOpOperands_DECL
233: #include "SPIRVGenTables.inc"
234: } // namespace SpecConstantOpOperands
235:
236: namespace FPEncoding {
237: #define GET_FPEncoding_DECL
238: #include "SPIRVGenTables.inc"
239: } // namespace FPEncoding
240:
```
- EN: This range contains connective backend logic, including helper statements, control flow, and data movement needed by the surrounding implementation.
- CN: 这一段包含衔接性的后端逻辑，包括辅助语句、控制流和数据处理，用于支撑周围实现。

### Lines 241-300
```cpp
241: struct ExtendedBuiltin {
242:   StringRef Name;
243:   InstructionSet::InstructionSet Set;
244:   uint32_t Number;
245: };
246:
247: enum InstFlags {
248:   // It is a half type
249:   INST_PRINTER_WIDTH16 = 1,
250:   // It is a 64-bit type
251:   INST_PRINTER_WIDTH64 = INST_PRINTER_WIDTH16 << 1,
252:
253: };
254: } // namespace SPIRV
255:
256: using CapabilityList = SmallVector<SPIRV::Capability::Capability, 8>;
257: using ExtensionList = SmallVector<SPIRV::Extension::Extension, 8>;
258: using EnvironmentList = SmallVector<SPIRV::Environment::Environment, 8>;
259:
260: using ExtensionSet = DenseSet<SPIRV::Extension::Extension>;
261:
262: std::string
263: getSymbolicOperandMnemonic(SPIRV::OperandCategory::OperandCategory Category,
264:                            int32_t Value);
265: VersionTuple
266: getSymbolicOperandMinVersion(SPIRV::OperandCategory::OperandCategory Category,
267:                              uint32_t Value);
268: VersionTuple
269: getSymbolicOperandMaxVersion(SPIRV::OperandCategory::OperandCategory Category,
270:                              uint32_t Value);
271: CapabilityList
272: getSymbolicOperandCapabilities(SPIRV::OperandCategory::OperandCategory Category,
273:                                uint32_t Value);
274: EnvironmentList getSymbolicOperandAllowedEnvironments(
275:     SPIRV::OperandCategory::OperandCategory Category, uint32_t Value);
276: CapabilityList
277: getCapabilitiesEnabledByExtension(SPIRV::Extension::Extension Extension);
278: ExtensionList
279: getSymbolicOperandExtensions(SPIRV::OperandCategory::OperandCategory Category,
280:                              uint32_t Value);
281: std::string getLinkStringForBuiltIn(SPIRV::BuiltIn::BuiltIn BuiltInValue);
282:
283: bool getSpirvBuiltInIdByName(StringRef Name, SPIRV::BuiltIn::BuiltIn &BI);
284:
285: std::string getExtInstSetName(SPIRV::InstructionSet::InstructionSet Set);
286: SPIRV::InstructionSet::InstructionSet
287: getExtInstSetFromString(std::string SetName);
288: std::string getExtInstName(SPIRV::InstructionSet::InstructionSet Set,
289:                            uint32_t InstructionNumber);
290:
291: // Return a string representation of the operands from startIndex onwards.
292: // Templated to allow both MachineInstr and MCInst to use the same logic.
293: template <class InstType>
294: std::string getSPIRVStringOperand(const InstType &MI, unsigned StartIndex) {
295:   std::string s; // Iteratively append to this string.
296:
297:   const unsigned NumOps = MI.getNumOperands();
298:   bool IsFinished = false;
299:   for (unsigned i = StartIndex; i < NumOps && !IsFinished; ++i) {
300:     const auto &Op = MI.getOperand(i);
```
- EN: This range defines or declares important types such as ExtendedBuiltin, InstFlags, getCapabilitiesEnabledByExtension, getLinkStringForBuiltIn, shaping the data model used by SPIRVBaseInfo.h.
- CN: 这一段定义或声明了 ExtendedBuiltin、InstFlags、getCapabilitiesEnabledByExtension、getLinkStringForBuiltIn 等关键类型，构成 SPIRVBaseInfo.h 使用的数据模型。

### Lines 301-317
```cpp
301:     if (!Op.isImm()) // Stop if we hit a register operand.
302:       break;
303:     assert((Op.getImm() >> 32) == 0 && "Imm operand should be i32 word");
304:     const uint32_t Imm = Op.getImm(); // Each i32 word is up to 4 characters.
305:     for (unsigned ShiftAmount = 0; ShiftAmount < 32; ShiftAmount += 8) {
306:       char c = (Imm >> ShiftAmount) & 0xff;
307:       if (c == 0) { // Stop if we hit a null-terminator character.
308:         IsFinished = true;
309:         break;
310:       }
311:       s += c; // Otherwise, append the character to the result string.
312:     }
313:   }
314:   return s;
315: }
316: } // namespace llvm
317: #endif // LLVM_LIB_TARGET_SPIRV_MCTARGETDESC_SPIRVBASEINFO_H
```
- EN: This range declares interfaces or inline helpers such as assert, getImm, defining how other backend pieces interact with this header.
- CN: 这一段声明了 assert、getImm 等接口或内联辅助函数，定义其他后端组件如何与该头文件协作。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include ExtendedBuiltin, InstFlags, getCapabilitiesEnabledByExtension, getLinkStringForBuiltIn, getSpirvBuiltInIdByName, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 ExtendedBuiltin, InstFlags, getCapabilitiesEnabledByExtension, getLinkStringForBuiltIn, getSpirvBuiltInIdByName，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseSet.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/Support/VersionTuple.h`
- System/standard headers / 系统或标准头文件:
  - `string`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
  - `SPIRVGenTables.inc`
