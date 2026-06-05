# SPIRVNonSemanticDebugHandler.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVNonSemanticDebugHandler.cpp`
- Repository: `llvm-project`
- Purpose (EN): SPIRVNonSemanticDebugHandler support code for the LLVM target backend.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVNonSemanticDebugHandler.cpp - NSDI AsmPrinter handler -*- C++
 2: //-*-===//
 3: //
 4: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 5: // See https://llvm.org/LICENSE.txt for license information.
 6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 7: //
 8: //===----------------------------------------------------------------------===//
 9:
10: #include "SPIRVNonSemanticDebugHandler.h"
11: #include "MCTargetDesc/SPIRVMCTargetDesc.h"
12: #include "SPIRVSubtarget.h"
13: #include "SPIRVUtils.h"
14: #include "llvm/ADT/SmallVectorExtras.h"
15: #include "llvm/BinaryFormat/Dwarf.h"
16: #include "llvm/CodeGen/AsmPrinter.h"
17: #include "llvm/IR/DebugInfoMetadata.h"
18: #include "llvm/IR/DebugProgramInstruction.h"
19: #include "llvm/IR/Module.h"
20: #include "llvm/MC/MCInst.h"
21: #include "llvm/MC/MCStreamer.h"
22: #include "llvm/Support/Path.h"
23:
24: using namespace llvm;
25:
26: SPIRVNonSemanticDebugHandler::SPIRVNonSemanticDebugHandler(AsmPrinter &AP)
27:     : DebugHandlerBase(&AP) {}
28:
29: // Map DWARF source language codes to NonSemantic.Shader.DebugInfo.100 source
30: // language codes. Values are from the SourceLanguage enum in the
31: // NonSemantic.Shader.DebugInfo.100 specification, section 4.3.
32: unsigned SPIRVNonSemanticDebugHandler::toNSDISrcLang(unsigned DwarfSrcLang) {
33:   switch (DwarfSrcLang) {
34:   case dwarf::DW_LANG_OpenCL:
35:     return 3; // OpenCL_C
36:   case dwarf::DW_LANG_OpenCL_CPP:
37:     return 4; // OpenCL_CPP
38:   case dwarf::DW_LANG_CPP_for_OpenCL:
39:     return 6; // CPP_for_OpenCL
40:   case dwarf::DW_LANG_GLSL:
41:     return 2; // GLSL
42:   case dwarf::DW_LANG_HLSL:
43:     return 5; // HLSL
44:   case dwarf::DW_LANG_SYCL:
45:     return 7; // SYCL
46:   case dwarf::DW_LANG_Zig:
47:     return 12; // Zig
48:   default:
49:     return 0; // Unknown
50:   }
51: }
52:
53: void SPIRVNonSemanticDebugHandler::beginModule(Module *M) {
54:   // The base class sets Asm = nullptr when the module has no compile units,
55:   // and initializes lexical scope tracking otherwise.
56:   DebugHandlerBase::beginModule(M);
57:
58:   if (!Asm)
59:     return;
60:
```
- EN: This range implements operational logic in helpers such as SPIRVNonSemanticDebugHandler::SPIRVNonSemanticDebugHandler, DebugHandlerBase, SPIRVNonSemanticDebugHandler::toNSDISrcLang, SPIRVNonSemanticDebugHandler::beginModule, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVNonSemanticDebugHandler::SPIRVNonSemanticDebugHandler、DebugHandlerBase、SPIRVNonSemanticDebugHandler::toNSDISrcLang、SPIRVNonSemanticDebugHandler::beginModule 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:   // Collect compile-unit info: file paths and source languages.
 62:   for (const DICompileUnit *CU : M->debug_compile_units()) {
 63:     const DIFile *File = CU->getFile();
 64:     CompileUnitInfo Info;
 65:     if (sys::path::is_absolute(File->getFilename()))
 66:       Info.FilePath = File->getFilename();
 67:     else
 68:       sys::path::append(Info.FilePath, File->getDirectory(),
 69:                         File->getFilename());
 70:     // getName() returns the language code regardless of whether the name is
 71:     // versioned. getUnversionedName() would assert on versioned names.
 72:     Info.SpirvSourceLanguage = toNSDISrcLang(CU->getSourceLanguage().getName());
 73:     CompileUnits.push_back(std::move(Info));
 74:   }
 75:
 76:   // Collect DWARF version from module flags. For CodeView modules there is no
 77:   // "Dwarf Version" flag; DwarfVersion remains 0, which is the correct value
 78:   // for the DebugCompilationUnit DWARF Version operand in that case.
 79:   if (const NamedMDNode *Flags = M->getNamedMetadata("llvm.module.flags")) {
 80:     for (const auto *Op : Flags->operands()) {
 81:       const MDOperand &NameOp = Op->getOperand(1);
 82:       if (NameOp.equalsStr("Dwarf Version"))
 83:         DwarfVersion =
 84:             cast<ConstantInt>(
 85:                 cast<ConstantAsMetadata>(Op->getOperand(2))->getValue())
 86:                 ->getSExtValue();
 87:     }
 88:   }
 89:
 90:   // Collect basic and pointer types referenced by debug variable records.
 91:   for (const auto &F : *M) {
 92:     for (const auto &BB : F) {
 93:       for (const auto &I : BB) {
 94:         for (const DbgVariableRecord &DVR :
 95:              filterDbgVars(I.getDbgRecordRange())) {
 96:           const DIType *Ty = DVR.getVariable()->getType();
 97:           if (const auto *BT = dyn_cast<DIBasicType>(Ty)) {
 98:             BasicTypes.insert(BT);
 99:           } else if (const auto *DT = dyn_cast<DIDerivedType>(Ty)) {
100:             if (DT->getTag() == dwarf::DW_TAG_pointer_type) {
101:               PointerTypes.insert(DT);
102:               if (const auto *BT =
103:                       dyn_cast_or_null<DIBasicType>(DT->getBaseType()))
104:                 BasicTypes.insert(BT);
105:             }
106:           }
107:         }
108:       }
109:     }
110:   }
111: }
112:
113: void SPIRVNonSemanticDebugHandler::prepareModuleOutput(
114:     const SPIRVSubtarget &ST, SPIRV::ModuleAnalysisInfo &MAI) {
115:   if (CompileUnits.empty())
116:     return;
117:   if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_non_semantic_info))
118:     return;
119:
120:   // Add the extension to requirements so OpExtension is output.
```
- EN: This range implements operational logic in helpers such as getFile, getFilename, toNSDISrcLang, push_back, translating backend policy into executable code.
- CN: 这一段实现了 getFile、getFilename、toNSDISrcLang、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   MAI.Reqs.addExtension(SPIRV::Extension::SPV_KHR_non_semantic_info);
122:
123:   // Add the NonSemantic.Shader.DebugInfo.100 entry to ExtInstSetMap so that
124:   // outputOpExtInstImports() emits the OpExtInstImport instruction. Allocate a
125:   // fresh result ID for it now; the same ID is used in emitExtInst() operands.
126:   constexpr unsigned NSSet = static_cast<unsigned>(
127:       SPIRV::InstructionSet::NonSemantic_Shader_DebugInfo_100);
128:   if (!MAI.ExtInstSetMap.count(NSSet))
129:     MAI.ExtInstSetMap[NSSet] = MAI.getNextIDRegister();
130: }
131:
132: void SPIRVNonSemanticDebugHandler::emitMCInst(MCInst &Inst) {
133:   Asm->OutStreamer->emitInstruction(Inst, Asm->getSubtargetInfo());
134: }
135:
136: MCRegister
137: SPIRVNonSemanticDebugHandler::emitOpString(StringRef S,
138:                                            SPIRV::ModuleAnalysisInfo &MAI) {
139:   MCRegister Reg = MAI.getNextIDRegister();
140:   MCInst Inst;
141:   Inst.setOpcode(SPIRV::OpString);
142:   Inst.addOperand(MCOperand::createReg(Reg));
143:   addStringImm(S, Inst);
144:   emitMCInst(Inst);
145:   return Reg;
146: }
147:
148: MCRegister SPIRVNonSemanticDebugHandler::emitOpConstantI32(
149:     uint32_t Value, MCRegister I32TypeReg, SPIRV::ModuleAnalysisInfo &MAI) {
150:   auto [It, Inserted] = I32ConstantCache.try_emplace(Value);
151:   if (!Inserted)
152:     return It->second;
153:
154:   MCRegister Reg = MAI.getNextIDRegister();
155:   It->second = Reg;
156:   MCInst Inst;
157:   Inst.setOpcode(SPIRV::OpConstantI);
158:   Inst.addOperand(MCOperand::createReg(Reg));
159:   Inst.addOperand(MCOperand::createReg(I32TypeReg));
160:   Inst.addOperand(MCOperand::createImm(static_cast<int64_t>(Value)));
161:   emitMCInst(Inst);
162:   return Reg;
163: }
164:
165: MCRegister SPIRVNonSemanticDebugHandler::emitExtInst(
166:     SPIRV::NonSemanticExtInst::NonSemanticExtInst Opcode,
167:     MCRegister VoidTypeReg, MCRegister ExtInstSetReg,
168:     ArrayRef<MCRegister> Operands, SPIRV::ModuleAnalysisInfo &MAI) {
169:   MCRegister Reg = MAI.getNextIDRegister();
170:   MCInst Inst;
171:   Inst.setOpcode(SPIRV::OpExtInst);
172:   Inst.addOperand(MCOperand::createReg(Reg));
173:   Inst.addOperand(MCOperand::createReg(VoidTypeReg));
174:   Inst.addOperand(MCOperand::createReg(ExtInstSetReg));
175:   Inst.addOperand(MCOperand::createImm(static_cast<int64_t>(Opcode)));
176:   for (MCRegister R : Operands)
177:     Inst.addOperand(MCOperand::createReg(R));
178:   emitMCInst(Inst);
179:   return Reg;
180: }
```
- EN: This range implements operational logic in helpers such as addExtension, getNextIDRegister, SPIRVNonSemanticDebugHandler::emitMCInst, emitInstruction, translating backend policy into executable code.
- CN: 这一段实现了 addExtension、getNextIDRegister、SPIRVNonSemanticDebugHandler::emitMCInst、emitInstruction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:
182: MCRegister SPIRVNonSemanticDebugHandler::findOrEmitOpTypeVoid(
183:     SPIRV::ModuleAnalysisInfo &MAI) {
184:   for (const MachineInstr *MI : MAI.getMSInstrs(SPIRV::MB_TypeConstVars)) {
185:     if (MI->getOpcode() == SPIRV::OpTypeVoid)
186:       return MAI.getRegisterAlias(MI->getMF(), MI->getOperand(0).getReg());
187:   }
188:   MCRegister Reg = MAI.getNextIDRegister();
189:   MCInst Inst;
190:   Inst.setOpcode(SPIRV::OpTypeVoid);
191:   Inst.addOperand(MCOperand::createReg(Reg));
192:   emitMCInst(Inst);
193:   return Reg;
194: }
195:
196: MCRegister SPIRVNonSemanticDebugHandler::findOrEmitOpTypeInt32(
197:     SPIRV::ModuleAnalysisInfo &MAI) {
198:   for (const MachineInstr *MI : MAI.getMSInstrs(SPIRV::MB_TypeConstVars)) {
199:     if (MI->getOpcode() == SPIRV::OpTypeInt &&
200:         MI->getOperand(1).getImm() == 32 && MI->getOperand(2).getImm() == 0)
201:       return MAI.getRegisterAlias(MI->getMF(), MI->getOperand(0).getReg());
202:   }
203:   MCRegister Reg = MAI.getNextIDRegister();
204:   MCInst Inst;
205:   Inst.setOpcode(SPIRV::OpTypeInt);
206:   Inst.addOperand(MCOperand::createReg(Reg));
207:   Inst.addOperand(MCOperand::createImm(32)); // width
208:   Inst.addOperand(MCOperand::createImm(0));  // signedness (unsigned)
209:   emitMCInst(Inst);
210:   return Reg;
211: }
212:
213: void SPIRVNonSemanticDebugHandler::emitDebugTypePointer(
214:     const DIDerivedType *PT, MCRegister VoidTypeReg, MCRegister I32TypeReg,
215:     MCRegister ExtInstSetReg, MCRegister I32ZeroReg,
216:     const DenseMap<const DIBasicType *, MCRegister> &BasicTypeRegs,
217:     SPIRV::ModuleAnalysisInfo &MAI) {
218:   // A DWARF address space is required to determine the SPIR-V storage class.
219:   // Skip pointer types that do not carry one.
220:   if (!PT->getDWARFAddressSpace().has_value())
221:     return;
222:
223:   // For SPIR-V targets, Clang sets DwarfAddressSpace to the LLVM IR address
224:   // space, which addressSpaceToStorageClass expects.
225:   const auto &ST = static_cast<const SPIRVSubtarget &>(Asm->getSubtargetInfo());
226:   MCRegister StorageClassReg = emitOpConstantI32(
227:       addressSpaceToStorageClass(PT->getDWARFAddressSpace().value(), ST),
228:       I32TypeReg, MAI);
229:
230:   if (const auto *BaseType = dyn_cast_or_null<DIBasicType>(PT->getBaseType())) {
231:     auto BTIt = BasicTypeRegs.find(BaseType);
232:     if (BTIt != BasicTypeRegs.end())
233:       emitExtInst(SPIRV::NonSemanticExtInst::DebugTypePointer, VoidTypeReg,
234:                   ExtInstSetReg, {BTIt->second, StorageClassReg, I32ZeroReg},
235:                   MAI);
236:   } else {
237:     // Void pointer: use DebugInfoNone for the base type. Note that
238:     // spirv-val currently rejects DebugInfoNone as the base type of
239:     // DebugTypePointer; see issue #109287 and the DISABLED spirv-val run
240:     // in debug-type-pointer.ll.
```
- EN: This range implements operational logic in helpers such as getRegisterAlias, getNextIDRegister, setOpcode, addOperand, translating backend policy into executable code.
- CN: 这一段实现了 getRegisterAlias、getNextIDRegister、setOpcode、addOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:     MCRegister NoneReg = emitExtInst(SPIRV::NonSemanticExtInst::DebugInfoNone,
242:                                      VoidTypeReg, ExtInstSetReg, {}, MAI);
243:     emitExtInst(SPIRV::NonSemanticExtInst::DebugTypePointer, VoidTypeReg,
244:                 ExtInstSetReg, {NoneReg, StorageClassReg, I32ZeroReg}, MAI);
245:   }
246: }
247:
248: void SPIRVNonSemanticDebugHandler::emitNonSemanticDebugStrings(
249:     SPIRV::ModuleAnalysisInfo &MAI) {
250:   if (CompileUnits.empty())
251:     return;
252:   // Check that prepareModuleOutput() registered the extended instruction set.
253:   // If the subtarget does not support the extension, neither strings nor ext
254:   // insts are emitted.
255:   constexpr unsigned NSSet = static_cast<unsigned>(
256:       SPIRV::InstructionSet::NonSemantic_Shader_DebugInfo_100);
257:   if (!MAI.getExtInstSetReg(NSSet).isValid())
258:     return;
259:
260:   for (const CompileUnitInfo &Info : CompileUnits)
261:     FileStringRegs.push_back(emitOpString(Info.FilePath, MAI));
262:
263:   for (const DIBasicType *BT : BasicTypes)
264:     BasicTypeNameRegs.push_back(emitOpString(BT->getName(), MAI));
265: }
266:
267: void SPIRVNonSemanticDebugHandler::emitNonSemanticGlobalDebugInfo(
268:     SPIRV::ModuleAnalysisInfo &MAI) {
269:   if (GlobalDIEmitted || CompileUnits.empty())
270:     return;
271:   GlobalDIEmitted = true;
272:
273:   // Retrieve the ext inst set register allocated by prepareModuleOutput().
274:   constexpr unsigned NSSet = static_cast<unsigned>(
275:       SPIRV::InstructionSet::NonSemantic_Shader_DebugInfo_100);
276:   MCRegister ExtInstSetReg = MAI.getExtInstSetReg(NSSet);
277:   if (!ExtInstSetReg.isValid())
278:     return; // Extension not available.
279:
280:   MCRegister VoidTypeReg = findOrEmitOpTypeVoid(MAI);
281:   MCRegister I32TypeReg = findOrEmitOpTypeInt32(MAI);
282:
283:   // Emit integer constants shared across all NSDI instructions. The constant
284:   // cache ensures each value is emitted at most once even when referenced from
285:   // multiple instructions. All constants are pre-emitted before any DebugSource
286:   // so that the output order is: constants, then
287:   // DebugSource+DebugCompilationUnit pairs. This keeps OpConstant instructions
288:   // grouped before the OpExtInst instructions.
289:
290:   // The Version operand of DebugCompilationUnit is the version of the
291:   // NonSemantic.Shader.DebugInfo instruction set, which is 100 for
292:   // "NonSemantic.Shader.DebugInfo.100" (NonSemanticShaderDebugInfo100Version).
293:   MCRegister DebugInfoVersionReg = emitOpConstantI32(100, I32TypeReg, MAI);
294:   MCRegister DwarfVersionReg =
295:       emitOpConstantI32(static_cast<uint32_t>(DwarfVersion), I32TypeReg, MAI);
296:
297:   // Pre-emit source language constants for all compile units before entering
298:   // the DebugSource loop.
299:   SmallVector<MCRegister> SrcLangRegs =
300:       map_to_vector(CompileUnits, [&](const CompileUnitInfo &Info) {
```
- EN: This range implements operational logic in helpers such as push_back, getExtInstSetReg, findOrEmitOpTypeVoid, findOrEmitOpTypeInt32, translating backend policy into executable code.
- CN: 这一段实现了 push_back、getExtInstSetReg、findOrEmitOpTypeVoid、findOrEmitOpTypeInt32 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:         return emitOpConstantI32(Info.SpirvSourceLanguage, I32TypeReg, MAI);
302:       });
303:
304:   // Emit DebugSource and DebugCompilationUnit for each compile unit.
305:   // FileStringRegs was populated by emitNonSemanticDebugStrings() in section 7.
306:   assert(FileStringRegs.size() == CompileUnits.size() &&
307:          "FileStringRegs must be populated by emitNonSemanticDebugStrings()");
308:   for (auto [Info, FileStrReg, SrcLangReg] :
309:        llvm::zip(CompileUnits, FileStringRegs, SrcLangRegs)) {
310:     MCRegister DebugSourceReg =
311:         emitExtInst(SPIRV::NonSemanticExtInst::DebugSource, VoidTypeReg,
312:                     ExtInstSetReg, {FileStrReg}, MAI);
313:     emitExtInst(
314:         SPIRV::NonSemanticExtInst::DebugCompilationUnit, VoidTypeReg,
315:         ExtInstSetReg,
316:         {DebugInfoVersionReg, DwarfVersionReg, DebugSourceReg, SrcLangReg},
317:         MAI);
318:   }
319:
320:   // Zero constant used as the Flags operand in DebugTypeBasic and
321:   // DebugTypePointer. Cached with other i32 constants.
322:   MCRegister I32ZeroReg = emitOpConstantI32(0, I32TypeReg, MAI);
323:
324:   // Maps each DIBasicType to its DebugTypeBasic result register for use as
325:   // operands in DebugTypePointer instructions.
326:   DenseMap<const DIBasicType *, MCRegister> BasicTypeRegs;
327:
328:   // BasicTypeNameRegs was populated by emitNonSemanticDebugStrings() in
329:   // section 7.
330:   assert(
331:       BasicTypeNameRegs.size() == BasicTypes.size() &&
332:       "BasicTypeNameRegs must be populated by emitNonSemanticDebugStrings()");
333:   unsigned BTIdx = 0;
334:   for (const DIBasicType *BT : BasicTypes) {
335:     MCRegister NameReg = BasicTypeNameRegs[BTIdx++];
336:     MCRegister SizeReg = emitOpConstantI32(
337:         static_cast<uint32_t>(BT->getSizeInBits()), I32TypeReg, MAI);
338:
339:     // Map DWARF base type encodings to NSDI encoding codes per
340:     // NonSemantic.Shader.DebugInfo.100 specification, section 4.5.
341:     unsigned Encoding = 0; // Unspecified
342:     switch (BT->getEncoding()) {
343:     case dwarf::DW_ATE_address:
344:       Encoding = 1;
345:       break;
346:     case dwarf::DW_ATE_boolean:
347:       Encoding = 2;
348:       break;
349:     case dwarf::DW_ATE_float:
350:       Encoding = 3;
351:       break;
352:     case dwarf::DW_ATE_signed:
353:       Encoding = 4;
354:       break;
355:     case dwarf::DW_ATE_signed_char:
356:       Encoding = 5;
357:       break;
358:     case dwarf::DW_ATE_unsigned:
359:       Encoding = 6;
360:       break;
```
- EN: This range implements operational logic in helpers such as emitOpConstantI32, emitNonSemanticDebugStrings, llvm::zip, getSizeInBits, translating backend policy into executable code.
- CN: 这一段实现了 emitOpConstantI32、emitNonSemanticDebugStrings、llvm::zip、getSizeInBits 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-377
```cpp
361:     case dwarf::DW_ATE_unsigned_char:
362:       Encoding = 7;
363:       break;
364:     }
365:     MCRegister EncodingReg = emitOpConstantI32(Encoding, I32TypeReg, MAI);
366:
367:     MCRegister BTReg = emitExtInst(
368:         SPIRV::NonSemanticExtInst::DebugTypeBasic, VoidTypeReg, ExtInstSetReg,
369:         {NameReg, SizeReg, EncodingReg, I32ZeroReg}, MAI);
370:     BasicTypeRegs[BT] = BTReg;
371:   }
372:
373:   // Emit DebugTypePointer for each referenced pointer type.
374:   for (const DIDerivedType *PT : PointerTypes)
375:     emitDebugTypePointer(PT, VoidTypeReg, I32TypeReg, ExtInstSetReg, I32ZeroReg,
376:                          BasicTypeRegs, MAI);
377: }
```
- EN: This range implements operational logic in helpers such as emitOpConstantI32, translating backend policy into executable code.
- CN: 这一段实现了 emitOpConstantI32 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVNonSemanticDebugHandler::SPIRVNonSemanticDebugHandler, DebugHandlerBase, SPIRVNonSemanticDebugHandler::toNSDISrcLang, SPIRVNonSemanticDebugHandler::beginModule, DebugHandlerBase::beginModule, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVNonSemanticDebugHandler::SPIRVNonSemanticDebugHandler, DebugHandlerBase, SPIRVNonSemanticDebugHandler::toNSDISrcLang, SPIRVNonSemanticDebugHandler::beginModule, DebugHandlerBase::beginModule，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVNonSemanticDebugHandler.h`
  - `MCTargetDesc/SPIRVMCTargetDesc.h`
  - `SPIRVSubtarget.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/SmallVectorExtras.h`
  - `llvm/BinaryFormat/Dwarf.h`
  - `llvm/CodeGen/AsmPrinter.h`
  - `llvm/IR/DebugInfoMetadata.h`
  - `llvm/IR/DebugProgramInstruction.h`
  - `llvm/IR/Module.h`
  - `llvm/MC/MCInst.h`
  - `llvm/MC/MCStreamer.h`
  - `llvm/Support/Path.h`
