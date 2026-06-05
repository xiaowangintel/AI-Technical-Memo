# SPIRVAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/SPIRVAsmPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to the SPIR-V assembly language.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVAsmPrinter.cpp - SPIR-V LLVM assembly writer ------*- C++ -*--===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains a printer that converts from our internal representation
10: // of machine-dependent LLVM code to the SPIR-V assembly language.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "MCTargetDesc/SPIRVInstPrinter.h"
15: #include "SPIRV.h"
16: #include "SPIRVInstrInfo.h"
17: #include "SPIRVMCInstLower.h"
18: #include "SPIRVModuleAnalysis.h"
19: #include "SPIRVNonSemanticDebugHandler.h"
20: #include "SPIRVSubtarget.h"
21: #include "SPIRVTargetMachine.h"
22: #include "SPIRVUtils.h"
23: #include "TargetInfo/SPIRVTargetInfo.h"
24: #include "llvm/ADT/DenseMap.h"
25: #include "llvm/Analysis/ValueTracking.h"
26: #include "llvm/CodeGen/AsmPrinter.h"
27: #include "llvm/CodeGen/MachineConstantPool.h"
28: #include "llvm/CodeGen/MachineInstr.h"
29: #include "llvm/CodeGen/MachineModuleInfo.h"
30: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
31: #include "llvm/MC/MCAsmInfo.h"
32: #include "llvm/MC/MCAssembler.h"
33: #include "llvm/MC/MCInst.h"
34: #include "llvm/MC/MCObjectStreamer.h"
35: #include "llvm/MC/MCSPIRVObjectWriter.h"
36: #include "llvm/MC/MCStreamer.h"
37: #include "llvm/MC/MCSymbol.h"
38: #include "llvm/MC/TargetRegistry.h"
39: #include "llvm/Support/Compiler.h"
40: #include "llvm/Support/raw_ostream.h"
41:
42: using namespace llvm;
43:
44: #define DEBUG_TYPE "asm-printer"
45:
46: namespace {
47: class SPIRVAsmPrinter : public AsmPrinter {
48:   unsigned NLabels = 0;
49:   SmallPtrSet<const MachineBasicBlock *, 8> LabeledMBB;
50:
51: public:
52:   explicit SPIRVAsmPrinter(TargetMachine &TM,
53:                            std::unique_ptr<MCStreamer> Streamer)
54:       : AsmPrinter(TM, std::move(Streamer), ID), ModuleSectionsEmitted(false),
55:         ST(nullptr), TII(nullptr), MAI(nullptr) {}
56:   static char ID;
57:   bool ModuleSectionsEmitted;
58:   const SPIRVSubtarget *ST;
59:   const SPIRVInstrInfo *TII;
60:
```
- EN: This range defines or declares important types such as SPIRVAsmPrinter, ST, shaping the data model used by SPIRVAsmPrinter.cpp.
- CN: 这一段定义或声明了 SPIRVAsmPrinter、ST 等关键类型，构成 SPIRVAsmPrinter.cpp 使用的数据模型。

### Lines 61-120
```cpp
 61:   StringRef getPassName() const override { return "SPIRV Assembly Printer"; }
 62:   void printOperand(const MachineInstr *MI, int OpNum, raw_ostream &O);
 63:   bool PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
 64:                        const char *ExtraCode, raw_ostream &O) override;
 65:
 66:   void outputMCInst(MCInst &Inst);
 67:   void outputInstruction(const MachineInstr *MI);
 68:   void outputModuleSection(SPIRV::ModuleSectionType MSType);
 69:   void outputGlobalRequirements();
 70:   void outputEntryPoints();
 71:   void outputDebugSourceAndStrings(const Module &M);
 72:   void outputOpExtInstImports(const Module &M);
 73:   void outputOpMemoryModel();
 74:   void outputOpFunctionEnd();
 75:   void outputExtFuncDecls();
 76:   void outputExecutionModeFromMDNode(MCRegister Reg, MDNode *Node,
 77:                                      SPIRV::ExecutionMode::ExecutionMode EM,
 78:                                      unsigned ExpectMDOps, int64_t DefVal);
 79:   void outputExecutionModeFromNumthreadsAttribute(
 80:       const MCRegister &Reg, const Attribute &Attr,
 81:       SPIRV::ExecutionMode::ExecutionMode EM);
 82:   void outputExecutionModeFromEnableMaximalReconvergenceAttr(
 83:       const MCRegister &Reg, const SPIRVSubtarget &ST);
 84:   void outputExecutionMode(const Module &M);
 85:   void outputAnnotations(const Module &M);
 86:   void outputModuleSections();
 87:   void outputFPFastMathDefaultInfo();
 88:   bool isHidden() {
 89:     return MF->getFunction()
 90:         .getFnAttribute(SPIRV_BACKEND_SERVICE_FUN_NAME)
 91:         .isValid();
 92:   }
 93:
 94:   void emitInstruction(const MachineInstr *MI) override;
 95:   void emitFunctionEntryLabel() override {}
 96:   void emitFunctionHeader() override;
 97:   void emitFunctionBodyStart() override {}
 98:   void emitFunctionBodyEnd() override;
 99:   void emitBasicBlockStart(const MachineBasicBlock &MBB) override;
100:   void emitBasicBlockEnd(const MachineBasicBlock &MBB) override {}
101:   void emitGlobalVariable(const GlobalVariable *GV) override {}
102:   void emitOpLabel(const MachineBasicBlock &MBB);
103:   void emitEndOfAsmFile(Module &M) override;
104:   bool doInitialization(Module &M) override;
105:
106:   void getAnalysisUsage(AnalysisUsage &AU) const override;
107:   SPIRV::ModuleAnalysisInfo *MAI;
108:
109:   // Non-owning pointer to the NSDI handler registered via addAsmPrinterHandler.
110:   // The handler's lifetime is managed by AsmPrinter (the base class of this
111:   // object), so this pointer cannot dangle.
112:   SPIRVNonSemanticDebugHandler *NSDebugHandler = nullptr;
113:
114: protected:
115:   void cleanUp(Module &M);
116: };
117: } // namespace
118:
119: void SPIRVAsmPrinter::getAnalysisUsage(AnalysisUsage &AU) const {
120:   AU.addRequired<SPIRVModuleAnalysis>();
```
- EN: This range implements operational logic in helpers such as getPassName, printOperand, outputMCInst, outputInstruction, translating backend policy into executable code.
- CN: 这一段实现了 getPassName、printOperand、outputMCInst、outputInstruction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:   AU.addPreserved<SPIRVModuleAnalysis>();
122:   AsmPrinter::getAnalysisUsage(AU);
123: }
124:
125: // If the module has no functions, we need output global info anyway.
126: void SPIRVAsmPrinter::emitEndOfAsmFile(Module &M) {
127:   if (!ModuleSectionsEmitted) {
128:     outputModuleSections();
129:     ModuleSectionsEmitted = true;
130:   }
131:
132:   ST = static_cast<const SPIRVTargetMachine &>(TM).getSubtargetImpl();
133:   // SPIRVModuleAnalysis sets GR->Bound = MAI->MaxID before printing. Any IDs
134:   // allocated by AsmPrinter handlers (e.g. SPIRVNonSemanticDebugHandler) during
135:   // outputModuleSections() are not counted. Refresh the bound here so the
136:   // formula below sees the final allocation count.
137:   if (MAI)
138:     ST->getSPIRVGlobalRegistry()->setBound(MAI->MaxID);
139:   VersionTuple SPIRVVersion = ST->getSPIRVVersion();
140:   uint32_t Major = SPIRVVersion.getMajor();
141:   uint32_t Minor = SPIRVVersion.getMinor().value_or(0);
142:   // Bound is an approximation that accounts for the maximum used register
143:   // number and number of generated OpLabels
144:   unsigned Bound = 2 * (ST->getBound() + 1) + NLabels;
145:   if (MCAssembler *Asm = OutStreamer->getAssemblerPtr())
146:     static_cast<SPIRVObjectWriter &>(Asm->getWriter())
147:         .setBuildVersion(Major, Minor, Bound);
148:
149:   cleanUp(M);
150: }
151:
152: // Any cleanup actions with the Module after we don't care about its content
153: // anymore.
154: void SPIRVAsmPrinter::cleanUp(Module &M) {
155:   // Verifier disallows uses of intrinsic global variables.
156:   for (StringRef GVName :
157:        {"llvm.global_ctors", "llvm.global_dtors", "llvm.used"}) {
158:     if (GlobalVariable *GV = M.getNamedGlobal(GVName))
159:       GV->setName("");
160:   }
161: }
162:
163: void SPIRVAsmPrinter::emitFunctionHeader() {
164:   if (!ModuleSectionsEmitted) {
165:     outputModuleSections();
166:     ModuleSectionsEmitted = true;
167:   }
168:   // Get the subtarget from the current MachineFunction.
169:   ST = &MF->getSubtarget<SPIRVSubtarget>();
170:   TII = ST->getInstrInfo();
171:   const Function &F = MF->getFunction();
172:
173:   if (isVerbose() && !isHidden()) {
174:     OutStreamer->getCommentOS()
175:         << "-- Begin function "
176:         << GlobalValue::dropLLVMManglingEscape(F.getName()) << '\n';
177:   }
178:
179:   auto Section = getObjFileLowering().SectionForGlobal(&F, TM);
180:   MF->setSection(Section);
```
- EN: This range implements operational logic in helpers such as AsmPrinter::getAnalysisUsage, SPIRVAsmPrinter::emitEndOfAsmFile, outputModuleSections, getSubtargetImpl, translating backend policy into executable code.
- CN: 这一段实现了 AsmPrinter::getAnalysisUsage、SPIRVAsmPrinter::emitEndOfAsmFile、outputModuleSections、getSubtargetImpl 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:
182:   // SPIRVAsmPrinter::emitFunctionHeader() does not call the base class,
183:   // so handlers never receive beginFunction() from the normal path. Drive the
184:   // per-function lifecycle here, matching what AsmPrinter::emitFunctionHeader()
185:   // does for other targets.
186:   for (auto &Handler : Handlers) {
187:     Handler->beginFunction(MF);
188:     Handler->beginBasicBlockSection(MF->front());
189:   }
190: }
191:
192: void SPIRVAsmPrinter::outputOpFunctionEnd() {
193:   MCInst FunctionEndInst;
194:   FunctionEndInst.setOpcode(SPIRV::OpFunctionEnd);
195:   outputMCInst(FunctionEndInst);
196: }
197:
198: void SPIRVAsmPrinter::emitFunctionBodyEnd() {
199:   if (!isHidden())
200:     outputOpFunctionEnd();
201: }
202:
203: void SPIRVAsmPrinter::emitOpLabel(const MachineBasicBlock &MBB) {
204:   // Do not emit anything if it's an internal service function.
205:   if (isHidden())
206:     return;
207:
208:   MCInst LabelInst;
209:   LabelInst.setOpcode(SPIRV::OpLabel);
210:   LabelInst.addOperand(MCOperand::createReg(MAI->getOrCreateMBBRegister(MBB)));
211:   outputMCInst(LabelInst);
212:   ++NLabels;
213:   LabeledMBB.insert(&MBB);
214: }
215:
216: void SPIRVAsmPrinter::emitBasicBlockStart(const MachineBasicBlock &MBB) {
217:   // Do not emit anything if it's an internal service function.
218:   if (MBB.empty() || isHidden())
219:     return;
220:
221:   // If it's the first MBB in MF, it has OpFunction and OpFunctionParameter, so
222:   // OpLabel should be output after them.
223:   if (MBB.getNumber() == MF->front().getNumber()) {
224:     for (const MachineInstr &MI : MBB)
225:       if (MI.getOpcode() == SPIRV::OpFunction)
226:         return;
227:     // TODO: this case should be checked by the verifier.
228:     report_fatal_error("OpFunction is expected in the front MBB of MF");
229:   }
230:   emitOpLabel(MBB);
231: }
232:
233: void SPIRVAsmPrinter::printOperand(const MachineInstr *MI, int OpNum,
234:                                    raw_ostream &O) {
235:   const MachineOperand &MO = MI->getOperand(OpNum);
236:
237:   switch (MO.getType()) {
238:   case MachineOperand::MO_Register:
239:     O << SPIRVInstPrinter::getRegisterName(MO.getReg());
240:     break;
```
- EN: This range implements operational logic in helpers such as beginFunction, beginBasicBlockSection, SPIRVAsmPrinter::outputOpFunctionEnd, setOpcode, translating backend policy into executable code.
- CN: 这一段实现了 beginFunction、beginBasicBlockSection、SPIRVAsmPrinter::outputOpFunctionEnd、setOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:
242:   case MachineOperand::MO_Immediate:
243:     O << MO.getImm();
244:     break;
245:
246:   case MachineOperand::MO_FPImmediate:
247:     O << MO.getFPImm();
248:     break;
249:
250:   case MachineOperand::MO_MachineBasicBlock:
251:     O << *MO.getMBB()->getSymbol();
252:     break;
253:
254:   case MachineOperand::MO_GlobalAddress:
255:     O << *getSymbol(MO.getGlobal());
256:     break;
257:
258:   case MachineOperand::MO_BlockAddress: {
259:     MCSymbol *BA = GetBlockAddressSymbol(MO.getBlockAddress());
260:     O << BA->getName();
261:     break;
262:   }
263:
264:   case MachineOperand::MO_ExternalSymbol:
265:     O << *GetExternalSymbolSymbol(MO.getSymbolName());
266:     break;
267:
268:   case MachineOperand::MO_JumpTableIndex:
269:   case MachineOperand::MO_ConstantPoolIndex:
270:   default:
271:     llvm_unreachable("<unknown operand type>");
272:   }
273: }
274:
275: bool SPIRVAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
276:                                       const char *ExtraCode, raw_ostream &O) {
277:   if (ExtraCode && ExtraCode[0])
278:     return true; // Invalid instruction - SPIR-V does not have special modifiers
279:
280:   printOperand(MI, OpNo, O);
281:   return false;
282: }
283:
284: static bool isFuncOrHeaderInstr(const MachineInstr *MI,
285:                                 const SPIRVInstrInfo *TII) {
286:   return TII->isHeaderInstr(*MI) || MI->getOpcode() == SPIRV::OpFunction ||
287:          MI->getOpcode() == SPIRV::OpFunctionParameter;
288: }
289:
290: void SPIRVAsmPrinter::outputMCInst(MCInst &Inst) {
291:   OutStreamer->emitInstruction(Inst, *OutContext.getSubtargetInfo());
292: }
293:
294: void SPIRVAsmPrinter::outputInstruction(const MachineInstr *MI) {
295:   SPIRVMCInstLower MCInstLowering;
296:   MCInst TmpInst;
297:   MCInstLowering.lower(MI, TmpInst, MAI);
298:   outputMCInst(TmpInst);
299: }
300:
```
- EN: This range implements operational logic in helpers such as getImm, getFPImm, getMBB, getSymbol, translating backend policy into executable code.
- CN: 这一段实现了 getImm、getFPImm、getMBB、getSymbol 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301: void SPIRVAsmPrinter::emitInstruction(const MachineInstr *MI) {
302:   SPIRV_MC::verifyInstructionPredicates(MI->getOpcode(),
303:                                         getSubtargetInfo().getFeatureBits());
304:
305:   if (!MAI->getSkipEmission(MI))
306:     outputInstruction(MI);
307:
308:   // Output OpLabel after OpFunction and OpFunctionParameter in the first MBB.
309:   const MachineInstr *NextMI = MI->getNextNode();
310:   if (!LabeledMBB.contains(MI->getParent()) && isFuncOrHeaderInstr(MI, TII) &&
311:       (!NextMI || !isFuncOrHeaderInstr(NextMI, TII))) {
312:     assert(MI->getParent()->getNumber() == MF->front().getNumber() &&
313:            "OpFunction is not in the front MBB of MF");
314:     emitOpLabel(*MI->getParent());
315:   }
316: }
317:
318: void SPIRVAsmPrinter::outputModuleSection(SPIRV::ModuleSectionType MSType) {
319:   for (const MachineInstr *MI : MAI->getMSInstrs(MSType))
320:     outputInstruction(MI);
321: }
322:
323: void SPIRVAsmPrinter::outputDebugSourceAndStrings(const Module &M) {
324:   // Output OpSourceExtensions.
325:   for (auto &Str : MAI->SrcExt) {
326:     MCInst Inst;
327:     Inst.setOpcode(SPIRV::OpSourceExtension);
328:     addStringImm(Str.first(), Inst);
329:     outputMCInst(Inst);
330:   }
331:   // Output OpString.
332:   outputModuleSection(SPIRV::MB_DebugStrings);
333:   // Output OpSource.
334:   MCInst Inst;
335:   Inst.setOpcode(SPIRV::OpSource);
336:   Inst.addOperand(MCOperand::createImm(static_cast<unsigned>(MAI->SrcLang)));
337:   Inst.addOperand(
338:       MCOperand::createImm(static_cast<unsigned>(MAI->SrcLangVersion)));
339:   outputMCInst(Inst);
340:   // Emit OpString instructions for NSDI file paths and type names here, in
341:   // section 7. OpString must precede type/constant declarations per the SPIR-V
342:   // module layout (section 2.4). The OpExtInst instructions that reference
343:   // these strings are emitted later at section 10 by
344:   // emitNonSemanticGlobalDebugInfo().
345:   if (NSDebugHandler)
346:     NSDebugHandler->emitNonSemanticDebugStrings(*MAI);
347: }
348:
349: void SPIRVAsmPrinter::outputOpExtInstImports(const Module &M) {
350:   for (auto &CU : MAI->ExtInstSetMap) {
351:     unsigned Set = CU.first;
352:     MCRegister Reg = CU.second;
353:     MCInst Inst;
354:     Inst.setOpcode(SPIRV::OpExtInstImport);
355:     Inst.addOperand(MCOperand::createReg(Reg));
356:     addStringImm(getExtInstSetName(
357:                      static_cast<SPIRV::InstructionSet::InstructionSet>(Set)),
358:                  Inst);
359:     outputMCInst(Inst);
360:   }
```
- EN: This range implements operational logic in helpers such as SPIRVAsmPrinter::emitInstruction, getSubtargetInfo, outputInstruction, getNextNode, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVAsmPrinter::emitInstruction、getSubtargetInfo、outputInstruction、getNextNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361: }
362:
363: void SPIRVAsmPrinter::outputOpMemoryModel() {
364:   MCInst Inst;
365:   Inst.setOpcode(SPIRV::OpMemoryModel);
366:   Inst.addOperand(MCOperand::createImm(static_cast<unsigned>(MAI->Addr)));
367:   Inst.addOperand(MCOperand::createImm(static_cast<unsigned>(MAI->Mem)));
368:   outputMCInst(Inst);
369: }
370:
371: // Before the OpEntryPoints' output, we need to add the entry point's
372: // interfaces. The interface is a list of IDs of global OpVariable instructions.
373: // These declare the set of global variables from a module that form
374: // the interface of this entry point.
375: void SPIRVAsmPrinter::outputEntryPoints() {
376:   // Find all OpVariable IDs with required StorageClass.
377:   DenseSet<MCRegister> InterfaceIDs;
378:   for (const MachineInstr *MI : MAI->GlobalVarList) {
379:     assert(MI->getOpcode() == SPIRV::OpVariable);
380:     auto SC = static_cast<SPIRV::StorageClass::StorageClass>(
381:         MI->getOperand(2).getImm());
382:     // Before version 1.4, the interface's storage classes are limited to
383:     // the Input and Output storage classes. Starting with version 1.4,
384:     // the interface's storage classes are all storage classes used in
385:     // declaring all global variables referenced by the entry point call tree.
386:     if (ST->isAtLeastSPIRVVer(VersionTuple(1, 4)) ||
387:         SC == SPIRV::StorageClass::Input || SC == SPIRV::StorageClass::Output) {
388:       const MachineFunction *MF = MI->getMF();
389:       MCRegister Reg = MAI->getRegisterAlias(MF, MI->getOperand(0).getReg());
390:       InterfaceIDs.insert(Reg);
391:     }
392:   }
393:
394:   // Output OpEntryPoints adding interface args to all of them.
395:   for (const MachineInstr *MI : MAI->getMSInstrs(SPIRV::MB_EntryPoints)) {
396:     SPIRVMCInstLower MCInstLowering;
397:     MCInst TmpInst;
398:     MCInstLowering.lower(MI, TmpInst, MAI);
399:     for (MCRegister Reg : InterfaceIDs) {
400:       assert(Reg.isValid());
401:       TmpInst.addOperand(MCOperand::createReg(Reg));
402:     }
403:     outputMCInst(TmpInst);
404:   }
405: }
406:
407: // Create global OpCapability instructions for the required capabilities.
408: void SPIRVAsmPrinter::outputGlobalRequirements() {
409:   // Abort here if not all requirements can be satisfied.
410:   MAI->Reqs.checkSatisfiable(*ST);
411:
412:   for (const auto &Cap : MAI->Reqs.getMinimalCapabilities()) {
413:     MCInst Inst;
414:     Inst.setOpcode(SPIRV::OpCapability);
415:     Inst.addOperand(MCOperand::createImm(Cap));
416:     outputMCInst(Inst);
417:   }
418:
419:   // Generate the final OpExtensions with strings instead of enums.
420:   for (const auto &Ext : MAI->Reqs.getExtensions()) {
```
- EN: This range implements operational logic in helpers such as SPIRVAsmPrinter::outputOpMemoryModel, setOpcode, addOperand, outputMCInst, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVAsmPrinter::outputOpMemoryModel、setOpcode、addOperand、outputMCInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:     MCInst Inst;
422:     Inst.setOpcode(SPIRV::OpExtension);
423:     addStringImm(getSymbolicOperandMnemonic(
424:                      SPIRV::OperandCategory::ExtensionOperand, Ext),
425:                  Inst);
426:     outputMCInst(Inst);
427:   }
428:   // TODO add a pseudo instr for version number.
429: }
430:
431: void SPIRVAsmPrinter::outputExtFuncDecls() {
432:   // Insert OpFunctionEnd after each declaration.
433:   auto I = MAI->getMSInstrs(SPIRV::MB_ExtFuncDecls).begin(),
434:        E = MAI->getMSInstrs(SPIRV::MB_ExtFuncDecls).end();
435:   for (; I != E; ++I) {
436:     outputInstruction(*I);
437:     if ((I + 1) == E || (*(I + 1))->getOpcode() == SPIRV::OpFunction)
438:       outputOpFunctionEnd();
439:   }
440: }
441:
442: // Encode LLVM type by SPIR-V execution mode VecTypeHint.
443: static unsigned encodeVecTypeHint(Type *Ty) {
444:   if (Ty->isHalfTy())
445:     return 4;
446:   if (Ty->isFloatTy())
447:     return 5;
448:   if (Ty->isDoubleTy())
449:     return 6;
450:   if (IntegerType *IntTy = dyn_cast<IntegerType>(Ty)) {
451:     switch (IntTy->getIntegerBitWidth()) {
452:     case 8:
453:       return 0;
454:     case 16:
455:       return 1;
456:     case 32:
457:       return 2;
458:     case 64:
459:       return 3;
460:     default:
461:       llvm_unreachable("invalid integer type");
462:     }
463:   }
464:   if (FixedVectorType *VecTy = dyn_cast<FixedVectorType>(Ty)) {
465:     Type *EleTy = VecTy->getElementType();
466:     unsigned Size = VecTy->getNumElements();
467:     return Size << 16 | encodeVecTypeHint(EleTy);
468:   }
469:   llvm_unreachable("invalid type");
470: }
471:
472: static void addOpsFromMDNode(MDNode *MDN, MCInst &Inst,
473:                              SPIRV::ModuleAnalysisInfo *MAI) {
474:   for (const MDOperand &MDOp : MDN->operands()) {
475:     if (auto *CMeta = dyn_cast<ConstantAsMetadata>(MDOp)) {
476:       Constant *C = CMeta->getValue();
477:       if (ConstantInt *Const = dyn_cast<ConstantInt>(C)) {
478:         Inst.addOperand(MCOperand::createImm(Const->getZExtValue()));
479:       } else if (auto *CE = dyn_cast<Function>(C)) {
480:         MCRegister FuncReg = MAI->getGlobalObjReg(CE);
```
- EN: This range implements operational logic in helpers such as setOpcode, outputMCInst, SPIRVAsmPrinter::outputExtFuncDecls, getMSInstrs, translating backend policy into executable code.
- CN: 这一段实现了 setOpcode、outputMCInst、SPIRVAsmPrinter::outputExtFuncDecls、getMSInstrs 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-540
```cpp
481:         assert(FuncReg.isValid());
482:         Inst.addOperand(MCOperand::createReg(FuncReg));
483:       }
484:     }
485:   }
486: }
487:
488: void SPIRVAsmPrinter::outputExecutionModeFromMDNode(
489:     MCRegister Reg, MDNode *Node, SPIRV::ExecutionMode::ExecutionMode EM,
490:     unsigned ExpectMDOps, int64_t DefVal) {
491:   MCInst Inst;
492:   Inst.setOpcode(SPIRV::OpExecutionMode);
493:   Inst.addOperand(MCOperand::createReg(Reg));
494:   Inst.addOperand(MCOperand::createImm(static_cast<unsigned>(EM)));
495:   addOpsFromMDNode(Node, Inst, MAI);
496:   // reqd_work_group_size and work_group_size_hint require 3 operands,
497:   // if metadata contains less operands, just add a default value
498:   unsigned NodeSz = Node->getNumOperands();
499:   if (ExpectMDOps > 0 && NodeSz < ExpectMDOps)
500:     for (unsigned i = NodeSz; i < ExpectMDOps; ++i)
501:       Inst.addOperand(MCOperand::createImm(DefVal));
502:   outputMCInst(Inst);
503: }
504:
505: void SPIRVAsmPrinter::outputExecutionModeFromNumthreadsAttribute(
506:     const MCRegister &Reg, const Attribute &Attr,
507:     SPIRV::ExecutionMode::ExecutionMode EM) {
508:   assert(Attr.isValid() && "Function called with an invalid attribute.");
509:
510:   MCInst Inst;
511:   Inst.setOpcode(SPIRV::OpExecutionMode);
512:   Inst.addOperand(MCOperand::createReg(Reg));
513:   Inst.addOperand(MCOperand::createImm(static_cast<unsigned>(EM)));
514:
515:   SmallVector<StringRef> NumThreads;
516:   Attr.getValueAsString().split(NumThreads, ',');
517:   assert(NumThreads.size() == 3 && "invalid numthreads");
518:   for (uint32_t i = 0; i < 3; ++i) {
519:     uint32_t V;
520:     [[maybe_unused]] bool Result = NumThreads[i].getAsInteger(10, V);
521:     assert(!Result && "Failed to parse numthreads");
522:     Inst.addOperand(MCOperand::createImm(V));
523:   }
524:
525:   outputMCInst(Inst);
526: }
527:
528: void SPIRVAsmPrinter::outputExecutionModeFromEnableMaximalReconvergenceAttr(
529:     const MCRegister &Reg, const SPIRVSubtarget &ST) {
530:   assert(ST.canUseExtension(SPIRV::Extension::SPV_KHR_maximal_reconvergence) &&
531:          "Function called when SPV_KHR_maximal_reconvergence is not enabled.");
532:
533:   MCInst Inst;
534:   Inst.setOpcode(SPIRV::OpExecutionMode);
535:   Inst.addOperand(MCOperand::createReg(Reg));
536:   unsigned EM =
537:       static_cast<unsigned>(SPIRV::ExecutionMode::MaximallyReconvergesKHR);
538:   Inst.addOperand(MCOperand::createImm(EM));
539:   outputMCInst(Inst);
540: }
```
- EN: This range implements operational logic in helpers such as assert, addOperand, setOpcode, addOpsFromMDNode, translating backend policy into executable code.
- CN: 这一段实现了 assert、addOperand、setOpcode、addOpsFromMDNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-600
```cpp
541:
542: void SPIRVAsmPrinter::outputExecutionMode(const Module &M) {
543:   NamedMDNode *Node = M.getNamedMetadata("spirv.ExecutionMode");
544:   if (Node) {
545:     for (unsigned i = 0; i < Node->getNumOperands(); i++) {
546:       // If SPV_KHR_float_controls2 is enabled and we find any of
547:       // FPFastMathDefault, ContractionOff or SignedZeroInfNanPreserve execution
548:       // modes, skip it, it'll be done somewhere else.
549:       if (ST->canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2)) {
550:         const auto EM =
551:             cast<ConstantInt>(
552:                 cast<ConstantAsMetadata>((Node->getOperand(i))->getOperand(1))
553:                     ->getValue())
554:                 ->getZExtValue();
555:         if (EM == SPIRV::ExecutionMode::FPFastMathDefault ||
556:             EM == SPIRV::ExecutionMode::ContractionOff ||
557:             EM == SPIRV::ExecutionMode::SignedZeroInfNanPreserve)
558:           continue;
559:       }
560:
561:       MCInst Inst;
562:       Inst.setOpcode(SPIRV::OpExecutionMode);
563:       addOpsFromMDNode(cast<MDNode>(Node->getOperand(i)), Inst, MAI);
564:       outputMCInst(Inst);
565:     }
566:     outputFPFastMathDefaultInfo();
567:   }
568:   for (auto FI = M.begin(), E = M.end(); FI != E; ++FI) {
569:     const Function &F = *FI;
570:     // Only operands of OpEntryPoint instructions are allowed to be
571:     // <Entry Point> operands of OpExecutionMode
572:     if (F.isDeclaration() || !isEntryPoint(F))
573:       continue;
574:     MCRegister FReg = MAI->getGlobalObjReg(&F);
575:     assert(FReg.isValid());
576:
577:     if (Attribute Attr = F.getFnAttribute("hlsl.shader"); Attr.isValid()) {
578:       // SPIR-V common validation: Fragment requires OriginUpperLeft or
579:       // OriginLowerLeft.
580:       // VUID-StandaloneSpirv-OriginLowerLeft-04653: Fragment must declare
581:       // OriginUpperLeft.
582:       if (Attr.getValueAsString() == "pixel") {
583:         MCInst Inst;
584:         Inst.setOpcode(SPIRV::OpExecutionMode);
585:         Inst.addOperand(MCOperand::createReg(FReg));
586:         unsigned EM =
587:             static_cast<unsigned>(SPIRV::ExecutionMode::OriginUpperLeft);
588:         Inst.addOperand(MCOperand::createImm(EM));
589:         outputMCInst(Inst);
590:       }
591:     }
592:     if (MDNode *Node = F.getMetadata("reqd_work_group_size"))
593:       outputExecutionModeFromMDNode(FReg, Node, SPIRV::ExecutionMode::LocalSize,
594:                                     3, 1);
595:     if (Attribute Attr = F.getFnAttribute("hlsl.numthreads"); Attr.isValid())
596:       outputExecutionModeFromNumthreadsAttribute(
597:           FReg, Attr, SPIRV::ExecutionMode::LocalSize);
598:     if (Attribute Attr = F.getFnAttribute("enable-maximal-reconvergence");
599:         Attr.getValueAsBool()) {
600:       outputExecutionModeFromEnableMaximalReconvergenceAttr(FReg, *ST);
```
- EN: This range implements operational logic in helpers such as SPIRVAsmPrinter::outputExecutionMode, getNamedMetadata, getNumOperands, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVAsmPrinter::outputExecutionMode、getNamedMetadata、getNumOperands、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 601-660
```cpp
601:     }
602:     if (MDNode *Node = F.getMetadata("work_group_size_hint"))
603:       outputExecutionModeFromMDNode(FReg, Node,
604:                                     SPIRV::ExecutionMode::LocalSizeHint, 3, 1);
605:     if (MDNode *Node = F.getMetadata("intel_reqd_sub_group_size"))
606:       outputExecutionModeFromMDNode(FReg, Node,
607:                                     SPIRV::ExecutionMode::SubgroupSize, 0, 0);
608:     if (MDNode *Node = F.getMetadata("max_work_group_size")) {
609:       if (ST->canUseExtension(SPIRV::Extension::SPV_INTEL_kernel_attributes))
610:         outputExecutionModeFromMDNode(
611:             FReg, Node, SPIRV::ExecutionMode::MaxWorkgroupSizeINTEL, 3, 1);
612:     }
613:     if (MDNode *Node = F.getMetadata("vec_type_hint")) {
614:       MCInst Inst;
615:       Inst.setOpcode(SPIRV::OpExecutionMode);
616:       Inst.addOperand(MCOperand::createReg(FReg));
617:       unsigned EM = static_cast<unsigned>(SPIRV::ExecutionMode::VecTypeHint);
618:       Inst.addOperand(MCOperand::createImm(EM));
619:       unsigned TypeCode = encodeVecTypeHint(getMDOperandAsType(Node, 0));
620:       Inst.addOperand(MCOperand::createImm(TypeCode));
621:       outputMCInst(Inst);
622:     }
623:     if (ST->isKernel() && !M.getNamedMetadata("spirv.ExecutionMode") &&
624:         !M.getNamedMetadata("opencl.enable.FP_CONTRACT")) {
625:       if (ST->canUseExtension(SPIRV::Extension::SPV_KHR_float_controls2)) {
626:         // When SPV_KHR_float_controls2 is enabled, ContractionOff is
627:         // deprecated. We need to use FPFastMathDefault with the appropriate
628:         // flags instead. Since FPFastMathDefault takes a target type, we need
629:         // to emit it for each floating-point type that exists in the module
630:         // to match the effect of ContractionOff. As of now, there are 3 FP
631:         // types: fp16, fp32 and fp64.
632:
633:         // We only end up here because there is no "spirv.ExecutionMode"
634:         // metadata, so that means no FPFastMathDefault. Therefore, we only
635:         // need to make sure AllowContract is set to 0, as the rest of flags.
636:         // We still need to emit the OpExecutionMode instruction, otherwise
637:         // it's up to the client API to define the flags. Therefore, we need
638:         // to find the constant with 0 value.
639:
640:         // Collect the SPIRVTypes for fp16, fp32, and fp64 and the constant of
641:         // type int32 with 0 value to represent the FP Fast Math Mode.
642:         std::vector<const MachineInstr *> SPIRVFloatTypes;
643:         const MachineInstr *ConstZeroInt32 = nullptr;
644:         for (const MachineInstr *MI :
645:              MAI->getMSInstrs(SPIRV::MB_TypeConstVars)) {
646:           unsigned OpCode = MI->getOpcode();
647:
648:           // Collect the SPIRV type if it's a float.
649:           if (OpCode == SPIRV::OpTypeFloat) {
650:             // Skip if the target type is not fp16, fp32, fp64.
651:             const unsigned OpTypeFloatSize = MI->getOperand(1).getImm();
652:             if (OpTypeFloatSize != 16 && OpTypeFloatSize != 32 &&
653:                 OpTypeFloatSize != 64) {
654:               continue;
655:             }
656:             SPIRVFloatTypes.push_back(MI);
657:             continue;
658:           }
659:
660:           if (OpCode == SPIRV::OpConstantNull) {
```
- EN: This range implements operational logic in helpers such as setOpcode, addOperand, encodeVecTypeHint, outputMCInst, translating backend policy into executable code.
- CN: 这一段实现了 setOpcode、addOperand、encodeVecTypeHint、outputMCInst 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 661-720
```cpp
661:             // Check if the constant is int32, if not skip it.
662:             const MachineRegisterInfo &MRI = MI->getMF()->getRegInfo();
663:             MachineInstr *TypeMI = MRI.getVRegDef(MI->getOperand(1).getReg());
664:             bool IsInt32Ty = TypeMI &&
665:                              TypeMI->getOpcode() == SPIRV::OpTypeInt &&
666:                              TypeMI->getOperand(1).getImm() == 32;
667:             if (IsInt32Ty)
668:               ConstZeroInt32 = MI;
669:           }
670:         }
671:
672:         // When SPV_KHR_float_controls2 is enabled, ContractionOff is
673:         // deprecated. We need to use FPFastMathDefault with the appropriate
674:         // flags instead. Since FPFastMathDefault takes a target type, we need
675:         // to emit it for each floating-point type that exists in the module
676:         // to match the effect of ContractionOff. As of now, there are 3 FP
677:         // types: fp16, fp32 and fp64.
678:         for (const MachineInstr *MI : SPIRVFloatTypes) {
679:           MCInst Inst;
680:           Inst.setOpcode(SPIRV::OpExecutionModeId);
681:           Inst.addOperand(MCOperand::createReg(FReg));
682:           unsigned EM =
683:               static_cast<unsigned>(SPIRV::ExecutionMode::FPFastMathDefault);
684:           Inst.addOperand(MCOperand::createImm(EM));
685:           const MachineFunction *MF = MI->getMF();
686:           MCRegister TypeReg =
687:               MAI->getRegisterAlias(MF, MI->getOperand(0).getReg());
688:           Inst.addOperand(MCOperand::createReg(TypeReg));
689:           assert(ConstZeroInt32 && "There should be a constant zero.");
690:           MCRegister ConstReg = MAI->getRegisterAlias(
691:               ConstZeroInt32->getMF(), ConstZeroInt32->getOperand(0).getReg());
692:           Inst.addOperand(MCOperand::createReg(ConstReg));
693:           outputMCInst(Inst);
694:         }
695:       } else {
696:         MCInst Inst;
697:         Inst.setOpcode(SPIRV::OpExecutionMode);
698:         Inst.addOperand(MCOperand::createReg(FReg));
699:         unsigned EM =
700:             static_cast<unsigned>(SPIRV::ExecutionMode::ContractionOff);
701:         Inst.addOperand(MCOperand::createImm(EM));
702:         outputMCInst(Inst);
703:       }
704:     }
705:   }
706: }
707:
708: void SPIRVAsmPrinter::outputAnnotations(const Module &M) {
709:   outputModuleSection(SPIRV::MB_Annotations);
710:   // Process llvm.global.annotations special global variable.
711:   for (auto F = M.global_begin(), E = M.global_end(); F != E; ++F) {
712:     if ((*F).getName() != "llvm.global.annotations")
713:       continue;
714:     const GlobalVariable *V = &(*F);
715:     const ConstantArray *CA = cast<ConstantArray>(V->getOperand(0));
716:     for (Value *Op : CA->operands()) {
717:       ConstantStruct *CS = cast<ConstantStruct>(Op);
718:       // The first field of the struct contains a pointer to
719:       // the annotated variable.
720:       Value *AnnotatedVar = CS->getOperand(0)->stripPointerCasts();
```
- EN: This range implements operational logic in helpers such as getMF, getVRegDef, setOpcode, addOperand, translating backend policy into executable code.
- CN: 这一段实现了 getMF、getVRegDef、setOpcode、addOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-780
```cpp
721:       auto *GO = dyn_cast<GlobalObject>(AnnotatedVar);
722:       MCRegister Reg = GO ? MAI->getGlobalObjReg(GO) : MCRegister();
723:       if (!Reg.isValid()) {
724:         std::string DiagMsg;
725:         raw_string_ostream OS(DiagMsg);
726:         AnnotatedVar->print(OS);
727:         DiagMsg = "Unsupported value in llvm.global.annotations: " + DiagMsg;
728:         report_fatal_error(DiagMsg.c_str());
729:       }
730:
731:       // The second field contains a pointer to a global annotation string.
732:       GlobalVariable *GV =
733:           cast<GlobalVariable>(CS->getOperand(1)->stripPointerCasts());
734:
735:       StringRef AnnotationString;
736:       [[maybe_unused]] bool Success =
737:           getConstantStringInfo(GV, AnnotationString);
738:       assert(Success && "Failed to get annotation string");
739:       MCInst Inst;
740:       Inst.setOpcode(SPIRV::OpDecorate);
741:       Inst.addOperand(MCOperand::createReg(Reg));
742:       unsigned Dec = static_cast<unsigned>(SPIRV::Decoration::UserSemantic);
743:       Inst.addOperand(MCOperand::createImm(Dec));
744:       addStringImm(AnnotationString, Inst);
745:       outputMCInst(Inst);
746:     }
747:   }
748: }
749:
750: void SPIRVAsmPrinter::outputFPFastMathDefaultInfo() {
751:   // Collect the SPIRVTypes that are OpTypeFloat and the constants of type
752:   // int32, that might be used as FP Fast Math Mode.
753:   std::vector<const MachineInstr *> SPIRVFloatTypes;
754:   // Hashtable to associate immediate values with the constant holding them.
755:   std::unordered_map<int, const MachineInstr *> ConstMap;
756:   for (const MachineInstr *MI : MAI->getMSInstrs(SPIRV::MB_TypeConstVars)) {
757:     // Skip if the instruction is not OpTypeFloat or OpConstant.
758:     unsigned OpCode = MI->getOpcode();
759:     if (OpCode != SPIRV::OpTypeFloat && OpCode != SPIRV::OpConstantI &&
760:         OpCode != SPIRV::OpConstantNull)
761:       continue;
762:
763:     // Collect the SPIRV type if it's a float.
764:     if (OpCode == SPIRV::OpTypeFloat) {
765:       SPIRVFloatTypes.push_back(MI);
766:     } else {
767:       // Check if the constant is int32, if not skip it.
768:       const MachineRegisterInfo &MRI = MI->getMF()->getRegInfo();
769:       MachineInstr *TypeMI = MRI.getVRegDef(MI->getOperand(1).getReg());
770:       if (!TypeMI || TypeMI->getOpcode() != SPIRV::OpTypeInt ||
771:           TypeMI->getOperand(1).getImm() != 32)
772:         continue;
773:
774:       if (OpCode == SPIRV::OpConstantI)
775:         ConstMap[MI->getOperand(2).getImm()] = MI;
776:       else
777:         ConstMap[0] = MI;
778:     }
779:   }
780:
```
- EN: This range implements operational logic in helpers such as getGlobalObjReg, OS, print, report_fatal_error, translating backend policy into executable code.
- CN: 这一段实现了 getGlobalObjReg、OS、print、report_fatal_error 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 781-840
```cpp
781:   for (const auto &[Func, FPFastMathDefaultInfoVec] :
782:        MAI->FPFastMathDefaultInfoMap) {
783:     if (FPFastMathDefaultInfoVec.empty())
784:       continue;
785:
786:     for (const MachineInstr *MI : SPIRVFloatTypes) {
787:       unsigned OpTypeFloatSize = MI->getOperand(1).getImm();
788:       unsigned Index = SPIRV::FPFastMathDefaultInfoVector::
789:           computeFPFastMathDefaultInfoVecIndex(OpTypeFloatSize);
790:       assert(Index < FPFastMathDefaultInfoVec.size() &&
791:              "Index out of bounds for FPFastMathDefaultInfoVec");
792:       const auto &FPFastMathDefaultInfo = FPFastMathDefaultInfoVec[Index];
793:       assert(FPFastMathDefaultInfo.Ty &&
794:              "Expected target type for FPFastMathDefaultInfo");
795:       assert(FPFastMathDefaultInfo.Ty->getScalarSizeInBits() ==
796:                  OpTypeFloatSize &&
797:              "Mismatched float type size");
798:       MCInst Inst;
799:       Inst.setOpcode(SPIRV::OpExecutionModeId);
800:       MCRegister FuncReg = MAI->getGlobalObjReg(Func);
801:       assert(FuncReg.isValid());
802:       Inst.addOperand(MCOperand::createReg(FuncReg));
803:       Inst.addOperand(
804:           MCOperand::createImm(SPIRV::ExecutionMode::FPFastMathDefault));
805:       MCRegister TypeReg =
806:           MAI->getRegisterAlias(MI->getMF(), MI->getOperand(0).getReg());
807:       Inst.addOperand(MCOperand::createReg(TypeReg));
808:       unsigned Flags = FPFastMathDefaultInfo.FastMathFlags;
809:       if (FPFastMathDefaultInfo.ContractionOff &&
810:           (Flags & SPIRV::FPFastMathMode::AllowContract))
811:         report_fatal_error(
812:             "Conflicting FPFastMathFlags: ContractionOff and AllowContract");
813:
814:       if (FPFastMathDefaultInfo.SignedZeroInfNanPreserve &&
815:           !(Flags &
816:             (SPIRV::FPFastMathMode::NotNaN | SPIRV::FPFastMathMode::NotInf |
817:              SPIRV::FPFastMathMode::NSZ))) {
818:         if (FPFastMathDefaultInfo.FPFastMathDefault)
819:           report_fatal_error("Conflicting FPFastMathFlags: "
820:                              "SignedZeroInfNanPreserve but at least one of "
821:                              "NotNaN/NotInf/NSZ is enabled.");
822:       }
823:
824:       // Don't emit if none of the execution modes was used.
825:       if (Flags == SPIRV::FPFastMathMode::None &&
826:           !FPFastMathDefaultInfo.ContractionOff &&
827:           !FPFastMathDefaultInfo.SignedZeroInfNanPreserve &&
828:           !FPFastMathDefaultInfo.FPFastMathDefault)
829:         continue;
830:
831:       // Retrieve the constant instruction for the immediate value.
832:       auto It = ConstMap.find(Flags);
833:       if (It == ConstMap.end())
834:         report_fatal_error("Expected constant instruction for FP Fast Math "
835:                            "Mode operand of FPFastMathDefault execution mode.");
836:       const MachineInstr *ConstMI = It->second;
837:       MCRegister ConstReg = MAI->getRegisterAlias(
838:           ConstMI->getMF(), ConstMI->getOperand(0).getReg());
839:       Inst.addOperand(MCOperand::createReg(ConstReg));
840:       outputMCInst(Inst);
```
- EN: This range implements operational logic in helpers such as getOperand, computeFPFastMathDefaultInfoVecIndex, setOpcode, getGlobalObjReg, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、computeFPFastMathDefaultInfoVecIndex、setOpcode、getGlobalObjReg 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 841-900
```cpp
841:     }
842:   }
843: }
844:
845: void SPIRVAsmPrinter::outputModuleSections() {
846:   const Module *M = MMI->getModule();
847:   // Get the global subtarget to output module-level info.
848:   ST = static_cast<const SPIRVTargetMachine &>(TM).getSubtargetImpl();
849:   TII = ST->getInstrInfo();
850:   MAI = &getAnalysis<SPIRVModuleAnalysis>().MAI;
851:   assert(ST && TII && MAI && M && "Module analysis is required");
852:
853:   // Let the NSDI handler add its extension and ext inst import entry to MAI
854:   // before the module header sections are emitted.
855:   if (NSDebugHandler)
856:     NSDebugHandler->prepareModuleOutput(*ST, *MAI);
857:
858:   // Output instructions according to the Logical Layout of a Module:
859:   // 1,2. All OpCapability instructions, then optional OpExtension
860:   // instructions.
861:   outputGlobalRequirements();
862:   // 3. Optional OpExtInstImport instructions.
863:   outputOpExtInstImports(*M);
864:   // 4. The single required OpMemoryModel instruction.
865:   outputOpMemoryModel();
866:   // 5. All entry point declarations, using OpEntryPoint.
867:   outputEntryPoints();
868:   // 6. Execution-mode declarations, using OpExecutionMode or
869:   // OpExecutionModeId.
870:   outputExecutionMode(*M);
871:   // 7a. Debug: all OpString, OpSourceExtension, OpSource, and
872:   // OpSourceContinued, without forward references.
873:   outputDebugSourceAndStrings(*M);
874:   // 7b. Debug: all OpName and all OpMemberName.
875:   outputModuleSection(SPIRV::MB_DebugNames);
876:   // 7c. Debug: all OpModuleProcessed instructions.
877:   outputModuleSection(SPIRV::MB_DebugModuleProcessed);
878:   // xxx. SPV_INTEL_memory_access_aliasing instructions go before 8.
879:   // "All annotation instructions"
880:   outputModuleSection(SPIRV::MB_AliasingInsts);
881:   // 8. All annotation instructions (all decorations).
882:   outputAnnotations(*M);
883:   // 9. All type declarations (OpTypeXXX instructions), all constant
884:   // instructions, and all global variable declarations. This section is
885:   // the first section to allow use of: OpLine and OpNoLine debug information;
886:   // non-semantic instructions with OpExtInst.
887:   outputModuleSection(SPIRV::MB_TypeConstVars);
888:   // 10. All global NonSemantic.Shader.DebugInfo.100 instructions. The
889:   // SPIRVNonSemanticDebugHandler emits these directly as MCInsts; the
890:   // MB_NonSemanticGlobalDI section in MAI is intentionally left empty.
891:   if (NSDebugHandler)
892:     NSDebugHandler->emitNonSemanticGlobalDebugInfo(*MAI);
893:   // 11. All function declarations (functions without a body).
894:   outputExtFuncDecls();
895:   // 12. All function definitions (functions with a body).
896:   // This is done in regular function output.
897: }
898:
899: bool SPIRVAsmPrinter::doInitialization(Module &M) {
900:   ModuleSectionsEmitted = false;
```
- EN: This range implements operational logic in helpers such as SPIRVAsmPrinter::outputModuleSections, getModule, getSubtargetImpl, getInstrInfo, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVAsmPrinter::outputModuleSections、getModule、getSubtargetImpl、getInstrInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-923
```cpp
901:   // Register the NSDI handler before calling the base class so that
902:   // AsmPrinter::doInitialization() calls Handler->beginModule(M) for it.
903:   if (M.getNamedMetadata("llvm.dbg.cu")) {
904:     auto Handler = std::make_unique<SPIRVNonSemanticDebugHandler>(*this);
905:     NSDebugHandler = Handler.get();
906:     addAsmPrinterHandler(std::move(Handler));
907:   }
908:   // We need to call the parent's one explicitly.
909:   return AsmPrinter::doInitialization(M);
910: }
911:
912: char SPIRVAsmPrinter::ID = 0;
913:
914: INITIALIZE_PASS(SPIRVAsmPrinter, "spirv-asm-printer", "SPIRV Assembly Printer",
915:                 false, false)
916:
917: // Force static initialization.
918: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
919: LLVMInitializeSPIRVAsmPrinter() {
920:   RegisterAsmPrinter<SPIRVAsmPrinter> X(getTheSPIRV32Target());
921:   RegisterAsmPrinter<SPIRVAsmPrinter> Y(getTheSPIRV64Target());
922:   RegisterAsmPrinter<SPIRVAsmPrinter> Z(getTheSPIRVLogicalTarget());
923: }
```
- EN: This range implements operational logic in helpers such as get, addAsmPrinterHandler, AsmPrinter::doInitialization, LLVMInitializeSPIRVAsmPrinter, translating backend policy into executable code.
- CN: 这一段实现了 get、addAsmPrinterHandler、AsmPrinter::doInitialization、LLVMInitializeSPIRVAsmPrinter 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: The asm printer layer serializes machine-level state into target assembly or object-oriented output.
  - CN: 汇编打印层负责把机器级状态序列化为目标汇编或面向目标的输出。
- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include SPIRVAsmPrinter, ST, getPassName, printOperand, outputMCInst, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 SPIRVAsmPrinter, ST, getPassName, printOperand, outputMCInst，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `MCTargetDesc/SPIRVInstPrinter.h`
  - `SPIRV.h`
  - `SPIRVInstrInfo.h`
  - `SPIRVMCInstLower.h`
  - `SPIRVModuleAnalysis.h`
  - `SPIRVNonSemanticDebugHandler.h`
  - `SPIRVSubtarget.h`
  - `SPIRVTargetMachine.h`
  - `SPIRVUtils.h`
  - `TargetInfo/SPIRVTargetInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/DenseMap.h`
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/CodeGen/AsmPrinter.h`
  - `llvm/CodeGen/MachineConstantPool.h`
  - `llvm/CodeGen/MachineInstr.h`
  - `llvm/CodeGen/MachineModuleInfo.h`
  - `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
  - `llvm/MC/MCAsmInfo.h`
  - `llvm/MC/MCAssembler.h`
  - `llvm/MC/MCInst.h`
  - `llvm/MC/MCObjectStreamer.h`
  - `llvm/MC/MCSPIRVObjectWriter.h`
  - `llvm/MC/MCStreamer.h`
  - `llvm/MC/MCSymbol.h`
  - `llvm/MC/TargetRegistry.h`
  - `llvm/Support/Compiler.h`
