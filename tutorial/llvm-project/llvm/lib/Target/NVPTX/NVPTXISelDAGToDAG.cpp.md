# NVPTXISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXISelDAGToDAG.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file defines an instruction selector for the NVPTX target.
- 目的（中文）: 该文件实现目标相关的指令选择逻辑，把 LLVM 或机器 IR 映射到后端可接受的表示。
- Language: C++ source

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-90
```cpp
 1: //===-- NVPTXISelDAGToDAG.cpp - A dag to dag inst selector for NVPTX ------===//
 2: //
 3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 4: // See https://llvm.org/LICENSE.txt for license information.
 5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 6: //
 7: //===----------------------------------------------------------------------===//
 8: //
 9: // This file defines an instruction selector for the NVPTX target.
10: //
11: //===----------------------------------------------------------------------===//
12:
13: #include "NVPTXISelDAGToDAG.h"
14: #include "NVPTX.h"
15: #include "NVPTXUtilities.h"
16: #include "llvm/ADT/APInt.h"
17: #include "llvm/Analysis/ValueTracking.h"
18: #include "llvm/CodeGen/ISDOpcodes.h"
19: #include "llvm/CodeGen/MachineJumpTableInfo.h"
20: #include "llvm/CodeGen/SelectionDAG.h"
21: #include "llvm/CodeGen/SelectionDAGNodes.h"
22: #include "llvm/IR/GlobalValue.h"
23: #include "llvm/IR/Instructions.h"
24: #include "llvm/IR/IntrinsicsNVPTX.h"
25: #include "llvm/IR/NVVMIntrinsicUtils.h"
26: #include "llvm/Support/AtomicOrdering.h"
27: #include "llvm/Support/CommandLine.h"
28: #include "llvm/Support/ErrorHandling.h"
29: #include "llvm/Support/FormatVariadic.h"
30: #include "llvm/Support/MathExtras.h"
31: #include <optional>
32:
33: using namespace llvm;
34:
35: #define DEBUG_TYPE "nvptx-isel"
36: #define PASS_NAME "NVPTX DAG->DAG Pattern Instruction Selection"
37:
38: static cl::opt<bool>
39:     EnableRsqrtOpt("nvptx-rsqrt-approx-opt", cl::init(true), cl::Hidden,
40:                    cl::desc("Enable reciprocal sqrt optimization"));
41:
42: // FIXME: This is a WAR to recover lost performance from #155024.
43: // We still need to investigate the regression and find a more permanent
44: // solution.
45: static cl::opt<bool> EnableMADWide("nvptx-mad-wide-opt", cl::init(false),
46:                                    cl::Hidden,
47:                                    cl::desc("Enable MAD wide optimization"));
48:
49: /// createNVPTXISelDag - This pass converts a legalized DAG into a
50: /// NVPTX-specific DAG, ready for instruction scheduling.
51: FunctionPass *llvm::createNVPTXISelDag(NVPTXTargetMachine &TM,
52:                                        llvm::CodeGenOptLevel OptLevel) {
53:   return new NVPTXDAGToDAGISelLegacy(TM, OptLevel);
54: }
55:
56: NVPTXDAGToDAGISelLegacy::NVPTXDAGToDAGISelLegacy(NVPTXTargetMachine &tm,
57:                                                  CodeGenOptLevel OptLevel)
58:     : SelectionDAGISelLegacy(
59:           ID, std::make_unique<NVPTXDAGToDAGISel>(tm, OptLevel)) {}
60:
61: char NVPTXDAGToDAGISelLegacy::ID = 0;
62:
63: INITIALIZE_PASS(NVPTXDAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false, false)
64:
65: NVPTXDAGToDAGISel::NVPTXDAGToDAGISel(NVPTXTargetMachine &tm,
66:                                      CodeGenOptLevel OptLevel)
67:     : SelectionDAGISel(tm, OptLevel), TM(tm) {}
68:
69: bool NVPTXDAGToDAGISel::runOnMachineFunction(MachineFunction &MF) {
70:   Subtarget = &MF.getSubtarget<NVPTXSubtarget>();
71:   Scopes = NVPTXScopes(MF.getFunction().getContext());
72:   return SelectionDAGISel::runOnMachineFunction(MF);
73: }
74:
75: NVPTX::DivPrecisionLevel
76: NVPTXDAGToDAGISel::getDivF32Level(const SDNode *N) const {
77:   return Subtarget->getTargetLowering()->getDivF32Level(*MF, *N);
78: }
79:
80: bool NVPTXDAGToDAGISel::usePrecSqrtF32(const SDNode *N) const {
81:   return Subtarget->getTargetLowering()->usePrecSqrtF32(N);
82: }
83:
84: bool NVPTXDAGToDAGISel::useF32FTZ() const {
85:   return Subtarget->getTargetLowering()->useF32FTZ(*MF);
86: }
87:
88: bool NVPTXDAGToDAGISel::allowFMA() const {
89:   const NVPTXTargetLowering *TL = Subtarget->getTargetLowering();
90:   return TL->allowFMA(*MF, OptLevel);
```
- EN: This range defines command-line tuning knobs that influence backend lowering, code generation, or diagnostics.
- CN: 这一段定义命令行调优选项，用来影响后端的降级、代码生成或诊断行为。

