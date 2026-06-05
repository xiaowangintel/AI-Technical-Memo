# NVPTXInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/NVPTX/NVPTXInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the PTX instructions in TableGen format.
- 目的（中文）: 该文件使用 TableGen DSL 描述目标后端元数据，并驱动自动生成代码。
- Language: TableGen DSL (TableGen DSL noted below)
- Note: This file is written in TableGen DSL, so many records are declarative descriptions consumed by LLVM code generators.
- 说明：该文件使用 TableGen DSL 编写，因此许多记录都是供 LLVM 代码生成器消费的声明式描述。

## Line-by-Line Analysis / 逐行分析

> Note / 说明: To keep the document readable, the source is analyzed in contiguous line ranges while preserving the original order. Each code block prefixes original line numbers.

### Lines 1-120
```tablegen
  1: //===- NVPTXInstrInfo.td - NVPTX Instruction defs -------------*- tblgen-*-===//
  2: //
  3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
  4: // See https://llvm.org/LICENSE.txt for license information.
  5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
  6: //
  7: //===----------------------------------------------------------------------===//
  8: //
  9: // This file describes the PTX instructions in TableGen format.
 10: //
 11: //===----------------------------------------------------------------------===//
 12:
 13: include "NVPTXInstrFormats.td"
 14:
 15: let OperandType = "OPERAND_IMMEDIATE" in {
 16:   def f16imm : Operand<f16>;
 17:   def bf16imm : Operand<bf16>;
 18: }
 19:
 20: //===----------------------------------------------------------------------===//
 21: // NVPTX Operand Definitions.
 22: //===----------------------------------------------------------------------===//
 23:
 24: def brtarget    : Operand<OtherVT>;
 25:
 26: // CVT conversion modes
 27: // These must match the enum in NVPTX.h
 28: def CvtNONE : PatLeaf<(i32 0x0)>;
 29: def CvtRNI  : PatLeaf<(i32 0x1)>;
 30: def CvtRZI  : PatLeaf<(i32 0x2)>;
 31: def CvtRMI  : PatLeaf<(i32 0x3)>;
 32: def CvtRPI  : PatLeaf<(i32 0x4)>;
 33: def CvtRN   : PatLeaf<(i32 0x5)>;
 34: def CvtRZ   : PatLeaf<(i32 0x6)>;
 35: def CvtRM   : PatLeaf<(i32 0x7)>;
 36: def CvtRP   : PatLeaf<(i32 0x8)>;
 37: def CvtRNA  : PatLeaf<(i32 0x9)>;
 38: def CvtRS   : PatLeaf<(i32 0xA)>;
 39:
 40: def CvtNONE_FTZ : PatLeaf<(i32 0x10)>;
 41: def CvtRNI_FTZ  : PatLeaf<(i32 0x11)>;
 42: def CvtRZI_FTZ  : PatLeaf<(i32 0x12)>;
 43: def CvtRMI_FTZ  : PatLeaf<(i32 0x13)>;
 44: def CvtRPI_FTZ  : PatLeaf<(i32 0x14)>;
 45: def CvtRN_FTZ   : PatLeaf<(i32 0x15)>;
 46: def CvtRZ_FTZ   : PatLeaf<(i32 0x16)>;
 47: def CvtRM_FTZ   : PatLeaf<(i32 0x17)>;
 48: def CvtRP_FTZ   : PatLeaf<(i32 0x18)>;
 49:
 50: def CvtSAT      : PatLeaf<(i32 0x20)>;
 51: def CvtSAT_FTZ  : PatLeaf<(i32 0x30)>;
 52:
 53: def CvtNONE_RELU   : PatLeaf<(i32 0x40)>;
 54: def CvtRN_RELU     : PatLeaf<(i32 0x45)>;
 55: def CvtRZ_RELU     : PatLeaf<(i32 0x46)>;
 56: def CvtRS_RELU     : PatLeaf<(i32 0x4A)>;
 57:
 58: def CvtNONE_SATFINITE  : PatLeaf<(i32 0x80)>;
 59: def CvtRN_SATFINITE   : PatLeaf<(i32 0x85)>;
 60: def CvtRN_RELU_SATFINITE : PatLeaf<(i32 0xC5)>;
 61:
 62: def CvtMode : Operand<i32> {
 63:   let PrintMethod = "printCvtMode";
 64: }
 65:
 66: // FTZ flag
 67:
 68: def FTZ : PatLeaf<(i1 1)>;
 69: def NoFTZ : PatLeaf<(i1 0)>;
 70:
 71: def getFTZFlag : SDNodeXForm<imm, [{
 72:   (void)N;
 73:   return CurDAG->getTargetConstant(useF32FTZ() ? 1 : 0, SDLoc(), MVT::i1);
 74: }]>;
 75:
 76: def FTZFlag : OperandWithDefaultOps<i1, (ops (getFTZFlag (i1 0)))> {
 77:   let PrintMethod = "printFTZFlag";
 78: }
 79:
 80: // Flag to indicate predicate negation (@!p) for conditional branches.
 81: def BranchFlag : OperandWithDefaultOps<i1, (ops (i1 0))> {
 82:   let PrintMethod = "printNegatedPredicate";
 83: }
 84:
 85: // Compare modes
 86: // These must match the enum in NVPTX.h
 87: def CmpEQ : PatLeaf<(i32 0)>;
 88: def CmpNE : PatLeaf<(i32 1)>;
 89:
 90: def CmpMode : Operand<i32> {
 91:   let PrintMethod = "printCmpMode";
 92: }
 93:
 94: // PRMT modes
 95: // These must match the enum in NVPTX.h
 96: def PrmtNONE : PatLeaf<(i32 0x0)>;
 97: def PrmtF4E  : PatLeaf<(i32 0x1)>;
 98: def PrmtB4E  : PatLeaf<(i32 0x2)>;
 99: def PrmtRC8  : PatLeaf<(i32 0x3)>;
100: def PrmtECL  : PatLeaf<(i32 0x4)>;
101: def PrmtECR  : PatLeaf<(i32 0x5)>;
102: def PrmtRC16 : PatLeaf<(i32 0x6)>;
103:
104: def PrmtMode : Operand<i32> {
105:   let PrintMethod = "printPrmtMode";
106: }
107:
108:
109: //===----------------------------------------------------------------------===//
110: // NVPTX Instruction Predicate Definitions
111: //===----------------------------------------------------------------------===//
112:
113: // Checks PTX version and family-specific and architecture-specific SM versions.
114: // For example, sm_100{f/a} and any future variants in the same family will match
115: // for any PTX version greater than or equal to `PTXVersion`.
116: class PTXWithFamilySMs<int PTXVersion, list<int> SMVersions> :
117:   Predicate<"Subtarget->hasPTXWithFamilySMs(" # PTXVersion # ", {" #
118:             !interleave(SMVersions, ", ") # "})">;
119:
120: // Checks PTX version and architecture-specific SM versions.
```
- EN: This range uses TableGen DSL to describe records such as f16imm, bf16imm, brtarget, CvtNONE; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 f16imm、bf16imm、brtarget、CvtNONE 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 121-240
```tablegen
121: // For example, sm_100{a} will match for any PTX version
122: // greater than or equal to `PTXVersion`.
123: class PTXWithAccelSMs<int PTXVersion, list<int> SMVersions> :
124:   Predicate<"Subtarget->hasPTXWithAccelSMs(" # PTXVersion # ", {" #
125:             !interleave(SMVersions, ", ") # "})">;
126:
127: // Helper predicate to call a subtarget method.
128: class callSubtarget<string SubtargetMethod> : Predicate<"Subtarget->" # SubtargetMethod # "()">;
129:
130: def hasAtomAddF64 : Predicate<"Subtarget->hasAtomAddF64()">;
131: def hasAtomScope : Predicate<"Subtarget->hasAtomScope()">;
132: def hasAtomBitwise64 : Predicate<"Subtarget->hasAtomBitwise64()">;
133: def hasAtomMinMax64 : Predicate<"Subtarget->hasAtomMinMax64()">;
134: def hasAtomSwap128 : Predicate<"Subtarget->hasAtomSwap128()">;
135: def hasClusters : Predicate<"Subtarget->hasClusters()">;
136: def hasPTXASUnreachableBug : Predicate<"Subtarget->hasPTXASUnreachableBug()">;
137: def noPTXASUnreachableBug : Predicate<"!Subtarget->hasPTXASUnreachableBug()">;
138: def hasOptEnabled : Predicate<"TM.getOptLevel() != CodeGenOptLevel::None">;
139: def hasArchAccelFeatures : Predicate<"Subtarget->hasArchAccelFeatures()">;
140: def hasFamilySpecificFeatures : Predicate<"Subtarget->hasFamilySpecificFeatures()">;
141:
142: def doF32FTZ : Predicate<"useF32FTZ()">;
143: def doNoF32FTZ : Predicate<"!useF32FTZ()">;
144: def doRsqrtOpt : Predicate<"doRsqrtOpt()">;
145: def doMADWideOpt : Predicate<"doMADWideOpt()">;
146:
147: def hasHWROT32 : Predicate<"Subtarget->hasHWROT32()">;
148: def noHWROT32 : Predicate<"!Subtarget->hasHWROT32()">;
149: def hasDotInstructions : Predicate<"Subtarget->hasDotInstructions()">;
150: def hasF32x2Instructions : Predicate<"Subtarget->hasF32x2Instructions()">;
151:
152: class hasPTX<int version>: Predicate<"Subtarget->getPTXVersion() >= " # version>;
153: class hasSM<int version>: Predicate<"Subtarget->getSmVersion() >= " # version>;
154:
155: // Explicit records for arch-accelerated SM versions
156: def hasSM90a : Predicate<"Subtarget->getSmVersion() == 90 && Subtarget->hasArchAccelFeatures()">;
157: def hasSM100a : Predicate<"Subtarget->getSmVersion() == 100 && Subtarget->hasArchAccelFeatures()">;
158: def hasSM101a : Predicate<"Subtarget->getSmVersion() == 101 && Subtarget->hasArchAccelFeatures()">;
159: def hasSM120a : Predicate<"Subtarget->getSmVersion() == 120 && Subtarget->hasArchAccelFeatures()">;
160:
161: def hasSM100aOrSM103a :
162:   Predicate<"(Subtarget->getSmVersion() == 100 || " # 
163:               "Subtarget->getSmVersion() == 103) " #
164:             "&& Subtarget->hasArchAccelFeatures()">;
165:
166: // non-sync shfl instructions are not available on sm_70+ in PTX6.4+
167: def hasSHFL : Predicate<"!(Subtarget->getSmVersion() >= 70"
168:                           "&& Subtarget->getPTXVersion() >= 64)">;
169:
170: def useFP16Math: Predicate<"Subtarget->allowFP16Math()">;
171: def hasBF16Math: Predicate<"Subtarget->hasBF16Math()">;
172:
173:
174: //===----------------------------------------------------------------------===//
175: // Some Common Instruction Class Templates
176: //===----------------------------------------------------------------------===//
177:
178: class OneUse1<SDPatternOperator operator>
179:     : PatFrag<(ops node:$A), (operator node:$A), [{ return N->hasOneUse(); }]>;
180: class OneUse2<SDPatternOperator operator>
181:     : PatFrag<(ops node:$A, node:$B), (operator node:$A, node:$B), [{ return N->hasOneUse(); }]>;
182:
183:
184: class zeroinitializer<ValueType vt> : 
185:   PatLeaf<(vt (bitconvert (!cast<ValueType>("i" # vt.Size) 0)))>;
186:
187:
188: def fpimm_pos_inf : FPImmLeaf<fAny, [{ return Imm.isPosInfinity(); }]>;
189: def fpimm_0 : FPImmLeaf<fAny, [{ return Imm.isZero(); }]>;
190: def fpimm_1 : FPImmLeaf<fAny, [{ return Imm.isExactlyValue(1.0); }]>;
191: def fpimm_neg_1 : FPImmLeaf<fAny, [{ return Imm.isExactlyValue(-1.0); }]>;
192:
193:
194: // Operands which can hold a Register or an Immediate.
195: //
196: // Unfortunately, since most register classes can hold multiple types, we must
197: // use the 'Any' type for these.
198:
199: def RI1  : Operand<i1>;
200: def RI16 : Operand<Any>;
201: def RI32 : Operand<Any>;
202: def RI64 : Operand<Any>;
203:
204: // Utility class to wrap up information about a register and DAG type for more
205: // convenient iteration and parameterization
206: class RegTyInfo<ValueType ty, NVPTXRegClass rc, string ptx_type, Operand imm, SDNode imm_node,
207:                 bit supports_imm = 1> {
208:   ValueType Ty = ty;
209:   NVPTXRegClass RC = rc;
210:   Operand Imm = imm;
211:   SDNode ImmNode = imm_node;
212:   bit SupportsImm = supports_imm;
213:   int Size = ty.Size;
214:   string PtxType = ptx_type;
215: }
216:
217: def I1RT     : RegTyInfo<i1,  B1,  "pred", i1imm,  imm>;
218: def I16RT    : RegTyInfo<i16, B16, "b16",  i16imm, imm>;
219: def I32RT    : RegTyInfo<i32, B32, "b32",  i32imm, imm>;
220: def I64RT    : RegTyInfo<i64, B64, "b64",  i64imm, imm>;
221:
222: def F32RT    : RegTyInfo<f32,  B32, "f32",  f32imm,  fpimm>;
223: def F64RT    : RegTyInfo<f64,  B64, "f64",  f64imm,  fpimm>;
224: def F16RT    : RegTyInfo<f16,  B16, "f16",  f16imm,  fpimm, supports_imm = 0>;
225: def BF16RT   : RegTyInfo<bf16, B16, "bf16", bf16imm, fpimm, supports_imm = 0>;
226:
227: def F16X2RT  : RegTyInfo<v2f16, B32, "f16x2", ?, ?, supports_imm = 0>;
228: def BF16X2RT : RegTyInfo<v2bf16, B32, "bf16x2", ?, ?, supports_imm = 0>;
229: def F32X2RT  : RegTyInfo<v2f32, B64, "f32x2", ?, ?, supports_imm = 0>;
230:
231:
232: // This class provides a basic wrapper around an NVPTXInst that abstracts the
233: // specific syntax of most PTX instructions. It automatically handles the
234: // construction of the asm string based on the provided dag arguments.
235: // For example, the following asm-strings would be computed:
236: //
237: //   * BasicFlagsNVPTXInst<(outs B32:$dst),
238: //                         (ins B32:$a, B32:$b), (ins),
239: //                         "add.s32">;
240: //         ---> "add.s32 \t$dst, $a, $b;"
```
- EN: This range uses TableGen DSL to describe records such as PTXWithAccelSMs, callSubtarget, hasAtomAddF64, hasAtomScope; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 PTXWithAccelSMs、callSubtarget、hasAtomAddF64、hasAtomScope 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 241-360
```tablegen
241: //
242: //   * BasicFlagsNVPTXInst<(outs B32:$d),
243: //                         (ins B32:$a, B32:$b, Hexu32imm:$c),
244: //                         (ins PrmtMode:$mode),
245: //                         "prmt.b32${mode}">;
246: //         ---> "prmt.b32${mode} \t$d, $a, $b, $c;"
247: //
248: //   * BasicFlagsNVPTXInst<(outs B64:$state),
249: //                         (ins ADDR:$addr),
250: //                         "mbarrier.arrive.b64">;
251: //         ---> "mbarrier.arrive.b64 \t$state, [$addr];"
252: //
253: class BasicFlagsNVPTXInst<dag outs_dag, dag ins_dag, dag flags_dag, string asmstr,
254:                           list<dag> pattern = []>
255:   : NVPTXInst<
256:       outs_dag,
257:       !con(ins_dag, flags_dag),
258:       !strconcat(
259:         asmstr,
260:         !if(!and(!empty(ins_dag), !empty(outs_dag)), "",
261:           !strconcat(
262:             " \t",
263:             !interleave(
264:               !foreach(i, !range(!size(outs_dag)),
265:                 "$" # !getdagname(outs_dag, i)),
266:               "|"),
267:             !if(!or(!empty(ins_dag), !empty(outs_dag)), "", ", "),
268:             !interleave(
269:               !foreach(i, !range(!size(ins_dag)),
270:                  !if(!eq(!cast<string>(!getdagarg<DAGOperand>(ins_dag, i)), "ADDR"),
271:                     "[$" # !getdagname(ins_dag, i) # "]",
272:                     "$" # !getdagname(ins_dag, i)
273:                  )
274:                 ),
275:               ", "))),
276:         ";"),
277:       pattern>;
278:
279: class BasicNVPTXInst<dag outs, dag insv, string asmstr, list<dag> pattern = []>
280:   : BasicFlagsNVPTXInst<outs, insv, (ins), asmstr, pattern>;
281:
282:
283: multiclass I3Inst<string op_str, SDPatternOperator op_node, RegTyInfo t,
284:                   bit commutative, list<Predicate> requires = []> {
285:   def rr :
286:     BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, t.RC:$b),
287:               op_str,
288:               [(set t.Ty:$dst, (op_node t.Ty:$a, t.Ty:$b))]>,
289:               Requires<requires>;
290:   def ri :
291:     BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, t.Imm:$b),
292:               op_str,
293:               [(set t.Ty:$dst, (op_node t.Ty:$a, (t.Ty imm:$b)))]>,
294:               Requires<requires>;
295:   if !not(commutative) then
296:     def ir :
297:       BasicNVPTXInst<(outs t.RC:$dst), (ins t.Imm:$a, t.RC:$b),
298:                 op_str,
299:                 [(set t.Ty:$dst, (op_node (t.Ty imm:$a), t.Ty:$b))]>,
300:                 Requires<requires>;
301: }
302:
303: // Template for instructions which take three int64, int32, or int16 args.
304: // The instructions are named "<OpcStr><Width>" (e.g. "add.s64").
305: multiclass I3<string op_str, SDPatternOperator op_node, bit commutative> {
306:   foreach t = [I16RT, I32RT, I64RT] in
307:     defm t.Size# : I3Inst<op_str # t.Size, op_node, t, commutative>;
308: }
309:
310: class I16x2<string OpcStr, SDNode OpNode> :
311:   BasicNVPTXInst<(outs B32:$dst), (ins B32:$a, B32:$b),
312:               OpcStr # "16x2",
313:               [(set v2i16:$dst, (OpNode v2i16:$a, v2i16:$b))]>,
314:               Requires<[hasPTX<80>, hasSM<90>]>;
315:
316: // Template for instructions which take 3 int args.  The instructions are
317: // named "<OpcStr>.s32" (e.g. "addc.cc.s32").
318: multiclass ADD_SUB_INT_CARRY<string op_str, SDNode op_node, bit commutative> {
319:   let hasSideEffects = 1 in {
320:     defm i32 : I3Inst<op_str # ".s32", op_node, I32RT, commutative>;
321:     defm i64 : I3Inst<op_str # ".s64", op_node, I64RT, commutative,
322:                      requires = [hasPTX<43>]>;
323:   }
324: }
325:
326: // Template for minimum/maximum instructions.
327: //
328: // Also defines ftz (flush subnormal inputs and results to sign-preserving
329: // zero) variants for fp32 functions.
330: multiclass FMINIMUMMAXIMUM<string OpcStr, bit NaN, SDPatternOperator OpNode> {
331:   defvar nan_str = !if(NaN, ".NaN", "");
332:   if !not(NaN) then {
333:    def _f64_rr :
334:      BasicNVPTXInst<(outs B64:$dst),
335:                (ins B64:$a, B64:$b),
336:                OpcStr # ".f64",
337:                [(set f64:$dst, (OpNode f64:$a, f64:$b))]>;
338:    def _f64_ri :
339:      BasicNVPTXInst<(outs B64:$dst),
340:                (ins B64:$a, f64imm:$b),
341:                OpcStr # ".f64",
342:                [(set f64:$dst, (OpNode f64:$a, fpimm:$b))]>;
343:   }
344:    def _f32_rr :
345:      BasicFlagsNVPTXInst<(outs B32:$dst),
346:                (ins B32:$a, B32:$b),
347:                (ins FTZFlag:$ftz),
348:                OpcStr # "$ftz" # nan_str # ".f32",
349:                [(set f32:$dst, (OpNode f32:$a, f32:$b))]>;
350:    def _f32_ri :
351:      BasicFlagsNVPTXInst<(outs B32:$dst),
352:                (ins B32:$a, f32imm:$b),
353:                (ins FTZFlag:$ftz),
354:                OpcStr # "$ftz" # nan_str # ".f32",
355:                [(set f32:$dst, (OpNode f32:$a, fpimm:$b))]>;
356:
357:    def _f16_rr :
358:      BasicFlagsNVPTXInst<(outs B16:$dst),
359:                (ins B16:$a, B16:$b),
360:                (ins FTZFlag:$ftz),
```
- EN: This range uses TableGen DSL to describe records such as BasicFlagsNVPTXInst, BasicNVPTXInst, I3Inst, rr; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 BasicFlagsNVPTXInst、BasicNVPTXInst、I3Inst、rr 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 361-480
```tablegen
361:                OpcStr # "$ftz" # nan_str # ".f16",
362:                [(set f16:$dst, (OpNode f16:$a, f16:$b))]>,
363:                Requires<[useFP16Math]>;
364:
365:    def _f16x2_rr :
366:      BasicFlagsNVPTXInst<(outs B32:$dst),
367:                (ins B32:$a, B32:$b),
368:                (ins FTZFlag:$ftz),
369:                OpcStr # "$ftz" # nan_str # ".f16x2",
370:                [(set v2f16:$dst, (OpNode v2f16:$a, v2f16:$b))]>,
371:                Requires<[useFP16Math, hasSM<80>, hasPTX<70>]>;
372:    def _bf16_rr :
373:      BasicNVPTXInst<(outs B16:$dst),
374:                (ins B16:$a, B16:$b),
375:                OpcStr # nan_str # ".bf16",
376:                [(set bf16:$dst, (OpNode bf16:$a, bf16:$b))]>,
377:                Requires<[hasBF16Math, hasSM<80>, hasPTX<70>]>;
378:    def _bf16x2_rr :
379:      BasicNVPTXInst<(outs B32:$dst),
380:                (ins B32:$a, B32:$b),
381:                OpcStr # nan_str # ".bf16x2",
382:                [(set v2bf16:$dst, (OpNode v2bf16:$a, v2bf16:$b))]>,
383:                Requires<[hasBF16Math, hasSM<80>, hasPTX<70>]>;
384: }
385:
386: // Template for 3-input minimum/maximum instructions
387: // (sm_100+/PTX 8.8 and f32 only)
388: //
389: // Also defines ftz (flush subnormal inputs and results to sign-preserving
390: // zero) variants for fp32 functions.
391: multiclass FMINIMUMMAXIMUM3<string OpcStr, bit NaN, SDNode OpNode> {
392:   defvar nan_str = !if(NaN, ".NaN", "");
393:    def f32rrr :
394:      BasicFlagsNVPTXInst<(outs B32:$dst),
395:                (ins B32:$a, B32:$b, B32:$c),
396:                (ins FTZFlag:$ftz),
397:                OpcStr # "$ftz" # nan_str # ".f32",
398:                [(set f32:$dst, (OpNode f32:$a, f32:$b, f32:$c))]>,
399:                Requires<[hasPTX<88>, hasSM<100>]>;
400:    def f32rri :
401:      BasicFlagsNVPTXInst<(outs B32:$dst),
402:                (ins B32:$a, B32:$b, f32imm:$c),
403:                (ins FTZFlag:$ftz),
404:                OpcStr # "$ftz" # nan_str # ".f32",
405:                [(set f32:$dst, (OpNode f32:$a, f32:$b, fpimm:$c))]>,
406:                Requires<[hasPTX<88>, hasSM<100>]>;
407:    def f32rii :
408:      BasicFlagsNVPTXInst<(outs B32:$dst),
409:                (ins B32:$a, f32imm:$b, f32imm:$c),
410:                (ins FTZFlag:$ftz),
411:                OpcStr # "$ftz" # nan_str # ".f32",
412:                [(set f32:$dst, (OpNode f32:$a, fpimm:$b, fpimm:$c))]>,
413:                Requires<[hasPTX<88>, hasSM<100>]>;
414: }
415:
416: // Template for instructions which take three FP args.  The
417: // instructions are named "<OpcStr>.f<Width>" (e.g. "add.f64").
418: //
419: // Also defines ftz (flush subnormal inputs and results to sign-preserving
420: // zero) variants for fp32/fp16 functions.
421: //
422: // This multiclass should be used for nodes that can be folded to make fma ops.
423: // In this case, we use the ".rn" variant when FMA is disabled, as this behaves
424: // just like the non ".rn" op, but prevents ptxas from creating FMAs.
425: multiclass F3<string op_str, SDPatternOperator op_pat> {
426:   def f64rr :
427:     BasicNVPTXInst<(outs B64:$dst),
428:               (ins B64:$a, B64:$b),
429:               op_str # ".f64",
430:               [(set f64:$dst, (op_pat f64:$a, f64:$b))]>;
431:   def f64ri :
432:     BasicNVPTXInst<(outs B64:$dst),
433:               (ins B64:$a, f64imm:$b),
434:               op_str # ".f64",
435:               [(set f64:$dst, (op_pat f64:$a, fpimm:$b))]>;
436:   def f32rr :
437:     BasicFlagsNVPTXInst<(outs B32:$dst),
438:               (ins B32:$a, B32:$b),
439:               (ins FTZFlag:$ftz),
440:               op_str # "$ftz.f32",
441:               [(set f32:$dst, (op_pat f32:$a, f32:$b))]>;
442:   def f32ri :
443:     BasicFlagsNVPTXInst<(outs B32:$dst),
444:               (ins B32:$a, f32imm:$b),
445:               (ins FTZFlag:$ftz),
446:               op_str # "$ftz.f32",
447:               [(set f32:$dst, (op_pat f32:$a, fpimm:$b))]>;
448:
449:   def f16rr :
450:     BasicFlagsNVPTXInst<(outs B16:$dst),
451:               (ins B16:$a, B16:$b),
452:               (ins FTZFlag:$ftz),
453:               op_str # "$ftz.f16",
454:               [(set f16:$dst, (op_pat f16:$a, f16:$b))]>,
455:               Requires<[useFP16Math]>;
456:   def f32x2rr :
457:     BasicFlagsNVPTXInst<(outs B64:$dst),
458:               (ins B64:$a, B64:$b),
459:               (ins FTZFlag:$ftz),
460:               op_str # "$ftz.f32x2",
461:               [(set v2f32:$dst, (op_pat v2f32:$a, v2f32:$b))]>,
462:               Requires<[hasF32x2Instructions]>;
463:   def f16x2rr :
464:     BasicFlagsNVPTXInst<(outs B32:$dst),
465:               (ins B32:$a, B32:$b),
466:               (ins FTZFlag:$ftz),
467:               op_str # "$ftz.f16x2",
468:               [(set v2f16:$dst, (op_pat v2f16:$a, v2f16:$b))]>,
469:               Requires<[useFP16Math]>;
470:   def bf16rr :
471:     BasicNVPTXInst<(outs B16:$dst),
472:               (ins B16:$a, B16:$b),
473:               op_str # ".bf16",
474:               [(set bf16:$dst, (op_pat bf16:$a, bf16:$b))]>,
475:               Requires<[hasBF16Math]>;
476:
477:   def bf16x2rr :
478:     BasicNVPTXInst<(outs B32:$dst),
479:               (ins B32:$a, B32:$b),
480:               op_str # ".bf16x2",
```
- EN: This range uses TableGen DSL to describe records such as _f16x2_rr, _bf16_rr, _bf16x2_rr, FMINIMUMMAXIMUM3; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 _f16x2_rr、_bf16_rr、_bf16x2_rr、FMINIMUMMAXIMUM3 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 481-600
```tablegen
481:               [(set v2bf16:$dst, (op_pat v2bf16:$a, v2bf16:$b))]>,
482:               Requires<[hasBF16Math]>;
483: }
484:
485: class BinOpAllowsFMA<SDPatternOperator operator>
486:     : PatFrag<(ops node:$A, node:$B),
487:               (operator node:$A, node:$B), [{
488:   return allowFMA() || N->getFlags().hasAllowContract();
489: }]>;
490:
491: multiclass F3_fma_component<string op_str, SDNode op_node> {
492:   defm "" : F3<op_str, BinOpAllowsFMA<op_node>>;
493:   defm _rn : F3<op_str # ".rn", op_node>;
494: }
495:
496: // Template for operations which take two f32 or f64 operands.  Provides three
497: // instructions: <OpcStr>.f64, <OpcStr>.f32, and <OpcStr>.ftz.f32 (flush
498: // subnormal inputs and results to zero).
499: multiclass F2<string OpcStr, SDNode OpNode> {
500:    def f64 : BasicNVPTXInst<(outs B64:$dst), (ins B64:$a),
501:                            OpcStr # ".f64",
502:                            [(set f64:$dst, (OpNode f64:$a))]>;
503:    def f32 : BasicFlagsNVPTXInst<(outs B32:$dst), (ins B32:$a),
504:                            (ins FTZFlag:$ftz),
505:                            OpcStr # "$ftz.f32",
506:                            [(set f32:$dst, (OpNode f32:$a))]>;
507: }
508:
509: multiclass F2_Support_Half<string OpcStr, SDNode OpNode> {
510:    def bf16 :      BasicNVPTXInst<(outs B16:$dst), (ins B16:$a),
511:                            OpcStr # ".bf16",
512:                            [(set bf16:$dst, (OpNode bf16:$a))]>,
513:                            Requires<[hasSM<80>, hasPTX<70>]>;
514:    def bf16x2 :    BasicNVPTXInst<(outs B32:$dst), (ins B32:$a),
515:                            OpcStr # ".bf16x2",
516:                            [(set v2bf16:$dst, (OpNode v2bf16:$a))]>,
517:                            Requires<[hasSM<80>, hasPTX<70>]>;
518:    def f16 :       BasicFlagsNVPTXInst<(outs B16:$dst), (ins B16:$a),
519:                            (ins FTZFlag:$ftz),
520:                            OpcStr # "$ftz.f16",
521:                            [(set f16:$dst, (OpNode f16:$a))]>,
522:                            Requires<[hasSM<53>, hasPTX<65>]>;
523:    def f16x2 :     BasicFlagsNVPTXInst<(outs B32:$dst), (ins B32:$a),
524:                            (ins FTZFlag:$ftz),
525:                            OpcStr # "$ftz.f16x2",
526:                            [(set v2f16:$dst, (OpNode v2f16:$a))]>,
527:                            Requires<[hasSM<53>, hasPTX<65>]>;
528:
529: }
530:
531: //===----------------------------------------------------------------------===//
532: // NVPTX Instructions.
533: //===----------------------------------------------------------------------===//
534:
535: //-----------------------------------
536: // Type Conversion
537: //-----------------------------------
538:
539: let hasSideEffects = false in {
540:   // Generate a cvt to the given type from all possible types.  Each instance
541:   // takes a CvtMode immediate that defines the conversion mode to use.  It can
542:   // be CvtNONE to omit a conversion mode.
543:   multiclass CVT_FROM_ALL<string ToType, RegisterClass RC, list<Predicate> Preds = []> {
544:     foreach sign = ["s", "u"] in {
545:       def _ # sign # "8" :
546:         BasicFlagsNVPTXInst<(outs RC:$dst),
547:                   (ins B16:$src), (ins CvtMode:$mode),
548:                   "cvt${mode:base}${mode:ftz}${mode:sat}." # ToType # "." # sign # "8">,
549:         Requires<Preds>;
550:       def _ # sign # "16" :
551:         BasicFlagsNVPTXInst<(outs RC:$dst),
552:                   (ins B16:$src), (ins CvtMode:$mode),
553:                   "cvt${mode:base}${mode:ftz}${mode:sat}." # ToType # "." # sign # "16">,
554:         Requires<Preds>;
555:       def _ # sign # "32" :
556:         BasicFlagsNVPTXInst<(outs RC:$dst),
557:                   (ins B32:$src), (ins CvtMode:$mode),
558:                   "cvt${mode:base}${mode:ftz}${mode:sat}." # ToType # "." # sign # "32">,
559:         Requires<Preds>;
560:       def _ # sign # "64" :
561:         BasicFlagsNVPTXInst<(outs RC:$dst),
562:                   (ins B64:$src), (ins CvtMode:$mode),
563:                   "cvt${mode:base}${mode:ftz}${mode:sat}." # ToType # "." # sign # "64">,
564:         Requires<Preds>;
565:     }
566:     def _f16 :
567:       BasicFlagsNVPTXInst<(outs RC:$dst),
568:                 (ins B16:$src), (ins CvtMode:$mode),
569:                 "cvt${mode:base}${mode:ftz}${mode:sat}." # ToType # ".f16">,
570:       Requires<Preds>;
571:     def _bf16 :
572:       BasicFlagsNVPTXInst<(outs RC:$dst),
573:                 (ins B16:$src), (ins CvtMode:$mode),
574:                 "cvt${mode:base}${mode:ftz}${mode:relu}${mode:sat}." # ToType # ".bf16">,
575:       Requires<!if(!eq(ToType, "f32"),
576:                    // bf16->f32 was introduced early.
577:                    [hasPTX<71>, hasSM<80>],
578:                    // bf16->everything else needs sm90/ptx78
579:                    [hasPTX<78>, hasSM<90>])>;
580:     def _f32 :
581:       BasicFlagsNVPTXInst<(outs RC:$dst),
582:                 (ins B32:$src), (ins CvtMode:$mode),
583:                 "cvt${mode:base}${mode:ftz}${mode:relu}${mode:sat}." # ToType # ".f32">,
584:       Requires<!if(!eq(ToType, "bf16"),
585:                    // f32->bf16 was introduced early.
586:                    [hasPTX<70>, hasSM<80>],
587:                    Preds)>;
588:     def _f64 :
589:       BasicFlagsNVPTXInst<(outs RC:$dst),
590:                 (ins B64:$src), (ins CvtMode:$mode),
591:                 "cvt${mode:base}${mode:ftz}${mode:sat}." # ToType # ".f64">,
592:       Requires<Preds>;
593:   }
594:
595:   // Generate cvts from all types to all types.
596:   foreach sign = ["s", "u"] in {
597:     defm CVT_ # sign # "8"  : CVT_FROM_ALL<sign # "8",  B16>;
598:     defm CVT_ # sign # "16" : CVT_FROM_ALL<sign # "16", B16>;
599:     defm CVT_ # sign # "32" : CVT_FROM_ALL<sign # "32", B32>;
600:     defm CVT_ # sign # "64" : CVT_FROM_ALL<sign # "64", B64>;
```
- EN: This range uses TableGen DSL to describe records such as BinOpAllowsFMA, F3_fma_component, "", _rn; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 BinOpAllowsFMA、F3_fma_component、""、_rn 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 601-720
```tablegen
601:   }
602:   defm CVT_f16 : CVT_FROM_ALL<"f16", B16>;
603:   defm CVT_bf16 : CVT_FROM_ALL<"bf16", B16, [hasPTX<78>, hasSM<90>]>;
604:   defm CVT_f32 : CVT_FROM_ALL<"f32", B32>;
605:   defm CVT_f64 : CVT_FROM_ALL<"f64", B64>;
606:   
607:   multiclass CVT_FROM_FLOAT_SATFINITE<string ToName, RegisterClass RC> {
608:     def _f32_sf :
609:       BasicFlagsNVPTXInst<(outs RC:$dst),
610:                 (ins B32:$src), (ins CvtMode:$mode),
611:                 "cvt${mode:base}${mode:relu}.satfinite." # ToName # ".f32">;
612:   }
613:   defm CVT_bf16 : CVT_FROM_FLOAT_SATFINITE<"bf16", B16>;
614:   defm CVT_f16 : CVT_FROM_FLOAT_SATFINITE<"f16", B16>;
615:
616:   // These cvts are different from those above: The source and dest registers
617:   // are of the same type.
618:   def CVT_INREG_s16_s8  : BasicNVPTXInst<(outs B16:$dst), (ins B16:$src), "cvt.s16.s8">;
619:   def CVT_INREG_s32_s8  : BasicNVPTXInst<(outs B32:$dst), (ins B32:$src), "cvt.s32.s8">;
620:   def CVT_INREG_s32_s16 : BasicNVPTXInst<(outs B32:$dst), (ins B32:$src), "cvt.s32.s16">;
621:   def CVT_INREG_s64_s8  : BasicNVPTXInst<(outs B64:$dst), (ins B64:$src), "cvt.s64.s8">;
622:   def CVT_INREG_s64_s16 : BasicNVPTXInst<(outs B64:$dst), (ins B64:$src), "cvt.s64.s16">;
623:   def CVT_INREG_s64_s32 : BasicNVPTXInst<(outs B64:$dst), (ins B64:$src), "cvt.s64.s32">;
624:
625:   multiclass CVT_FROM_FLOAT_V2_SM80<string FromName, RegisterClass RC> {
626:     def _f32 :
627:       BasicFlagsNVPTXInst<(outs RC:$dst),
628:                 (ins B32:$src1, B32:$src2), (ins CvtMode:$mode),
629:                 "cvt${mode:base}${mode:relu}." # FromName # ".f32">,
630:     Requires<[hasPTX<70>, hasSM<80>]>;
631:     
632:     def _f32_sf :
633:       BasicFlagsNVPTXInst<(outs RC:$dst),
634:                 (ins B32:$src1, B32:$src2), (ins CvtMode:$mode),
635:                 "cvt${mode:base}${mode:relu}.satfinite." # FromName # ".f32">;
636:   }
637:
638:   defm CVT_f16x2 : CVT_FROM_FLOAT_V2_SM80<"f16x2", B32>;
639:   defm CVT_bf16x2 : CVT_FROM_FLOAT_V2_SM80<"bf16x2", B32>;
640:   
641:   multiclass CVT_FROM_FLOAT_V2_RS<string FromName, RegisterClass RC> {
642:     def _f32_rs :
643:       BasicFlagsNVPTXInst<(outs RC:$dst),
644:                 (ins B32:$src1, B32:$src2, B32:$src3),
645:                 (ins CvtMode:$mode),
646:                 "cvt${mode:base}${mode:relu}." # FromName # ".f32">;
647:
648:     def _f32_rs_sf :
649:       BasicFlagsNVPTXInst<(outs RC:$dst),
650:                 (ins B32:$src1, B32:$src2, B32:$src3),
651:                 (ins CvtMode:$mode),
652:                 "cvt${mode:base}${mode:relu}.satfinite." # FromName # ".f32">;
653:   }
654:
655:   defm CVT_f16x2 : CVT_FROM_FLOAT_V2_RS<"f16x2", B32>;
656:   defm CVT_bf16x2 : CVT_FROM_FLOAT_V2_RS<"bf16x2", B32>;
657:
658:   // FP8 conversions.
659:   multiclass CVT_TO_F8X2<string F8Name> {
660:     def _f32 :
661:       BasicFlagsNVPTXInst<(outs B16:$dst),
662:                 (ins B32:$src1, B32:$src2), (ins CvtMode:$mode),
663:                 "cvt${mode:base}.satfinite${mode:relu}." # F8Name # "x2.f32">,
664:       Requires<[hasPTX<81>, hasSM<89>]>;
665:     def _f16x2 :
666:       BasicFlagsNVPTXInst<(outs B16:$dst),
667:                 (ins B32:$src), (ins CvtMode:$mode),
668:                 "cvt${mode:base}.satfinite${mode:relu}." # F8Name # "x2.f16x2">,
669:       Requires<[hasPTX<81>, hasSM<89>]>;
670:     def _bf16x2 :
671:       BasicFlagsNVPTXInst<(outs B16:$dst), (ins B32:$src), (ins CvtMode:$mode),
672:                 "cvt${mode:base}.satfinite${mode:relu}." # F8Name # "x2.bf16x2">,
673:       Requires<[callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">]>;
674:   }
675:
676:   defm CVT_e4m3x2 : CVT_TO_F8X2<"e4m3">;
677:   defm CVT_e5m2x2 : CVT_TO_F8X2<"e5m2">;
678:   
679:   multiclass CVT_F8x2_TO_FP16x2<string F8Name> {
680:     def f16x2_ # F8Name # x2 :
681:       BasicFlagsNVPTXInst<(outs B32:$dst),
682:                 (ins B16:$src), (ins CvtMode:$mode),
683:                 "cvt${mode:base}${mode:relu}.f16x2." # F8Name # "x2">,
684:       Requires<[hasPTX<81>, hasSM<89>]>;
685:     def bf16x2_ # F8Name # x2_scale :
686:       BasicFlagsNVPTXInst<(outs B32:$dst),
687:                 (ins B16:$src, B16:$src2), (ins CvtMode:$mode),
688:                 "cvt${mode:base}${mode:relu}${mode:satfinite}.scaled::n2::ue8m0.bf16x2." # F8Name # "x2">,
689:       Requires<[callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">]>;
690:   }
691:
692:   defm CVT_ : CVT_F8x2_TO_FP16x2<"e4m3">;
693:   defm CVT_ : CVT_F8x2_TO_FP16x2<"e5m2">;
694:   
695:   class CVT_TO_FP8X4<string F8Name> :
696:     NVPTXInst<(outs B32:$dst),
697:               (ins B32:$src1, B32:$src2, B32:$src3, B32:$src4, B32:$src5, CvtMode:$mode),
698:               "cvt${mode:base}${mode:relu}.satfinite." # F8Name # 
699:                 "x4.f32 \t$dst, {{$src1, $src2, $src3, $src4}}, $src5;">;
700:   
701:   def CVT_e4m3x4_f32x4_rs_sf : CVT_TO_FP8X4<"e4m3">;
702:   def CVT_e5m2x4_f32x4_rs_sf : CVT_TO_FP8X4<"e5m2">;
703:
704:   // Float to TF32 conversions
705:   multiclass CVT_TO_TF32<string Modifier, list<Predicate> Preds = [hasPTX<78>, hasSM<90>]> {
706:     defvar Intr = !cast<Intrinsic>("int_nvvm_f2tf32_" # !subst(".", "_", Modifier));
707:
708:     def NAME : BasicNVPTXInst<(outs B32:$dst), (ins B32:$src),
709:                "cvt." # Modifier # ".tf32.f32",
710:                [(set i32:$dst, (Intr f32:$src))]>,
711:                Requires<Preds>;
712:   }
713:
714:   defm CVT_to_tf32_rn : CVT_TO_TF32<"rn">;
715:   defm CVT_to_tf32_rz : CVT_TO_TF32<"rz">;
716:   defm CVT_to_tf32_rn_relu  : CVT_TO_TF32<"rn.relu">;
717:   defm CVT_to_tf32_rz_relu  : CVT_TO_TF32<"rz.relu">;
718:   defm CVT_to_tf32_rna      : CVT_TO_TF32<"rna", [hasPTX<70>, hasSM<80>]>;
719:   defm CVT_to_tf32_rna_satf : CVT_TO_TF32<"rna.satfinite", [hasPTX<81>, hasSM<80>]>;
720:
```
- EN: This range uses TableGen DSL to describe records such as CVT_f16, CVT_bf16, CVT_f32, CVT_f64; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 CVT_f16、CVT_bf16、CVT_f32、CVT_f64 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 721-840
```tablegen
721:   defm CVT_to_tf32_rn_satf : CVT_TO_TF32<"rn.satfinite", [hasPTX<86>, hasSM<100>]>;
722:   defm CVT_to_tf32_rz_satf : CVT_TO_TF32<"rz.satfinite", [hasPTX<86>, hasSM<100>]>;
723:   defm CVT_to_tf32_rn_relu_satf  : CVT_TO_TF32<"rn.relu.satfinite", [hasPTX<86>, hasSM<100>]>;
724:   defm CVT_to_tf32_rz_relu_satf  : CVT_TO_TF32<"rz.relu.satfinite", [hasPTX<86>, hasSM<100>]>;
725:   
726: let Predicates = [callSubtarget<"hasS2F6X2ConversionSupport">] in {
727:   def CVT_s2f6x2_f32_sf_scale : BasicFlagsNVPTXInst<(outs B16:$dst),
728:       (ins B32:$src1, B32:$src2, B16:$scale), (ins CvtMode:$mode),
729:       "cvt${mode:base}.satfinite${mode:relu}.scaled::n2::ue8m0.s2f6x2.f32">;
730:   def CVT_s2f6x2_bf16x2_sf_scale : BasicFlagsNVPTXInst<(outs B16:$dst),
731:       (ins B32:$src, B16:$scale), (ins CvtMode:$mode),
732:       "cvt${mode:base}.satfinite${mode:relu}.scaled::n2::ue8m0.s2f6x2.bf16x2">;
733:   def CVT_bf16x2_s2f6x2_scale : BasicFlagsNVPTXInst<(outs B32:$dst),
734:       (ins B16:$src, B16:$scale), (ins CvtMode:$mode),
735:       "cvt${mode:base}${mode:relu}.scaled::n2::ue8m0.bf16x2.s2f6x2">;
736:   def CVT_bf16x2_s2f6x2_sf_scale : BasicFlagsNVPTXInst<(outs B32:$dst),
737:       (ins B16:$src, B16:$scale), (ins CvtMode:$mode),
738:       "cvt${mode:base}.satfinite${mode:relu}.scaled::n2::ue8m0.bf16x2.s2f6x2">;
739: }
740:
741:   // FP6 conversions.
742:   multiclass CVT_F6x2_TO_FP16x2<string F6Name> {
743:     def f16x2_ # F6Name # x2 :
744:       BasicFlagsNVPTXInst<(outs B32:$dst),
745:                 (ins B16:$src), (ins CvtMode:$mode),
746:                 "cvt${mode:base}${mode:relu}.f16x2." # F6Name # "x2">,
747:       Requires<[callSubtarget<"hasNarrowFPConversionSupport">]>;
748:     def bf16x2_ # F6Name # x2_scale :
749:       BasicFlagsNVPTXInst<(outs B32:$dst),
750:                 (ins B16:$src, B16:$src2), (ins CvtMode:$mode),
751:                 "cvt${mode:base}${mode:relu}${mode:satfinite}.scaled::n2::ue8m0.bf16x2." # F6Name # "x2">,
752:       Requires<[callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">]>;
753:   }
754:
755:   defm CVT_ : CVT_F6x2_TO_FP16x2<"e2m3">;
756:   defm CVT_ : CVT_F6x2_TO_FP16x2<"e3m2">;
757:   
758:   class CVT_TO_FP6X4<string F6Name> :
759:     NVPTXInst<(outs B32:$dst),
760:               (ins B32:$src1, B32:$src2, B32:$src3, B32:$src4, B32:$src5, CvtMode:$mode),
761:               "cvt${mode:base}${mode:relu}.satfinite." # F6Name #
762:                 "x4.f32 \t$dst, {{$src1, $src2, $src3, $src4}}, $src5;">;
763:
764:   def CVT_e2m3x4_f32x4_rs_sf : CVT_TO_FP6X4<"e2m3">;
765:   def CVT_e3m2x4_f32x4_rs_sf : CVT_TO_FP6X4<"e3m2">;
766:   
767:   multiclass CVT_TO_FP6X2<string FP6Name> {
768:     def _f32_sf :
769:       BasicFlagsNVPTXInst<(outs B16:$dst),
770:                 (ins B32:$src1, B32:$src2), (ins CvtMode:$mode),
771:                 "cvt${mode:base}.satfinite${mode:relu}." # FP6Name # "x2.f32">;
772:     def _f16x2_sf : 
773:       BasicFlagsNVPTXInst<(outs B16:$dst), (ins B32:$src), (ins CvtMode:$mode),
774:               "cvt${mode:base}.satfinite${mode:relu}." # FP6Name # "x2.f16x2">,
775:       Requires<[callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">]>;
776:     def _bf16x2_sf : 
777:       BasicFlagsNVPTXInst<(outs B16:$dst), (ins B32:$src), (ins CvtMode:$mode),
778:               "cvt${mode:base}.satfinite${mode:relu}." # FP6Name # "x2.bf16x2">,
779:       Requires<[callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">]>;
780:   }
781:
782:   defm CVT_e2m3x2 : CVT_TO_FP6X2<"e2m3">;
783:   defm CVT_e3m2x2 : CVT_TO_FP6X2<"e3m2">;
784:   
785:   // FP4 conversions.
786:   def CVT_e2m1x2_f32_sf : NVPTXInst<(outs B16:$dst),
787:       (ins B32:$src1, B32:$src2, CvtMode:$mode),
788:       !strconcat("{{ \n\t",
789:                  ".reg .b8 \t%e2m1x2_out; \n\t",
790:                  "cvt${mode:base}.satfinite${mode:relu}.e2m1x2.f32 \t%e2m1x2_out, $src1, $src2; \n\t",
791:                  "cvt.u16.u8 \t$dst, %e2m1x2_out; \n\t",
792:                  "}}"), []>;
793:
794:   def CVT_f16x2_e2m1x2 : NVPTXInst<(outs B32:$dst),
795:       (ins B16:$src, CvtMode:$mode),
796:       !strconcat("{{ \n\t",
797:                  ".reg .b8 \t%e2m1x2_in; \n\t",
798:                  "cvt.u8.u16 \t%e2m1x2_in, $src; \n\t",
799:                  "cvt${mode:base}${mode:relu}.f16x2.e2m1x2 \t$dst, %e2m1x2_in; \n\t",
800:                  "}}"), []>;
801:
802:   def CVT_bf16x2_e2m1x2_scale :
803:     NVPTXInst<(outs B32:$dst),
804:               (ins B16:$src1, B16:$src2, CvtMode:$mode),
805:               "{{ \n\t" #
806:               ".reg .b8 \t%e2m1x2_in; \n\t" #
807:               "cvt.u8.u16 \t%e2m1x2_in, $src1; \n\t" #
808:               "cvt${mode:base}${mode:relu}${mode:satfinite}.scaled::n2::ue8m0.bf16x2.e2m1x2 \t$dst, %e2m1x2_in, $src2; \n\t" #
809:               "}}", []>,
810:     Requires<[callSubtarget<"hasNarrowFPToBF16x2ConversionSupport">]>;
811:                  
812:   def CVT_e2m1x4_f32x4_rs_sf :
813:     NVPTXInst<(outs B16:$dst),
814:               (ins B32:$src1, B32:$src2, B32:$src3, B32:$src4, B32:$src5, CvtMode:$mode),
815:               "cvt${mode:base}${mode:relu}.satfinite.e2m1x4.f32 \t" # 
816:                 "$dst, {{$src1, $src2, $src3, $src4}}, $src5;">;
817:
818:   def CVT_e2m1x2_f16x2_sf : NVPTXInst<(outs B16:$dst), 
819:       (ins B32:$src, CvtMode:$mode),
820:       "{{ \n\t" #  
821:       ".reg .b8 \t%e2m1x2_out; \n\t" #
822:       "cvt${mode:base}.satfinite${mode:relu}.e2m1x2.f16x2 \t%e2m1x2_out, $src; \n\t" #
823:       "cvt.u16.u8 \t$dst, %e2m1x2_out; \n\t" #
824:       "}}", []>,
825:       Requires<[callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">]>;
826:
827:   def CVT_e2m1x2_bf16x2_sf : NVPTXInst<(outs B16:$dst), 
828:       (ins B32:$src, CvtMode:$mode),
829:       "{{ \n\t" #  
830:       ".reg .b8 \t%e2m1x2_out; \n\t" #
831:       "cvt${mode:base}.satfinite${mode:relu}.e2m1x2.bf16x2 \t%e2m1x2_out, $src; \n\t" #
832:       "cvt.u16.u8 \t$dst, %e2m1x2_out; \n\t" #
833:       "}}", []>,
834:       Requires<[callSubtarget<"hasFP16X2ToNarrowFPConversionSupport">]>;
835:
836:   // UE8M0x2 conversions.
837:   class CVT_f32_to_ue8m0x2<string sat = ""> :
838:     BasicFlagsNVPTXInst<(outs B16:$dst),
839:               (ins B32:$src1, B32:$src2), (ins CvtMode:$mode),
840:               "cvt${mode:base}" # sat # ".ue8m0x2.f32">;
```
- EN: This range uses TableGen DSL to describe records such as CVT_to_tf32_rn_satf, CVT_to_tf32_rz_satf, CVT_to_tf32_rn_relu_satf, CVT_to_tf32_rz_relu_satf; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 CVT_to_tf32_rn_satf、CVT_to_tf32_rz_satf、CVT_to_tf32_rn_relu_satf、CVT_to_tf32_rz_relu_satf 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 841-960
```tablegen
841:   
842:   class CVT_bf16x2_to_ue8m0x2<string sat = ""> :
843:     BasicFlagsNVPTXInst<(outs B16:$dst),
844:               (ins B32:$src), (ins CvtMode:$mode),
845:               "cvt${mode:base}" # sat # ".ue8m0x2.bf16x2">;
846:               
847:   def CVT_ue8m0x2_f32 : CVT_f32_to_ue8m0x2;
848:   def CVT_ue8m0x2_f32_sf : CVT_f32_to_ue8m0x2<".satfinite">;
849:   def CVT_ue8m0x2_bf16x2 : CVT_bf16x2_to_ue8m0x2;
850:   def CVT_ue8m0x2_bf16x2_sf : CVT_bf16x2_to_ue8m0x2<".satfinite">;
851:
852:   def CVT_bf16x2_ue8m0x2 :
853:     BasicNVPTXInst<(outs B32:$dst),
854:                    (ins B16:$src),
855:                    "cvt.rn.bf16x2.ue8m0x2">;
856:
857: }
858:
859: def fpround_oneuse : OneUse1<fpround>;
860: def : Pat<(v2bf16 (build_vector (bf16 (fpround_oneuse f32:$lo)),
861:                                 (bf16 (fpround_oneuse f32:$hi)))),
862:           (CVT_bf16x2_f32 $hi, $lo, CvtRN)>,
863:       Requires<[hasPTX<70>, hasSM<80>, hasBF16Math]>;
864:
865: def : Pat<(v2f16 (build_vector (f16 (fpround_oneuse f32:$lo)),
866:                                (f16 (fpround_oneuse f32:$hi)))),
867:           (CVT_f16x2_f32 $hi, $lo, CvtRN)>,
868:       Requires<[hasPTX<70>, hasSM<80>, useFP16Math]>;
869:
870: //-----------------------------------
871: // Selection instructions (selp)
872: //-----------------------------------
873:
874: // TODO: Missing slct
875:
876: // selp instructions that don't have any pattern matches; we explicitly use
877: // them within this file.
878: let hasSideEffects = false in {
879:   multiclass SELP_PATTERN<string TypeStr, RegTyInfo t> {
880:     defvar asm_str = "selp." # TypeStr;
881:     def rr :
882:       BasicNVPTXInst<(outs t.RC:$dst),
883:                 (ins t.RC:$a, t.RC:$b, B1:$p),
884:                 asm_str,
885:                 [(set t.Ty:$dst, (select i1:$p, t.Ty:$a, t.Ty:$b))]>;
886:     def ri :
887:       BasicNVPTXInst<(outs t.RC:$dst),
888:                 (ins t.RC:$a, t.Imm:$b, B1:$p),
889:                 asm_str,
890:                 [(set t.Ty:$dst, (select i1:$p, t.Ty:$a, t.ImmNode:$b))]>;
891:     def ir :
892:       BasicNVPTXInst<(outs t.RC:$dst),
893:                 (ins t.Imm:$a, t.RC:$b, B1:$p),
894:                 asm_str,
895:                 [(set t.Ty:$dst, (select i1:$p, t.ImmNode:$a, t.Ty:$b))]>;
896:     def ii :
897:       BasicNVPTXInst<(outs t.RC:$dst),
898:                 (ins t.Imm:$a, t.Imm:$b, B1:$p),
899:                 asm_str,
900:                 [(set t.Ty:$dst, (select i1:$p, t.ImmNode:$a, t.ImmNode:$b))]>;
901:   }
902: }
903:
904: // Don't pattern match on selp.{s,u}{16,32,64} -- selp.b{16,32,64} is just as
905: // good.
906: defm SELP_b16  : SELP_PATTERN<"b16", I16RT>;
907: defm SELP_b32  : SELP_PATTERN<"b32", I32RT>;
908: defm SELP_b64  : SELP_PATTERN<"b64", I64RT>;
909: defm SELP_f16  : SELP_PATTERN<"b16", F16RT>;
910: defm SELP_bf16 : SELP_PATTERN<"b16", BF16RT>;
911: defm SELP_f32  : SELP_PATTERN<"f32", F32RT>;
912: defm SELP_f64  : SELP_PATTERN<"f64", F64RT>;
913:
914: // This does not work as tablegen fails to infer the type of 'imm'.
915: // def v2f16imm : Operand<v2f16>;
916: // defm SELP_f16x2 : SELP_PATTERN<"b32", v2f16, B32, v2f16imm, imm>;
917:
918: foreach vt = [v2f16, v2bf16, v2i16, v4i8] in {
919: def : Pat<(vt (select i1:$p, vt:$a, vt:$b)),
920:           (SELP_b32rr $a, $b, $p)>;
921: }
922:
923: foreach vt = [v2f32, v2i32] in {
924: def : Pat<(vt (select i1:$p, vt:$a, vt:$b)),
925:           (SELP_b64rr $a, $b, $p)>;
926: }
927:
928: //-----------------------------------
929: // Test Instructions
930: //-----------------------------------
931:
932: def fabs_oneuse : OneUse1<fabs>;
933:
934: def TESTINF_f32r : BasicNVPTXInst<(outs B1:$p), (ins B32:$a),
935:                              "testp.infinite.f32",
936:                              [(set i1:$p, (seteq (fabs_oneuse f32:$a), fpimm_pos_inf))]>;
937: def TESTINF_f64r : BasicNVPTXInst<(outs B1:$p), (ins B64:$a),
938:                              "testp.infinite.f64",
939:                              [(set i1:$p, (seteq (fabs_oneuse f64:$a), fpimm_pos_inf))]>;
940:
941: //-----------------------------------
942: // Integer Arithmetic
943: //-----------------------------------
944:
945: // int16, int32, and int64 signed addition.  Since nvptx is 2's complement, we
946: // also use these for unsigned arithmetic.
947: defm ADD : I3<"add.s", add, commutative = true>;
948: defm SUB : I3<"sub.s", sub, commutative = false>;
949:
950: def ADD16x2 : I16x2<"add.s", add>;
951:
952: // int32 and int64 addition and subtraction with carry-out.
953: defm ADDCC : ADD_SUB_INT_CARRY<"add.cc", addc, commutative = true>;
954: defm SUBCC : ADD_SUB_INT_CARRY<"sub.cc", subc, commutative = false>;
955:
956: // int32 and int64 addition and subtraction with carry-in and carry-out.
957: defm ADDCCC : ADD_SUB_INT_CARRY<"addc.cc", adde, commutative = true>;
958: defm SUBCCC : ADD_SUB_INT_CARRY<"subc.cc", sube, commutative = false>;
959:
960: defm MULT : I3<"mul.lo.s", mul, commutative = true>;
```
- EN: This range uses TableGen DSL to describe records such as CVT_bf16x2_to_ue8m0x2, CVT_ue8m0x2_f32, CVT_ue8m0x2_f32_sf, CVT_ue8m0x2_bf16x2; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 CVT_bf16x2_to_ue8m0x2、CVT_ue8m0x2_f32、CVT_ue8m0x2_f32_sf、CVT_ue8m0x2_bf16x2 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 961-1080
```tablegen
 961:
 962: defm MUL_HI_S : I3<"mul.hi.s", mulhs, commutative = true>;
 963: defm MUL_HI_U : I3<"mul.hi.u", mulhu, commutative = true>;
 964:
 965: defm SDIV : I3<"div.s", sdiv, commutative = false>;
 966: defm UDIV : I3<"div.u", udiv, commutative = false>;
 967:
 968: // The ri versions of rem.s and rem.u won't be selected; DAGCombiner::visitSREM
 969: // will lower it.
 970: defm SREM : I3<"rem.s", srem, commutative = false>;
 971: defm UREM : I3<"rem.u", urem, commutative = false>;
 972:
 973: foreach t = [I16RT, I32RT, I64RT] in {
 974:   // PTX abs.s is undefined for INT_MIN, so it matches ABS_MIN_POISON
 975:   // semantics (where INT_MIN input is poison). ISD::ABS requires
 976:   // abs(INT_MIN) == INT_MIN and is expanded separately.
 977:   def ABS_S # t.Size :
 978:     BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a),
 979:                    "abs.s" # t.Size,
 980:                    [(set t.Ty:$dst, (abs_min_poison t.Ty:$a))]>;
 981:
 982:   def NEG_S # t.Size :
 983:     BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$src),
 984:                    "neg.s" # t.Size,
 985:                    [(set t.Ty:$dst, (ineg t.Ty:$src))]>;
 986: }
 987:
 988: // Integer min/max.
 989: defm SMAX : I3<"max.s", smax, commutative = true>;
 990: defm UMAX : I3<"max.u", umax, commutative = true>;
 991: defm SMIN : I3<"min.s", smin, commutative = true>;
 992: defm UMIN : I3<"min.u", umin, commutative = true>;
 993:
 994: def SMAX16x2 : I16x2<"max.s", smax>;
 995: def UMAX16x2 : I16x2<"max.u", umax>;
 996: def SMIN16x2 : I16x2<"min.s", smin>;
 997: def UMIN16x2 : I16x2<"min.u", umin>;
 998:
 999: let Predicates = [hasPTX<80>, hasSM<90>] in {
1000:
1001:   def MIN_RELU_S32 : BasicNVPTXInst<(outs B32:$dst), (ins B32:$a, B32:$b),
1002:                      "min.relu.s32",
1003:                      [(set i32:$dst, (smax (smin i32:$a, i32:$b), 0))]>;
1004:   def MAX_RELU_S32 : BasicNVPTXInst<(outs B32:$dst), (ins B32:$a, B32:$b),
1005:                      "max.relu.s32",
1006:                      [(set i32:$dst, (smax (smax i32:$a, i32:$b), 0))]>;
1007:   def MIN_RELU_S16x2 : BasicNVPTXInst<(outs B32:$dst), (ins B32:$a, B32:$b),
1008:                      "min.relu.s16x2",
1009:                      [(set v2i16:$dst, (smax (smin v2i16:$a, v2i16:$b),
1010:                                              zeroinitializer<v2i16>))]>;
1011:   def MAX_RELU_S16x2 : BasicNVPTXInst<(outs B32:$dst), (ins B32:$a, B32:$b),
1012:                      "max.relu.s16x2",
1013:                      [(set v2i16:$dst, (smax (smax v2i16:$a, v2i16:$b),
1014:                                              zeroinitializer<v2i16>))]>;
1015: }
1016:
1017: //
1018: // Wide multiplication
1019: //
1020:
1021: def SDTMulWide : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisInt<1>, SDTCisSameAs<1, 2>]>;
1022: def smul_wide : SDNode<"NVPTXISD::MUL_WIDE_SIGNED", SDTMulWide, [SDNPCommutative]>;
1023: def umul_wide : SDNode<"NVPTXISD::MUL_WIDE_UNSIGNED", SDTMulWide, [SDNPCommutative]>;
1024:
1025:
1026: multiclass MULWIDEInst<string suffix, SDPatternOperator op, RegTyInfo big_t, RegTyInfo small_t> {
1027:   def suffix # _rr :
1028:     BasicNVPTXInst<(outs big_t.RC:$dst), (ins small_t.RC:$a, small_t.RC:$b), 
1029:                    "mul.wide." # suffix,
1030:                    [(set big_t.Ty:$dst, (op small_t.Ty:$a, small_t.Ty:$b))]>;
1031:   def suffix # _ri :
1032:     BasicNVPTXInst<(outs big_t.RC:$dst), (ins small_t.RC:$a, small_t.Imm:$b), 
1033:                    "mul.wide." # suffix,
1034:                    [(set big_t.Ty:$dst, (op small_t.Ty:$a, imm:$b))]>;
1035: }
1036:
1037: defm MUL_WIDE : MULWIDEInst<"s32", smul_wide, I64RT, I32RT>;
1038: defm MUL_WIDE : MULWIDEInst<"u32", umul_wide, I64RT, I32RT>;
1039: defm MUL_WIDE : MULWIDEInst<"s16", smul_wide, I32RT, I16RT>;
1040: defm MUL_WIDE : MULWIDEInst<"u16", umul_wide, I32RT, I16RT>;
1041:
1042: //
1043: // Integer multiply-add
1044: //
1045: multiclass MADInst<string suffix, SDPatternOperator op, RegTyInfo big_t, RegTyInfo small_t> {
1046:   def rrr:
1047:     BasicNVPTXInst<(outs big_t.RC:$dst),
1048:               (ins small_t.RC:$a, small_t.RC:$b, big_t.RC:$c),
1049:               "mad." # suffix,
1050:               [(set big_t.Ty:$dst, (add (OneUse2<op> small_t.Ty:$a, small_t.Ty:$b), big_t.Ty:$c))]>;
1051:   def rri:
1052:     BasicNVPTXInst<(outs big_t.RC:$dst),
1053:               (ins small_t.RC:$a, small_t.RC:$b, big_t.Imm:$c),
1054:               "mad." # suffix,
1055:               [(set big_t.Ty:$dst, (add (OneUse2<op> small_t.Ty:$a, small_t.Ty:$b), imm:$c))]>;
1056:   def rir:
1057:     BasicNVPTXInst<(outs big_t.RC:$dst),
1058:               (ins small_t.RC:$a, small_t.Imm:$b, big_t.RC:$c),
1059:               "mad." # suffix,
1060:               [(set big_t.Ty:$dst, (add (OneUse2<op> small_t.Ty:$a, imm:$b), big_t.Ty:$c))]>;
1061:   def rii:
1062:     BasicNVPTXInst<(outs big_t.RC:$dst),
1063:               (ins small_t.RC:$a, small_t.Imm:$b, big_t.Imm:$c),
1064:               "mad." # suffix,
1065:               [(set big_t.Ty:$dst, (add (OneUse2<op> small_t.Ty:$a, imm:$b), imm:$c))]>;
1066: }
1067:
1068: let Predicates = [hasOptEnabled] in {
1069:   defm MAD_LO_S16 : MADInst<"lo.s16", mul, I16RT, I16RT>;
1070:   defm MAD_LO_S32 : MADInst<"lo.s32", mul, I32RT, I32RT>;
1071:   defm MAD_LO_S64 : MADInst<"lo.s64", mul, I64RT, I64RT>;
1072:
1073:   // Generating mad.wide causes a regression in some cases: 
1074:   // https://github.com/llvm/llvm-project/pull/150477#issuecomment-3191367837
1075:   // Only do so when the user requests it.
1076:   let Predicates = [doMADWideOpt] in {
1077:     defm MAD_WIDE_U16 : MADInst<"wide.u16", umul_wide, I32RT, I16RT>;
1078:     defm MAD_WIDE_S16 : MADInst<"wide.s16", smul_wide, I32RT, I16RT>;
1079:     defm MAD_WIDE_U32 : MADInst<"wide.u32", umul_wide, I64RT, I32RT>;
1080:     defm MAD_WIDE_S32 : MADInst<"wide.s32", smul_wide, I64RT, I32RT>;
```
- EN: This range uses TableGen DSL to describe records such as MUL_HI_S, MUL_HI_U, SDIV, UDIV; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 MUL_HI_S、MUL_HI_U、SDIV、UDIV 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1081-1200
```tablegen
1081:   }
1082: }
1083:
1084: //-----------------------------------
1085: // Floating Point Arithmetic
1086: //-----------------------------------
1087:
1088: defm FADD : F3_fma_component<"add", fadd>;
1089: defm FSUB : F3_fma_component<"sub", fsub>;
1090: defm FMUL : F3_fma_component<"mul", fmul>;
1091:
1092: def fminnum_or_fminimumnum : PatFrags<(ops node:$a, node:$b),
1093:                                      [(fminnum node:$a, node:$b),
1094:                                       (fminimumnum node:$a, node:$b)]>;
1095: def fmaxnum_or_fmaximumnum : PatFrags<(ops node:$a, node:$b),
1096:                                      [(fmaxnum node:$a, node:$b),
1097:                                       (fmaximumnum node:$a, node:$b)]>;
1098:
1099: defm MIN : FMINIMUMMAXIMUM<"min", /* NaN */ false, fminnum_or_fminimumnum>;
1100: defm MAX : FMINIMUMMAXIMUM<"max", /* NaN */ false, fmaxnum_or_fmaximumnum>;
1101: defm MIN_NAN : FMINIMUMMAXIMUM<"min", /* NaN */ true, fminimum>;
1102: defm MAX_NAN : FMINIMUMMAXIMUM<"max", /* NaN */ true, fmaximum>;
1103:
1104: def nvptx_fminnum3 : SDNode<"NVPTXISD::FMINNUM3", SDTFPTernaryOp,
1105:                             [SDNPCommutative]>;
1106: def nvptx_fmaxnum3 : SDNode<"NVPTXISD::FMAXNUM3", SDTFPTernaryOp,
1107:                              [SDNPCommutative]>;
1108: def nvptx_fminimum3 : SDNode<"NVPTXISD::FMINIMUM3", SDTFPTernaryOp,
1109:                              [SDNPCommutative]>;
1110: def nvptx_fmaximum3 : SDNode<"NVPTXISD::FMAXIMUM3", SDTFPTernaryOp,
1111:                              [SDNPCommutative]>;
1112:
1113: defm FMIN3 : FMINIMUMMAXIMUM3<"min", /* NaN */ false, nvptx_fminnum3>;
1114: defm FMAX3 : FMINIMUMMAXIMUM3<"max", /* NaN */ false, nvptx_fmaxnum3>;
1115: defm FMINNAN3 : FMINIMUMMAXIMUM3<"min", /* NaN */ true, nvptx_fminimum3>;
1116: defm FMAXNAN3 : FMINIMUMMAXIMUM3<"max", /* NaN */ true, nvptx_fmaximum3>;
1117:
1118: defm FABS  : F2<"abs", fabs>;
1119: defm FNEG  : F2<"neg", fneg>;
1120: defm FABS_H: F2_Support_Half<"abs", fabs>;
1121: defm FNEG_H: F2_Support_Half<"neg", fneg>;
1122:
1123: defm FSQRT : F2<"sqrt.rn", fsqrt>;
1124:
1125: //
1126: // F16 NEG
1127: //
1128: class FNEG16<RegTyInfo t> :
1129:       BasicFlagsNVPTXInst<(outs t.RC:$dst), (ins t.RC:$src), (ins FTZFlag:$ftz),
1130:                 "neg$ftz." # t.PtxType,
1131:                 [(set t.Ty:$dst, (fneg t.Ty:$src))]>;
1132:
1133: let Predicates = [useFP16Math, hasPTX<60>, hasSM<53>] in {
1134:   def NEG_F16    : FNEG16<F16RT>;
1135:   def NEG_F16x2  : FNEG16<F16X2RT>;
1136: }
1137: let Predicates = [hasBF16Math, hasPTX<70>, hasSM<80>] in {
1138:   def NEG_BF16   : FNEG16<BF16RT>;
1139:   def NEG_BF16x2 : FNEG16<BF16X2RT>;
1140: }
1141:
1142: //
1143: // EX2
1144: //
1145:
1146: class FEXP2Inst<RegTyInfo t, dag flags, string flag_str> :
1147:       BasicFlagsNVPTXInst<(outs t.RC:$dst), (ins t.RC:$src),
1148:                 flags, "ex2.approx" # flag_str # "." # t.PtxType,
1149:                 [(set t.Ty:$dst, (fexp2 t.Ty:$src))]>;
1150:
1151: def EX2_APPROX_f32 : FEXP2Inst<F32RT, (ins FTZFlag:$ftz), "$ftz">;
1152:
1153: let Predicates = [useFP16Math, hasPTX<70>, hasSM<75>] in {
1154:   def EX2_APPROX_f16 : FEXP2Inst<F16RT, (ins), "">;
1155:   def EX2_APPROX_f16x2 : FEXP2Inst<F16X2RT, (ins), "">;
1156: }
1157: let Predicates = [hasPTX<78>, hasSM<90>] in {
1158:   def EX2_APPROX_bf16 : FEXP2Inst<BF16RT, (ins), ".ftz">;
1159:   def EX2_APPROX_bf16x2 : FEXP2Inst<BF16X2RT, (ins), ".ftz">;
1160: }
1161:
1162: // F64 division
1163: //
1164: def FRCP64r :
1165:   BasicNVPTXInst<(outs B64:$dst),
1166:                  (ins B64:$b),
1167:                  "rcp.rn.f64",
1168:                  [(set f64:$dst, (fdiv fpimm_1, f64:$b))]>;
1169: def FDIV64rr :
1170:   BasicNVPTXInst<(outs B64:$dst),
1171:                  (ins B64:$a, B64:$b),
1172:                  "div.rn.f64",
1173:                  [(set f64:$dst, (fdiv f64:$a, f64:$b))]>;
1174: def FDIV64ri :
1175:   BasicNVPTXInst<(outs B64:$dst),
1176:                  (ins B64:$a, f64imm:$b),
1177:                  "div.rn.f64",
1178:                  [(set f64:$dst, (fdiv f64:$a, fpimm:$b))]>;
1179:
1180: // fdiv will be converted to rcp
1181: // fneg (fdiv 1.0, X) => fneg (rcp.rn X)
1182: def : Pat<(fdiv fpimm_neg_1, f64:$b),
1183:           (FNEGf64 (FRCP64r $b))>;
1184:
1185: //
1186: // F32 Approximate reciprocal
1187: //
1188:
1189: def fdiv_approx : PatFrag<(ops node:$a, node:$b),
1190:                           (fdiv node:$a, node:$b), [{
1191:   return getDivF32Level(N) == NVPTX::DivPrecisionLevel::Approx;
1192: }]>;
1193:
1194:
1195: def RCP_APPROX_F32_r :
1196:   BasicFlagsNVPTXInst<(outs B32:$dst),
1197:                  (ins B32:$b), (ins FTZFlag:$ftz),
1198:                  "rcp.approx$ftz.f32",
1199:                  [(set f32:$dst, (fdiv_approx fpimm_1, f32:$b))]>;
1200:
```
- EN: This range uses TableGen DSL to describe records such as FADD, FSUB, FMUL, fminnum_or_fminimumnum; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 FADD、FSUB、FMUL、fminnum_or_fminimumnum 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1201-1320
```tablegen
1201: //
1202: // F32 Approximate division
1203: //
1204: def DIV_APPROX_F32_rr :
1205:   BasicFlagsNVPTXInst<(outs B32:$dst),
1206:                  (ins B32:$a, B32:$b), (ins FTZFlag:$ftz),
1207:                  "div.approx$ftz.f32",
1208:                  [(set f32:$dst, (fdiv_approx f32:$a, f32:$b))]>;
1209: def DIV_APPROX_F32_ri :
1210:   BasicFlagsNVPTXInst<(outs B32:$dst),
1211:                  (ins B32:$a, f32imm:$b), (ins FTZFlag:$ftz),
1212:                  "div.approx$ftz.f32",
1213:                  [(set f32:$dst, (fdiv_approx f32:$a, fpimm:$b))]>;
1214: //
1215: // F32 Semi-accurate reciprocal
1216: //
1217: // rcp.approx gives the same result as div.full(1.0f, a) and is faster.
1218: //
1219:
1220: def fdiv_full : PatFrag<(ops node:$a, node:$b),
1221:                         (fdiv node:$a, node:$b), [{
1222:   return getDivF32Level(N) == NVPTX::DivPrecisionLevel::Full;
1223: }]>;
1224:
1225:
1226: def : Pat<(fdiv_full fpimm_1, f32:$b),
1227:           (RCP_APPROX_F32_r $b)>;
1228:
1229: //
1230: // F32 Semi-accurate division
1231: //
1232: def FDIV32rr :
1233:   BasicFlagsNVPTXInst<(outs B32:$dst),
1234:                  (ins B32:$a, B32:$b), (ins FTZFlag:$ftz),
1235:                  "div.full$ftz.f32",
1236:                  [(set f32:$dst, (fdiv_full f32:$a, f32:$b))]>;
1237: def FDIV32ri :
1238:   BasicFlagsNVPTXInst<(outs B32:$dst),
1239:                  (ins B32:$a, f32imm:$b), (ins FTZFlag:$ftz),
1240:                  "div.full$ftz.f32",
1241:                  [(set f32:$dst, (fdiv_full f32:$a, fpimm:$b))]>;
1242: //
1243: // F32 Accurate reciprocal
1244: //
1245:
1246: def fdiv_ftz : PatFrag<(ops node:$a, node:$b),
1247:                        (fdiv node:$a, node:$b), [{
1248:   return getDivF32Level(N) == NVPTX::DivPrecisionLevel::IEEE754;
1249: }]>;
1250:
1251: def FRCP32r_prec :
1252:   BasicFlagsNVPTXInst<(outs B32:$dst),
1253:                  (ins B32:$b), (ins FTZFlag:$ftz),
1254:                  "rcp.rn$ftz.f32",
1255:                  [(set f32:$dst, (fdiv_ftz fpimm_1, f32:$b))]>;
1256: //
1257: // F32 Accurate division
1258: //
1259: def FDIV32rr_prec :
1260:   BasicFlagsNVPTXInst<(outs B32:$dst),
1261:                  (ins B32:$a, B32:$b), (ins FTZFlag:$ftz),
1262:                  "div.rn$ftz.f32",
1263:                  [(set f32:$dst, (fdiv_ftz f32:$a, f32:$b))]>;
1264: def FDIV32ri_prec :
1265:   BasicFlagsNVPTXInst<(outs B32:$dst),
1266:                  (ins B32:$a, f32imm:$b), (ins FTZFlag:$ftz),
1267:                  "div.rn$ftz.f32",
1268:                  [(set f32:$dst, (fdiv_ftz f32:$a, fpimm:$b))]>;
1269:
1270: def : Pat<(fdiv fpimm_1, f32:$b), (FRCP32r_prec $b, NoFTZ)>;
1271: def : Pat<(fdiv f32:$a, f32:$b), (FDIV32rr_prec $a, $b, NoFTZ)>;
1272: def : Pat<(fdiv f32:$a, fpimm:$b), (FDIV32ri_prec $a, fpimm:$b, NoFTZ)>;
1273:
1274: //
1275: // FMA
1276: //
1277:
1278: multiclass FMA<RegTyInfo t, bit allow_ftz = true, list<Predicate> preds = []> {
1279:   defvar flag_str = !if(allow_ftz, "$ftz", "");
1280:   defvar flag_ops = !if(allow_ftz, (ins FTZFlag:$ftz), (ins));
1281:   defvar op_str = "fma.rn" # flag_str # "." # t.PtxType;
1282:
1283:   let Predicates = preds in {
1284:     def rrr : BasicFlagsNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, t.RC:$b, t.RC:$c),
1285:                         flag_ops, op_str,
1286:                         [(set t.Ty:$dst, (fma t.Ty:$a, t.Ty:$b, t.Ty:$c))]>;
1287:
1288:     if t.SupportsImm then {
1289:       def rri : BasicFlagsNVPTXInst<(outs t.RC:$dst),
1290:                           (ins t.RC:$a, t.RC:$b, t.Imm:$c),
1291:                           flag_ops, op_str,
1292:                           [(set t.Ty:$dst, (fma t.Ty:$a, t.Ty:$b, fpimm:$c))]>;
1293:       def rir : BasicFlagsNVPTXInst<(outs t.RC:$dst),
1294:                           (ins t.RC:$a, t.Imm:$b, t.RC:$c),
1295:                           flag_ops, op_str,
1296:                           [(set t.Ty:$dst, (fma t.Ty:$a, fpimm:$b, t.Ty:$c))]>;
1297:       def rii : BasicFlagsNVPTXInst<(outs t.RC:$dst),
1298:                           (ins t.RC:$a, t.Imm:$b, t.Imm:$c),
1299:                           flag_ops, op_str,
1300:                           [(set t.Ty:$dst, (fma t.Ty:$a, fpimm:$b, fpimm:$c))]>;
1301:       def iir : BasicFlagsNVPTXInst<(outs t.RC:$dst),
1302:                           (ins t.Imm:$a, t.Imm:$b, t.RC:$c),
1303:                           flag_ops, op_str,
1304:                           [(set t.Ty:$dst, (fma fpimm:$a, fpimm:$b, t.Ty:$c))]>;
1305:     }
1306:   }
1307: }
1308:
1309: defm FMA_F16    : FMA<F16RT,    allow_ftz = true, preds = [useFP16Math]>;
1310: defm FMA_F16x2  : FMA<F16X2RT,  allow_ftz = true, preds = [useFP16Math]>;
1311: defm FMA_BF16   : FMA<BF16RT,   allow_ftz = false, preds = [hasBF16Math]>;
1312: defm FMA_BF16x2 : FMA<BF16X2RT, allow_ftz = false, preds = [hasBF16Math]>;
1313: defm FMA_F32    : FMA<F32RT,    allow_ftz = true>;
1314: defm FMA_F32x2  : FMA<F32X2RT,  allow_ftz = true, preds = [hasF32x2Instructions]>;
1315: defm FMA_F64    : FMA<F64RT,    allow_ftz = false>;
1316:
1317: // sin/cos/tanh
1318:
1319: class UnaryOpAllowsApproxFn<SDPatternOperator operator>
1320:     : PatFrag<(ops node:$A), (operator node:$A), [{
```
- EN: This range uses TableGen DSL to describe records such as DIV_APPROX_F32_rr, DIV_APPROX_F32_ri, fdiv_full, FDIV32rr; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DIV_APPROX_F32_rr、DIV_APPROX_F32_ri、fdiv_full、FDIV32rr 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1321-1440
```tablegen
1321:   return N->getFlags().hasApproximateFuncs();
1322: }]>;
1323:
1324: def SIN_APPROX_f32 :
1325:   BasicFlagsNVPTXInst<(outs B32:$dst), (ins B32:$src), (ins FTZFlag:$ftz),
1326:                       "sin.approx$ftz.f32",
1327:                       [(set f32:$dst, (UnaryOpAllowsApproxFn<fsin> f32:$src))]>;
1328: def COS_APPROX_f32 :
1329:   BasicFlagsNVPTXInst<(outs B32:$dst), (ins B32:$src), (ins FTZFlag:$ftz),
1330:                       "cos.approx$ftz.f32",
1331:                       [(set f32:$dst, (UnaryOpAllowsApproxFn<fcos> f32:$src))]>;
1332: def TANH_APPROX_f32 :
1333:   BasicNVPTXInst<(outs B32:$dst), (ins B32:$src), "tanh.approx.f32",
1334:                  [(set f32:$dst, (UnaryOpAllowsApproxFn<ftanh> f32:$src))]>,
1335:                  Requires<[hasPTX<70>, hasSM<75>]>;
1336:
1337: //-----------------------------------
1338: // Bitwise operations
1339: //-----------------------------------
1340:
1341: // Template for three-arg bitwise operations.  Takes three args, Creates .b16,
1342: // .b32, .b64, and .pred (predicate registers -- i.e., i1) versions of OpcStr.
1343: multiclass BITWISE<string OpcStr, SDNode OpNode> {
1344:   foreach t = [I1RT, I16RT, I32RT, I64RT] in
1345:     defm _ # t.PtxType : I3Inst<OpcStr # "." # t.PtxType, OpNode, t, commutative = true>;
1346: }
1347:
1348: defm OR  : BITWISE<"or", or>;
1349: defm AND : BITWISE<"and", and>;
1350: defm XOR : BITWISE<"xor", xor>;
1351:
1352: // PTX does not support mul on predicates, convert to and instructions
1353: def : Pat<(mul i1:$a, i1:$b), (AND_predrr $a, $b)>;
1354: def : Pat<(mul i1:$a, imm:$b), (AND_predri $a, imm:$b)>;
1355:
1356: foreach op = [add, sub] in {
1357:   def : Pat<(op i1:$a, i1:$b), (XOR_predrr $a, $b)>;
1358:   def : Pat<(op i1:$a, imm:$b), (XOR_predri $a, imm:$b)>;
1359: }
1360:
1361: // These transformations were once reliably performed by instcombine, but thanks
1362: // to poison semantics they are no longer safe for LLVM IR, perform them here
1363: // instead.
1364: def : Pat<(select i1:$a, i1:$b, 0), (AND_predrr $a, $b)>;
1365: def : Pat<(select i1:$a, 1, i1:$b), (OR_predrr $a, $b)>;
1366:
1367: // Lower logical v2i16/v4i8 ops as bitwise ops on b32.
1368: foreach vt = [v2i16, v4i8] in {
1369:   def : Pat<(or vt:$a, vt:$b), (OR_b32rr $a, $b)>;
1370:   def : Pat<(xor vt:$a, vt:$b), (XOR_b32rr $a, $b)>;
1371:   def : Pat<(and vt:$a, vt:$b), (AND_b32rr $a, $b)>;
1372:
1373:   // The constants get legalized into a bitcast from i32, so that's what we need
1374:   // to match here.
1375:   def: Pat<(or vt:$a, (vt (bitconvert (i32 imm:$b)))),
1376:            (OR_b32ri $a, imm:$b)>;
1377:   def: Pat<(xor vt:$a, (vt (bitconvert (i32 imm:$b)))),
1378:            (XOR_b32ri $a, imm:$b)>;
1379:   def: Pat<(and vt:$a, (vt (bitconvert (i32 imm:$b)))),
1380:            (AND_b32ri $a, imm:$b)>;
1381: }
1382:
1383: foreach t = [I1RT, I16RT, I32RT, I64RT] in
1384:   def NOT_ # t.PtxType : BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$src),
1385:                                         "not." # t.PtxType,
1386:                                         [(set t.Ty:$dst, (not t.Ty:$src))]>;
1387:
1388: // Template for left/right shifts.  Takes three operands,
1389: //   [dest (reg), src (reg), shift (reg or imm)].
1390: // dest and src may be int64, int32, or int16, but shift is always int32.
1391: //
1392: // This template also defines a 32-bit shift (imm, imm) instruction.
1393: multiclass SHIFT<string OpcStr, SDNode OpNode> {
1394:   let hasSideEffects = false in {
1395:     foreach t = [I64RT, I32RT, I16RT] in {
1396:       def t.Size # _rr :
1397:         BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, B32:$b),
1398:                   OpcStr # t.Size,
1399:                   [(set t.Ty:$dst, (OpNode t.Ty:$a, i32:$b))]>;
1400:       def t.Size # _ri :
1401:         BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, i32imm:$b),
1402:                   OpcStr # t.Size,
1403:                   [(set t.Ty:$dst, (OpNode t.Ty:$a, (i32 imm:$b)))]>;
1404:       def t.Size # _ii :
1405:         BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, i32imm:$b),
1406:                   OpcStr # t.Size,
1407:                   [(set t.Ty:$dst, (OpNode (t.Ty imm:$a), (i32 imm:$b)))]>;
1408:     }
1409:   }
1410: }
1411:
1412: defm SHL : SHIFT<"shl.b", shl>;
1413: defm SRA : SHIFT<"shr.s", sra>;
1414: defm SRL : SHIFT<"shr.u", srl>;
1415:
1416: // Shift with clamping semantics - these have defined behavior for shift amounts
1417: // >= BitWidth (returning 0 for logical shifts). Used to optimize guarded shift
1418: // patterns like `shift >= 32 ? 0 : x >> shift`.
1419: def shl_clamp : SDNode<"NVPTXISD::SHL_CLAMP", SDTIntShiftOp, []>;
1420: def srl_clamp : SDNode<"NVPTXISD::SRL_CLAMP", SDTIntShiftOp, []>;
1421:
1422: defm SHL_CLAMP : SHIFT<"shl.b", shl_clamp>;
1423: defm SRL_CLAMP : SHIFT<"shr.u", srl_clamp>;
1424:
1425: // Bit-reverse
1426: foreach t = [I64RT, I32RT] in
1427:   def BREV_ # t.PtxType :
1428:     BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a),
1429:                "brev." # t.PtxType,
1430:                [(set t.Ty:$dst, (bitreverse t.Ty:$a))]>;
1431:
1432:
1433: //
1434: // BFE - bit-field extract
1435: //
1436:
1437: // Template for BFE/BFI instructions.
1438: // Args: [dest (reg), src (reg), start (reg or imm), end (reg or imm)].
1439: // Start may be an imm only if end is also an imm.  FIXME: Is this a
1440: // restriction in PTX?
```
- EN: This range uses TableGen DSL to describe records such as SIN_APPROX_f32, COS_APPROX_f32, TANH_APPROX_f32, BITWISE; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SIN_APPROX_f32、COS_APPROX_f32、TANH_APPROX_f32、BITWISE 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1441-1560
```tablegen
1441: //
1442: // dest and src may be int32 or int64, but start and end are always int32.
1443: def SDTBFI :
1444:   SDTypeProfile<1, 4, [SDTCisInt<0>, SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, 
1445:                        SDTCisVT<3, i32>, SDTCisVT<4, i32>]>;
1446: def bfi : SDNode<"NVPTXISD::BFI", SDTBFI>;
1447:
1448: def SDTPRMT :
1449:   SDTypeProfile<1, 4, [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
1450:                        SDTCisVT<2, i32>, SDTCisVT<3, i32>, SDTCisVT<4, i32>]>;
1451: def prmt : SDNode<"NVPTXISD::PRMT", SDTPRMT>;
1452:
1453: multiclass BFE<string Instr, RegisterClass RC> {
1454:   def rrr
1455:     : BasicNVPTXInst<(outs RC:$d), (ins RC:$a, B32:$b, B32:$c), Instr>;
1456:   def rri
1457:     : BasicNVPTXInst<(outs RC:$d), (ins RC:$a, B32:$b, i32imm:$c), Instr>;
1458:   def rii
1459:     : BasicNVPTXInst<(outs RC:$d), (ins RC:$a, i32imm:$b, i32imm:$c), Instr>;
1460: }
1461:
1462: multiclass BFI<string Instr, ValueType T, RegisterClass RC, Operand ImmCls> {
1463:   def rrrr
1464:     : BasicNVPTXInst<(outs RC:$f),
1465:                 (ins RC:$a, RC:$b, B32:$c, B32:$d),
1466:                 Instr,
1467:                 [(set T:$f, (bfi T:$a, T:$b, i32:$c, i32:$d))]>;
1468:   def rrri
1469:     : BasicNVPTXInst<(outs RC:$f),
1470:                 (ins RC:$a, RC:$b, B32:$c, i32imm:$d),
1471:                 Instr,
1472:                 [(set T:$f, (bfi T:$a, T:$b, i32:$c, imm:$d))]>;
1473:   def rrii
1474:     : BasicNVPTXInst<(outs RC:$f),
1475:                 (ins RC:$a, RC:$b, i32imm:$c, i32imm:$d),
1476:                 Instr,
1477:                 [(set T:$f, (bfi T:$a, T:$b, imm:$c, imm:$d))]>;
1478:   def irrr
1479:     : BasicNVPTXInst<(outs RC:$f),
1480:                 (ins ImmCls:$a, RC:$b, B32:$c, B32:$d),
1481:                 Instr,
1482:                 [(set T:$f, (bfi (T imm:$a), T:$b, i32:$c, i32:$d))]>;
1483:   def irri
1484:     : BasicNVPTXInst<(outs RC:$f),
1485:                 (ins ImmCls:$a, RC:$b, B32:$c, i32imm:$d),
1486:                 Instr,
1487:                 [(set T:$f, (bfi (T imm:$a), T:$b, i32:$c, imm:$d))]>;
1488:   def irii
1489:     : BasicNVPTXInst<(outs RC:$f),
1490:                 (ins ImmCls:$a, RC:$b, i32imm:$c, i32imm:$d),
1491:                 Instr,
1492:                 [(set T:$f, (bfi (T imm:$a), T:$b, imm:$c, imm:$d))]>;
1493: }
1494:
1495: def Hexu32imm : Operand<i32> {
1496:   let PrintMethod = "printHexu32imm";
1497: }
1498:
1499: let hasSideEffects = false in {
1500:   // order is somewhat important here. signed/unsigned variants match
1501:   // the same patterns, so the first one wins. Having unsigned byte extraction
1502:   // has the benefit of always having zero in unused bits, which makes some
1503:   // optimizations easier (e.g. no need to mask them).
1504:   defm BFE_U32 : BFE<"bfe.u32", B32>;
1505:   defm BFE_S32 : BFE<"bfe.s32", B32>;
1506:   defm BFE_U64 : BFE<"bfe.u64", B64>;
1507:   defm BFE_S64 : BFE<"bfe.s64", B64>;
1508:
1509:   defm BFI_B32 : BFI<"bfi.b32", i32, B32, i32imm>;
1510:   defm BFI_B64 : BFI<"bfi.b64", i64, B64, i64imm>;
1511:
1512:   def PRMT_B32rrr
1513:     : BasicFlagsNVPTXInst<(outs B32:$d),
1514:                 (ins B32:$a, B32:$b, B32:$c),
1515:                 (ins PrmtMode:$mode),
1516:                 "prmt.b32$mode",
1517:                 [(set i32:$d, (prmt i32:$a, i32:$b, i32:$c, imm:$mode))]>;
1518:   def PRMT_B32rri
1519:     : BasicFlagsNVPTXInst<(outs B32:$d),
1520:                 (ins B32:$a, B32:$b, Hexu32imm:$c),
1521:                 (ins PrmtMode:$mode),
1522:                 "prmt.b32$mode",
1523:                 [(set i32:$d, (prmt i32:$a, i32:$b, imm:$c, imm:$mode))]>;
1524:   def PRMT_B32rir
1525:   : BasicFlagsNVPTXInst<(outs B32:$d),
1526:               (ins B32:$a, i32imm:$b, B32:$c),
1527:               (ins PrmtMode:$mode),
1528:               "prmt.b32$mode",
1529:               [(set i32:$d, (prmt i32:$a, imm:$b, i32:$c, imm:$mode))]>;
1530:   def PRMT_B32rii
1531:     : BasicFlagsNVPTXInst<(outs B32:$d),
1532:                 (ins B32:$a, i32imm:$b, Hexu32imm:$c),
1533:                 (ins PrmtMode:$mode),
1534:                 "prmt.b32$mode",
1535:                 [(set i32:$d, (prmt i32:$a, imm:$b, imm:$c, imm:$mode))]>;
1536:   def PRMT_B32irr
1537:     : BasicFlagsNVPTXInst<(outs B32:$d),
1538:                 (ins i32imm:$a, B32:$b, B32:$c), (ins PrmtMode:$mode),
1539:                 "prmt.b32$mode",
1540:                 [(set i32:$d, (prmt imm:$a, i32:$b, i32:$c, imm:$mode))]>;
1541:   def PRMT_B32iri
1542:     : BasicFlagsNVPTXInst<(outs B32:$d),
1543:                 (ins i32imm:$a, B32:$b, Hexu32imm:$c), (ins PrmtMode:$mode),
1544:                 "prmt.b32$mode",
1545:                 [(set i32:$d, (prmt imm:$a, i32:$b, imm:$c, imm:$mode))]>;
1546:   def PRMT_B32iir
1547:     : BasicFlagsNVPTXInst<(outs B32:$d),
1548:                 (ins i32imm:$a, i32imm:$b, B32:$c), (ins PrmtMode:$mode),
1549:                 "prmt.b32$mode",
1550:                 [(set i32:$d, (prmt imm:$a, imm:$b, i32:$c, imm:$mode))]>;
1551:
1552: }
1553:
1554: // PRMT folding patterns
1555: def : Pat<(fshr i32:$hi, i32:$lo, (shl i32:$amt, (i32 3))),
1556:           (PRMT_B32rrr $lo, $hi, $amt, PrmtF4E)>;
1557:
1558:
1559: def byte_extract_prmt : ImmLeaf<i32, [{
1560:   return (Imm == 0x7770) || (Imm == 0x7771) || (Imm == 0x7772) || (Imm == 0x7773);
```
- EN: This range uses TableGen DSL to describe records such as SDTBFI, bfi, SDTPRMT, prmt; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SDTBFI、bfi、SDTPRMT、prmt 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1561-1680
```tablegen
1561: }]>;
1562:
1563: def to_sign_extend_selector : SDNodeXForm<imm, [{
1564:   const APInt &V = N->getAPIntValue();
1565:   const APInt B = V.trunc(4);
1566:   const APInt BSext = B | 8;
1567:   const APInt R = BSext.concat(BSext).concat(BSext).concat(B).zext(32);
1568:   return CurDAG->getTargetConstant(R, SDLoc(N), MVT::i32);
1569: }]>;
1570:
1571:
1572: // byte extraction + signed/unsigned extension to i32.
1573: def : Pat<(i32 (sext_inreg (prmt i32:$s, 0, byte_extract_prmt:$sel, PrmtNONE), i8)),
1574:           (PRMT_B32rii $s, 0, (to_sign_extend_selector $sel), PrmtNONE)>;
1575:
1576: // byte extraction + signed extension to i16
1577: def : Pat<(i16 (sext_inreg (trunc (prmt i32:$s, 0, byte_extract_prmt:$sel, PrmtNONE)), i8)),
1578:           (CVT_u16_u32 (PRMT_B32rii $s, 0, (to_sign_extend_selector $sel), PrmtNONE), CvtNONE)>;
1579:
1580:
1581: // Byte extraction via shift/trunc/sext
1582: def : Pat<(i16 (sext_inreg (trunc i32:$s), i8)), (CVT_s8_s32 $s, CvtNONE)>;
1583: def : Pat<(i16 (sext_inreg (trunc i64:$s), i8)), (CVT_s8_s64 $s, CvtNONE)>;
1584:
1585: def : Pat<(sext_inreg (srl i32:$s, (i32 imm:$o)), i8), (BFE_S32rii $s, imm:$o, 8)>;
1586: def : Pat<(sext_inreg (srl i64:$s, (i32 imm:$o)), i8), (BFE_S64rii $s, imm:$o, 8)>;
1587:
1588: def : Pat<(i16 (sext_inreg (trunc (srl i32:$s, (i32 imm:$o))), i8)),
1589:           (CVT_s8_s32 (BFE_S32rii $s, imm:$o, 8), CvtNONE)>;
1590: def : Pat<(i16 (sext_inreg (trunc (srl i64:$s, (i32 imm:$o))), i8)),
1591:           (CVT_s8_s64 (BFE_S64rii $s, imm:$o, 8), CvtNONE)>;
1592:
1593: def : Pat<(i16 (sra (i16 (trunc i32:$s)), (i32 8))),
1594:           (CVT_s8_s32 (BFE_S32rii $s, 8, 8), CvtNONE)>;
1595:
1596: //-----------------------------------
1597: // Comparison instructions (setp, set)
1598: //-----------------------------------
1599:
1600: // FIXME: This doesn't cover versions of set and setp that combine with a
1601: // boolean predicate, e.g. setp.eq.and.b16.
1602: def cond2cc : SDNodeXForm<cond, [{
1603:   return getPTXCmpMode(*N);
1604: }]>;
1605:
1606: multiclass FSETP<RegTyInfo t, bit allow_ftz = true> {
1607:   defvar ftz_str = !if(allow_ftz, "$ftz", "");
1608:   defvar op_str = "setp.${cmp:FCmp}" # ftz_str # "." # t.PtxType;
1609:   defvar flags = !con((ins CmpMode:$cmp), !if(allow_ftz, (ins  FTZFlag:$ftz), (ins)));
1610:   let hasSideEffects = false in {
1611:     def rr :
1612:       BasicFlagsNVPTXInst<(outs B1:$dst), (ins t.RC:$a, t.RC:$b),
1613:                           flags, op_str>;
1614:     
1615:     if t.SupportsImm then {
1616:       def ri :
1617:         BasicFlagsNVPTXInst<(outs B1:$dst), (ins t.RC:$a, t.Imm:$b),
1618:                             flags, op_str>;
1619:       def ir :
1620:         BasicFlagsNVPTXInst<(outs B1:$dst), (ins t.Imm:$a, t.RC:$b),
1621:                             flags, op_str>;
1622:     }
1623:   }
1624:   def : Pat<(i1 (setcc t.Ty:$a, t.Ty:$b, cond:$cc)),
1625:             (!cast<NVPTXInst>(NAME # "rr") $a, $b, (cond2cc $cc))>;
1626:   if t.SupportsImm then {
1627:     def : Pat<(i1 (setcc t.Ty:$a, fpimm:$b, cond:$cc)),
1628:               (!cast<NVPTXInst>(NAME # "ri") $a, fpimm:$b, (cond2cc $cc))>;
1629:     def : Pat<(i1 (setcc fpimm:$a, t.Ty:$b, cond:$cc)),
1630:               (!cast<NVPTXInst>(NAME # "ir") fpimm:$a, $b, (cond2cc $cc))>;
1631:   }
1632: }
1633:
1634: multiclass ISETP<RegTyInfo t> {
1635:   defvar op_str = "setp.${cmp:ICmp}.${cmp:IType}" # t.Size;
1636:   let hasSideEffects = false in {
1637:     def rr :
1638:       BasicFlagsNVPTXInst<(outs B1:$dst), (ins t.RC:$a, t.RC:$b),
1639:                           (ins CmpMode:$cmp), op_str>;
1640:     def ri :
1641:       BasicFlagsNVPTXInst<(outs B1:$dst), (ins t.RC:$a, t.Imm:$b),
1642:                           (ins CmpMode:$cmp), op_str>;
1643:     def ir :
1644:       BasicFlagsNVPTXInst<(outs B1:$dst), (ins t.Imm:$a, t.RC:$b),
1645:                           (ins CmpMode:$cmp), op_str>;
1646:   }
1647:   def : Pat<(i1 (setcc t.Ty:$a, t.Ty:$b, cond:$cc)),
1648:             (!cast<NVPTXInst>(NAME # "rr") $a, $b, (cond2cc $cc))>;
1649:   def : Pat<(i1 (setcc t.Ty:$a, imm:$b, cond:$cc)),
1650:             (!cast<NVPTXInst>(NAME # "ri") $a, imm:$b, (cond2cc $cc))>;
1651:   def : Pat<(i1 (setcc imm:$a, t.Ty:$b, cond:$cc)),
1652:             (!cast<NVPTXInst>(NAME # "ir") imm:$a, $b, (cond2cc $cc))>;
1653: }
1654:
1655: defm SETP_i16 : ISETP<I16RT>;
1656: defm SETP_i32 : ISETP<I32RT>;
1657: defm SETP_i64 : ISETP<I64RT>;
1658:
1659: defm SETP_f32 : FSETP<F32RT>;
1660: defm SETP_f64 : FSETP<F64RT, allow_ftz = false>;
1661: let Predicates = [useFP16Math] in
1662:   defm SETP_f16 : FSETP<F16RT>;
1663: let Predicates = [hasBF16Math, hasPTX<78>, hasSM<90>] in
1664:   defm SETP_bf16 : FSETP<BF16RT>;
1665:
1666: def SETP_f16x2rr :
1667:       BasicFlagsNVPTXInst<(outs B1:$p, B1:$q),
1668:                 (ins B32:$a, B32:$b), (ins CmpMode:$cmp, FTZFlag:$ftz),
1669:                 "setp.${cmp:FCmp}$ftz.f16x2">,
1670:                 Requires<[useFP16Math]>;
1671:
1672: def SETP_bf16x2rr :
1673:       BasicFlagsNVPTXInst<(outs B1:$p, B1:$q),
1674:                 (ins B32:$a, B32:$b), (ins CmpMode:$cmp, FTZFlag:$ftz),
1675:                 "setp.${cmp:FCmp}$ftz.bf16x2">,
1676:                 Requires<[hasBF16Math, hasPTX<78>, hasSM<90>]>;
1677:
1678: //-----------------------------------
1679: // Data Movement (Load / Store, Move)
1680: //-----------------------------------
```
- EN: This range uses TableGen DSL to describe records such as to_sign_extend_selector, cond2cc, FSETP, rr; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 to_sign_extend_selector、cond2cc、FSETP、rr 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1681-1800
```tablegen
1681:
1682: def addr : ComplexPattern<pAny, 2, "SelectADDR">;
1683:
1684: def ADDR_base : Operand<pAny>;
1685: def ADDR : Operand<pAny> {
1686:   let PrintMethod = "printMemOperand";
1687:   let MIOperandInfo = (ops ADDR_base, i32imm);
1688: }
1689:
1690: def UsedBytesMask : Operand<i32> {
1691:   let PrintMethod = "printUsedBytesMaskPragma";
1692: }
1693:
1694: def RegOrSink : Operand<Any> {
1695:   let PrintMethod = "printRegisterOrSinkSymbol";
1696: }
1697:
1698: def AtomicCode : Operand<i32> {
1699:   let PrintMethod = "printAtomicCode";
1700: }
1701:
1702: def MmaCode : Operand<i32> {
1703:   let PrintMethod = "printMmaCode";
1704: }
1705:
1706: // Get pointer to local stack.
1707: let hasSideEffects = false in {
1708:   def MOV_DEPOT_ADDR :    NVPTXInst<(outs B32:$d), (ins i32imm:$num),
1709:                                      "mov.b32 \t$d, __local_depot$num;">;
1710:   def MOV_DEPOT_ADDR_64 : NVPTXInst<(outs B64:$d), (ins i32imm:$num),
1711:                                     "mov.b64 \t$d, __local_depot$num;">;
1712: }
1713:
1714: let hasSideEffects = false in {
1715:   let isMoveReg = true, isAsCheapAsAMove = true in
1716:     class MOVr<RegisterClass RC, string OpStr> :
1717:       BasicNVPTXInst<(outs RC:$dst), (ins RC:$src), "mov." # OpStr>;
1718:
1719:   let isMoveImm = true, isAsCheapAsAMove = true in
1720:     class MOVi<RegTyInfo t, string suffix> :
1721:       BasicNVPTXInst<(outs t.RC:$dst), (ins t.Imm:$src),
1722:               "mov." # suffix,
1723:               [(set t.Ty:$dst, t.ImmNode:$src)]>;
1724:
1725:   // We don't want to set isAsCheapAsAMove to true for these instructions as
1726:   // this would prevent CSE and resulted in regressions (see discussion after
1727:   // PR-145581 in llvm-project).
1728:   class MovSymInst<RegTyInfo t> :
1729:     BasicNVPTXInst<(outs t.RC:$dst), (ins Operand<t.Ty>:$src),
1730:                    "mov.b" # t.Size>;
1731: }
1732:
1733: def MOV_B1_r : MOVr<B1, "pred">;
1734: def MOV_B16_r : MOVr<B16, "b16">;
1735: def MOV_B32_r : MOVr<B32, "b32">;
1736: def MOV_B64_r : MOVr<B64, "b64">;
1737: def MOV_B128_r : MOVr<B128, "b128">;
1738:
1739: def MOV_B1_i   : MOVi<I1RT, "pred">;
1740: def MOV_B16_i  : MOVi<I16RT, "b16">;
1741: def MOV_B32_i  : MOVi<I32RT, "b32">;
1742: def MOV_B64_i  : MOVi<I64RT, "b64">;
1743: def MOV_F16_i  : MOVi<F16RT, "b16">;
1744: def MOV_BF16_i : MOVi<BF16RT, "b16">;
1745: def MOV_F32_i  : MOVi<F32RT, "b32">;
1746: def MOV_F64_i  : MOVi<F64RT, "b64">;
1747:
1748: def MOV_B32_sym : MovSymInst<I32RT>;
1749: def MOV_B64_sym : MovSymInst<I64RT>;
1750:
1751:
1752: def to_tglobaladdr : SDNodeXForm<globaladdr, [{
1753:   return CurDAG->getTargetGlobalAddress(N->getGlobal(), SDLoc(N),
1754:                                         N->getValueType(0), N->getOffset(),
1755:                                         N->getTargetFlags());
1756: }]>;
1757:
1758: def to_texternsym : SDNodeXForm<externalsym, [{
1759:   return CurDAG->getTargetExternalSymbol(N->getSymbol(), N->getValueType(0),
1760:                                          N->getTargetFlags());
1761: }]>;
1762:
1763: def to_tframeindex : SDNodeXForm<frameindex, [{
1764:   return CurDAG->getTargetFrameIndex(N->getIndex(), N->getValueType(0));
1765: }]>;
1766:
1767: def : Pat<(i32 globaladdr:$dst), (MOV_B32_sym (to_tglobaladdr $dst))>;
1768: def : Pat<(i64 globaladdr:$dst), (MOV_B64_sym (to_tglobaladdr $dst))>;
1769:
1770: def : Pat<(i32 externalsym:$dst), (MOV_B32_sym (to_texternsym $dst))>;
1771: def : Pat<(i64 externalsym:$dst), (MOV_B64_sym (to_texternsym $dst))>;
1772:
1773: //---- Copy Frame Index ----
1774: def LEA_ADDRi :   NVPTXInst<(outs B32:$dst), (ins ADDR:$addr),
1775:                             "add.u32 \t$dst, ${addr:add};">;
1776: def LEA_ADDRi64 : NVPTXInst<(outs B64:$dst), (ins ADDR:$addr),
1777:                             "add.u64 \t$dst, ${addr:add};">;
1778:
1779: def : Pat<(i32 frameindex:$fi), (LEA_ADDRi (to_tframeindex $fi), 0)>;
1780: def : Pat<(i64 frameindex:$fi), (LEA_ADDRi64 (to_tframeindex $fi), 0)>;
1781:
1782: //-----------------------------------
1783: // Comparison and Selection
1784: //-----------------------------------
1785: // TODO: These patterns seem very specific and brittle. We should try to find
1786: // a more general solution.
1787:
1788: def cond_signed : PatLeaf<(cond), [{
1789:   return isSignedIntSetCC(N->get());
1790: }]>;
1791:
1792: // A 16-bit signed comparison of sign-extended byte extracts can be converted
1793: // to 32-bit comparison if we change the PRMT to sign-extend the extracted
1794: // bytes.
1795: def : Pat<(setcc (i16 (sext_inreg (trunc (prmt i32:$a, 0, byte_extract_prmt:$sel_a, PrmtNONE)), i8)),
1796:                  (i16 (sext_inreg (trunc (prmt i32:$b, 0, byte_extract_prmt:$sel_b, PrmtNONE)), i8)),
1797:                  cond_signed:$cc),
1798:           (SETP_i32rr (PRMT_B32rii i32:$a, 0, (to_sign_extend_selector $sel_a), PrmtNONE),
1799:                       (PRMT_B32rii i32:$b, 0, (to_sign_extend_selector $sel_b), PrmtNONE),
1800:                       (cond2cc $cc))>;
```
- EN: This range uses TableGen DSL to describe records such as addr, ADDR_base, ADDR, UsedBytesMask; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 addr、ADDR_base、ADDR、UsedBytesMask 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1801-1920
```tablegen
1801:
1802: def SDTDeclareArrayParam :
1803:   SDTypeProfile<0, 3, [SDTCisVT<0, i32>, SDTCisVT<1, i32>, SDTCisVT<2, i32>]>;
1804: def SDTDeclareScalarParam :
1805:   SDTypeProfile<0, 2, [SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
1806: def SDTMoveParamProfile : SDTypeProfile<1, 1, [SDTCisInt<0>, SDTCisSameAs<0, 1>]>;
1807:
1808: def SDTProxyReg : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>]>;
1809:
1810: // These nodes represent a parameter declaration. In PTX this will look like:
1811: //   .param .align 16 .b8 param0[1024];
1812: //   .param .b32 retval0;
1813: //
1814: // DeclareArrayParam(Chain, Externalsym, Align, Size, Glue)
1815: // DeclareScalarParam(Chain, Externalsym, Size, Glue)
1816: def declare_array_param :
1817:   SDNode<"NVPTXISD::DeclareArrayParam", SDTDeclareArrayParam,
1818:          [SDNPHasChain, SDNPOutGlue, SDNPInGlue, SDNPSideEffect]>;
1819: def declare_scalar_param :
1820:   SDNode<"NVPTXISD::DeclareScalarParam", SDTDeclareScalarParam,
1821:          [SDNPHasChain, SDNPOutGlue, SDNPInGlue, SDNPSideEffect]>;
1822:
1823: def MoveParam :
1824:   SDNode<"NVPTXISD::MoveParam", SDTMoveParamProfile, []>;
1825: def proxy_reg :
1826:   SDNode<"NVPTXISD::ProxyReg", SDTProxyReg, [SDNPHasChain]>;
1827:
1828:   /// CALL(Chain, IsConvergent, IsIndirectCall/IsUniform, NumReturns,
1829:   ///      NumParams, Callee, Proto)
1830: def SDTCallProfile : SDTypeProfile<0, 6,
1831:                        [SDTCisVT<0, i32>, SDTCisVT<1, i32>, SDTCisVT<2, i32>,
1832:                         SDTCisVT<3, i32>, SDTCisVT<5, i32>]>;
1833: def call : SDNode<"NVPTXISD::CALL", SDTCallProfile, [SDNPHasChain, SDNPSideEffect]>;
1834:
1835: /// CALL(Chain, IsConvergent, IsIndirectCall/IsUniform, NumReturns,
1836: ///      NumParams, Callee, Proto)
1837:
1838: def CallOperand : Operand<i32> { let PrintMethod = "printCallOperand"; }
1839:
1840: foreach is_convergent = [0, 1] in {
1841:   defvar convergent_suffix = !if(is_convergent, "_conv", "");
1842:
1843:   let isCall = 1, isConvergent = is_convergent in {
1844:     def CALL # convergent_suffix :
1845:       NVPTXInst<(outs),
1846:                 (ins ADDR_base:$addr, CallOperand:$rets, CallOperand:$params, 
1847:                      i32imm:$proto),
1848:                 "call${rets:RetList} $addr, (${params:ParamList}), prototype_$proto;">;
1849:
1850:     def CALL_UNI # convergent_suffix :
1851:       NVPTXInst<(outs),
1852:                 (ins ADDR_base:$addr, CallOperand:$rets, CallOperand:$params),
1853:                 "call.uni${rets:RetList} $addr, (${params:ParamList});">;
1854:   }
1855:
1856:   defvar call_inst = !cast<NVPTXInst>("CALL" # convergent_suffix);
1857:   def : Pat<(call is_convergent, 1, imm:$rets, imm:$params, i32:$addr, imm:$proto),
1858:             (call_inst $addr, imm:$rets, imm:$params, imm:$proto)>;
1859:   def : Pat<(call is_convergent, 1, imm:$rets, imm:$params, i64:$addr, imm:$proto),
1860:             (call_inst $addr, imm:$rets, imm:$params, imm:$proto)>;
1861:
1862:   defvar call_uni_inst = !cast<NVPTXInst>("CALL_UNI" # convergent_suffix);
1863:   def : Pat<(call is_convergent, 0, imm:$rets, imm:$params, globaladdr:$addr, 0),
1864:             (call_uni_inst (to_tglobaladdr $addr), imm:$rets, imm:$params)>;
1865: }
1866:
1867: def DECLARE_PARAM_array :
1868:   NVPTXInst<(outs), (ins i32imm:$a, i32imm:$align, i32imm:$size),
1869:             ".param .align $align .b8 \t$a[$size];">;
1870: def DECLARE_PARAM_scalar :
1871:   NVPTXInst<(outs), (ins i32imm:$a, i32imm:$size),
1872:             ".param .b$size \t$a;">;
1873:
1874: def : Pat<(declare_array_param externalsym:$a, imm:$align, imm:$size),
1875:           (DECLARE_PARAM_array (to_texternsym $a), imm:$align, imm:$size)>;
1876: def : Pat<(declare_scalar_param externalsym:$a, imm:$size),
1877:           (DECLARE_PARAM_scalar (to_texternsym $a), imm:$size)>;
1878:
1879: // Call prototype wrapper, this is a dummy instruction that just prints it's
1880: // operand which is string defining the prototype.
1881: def SDTCallPrototype : SDTypeProfile<0, 1, [SDTCisInt<0>]>;
1882: def CallPrototype :
1883:   SDNode<"NVPTXISD::CallPrototype", SDTCallPrototype,
1884:          [SDNPHasChain, SDNPSideEffect]>;
1885: def ProtoIdent : Operand<i32> { let PrintMethod = "printProtoIdent"; }
1886: def CALL_PROTOTYPE :
1887:   NVPTXInst<(outs), (ins ProtoIdent:$ident),
1888:             "$ident", [(CallPrototype (i32 texternalsym:$ident))]>;
1889:
1890:
1891: foreach t = [I32RT, I64RT] in {
1892:   defvar inst_name = "MOV" # t.Size # "_PARAM";
1893:   def inst_name : BasicNVPTXInst<(outs t.RC:$dst), (ins t.RC:$src), "mov.b" # t.Size>;
1894:   def : Pat<(MoveParam (t.Ty externalsym:$src)),
1895:             (!cast<NVPTXInst>(inst_name) (t.Ty (to_texternsym $src)))>;
1896: }
1897:
1898: multiclass ProxyRegInst<string SzStr, NVPTXRegClass rc> {
1899:   def NAME : BasicNVPTXInst<(outs rc:$dst), (ins rc:$src),
1900:                  "mov." # SzStr>;
1901:   foreach vt = rc.RegTypes in
1902:     def : Pat<(vt (proxy_reg vt:$src)), (!cast<NVPTXInst>(NAME) $src)>;
1903: }
1904:
1905: defm ProxyRegB1  : ProxyRegInst<"pred", B1>;
1906: defm ProxyRegB16 : ProxyRegInst<"b16",  B16>;
1907: defm ProxyRegB32 : ProxyRegInst<"b32",  B32>;
1908: defm ProxyRegB64 : ProxyRegInst<"b64",  B64>;
1909:
1910:
1911: // Callseq start and end
1912:
1913: // Note: these nodes are marked as SDNPMayStore and SDNPMayLoad because
1914: // they define the scope in which the declared params may be used. Therefore
1915: // we add these flags to ensure ld.param and st.param are not sunk or hoisted
1916: // out of that scope.
1917:
1918: def callseq_start : SDNode<"ISD::CALLSEQ_START",
1919:                            SDCallSeqStart<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>,
1920:                            [SDNPHasChain, SDNPOutGlue,
```
- EN: This range uses TableGen DSL to describe records such as SDTDeclareArrayParam, SDTDeclareScalarParam, SDTMoveParamProfile, SDTProxyReg; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 SDTDeclareArrayParam、SDTDeclareScalarParam、SDTMoveParamProfile、SDTProxyReg 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 1921-2040
```tablegen
1921:                             SDNPSideEffect, SDNPMayStore, SDNPMayLoad]>;
1922: def callseq_end   : SDNode<"ISD::CALLSEQ_END",
1923:                            SDCallSeqEnd<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>,
1924:                            [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
1925:                             SDNPSideEffect, SDNPMayStore, SDNPMayLoad]>;
1926:
1927: def Callseq_Start :
1928:   NVPTXInst<(outs), (ins i32imm:$amt1, i32imm:$amt2),
1929:             "\\{ // callseq $amt1, $amt2",
1930:             [(callseq_start timm:$amt1, timm:$amt2)]>;
1931: def Callseq_End :
1932:   NVPTXInst<(outs), (ins i32imm:$amt1, i32imm:$amt2),
1933:             "\\} // callseq $amt1",
1934:             [(callseq_end timm:$amt1, timm:$amt2)]>;
1935:
1936: //
1937: // Load / Store Handling
1938: //
1939: class LD<NVPTXRegClass regclass>
1940:   : NVPTXInst<
1941:     (outs regclass:$dst),
1942:     (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp,
1943:          AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
1944:          ADDR:$addr),
1945:     "${usedBytes}"
1946:     "ld${sem:sem}${scope:scope}${addsp:addsp}.${Sign:sign}$fromWidth "
1947:     "\t$dst, [$addr];">;
1948:
1949: let mayLoad=1, hasSideEffects=0 in {
1950:   def LD_i16 : LD<B16>;
1951:   def LD_i32 : LD<B32>;
1952:   def LD_i64 : LD<B64>;
1953: }
1954:
1955: class ST<DAGOperand O>
1956:   : NVPTXInst<
1957:     (outs),
1958:     (ins O:$src,
1959:          AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp, i32imm:$toWidth,
1960:          ADDR:$addr),
1961:     "st${sem:sem}${scope:scope}${addsp:addsp}.b$toWidth"
1962:     " \t[$addr], $src;">;
1963:
1964: let mayStore=1, hasSideEffects=0 in {
1965:   def ST_i16 : ST<RI16>;
1966:   def ST_i32 : ST<RI32>;
1967:   def ST_i64 : ST<RI64>;
1968: }
1969:
1970: // The following is used only in and after vector elementizations.  Vector
1971: // elementization happens at the machine instruction level, so the following
1972: // instructions never appear in the DAG.
1973: multiclass LD_VEC<NVPTXRegClass regclass, bit support_v8 = false> {
1974:   def _v2 : NVPTXInst<
1975:     (outs regclass:$dst1, regclass:$dst2),
1976:     (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp,
1977:          AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
1978:          ADDR:$addr),
1979:     "${usedBytes}"
1980:     "ld${sem:sem}${scope:scope}${addsp:addsp}.v2.${Sign:sign}$fromWidth "
1981:     "\t{{$dst1, $dst2}}, [$addr];">;
1982:   def _v4 : NVPTXInst<
1983:     (outs regclass:$dst1, regclass:$dst2, regclass:$dst3, regclass:$dst4),
1984:     (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp,
1985:          AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
1986:          ADDR:$addr),
1987:     "${usedBytes}"
1988:     "ld${sem:sem}${scope:scope}${addsp:addsp}.v4.${Sign:sign}$fromWidth "
1989:     "\t{{$dst1, $dst2, $dst3, $dst4}}, [$addr];">;
1990:   if support_v8 then
1991:     def _v8 : NVPTXInst<
1992:       (outs regclass:$dst1, regclass:$dst2, regclass:$dst3, regclass:$dst4,
1993:             regclass:$dst5, regclass:$dst6, regclass:$dst7, regclass:$dst8),
1994:       (ins AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp,
1995:            AtomicCode:$Sign, i32imm:$fromWidth, UsedBytesMask:$usedBytes,
1996:            ADDR:$addr),
1997:       "${usedBytes}"
1998:       "ld${sem:sem}${scope:scope}${addsp:addsp}.v8.${Sign:sign}$fromWidth "
1999:       "\t{{$dst1, $dst2, $dst3, $dst4, $dst5, $dst6, $dst7, $dst8}}, "
2000:       "[$addr];">;
2001: }
2002: let mayLoad=1, hasSideEffects=0 in {
2003:   defm LDV_i16 : LD_VEC<B16>;
2004:   defm LDV_i32 : LD_VEC<B32, support_v8 = true>;
2005:   defm LDV_i64 : LD_VEC<B64>;
2006: }
2007:
2008: multiclass ST_VEC<DAGOperand O, bit support_v8 = false> {
2009:   def _v2 : NVPTXInst<
2010:     (outs),
2011:     (ins O:$src1, O:$src2,
2012:          AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp, i32imm:$fromWidth,
2013:          ADDR:$addr),
2014:     "st${sem:sem}${scope:scope}${addsp:addsp}.v2.b$fromWidth "
2015:     "\t[$addr], {{$src1, $src2}};">;
2016:   def _v4 : NVPTXInst<
2017:     (outs),
2018:     (ins RegOrSink:$src1, RegOrSink:$src2, RegOrSink:$src3, RegOrSink:$src4,
2019:          AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp, i32imm:$fromWidth,
2020:          ADDR:$addr),
2021:     "st${sem:sem}${scope:scope}${addsp:addsp}.v4.b$fromWidth "
2022:     "\t[$addr], {{$src1, $src2, $src3, $src4}};">;
2023:   if support_v8 then
2024:     def _v8 : NVPTXInst<
2025:       (outs),
2026:       (ins RegOrSink:$src1, RegOrSink:$src2, RegOrSink:$src3, RegOrSink:$src4,
2027:            RegOrSink:$src5, RegOrSink:$src6, RegOrSink:$src7, RegOrSink:$src8,
2028:            AtomicCode:$sem, AtomicCode:$scope, AtomicCode:$addsp, i32imm:$fromWidth,
2029:            ADDR:$addr),
2030:       "st${sem:sem}${scope:scope}${addsp:addsp}.v8.b$fromWidth "
2031:       "\t[$addr], "
2032:       "{{$src1, $src2, $src3, $src4, $src5, $src6, $src7, $src8}};">;
2033: }
2034:
2035: let mayStore=1, hasSideEffects=0 in {
2036:   defm STV_i16 : ST_VEC<RI16>;
2037:   defm STV_i32 : ST_VEC<RI32, support_v8 = true>;
2038:   defm STV_i64 : ST_VEC<RI64>;
2039: }
2040:
```
- EN: This range uses TableGen DSL to describe records such as callseq_end, Callseq_Start, Callseq_End, LD; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 callseq_end、Callseq_Start、Callseq_End、LD 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2041-2160
```tablegen
2041: //---- Conversion ----
2042:
2043: foreach rc = [B16, B32, B64] in
2044:   foreach ta = rc.RegTypes in
2045:     foreach tb = rc.RegTypes in
2046:       if !ne(ta, tb) then
2047:         def : Pat<(ta (bitconvert tb:$a)),
2048:                   (ta rc:$a)>;
2049:
2050: // NOTE: pred->fp are currently sub-optimal due to an issue in TableGen where
2051: // we cannot specify floating-point literals in isel patterns.  Therefore, we
2052: // use an integer selp to select either 1 (or -1 in case of signed) or 0
2053: // and then cvt to floating-point.
2054:
2055: // sint -> f16
2056: def : Pat<(f16 (sint_to_fp  i1:$a)), (CVT_f16_s32 (SELP_b32ii -1, 0, $a), CvtRN)>;
2057: def : Pat<(f16 (sint_to_fp i16:$a)), (CVT_f16_s16 $a, CvtRN)>;
2058: def : Pat<(f16 (sint_to_fp i32:$a)), (CVT_f16_s32 $a, CvtRN)>;
2059: def : Pat<(f16 (sint_to_fp i64:$a)), (CVT_f16_s64 $a, CvtRN)>;
2060:
2061: // uint -> f16
2062: def : Pat<(f16 (uint_to_fp  i1:$a)), (CVT_f16_u32 (SELP_b32ii 1, 0, $a), CvtRN)>;
2063: def : Pat<(f16 (uint_to_fp i16:$a)), (CVT_f16_u16 $a, CvtRN)>;
2064: def : Pat<(f16 (uint_to_fp i32:$a)), (CVT_f16_u32 $a, CvtRN)>;
2065: def : Pat<(f16 (uint_to_fp i64:$a)), (CVT_f16_u64 $a, CvtRN)>;
2066:
2067: // sint -> bf16
2068: let Predicates = [hasPTX<78>, hasSM<90>] in {
2069:   def : Pat<(bf16 (sint_to_fp i1:$a)), (CVT_bf16_s32 (SELP_b32ii -1, 0, $a), CvtRN)>;
2070:   def : Pat<(bf16 (sint_to_fp i16:$a)), (CVT_bf16_s16 $a, CvtRN)>;
2071:   def : Pat<(bf16 (sint_to_fp i32:$a)), (CVT_bf16_s32 $a, CvtRN)>;
2072:   def : Pat<(bf16 (sint_to_fp i64:$a)), (CVT_bf16_s64 $a, CvtRN)>;
2073: }
2074:
2075: // uint -> bf16
2076: let Predicates = [hasPTX<78>, hasSM<90>] in {
2077:   def : Pat<(bf16 (uint_to_fp i1:$a)), (CVT_bf16_u32 (SELP_b32ii 1, 0, $a), CvtRN)>;
2078:   def : Pat<(bf16 (uint_to_fp i16:$a)), (CVT_bf16_u16 $a, CvtRN)>;
2079:   def : Pat<(bf16 (uint_to_fp i32:$a)), (CVT_bf16_u32 $a, CvtRN)>;
2080:   def : Pat<(bf16 (uint_to_fp i64:$a)), (CVT_bf16_u64 $a, CvtRN)>;
2081: }
2082:
2083: // sint -> f32
2084: def : Pat<(f32 (sint_to_fp  i1:$a)), (CVT_f32_s32 (SELP_b32ii -1, 0, $a), CvtRN)>;
2085: def : Pat<(f32 (sint_to_fp i16:$a)), (CVT_f32_s16 $a, CvtRN)>;
2086: def : Pat<(f32 (sint_to_fp i32:$a)), (CVT_f32_s32 $a, CvtRN)>;
2087: def : Pat<(f32 (sint_to_fp i64:$a)), (CVT_f32_s64 $a, CvtRN)>;
2088:
2089: // uint -> f32
2090: def : Pat<(f32 (uint_to_fp  i1:$a)), (CVT_f32_u32 (SELP_b32ii 1, 0, $a), CvtRN)>;
2091: def : Pat<(f32 (uint_to_fp i16:$a)), (CVT_f32_u16 $a, CvtRN)>;
2092: def : Pat<(f32 (uint_to_fp i32:$a)), (CVT_f32_u32 $a, CvtRN)>;
2093: def : Pat<(f32 (uint_to_fp i64:$a)), (CVT_f32_u64 $a, CvtRN)>;
2094:
2095: // sint -> f64
2096: def : Pat<(f64 (sint_to_fp i1:$a)), (CVT_f64_s32 (SELP_b32ii -1, 0, $a), CvtRN)>;
2097: def : Pat<(f64 (sint_to_fp i16:$a)), (CVT_f64_s16 $a, CvtRN)>;
2098: def : Pat<(f64 (sint_to_fp i32:$a)), (CVT_f64_s32 $a, CvtRN)>;
2099: def : Pat<(f64 (sint_to_fp i64:$a)), (CVT_f64_s64 $a, CvtRN)>;
2100:
2101: // uint -> f64
2102: def : Pat<(f64 (uint_to_fp i1:$a)),  (CVT_f64_u32 (SELP_b32ii 1, 0, $a), CvtRN)>;
2103: def : Pat<(f64 (uint_to_fp i16:$a)), (CVT_f64_u16 $a, CvtRN)>;
2104: def : Pat<(f64 (uint_to_fp i32:$a)), (CVT_f64_u32 $a, CvtRN)>;
2105: def : Pat<(f64 (uint_to_fp i64:$a)), (CVT_f64_u64 $a, CvtRN)>;
2106:
2107:
2108: // f16 -> sint
2109: def : Pat<(i1  (fp_to_sint f16:$a)), (SETP_i16ri $a, 0, CmpEQ)>;
2110: def : Pat<(i16 (fp_to_sint f16:$a)), (CVT_s16_f16 $a, CvtRZI)>;
2111: def : Pat<(i32 (fp_to_sint f16:$a)), (CVT_s32_f16 $a, CvtRZI)>;
2112: def : Pat<(i64 (fp_to_sint f16:$a)), (CVT_s64_f16 $a, CvtRZI)>;
2113:
2114: // f16 -> uint
2115: def : Pat<(i1  (fp_to_uint f16:$a)), (SETP_i16ri $a, 0, CmpEQ)>;
2116: def : Pat<(i16 (fp_to_uint f16:$a)), (CVT_u16_f16 $a, CvtRZI)>;
2117: def : Pat<(i32 (fp_to_uint f16:$a)), (CVT_u32_f16 $a, CvtRZI)>;
2118: def : Pat<(i64 (fp_to_uint f16:$a)), (CVT_u64_f16 $a, CvtRZI)>;
2119:
2120: // bf16 -> sint
2121: def : Pat<(i1  (fp_to_sint bf16:$a)), (SETP_i16ri $a, 0, CmpEQ)>;
2122: def : Pat<(i16 (fp_to_sint bf16:$a)), (CVT_s16_bf16 $a, CvtRZI)>;
2123: def : Pat<(i32 (fp_to_sint bf16:$a)), (CVT_s32_bf16 $a, CvtRZI)>;
2124: def : Pat<(i64 (fp_to_sint bf16:$a)), (CVT_s64_bf16 $a, CvtRZI)>;
2125:
2126: // bf16 -> uint
2127: def : Pat<(i1 (fp_to_uint bf16:$a)),  (SETP_i16ri $a, 0, CmpEQ)>;
2128: def : Pat<(i16 (fp_to_uint bf16:$a)), (CVT_u16_bf16 $a, CvtRZI)>;
2129: def : Pat<(i32 (fp_to_uint bf16:$a)), (CVT_u32_bf16 $a, CvtRZI)>;
2130: def : Pat<(i64 (fp_to_uint bf16:$a)), (CVT_u64_bf16 $a, CvtRZI)>;
2131: // f32 -> sint
2132: let Predicates = [doF32FTZ] in {
2133:   def : Pat<(i16 (fp_to_sint f32:$a)), (CVT_s16_f32 $a, CvtRZI_FTZ)>;
2134:   def : Pat<(i32 (fp_to_sint f32:$a)), (CVT_s32_f32 $a, CvtRZI_FTZ)>;
2135:   def : Pat<(i64 (fp_to_sint f32:$a)), (CVT_s64_f32 $a, CvtRZI_FTZ)>;
2136: }
2137: def : Pat<(i1  (fp_to_sint f32:$a)), (SETP_i32ri $a, 0, CmpEQ)>;
2138: def : Pat<(i16 (fp_to_sint f32:$a)), (CVT_s16_f32 $a, CvtRZI)>;
2139: def : Pat<(i32 (fp_to_sint f32:$a)), (CVT_s32_f32 $a, CvtRZI)>;
2140: def : Pat<(i64 (fp_to_sint f32:$a)), (CVT_s64_f32 $a, CvtRZI)>;
2141:
2142: // f32 -> uint
2143: let Predicates = [doF32FTZ] in {
2144:   def : Pat<(i16 (fp_to_uint f32:$a)), (CVT_u16_f32 $a, CvtRZI_FTZ)>;
2145:   def : Pat<(i32 (fp_to_uint f32:$a)), (CVT_u32_f32 $a, CvtRZI_FTZ)>;
2146:   def : Pat<(i64 (fp_to_uint f32:$a)), (CVT_u64_f32 $a, CvtRZI_FTZ)>;
2147: }
2148: def : Pat<(i1  (fp_to_uint f32:$a)), (SETP_i32ri $a, 0, CmpEQ)>;
2149: def : Pat<(i16 (fp_to_uint f32:$a)), (CVT_u16_f32 $a, CvtRZI)>;
2150: def : Pat<(i32 (fp_to_uint f32:$a)), (CVT_u32_f32 $a, CvtRZI)>;
2151: def : Pat<(i64 (fp_to_uint f32:$a)), (CVT_u64_f32 $a, CvtRZI)>;
2152:
2153: // f64 -> sint
2154: def : Pat<(i1  (fp_to_sint f64:$a)), (SETP_i64ri $a, 0, CmpEQ)>;
2155: def : Pat<(i16 (fp_to_sint f64:$a)), (CVT_s16_f64 $a, CvtRZI)>;
2156: def : Pat<(i32 (fp_to_sint f64:$a)), (CVT_s32_f64 $a, CvtRZI)>;
2157: def : Pat<(i64 (fp_to_sint f64:$a)), (CVT_s64_f64 $a, CvtRZI)>;
2158:
2159: // f64 -> uint
2160: def : Pat<(i1  (fp_to_uint f64:$a)), (SETP_i64ri $a, 0, CmpEQ)>;
```
- EN: This range uses TableGen DSL to describe records such as backend logic; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 后端逻辑 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2161-2280
```tablegen
2161: def : Pat<(i16 (fp_to_uint f64:$a)), (CVT_u16_f64 $a, CvtRZI)>;
2162: def : Pat<(i32 (fp_to_uint f64:$a)), (CVT_u32_f64 $a, CvtRZI)>;
2163: def : Pat<(i64 (fp_to_uint f64:$a)), (CVT_u64_f64 $a, CvtRZI)>;
2164:
2165: // sext i1
2166: def : Pat<(i16 (sext i1:$a)), (SELP_b16ii -1, 0, $a)>;
2167: def : Pat<(i32 (sext i1:$a)), (SELP_b32ii -1, 0, $a)>;
2168: def : Pat<(i64 (sext i1:$a)), (SELP_b64ii -1, 0, $a)>;
2169:
2170: // zext i1
2171: def : Pat<(i16 (zext i1:$a)), (SELP_b16ii 1, 0, $a)>;
2172: def : Pat<(i32 (zext i1:$a)), (SELP_b32ii 1, 0, $a)>;
2173: def : Pat<(i64 (zext i1:$a)), (SELP_b64ii 1, 0, $a)>;
2174:
2175: // anyext i1
2176: def : Pat<(i16 (anyext i1:$a)), (SELP_b16ii -1, 0, $a)>;
2177: def : Pat<(i32 (anyext i1:$a)), (SELP_b32ii -1, 0, $a)>;
2178: def : Pat<(i64 (anyext i1:$a)), (SELP_b64ii -1, 0, $a)>;
2179:
2180: // sext i16
2181: def : Pat<(i32 (sext i16:$a)), (CVT_s32_s16 $a, CvtNONE)>;
2182: def : Pat<(i64 (sext i16:$a)), (CVT_s64_s16 $a, CvtNONE)>;
2183:
2184: // zext i16
2185: def : Pat<(i32 (zext i16:$a)), (CVT_u32_u16 $a, CvtNONE)>;
2186: def : Pat<(i64 (zext i16:$a)), (CVT_u64_u16 $a, CvtNONE)>;
2187:
2188: // anyext i16
2189: def : Pat<(i32 (anyext i16:$a)), (CVT_u32_u16 $a, CvtNONE)>;
2190: def : Pat<(i64 (anyext i16:$a)), (CVT_u64_u16 $a, CvtNONE)>;
2191:
2192: // sext i32
2193: def : Pat<(i64 (sext i32:$a)), (CVT_s64_s32 $a, CvtNONE)>;
2194:
2195: // zext i32
2196: def : Pat<(i64 (zext i32:$a)), (CVT_u64_u32 $a, CvtNONE)>;
2197:
2198: // anyext i32
2199: def : Pat<(i64 (anyext i32:$a)), (CVT_u64_u32 $a, CvtNONE)>;
2200:
2201:
2202: // truncate i64
2203: def : Pat<(i32 (trunc i64:$a)), (CVT_u32_u64 $a, CvtNONE)>;
2204: def : Pat<(i16 (trunc i64:$a)), (CVT_u16_u64 $a, CvtNONE)>;
2205: def : Pat<(i1  (trunc i64:$a)), (SETP_i64ri (AND_b64ri $a, 1), 0, CmpNE)>;
2206:
2207: // truncate i32
2208: def : Pat<(i16 (trunc i32:$a)), (CVT_u16_u32 $a, CvtNONE)>;
2209: def : Pat<(i1  (trunc i32:$a)), (SETP_i32ri (AND_b32ri $a, 1), 0, CmpNE)>;
2210:
2211: // truncate i16
2212: def : Pat<(i1 (trunc i16:$a)), (SETP_i16ri (AND_b16ri $a, 1), 0, CmpNE)>;
2213:
2214: // sext_inreg
2215: def : Pat<(sext_inreg i16:$a, i8), (CVT_INREG_s16_s8 $a)>;
2216: def : Pat<(sext_inreg i32:$a, i8), (CVT_INREG_s32_s8 $a)>;
2217: def : Pat<(sext_inreg i32:$a, i16), (CVT_INREG_s32_s16 $a)>;
2218: def : Pat<(sext_inreg i64:$a, i8), (CVT_INREG_s64_s8 $a)>;
2219: def : Pat<(sext_inreg i64:$a, i16), (CVT_INREG_s64_s16 $a)>;
2220: def : Pat<(sext_inreg i64:$a, i32), (CVT_INREG_s64_s32 $a)>;
2221:
2222: let hasSideEffects = false in {
2223:   // pack a set of smaller int registers to a larger int register
2224:   def V4I16toI64 : NVPTXInst<(outs B64:$d),
2225:                              (ins B16:$s1, B16:$s2,
2226:                                   B16:$s3, B16:$s4),
2227:                              "mov.b64 \t$d, {{$s1, $s2, $s3, $s4}};">;
2228:   def V2I16toI32 : NVPTXInst<(outs B32:$d),
2229:                              (ins B16:$s1, B16:$s2),
2230:                              "mov.b32 \t$d, {{$s1, $s2}};">;
2231:   def V2I32toI64 : NVPTXInst<(outs B64:$d),
2232:                              (ins B32:$s1, B32:$s2),
2233:                              "mov.b64 \t$d, {{$s1, $s2}};">;
2234:   def V2I64toI128 : NVPTXInst<(outs B128:$d),
2235:                               (ins B64:$s1, B64:$s2),
2236:                               "mov.b128 \t$d, {{$s1, $s2}};">;
2237:
2238:   // unpack a larger int register to a set of smaller int registers
2239:   def I64toV4I16 : NVPTXInst<(outs B16:$d1, B16:$d2,
2240:                                    B16:$d3, B16:$d4),
2241:                              (ins B64:$s),
2242:                              "mov.b64 \t{{$d1, $d2, $d3, $d4}}, $s;">;
2243:   def I32toV2I16 : NVPTXInst<(outs B16:$d1, B16:$d2),
2244:                              (ins B32:$s),
2245:                              "mov.b32 \t{{$d1, $d2}}, $s;">;
2246:   def I64toV2I32 : NVPTXInst<(outs B32:$d1, B32:$d2),
2247:                              (ins B64:$s),
2248:                              "mov.b64 \t{{$d1, $d2}}, $s;">;
2249:   def I128toV2I64: NVPTXInst<(outs B64:$d1, B64:$d2),
2250:                               (ins B128:$s),
2251:                               "mov.b128 \t{{$d1, $d2}}, $s;">;
2252:
2253:   def I32toI16H  : NVPTXInst<(outs B16:$high), (ins B32:$s),
2254:                              "{{ .reg .b16 tmp; mov.b32 {tmp, $high}, $s; }}">;
2255:   def I32toI16L  : NVPTXInst<(outs B16:$low), (ins B32:$s),
2256:                              "{{ .reg .b16 tmp; mov.b32 {$low, tmp}, $s; }}">;
2257:   def I64toI32H  : NVPTXInst<(outs B32:$high), (ins B64:$s),
2258:                             "{{ .reg .b32 tmp; mov.b64 {tmp, $high}, $s; }}">;
2259:   def I64toI32L  : NVPTXInst<(outs B32:$low), (ins B64:$s),
2260:                              "{{ .reg .b32 tmp; mov.b64 {$low, tmp}, $s; }}">;
2261:
2262:   // PTX 7.1 lets you avoid a temp register and just use _ as a "sink" for the
2263:   // unused high/low part.
2264:   let Predicates = [hasPTX<71>] in {
2265:     def I32toI16H_Sink  : NVPTXInst<(outs B16:$high), (ins B32:$s),
2266:                               "mov.b32 \t{{_, $high}}, $s;">;
2267:     def I32toI16L_Sink  : NVPTXInst<(outs B16:$low), (ins B32:$s),
2268:                               "mov.b32 \t{{$low, _}}, $s;">;
2269:     def I64toI32H_Sink  : NVPTXInst<(outs B32:$high), (ins B64:$s),
2270:                               "mov.b64 \t{{_, $high}}, $s;">;
2271:     def I64toI32L_Sink  : NVPTXInst<(outs B32:$low), (ins B64:$s),
2272:                               "mov.b64 \t{{$low, _}}, $s;">;
2273:   }
2274: }
2275:
2276: let Predicates = [hasPTX<71>] in {
2277:   def : Pat<(i16 (trunc (srl i32:$s, (i32 16)))), (I32toI16H_Sink i32:$s)>;
2278:   def : Pat<(i16 (trunc (sra i32:$s, (i32 16)))), (I32toI16H_Sink i32:$s)>;
2279:   def : Pat<(i32 (trunc (srl i64:$s, (i32 32)))), (I64toI32H_Sink i64:$s)>;
2280:   def : Pat<(i32 (trunc (sra i64:$s, (i32 32)))), (I64toI32H_Sink i64:$s)>;
```
- EN: This range uses TableGen DSL to describe records such as V4I16toI64, V2I16toI32, V2I32toI64, V2I64toI128; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 V4I16toI64、V2I16toI32、V2I32toI64、V2I64toI128 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2281-2400
```tablegen
2281: }
2282:
2283: // Fall back to the old way if we don't have PTX 7.1.
2284: def : Pat<(i16 (trunc (srl i32:$s, (i32 16)))), (I32toI16H $s)>;
2285: def : Pat<(i16 (trunc (sra i32:$s, (i32 16)))), (I32toI16H $s)>;
2286: def : Pat<(i32 (trunc (srl i64:$s, (i32 32)))), (I64toI32H $s)>;
2287: def : Pat<(i32 (trunc (sra i64:$s, (i32 32)))), (I64toI32H $s)>;
2288:
2289: def: Pat<(i32 (sext (extractelt v2i16:$src, 0))),
2290:          (CVT_INREG_s32_s16 $src)>;
2291:
2292: // Handle extracting one element from the pair (32-bit types)
2293: foreach vt = [v2f16, v2bf16, v2i16] in {
2294:   def : Pat<(extractelt vt:$src, 0), (I32toI16L_Sink $src)>, Requires<[hasPTX<71>]>;
2295:   def : Pat<(extractelt vt:$src, 1), (I32toI16H_Sink $src)>, Requires<[hasPTX<71>]>;
2296:
2297:   def : Pat<(extractelt vt:$src, 0), (I32toI16L $src)>;
2298:   def : Pat<(extractelt vt:$src, 1), (I32toI16H $src)>;
2299:
2300:   def : Pat<(vt (build_vector vt.ElementType:$a, vt.ElementType:$b)), 
2301:             (V2I16toI32 $a, $b)>;
2302: }
2303:
2304: // Handle extracting one element from the pair (64-bit types)
2305: foreach vt = [v2f32, v2i32] in {
2306:   def : Pat<(extractelt vt:$src, 0), (I64toI32L_Sink $src)>, Requires<[hasPTX<71>]>;
2307:   def : Pat<(extractelt vt:$src, 1), (I64toI32H_Sink $src)>, Requires<[hasPTX<71>]>;
2308:
2309:   def : Pat<(extractelt vt:$src, 0), (I64toI32L $src)>;
2310:   def : Pat<(extractelt vt:$src, 1), (I64toI32H $src)>;
2311:
2312:   def : Pat<(vt (build_vector vt.ElementType:$a, vt.ElementType:$b)), 
2313:             (V2I32toI64 $a, $b)>;
2314: }
2315:
2316: def: Pat<(v2i16 (scalar_to_vector i16:$a)),
2317:          (CVT_u32_u16 $a, CvtNONE)>;
2318: def: Pat<(v2i32 (scalar_to_vector i32:$a)),
2319:          (CVT_u64_u32 $a, CvtNONE)>;
2320: def: Pat<(v2f16 (scalar_to_vector f16:$a)),
2321:          (V2I16toI32 $a, 0)>;
2322: def: Pat<(v2f32 (scalar_to_vector f32:$a)),
2323:          (V2I32toI64 $a, (i32 0))>;
2324:
2325: // This node is similar to ISD::BUILD_VECTOR except that the output may be
2326: // implicitly bitcast to a scalar. This allows for the representation of
2327: // packing move instructions for vector types which are not legal i.e. v2i32
2328: def nvptx_build_vector : SDNode<"NVPTXISD::BUILD_VECTOR", SDTypeProfile<1, 2, []>, []>;
2329:
2330: def : Pat<(i64 (nvptx_build_vector i32:$a, i32:$b)),
2331:           (V2I32toI64 $a, $b)>;
2332:
2333: //
2334: // Funnel-Shift
2335: //
2336:
2337: // Create SDNodes so they can be used in the DAG code, e.g.
2338: // NVPTXISelLowering (LowerShiftLeftParts and LowerShiftRightParts)
2339: def fshl_clamp : SDNode<"NVPTXISD::FSHL_CLAMP", SDTIntShiftDOp, []>;
2340: def fshr_clamp : SDNode<"NVPTXISD::FSHR_CLAMP", SDTIntShiftDOp, []>;
2341:
2342: // Funnel shift, requires >= sm_32.  Does not trap if amt is out of range, so
2343: // no side effects.
2344: let hasSideEffects = false in {
2345:   multiclass ShfInst<string mode, SDNode op> {
2346:     def _i
2347:       : BasicNVPTXInst<(outs B32:$dst),
2348:                   (ins  B32:$lo, B32:$hi, i32imm:$amt),
2349:                   "shf." # mode # ".b32",
2350:                   [(set i32:$dst,
2351:                       (op i32:$hi, i32:$lo, (i32 imm:$amt)))]>,
2352:         Requires<[hasHWROT32]>;
2353:
2354:     def _r
2355:       : BasicNVPTXInst<(outs B32:$dst),
2356:                   (ins  B32:$lo, B32:$hi, B32:$amt),
2357:                   "shf." # mode # ".b32",
2358:                   [(set i32:$dst,
2359:                       (op i32:$hi, i32:$lo, i32:$amt))]>,
2360:         Requires<[hasHWROT32]>;
2361:   }
2362:
2363:   defm SHF_L_CLAMP : ShfInst<"l.clamp", fshl_clamp>;
2364:   defm SHF_R_CLAMP : ShfInst<"r.clamp", fshr_clamp>;
2365:   defm SHF_L_WRAP  : ShfInst<"l.wrap", fshl>;
2366:   defm SHF_R_WRAP  : ShfInst<"r.wrap", fshr>;
2367: }
2368:
2369: def : Pat<(i32 (int_nvvm_fshl_clamp i32:$hi, i32:$lo, i32:$amt)),
2370:           (SHF_L_CLAMP_r $lo, $hi, $amt)>;
2371: def : Pat<(i32 (int_nvvm_fshl_clamp i32:$hi, i32:$lo, (i32 imm:$amt))),
2372:           (SHF_L_CLAMP_i $lo, $hi, imm:$amt)>;
2373: def : Pat<(i32 (int_nvvm_fshr_clamp i32:$hi, i32:$lo, i32:$amt)),
2374:           (SHF_R_CLAMP_r $lo, $hi, $amt)>;
2375: def : Pat<(i32 (int_nvvm_fshr_clamp i32:$hi, i32:$lo, (i32 imm:$amt))),
2376:           (SHF_R_CLAMP_i $lo, $hi, imm:$amt)>;
2377:
2378: let hasSideEffects = false in {
2379:   foreach RT = [I32RT, I64RT] in {
2380:     // Count leading zeros
2381:     def CLZr # RT.Size : BasicNVPTXInst<(outs B32:$d), (ins RT.RC:$a),
2382:                                    "clz.b" # RT.Size,
2383:                                    [(set i32:$d, (ctlz RT.Ty:$a))]>;
2384:
2385:     // Population count
2386:     def POPCr # RT.Size : BasicNVPTXInst<(outs B32:$d), (ins RT.RC:$a),
2387:                                     "popc.b" # RT.Size,
2388:                                     [(set i32:$d, (ctpop RT.Ty:$a))]>;
2389:   }
2390: }
2391:
2392: // fpround f32 -> f16
2393: def : Pat<(f16 (fpround f32:$a)), (CVT_f16_f32 $a, CvtRN)>;
2394:
2395: // fpround f32 -> bf16
2396: def : Pat<(bf16 (fpround f32:$a)), (CVT_bf16_f32 $a, CvtRN)>, 
2397:       Requires<[hasPTX<70>, hasSM<80>]>;
2398:
2399: // fpround f64 -> f16
2400: def : Pat<(f16 (fpround f64:$a)), (CVT_f16_f64 $a, CvtRN)>;
```
- EN: This range uses TableGen DSL to describe records such as nvptx_build_vector, fshl_clamp, fshr_clamp, ShfInst; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 nvptx_build_vector、fshl_clamp、fshr_clamp、ShfInst 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2401-2520
```tablegen
2401:
2402: // fpround f64 -> bf16
2403: def : Pat<(bf16 (fpround f64:$a)), (CVT_bf16_f64 $a, CvtRN)>, 
2404:       Requires<[hasPTX<78>, hasSM<90>]>;
2405:
2406: // fpround f64 -> f32
2407: def : Pat<(f32 (fpround f64:$a)), (CVT_f32_f64 $a, CvtRN_FTZ)>, Requires<[doF32FTZ]>;
2408: def : Pat<(f32 (fpround f64:$a)), (CVT_f32_f64 $a, CvtRN)>;
2409:
2410: // fpextend f16 -> f32
2411: def : Pat<(f32 (fpextend f16:$a)), (CVT_f32_f16 $a, CvtNONE_FTZ)>, Requires<[doF32FTZ]>;
2412: def : Pat<(f32 (fpextend f16:$a)), (CVT_f32_f16 $a, CvtNONE)>;
2413: // fpextend bf16 -> f32
2414: def : Pat<(f32 (fpextend bf16:$a)), (CVT_f32_bf16 $a, CvtNONE_FTZ)>, Requires<[doF32FTZ, hasPTX<78>, hasSM<90>]>;
2415: def : Pat<(f32 (fpextend bf16:$a)), (CVT_f32_bf16 $a, CvtNONE)>, Requires<[hasPTX<71>, hasSM<80>]>;
2416:
2417: // fpextend f16 -> f64
2418: def : Pat<(f64 (fpextend f16:$a)), (CVT_f64_f16 $a, CvtNONE)>;
2419:
2420: // fpextend bf16 -> f64
2421: def : Pat<(f64 (fpextend bf16:$a)), (CVT_f64_bf16 $a, CvtNONE)>, Requires<[hasPTX<78>, hasSM<90>]>;
2422:
2423: // fpextend f32 -> f64
2424: def : Pat<(f64 (fpextend f32:$a)), (CVT_f64_f32 $a, CvtNONE_FTZ)>, Requires<[doF32FTZ]>;
2425: def : Pat<(f64 (fpextend f32:$a)), (CVT_f64_f32 $a, CvtNONE)>;
2426:
2427: def retglue : SDNode<"NVPTXISD::RET_GLUE", SDTNone,
2428:                      [SDNPHasChain, SDNPOptInGlue]>;
2429:
2430: // fceil, ffloor, froundeven, ftrunc.
2431:
2432: multiclass CVT_ROUND<SDNode OpNode, PatLeaf Mode, PatLeaf ModeFTZ> {
2433:   def : Pat<(OpNode  f16:$a), (CVT_f16_f16 $a, Mode)>;
2434:   def : Pat<(OpNode bf16:$a), (CVT_bf16_bf16 $a, Mode)>;
2435:   def : Pat<(OpNode  f32:$a), (CVT_f32_f32 $a, ModeFTZ)>, Requires<[doF32FTZ]>;
2436:   def : Pat<(OpNode  f32:$a), (CVT_f32_f32 $a, Mode)>, Requires<[doNoF32FTZ]>;
2437:   def : Pat<(OpNode  f64:$a), (CVT_f64_f64 $a, Mode)>;
2438: }
2439:
2440: defm : CVT_ROUND<fceil, CvtRPI, CvtRPI_FTZ>;
2441: defm : CVT_ROUND<ffloor, CvtRMI, CvtRMI_FTZ>;
2442: defm : CVT_ROUND<froundeven, CvtRNI, CvtRNI_FTZ>;
2443: defm : CVT_ROUND<ftrunc, CvtRZI, CvtRZI_FTZ>;
2444:
2445: // nearbyint and rint are implemented as rounding to nearest even.  This isn't
2446: // strictly correct, because it causes us to ignore the rounding mode.  But it
2447: // matches what CUDA's "libm" does.
2448:
2449: defm : CVT_ROUND<fnearbyint, CvtRNI, CvtRNI_FTZ>;
2450: defm : CVT_ROUND<frint, CvtRNI, CvtRNI_FTZ>;
2451:
2452: //-----------------------------------
2453: // Control-flow
2454: //-----------------------------------
2455:
2456: let isTerminator=1 in {
2457:   let isReturn=1, isBarrier=1 in
2458:       def Return : BasicNVPTXInst<(outs), (ins), "ret", [(retglue)]>;
2459:
2460:   let isBranch=1 in {
2461:     def CBranch : NVPTXInst<(outs),
2462:                               (ins B1:$a, brtarget:$target, BranchFlag:$not),
2463:                               "@${not}$a bra \t$target;",
2464:                               [(brcond i1:$a, bb:$target)]>;
2465:
2466:     let isBarrier=1 in
2467:       def GOTO : BasicNVPTXInst<(outs), (ins brtarget:$target),
2468:                             "bra.uni", [(br bb:$target)]>;
2469:   }
2470: }
2471:
2472:
2473: // trap instruction
2474: def trapinst : BasicNVPTXInst<(outs), (ins), "trap", [(trap)]>, Requires<[noPTXASUnreachableBug]>;
2475: // Emit an `exit` as well to convey to ptxas that `trap` exits the CFG.
2476: // This won't be necessary in a future version of ptxas.
2477: def trapexitinst : NVPTXInst<(outs), (ins), "trap; exit;", [(trap)]>, Requires<[hasPTXASUnreachableBug]>;
2478: // brkpt instruction
2479: def debugtrapinst : BasicNVPTXInst<(outs), (ins), "brkpt", [(debugtrap)]>;
2480:
2481: def SDTDynAllocaOp :
2482:   SDTypeProfile<1, 2, [SDTCisSameAs<0, 1>, SDTCisInt<1>, SDTCisVT<2, i32>]>;
2483:
2484: def dyn_alloca :
2485:   SDNode<"NVPTXISD::DYNAMIC_STACKALLOC", SDTDynAllocaOp,
2486:          [SDNPHasChain, SDNPSideEffect]>;
2487:
2488: foreach t = [I32RT, I64RT] in {
2489:   def DYNAMIC_STACKALLOC # t.Size :
2490:     BasicNVPTXInst<(outs t.RC:$ptr),
2491:               (ins t.RC:$size, i32imm:$align),
2492:               "alloca.u" # t.Size,
2493:               [(set t.Ty:$ptr, (dyn_alloca t.Ty:$size, timm:$align))]>,
2494:               Requires<[hasPTX<73>, hasSM<52>]>;
2495: }
2496:
2497: //
2498: // BRX
2499: //
2500:
2501: let isTerminator = 1, isBranch = 1, isIndirectBranch = 1, isNotDuplicable = 1 in {
2502:
2503:   def BRX_START :
2504:     NVPTXInst<(outs), (ins i32imm:$id), "$$L_brx_$id: .branchtargets">;
2505:
2506:   def BRX_ITEM :
2507:     NVPTXInst<(outs), (ins brtarget:$target), "\t$target,">;
2508:
2509:   def BRX_END :
2510:     NVPTXInst<(outs), (ins brtarget:$target, B32:$val, i32imm:$id),
2511:               "\t$target;\n\tbrx.idx \t$val, $$L_brx_$id;"> {
2512:       let isBarrier = 1;
2513:     }
2514: }
2515:
2516:
2517: foreach a_type = ["s", "u"] in {
2518:   foreach b_type = ["s", "u"] in {
2519:
2520:     def DOT4_ # a_type # b_type :
```
- EN: This range uses TableGen DSL to describe records such as retglue, CVT_ROUND, Return, CBranch; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 retglue、CVT_ROUND、Return、CBranch 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

