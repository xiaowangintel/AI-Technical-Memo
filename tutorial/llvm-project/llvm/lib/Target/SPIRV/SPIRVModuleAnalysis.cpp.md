# SPIRVModuleAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVModuleAnalysis.cpp`
- Repository: `llvm-project`
- Purpose (EN): The analysis collects instructions that should be output at the module level and performs the global register numbering.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-120
```cpp
  1: //===- SPIRVModuleAnalysis.cpp - analysis of global instrs & regs - C++ -*-===//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8: //
  9: // The analysis collects instructions that should be output at the module level
 10: // and performs the global register numbering.
 11: //
 12: // The results of this analysis are used in AsmPrinter to rename registers
 13: // globally and to output required instructions at the module level.
 14: //
 15: //===----------------------------------------------------------------------===//
 16:
 17: // TODO: uses or report_fatal_error (which is also deprecated) /
 18: //       ReportFatalUsageError in this file should be refactored, as per LLVM
 19: //       best practices, to rely on the Diagnostic infrastructure.
 20:
 21: #include "SPIRVModuleAnalysis.h"
 22: #include "MCTargetDesc/SPIRVBaseInfo.h"
 23: #include "MCTargetDesc/SPIRVMCTargetDesc.h"
 24: #include "SPIRV.h"
 25: #include "SPIRVSubtarget.h"
 26: #include "SPIRVTargetMachine.h"
 27: #include "SPIRVUtils.h"
 28: #include "llvm/ADT/STLExtras.h"
 29: #include "llvm/CodeGen/MachineModuleInfo.h"
 30: #include "llvm/CodeGen/TargetPassConfig.h"
 31:
 32: using namespace llvm;
 33:
 34: #define DEBUG_TYPE "spirv-module-analysis"
 35:
 36: static cl::opt<bool>
 37:     SPVDumpDeps("spv-dump-deps",
 38:                 cl::desc("Dump MIR with SPIR-V dependencies info"),
 39:                 cl::Optional, cl::init(false));
 40:
 41: static cl::list<SPIRV::Capability::Capability>
 42:     AvoidCapabilities("avoid-spirv-capabilities",
 43:                       cl::desc("SPIR-V capabilities to avoid if there are "
 44:                                "other options enabling a feature"),
 45:                       cl::Hidden,
 46:                       cl::values(clEnumValN(SPIRV::Capability::Shader, "Shader",
 47:                                             "SPIR-V Shader capability")));
 48: // Use sets instead of cl::list to check "if contains" condition
 49: struct AvoidCapabilitiesSet {
 50:   SmallSet<SPIRV::Capability::Capability, 4> S;
 51:   AvoidCapabilitiesSet() { S.insert_range(AvoidCapabilities); }
 52: };
 53:
 54: char llvm::SPIRVModuleAnalysis::ID = 0;
 55:
 56: INITIALIZE_PASS(SPIRVModuleAnalysis, DEBUG_TYPE, "SPIRV module analysis", true,
 57:                 true)
 58:
 59: // Retrieve an unsigned from an MDNode with a list of them as operands.
 60: static unsigned getMetadataUInt(MDNode *MdNode, unsigned OpIndex,
 61:                                 unsigned DefaultVal = 0) {
 62:   if (MdNode && OpIndex < MdNode->getNumOperands()) {
 63:     const auto &Op = MdNode->getOperand(OpIndex);
 64:     return mdconst::extract<ConstantInt>(Op)->getZExtValue();
 65:   }
 66:   return DefaultVal;
 67: }
 68:
 69: static SPIRV::Requirements
 70: getSymbolicOperandRequirements(SPIRV::OperandCategory::OperandCategory Category,
 71:                                unsigned i, const SPIRVSubtarget &ST,
 72:                                SPIRV::RequirementHandler &Reqs) {
 73:   // A set of capabilities to avoid if there is another option.
 74:   AvoidCapabilitiesSet AvoidCaps;
 75:   if (!ST.isShader())
 76:     AvoidCaps.S.insert(SPIRV::Capability::Shader);
 77:   else
 78:     AvoidCaps.S.insert(SPIRV::Capability::Kernel);
 79:
 80:   VersionTuple ReqMinVer = getSymbolicOperandMinVersion(Category, i);
 81:   VersionTuple ReqMaxVer = getSymbolicOperandMaxVersion(Category, i);
 82:   VersionTuple SPIRVVersion = ST.getSPIRVVersion();
 83:   bool MinVerOK = SPIRVVersion.empty() || SPIRVVersion >= ReqMinVer;
 84:   bool MaxVerOK =
 85:       ReqMaxVer.empty() || SPIRVVersion.empty() || SPIRVVersion <= ReqMaxVer;
 86:   CapabilityList ReqCaps = getSymbolicOperandCapabilities(Category, i);
 87:   ExtensionList ReqExts = getSymbolicOperandExtensions(Category, i);
 88:   if (ReqCaps.empty()) {
 89:     if (ReqExts.empty()) {
 90:       if (MinVerOK && MaxVerOK)
 91:         return {true, {}, {}, ReqMinVer, ReqMaxVer};
 92:       return {false, {}, {}, VersionTuple(), VersionTuple()};
 93:     }
 94:   } else if (MinVerOK && MaxVerOK) {
 95:     if (ReqCaps.size() == 1) {
 96:       auto Cap = ReqCaps[0];
 97:       if (Reqs.isCapabilityAvailable(Cap)) {
 98:         ReqExts.append(getSymbolicOperandExtensions(
 99:             SPIRV::OperandCategory::CapabilityOperand, Cap));
100:         return {true, {Cap}, std::move(ReqExts), ReqMinVer, ReqMaxVer};
101:       }
102:     } else {
103:       // By SPIR-V specification: "If an instruction, enumerant, or other
104:       // feature specifies multiple enabling capabilities, only one such
105:       // capability needs to be declared to use the feature." However, one
106:       // capability may be preferred over another. We use command line
107:       // argument(s) and AvoidCapabilities to avoid selection of certain
108:       // capabilities if there are other options.
109:       CapabilityList UseCaps;
110:       for (auto Cap : ReqCaps)
111:         if (Reqs.isCapabilityAvailable(Cap))
112:           UseCaps.push_back(Cap);
113:       for (size_t i = 0, Sz = UseCaps.size(); i < Sz; ++i) {
114:         auto Cap = UseCaps[i];
115:         if (i == Sz - 1 || !AvoidCaps.S.contains(Cap)) {
116:           ReqExts.append(getSymbolicOperandExtensions(
117:               SPIRV::OperandCategory::CapabilityOperand, Cap));
118:           return {true, {Cap}, std::move(ReqExts), ReqMinVer, ReqMaxVer};
119:         }
120:       }
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 121-240
```cpp
121:     }
122:   }
123:   // If there are no capabilities, or we can't satisfy the version or
124:   // capability requirements, use the list of extensions (if the subtarget
125:   // can handle them all).
126:   if (llvm::all_of(ReqExts, [&ST](const SPIRV::Extension::Extension &Ext) {
127:         return ST.canUseExtension(Ext);
128:       })) {
129:     return {true,
130:             {},
131:             std::move(ReqExts),
132:             VersionTuple(),
133:             VersionTuple()}; // TODO: add versions to extensions.
134:   }
135:   return {false, {}, {}, VersionTuple(), VersionTuple()};
136: }
137:
138: void SPIRVModuleAnalysis::setBaseInfo(const Module &M) {
139:   MAI.MaxID = 0;
140:   for (int i = 0; i < SPIRV::NUM_MODULE_SECTIONS; i++)
141:     MAI.MS[i].clear();
142:   MAI.RegisterAliasTable.clear();
143:   MAI.InstrsToDelete.clear();
144:   MAI.GlobalObjMap.clear();
145:   MAI.GlobalVarList.clear();
146:   MAI.ExtInstSetMap.clear();
147:   MAI.Reqs.clear();
148:   MAI.Reqs.initAvailableCapabilities(*ST);
149:
150:   // TODO: determine memory model and source language from the configuratoin.
151:   if (auto MemModel = M.getNamedMetadata("spirv.MemoryModel")) {
152:     auto MemMD = MemModel->getOperand(0);
153:     MAI.Addr = static_cast<SPIRV::AddressingModel::AddressingModel>(
154:         getMetadataUInt(MemMD, 0));
155:     MAI.Mem =
156:         static_cast<SPIRV::MemoryModel::MemoryModel>(getMetadataUInt(MemMD, 1));
157:   } else {
158:     // TODO: Add support for VulkanMemoryModel.
159:     MAI.Mem = ST->isShader() ? SPIRV::MemoryModel::GLSL450
160:                              : SPIRV::MemoryModel::OpenCL;
161:     if (MAI.Mem == SPIRV::MemoryModel::OpenCL) {
162:       unsigned PtrSize = ST->getPointerSize();
163:       MAI.Addr = PtrSize == 32   ? SPIRV::AddressingModel::Physical32
164:                  : PtrSize == 64 ? SPIRV::AddressingModel::Physical64
165:                                  : SPIRV::AddressingModel::Logical;
166:     } else {
167:       // TODO: Add support for PhysicalStorageBufferAddress.
168:       MAI.Addr = SPIRV::AddressingModel::Logical;
169:     }
170:   }
171:   // Get the OpenCL version number from metadata.
172:   // TODO: support other source languages.
173:   if (auto VerNode = M.getNamedMetadata("opencl.ocl.version")) {
174:     MAI.SrcLang = SPIRV::SourceLanguage::OpenCL_C;
175:     // Construct version literal in accordance with SPIRV-LLVM-Translator.
176:     // TODO: support multiple OCL version metadata.
177:     assert(VerNode->getNumOperands() > 0 && "Invalid SPIR");
178:     auto VersionMD = VerNode->getOperand(0);
179:     unsigned MajorNum = getMetadataUInt(VersionMD, 0, 2);
180:     unsigned MinorNum = getMetadataUInt(VersionMD, 1);
181:     unsigned RevNum = getMetadataUInt(VersionMD, 2);
182:     // Prevent Major part of OpenCL version to be 0
183:     MAI.SrcLangVersion =
184:         (std::max(1U, MajorNum) * 100 + MinorNum) * 1000 + RevNum;
185:     // When opencl.cxx.version is also present, validate compatibility
186:     // and use C++ for OpenCL as source language with the C++ version.
187:     if (auto *CxxVerNode = M.getNamedMetadata("opencl.cxx.version")) {
188:       assert(CxxVerNode->getNumOperands() > 0 && "Invalid SPIR");
189:       auto *CxxMD = CxxVerNode->getOperand(0);
190:       unsigned CxxVer =
191:           (getMetadataUInt(CxxMD, 0) * 100 + getMetadataUInt(CxxMD, 1)) * 1000 +
192:           getMetadataUInt(CxxMD, 2);
193:       if ((MAI.SrcLangVersion == 200000 && CxxVer == 100000) ||
194:           (MAI.SrcLangVersion == 300000 && CxxVer == 202100000)) {
195:         MAI.SrcLang = SPIRV::SourceLanguage::CPP_for_OpenCL;
196:         MAI.SrcLangVersion = CxxVer;
197:       } else {
198:         report_fatal_error(
199:             "opencl cxx version is not compatible with opencl c version!");
200:       }
201:     }
202:   } else {
203:     // If there is no information about OpenCL version we are forced to generate
204:     // OpenCL 1.0 by default for the OpenCL environment to avoid puzzling
205:     // run-times with Unknown/0.0 version output. For a reference, LLVM-SPIRV
206:     // Translator avoids potential issues with run-times in a similar manner.
207:     if (!ST->isShader()) {
208:       MAI.SrcLang = SPIRV::SourceLanguage::OpenCL_CPP;
209:       MAI.SrcLangVersion = 100000;
210:     } else {
211:       MAI.SrcLang = SPIRV::SourceLanguage::Unknown;
212:       MAI.SrcLangVersion = 0;
213:     }
214:   }
215:
216:   if (auto ExtNode = M.getNamedMetadata("opencl.used.extensions")) {
217:     for (unsigned I = 0, E = ExtNode->getNumOperands(); I != E; ++I) {
218:       MDNode *MD = ExtNode->getOperand(I);
219:       if (!MD || MD->getNumOperands() == 0)
220:         continue;
221:       for (unsigned J = 0, N = MD->getNumOperands(); J != N; ++J)
222:         MAI.SrcExt.insert(cast<MDString>(MD->getOperand(J))->getString());
223:     }
224:   }
225:
226:   // Update required capabilities for this memory model, addressing model and
227:   // source language.
228:   MAI.Reqs.getAndAddRequirements(SPIRV::OperandCategory::MemoryModelOperand,
229:                                  MAI.Mem, *ST);
230:   MAI.Reqs.getAndAddRequirements(SPIRV::OperandCategory::SourceLanguageOperand,
231:                                  MAI.SrcLang, *ST);
232:   MAI.Reqs.getAndAddRequirements(SPIRV::OperandCategory::AddressingModelOperand,
233:                                  MAI.Addr, *ST);
234:
235:   if (MAI.Mem == SPIRV::MemoryModel::VulkanKHR)
236:     MAI.Reqs.addExtension(SPIRV::Extension::SPV_KHR_vulkan_memory_model);
237:
238:   if (!ST->isShader()) {
239:     // TODO: check if it's required by default.
240:     MAI.ExtInstSetMap[static_cast<unsigned>(
```
- EN: This range implements operational logic in helpers such as canUseExtension, SPIRVModuleAnalysis::setBaseInfo, clear, initAvailableCapabilities, translating backend policy into executable code.
- CN: 这一段实现了 canUseExtension、SPIRVModuleAnalysis::setBaseInfo、clear、initAvailableCapabilities 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-360
```cpp
241:         SPIRV::InstructionSet::OpenCL_std)] = MAI.getNextIDRegister();
242:   }
243: }
244:
245: // Appends the signature of the decoration instructions that decorate R to
246: // Signature.
247: static void appendDecorationsForReg(const MachineRegisterInfo &MRI, Register R,
248:                                     InstrSignature &Signature) {
249:   for (MachineInstr &UseMI : MRI.use_instructions(R)) {
250:     // We don't handle OpDecorateId because getting the register alias for the
251:     // ID can cause problems, and we do not need it for now.
252:     if (UseMI.getOpcode() != SPIRV::OpDecorate &&
253:         UseMI.getOpcode() != SPIRV::OpMemberDecorate)
254:       continue;
255:
256:     for (unsigned I = 0; I < UseMI.getNumOperands(); ++I) {
257:       const MachineOperand &MO = UseMI.getOperand(I);
258:       if (MO.isReg())
259:         continue;
260:       Signature.push_back(hash_value(MO));
261:     }
262:   }
263: }
264:
265: // Returns a representation of an instruction as a vector of MachineOperand
266: // hash values, see llvm::hash_value(const MachineOperand &MO) for details.
267: // This creates a signature of the instruction with the same content
268: // that MachineOperand::isIdenticalTo uses for comparison.
269: static InstrSignature instrToSignature(const MachineInstr &MI,
270:                                        SPIRV::ModuleAnalysisInfo &MAI,
271:                                        bool UseDefReg) {
272:   Register DefReg;
273:   InstrSignature Signature{MI.getOpcode()};
274:   for (unsigned i = 0; i < MI.getNumOperands(); ++i) {
275:     // The only decorations that can be applied more than once to a given <id>
276:     // or structure member are FuncParamAttr (38), UserSemantic (5635),
277:     // CacheControlLoadINTEL (6442), and CacheControlStoreINTEL (6443). For all
278:     // the rest of decorations, we will only add to the signature the Opcode,
279:     // the id to which it applies, and the decoration id, disregarding any
280:     // decoration flags. This will ensure that any subsequent decoration with
281:     // the same id will be deemed as a duplicate. Then, at the call site, we
282:     // will be able to handle duplicates in the best way.
283:     unsigned Opcode = MI.getOpcode();
284:     if ((Opcode == SPIRV::OpDecorate) && i >= 2) {
285:       unsigned DecorationID = MI.getOperand(1).getImm();
286:       if (DecorationID != SPIRV::Decoration::FuncParamAttr &&
287:           DecorationID != SPIRV::Decoration::UserSemantic &&
288:           DecorationID != SPIRV::Decoration::CacheControlLoadINTEL &&
289:           DecorationID != SPIRV::Decoration::CacheControlStoreINTEL)
290:         continue;
291:     }
292:     const MachineOperand &MO = MI.getOperand(i);
293:     size_t h;
294:     if (MO.isReg()) {
295:       if (!UseDefReg && MO.isDef()) {
296:         assert(!DefReg.isValid() && "Multiple def registers.");
297:         DefReg = MO.getReg();
298:         continue;
299:       }
300:       Register RegAlias = MAI.getRegisterAlias(MI.getMF(), MO.getReg());
301:       if (!RegAlias.isValid()) {
302:         LLVM_DEBUG({
303:           dbgs() << "Unexpectedly, no global id found for the operand ";
304:           MO.print(dbgs());
305:           dbgs() << "\nInstruction: ";
306:           MI.print(dbgs());
307:           dbgs() << "\n";
308:         });
309:         report_fatal_error("All v-regs must have been mapped to global id's");
310:       }
311:       // mimic llvm::hash_value(const MachineOperand &MO)
312:       h = hash_combine(MO.getType(), (unsigned)RegAlias, MO.getSubReg(),
313:                        MO.isDef());
314:     } else {
315:       h = hash_value(MO);
316:     }
317:     Signature.push_back(h);
318:   }
319:
320:   if (DefReg.isValid()) {
321:     // Decorations change the semantics of the current instruction. So two
322:     // identical instruction with different decorations cannot be merged. That
323:     // is why we add the decorations to the signature.
324:     appendDecorationsForReg(MI.getMF()->getRegInfo(), DefReg, Signature);
325:   }
326:   return Signature;
327: }
328:
329: bool SPIRVModuleAnalysis::isDeclSection(const MachineRegisterInfo &MRI,
330:                                         const MachineInstr &MI) {
331:   unsigned Opcode = MI.getOpcode();
332:   switch (Opcode) {
333:   case SPIRV::OpTypeForwardPointer:
334:     // omit now, collect later
335:     return false;
336:   case SPIRV::OpVariable:
337:     return static_cast<SPIRV::StorageClass::StorageClass>(
338:                MI.getOperand(2).getImm()) != SPIRV::StorageClass::Function;
339:   case SPIRV::OpFunction:
340:   case SPIRV::OpFunctionParameter:
341:     return true;
342:   }
343:   if (GR->hasConstFunPtr() && Opcode == SPIRV::OpUndef) {
344:     Register DefReg = MI.getOperand(0).getReg();
345:     for (MachineInstr &UseMI : MRI.use_instructions(DefReg)) {
346:       if (UseMI.getOpcode() != SPIRV::OpConstantFunctionPointerINTEL)
347:         continue;
348:       // it's a dummy definition, FP constant refers to a function,
349:       // and this is resolved in another way; let's skip this definition
350:       assert(UseMI.getOperand(2).isReg() &&
351:              UseMI.getOperand(2).getReg() == DefReg);
352:       MAI.setSkipEmission(&MI);
353:       return false;
354:     }
355:   }
356:   return TII->isTypeDeclInstr(MI) || TII->isConstantInstr(MI) ||
357:          TII->isInlineAsmDefInstr(MI);
358: }
359:
360: // This is a special case of a function pointer refering to a possibly
```
- EN: This range implements operational logic in helpers such as getNextIDRegister, getOpcode, getNumOperands, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getNextIDRegister、getOpcode、getNumOperands、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-480
```cpp
361: // forward function declaration. The operand is a dummy OpUndef that
362: // requires a special treatment.
363: void SPIRVModuleAnalysis::visitFunPtrUse(
364:     Register OpReg, InstrGRegsMap &SignatureToGReg,
365:     std::map<const Value *, unsigned> &GlobalToGReg, const MachineFunction *MF,
366:     const MachineInstr &MI) {
367:   const MachineOperand *OpFunDef =
368:       GR->getFunctionDefinitionByUse(&MI.getOperand(2));
369:   assert(OpFunDef && OpFunDef->isReg());
370:   // find the actual function definition and number it globally in advance
371:   const MachineInstr *OpDefMI = OpFunDef->getParent();
372:   assert(OpDefMI && OpDefMI->getOpcode() == SPIRV::OpFunction);
373:   const MachineFunction *FunDefMF = OpDefMI->getParent()->getParent();
374:   const MachineRegisterInfo &FunDefMRI = FunDefMF->getRegInfo();
375:   do {
376:     visitDecl(FunDefMRI, SignatureToGReg, GlobalToGReg, FunDefMF, *OpDefMI);
377:     OpDefMI = OpDefMI->getNextNode();
378:   } while (OpDefMI && (OpDefMI->getOpcode() == SPIRV::OpFunction ||
379:                        OpDefMI->getOpcode() == SPIRV::OpFunctionParameter));
380:   // associate the function pointer with the newly assigned global number
381:   MCRegister GlobalFunDefReg =
382:       MAI.getRegisterAlias(FunDefMF, OpFunDef->getReg());
383:   assert(GlobalFunDefReg.isValid() &&
384:          "Function definition must refer to a global register");
385:   MAI.setRegisterAlias(MF, OpReg, GlobalFunDefReg);
386: }
387:
388: // Depth first recursive traversal of dependencies. Repeated visits are guarded
389: // by MAI.hasRegisterAlias().
390: void SPIRVModuleAnalysis::visitDecl(
391:     const MachineRegisterInfo &MRI, InstrGRegsMap &SignatureToGReg,
392:     std::map<const Value *, unsigned> &GlobalToGReg, const MachineFunction *MF,
393:     const MachineInstr &MI) {
394:   unsigned Opcode = MI.getOpcode();
395:
396:   // Process each operand of the instruction to resolve dependencies
397:   for (const MachineOperand &MO : MI.operands()) {
398:     if (!MO.isReg() || MO.isDef())
399:       continue;
400:     Register OpReg = MO.getReg();
401:     // Handle function pointers special case
402:     if (Opcode == SPIRV::OpConstantFunctionPointerINTEL &&
403:         MRI.getRegClass(OpReg) == &SPIRV::pIDRegClass) {
404:       visitFunPtrUse(OpReg, SignatureToGReg, GlobalToGReg, MF, MI);
405:       continue;
406:     }
407:     // Skip already processed instructions
408:     if (MAI.hasRegisterAlias(MF, MO.getReg()))
409:       continue;
410:     // Recursively visit dependencies
411:     if (const MachineInstr *OpDefMI = MRI.getUniqueVRegDef(OpReg)) {
412:       if (isDeclSection(MRI, *OpDefMI))
413:         visitDecl(MRI, SignatureToGReg, GlobalToGReg, MF, *OpDefMI);
414:       continue;
415:     }
416:     // Handle the unexpected case of no unique definition for the SPIR-V
417:     // instruction
418:     LLVM_DEBUG({
419:       dbgs() << "Unexpectedly, no unique definition for the operand ";
420:       MO.print(dbgs());
421:       dbgs() << "\nInstruction: ";
422:       MI.print(dbgs());
423:       dbgs() << "\n";
424:     });
425:     report_fatal_error(
426:         "No unique definition is found for the virtual register");
427:   }
428:
429:   MCRegister GReg;
430:   bool IsFunDef = false;
431:   if (TII->isSpecConstantInstr(MI)) {
432:     GReg = MAI.getNextIDRegister();
433:     MAI.MS[SPIRV::MB_TypeConstVars].push_back(&MI);
434:   } else if (Opcode == SPIRV::OpFunction ||
435:              Opcode == SPIRV::OpFunctionParameter) {
436:     GReg = handleFunctionOrParameter(MF, MI, GlobalToGReg, IsFunDef);
437:   } else if (Opcode == SPIRV::OpTypeStruct ||
438:              Opcode == SPIRV::OpConstantComposite) {
439:     GReg = handleTypeDeclOrConstant(MI, SignatureToGReg);
440:     const MachineInstr *NextInstr = MI.getNextNode();
441:     while (NextInstr &&
442:            ((Opcode == SPIRV::OpTypeStruct &&
443:              NextInstr->getOpcode() == SPIRV::OpTypeStructContinuedINTEL) ||
444:             (Opcode == SPIRV::OpConstantComposite &&
445:              NextInstr->getOpcode() ==
446:                  SPIRV::OpConstantCompositeContinuedINTEL))) {
447:       MCRegister Tmp = handleTypeDeclOrConstant(*NextInstr, SignatureToGReg);
448:       MAI.setRegisterAlias(MF, NextInstr->getOperand(0).getReg(), Tmp);
449:       MAI.setSkipEmission(NextInstr);
450:       NextInstr = NextInstr->getNextNode();
451:     }
452:   } else if (TII->isTypeDeclInstr(MI) || TII->isConstantInstr(MI) ||
453:              TII->isInlineAsmDefInstr(MI)) {
454:     GReg = handleTypeDeclOrConstant(MI, SignatureToGReg);
455:   } else if (Opcode == SPIRV::OpVariable) {
456:     GReg = handleVariable(MF, MI, GlobalToGReg);
457:   } else {
458:     LLVM_DEBUG({
459:       dbgs() << "\nInstruction: ";
460:       MI.print(dbgs());
461:       dbgs() << "\n";
462:     });
463:     llvm_unreachable("Unexpected instruction is visited");
464:   }
465:   MAI.setRegisterAlias(MF, MI.getOperand(0).getReg(), GReg);
466:   if (!IsFunDef)
467:     MAI.setSkipEmission(&MI);
468: }
469:
470: MCRegister SPIRVModuleAnalysis::handleFunctionOrParameter(
471:     const MachineFunction *MF, const MachineInstr &MI,
472:     std::map<const Value *, unsigned> &GlobalToGReg, bool &IsFunDef) {
473:   const Value *GObj = GR->getGlobalObject(MF, MI.getOperand(0).getReg());
474:   assert(GObj && "Unregistered global definition");
475:   const Function *F = dyn_cast<Function>(GObj);
476:   if (!F)
477:     F = dyn_cast<Argument>(GObj)->getParent();
478:   assert(F && "Expected a reference to a function or an argument");
479:   IsFunDef = !F->isDeclaration();
480:   auto [It, Inserted] = GlobalToGReg.try_emplace(GObj);
```
- EN: This range implements operational logic in helpers such as getFunctionDefinitionByUse, assert, getParent, getRegInfo, translating backend policy into executable code.
- CN: 这一段实现了 getFunctionDefinitionByUse、assert、getParent、getRegInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-600
```cpp
481:   if (!Inserted)
482:     return It->second;
483:   MCRegister GReg = MAI.getNextIDRegister();
484:   It->second = GReg;
485:   if (!IsFunDef)
486:     MAI.MS[SPIRV::MB_ExtFuncDecls].push_back(&MI);
487:   return GReg;
488: }
489:
490: MCRegister
491: SPIRVModuleAnalysis::handleTypeDeclOrConstant(const MachineInstr &MI,
492:                                               InstrGRegsMap &SignatureToGReg) {
493:   InstrSignature MISign = instrToSignature(MI, MAI, false);
494:   auto [It, Inserted] = SignatureToGReg.try_emplace(MISign);
495:   if (!Inserted)
496:     return It->second;
497:   MCRegister GReg = MAI.getNextIDRegister();
498:   It->second = GReg;
499:   MAI.MS[SPIRV::MB_TypeConstVars].push_back(&MI);
500:   return GReg;
501: }
502:
503: MCRegister SPIRVModuleAnalysis::handleVariable(
504:     const MachineFunction *MF, const MachineInstr &MI,
505:     std::map<const Value *, unsigned> &GlobalToGReg) {
506:   MAI.GlobalVarList.push_back(&MI);
507:   const Value *GObj = GR->getGlobalObject(MF, MI.getOperand(0).getReg());
508:   assert(GObj && "Unregistered global definition");
509:   auto [It, Inserted] = GlobalToGReg.try_emplace(GObj);
510:   if (!Inserted)
511:     return It->second;
512:   MCRegister GReg = MAI.getNextIDRegister();
513:   It->second = GReg;
514:   MAI.MS[SPIRV::MB_TypeConstVars].push_back(&MI);
515:   if (const auto *GV = dyn_cast<GlobalVariable>(GObj))
516:     MAI.GlobalObjMap[GV] = GReg;
517:   return GReg;
518: }
519:
520: void SPIRVModuleAnalysis::collectDeclarations(const Module &M) {
521:   InstrGRegsMap SignatureToGReg;
522:   std::map<const Value *, unsigned> GlobalToGReg;
523:   for (const Function &F : M) {
524:     MachineFunction *MF = MMI->getMachineFunction(F);
525:     if (!MF)
526:       continue;
527:     const MachineRegisterInfo &MRI = MF->getRegInfo();
528:     unsigned PastHeader = 0;
529:     for (MachineBasicBlock &MBB : *MF) {
530:       for (MachineInstr &MI : MBB) {
531:         if (MI.getNumOperands() == 0)
532:           continue;
533:         unsigned Opcode = MI.getOpcode();
534:         if (Opcode == SPIRV::OpFunction) {
535:           if (PastHeader == 0) {
536:             PastHeader = 1;
537:             continue;
538:           }
539:         } else if (Opcode == SPIRV::OpFunctionParameter) {
540:           if (PastHeader < 2)
541:             continue;
542:         } else if (PastHeader > 0) {
543:           PastHeader = 2;
544:         }
545:
546:         const MachineOperand &DefMO = MI.getOperand(0);
547:         switch (Opcode) {
548:         case SPIRV::OpExtension:
549:           MAI.Reqs.addExtension(SPIRV::Extension::Extension(DefMO.getImm()));
550:           MAI.setSkipEmission(&MI);
551:           break;
552:         case SPIRV::OpCapability:
553:           MAI.Reqs.addCapability(SPIRV::Capability::Capability(DefMO.getImm()));
554:           MAI.setSkipEmission(&MI);
555:           if (PastHeader > 0)
556:             PastHeader = 2;
557:           break;
558:         default:
559:           if (DefMO.isReg() && isDeclSection(MRI, MI) &&
560:               !MAI.hasRegisterAlias(MF, DefMO.getReg()))
561:             visitDecl(MRI, SignatureToGReg, GlobalToGReg, MF, MI);
562:         }
563:       }
564:     }
565:   }
566: }
567:
568: // Look for IDs declared with Import linkage, and map the corresponding function
569: // to the register defining that variable (which will usually be the result of
570: // an OpFunction). This lets us call externally imported functions using
571: // the correct ID registers.
572: void SPIRVModuleAnalysis::collectFuncNames(MachineInstr &MI,
573:                                            const Function *F) {
574:   if (MI.getOpcode() == SPIRV::OpDecorate) {
575:     // If it's got Import linkage.
576:     auto Dec = MI.getOperand(1).getImm();
577:     if (Dec == SPIRV::Decoration::LinkageAttributes) {
578:       auto Lnk = MI.getOperand(MI.getNumOperands() - 1).getImm();
579:       if (Lnk == SPIRV::LinkageType::Import) {
580:         // Map imported function name to function ID register.
581:         const Function *ImportedFunc =
582:             F->getParent()->getFunction(getStringImm(MI, 2));
583:         Register Target = MI.getOperand(0).getReg();
584:         MAI.GlobalObjMap[ImportedFunc] =
585:             MAI.getRegisterAlias(MI.getMF(), Target);
586:       }
587:     }
588:   } else if (MI.getOpcode() == SPIRV::OpFunction) {
589:     // Record all internal OpFunction declarations.
590:     Register Reg = MI.defs().begin()->getReg();
591:     MCRegister GlobalReg = MAI.getRegisterAlias(MI.getMF(), Reg);
592:     assert(GlobalReg.isValid());
593:     MAI.GlobalObjMap[F] = GlobalReg;
594:   }
595: }
596:
597: // Collect the given instruction in the specified MS. We assume global register
598: // numbering has already occurred by this point. We can directly compare reg
599: // arguments when detecting duplicates.
600: static void collectOtherInstr(MachineInstr &MI, SPIRV::ModuleAnalysisInfo &MAI,
```
- EN: This range implements operational logic in helpers such as getNextIDRegister, push_back, instrToSignature, try_emplace, translating backend policy into executable code.
- CN: 这一段实现了 getNextIDRegister、push_back、instrToSignature、try_emplace 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-720
```cpp
601:                               SPIRV::ModuleSectionType MSType, InstrTraces &IS,
602:                               bool Append = true) {
603:   MAI.setSkipEmission(&MI);
604:   InstrSignature MISign = instrToSignature(MI, MAI, true);
605:   auto FoundMI = IS.insert(std::move(MISign));
606:   if (!FoundMI.second) {
607:     if (MI.getOpcode() == SPIRV::OpDecorate) {
608:       assert(MI.getNumOperands() >= 2 &&
609:              "Decoration instructions must have at least 2 operands");
610:       assert(MSType == SPIRV::MB_Annotations &&
611:              "Only OpDecorate instructions can be duplicates");
612:       // For FPFastMathMode decoration, we need to merge the flags of the
613:       // duplicate decoration with the original one, so we need to find the
614:       // original instruction that has the same signature. For the rest of
615:       // instructions, we will simply skip the duplicate.
616:       if (MI.getOperand(1).getImm() != SPIRV::Decoration::FPFastMathMode)
617:         return; // Skip duplicates of other decorations.
618:
619:       const SPIRV::InstrList &Decorations = MAI.MS[MSType];
620:       for (const MachineInstr *OrigMI : Decorations) {
621:         if (instrToSignature(*OrigMI, MAI, true) == MISign) {
622:           assert(OrigMI->getNumOperands() == MI.getNumOperands() &&
623:                  "Original instruction must have the same number of operands");
624:           assert(
625:               OrigMI->getNumOperands() == 3 &&
626:               "FPFastMathMode decoration must have 3 operands for OpDecorate");
627:           unsigned OrigFlags = OrigMI->getOperand(2).getImm();
628:           unsigned NewFlags = MI.getOperand(2).getImm();
629:           if (OrigFlags == NewFlags)
630:             return; // No need to merge, the flags are the same.
631:
632:           // Emit warning about possible conflict between flags.
633:           unsigned FinalFlags = OrigFlags | NewFlags;
634:           llvm::errs()
635:               << "Warning: Conflicting FPFastMathMode decoration flags "
636:                  "in instruction: "
637:               << *OrigMI << "Original flags: " << OrigFlags
638:               << ", new flags: " << NewFlags
639:               << ". They will be merged on a best effort basis, but not "
640:                  "validated. Final flags: "
641:               << FinalFlags << "\n";
642:           MachineInstr *OrigMINonConst = const_cast<MachineInstr *>(OrigMI);
643:           MachineOperand &OrigFlagsOp = OrigMINonConst->getOperand(2);
644:           OrigFlagsOp = MachineOperand::CreateImm(FinalFlags);
645:           return; // Merge done, so we found a duplicate; don't add it to MAI.MS
646:         }
647:       }
648:       assert(false && "No original instruction found for the duplicate "
649:                       "OpDecorate, but we found one in IS.");
650:     }
651:     return; // insert failed, so we found a duplicate; don't add it to MAI.MS
652:   }
653:   // No duplicates, so add it.
654:   if (Append)
655:     MAI.MS[MSType].push_back(&MI);
656:   else
657:     MAI.MS[MSType].insert(MAI.MS[MSType].begin(), &MI);
658: }
659:
660: // Some global instructions make reference to function-local ID regs, so cannot
661: // be correctly collected until these registers are globally numbered.
662: void SPIRVModuleAnalysis::processOtherInstrs(const Module &M) {
663:   InstrTraces IS;
664:   for (const Function &F : M) {
665:     if (F.isDeclaration())
666:       continue;
667:     MachineFunction *MF = MMI->getMachineFunction(F);
668:     assert(MF);
669:
670:     for (MachineBasicBlock &MBB : *MF)
671:       for (MachineInstr &MI : MBB) {
672:         if (MAI.getSkipEmission(&MI))
673:           continue;
674:         const unsigned OpCode = MI.getOpcode();
675:         if (OpCode == SPIRV::OpString) {
676:           collectOtherInstr(MI, MAI, SPIRV::MB_DebugStrings, IS);
677:         } else if (OpCode == SPIRV::OpExtInst && MI.getOperand(2).isImm() &&
678:                    MI.getOperand(2).getImm() ==
679:                        SPIRV::InstructionSet::
680:                            NonSemantic_Shader_DebugInfo_100) {
681:           // TODO: This branch is dead. SPIRVNonSemanticDebugHandler emits NSDI
682:           // instructions directly as MCInsts at print time; no
683:           // MachineInstructions with the NSDI ext set are created anymore.
684:           // Remove this block and
685:           // MB_NonSemanticGlobalDI once per-function NSDI emission is confirmed
686:           // not to need MIR routing.
687:           MachineOperand Ins = MI.getOperand(3);
688:           namespace NS = SPIRV::NonSemanticExtInst;
689:           static constexpr int64_t GlobalNonSemanticDITy[] = {
690:               NS::DebugSource, NS::DebugCompilationUnit, NS::DebugInfoNone,
691:               NS::DebugTypeBasic, NS::DebugTypePointer};
692:           bool IsGlobalDI = false;
693:           for (unsigned Idx = 0; Idx < std::size(GlobalNonSemanticDITy); ++Idx)
694:             IsGlobalDI |= Ins.getImm() == GlobalNonSemanticDITy[Idx];
695:           if (IsGlobalDI)
696:             collectOtherInstr(MI, MAI, SPIRV::MB_NonSemanticGlobalDI, IS);
697:         } else if (OpCode == SPIRV::OpName || OpCode == SPIRV::OpMemberName) {
698:           collectOtherInstr(MI, MAI, SPIRV::MB_DebugNames, IS);
699:         } else if (OpCode == SPIRV::OpEntryPoint) {
700:           collectOtherInstr(MI, MAI, SPIRV::MB_EntryPoints, IS);
701:         } else if (TII->isAliasingInstr(MI)) {
702:           collectOtherInstr(MI, MAI, SPIRV::MB_AliasingInsts, IS);
703:         } else if (TII->isDecorationInstr(MI)) {
704:           collectOtherInstr(MI, MAI, SPIRV::MB_Annotations, IS);
705:           collectFuncNames(MI, &F);
706:         } else if (TII->isConstantInstr(MI)) {
707:           // Now OpSpecConstant*s are not in DT,
708:           // but they need to be collected anyway.
709:           collectOtherInstr(MI, MAI, SPIRV::MB_TypeConstVars, IS);
710:         } else if (OpCode == SPIRV::OpFunction) {
711:           collectFuncNames(MI, &F);
712:         } else if (OpCode == SPIRV::OpTypeForwardPointer) {
713:           collectOtherInstr(MI, MAI, SPIRV::MB_TypeConstVars, IS, false);
714:         }
715:       }
716:   }
717: }
718:
719: // Number registers in all functions globally from 0 onwards and store
720: // the result in global register alias table. Some registers are already
```
- EN: This range implements operational logic in helpers such as setSkipEmission, instrToSignature, insert, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 setSkipEmission、instrToSignature、insert、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-840
```cpp
721: // numbered.
722: void SPIRVModuleAnalysis::numberRegistersGlobally(const Module &M) {
723:   for (const Function &F : M) {
724:     if (F.isDeclaration())
725:       continue;
726:     MachineFunction *MF = MMI->getMachineFunction(F);
727:     assert(MF);
728:     for (MachineBasicBlock &MBB : *MF) {
729:       for (MachineInstr &MI : MBB) {
730:         for (MachineOperand &Op : MI.operands()) {
731:           if (!Op.isReg())
732:             continue;
733:           Register Reg = Op.getReg();
734:           if (MAI.hasRegisterAlias(MF, Reg))
735:             continue;
736:           MCRegister NewReg = MAI.getNextIDRegister();
737:           MAI.setRegisterAlias(MF, Reg, NewReg);
738:         }
739:         if (MI.getOpcode() != SPIRV::OpExtInst)
740:           continue;
741:         auto Set = MI.getOperand(2).getImm();
742:         auto [It, Inserted] = MAI.ExtInstSetMap.try_emplace(Set);
743:         if (Inserted)
744:           It->second = MAI.getNextIDRegister();
745:       }
746:     }
747:   }
748: }
749:
750: // RequirementHandler implementations.
751: void SPIRV::RequirementHandler::getAndAddRequirements(
752:     SPIRV::OperandCategory::OperandCategory Category, uint32_t i,
753:     const SPIRVSubtarget &ST) {
754:   addRequirements(getSymbolicOperandRequirements(Category, i, ST, *this));
755: }
756:
757: void SPIRV::RequirementHandler::recursiveAddCapabilities(
758:     const CapabilityList &ToPrune) {
759:   for (const auto &Cap : ToPrune) {
760:     AllCaps.insert(Cap);
761:     CapabilityList ImplicitDecls =
762:         getSymbolicOperandCapabilities(OperandCategory::CapabilityOperand, Cap);
763:     recursiveAddCapabilities(ImplicitDecls);
764:   }
765: }
766:
767: void SPIRV::RequirementHandler::addCapabilities(const CapabilityList &ToAdd) {
768:   for (const auto &Cap : ToAdd) {
769:     bool IsNewlyInserted = AllCaps.insert(Cap).second;
770:     if (!IsNewlyInserted) // Don't re-add if it's already been declared.
771:       continue;
772:     CapabilityList ImplicitDecls =
773:         getSymbolicOperandCapabilities(OperandCategory::CapabilityOperand, Cap);
774:     recursiveAddCapabilities(ImplicitDecls);
775:     MinimalCaps.push_back(Cap);
776:   }
777: }
778:
779: void SPIRV::RequirementHandler::addRequirements(
780:     const SPIRV::Requirements &Req) {
781:   if (!Req.IsSatisfiable)
782:     report_fatal_error("Adding SPIR-V requirements this target can't satisfy.");
783:
784:   if (Req.Cap.has_value())
785:     addCapabilities({Req.Cap.value()});
786:
787:   addExtensions(Req.Exts);
788:
789:   if (!Req.MinVer.empty()) {
790:     if (!MaxVersion.empty() && Req.MinVer > MaxVersion) {
791:       LLVM_DEBUG(dbgs() << "Conflicting version requirements: >= " << Req.MinVer
792:                         << " and <= " << MaxVersion << "\n");
793:       report_fatal_error("Adding SPIR-V requirements that can't be satisfied.");
794:     }
795:
796:     if (MinVersion.empty() || Req.MinVer > MinVersion)
797:       MinVersion = Req.MinVer;
798:   }
799:
800:   if (!Req.MaxVer.empty()) {
801:     if (!MinVersion.empty() && Req.MaxVer < MinVersion) {
802:       LLVM_DEBUG(dbgs() << "Conflicting version requirements: <= " << Req.MaxVer
803:                         << " and >= " << MinVersion << "\n");
804:       report_fatal_error("Adding SPIR-V requirements that can't be satisfied.");
805:     }
806:
807:     if (MaxVersion.empty() || Req.MaxVer < MaxVersion)
808:       MaxVersion = Req.MaxVer;
809:   }
810: }
811:
812: void SPIRV::RequirementHandler::checkSatisfiable(
813:     const SPIRVSubtarget &ST) const {
814:   // Report as many errors as possible before aborting the compilation.
815:   bool IsSatisfiable = true;
816:   auto TargetVer = ST.getSPIRVVersion();
817:
818:   if (!MaxVersion.empty() && !TargetVer.empty() && MaxVersion < TargetVer) {
819:     LLVM_DEBUG(
820:         dbgs() << "Target SPIR-V version too high for required features\n"
821:                << "Required max version: " << MaxVersion << " target version "
822:                << TargetVer << "\n");
823:     IsSatisfiable = false;
824:   }
825:
826:   if (!MinVersion.empty() && !TargetVer.empty() && MinVersion > TargetVer) {
827:     LLVM_DEBUG(dbgs() << "Target SPIR-V version too low for required features\n"
828:                       << "Required min version: " << MinVersion
829:                       << " target version " << TargetVer << "\n");
830:     IsSatisfiable = false;
831:   }
832:
833:   if (!MinVersion.empty() && !MaxVersion.empty() && MinVersion > MaxVersion) {
834:     LLVM_DEBUG(
835:         dbgs()
836:         << "Version is too low for some features and too high for others.\n"
837:         << "Required SPIR-V min version: " << MinVersion
838:         << " required SPIR-V max version " << MaxVersion << "\n");
839:     IsSatisfiable = false;
840:   }
```
- EN: This range implements operational logic in helpers such as SPIRVModuleAnalysis::numberRegistersGlobally, getMachineFunction, assert, getReg, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVModuleAnalysis::numberRegistersGlobally、getMachineFunction、assert、getReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 841-960
```cpp
841:
842:   AvoidCapabilitiesSet AvoidCaps;
843:   if (!ST.isShader())
844:     AvoidCaps.S.insert(SPIRV::Capability::Shader);
845:   else
846:     AvoidCaps.S.insert(SPIRV::Capability::Kernel);
847:
848:   for (auto Cap : MinimalCaps) {
849:     if (AvailableCaps.contains(Cap) && !AvoidCaps.S.contains(Cap))
850:       continue;
851:     LLVM_DEBUG(dbgs() << "Capability not supported: "
852:                       << getSymbolicOperandMnemonic(
853:                              OperandCategory::CapabilityOperand, Cap)
854:                       << "\n");
855:     IsSatisfiable = false;
856:   }
857:
858:   for (auto Ext : AllExtensions) {
859:     if (ST.canUseExtension(Ext))
860:       continue;
861:     LLVM_DEBUG(dbgs() << "Extension not supported: "
862:                       << getSymbolicOperandMnemonic(
863:                              OperandCategory::ExtensionOperand, Ext)
864:                       << "\n");
865:     IsSatisfiable = false;
866:   }
867:
868:   if (!IsSatisfiable)
869:     report_fatal_error("Unable to meet SPIR-V requirements for this target.");
870: }
871:
872: // Add the given capabilities and all their implicitly defined capabilities too.
873: void SPIRV::RequirementHandler::addAvailableCaps(const CapabilityList &ToAdd) {
874:   for (const auto Cap : ToAdd)
875:     if (AvailableCaps.insert(Cap).second)
876:       addAvailableCaps(getSymbolicOperandCapabilities(
877:           SPIRV::OperandCategory::CapabilityOperand, Cap));
878: }
879:
880: void SPIRV::RequirementHandler::removeCapabilityIf(
881:     const Capability::Capability ToRemove,
882:     const Capability::Capability IfPresent) {
883:   if (AllCaps.contains(IfPresent))
884:     AllCaps.erase(ToRemove);
885: }
886:
887: namespace llvm {
888: namespace SPIRV {
889: void RequirementHandler::initAvailableCapabilities(const SPIRVSubtarget &ST) {
890:   // Provided by both all supported Vulkan versions and OpenCl.
891:   addAvailableCaps({Capability::Shader, Capability::Linkage, Capability::Int8,
892:                     Capability::Int16});
893:
894:   if (ST.isAtLeastSPIRVVer(VersionTuple(1, 3)))
895:     addAvailableCaps({Capability::GroupNonUniform,
896:                       Capability::GroupNonUniformVote,
897:                       Capability::GroupNonUniformArithmetic,
898:                       Capability::GroupNonUniformBallot,
899:                       Capability::GroupNonUniformClustered,
900:                       Capability::GroupNonUniformShuffle,
901:                       Capability::GroupNonUniformShuffleRelative,
902:                       Capability::GroupNonUniformQuad});
903:
904:   if (ST.isAtLeastSPIRVVer(VersionTuple(1, 6)))
905:     addAvailableCaps({Capability::DotProduct, Capability::DotProductInputAll,
906:                       Capability::DotProductInput4x8Bit,
907:                       Capability::DotProductInput4x8BitPacked,
908:                       Capability::DemoteToHelperInvocation});
909:
910:   // Add capabilities enabled by extensions.
911:   for (auto Extension : ST.getAllAvailableExtensions()) {
912:     CapabilityList EnabledCapabilities =
913:         getCapabilitiesEnabledByExtension(Extension);
914:     addAvailableCaps(EnabledCapabilities);
915:   }
916:
917:   if (!ST.isShader()) {
918:     initAvailableCapabilitiesForOpenCL(ST);
919:     return;
920:   }
921:
922:   if (ST.isShader()) {
923:     initAvailableCapabilitiesForVulkan(ST);
924:     return;
925:   }
926:
927:   report_fatal_error("Unimplemented environment for SPIR-V generation.");
928: }
929:
930: void RequirementHandler::initAvailableCapabilitiesForOpenCL(
931:     const SPIRVSubtarget &ST) {
932:   // Add the min requirements for different OpenCL and SPIR-V versions.
933:   addAvailableCaps({Capability::Addresses, Capability::Float16Buffer,
934:                     Capability::Kernel, Capability::Vector16,
935:                     Capability::Groups, Capability::GenericPointer,
936:                     Capability::StorageImageWriteWithoutFormat,
937:                     Capability::StorageImageReadWithoutFormat});
938:   if (ST.hasOpenCLFullProfile())
939:     addAvailableCaps({Capability::Int64, Capability::Int64Atomics});
940:   if (ST.hasOpenCLImageSupport()) {
941:     addAvailableCaps({Capability::ImageBasic, Capability::LiteralSampler,
942:                       Capability::Image1D, Capability::SampledBuffer,
943:                       Capability::ImageBuffer});
944:     if (ST.isAtLeastOpenCLVer(VersionTuple(2, 0)))
945:       addAvailableCaps({Capability::ImageReadWrite});
946:   }
947:   if (ST.isAtLeastSPIRVVer(VersionTuple(1, 1)) &&
948:       ST.isAtLeastOpenCLVer(VersionTuple(2, 2)))
949:     addAvailableCaps({Capability::SubgroupDispatch, Capability::PipeStorage});
950:   if (ST.isAtLeastSPIRVVer(VersionTuple(1, 4)))
951:     addAvailableCaps({Capability::DenormPreserve, Capability::DenormFlushToZero,
952:                       Capability::SignedZeroInfNanPreserve,
953:                       Capability::RoundingModeRTE,
954:                       Capability::RoundingModeRTZ});
955:   // TODO: verify if this needs some checks.
956:   addAvailableCaps({Capability::Float16, Capability::Float64});
957:
958:   // TODO: add OpenCL extensions.
959: }
960:
```
- EN: This range implements operational logic in helpers such as insert, report_fatal_error, SPIRV::RequirementHandler::addAvailableCaps, erase, translating backend policy into executable code.
- CN: 这一段实现了 insert、report_fatal_error、SPIRV::RequirementHandler::addAvailableCaps、erase 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 961-1080
```cpp
 961: void RequirementHandler::initAvailableCapabilitiesForVulkan(
 962:     const SPIRVSubtarget &ST) {
 963:
 964:   // Core in Vulkan 1.1 and earlier.
 965:   addAvailableCaps({Capability::Int64,
 966:                     Capability::Float16,
 967:                     Capability::Float64,
 968:                     Capability::GroupNonUniform,
 969:                     Capability::Image1D,
 970:                     Capability::SampledBuffer,
 971:                     Capability::ImageBuffer,
 972:                     Capability::UniformBufferArrayDynamicIndexing,
 973:                     Capability::SampledImageArrayDynamicIndexing,
 974:                     Capability::StorageBufferArrayDynamicIndexing,
 975:                     Capability::StorageImageArrayDynamicIndexing,
 976:                     Capability::DerivativeControl,
 977:                     Capability::MinLod,
 978:                     Capability::ImageQuery,
 979:                     Capability::ImageGatherExtended,
 980:                     Capability::Addresses,
 981:                     Capability::VulkanMemoryModelKHR,
 982:                     Capability::StorageImageExtendedFormats,
 983:                     Capability::StorageImageMultisample,
 984:                     Capability::ImageMSArray});
 985:
 986:   // Became core in Vulkan 1.2
 987:   if (ST.isAtLeastSPIRVVer(VersionTuple(1, 5))) {
 988:     addAvailableCaps(
 989:         {Capability::ShaderNonUniformEXT, Capability::RuntimeDescriptorArrayEXT,
 990:          Capability::InputAttachmentArrayDynamicIndexingEXT,
 991:          Capability::UniformTexelBufferArrayDynamicIndexingEXT,
 992:          Capability::StorageTexelBufferArrayDynamicIndexingEXT,
 993:          Capability::UniformBufferArrayNonUniformIndexingEXT,
 994:          Capability::SampledImageArrayNonUniformIndexingEXT,
 995:          Capability::StorageBufferArrayNonUniformIndexingEXT,
 996:          Capability::StorageImageArrayNonUniformIndexingEXT,
 997:          Capability::InputAttachmentArrayNonUniformIndexingEXT,
 998:          Capability::UniformTexelBufferArrayNonUniformIndexingEXT,
 999:          Capability::StorageTexelBufferArrayNonUniformIndexingEXT});
1000:   }
1001:
1002:   // Became core in Vulkan 1.3
1003:   if (ST.isAtLeastSPIRVVer(VersionTuple(1, 6)))
1004:     addAvailableCaps({Capability::StorageImageWriteWithoutFormat,
1005:                       Capability::StorageImageReadWithoutFormat});
1006: }
1007:
1008: } // namespace SPIRV
1009: } // namespace llvm
1010:
1011: // Add the required capabilities from a decoration instruction (including
1012: // BuiltIns).
1013: static void addOpDecorateReqs(const MachineInstr &MI, unsigned DecIndex,
1014:                               SPIRV::RequirementHandler &Reqs,
1015:                               const SPIRVSubtarget &ST) {
1016:   int64_t DecOp = MI.getOperand(DecIndex).getImm();
1017:   auto Dec = static_cast<SPIRV::Decoration::Decoration>(DecOp);
1018:   Reqs.addRequirements(getSymbolicOperandRequirements(
1019:       SPIRV::OperandCategory::DecorationOperand, Dec, ST, Reqs));
1020:
1021:   if (Dec == SPIRV::Decoration::BuiltIn) {
1022:     int64_t BuiltInOp = MI.getOperand(DecIndex + 1).getImm();
1023:     auto BuiltIn = static_cast<SPIRV::BuiltIn::BuiltIn>(BuiltInOp);
1024:     Reqs.addRequirements(getSymbolicOperandRequirements(
1025:         SPIRV::OperandCategory::BuiltInOperand, BuiltIn, ST, Reqs));
1026:   } else if (Dec == SPIRV::Decoration::LinkageAttributes) {
1027:     int64_t LinkageOp = MI.getOperand(MI.getNumOperands() - 1).getImm();
1028:     SPIRV::LinkageType::LinkageType LnkType =
1029:         static_cast<SPIRV::LinkageType::LinkageType>(LinkageOp);
1030:     if (LnkType == SPIRV::LinkageType::LinkOnceODR)
1031:       Reqs.addExtension(SPIRV::Extension::SPV_KHR_linkonce_odr);
1032:     else if (LnkType == SPIRV::LinkageType::WeakAMD) {
1033:       Reqs.addExtension(SPIRV::Extension::SPV_AMD_weak_linkage);
1034:       Reqs.addCapability(SPIRV::Capability::WeakLinkageAMD);
1035:     }
1036:   } else if (Dec == SPIRV::Decoration::CacheControlLoadINTEL ||
1037:              Dec == SPIRV::Decoration::CacheControlStoreINTEL) {
1038:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_cache_controls);
1039:   } else if (Dec == SPIRV::Decoration::HostAccessINTEL) {
1040:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_global_variable_host_access);
1041:   } else if (Dec == SPIRV::Decoration::InitModeINTEL ||
1042:              Dec == SPIRV::Decoration::ImplementInRegisterMapINTEL) {
1043:     Reqs.addExtension(
1044:         SPIRV::Extension::SPV_INTEL_global_variable_fpga_decorations);
1045:   } else if (Dec == SPIRV::Decoration::NonUniformEXT) {
1046:     Reqs.addRequirements(SPIRV::Capability::ShaderNonUniformEXT);
1047:   } else if (Dec == SPIRV::Decoration::FPMaxErrorDecorationINTEL) {
1048:     Reqs.addRequirements(SPIRV::Capability::FPMaxErrorINTEL);
1049:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_fp_max_error);
1050:   } else if (Dec == SPIRV::Decoration::FPFastMathMode) {
1051:     if (ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2)) {
1052:       Reqs.addRequirements(SPIRV::Capability::FloatControls2);
1053:       Reqs.addExtension(SPIRV::Extension::SPV_KHR_float_controls2);
1054:     }
1055:   }
1056: }
1057:
1058: // Add requirements for image handling.
1059: static void addOpTypeImageReqs(const MachineInstr &MI,
1060:                                SPIRV::RequirementHandler &Reqs,
1061:                                const SPIRVSubtarget &ST) {
1062:   assert(MI.getNumOperands() >= 8 && "Insufficient operands for OpTypeImage");
1063:   // The operand indices used here are based on the OpTypeImage layout, which
1064:   // the MachineInstr follows as well.
1065:   int64_t ImgFormatOp = MI.getOperand(7).getImm();
1066:   auto ImgFormat = static_cast<SPIRV::ImageFormat::ImageFormat>(ImgFormatOp);
1067:   Reqs.getAndAddRequirements(SPIRV::OperandCategory::ImageFormatOperand,
1068:                              ImgFormat, ST);
1069:
1070:   bool IsArrayed = MI.getOperand(4).getImm() == 1;
1071:   bool IsMultisampled = MI.getOperand(5).getImm() == 1;
1072:   bool NoSampler = MI.getOperand(6).getImm() == 2;
1073:   // Add dimension requirements.
1074:   assert(MI.getOperand(2).isImm());
1075:   switch (MI.getOperand(2).getImm()) {
1076:   case SPIRV::Dim::DIM_1D:
1077:     Reqs.addRequirements(NoSampler ? SPIRV::Capability::Image1D
1078:                                    : SPIRV::Capability::Sampled1D);
1079:     break;
1080:   case SPIRV::Dim::DIM_2D:
```
- EN: This range implements operational logic in helpers such as getOperand, addExtension, addCapability, addRequirements, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、addExtension、addCapability、addRequirements 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1200
```cpp
1081:     if (IsMultisampled && NoSampler)
1082:       Reqs.addRequirements(SPIRV::Capability::StorageImageMultisample);
1083:     if (IsMultisampled && IsArrayed)
1084:       Reqs.addRequirements(SPIRV::Capability::ImageMSArray);
1085:     break;
1086:   case SPIRV::Dim::DIM_3D:
1087:     break;
1088:   case SPIRV::Dim::DIM_Cube:
1089:     Reqs.addRequirements(SPIRV::Capability::Shader);
1090:     if (IsArrayed)
1091:       Reqs.addRequirements(NoSampler ? SPIRV::Capability::ImageCubeArray
1092:                                      : SPIRV::Capability::SampledCubeArray);
1093:     break;
1094:   case SPIRV::Dim::DIM_Rect:
1095:     Reqs.addRequirements(NoSampler ? SPIRV::Capability::ImageRect
1096:                                    : SPIRV::Capability::SampledRect);
1097:     break;
1098:   case SPIRV::Dim::DIM_Buffer:
1099:     Reqs.addRequirements(NoSampler ? SPIRV::Capability::ImageBuffer
1100:                                    : SPIRV::Capability::SampledBuffer);
1101:     break;
1102:   case SPIRV::Dim::DIM_SubpassData:
1103:     Reqs.addRequirements(SPIRV::Capability::InputAttachment);
1104:     break;
1105:   }
1106:
1107:   // Check if the sampled type is a 64-bit integer, which requires
1108:   // Int64ImageEXT capability.
1109:   assert(MI.getOperand(1).isReg());
1110:   const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1111:   SPIRVTypeInst SampledTypeDef = MRI.getVRegDef(MI.getOperand(1).getReg());
1112:   if (SampledTypeDef.isTypeIntN(64)) {
1113:     Reqs.addCapability(SPIRV::Capability::Int64ImageEXT);
1114:     Reqs.addExtension(SPIRV::Extension::SPV_EXT_shader_image_int64);
1115:   }
1116:
1117:   // Has optional access qualifier.
1118:   if (!ST.isShader()) {
1119:     if (MI.getNumOperands() > 8 &&
1120:         MI.getOperand(8).getImm() == SPIRV::AccessQualifier::ReadWrite)
1121:       Reqs.addRequirements(SPIRV::Capability::ImageReadWrite);
1122:     else
1123:       Reqs.addRequirements(SPIRV::Capability::ImageBasic);
1124:   }
1125: }
1126:
1127: static bool isBFloat16Type(SPIRVTypeInst TypeDef) {
1128:   return TypeDef && TypeDef->getNumOperands() == 3 &&
1129:          TypeDef->getOpcode() == SPIRV::OpTypeFloat &&
1130:          TypeDef->getOperand(1).getImm() == 16 &&
1131:          TypeDef->getOperand(2).getImm() == SPIRV::FPEncoding::BFloat16KHR;
1132: }
1133:
1134: // Add requirements for handling atomic float instructions
1135: #define ATOM_FLT_REQ_EXT_MSG(ExtName)                                          \
1136:   "The atomic float instruction requires the following SPIR-V "                \
1137:   "extension: SPV_EXT_shader_atomic_float" ExtName
1138: static void AddAtomicVectorFloatRequirements(const MachineInstr &MI,
1139:                                              SPIRV::RequirementHandler &Reqs,
1140:                                              const SPIRVSubtarget &ST) {
1141:   SPIRVTypeInst VecTypeDef =
1142:       MI.getMF()->getRegInfo().getVRegDef(MI.getOperand(1).getReg());
1143:
1144:   const unsigned Rank = VecTypeDef->getOperand(2).getImm();
1145:   if (Rank != 2 && Rank != 4)
1146:     reportFatalUsageError("Result type of an atomic vector float instruction "
1147:                           "must be a 2-component or 4 component vector");
1148:
1149:   SPIRVTypeInst EltTypeDef =
1150:       MI.getMF()->getRegInfo().getVRegDef(VecTypeDef->getOperand(1).getReg());
1151:
1152:   if (EltTypeDef->getOpcode() != SPIRV::OpTypeFloat ||
1153:       EltTypeDef->getOperand(1).getImm() != 16)
1154:     reportFatalUsageError(
1155:         "The element type for the result type of an atomic vector float "
1156:         "instruction must be a 16-bit floating-point scalar");
1157:
1158:   if (isBFloat16Type(EltTypeDef))
1159:     reportFatalUsageError(
1160:         "The element type for the result type of an atomic vector float "
1161:         "instruction cannot be a bfloat16 scalar");
1162:   if (!ST.canUseExtension(SPIRV::Extension::SPV_NV_shader_atomic_fp16_vector))
1163:     reportFatalUsageError(
1164:         "The atomic float16 vector instruction requires the following SPIR-V "
1165:         "extension: SPV_NV_shader_atomic_fp16_vector");
1166:
1167:   Reqs.addExtension(SPIRV::Extension::SPV_NV_shader_atomic_fp16_vector);
1168:   Reqs.addCapability(SPIRV::Capability::AtomicFloat16VectorNV);
1169: }
1170:
1171: static void AddAtomicFloatRequirements(const MachineInstr &MI,
1172:                                        SPIRV::RequirementHandler &Reqs,
1173:                                        const SPIRVSubtarget &ST) {
1174:   assert(MI.getOperand(1).isReg() &&
1175:          "Expect register operand in atomic float instruction");
1176:   Register TypeReg = MI.getOperand(1).getReg();
1177:   SPIRVTypeInst TypeDef = MI.getMF()->getRegInfo().getVRegDef(TypeReg);
1178:
1179:   if (TypeDef->getOpcode() == SPIRV::OpTypeVector)
1180:     return AddAtomicVectorFloatRequirements(MI, Reqs, ST);
1181:
1182:   if (TypeDef->getOpcode() != SPIRV::OpTypeFloat)
1183:     report_fatal_error("Result type of an atomic float instruction must be a "
1184:                        "floating-point type scalar");
1185:
1186:   unsigned BitWidth = TypeDef->getOperand(1).getImm();
1187:   unsigned Op = MI.getOpcode();
1188:   if (Op == SPIRV::OpAtomicFAddEXT) {
1189:     if (!ST.canUseExtension(SPIRV::Extension::SPV_EXT_shader_atomic_float_add))
1190:       report_fatal_error(ATOM_FLT_REQ_EXT_MSG("_add"), false);
1191:     Reqs.addExtension(SPIRV::Extension::SPV_EXT_shader_atomic_float_add);
1192:     switch (BitWidth) {
1193:     case 16:
1194:       if (isBFloat16Type(TypeDef)) {
1195:         if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics))
1196:           report_fatal_error(
1197:               "The atomic bfloat16 instruction requires the following SPIR-V "
1198:               "extension: SPV_INTEL_16bit_atomics",
1199:               false);
1200:         Reqs.addExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics);
```
- EN: This range implements operational logic in helpers such as addRequirements, assert, getMF, getVRegDef, translating backend policy into executable code.
- CN: 这一段实现了 addRequirements、assert、getMF、getVRegDef 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1201-1320
```cpp
1201:         Reqs.addCapability(SPIRV::Capability::AtomicBFloat16AddINTEL);
1202:       } else {
1203:         if (!ST.canUseExtension(
1204:                 SPIRV::Extension::SPV_EXT_shader_atomic_float16_add))
1205:           report_fatal_error(ATOM_FLT_REQ_EXT_MSG("16_add"), false);
1206:         Reqs.addExtension(SPIRV::Extension::SPV_EXT_shader_atomic_float16_add);
1207:         Reqs.addCapability(SPIRV::Capability::AtomicFloat16AddEXT);
1208:       }
1209:       break;
1210:     case 32:
1211:       Reqs.addCapability(SPIRV::Capability::AtomicFloat32AddEXT);
1212:       break;
1213:     case 64:
1214:       Reqs.addCapability(SPIRV::Capability::AtomicFloat64AddEXT);
1215:       break;
1216:     default:
1217:       report_fatal_error(
1218:           "Unexpected floating-point type width in atomic float instruction");
1219:     }
1220:   } else {
1221:     if (!ST.canUseExtension(
1222:             SPIRV::Extension::SPV_EXT_shader_atomic_float_min_max))
1223:       report_fatal_error(ATOM_FLT_REQ_EXT_MSG("_min_max"), false);
1224:     Reqs.addExtension(SPIRV::Extension::SPV_EXT_shader_atomic_float_min_max);
1225:     switch (BitWidth) {
1226:     case 16:
1227:       if (isBFloat16Type(TypeDef)) {
1228:         if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics))
1229:           report_fatal_error(
1230:               "The atomic bfloat16 instruction requires the following SPIR-V "
1231:               "extension: SPV_INTEL_16bit_atomics",
1232:               false);
1233:         Reqs.addExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics);
1234:         Reqs.addCapability(SPIRV::Capability::AtomicBFloat16MinMaxINTEL);
1235:       } else {
1236:         Reqs.addCapability(SPIRV::Capability::AtomicFloat16MinMaxEXT);
1237:       }
1238:       break;
1239:     case 32:
1240:       Reqs.addCapability(SPIRV::Capability::AtomicFloat32MinMaxEXT);
1241:       break;
1242:     case 64:
1243:       Reqs.addCapability(SPIRV::Capability::AtomicFloat64MinMaxEXT);
1244:       break;
1245:     default:
1246:       report_fatal_error(
1247:           "Unexpected floating-point type width in atomic float instruction");
1248:     }
1249:   }
1250: }
1251:
1252: bool isUniformTexelBuffer(MachineInstr *ImageInst) {
1253:   if (ImageInst->getOpcode() != SPIRV::OpTypeImage)
1254:     return false;
1255:   uint32_t Dim = ImageInst->getOperand(2).getImm();
1256:   uint32_t Sampled = ImageInst->getOperand(6).getImm();
1257:   return Dim == SPIRV::Dim::DIM_Buffer && Sampled == 1;
1258: }
1259:
1260: bool isStorageTexelBuffer(MachineInstr *ImageInst) {
1261:   if (ImageInst->getOpcode() != SPIRV::OpTypeImage)
1262:     return false;
1263:   uint32_t Dim = ImageInst->getOperand(2).getImm();
1264:   uint32_t Sampled = ImageInst->getOperand(6).getImm();
1265:   return Dim == SPIRV::Dim::DIM_Buffer && Sampled == 2;
1266: }
1267:
1268: bool isSampledImage(MachineInstr *ImageInst) {
1269:   if (ImageInst->getOpcode() != SPIRV::OpTypeImage)
1270:     return false;
1271:   uint32_t Dim = ImageInst->getOperand(2).getImm();
1272:   uint32_t Sampled = ImageInst->getOperand(6).getImm();
1273:   return Dim != SPIRV::Dim::DIM_Buffer && Sampled == 1;
1274: }
1275:
1276: bool isInputAttachment(MachineInstr *ImageInst) {
1277:   if (ImageInst->getOpcode() != SPIRV::OpTypeImage)
1278:     return false;
1279:   uint32_t Dim = ImageInst->getOperand(2).getImm();
1280:   uint32_t Sampled = ImageInst->getOperand(6).getImm();
1281:   return Dim == SPIRV::Dim::DIM_SubpassData && Sampled == 2;
1282: }
1283:
1284: bool isStorageImage(MachineInstr *ImageInst) {
1285:   if (ImageInst->getOpcode() != SPIRV::OpTypeImage)
1286:     return false;
1287:   uint32_t Dim = ImageInst->getOperand(2).getImm();
1288:   uint32_t Sampled = ImageInst->getOperand(6).getImm();
1289:   return Dim != SPIRV::Dim::DIM_Buffer && Sampled == 2;
1290: }
1291:
1292: bool isCombinedImageSampler(MachineInstr *SampledImageInst) {
1293:   if (SampledImageInst->getOpcode() != SPIRV::OpTypeSampledImage)
1294:     return false;
1295:
1296:   const MachineRegisterInfo &MRI = SampledImageInst->getMF()->getRegInfo();
1297:   Register ImageReg = SampledImageInst->getOperand(1).getReg();
1298:   auto *ImageInst = MRI.getUniqueVRegDef(ImageReg);
1299:   return isSampledImage(ImageInst);
1300: }
1301:
1302: bool hasNonUniformDecoration(Register Reg, const MachineRegisterInfo &MRI) {
1303:   for (const auto &MI : MRI.reg_instructions(Reg)) {
1304:     if (MI.getOpcode() != SPIRV::OpDecorate)
1305:       continue;
1306:
1307:     uint32_t Dec = MI.getOperand(1).getImm();
1308:     if (Dec == SPIRV::Decoration::NonUniformEXT)
1309:       return true;
1310:   }
1311:   return false;
1312: }
1313:
1314: void addOpAccessChainReqs(const MachineInstr &Instr,
1315:                           SPIRV::RequirementHandler &Handler,
1316:                           const SPIRVSubtarget &Subtarget) {
1317:   const MachineRegisterInfo &MRI = Instr.getMF()->getRegInfo();
1318:   // Get the result type. If it is an image type, then the shader uses
1319:   // descriptor indexing. The appropriate capabilities will be added based
1320:   // on the specifics of the image.
```
- EN: This range implements operational logic in helpers such as addCapability, report_fatal_error, addExtension, isUniformTexelBuffer, translating backend policy into executable code.
- CN: 这一段实现了 addCapability、report_fatal_error、addExtension、isUniformTexelBuffer 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1321-1440
```cpp
1321:   Register ResTypeReg = Instr.getOperand(1).getReg();
1322:   MachineInstr *ResTypeInst = MRI.getUniqueVRegDef(ResTypeReg);
1323:
1324:   assert(ResTypeInst->getOpcode() == SPIRV::OpTypePointer);
1325:   uint32_t StorageClass = ResTypeInst->getOperand(1).getImm();
1326:   if (StorageClass != SPIRV::StorageClass::StorageClass::UniformConstant &&
1327:       StorageClass != SPIRV::StorageClass::StorageClass::Uniform &&
1328:       StorageClass != SPIRV::StorageClass::StorageClass::StorageBuffer) {
1329:     return;
1330:   }
1331:
1332:   bool IsNonUniform =
1333:       hasNonUniformDecoration(Instr.getOperand(0).getReg(), MRI);
1334:
1335:   auto FirstIndexReg = Instr.getOperand(3).getReg();
1336:   bool FirstIndexIsConstant =
1337:       Subtarget.getInstrInfo()->isConstantInstr(*MRI.getVRegDef(FirstIndexReg));
1338:
1339:   if (StorageClass == SPIRV::StorageClass::StorageClass::StorageBuffer) {
1340:     if (IsNonUniform)
1341:       Handler.addRequirements(
1342:           SPIRV::Capability::StorageBufferArrayNonUniformIndexingEXT);
1343:     else if (!FirstIndexIsConstant)
1344:       Handler.addRequirements(
1345:           SPIRV::Capability::StorageBufferArrayDynamicIndexing);
1346:     return;
1347:   }
1348:
1349:   Register PointeeTypeReg = ResTypeInst->getOperand(2).getReg();
1350:   MachineInstr *PointeeType = MRI.getUniqueVRegDef(PointeeTypeReg);
1351:   if (PointeeType->getOpcode() != SPIRV::OpTypeImage &&
1352:       PointeeType->getOpcode() != SPIRV::OpTypeSampledImage &&
1353:       PointeeType->getOpcode() != SPIRV::OpTypeSampler) {
1354:     return;
1355:   }
1356:
1357:   if (isUniformTexelBuffer(PointeeType)) {
1358:     if (IsNonUniform)
1359:       Handler.addRequirements(
1360:           SPIRV::Capability::UniformTexelBufferArrayNonUniformIndexingEXT);
1361:     else if (!FirstIndexIsConstant)
1362:       Handler.addRequirements(
1363:           SPIRV::Capability::UniformTexelBufferArrayDynamicIndexingEXT);
1364:   } else if (isInputAttachment(PointeeType)) {
1365:     if (IsNonUniform)
1366:       Handler.addRequirements(
1367:           SPIRV::Capability::InputAttachmentArrayNonUniformIndexingEXT);
1368:     else if (!FirstIndexIsConstant)
1369:       Handler.addRequirements(
1370:           SPIRV::Capability::InputAttachmentArrayDynamicIndexingEXT);
1371:   } else if (isStorageTexelBuffer(PointeeType)) {
1372:     if (IsNonUniform)
1373:       Handler.addRequirements(
1374:           SPIRV::Capability::StorageTexelBufferArrayNonUniformIndexingEXT);
1375:     else if (!FirstIndexIsConstant)
1376:       Handler.addRequirements(
1377:           SPIRV::Capability::StorageTexelBufferArrayDynamicIndexingEXT);
1378:   } else if (isSampledImage(PointeeType) ||
1379:              isCombinedImageSampler(PointeeType) ||
1380:              PointeeType->getOpcode() == SPIRV::OpTypeSampler) {
1381:     if (IsNonUniform)
1382:       Handler.addRequirements(
1383:           SPIRV::Capability::SampledImageArrayNonUniformIndexingEXT);
1384:     else if (!FirstIndexIsConstant)
1385:       Handler.addRequirements(
1386:           SPIRV::Capability::SampledImageArrayDynamicIndexing);
1387:   } else if (isStorageImage(PointeeType)) {
1388:     if (IsNonUniform)
1389:       Handler.addRequirements(
1390:           SPIRV::Capability::StorageImageArrayNonUniformIndexingEXT);
1391:     else if (!FirstIndexIsConstant)
1392:       Handler.addRequirements(
1393:           SPIRV::Capability::StorageImageArrayDynamicIndexing);
1394:   }
1395: }
1396:
1397: static bool isImageTypeWithUnknownFormat(SPIRVTypeInst TypeInst) {
1398:   if (TypeInst->getOpcode() != SPIRV::OpTypeImage)
1399:     return false;
1400:   assert(TypeInst->getOperand(7).isImm() && "The image format must be an imm.");
1401:   return TypeInst->getOperand(7).getImm() == 0;
1402: }
1403:
1404: static void AddDotProductRequirements(const MachineInstr &MI,
1405:                                       SPIRV::RequirementHandler &Reqs,
1406:                                       const SPIRVSubtarget &ST) {
1407:   if (ST.canUseExtension(SPIRV::Extension::SPV_KHR_integer_dot_product))
1408:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_integer_dot_product);
1409:   Reqs.addCapability(SPIRV::Capability::DotProduct);
1410:
1411:   const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1412:   assert(MI.getOperand(2).isReg() && "Unexpected operand in dot");
1413:   // We do not consider what the previous instruction is. This is just used
1414:   // to get the input register and to check the type.
1415:   const MachineInstr *Input = MRI.getVRegDef(MI.getOperand(2).getReg());
1416:   assert(Input->getOperand(1).isReg() && "Unexpected operand in dot input");
1417:   Register InputReg = Input->getOperand(1).getReg();
1418:
1419:   SPIRVTypeInst TypeDef = MRI.getVRegDef(InputReg);
1420:   if (TypeDef->getOpcode() == SPIRV::OpTypeInt) {
1421:     assert(TypeDef->getOperand(1).getImm() == 32);
1422:     Reqs.addCapability(SPIRV::Capability::DotProductInput4x8BitPacked);
1423:   } else if (TypeDef->getOpcode() == SPIRV::OpTypeVector) {
1424:     SPIRVTypeInst ScalarTypeDef =
1425:         MRI.getVRegDef(TypeDef->getOperand(1).getReg());
1426:     assert(ScalarTypeDef->getOpcode() == SPIRV::OpTypeInt);
1427:     if (ScalarTypeDef->getOperand(1).getImm() == 8) {
1428:       assert(TypeDef->getOperand(2).getImm() == 4 &&
1429:              "Dot operand of 8-bit integer type requires 4 components");
1430:       Reqs.addCapability(SPIRV::Capability::DotProductInput4x8Bit);
1431:     } else {
1432:       Reqs.addCapability(SPIRV::Capability::DotProductInputAll);
1433:     }
1434:   }
1435: }
1436:
1437: void addPrintfRequirements(const MachineInstr &MI,
1438:                            SPIRV::RequirementHandler &Reqs,
1439:                            const SPIRVSubtarget &ST) {
1440:   SPIRVGlobalRegistry *GR = ST.getSPIRVGlobalRegistry();
```
- EN: This range implements operational logic in helpers such as getOperand, getUniqueVRegDef, assert, hasNonUniformDecoration, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getUniqueVRegDef、assert、hasNonUniformDecoration 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1441-1560
```cpp
1441:   SPIRVTypeInst PtrType = GR->getSPIRVTypeForVReg(MI.getOperand(4).getReg());
1442:   if (PtrType) {
1443:     MachineOperand ASOp = PtrType->getOperand(1);
1444:     if (ASOp.isImm()) {
1445:       unsigned AddrSpace = ASOp.getImm();
1446:       if (AddrSpace != SPIRV::StorageClass::UniformConstant) {
1447:         if (!ST.canUseExtension(
1448:                 SPIRV::Extension::
1449:                     SPV_EXT_relaxed_printf_string_address_space)) {
1450:           report_fatal_error("SPV_EXT_relaxed_printf_string_address_space is "
1451:                              "required because printf uses a format string not "
1452:                              "in constant address space.",
1453:                              false);
1454:         }
1455:         Reqs.addExtension(
1456:             SPIRV::Extension::SPV_EXT_relaxed_printf_string_address_space);
1457:       }
1458:     }
1459:   }
1460: }
1461:
1462: static void addImageOperandReqs(const MachineInstr &MI,
1463:                                 SPIRV::RequirementHandler &Reqs,
1464:                                 const SPIRVSubtarget &ST, unsigned OpIdx) {
1465:   if (MI.getNumOperands() <= OpIdx)
1466:     return;
1467:   uint32_t Mask = MI.getOperand(OpIdx).getImm();
1468:   for (uint32_t I = 0; I < 32; ++I)
1469:     if (Mask & (1U << I))
1470:       Reqs.getAndAddRequirements(SPIRV::OperandCategory::ImageOperandOperand,
1471:                                  1U << I, ST);
1472: }
1473:
1474: void addInstrRequirements(const MachineInstr &MI,
1475:                           SPIRV::ModuleAnalysisInfo &MAI,
1476:                           const SPIRVSubtarget &ST) {
1477:   SPIRV::RequirementHandler &Reqs = MAI.Reqs;
1478:   unsigned Op = MI.getOpcode();
1479:   switch (Op) {
1480:   case SPIRV::OpMemoryModel: {
1481:     int64_t Addr = MI.getOperand(0).getImm();
1482:     Reqs.getAndAddRequirements(SPIRV::OperandCategory::AddressingModelOperand,
1483:                                Addr, ST);
1484:     int64_t Mem = MI.getOperand(1).getImm();
1485:     Reqs.getAndAddRequirements(SPIRV::OperandCategory::MemoryModelOperand, Mem,
1486:                                ST);
1487:     break;
1488:   }
1489:   case SPIRV::OpEntryPoint: {
1490:     int64_t Exe = MI.getOperand(0).getImm();
1491:     Reqs.getAndAddRequirements(SPIRV::OperandCategory::ExecutionModelOperand,
1492:                                Exe, ST);
1493:     break;
1494:   }
1495:   case SPIRV::OpExecutionMode:
1496:   case SPIRV::OpExecutionModeId: {
1497:     int64_t Exe = MI.getOperand(1).getImm();
1498:     Reqs.getAndAddRequirements(SPIRV::OperandCategory::ExecutionModeOperand,
1499:                                Exe, ST);
1500:     break;
1501:   }
1502:   case SPIRV::OpTypeMatrix:
1503:     Reqs.addCapability(SPIRV::Capability::Matrix);
1504:     break;
1505:   case SPIRV::OpTypeInt: {
1506:     unsigned BitWidth = MI.getOperand(1).getImm();
1507:     if (BitWidth == 64)
1508:       Reqs.addCapability(SPIRV::Capability::Int64);
1509:     else if (BitWidth == 16)
1510:       Reqs.addCapability(SPIRV::Capability::Int16);
1511:     else if (BitWidth == 8)
1512:       Reqs.addCapability(SPIRV::Capability::Int8);
1513:     else if (BitWidth == 4 &&
1514:              ST.canUseExtension(SPIRV::Extension::SPV_INTEL_int4)) {
1515:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_int4);
1516:       Reqs.addCapability(SPIRV::Capability::Int4TypeINTEL);
1517:     } else if (BitWidth != 32) {
1518:       if (!ST.canUseExtension(
1519:               SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers))
1520:         reportFatalUsageError(
1521:             "OpTypeInt type with a width other than 8, 16, 32 or 64 bits "
1522:             "requires the following SPIR-V extension: "
1523:             "SPV_ALTERA_arbitrary_precision_integers");
1524:       Reqs.addExtension(
1525:           SPIRV::Extension::SPV_ALTERA_arbitrary_precision_integers);
1526:       Reqs.addCapability(SPIRV::Capability::ArbitraryPrecisionIntegersALTERA);
1527:     }
1528:     break;
1529:   }
1530:   case SPIRV::OpDot: {
1531:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1532:     SPIRVTypeInst TypeDef = MRI.getVRegDef(MI.getOperand(1).getReg());
1533:     if (isBFloat16Type(TypeDef))
1534:       Reqs.addCapability(SPIRV::Capability::BFloat16DotProductKHR);
1535:     break;
1536:   }
1537:   case SPIRV::OpTypeFloat: {
1538:     unsigned BitWidth = MI.getOperand(1).getImm();
1539:     if (BitWidth == 64)
1540:       Reqs.addCapability(SPIRV::Capability::Float64);
1541:     else if (BitWidth == 16) {
1542:       if (isBFloat16Type(&MI)) {
1543:         if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_bfloat16))
1544:           report_fatal_error("OpTypeFloat type with bfloat requires the "
1545:                              "following SPIR-V extension: SPV_KHR_bfloat16",
1546:                              false);
1547:         Reqs.addExtension(SPIRV::Extension::SPV_KHR_bfloat16);
1548:         Reqs.addCapability(SPIRV::Capability::BFloat16TypeKHR);
1549:       } else {
1550:         Reqs.addCapability(SPIRV::Capability::Float16);
1551:       }
1552:     }
1553:     break;
1554:   }
1555:   case SPIRV::OpTypeVector: {
1556:     unsigned NumComponents = MI.getOperand(2).getImm();
1557:     if (NumComponents == 8 || NumComponents == 16)
1558:       Reqs.addCapability(SPIRV::Capability::Vector16);
1559:
1560:     assert(MI.getOperand(1).isReg());
```
- EN: This range implements operational logic in helpers such as getSPIRVTypeForVReg, getOperand, getImm, getOpcode, translating backend policy into executable code.
- CN: 这一段实现了 getSPIRVTypeForVReg、getOperand、getImm、getOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1561-1680
```cpp
1561:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1562:     SPIRVTypeInst ElemTypeDef = MRI.getVRegDef(MI.getOperand(1).getReg());
1563:     if (ElemTypeDef->getOpcode() == SPIRV::OpTypePointer &&
1564:         ST.canUseExtension(SPIRV::Extension::SPV_INTEL_masked_gather_scatter)) {
1565:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_masked_gather_scatter);
1566:       Reqs.addCapability(SPIRV::Capability::MaskedGatherScatterINTEL);
1567:     }
1568:     break;
1569:   }
1570:   case SPIRV::OpTypePointer: {
1571:     auto SC = MI.getOperand(1).getImm();
1572:     Reqs.getAndAddRequirements(SPIRV::OperandCategory::StorageClassOperand, SC,
1573:                                ST);
1574:     // If it's a type of pointer to float16 targeting OpenCL, add Float16Buffer
1575:     // capability.
1576:     if (ST.isShader())
1577:       break;
1578:     assert(MI.getOperand(2).isReg());
1579:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1580:     SPIRVTypeInst TypeDef = MRI.getVRegDef(MI.getOperand(2).getReg());
1581:     if ((TypeDef->getNumOperands() == 2) &&
1582:         (TypeDef->getOpcode() == SPIRV::OpTypeFloat) &&
1583:         (TypeDef->getOperand(1).getImm() == 16))
1584:       Reqs.addCapability(SPIRV::Capability::Float16Buffer);
1585:     break;
1586:   }
1587:   case SPIRV::OpExtInst: {
1588:     if (MI.getOperand(2).getImm() ==
1589:         static_cast<int64_t>(
1590:             SPIRV::InstructionSet::NonSemantic_Shader_DebugInfo_100)) {
1591:       Reqs.addExtension(SPIRV::Extension::SPV_KHR_non_semantic_info);
1592:       break;
1593:     }
1594:     if (MI.getOperand(3).getImm() ==
1595:         static_cast<int64_t>(SPIRV::OpenCLExtInst::printf)) {
1596:       addPrintfRequirements(MI, Reqs, ST);
1597:       break;
1598:     }
1599:     // TODO: handle bfloat16 extended instructions when
1600:     // SPV_INTEL_bfloat16_arithmetic is enabled.
1601:     break;
1602:   }
1603:   case SPIRV::OpAliasDomainDeclINTEL:
1604:   case SPIRV::OpAliasScopeDeclINTEL:
1605:   case SPIRV::OpAliasScopeListDeclINTEL: {
1606:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_memory_access_aliasing);
1607:     Reqs.addCapability(SPIRV::Capability::MemoryAccessAliasingINTEL);
1608:     break;
1609:   }
1610:   case SPIRV::OpBitReverse:
1611:   case SPIRV::OpBitFieldInsert:
1612:   case SPIRV::OpBitFieldSExtract:
1613:   case SPIRV::OpBitFieldUExtract:
1614:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_bit_instructions)) {
1615:       Reqs.addCapability(SPIRV::Capability::Shader);
1616:       break;
1617:     }
1618:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_bit_instructions);
1619:     Reqs.addCapability(SPIRV::Capability::BitInstructions);
1620:     break;
1621:   case SPIRV::OpTypeRuntimeArray:
1622:     Reqs.addCapability(SPIRV::Capability::Shader);
1623:     break;
1624:   case SPIRV::OpTypeOpaque:
1625:   case SPIRV::OpTypeEvent:
1626:     Reqs.addCapability(SPIRV::Capability::Kernel);
1627:     break;
1628:   case SPIRV::OpTypePipe:
1629:   case SPIRV::OpTypeReserveId:
1630:     Reqs.addCapability(SPIRV::Capability::Pipes);
1631:     break;
1632:   case SPIRV::OpTypeDeviceEvent:
1633:   case SPIRV::OpTypeQueue:
1634:   case SPIRV::OpBuildNDRange:
1635:     Reqs.addCapability(SPIRV::Capability::DeviceEnqueue);
1636:     break;
1637:   case SPIRV::OpDecorate:
1638:   case SPIRV::OpDecorateId:
1639:   case SPIRV::OpDecorateString:
1640:     addOpDecorateReqs(MI, 1, Reqs, ST);
1641:     break;
1642:   case SPIRV::OpMemberDecorate:
1643:   case SPIRV::OpMemberDecorateString:
1644:     addOpDecorateReqs(MI, 2, Reqs, ST);
1645:     break;
1646:   case SPIRV::OpInBoundsPtrAccessChain:
1647:     Reqs.addCapability(SPIRV::Capability::Addresses);
1648:     break;
1649:   case SPIRV::OpConstantSampler:
1650:     Reqs.addCapability(SPIRV::Capability::LiteralSampler);
1651:     break;
1652:   case SPIRV::OpInBoundsAccessChain:
1653:   case SPIRV::OpAccessChain:
1654:     addOpAccessChainReqs(MI, Reqs, ST);
1655:     break;
1656:   case SPIRV::OpTypeImage:
1657:     addOpTypeImageReqs(MI, Reqs, ST);
1658:     break;
1659:   case SPIRV::OpTypeSampler:
1660:     if (!ST.isShader()) {
1661:       Reqs.addCapability(SPIRV::Capability::ImageBasic);
1662:     }
1663:     break;
1664:   case SPIRV::OpTypeForwardPointer:
1665:     // TODO: check if it's OpenCL's kernel.
1666:     Reqs.addCapability(SPIRV::Capability::Addresses);
1667:     break;
1668:   case SPIRV::OpAtomicFlagTestAndSet:
1669:   case SPIRV::OpAtomicLoad:
1670:   case SPIRV::OpAtomicStore:
1671:   case SPIRV::OpAtomicExchange:
1672:   case SPIRV::OpAtomicCompareExchange:
1673:   case SPIRV::OpAtomicIIncrement:
1674:   case SPIRV::OpAtomicIDecrement:
1675:   case SPIRV::OpAtomicIAdd:
1676:   case SPIRV::OpAtomicISub:
1677:   case SPIRV::OpAtomicUMin:
1678:   case SPIRV::OpAtomicUMax:
1679:   case SPIRV::OpAtomicSMin:
1680:   case SPIRV::OpAtomicSMax:
```
- EN: This range implements operational logic in helpers such as getMF, getVRegDef, canUseExtension, addExtension, translating backend policy into executable code.
- CN: 这一段实现了 getMF、getVRegDef、canUseExtension、addExtension 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1681-1800
```cpp
1681:   case SPIRV::OpAtomicAnd:
1682:   case SPIRV::OpAtomicOr:
1683:   case SPIRV::OpAtomicXor: {
1684:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1685:     const MachineInstr *InstrPtr = &MI;
1686:     if (Op == SPIRV::OpAtomicStore) {
1687:       assert(MI.getOperand(3).isReg());
1688:       InstrPtr = MRI.getVRegDef(MI.getOperand(3).getReg());
1689:       assert(InstrPtr && "Unexpected type instruction for OpAtomicStore");
1690:     }
1691:     assert(InstrPtr->getOperand(1).isReg() && "Unexpected operand in atomic");
1692:     Register TypeReg = InstrPtr->getOperand(1).getReg();
1693:     SPIRVTypeInst TypeDef = MRI.getVRegDef(TypeReg);
1694:
1695:     if (TypeDef->getOpcode() == SPIRV::OpTypeInt) {
1696:       unsigned BitWidth = TypeDef->getOperand(1).getImm();
1697:       if (BitWidth == 64)
1698:         Reqs.addCapability(SPIRV::Capability::Int64Atomics);
1699:       else if (BitWidth == 16) {
1700:         if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics))
1701:           report_fatal_error(
1702:               "16-bit integer atomic operations require the following SPIR-V "
1703:               "extension: SPV_INTEL_16bit_atomics",
1704:               false);
1705:         Reqs.addExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics);
1706:         switch (Op) {
1707:         case SPIRV::OpAtomicLoad:
1708:         case SPIRV::OpAtomicStore:
1709:         case SPIRV::OpAtomicExchange:
1710:         case SPIRV::OpAtomicCompareExchange:
1711:         case SPIRV::OpAtomicCompareExchangeWeak:
1712:           Reqs.addCapability(
1713:               SPIRV::Capability::AtomicInt16CompareExchangeINTEL);
1714:           break;
1715:         default:
1716:           Reqs.addCapability(SPIRV::Capability::Int16AtomicsINTEL);
1717:           break;
1718:         }
1719:       }
1720:     } else if (isBFloat16Type(TypeDef)) {
1721:       if (is_contained({SPIRV::OpAtomicLoad, SPIRV::OpAtomicStore,
1722:                         SPIRV::OpAtomicExchange},
1723:                        Op)) {
1724:         if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics))
1725:           report_fatal_error(
1726:               "The atomic bfloat16 instruction requires the following SPIR-V "
1727:               "extension: SPV_INTEL_16bit_atomics",
1728:               false);
1729:         Reqs.addExtension(SPIRV::Extension::SPV_INTEL_16bit_atomics);
1730:         Reqs.addCapability(SPIRV::Capability::AtomicBFloat16LoadStoreINTEL);
1731:       }
1732:     }
1733:     break;
1734:   }
1735:   case SPIRV::OpGroupNonUniformIAdd:
1736:   case SPIRV::OpGroupNonUniformFAdd:
1737:   case SPIRV::OpGroupNonUniformIMul:
1738:   case SPIRV::OpGroupNonUniformFMul:
1739:   case SPIRV::OpGroupNonUniformSMin:
1740:   case SPIRV::OpGroupNonUniformUMin:
1741:   case SPIRV::OpGroupNonUniformFMin:
1742:   case SPIRV::OpGroupNonUniformSMax:
1743:   case SPIRV::OpGroupNonUniformUMax:
1744:   case SPIRV::OpGroupNonUniformFMax:
1745:   case SPIRV::OpGroupNonUniformBitwiseAnd:
1746:   case SPIRV::OpGroupNonUniformBitwiseOr:
1747:   case SPIRV::OpGroupNonUniformBitwiseXor:
1748:   case SPIRV::OpGroupNonUniformLogicalAnd:
1749:   case SPIRV::OpGroupNonUniformLogicalOr:
1750:   case SPIRV::OpGroupNonUniformLogicalXor: {
1751:     assert(MI.getOperand(3).isImm());
1752:     int64_t GroupOp = MI.getOperand(3).getImm();
1753:     switch (GroupOp) {
1754:     case SPIRV::GroupOperation::Reduce:
1755:     case SPIRV::GroupOperation::InclusiveScan:
1756:     case SPIRV::GroupOperation::ExclusiveScan:
1757:       Reqs.addCapability(SPIRV::Capability::GroupNonUniformArithmetic);
1758:       break;
1759:     case SPIRV::GroupOperation::ClusteredReduce:
1760:       Reqs.addCapability(SPIRV::Capability::GroupNonUniformClustered);
1761:       break;
1762:     case SPIRV::GroupOperation::PartitionedReduceNV:
1763:     case SPIRV::GroupOperation::PartitionedInclusiveScanNV:
1764:     case SPIRV::GroupOperation::PartitionedExclusiveScanNV:
1765:       Reqs.addCapability(SPIRV::Capability::GroupNonUniformPartitionedNV);
1766:       break;
1767:     }
1768:     break;
1769:   }
1770:   case SPIRV::OpGroupNonUniformQuadSwap:
1771:     Reqs.addCapability(SPIRV::Capability::GroupNonUniformQuad);
1772:     break;
1773:   case SPIRV::OpImageQueryLod:
1774:     Reqs.addCapability(SPIRV::Capability::ImageQuery);
1775:     break;
1776:   case SPIRV::OpImageQuerySize:
1777:   case SPIRV::OpImageQuerySizeLod:
1778:   case SPIRV::OpImageQueryLevels:
1779:   case SPIRV::OpImageQuerySamples:
1780:     if (ST.isShader())
1781:       Reqs.addCapability(SPIRV::Capability::ImageQuery);
1782:     break;
1783:   case SPIRV::OpImageQueryFormat: {
1784:     Register ResultReg = MI.getOperand(0).getReg();
1785:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
1786:     static const unsigned CompareOps[] = {
1787:         SPIRV::OpIEqual,       SPIRV::OpINotEqual,
1788:         SPIRV::OpUGreaterThan, SPIRV::OpUGreaterThanEqual,
1789:         SPIRV::OpULessThan,    SPIRV::OpULessThanEqual,
1790:         SPIRV::OpSGreaterThan, SPIRV::OpSGreaterThanEqual,
1791:         SPIRV::OpSLessThan,    SPIRV::OpSLessThanEqual};
1792:
1793:     auto CheckAndAddExtension = [&](int64_t ImmVal) {
1794:       if (ImmVal == 4323 || ImmVal == 4324) {
1795:         if (ST.canUseExtension(SPIRV::Extension::SPV_EXT_image_raw10_raw12))
1796:           Reqs.addExtension(SPIRV::Extension::SPV_EXT_image_raw10_raw12);
1797:         else
1798:           report_fatal_error("This requires the "
1799:                              "SPV_EXT_image_raw10_raw12 extension");
1800:       }
```
- EN: This range implements operational logic in helpers such as getMF, assert, getVRegDef, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getMF、assert、getVRegDef、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1801-1920
```cpp
1801:     };
1802:
1803:     for (MachineInstr &UseInst : MRI.use_instructions(ResultReg)) {
1804:       unsigned Opc = UseInst.getOpcode();
1805:
1806:       if (Opc == SPIRV::OpSwitch) {
1807:         for (const MachineOperand &Op : UseInst.operands())
1808:           if (Op.isImm())
1809:             CheckAndAddExtension(Op.getImm());
1810:       } else if (llvm::is_contained(CompareOps, Opc)) {
1811:         for (unsigned i = 1; i < UseInst.getNumOperands(); ++i) {
1812:           Register UseReg = UseInst.getOperand(i).getReg();
1813:           MachineInstr *ConstInst = MRI.getVRegDef(UseReg);
1814:           if (ConstInst && ConstInst->getOpcode() == SPIRV::OpConstantI) {
1815:             int64_t ImmVal = ConstInst->getOperand(2).getImm();
1816:             if (ImmVal)
1817:               CheckAndAddExtension(ImmVal);
1818:           }
1819:         }
1820:       }
1821:     }
1822:     break;
1823:   }
1824:
1825:   case SPIRV::OpGroupNonUniformShuffle:
1826:   case SPIRV::OpGroupNonUniformShuffleXor:
1827:     Reqs.addCapability(SPIRV::Capability::GroupNonUniformShuffle);
1828:     break;
1829:   case SPIRV::OpGroupNonUniformShuffleUp:
1830:   case SPIRV::OpGroupNonUniformShuffleDown:
1831:     Reqs.addCapability(SPIRV::Capability::GroupNonUniformShuffleRelative);
1832:     break;
1833:   case SPIRV::OpGroupAll:
1834:   case SPIRV::OpGroupAny:
1835:   case SPIRV::OpGroupBroadcast:
1836:   case SPIRV::OpGroupIAdd:
1837:   case SPIRV::OpGroupFAdd:
1838:   case SPIRV::OpGroupFMin:
1839:   case SPIRV::OpGroupUMin:
1840:   case SPIRV::OpGroupSMin:
1841:   case SPIRV::OpGroupFMax:
1842:   case SPIRV::OpGroupUMax:
1843:   case SPIRV::OpGroupSMax:
1844:     Reqs.addCapability(SPIRV::Capability::Groups);
1845:     break;
1846:   case SPIRV::OpGroupNonUniformElect:
1847:     Reqs.addCapability(SPIRV::Capability::GroupNonUniform);
1848:     break;
1849:   case SPIRV::OpGroupNonUniformAll:
1850:   case SPIRV::OpGroupNonUniformAny:
1851:   case SPIRV::OpGroupNonUniformAllEqual:
1852:     Reqs.addCapability(SPIRV::Capability::GroupNonUniformVote);
1853:     break;
1854:   case SPIRV::OpGroupNonUniformBroadcast:
1855:   case SPIRV::OpGroupNonUniformBroadcastFirst:
1856:   case SPIRV::OpGroupNonUniformBallot:
1857:   case SPIRV::OpGroupNonUniformInverseBallot:
1858:   case SPIRV::OpGroupNonUniformBallotBitExtract:
1859:   case SPIRV::OpGroupNonUniformBallotBitCount:
1860:   case SPIRV::OpGroupNonUniformBallotFindLSB:
1861:   case SPIRV::OpGroupNonUniformBallotFindMSB:
1862:     Reqs.addCapability(SPIRV::Capability::GroupNonUniformBallot);
1863:     break;
1864:   case SPIRV::OpSubgroupShuffleINTEL:
1865:   case SPIRV::OpSubgroupShuffleDownINTEL:
1866:   case SPIRV::OpSubgroupShuffleUpINTEL:
1867:   case SPIRV::OpSubgroupShuffleXorINTEL:
1868:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_subgroups)) {
1869:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_subgroups);
1870:       Reqs.addCapability(SPIRV::Capability::SubgroupShuffleINTEL);
1871:     }
1872:     break;
1873:   case SPIRV::OpSubgroupBlockReadINTEL:
1874:   case SPIRV::OpSubgroupBlockWriteINTEL:
1875:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_subgroups)) {
1876:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_subgroups);
1877:       Reqs.addCapability(SPIRV::Capability::SubgroupBufferBlockIOINTEL);
1878:     }
1879:     break;
1880:   case SPIRV::OpSubgroupImageBlockReadINTEL:
1881:   case SPIRV::OpSubgroupImageBlockWriteINTEL:
1882:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_subgroups)) {
1883:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_subgroups);
1884:       Reqs.addCapability(SPIRV::Capability::SubgroupImageBlockIOINTEL);
1885:     }
1886:     break;
1887:   case SPIRV::OpSubgroupImageMediaBlockReadINTEL:
1888:   case SPIRV::OpSubgroupImageMediaBlockWriteINTEL:
1889:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_media_block_io)) {
1890:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_media_block_io);
1891:       Reqs.addCapability(SPIRV::Capability::SubgroupImageMediaBlockIOINTEL);
1892:     }
1893:     break;
1894:   case SPIRV::OpAssumeTrueKHR:
1895:   case SPIRV::OpExpectKHR:
1896:     if (ST.canUseExtension(SPIRV::Extension::SPV_KHR_expect_assume)) {
1897:       Reqs.addExtension(SPIRV::Extension::SPV_KHR_expect_assume);
1898:       Reqs.addCapability(SPIRV::Capability::ExpectAssumeKHR);
1899:     }
1900:     break;
1901:   case SPIRV::OpFmaKHR:
1902:     if (ST.canUseExtension(SPIRV::Extension::SPV_KHR_fma)) {
1903:       Reqs.addExtension(SPIRV::Extension::SPV_KHR_fma);
1904:       Reqs.addCapability(SPIRV::Capability::FmaKHR);
1905:     }
1906:     break;
1907:   case SPIRV::OpPtrCastToCrossWorkgroupINTEL:
1908:   case SPIRV::OpCrossWorkgroupCastToPtrINTEL:
1909:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_usm_storage_classes)) {
1910:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_usm_storage_classes);
1911:       Reqs.addCapability(SPIRV::Capability::USMStorageClassesINTEL);
1912:     }
1913:     break;
1914:   case SPIRV::OpConstantFunctionPointerINTEL:
1915:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers)) {
1916:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_function_pointers);
1917:       Reqs.addCapability(SPIRV::Capability::FunctionPointersINTEL);
1918:     }
1919:     break;
1920:   case SPIRV::OpGroupNonUniformRotateKHR:
```
- EN: This range implements operational logic in helpers such as getOpcode, CheckAndAddExtension, getNumOperands, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getOpcode、CheckAndAddExtension、getNumOperands、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1921-2040
```cpp
1921:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_subgroup_rotate))
1922:       report_fatal_error("OpGroupNonUniformRotateKHR instruction requires the "
1923:                          "following SPIR-V extension: SPV_KHR_subgroup_rotate",
1924:                          false);
1925:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_subgroup_rotate);
1926:     Reqs.addCapability(SPIRV::Capability::GroupNonUniformRotateKHR);
1927:     Reqs.addCapability(SPIRV::Capability::GroupNonUniform);
1928:     break;
1929:   case SPIRV::OpFixedCosALTERA:
1930:   case SPIRV::OpFixedSinALTERA:
1931:   case SPIRV::OpFixedCosPiALTERA:
1932:   case SPIRV::OpFixedSinPiALTERA:
1933:   case SPIRV::OpFixedExpALTERA:
1934:   case SPIRV::OpFixedLogALTERA:
1935:   case SPIRV::OpFixedRecipALTERA:
1936:   case SPIRV::OpFixedSqrtALTERA:
1937:   case SPIRV::OpFixedSinCosALTERA:
1938:   case SPIRV::OpFixedSinCosPiALTERA:
1939:   case SPIRV::OpFixedRsqrtALTERA:
1940:     if (!ST.canUseExtension(
1941:             SPIRV::Extension::SPV_ALTERA_arbitrary_precision_fixed_point))
1942:       report_fatal_error("This instruction requires the "
1943:                          "following SPIR-V extension: "
1944:                          "SPV_ALTERA_arbitrary_precision_fixed_point",
1945:                          false);
1946:     Reqs.addExtension(
1947:         SPIRV::Extension::SPV_ALTERA_arbitrary_precision_fixed_point);
1948:     Reqs.addCapability(SPIRV::Capability::ArbitraryPrecisionFixedPointALTERA);
1949:     break;
1950:   case SPIRV::OpGroupIMulKHR:
1951:   case SPIRV::OpGroupFMulKHR:
1952:   case SPIRV::OpGroupBitwiseAndKHR:
1953:   case SPIRV::OpGroupBitwiseOrKHR:
1954:   case SPIRV::OpGroupBitwiseXorKHR:
1955:   case SPIRV::OpGroupLogicalAndKHR:
1956:   case SPIRV::OpGroupLogicalOrKHR:
1957:   case SPIRV::OpGroupLogicalXorKHR:
1958:     if (ST.canUseExtension(
1959:             SPIRV::Extension::SPV_KHR_uniform_group_instructions)) {
1960:       Reqs.addExtension(SPIRV::Extension::SPV_KHR_uniform_group_instructions);
1961:       Reqs.addCapability(SPIRV::Capability::GroupUniformArithmeticKHR);
1962:     }
1963:     break;
1964:   case SPIRV::OpReadClockKHR:
1965:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_shader_clock))
1966:       report_fatal_error("OpReadClockKHR instruction requires the "
1967:                          "following SPIR-V extension: SPV_KHR_shader_clock",
1968:                          false);
1969:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_shader_clock);
1970:     Reqs.addCapability(SPIRV::Capability::ShaderClockKHR);
1971:     break;
1972:   case SPIRV::OpAbortKHR:
1973:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_abort))
1974:       report_fatal_error("OpAbortKHR instruction requires the "
1975:                          "following SPIR-V extension: SPV_KHR_abort",
1976:                          false);
1977:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_abort);
1978:     Reqs.addCapability(SPIRV::Capability::AbortKHR);
1979:     break;
1980:   case SPIRV::OpFunctionPointerCallINTEL:
1981:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_function_pointers)) {
1982:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_function_pointers);
1983:       Reqs.addCapability(SPIRV::Capability::FunctionPointersINTEL);
1984:     }
1985:     break;
1986:   case SPIRV::OpAtomicFAddEXT:
1987:   case SPIRV::OpAtomicFMinEXT:
1988:   case SPIRV::OpAtomicFMaxEXT:
1989:     AddAtomicFloatRequirements(MI, Reqs, ST);
1990:     break;
1991:   case SPIRV::OpConvertBF16ToFINTEL:
1992:   case SPIRV::OpConvertFToBF16INTEL:
1993:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_bfloat16_conversion)) {
1994:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_bfloat16_conversion);
1995:       Reqs.addCapability(SPIRV::Capability::BFloat16ConversionINTEL);
1996:     }
1997:     break;
1998:   case SPIRV::OpRoundFToTF32INTEL:
1999:     if (ST.canUseExtension(
2000:             SPIRV::Extension::SPV_INTEL_tensor_float32_conversion)) {
2001:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_tensor_float32_conversion);
2002:       Reqs.addCapability(SPIRV::Capability::TensorFloat32RoundingINTEL);
2003:     }
2004:     break;
2005:   case SPIRV::OpVariableLengthArrayINTEL:
2006:   case SPIRV::OpSaveMemoryINTEL:
2007:   case SPIRV::OpRestoreMemoryINTEL:
2008:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_variable_length_array)) {
2009:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_variable_length_array);
2010:       Reqs.addCapability(SPIRV::Capability::VariableLengthArrayINTEL);
2011:     }
2012:     break;
2013:   case SPIRV::OpAsmTargetINTEL:
2014:   case SPIRV::OpAsmINTEL:
2015:   case SPIRV::OpAsmCallINTEL:
2016:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_inline_assembly)) {
2017:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_inline_assembly);
2018:       Reqs.addCapability(SPIRV::Capability::AsmINTEL);
2019:     }
2020:     break;
2021:   case SPIRV::OpTypeCooperativeMatrixKHR: {
2022:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_cooperative_matrix))
2023:       report_fatal_error(
2024:           "OpTypeCooperativeMatrixKHR type requires the "
2025:           "following SPIR-V extension: SPV_KHR_cooperative_matrix",
2026:           false);
2027:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_cooperative_matrix);
2028:     Reqs.addCapability(SPIRV::Capability::CooperativeMatrixKHR);
2029:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
2030:     SPIRVTypeInst TypeDef = MRI.getVRegDef(MI.getOperand(1).getReg());
2031:     if (isBFloat16Type(TypeDef))
2032:       Reqs.addCapability(SPIRV::Capability::BFloat16CooperativeMatrixKHR);
2033:     break;
2034:   }
2035:   case SPIRV::OpArithmeticFenceEXT:
2036:     if (!ST.canUseExtension(SPIRV::Extension::SPV_EXT_arithmetic_fence))
2037:       report_fatal_error("OpArithmeticFenceEXT requires the "
2038:                          "following SPIR-V extension: SPV_EXT_arithmetic_fence",
2039:                          false);
2040:     Reqs.addExtension(SPIRV::Extension::SPV_EXT_arithmetic_fence);
```
- EN: This range implements operational logic in helpers such as addExtension, addCapability, AddAtomicFloatRequirements, getMF, translating backend policy into executable code.
- CN: 这一段实现了 addExtension、addCapability、AddAtomicFloatRequirements、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2041-2160
```cpp
2041:     Reqs.addCapability(SPIRV::Capability::ArithmeticFenceEXT);
2042:     break;
2043:   case SPIRV::OpControlBarrierArriveINTEL:
2044:   case SPIRV::OpControlBarrierWaitINTEL:
2045:     if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_split_barrier)) {
2046:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_split_barrier);
2047:       Reqs.addCapability(SPIRV::Capability::SplitBarrierINTEL);
2048:     }
2049:     break;
2050:   case SPIRV::OpCooperativeMatrixMulAddKHR: {
2051:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_cooperative_matrix))
2052:       report_fatal_error("Cooperative matrix instructions require the "
2053:                          "following SPIR-V extension: "
2054:                          "SPV_KHR_cooperative_matrix",
2055:                          false);
2056:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_cooperative_matrix);
2057:     Reqs.addCapability(SPIRV::Capability::CooperativeMatrixKHR);
2058:     constexpr unsigned MulAddMaxSize = 6;
2059:     if (MI.getNumOperands() != MulAddMaxSize)
2060:       break;
2061:     const int64_t CoopOperands = MI.getOperand(MulAddMaxSize - 1).getImm();
2062:     if (CoopOperands &
2063:         SPIRV::CooperativeMatrixOperands::MatrixAAndBTF32ComponentsINTEL) {
2064:       if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_joint_matrix))
2065:         report_fatal_error("MatrixAAndBTF32ComponentsINTEL type interpretation "
2066:                            "require the following SPIR-V extension: "
2067:                            "SPV_INTEL_joint_matrix",
2068:                            false);
2069:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_joint_matrix);
2070:       Reqs.addCapability(
2071:           SPIRV::Capability::CooperativeMatrixTF32ComponentTypeINTEL);
2072:     }
2073:     if (CoopOperands & SPIRV::CooperativeMatrixOperands::
2074:                            MatrixAAndBBFloat16ComponentsINTEL ||
2075:         CoopOperands &
2076:             SPIRV::CooperativeMatrixOperands::MatrixCBFloat16ComponentsINTEL ||
2077:         CoopOperands & SPIRV::CooperativeMatrixOperands::
2078:                            MatrixResultBFloat16ComponentsINTEL) {
2079:       if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_joint_matrix))
2080:         report_fatal_error("***BF16ComponentsINTEL type interpretations "
2081:                            "require the following SPIR-V extension: "
2082:                            "SPV_INTEL_joint_matrix",
2083:                            false);
2084:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_joint_matrix);
2085:       Reqs.addCapability(
2086:           SPIRV::Capability::CooperativeMatrixBFloat16ComponentTypeINTEL);
2087:     }
2088:     break;
2089:   }
2090:   case SPIRV::OpCooperativeMatrixLoadKHR:
2091:   case SPIRV::OpCooperativeMatrixStoreKHR:
2092:   case SPIRV::OpCooperativeMatrixLoadCheckedINTEL:
2093:   case SPIRV::OpCooperativeMatrixStoreCheckedINTEL:
2094:   case SPIRV::OpCooperativeMatrixPrefetchINTEL: {
2095:     if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_cooperative_matrix))
2096:       report_fatal_error("Cooperative matrix instructions require the "
2097:                          "following SPIR-V extension: "
2098:                          "SPV_KHR_cooperative_matrix",
2099:                          false);
2100:     Reqs.addExtension(SPIRV::Extension::SPV_KHR_cooperative_matrix);
2101:     Reqs.addCapability(SPIRV::Capability::CooperativeMatrixKHR);
2102:
2103:     // Check Layout operand in case if it's not a standard one and add the
2104:     // appropriate capability.
2105:     std::unordered_map<unsigned, unsigned> LayoutToInstMap = {
2106:         {SPIRV::OpCooperativeMatrixLoadKHR, 3},
2107:         {SPIRV::OpCooperativeMatrixStoreKHR, 2},
2108:         {SPIRV::OpCooperativeMatrixLoadCheckedINTEL, 5},
2109:         {SPIRV::OpCooperativeMatrixStoreCheckedINTEL, 4},
2110:         {SPIRV::OpCooperativeMatrixPrefetchINTEL, 4}};
2111:
2112:     const unsigned LayoutNum = LayoutToInstMap[Op];
2113:     Register RegLayout = MI.getOperand(LayoutNum).getReg();
2114:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
2115:     MachineInstr *MILayout = MRI.getUniqueVRegDef(RegLayout);
2116:     if (MILayout->getOpcode() == SPIRV::OpConstantI) {
2117:       const unsigned LayoutVal = MILayout->getOperand(2).getImm();
2118:       if (LayoutVal ==
2119:           static_cast<unsigned>(SPIRV::CooperativeMatrixLayout::PackedINTEL)) {
2120:         if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_joint_matrix))
2121:           report_fatal_error("PackedINTEL layout require the following SPIR-V "
2122:                              "extension: SPV_INTEL_joint_matrix",
2123:                              false);
2124:         Reqs.addExtension(SPIRV::Extension::SPV_INTEL_joint_matrix);
2125:         Reqs.addCapability(SPIRV::Capability::PackedCooperativeMatrixINTEL);
2126:       }
2127:     }
2128:
2129:     // Nothing to do.
2130:     if (Op == SPIRV::OpCooperativeMatrixLoadKHR ||
2131:         Op == SPIRV::OpCooperativeMatrixStoreKHR)
2132:       break;
2133:
2134:     std::string InstName;
2135:     switch (Op) {
2136:     case SPIRV::OpCooperativeMatrixPrefetchINTEL:
2137:       InstName = "OpCooperativeMatrixPrefetchINTEL";
2138:       break;
2139:     case SPIRV::OpCooperativeMatrixLoadCheckedINTEL:
2140:       InstName = "OpCooperativeMatrixLoadCheckedINTEL";
2141:       break;
2142:     case SPIRV::OpCooperativeMatrixStoreCheckedINTEL:
2143:       InstName = "OpCooperativeMatrixStoreCheckedINTEL";
2144:       break;
2145:     }
2146:
2147:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_joint_matrix)) {
2148:       const std::string ErrorMsg =
2149:           InstName + " instruction requires the "
2150:                      "following SPIR-V extension: SPV_INTEL_joint_matrix";
2151:       report_fatal_error(ErrorMsg.c_str(), false);
2152:     }
2153:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_joint_matrix);
2154:     if (Op == SPIRV::OpCooperativeMatrixPrefetchINTEL) {
2155:       Reqs.addCapability(SPIRV::Capability::CooperativeMatrixPrefetchINTEL);
2156:       break;
2157:     }
2158:     Reqs.addCapability(
2159:         SPIRV::Capability::CooperativeMatrixCheckedInstructionsINTEL);
2160:     break;
```
- EN: This range implements operational logic in helpers such as addCapability, addExtension, getOperand, getMF, translating backend policy into executable code.
- CN: 这一段实现了 addCapability、addExtension、getOperand、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2161-2280
```cpp
2161:   }
2162:   case SPIRV::OpCooperativeMatrixConstructCheckedINTEL:
2163:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_joint_matrix))
2164:       report_fatal_error("OpCooperativeMatrixConstructCheckedINTEL "
2165:                          "instructions require the following SPIR-V extension: "
2166:                          "SPV_INTEL_joint_matrix",
2167:                          false);
2168:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_joint_matrix);
2169:     Reqs.addCapability(
2170:         SPIRV::Capability::CooperativeMatrixCheckedInstructionsINTEL);
2171:     break;
2172:   case SPIRV::OpReadPipeBlockingALTERA:
2173:   case SPIRV::OpWritePipeBlockingALTERA:
2174:     if (ST.canUseExtension(SPIRV::Extension::SPV_ALTERA_blocking_pipes)) {
2175:       Reqs.addExtension(SPIRV::Extension::SPV_ALTERA_blocking_pipes);
2176:       Reqs.addCapability(SPIRV::Capability::BlockingPipesALTERA);
2177:     }
2178:     break;
2179:   case SPIRV::OpCooperativeMatrixGetElementCoordINTEL:
2180:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_joint_matrix))
2181:       report_fatal_error("OpCooperativeMatrixGetElementCoordINTEL requires the "
2182:                          "following SPIR-V extension: SPV_INTEL_joint_matrix",
2183:                          false);
2184:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_joint_matrix);
2185:     Reqs.addCapability(
2186:         SPIRV::Capability::CooperativeMatrixInvocationInstructionsINTEL);
2187:     break;
2188:   case SPIRV::OpConvertHandleToImageINTEL:
2189:   case SPIRV::OpConvertHandleToSamplerINTEL:
2190:   case SPIRV::OpConvertHandleToSampledImageINTEL: {
2191:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_bindless_images))
2192:       report_fatal_error("OpConvertHandleTo[Image/Sampler/SampledImage]INTEL "
2193:                          "instructions require the following SPIR-V extension: "
2194:                          "SPV_INTEL_bindless_images",
2195:                          false);
2196:     SPIRVGlobalRegistry *GR = ST.getSPIRVGlobalRegistry();
2197:     SPIRV::AddressingModel::AddressingModel AddrModel = MAI.Addr;
2198:     SPIRVTypeInst TyDef = GR->getSPIRVTypeForVReg(MI.getOperand(1).getReg());
2199:     if (Op == SPIRV::OpConvertHandleToImageINTEL &&
2200:         TyDef->getOpcode() != SPIRV::OpTypeImage) {
2201:       report_fatal_error("Incorrect return type for the instruction "
2202:                          "OpConvertHandleToImageINTEL",
2203:                          false);
2204:     } else if (Op == SPIRV::OpConvertHandleToSamplerINTEL &&
2205:                TyDef->getOpcode() != SPIRV::OpTypeSampler) {
2206:       report_fatal_error("Incorrect return type for the instruction "
2207:                          "OpConvertHandleToSamplerINTEL",
2208:                          false);
2209:     } else if (Op == SPIRV::OpConvertHandleToSampledImageINTEL &&
2210:                TyDef->getOpcode() != SPIRV::OpTypeSampledImage) {
2211:       report_fatal_error("Incorrect return type for the instruction "
2212:                          "OpConvertHandleToSampledImageINTEL",
2213:                          false);
2214:     }
2215:     SPIRVTypeInst SpvTy = GR->getSPIRVTypeForVReg(MI.getOperand(2).getReg());
2216:     unsigned Bitwidth = GR->getScalarOrVectorBitWidth(SpvTy);
2217:     if (!(Bitwidth == 32 && AddrModel == SPIRV::AddressingModel::Physical32) &&
2218:         !(Bitwidth == 64 && AddrModel == SPIRV::AddressingModel::Physical64)) {
2219:       report_fatal_error(
2220:           "Parameter value must be a 32-bit scalar in case of "
2221:           "Physical32 addressing model or a 64-bit scalar in case of "
2222:           "Physical64 addressing model",
2223:           false);
2224:     }
2225:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_bindless_images);
2226:     Reqs.addCapability(SPIRV::Capability::BindlessImagesINTEL);
2227:     break;
2228:   }
2229:   case SPIRV::OpSubgroup2DBlockLoadINTEL:
2230:   case SPIRV::OpSubgroup2DBlockLoadTransposeINTEL:
2231:   case SPIRV::OpSubgroup2DBlockLoadTransformINTEL:
2232:   case SPIRV::OpSubgroup2DBlockPrefetchINTEL:
2233:   case SPIRV::OpSubgroup2DBlockStoreINTEL: {
2234:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_2d_block_io))
2235:       report_fatal_error("OpSubgroup2DBlock[Load/LoadTranspose/LoadTransform/"
2236:                          "Prefetch/Store]INTEL instructions require the "
2237:                          "following SPIR-V extension: SPV_INTEL_2d_block_io",
2238:                          false);
2239:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_2d_block_io);
2240:     Reqs.addCapability(SPIRV::Capability::Subgroup2DBlockIOINTEL);
2241:
2242:     if (Op == SPIRV::OpSubgroup2DBlockLoadTransposeINTEL) {
2243:       Reqs.addCapability(SPIRV::Capability::Subgroup2DBlockTransposeINTEL);
2244:       break;
2245:     }
2246:     if (Op == SPIRV::OpSubgroup2DBlockLoadTransformINTEL) {
2247:       Reqs.addCapability(SPIRV::Capability::Subgroup2DBlockTransformINTEL);
2248:       break;
2249:     }
2250:     break;
2251:   }
2252:   case SPIRV::OpKill: {
2253:     Reqs.addCapability(SPIRV::Capability::Shader);
2254:   } break;
2255:   case SPIRV::OpDemoteToHelperInvocation:
2256:     Reqs.addCapability(SPIRV::Capability::DemoteToHelperInvocation);
2257:
2258:     if (ST.canUseExtension(
2259:             SPIRV::Extension::SPV_EXT_demote_to_helper_invocation)) {
2260:       if (!ST.isAtLeastSPIRVVer(llvm::VersionTuple(1, 6)))
2261:         Reqs.addExtension(
2262:             SPIRV::Extension::SPV_EXT_demote_to_helper_invocation);
2263:     }
2264:     break;
2265:   case SPIRV::OpSDot:
2266:   case SPIRV::OpUDot:
2267:   case SPIRV::OpSUDot:
2268:   case SPIRV::OpSDotAccSat:
2269:   case SPIRV::OpUDotAccSat:
2270:   case SPIRV::OpSUDotAccSat:
2271:     AddDotProductRequirements(MI, Reqs, ST);
2272:     break;
2273:   case SPIRV::OpImageSampleImplicitLod:
2274:     Reqs.addCapability(SPIRV::Capability::Shader);
2275:     addImageOperandReqs(MI, Reqs, ST, 4);
2276:     break;
2277:   case SPIRV::OpImageSampleExplicitLod:
2278:     addImageOperandReqs(MI, Reqs, ST, 4);
2279:     break;
2280:   case SPIRV::OpImageSampleDrefImplicitLod:
```
- EN: This range implements operational logic in helpers such as addExtension, addCapability, getSPIRVGlobalRegistry, getSPIRVTypeForVReg, translating backend policy into executable code.
- CN: 这一段实现了 addExtension、addCapability、getSPIRVGlobalRegistry、getSPIRVTypeForVReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2281-2400
```cpp
2281:     Reqs.addCapability(SPIRV::Capability::Shader);
2282:     addImageOperandReqs(MI, Reqs, ST, 5);
2283:     break;
2284:   case SPIRV::OpImageSampleDrefExplicitLod:
2285:     Reqs.addCapability(SPIRV::Capability::Shader);
2286:     addImageOperandReqs(MI, Reqs, ST, 5);
2287:     break;
2288:   case SPIRV::OpImageFetch:
2289:     Reqs.addCapability(SPIRV::Capability::Shader);
2290:     addImageOperandReqs(MI, Reqs, ST, 4);
2291:     break;
2292:   case SPIRV::OpImageDrefGather:
2293:   case SPIRV::OpImageGather:
2294:     Reqs.addCapability(SPIRV::Capability::Shader);
2295:     addImageOperandReqs(MI, Reqs, ST, 5);
2296:     break;
2297:   case SPIRV::OpImageRead: {
2298:     Register ImageReg = MI.getOperand(2).getReg();
2299:     SPIRVTypeInst TypeDef = ST.getSPIRVGlobalRegistry()->getResultType(
2300:         ImageReg, const_cast<MachineFunction *>(MI.getMF()));
2301:     // OpImageRead and OpImageWrite can use Unknown Image Formats
2302:     // when the Kernel capability is declared. In the OpenCL environment we are
2303:     // not allowed to produce
2304:     // StorageImageReadWithoutFormat/StorageImageWriteWithoutFormat, see
2305:     // https://github.com/KhronosGroup/SPIRV-Headers/issues/487
2306:
2307:     if (isImageTypeWithUnknownFormat(TypeDef) && ST.isShader())
2308:       Reqs.addCapability(SPIRV::Capability::StorageImageReadWithoutFormat);
2309:     break;
2310:   }
2311:   case SPIRV::OpImageWrite: {
2312:     Register ImageReg = MI.getOperand(0).getReg();
2313:     SPIRVTypeInst TypeDef = ST.getSPIRVGlobalRegistry()->getResultType(
2314:         ImageReg, const_cast<MachineFunction *>(MI.getMF()));
2315:     // OpImageRead and OpImageWrite can use Unknown Image Formats
2316:     // when the Kernel capability is declared. In the OpenCL environment we are
2317:     // not allowed to produce
2318:     // StorageImageReadWithoutFormat/StorageImageWriteWithoutFormat, see
2319:     // https://github.com/KhronosGroup/SPIRV-Headers/issues/487
2320:
2321:     if (isImageTypeWithUnknownFormat(TypeDef) && ST.isShader())
2322:       Reqs.addCapability(SPIRV::Capability::StorageImageWriteWithoutFormat);
2323:     break;
2324:   }
2325:   case SPIRV::OpTypeStructContinuedINTEL:
2326:   case SPIRV::OpConstantCompositeContinuedINTEL:
2327:   case SPIRV::OpSpecConstantCompositeContinuedINTEL:
2328:   case SPIRV::OpCompositeConstructContinuedINTEL: {
2329:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_long_composites))
2330:       report_fatal_error(
2331:           "Continued instructions require the "
2332:           "following SPIR-V extension: SPV_INTEL_long_composites",
2333:           false);
2334:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_long_composites);
2335:     Reqs.addCapability(SPIRV::Capability::LongCompositesINTEL);
2336:     break;
2337:   }
2338:   case SPIRV::OpArbitraryFloatEQALTERA:
2339:   case SPIRV::OpArbitraryFloatGEALTERA:
2340:   case SPIRV::OpArbitraryFloatGTALTERA:
2341:   case SPIRV::OpArbitraryFloatLEALTERA:
2342:   case SPIRV::OpArbitraryFloatLTALTERA:
2343:   case SPIRV::OpArbitraryFloatCbrtALTERA:
2344:   case SPIRV::OpArbitraryFloatCosALTERA:
2345:   case SPIRV::OpArbitraryFloatCosPiALTERA:
2346:   case SPIRV::OpArbitraryFloatExp10ALTERA:
2347:   case SPIRV::OpArbitraryFloatExp2ALTERA:
2348:   case SPIRV::OpArbitraryFloatExpALTERA:
2349:   case SPIRV::OpArbitraryFloatExpm1ALTERA:
2350:   case SPIRV::OpArbitraryFloatHypotALTERA:
2351:   case SPIRV::OpArbitraryFloatLog10ALTERA:
2352:   case SPIRV::OpArbitraryFloatLog1pALTERA:
2353:   case SPIRV::OpArbitraryFloatLog2ALTERA:
2354:   case SPIRV::OpArbitraryFloatLogALTERA:
2355:   case SPIRV::OpArbitraryFloatRecipALTERA:
2356:   case SPIRV::OpArbitraryFloatSinCosALTERA:
2357:   case SPIRV::OpArbitraryFloatSinCosPiALTERA:
2358:   case SPIRV::OpArbitraryFloatSinALTERA:
2359:   case SPIRV::OpArbitraryFloatSinPiALTERA:
2360:   case SPIRV::OpArbitraryFloatSqrtALTERA:
2361:   case SPIRV::OpArbitraryFloatACosALTERA:
2362:   case SPIRV::OpArbitraryFloatACosPiALTERA:
2363:   case SPIRV::OpArbitraryFloatAddALTERA:
2364:   case SPIRV::OpArbitraryFloatASinALTERA:
2365:   case SPIRV::OpArbitraryFloatASinPiALTERA:
2366:   case SPIRV::OpArbitraryFloatATan2ALTERA:
2367:   case SPIRV::OpArbitraryFloatATanALTERA:
2368:   case SPIRV::OpArbitraryFloatATanPiALTERA:
2369:   case SPIRV::OpArbitraryFloatCastFromIntALTERA:
2370:   case SPIRV::OpArbitraryFloatCastALTERA:
2371:   case SPIRV::OpArbitraryFloatCastToIntALTERA:
2372:   case SPIRV::OpArbitraryFloatDivALTERA:
2373:   case SPIRV::OpArbitraryFloatMulALTERA:
2374:   case SPIRV::OpArbitraryFloatPowALTERA:
2375:   case SPIRV::OpArbitraryFloatPowNALTERA:
2376:   case SPIRV::OpArbitraryFloatPowRALTERA:
2377:   case SPIRV::OpArbitraryFloatRSqrtALTERA:
2378:   case SPIRV::OpArbitraryFloatSubALTERA: {
2379:     if (!ST.canUseExtension(
2380:             SPIRV::Extension::SPV_ALTERA_arbitrary_precision_floating_point))
2381:       report_fatal_error(
2382:           "Floating point instructions can't be translated correctly without "
2383:           "enabled SPV_ALTERA_arbitrary_precision_floating_point extension!",
2384:           false);
2385:     Reqs.addExtension(
2386:         SPIRV::Extension::SPV_ALTERA_arbitrary_precision_floating_point);
2387:     Reqs.addCapability(
2388:         SPIRV::Capability::ArbitraryPrecisionFloatingPointALTERA);
2389:     break;
2390:   }
2391:   case SPIRV::OpSubgroupMatrixMultiplyAccumulateINTEL: {
2392:     if (!ST.canUseExtension(
2393:             SPIRV::Extension::SPV_INTEL_subgroup_matrix_multiply_accumulate))
2394:       report_fatal_error(
2395:           "OpSubgroupMatrixMultiplyAccumulateINTEL instruction requires the "
2396:           "following SPIR-V "
2397:           "extension: SPV_INTEL_subgroup_matrix_multiply_accumulate",
2398:           false);
2399:     Reqs.addExtension(
2400:         SPIRV::Extension::SPV_INTEL_subgroup_matrix_multiply_accumulate);
```
- EN: This range implements operational logic in helpers such as addCapability, addImageOperandReqs, getOperand, getMF, translating backend policy into executable code.
- CN: 这一段实现了 addCapability、addImageOperandReqs、getOperand、getMF 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2401-2520
```cpp
2401:     Reqs.addCapability(
2402:         SPIRV::Capability::SubgroupMatrixMultiplyAccumulateINTEL);
2403:     break;
2404:   }
2405:   case SPIRV::OpBitwiseFunctionINTEL: {
2406:     if (!ST.canUseExtension(
2407:             SPIRV::Extension::SPV_INTEL_ternary_bitwise_function))
2408:       report_fatal_error(
2409:           "OpBitwiseFunctionINTEL instruction requires the following SPIR-V "
2410:           "extension: SPV_INTEL_ternary_bitwise_function",
2411:           false);
2412:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_ternary_bitwise_function);
2413:     Reqs.addCapability(SPIRV::Capability::TernaryBitwiseFunctionINTEL);
2414:     break;
2415:   }
2416:   case SPIRV::OpCopyMemorySized: {
2417:     Reqs.addCapability(SPIRV::Capability::Addresses);
2418:     // TODO: Add UntypedPointersKHR when implemented.
2419:     break;
2420:   }
2421:   case SPIRV::OpPredicatedLoadINTEL:
2422:   case SPIRV::OpPredicatedStoreINTEL: {
2423:     if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_predicated_io))
2424:       report_fatal_error(
2425:           "OpPredicated[Load/Store]INTEL instructions require "
2426:           "the following SPIR-V extension: SPV_INTEL_predicated_io",
2427:           false);
2428:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_predicated_io);
2429:     Reqs.addCapability(SPIRV::Capability::PredicatedIOINTEL);
2430:     break;
2431:   }
2432:   case SPIRV::OpFAddS:
2433:   case SPIRV::OpFSubS:
2434:   case SPIRV::OpFMulS:
2435:   case SPIRV::OpFDivS:
2436:   case SPIRV::OpFRemS:
2437:   case SPIRV::OpFMod:
2438:   case SPIRV::OpFNegate:
2439:   case SPIRV::OpFAddV:
2440:   case SPIRV::OpFSubV:
2441:   case SPIRV::OpFMulV:
2442:   case SPIRV::OpFDivV:
2443:   case SPIRV::OpFRemV:
2444:   case SPIRV::OpFNegateV: {
2445:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
2446:     SPIRVTypeInst TypeDef = MRI.getVRegDef(MI.getOperand(1).getReg());
2447:     if (TypeDef->getOpcode() == SPIRV::OpTypeVector)
2448:       TypeDef = MRI.getVRegDef(TypeDef->getOperand(1).getReg());
2449:     if (isBFloat16Type(TypeDef)) {
2450:       if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_bfloat16_arithmetic))
2451:         report_fatal_error(
2452:             "Arithmetic instructions with bfloat16 arguments require the "
2453:             "following SPIR-V extension: SPV_INTEL_bfloat16_arithmetic",
2454:             false);
2455:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_bfloat16_arithmetic);
2456:       Reqs.addCapability(SPIRV::Capability::BFloat16ArithmeticINTEL);
2457:     }
2458:     break;
2459:   }
2460:   case SPIRV::OpOrdered:
2461:   case SPIRV::OpUnordered:
2462:   case SPIRV::OpFOrdEqual:
2463:   case SPIRV::OpFOrdNotEqual:
2464:   case SPIRV::OpFOrdLessThan:
2465:   case SPIRV::OpFOrdLessThanEqual:
2466:   case SPIRV::OpFOrdGreaterThan:
2467:   case SPIRV::OpFOrdGreaterThanEqual:
2468:   case SPIRV::OpFUnordEqual:
2469:   case SPIRV::OpFUnordNotEqual:
2470:   case SPIRV::OpFUnordLessThan:
2471:   case SPIRV::OpFUnordLessThanEqual:
2472:   case SPIRV::OpFUnordGreaterThan:
2473:   case SPIRV::OpFUnordGreaterThanEqual: {
2474:     const MachineRegisterInfo &MRI = MI.getMF()->getRegInfo();
2475:     MachineInstr *OperandDef = MRI.getVRegDef(MI.getOperand(2).getReg());
2476:     SPIRVTypeInst TypeDef = MRI.getVRegDef(OperandDef->getOperand(1).getReg());
2477:     if (TypeDef->getOpcode() == SPIRV::OpTypeVector)
2478:       TypeDef = MRI.getVRegDef(TypeDef->getOperand(1).getReg());
2479:     if (isBFloat16Type(TypeDef)) {
2480:       if (!ST.canUseExtension(SPIRV::Extension::SPV_INTEL_bfloat16_arithmetic))
2481:         report_fatal_error(
2482:             "Relational instructions with bfloat16 arguments require the "
2483:             "following SPIR-V extension: SPV_INTEL_bfloat16_arithmetic",
2484:             false);
2485:       Reqs.addExtension(SPIRV::Extension::SPV_INTEL_bfloat16_arithmetic);
2486:       Reqs.addCapability(SPIRV::Capability::BFloat16ArithmeticINTEL);
2487:     }
2488:     break;
2489:   }
2490:   case SPIRV::OpDPdxCoarse:
2491:   case SPIRV::OpDPdyCoarse:
2492:   case SPIRV::OpDPdxFine:
2493:   case SPIRV::OpDPdyFine: {
2494:     Reqs.addCapability(SPIRV::Capability::DerivativeControl);
2495:     break;
2496:   }
2497:   case SPIRV::OpLoopControlINTEL: {
2498:     Reqs.addExtension(SPIRV::Extension::SPV_INTEL_unstructured_loop_controls);
2499:     Reqs.addCapability(SPIRV::Capability::UnstructuredLoopControlsINTEL);
2500:     break;
2501:   }
2502:
2503:   default:
2504:     break;
2505:   }
2506:
2507:   // If we require capability Shader, then we can remove the requirement for
2508:   // the BitInstructions capability, since Shader is a superset capability
2509:   // of BitInstructions.
2510:   Reqs.removeCapabilityIf(SPIRV::Capability::BitInstructions,
2511:                           SPIRV::Capability::Shader);
2512: }
2513:
2514: static void collectReqs(const Module &M, SPIRV::ModuleAnalysisInfo &MAI,
2515:                         MachineModuleInfo *MMI, const SPIRVSubtarget &ST) {
2516:   // Collect requirements for existing instructions.
2517:   for (const Function &F : M) {
2518:     MachineFunction *MF = MMI->getMachineFunction(F);
2519:     if (!MF)
2520:       continue;
```
- EN: This range implements operational logic in helpers such as addExtension, addCapability, getMF, getVRegDef, translating backend policy into executable code.
- CN: 这一段实现了 addExtension、addCapability、getMF、getVRegDef 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2521-2640
```cpp
2521:     for (const MachineBasicBlock &MBB : *MF)
2522:       for (const MachineInstr &MI : MBB)
2523:         addInstrRequirements(MI, MAI, ST);
2524:   }
2525:   // Collect requirements for OpExecutionMode instructions.
2526:   auto Node = M.getNamedMetadata("spirv.ExecutionMode");
2527:   if (Node) {
2528:     bool RequireFloatControls = false, RequireIntelFloatControls2 = false,
2529:          RequireKHRFloatControls2 = false,
2530:          VerLower14 = !ST.isAtLeastSPIRVVer(VersionTuple(1, 4));
2531:     bool HasIntelFloatControls2 =
2532:         ST.canUseExtension(SPIRV::Extension::SPV_INTEL_float_controls2);
2533:     bool HasKHRFloatControls2 =
2534:         ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2);
2535:     for (unsigned i = 0; i < Node->getNumOperands(); i++) {
2536:       MDNode *MDN = cast<MDNode>(Node->getOperand(i));
2537:       const MDOperand &MDOp = MDN->getOperand(1);
2538:       if (auto *CMeta = dyn_cast<ConstantAsMetadata>(MDOp)) {
2539:         Constant *C = CMeta->getValue();
2540:         if (ConstantInt *Const = dyn_cast<ConstantInt>(C)) {
2541:           auto EM = Const->getZExtValue();
2542:           // SPV_KHR_float_controls is not available until v1.4:
2543:           // add SPV_KHR_float_controls if the version is too low
2544:           switch (EM) {
2545:           case SPIRV::ExecutionMode::DenormPreserve:
2546:           case SPIRV::ExecutionMode::DenormFlushToZero:
2547:           case SPIRV::ExecutionMode::RoundingModeRTE:
2548:           case SPIRV::ExecutionMode::RoundingModeRTZ:
2549:             RequireFloatControls = VerLower14;
2550:             MAI.Reqs.getAndAddRequirements(
2551:                 SPIRV::OperandCategory::ExecutionModeOperand, EM, ST);
2552:             break;
2553:           case SPIRV::ExecutionMode::RoundingModeRTPINTEL:
2554:           case SPIRV::ExecutionMode::RoundingModeRTNINTEL:
2555:           case SPIRV::ExecutionMode::FloatingPointModeALTINTEL:
2556:           case SPIRV::ExecutionMode::FloatingPointModeIEEEINTEL:
2557:             if (HasIntelFloatControls2) {
2558:               RequireIntelFloatControls2 = true;
2559:               MAI.Reqs.getAndAddRequirements(
2560:                   SPIRV::OperandCategory::ExecutionModeOperand, EM, ST);
2561:             }
2562:             break;
2563:           case SPIRV::ExecutionMode::FPFastMathDefault: {
2564:             if (HasKHRFloatControls2) {
2565:               RequireKHRFloatControls2 = true;
2566:               MAI.Reqs.getAndAddRequirements(
2567:                   SPIRV::OperandCategory::ExecutionModeOperand, EM, ST);
2568:             }
2569:             break;
2570:           }
2571:           case SPIRV::ExecutionMode::ContractionOff:
2572:           case SPIRV::ExecutionMode::SignedZeroInfNanPreserve:
2573:             if (HasKHRFloatControls2) {
2574:               RequireKHRFloatControls2 = true;
2575:               MAI.Reqs.getAndAddRequirements(
2576:                   SPIRV::OperandCategory::ExecutionModeOperand,
2577:                   SPIRV::ExecutionMode::FPFastMathDefault, ST);
2578:             } else {
2579:               MAI.Reqs.getAndAddRequirements(
2580:                   SPIRV::OperandCategory::ExecutionModeOperand, EM, ST);
2581:             }
2582:             break;
2583:           default:
2584:             MAI.Reqs.getAndAddRequirements(
2585:                 SPIRV::OperandCategory::ExecutionModeOperand, EM, ST);
2586:           }
2587:         }
2588:       }
2589:     }
2590:     if (RequireFloatControls &&
2591:         ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls))
2592:       MAI.Reqs.addExtension(SPIRV::Extension::SPV_KHR_float_controls);
2593:     if (RequireIntelFloatControls2)
2594:       MAI.Reqs.addExtension(SPIRV::Extension::SPV_INTEL_float_controls2);
2595:     if (RequireKHRFloatControls2)
2596:       MAI.Reqs.addExtension(SPIRV::Extension::SPV_KHR_float_controls2);
2597:   }
2598:   for (const Function &F : M) {
2599:     if (F.isDeclaration())
2600:       continue;
2601:     if (F.getMetadata("reqd_work_group_size"))
2602:       MAI.Reqs.getAndAddRequirements(
2603:           SPIRV::OperandCategory::ExecutionModeOperand,
2604:           SPIRV::ExecutionMode::LocalSize, ST);
2605:     if (F.getFnAttribute("hlsl.numthreads").isValid()) {
2606:       MAI.Reqs.getAndAddRequirements(
2607:           SPIRV::OperandCategory::ExecutionModeOperand,
2608:           SPIRV::ExecutionMode::LocalSize, ST);
2609:     }
2610:     if (F.getFnAttribute("enable-maximal-reconvergence").getValueAsBool()) {
2611:       MAI.Reqs.addExtension(SPIRV::Extension::SPV_KHR_maximal_reconvergence);
2612:     }
2613:     if (F.getMetadata("work_group_size_hint"))
2614:       MAI.Reqs.getAndAddRequirements(
2615:           SPIRV::OperandCategory::ExecutionModeOperand,
2616:           SPIRV::ExecutionMode::LocalSizeHint, ST);
2617:     if (F.getMetadata("intel_reqd_sub_group_size"))
2618:       MAI.Reqs.getAndAddRequirements(
2619:           SPIRV::OperandCategory::ExecutionModeOperand,
2620:           SPIRV::ExecutionMode::SubgroupSize, ST);
2621:     if (F.getMetadata("max_work_group_size"))
2622:       MAI.Reqs.getAndAddRequirements(
2623:           SPIRV::OperandCategory::ExecutionModeOperand,
2624:           SPIRV::ExecutionMode::MaxWorkgroupSizeINTEL, ST);
2625:     if (F.getMetadata("vec_type_hint"))
2626:       MAI.Reqs.getAndAddRequirements(
2627:           SPIRV::OperandCategory::ExecutionModeOperand,
2628:           SPIRV::ExecutionMode::VecTypeHint, ST);
2629:
2630:     if (F.hasOptNone()) {
2631:       if (ST.canUseExtension(SPIRV::Extension::SPV_INTEL_optnone)) {
2632:         MAI.Reqs.addExtension(SPIRV::Extension::SPV_INTEL_optnone);
2633:         MAI.Reqs.addCapability(SPIRV::Capability::OptNoneINTEL);
2634:       } else if (ST.canUseExtension(SPIRV::Extension::SPV_EXT_optnone)) {
2635:         MAI.Reqs.addExtension(SPIRV::Extension::SPV_EXT_optnone);
2636:         MAI.Reqs.addCapability(SPIRV::Capability::OptNoneEXT);
2637:       }
2638:     }
2639:   }
2640: }
```
- EN: This range implements operational logic in helpers such as addInstrRequirements, getNamedMetadata, isAtLeastSPIRVVer, canUseExtension, translating backend policy into executable code.
- CN: 这一段实现了 addInstrRequirements、getNamedMetadata、isAtLeastSPIRVVer、canUseExtension 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2641-2760
```cpp
2641:
2642: static unsigned getFastMathFlags(const MachineInstr &I,
2643:                                  const SPIRVSubtarget &ST) {
2644:   unsigned Flags = SPIRV::FPFastMathMode::None;
2645:   bool CanUseKHRFloatControls2 =
2646:       ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2);
2647:   if (I.getFlag(MachineInstr::MIFlag::FmNoNans))
2648:     Flags |= SPIRV::FPFastMathMode::NotNaN;
2649:   if (I.getFlag(MachineInstr::MIFlag::FmNoInfs))
2650:     Flags |= SPIRV::FPFastMathMode::NotInf;
2651:   if (I.getFlag(MachineInstr::MIFlag::FmNsz))
2652:     Flags |= SPIRV::FPFastMathMode::NSZ;
2653:   if (I.getFlag(MachineInstr::MIFlag::FmArcp))
2654:     Flags |= SPIRV::FPFastMathMode::AllowRecip;
2655:   if (I.getFlag(MachineInstr::MIFlag::FmContract) && CanUseKHRFloatControls2)
2656:     Flags |= SPIRV::FPFastMathMode::AllowContract;
2657:   if (I.getFlag(MachineInstr::MIFlag::FmReassoc)) {
2658:     if (CanUseKHRFloatControls2)
2659:       // LLVM reassoc maps to SPIRV transform, see
2660:       // https://github.com/KhronosGroup/SPIRV-Registry/issues/326 for details.
2661:       // Because we are enabling AllowTransform, we must enable AllowReassoc and
2662:       // AllowContract too, as required by SPIRV spec. Also, we used to map
2663:       // MIFlag::FmReassoc to FPFastMathMode::Fast, which now should instead by
2664:       // replaced by turning all the other bits instead. Therefore, we're
2665:       // enabling every bit here except None and Fast.
2666:       Flags |= SPIRV::FPFastMathMode::NotNaN | SPIRV::FPFastMathMode::NotInf |
2667:                SPIRV::FPFastMathMode::NSZ | SPIRV::FPFastMathMode::AllowRecip |
2668:                SPIRV::FPFastMathMode::AllowTransform |
2669:                SPIRV::FPFastMathMode::AllowReassoc |
2670:                SPIRV::FPFastMathMode::AllowContract;
2671:     else
2672:       Flags |= SPIRV::FPFastMathMode::Fast;
2673:   }
2674:
2675:   if (CanUseKHRFloatControls2) {
2676:     // Error out if SPIRV::FPFastMathMode::Fast is enabled.
2677:     assert(!(Flags & SPIRV::FPFastMathMode::Fast) &&
2678:            "SPIRV::FPFastMathMode::Fast is deprecated and should not be used "
2679:            "anymore.");
2680:
2681:     // Error out if AllowTransform is enabled without AllowReassoc and
2682:     // AllowContract.
2683:     assert((!(Flags & SPIRV::FPFastMathMode::AllowTransform) ||
2684:             ((Flags & SPIRV::FPFastMathMode::AllowReassoc &&
2685:               Flags & SPIRV::FPFastMathMode::AllowContract))) &&
2686:            "SPIRV::FPFastMathMode::AllowTransform requires AllowReassoc and "
2687:            "AllowContract flags to be enabled as well.");
2688:   }
2689:
2690:   return Flags;
2691: }
2692:
2693: static bool isFastMathModeAvailable(const SPIRVSubtarget &ST) {
2694:   if (ST.isKernel())
2695:     return true;
2696:   if (ST.getSPIRVVersion() < VersionTuple(1, 2))
2697:     return false;
2698:   return ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2);
2699: }
2700:
2701: static void handleMIFlagDecoration(
2702:     MachineInstr &I, const SPIRVSubtarget &ST, const SPIRVInstrInfo &TII,
2703:     SPIRV::RequirementHandler &Reqs, const SPIRVGlobalRegistry *GR,
2704:     SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec) {
2705:   if (I.getFlag(MachineInstr::MIFlag::NoSWrap) && TII.canUseNSW(I) &&
2706:       getSymbolicOperandRequirements(SPIRV::OperandCategory::DecorationOperand,
2707:                                      SPIRV::Decoration::NoSignedWrap, ST, Reqs)
2708:           .IsSatisfiable) {
2709:     buildOpDecorate(I.getOperand(0).getReg(), I, TII,
2710:                     SPIRV::Decoration::NoSignedWrap, {});
2711:   }
2712:   if (I.getFlag(MachineInstr::MIFlag::NoUWrap) && TII.canUseNUW(I) &&
2713:       getSymbolicOperandRequirements(SPIRV::OperandCategory::DecorationOperand,
2714:                                      SPIRV::Decoration::NoUnsignedWrap, ST,
2715:                                      Reqs)
2716:           .IsSatisfiable) {
2717:     buildOpDecorate(I.getOperand(0).getReg(), I, TII,
2718:                     SPIRV::Decoration::NoUnsignedWrap, {});
2719:   }
2720:   // In Kernel environments, FPFastMathMode on OpExtInst is valid per core
2721:   // spec. For other instruction types, SPV_KHR_float_controls2 is required.
2722:   bool CanUseFM =
2723:       TII.canUseFastMathFlags(
2724:           I, ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2)) ||
2725:       (ST.isKernel() && I.getOpcode() == SPIRV::OpExtInst);
2726:   if (!CanUseFM)
2727:     return;
2728:
2729:   unsigned FMFlags = getFastMathFlags(I, ST);
2730:   if (FMFlags == SPIRV::FPFastMathMode::None) {
2731:     // We also need to check if any FPFastMathDefault info was set for the
2732:     // types used in this instruction.
2733:     if (FPFastMathDefaultInfoVec.empty())
2734:       return;
2735:
2736:     // There are three types of instructions that can use fast math flags:
2737:     // 1. Arithmetic instructions (FAdd, FMul, FSub, FDiv, FRem, etc.)
2738:     // 2. Relational instructions (FCmp, FOrd, FUnord, etc.)
2739:     // 3. Extended instructions (ExtInst)
2740:     // For arithmetic instructions, the floating point type can be in the
2741:     // result type or in the operands, but they all must be the same.
2742:     // For the relational and logical instructions, the floating point type
2743:     // can only be in the operands 1 and 2, not the result type. Also, the
2744:     // operands must have the same type. For the extended instructions, the
2745:     // floating point type can be in the result type or in the operands. It's
2746:     // unclear if the operands and the result type must be the same. Let's
2747:     // assume they must be. Therefore, for 1. and 2., we can check the first
2748:     // operand type, and for 3. we can check the result type.
2749:     assert(I.getNumOperands() >= 3 && "Expected at least 3 operands");
2750:     Register ResReg = I.getOpcode() == SPIRV::OpExtInst
2751:                           ? I.getOperand(1).getReg()
2752:                           : I.getOperand(2).getReg();
2753:     SPIRVTypeInst ResType = GR->getSPIRVTypeForVReg(ResReg, I.getMF());
2754:     const Type *Ty = GR->getTypeForSPIRVType(ResType);
2755:     Ty = Ty->isVectorTy() ? cast<VectorType>(Ty)->getElementType() : Ty;
2756:
2757:     // Match instruction type with the FPFastMathDefaultInfoVec.
2758:     bool Emit = false;
2759:     for (SPIRV::FPFastMathDefaultInfo &Elem : FPFastMathDefaultInfoVec) {
2760:       if (Ty == Elem.Ty) {
```
- EN: This range implements operational logic in helpers such as canUseExtension, isFastMathModeAvailable, isKernel, getFastMathFlags, translating backend policy into executable code.
- CN: 这一段实现了 canUseExtension、isFastMathModeAvailable、isKernel、getFastMathFlags 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2761-2880
```cpp
2761:         FMFlags = Elem.FastMathFlags;
2762:         Emit = Elem.ContractionOff || Elem.SignedZeroInfNanPreserve ||
2763:                Elem.FPFastMathDefault;
2764:         break;
2765:       }
2766:     }
2767:
2768:     if (FMFlags == SPIRV::FPFastMathMode::None && !Emit)
2769:       return;
2770:   }
2771:   if (isFastMathModeAvailable(ST)) {
2772:     Register DstReg = I.getOperand(0).getReg();
2773:     buildOpDecorate(DstReg, I, TII, SPIRV::Decoration::FPFastMathMode,
2774:                     {FMFlags});
2775:   }
2776: }
2777:
2778: // Walk all functions and add decorations related to MI flags.
2779: static void addDecorations(const Module &M, const SPIRVInstrInfo &TII,
2780:                            MachineModuleInfo *MMI, const SPIRVSubtarget &ST,
2781:                            SPIRV::ModuleAnalysisInfo &MAI,
2782:                            const SPIRVGlobalRegistry *GR) {
2783:   for (const Function &F : M) {
2784:     MachineFunction *MF = MMI->getMachineFunction(F);
2785:     if (!MF)
2786:       continue;
2787:
2788:     for (auto &MBB : *MF)
2789:       for (auto &MI : MBB)
2790:         handleMIFlagDecoration(MI, ST, TII, MAI.Reqs, GR,
2791:                                MAI.FPFastMathDefaultInfoMap[&F]);
2792:   }
2793: }
2794:
2795: static void addMBBNames(const Module &M, const SPIRVInstrInfo &TII,
2796:                         MachineModuleInfo *MMI, const SPIRVSubtarget &ST,
2797:                         SPIRV::ModuleAnalysisInfo &MAI) {
2798:   for (const Function &F : M) {
2799:     MachineFunction *MF = MMI->getMachineFunction(F);
2800:     if (!MF)
2801:       continue;
2802:     if (MF->getFunction()
2803:             .getFnAttribute(SPIRV_BACKEND_SERVICE_FUN_NAME)
2804:             .isValid())
2805:       continue;
2806:     MachineRegisterInfo &MRI = MF->getRegInfo();
2807:     for (auto &MBB : *MF) {
2808:       if (!MBB.hasName() || MBB.empty())
2809:         continue;
2810:       // Emit basic block names.
2811:       Register Reg = MRI.createGenericVirtualRegister(LLT::scalar(64));
2812:       MRI.setRegClass(Reg, &SPIRV::IDRegClass);
2813:       buildOpName(Reg, MBB.getName(), *std::prev(MBB.end()), TII);
2814:       MCRegister GlobalReg = MAI.getOrCreateMBBRegister(MBB);
2815:       MAI.setRegisterAlias(MF, Reg, GlobalReg);
2816:     }
2817:   }
2818: }
2819:
2820: // patching Instruction::PHI to SPIRV::OpPhi
2821: static void patchPhis(const Module &M, SPIRVGlobalRegistry *GR,
2822:                       const SPIRVInstrInfo &TII, MachineModuleInfo *MMI) {
2823:   for (const Function &F : M) {
2824:     MachineFunction *MF = MMI->getMachineFunction(F);
2825:     if (!MF)
2826:       continue;
2827:     for (auto &MBB : *MF) {
2828:       for (MachineInstr &MI : MBB.phis()) {
2829:         MI.setDesc(TII.get(SPIRV::OpPhi));
2830:         Register ResTypeReg = GR->getSPIRVTypeID(
2831:             GR->getSPIRVTypeForVReg(MI.getOperand(0).getReg(), MF));
2832:         MI.insert(MI.operands_begin() + 1,
2833:                   {MachineOperand::CreateReg(ResTypeReg, false)});
2834:       }
2835:     }
2836:
2837:     MF->getProperties().setNoPHIs();
2838:   }
2839: }
2840:
2841: static SPIRV::FPFastMathDefaultInfoVector &getOrCreateFPFastMathDefaultInfoVec(
2842:     const Module &M, SPIRV::ModuleAnalysisInfo &MAI, const Function *F) {
2843:   auto it = MAI.FPFastMathDefaultInfoMap.find(F);
2844:   if (it != MAI.FPFastMathDefaultInfoMap.end())
2845:     return it->second;
2846:
2847:   // If the map does not contain the entry, create a new one. Initialize it to
2848:   // contain all 3 elements sorted by bit width of target type: {half, float,
2849:   // double}.
2850:   SPIRV::FPFastMathDefaultInfoVector FPFastMathDefaultInfoVec;
2851:   FPFastMathDefaultInfoVec.emplace_back(Type::getHalfTy(M.getContext()),
2852:                                         SPIRV::FPFastMathMode::None);
2853:   FPFastMathDefaultInfoVec.emplace_back(Type::getFloatTy(M.getContext()),
2854:                                         SPIRV::FPFastMathMode::None);
2855:   FPFastMathDefaultInfoVec.emplace_back(Type::getDoubleTy(M.getContext()),
2856:                                         SPIRV::FPFastMathMode::None);
2857:   return MAI.FPFastMathDefaultInfoMap[F] = std::move(FPFastMathDefaultInfoVec);
2858: }
2859:
2860: static SPIRV::FPFastMathDefaultInfo &getFPFastMathDefaultInfo(
2861:     SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec,
2862:     const Type *Ty) {
2863:   size_t BitWidth = Ty->getScalarSizeInBits();
2864:   int Index =
2865:       SPIRV::FPFastMathDefaultInfoVector::computeFPFastMathDefaultInfoVecIndex(
2866:           BitWidth);
2867:   assert(Index >= 0 && Index < 3 &&
2868:          "Expected FPFastMathDefaultInfo for half, float, or double");
2869:   assert(FPFastMathDefaultInfoVec.size() == 3 &&
2870:          "Expected FPFastMathDefaultInfoVec to have exactly 3 elements");
2871:   return FPFastMathDefaultInfoVec[Index];
2872: }
2873:
2874: static void collectFPFastMathDefaults(const Module &M,
2875:                                       SPIRV::ModuleAnalysisInfo &MAI,
2876:                                       const SPIRVSubtarget &ST) {
2877:   if (!ST.canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2))
2878:     return;
2879:
2880:   // Store the FPFastMathDefaultInfo in the FPFastMathDefaultInfoMap.
```
- EN: This range implements operational logic in helpers such as getOperand, getMachineFunction, getFnAttribute, isValid, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、getMachineFunction、getFnAttribute、isValid 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2881-2989
```cpp
2881:   // We need the entry point (function) as the key, and the target
2882:   // type and flags as the value.
2883:   // We also need to check ContractionOff and SignedZeroInfNanPreserve
2884:   // execution modes, as they are now deprecated and must be replaced
2885:   // with FPFastMathDefaultInfo.
2886:   auto Node = M.getNamedMetadata("spirv.ExecutionMode");
2887:   if (!Node)
2888:     return;
2889:
2890:   for (unsigned i = 0; i < Node->getNumOperands(); i++) {
2891:     MDNode *MDN = cast<MDNode>(Node->getOperand(i));
2892:     assert(MDN->getNumOperands() >= 2 && "Expected at least 2 operands");
2893:     const Function *F = cast<Function>(
2894:         cast<ConstantAsMetadata>(MDN->getOperand(0))->getValue());
2895:     const auto EM =
2896:         cast<ConstantInt>(
2897:             cast<ConstantAsMetadata>(MDN->getOperand(1))->getValue())
2898:             ->getZExtValue();
2899:     if (EM == SPIRV::ExecutionMode::FPFastMathDefault) {
2900:       assert(MDN->getNumOperands() == 4 &&
2901:              "Expected 4 operands for FPFastMathDefault");
2902:
2903:       const Type *T = cast<ValueAsMetadata>(MDN->getOperand(2))->getType();
2904:       unsigned Flags =
2905:           cast<ConstantInt>(
2906:               cast<ConstantAsMetadata>(MDN->getOperand(3))->getValue())
2907:               ->getZExtValue();
2908:       SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec =
2909:           getOrCreateFPFastMathDefaultInfoVec(M, MAI, F);
2910:       SPIRV::FPFastMathDefaultInfo &Info =
2911:           getFPFastMathDefaultInfo(FPFastMathDefaultInfoVec, T);
2912:       Info.FastMathFlags = Flags;
2913:       Info.FPFastMathDefault = true;
2914:     } else if (EM == SPIRV::ExecutionMode::ContractionOff) {
2915:       assert(MDN->getNumOperands() == 2 &&
2916:              "Expected no operands for ContractionOff");
2917:
2918:       // We need to save this info for every possible FP type, i.e. {half,
2919:       // float, double, fp128}.
2920:       SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec =
2921:           getOrCreateFPFastMathDefaultInfoVec(M, MAI, F);
2922:       for (SPIRV::FPFastMathDefaultInfo &Info : FPFastMathDefaultInfoVec) {
2923:         Info.ContractionOff = true;
2924:       }
2925:     } else if (EM == SPIRV::ExecutionMode::SignedZeroInfNanPreserve) {
2926:       assert(MDN->getNumOperands() == 3 &&
2927:              "Expected 1 operand for SignedZeroInfNanPreserve");
2928:       unsigned TargetWidth =
2929:           cast<ConstantInt>(
2930:               cast<ConstantAsMetadata>(MDN->getOperand(2))->getValue())
2931:               ->getZExtValue();
2932:       // We need to save this info only for the FP type with TargetWidth.
2933:       SPIRV::FPFastMathDefaultInfoVector &FPFastMathDefaultInfoVec =
2934:           getOrCreateFPFastMathDefaultInfoVec(M, MAI, F);
2935:       int Index = SPIRV::FPFastMathDefaultInfoVector::
2936:           computeFPFastMathDefaultInfoVecIndex(TargetWidth);
2937:       assert(Index >= 0 && Index < 3 &&
2938:              "Expected FPFastMathDefaultInfo for half, float, or double");
2939:       assert(FPFastMathDefaultInfoVec.size() == 3 &&
2940:              "Expected FPFastMathDefaultInfoVec to have exactly 3 elements");
2941:       FPFastMathDefaultInfoVec[Index].SignedZeroInfNanPreserve = true;
2942:     }
2943:   }
2944: }
2945:
2946: void SPIRVModuleAnalysis::getAnalysisUsage(AnalysisUsage &AU) const {
2947:   AU.addRequired<TargetPassConfig>();
2948:   AU.addRequired<MachineModuleInfoWrapperPass>();
2949: }
2950:
2951: bool SPIRVModuleAnalysis::runOnModule(Module &M) {
2952:   SPIRVTargetMachine &TM =
2953:       getAnalysis<TargetPassConfig>().getTM<SPIRVTargetMachine>();
2954:   ST = TM.getSubtargetImpl();
2955:   GR = ST->getSPIRVGlobalRegistry();
2956:   TII = ST->getInstrInfo();
2957:
2958:   MMI = &getAnalysis<MachineModuleInfoWrapperPass>().getMMI();
2959:
2960:   setBaseInfo(M);
2961:
2962:   patchPhis(M, GR, *TII, MMI);
2963:
2964:   addMBBNames(M, *TII, MMI, *ST, MAI);
2965:   collectFPFastMathDefaults(M, MAI, *ST);
2966:   addDecorations(M, *TII, MMI, *ST, MAI, GR);
2967:
2968:   collectReqs(M, MAI, MMI, *ST);
2969:
2970:   // Process type/const/global var/func decl instructions, number their
2971:   // destination registers from 0 to N, collect Extensions and Capabilities.
2972:   collectReqs(M, MAI, MMI, *ST);
2973:   collectDeclarations(M);
2974:
2975:   // Number rest of registers from N+1 onwards.
2976:   numberRegistersGlobally(M);
2977:
2978:   // Collect OpName, OpEntryPoint, OpDecorate etc, process other instructions.
2979:   processOtherInstrs(M);
2980:
2981:   // If there are no entry points, we need the Linkage capability.
2982:   if (MAI.MS[SPIRV::MB_EntryPoints].empty())
2983:     MAI.Reqs.addCapability(SPIRV::Capability::Linkage);
2984:
2985:   // Set maximum ID used.
2986:   GR->setBound(MAI.MaxID);
2987:
2988:   return false;
2989: }
```
- EN: This range implements operational logic in helpers such as getNamedMetadata, getNumOperands, getOperand, assert, translating backend policy into executable code.
- CN: 这一段实现了 getNamedMetadata、getNumOperands、getOperand、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include cl::init, AvoidCapabilitiesSet, getOperand, getZExtValue, insert, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::init, AvoidCapabilitiesSet, getOperand, getZExtValue, insert，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVModuleAnalysis.h`
  - `MCTargetDesc/SPIRVBaseInfo.h`
  - `MCTargetDesc/SPIRVMCTargetDesc.h`
  - `SPIRV.h`
  - `SPIRVSubtarget.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/STLExtras.h`
  - `llvm/CodeGen/MachineModuleInfo.h`
  - `llvm/CodeGen/TargetPassConfig.h`