### Lines 91-180
```cpp
 91: }
 92:
 93: bool NVPTXDAGToDAGISel::doRsqrtOpt() const { return EnableRsqrtOpt; }
 94:
 95: bool NVPTXDAGToDAGISel::doMADWideOpt() const { return EnableMADWide; }
 96:
 97: /// Select - Select instructions not customized! Used for
 98: /// expanded, promoted and normal instructions.
 99: void NVPTXDAGToDAGISel::Select(SDNode *N) {
100:
101:   if (N->isMachineOpcode()) {
102:     N->setNodeId(-1);
103:     return; // Already selected.
104:   }
105:
106:   switch (N->getOpcode()) {
107:   case ISD::LOAD:
108:   case ISD::ATOMIC_LOAD:
109:   case NVPTXISD::MLoad:
110:     if (tryLoad(N))
111:       return;
112:     break;
113:   case ISD::STORE:
114:   case ISD::ATOMIC_STORE:
115:     if (tryStore(N))
116:       return;
117:     break;
118:   case ISD::ATOMIC_FENCE:
119:     if (tryFence(N))
120:       return;
121:     break;
122:   case NVPTXISD::UNPACK_VECTOR:
123:     tryUNPACK_VECTOR(N);
124:     return;
125:   case ISD::EXTRACT_VECTOR_ELT:
126:     if (tryEXTRACT_VECTOR_ELEMENT(N))
127:       return;
128:     break;
129:   case NVPTXISD::SETP_F16X2:
130:     SelectSETP_F16X2(N);
131:     return;
132:   case NVPTXISD::SETP_BF16X2:
133:     SelectSETP_BF16X2(N);
134:     return;
135:   case NVPTXISD::LoadV2:
136:   case NVPTXISD::LoadV4:
137:   case NVPTXISD::LoadV8:
138:     if (tryLoadVector(N))
139:       return;
140:     break;
141:   case NVPTXISD::LDUV2:
142:   case NVPTXISD::LDUV4:
143:     if (tryLDU(N))
144:       return;
145:     break;
146:   case NVPTXISD::StoreV2:
147:   case NVPTXISD::StoreV4:
148:   case NVPTXISD::StoreV8:
149:     if (tryStoreVector(N))
150:       return;
151:     break;
152:   case ISD::INTRINSIC_W_CHAIN:
153:     if (tryIntrinsicChain(N))
154:       return;
155:     break;
156:   case ISD::INTRINSIC_VOID:
157:     if (tryIntrinsicVoid(N))
158:       return;
159:     break;
160:   case ISD::AND:
161:   case ISD::SRA:
162:   case ISD::SRL:
163:     // Try to select BFE
164:     if (tryBFE(N))
165:       return;
166:     break;
167:   case ISD::ADDRSPACECAST:
168:     SelectAddrSpaceCast(N);
169:     return;
170:   case ISD::CopyToReg: {
171:     if (N->getOperand(1).getValueType() == MVT::i128) {
172:       SelectV2I64toI128(N);
173:       return;
174:     }
175:     break;
176:   }
177:   case ISD::CopyFromReg: {
178:     if (N->getOperand(1).getValueType() == MVT::i128) {
179:       SelectI128toV2I64(N);
180:       return;
```
- EN: This range implements operational logic in helpers such as NVPTXDAGToDAGISel::doRsqrtOpt, NVPTXDAGToDAGISel::doMADWideOpt, NVPTXDAGToDAGISel::Select, setNodeId, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXDAGToDAGISel::doRsqrtOpt、NVPTXDAGToDAGISel::doMADWideOpt、NVPTXDAGToDAGISel::Select、setNodeId 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 181-270
```cpp
181:     }
182:     break;
183:   }
184:   case NVPTXISD::ATOMIC_CMP_SWAP_B128:
185:   case NVPTXISD::ATOMIC_SWAP_B128:
186:     selectAtomicSwap128(N);
187:     return;
188:   case ISD::FADD:
189:   case ISD::FMUL:
190:   case ISD::FSUB:
191:     if (tryBF16ArithToFMA(N))
192:       return;
193:     break;
194:   case ISD::BR_JT:
195:     return selectBR_JT(N);
196:   default:
197:     break;
198:   }
199:   SelectCode(N);
200: }
201:
202: #define TCGEN05_LD_OPCODE(SHAPE, NUM)                                          \
203:   (enablePack ? NVPTX::TCGEN05_LD_##SHAPE##_##NUM##_PACK                       \
204:               : NVPTX::TCGEN05_LD_##SHAPE##_##NUM)
205:
206: static unsigned getTcgen05LdOpcode(unsigned IID, bool enablePack) {
207:   switch (IID) {
208:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x1:
209:     return TCGEN05_LD_OPCODE(16x64b, x1);
210:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x2:
211:     return TCGEN05_LD_OPCODE(16x64b, x2);
212:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x4:
213:     return TCGEN05_LD_OPCODE(16x64b, x4);
214:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x8:
215:     return TCGEN05_LD_OPCODE(16x64b, x8);
216:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x16:
217:     return TCGEN05_LD_OPCODE(16x64b, x16);
218:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x32:
219:     return TCGEN05_LD_OPCODE(16x64b, x32);
220:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x64:
221:     return TCGEN05_LD_OPCODE(16x64b, x64);
222:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x128:
223:     return TCGEN05_LD_OPCODE(16x64b, x128);
224:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x1:
225:     return TCGEN05_LD_OPCODE(16x128b, x1);
226:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x2:
227:     return TCGEN05_LD_OPCODE(16x128b, x2);
228:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x4:
229:     return TCGEN05_LD_OPCODE(16x128b, x4);
230:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x8:
231:     return TCGEN05_LD_OPCODE(16x128b, x8);
232:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x16:
233:     return TCGEN05_LD_OPCODE(16x128b, x16);
234:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x32:
235:     return TCGEN05_LD_OPCODE(16x128b, x32);
236:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x64:
237:     return TCGEN05_LD_OPCODE(16x128b, x64);
238:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x1:
239:     return TCGEN05_LD_OPCODE(16x256b, x1);
240:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x2:
241:     return TCGEN05_LD_OPCODE(16x256b, x2);
242:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x4:
243:     return TCGEN05_LD_OPCODE(16x256b, x4);
244:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x8:
245:     return TCGEN05_LD_OPCODE(16x256b, x8);
246:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x16:
247:     return TCGEN05_LD_OPCODE(16x256b, x16);
248:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x32:
249:     return TCGEN05_LD_OPCODE(16x256b, x32);
250:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x1:
251:     return TCGEN05_LD_OPCODE(16x32bx2, x1);
252:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x2:
253:     return TCGEN05_LD_OPCODE(16x32bx2, x2);
254:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x4:
255:     return TCGEN05_LD_OPCODE(16x32bx2, x4);
256:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x8:
257:     return TCGEN05_LD_OPCODE(16x32bx2, x8);
258:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x16:
259:     return TCGEN05_LD_OPCODE(16x32bx2, x16);
260:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x32:
261:     return TCGEN05_LD_OPCODE(16x32bx2, x32);
262:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x64:
263:     return TCGEN05_LD_OPCODE(16x32bx2, x64);
264:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x128:
265:     return TCGEN05_LD_OPCODE(16x32bx2, x128);
266:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x1:
267:     return TCGEN05_LD_OPCODE(32x32b, x1);
268:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x2:
269:     return TCGEN05_LD_OPCODE(32x32b, x2);
270:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x4:
```
- EN: This range implements operational logic in helpers such as selectAtomicSwap128, selectBR_JT, SelectCode, getTcgen05LdOpcode, translating backend policy into executable code.
- CN: 这一段实现了 selectAtomicSwap128、selectBR_JT、SelectCode、getTcgen05LdOpcode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 271-360
```cpp
271:     return TCGEN05_LD_OPCODE(32x32b, x4);
272:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x8:
273:     return TCGEN05_LD_OPCODE(32x32b, x8);
274:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x16:
275:     return TCGEN05_LD_OPCODE(32x32b, x16);
276:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x32:
277:     return TCGEN05_LD_OPCODE(32x32b, x32);
278:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x64:
279:     return TCGEN05_LD_OPCODE(32x32b, x64);
280:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x128:
281:     return TCGEN05_LD_OPCODE(32x32b, x128);
282:   }
283:   llvm_unreachable("unhandled tcgen05.ld lowering");
284: }
285:
286: void NVPTXDAGToDAGISel::SelectTcgen05Ld(SDNode *N, bool hasOffset) {
287:   if (!Subtarget->hasTcgen05InstSupport())
288:     report_fatal_error(
289:         "tcgen05.ld is not supported on this architecture variant");
290:
291:   SDLoc DL(N);
292:   unsigned IID = cast<ConstantSDNode>(N->getOperand(1))->getZExtValue();
293:
294:   if (hasOffset) {
295:     bool enablePack = cast<ConstantSDNode>(N->getOperand(4))->getZExtValue();
296:     auto OffsetNode = CurDAG->getTargetConstant(
297:         cast<ConstantSDNode>(N->getOperand(3))->getZExtValue(), DL, MVT::i32);
298:     ReplaceNode(N, CurDAG->getMachineNode(
299:                        getTcgen05LdOpcode(IID, enablePack), DL, N->getVTList(),
300:                        {N->getOperand(2), OffsetNode, N->getOperand(0)}));
301:   } else {
302:     bool enablePack = cast<ConstantSDNode>(N->getOperand(3))->getZExtValue();
303:     ReplaceNode(N, CurDAG->getMachineNode(
304:                        getTcgen05LdOpcode(IID, enablePack), DL, N->getVTList(),
305:                        {N->getOperand(2), N->getOperand(0)}));
306:   }
307: }
308:
309: bool NVPTXDAGToDAGISel::tryIntrinsicChain(SDNode *N) {
310:   unsigned IID = N->getConstantOperandVal(1);
311:   switch (IID) {
312:   default:
313:     return false;
314:   case Intrinsic::nvvm_ldu_global_f:
315:   case Intrinsic::nvvm_ldu_global_i:
316:   case Intrinsic::nvvm_ldu_global_p:
317:     return tryLDU(N);
318:
319:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x1:
320:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x2:
321:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x4:
322:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x8:
323:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x16:
324:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x32:
325:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x64:
326:   case Intrinsic::nvvm_tcgen05_ld_16x64b_x128:
327:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x1:
328:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x2:
329:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x4:
330:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x16:
331:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x32:
332:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x64:
333:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x1:
334:   case Intrinsic::nvvm_tcgen05_ld_16x128b_x8:
335:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x2:
336:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x4:
337:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x8:
338:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x16:
339:   case Intrinsic::nvvm_tcgen05_ld_16x256b_x32:
340:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x1:
341:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x2:
342:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x4:
343:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x8:
344:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x16:
345:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x32:
346:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x64:
347:   case Intrinsic::nvvm_tcgen05_ld_32x32b_x128: {
348:     SelectTcgen05Ld(N);
349:     return true;
350:   }
351:
352:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x1:
353:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x2:
354:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x4:
355:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x8:
356:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x16:
357:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x32:
358:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x64:
359:   case Intrinsic::nvvm_tcgen05_ld_16x32bx2_x128: {
360:     SelectTcgen05Ld(N, /* hasOffset */ true);
```
- EN: This range implements operational logic in helpers such as TCGEN05_LD_OPCODE, llvm_unreachable, NVPTXDAGToDAGISel::SelectTcgen05Ld, DL, translating backend policy into executable code.
- CN: 这一段实现了 TCGEN05_LD_OPCODE、llvm_unreachable、NVPTXDAGToDAGISel::SelectTcgen05Ld、DL 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 361-450
```cpp
361:     return true;
362:   }
363:   }
364: }
365:
366: // Map ISD:CONDCODE value to appropriate CmpMode expected by
367: // NVPTXInstPrinter::printCmpMode()
368: SDValue NVPTXDAGToDAGISel::getPTXCmpMode(const CondCodeSDNode &CondCode) {
369:   using NVPTX::PTXCmpMode::CmpMode;
370:   const unsigned PTXCmpMode = [](ISD::CondCode CC) {
371:     switch (CC) {
372:     default:
373:       llvm_unreachable("Unexpected condition code.");
374:     case ISD::SETOEQ:
375:     case ISD::SETEQ:
376:       return CmpMode::EQ;
377:     case ISD::SETOGT:
378:     case ISD::SETGT:
379:       return CmpMode::GT;
380:     case ISD::SETOGE:
381:     case ISD::SETGE:
382:       return CmpMode::GE;
383:     case ISD::SETOLT:
384:     case ISD::SETLT:
385:       return CmpMode::LT;
386:     case ISD::SETOLE:
387:     case ISD::SETLE:
388:       return CmpMode::LE;
389:     case ISD::SETONE:
390:     case ISD::SETNE:
391:       return CmpMode::NE;
392:     case ISD::SETO:
393:       return CmpMode::NUM;
394:     case ISD::SETUO:
395:       return CmpMode::NotANumber;
396:     case ISD::SETUEQ:
397:       return CmpMode::EQU;
398:     case ISD::SETUGT:
399:       return CmpMode::GTU;
400:     case ISD::SETUGE:
401:       return CmpMode::GEU;
402:     case ISD::SETULT:
403:       return CmpMode::LTU;
404:     case ISD::SETULE:
405:       return CmpMode::LEU;
406:     case ISD::SETUNE:
407:       return CmpMode::NEU;
408:     }
409:   }(CondCode.get());
410:   return CurDAG->getTargetConstant(PTXCmpMode, SDLoc(), MVT::i32);
411: }
412:
413: bool NVPTXDAGToDAGISel::SelectSETP_F16X2(SDNode *N) {
414:   SDValue PTXCmpMode = getPTXCmpMode(*cast<CondCodeSDNode>(N->getOperand(2)));
415:   SDLoc DL(N);
416:   SDNode *SetP = CurDAG->getMachineNode(
417:       NVPTX::SETP_f16x2rr, DL, MVT::i1, MVT::i1,
418:       {N->getOperand(0), N->getOperand(1), PTXCmpMode,
419:        CurDAG->getTargetConstant(useF32FTZ() ? 1 : 0, DL, MVT::i1)});
420:   ReplaceNode(N, SetP);
421:   return true;
422: }
423:
424: bool NVPTXDAGToDAGISel::SelectSETP_BF16X2(SDNode *N) {
425:   SDValue PTXCmpMode = getPTXCmpMode(*cast<CondCodeSDNode>(N->getOperand(2)));
426:   SDLoc DL(N);
427:   SDNode *SetP = CurDAG->getMachineNode(
428:       NVPTX::SETP_bf16x2rr, DL, MVT::i1, MVT::i1,
429:       {N->getOperand(0), N->getOperand(1), PTXCmpMode,
430:        CurDAG->getTargetConstant(useF32FTZ() ? 1 : 0, DL, MVT::i1)});
431:   ReplaceNode(N, SetP);
432:   return true;
433: }
434:
435: bool NVPTXDAGToDAGISel::tryUNPACK_VECTOR(SDNode *N) {
436:   SDValue Vector = N->getOperand(0);
437:   MVT EltVT = N->getSimpleValueType(0);
438:
439:   MachineSDNode *N2 =
440:       CurDAG->getMachineNode(NVPTX::I64toV2I32, SDLoc(N), EltVT, EltVT, Vector);
441:
442:   ReplaceNode(N, N2);
443:   return true;
444: }
445:
446: // Find all instances of extract_vector_elt that use this v2f16 vector
447: // and coalesce them into a scattering move instruction.
448: bool NVPTXDAGToDAGISel::tryEXTRACT_VECTOR_ELEMENT(SDNode *N) {
449:   SDValue Vector = N->getOperand(0);
450:
```
- EN: This range implements operational logic in helpers such as NVPTXDAGToDAGISel::getPTXCmpMode, llvm_unreachable, get, getTargetConstant, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXDAGToDAGISel::getPTXCmpMode、llvm_unreachable、get、getTargetConstant 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 451-540
```cpp
451:   MVT VT = Vector.getSimpleValueType();
452:   if (!(NVPTX::isPackedVectorTy(VT) && VT.getVectorNumElements() == 2))
453:     return false;
454:
455:   unsigned Opcode;
456:   if (VT.is32BitVector())
457:     Opcode = NVPTX::I32toV2I16;
458:   else if (VT.is64BitVector())
459:     Opcode = NVPTX::I64toV2I32;
460:   else
461:     llvm_unreachable("Unhandled packed type");
462:
463:   // Find and record all uses of this vector that extract element 0 or 1.
464:   SmallVector<SDNode *, 4> E0, E1;
465:   for (auto *U : Vector.getNode()->users()) {
466:     if (U->getOpcode() != ISD::EXTRACT_VECTOR_ELT)
467:       continue;
468:     if (U->getOperand(0) != Vector)
469:       continue;
470:     if (const ConstantSDNode *IdxConst =
471:             dyn_cast<ConstantSDNode>(U->getOperand(1))) {
472:       if (IdxConst->getZExtValue() == 0)
473:         E0.push_back(U);
474:       else if (IdxConst->getZExtValue() == 1)
475:         E1.push_back(U);
476:       else
477:         llvm_unreachable("Invalid vector index.");
478:     }
479:   }
480:
481:   // There's no point scattering f16x2 if we only ever access one
482:   // element of it.
483:   if (E0.empty() || E1.empty())
484:     return false;
485:
486:   // Merge (EltTy extractelt(V, 0), EltTy extractelt(V,1))
487:   // into EltTy,EltTy Split[EltTy]x2(V)
488:   MVT EltVT = VT.getVectorElementType();
489:   SDNode *ScatterOp =
490:       CurDAG->getMachineNode(Opcode, SDLoc(N), EltVT, EltVT, Vector);
491:   for (auto *Node : E0)
492:     ReplaceUses(SDValue(Node, 0), SDValue(ScatterOp, 0));
493:   for (auto *Node : E1)
494:     ReplaceUses(SDValue(Node, 0), SDValue(ScatterOp, 1));
495:
496:   return true;
497: }
498:
499: NVPTX::AddressSpace NVPTXDAGToDAGISel::getAddrSpace(const MemSDNode *N) {
500:   auto AS =
501:       static_cast<NVPTX::AddressSpace>(N->getMemOperand()->getAddrSpace());
502:   switch (AS) {
503:   case NVPTX::AddressSpace::Generic:
504:   case NVPTX::AddressSpace::Global:
505:   case NVPTX::AddressSpace::Shared:
506:   case NVPTX::AddressSpace::Const:
507:   case NVPTX::AddressSpace::Local:
508:   case NVPTX::AddressSpace::SharedCluster:
509:   case NVPTX::AddressSpace::EntryParam:
510:   case NVPTX::AddressSpace::DeviceParam:
511:     return AS;
512:   }
513:   llvm_unreachable("Unexpected address space");
514: }
515:
516: NVPTX::Ordering NVPTXDAGToDAGISel::getMemOrder(const MemSDNode *N) const {
517:   // No "sem" orderings for SM/PTX versions which do not support memory ordering
518:   if (!Subtarget->hasMemoryOrdering())
519:     return NVPTX::Ordering::NotAtomic;
520:   auto Ordering = N->getMergedOrdering();
521:   switch (Ordering) {
522:   case AtomicOrdering::NotAtomic:
523:     return NVPTX::Ordering::NotAtomic;
524:   case AtomicOrdering::Unordered:
525:   case AtomicOrdering::Monotonic:
526:     return NVPTX::Ordering::Relaxed;
527:   case AtomicOrdering::Acquire:
528:     return NVPTX::Ordering::Acquire;
529:   case AtomicOrdering::Release:
530:     return NVPTX::Ordering::Release;
531:   case AtomicOrdering::AcquireRelease:
532:     return NVPTX::Ordering::AcquireRelease;
533:   case AtomicOrdering::SequentiallyConsistent:
534:     return NVPTX::Ordering::SequentiallyConsistent;
535:   }
536:   llvm_unreachable("Invalid atomic ordering");
537: }
538:
539: // Clusters contain exactly 1 block on targets without cluster support.
540: static NVPTX::Scope resolveScope(NVPTX::Scope S, const NVPTXSubtarget *T) {
```
- EN: This range implements operational logic in helpers such as getSimpleValueType, llvm_unreachable, getOperand, push_back, translating backend policy into executable code.
- CN: 这一段实现了 getSimpleValueType、llvm_unreachable、getOperand、push_back 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 541-630
```cpp
541:   if (S == NVPTX::Scope::Cluster && !T->hasClusters())
542:     return NVPTX::Scope::Block;
543:   return S;
544: }
545:
546: NVPTX::Scope NVPTXDAGToDAGISel::getAtomicScope(const MemSDNode *N) const {
547:   if (!Subtarget->hasAtomScope())
548:     return NVPTX::Scope::DefaultDevice;
549:   return resolveScope(Scopes[N->getSyncScopeID()], Subtarget);
550: }
551:
552: namespace {
553:
554: struct OperationOrderings {
555:   NVPTX::Ordering InstructionOrdering, FenceOrdering;
556:   OperationOrderings(NVPTX::Ordering IO = NVPTX::Ordering::NotAtomic,
557:                      NVPTX::Ordering FO = NVPTX::Ordering::NotAtomic)
558:       : InstructionOrdering(IO), FenceOrdering(FO) {}
559: };
560:
561: static OperationOrderings
562: getOperationOrderings(MemSDNode *N, const NVPTXSubtarget *Subtarget) {
563:   AtomicOrdering Ordering = N->getSuccessOrdering();
564:   auto CodeAddrSpace = NVPTXDAGToDAGISel::getAddrSpace(N);
565:
566:   bool HasMemoryOrdering = Subtarget->hasMemoryOrdering();
567:   bool HasRelaxedMMIO = Subtarget->hasRelaxedMMIO();
568:
569:   // clang-format off
570:
571:   // Lowering for Load/Store Operations (note: AcquireRelease Loads or Stores error).
572:   // Note: uses of Relaxed in the Atomic column of this table refer
573:   // to LLVM AtomicOrdering::Monotonic.
574:   //
575:   // | Atomic  | Volatile | Statespace         | PTX sm_60- | PTX sm_70+                   |
576:   // |---------|----------|--------------------|------------|------------------------------|
577:   // | No      | No       | All                | plain      | .weak                        |
578:   // | No      | Yes      | Generic,Shared,    | .volatile  | .volatile                    |
579:   // |         |          | Global [0]         |            |                              |
580:   // | No      | Yes      | Local,Const,Param  | plain [1]  | .weak [1]                    |
581:   // | Unorder | Yes/No   | All                | == Relaxed | == Relaxed                   |
582:   // | Relaxed | No       | Generic,Shared,    | .volatile  | <atomic sem>                 |
583:   // |         |          | Global [0]         |            |                              |
584:   // | Other   | No       | Generic,Shared,    | Error [2]  | <atomic sem>                 |
585:   // |         |          | Global [0]         |            |                              |
586:   // | Yes     | No       | Local,Const,Param  | plain [1]  | .weak [1]                    |
587:   // | Relaxed | Yes      | Generic,Shared [0] | .volatile  | .volatile                    |
588:   // | Relaxed | Yes      | Global [0]         | .volatile  | .mmio.relaxed.sys (PTX 8.2+) |
589:   // |         |          |                    |            |  or .volatile (PTX 8.1-)     |
590:   // | Relaxed | Yes      | Local,Const,Param  | plain [1]  | .weak [1]                    |
591:   // | Other   | Yes      | Generic, Shared,   | Error [2]  | <atomic sem> [3]             |
592:   // |         |          | / Global [0]       |            |                              |
593:
594:   // Lowering of CUDA C++ SequentiallyConsistent Operations and Fences to PTX
595:   // by following the ABI proven sound in:
596:   //   Lustig et al, A Formal Analysis of the NVIDIA PTX Memory Consistency Model, ASPLOS’19.
597:   //   https://dl.acm.org/doi/pdf/10.1145/3297858.3304043
598:   //
599:   // | CUDA C++ Atomic Operation or Atomic Fence            | PTX Atomic Operation or Fence |
600:   // |------------------------------------------------------|-------------------------------|
601:   // | cuda::atomic_thread_fence                            | fence.sc.<scope>;             |
602:   // |   (memory_order_seq_cst, cuda::thread_scope_<scope>) |                               |
603:   // |------------------------------------------------------|-------------------------------|
604:   // | cuda::atomic_load                                    | fence.sc.<scope>;             |
605:   // |   (memory_order_seq_cst, cuda::thread_scope_<scope>) | ld.acquire.<scope>;           |
606:   // |------------------------------------------------------|-------------------------------|  
607:   // | cuda::atomic_store                                   | fence.sc.<scope>;             |
608:   // |   (memory_order_seq_cst, cuda::thread_scope_<scope>) | st.release.<scope>;           |
609:   // |------------------------------------------------------|-------------------------------|
610:   // | cuda::atomic_fetch_<op>                              | fence.sc.<scope>;             |
611:   // |   (memory_order_seq_cst, cuda::thread_scope_<scope>) | atom.acq_rel.<scope>;         |
612:
613:   // clang-format on
614:
615:   // [0]: volatile and atomics are only supported on global or shared
616:   //      memory locations, accessed via generic/shared/global pointers.
617:   //      MMIO is only supported on global memory locations,
618:   //      accessed via generic/global pointers.
619:   // TODO: Implement MMIO access via generic pointer to global.
620:   //       Currently implemented for global pointers only.
621:
622:   // [1]: Lowering volatile/atomic operations to non-volatile/non-atomic
623:   //      PTX instructions fails to preserve their C++ side-effects.
624:   //
625:   //      Example (https://github.com/llvm/llvm-project/issues/62057):
626:   //
627:   //          void example() {
628:   //              std::atomic<bool> True = true;
629:   //              while (True.load(std::memory_order_relaxed));
630:   //          }
```
- EN: This range defines or declares important types such as NVPTXDAGToDAGISel::getAtomicScope, resolveScope, OperationOrderings, InstructionOrdering, shaping the data model used by NVPTXISelDAGToDAG.cpp.
- CN: 这一段定义或声明了 NVPTXDAGToDAGISel::getAtomicScope、resolveScope、OperationOrderings、InstructionOrdering 等关键类型，构成 NVPTXISelDAGToDAG.cpp 使用的数据模型。