### Lines 2521-2616
```tablegen
2521:       BasicNVPTXInst<(outs B32:$dst),
2522:                 (ins B32:$a, B32:$b, B32:$c),
2523:                 "dp4a." # a_type # "32." # b_type # "32",
2524:                 [(set i32:$dst,
2525:                     (!cast<Intrinsic>("int_nvvm_idp4a_" # a_type # "_" # b_type)
2526:                      i32:$a, i32:$b, i32:$c))]>,
2527:                 Requires<[hasDotInstructions]>;
2528:
2529:     foreach is_hi = [0, -1] in {
2530:       defvar lohi_suffix = !if(is_hi, "hi", "lo");
2531:
2532:       def DOT2_ # lohi_suffix # _ # a_type # b_type :
2533:         BasicNVPTXInst<(outs B32:$dst),
2534:                   (ins B32:$a, B32:$b, B32:$c),
2535:                   "dp2a." # lohi_suffix # "." # a_type # "32." # b_type # "32",
2536:                   [(set i32:$dst,
2537:                       (!cast<Intrinsic>("int_nvvm_idp2a_" # a_type # "_" # b_type)
2538:                        i32:$a, i32:$b, is_hi, i32:$c))]>,
2539:                   Requires<[hasDotInstructions]>;
2540:     }
2541:   }
2542: }
2543:
2544: //
2545: // Stack Manipulation
2546: //
2547:
2548: def SDTStackRestore : SDTypeProfile<0, 1, [SDTCisInt<0>]>;
2549:
2550: def stackrestore :
2551:   SDNode<"NVPTXISD::STACKRESTORE", SDTStackRestore,
2552:          [SDNPHasChain, SDNPSideEffect]>;
2553:
2554: def stacksave :
2555:   SDNode<"NVPTXISD::STACKSAVE", SDTIntLeaf,
2556:          [SDNPHasChain, SDNPSideEffect]>;
2557:
2558: let Predicates = [hasPTX<73>, hasSM<52>] in {
2559:   foreach t = [I32RT, I64RT] in {
2560:     def STACKRESTORE_ # t.Size :
2561:       BasicNVPTXInst<(outs), (ins t.RC:$ptr),
2562:                 "stackrestore.u" # t.Size,
2563:               [(stackrestore t.Ty:$ptr)]>;
2564:
2565:     def STACKSAVE_ # t.Size :
2566:       BasicNVPTXInst<(outs t.RC:$dst), (ins),
2567:                 "stacksave.u" # t.Size,
2568:               [(set t.Ty:$dst, (t.Ty stacksave))]>;
2569:   }
2570: }
2571:
2572: include "NVPTXIntrinsics.td"
2573:
2574: ////////////////////////////////////////////////////////////////////////////////
2575: // PTX Fence instructions
2576: ////////////////////////////////////////////////////////////////////////////////
2577:
2578: class NVPTXFenceInst<string scope, string sem, Predicate ptx>:
2579:     BasicNVPTXInst<(outs), (ins), "fence."#sem#"."#scope>,
2580:     Requires<[ptx, hasSM<70>]>;
2581:
2582: foreach scope = ["sys", "gpu", "cluster", "cta"] in {
2583:   def atomic_thread_fence_seq_cst_#scope: NVPTXFenceInst<scope, "sc", hasPTX<60>>;
2584:   def atomic_thread_fence_acq_rel_#scope: NVPTXFenceInst<scope, "acq_rel", hasPTX<60>>;
2585:   def atomic_thread_fence_acquire_#scope: NVPTXFenceInst<scope, "acquire", hasPTX<87>>;
2586:   def atomic_thread_fence_release_#scope: NVPTXFenceInst<scope, "release", hasPTX<87>>;
2587: }
2588:
2589: // Perform substitution if fma only has one use, and also if instruction has
2590: // nnan instruction flag
2591: def NVPTX_fma_oneuse_and_nnan : PatFrag<(ops node:$a, node:$b, node:$c),
2592:                                   (fma node:$a, node:$b, node:$c), [{
2593:   return N->hasOneUse() && N->getFlags().hasNoNaNs();
2594: }]>;
2595: // fmaxnum/fmaximumnum differentiate between signed and unsigned zeros, so this
2596: // PatFrag is for a fmaxnum/fmaximumnum node with nsz
2597: def NVPTX_fmaxnum_or_fmaximumnum_nsz : PatFrag<(ops node:$a, node:$b),
2598:                                   (fmaxnum_or_fmaximumnum node:$a, node:$b), [{
2599:   return N->getFlags().hasNoSignedZeros();
2600: }]>;
2601:
2602: class FMARELUInst<RegTyInfo t, bit allow_ftz, PatFrag zero_pat>
2603:   : BasicFlagsNVPTXInst<(outs t.RC:$dst), (ins t.RC:$a, t.RC:$b, t.RC:$c),
2604:                    !if(allow_ftz, (ins FTZFlag:$ftz), (ins)),
2605:                    "fma.rn" # !if(allow_ftz, "$ftz", "") # ".relu." # t.PtxType,
2606:                    [(set t.Ty:$dst, (NVPTX_fmaxnum_or_fmaximumnum_nsz (NVPTX_fma_oneuse_and_nnan t.Ty:$a, t.Ty:$b, t.Ty:$c), zero_pat))]>;
2607:
2608: let Predicates = [useFP16Math, hasPTX<70>, hasSM<80>] in {
2609:   def FMARELU_F16 : FMARELUInst<F16RT, true, fpimm_0>;
2610:   def FMARELU_F16X2 : FMARELUInst<F16X2RT, true, zeroinitializer<v2f16>>;
2611: }
2612:
2613: let Predicates = [hasBF16Math, hasPTX<70>, hasSM<80>] in {
2614:   def FMARELU_BF16 : FMARELUInst<BF16RT, false, fpimm_0>;
2615:   def FMARELU_BF16X2 : FMARELUInst<BF16X2RT, false, zeroinitializer<v2bf16>>;
2616: }
```
- EN: This range uses TableGen DSL to describe records such as DOT2_, SDTStackRestore, stackrestore, stacksave; LLVM later expands these declarations into generated backend tables and helper code.
- CN: 这一段使用 TableGen DSL 描述 DOT2_、SDTStackRestore、stackrestore、stacksave 等记录；LLVM 后续会把这些声明展开为生成的后端表和辅助代码。

