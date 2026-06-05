# NVPTXAsmPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXAsmPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file contains a printer that converts from our internal representation of machine-dependent LLVM code to NVPTX assembly language.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===-- NVPTXAsmPrinter.cpp - NVPTX LLVM assembly writer ------------------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file contains a printer that converts from our internal representation
10: // of machine-dependent LLVM code to NVPTX assembly language.
11: //
12: //===----------------------------------------------------------------------===//
13:
14: #include "NVPTXAsmPrinter.h"
15: #include "MCTargetDesc/NVPTXBaseInfo.h"
16: #include "MCTargetDesc/NVPTXInstPrinter.h"
17: #include "MCTargetDesc/NVPTXMCAsmInfo.h"
18: #include "MCTargetDesc/NVPTXTargetStreamer.h"
19: #include "NVPTX.h"
20: #include "NVPTXDwarfDebug.h"
21: #include "NVPTXMCExpr.h"
22: #include "NVPTXMachineFunctionInfo.h"
23: #include "NVPTXRegisterInfo.h"
24: #include "NVPTXSubtarget.h"
25: #include "NVPTXTargetMachine.h"
26: #include "NVPTXUtilities.h"
27: #include "NVVMProperties.h"
28: #include "TargetInfo/NVPTXTargetInfo.h"
29: #include "cl_common_defines.h"
30: #include "llvm/ADT/APFloat.h"
31: #include "llvm/ADT/APInt.h"
32: #include "llvm/ADT/ArrayRef.h"
33: #include "llvm/ADT/DenseMap.h"
34: #include "llvm/ADT/DenseSet.h"
35: #include "llvm/ADT/SmallString.h"
36: #include "llvm/ADT/SmallVector.h"
37: #include "llvm/ADT/StringExtras.h"
38: #include "llvm/ADT/StringRef.h"
39: #include "llvm/ADT/Twine.h"
40: #include "llvm/ADT/iterator_range.h"
41: #include "llvm/Analysis/ConstantFolding.h"
42: #include "llvm/CodeGen/Analysis.h"
43: #include "llvm/CodeGen/MachineBasicBlock.h"
44: #include "llvm/CodeGen/MachineFrameInfo.h"
45: #include "llvm/CodeGen/MachineFunction.h"
46: #include "llvm/CodeGen/MachineInstr.h"
47: #include "llvm/CodeGen/MachineLoopInfo.h"
48: #include "llvm/CodeGen/MachineModuleInfo.h"
49: #include "llvm/CodeGen/MachineOperand.h"
50: #include "llvm/CodeGen/MachineRegisterInfo.h"
51: #include "llvm/CodeGen/TargetRegisterInfo.h"
52: #include "llvm/CodeGen/ValueTypes.h"
53: #include "llvm/CodeGenTypes/MachineValueType.h"
54: #include "llvm/IR/Argument.h"
55: #include "llvm/IR/Attributes.h"
56: #include "llvm/IR/BasicBlock.h"
57: #include "llvm/IR/Constant.h"
58: #include "llvm/IR/Constants.h"
59: #include "llvm/IR/DataLayout.h"
60: #include "llvm/IR/DebugInfo.h"
61: #include "llvm/IR/DebugInfoMetadata.h"
62: #include "llvm/IR/DebugLoc.h"
63: #include "llvm/IR/DerivedTypes.h"
64: #include "llvm/IR/Function.h"
65: #include "llvm/IR/GlobalAlias.h"
66: #include "llvm/IR/GlobalValue.h"
67: #include "llvm/IR/GlobalVariable.h"
68: #include "llvm/IR/Instruction.h"
69: #include "llvm/IR/LLVMContext.h"
70: #include "llvm/IR/Module.h"
71: #include "llvm/IR/Operator.h"
72: #include "llvm/IR/Type.h"
73: #include "llvm/IR/User.h"
74: #include "llvm/MC/MCExpr.h"
75: #include "llvm/MC/MCInst.h"
76: #include "llvm/MC/MCInstrDesc.h"
77: #include "llvm/MC/MCStreamer.h"
78: #include "llvm/MC/MCSymbol.h"
79: #include "llvm/MC/TargetRegistry.h"
80: #include "llvm/Support/Alignment.h"
81: #include "llvm/Support/Casting.h"
82: #include "llvm/Support/Compiler.h"
83: #include "llvm/Support/Endian.h"
84: #include "llvm/Support/ErrorHandling.h"
85: #include "llvm/Support/NativeFormatting.h"
86: #include "llvm/Support/raw_ostream.h"
87: #include "llvm/Target/TargetLoweringObjectFile.h"
88: #include "llvm/Target/TargetMachine.h"
89: #include "llvm/Transforms/Utils/UnrollLoop.h"
90: #include <cassert>
```
- EN: This range pulls in local backend headers and core LLVM infrastructure needed by the rest of the file.
- CN: 这一段引入本地后端头文件以及后续实现所依赖的 LLVM 基础设施。

### Lines 91-180
```cpp
 91: #include <cstdint>
 92: #include <cstring>
 93: #include <string>
 94:
 95: using namespace llvm;
 96:
 97: #define DEPOTNAME "__local_depot"
 98:
 99: static StringRef getTextureName(const Value &V) {
100:   assert(V.hasName() && "Found texture variable with no name");
101:   return V.getName();
102: }
103:
104: static StringRef getSurfaceName(const Value &V) {
105:   assert(V.hasName() && "Found surface variable with no name");
106:   return V.getName();
107: }
108:
109: static StringRef getSamplerName(const Value &V) {
110:   assert(V.hasName() && "Found sampler variable with no name");
111:   return V.getName();
112: }
113:
114: /// Emits initial debug location directive.
115: static void emitInitialRawDwarfLocDirective(const MachineFunction &MF,
116:                                             DwarfDebug *DD,
117:                                             MCStreamer &OutStreamer) {
118:   if (!DD)
119:     return;
120:
121:   assert(OutStreamer.hasRawTextSupport() && "Expected assembly output mode.");
122:   // This is NVPTX specific and it's unclear why.
123:   // PR51079: If we have code without debug information we need to give up.
124:   const DISubprogram *SP = MF.getFunction().getSubprogram();
125:   if (!SP)
126:     return;
127:   assert(SP->getUnit());
128:   // NoDebug and DebugDirectivesOnly do not require emitting the initial loc
129:   // directive. NoDebug does not require any debug directives and the initial
130:   // loc directive is not needed for DebugDirectivesOnly as it is redundant
131:   // assuming this is a non-empty function.
132:   if (SP->getUnit()->isDebugDirectivesOnly() || SP->getUnit()->isNoDebug())
133:     return;
134:
135:   (void)DD->emitInitialLocDirective(MF, /*CUID=*/0);
136: }
137:
138: /// discoverDependentGlobals - Return a set of GlobalVariables on which \p V
139: /// depends.
140: static void
141: discoverDependentGlobals(const Value *V,
142:                          DenseSet<const GlobalVariable *> &Globals) {
143:   if (const GlobalVariable *GV = dyn_cast<GlobalVariable>(V)) {
144:     Globals.insert(GV);
145:     return;
146:   }
147:
148:   if (const User *U = dyn_cast<User>(V))
149:     for (const auto &O : U->operands())
150:       discoverDependentGlobals(O, Globals);
151: }
152:
153: /// VisitGlobalVariableForEmission - Add \p GV to the list of GlobalVariable
154: /// instances to be emitted, but only after any dependents have been added
155: /// first.s
156: static void
157: VisitGlobalVariableForEmission(const GlobalVariable *GV,
158:                                SmallVectorImpl<const GlobalVariable *> &Order,
159:                                DenseSet<const GlobalVariable *> &Visited,
160:                                DenseSet<const GlobalVariable *> &Visiting) {
161:   // Have we already visited this one?
162:   if (Visited.count(GV))
163:     return;
164:
165:   // Do we have a circular dependency?
166:   if (!Visiting.insert(GV).second)
167:     report_fatal_error("Circular dependency found in global variable set");
168:
169:   // Make sure we visit all dependents first
170:   DenseSet<const GlobalVariable *> Others;
171:   for (const auto &O : GV->operands())
172:     discoverDependentGlobals(O, Others);
173:
174:   for (const GlobalVariable *GV : Others)
175:     VisitGlobalVariableForEmission(GV, Order, Visited, Visiting);
176:
177:   // Now we can visit ourself
178:   Order.push_back(GV);
179:   Visited.insert(GV);
180:   Visiting.erase(GV);
```
- EN: This range implements operational logic in helpers such as getTextureName, assert, getName, getSurfaceName, translating backend policy into executable code.
- CN: 这一段实现了 getTextureName、assert、getName、getSurfaceName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181: }
182:
183: void NVPTXAsmPrinter::emitInstruction(const MachineInstr *MI) {
184:   NVPTX_MC::verifyInstructionPredicates(MI->getOpcode(),
185:                                         getSubtargetInfo().getFeatureBits());
186:
187:   MCInst Inst;
188:   lowerToMCInst(MI, Inst);
189:   EmitToStreamer(*OutStreamer, Inst);
190: }
191:
192: void NVPTXAsmPrinter::lowerToMCInst(const MachineInstr *MI, MCInst &OutMI) {
193:   OutMI.setOpcode(MI->getOpcode());
194:   // Special: Do not mangle symbol operand of CALL_PROTOTYPE
195:   if (MI->getOpcode() == NVPTX::CALL_PROTOTYPE) {
196:     const MachineOperand &MO = MI->getOperand(0);
197:     OutMI.addOperand(GetSymbolRef(
198:       OutContext.getOrCreateSymbol(Twine(MO.getSymbolName()))));
199:     return;
200:   }
201:
202:   for (const auto MO : MI->operands())
203:     OutMI.addOperand(lowerOperand(MO));
204: }
205:
206: MCOperand NVPTXAsmPrinter::lowerOperand(const MachineOperand &MO) {
207:   switch (MO.getType()) {
208:   default:
209:     llvm_unreachable("unknown operand type");
210:   case MachineOperand::MO_Register:
211:     return MCOperand::createReg(encodeVirtualRegister(MO.getReg()));
212:   case MachineOperand::MO_Immediate:
213:     return MCOperand::createImm(MO.getImm());
214:   case MachineOperand::MO_MachineBasicBlock:
215:     return MCOperand::createExpr(
216:         MCSymbolRefExpr::create(MO.getMBB()->getSymbol(), OutContext));
217:   case MachineOperand::MO_ExternalSymbol:
218:     return GetSymbolRef(GetExternalSymbolSymbol(MO.getSymbolName()));
219:   case MachineOperand::MO_GlobalAddress:
220:     return GetSymbolRef(getSymbol(MO.getGlobal()));
221:   case MachineOperand::MO_FPImmediate: {
222:     const ConstantFP *Cnt = MO.getFPImm();
223:     const APFloat &Val = Cnt->getValueAPF();
224:
225:     switch (Cnt->getType()->getTypeID()) {
226:     default:
227:       report_fatal_error("Unsupported FP type");
228:       break;
229:     case Type::HalfTyID:
230:       return MCOperand::createExpr(
231:           NVPTXFloatMCExpr::createConstantFPHalf(Val, OutContext));
232:     case Type::BFloatTyID:
233:       return MCOperand::createExpr(
234:           NVPTXFloatMCExpr::createConstantBFPHalf(Val, OutContext));
235:     case Type::FloatTyID:
236:       return MCOperand::createExpr(
237:           NVPTXFloatMCExpr::createConstantFPSingle(Val, OutContext));
238:     case Type::DoubleTyID:
239:       return MCOperand::createExpr(
240:           NVPTXFloatMCExpr::createConstantFPDouble(Val, OutContext));
241:     }
242:     break;
243:   }
244:   }
245: }
246:
247: unsigned NVPTXAsmPrinter::encodeVirtualRegister(unsigned Reg) {
248:   if (Register::isVirtualRegister(Reg)) {
249:     const TargetRegisterClass *RC = MRI->getRegClass(Reg);
250:
251:     DenseMap<unsigned, unsigned> &RegMap = VRegMapping[RC];
252:     unsigned RegNum = RegMap[Reg];
253:
254:     // Encode the register class in the upper 4 bits
255:     // Must be kept in sync with NVPTXInstPrinter::printRegName
256:     unsigned Ret = 0;
257:     if (RC == &NVPTX::B1RegClass) {
258:       Ret = (1 << 28);
259:     } else if (RC == &NVPTX::B16RegClass) {
260:       Ret = (2 << 28);
261:     } else if (RC == &NVPTX::B32RegClass) {
262:       Ret = (3 << 28);
263:     } else if (RC == &NVPTX::B64RegClass) {
264:       Ret = (4 << 28);
265:     } else if (RC == &NVPTX::B128RegClass) {
266:       Ret = (7 << 28);
267:     } else {
268:       report_fatal_error("Bad register class");
269:     }
270:
```
- EN: This range defines or declares important types such as NVPTXAsmPrinter::emitInstruction, getSubtargetInfo, lowerToMCInst, EmitToStreamer, shaping the data model used by NVPTXAsmPrinter.cpp.
- CN: 这一段定义或声明了 NVPTXAsmPrinter::emitInstruction、getSubtargetInfo、lowerToMCInst、EmitToStreamer 等关键类型，构成 NVPTXAsmPrinter.cpp 使用的数据模型。

### Lines 271-360
```cpp
271:     // Insert the vreg number
272:     Ret |= (RegNum & 0x0FFFFFFF);
273:     return Ret;
274:   } else {
275:     // Some special-use registers are actually physical registers.
276:     // Encode this as the register class ID of 0 and the real register ID.
277:     return Reg & 0x0FFFFFFF;
278:   }
279: }
280:
281: MCOperand NVPTXAsmPrinter::GetSymbolRef(const MCSymbol *Symbol) {
282:   const MCExpr *Expr;
283:   Expr = MCSymbolRefExpr::create(Symbol, OutContext);
284:   return MCOperand::createExpr(Expr);
285: }
286:
287: void NVPTXAsmPrinter::printReturnValStr(const Function *F, raw_ostream &O) {
288:   const DataLayout &DL = getDataLayout();
289:   const NVPTXSubtarget &STI = TM.getSubtarget<NVPTXSubtarget>(*F);
290:   const auto *TLI = cast<NVPTXTargetLowering>(STI.getTargetLowering());
291:
292:   Type *Ty = F->getReturnType();
293:   if (Ty->getTypeID() == Type::VoidTyID)
294:     return;
295:   O << " (";
296:
297:   auto PrintScalarRetVal = [&](unsigned Size) {
298:     O << ".param .b" << promoteScalarArgumentSize(Size) << " func_retval0";
299:   };
300:   if (shouldPassAsArray(Ty)) {
301:     const unsigned TotalSize = DL.getTypeAllocSize(Ty);
302:     const Align RetAlignment =
303:         getFunctionArgumentAlignment(F, Ty, AttributeList::ReturnIndex, DL);
304:     O << ".param .align " << RetAlignment.value() << " .b8 func_retval0["
305:       << TotalSize << "]";
306:   } else if (Ty->isFloatingPointTy()) {
307:     PrintScalarRetVal(Ty->getPrimitiveSizeInBits());
308:   } else if (auto *ITy = dyn_cast<IntegerType>(Ty)) {
309:     PrintScalarRetVal(ITy->getBitWidth());
310:   } else if (isa<PointerType>(Ty)) {
311:     PrintScalarRetVal(TLI->getPointerTy(DL).getSizeInBits());
312:   } else
313:     llvm_unreachable("Unknown return type");
314:   O << ") ";
315: }
316:
317: void NVPTXAsmPrinter::printReturnValStr(const MachineFunction &MF,
318:                                         raw_ostream &O) {
319:   const Function &F = MF.getFunction();
320:   printReturnValStr(&F, O);
321: }
322:
323: // Return true if MBB is the header of a loop marked with
324: // llvm.loop.unroll.disable or llvm.loop.unroll.count=1.
325: bool NVPTXAsmPrinter::isLoopHeaderOfNoUnroll(
326:     const MachineBasicBlock &MBB) const {
327:   MachineLoopInfo &LI = getAnalysis<MachineLoopInfoWrapperPass>().getLI();
328:   // We insert .pragma "nounroll" only to the loop header.
329:   if (!LI.isLoopHeader(&MBB))
330:     return false;
331:
332:   // llvm.loop.unroll.disable is marked on the back edges of a loop. Therefore,
333:   // we iterate through each back edge of the loop with header MBB, and check
334:   // whether its metadata contains llvm.loop.unroll.disable.
335:   for (const MachineBasicBlock *PMBB : MBB.predecessors()) {
336:     if (LI.getLoopFor(PMBB) != LI.getLoopFor(&MBB)) {
337:       // Edges from other loops to MBB are not back edges.
338:       continue;
339:     }
340:     if (const BasicBlock *PBB = PMBB->getBasicBlock()) {
341:       if (MDNode *LoopID =
342:               PBB->getTerminator()->getMetadata(LLVMContext::MD_loop)) {
343:         if (GetUnrollMetadata(LoopID, "llvm.loop.unroll.disable"))
344:           return true;
345:         if (MDNode *UnrollCountMD =
346:                 GetUnrollMetadata(LoopID, "llvm.loop.unroll.count")) {
347:           if (mdconst::extract<ConstantInt>(UnrollCountMD->getOperand(1))
348:                   ->isOne())
349:             return true;
350:         }
351:       }
352:     }
353:   }
354:   return false;
355: }
356:
357: void NVPTXAsmPrinter::emitBasicBlockStart(const MachineBasicBlock &MBB) {
358:   AsmPrinter::emitBasicBlockStart(MBB);
359:   if (isLoopHeaderOfNoUnroll(MBB))
360:     OutStreamer->emitRawText(StringRef("\t.pragma \"nounroll\";\n"));
```
- EN: This range implements operational logic in helpers such as NVPTXAsmPrinter::GetSymbolRef, MCSymbolRefExpr::create, MCOperand::createExpr, NVPTXAsmPrinter::printReturnValStr, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAsmPrinter::GetSymbolRef、MCSymbolRefExpr::create、MCOperand::createExpr、NVPTXAsmPrinter::printReturnValStr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361: }
362:
363: void NVPTXAsmPrinter::emitFunctionEntryLabel() {
364:   SmallString<128> Str;
365:   raw_svector_ostream O(Str);
366:
367:   if (!GlobalsEmitted) {
368:     emitGlobals(*MF->getFunction().getParent());
369:     GlobalsEmitted = true;
370:   }
371:
372:   // Set up
373:   MRI = &MF->getRegInfo();
374:   F = &MF->getFunction();
375:   emitLinkageDirective(F, O);
376:   if (isKernelFunction(*F))
377:     O << ".entry ";
378:   else {
379:     O << ".func ";
380:     printReturnValStr(*MF, O);
381:   }
382:
383:   CurrentFnSym->print(O, MAI);
384:
385:   emitFunctionParamList(F, O);
386:   O << "\n";
387:
388:   if (isKernelFunction(*F))
389:     emitKernelFunctionDirectives(*F, O);
390:
391:   if (shouldEmitPTXNoReturn(F, TM))
392:     O << ".noreturn";
393:
394:   OutStreamer->emitRawText(O.str());
395:
396:   VRegMapping.clear();
397:   // Emit open brace for function body.
398:   OutStreamer->emitRawText(StringRef("{\n"));
399:   setAndEmitFunctionVirtualRegisters(*MF);
400:   encodeDebugInfoRegisterNumbers(*MF);
401:   // Emit initial .loc debug directive for correct relocation symbol data.
402:   emitInitialRawDwarfLocDirective(*MF, getDwarfDebug(), *OutStreamer);
403: }
404:
405: bool NVPTXAsmPrinter::runOnMachineFunction(MachineFunction &F) {
406:   bool Result = AsmPrinter::runOnMachineFunction(F);
407:   // Emit closing brace for the body of function F.
408:   // The closing brace must be emitted here because we need to emit additional
409:   // debug labels/data after the last basic block.
410:   // We need to emit the closing brace here because we don't have function that
411:   // finished emission of the function body.
412:   OutStreamer->emitRawText(StringRef("}\n"));
413:   return Result;
414: }
415:
416: void NVPTXAsmPrinter::emitFunctionBodyStart() {
417:   SmallString<128> Str;
418:   raw_svector_ostream O(Str);
419:   emitDemotedVars(&MF->getFunction(), O);
420:   OutStreamer->emitRawText(O.str());
421: }
422:
423: void NVPTXAsmPrinter::emitFunctionBodyEnd() {
424:   VRegMapping.clear();
425: }
426:
427: const MCSymbol *NVPTXAsmPrinter::getFunctionFrameSymbol() const {
428:     SmallString<128> Str;
429:     raw_svector_ostream(Str) << DEPOTNAME << getFunctionNumber();
430:     return OutContext.getOrCreateSymbol(Str);
431: }
432:
433: void NVPTXAsmPrinter::emitImplicitDef(const MachineInstr *MI) const {
434:   Register RegNo = MI->getOperand(0).getReg();
435:   if (RegNo.isVirtual()) {
436:     OutStreamer->AddComment(Twine("implicit-def: ") +
437:                             getVirtualRegisterName(RegNo));
438:   } else {
439:     const NVPTXSubtarget &STI = MI->getMF()->getSubtarget<NVPTXSubtarget>();
440:     OutStreamer->AddComment(Twine("implicit-def: ") +
441:                             STI.getRegisterInfo()->getName(RegNo));
442:   }
443:   OutStreamer->addBlankLine();
444: }
445:
446: void NVPTXAsmPrinter::emitKernelFunctionDirectives(const Function &F,
447:                                                    raw_ostream &O) const {
448:   // If the NVVM IR has some of reqntid* specified, then output
449:   // the reqntid directive, and set the unspecified ones to 1.
450:   // If none of Reqntid* is specified, don't output reqntid directive.
```
- EN: This range implements operational logic in helpers such as NVPTXAsmPrinter::emitFunctionEntryLabel, O, emitGlobals, getRegInfo, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAsmPrinter::emitFunctionEntryLabel、O、emitGlobals、getRegInfo 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:   const auto ReqNTID = getReqNTID(F);
452:   if (!ReqNTID.empty())
453:     O << formatv(".reqntid {0:$[, ]}\n",
454:                  make_range(ReqNTID.begin(), ReqNTID.end()));
455:
456:   const auto MaxNTID = getMaxNTID(F);
457:   if (!MaxNTID.empty())
458:     O << formatv(".maxntid {0:$[, ]}\n",
459:                  make_range(MaxNTID.begin(), MaxNTID.end()));
460:
461:   if (const auto Mincta = getMinCTASm(F))
462:     O << ".minnctapersm " << *Mincta << "\n";
463:
464:   if (const auto Maxnreg = getMaxNReg(F))
465:     O << ".maxnreg " << *Maxnreg << "\n";
466:
467:   // .maxclusterrank directive requires SM_90 or higher, make sure that we
468:   // filter it out for lower SM versions, as it causes a hard ptxas crash.
469:   const NVPTXTargetMachine &NTM = static_cast<const NVPTXTargetMachine &>(TM);
470:   const NVPTXSubtarget *STI = &NTM.getSubtarget<NVPTXSubtarget>(F);
471:
472:   if (STI->getSmVersion() >= 90) {
473:     const auto ClusterDim = getClusterDim(F);
474:     const bool BlocksAreClusters = hasBlocksAreClusters(F);
475:
476:     if (!ClusterDim.empty()) {
477:
478:       if (!BlocksAreClusters)
479:         O << ".explicitcluster\n";
480:
481:       if (ClusterDim[0] != 0) {
482:         assert(llvm::all_of(ClusterDim, not_equal_to(0)) &&
483:                "cluster_dim_x != 0 implies cluster_dim_y and cluster_dim_z "
484:                "should be non-zero as well");
485:
486:         O << formatv(".reqnctapercluster {0:$[, ]}\n",
487:                      make_range(ClusterDim.begin(), ClusterDim.end()));
488:       } else {
489:         assert(llvm::all_of(ClusterDim, equal_to(0)) &&
490:                "cluster_dim_x == 0 implies cluster_dim_y and cluster_dim_z "
491:                "should be 0 as well");
492:       }
493:     }
494:
495:     if (BlocksAreClusters) {
496:       LLVMContext &Ctx = F.getContext();
497:       if (ReqNTID.empty() || ClusterDim.empty())
498:         Ctx.diagnose(DiagnosticInfoUnsupported(
499:             F, "blocksareclusters requires reqntid and cluster_dim attributes",
500:             F.getSubprogram()));
501:       else if (STI->getPTXVersion() < 90)
502:         Ctx.diagnose(DiagnosticInfoUnsupported(
503:             F, "blocksareclusters requires PTX version >= 9.0",
504:             F.getSubprogram()));
505:       else
506:         O << ".blocksareclusters\n";
507:     }
508:
509:     if (const auto Maxclusterrank = getMaxClusterRank(F))
510:       O << ".maxclusterrank " << *Maxclusterrank << "\n";
511:   }
512: }
513:
514: std::string NVPTXAsmPrinter::getVirtualRegisterName(unsigned Reg) const {
515:   const TargetRegisterClass *RC = MRI->getRegClass(Reg);
516:
517:   std::string Name;
518:   raw_string_ostream NameStr(Name);
519:
520:   VRegRCMap::const_iterator I = VRegMapping.find(RC);
521:   assert(I != VRegMapping.end() && "Bad register class");
522:   const DenseMap<unsigned, unsigned> &RegMap = I->second;
523:
524:   VRegMap::const_iterator VI = RegMap.find(Reg);
525:   assert(VI != RegMap.end() && "Bad virtual register");
526:   unsigned MappedVR = VI->second;
527:
528:   NameStr << getNVPTXRegClassStr(RC) << MappedVR;
529:
530:   return Name;
531: }
532:
533: void NVPTXAsmPrinter::emitVirtualRegister(unsigned int vr,
534:                                           raw_ostream &O) {
535:   O << getVirtualRegisterName(vr);
536: }
537:
538: void NVPTXAsmPrinter::emitAliasDeclaration(const GlobalAlias *GA,
539:                                            raw_ostream &O) {
540:   const Function *F = dyn_cast_or_null<Function>(GA->getAliaseeObject());
```
- EN: This range defines or declares important types such as getReqNTID, make_range, getMaxNTID, getClusterDim, shaping the data model used by NVPTXAsmPrinter.cpp.
- CN: 这一段定义或声明了 getReqNTID、make_range、getMaxNTID、getClusterDim 等关键类型，构成 NVPTXAsmPrinter.cpp 使用的数据模型。

### Lines 541-630
```cpp
541:   if (!F || isKernelFunction(*F) || F->isDeclaration())
542:     report_fatal_error(
543:         "NVPTX aliasee must be a non-kernel function definition");
544:
545:   if (GA->hasLinkOnceLinkage() || GA->hasWeakLinkage() ||
546:       GA->hasAvailableExternallyLinkage() || GA->hasCommonLinkage())
547:     report_fatal_error("NVPTX aliasee must not be '.weak'");
548:
549:   emitDeclarationWithName(F, getSymbol(GA), O);
550: }
551:
552: void NVPTXAsmPrinter::emitDeclaration(const Function *F, raw_ostream &O) {
553:   emitDeclarationWithName(F, getSymbol(F), O);
554: }
555:
556: void NVPTXAsmPrinter::emitDeclarationWithName(const Function *F, MCSymbol *S,
557:                                               raw_ostream &O) {
558:   emitLinkageDirective(F, O);
559:   if (isKernelFunction(*F))
560:     O << ".entry ";
561:   else
562:     O << ".func ";
563:   printReturnValStr(F, O);
564:   S->print(O, MAI);
565:   O << "\n";
566:   emitFunctionParamList(F, O);
567:   O << "\n";
568:   if (shouldEmitPTXNoReturn(F, TM))
569:     O << ".noreturn";
570:   O << ";\n";
571: }
572:
573: static bool usedInGlobalVarDef(const Constant *C) {
574:   if (!C)
575:     return false;
576:
577:   if (const GlobalVariable *GV = dyn_cast<GlobalVariable>(C))
578:     return GV->getName() != "llvm.used";
579:
580:   for (const User *U : C->users())
581:     if (const Constant *C = dyn_cast<Constant>(U))
582:       if (usedInGlobalVarDef(C))
583:         return true;
584:
585:   return false;
586: }
587:
588: static bool usedInOneFunc(const User *U, Function const *&OneFunc) {
589:   if (const GlobalVariable *OtherGV = dyn_cast<GlobalVariable>(U))
590:     if (OtherGV->getName() == "llvm.used")
591:       return true;
592:
593:   if (const Instruction *I = dyn_cast<Instruction>(U)) {
594:     if (const Function *CurFunc = I->getFunction()) {
595:       if (OneFunc && (CurFunc != OneFunc))
596:         return false;
597:       OneFunc = CurFunc;
598:       return true;
599:     }
600:     return false;
601:   }
602:
603:   for (const User *UU : U->users())
604:     if (!usedInOneFunc(UU, OneFunc))
605:       return false;
606:
607:   return true;
608: }
609:
610: /* Find out if a global variable can be demoted to local scope.
611:  * Currently, this is valid for CUDA shared variables, which have local
612:  * scope and global lifetime. So the conditions to check are :
613:  * 1. Is the global variable in shared address space?
614:  * 2. Does it have local linkage?
615:  * 3. Is the global variable referenced only in one function?
616:  */
617: static bool canDemoteGlobalVar(const GlobalVariable *GV, Function const *&f) {
618:   if (!GV->hasLocalLinkage())
619:     return false;
620:   if (GV->getAddressSpace() != ADDRESS_SPACE_SHARED)
621:     return false;
622:
623:   const Function *oneFunc = nullptr;
624:
625:   bool flag = usedInOneFunc(GV, oneFunc);
626:   if (!flag)
627:     return false;
628:   if (!oneFunc)
629:     return false;
630:   f = oneFunc;
```
- EN: This range implements operational logic in helpers such as hasAvailableExternallyLinkage, report_fatal_error, emitDeclarationWithName, NVPTXAsmPrinter::emitDeclaration, translating backend policy into executable code.
- CN: 这一段实现了 hasAvailableExternallyLinkage、report_fatal_error、emitDeclarationWithName、NVPTXAsmPrinter::emitDeclaration 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 631-720
```cpp
631:   return true;
632: }
633:
634: static bool useFuncSeen(const Constant *C,
635:                         const SmallPtrSetImpl<const Function *> &SeenSet) {
636:   for (const User *U : C->users()) {
637:     if (const Constant *cu = dyn_cast<Constant>(U)) {
638:       if (useFuncSeen(cu, SeenSet))
639:         return true;
640:     } else if (const Instruction *I = dyn_cast<Instruction>(U)) {
641:       if (const Function *Caller = I->getFunction())
642:         if (SeenSet.contains(Caller))
643:           return true;
644:     }
645:   }
646:   return false;
647: }
648:
649: void NVPTXAsmPrinter::emitDeclarations(const Module &M, raw_ostream &O) {
650:   SmallPtrSet<const Function *, 32> SeenSet;
651:   for (const Function &F : M) {
652:     if (F.getAttributes().hasFnAttr("nvptx-libcall-callee")) {
653:       emitDeclaration(&F, O);
654:       continue;
655:     }
656:
657:     if (F.isDeclaration()) {
658:       if (F.use_empty())
659:         continue;
660:       if (F.getIntrinsicID())
661:         continue;
662:       // An unrecognized intrinsic would produce an invalid PTX declaration. Let
663:       // the user know that, and skip it.
664:       if (F.isIntrinsic()) {
665:         LLVMContext &Ctx = F.getContext();
666:         Ctx.diagnose(DiagnosticInfoUnsupported(
667:             F, "unknown intrinsic '" + F.getName() +
668:                    "' cannot be lowered by the NVPTX backend"));
669:         continue;
670:       }
671:       emitDeclaration(&F, O);
672:       continue;
673:     }
674:     for (const User *U : F.users()) {
675:       if (const Constant *C = dyn_cast<Constant>(U)) {
676:         if (usedInGlobalVarDef(C)) {
677:           // The use is in the initialization of a global variable
678:           // that is a function pointer, so print a declaration
679:           // for the original function
680:           emitDeclaration(&F, O);
681:           break;
682:         }
683:         // Emit a declaration of this function if the function that
684:         // uses this constant expr has already been seen.
685:         if (useFuncSeen(C, SeenSet)) {
686:           emitDeclaration(&F, O);
687:           break;
688:         }
689:       }
690:
691:       if (!isa<Instruction>(U))
692:         continue;
693:       const Function *Caller = cast<Instruction>(U)->getFunction();
694:       if (!Caller)
695:         continue;
696:
697:       // If a caller has already been seen, then the caller is
698:       // appearing in the module before the callee. so print out
699:       // a declaration for the callee.
700:       if (SeenSet.contains(Caller)) {
701:         emitDeclaration(&F, O);
702:         break;
703:       }
704:     }
705:     SeenSet.insert(&F);
706:   }
707:   for (const GlobalAlias &GA : M.aliases())
708:     emitAliasDeclaration(&GA, O);
709: }
710:
711: void NVPTXAsmPrinter::emitStartOfAsmFile(Module &M) {
712:   // Construct a default subtarget off of the TargetMachine defaults. The
713:   // rest of NVPTX isn't friendly to change subtargets per function and
714:   // so the default TargetMachine will have all of the options.
715:   const NVPTXTargetMachine &NTM = static_cast<const NVPTXTargetMachine &>(TM);
716:   const NVPTXSubtarget *STI = NTM.getSubtargetImpl();
717:
718:   // Emit header before any dwarf directives are emitted below.
719:   emitHeader(M, *STI);
720: }
```
- EN: This range implements operational logic in helpers such as NVPTXAsmPrinter::emitDeclarations, emitDeclaration, getContext, getFunction, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAsmPrinter::emitDeclarations、emitDeclaration、getContext、getFunction 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:
722: /// Create NVPTX-specific DwarfDebug handler.
723: DwarfDebug *NVPTXAsmPrinter::createDwarfDebug() {
724:   return new NVPTXDwarfDebug(this);
725: }
726:
727: bool NVPTXAsmPrinter::doInitialization(Module &M) {
728:   const NVPTXTargetMachine &NTM = static_cast<const NVPTXTargetMachine &>(TM);
729:   const NVPTXSubtarget &STI = *NTM.getSubtargetImpl();
730:   if (M.alias_size() && (STI.getPTXVersion() < 63 || STI.getSmVersion() < 30))
731:     report_fatal_error(".alias requires PTX version >= 6.3 and sm_30");
732:
733:   // We need to call the parent's one explicitly.
734:   bool Result = AsmPrinter::doInitialization(M);
735:
736:   GlobalsEmitted = false;
737:
738:   return Result;
739: }
740:
741: void NVPTXAsmPrinter::emitGlobals(const Module &M) {
742:   SmallString<128> Str2;
743:   raw_svector_ostream OS2(Str2);
744:
745:   emitDeclarations(M, OS2);
746:
747:   // As ptxas does not support forward references of globals, we need to first
748:   // sort the list of module-level globals in def-use order. We visit each
749:   // global variable in order, and ensure that we emit it *after* its dependent
750:   // globals. We use a little extra memory maintaining both a set and a list to
751:   // have fast searches while maintaining a strict ordering.
752:   SmallVector<const GlobalVariable *, 8> Globals;
753:   DenseSet<const GlobalVariable *> GVVisited;
754:   DenseSet<const GlobalVariable *> GVVisiting;
755:
756:   // Visit each global variable, in order
757:   for (const GlobalVariable &I : M.globals())
758:     VisitGlobalVariableForEmission(&I, Globals, GVVisited, GVVisiting);
759:
760:   assert(GVVisited.size() == M.global_size() && "Missed a global variable");
761:   assert(GVVisiting.size() == 0 && "Did not fully process a global variable");
762:
763:   const NVPTXTargetMachine &NTM = static_cast<const NVPTXTargetMachine &>(TM);
764:   const NVPTXSubtarget &STI = *NTM.getSubtargetImpl();
765:
766:   // Print out module-level global variables in proper order
767:   for (const GlobalVariable *GV : Globals)
768:     printModuleLevelGV(GV, OS2, /*ProcessDemoted=*/false, STI);
769:
770:   OS2 << '\n';
771:
772:   OutStreamer->emitRawText(OS2.str());
773: }
774:
775: void NVPTXAsmPrinter::emitGlobalAlias(const Module &M, const GlobalAlias &GA) {
776:   SmallString<128> Str;
777:   raw_svector_ostream OS(Str);
778:
779:   MCSymbol *Name = getSymbol(&GA);
780:
781:   OS << ".alias " << Name->getName() << ", " << GA.getAliaseeObject()->getName()
782:      << ";\n";
783:
784:   OutStreamer->emitRawText(OS.str());
785: }
786:
787: NVPTXTargetStreamer *NVPTXAsmPrinter::getTargetStreamer() const {
788:   return static_cast<NVPTXTargetStreamer *>(OutStreamer->getTargetStreamer());
789: }
790:
791: static bool hasFullDebugInfo(Module &M) {
792:   for (DICompileUnit *CU : M.debug_compile_units()) {
793:     switch(CU->getEmissionKind()) {
794:     case DICompileUnit::NoDebug:
795:     case DICompileUnit::DebugDirectivesOnly:
796:       break;
797:     case DICompileUnit::LineTablesOnly:
798:     case DICompileUnit::FullDebug:
799:       return true;
800:     }
801:   }
802:
803:   return false;
804: }
805:
806: void NVPTXAsmPrinter::emitHeader(Module &M, const NVPTXSubtarget &STI) {
807:   auto *TS = getTargetStreamer();
808:
809:   TS->emitBanner();
810:
```
- EN: This range implements operational logic in helpers such as NVPTXAsmPrinter::createDwarfDebug, NVPTXDwarfDebug, NVPTXAsmPrinter::doInitialization, getSubtargetImpl, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXAsmPrinter::createDwarfDebug、NVPTXDwarfDebug、NVPTXAsmPrinter::doInitialization、getSubtargetImpl 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:   const unsigned PTXVersion = STI.getPTXVersion();
812:   TS->emitVersionDirective(PTXVersion);
813:
814:   const NVPTXTargetMachine &NTM = static_cast<const NVPTXTargetMachine &>(TM);
815:   bool TexModeIndependent = NTM.getDrvInterface() == NVPTX::NVCL;
816:
817:   TS->emitTargetDirective(STI.getTargetName(), TexModeIndependent,
818:                           hasFullDebugInfo(M));
819:   TS->emitAddressSizeDirective(M.getDataLayout().getPointerSizeInBits());
820: }
821:
822: bool NVPTXAsmPrinter::doFinalization(Module &M) {
823:   // If we did not emit any functions, then the global declarations have not
824:   // yet been emitted.
825:   if (!GlobalsEmitted) {
826:     emitGlobals(M);
827:     GlobalsEmitted = true;
828:   }
829:
830:   // call doFinalization
831:   bool ret = AsmPrinter::doFinalization(M);
832:
833:   clearAnnotationCache(&M);
834:
835:   auto *TS =
836:       static_cast<NVPTXTargetStreamer *>(OutStreamer->getTargetStreamer());
837:   // Close the last emitted section
838:   if (hasDebugInfo()) {
839:     TS->closeLastSection();
840:     // Emit empty .debug_macinfo section for better support of the empty files.
841:     OutStreamer->emitRawText("\t.section\t.debug_macinfo\t{\t}");
842:   }
843:
844:   // Output last DWARF .file directives, if any.
845:   TS->outputDwarfFileDirectives();
846:
847:   return ret;
848: }
849:
850: // This function emits appropriate linkage directives for
851: // functions and global variables.
852: //
853: // extern function declaration            -> .extern
854: // extern function definition             -> .visible
855: // external global variable with init     -> .visible
856: // external without init                  -> .extern
857: // appending                              -> not allowed, assert.
858: // for any linkage other than
859: // internal, private, linker_private,
860: // linker_private_weak, linker_private_weak_def_auto,
861: // we emit                                -> .weak.
862:
863: void NVPTXAsmPrinter::emitLinkageDirective(const GlobalValue *V,
864:                                            raw_ostream &O) {
865:   if (static_cast<NVPTXTargetMachine &>(TM).getDrvInterface() == NVPTX::CUDA) {
866:     if (V->hasExternalLinkage()) {
867:       if (const auto *GVar = dyn_cast<GlobalVariable>(V))
868:         O << (GVar->hasInitializer() ? ".visible " : ".extern ");
869:       else if (V->isDeclaration())
870:         O << ".extern ";
871:       else
872:         O << ".visible ";
873:     } else if (V->hasAppendingLinkage()) {
874:       report_fatal_error("Symbol '" + (V->hasName() ? V->getName() : "") +
875:                          "' has unsupported appending linkage type");
876:     } else if (!V->hasInternalLinkage() && !V->hasPrivateLinkage()) {
877:       O << ".weak ";
878:     }
879:   }
880: }
881:
882: void NVPTXAsmPrinter::printModuleLevelGV(const GlobalVariable *GVar,
883:                                          raw_ostream &O, bool ProcessDemoted,
884:                                          const NVPTXSubtarget &STI) {
885:   // Skip meta data
886:   if (GVar->hasSection())
887:     if (GVar->getSection() == "llvm.metadata")
888:       return;
889:
890:   // Skip LLVM intrinsic global variables
891:   if (GVar->getName().starts_with("llvm.") ||
892:       GVar->getName().starts_with("nvvm."))
893:     return;
894:
895:   const DataLayout &DL = getDataLayout();
896:
897:   // GlobalVariables are always constant pointers themselves.
898:   Type *ETy = GVar->getValueType();
899:
900:   if (GVar->hasExternalLinkage()) {
```
- EN: This range implements operational logic in helpers such as getPTXVersion, emitVersionDirective, hasFullDebugInfo, emitAddressSizeDirective, translating backend policy into executable code.
- CN: 这一段实现了 getPTXVersion、emitVersionDirective、hasFullDebugInfo、emitAddressSizeDirective 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:     if (GVar->hasInitializer())
902:       O << ".visible ";
903:     else
904:       O << ".extern ";
905:   } else if (STI.getPTXVersion() >= 50 && GVar->hasCommonLinkage() &&
906:              GVar->getAddressSpace() == ADDRESS_SPACE_GLOBAL) {
907:     O << ".common ";
908:   } else if (GVar->hasLinkOnceLinkage() || GVar->hasWeakLinkage() ||
909:              GVar->hasAvailableExternallyLinkage() ||
910:              GVar->hasCommonLinkage()) {
911:     O << ".weak ";
912:   }
913:
914:   const PTXOpaqueType OpaqueType = getPTXOpaqueType(*GVar);
915:
916:   if (OpaqueType == PTXOpaqueType::Texture) {
917:     O << ".global .texref " << getTextureName(*GVar) << ";\n";
918:     return;
919:   }
920:
921:   if (OpaqueType == PTXOpaqueType::Surface) {
922:     O << ".global .surfref " << getSurfaceName(*GVar) << ";\n";
923:     return;
924:   }
925:
926:   if (GVar->isDeclaration()) {
927:     // (extern) declarations, no definition or initializer
928:     // Currently the only known declaration is for an automatic __local
929:     // (.shared) promoted to global.
930:     emitPTXGlobalVariable(GVar, O, STI);
931:     O << ";\n";
932:     return;
933:   }
934:
935:   if (OpaqueType == PTXOpaqueType::Sampler) {
936:     O << ".global .samplerref " << getSamplerName(*GVar);
937:
938:     const Constant *Initializer = nullptr;
939:     if (GVar->hasInitializer())
940:       Initializer = GVar->getInitializer();
941:     const ConstantInt *CI = nullptr;
942:     if (Initializer)
943:       CI = dyn_cast<ConstantInt>(Initializer);
944:     if (CI) {
945:       unsigned sample = CI->getZExtValue();
946:
947:       O << " = { ";
948:
949:       for (int i = 0,
950:                addr = ((sample & __CLK_ADDRESS_MASK) >> __CLK_ADDRESS_BASE);
951:            i < 3; i++) {
952:         O << "addr_mode_" << i << " = ";
953:         switch (addr) {
954:         case 0:
955:           O << "wrap";
956:           break;
957:         case 1:
958:           O << "clamp_to_border";
959:           break;
960:         case 2:
961:           O << "clamp_to_edge";
962:           break;
963:         case 3:
964:           O << "wrap";
965:           break;
966:         case 4:
967:           O << "mirror";
968:           break;
969:         }
970:         O << ", ";
971:       }
972:       O << "filter_mode = ";
973:       switch ((sample & __CLK_FILTER_MASK) >> __CLK_FILTER_BASE) {
974:       case 0:
975:         O << "nearest";
976:         break;
977:       case 1:
978:         O << "linear";
979:         break;
980:       case 2:
981:         llvm_unreachable("Anisotropic filtering is not supported");
982:       default:
983:         O << "nearest";
984:         break;
985:       }
986:       if (!((sample & __CLK_NORMALIZED_MASK) >> __CLK_NORMALIZED_BASE)) {
987:         O << ", force_unnormalized_coords = 1";
988:       }
989:       O << " }";
990:     }
```
- EN: This range implements operational logic in helpers such as getAddressSpace, hasCommonLinkage, getPTXOpaqueType, emitPTXGlobalVariable, translating backend policy into executable code.
- CN: 这一段实现了 getAddressSpace、hasCommonLinkage、getPTXOpaqueType、emitPTXGlobalVariable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:
 992:     O << ";\n";
 993:     return;
 994:   }
 995:
 996:   if (GVar->hasPrivateLinkage()) {
 997:     if (GVar->getName().starts_with("unrollpragma"))
 998:       return;
 999:
1000:     // FIXME - need better way (e.g. Metadata) to avoid generating this global
1001:     if (GVar->getName().starts_with("filename"))
1002:       return;
1003:     if (GVar->use_empty())
1004:       return;
1005:   }
1006:
1007:   const Function *DemotedFunc = nullptr;
1008:   if (!ProcessDemoted && canDemoteGlobalVar(GVar, DemotedFunc)) {
1009:     O << "// " << GVar->getName() << " has been demoted\n";
1010:     localDecls[DemotedFunc].push_back(GVar);
1011:     return;
1012:   }
1013:
1014:   O << ".";
1015:   emitPTXAddressSpace(GVar->getAddressSpace(), O);
1016:
1017:   if (isManaged(*GVar)) {
1018:     if (STI.getPTXVersion() < 40 || STI.getSmVersion() < 30)
1019:       report_fatal_error(
1020:           ".attribute(.managed) requires PTX version >= 4.0 and sm_30");
1021:     O << " .attribute(.managed)";
1022:   }
1023:
1024:   O << " .align "
1025:     << GVar->getAlign().value_or(DL.getPrefTypeAlign(ETy)).value();
1026:
1027:   if (ETy->isPointerTy() || ((ETy->isIntegerTy() || ETy->isFloatingPointTy()) &&
1028:                              ETy->getScalarSizeInBits() <= 64)) {
1029:     O << " .";
1030:     // Special case: ABI requires that we use .u8 for predicates
1031:     if (ETy->isIntegerTy(1))
1032:       O << "u8";
1033:     else
1034:       O << getPTXFundamentalTypeStr(ETy, false);
1035:     O << " ";
1036:     getSymbol(GVar)->print(O, MAI);
1037:
1038:     // Ptx allows variable initilization only for constant and global state
1039:     // spaces.
1040:     if (GVar->hasInitializer()) {
1041:       if ((GVar->getAddressSpace() == ADDRESS_SPACE_GLOBAL) ||
1042:           (GVar->getAddressSpace() == ADDRESS_SPACE_CONST)) {
1043:         const Constant *Initializer = GVar->getInitializer();
1044:         // 'undef' is treated as there is no value specified.
1045:         if (!Initializer->isNullValue() && !isa<UndefValue>(Initializer)) {
1046:           O << " = ";
1047:           printScalarConstant(Initializer, O);
1048:         }
1049:       } else {
1050:         // The frontend adds zero-initializer to device and constant variables
1051:         // that don't have an initial value, and UndefValue to shared
1052:         // variables, so skip warning for this case.
1053:         if (!GVar->getInitializer()->isNullValue() &&
1054:             !isa<UndefValue>(GVar->getInitializer())) {
1055:           report_fatal_error("initial value of '" + GVar->getName() +
1056:                              "' is not allowed in addrspace(" +
1057:                              Twine(GVar->getAddressSpace()) + ")");
1058:         }
1059:       }
1060:     }
1061:   } else {
1062:     // Although PTX has direct support for struct type and array type and
1063:     // LLVM IR is very similar to PTX, the LLVM CodeGen does not support for
1064:     // targets that support these high level field accesses. Structs, arrays
1065:     // and vectors are lowered into arrays of bytes.
1066:     switch (ETy->getTypeID()) {
1067:     case Type::IntegerTyID: // Integers larger than 64 bits
1068:     case Type::FP128TyID:
1069:     case Type::StructTyID:
1070:     case Type::ArrayTyID:
1071:     case Type::FixedVectorTyID: {
1072:       const uint64_t ElementSize = DL.getTypeStoreSize(ETy);
1073:       // Ptx allows variable initilization only for constant and
1074:       // global state spaces.
1075:       if (((GVar->getAddressSpace() == ADDRESS_SPACE_GLOBAL) ||
1076:            (GVar->getAddressSpace() == ADDRESS_SPACE_CONST)) &&
1077:           GVar->hasInitializer()) {
1078:         const Constant *Initializer = GVar->getInitializer();
1079:         if (!isa<UndefValue>(Initializer) && !Initializer->isNullValue()) {
1080:           AggBuffer aggBuffer(ElementSize, *this);
```
- EN: This range implements operational logic in helpers such as push_back, emitPTXAddressSpace, attribute, getAlign, translating backend policy into executable code.
- CN: 这一段实现了 push_back、emitPTXAddressSpace、attribute、getAlign 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:           bufferAggregateConstant(Initializer, &aggBuffer);
1082:           if (aggBuffer.numSymbols()) {
1083:             const unsigned int ptrSize = MAI.getCodePointerSize();
1084:             if (ElementSize % ptrSize ||
1085:                 !aggBuffer.allSymbolsAligned(ptrSize)) {
1086:               // Print in bytes and use the mask() operator for pointers.
1087:               if (!STI.hasMaskOperator())
1088:                 report_fatal_error(
1089:                     "initialized packed aggregate with pointers '" +
1090:                     GVar->getName() +
1091:                     "' requires at least PTX ISA version 7.1");
1092:               O << " .u8 ";
1093:               getSymbol(GVar)->print(O, MAI);
1094:               O << "[" << ElementSize << "] = {";
1095:               aggBuffer.printBytes(O);
1096:               O << "}";
1097:             } else {
1098:               O << " .u" << ptrSize * 8 << " ";
1099:               getSymbol(GVar)->print(O, MAI);
1100:               O << "[" << ElementSize / ptrSize << "] = {";
1101:               aggBuffer.printWords(O);
1102:               O << "}";
1103:             }
1104:           } else {
1105:             O << " .b8 ";
1106:             getSymbol(GVar)->print(O, MAI);
1107:             O << "[" << ElementSize << "] = {";
1108:             aggBuffer.printBytes(O);
1109:             O << "}";
1110:           }
1111:         } else {
1112:           O << " .b8 ";
1113:           getSymbol(GVar)->print(O, MAI);
1114:           if (ElementSize)
1115:             O << "[" << ElementSize << "]";
1116:         }
1117:       } else {
1118:         O << " .b8 ";
1119:         getSymbol(GVar)->print(O, MAI);
1120:         if (ElementSize)
1121:           O << "[" << ElementSize << "]";
1122:       }
1123:       break;
1124:     }
1125:     default:
1126:       llvm_unreachable("type not supported yet");
1127:     }
1128:   }
1129:   O << ";\n";
1130: }
1131:
1132: void NVPTXAsmPrinter::AggBuffer::printSymbol(unsigned nSym, raw_ostream &os) {
1133:   const Value *v = Symbols[nSym];
1134:   const Value *v0 = SymbolsBeforeStripping[nSym];
1135:   if (const GlobalValue *GVar = dyn_cast<GlobalValue>(v)) {
1136:     MCSymbol *Name = AP.getSymbol(GVar);
1137:     PointerType *PTy = dyn_cast<PointerType>(v0->getType());
1138:     // Is v0 a generic pointer?
1139:     bool isGenericPointer = PTy && PTy->getAddressSpace() == 0;
1140:     if (EmitGeneric && isGenericPointer && !isa<Function>(v)) {
1141:       os << "generic(";
1142:       Name->print(os, AP.MAI);
1143:       os << ")";
1144:     } else {
1145:       Name->print(os, AP.MAI);
1146:     }
1147:   } else if (const ConstantExpr *CExpr = dyn_cast<ConstantExpr>(v0)) {
1148:     const MCExpr *Expr = AP.lowerConstantForGV(CExpr, false);
1149:     AP.printMCExpr(*Expr, os);
1150:   } else
1151:     llvm_unreachable("symbol type unknown");
1152: }
1153:
1154: void NVPTXAsmPrinter::AggBuffer::printBytes(raw_ostream &os) {
1155:   unsigned int ptrSize = AP.MAI.getCodePointerSize();
1156:   // Do not emit trailing zero initializers. They will be zero-initialized by
1157:   // ptxas. This saves on both space requirements for the generated PTX and on
1158:   // memory use by ptxas. (See:
1159:   // https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#global-state-space)
1160:   unsigned int InitializerCount = Size;
1161:   // TODO: symbols make this harder, but it would still be good to trim trailing
1162:   // 0s for aggs with symbols as well.
1163:   if (numSymbols() == 0)
1164:     while (InitializerCount >= 1 && !buffer[InitializerCount - 1])
1165:       InitializerCount--;
1166:
1167:   symbolPosInBuffer.push_back(InitializerCount);
1168:   unsigned int nSym = 0;
1169:   unsigned int nextSymbolPos = symbolPosInBuffer[nSym];
1170:   for (unsigned int pos = 0; pos < InitializerCount;) {
```
- EN: This range implements operational logic in helpers such as bufferAggregateConstant, getCodePointerSize, allSymbolsAligned, getSymbol, translating backend policy into executable code.
- CN: 这一段实现了 bufferAggregateConstant、getCodePointerSize、allSymbolsAligned、getSymbol 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1260
```cpp
1171:     if (pos)
1172:       os << ", ";
1173:     if (pos != nextSymbolPos) {
1174:       os << (unsigned int)buffer[pos];
1175:       ++pos;
1176:       continue;
1177:     }
1178:     // Generate a per-byte mask() operator for the symbol, which looks like:
1179:     //   .global .u8 addr[] = {0xFF(foo), 0xFF00(foo), 0xFF0000(foo), ...};
1180:     // See https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#initializers
1181:     std::string symText;
1182:     llvm::raw_string_ostream oss(symText);
1183:     printSymbol(nSym, oss);
1184:     for (unsigned i = 0; i < ptrSize; ++i) {
1185:       if (i)
1186:         os << ", ";
1187:       llvm::write_hex(os, 0xFFULL << i * 8, HexPrintStyle::PrefixUpper);
1188:       os << "(" << symText << ")";
1189:     }
1190:     pos += ptrSize;
1191:     nextSymbolPos = symbolPosInBuffer[++nSym];
1192:     assert(nextSymbolPos >= pos);
1193:   }
1194: }
1195:
1196: void NVPTXAsmPrinter::AggBuffer::printWords(raw_ostream &os) {
1197:   unsigned int ptrSize = AP.MAI.getCodePointerSize();
1198:   symbolPosInBuffer.push_back(Size);
1199:   unsigned int nSym = 0;
1200:   unsigned int nextSymbolPos = symbolPosInBuffer[nSym];
1201:   assert(nextSymbolPos % ptrSize == 0);
1202:   for (unsigned int pos = 0; pos < Size; pos += ptrSize) {
1203:     if (pos)
1204:       os << ", ";
1205:     if (pos == nextSymbolPos) {
1206:       printSymbol(nSym, os);
1207:       nextSymbolPos = symbolPosInBuffer[++nSym];
1208:       assert(nextSymbolPos % ptrSize == 0);
1209:       assert(nextSymbolPos >= pos + ptrSize);
1210:     } else if (ptrSize == 4)
1211:       os << support::endian::read32le(&buffer[pos]);
1212:     else
1213:       os << support::endian::read64le(&buffer[pos]);
1214:   }
1215: }
1216:
1217: void NVPTXAsmPrinter::emitDemotedVars(const Function *F, raw_ostream &O) {
1218:   auto It = localDecls.find(F);
1219:   if (It == localDecls.end())
1220:     return;
1221:
1222:   ArrayRef<const GlobalVariable *> GVars = It->second;
1223:
1224:   const NVPTXTargetMachine &NTM = static_cast<const NVPTXTargetMachine &>(TM);
1225:   const NVPTXSubtarget &STI = *NTM.getSubtargetImpl();
1226:
1227:   for (const GlobalVariable *GV : GVars) {
1228:     O << "\t// demoted variable\n\t";
1229:     printModuleLevelGV(GV, O, /*processDemoted=*/true, STI);
1230:   }
1231: }
1232:
1233: void NVPTXAsmPrinter::emitPTXAddressSpace(unsigned int AddressSpace,
1234:                                           raw_ostream &O) const {
1235:   switch (AddressSpace) {
1236:   case ADDRESS_SPACE_LOCAL:
1237:     O << "local";
1238:     break;
1239:   case ADDRESS_SPACE_GLOBAL:
1240:     O << "global";
1241:     break;
1242:   case ADDRESS_SPACE_CONST:
1243:     O << "const";
1244:     break;
1245:   case ADDRESS_SPACE_SHARED:
1246:     O << "shared";
1247:     break;
1248:   default:
1249:     report_fatal_error("Bad address space found while emitting PTX: " +
1250:                        llvm::Twine(AddressSpace));
1251:     break;
1252:   }
1253: }
1254:
1255: std::string
1256: NVPTXAsmPrinter::getPTXFundamentalTypeStr(Type *Ty, bool useB4PTR) const {
1257:   switch (Ty->getTypeID()) {
1258:   case Type::IntegerTyID: {
1259:     unsigned NumBits = cast<IntegerType>(Ty)->getBitWidth();
1260:     if (NumBits == 1)
```
- EN: This range implements operational logic in helpers such as oss, printSymbol, llvm::write_hex, assert, translating backend policy into executable code.
- CN: 这一段实现了 oss、printSymbol、llvm::write_hex、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1261-1350
```cpp
1261:       return "pred";
1262:     if (NumBits <= 64) {
1263:       std::string name = "u";
1264:       return name + utostr(NumBits);
1265:     }
1266:     llvm_unreachable("Integer too large");
1267:     break;
1268:   }
1269:   case Type::BFloatTyID:
1270:   case Type::HalfTyID:
1271:     // fp16 and bf16 are stored as .b16 for compatibility with pre-sm_53
1272:     // PTX assembly.
1273:     return "b16";
1274:   case Type::FloatTyID:
1275:     return "f32";
1276:   case Type::DoubleTyID:
1277:     return "f64";
1278:   case Type::PointerTyID: {
1279:     unsigned PtrSize = TM.getPointerSizeInBits(Ty->getPointerAddressSpace());
1280:     assert((PtrSize == 64 || PtrSize == 32) && "Unexpected pointer size");
1281:
1282:     if (PtrSize == 64)
1283:       if (useB4PTR)
1284:         return "b64";
1285:       else
1286:         return "u64";
1287:     else if (useB4PTR)
1288:       return "b32";
1289:     else
1290:       return "u32";
1291:   }
1292:   default:
1293:     break;
1294:   }
1295:   llvm_unreachable("unexpected type");
1296: }
1297:
1298: void NVPTXAsmPrinter::emitPTXGlobalVariable(const GlobalVariable *GVar,
1299:                                             raw_ostream &O,
1300:                                             const NVPTXSubtarget &STI) {
1301:   const DataLayout &DL = getDataLayout();
1302:
1303:   // GlobalVariables are always constant pointers themselves.
1304:   Type *ETy = GVar->getValueType();
1305:
1306:   O << ".";
1307:   emitPTXAddressSpace(GVar->getType()->getAddressSpace(), O);
1308:   if (isManaged(*GVar)) {
1309:     if (STI.getPTXVersion() < 40 || STI.getSmVersion() < 30)
1310:       report_fatal_error(
1311:           ".attribute(.managed) requires PTX version >= 4.0 and sm_30");
1312:
1313:     O << " .attribute(.managed)";
1314:   }
1315:   O << " .align "
1316:     << GVar->getAlign().value_or(DL.getPrefTypeAlign(ETy)).value();
1317:
1318:   // Special case for i128/fp128
1319:   if (ETy->getScalarSizeInBits() == 128) {
1320:     O << " .b8 ";
1321:     getSymbol(GVar)->print(O, MAI);
1322:     O << "[16]";
1323:     return;
1324:   }
1325:
1326:   if (ETy->isFloatingPointTy() || ETy->isIntOrPtrTy()) {
1327:     O << " ." << getPTXFundamentalTypeStr(ETy) << " ";
1328:     getSymbol(GVar)->print(O, MAI);
1329:     return;
1330:   }
1331:
1332:   int64_t ElementSize = 0;
1333:
1334:   // Although PTX has direct support for struct type and array type and LLVM IR
1335:   // is very similar to PTX, the LLVM CodeGen does not support for targets that
1336:   // support these high level field accesses. Structs and arrays are lowered
1337:   // into arrays of bytes.
1338:   switch (ETy->getTypeID()) {
1339:   case Type::StructTyID:
1340:   case Type::ArrayTyID:
1341:   case Type::FixedVectorTyID:
1342:     ElementSize = DL.getTypeStoreSize(ETy);
1343:     O << " .b8 ";
1344:     getSymbol(GVar)->print(O, MAI);
1345:     O << "[";
1346:     if (ElementSize) {
1347:       O << ElementSize;
1348:     }
1349:     O << "]";
1350:     break;
```
- EN: This range implements operational logic in helpers such as utostr, llvm_unreachable, getPointerSizeInBits, assert, translating backend policy into executable code.
- CN: 这一段实现了 utostr、llvm_unreachable、getPointerSizeInBits、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1351-1440
```cpp
1351:   default:
1352:     llvm_unreachable("type not supported yet");
1353:   }
1354: }
1355:
1356: void NVPTXAsmPrinter::emitFunctionParamList(const Function *F, raw_ostream &O) {
1357:   const DataLayout &DL = getDataLayout();
1358:   const NVPTXSubtarget &STI = TM.getSubtarget<NVPTXSubtarget>(*F);
1359:   const auto *TLI = cast<NVPTXTargetLowering>(STI.getTargetLowering());
1360:   const NVPTXMachineFunctionInfo *MFI =
1361:       MF ? MF->getInfo<NVPTXMachineFunctionInfo>() : nullptr;
1362:
1363:   bool IsFirst = true;
1364:   const bool IsKernelFunc = isKernelFunction(*F);
1365:
1366:   if (F->arg_empty() && !F->isVarArg()) {
1367:     O << "()";
1368:     return;
1369:   }
1370:
1371:   O << "(\n";
1372:
1373:   for (const Argument &Arg : F->args()) {
1374:     Type *Ty = Arg.getType();
1375:     const std::string ParamSym = TLI->getParamName(F, Arg.getArgNo());
1376:
1377:     if (!IsFirst)
1378:       O << ",\n";
1379:
1380:     IsFirst = false;
1381:
1382:     // Handle image/sampler parameters
1383:     if (IsKernelFunc) {
1384:       const PTXOpaqueType ArgOpaqueType = getPTXOpaqueType(Arg);
1385:       if (ArgOpaqueType != PTXOpaqueType::None) {
1386:         const bool EmitImgPtr = !MFI || !MFI->checkImageHandleSymbol(ParamSym);
1387:         O << "\t.param ";
1388:         if (EmitImgPtr)
1389:           O << ".u64 .ptr ";
1390:
1391:         switch (ArgOpaqueType) {
1392:         case PTXOpaqueType::Sampler:
1393:           O << ".samplerref ";
1394:           break;
1395:         case PTXOpaqueType::Texture:
1396:           O << ".texref ";
1397:           break;
1398:         case PTXOpaqueType::Surface:
1399:           O << ".surfref ";
1400:           break;
1401:         case PTXOpaqueType::None:
1402:           llvm_unreachable("handled above");
1403:         }
1404:         O << ParamSym;
1405:         continue;
1406:       }
1407:     }
1408:
1409:     auto GetOptimalAlignForParam = [&DL, F, &Arg](Type *Ty) -> Align {
1410:       if (MaybeAlign StackAlign =
1411:               getAlign(*F, Arg.getArgNo() + AttributeList::FirstArgIndex))
1412:         return StackAlign.value();
1413:
1414:       Align TypeAlign = getFunctionParamOptimizedAlign(F, Ty, DL);
1415:       MaybeAlign ParamAlign =
1416:           Arg.hasByValAttr() ? Arg.getParamAlign() : MaybeAlign();
1417:       return std::max(TypeAlign, ParamAlign.valueOrOne());
1418:     };
1419:
1420:     if (Arg.hasByValAttr()) {
1421:       // param has byVal attribute.
1422:       Type *ETy = Arg.getParamByValType();
1423:       assert(ETy && "Param should have byval type");
1424:
1425:       // Print .param .align <a> .b8 .param[size];
1426:       // <a>  = optimal alignment for the element type; always multiple of
1427:       //        PAL.getParamAlignment
1428:       // size = typeallocsize of element type
1429:       const Align OptimalAlign =
1430:           IsKernelFunc ? GetOptimalAlignForParam(ETy)
1431:                        : getFunctionByValParamAlign(
1432:                              F, ETy, Arg.getParamAlign().valueOrOne(), DL);
1433:
1434:       O << "\t.param .align " << OptimalAlign.value() << " .b8 " << ParamSym
1435:         << "[" << DL.getTypeAllocSize(ETy) << "]";
1436:       continue;
1437:     }
1438:
1439:     if (shouldPassAsArray(Ty)) {
1440:       // Just print .param .align <a> .b8 .param[size];
```
- EN: This range implements operational logic in helpers such as llvm_unreachable, NVPTXAsmPrinter::emitFunctionParamList, getDataLayout, getTargetLowering, translating backend policy into executable code.
- CN: 这一段实现了 llvm_unreachable、NVPTXAsmPrinter::emitFunctionParamList、getDataLayout、getTargetLowering 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1441-1530
```cpp
1441:       // <a>  = optimal alignment for the element type; always multiple of
1442:       //        PAL.getParamAlignment
1443:       // size = typeallocsize of element type
1444:       Align OptimalAlign = GetOptimalAlignForParam(Ty);
1445:
1446:       O << "\t.param .align " << OptimalAlign.value() << " .b8 " << ParamSym
1447:         << "[" << DL.getTypeAllocSize(Ty) << "]";
1448:
1449:       continue;
1450:     }
1451:     // Just a scalar
1452:     auto *PTy = dyn_cast<PointerType>(Ty);
1453:     unsigned PTySizeInBits = 0;
1454:     if (PTy) {
1455:       PTySizeInBits =
1456:           TLI->getPointerTy(DL, PTy->getAddressSpace()).getSizeInBits();
1457:       assert(PTySizeInBits && "Invalid pointer size");
1458:     }
1459:
1460:     if (IsKernelFunc) {
1461:       if (PTy) {
1462:         O << "\t.param .u" << PTySizeInBits << " .ptr";
1463:
1464:         switch (PTy->getAddressSpace()) {
1465:         default:
1466:           break;
1467:         case ADDRESS_SPACE_GLOBAL:
1468:           O << " .global";
1469:           break;
1470:         case ADDRESS_SPACE_SHARED:
1471:           O << " .shared";
1472:           break;
1473:         case ADDRESS_SPACE_CONST:
1474:           O << " .const";
1475:           break;
1476:         case ADDRESS_SPACE_LOCAL:
1477:           O << " .local";
1478:           break;
1479:         }
1480:
1481:         O << " .align " << Arg.getParamAlign().valueOrOne().value() << " "
1482:           << ParamSym;
1483:         continue;
1484:       }
1485:
1486:       // non-pointer scalar to kernel func
1487:       O << "\t.param .";
1488:       // Special case: predicate operands become .u8 types
1489:       if (Ty->isIntegerTy(1))
1490:         O << "u8";
1491:       else
1492:         O << getPTXFundamentalTypeStr(Ty);
1493:       O << " " << ParamSym;
1494:       continue;
1495:     }
1496:     // Non-kernel function, just print .param .b<size> for ABI
1497:     // and .reg .b<size> for non-ABI
1498:     unsigned Size;
1499:     if (auto *ITy = dyn_cast<IntegerType>(Ty)) {
1500:       Size = promoteScalarArgumentSize(ITy->getBitWidth());
1501:     } else if (PTy) {
1502:       assert(PTySizeInBits && "Invalid pointer size");
1503:       Size = PTySizeInBits;
1504:     } else
1505:       Size = Ty->getPrimitiveSizeInBits();
1506:     O << "\t.param .b" << Size << " " << ParamSym;
1507:   }
1508:
1509:   if (F->isVarArg()) {
1510:     if (!IsFirst)
1511:       O << ",\n";
1512:     O << "\t.param .align " << STI.getMaxRequiredAlignment() << " .b8 "
1513:       << TLI->getParamName(F, /* vararg */ -1) << "[]";
1514:   }
1515:
1516:   O << "\n)";
1517: }
1518:
1519: void NVPTXAsmPrinter::setAndEmitFunctionVirtualRegisters(
1520:     const MachineFunction &MF) {
1521:   SmallString<128> Str;
1522:   raw_svector_ostream O(Str);
1523:
1524:   // Map the global virtual register number to a register class specific
1525:   // virtual register number starting from 1 with that class.
1526:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
1527:
1528:   // Emit the Fake Stack Object
1529:   const MachineFrameInfo &MFI = MF.getFrameInfo();
1530:   int64_t NumBytes = MFI.getStackSize();
```
- EN: This range implements operational logic in helpers such as GetOptimalAlignForParam, getPointerTy, assert, getPTXFundamentalTypeStr, translating backend policy into executable code.
- CN: 这一段实现了 GetOptimalAlignForParam、getPointerTy、assert、getPTXFundamentalTypeStr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1531-1620
```cpp
1531:   if (NumBytes) {
1532:     O << "\t.local .align " << MFI.getMaxAlign().value() << " .b8 \t"
1533:       << DEPOTNAME << getFunctionNumber() << "[" << NumBytes << "];\n";
1534:     if (static_cast<const NVPTXTargetMachine &>(MF.getTarget()).is64Bit()) {
1535:       O << "\t.reg .b64 \t%SP;\n"
1536:         << "\t.reg .b64 \t%SPL;\n";
1537:     } else {
1538:       O << "\t.reg .b32 \t%SP;\n"
1539:         << "\t.reg .b32 \t%SPL;\n";
1540:     }
1541:   }
1542:
1543:   // Go through all virtual registers to establish the mapping between the
1544:   // global virtual
1545:   // register number and the per class virtual register number.
1546:   // We use the per class virtual register number in the ptx output.
1547:   for (unsigned I : llvm::seq(MRI->getNumVirtRegs())) {
1548:     Register VR = Register::index2VirtReg(I);
1549:     if (MRI->use_empty(VR) && MRI->def_empty(VR))
1550:       continue;
1551:     auto &RCRegMap = VRegMapping[MRI->getRegClass(VR)];
1552:     RCRegMap[VR] = RCRegMap.size() + 1;
1553:   }
1554:
1555:   // Emit declaration of the virtual registers or 'physical' registers for
1556:   // each register class
1557:   for (const TargetRegisterClass *RC : TRI->regclasses()) {
1558:     const unsigned N = VRegMapping[RC].size();
1559:
1560:     // Only declare those registers that may be used.
1561:     if (N) {
1562:       const StringRef RCName = getNVPTXRegClassName(RC);
1563:       const StringRef RCStr = getNVPTXRegClassStr(RC);
1564:       O << "\t.reg " << RCName << " \t" << RCStr << "<" << (N + 1) << ">;\n";
1565:     }
1566:   }
1567:
1568:   OutStreamer->emitRawText(O.str());
1569: }
1570:
1571: /// Translate virtual register numbers in DebugInfo locations to their printed
1572: /// encodings, as used by CUDA-GDB.
1573: void NVPTXAsmPrinter::encodeDebugInfoRegisterNumbers(
1574:     const MachineFunction &MF) {
1575:   const NVPTXSubtarget &STI = MF.getSubtarget<NVPTXSubtarget>();
1576:   const NVPTXRegisterInfo *registerInfo = STI.getRegisterInfo();
1577:
1578:   // Clear the old mapping, and add the new one.  This mapping is used after the
1579:   // printing of the current function is complete, but before the next function
1580:   // is printed.
1581:   registerInfo->clearDebugRegisterMap();
1582:
1583:   for (auto &classMap : VRegMapping) {
1584:     for (auto &registerMapping : classMap.getSecond()) {
1585:       auto reg = registerMapping.getFirst();
1586:       registerInfo->addToDebugRegisterMap(reg, getVirtualRegisterName(reg));
1587:     }
1588:   }
1589: }
1590:
1591: void NVPTXAsmPrinter::printFPConstant(const ConstantFP *Fp,
1592:                                       raw_ostream &O) const {
1593:   APFloat APF = APFloat(Fp->getValueAPF()); // make a copy
1594:   bool ignored;
1595:   unsigned int numHex;
1596:   const char *lead;
1597:
1598:   if (Fp->getType()->getTypeID() == Type::FloatTyID) {
1599:     numHex = 8;
1600:     lead = "0f";
1601:     APF.convert(APFloat::IEEEsingle(), APFloat::rmNearestTiesToEven, &ignored);
1602:   } else if (Fp->getType()->getTypeID() == Type::DoubleTyID) {
1603:     numHex = 16;
1604:     lead = "0d";
1605:     APF.convert(APFloat::IEEEdouble(), APFloat::rmNearestTiesToEven, &ignored);
1606:   } else
1607:     llvm_unreachable("unsupported fp type");
1608:
1609:   APInt API = APF.bitcastToAPInt();
1610:   O << lead << format_hex_no_prefix(API.getZExtValue(), numHex, /*Upper=*/true);
1611: }
1612:
1613: void NVPTXAsmPrinter::printScalarConstant(const Constant *CPV, raw_ostream &O) {
1614:   if (const ConstantInt *CI = dyn_cast<ConstantInt>(CPV)) {
1615:     O << CI->getValue();
1616:     return;
1617:   }
1618:   if (const ConstantFP *CFP = dyn_cast<ConstantFP>(CPV)) {
1619:     printFPConstant(CFP, O);
1620:     return;
```
- EN: This range implements operational logic in helpers such as Register::index2VirtReg, size, getNVPTXRegClassName, getNVPTXRegClassStr, translating backend policy into executable code.
- CN: 这一段实现了 Register::index2VirtReg、size、getNVPTXRegClassName、getNVPTXRegClassStr 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1621-1710
```cpp
1621:   }
1622:   if (isa<ConstantPointerNull>(CPV)) {
1623:     O << "0";
1624:     return;
1625:   }
1626:   if (const GlobalValue *GVar = dyn_cast<GlobalValue>(CPV)) {
1627:     const bool IsNonGenericPointer = GVar->getAddressSpace() != 0;
1628:     if (EmitGeneric && !isa<Function>(CPV) && !IsNonGenericPointer) {
1629:       O << "generic(";
1630:       getSymbol(GVar)->print(O, MAI);
1631:       O << ")";
1632:     } else {
1633:       getSymbol(GVar)->print(O, MAI);
1634:     }
1635:     return;
1636:   }
1637:   if (const ConstantExpr *Cexpr = dyn_cast<ConstantExpr>(CPV)) {
1638:     const MCExpr *E = lowerConstantForGV(cast<Constant>(Cexpr), false);
1639:     printMCExpr(*E, O);
1640:     return;
1641:   }
1642:   llvm_unreachable("Not scalar type found in printScalarConstant()");
1643: }
1644:
1645: void NVPTXAsmPrinter::bufferLEByte(const Constant *CPV, int Bytes,
1646:                                    AggBuffer *AggBuffer) {
1647:   const DataLayout &DL = getDataLayout();
1648:   int AllocSize = DL.getTypeAllocSize(CPV->getType());
1649:   if (isa<UndefValue>(CPV) || CPV->isNullValue()) {
1650:     // Non-zero Bytes indicates that we need to zero-fill everything. Otherwise,
1651:     // only the space allocated by CPV.
1652:     AggBuffer->addZeros(Bytes ? Bytes : AllocSize);
1653:     return;
1654:   }
1655:
1656:   // Helper for filling AggBuffer with APInts.
1657:   auto AddIntToBuffer = [AggBuffer, Bytes](const APInt &Val) {
1658:     size_t NumBytes = (Val.getBitWidth() + 7) / 8;
1659:     SmallVector<unsigned char, 16> Buf(NumBytes);
1660:     // `extractBitsAsZExtValue` does not allow the extraction of bits beyond the
1661:     // input's bit width, and i1 arrays may not have a length that is a multuple
1662:     // of 8. We handle the last byte separately, so we never request out of
1663:     // bounds bits.
1664:     for (unsigned I = 0; I < NumBytes - 1; ++I) {
1665:       Buf[I] = Val.extractBitsAsZExtValue(8, I * 8);
1666:     }
1667:     size_t LastBytePosition = (NumBytes - 1) * 8;
1668:     size_t LastByteBits = Val.getBitWidth() - LastBytePosition;
1669:     Buf[NumBytes - 1] =
1670:         Val.extractBitsAsZExtValue(LastByteBits, LastBytePosition);
1671:     AggBuffer->addBytes(Buf.data(), NumBytes, Bytes);
1672:   };
1673:
1674:   switch (CPV->getType()->getTypeID()) {
1675:   case Type::IntegerTyID:
1676:     if (const auto *CI = dyn_cast<ConstantInt>(CPV)) {
1677:       AddIntToBuffer(CI->getValue());
1678:       break;
1679:     }
1680:     if (const auto *Cexpr = dyn_cast<ConstantExpr>(CPV)) {
1681:       if (const auto *CI =
1682:               dyn_cast<ConstantInt>(ConstantFoldConstant(Cexpr, DL))) {
1683:         AddIntToBuffer(CI->getValue());
1684:         break;
1685:       }
1686:       if (Cexpr->getOpcode() == Instruction::PtrToInt) {
1687:         Value *V = Cexpr->getOperand(0)->stripPointerCasts();
1688:         AggBuffer->addSymbol(V, Cexpr->getOperand(0));
1689:         AggBuffer->addZeros(AllocSize);
1690:         break;
1691:       }
1692:     }
1693:     llvm_unreachable("unsupported integer const type");
1694:     break;
1695:
1696:   case Type::HalfTyID:
1697:   case Type::BFloatTyID:
1698:   case Type::FloatTyID:
1699:   case Type::DoubleTyID:
1700:     AddIntToBuffer(cast<ConstantFP>(CPV)->getValueAPF().bitcastToAPInt());
1701:     break;
1702:
1703:   case Type::PointerTyID: {
1704:     if (const GlobalValue *GVar = dyn_cast<GlobalValue>(CPV)) {
1705:       AggBuffer->addSymbol(GVar, GVar);
1706:     } else if (const ConstantExpr *Cexpr = dyn_cast<ConstantExpr>(CPV)) {
1707:       const Value *v = Cexpr->stripPointerCasts();
1708:       AggBuffer->addSymbol(v, Cexpr);
1709:     }
1710:     AggBuffer->addZeros(AllocSize);
```
- EN: This range implements operational logic in helpers such as getSymbol, lowerConstantForGV, printMCExpr, llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 getSymbol、lowerConstantForGV、printMCExpr、llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1711-1800
```cpp
1711:     break;
1712:   }
1713:
1714:   case Type::ArrayTyID:
1715:   case Type::FixedVectorTyID:
1716:   case Type::StructTyID: {
1717:     if (isa<ConstantAggregate>(CPV) || isa<ConstantDataSequential>(CPV)) {
1718:       bufferAggregateConstant(CPV, AggBuffer);
1719:       if (Bytes > AllocSize)
1720:         AggBuffer->addZeros(Bytes - AllocSize);
1721:     } else if (isa<ConstantAggregateZero>(CPV))
1722:       AggBuffer->addZeros(Bytes);
1723:     else
1724:       llvm_unreachable("Unexpected Constant type");
1725:     break;
1726:   }
1727:
1728:   default:
1729:     llvm_unreachable("unsupported type");
1730:   }
1731: }
1732:
1733: void NVPTXAsmPrinter::bufferAggregateConstant(const Constant *CPV,
1734:                                               AggBuffer *aggBuffer) {
1735:   const DataLayout &DL = getDataLayout();
1736:
1737:   auto ExtendBuffer = [](APInt Val, AggBuffer *Buffer) {
1738:     for (unsigned I : llvm::seq(Val.getBitWidth() / 8))
1739:       Buffer->addByte(Val.extractBitsAsZExtValue(8, I * 8));
1740:   };
1741:
1742:   // Integer or floating point vector splats.
1743:   if (isa<ConstantInt, ConstantFP>(CPV)) {
1744:     if (auto *VTy = dyn_cast<FixedVectorType>(CPV->getType())) {
1745:       for (unsigned I : llvm::seq(VTy->getNumElements()))
1746:         bufferLEByte(CPV->getAggregateElement(I), 0, aggBuffer);
1747:       return;
1748:     }
1749:   }
1750:
1751:   // Integers of arbitrary width
1752:   if (const ConstantInt *CI = dyn_cast<ConstantInt>(CPV)) {
1753:     assert(CI->getType()->isIntegerTy() && "Expected integer constant!");
1754:     ExtendBuffer(CI->getValue(), aggBuffer);
1755:     return;
1756:   }
1757:
1758:   // f128
1759:   if (const ConstantFP *CFP = dyn_cast<ConstantFP>(CPV)) {
1760:     assert(CFP->getType()->isFloatingPointTy() && "Expected fp constant!");
1761:     if (CFP->getType()->isFP128Ty()) {
1762:       ExtendBuffer(CFP->getValueAPF().bitcastToAPInt(), aggBuffer);
1763:       return;
1764:     }
1765:   }
1766:
1767:   // Buffer arrays one element at a time.
1768:   if (isa<ConstantArray>(CPV)) {
1769:     for (const auto &Op : CPV->operands())
1770:       bufferLEByte(cast<Constant>(Op), 0, aggBuffer);
1771:     return;
1772:   }
1773:
1774:   // Constant vectors
1775:   if (const auto *CVec = dyn_cast<ConstantVector>(CPV)) {
1776:     bufferAggregateConstVec(CVec, aggBuffer);
1777:     return;
1778:   }
1779:
1780:   if (const auto *CDS = dyn_cast<ConstantDataSequential>(CPV)) {
1781:     for (unsigned I : llvm::seq(CDS->getNumElements()))
1782:       bufferLEByte(cast<Constant>(CDS->getElementAsConstant(I)), 0, aggBuffer);
1783:     return;
1784:   }
1785:
1786:   if (isa<ConstantStruct>(CPV)) {
1787:     if (CPV->getNumOperands()) {
1788:       StructType *ST = cast<StructType>(CPV->getType());
1789:       for (unsigned I : llvm::seq(CPV->getNumOperands())) {
1790:         int EndOffset = (I + 1 == CPV->getNumOperands())
1791:                             ? DL.getStructLayout(ST)->getElementOffset(0) +
1792:                                   DL.getTypeAllocSize(ST)
1793:                             : DL.getStructLayout(ST)->getElementOffset(I + 1);
1794:         int Bytes = EndOffset - DL.getStructLayout(ST)->getElementOffset(I);
1795:         bufferLEByte(cast<Constant>(CPV->getOperand(I)), Bytes, aggBuffer);
1796:       }
1797:     }
1798:     return;
1799:   }
1800:   llvm_unreachable("unsupported constant type in printAggregateConstant()");
```
- EN: This range implements operational logic in helpers such as bufferAggregateConstant, addZeros, llvm_unreachable, getDataLayout, translating backend policy into executable code.
- CN: 这一段实现了 bufferAggregateConstant、addZeros、llvm_unreachable、getDataLayout 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1801-1890
```cpp
1801: }
1802:
1803: void NVPTXAsmPrinter::bufferAggregateConstVec(const ConstantVector *CV,
1804:                                               AggBuffer *aggBuffer) {
1805:   unsigned NumElems = CV->getType()->getNumElements();
1806:   const unsigned BuffSize = aggBuffer->getBufferSize();
1807:
1808:   // Buffer one element at a time if we have allocated enough buffer space.
1809:   if (BuffSize >= NumElems) {
1810:     for (const auto &Op : CV->operands())
1811:       bufferLEByte(cast<Constant>(Op), 0, aggBuffer);
1812:     return;
1813:   }
1814:
1815:   // Sub-byte datatypes will have more elements than bytes allocated for the
1816:   // buffer. Merge consecutive elements to form a full byte. We expect that 8 %
1817:   // sub-byte-elem-size should be 0 and current expected usage is for i4 (for
1818:   // e2m1-fp4 types).
1819:   Type *ElemTy = CV->getType()->getElementType();
1820:   assert(ElemTy->isIntegerTy() && "Expected integer data type.");
1821:   unsigned ElemTySize = ElemTy->getPrimitiveSizeInBits();
1822:   assert(ElemTySize < 8 && "Expected sub-byte data type.");
1823:   assert(8 % ElemTySize == 0 && "Element type size must evenly divide a byte.");
1824:   // Number of elements to merge to form a full byte.
1825:   unsigned NumElemsPerByte = 8 / ElemTySize;
1826:   unsigned NumCompleteBytes = NumElems / NumElemsPerByte;
1827:   unsigned NumTailElems = NumElems % NumElemsPerByte;
1828:
1829:   // Helper lambda to constant-fold sub-vector of sub-byte type elements into
1830:   // i8. Start and end indices of the sub-vector is provided, along with number
1831:   // of padding zeros if required.
1832:   auto ConvertSubCVtoInt8 = [this, &ElemTy](const ConstantVector *CV,
1833:                                             unsigned Start, unsigned End,
1834:                                             unsigned NumPaddingZeros = 0) {
1835:     // Collect elements to create sub-vector.
1836:     SmallVector<Constant *, 8> SubCVElems;
1837:     for (unsigned I : llvm::seq(Start, End))
1838:       SubCVElems.push_back(CV->getAggregateElement(I));
1839:
1840:     // Optionally pad with zeros.
1841:     if (NumPaddingZeros)
1842:       SubCVElems.append(NumPaddingZeros, ConstantInt::getNullValue(ElemTy));
1843:
1844:     auto SubCV = ConstantVector::get(SubCVElems);
1845:     Type *Int8Ty = IntegerType::get(SubCV->getContext(), 8);
1846:
1847:     // Merge elements of the sub-vector using ConstantFolding.
1848:     ConstantInt *MergedElem =
1849:         dyn_cast_or_null<ConstantInt>(ConstantFoldConstant(
1850:             ConstantExpr::getBitCast(const_cast<Constant *>(SubCV), Int8Ty),
1851:             getDataLayout()));
1852:
1853:     if (!MergedElem)
1854:       report_fatal_error(
1855:           "Cannot lower vector global with unusual element type");
1856:
1857:     return MergedElem;
1858:   };
1859:
1860:   // Iterate through elements of vector one chunk at a time and buffer that
1861:   // chunk.
1862:   for (unsigned ByteIdx : llvm::seq(NumCompleteBytes))
1863:     bufferLEByte(ConvertSubCVtoInt8(CV, ByteIdx * NumElemsPerByte,
1864:                                     (ByteIdx + 1) * NumElemsPerByte),
1865:                  0, aggBuffer);
1866:
1867:   // For unevenly sized vectors add tail padding zeros.
1868:   if (NumTailElems > 0)
1869:     bufferLEByte(ConvertSubCVtoInt8(CV, NumElems - NumTailElems, NumElems,
1870:                                     NumElemsPerByte - NumTailElems),
1871:                  0, aggBuffer);
1872: }
1873:
1874: /// lowerConstantForGV - Return an MCExpr for the given Constant.  This is mostly
1875: /// a copy from AsmPrinter::lowerConstant, except customized to only handle
1876: /// expressions that are representable in PTX and create
1877: /// NVPTXGenericMCSymbolRefExpr nodes for addrspacecast instructions.
1878: const MCExpr *
1879: NVPTXAsmPrinter::lowerConstantForGV(const Constant *CV,
1880:                                     bool ProcessingGeneric) const {
1881:   MCContext &Ctx = OutContext;
1882:
1883:   if (CV->isNullValue() || isa<UndefValue>(CV))
1884:     return MCConstantExpr::create(0, Ctx);
1885:
1886:   if (const ConstantInt *CI = dyn_cast<ConstantInt>(CV))
1887:     return MCConstantExpr::create(CI->getZExtValue(), Ctx);
1888:
1889:   if (const GlobalValue *GV = dyn_cast<GlobalValue>(CV)) {
1890:     const MCSymbolRefExpr *Expr = MCSymbolRefExpr::create(getSymbol(GV), Ctx);
```
- EN: This range implements operational logic in helpers such as getType, getBufferSize, bufferLEByte, assert, translating backend policy into executable code.
- CN: 这一段实现了 getType、getBufferSize、bufferLEByte、assert 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1891-1980
```cpp
1891:     if (ProcessingGeneric)
1892:       return NVPTXGenericMCSymbolRefExpr::create(Expr, Ctx);
1893:     return Expr;
1894:   }
1895:
1896:   const ConstantExpr *CE = dyn_cast<ConstantExpr>(CV);
1897:   if (!CE) {
1898:     llvm_unreachable("Unknown constant value to lower!");
1899:   }
1900:
1901:   switch (CE->getOpcode()) {
1902:   default:
1903:     break; // Error
1904:
1905:   case Instruction::AddrSpaceCast: {
1906:     // Strip the addrspacecast and pass along the operand
1907:     PointerType *DstTy = cast<PointerType>(CE->getType());
1908:     if (DstTy->getAddressSpace() == 0)
1909:       return lowerConstantForGV(cast<const Constant>(CE->getOperand(0)), true);
1910:
1911:     break; // Error
1912:   }
1913:
1914:   case Instruction::GetElementPtr: {
1915:     const DataLayout &DL = getDataLayout();
1916:
1917:     // Generate a symbolic expression for the byte address
1918:     APInt OffsetAI(DL.getPointerTypeSizeInBits(CE->getType()), 0);
1919:     cast<GEPOperator>(CE)->accumulateConstantOffset(DL, OffsetAI);
1920:
1921:     const MCExpr *Base = lowerConstantForGV(CE->getOperand(0),
1922:                                             ProcessingGeneric);
1923:     if (!OffsetAI)
1924:       return Base;
1925:
1926:     int64_t Offset = OffsetAI.getSExtValue();
1927:     return MCBinaryExpr::createAdd(Base, MCConstantExpr::create(Offset, Ctx),
1928:                                    Ctx);
1929:   }
1930:
1931:   case Instruction::Trunc:
1932:     // We emit the value and depend on the assembler to truncate the generated
1933:     // expression properly.  This is important for differences between
1934:     // blockaddress labels.  Since the two labels are in the same function, it
1935:     // is reasonable to treat their delta as a 32-bit value.
1936:     [[fallthrough]];
1937:   case Instruction::BitCast:
1938:     return lowerConstantForGV(CE->getOperand(0), ProcessingGeneric);
1939:
1940:   case Instruction::IntToPtr: {
1941:     const DataLayout &DL = getDataLayout();
1942:
1943:     // Handle casts to pointers by changing them into casts to the appropriate
1944:     // integer type.  This promotes constant folding and simplifies this code.
1945:     Constant *Op = CE->getOperand(0);
1946:     Op = ConstantFoldIntegerCast(Op, DL.getIntPtrType(CV->getType()),
1947:                                  /*IsSigned*/ false, DL);
1948:     if (Op)
1949:       return lowerConstantForGV(Op, ProcessingGeneric);
1950:
1951:     break; // Error
1952:   }
1953:
1954:   case Instruction::PtrToInt: {
1955:     const DataLayout &DL = getDataLayout();
1956:
1957:     // Support only foldable casts to/from pointers that can be eliminated by
1958:     // changing the pointer to the appropriately sized integer type.
1959:     Constant *Op = CE->getOperand(0);
1960:     Type *Ty = CE->getType();
1961:
1962:     const MCExpr *OpExpr = lowerConstantForGV(Op, ProcessingGeneric);
1963:
1964:     // We can emit the pointer value into this slot if the slot is an
1965:     // integer slot equal to the size of the pointer.
1966:     if (DL.getTypeAllocSize(Ty) == DL.getTypeAllocSize(Op->getType()))
1967:       return OpExpr;
1968:
1969:     // Otherwise the pointer is smaller than the resultant integer, mask off
1970:     // the high bits so we are sure to get a proper truncation if the input is
1971:     // a constant expr.
1972:     unsigned InBits = DL.getTypeAllocSizeInBits(Op->getType());
1973:     const MCExpr *MaskExpr = MCConstantExpr::create(~0ULL >> (64-InBits), Ctx);
1974:     return MCBinaryExpr::createAnd(OpExpr, MaskExpr, Ctx);
1975:   }
1976:
1977:   // The MC library also has a right-shift operator, but it isn't consistently
1978:   // signed or unsigned between different targets.
1979:   case Instruction::Add: {
1980:     const MCExpr *LHS = lowerConstantForGV(CE->getOperand(0), ProcessingGeneric);
```
- EN: This range implements operational logic in helpers such as NVPTXGenericMCSymbolRefExpr::create, llvm_unreachable, getType, lowerConstantForGV, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXGenericMCSymbolRefExpr::create、llvm_unreachable、getType、lowerConstantForGV 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1981-2070
```cpp
1981:     const MCExpr *RHS = lowerConstantForGV(CE->getOperand(1), ProcessingGeneric);
1982:     switch (CE->getOpcode()) {
1983:     default: llvm_unreachable("Unknown binary operator constant cast expr");
1984:     case Instruction::Add: return MCBinaryExpr::createAdd(LHS, RHS, Ctx);
1985:     }
1986:   }
1987:   }
1988:
1989:   // If the code isn't optimized, there may be outstanding folding
1990:   // opportunities. Attempt to fold the expression using DataLayout as a
1991:   // last resort before giving up.
1992:   Constant *C = ConstantFoldConstant(CE, getDataLayout());
1993:   if (C != CE)
1994:     return lowerConstantForGV(C, ProcessingGeneric);
1995:
1996:   // Otherwise report the problem to the user.
1997:   std::string S;
1998:   raw_string_ostream OS(S);
1999:   OS << "Unsupported expression in static initializer: ";
2000:   CE->printAsOperand(OS, /*PrintType=*/false,
2001:                  !MF ? nullptr : MF->getFunction().getParent());
2002:   report_fatal_error(Twine(OS.str()));
2003: }
2004:
2005: void NVPTXAsmPrinter::printMCExpr(const MCExpr &Expr, raw_ostream &OS) const {
2006:   OutContext.getAsmInfo().printExpr(OS, Expr);
2007: }
2008:
2009: /// PrintAsmOperand - Print out an operand for an inline asm expression.
2010: ///
2011: bool NVPTXAsmPrinter::PrintAsmOperand(const MachineInstr *MI, unsigned OpNo,
2012:                                       const char *ExtraCode, raw_ostream &O) {
2013:   if (ExtraCode && ExtraCode[0]) {
2014:     if (ExtraCode[1] != 0)
2015:       return true; // Unknown modifier.
2016:
2017:     switch (ExtraCode[0]) {
2018:     default:
2019:       // See if this is a generic print operand
2020:       return AsmPrinter::PrintAsmOperand(MI, OpNo, ExtraCode, O);
2021:     case 'r':
2022:       break;
2023:     }
2024:   }
2025:
2026:   printOperand(MI, OpNo, O);
2027:
2028:   return false;
2029: }
2030:
2031: bool NVPTXAsmPrinter::PrintAsmMemoryOperand(const MachineInstr *MI,
2032:                                             unsigned OpNo,
2033:                                             const char *ExtraCode,
2034:                                             raw_ostream &O) {
2035:   if (ExtraCode && ExtraCode[0])
2036:     return true; // Unknown modifier
2037:
2038:   O << '[';
2039:   printMemOperand(MI, OpNo, O);
2040:   O << ']';
2041:
2042:   return false;
2043: }
2044:
2045: void NVPTXAsmPrinter::printOperand(const MachineInstr *MI, unsigned OpNum,
2046:                                    raw_ostream &O) {
2047:   const MachineOperand &MO = MI->getOperand(OpNum);
2048:   switch (MO.getType()) {
2049:   case MachineOperand::MO_Register:
2050:     if (MO.getReg().isPhysical()) {
2051:       if (MO.getReg() == NVPTX::VRDepot)
2052:         O << DEPOTNAME << getFunctionNumber();
2053:       else
2054:         O << NVPTXInstPrinter::getRegisterName(MO.getReg());
2055:     } else {
2056:       emitVirtualRegister(MO.getReg(), O);
2057:     }
2058:     break;
2059:
2060:   case MachineOperand::MO_Immediate:
2061:     O << MO.getImm();
2062:     break;
2063:
2064:   case MachineOperand::MO_FPImmediate:
2065:     printFPConstant(MO.getFPImm(), O);
2066:     break;
2067:
2068:   case MachineOperand::MO_GlobalAddress:
2069:     PrintSymbolOperand(MO, O);
2070:     break;
```
- EN: This range implements operational logic in helpers such as lowerConstantForGV, llvm_unreachable, MCBinaryExpr::createAdd, ConstantFoldConstant, translating backend policy into executable code.
- CN: 这一段实现了 lowerConstantForGV、llvm_unreachable、MCBinaryExpr::createAdd、ConstantFoldConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2071-2160
```cpp
2071:
2072:   case MachineOperand::MO_MachineBasicBlock:
2073:     MO.getMBB()->getSymbol()->print(O, MAI);
2074:     break;
2075:
2076:   default:
2077:     llvm_unreachable("Operand type not supported.");
2078:   }
2079: }
2080:
2081: void NVPTXAsmPrinter::printMemOperand(const MachineInstr *MI, unsigned OpNum,
2082:                                       raw_ostream &O, const char *Modifier) {
2083:   printOperand(MI, OpNum, O);
2084:
2085:   if (Modifier && strcmp(Modifier, "add") == 0) {
2086:     O << ", ";
2087:     printOperand(MI, OpNum + 1, O);
2088:   } else {
2089:     if (MI->getOperand(OpNum + 1).isImm() &&
2090:         MI->getOperand(OpNum + 1).getImm() == 0)
2091:       return; // don't print ',0' or '+0'
2092:     O << "+";
2093:     printOperand(MI, OpNum + 1, O);
2094:   }
2095: }
2096:
2097: /// Returns true if \p Line begins with an alphabetic character or underscore,
2098: /// indicating it is a PTX instruction that should receive a .loc directive.
2099: static bool isPTXInstruction(StringRef Line) {
2100:   StringRef Trimmed = Line.ltrim();
2101:   return !Trimmed.empty() &&
2102:          (std::isalpha(static_cast<unsigned char>(Trimmed[0])) ||
2103:           Trimmed[0] == '_');
2104: }
2105:
2106: /// Returns the DILocation for an inline asm MachineInstr if debug line info
2107: /// should be emitted, or nullptr otherwise.
2108: static const DILocation *getInlineAsmDebugLoc(const MachineInstr *MI) {
2109:   if (!MI || !MI->getDebugLoc())
2110:     return nullptr;
2111:   const DISubprogram *SP = MI->getMF()->getFunction().getSubprogram();
2112:   if (!SP || SP->getUnit()->getEmissionKind() == DICompileUnit::NoDebug)
2113:     return nullptr;
2114:   const DILocation *DL = MI->getDebugLoc();
2115:   if (!DL->getFile() || !DL->getLine())
2116:     return nullptr;
2117:   return DL;
2118: }
2119:
2120: namespace {
2121: struct InlineAsmInliningContext {
2122:   MCSymbol *FuncNameSym = nullptr;
2123:   unsigned FileIA = 0;
2124:   unsigned LineIA = 0;
2125:   unsigned ColIA = 0;
2126:
2127:   bool hasInlinedAt() const { return FuncNameSym != nullptr; }
2128: };
2129: } // namespace
2130:
2131: /// Resolves the enhanced-lineinfo inlining context for an inline asm debug
2132: /// location. Returns a default (empty) context if inlining info is unavailable.
2133: static InlineAsmInliningContext
2134: getInlineAsmInliningContext(const DILocation *DL, const MachineFunction &MF,
2135:                             NVPTXDwarfDebug *NVDD, MCStreamer &Streamer,
2136:                             unsigned CUID) {
2137:   InlineAsmInliningContext Ctx;
2138:   const DILocation *InlinedAt = DL->getInlinedAt();
2139:   if (!InlinedAt || !InlinedAt->getFile() || !NVDD ||
2140:       !NVDD->isEnhancedLineinfo(MF))
2141:     return Ctx;
2142:   const auto *SubProg = getDISubprogram(DL->getScope());
2143:   if (!SubProg)
2144:     return Ctx;
2145:   Ctx.FuncNameSym = NVDD->getOrCreateFuncNameSymbol(SubProg->getLinkageName());
2146:   Ctx.FileIA = Streamer.emitDwarfFileDirective(
2147:       0, InlinedAt->getFile()->getDirectory(),
2148:       InlinedAt->getFile()->getFilename(), std::nullopt, std::nullopt, CUID);
2149:   Ctx.LineIA = InlinedAt->getLine();
2150:   Ctx.ColIA = InlinedAt->getColumn();
2151:   return Ctx;
2152: }
2153:
2154: void NVPTXAsmPrinter::emitInlineAsm(StringRef Str, const MCSubtargetInfo &STI,
2155:                                     const MCTargetOptions &MCOptions,
2156:                                     const MDNode *LocMDNode,
2157:                                     InlineAsm::AsmDialect Dialect,
2158:                                     const MachineInstr *MI) {
2159:   assert(!Str.empty() && "Can't emit empty inline asm block");
2160:   if (Str.back() == 0)
```
- EN: This range defines or declares important types such as getMBB, llvm_unreachable, printOperand, getOperand, shaping the data model used by NVPTXAsmPrinter.cpp.
- CN: 这一段定义或声明了 getMBB、llvm_unreachable、printOperand、getOperand 等关键类型，构成 NVPTXAsmPrinter.cpp 使用的数据模型。

### Lines 2161-2220
```cpp
2161:     Str = Str.substr(0, Str.size() - 1);
2162:
2163:   auto emitAsmStr = [&](StringRef AsmStr) {
2164:     emitInlineAsmStart();
2165:     OutStreamer->emitRawText(AsmStr);
2166:     emitInlineAsmEnd(STI, nullptr, MI);
2167:   };
2168:
2169:   const DILocation *DL = getInlineAsmDebugLoc(MI);
2170:   if (!DL) {
2171:     emitAsmStr(Str);
2172:     return;
2173:   }
2174:
2175:   const DIFile *File = DL->getFile();
2176:   unsigned Line = DL->getLine();
2177:   const unsigned Column = DL->getColumn();
2178:   const unsigned CUID = OutStreamer->getContext().getDwarfCompileUnitID();
2179:   const unsigned FileNumber = OutStreamer->emitDwarfFileDirective(
2180:       0, File->getDirectory(), File->getFilename(), std::nullopt, std::nullopt,
2181:       CUID);
2182:
2183:   auto *NVDD = static_cast<NVPTXDwarfDebug *>(getDwarfDebug());
2184:   InlineAsmInliningContext InlineCtx =
2185:       getInlineAsmInliningContext(DL, *MI->getMF(), NVDD, *OutStreamer, CUID);
2186:
2187:   SmallVector<StringRef, 16> Lines;
2188:   Str.split(Lines, '\n');
2189:   emitInlineAsmStart();
2190:   for (const StringRef &L : Lines) {
2191:     StringRef RTrimmed = L.rtrim('\r');
2192:     if (isPTXInstruction(L)) {
2193:       if (InlineCtx.hasInlinedAt()) {
2194:         OutStreamer->emitDwarfLocDirectiveWithInlinedAt(
2195:             FileNumber, Line, Column, InlineCtx.FileIA, InlineCtx.LineIA,
2196:             InlineCtx.ColIA, InlineCtx.FuncNameSym, DWARF2_FLAG_IS_STMT, 0, 0,
2197:             File->getFilename());
2198:       } else {
2199:         OutStreamer->emitDwarfLocDirective(FileNumber, Line, Column,
2200:                                            DWARF2_FLAG_IS_STMT, 0, 0,
2201:                                            File->getFilename());
2202:       }
2203:     }
2204:     OutStreamer->emitRawText(RTrimmed);
2205:     ++Line;
2206:   }
2207:   emitInlineAsmEnd(STI, nullptr, MI);
2208: }
2209:
2210: char NVPTXAsmPrinter::ID = 0;
2211:
2212: INITIALIZE_PASS(NVPTXAsmPrinter, "nvptx-asm-printer", "NVPTX Assembly Printer",
2213:                 false, false)
2214:
2215: // Force static initialization.
2216: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
2217: LLVMInitializeNVPTXAsmPrinter() {
2218:   RegisterAsmPrinter<NVPTXAsmPrinter> X(getTheNVPTXTarget32());
2219:   RegisterAsmPrinter<NVPTXAsmPrinter> Y(getTheNVPTXTarget64());
2220: }
```
- EN: This range implements operational logic in helpers such as substr, emitInlineAsmStart, emitRawText, emitInlineAsmEnd, translating backend policy into executable code.
- CN: 这一段实现了 substr、emitInlineAsmStart、emitRawText、emitInlineAsmEnd 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: The asm printer layer serializes machine-level state into target assembly or object-oriented output.
  - CN: 汇编打印层负责把机器级状态序列化为目标汇编或面向目标的输出。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include getTextureName, assert, getName, getSurfaceName, getSamplerName, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getTextureName, assert, getName, getSurfaceName, getSamplerName，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXAsmPrinter.h`
  - `MCTargetDesc/NVPTXBaseInfo.h`
  - `MCTargetDesc/NVPTXInstPrinter.h`
  - `MCTargetDesc/NVPTXMCAsmInfo.h`
  - `MCTargetDesc/NVPTXTargetStreamer.h`
  - `NVPTX.h`
  - `NVPTXDwarfDebug.h`
  - `NVPTXMCExpr.h`
  - `NVPTXMachineFunctionInfo.h`
  - `NVPTXRegisterInfo.h`
  - `NVPTXSubtarget.h`
  - `NVPTXTargetMachine.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APFloat.h`
  - `llvm/ADT/APInt.h`
  - `llvm/ADT/ArrayRef.h`
  - `llvm/ADT/DenseMap.h`
  - `llvm/ADT/DenseSet.h`
  - `llvm/ADT/SmallString.h`
  - `llvm/ADT/SmallVector.h`
  - `llvm/ADT/StringExtras.h`
  - `llvm/ADT/StringRef.h`
  - `llvm/ADT/Twine.h`
  - `llvm/ADT/iterator_range.h`
  - `llvm/Analysis/ConstantFolding.h`
  - `llvm/CodeGen/Analysis.h`
  - `llvm/CodeGen/MachineBasicBlock.h`
  - `llvm/CodeGen/MachineFrameInfo.h`
  - `llvm/CodeGen/MachineFunction.h`
- System/standard headers / 系统或标准头文件:
  - `cassert`
  - `cstdint`
  - `cstring`
  - `string`