### Lines 631-720
```cpp
631:   //
632:   //      A C++ program that calls "example" is well-defined: the infinite loop
633:   //      performs an atomic operation. By lowering volatile/atomics to
634:   //      "weak" memory operations, we are transforming the above into:
635:   //
636:   //          void undefined_behavior() {
637:   //              bool True = true;
638:   //              while (True);
639:   //          }
640:   //
641:   //      which exhibits undefined behavior in both C++ and PTX.
642:   //
643:   //      Calling "example" in CUDA C++ compiled for sm_60- exhibits undefined
644:   //      behavior due to lack of Independent Forward Progress. Lowering these
645:   //      to weak memory operations in sm_60- is therefore fine.
646:   //
647:   //      TODO: lower atomic and volatile operations to memory locations
648:   //      in local, const, and param to two PTX instructions in sm_70+:
649:   //        - the "weak" memory instruction we are currently lowering to, and
650:   //        - some other instruction that preserves the side-effect, e.g.,
651:   //          a dead dummy volatile load.
652:   if (CodeAddrSpace == NVPTX::AddressSpace::Local ||
653:       CodeAddrSpace == NVPTX::AddressSpace::Const ||
654:       CodeAddrSpace == NVPTX::AddressSpace::EntryParam ||
655:       CodeAddrSpace == NVPTX::AddressSpace::DeviceParam) {
656:     return NVPTX::Ordering::NotAtomic;
657:   }
658:
659:   // [2]: Atomics with Ordering different than Unordered or Relaxed are not
660:   //      supported on sm_60 and older; this includes volatile atomics.
661:   if (!(Ordering == AtomicOrdering::NotAtomic ||
662:         Ordering == AtomicOrdering::Unordered ||
663:         Ordering == AtomicOrdering::Monotonic) &&
664:       !HasMemoryOrdering) {
665:     report_fatal_error(
666:         formatv("PTX does not support \"atomic\" for orderings different than"
667:                 "\"NotAtomic\" or \"Monotonic\" for sm_60 or older, but order "
668:                 "is: \"{}\".",
669:                 toIRString(Ordering)));
670:   }
671:
672:   // [3]: TODO: these should eventually use .mmio<.atomic sem>; for now we drop
673:   // the volatile semantics and preserve the atomic ones.
674:
675:   // PTX volatile and PTX atomics are not available for statespace that differ
676:   // from .generic, .global, or .shared. The behavior of PTX volatile and PTX
677:   // atomics is undefined if the generic address does not refer to a .global or
678:   // .shared memory location.
679:   bool AddrGenericOrGlobalOrShared =
680:       (CodeAddrSpace == NVPTX::AddressSpace::Generic ||
681:        CodeAddrSpace == NVPTX::AddressSpace::Global ||
682:        CodeAddrSpace == NVPTX::AddressSpace::Shared ||
683:        CodeAddrSpace == NVPTX::AddressSpace::SharedCluster);
684:   if (!AddrGenericOrGlobalOrShared)
685:     return NVPTX::Ordering::NotAtomic;
686:
687:   bool UseRelaxedMMIO =
688:       HasRelaxedMMIO && CodeAddrSpace == NVPTX::AddressSpace::Global;
689:
690:   switch (Ordering) {
691:   case AtomicOrdering::NotAtomic:
692:     return N->isVolatile() ? NVPTX::Ordering::Volatile
693:                            : NVPTX::Ordering::NotAtomic;
694:   case AtomicOrdering::Unordered:
695:     // We lower unordered in the exact same way as 'monotonic' to respect
696:     // LLVM IR atomicity requirements.
697:   case AtomicOrdering::Monotonic:
698:     if (N->isVolatile())
699:       return UseRelaxedMMIO ? NVPTX::Ordering::RelaxedMMIO
700:                             : NVPTX::Ordering::Volatile;
701:     else
702:       return HasMemoryOrdering ? NVPTX::Ordering::Relaxed
703:                                : NVPTX::Ordering::Volatile;
704:   // case AtomicOrdering::Consume: // If LLVM ever provides this, lower it to
705:   // Acquire.
706:   case AtomicOrdering::Acquire:
707:     if (!N->readMem())
708:       report_fatal_error(
709:           formatv("PTX only supports Acquire Ordering on reads: {}",
710:                   N->getOperationName()));
711:     return NVPTX::Ordering::Acquire;
712:   case AtomicOrdering::Release:
713:     if (!N->writeMem())
714:       report_fatal_error(
715:           formatv("PTX only supports Release Ordering on writes: {}",
716:                   N->getOperationName()));
717:     return NVPTX::Ordering::Release;
718:   case AtomicOrdering::AcquireRelease: {
719:     report_fatal_error(
720:         formatv("NVPTX does not support AcquireRelease Ordering on "
```
- EN: This range implements operational logic in helpers such as toIRString, getOperationName, translating backend policy into executable code.
- CN: 这一段实现了 toIRString、getOperationName 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 721-810
```cpp
721:                 "read-modify-write "
722:                 "yet and PTX does not support it on loads or stores: {}",
723:                 N->getOperationName()));
724:   }
725:   case AtomicOrdering::SequentiallyConsistent: {
726:     // LLVM-IR SequentiallyConsistent atomics map to a two-instruction PTX
727:     // sequence including a "fence.sc.sco" and the memory instruction with an
728:     // Ordering that differs from "sc": acq, rel, or acq_rel, depending on
729:     // whether the memory operation is a read, write, or read-modify-write.
730:     //
731:     // This sets the ordering of the fence to SequentiallyConsistent, and
732:     // sets the corresponding ordering for the instruction.
733:     NVPTX::Ordering InstrOrder;
734:     if (N->readMem())
735:       InstrOrder = NVPTX::Ordering::Acquire;
736:     else if (N->writeMem())
737:       InstrOrder = NVPTX::Ordering::Release;
738:     else
739:       report_fatal_error(
740:           formatv("NVPTX does not support SequentiallyConsistent Ordering on "
741:                   "read-modify-writes yet: {}",
742:                   N->getOperationName()));
743:     return OperationOrderings(InstrOrder,
744:                               NVPTX::Ordering::SequentiallyConsistent);
745:   }
746:   }
747:   report_fatal_error(
748:       formatv("NVPTX backend does not support AtomicOrdering \"{}\" yet.",
749:               toIRString(Ordering)));
750: }
751:
752: } // namespace
753:
754: NVPTX::Scope NVPTXDAGToDAGISel::getOperationScope(MemSDNode *N,
755:                                                   NVPTX::Ordering O) const {
756:   switch (O) {
757:   case NVPTX::Ordering::NotAtomic:
758:   case NVPTX::Ordering::Volatile: // Non-atomic volatile operations
759:     // NVPTX uses Thread scope as the scope of non-atomic operations.
760:     return NVPTX::Scope::Thread;
761:   case NVPTX::Ordering::RelaxedMMIO:
762:     // RelaxedMMIO operations are always system scope.
763:     // If a RelaxedMMIO order was generated from an atomic volatile operation
764:     // with a smaller thread scope, we bump it here to system scope.
765:     return NVPTX::Scope::System;
766:   case NVPTX::Ordering::Relaxed:
767:   case NVPTX::Ordering::Acquire:
768:   case NVPTX::Ordering::Release:
769:   case NVPTX::Ordering::AcquireRelease:
770:   case NVPTX::Ordering::SequentiallyConsistent:
771:     auto S = Scopes[N->getSyncScopeID()];
772:
773:     S = resolveScope(S, Subtarget);
774:
775:     // If operation is volatile, then its scope is system.
776:     return N->isVolatile() ? NVPTX::Scope::System : S;
777:   }
778:   llvm_unreachable("unhandled ordering");
779: }
780:
781: static bool canLowerToLDG(const MemSDNode &N, const NVPTXSubtarget &Subtarget,
782:                           NVPTX::AddressSpace CodeAddrSpace) {
783:   // We use ldg (i.e. ld.global.nc) for invariant loads from the global address
784:   // space.
785:   return Subtarget.hasLDG() && CodeAddrSpace == NVPTX::AddressSpace::Global &&
786:          N.isInvariant();
787: }
788:
789: static unsigned int getFenceOp(NVPTX::Ordering O, NVPTX::Scope S,
790:                                NVPTXSubtarget const *T) {
791:   S = resolveScope(S, T);
792:
793:   // Fall back to .acq_rel if .acquire, .release is not supported.
794:   if (!T->hasSplitAcquireAndReleaseFences() &&
795:       (O == NVPTX::Ordering::Acquire || O == NVPTX::Ordering::Release))
796:     O = NVPTX::Ordering::AcquireRelease;
797:
798:   switch (O) {
799:   case NVPTX::Ordering::Acquire:
800:     switch (S) {
801:     case NVPTX::Scope::System:
802:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_acquire_sys
803:                                     : NVPTX::INT_MEMBAR_SYS;
804:     case NVPTX::Scope::Block:
805:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_acquire_cta
806:                                     : NVPTX::INT_MEMBAR_CTA;
807:     case NVPTX::Scope::Cluster:
808:       return NVPTX::atomic_thread_fence_acquire_cluster;
809:     case NVPTX::Scope::Device:
810:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_acquire_gpu
```
- EN: This range implements operational logic in helpers such as getOperationName, toIRString, resolveScope, llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 getOperationName、toIRString、resolveScope、llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 811-900
```cpp
811:                                     : NVPTX::INT_MEMBAR_GL;
812:     case NVPTX::Scope::Thread:
813:     case NVPTX::Scope::DefaultDevice:
814:       report_fatal_error(
815:           formatv("Unsupported scope \"{}\" for acquire/release/acq_rel fence.",
816:                   ScopeToString(S)));
817:     }
818:     break;
819:   case NVPTX::Ordering::Release:
820:     switch (S) {
821:     case NVPTX::Scope::System:
822:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_release_sys
823:                                     : NVPTX::INT_MEMBAR_SYS;
824:     case NVPTX::Scope::Block:
825:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_release_cta
826:                                     : NVPTX::INT_MEMBAR_CTA;
827:     case NVPTX::Scope::Cluster:
828:       return NVPTX::atomic_thread_fence_release_cluster;
829:     case NVPTX::Scope::Device:
830:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_release_gpu
831:                                     : NVPTX::INT_MEMBAR_GL;
832:     case NVPTX::Scope::Thread:
833:     case NVPTX::Scope::DefaultDevice:
834:       report_fatal_error(
835:           formatv("Unsupported scope \"{}\" for acquire/release/acq_rel fence.",
836:                   ScopeToString(S)));
837:     }
838:     break;
839:   case NVPTX::Ordering::AcquireRelease: {
840:     switch (S) {
841:     case NVPTX::Scope::System:
842:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_acq_rel_sys
843:                                     : NVPTX::INT_MEMBAR_SYS;
844:     case NVPTX::Scope::Block:
845:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_acq_rel_cta
846:                                     : NVPTX::INT_MEMBAR_CTA;
847:     case NVPTX::Scope::Cluster:
848:       return NVPTX::atomic_thread_fence_acq_rel_cluster;
849:     case NVPTX::Scope::Device:
850:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_acq_rel_gpu
851:                                     : NVPTX::INT_MEMBAR_GL;
852:     case NVPTX::Scope::Thread:
853:     case NVPTX::Scope::DefaultDevice:
854:       report_fatal_error(
855:           formatv("Unsupported scope \"{}\" for acquire/release/acq_rel fence.",
856:                   ScopeToString(S)));
857:     }
858:     break;
859:   }
860:   case NVPTX::Ordering::SequentiallyConsistent: {
861:     switch (S) {
862:     case NVPTX::Scope::System:
863:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_seq_cst_sys
864:                                     : NVPTX::INT_MEMBAR_SYS;
865:     case NVPTX::Scope::Block:
866:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_seq_cst_cta
867:                                     : NVPTX::INT_MEMBAR_CTA;
868:     case NVPTX::Scope::Cluster:
869:       return NVPTX::atomic_thread_fence_seq_cst_cluster;
870:     case NVPTX::Scope::Device:
871:       return T->hasMemoryOrdering() ? NVPTX::atomic_thread_fence_seq_cst_gpu
872:                                     : NVPTX::INT_MEMBAR_GL;
873:     case NVPTX::Scope::Thread:
874:     case NVPTX::Scope::DefaultDevice:
875:       report_fatal_error(formatv("Unsupported scope \"{}\" for seq_cst fence.",
876:                                  ScopeToString(S)));
877:     }
878:     break;
879:   }
880:   case NVPTX::Ordering::NotAtomic:
881:   case NVPTX::Ordering::Relaxed:
882:   case NVPTX::Ordering::Volatile:
883:   case NVPTX::Ordering::RelaxedMMIO:
884:     report_fatal_error(
885:         formatv("Unsupported \"{}\" ordering and \"{}\" scope for fence.",
886:                 OrderingToString(O), ScopeToString(S)));
887:   }
888:   llvm_unreachable("unhandled ordering");
889: }
890:
891: // Returns Memory Order and Scope of a memory instruction, and
892: // inserts any fence before the instruction that's required to
893: // implement its memory ordering.
894: std::pair<NVPTX::Ordering, NVPTX::Scope>
895: NVPTXDAGToDAGISel::insertMemoryInstructionFence(SDLoc DL, SDValue &Chain,
896:                                                 MemSDNode *N) {
897:   auto [InstructionOrdering, FenceOrdering] =
898:       getOperationOrderings(N, Subtarget);
899:   auto Scope = getOperationScope(N, InstructionOrdering);
900:
```
- EN: This range implements operational logic in helpers such as ScopeToString, OrderingToString, llvm_unreachable, getOperationOrderings, translating backend policy into executable code.
- CN: 这一段实现了 ScopeToString、OrderingToString、llvm_unreachable、getOperationOrderings 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 901-990
```cpp
901:   // Singlethread scope has no inter-thread synchronization requirements, so
902:   // the atomic operation is lowered as plain and the fence is skipped.
903:   // NotAtomic and Volatile operations naturally have Thread scope and must
904:   // preserve their ordering.
905:   if (Scope == NVPTX::Scope::Thread &&
906:       InstructionOrdering != NVPTX::Ordering::NotAtomic &&
907:       InstructionOrdering != NVPTX::Ordering::Volatile)
908:     return {NVPTX::Ordering::NotAtomic, Scope};
909:
910:   // If a fence is required before the operation, insert it:
911:   switch (NVPTX::Ordering(FenceOrdering)) {
912:   case NVPTX::Ordering::NotAtomic:
913:     break;
914:   case NVPTX::Ordering::SequentiallyConsistent: {
915:     auto Op = getFenceOp(FenceOrdering, Scope, Subtarget);
916:     Chain = SDValue(CurDAG->getMachineNode(Op, DL, MVT::Other, Chain), 0);
917:     break;
918:   }
919:   default:
920:     report_fatal_error(
921:         formatv("Unexpected fence ordering: \"{}\".",
922:                 OrderingToString(NVPTX::Ordering(FenceOrdering))));
923:   }
924:   return {InstructionOrdering, Scope};
925: }
926:
927: void NVPTXDAGToDAGISel::SelectAddrSpaceCast(SDNode *N) {
928:   SDValue Src = N->getOperand(0);
929:   AddrSpaceCastSDNode *CastN = cast<AddrSpaceCastSDNode>(N);
930:   unsigned SrcAddrSpace = CastN->getSrcAddressSpace();
931:   unsigned DstAddrSpace = CastN->getDestAddressSpace();
932:   SDLoc DL(N);
933:   assert(SrcAddrSpace != DstAddrSpace &&
934:          "addrspacecast must be between different address spaces");
935:
936:   if (DstAddrSpace == ADDRESS_SPACE_GENERIC) {
937:     // Specific to generic
938:
939:     if (TM.is64Bit() && TM.getPointerSizeInBits(SrcAddrSpace) == 32) {
940:       SDValue CvtNone =
941:           CurDAG->getTargetConstant(NVPTX::PTXCvtMode::NONE, DL, MVT::i32);
942:       SDNode *Cvt = CurDAG->getMachineNode(NVPTX::CVT_u64_u32, DL, MVT::i64,
943:                                            Src, CvtNone);
944:       Src = SDValue(Cvt, 0);
945:     }
946:
947:     unsigned Opc;
948:     switch (SrcAddrSpace) {
949:     default: report_fatal_error("Bad address space in addrspacecast");
950:     case ADDRESS_SPACE_GLOBAL:
951:       Opc = TM.is64Bit() ? NVPTX::cvta_global_64 : NVPTX::cvta_global;
952:       break;
953:     case ADDRESS_SPACE_SHARED:
954:       Opc = TM.is64Bit() ? NVPTX::cvta_shared_64 : NVPTX::cvta_shared;
955:       break;
956:     case ADDRESS_SPACE_SHARED_CLUSTER:
957:       if (!TM.is64Bit())
958:         report_fatal_error(
959:             "Shared cluster address space is only supported in 64-bit mode");
960:       Opc = NVPTX::cvta_shared_cluster_64;
961:       break;
962:     case ADDRESS_SPACE_CONST:
963:       Opc = TM.is64Bit() ? NVPTX::cvta_const_64 : NVPTX::cvta_const;
964:       break;
965:     case ADDRESS_SPACE_LOCAL:
966:       Opc = TM.is64Bit() ? NVPTX::cvta_local_64 : NVPTX::cvta_local;
967:       break;
968:     case ADDRESS_SPACE_ENTRY_PARAM:
969:       Opc = TM.is64Bit() ? NVPTX::cvta_param_64 : NVPTX::cvta_param;
970:       break;
971:     }
972:     ReplaceNode(N, CurDAG->getMachineNode(Opc, DL, N->getValueType(0), Src));
973:     return;
974:   } else {
975:     // Generic to specific
976:     if (SrcAddrSpace != 0)
977:       report_fatal_error("Cannot cast between two non-generic address spaces");
978:     unsigned Opc;
979:     switch (DstAddrSpace) {
980:     default: report_fatal_error("Bad address space in addrspacecast");
981:     case ADDRESS_SPACE_GLOBAL:
982:       Opc = TM.is64Bit() ? NVPTX::cvta_to_global_64 : NVPTX::cvta_to_global;
983:       break;
984:     case ADDRESS_SPACE_SHARED:
985:       Opc = TM.is64Bit() ? NVPTX::cvta_to_shared_64 : NVPTX::cvta_to_shared;
986:       break;
987:     case ADDRESS_SPACE_SHARED_CLUSTER:
988:       if (!TM.is64Bit())
989:         report_fatal_error(
990:             "Shared cluster address space is only supported in 64-bit mode");
```
- EN: This range implements operational logic in helpers such as getFenceOp, SDValue, OrderingToString, NVPTXDAGToDAGISel::SelectAddrSpaceCast, translating backend policy into executable code.
- CN: 这一段实现了 getFenceOp、SDValue、OrderingToString、NVPTXDAGToDAGISel::SelectAddrSpaceCast 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 991-1080
```cpp
 991:       Opc = NVPTX::cvta_to_shared_cluster_64;
 992:       break;
 993:     case ADDRESS_SPACE_CONST:
 994:       Opc = TM.is64Bit() ? NVPTX::cvta_to_const_64 : NVPTX::cvta_to_const;
 995:       break;
 996:     case ADDRESS_SPACE_LOCAL:
 997:       Opc = TM.is64Bit() ? NVPTX::cvta_to_local_64 : NVPTX::cvta_to_local;
 998:       break;
 999:     case ADDRESS_SPACE_ENTRY_PARAM:
1000:       Opc = TM.is64Bit() ? NVPTX::cvta_to_param_64 : NVPTX::cvta_to_param;
1001:       break;
1002:     }
1003:
1004:     SDNode *CVTA = CurDAG->getMachineNode(Opc, DL, N->getValueType(0), Src);
1005:     if (TM.is64Bit() && TM.getPointerSizeInBits(DstAddrSpace) == 32) {
1006:       SDValue CvtNone =
1007:           CurDAG->getTargetConstant(NVPTX::PTXCvtMode::NONE, DL, MVT::i32);
1008:       CVTA = CurDAG->getMachineNode(NVPTX::CVT_u32_u64, DL, MVT::i32,
1009:                                     SDValue(CVTA, 0), CvtNone);
1010:     }
1011:
1012:     ReplaceNode(N, CVTA);
1013:     return;
1014:   }
1015: }
1016:
1017: // Helper function template to reduce amount of boilerplate code for
1018: // opcode selection.
1019: static std::optional<unsigned>
1020: pickOpcodeForVT(MVT::SimpleValueType VT, std::optional<unsigned> Opcode_i16,
1021:                 std::optional<unsigned> Opcode_i32,
1022:                 std::optional<unsigned> Opcode_i64) {
1023:   switch (VT) {
1024:   case MVT::f16:
1025:   case MVT::i16:
1026:   case MVT::bf16:
1027:     return Opcode_i16;
1028:   case MVT::v2f16:
1029:   case MVT::v2bf16:
1030:   case MVT::v2i16:
1031:   case MVT::v4i8:
1032:   case MVT::i32:
1033:   case MVT::f32:
1034:     return Opcode_i32;
1035:   case MVT::v2f32:
1036:   case MVT::v2i32:
1037:   case MVT::i64:
1038:   case MVT::f64:
1039:     return Opcode_i64;
1040:   default:
1041:     return std::nullopt;
1042:   }
1043: }
1044:
1045: static inline bool isAddLike(const SDValue V) {
1046:   return V.getOpcode() == ISD::ADD ||
1047:          (V->getOpcode() == ISD::OR && V->getFlags().hasDisjoint());
1048: }
1049:
1050: static SDValue stripAssertAlign(SDValue N) {
1051:   if (N.getOpcode() == ISD::AssertAlign)
1052:     N = N.getOperand(0);
1053:   return N;
1054: }
1055:
1056: // selectBaseADDR - Match a dag node which will serve as the base address for an
1057: // ADDR operand pair.
1058: static SDValue selectBaseADDR(SDValue N, SelectionDAG *DAG) {
1059:   N = stripAssertAlign(N);
1060:   if (const auto *GA = dyn_cast<GlobalAddressSDNode>(N))
1061:     return DAG->getTargetGlobalAddress(GA->getGlobal(), SDLoc(N),
1062:                                        GA->getValueType(0), GA->getOffset(),
1063:                                        GA->getTargetFlags());
1064:   if (const auto *ES = dyn_cast<ExternalSymbolSDNode>(N))
1065:     return DAG->getTargetExternalSymbol(ES->getSymbol(), ES->getValueType(0),
1066:                                         ES->getTargetFlags());
1067:   if (const auto *FIN = dyn_cast<FrameIndexSDNode>(N))
1068:     return DAG->getTargetFrameIndex(FIN->getIndex(), FIN->getValueType(0));
1069:
1070:   return N;
1071: }
1072:
1073: static SDValue accumulateOffset(SDValue &Addr, SDLoc DL, SelectionDAG *DAG) {
1074:   Addr = stripAssertAlign(Addr);
1075:   APInt AccumulatedOffset(64u, 0);
1076:   while (isAddLike(Addr)) {
1077:     const auto *CN = dyn_cast<ConstantSDNode>(Addr.getOperand(1));
1078:     if (!CN)
1079:       break;
1080:
```
- EN: This range implements operational logic in helpers such as getMachineNode, getTargetConstant, SDValue, ReplaceNode, translating backend policy into executable code.
- CN: 这一段实现了 getMachineNode、getTargetConstant、SDValue、ReplaceNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1081-1170
```cpp
1081:     const APInt CI = CN->getAPIntValue().sext(64);
1082:     if (!(CI + AccumulatedOffset).isSignedIntN(32))
1083:       break;
1084:
1085:     AccumulatedOffset += CI;
1086:     Addr = stripAssertAlign(Addr->getOperand(0));
1087:   }
1088:   return DAG->getSignedTargetConstant(AccumulatedOffset.getSExtValue(), DL,
1089:                                       MVT::i32);
1090: }
1091:
1092: static std::pair<SDValue, SDValue> selectADDR(SDValue Addr, SelectionDAG *DAG) {
1093:   SDValue Offset = accumulateOffset(Addr, SDLoc(Addr), DAG);
1094:   SDValue Base = selectBaseADDR(Addr, DAG);
1095:   return {Base, Offset};
1096: }
1097:
1098: // Select a pair of operands which represent a valid PTX address, this could be
1099: // one of the following things:
1100: //  - [var] - Offset is simply set to 0
1101: //  - [reg] - Offset is simply set to 0
1102: //  - [reg+immOff]
1103: //  - [var+immOff]
1104: // Note that immOff must fit into a 32-bit signed integer.
1105: bool NVPTXDAGToDAGISel::SelectADDR(SDValue Addr, SDValue &Base,
1106:                                    SDValue &Offset) {
1107:   std::tie(Base, Offset) = selectADDR(Addr, CurDAG);
1108:   return true;
1109: }
1110:
1111: bool NVPTXDAGToDAGISel::tryLoad(SDNode *N) {
1112:   MemSDNode *LD = cast<MemSDNode>(N);
1113:   assert(LD->readMem() && "Expected load");
1114:
1115:   // do not support pre/post inc/dec
1116:   const LoadSDNode *PlainLoad = dyn_cast<LoadSDNode>(LD);
1117:   if (PlainLoad && PlainLoad->isIndexed())
1118:     return false;
1119:
1120:   // Address Space Setting
1121:   const auto CodeAddrSpace = getAddrSpace(LD);
1122:   if (canLowerToLDG(*LD, *Subtarget, CodeAddrSpace))
1123:     return tryLDG(LD);
1124:
1125:   SDLoc DL(LD);
1126:   SDValue Chain = N->getOperand(0);
1127:   const auto [Ordering, Scope] = insertMemoryInstructionFence(DL, Chain, LD);
1128:
1129:   const unsigned FromTypeWidth = LD->getMemoryVT().getSizeInBits();
1130:
1131:   // Vector Setting
1132:   const unsigned FromType =
1133:       (PlainLoad && (PlainLoad->getExtensionType() == ISD::SEXTLOAD))
1134:           ? NVPTX::PTXLdStInstCode::Signed
1135:           : NVPTX::PTXLdStInstCode::Untyped;
1136:
1137:   uint32_t UsedBytesMask;
1138:   switch (N->getOpcode()) {
1139:   case ISD::LOAD:
1140:   case ISD::ATOMIC_LOAD:
1141:     UsedBytesMask = UINT32_MAX;
1142:     break;
1143:   case NVPTXISD::MLoad:
1144:     UsedBytesMask = N->getConstantOperandVal(3);
1145:     break;
1146:   default:
1147:     llvm_unreachable("Unexpected opcode");
1148:   }
1149:
1150:   assert(isPowerOf2_32(FromTypeWidth) && FromTypeWidth >= 8 &&
1151:          FromTypeWidth <= 128 && "Invalid width for load");
1152:
1153:   // Create the machine instruction DAG
1154:   const auto [Base, Offset] = selectADDR(N->getOperand(1), CurDAG);
1155:   SDValue Ops[] = {getI32Imm(Ordering, DL),
1156:                    getI32Imm(Scope, DL),
1157:                    getI32Imm(CodeAddrSpace, DL),
1158:                    getI32Imm(FromType, DL),
1159:                    getI32Imm(FromTypeWidth, DL),
1160:                    getI32Imm(UsedBytesMask, DL),
1161:                    Base,
1162:                    Offset,
1163:                    Chain};
1164:
1165:   const MVT::SimpleValueType TargetVT = LD->getSimpleValueType(0).SimpleTy;
1166:   const std::optional<unsigned> Opcode =
1167:       pickOpcodeForVT(TargetVT, NVPTX::LD_i16, NVPTX::LD_i32, NVPTX::LD_i64);
1168:   if (!Opcode)
1169:     return false;
1170:
```
- EN: This range implements operational logic in helpers such as getAPIntValue, stripAssertAlign, selectADDR, accumulateOffset, translating backend policy into executable code.
- CN: 这一段实现了 getAPIntValue、stripAssertAlign、selectADDR、accumulateOffset 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1171-1260
```cpp
1171:   SDNode *NVPTXLD = CurDAG->getMachineNode(*Opcode, DL, LD->getVTList(), Ops);
1172:   if (!NVPTXLD)
1173:     return false;
1174:
1175:   MachineMemOperand *MemRef = LD->getMemOperand();
1176:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(NVPTXLD), {MemRef});
1177:
1178:   ReplaceNode(LD, NVPTXLD);
1179:   return true;
1180: }
1181:
1182: static unsigned getStoreVectorNumElts(SDNode *N) {
1183:   switch (N->getOpcode()) {
1184:   case NVPTXISD::StoreV2:
1185:     return 2;
1186:   case NVPTXISD::StoreV4:
1187:     return 4;
1188:   case NVPTXISD::StoreV8:
1189:     return 8;
1190:   default:
1191:     llvm_unreachable("Unexpected opcode");
1192:   }
1193: }
1194:
1195: bool NVPTXDAGToDAGISel::tryLoadVector(SDNode *N) {
1196:   MemSDNode *LD = cast<MemSDNode>(N);
1197:
1198:   // Address Space Setting
1199:   const auto CodeAddrSpace = getAddrSpace(LD);
1200:   if (canLowerToLDG(*LD, *Subtarget, CodeAddrSpace))
1201:     return tryLDG(LD);
1202:
1203:   const MVT EltVT = LD->getSimpleValueType(0);
1204:   SDLoc DL(LD);
1205:   SDValue Chain = LD->getChain();
1206:   const auto [Ordering, Scope] = insertMemoryInstructionFence(DL, Chain, LD);
1207:
1208:   // Type Setting: fromType + fromTypeWidth
1209:   //
1210:   // Sign   : ISD::SEXTLOAD
1211:   // Unsign : ISD::ZEXTLOAD, ISD::NON_EXTLOAD or ISD::EXTLOAD and the
1212:   //          type is integer
1213:   // Float  : ISD::NON_EXTLOAD or ISD::EXTLOAD and the type is float
1214:   // Read at least 8 bits (predicates are stored as 8-bit values)
1215:   // Get the original LoadSDNode::getExtensionType() value
1216:   const unsigned ExtensionType = N->getConstantOperandVal(4);
1217:   const unsigned FromType = (ExtensionType == ISD::SEXTLOAD)
1218:                                 ? NVPTX::PTXLdStInstCode::Signed
1219:                                 : NVPTX::PTXLdStInstCode::Untyped;
1220:
1221:   const unsigned FromTypeWidth = getFromTypeWidthForLoad(LD);
1222:   const uint32_t UsedBytesMask = N->getConstantOperandVal(3);
1223:
1224:   assert(!(EltVT.isVector() && ExtensionType != ISD::NON_EXTLOAD));
1225:
1226:   const auto [Base, Offset] = selectADDR(N->getOperand(1), CurDAG);
1227:   SDValue Ops[] = {getI32Imm(Ordering, DL),
1228:                    getI32Imm(Scope, DL),
1229:                    getI32Imm(CodeAddrSpace, DL),
1230:                    getI32Imm(FromType, DL),
1231:                    getI32Imm(FromTypeWidth, DL),
1232:                    getI32Imm(UsedBytesMask, DL),
1233:                    Base,
1234:                    Offset,
1235:                    Chain};
1236:
1237:   std::optional<unsigned> Opcode;
1238:   switch (N->getOpcode()) {
1239:   default:
1240:     llvm_unreachable("Unexpected opcode");
1241:   case NVPTXISD::LoadV2:
1242:     Opcode = pickOpcodeForVT(EltVT.SimpleTy, NVPTX::LDV_i16_v2,
1243:                              NVPTX::LDV_i32_v2, NVPTX::LDV_i64_v2);
1244:     break;
1245:   case NVPTXISD::LoadV4:
1246:     Opcode = pickOpcodeForVT(EltVT.SimpleTy, NVPTX::LDV_i16_v4,
1247:                              NVPTX::LDV_i32_v4, NVPTX::LDV_i64_v4);
1248:     break;
1249:   case NVPTXISD::LoadV8:
1250:     Opcode = pickOpcodeForVT(EltVT.SimpleTy, {/* no v8i16 */},
1251:                              NVPTX::LDV_i32_v8, {/* no v8i64 */});
1252:     break;
1253:   }
1254:   if (!Opcode)
1255:     return false;
1256:
1257:   SDNode *NVPTXLD = CurDAG->getMachineNode(*Opcode, DL, LD->getVTList(), Ops);
1258:
1259:   MachineMemOperand *MemRef = LD->getMemOperand();
1260:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(NVPTXLD), {MemRef});
```
- EN: This range implements operational logic in helpers such as getMachineNode, getMemOperand, setNodeMemRefs, ReplaceNode, translating backend policy into executable code.
- CN: 这一段实现了 getMachineNode、getMemOperand、setNodeMemRefs、ReplaceNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1261-1350
```cpp
1261:
1262:   ReplaceNode(LD, NVPTXLD);
1263:   return true;
1264: }
1265:
1266: bool NVPTXDAGToDAGISel::tryLDG(MemSDNode *LD) {
1267:   SDLoc DL(LD);
1268:
1269:   unsigned ExtensionType;
1270:   uint32_t UsedBytesMask;
1271:   if (const auto *Load = dyn_cast<LoadSDNode>(LD)) {
1272:     ExtensionType = Load->getExtensionType();
1273:     UsedBytesMask = UINT32_MAX;
1274:   } else {
1275:     ExtensionType = LD->getConstantOperandVal(4);
1276:     UsedBytesMask = LD->getConstantOperandVal(3);
1277:   }
1278:   const unsigned FromType = (ExtensionType == ISD::SEXTLOAD)
1279:                                 ? NVPTX::PTXLdStInstCode::Signed
1280:                                 : NVPTX::PTXLdStInstCode::Untyped;
1281:
1282:   const unsigned FromTypeWidth = getFromTypeWidthForLoad(LD);
1283:
1284:   assert(!(LD->getSimpleValueType(0).isVector() &&
1285:            ExtensionType != ISD::NON_EXTLOAD));
1286:
1287:   const auto [Base, Offset] = selectADDR(LD->getOperand(1), CurDAG);
1288:   SDValue Ops[] = {getI32Imm(FromType, DL),
1289:                    getI32Imm(FromTypeWidth, DL),
1290:                    getI32Imm(UsedBytesMask, DL),
1291:                    Base,
1292:                    Offset,
1293:                    LD->getChain()};
1294:
1295:   const MVT::SimpleValueType TargetVT = LD->getSimpleValueType(0).SimpleTy;
1296:   std::optional<unsigned> Opcode;
1297:   switch (LD->getOpcode()) {
1298:   default:
1299:     llvm_unreachable("Unexpected opcode");
1300:   case ISD::LOAD:
1301:     Opcode = pickOpcodeForVT(TargetVT, NVPTX::LD_GLOBAL_NC_i16,
1302:                              NVPTX::LD_GLOBAL_NC_i32, NVPTX::LD_GLOBAL_NC_i64);
1303:     break;
1304:   case NVPTXISD::MLoad:
1305:     Opcode = pickOpcodeForVT(TargetVT, std::nullopt, NVPTX::LD_GLOBAL_NC_i32,
1306:                              NVPTX::LD_GLOBAL_NC_i64);
1307:     break;
1308:   case NVPTXISD::LoadV2:
1309:     Opcode =
1310:         pickOpcodeForVT(TargetVT, NVPTX::LD_GLOBAL_NC_v2i16,
1311:                         NVPTX::LD_GLOBAL_NC_v2i32, NVPTX::LD_GLOBAL_NC_v2i64);
1312:     break;
1313:   case NVPTXISD::LoadV4:
1314:     Opcode =
1315:         pickOpcodeForVT(TargetVT, NVPTX::LD_GLOBAL_NC_v4i16,
1316:                         NVPTX::LD_GLOBAL_NC_v4i32, NVPTX::LD_GLOBAL_NC_v4i64);
1317:     break;
1318:   case NVPTXISD::LoadV8:
1319:     Opcode = pickOpcodeForVT(TargetVT, {/* no v8i16 */},
1320:                              NVPTX::LD_GLOBAL_NC_v8i32, {/* no v8i64 */});
1321:     break;
1322:   }
1323:   if (!Opcode)
1324:     return false;
1325:
1326:   SDNode *NVPTXLDG = CurDAG->getMachineNode(*Opcode, DL, LD->getVTList(), Ops);
1327:
1328:   ReplaceNode(LD, NVPTXLDG);
1329:   return true;
1330: }
1331:
1332: unsigned NVPTXDAGToDAGISel::getFromTypeWidthForLoad(const MemSDNode *Mem) {
1333:   auto TotalWidth = Mem->getMemoryVT().getSizeInBits();
1334:   auto NumElts = Mem->getNumValues() - 1;
1335:   auto ElementBitWidth = TotalWidth / NumElts;
1336:   assert(isPowerOf2_32(ElementBitWidth) && ElementBitWidth >= 8 &&
1337:          ElementBitWidth <= 128 && TotalWidth <= 256 &&
1338:          "Invalid width for load");
1339:   return ElementBitWidth;
1340: }
1341:
1342: bool NVPTXDAGToDAGISel::tryLDU(SDNode *N) {
1343:   auto *LD = cast<MemSDNode>(N);
1344:
1345:   SDLoc DL(N);
1346:   const unsigned FromTypeWidth = getFromTypeWidthForLoad(LD);
1347:   const MVT::SimpleValueType TargetVT = LD->getSimpleValueType(0).SimpleTy;
1348:
1349:   // If this is an LDU intrinsic, the address is the third operand. If its an
1350:   // LDU SD node (from custom vector handling), then its the second operand
```
- EN: This range implements operational logic in helpers such as ReplaceNode, NVPTXDAGToDAGISel::tryLDG, DL, getExtensionType, translating backend policy into executable code.
- CN: 这一段实现了 ReplaceNode、NVPTXDAGToDAGISel::tryLDG、DL、getExtensionType 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1351-1440
```cpp
1351:   SDValue Addr =
1352:       LD->getOperand(LD->getOpcode() == ISD::INTRINSIC_W_CHAIN ? 2 : 1);
1353:
1354:   const auto [Base, Offset] = selectADDR(Addr, CurDAG);
1355:   SDValue Ops[] = {getI32Imm(FromTypeWidth, DL), Base, Offset, LD->getChain()};
1356:
1357:   std::optional<unsigned> Opcode;
1358:   switch (N->getOpcode()) {
1359:   default:
1360:     llvm_unreachable("Unexpected opcode");
1361:   case ISD::INTRINSIC_W_CHAIN:
1362:     Opcode = pickOpcodeForVT(TargetVT, NVPTX::LDU_GLOBAL_i16,
1363:                              NVPTX::LDU_GLOBAL_i32, NVPTX::LDU_GLOBAL_i64);
1364:     break;
1365:   case NVPTXISD::LDUV2:
1366:     Opcode = pickOpcodeForVT(TargetVT, NVPTX::LDU_GLOBAL_v2i16,
1367:                              NVPTX::LDU_GLOBAL_v2i32, NVPTX::LDU_GLOBAL_v2i64);
1368:     break;
1369:   case NVPTXISD::LDUV4:
1370:     Opcode = pickOpcodeForVT(TargetVT, NVPTX::LDU_GLOBAL_v4i16,
1371:                              NVPTX::LDU_GLOBAL_v4i32, {/* no v4i64 */});
1372:     break;
1373:   }
1374:   if (!Opcode)
1375:     return false;
1376:
1377:   SDNode *NVPTXLDU = CurDAG->getMachineNode(*Opcode, DL, LD->getVTList(), Ops);
1378:
1379:   ReplaceNode(LD, NVPTXLDU);
1380:   return true;
1381: }
1382:
1383: bool NVPTXDAGToDAGISel::tryStore(SDNode *N) {
1384:   MemSDNode *ST = cast<MemSDNode>(N);
1385:   assert(ST->writeMem() && "Expected store");
1386:   StoreSDNode *PlainStore = dyn_cast<StoreSDNode>(ST);
1387:   AtomicSDNode *AtomicStore = dyn_cast<AtomicSDNode>(ST);
1388:   assert((PlainStore || AtomicStore) && "Expected store");
1389:
1390:   // do not support pre/post inc/dec
1391:   if (PlainStore && PlainStore->isIndexed())
1392:     return false;
1393:
1394:   // Address Space Setting
1395:   const auto CodeAddrSpace = getAddrSpace(ST);
1396:
1397:   SDLoc DL(ST);
1398:   SDValue Chain = ST->getChain();
1399:   const auto [Ordering, Scope] = insertMemoryInstructionFence(DL, Chain, ST);
1400:
1401:   // Vector Setting
1402:   const unsigned ToTypeWidth = ST->getMemoryVT().getSizeInBits();
1403:
1404:   // Create the machine instruction DAG
1405:   SDValue Value = PlainStore ? PlainStore->getValue() : AtomicStore->getVal();
1406:
1407:   assert(isPowerOf2_32(ToTypeWidth) && ToTypeWidth >= 8 && ToTypeWidth <= 128 &&
1408:          "Invalid width for store");
1409:
1410:   const auto [Base, Offset] = selectADDR(ST->getBasePtr(), CurDAG);
1411:   SDValue Ops[] = {selectPossiblyImm(Value),
1412:                    getI32Imm(Ordering, DL),
1413:                    getI32Imm(Scope, DL),
1414:                    getI32Imm(CodeAddrSpace, DL),
1415:                    getI32Imm(ToTypeWidth, DL),
1416:                    Base,
1417:                    Offset,
1418:                    Chain};
1419:
1420:   const std::optional<unsigned> Opcode =
1421:       pickOpcodeForVT(Value.getSimpleValueType().SimpleTy, NVPTX::ST_i16,
1422:                       NVPTX::ST_i32, NVPTX::ST_i64);
1423:   if (!Opcode)
1424:     return false;
1425:
1426:   SDNode *NVPTXST = CurDAG->getMachineNode(*Opcode, DL, MVT::Other, Ops);
1427:
1428:   if (!NVPTXST)
1429:     return false;
1430:
1431:   MachineMemOperand *MemRef = ST->getMemOperand();
1432:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(NVPTXST), {MemRef});
1433:   ReplaceNode(ST, NVPTXST);
1434:   return true;
1435: }
1436:
1437: bool NVPTXDAGToDAGISel::tryStoreVector(SDNode *N) {
1438:   MemSDNode *ST = cast<MemSDNode>(N);
1439:   const unsigned TotalWidth = ST->getMemoryVT().getSizeInBits();
1440:
```
- EN: This range implements operational logic in helpers such as getOperand, selectADDR, llvm_unreachable, getMachineNode, translating backend policy into executable code.
- CN: 这一段实现了 getOperand、selectADDR、llvm_unreachable、getMachineNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1441-1530
```cpp
1441:   // Address Space Setting
1442:   const auto CodeAddrSpace = getAddrSpace(ST);
1443:   if (CodeAddrSpace == NVPTX::AddressSpace::Const) {
1444:     report_fatal_error("Cannot store to pointer that points to constant "
1445:                        "memory space");
1446:   }
1447:
1448:   SDLoc DL(ST);
1449:   SDValue Chain = ST->getChain();
1450:   const auto [Ordering, Scope] = insertMemoryInstructionFence(DL, Chain, ST);
1451:
1452:   const unsigned NumElts = getStoreVectorNumElts(ST);
1453:
1454:   SmallVector<SDValue, 16> Ops;
1455:   for (auto &V : ST->ops().slice(1, NumElts))
1456:     Ops.push_back(selectPossiblyImm(V));
1457:   SDValue Addr = N->getOperand(NumElts + 1);
1458:   const unsigned ToTypeWidth = TotalWidth / NumElts;
1459:
1460:   assert(isPowerOf2_32(ToTypeWidth) && ToTypeWidth >= 8 && ToTypeWidth <= 128 &&
1461:          TotalWidth <= 256 && "Invalid width for store");
1462:
1463:   const auto [Base, Offset] = selectADDR(Addr, CurDAG);
1464:   Ops.append({getI32Imm(Ordering, DL), getI32Imm(Scope, DL),
1465:               getI32Imm(CodeAddrSpace, DL), getI32Imm(ToTypeWidth, DL), Base,
1466:               Offset, Chain});
1467:
1468:   const MVT::SimpleValueType EltVT =
1469:       ST->getOperand(1).getSimpleValueType().SimpleTy;
1470:   std::optional<unsigned> Opcode;
1471:   switch (ST->getOpcode()) {
1472:   default:
1473:     return false;
1474:   case NVPTXISD::StoreV2:
1475:     Opcode = pickOpcodeForVT(EltVT, NVPTX::STV_i16_v2, NVPTX::STV_i32_v2,
1476:                              NVPTX::STV_i64_v2);
1477:     break;
1478:   case NVPTXISD::StoreV4:
1479:     Opcode = pickOpcodeForVT(EltVT, NVPTX::STV_i16_v4, NVPTX::STV_i32_v4,
1480:                              NVPTX::STV_i64_v4);
1481:     break;
1482:   case NVPTXISD::StoreV8:
1483:     Opcode = pickOpcodeForVT(EltVT, {/* no v8i16 */}, NVPTX::STV_i32_v8,
1484:                              {/* no v8i64 */});
1485:     break;
1486:   }
1487:
1488:   if (!Opcode)
1489:     return false;
1490:
1491:   SDNode *NVPTXST = CurDAG->getMachineNode(*Opcode, DL, MVT::Other, Ops);
1492:
1493:   MachineMemOperand *MemRef = ST->getMemOperand();
1494:   CurDAG->setNodeMemRefs(cast<MachineSDNode>(NVPTXST), {MemRef});
1495:
1496:   ReplaceNode(ST, NVPTXST);
1497:   return true;
1498: }
1499:
1500: /// SelectBFE - Look for instruction sequences that can be made more efficient
1501: /// by using the 'bfe' (bit-field extract) PTX instruction
1502: bool NVPTXDAGToDAGISel::tryBFE(SDNode *N) {
1503:   SDLoc DL(N);
1504:   SDValue LHS = N->getOperand(0);
1505:   SDValue RHS = N->getOperand(1);
1506:   SDValue Len;
1507:   SDValue Start;
1508:   SDValue Val;
1509:   bool IsSigned = false;
1510:
1511:   if (N->getOpcode() == ISD::AND) {
1512:     // Canonicalize the operands
1513:     // We want 'and %val, %mask'
1514:     if (isa<ConstantSDNode>(LHS) && !isa<ConstantSDNode>(RHS)) {
1515:       std::swap(LHS, RHS);
1516:     }
1517:
1518:     ConstantSDNode *Mask = dyn_cast<ConstantSDNode>(RHS);
1519:     if (!Mask) {
1520:       // We need a constant mask on the RHS of the AND
1521:       return false;
1522:     }
1523:
1524:     // Extract the mask bits
1525:     uint64_t MaskVal = Mask->getZExtValue();
1526:     if (!isMask_64(MaskVal)) {
1527:       // We *could* handle shifted masks here, but doing so would require an
1528:       // 'and' operation to fix up the low-order bits so we would trade
1529:       // shr+and for bfe+and, which has the same throughput
1530:       return false;
```
- EN: This range implements operational logic in helpers such as getAddrSpace, DL, getChain, insertMemoryInstructionFence, translating backend policy into executable code.
- CN: 这一段实现了 getAddrSpace、DL、getChain、insertMemoryInstructionFence 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1531-1620
```cpp
1531:     }
1532:
1533:     // How many bits are in our mask?
1534:     int64_t NumBits = countr_one(MaskVal);
1535:     Len = CurDAG->getTargetConstant(NumBits, DL, MVT::i32);
1536:
1537:     if (LHS.getOpcode() == ISD::SRL || LHS.getOpcode() == ISD::SRA) {
1538:       // We have a 'srl/and' pair, extract the effective start bit and length
1539:       Val = LHS.getNode()->getOperand(0);
1540:       Start = LHS.getNode()->getOperand(1);
1541:       ConstantSDNode *StartConst = dyn_cast<ConstantSDNode>(Start);
1542:       if (StartConst) {
1543:         uint64_t StartVal = StartConst->getZExtValue();
1544:         // How many "good" bits do we have left?  "good" is defined here as bits
1545:         // that exist in the original value, not shifted in.
1546:         int64_t GoodBits = Start.getValueSizeInBits() - StartVal;
1547:         if (NumBits > GoodBits) {
1548:           // Do not handle the case where bits have been shifted in. In theory
1549:           // we could handle this, but the cost is likely higher than just
1550:           // emitting the srl/and pair.
1551:           return false;
1552:         }
1553:         Start = CurDAG->getTargetConstant(StartVal, DL, MVT::i32);
1554:       } else {
1555:         // Do not handle the case where the shift amount (can be zero if no srl
1556:         // was found) is not constant. We could handle this case, but it would
1557:         // require run-time logic that would be more expensive than just
1558:         // emitting the srl/and pair.
1559:         return false;
1560:       }
1561:     } else {
1562:       // Do not handle the case where the LHS of the and is not a shift. While
1563:       // it would be trivial to handle this case, it would just transform
1564:       // 'and' -> 'bfe', but 'and' has higher-throughput.
1565:       return false;
1566:     }
1567:   } else if (N->getOpcode() == ISD::SRL || N->getOpcode() == ISD::SRA) {
1568:     if (LHS->getOpcode() == ISD::AND) {
1569:       ConstantSDNode *ShiftCnst = dyn_cast<ConstantSDNode>(RHS);
1570:       if (!ShiftCnst) {
1571:         // Shift amount must be constant
1572:         return false;
1573:       }
1574:
1575:       uint64_t ShiftAmt = ShiftCnst->getZExtValue();
1576:
1577:       SDValue AndLHS = LHS->getOperand(0);
1578:       SDValue AndRHS = LHS->getOperand(1);
1579:
1580:       // Canonicalize the AND to have the mask on the RHS
1581:       if (isa<ConstantSDNode>(AndLHS)) {
1582:         std::swap(AndLHS, AndRHS);
1583:       }
1584:
1585:       ConstantSDNode *MaskCnst = dyn_cast<ConstantSDNode>(AndRHS);
1586:       if (!MaskCnst) {
1587:         // Mask must be constant
1588:         return false;
1589:       }
1590:
1591:       uint64_t MaskVal = MaskCnst->getZExtValue();
1592:       uint64_t NumZeros;
1593:       uint64_t NumBits;
1594:       if (isMask_64(MaskVal)) {
1595:         NumZeros = 0;
1596:         // The number of bits in the result bitfield will be the number of
1597:         // trailing ones (the AND) minus the number of bits we shift off
1598:         NumBits = llvm::countr_one(MaskVal) - ShiftAmt;
1599:       } else if (isShiftedMask_64(MaskVal)) {
1600:         NumZeros = llvm::countr_zero(MaskVal);
1601:         unsigned NumOnes = llvm::countr_one(MaskVal >> NumZeros);
1602:         // The number of bits in the result bitfield will be the number of
1603:         // trailing zeros plus the number of set bits in the mask minus the
1604:         // number of bits we shift off
1605:         NumBits = NumZeros + NumOnes - ShiftAmt;
1606:       } else {
1607:         // This is not a mask we can handle
1608:         return false;
1609:       }
1610:
1611:       if (ShiftAmt < NumZeros) {
1612:         // Handling this case would require extra logic that would make this
1613:         // transformation non-profitable
1614:         return false;
1615:       }
1616:
1617:       Val = AndLHS;
1618:       Start = CurDAG->getTargetConstant(ShiftAmt, DL, MVT::i32);
1619:       Len = CurDAG->getTargetConstant(NumBits, DL, MVT::i32);
1620:
```
- EN: This range implements operational logic in helpers such as countr_one, getTargetConstant, getNode, getZExtValue, translating backend policy into executable code.
- CN: 这一段实现了 countr_one、getTargetConstant、getNode、getZExtValue 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1621-1710
```cpp
1621:       // If pre-shift AND includes the sign bit in the bitfield, we must use
1622:       // signed BFE to replicate that bit during bitfield extraction. If the
1623:       // sign bit is not part of the mask, unsigned BFE will zero out upper bits
1624:       // of the result
1625:       if (N->getOpcode() == ISD::SRA)
1626:         IsSigned = (ShiftAmt + NumBits) == Val.getValueSizeInBits();
1627:     } else if (LHS->getOpcode() == ISD::SHL) {
1628:       // Here, we have a pattern like:
1629:       //
1630:       // (sra (shl val, NN), MM)
1631:       // or
1632:       // (srl (shl val, NN), MM)
1633:       //
1634:       // If MM >= NN, we can efficiently optimize this with bfe
1635:       Val = LHS->getOperand(0);
1636:
1637:       SDValue ShlRHS = LHS->getOperand(1);
1638:       ConstantSDNode *ShlCnst = dyn_cast<ConstantSDNode>(ShlRHS);
1639:       if (!ShlCnst) {
1640:         // Shift amount must be constant
1641:         return false;
1642:       }
1643:       uint64_t InnerShiftAmt = ShlCnst->getZExtValue();
1644:
1645:       SDValue ShrRHS = RHS;
1646:       ConstantSDNode *ShrCnst = dyn_cast<ConstantSDNode>(ShrRHS);
1647:       if (!ShrCnst) {
1648:         // Shift amount must be constant
1649:         return false;
1650:       }
1651:       uint64_t OuterShiftAmt = ShrCnst->getZExtValue();
1652:
1653:       // To avoid extra codegen and be profitable, we need Outer >= Inner
1654:       if (OuterShiftAmt < InnerShiftAmt) {
1655:         return false;
1656:       }
1657:
1658:       // If the outer shift is more than the type size, we have no bitfield to
1659:       // extract (since we also check that the inner shift is <= the outer shift
1660:       // then this also implies that the inner shift is < the type size)
1661:       if (OuterShiftAmt >= Val.getValueSizeInBits()) {
1662:         return false;
1663:       }
1664:
1665:       Start = CurDAG->getTargetConstant(OuterShiftAmt - InnerShiftAmt, DL,
1666:                                         MVT::i32);
1667:       Len = CurDAG->getTargetConstant(Val.getValueSizeInBits() - OuterShiftAmt,
1668:                                       DL, MVT::i32);
1669:
1670:       if (N->getOpcode() == ISD::SRA) {
1671:         // If we have a arithmetic right shift, we need to use the signed bfe
1672:         // variant
1673:         IsSigned = true;
1674:       }
1675:     } else {
1676:       // No can do...
1677:       return false;
1678:     }
1679:   } else {
1680:     // No can do...
1681:     return false;
1682:   }
1683:
1684:
1685:   unsigned Opc;
1686:   // For the BFE operations we form here from "and" and "srl", always use the
1687:   // unsigned variants.
1688:   if (Val.getValueType() == MVT::i32) {
1689:     if (IsSigned) {
1690:       Opc = NVPTX::BFE_S32rii;
1691:     } else {
1692:       Opc = NVPTX::BFE_U32rii;
1693:     }
1694:   } else if (Val.getValueType() == MVT::i64) {
1695:     if (IsSigned) {
1696:       Opc = NVPTX::BFE_S64rii;
1697:     } else {
1698:       Opc = NVPTX::BFE_U64rii;
1699:     }
1700:   } else {
1701:     // We cannot handle this type
1702:     return false;
1703:   }
1704:
1705:   SDValue Ops[] = {
1706:     Val, Start, Len
1707:   };
1708:
1709:   ReplaceNode(N, CurDAG->getMachineNode(Opc, DL, N->getVTList(), Ops));
1710:   return true;
```
- EN: This range implements operational logic in helpers such as getValueSizeInBits, getOperand, getZExtValue, ReplaceNode, translating backend policy into executable code.
- CN: 这一段实现了 getValueSizeInBits、getOperand、getZExtValue、ReplaceNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1711-1800
```cpp
1711: }
1712:
1713: // Select bf16/bf16v2 FADD, FSUB, FMUL as fma on targets with only fma
1714: bool NVPTXDAGToDAGISel::tryBF16ArithToFMA(SDNode *N) {
1715:   EVT VT = SDValue(N, 0).getValueType();
1716:   if (VT.getScalarType() != MVT::bf16)
1717:     return false;
1718:
1719:   const NVPTXSubtarget *STI = TM.getSubtargetImpl();
1720:   if (STI->hasNativeBF16Support(N->getOpcode()))
1721:     return false;
1722:
1723:   const bool IsVec = VT.isVector();
1724:   assert(!IsVec || VT.getVectorNumElements() == 2);
1725:   SDLoc DL(N);
1726:   SDValue N0 = N->getOperand(0);
1727:   SDValue N1 = N->getOperand(1);
1728:   SmallVector<SDValue, 3> Operands;
1729:   auto GetConstant = [&](float Value) -> SDValue {
1730:     // BF16 immediates must be legalized to integer register values
1731:     APFloat APF(Value);
1732:     bool LosesInfo;
1733:     APF.convert(APFloat::BFloat(), APFloat::rmNearestTiesToEven, &LosesInfo);
1734:     assert(!LosesInfo);
1735:     if (IsVec) {
1736:       auto API = APF.bitcastToAPInt();
1737:       API = API.concat(API);
1738:       auto Const = CurDAG->getTargetConstant(API, DL, MVT::i32);
1739:       return SDValue(CurDAG->getMachineNode(NVPTX::MOV_B32_i, DL, VT, Const),
1740:                      0);
1741:     }
1742:     auto Const = CurDAG->getTargetConstantFP(APF, DL, VT);
1743:     return SDValue(CurDAG->getMachineNode(NVPTX::MOV_BF16_i, DL, VT, Const), 0);
1744:   };
1745:
1746:   switch (N->getOpcode()) {
1747:   case ISD::FADD:
1748:     // add(a, b) -> fma(a, 1.0, b)
1749:     Operands = {N0, GetConstant(1.0), N1};
1750:     break;
1751:   case ISD::FSUB:
1752:     // sub(a, b) -> fma(b, -1.0, a)
1753:     Operands = {N1, GetConstant(-1.0), N0};
1754:     break;
1755:   case ISD::FMUL:
1756:     // mul(a, b) -> fma(a, b, -0.0)
1757:     // NOTE: The identity is -0, not 0, because -0 + 0 == 0 for floats
1758:     Operands = {N0, N1, GetConstant(-0.0)};
1759:     break;
1760:   default:
1761:     llvm_unreachable("Unexpected opcode");
1762:   };
1763:
1764:   int Opcode = IsVec ? NVPTX::FMA_BF16x2rrr : NVPTX::FMA_BF16rrr;
1765:   MachineSDNode *FMA = CurDAG->getMachineNode(Opcode, DL, VT, Operands);
1766:   ReplaceNode(N, FMA);
1767:   return true;
1768: }
1769:
1770: SDValue NVPTXDAGToDAGISel::selectPossiblyImm(SDValue V) {
1771:   if (V.getOpcode() == ISD::BITCAST)
1772:     V = V.getOperand(0);
1773:
1774:   if (auto *CN = dyn_cast<ConstantSDNode>(V))
1775:     return CurDAG->getTargetConstant(CN->getAPIntValue(), SDLoc(V),
1776:                                      V.getValueType());
1777:   if (auto *CN = dyn_cast<ConstantFPSDNode>(V))
1778:     return CurDAG->getTargetConstantFP(CN->getValueAPF(), SDLoc(V),
1779:                                        V.getValueType());
1780:   return V;
1781: }
1782:
1783: /// SelectInlineAsmMemoryOperand - Implement addressing mode selection for
1784: /// inline asm expressions.
1785: bool NVPTXDAGToDAGISel::SelectInlineAsmMemoryOperand(
1786:     const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
1787:     std::vector<SDValue> &OutOps) {
1788:   switch (ConstraintID) {
1789:   default:
1790:     return true;
1791:   case InlineAsm::ConstraintCode::m: { // memory
1792:     const auto [Base, Offset] = selectADDR(Op, CurDAG);
1793:     OutOps.push_back(Base);
1794:     OutOps.push_back(Offset);
1795:     return false;
1796:   }
1797:   }
1798:   return true;
1799: }
1800:
```
- EN: This range implements operational logic in helpers such as NVPTXDAGToDAGISel::tryBF16ArithToFMA, SDValue, getSubtargetImpl, isVector, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXDAGToDAGISel::tryBF16ArithToFMA、SDValue、getSubtargetImpl、isVector 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1801-1890
```cpp
1801: void NVPTXDAGToDAGISel::SelectV2I64toI128(SDNode *N) {
1802:   // Lower a CopyToReg with two 64-bit inputs
1803:   // Dst:i128, lo:i64, hi:i64
1804:   //
1805:   // CopyToReg Dst, lo, hi;
1806:   //
1807:   // ==>
1808:   //
1809:   // tmp = V2I64toI128 {lo, hi};
1810:   // CopyToReg Dst, tmp;
1811:   SDValue Dst = N->getOperand(1);
1812:   SDValue Lo = N->getOperand(2);
1813:   SDValue Hi = N->getOperand(3);
1814:
1815:   SDLoc DL(N);
1816:   SDNode *Mov =
1817:       CurDAG->getMachineNode(NVPTX::V2I64toI128, DL, MVT::i128, {Lo, Hi});
1818:
1819:   SmallVector<SDValue, 4> NewOps(N->getNumOperands() - 1);
1820:   NewOps[0] = N->getOperand(0);
1821:   NewOps[1] = Dst;
1822:   NewOps[2] = SDValue(Mov, 0);
1823:   if (N->getNumOperands() == 5)
1824:     NewOps[3] = N->getOperand(4);
1825:   SDValue NewValue = CurDAG->getNode(ISD::CopyToReg, DL, SmallVector<EVT>(N->values()), NewOps);
1826:
1827:   ReplaceNode(N, NewValue.getNode());
1828: }
1829:
1830: void NVPTXDAGToDAGISel::SelectI128toV2I64(SDNode *N) {
1831:   // Lower CopyFromReg from a 128-bit regs to two 64-bit regs
1832:   // Dst:i128, Src:i128
1833:   //
1834:   // {lo, hi} = CopyFromReg Src
1835:   //
1836:   // ==>
1837:   //
1838:   // {lo, hi} = I128toV2I64 Src
1839:   //
1840:   SDValue Ch = N->getOperand(0);
1841:   SDValue Src = N->getOperand(1);
1842:   SDValue Glue = N->getOperand(2);
1843:   SDLoc DL(N);
1844:
1845:   // Add Glue and Ch to the operands and results to avoid break the execution
1846:   // order
1847:   SDNode *Mov = CurDAG->getMachineNode(
1848:       NVPTX::I128toV2I64, DL,
1849:       {MVT::i64, MVT::i64, Ch.getValueType(), Glue.getValueType()},
1850:       {Src, Ch, Glue});
1851:
1852:   ReplaceNode(N, Mov);
1853: }
1854:
1855: bool NVPTXDAGToDAGISel::tryFence(SDNode *N) {
1856:   SDLoc DL(N);
1857:   assert(N->getOpcode() == ISD::ATOMIC_FENCE);
1858:   auto Scope = Scopes[N->getConstantOperandVal(2)];
1859:
1860:   // Singlethread fences have no inter-thread synchronization requirements.
1861:   // Note: std::atomic_signal_fence lowers to singlethread LLVM IR fences;
1862:   // this intentionally drops these before emitting PTX.
1863:   if (Scope == NVPTX::Scope::Thread) {
1864:     CurDAG->ReplaceAllUsesOfValueWith(SDValue(N, 0), N->getOperand(0));
1865:     CurDAG->RemoveDeadNode(N);
1866:     return true;
1867:   }
1868:
1869:   unsigned int FenceOp = getFenceOp(
1870:       NVPTX::Ordering(N->getConstantOperandVal(1)), Scope, Subtarget);
1871:   SDValue Chain = N->getOperand(0);
1872:   SDNode *FenceNode = CurDAG->getMachineNode(FenceOp, DL, MVT::Other, Chain);
1873:   ReplaceNode(N, FenceNode);
1874:   return true;
1875: }
1876:
1877: NVPTXScopes::NVPTXScopes(LLVMContext &C) : Context(&C) {
1878:   Scopes[C.getOrInsertSyncScopeID("singlethread")] = NVPTX::Scope::Thread;
1879:   Scopes[C.getOrInsertSyncScopeID("")] = NVPTX::Scope::System;
1880:   Scopes[C.getOrInsertSyncScopeID("block")] = NVPTX::Scope::Block;
1881:   Scopes[C.getOrInsertSyncScopeID("cluster")] = NVPTX::Scope::Cluster;
1882:   Scopes[C.getOrInsertSyncScopeID("device")] = NVPTX::Scope::Device;
1883: }
1884:
1885: NVPTX::Scope NVPTXScopes::operator[](SyncScope::ID ID) const {
1886:   if (Scopes.empty())
1887:     llvm_unreachable("NVPTX Scopes must be initialized before calling "
1888:                      "NVPTXScopes::operator[]");
1889:
1890:   auto S = Scopes.find(ID);
```
- EN: This range implements operational logic in helpers such as NVPTXDAGToDAGISel::SelectV2I64toI128, getOperand, DL, getMachineNode, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXDAGToDAGISel::SelectV2I64toI128、getOperand、DL、getMachineNode 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1891-1980
```cpp
1891:   if (S == Scopes.end()) {
1892:     auto scopeName = Context->getSyncScopeName(ID);
1893:     assert(scopeName.has_value() && "Scope name must exist.");
1894:
1895:     // Build list of supported syncscopes programmatically
1896:     SmallVector<StringRef> supportedScopes;
1897:     for (const auto &Entry : Scopes) {
1898:       if (auto name = Context->getSyncScopeName(Entry.first))
1899:         supportedScopes.push_back(name->empty() ? "<empty string>" : *name);
1900:     }
1901:
1902:     reportFatalUsageError(
1903:         formatv("NVPTX backend does not support syncscope \"{0}\" (ID={1}).\n"
1904:                 "Supported syncscopes are: {2}.",
1905:                 scopeName.value(), int(ID),
1906:                 make_range(supportedScopes.begin(), supportedScopes.end())));
1907:   }
1908:   return S->second;
1909: }
1910:
1911: bool NVPTXScopes::empty() const { return Scopes.size() == 0; }
1912:
1913: #define CP_ASYNC_BULK_TENSOR_OPCODE(dir, dim, mode, is_s32, suffix)            \
1914:   (is_s32                                                                      \
1915:        ? NVPTX::CP_ASYNC_BULK_TENSOR_##dir##_##dim##_SHARED32_##mode##suffix   \
1916:        : NVPTX::CP_ASYNC_BULK_TENSOR_##dir##_##dim##_##mode##suffix)
1917:
1918: #define GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(dim, mode, is_ch, is_s32)      \
1919:   (is_ch ? (CP_ASYNC_BULK_TENSOR_OPCODE(RED, dim, mode, is_s32, _CH))          \
1920:          : (CP_ASYNC_BULK_TENSOR_OPCODE(RED, dim, mode, is_s32, )))
1921:
1922: static unsigned GetCpAsyncBulkTensorS2GReductionOpcode(size_t Dim,
1923:                                                        bool IsShared32,
1924:                                                        bool IsCacheHint,
1925:                                                        bool IsIm2Col) {
1926:   if (IsIm2Col) {
1927:     switch (Dim) {
1928:     case 3:
1929:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(3D, IM2COL, IsCacheHint,
1930:                                                      IsShared32);
1931:     case 4:
1932:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(4D, IM2COL, IsCacheHint,
1933:                                                      IsShared32);
1934:     case 5:
1935:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(5D, IM2COL, IsCacheHint,
1936:                                                      IsShared32);
1937:     default:
1938:       llvm_unreachable("Invalid Dimension in im2col mode for "
1939:                        "GetCpAsyncBulkTensorS2GReductionOpcode.");
1940:     }
1941:   } else {
1942:     switch (Dim) {
1943:     case 1:
1944:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(1D, TILE, IsCacheHint,
1945:                                                      IsShared32);
1946:     case 2:
1947:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(2D, TILE, IsCacheHint,
1948:                                                      IsShared32);
1949:     case 3:
1950:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(3D, TILE, IsCacheHint,
1951:                                                      IsShared32);
1952:     case 4:
1953:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(4D, TILE, IsCacheHint,
1954:                                                      IsShared32);
1955:     case 5:
1956:       return GET_CP_ASYNC_BULK_TENSOR_OPCODE_S2G_RED(5D, TILE, IsCacheHint,
1957:                                                      IsShared32);
1958:     default:
1959:       llvm_unreachable("Invalid Dimension in tile mode for "
1960:                        "GetCpAsyncBulkTensorS2GReductionOpcode.");
1961:     }
1962:   }
1963: }
1964:
1965: void NVPTXDAGToDAGISel::SelectCpAsyncBulkTensorReduceCommon(SDNode *N,
1966:                                                             unsigned RedOp,
1967:                                                             bool IsIm2Col) {
1968:   // We have {Chain, Intrinsic-ID} followed by the actual intrisic args:
1969:   // src, dst, dims{d0...dN}, cache_hint, cache_hint_flag
1970:   // NumOperands = {Chain, IID} + {Actual intrinsic args}
1971:   //             = {2}          + {4 + dims}
1972:   size_t NumOps = N->getNumOperands();
1973:   size_t NumDims = NumOps - 6;
1974:   bool IsCacheHint = N->getConstantOperandVal(NumOps - 1) == 1;
1975:   size_t NumArgs = NumDims + (IsCacheHint ? 3 : 2); // src, dst, cache_hint
1976:
1977:   SDLoc DL(N);
1978:   SmallVector<SDValue, 12> Ops(N->ops().slice(2, NumArgs));
1979:   Ops.push_back(getI32Imm(RedOp, DL)); // Reduction Op
1980:   Ops.push_back(N->getOperand(0));     // Chain operand
```
- EN: This range implements operational logic in helpers such as getSyncScopeName, assert, push_back, make_range, translating backend policy into executable code.
- CN: 这一段实现了 getSyncScopeName、assert、push_back、make_range 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 1981-2070
```cpp
1981:
1982:   bool IsShared32 =
1983:       CurDAG->getDataLayout().getPointerSizeInBits(ADDRESS_SPACE_SHARED) == 32;
1984:   unsigned Opcode = GetCpAsyncBulkTensorS2GReductionOpcode(
1985:       NumDims, IsShared32, IsCacheHint, IsIm2Col);
1986:   ReplaceNode(N, CurDAG->getMachineNode(Opcode, DL, N->getVTList(), Ops));
1987: }
1988:
1989: #define TCGEN05_ST_OPCODE(SHAPE, NUM)                                          \
1990:   (enableUnpack ? NVPTX::TCGEN05_ST_##SHAPE##_##NUM##_UNPACK                   \
1991:                 : NVPTX::TCGEN05_ST_##SHAPE##_##NUM)
1992:
1993: static unsigned getTcgen05StOpcode(unsigned IID, bool enableUnpack) {
1994:   switch (IID) {
1995:   case Intrinsic::nvvm_tcgen05_st_16x64b_x1:
1996:     return TCGEN05_ST_OPCODE(16x64b, x1);
1997:   case Intrinsic::nvvm_tcgen05_st_16x64b_x2:
1998:     return TCGEN05_ST_OPCODE(16x64b, x2);
1999:   case Intrinsic::nvvm_tcgen05_st_16x64b_x4:
2000:     return TCGEN05_ST_OPCODE(16x64b, x4);
2001:   case Intrinsic::nvvm_tcgen05_st_16x64b_x8:
2002:     return TCGEN05_ST_OPCODE(16x64b, x8);
2003:   case Intrinsic::nvvm_tcgen05_st_16x64b_x16:
2004:     return TCGEN05_ST_OPCODE(16x64b, x16);
2005:   case Intrinsic::nvvm_tcgen05_st_16x64b_x32:
2006:     return TCGEN05_ST_OPCODE(16x64b, x32);
2007:   case Intrinsic::nvvm_tcgen05_st_16x64b_x64:
2008:     return TCGEN05_ST_OPCODE(16x64b, x64);
2009:   case Intrinsic::nvvm_tcgen05_st_16x64b_x128:
2010:     return TCGEN05_ST_OPCODE(16x64b, x128);
2011:   case Intrinsic::nvvm_tcgen05_st_16x128b_x1:
2012:     return TCGEN05_ST_OPCODE(16x128b, x1);
2013:   case Intrinsic::nvvm_tcgen05_st_16x128b_x2:
2014:     return TCGEN05_ST_OPCODE(16x128b, x2);
2015:   case Intrinsic::nvvm_tcgen05_st_16x128b_x4:
2016:     return TCGEN05_ST_OPCODE(16x128b, x4);
2017:   case Intrinsic::nvvm_tcgen05_st_16x128b_x8:
2018:     return TCGEN05_ST_OPCODE(16x128b, x8);
2019:   case Intrinsic::nvvm_tcgen05_st_16x128b_x16:
2020:     return TCGEN05_ST_OPCODE(16x128b, x16);
2021:   case Intrinsic::nvvm_tcgen05_st_16x128b_x32:
2022:     return TCGEN05_ST_OPCODE(16x128b, x32);
2023:   case Intrinsic::nvvm_tcgen05_st_16x128b_x64:
2024:     return TCGEN05_ST_OPCODE(16x128b, x64);
2025:   case Intrinsic::nvvm_tcgen05_st_16x256b_x1:
2026:     return TCGEN05_ST_OPCODE(16x256b, x1);
2027:   case Intrinsic::nvvm_tcgen05_st_16x256b_x2:
2028:     return TCGEN05_ST_OPCODE(16x256b, x2);
2029:   case Intrinsic::nvvm_tcgen05_st_16x256b_x4:
2030:     return TCGEN05_ST_OPCODE(16x256b, x4);
2031:   case Intrinsic::nvvm_tcgen05_st_16x256b_x8:
2032:     return TCGEN05_ST_OPCODE(16x256b, x8);
2033:   case Intrinsic::nvvm_tcgen05_st_16x256b_x16:
2034:     return TCGEN05_ST_OPCODE(16x256b, x16);
2035:   case Intrinsic::nvvm_tcgen05_st_16x256b_x32:
2036:     return TCGEN05_ST_OPCODE(16x256b, x32);
2037:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x1:
2038:     return TCGEN05_ST_OPCODE(16x32bx2, x1);
2039:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x2:
2040:     return TCGEN05_ST_OPCODE(16x32bx2, x2);
2041:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x4:
2042:     return TCGEN05_ST_OPCODE(16x32bx2, x4);
2043:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x8:
2044:     return TCGEN05_ST_OPCODE(16x32bx2, x8);
2045:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x16:
2046:     return TCGEN05_ST_OPCODE(16x32bx2, x16);
2047:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x32:
2048:     return TCGEN05_ST_OPCODE(16x32bx2, x32);
2049:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x64:
2050:     return TCGEN05_ST_OPCODE(16x32bx2, x64);
2051:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x128:
2052:     return TCGEN05_ST_OPCODE(16x32bx2, x128);
2053:   case Intrinsic::nvvm_tcgen05_st_32x32b_x1:
2054:     return TCGEN05_ST_OPCODE(32x32b, x1);
2055:   case Intrinsic::nvvm_tcgen05_st_32x32b_x2:
2056:     return TCGEN05_ST_OPCODE(32x32b, x2);
2057:   case Intrinsic::nvvm_tcgen05_st_32x32b_x4:
2058:     return TCGEN05_ST_OPCODE(32x32b, x4);
2059:   case Intrinsic::nvvm_tcgen05_st_32x32b_x8:
2060:     return TCGEN05_ST_OPCODE(32x32b, x8);
2061:   case Intrinsic::nvvm_tcgen05_st_32x32b_x16:
2062:     return TCGEN05_ST_OPCODE(32x32b, x16);
2063:   case Intrinsic::nvvm_tcgen05_st_32x32b_x32:
2064:     return TCGEN05_ST_OPCODE(32x32b, x32);
2065:   case Intrinsic::nvvm_tcgen05_st_32x32b_x64:
2066:     return TCGEN05_ST_OPCODE(32x32b, x64);
2067:   case Intrinsic::nvvm_tcgen05_st_32x32b_x128:
2068:     return TCGEN05_ST_OPCODE(32x32b, x128);
2069:   }
2070:   llvm_unreachable("unhandled tcgen05.st lowering");
```
- EN: This range implements operational logic in helpers such as ReplaceNode, getTcgen05StOpcode, TCGEN05_ST_OPCODE, llvm_unreachable, translating backend policy into executable code.
- CN: 这一段实现了 ReplaceNode、getTcgen05StOpcode、TCGEN05_ST_OPCODE、llvm_unreachable 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2071-2160
```cpp
2071: }
2072:
2073: void NVPTXDAGToDAGISel::SelectTcgen05St(SDNode *N, bool hasOffset) {
2074:   if (!Subtarget->hasTcgen05InstSupport())
2075:     report_fatal_error(
2076:         "tcgen05.st is not supported on this architecture variant");
2077:
2078:   SDLoc DL(N);
2079:   unsigned IID = cast<ConstantSDNode>(N->getOperand(1))->getZExtValue();
2080:
2081:   SmallVector<SDValue, 128> Operands = {
2082:       N->getOperand(2) // taddr
2083:   };
2084:
2085:   if (hasOffset)
2086:     Operands.push_back(CurDAG->getTargetConstant(
2087:         cast<ConstantSDNode>(N->getOperand(3))->getZExtValue(), DL,
2088:         MVT::i32)); // Offset
2089:
2090:   for (unsigned I = hasOffset ? 4 : 3; I < (N->getNumOperands() - 1); I++)
2091:     Operands.push_back(N->getOperand(I));
2092:
2093:   bool enableUnpack =
2094:       cast<ConstantSDNode>(N->getOperand(N->getNumOperands() - 1))
2095:           ->getZExtValue();
2096:
2097:   Operands.push_back(N->getOperand(0)); // Chain
2098:   ReplaceNode(N, CurDAG->getMachineNode(getTcgen05StOpcode(IID, enableUnpack),
2099:                                         DL, N->getVTList(), Operands));
2100: }
2101:
2102: bool NVPTXDAGToDAGISel::tryIntrinsicVoid(SDNode *N) {
2103:   unsigned IID = N->getConstantOperandVal(1);
2104:   using TMARedTy = llvm::nvvm::TMAReductionOp;
2105:   auto CastTy = [](TMARedTy Op) { return static_cast<unsigned>(Op); };
2106:   switch (IID) {
2107:   default:
2108:     return false;
2109:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_tile_1d:
2110:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_tile_2d:
2111:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_tile_3d:
2112:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_tile_4d:
2113:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_tile_5d:
2114:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::ADD));
2115:     return true;
2116:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_im2col_3d:
2117:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_im2col_4d:
2118:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_add_im2col_5d:
2119:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::ADD),
2120:                                         /*IsIm2Col=*/true);
2121:     return true;
2122:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_tile_1d:
2123:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_tile_2d:
2124:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_tile_3d:
2125:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_tile_4d:
2126:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_tile_5d:
2127:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::MIN));
2128:     return true;
2129:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_im2col_3d:
2130:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_im2col_4d:
2131:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_min_im2col_5d:
2132:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::MIN),
2133:                                         /*IsIm2Col=*/true);
2134:     return true;
2135:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_tile_1d:
2136:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_tile_2d:
2137:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_tile_3d:
2138:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_tile_4d:
2139:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_tile_5d:
2140:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::MAX));
2141:     return true;
2142:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_im2col_3d:
2143:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_im2col_4d:
2144:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_max_im2col_5d:
2145:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::MAX),
2146:                                         /*IsIm2Col=*/true);
2147:     return true;
2148:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_tile_1d:
2149:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_tile_2d:
2150:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_tile_3d:
2151:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_tile_4d:
2152:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_tile_5d:
2153:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::INC));
2154:     return true;
2155:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_im2col_3d:
2156:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_im2col_4d:
2157:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_inc_im2col_5d:
2158:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::INC),
2159:                                         /*IsIm2Col=*/true);
2160:     return true;
```
- EN: This range implements operational logic in helpers such as NVPTXDAGToDAGISel::SelectTcgen05St, DL, getOperand, getNumOperands, translating backend policy into executable code.
- CN: 这一段实现了 NVPTXDAGToDAGISel::SelectTcgen05St、DL、getOperand、getNumOperands 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2161-2250
```cpp
2161:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_tile_1d:
2162:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_tile_2d:
2163:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_tile_3d:
2164:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_tile_4d:
2165:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_tile_5d:
2166:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::DEC));
2167:     return true;
2168:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_im2col_3d:
2169:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_im2col_4d:
2170:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_dec_im2col_5d:
2171:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::DEC),
2172:                                         /*IsIm2Col=*/true);
2173:     return true;
2174:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_tile_1d:
2175:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_tile_2d:
2176:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_tile_3d:
2177:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_tile_4d:
2178:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_tile_5d:
2179:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::AND));
2180:     return true;
2181:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_im2col_3d:
2182:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_im2col_4d:
2183:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_and_im2col_5d:
2184:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::AND),
2185:                                         /*IsIm2Col=*/true);
2186:     return true;
2187:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_tile_1d:
2188:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_tile_2d:
2189:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_tile_3d:
2190:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_tile_4d:
2191:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_tile_5d:
2192:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::OR));
2193:     return true;
2194:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_im2col_3d:
2195:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_im2col_4d:
2196:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_or_im2col_5d:
2197:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::OR),
2198:                                         /*IsIm2Col=*/true);
2199:     return true;
2200:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_tile_1d:
2201:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_tile_2d:
2202:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_tile_3d:
2203:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_tile_4d:
2204:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_tile_5d:
2205:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::XOR));
2206:     return true;
2207:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_im2col_3d:
2208:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_im2col_4d:
2209:   case Intrinsic::nvvm_cp_async_bulk_tensor_reduce_xor_im2col_5d:
2210:     SelectCpAsyncBulkTensorReduceCommon(N, CastTy(TMARedTy::XOR),
2211:                                         /*IsIm2Col=*/true);
2212:     return true;
2213:
2214:   case Intrinsic::nvvm_tcgen05_st_16x64b_x1:
2215:   case Intrinsic::nvvm_tcgen05_st_16x64b_x2:
2216:   case Intrinsic::nvvm_tcgen05_st_16x64b_x4:
2217:   case Intrinsic::nvvm_tcgen05_st_16x64b_x8:
2218:   case Intrinsic::nvvm_tcgen05_st_16x64b_x16:
2219:   case Intrinsic::nvvm_tcgen05_st_16x64b_x32:
2220:   case Intrinsic::nvvm_tcgen05_st_16x64b_x64:
2221:   case Intrinsic::nvvm_tcgen05_st_16x64b_x128:
2222:   case Intrinsic::nvvm_tcgen05_st_32x32b_x1:
2223:   case Intrinsic::nvvm_tcgen05_st_32x32b_x2:
2224:   case Intrinsic::nvvm_tcgen05_st_32x32b_x4:
2225:   case Intrinsic::nvvm_tcgen05_st_32x32b_x8:
2226:   case Intrinsic::nvvm_tcgen05_st_32x32b_x16:
2227:   case Intrinsic::nvvm_tcgen05_st_32x32b_x32:
2228:   case Intrinsic::nvvm_tcgen05_st_32x32b_x64:
2229:   case Intrinsic::nvvm_tcgen05_st_32x32b_x128:
2230:   case Intrinsic::nvvm_tcgen05_st_16x128b_x1:
2231:   case Intrinsic::nvvm_tcgen05_st_16x128b_x2:
2232:   case Intrinsic::nvvm_tcgen05_st_16x128b_x4:
2233:   case Intrinsic::nvvm_tcgen05_st_16x128b_x8:
2234:   case Intrinsic::nvvm_tcgen05_st_16x128b_x16:
2235:   case Intrinsic::nvvm_tcgen05_st_16x128b_x32:
2236:   case Intrinsic::nvvm_tcgen05_st_16x128b_x64:
2237:   case Intrinsic::nvvm_tcgen05_st_16x256b_x1:
2238:   case Intrinsic::nvvm_tcgen05_st_16x256b_x2:
2239:   case Intrinsic::nvvm_tcgen05_st_16x256b_x4:
2240:   case Intrinsic::nvvm_tcgen05_st_16x256b_x8:
2241:   case Intrinsic::nvvm_tcgen05_st_16x256b_x16:
2242:   case Intrinsic::nvvm_tcgen05_st_16x256b_x32: {
2243:     SelectTcgen05St(N);
2244:     return true;
2245:   }
2246:
2247:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x1:
2248:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x2:
2249:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x4:
2250:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x8:
```
- EN: This range implements operational logic in helpers such as SelectCpAsyncBulkTensorReduceCommon, SelectTcgen05St, translating backend policy into executable code.
- CN: 这一段实现了 SelectCpAsyncBulkTensorReduceCommon、SelectTcgen05St 等操作逻辑，把后端策略落实为可执行的代码路径。