## Key Concepts / 关键概念

- EN: Instruction info files centralize opcode semantics, scheduling hooks, and target-specific machine properties.
  - CN: 指令信息文件集中描述操作码语义、调度钩子以及目标相关机器属性。
- EN: TableGen files use declarative records that LLVM expands into generated C++ tables and helper code.
  - CN: TableGen 文件使用声明式记录，LLVM 会将其展开为生成的 C++ 表和辅助代码。
- EN: NVPTX backend code reflects GPU execution concerns such as address spaces, intrinsics, and PTX-specific lowering.
  - CN: NVPTX 后端代码体现了 GPU 执行特性，例如地址空间、内建函数以及 PTX 特定降级。
- EN: Key symbols in this file include f16imm, bf16imm, brtarget, CvtNONE, CvtRNI, which anchor the file's main abstractions.
  - CN: 该文件中的关键符号包括 f16imm, bf16imm, brtarget, CvtNONE, CvtRNI，它们构成了本文件的核心抽象。

## Dependencies / 依赖关系

- Local backend headers / 本地后端头文件:
  - `NVPTXInstrFormats.td`
  - `NVPTXIntrinsics.td`
- TableGen pipeline / TableGen 流水线: records in this file are consumed by LLVM TableGen emitters to produce generated lookup tables and helper code.
- TableGen 流水线说明：该文件中的记录会被 LLVM 的 TableGen 生成器消费，进而产出查找表和辅助代码。
