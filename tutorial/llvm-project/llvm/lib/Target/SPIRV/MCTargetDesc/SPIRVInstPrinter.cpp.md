# SPIRVInstPrinter.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/SPIRV/MCTargetDesc/SPIRVInstPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): This class prints a SPIR-V MCInst to a .s file.
- 目的（中文）: 该文件用于支撑 LLVM 目标后端的相关功能。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-60
```cpp
 1: //===-- SPIRVInstPrinter.cpp - Output SPIR-V MCInsts as ASM -----*- C++ -*-===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This class prints a SPIR-V MCInst to a .s file.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "SPIRVInstPrinter.h"
14: #include "SPIRV.h"
15: #include "SPIRVBaseInfo.h"
16: #include "llvm/ADT/APFloat.h"
17: #include "llvm/MC/MCAsmInfo.h"
18: #include "llvm/MC/MCExpr.h"
19: #include "llvm/MC/MCInst.h"
20: #include "llvm/MC/MCInstrInfo.h"
21: #include "llvm/MC/MCSymbol.h"
22: #include "llvm/Support/ErrorHandling.h"
23:
24: using namespace llvm;
25: using namespace llvm::SPIRV;
26:
27: #define DEBUG_TYPE "asm-printer"
28:
29: // Include the auto-generated portion of the assembly writer.
30: #include "SPIRVGenAsmWriter.inc"
31:
32: void SPIRVInstPrinter::printRemainingVariableOps(const MCInst *MI,
33:                                                  unsigned StartIndex,
34:                                                  raw_ostream &O,
35:                                                  bool SkipFirstSpace,
36:                                                  bool SkipImmediates) {
37:   const unsigned NumOps = MI->getNumOperands();
38:   for (unsigned i = StartIndex; i < NumOps; ++i) {
39:     if (!SkipImmediates || !MI->getOperand(i).isImm()) {
40:       if (!SkipFirstSpace || i != StartIndex)
41:         O << ' ';
42:       printOperand(MI, i, O);
43:     }
44:   }
45: }
46:
47: void SPIRVInstPrinter::printOpConstantVarOps(const MCInst *MI,
48:                                              unsigned StartIndex,
49:                                              raw_ostream &O) {
50:   unsigned IsBitwidth16 = MI->getFlags() & SPIRV::INST_PRINTER_WIDTH16;
51:   const unsigned NumVarOps = MI->getNumOperands() - StartIndex;
52:
53:   if (MI->getOpcode() == SPIRV::OpConstantI && NumVarOps > 2) {
54:     // Look up the bitwidth of this int type register from
55:     // IntTypeBitwidths map.
56:     MCRegister IntTypeReg = MI->getOperand(1).getReg();
57:     unsigned Bitwidth = IntTypeBitwidths.at(IntTypeReg);
58:
59:     // SPV_ALTERA_arbitrary_precision_integers allows for integer widths greater
60:     // than 64, which will be encoded via multiple operands.
```
- EN: This range implements operational logic in helpers such as getNumOperands, printOperand, getOperand, at, translating backend policy into executable code.
- CN: 这一段实现了 getNumOperands、printOperand、getOperand、at 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 61-120
```cpp
 61:     const unsigned TotalBits = NumVarOps * 32;
 62:     APInt Val(TotalBits, 0);
 63:     for (unsigned i = 0; i < NumVarOps; ++i) {
 64:       uint64_t Word = MI->getOperand(StartIndex + i).getImm();
 65:       Val |= APInt(TotalBits, Word) << (i * 32);
 66:     }
 67:     APInt ActualVal = Val.trunc(Bitwidth);
 68:     O << ' ';
 69:     ActualVal.print(O, /*isSigned=*/false);
 70:     return;
 71:   }
 72:
 73:   assert((NumVarOps == 1 || NumVarOps == 2) &&
 74:          "Unsupported number of bits for literal variable");
 75:
 76:   O << ' ';
 77:
 78:   uint64_t Imm = MI->getOperand(StartIndex).getImm();
 79:
 80:   // Handle 64 bit literals.
 81:   if (NumVarOps == 2) {
 82:     Imm |= (MI->getOperand(StartIndex + 1).getImm() << 32);
 83:   }
 84:
 85:   // Format and print float values.
 86:   if (MI->getOpcode() == SPIRV::OpConstantF && IsBitwidth16 == 0) {
 87:     APFloat FP = NumVarOps == 1 ? APFloat(APInt(32, Imm).bitsToFloat())
 88:                                 : APFloat(APInt(64, Imm).bitsToDouble());
 89:
 90:     // Print infinity and NaN as hex floats.
 91:     // TODO: Make sure subnormal numbers are handled correctly as they may also
 92:     // require hex float notation.
 93:     if (FP.isInfinity()) {
 94:       if (FP.isNegative())
 95:         O << '-';
 96:       O << "0x1p+128";
 97:       return;
 98:     }
 99:     if (FP.isNaN()) {
100:       O << "0x1.8p+128";
101:       return;
102:     }
103:
104:     // Format val as a decimal floating point or scientific notation (whichever
105:     // is shorter), with enough digits of precision to produce the exact value.
106:     O << format("%.*g", std::numeric_limits<double>::max_digits10,
107:                 FP.convertToDouble());
108:
109:     return;
110:   }
111:
112:   // Print integer values directly.
113:   O << Imm;
114: }
115:
116: void SPIRVInstPrinter::recordIntType(const MCInst *MI) {
117:   MCRegister IntTypeReg = MI->getOperand(0).getReg();
118:   unsigned Bitwidth = MI->getOperand(1).getImm();
119:   IntTypeBitwidths[IntTypeReg] = Bitwidth;
120: }
```
- EN: This range implements operational logic in helpers such as Val, getOperand, APInt, trunc, translating backend policy into executable code.
- CN: 这一段实现了 Val、getOperand、APInt、trunc 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 121-180
```cpp
121:
122: void SPIRVInstPrinter::recordOpExtInstImport(const MCInst *MI) {
123:   MCRegister Reg = MI->getOperand(0).getReg();
124:   auto Name = getSPIRVStringOperand(*MI, 1);
125:   auto Set = getExtInstSetFromString(std::move(Name));
126:   ExtInstSetIDs.insert({Reg, Set});
127: }
128:
129: void SPIRVInstPrinter::printInst(const MCInst *MI, uint64_t Address,
130:                                  StringRef Annot, const MCSubtargetInfo &STI,
131:                                  raw_ostream &OS) {
132:   const unsigned OpCode = MI->getOpcode();
133:   printInstruction(MI, Address, OS);
134:   if (OpCode == SPIRV::OpTypeInt) {
135:     recordIntType(MI);
136:   }
137:
138:   if (OpCode == SPIRV::OpDecorate || OpCode == SPIRV::OpDecorateId) {
139:     printOpDecorate(MI, OS);
140:   } else if (OpCode == SPIRV::OpExtInstImport) {
141:     recordOpExtInstImport(MI);
142:   } else if (OpCode == SPIRV::OpExtInst) {
143:     printOpExtInst(MI, OS);
144:   } else if (OpCode == SPIRV::UNKNOWN_type) {
145:     printUnknownType(MI, OS);
146:   } else {
147:     // Print any extra operands for variadic instructions.
148:     const MCInstrDesc &MCDesc = MII.get(OpCode);
149:     if (MCDesc.isVariadic()) {
150:       const unsigned NumFixedOps = MCDesc.getNumOperands();
151:       const unsigned LastFixedIndex = NumFixedOps - 1;
152:       const int FirstVariableIndex = NumFixedOps;
153:       if (NumFixedOps > 0 && MCDesc.operands()[LastFixedIndex].OperandType ==
154:                                  MCOI::OPERAND_UNKNOWN) {
155:         // For instructions where a custom type (not reg or immediate) comes as
156:         // the last operand before the variable_ops. This is usually a StringImm
157:         // operand, but there are a few other cases.
158:         switch (OpCode) {
159:         case SPIRV::OpTypeImage:
160:           OS << ' ';
161:           printSymbolicOperand<OperandCategory::AccessQualifierOperand>(
162:               MI, FirstVariableIndex, OS);
163:           break;
164:         case SPIRV::OpVariable:
165:           OS << ' ';
166:           printOperand(MI, FirstVariableIndex, OS);
167:           break;
168:         case SPIRV::OpEntryPoint: {
169:           // Print the interface ID operands, skipping the name's string
170:           // literal.
171:           printRemainingVariableOps(MI, NumFixedOps, OS, false, true);
172:           break;
173:         }
174:         case SPIRV::OpMemberDecorate:
175:           printRemainingVariableOps(MI, NumFixedOps, OS);
176:           break;
177:         case SPIRV::OpExecutionMode:
178:         case SPIRV::OpExecutionModeId:
179:         case SPIRV::OpLoopMerge:
180:         case SPIRV::OpLoopControlINTEL: {
```
- EN: This range implements operational logic in helpers such as SPIRVInstPrinter::recordOpExtInstImport, getOperand, getSPIRVStringOperand, getExtInstSetFromString, translating backend policy into executable code.
- CN: 这一段实现了 SPIRVInstPrinter::recordOpExtInstImport、getOperand、getSPIRVStringOperand、getExtInstSetFromString 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-240
```cpp
181:           // Print any literals after the OPERAND_UNKNOWN argument normally.
182:           printRemainingVariableOps(MI, NumFixedOps, OS);
183:           break;
184:         }
185:         default:
186:           break; // printStringImm has already been handled.
187:         }
188:       } else {
189:         // For instructions with no fixed ops or a reg/immediate as the final
190:         // fixed operand, we can usually print the rest with "printOperand", but
191:         // check for a few cases with custom types first.
192:         switch (OpCode) {
193:         case SPIRV::OpLoad:
194:         case SPIRV::OpStore:
195:           OS << ' ';
196:           printSymbolicOperand<OperandCategory::MemoryOperandOperand>(
197:               MI, FirstVariableIndex, OS);
198:           printRemainingVariableOps(MI, FirstVariableIndex + 1, OS);
199:           break;
200:         case SPIRV::OpSwitch:
201:           if (MI->getFlags() & SPIRV::INST_PRINTER_WIDTH64) {
202:             // In binary format 64-bit types are split into two 32-bit operands,
203:             // but in text format combine these into a single 64-bit value as
204:             // this is what tools such as spirv-as require.
205:             const unsigned NumOps = MI->getNumOperands();
206:             for (unsigned OpIdx = NumFixedOps; OpIdx < NumOps;) {
207:               if (OpIdx + 1 >= NumOps || !MI->getOperand(OpIdx).isImm() ||
208:                   !MI->getOperand(OpIdx + 1).isImm()) {
209:                 llvm_unreachable("Unexpected OpSwitch operands");
210:                 continue;
211:               }
212:               OS << ' ';
213:               uint64_t LowBits = MI->getOperand(OpIdx).getImm();
214:               uint64_t HighBits = MI->getOperand(OpIdx + 1).getImm();
215:               uint64_t CombinedValue = (HighBits << 32) | LowBits;
216:               OS << formatImm(CombinedValue);
217:               OpIdx += 2;
218:
219:               // Next should be the label
220:               if (OpIdx < NumOps) {
221:                 OS << ' ';
222:                 printOperand(MI, OpIdx, OS);
223:                 OpIdx++;
224:               }
225:             }
226:           } else {
227:             printRemainingVariableOps(MI, NumFixedOps, OS);
228:           }
229:           break;
230:         case SPIRV::OpImageSampleImplicitLod:
231:         case SPIRV::OpImageSampleDrefImplicitLod:
232:         case SPIRV::OpImageSampleProjImplicitLod:
233:         case SPIRV::OpImageSampleProjDrefImplicitLod:
234:         case SPIRV::OpImageFetch:
235:         case SPIRV::OpImageGather:
236:         case SPIRV::OpImageDrefGather:
237:         case SPIRV::OpImageRead:
238:         case SPIRV::OpImageWrite:
239:         case SPIRV::OpImageSparseSampleImplicitLod:
240:         case SPIRV::OpImageSparseSampleDrefImplicitLod:
```
- EN: This range implements operational logic in helpers such as printRemainingVariableOps, getNumOperands, getOperand, llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 printRemainingVariableOps、getNumOperands、getOperand、llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 241-300
```cpp
241:         case SPIRV::OpImageSparseSampleProjImplicitLod:
242:         case SPIRV::OpImageSparseSampleProjDrefImplicitLod:
243:         case SPIRV::OpImageSparseFetch:
244:         case SPIRV::OpImageSparseGather:
245:         case SPIRV::OpImageSparseDrefGather:
246:         case SPIRV::OpImageSparseRead:
247:         case SPIRV::OpImageSampleFootprintNV:
248:           OS << ' ';
249:           printSymbolicOperand<OperandCategory::ImageOperandOperand>(
250:               MI, FirstVariableIndex, OS);
251:           printRemainingVariableOps(MI, NumFixedOps + 1, OS);
252:           break;
253:         case SPIRV::OpCopyMemory:
254:         case SPIRV::OpCopyMemorySized: {
255:           const unsigned NumOps = MI->getNumOperands();
256:           for (unsigned i = NumFixedOps; i < NumOps; ++i) {
257:             OS << ' ';
258:             printSymbolicOperand<OperandCategory::MemoryOperandOperand>(MI, i,
259:                                                                         OS);
260:             if (MI->getOperand(i).getImm() & MemoryOperand::Aligned) {
261:               assert(i + 1 < NumOps && "Missing alignment operand");
262:               OS << ' ';
263:               printOperand(MI, i + 1, OS);
264:               i += 1;
265:             }
266:           }
267:           break;
268:         }
269:         case SPIRV::OpConstantI:
270:         case SPIRV::OpConstantF:
271:           // The last fixed operand along with any variadic operands that follow
272:           // are part of the variable value.
273:           assert(NumFixedOps > 0 && "Expected at least one fixed operand");
274:           printOpConstantVarOps(MI, NumFixedOps - 1, OS);
275:           break;
276:         case SPIRV::OpCooperativeMatrixMulAddKHR: {
277:           const unsigned NumOps = MI->getNumOperands();
278:           if (NumFixedOps == NumOps)
279:             break;
280:
281:           OS << ' ';
282:           const unsigned MulAddOp = MI->getOperand(FirstVariableIndex).getImm();
283:           if (MulAddOp == 0) {
284:             printSymbolicOperand<
285:                 OperandCategory::CooperativeMatrixOperandsOperand>(
286:                 MI, FirstVariableIndex, OS);
287:           } else {
288:             std::string Buffer;
289:             for (unsigned Mask = 0x1;
290:                  Mask != SPIRV::CooperativeMatrixOperands::
291:                              MatrixResultBFloat16ComponentsINTEL;
292:                  Mask <<= 1) {
293:               if (MulAddOp & Mask) {
294:                 if (!Buffer.empty())
295:                   Buffer += '|';
296:                 Buffer += getSymbolicOperandMnemonic(
297:                     OperandCategory::CooperativeMatrixOperandsOperand, Mask);
298:               }
299:             }
300:             OS << Buffer;
```
- EN: This range implements operational logic in helpers such as printRemainingVariableOps, getNumOperands, assert, printOperand, translating backend policy into executable code.
- CN: 这一段实现了 printRemainingVariableOps、getNumOperands、assert、printOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 301-360
```cpp
301:           }
302:           break;
303:         }
304:         case SPIRV::OpSubgroupMatrixMultiplyAccumulateINTEL: {
305:           const unsigned NumOps = MI->getNumOperands();
306:           if (NumFixedOps >= NumOps)
307:             break;
308:           OS << ' ';
309:           const unsigned Flags = MI->getOperand(NumOps - 1).getImm();
310:           if (Flags == 0) {
311:             printSymbolicOperand<
312:                 OperandCategory::MatrixMultiplyAccumulateOperandsOperand>(
313:                 MI, NumOps - 1, OS);
314:           } else {
315:             std::string Buffer;
316:             for (unsigned Mask = 0x1;
317:                  Mask <= SPIRV::MatrixMultiplyAccumulateOperands::
318:                              MatrixBPackedBFloat16INTEL;
319:                  Mask <<= 1) {
320:               if (Flags & Mask) {
321:                 if (!Buffer.empty())
322:                   Buffer += '|';
323:                 Buffer += getSymbolicOperandMnemonic(
324:                     OperandCategory::MatrixMultiplyAccumulateOperandsOperand,
325:                     Mask);
326:               }
327:             }
328:             OS << Buffer;
329:           }
330:           break;
331:         }
332:         case SPIRV::OpSDot:
333:         case SPIRV::OpUDot:
334:         case SPIRV::OpSUDot:
335:         case SPIRV::OpSDotAccSat:
336:         case SPIRV::OpUDotAccSat:
337:         case SPIRV::OpSUDotAccSat: {
338:           const unsigned NumOps = MI->getNumOperands();
339:           if (NumOps > NumFixedOps) {
340:             OS << ' ';
341:             printSymbolicOperand<OperandCategory::PackedVectorFormatsOperand>(
342:                 MI, NumOps - 1, OS);
343:             break;
344:           }
345:           break;
346:         }
347:         case SPIRV::OpPredicatedLoadINTEL:
348:         case SPIRV::OpPredicatedStoreINTEL: {
349:           const unsigned NumOps = MI->getNumOperands();
350:           if (NumOps > NumFixedOps) {
351:             OS << ' ';
352:             printSymbolicOperand<OperandCategory::MemoryOperandOperand>(
353:                 MI, NumOps - 1, OS);
354:             break;
355:           }
356:           break;
357:         }
358:         default:
359:           printRemainingVariableOps(MI, NumFixedOps, OS);
360:           break;
```
- EN: This range implements operational logic in helpers such as getNumOperands, getOperand, printRemainingVariableOps, translating backend policy into executable code.
- CN: 这一段实现了 getNumOperands、getOperand、printRemainingVariableOps 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-420
```cpp
361:         }
362:       }
363:     }
364:   }
365:
366:   printAnnotation(OS, Annot);
367: }
368:
369: void SPIRVInstPrinter::printOpExtInst(const MCInst *MI, raw_ostream &O) {
370:   // The fixed operands have already been printed, so just need to decide what
371:   // type of ExtInst operands to print based on the instruction set and number.
372:   const MCInstrDesc &MCDesc = MII.get(MI->getOpcode());
373:   unsigned NumFixedOps = MCDesc.getNumOperands();
374:   const auto NumOps = MI->getNumOperands();
375:   if (NumOps == NumFixedOps)
376:     return;
377:
378:   O << ' ';
379:
380:   // TODO: implement special printing for OpenCLExtInst::vstor*.
381:   printRemainingVariableOps(MI, NumFixedOps, O, true);
382: }
383:
384: void SPIRVInstPrinter::printOpDecorate(const MCInst *MI, raw_ostream &O) {
385:   // The fixed operands have already been printed, so just need to decide what
386:   // type of decoration operands to print based on the Decoration type.
387:   const MCInstrDesc &MCDesc = MII.get(MI->getOpcode());
388:   unsigned NumFixedOps = MCDesc.getNumOperands();
389:
390:   if (NumFixedOps != MI->getNumOperands()) {
391:     auto DecOp = MI->getOperand(NumFixedOps - 1);
392:     auto Dec = static_cast<Decoration::Decoration>(DecOp.getImm());
393:
394:     O << ' ';
395:
396:     switch (Dec) {
397:     case Decoration::BuiltIn:
398:       printSymbolicOperand<OperandCategory::BuiltInOperand>(MI, NumFixedOps, O);
399:       break;
400:     case Decoration::UniformId:
401:       printSymbolicOperand<OperandCategory::ScopeOperand>(MI, NumFixedOps, O);
402:       break;
403:     case Decoration::FuncParamAttr:
404:       printSymbolicOperand<OperandCategory::FunctionParameterAttributeOperand>(
405:           MI, NumFixedOps, O);
406:       break;
407:     case Decoration::FPRoundingMode:
408:       printSymbolicOperand<OperandCategory::FPRoundingModeOperand>(
409:           MI, NumFixedOps, O);
410:       break;
411:     case Decoration::FPFastMathMode:
412:       printSymbolicOperand<OperandCategory::FPFastMathModeOperand>(
413:           MI, NumFixedOps, O);
414:       break;
415:     case Decoration::LinkageAttributes:
416:     case Decoration::UserSemantic:
417:       printStringImm(MI, NumFixedOps, O);
418:       break;
419:     case Decoration::HostAccessINTEL:
420:       printOperand(MI, NumFixedOps, O);
```
- EN: This range implements operational logic in helpers such as printAnnotation, SPIRVInstPrinter::printOpExtInst, get, getNumOperands, translating backend policy into executable code.
- CN: 这一段实现了 printAnnotation、SPIRVInstPrinter::printOpExtInst、get、getNumOperands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 421-480
```cpp
421:       if (NumFixedOps + 1 < MI->getNumOperands()) {
422:         O << ' ';
423:         printStringImm(MI, NumFixedOps + 1, O);
424:       }
425:       break;
426:     default:
427:       printRemainingVariableOps(MI, NumFixedOps, O, true);
428:       break;
429:     }
430:   }
431: }
432:
433: void SPIRVInstPrinter::printUnknownType(const MCInst *MI, raw_ostream &O) {
434:   const auto EnumOperand = MI->getOperand(1);
435:   assert(EnumOperand.isImm() &&
436:          "second operand of UNKNOWN_type must be opcode!");
437:
438:   const auto Enumerant = EnumOperand.getImm();
439:   const auto NumOps = MI->getNumOperands();
440:
441:   // Print the opcode using the spirv-as unknown opcode syntax
442:   O << "OpUnknown(" << Enumerant << ", " << NumOps << ") ";
443:
444:   // The result ID must be printed after the opcode when using this syntax
445:   printOperand(MI, 0, O);
446:
447:   O << " ";
448:
449:   const MCInstrDesc &MCDesc = MII.get(MI->getOpcode());
450:   unsigned NumFixedOps = MCDesc.getNumOperands();
451:   if (NumOps == NumFixedOps)
452:     return;
453:
454:   // Print the rest of the operands
455:   printRemainingVariableOps(MI, NumFixedOps, O, true);
456: }
457:
458: void SPIRVInstPrinter::printOperand(const MCInst *MI, unsigned OpNo,
459:                                     raw_ostream &O) {
460:   if (OpNo < MI->getNumOperands()) {
461:     const MCOperand &Op = MI->getOperand(OpNo);
462:     if (Op.isReg())
463:       O << '%' << (getIDFromRegister(Op.getReg().id()) + 1);
464:     else if (Op.isImm()) {
465:       int64_t Imm = Op.getImm();
466:       // For OpVectorShuffle:
467:       // A Component literal may also be FFFFFFFF, which means the corresponding
468:       // result component has no source and is undefined.
469:       // LLVM representation of poison/undef becomes -1 when lowered to MI.
470:       if (MI->getOpcode() == SPIRV::OpVectorShuffle && Imm == -1)
471:         O << "0xFFFFFFFF";
472:       else
473:         O << formatImm(Imm);
474:     } else if (Op.isDFPImm())
475:       O << formatImm((double)Op.getDFPImm());
476:     else if (Op.isExpr())
477:       MAI.printExpr(O, *Op.getExpr());
478:     else
479:       llvm_unreachable("Unexpected operand type");
480:   }
```
- EN: This range implements operational logic in helpers such as printStringImm, printRemainingVariableOps, SPIRVInstPrinter::printUnknownType, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 printStringImm、printRemainingVariableOps、SPIRVInstPrinter::printUnknownType、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 481-536
```cpp
481: }
482:
483: void SPIRVInstPrinter::printStringImm(const MCInst *MI, unsigned OpNo,
484:                                       raw_ostream &O) {
485:   const unsigned NumOps = MI->getNumOperands();
486:   unsigned StrStartIndex = OpNo;
487:   while (StrStartIndex < NumOps) {
488:     if (MI->getOperand(StrStartIndex).isReg())
489:       break;
490:
491:     std::string Str = getSPIRVStringOperand(*MI, StrStartIndex);
492:     if (StrStartIndex != OpNo)
493:       O << ' '; // Add a space if we're starting a new string/argument.
494:     O << '"';
495:     for (char c : Str) {
496:       // Escape ", \n characters (might break for complex UTF-8).
497:       if (c == '\n') {
498:         O.write("\\n", 2);
499:       } else {
500:         if (c == '"')
501:           O.write('\\');
502:         O.write(c);
503:       }
504:     }
505:     O << '"';
506:
507:     unsigned numOpsInString = (Str.size() / 4) + 1;
508:     StrStartIndex += numOpsInString;
509:
510:     // Check for final Op of "OpDecorate %x %stringImm %linkageAttribute".
511:     if (MI->getOpcode() == SPIRV::OpDecorate &&
512:         MI->getOperand(1).getImm() ==
513:             static_cast<unsigned>(Decoration::LinkageAttributes)) {
514:       O << ' ';
515:       printSymbolicOperand<OperandCategory::LinkageTypeOperand>(
516:           MI, StrStartIndex, O);
517:       break;
518:     }
519:   }
520: }
521:
522: void SPIRVInstPrinter::printExtension(const MCInst *MI, unsigned OpNo,
523:                                       raw_ostream &O) {
524:   auto SetReg = MI->getOperand(2).getReg();
525:   auto Set = ExtInstSetIDs[SetReg];
526:   auto Op = MI->getOperand(OpNo).getImm();
527:   O << getExtInstName(Set, Op);
528: }
529:
530: template <OperandCategory::OperandCategory category>
531: void SPIRVInstPrinter::printSymbolicOperand(const MCInst *MI, unsigned OpNo,
532:                                             raw_ostream &O) {
533:   if (OpNo < MI->getNumOperands()) {
534:     O << getSymbolicOperandMnemonic(category, MI->getOperand(OpNo).getImm());
535:   }
536: }
```
- EN: This range implements operational logic in helpers such as getNumOperands, getSPIRVStringOperand, write, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 getNumOperands、getSPIRVStringOperand、write、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: SPIR-V backend code must preserve rich type information and encode capabilities/extensions explicitly.
  - CN: SPIR-V 后端需要保留丰富的类型信息，并显式编码 capability 与 extension。
- EN: Key symbols in this file include getNumOperands, printOperand, getOperand, at, Val, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 getNumOperands, printOperand, getOperand, at, Val，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `SPIRVInstPrinter.h`
  - `SPIRV.h`
  - `SPIRVBaseInfo.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APFloat.h`
  - `llvm/MC/MCAsmInfo.h`
  - `llvm/MC/MCExpr.h`
  - `llvm/MC/MCInst.h`
  - `llvm/MC/MCInstrInfo.h`
  - `llvm/MC/MCSymbol.h`
  - `llvm/Support/ErrorHandling.h`
- System/standard headers / 系统或标准头文件:
  - `SPIRVGenAsmWriter.inc`