### Lines 2251-2321
```cpp
2251:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x16:
2252:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x32:
2253:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x64:
2254:   case Intrinsic::nvvm_tcgen05_st_16x32bx2_x128: {
2255:     SelectTcgen05St(N, /*  hasOffset */ true);
2256:     return true;
2257:   }
2258:   }
2259: }
2260:
2261: void NVPTXDAGToDAGISel::selectAtomicSwap128(SDNode *N) {
2262:   MemSDNode *AN = cast<MemSDNode>(N);
2263:   SDLoc dl(N);
2264:
2265:   const SDValue Chain = N->getOperand(0);
2266:   const auto [Base, Offset] = selectADDR(N->getOperand(1), CurDAG);
2267:   SmallVector<SDValue, 5> Ops{Base, Offset};
2268:   Ops.append(N->op_begin() + 2, N->op_end());
2269:   Ops.append({
2270:       getI32Imm(getMemOrder(AN), dl),
2271:       getI32Imm(getAtomicScope(AN), dl),
2272:       getI32Imm(getAddrSpace(AN), dl),
2273:       Chain,
2274:   });
2275:
2276:   assert(N->getOpcode() == NVPTXISD::ATOMIC_CMP_SWAP_B128 ||
2277:          N->getOpcode() == NVPTXISD::ATOMIC_SWAP_B128);
2278:   unsigned Opcode = N->getOpcode() == NVPTXISD::ATOMIC_SWAP_B128
2279:                         ? NVPTX::ATOM_EXCH_B128
2280:                         : NVPTX::ATOM_CAS_B128;
2281:
2282:   auto *ATOM = CurDAG->getMachineNode(Opcode, dl, N->getVTList(), Ops);
2283:   CurDAG->setNodeMemRefs(ATOM, AN->getMemOperand());
2284:
2285:   ReplaceNode(N, ATOM);
2286: }
2287:
2288: void NVPTXDAGToDAGISel::selectBR_JT(SDNode *N) {
2289:   assert(Subtarget->hasBrx() &&
2290:          "BR_JT should be expanded during legalization on unsupported targets");
2291:
2292:   SDLoc DL(N);
2293:   const SDValue InChain = N->getOperand(0);
2294:   const auto *JT = cast<JumpTableSDNode>(N->getOperand(1));
2295:   const SDValue Index = N->getOperand(2);
2296:
2297:   unsigned JId = JT->getIndex();
2298:   MachineJumpTableInfo *MJTI = CurDAG->getMachineFunction().getJumpTableInfo();
2299:   ArrayRef<MachineBasicBlock *> MBBs = MJTI->getJumpTables()[JId].MBBs;
2300:
2301:   SDValue IdV = getI32Imm(JId, DL);
2302:
2303:   // Generate BrxStart node
2304:   MachineSDNode *Chain = CurDAG->getMachineNode(
2305:       NVPTX::BRX_START, DL, {MVT::Other, MVT::Glue}, {IdV, InChain});
2306:
2307:   // Generate BrxItem nodes
2308:   assert(!MBBs.empty());
2309:   for (MachineBasicBlock *MBB : MBBs.drop_back())
2310:     Chain = CurDAG->getMachineNode(
2311:         NVPTX::BRX_ITEM, DL, {MVT::Other, MVT::Glue},
2312:         {CurDAG->getBasicBlock(MBB), SDValue(Chain, 0), SDValue(Chain, 1)});
2313:
2314:   // Generate BrxEnd nodes
2315:   MachineSDNode *BrxEnd =
2316:       CurDAG->getMachineNode(NVPTX::BRX_END, DL, MVT::Other,
2317:                              {CurDAG->getBasicBlock(MBBs.back()), Index, IdV,
2318:                               SDValue(Chain, 0), SDValue(Chain, 1)});
2319:
2320:   ReplaceNode(N, BrxEnd);
2321: }
```
- EN: This range implements operational logic in helpers such as SelectTcgen05St, NVPTXDAGToDAGISel::selectAtomicSwap128, dl, getOperand, translating backend policy into executable code.
- CN: 这一段实现了 SelectTcgen05St、NVPTXDAGToDAGISel::selectAtomicSwap128、dl、getOperand 等操作逻辑，把后端策略落实为可执行的代码路径。

## Key Concepts / 关键概念

- EN: Instruction selection maps generic LLVM operations onto target-specific machine instructions.
  - CN: 指令选择负责把通用 LLVM 操作映射为目标相关的机器指令。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include cl::desc, NVPTXDAGToDAGISelLegacy, INITIALIZE_PASS, SelectionDAGISel, NVPTXDAGToDAGISel::runOnMachineFunction, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 cl::desc, NVPTXDAGToDAGISelLegacy, INITIALIZE_PASS, SelectionDAGISel, NVPTXDAGToDAGISel::runOnMachineFunction，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXISelDAGToDAG.h`
  - `NVPTX.h`
  - `NVPTXUtilities.h`
- LLVM infrastructure / LLVM 基础设施:
  - `llvm/ADT/APInt.h`
  - `llvm/Analysis/ValueTracking.h`
  - `llvm/CodeGen/ISDOpcodes.h`
  - `llvm/CodeGen/MachineJumpTableInfo.h`
  - `llvm/CodeGen/SelectionDAG.h`
  - `llvm/CodeGen/SelectionDAGNodes.h`
  - `llvm/IR/GlobalValue.h`
  - `llvm/IR/Instructions.h`
  - `llvm/IR/IntrinsicsNVPTX.h`
  - `llvm/IR/NVVMIntrinsicUtils.h`
  - `llvm/Support/AtomicOrdering.h`
  - `llvm/Support/CommandLine.h`
  - `llvm/Support/ErrorHandling.h`
  - `llvm/Support/FormatVariadic.h`
  - `llvm/Support/MathExtras.h`
- System/standard headers / 系统或标准头文件:
  - `optional`
