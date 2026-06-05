# SystemZInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-55 / 第 1-55 行
```tablegen
   1: //==- SystemZInstrFormats.td - SystemZ Instruction Formats --*- tablegen -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Basic SystemZ instruction definition
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class InstSystemZ<int size, dag outs, dag ins, string asmstr,
  14:                   list<dag> pattern> : Instruction {
  15:   let Namespace = "SystemZ";
  16: 
  17:   dag OutOperandList = outs;
  18:   dag InOperandList = ins;
  19:   let Size = size;
  20:   let Pattern = pattern;
  21:   // Convert tabs to spaces, and remove space after comma for HLASM syntax
  22:   let AsmString = !subst("\t", "{\t| }", !subst(", ", "{, |,}", asmstr));
  23: 
  24:   let hasSideEffects = 0;
  25:   let mayLoad = 0;
  26:   let mayStore = 0;
  27: 
  28:   // Some instructions come in pairs, one having a 12-bit displacement
  29:   // and the other having a 20-bit displacement.  Both instructions in
  30:   // the pair have the same DispKey and their DispSizes are "12" and "20"
  31:   // respectively.
  32:   string DispKey = "";
  33:   string DispSize = "none";
  34: 
  35:   // Many register-based <INSN>R instructions have a memory-based <INSN>
  36:   // counterpart.  OpKey uniquely identifies <INSN>R, while OpType is
  37:   // "reg" for <INSN>R and "mem" for <INSN>.
  38:   string OpKey = "";
  39:   string OpType = "none";
  40: 
  41:   // MemKey identifies a targe reg-mem opcode, while MemType can be either
  42:   // "pseudo" or "target". This is used to map a pseduo memory instruction to
  43:   // its corresponding target opcode. See comment at MemFoldPseudo.
  44:   string MemKey = "";
  45:   string MemType = "none";
  46: 
  47:   // Many distinct-operands instructions have older 2-operand equivalents.
  48:   // NumOpsKey uniquely identifies one of these 2-operand and 3-operand pairs,
  49:   // with NumOpsValue being "2" or "3" as appropriate.
  50:   string NumOpsKey = "";
  51:   string NumOpsValue = "none";
  52: 
  53:   // True if this instruction is a simple D(X,B) load of a register
  54:   // (with no sign or zero extension).
  55:   bit SimpleBDXLoad = 0;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `InstSystemZ`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `InstSystemZ` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 56-110 / 第 56-110 行
```tablegen
  56: 
  57:   // True if this instruction is a simple D(X,B) store of a register
  58:   // (with no truncation).
  59:   bit SimpleBDXStore = 0;
  60: 
  61:   // True if this instruction has a 20-bit displacement field.
  62:   bit Has20BitOffset = 0;
  63: 
  64:   // True if addresses in this instruction have an index register.
  65:   bit HasIndex = 0;
  66: 
  67:   // True if this is a 128-bit pseudo instruction that combines two 64-bit
  68:   // operations.
  69:   bit Is128Bit = 0;
  70: 
  71:   // The access size of all memory operands in bytes, or 0 if not known.
  72:   bits<5> AccessBytes = 0;
  73: 
  74:   // If the instruction sets CC to a useful value, this gives the mask
  75:   // of all possible CC results.  The mask has the same form as
  76:   // SystemZ::CCMASK_*.
  77:   bits<4> CCValues = 0;
  78: 
  79:   // The subset of CCValues that have the same meaning as they would after a
  80:   // comparison of the first operand against zero. "Logical" instructions
  81:   // leave this blank as they set CC in a different way.
  82:   bits<4> CompareZeroCCMask = 0;
  83: 
  84:   // True if the instruction is conditional and if the CC mask operand
  85:   // comes first (as for BRC, etc.).
  86:   bit CCMaskFirst = 0;
  87: 
  88:   // Similar, but true if the CC mask operand comes last (as for LOC, etc.).
  89:   bit CCMaskLast = 0;
  90: 
  91:   // True if the instruction is the "logical" rather than "arithmetic" form,
  92:   // in cases where a distinction exists. Except for logical compares, if the
  93:   // instruction sets this flag along with a non-zero CCValues field, it is
  94:   // assumed to set CC to either CCMASK_LOGICAL_ZERO or
  95:   // CCMASK_LOGICAL_NONZERO.
  96:   bit IsLogical = 0;
  97: 
  98:   // True if the (add or sub) instruction sets CC like a compare of the
  99:   // result against zero, but only if the 'nsw' flag is set.
 100:   bit CCIfNoSignedWrap = 0;
 101: 
 102:   let TSFlags{0}     = SimpleBDXLoad;
 103:   let TSFlags{1}     = SimpleBDXStore;
 104:   let TSFlags{2}     = Has20BitOffset;
 105:   let TSFlags{3}     = HasIndex;
 106:   let TSFlags{4}     = Is128Bit;
 107:   let TSFlags{9-5}   = AccessBytes;
 108:   let TSFlags{13-10} = CCValues;
 109:   let TSFlags{17-14} = CompareZeroCCMask;
 110:   let TSFlags{18}    = CCMaskFirst;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 111-165 / 第 111-165 行
```tablegen
 111:   let TSFlags{19}    = CCMaskLast;
 112:   let TSFlags{20}    = IsLogical;
 113:   let TSFlags{21}    = CCIfNoSignedWrap;
 114: }
 115: 
 116: //===----------------------------------------------------------------------===//
 117: // Mappings between instructions
 118: //===----------------------------------------------------------------------===//
 119: 
 120: // Return the version of an instruction that has an unsigned 12-bit
 121: // displacement.
 122: def getDisp12Opcode : InstrMapping {
 123:   let FilterClass = "InstSystemZ";
 124:   let RowFields = ["DispKey"];
 125:   let ColFields = ["DispSize"];
 126:   let KeyCol = ["20"];
 127:   let ValueCols = [["12"]];
 128: }
 129: 
 130: // Return the version of an instruction that has a signed 20-bit displacement.
 131: def getDisp20Opcode : InstrMapping {
 132:   let FilterClass = "InstSystemZ";
 133:   let RowFields = ["DispKey"];
 134:   let ColFields = ["DispSize"];
 135:   let KeyCol = ["12"];
 136:   let ValueCols = [["20"]];
 137: }
 138: 
 139: // Return the memory form of a register instruction. Note that this may
 140: // return a MemFoldPseudo instruction (see below).
 141: def getMemOpcode : InstrMapping {
 142:   let FilterClass = "InstSystemZ";
 143:   let RowFields = ["OpKey"];
 144:   let ColFields = ["OpType"];
 145:   let KeyCol = ["reg"];
 146:   let ValueCols = [["mem"]];
 147: }
 148: 
 149: // Return the target memory instruction for a MemFoldPseudo.
 150: def getTargetMemOpcode : InstrMapping {
 151:   let FilterClass = "InstSystemZ";
 152:   let RowFields = ["MemKey"];
 153:   let ColFields = ["MemType"];
 154:   let KeyCol = ["pseudo"];
 155:   let ValueCols = [["target"]];
 156: }
 157: 
 158: // Return the 2-operand form of a 3-operand instruction.
 159: def getTwoOperandOpcode : InstrMapping {
 160:   let FilterClass = "InstSystemZ";
 161:   let RowFields = ["NumOpsKey"];
 162:   let ColFields = ["NumOpsValue"];
 163:   let KeyCol = ["3"];
 164:   let ValueCols = [["2"]];
 165: }
```
- **EN**: This block declares or refines TableGen records such as `getDisp12Opcode`, `getDisp20Opcode`, `getMemOpcode`, `getTargetMemOpcode`, `getTwoOperandOpcode`.
- **CN**: 该代码块声明或细化了 `getDisp12Opcode`, `getDisp20Opcode`, `getMemOpcode`, `getTargetMemOpcode`, `getTwoOperandOpcode` 等 TableGen 记录。

### Lines 166-220 / 第 166-220 行
```tablegen
 166: 
 167: //===----------------------------------------------------------------------===//
 168: // Instruction formats
 169: //===----------------------------------------------------------------------===//
 170: //
 171: // Formats are specified using operand field declarations of the form:
 172: //
 173: //   bits<4> Rn   : register input or output for operand n
 174: //   bits<5> Vn   : vector register input or output for operand n
 175: //   bits<m> In   : immediate value of width m for operand n
 176: //   bits<4> Bn   : base register for address operand n
 177: //   bits<m> Dn   : displacement for address operand n
 178: //   bits<5> Vn   : vector index for address operand n
 179: //   bits<4> Xn   : index register for address operand n
 180: //   bits<4> Mn   : mode value for operand n
 181: //
 182: // The operand numbers ("n" in the list above) follow the architecture manual.
 183: // Assembly operands sometimes have a different order; in particular, R3 often
 184: // is often written between operands 1 and 2.
 185: //
 186: //===----------------------------------------------------------------------===//
 187: 
 188: class InstE<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 189:   : InstSystemZ<2, outs, ins, asmstr, pattern> {
 190:   field bits<16> Inst;
 191: 
 192:   let Inst = op;
 193: }
 194: 
 195: class InstI<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 196:   : InstSystemZ<2, outs, ins, asmstr, pattern> {
 197:   field bits<16> Inst;
 198: 
 199:   bits<8> I1;
 200: 
 201:   let Inst{15-8} = op;
 202:   let Inst{7-0}  = I1;
 203: }
 204: 
 205: class InstIE<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 206:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 207:   field bits<32> Inst;
 208: 
 209:   bits<4> I1;
 210:   bits<4> I2;
 211: 
 212:   let Inst{31-16} = op;
 213:   let Inst{15-8}  = 0;
 214:   let Inst{7-4}   = I1;
 215:   let Inst{3-0}   = I2;
 216: }
 217: 
 218: class InstMII<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 219:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 220:   field bits<48> Inst;
```
- **EN**: This block declares or refines TableGen records such as `InstE`, `InstI`, `InstIE`, `InstMII`.
- **CN**: 该代码块声明或细化了 `InstE`, `InstI`, `InstIE`, `InstMII` 等 TableGen 记录。

### Lines 221-275 / 第 221-275 行
```tablegen
 221: 
 222:   bits<4> M1;
 223:   bits<12> RI2;
 224:   bits<24> RI3;
 225: 
 226:   let Inst{47-40} = op;
 227:   let Inst{39-36} = M1;
 228:   let Inst{35-24} = RI2;
 229:   let Inst{23-0}  = RI3;
 230: }
 231: 
 232: class InstRIa<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 233:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 234:   field bits<32> Inst;
 235: 
 236:   bits<4> R1;
 237:   bits<16> I2;
 238: 
 239:   let Inst{31-24} = op{11-4};
 240:   let Inst{23-20} = R1;
 241:   let Inst{19-16} = op{3-0};
 242:   let Inst{15-0}  = I2;
 243: }
 244: 
 245: class InstRIb<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 246:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 247:   field bits<32> Inst;
 248: 
 249:   bits<4> R1;
 250:   bits<16> RI2;
 251: 
 252:   let Inst{31-24} = op{11-4};
 253:   let Inst{23-20} = R1;
 254:   let Inst{19-16} = op{3-0};
 255:   let Inst{15-0}  = RI2;
 256: }
 257: 
 258: class InstRIc<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 259:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 260:   field bits<32> Inst;
 261: 
 262:   bits<4> M1;
 263:   bits<16> RI2;
 264: 
 265:   let Inst{31-24} = op{11-4};
 266:   let Inst{23-20} = M1;
 267:   let Inst{19-16} = op{3-0};
 268:   let Inst{15-0}  = RI2;
 269: }
 270: 
 271: class InstRIEa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 272:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 273:   field bits<48> Inst;
 274: 
 275:   bits<4> R1;
```
- **EN**: This block declares or refines TableGen records such as `InstRIa`, `InstRIb`, `InstRIc`, `InstRIEa`.
- **CN**: 该代码块声明或细化了 `InstRIa`, `InstRIb`, `InstRIc`, `InstRIEa` 等 TableGen 记录。

### Lines 276-330 / 第 276-330 行
```tablegen
 276:   bits<16> I2;
 277:   bits<4> M3;
 278: 
 279:   let Inst{47-40} = op{15-8};
 280:   let Inst{39-36} = R1;
 281:   let Inst{35-32} = 0;
 282:   let Inst{31-16} = I2;
 283:   let Inst{15-12} = M3;
 284:   let Inst{11-8}  = 0;
 285:   let Inst{7-0}   = op{7-0};
 286: }
 287: 
 288: class InstRIEb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 289:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 290:   field bits<48> Inst;
 291: 
 292:   bits<4> R1;
 293:   bits<4> R2;
 294:   bits<4> M3;
 295:   bits<16> RI4;
 296: 
 297:   let Inst{47-40} = op{15-8};
 298:   let Inst{39-36} = R1;
 299:   let Inst{35-32} = R2;
 300:   let Inst{31-16} = RI4;
 301:   let Inst{15-12} = M3;
 302:   let Inst{11-8}  = 0;
 303:   let Inst{7-0}   = op{7-0};
 304: }
 305: 
 306: class InstRIEc<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 307:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 308:   field bits<48> Inst;
 309: 
 310:   bits<4> R1;
 311:   bits<8> I2;
 312:   bits<4> M3;
 313:   bits<16> RI4;
 314: 
 315:   let Inst{47-40} = op{15-8};
 316:   let Inst{39-36} = R1;
 317:   let Inst{35-32} = M3;
 318:   let Inst{31-16} = RI4;
 319:   let Inst{15-8}  = I2;
 320:   let Inst{7-0}   = op{7-0};
 321: }
 322: 
 323: class InstRIEd<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 324:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 325:   field bits<48> Inst;
 326: 
 327:   bits<4> R1;
 328:   bits<4> R3;
 329:   bits<16> I2;
 330: 
```
- **EN**: This block declares or refines TableGen records such as `InstRIEb`, `InstRIEc`, `InstRIEd`.
- **CN**: 该代码块声明或细化了 `InstRIEb`, `InstRIEc`, `InstRIEd` 等 TableGen 记录。

### Lines 331-385 / 第 331-385 行
```tablegen
 331:   let Inst{47-40} = op{15-8};
 332:   let Inst{39-36} = R1;
 333:   let Inst{35-32} = R3;
 334:   let Inst{31-16} = I2;
 335:   let Inst{15-8}  = 0;
 336:   let Inst{7-0}   = op{7-0};
 337: }
 338: 
 339: class InstRIEe<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 340:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 341:   field bits<48> Inst;
 342: 
 343:   bits<4> R1;
 344:   bits<4> R3;
 345:   bits<16> RI2;
 346: 
 347:   let Inst{47-40} = op{15-8};
 348:   let Inst{39-36} = R1;
 349:   let Inst{35-32} = R3;
 350:   let Inst{31-16} = RI2;
 351:   let Inst{15-8}  = 0;
 352:   let Inst{7-0}   = op{7-0};
 353: }
 354: 
 355: class InstRIEf<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern,
 356:                bits<8> I3Or = 0, bits<8> I4Or = 0>
 357:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 358:   field bits<48> Inst;
 359: 
 360:   bits<4> R1;
 361:   bits<4> R2;
 362:   bits<8> I3;
 363:   bits<8> I4;
 364:   bits<8> I5;
 365: 
 366:   let Inst{47-40} = op{15-8};
 367:   let Inst{39-36} = R1;
 368:   let Inst{35-32} = R2;
 369:   let Inst{31} = !if(I3Or{7}, 1, I3{7});
 370:   let Inst{30} = !if(I3Or{6}, 1, I3{6});
 371:   let Inst{29} = !if(I3Or{5}, 1, I3{5});
 372:   let Inst{28} = !if(I3Or{4}, 1, I3{4});
 373:   let Inst{27} = !if(I3Or{3}, 1, I3{3});
 374:   let Inst{26} = !if(I3Or{2}, 1, I3{2});
 375:   let Inst{25} = !if(I3Or{1}, 1, I3{1});
 376:   let Inst{24} = !if(I3Or{0}, 1, I3{0});
 377:   let Inst{23} = !if(I4Or{7}, 1, I4{7});
 378:   let Inst{22} = !if(I4Or{6}, 1, I4{6});
 379:   let Inst{21} = !if(I4Or{5}, 1, I4{5});
 380:   let Inst{20} = !if(I4Or{4}, 1, I4{4});
 381:   let Inst{19} = !if(I4Or{3}, 1, I4{3});
 382:   let Inst{18} = !if(I4Or{2}, 1, I4{2});
 383:   let Inst{17} = !if(I4Or{1}, 1, I4{1});
 384:   let Inst{16} = !if(I4Or{0}, 1, I4{0});
 385:   let Inst{15-8}  = I5;
```
- **EN**: This block declares or refines TableGen records such as `InstRIEe`, `InstRIEf`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `InstRIEe`, `InstRIEf` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 386-440 / 第 386-440 行
```tablegen
 386:   let Inst{7-0}   = op{7-0};
 387: }
 388: 
 389: class InstRIEg<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 390:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 391:   field bits<48> Inst;
 392: 
 393:   bits<4> R1;
 394:   bits<4> M3;
 395:   bits<16> I2;
 396: 
 397:   let Inst{47-40} = op{15-8};
 398:   let Inst{39-36} = R1;
 399:   let Inst{35-32} = M3;
 400:   let Inst{31-16} = I2;
 401:   let Inst{15-8}  = 0;
 402:   let Inst{7-0}   = op{7-0};
 403: }
 404: 
 405: class InstRILa<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 406:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 407:   field bits<48> Inst;
 408: 
 409:   bits<4> R1;
 410:   bits<32> I2;
 411: 
 412:   let Inst{47-40} = op{11-4};
 413:   let Inst{39-36} = R1;
 414:   let Inst{35-32} = op{3-0};
 415:   let Inst{31-0}  = I2;
 416: }
 417: 
 418: class InstRILb<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 419:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 420:   field bits<48> Inst;
 421: 
 422:   bits<4> R1;
 423:   bits<32> RI2;
 424: 
 425:   let Inst{47-40} = op{11-4};
 426:   let Inst{39-36} = R1;
 427:   let Inst{35-32} = op{3-0};
 428:   let Inst{31-0}  = RI2;
 429: }
 430: 
 431: class InstRILc<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 432:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 433:   field bits<48> Inst;
 434: 
 435:   bits<4> M1;
 436:   bits<32> RI2;
 437: 
 438:   let Inst{47-40} = op{11-4};
 439:   let Inst{39-36} = M1;
 440:   let Inst{35-32} = op{3-0};
```
- **EN**: This block declares or refines TableGen records such as `InstRIEg`, `InstRILa`, `InstRILb`, `InstRILc`.
- **CN**: 该代码块声明或细化了 `InstRIEg`, `InstRILa`, `InstRILb`, `InstRILc` 等 TableGen 记录。

### Lines 441-495 / 第 441-495 行
```tablegen
 441:   let Inst{31-0}  = RI2;
 442: }
 443: 
 444: class InstRIS<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 445:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 446:   field bits<48> Inst;
 447: 
 448:   bits<4> R1;
 449:   bits<8> I2;
 450:   bits<4> M3;
 451:   bits<4> B4;
 452:   bits<12> D4;
 453: 
 454:   let Inst{47-40} = op{15-8};
 455:   let Inst{39-36} = R1;
 456:   let Inst{35-32} = M3;
 457:   let Inst{31-28} = B4;
 458:   let Inst{27-16} = D4;
 459:   let Inst{15-8}  = I2;
 460:   let Inst{7-0}   = op{7-0};
 461: }
 462: 
 463: class InstRR<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 464:   : InstSystemZ<2, outs, ins, asmstr, pattern> {
 465:   field bits<16> Inst;
 466: 
 467:   bits<4> R1;
 468:   bits<4> R2;
 469: 
 470:   let Inst{15-8} = op;
 471:   let Inst{7-4}  = R1;
 472:   let Inst{3-0}  = R2;
 473: }
 474: 
 475: class InstRRD<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 476:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 477:   field bits<32> Inst;
 478: 
 479:   bits<4> R1;
 480:   bits<4> R3;
 481:   bits<4> R2;
 482: 
 483:   let Inst{31-16} = op;
 484:   let Inst{15-12} = R1;
 485:   let Inst{11-8}  = 0;
 486:   let Inst{7-4}   = R3;
 487:   let Inst{3-0}   = R2;
 488: }
 489: 
 490: class InstRRE<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 491:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 492:   field bits<32> Inst;
 493: 
 494:   bits<4> R1;
 495:   bits<4> R2;
```
- **EN**: This block declares or refines TableGen records such as `InstRIS`, `InstRR`, `InstRRD`, `InstRRE`.
- **CN**: 该代码块声明或细化了 `InstRIS`, `InstRR`, `InstRRD`, `InstRRE` 等 TableGen 记录。

### Lines 496-550 / 第 496-550 行
```tablegen
 496: 
 497:   let Inst{31-16} = op;
 498:   let Inst{15-8}  = 0;
 499:   let Inst{7-4}   = R1;
 500:   let Inst{3-0}   = R2;
 501: }
 502: 
 503: class InstRRFa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 504:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 505:   field bits<32> Inst;
 506: 
 507:   bits<4> R1;
 508:   bits<4> R2;
 509:   bits<4> R3;
 510:   bits<4> M4;
 511: 
 512:   let Inst{31-16} = op;
 513:   let Inst{15-12} = R3;
 514:   let Inst{11-8}  = M4;
 515:   let Inst{7-4}   = R1;
 516:   let Inst{3-0}   = R2;
 517: }
 518: 
 519: class InstRRFb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 520:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 521:   field bits<32> Inst;
 522: 
 523:   bits<4> R1;
 524:   bits<4> R2;
 525:   bits<4> R3;
 526:   bits<4> M4;
 527: 
 528:   let Inst{31-16} = op;
 529:   let Inst{15-12} = R3;
 530:   let Inst{11-8}  = M4;
 531:   let Inst{7-4}   = R1;
 532:   let Inst{3-0}   = R2;
 533: }
 534: 
 535: class InstRRFc<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 536:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 537:   field bits<32> Inst;
 538: 
 539:   bits<4> R1;
 540:   bits<4> R2;
 541:   bits<4> M3;
 542: 
 543:   let Inst{31-16} = op;
 544:   let Inst{15-12} = M3;
 545:   let Inst{11-8}  = 0;
 546:   let Inst{7-4}   = R1;
 547:   let Inst{3-0}   = R2;
 548: }
 549: 
 550: class InstRRFd<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
```
- **EN**: This block declares or refines TableGen records such as `InstRRFa`, `InstRRFb`, `InstRRFc`, `InstRRFd`.
- **CN**: 该代码块声明或细化了 `InstRRFa`, `InstRRFb`, `InstRRFc`, `InstRRFd` 等 TableGen 记录。

### Lines 551-605 / 第 551-605 行
```tablegen
 551:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 552:   field bits<32> Inst;
 553: 
 554:   bits<4> R1;
 555:   bits<4> R2;
 556:   bits<4> M4;
 557: 
 558:   let Inst{31-16} = op;
 559:   let Inst{15-12} = 0;
 560:   let Inst{11-8}  = M4;
 561:   let Inst{7-4}   = R1;
 562:   let Inst{3-0}   = R2;
 563: }
 564: 
 565: class InstRRFe<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 566:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 567:   field bits<32> Inst;
 568: 
 569:   bits<4> R1;
 570:   bits<4> R2;
 571:   bits<4> M3;
 572:   bits<4> M4;
 573: 
 574:   let Inst{31-16} = op;
 575:   let Inst{15-12} = M3;
 576:   let Inst{11-8}  = M4;
 577:   let Inst{7-4}   = R1;
 578:   let Inst{3-0}   = R2;
 579: }
 580: 
 581: class InstRRS<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 582:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 583:   field bits<48> Inst;
 584: 
 585:   bits<4> R1;
 586:   bits<4> R2;
 587:   bits<4> M3;
 588:   bits<4> B4;
 589:   bits<12> D4;
 590: 
 591:   let Inst{47-40} = op{15-8};
 592:   let Inst{39-36} = R1;
 593:   let Inst{35-32} = R2;
 594:   let Inst{31-28} = B4;
 595:   let Inst{27-16} = D4;
 596:   let Inst{15-12} = M3;
 597:   let Inst{11-8}  = 0;
 598:   let Inst{7-0}   = op{7-0};
 599: }
 600: 
 601: class InstRXa<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 602:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 603:   field bits<32> Inst;
 604: 
 605:   bits<4> R1;
```
- **EN**: This block declares or refines TableGen records such as `InstRRFe`, `InstRRS`, `InstRXa`.
- **CN**: 该代码块声明或细化了 `InstRRFe`, `InstRRS`, `InstRXa` 等 TableGen 记录。

### Lines 606-660 / 第 606-660 行
```tablegen
 606:   bits<4> X2;
 607:   bits<4> B2;
 608:   bits<12> D2;
 609: 
 610:   let Inst{31-24} = op;
 611:   let Inst{23-20} = R1;
 612:   let Inst{19-16} = X2;
 613:   let Inst{15-12} = B2;
 614:   let Inst{11-0}  = D2;
 615: 
 616:   let HasIndex = 1;
 617: }
 618: 
 619: class InstRXb<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 620:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 621:   field bits<32> Inst;
 622: 
 623:   bits<4> M1;
 624:   bits<4> X2;
 625:   bits<4> B2;
 626:   bits<12> D2;
 627: 
 628:   let Inst{31-24} = op;
 629:   let Inst{23-20} = M1;
 630:   let Inst{19-16} = X2;
 631:   let Inst{15-12} = B2;
 632:   let Inst{11-0}  = D2;
 633: 
 634:   let HasIndex = 1;
 635: }
 636: 
 637: class InstRXE<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 638:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 639:   field bits<48> Inst;
 640: 
 641:   bits<4> R1;
 642:   bits<4> X2;
 643:   bits<4> B2;
 644:   bits<12> D2;
 645:   bits<4> M3;
 646: 
 647:   let Inst{47-40} = op{15-8};
 648:   let Inst{39-36} = R1;
 649:   let Inst{35-32} = X2;
 650:   let Inst{31-28} = B2;
 651:   let Inst{27-16} = D2;
 652:   let Inst{15-12} = M3;
 653:   let Inst{11-8}  = 0;
 654:   let Inst{7-0}   = op{7-0};
 655: 
 656:   let HasIndex = 1;
 657: }
 658: 
 659: class InstRXF<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 660:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `InstRXb`, `InstRXE`, `InstRXF`.
- **CN**: 该代码块声明或细化了 `InstRXb`, `InstRXE`, `InstRXF` 等 TableGen 记录。

### Lines 661-715 / 第 661-715 行
```tablegen
 661:   field bits<48> Inst;
 662: 
 663:   bits<4> R1;
 664:   bits<4> R3;
 665:   bits<4> X2;
 666:   bits<4> B2;
 667:   bits<12> D2;
 668: 
 669:   let Inst{47-40} = op{15-8};
 670:   let Inst{39-36} = R3;
 671:   let Inst{35-32} = X2;
 672:   let Inst{31-28} = B2;
 673:   let Inst{27-16} = D2;
 674:   let Inst{15-12} = R1;
 675:   let Inst{11-8}  = 0;
 676:   let Inst{7-0}   = op{7-0};
 677: 
 678:   let HasIndex = 1;
 679: }
 680: 
 681: class InstRXYa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 682:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 683:   field bits<48> Inst;
 684: 
 685:   bits<4> R1;
 686:   bits<4> X2;
 687:   bits<4> B2;
 688:   bits<20> D2;
 689: 
 690:   let Inst{47-40} = op{15-8};
 691:   let Inst{39-36} = R1;
 692:   let Inst{35-32} = X2;
 693:   let Inst{31-28} = B2;
 694:   let Inst{27-16} = D2{11-0};
 695:   let Inst{15-8}  = D2{19-12};
 696:   let Inst{7-0}   = op{7-0};
 697: 
 698:   let Has20BitOffset = 1;
 699:   let HasIndex = 1;
 700: }
 701: 
 702: class InstRXYb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 703:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 704:   field bits<48> Inst;
 705: 
 706:   bits<4> M1;
 707:   bits<4> X2;
 708:   bits<4> B2;
 709:   bits<20> D2;
 710: 
 711:   let Inst{47-40} = op{15-8};
 712:   let Inst{39-36} = M1;
 713:   let Inst{35-32} = X2;
 714:   let Inst{31-28} = B2;
 715:   let Inst{27-16} = D2{11-0};
```
- **EN**: This block declares or refines TableGen records such as `InstRXYa`, `InstRXYb`.
- **CN**: 该代码块声明或细化了 `InstRXYa`, `InstRXYb` 等 TableGen 记录。

### Lines 716-770 / 第 716-770 行
```tablegen
 716:   let Inst{15-8}  = D2{19-12};
 717:   let Inst{7-0}   = op{7-0};
 718: 
 719:   let Has20BitOffset = 1;
 720:   let HasIndex = 1;
 721: }
 722: 
 723: class InstRSa<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 724:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 725:   field bits<32> Inst;
 726: 
 727:   bits<4> R1;
 728:   bits<4> R3;
 729:   bits<4> B2;
 730:   bits<12> D2;
 731: 
 732:   let Inst{31-24} = op;
 733:   let Inst{23-20} = R1;
 734:   let Inst{19-16} = R3;
 735:   let Inst{15-12} = B2;
 736:   let Inst{11-0}  = D2;
 737: }
 738: 
 739: class InstRSb<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 740:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 741:   field bits<32> Inst;
 742: 
 743:   bits<4> R1;
 744:   bits<4> M3;
 745:   bits<4> B2;
 746:   bits<12> D2;
 747: 
 748:   let Inst{31-24} = op;
 749:   let Inst{23-20} = R1;
 750:   let Inst{19-16} = M3;
 751:   let Inst{15-12} = B2;
 752:   let Inst{11-0}  = D2;
 753: }
 754: 
 755: class InstRSEa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 756:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 757:   field bits<48> Inst;
 758: 
 759:   bits<4> R1;
 760:   bits<4> R3;
 761:   bits<4> B2;
 762:   bits<12> D2;
 763: 
 764:   let Inst{47-40} = op{15-8};
 765:   let Inst{39-36} = R1;
 766:   let Inst{35-32} = R3;
 767:   let Inst{31-28} = B2;
 768:   let Inst{27-16} = D2;
 769:   let Inst{15-8}  = 0;
 770:   let Inst{7-0}   = op{7-0};
```
- **EN**: This block declares or refines TableGen records such as `InstRSa`, `InstRSb`, `InstRSEa`.
- **CN**: 该代码块声明或细化了 `InstRSa`, `InstRSb`, `InstRSEa` 等 TableGen 记录。

### Lines 771-825 / 第 771-825 行
```tablegen
 771: }
 772: 
 773: class InstRSI<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 774:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 775:   field bits<32> Inst;
 776: 
 777:   bits<4> R1;
 778:   bits<4> R3;
 779:   bits<16> RI2;
 780: 
 781:   let Inst{31-24} = op;
 782:   let Inst{23-20} = R1;
 783:   let Inst{19-16} = R3;
 784:   let Inst{15-0}  = RI2;
 785: }
 786: 
 787: class InstRSLa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 788:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 789:   field bits<48> Inst;
 790: 
 791:   bits<4> B1;
 792:   bits<12> D1;
 793:   bits<4> L1;
 794: 
 795:   let Inst{47-40} = op{15-8};
 796:   let Inst{39-36} = L1;
 797:   let Inst{35-32} = 0;
 798:   let Inst{31-28} = B1;
 799:   let Inst{27-16} = D1;
 800:   let Inst{15-8}  = 0;
 801:   let Inst{7-0}   = op{7-0};
 802: }
 803: 
 804: class InstRSLb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 805:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 806:   field bits<48> Inst;
 807: 
 808:   bits<4> R1;
 809:   bits<4> B2;
 810:   bits<12> D2;
 811:   bits<8> L2;
 812:   bits<4> M3;
 813: 
 814:   let Inst{47-40} = op{15-8};
 815:   let Inst{39-32} = L2;
 816:   let Inst{31-28} = B2;
 817:   let Inst{27-16} = D2;
 818:   let Inst{15-12} = R1;
 819:   let Inst{11-8}  = M3;
 820:   let Inst{7-0}   = op{7-0};
 821: }
 822: 
 823: class InstRSYa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 824:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 825:   field bits<48> Inst;
```
- **EN**: This block declares or refines TableGen records such as `InstRSI`, `InstRSLa`, `InstRSLb`, `InstRSYa`.
- **CN**: 该代码块声明或细化了 `InstRSI`, `InstRSLa`, `InstRSLb`, `InstRSYa` 等 TableGen 记录。

### Lines 826-880 / 第 826-880 行
```tablegen
 826: 
 827:   bits<4> R1;
 828:   bits<4> R3;
 829:   bits<4> B2;
 830:   bits<20> D2;
 831: 
 832:   let Inst{47-40} = op{15-8};
 833:   let Inst{39-36} = R1;
 834:   let Inst{35-32} = R3;
 835:   let Inst{31-28} = B2;
 836:   let Inst{27-16} = D2{11-0};
 837:   let Inst{15-8}  = D2{19-12};
 838:   let Inst{7-0}   = op{7-0};
 839: 
 840:   let Has20BitOffset = 1;
 841: }
 842: 
 843: class InstRSYb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 844:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 845:   field bits<48> Inst;
 846: 
 847:   bits<4> R1;
 848:   bits<4> M3;
 849:   bits<4> B2;
 850:   bits<20> D2;
 851: 
 852:   let Inst{47-40} = op{15-8};
 853:   let Inst{39-36} = R1;
 854:   let Inst{35-32} = M3;
 855:   let Inst{31-28} = B2;
 856:   let Inst{27-16} = D2{11-0};
 857:   let Inst{15-8}  = D2{19-12};
 858:   let Inst{7-0}   = op{7-0};
 859: 
 860:   let Has20BitOffset = 1;
 861: }
 862: 
 863: class InstSI<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 864:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
 865:   field bits<32> Inst;
 866: 
 867:   bits<4> B1;
 868:   bits<12> D1;
 869:   bits<8> I2;
 870: 
 871:   let Inst{31-24} = op;
 872:   let Inst{23-16} = I2;
 873:   let Inst{15-12} = B1;
 874:   let Inst{11-0}  = D1;
 875: }
 876: 
 877: class InstSIL<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 878:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 879:   field bits<48> Inst;
 880: 
```
- **EN**: This block declares or refines TableGen records such as `InstRSYb`, `InstSI`, `InstSIL`.
- **CN**: 该代码块声明或细化了 `InstRSYb`, `InstSI`, `InstSIL` 等 TableGen 记录。

### Lines 881-935 / 第 881-935 行
```tablegen
 881:   bits<4> B1;
 882:   bits<12> D1;
 883:   bits<16> I2;
 884: 
 885:   let Inst{47-32} = op;
 886:   let Inst{31-28} = B1;
 887:   let Inst{27-16} = D1;
 888:   let Inst{15-0}  = I2;
 889: }
 890: 
 891: class InstSIY<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 892:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 893:   field bits<48> Inst;
 894: 
 895:   bits<4> B1;
 896:   bits<20> D1;
 897:   bits<8> I2;
 898: 
 899:   let Inst{47-40} = op{15-8};
 900:   let Inst{39-32} = I2;
 901:   let Inst{31-28} = B1;
 902:   let Inst{27-16} = D1{11-0};
 903:   let Inst{15-8}  = D1{19-12};
 904:   let Inst{7-0}   = op{7-0};
 905: 
 906:   let Has20BitOffset = 1;
 907: }
 908: 
 909: class InstSMI<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 910:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 911:   field bits<48> Inst;
 912: 
 913:   bits<4> M1;
 914:   bits<16> RI2;
 915:   bits<4> B3;
 916:   bits<12> D3;
 917: 
 918:   let Inst{47-40} = op;
 919:   let Inst{39-36} = M1;
 920:   let Inst{35-32} = 0;
 921:   let Inst{31-28} = B3;
 922:   let Inst{27-16} = D3;
 923:   let Inst{15-0}  = RI2;
 924: }
 925: 
 926: class InstSSa<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 927:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 928:   field bits<48> Inst;
 929: 
 930:   bits<4> B1;
 931:   bits<12> D1;
 932:   bits<8> L1;
 933:   bits<4> B2;
 934:   bits<12> D2;
 935: 
```
- **EN**: This block declares or refines TableGen records such as `InstSIY`, `InstSMI`, `InstSSa`.
- **CN**: 该代码块声明或细化了 `InstSIY`, `InstSMI`, `InstSSa` 等 TableGen 记录。

### Lines 936-990 / 第 936-990 行
```tablegen
 936:   let Inst{47-40} = op;
 937:   let Inst{39-32} = L1;
 938:   let Inst{31-28} = B1;
 939:   let Inst{27-16} = D1;
 940:   let Inst{15-12} = B2;
 941:   let Inst{11-0}  = D2;
 942: }
 943: 
 944: class InstSSb<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 945:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 946:   field bits<48> Inst;
 947: 
 948:   bits<4> B1;
 949:   bits<12> D1;
 950:   bits<4> L1;
 951:   bits<4> B2;
 952:   bits<12> D2;
 953:   bits<4> L2;
 954: 
 955:   let Inst{47-40} = op;
 956:   let Inst{39-36} = L1;
 957:   let Inst{35-32} = L2;
 958:   let Inst{31-28} = B1;
 959:   let Inst{27-16} = D1;
 960:   let Inst{15-12} = B2;
 961:   let Inst{11-0} = D2;
 962: }
 963: 
 964: class InstSSc<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 965:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 966:   field bits<48> Inst;
 967: 
 968:   bits<4> B1;
 969:   bits<12> D1;
 970:   bits<4> L1;
 971:   bits<4> B2;
 972:   bits<12> D2;
 973:   bits<4> I3;
 974: 
 975:   let Inst{47-40} = op;
 976:   let Inst{39-36} = L1;
 977:   let Inst{35-32} = I3;
 978:   let Inst{31-28} = B1;
 979:   let Inst{27-16} = D1;
 980:   let Inst{15-12} = B2;
 981:   let Inst{11-0}  = D2;
 982: }
 983: 
 984: class InstSSd<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
 985:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
 986:   field bits<48> Inst;
 987: 
 988:   bits<4> R1;
 989:   bits<4> B1;
 990:   bits<12> D1;
```
- **EN**: This block declares or refines TableGen records such as `InstSSb`, `InstSSc`, `InstSSd`.
- **CN**: 该代码块声明或细化了 `InstSSb`, `InstSSc`, `InstSSd` 等 TableGen 记录。

### Lines 991-1045 / 第 991-1045 行
```tablegen
 991:   bits<4> B2;
 992:   bits<12> D2;
 993:   bits<4> R3;
 994: 
 995:   let Inst{47-40} = op;
 996:   let Inst{39-36} = R1;
 997:   let Inst{35-32} = R3;
 998:   let Inst{31-28} = B1;
 999:   let Inst{27-16} = D1;
1000:   let Inst{15-12} = B2;
1001:   let Inst{11-0}  = D2;
1002: }
1003: 
1004: class InstSSe<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1005:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1006:   field bits<48> Inst;
1007: 
1008:   bits<4> R1;
1009:   bits<4> B2;
1010:   bits<12> D2;
1011:   bits<4> R3;
1012:   bits<4> B4;
1013:   bits<12> D4;
1014: 
1015:   let Inst{47-40} = op;
1016:   let Inst{39-36} = R1;
1017:   let Inst{35-32} = R3;
1018:   let Inst{31-28} = B2;
1019:   let Inst{27-16} = D2;
1020:   let Inst{15-12} = B4;
1021:   let Inst{11-0}  = D4;
1022: }
1023: 
1024: class InstSSf<bits<8> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1025:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1026:   field bits<48> Inst;
1027: 
1028:   bits<4> B1;
1029:   bits<12> D1;
1030:   bits<4> B2;
1031:   bits<12> D2;
1032:   bits<8> L2;
1033: 
1034:   let Inst{47-40} = op;
1035:   let Inst{39-32} = L2;
1036:   let Inst{31-28} = B1;
1037:   let Inst{27-16} = D1;
1038:   let Inst{15-12} = B2;
1039:   let Inst{11-0}  = D2;
1040: }
1041: 
1042: class InstSSE<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1043:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1044:   field bits<48> Inst;
1045: 
```
- **EN**: This block declares or refines TableGen records such as `InstSSe`, `InstSSf`, `InstSSE`.
- **CN**: 该代码块声明或细化了 `InstSSe`, `InstSSf`, `InstSSE` 等 TableGen 记录。

### Lines 1046-1100 / 第 1046-1100 行
```tablegen
1046:   bits<4> B1;
1047:   bits<12> D1;
1048:   bits<4> B2;
1049:   bits<12> D2;
1050: 
1051:   let Inst{47-32} = op;
1052:   let Inst{31-28} = B1;
1053:   let Inst{27-16} = D1;
1054:   let Inst{15-12} = B2;
1055:   let Inst{11-0}  = D2;
1056: }
1057: 
1058: class InstSSF<bits<12> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1059:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1060:   field bits<48> Inst;
1061: 
1062:   bits<4> B1;
1063:   bits<12> D1;
1064:   bits<4> B2;
1065:   bits<12> D2;
1066:   bits<4>  R3;
1067: 
1068:   let Inst{47-40} = op{11-4};
1069:   let Inst{39-36} = R3;
1070:   let Inst{35-32} = op{3-0};
1071:   let Inst{31-28} = B1;
1072:   let Inst{27-16} = D1;
1073:   let Inst{15-12} = B2;
1074:   let Inst{11-0}  = D2;
1075: }
1076: 
1077: class InstS<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1078:   : InstSystemZ<4, outs, ins, asmstr, pattern> {
1079:   field bits<32> Inst;
1080: 
1081:   bits<4> B2;
1082:   bits<12> D2;
1083: 
1084:   let Inst{31-16} = op;
1085:   let Inst{15-12} = B2;
1086:   let Inst{11-0}  = D2;
1087: }
1088: 
1089: class InstVRIa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1090:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1091:   field bits<48> Inst;
1092: 
1093:   bits<5> V1;
1094:   bits<16> I2;
1095:   bits<4> M3;
1096: 
1097:   let Inst{47-40} = op{15-8};
1098:   let Inst{39-36} = V1{3-0};
1099:   let Inst{35-32} = 0;
1100:   let Inst{31-16} = I2;
```
- **EN**: This block declares or refines TableGen records such as `InstSSF`, `InstS`, `InstVRIa`.
- **CN**: 该代码块声明或细化了 `InstSSF`, `InstS`, `InstVRIa` 等 TableGen 记录。

### Lines 1101-1155 / 第 1101-1155 行
```tablegen
1101:   let Inst{15-12} = M3;
1102:   let Inst{11}    = V1{4};
1103:   let Inst{10-8}  = 0;
1104:   let Inst{7-0}   = op{7-0};
1105: }
1106: 
1107: class InstVRIb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1108:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1109:   field bits<48> Inst;
1110: 
1111:   bits<5> V1;
1112:   bits<8> I2;
1113:   bits<8> I3;
1114:   bits<4> M4;
1115: 
1116:   let Inst{47-40} = op{15-8};
1117:   let Inst{39-36} = V1{3-0};
1118:   let Inst{35-32} = 0;
1119:   let Inst{31-24} = I2;
1120:   let Inst{23-16} = I3;
1121:   let Inst{15-12} = M4;
1122:   let Inst{11}    = V1{4};
1123:   let Inst{10-8}  = 0;
1124:   let Inst{7-0}   = op{7-0};
1125: }
1126: 
1127: class InstVRIc<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1128:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1129:   field bits<48> Inst;
1130: 
1131:   bits<5> V1;
1132:   bits<5> V3;
1133:   bits<16> I2;
1134:   bits<4> M4;
1135: 
1136:   let Inst{47-40} = op{15-8};
1137:   let Inst{39-36} = V1{3-0};
1138:   let Inst{35-32} = V3{3-0};
1139:   let Inst{31-16} = I2;
1140:   let Inst{15-12} = M4;
1141:   let Inst{11}    = V1{4};
1142:   let Inst{10}    = V3{4};
1143:   let Inst{9-8}   = 0;
1144:   let Inst{7-0}   = op{7-0};
1145: }
1146: 
1147: class InstVRId<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1148:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1149:   field bits<48> Inst;
1150: 
1151:   bits<5> V1;
1152:   bits<5> V2;
1153:   bits<5> V3;
1154:   bits<8> I4;
1155:   bits<4> M5;
```
- **EN**: This block declares or refines TableGen records such as `InstVRIb`, `InstVRIc`, `InstVRId`.
- **CN**: 该代码块声明或细化了 `InstVRIb`, `InstVRIc`, `InstVRId` 等 TableGen 记录。

### Lines 1156-1210 / 第 1156-1210 行
```tablegen
1156: 
1157:   let Inst{47-40} = op{15-8};
1158:   let Inst{39-36} = V1{3-0};
1159:   let Inst{35-32} = V2{3-0};
1160:   let Inst{31-28} = V3{3-0};
1161:   let Inst{27-24} = 0;
1162:   let Inst{23-16} = I4;
1163:   let Inst{15-12} = M5;
1164:   let Inst{11}    = V1{4};
1165:   let Inst{10}    = V2{4};
1166:   let Inst{9}     = V3{4};
1167:   let Inst{8}     = 0;
1168:   let Inst{7-0}   = op{7-0};
1169: }
1170: 
1171: class InstVRIe<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1172:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1173:   field bits<48> Inst;
1174: 
1175:   bits<5> V1;
1176:   bits<5> V2;
1177:   bits<12> I3;
1178:   bits<4> M4;
1179:   bits<4> M5;
1180: 
1181:   let Inst{47-40} = op{15-8};
1182:   let Inst{39-36} = V1{3-0};
1183:   let Inst{35-32} = V2{3-0};
1184:   let Inst{31-20} = I3;
1185:   let Inst{19-16} = M5;
1186:   let Inst{15-12} = M4;
1187:   let Inst{11}    = V1{4};
1188:   let Inst{10}    = V2{4};
1189:   let Inst{9-8}   = 0;
1190:   let Inst{7-0}   = op{7-0};
1191: }
1192: 
1193: class InstVRIf<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1194:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1195:   field bits<48> Inst;
1196: 
1197:   bits<5> V1;
1198:   bits<5> V2;
1199:   bits<5> V3;
1200:   bits<8> I4;
1201:   bits<4> M5;
1202: 
1203:   let Inst{47-40} = op{15-8};
1204:   let Inst{39-36} = V1{3-0};
1205:   let Inst{35-32} = V2{3-0};
1206:   let Inst{31-28} = V3{3-0};
1207:   let Inst{27-24} = 0;
1208:   let Inst{23-20} = M5;
1209:   let Inst{19-12} = I4;
1210:   let Inst{11}    = V1{4};
```
- **EN**: This block declares or refines TableGen records such as `InstVRIe`, `InstVRIf`.
- **CN**: 该代码块声明或细化了 `InstVRIe`, `InstVRIf` 等 TableGen 记录。

### Lines 1211-1265 / 第 1211-1265 行
```tablegen
1211:   let Inst{10}    = V2{4};
1212:   let Inst{9}     = V3{4};
1213:   let Inst{8}     = 0;
1214:   let Inst{7-0}   = op{7-0};
1215: }
1216: 
1217: class InstVRIg<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1218:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1219:   field bits<48> Inst;
1220: 
1221:   bits<5> V1;
1222:   bits<5> V2;
1223:   bits<8> I3;
1224:   bits<8> I4;
1225:   bits<4> M5;
1226: 
1227:   let Inst{47-40} = op{15-8};
1228:   let Inst{39-36} = V1{3-0};
1229:   let Inst{35-32} = V2{3-0};
1230:   let Inst{31-24} = I4;
1231:   let Inst{23-20} = M5;
1232:   let Inst{19-12} = I3;
1233:   let Inst{11}    = V1{4};
1234:   let Inst{10}    = V2{4};
1235:   let Inst{9-8}   = 0;
1236:   let Inst{7-0}   = op{7-0};
1237: }
1238: 
1239: class InstVRIh<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1240:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1241:   field bits<48> Inst;
1242: 
1243:   bits<5> V1;
1244:   bits<16> I2;
1245:   bits<4> I3;
1246: 
1247:   let Inst{47-40} = op{15-8};
1248:   let Inst{39-36} = V1{3-0};
1249:   let Inst{35-32} = 0;
1250:   let Inst{31-16} = I2;
1251:   let Inst{15-12} = I3;
1252:   let Inst{11}    = V1{4};
1253:   let Inst{10-8}  = 0;
1254:   let Inst{7-0}   = op{7-0};
1255: }
1256: 
1257: class InstVRIi<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1258:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1259:   field bits<48> Inst;
1260: 
1261:   bits<5> V1;
1262:   bits<4> R2;
1263:   bits<8> I3;
1264:   bits<4> M4;
1265: 
```
- **EN**: This block declares or refines TableGen records such as `InstVRIg`, `InstVRIh`, `InstVRIi`.
- **CN**: 该代码块声明或细化了 `InstVRIg`, `InstVRIh`, `InstVRIi` 等 TableGen 记录。

### Lines 1266-1320 / 第 1266-1320 行
```tablegen
1266:   let Inst{47-40} = op{15-8};
1267:   let Inst{39-36} = V1{3-0};
1268:   let Inst{35-32} = R2;
1269:   let Inst{31-24} = 0;
1270:   let Inst{23-20} = M4;
1271:   let Inst{19-12} = I3;
1272:   let Inst{11}    = V1{4};
1273:   let Inst{10-8}  = 0;
1274:   let Inst{7-0}   = op{7-0};
1275: }
1276: 
1277: class InstVRIj<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1278:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1279:   field bits<48> Inst;
1280: 
1281:   bits<5> V1;
1282:   bits<5> V2;
1283:   bits<8> I3;
1284:   bits<4> M4;
1285: 
1286:   let Inst{47-40} = op{15-8};
1287:   let Inst{39-36} = V1{3-0};
1288:   let Inst{35-32} = V2{3-0};
1289:   let Inst{31-24} = 0;
1290:   let Inst{23-20} = M4;
1291:   let Inst{19-12} = I3;
1292:   let Inst{11}    = V1{4};
1293:   let Inst{10}    = V2{4};
1294:   let Inst{9-8}   = 0;
1295:   let Inst{7-0}   = op{7-0};
1296: }
1297: 
1298: class InstVRIk<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1299:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1300:   field bits<48> Inst;
1301: 
1302:   bits<5> V1;
1303:   bits<5> V2;
1304:   bits<5> V3;
1305:   bits<5> V4;
1306:   bits<8> I5;
1307: 
1308:   let Inst{47-40} = op{15-8};
1309:   let Inst{39-36} = V1{3-0};
1310:   let Inst{35-32} = V2{3-0};
1311:   let Inst{31-28} = V3{3-0};
1312:   let Inst{27-24} = 0;
1313:   let Inst{23-16} = I5;
1314:   let Inst{15-12} = V4{3-0};
1315:   let Inst{11}    = V1{4};
1316:   let Inst{10}    = V2{4};
1317:   let Inst{9}     = V3{4};
1318:   let Inst{8}     = V4{4};
1319:   let Inst{7-0}   = op{7-0};
1320: }
```
- **EN**: This block declares or refines TableGen records such as `InstVRIj`, `InstVRIk`.
- **CN**: 该代码块声明或细化了 `InstVRIj`, `InstVRIk` 等 TableGen 记录。

### Lines 1321-1375 / 第 1321-1375 行
```tablegen
1321: 
1322: class InstVRIl<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1323:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1324:   field bits<48> Inst;
1325: 
1326:   bits<5> V1;
1327:   bits<5> V2;
1328:   bits<16> I3;
1329: 
1330:   let Inst{47-40} = op{15-8};
1331:   let Inst{39-36} = 0;
1332:   let Inst{35-32} = V1{3-0};
1333:   let Inst{31-28} = V2{3-0};
1334:   let Inst{27-12} = I3;
1335:   let Inst{11}    = 0;
1336:   let Inst{10}    = V1{4};
1337:   let Inst{9}     = V2{4};
1338:   let Inst{8}     = 0;
1339:   let Inst{7-0}   = op{7-0};
1340: }
1341: 
1342: // Depending on the instruction mnemonic, certain bits may be or-ed into
1343: // the M4 value provided as explicit operand.  These are passed as m4or.
1344: class InstVRRa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern,
1345:                bits<4> m4or = 0>
1346:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1347:   field bits<48> Inst;
1348: 
1349:   bits<5> V1;
1350:   bits<5> V2;
1351:   bits<4> M3;
1352:   bits<4> M4;
1353:   bits<4> M5;
1354: 
1355:   let Inst{47-40} = op{15-8};
1356:   let Inst{39-36} = V1{3-0};
1357:   let Inst{35-32} = V2{3-0};
1358:   let Inst{31-24} = 0;
1359:   let Inst{23-20} = M5;
1360:   let Inst{19}    = !if (!eq (m4or{3}, 1), 1, M4{3});
1361:   let Inst{18}    = !if (!eq (m4or{2}, 1), 1, M4{2});
1362:   let Inst{17}    = !if (!eq (m4or{1}, 1), 1, M4{1});
1363:   let Inst{16}    = !if (!eq (m4or{0}, 1), 1, M4{0});
1364:   let Inst{15-12} = M3;
1365:   let Inst{11}    = V1{4};
1366:   let Inst{10}    = V2{4};
1367:   let Inst{9-8}   = 0;
1368:   let Inst{7-0}   = op{7-0};
1369: }
1370: 
1371: // Depending on the instruction mnemonic, certain bits may be or-ed into
1372: // the M5 value provided as explicit operand.  These are passed as m5or.
1373: class InstVRRb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern,
1374:                bits<4> m5or = 0>
1375:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `InstVRIl`, `InstVRRa`, `InstVRRb`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `InstVRIl`, `InstVRRa`, `InstVRRb` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1376-1430 / 第 1376-1430 行
```tablegen
1376:   field bits<48> Inst;
1377: 
1378:   bits<5> V1;
1379:   bits<5> V2;
1380:   bits<5> V3;
1381:   bits<4> M4;
1382:   bits<4> M5;
1383: 
1384:   let Inst{47-40} = op{15-8};
1385:   let Inst{39-36} = V1{3-0};
1386:   let Inst{35-32} = V2{3-0};
1387:   let Inst{31-28} = V3{3-0};
1388:   let Inst{27-24} = 0;
1389:   let Inst{23}    = !if (!eq (m5or{3}, 1), 1, M5{3});
1390:   let Inst{22}    = !if (!eq (m5or{2}, 1), 1, M5{2});
1391:   let Inst{21}    = !if (!eq (m5or{1}, 1), 1, M5{1});
1392:   let Inst{20}    = !if (!eq (m5or{0}, 1), 1, M5{0});
1393:   let Inst{19-16} = 0;
1394:   let Inst{15-12} = M4;
1395:   let Inst{11}    = V1{4};
1396:   let Inst{10}    = V2{4};
1397:   let Inst{9}     = V3{4};
1398:   let Inst{8}     = 0;
1399:   let Inst{7-0}   = op{7-0};
1400: }
1401: 
1402: class InstVRRc<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1403:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1404:   field bits<48> Inst;
1405: 
1406:   bits<5> V1;
1407:   bits<5> V2;
1408:   bits<5> V3;
1409:   bits<4> M4;
1410:   bits<4> M5;
1411:   bits<4> M6;
1412: 
1413:   let Inst{47-40} = op{15-8};
1414:   let Inst{39-36} = V1{3-0};
1415:   let Inst{35-32} = V2{3-0};
1416:   let Inst{31-28} = V3{3-0};
1417:   let Inst{27-24} = 0;
1418:   let Inst{23-20} = M6;
1419:   let Inst{19-16} = M5;
1420:   let Inst{15-12} = M4;
1421:   let Inst{11}    = V1{4};
1422:   let Inst{10}    = V2{4};
1423:   let Inst{9}     = V3{4};
1424:   let Inst{8}     = 0;
1425:   let Inst{7-0}   = op{7-0};
1426: }
1427: 
1428: // Depending on the instruction mnemonic, certain bits may be or-ed into
1429: // the M6 value provided as explicit operand.  These are passed as m6or.
1430: class InstVRRd<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern,
```
- **EN**: This block declares or refines TableGen records such as `InstVRRc`, `InstVRRd`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `InstVRRc`, `InstVRRd` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1431-1485 / 第 1431-1485 行
```tablegen
1431:                bits<4> m6or = 0>
1432:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1433:   field bits<48> Inst;
1434: 
1435:   bits<5> V1;
1436:   bits<5> V2;
1437:   bits<5> V3;
1438:   bits<5> V4;
1439:   bits<4> M5;
1440:   bits<4> M6;
1441: 
1442:   let Inst{47-40} = op{15-8};
1443:   let Inst{39-36} = V1{3-0};
1444:   let Inst{35-32} = V2{3-0};
1445:   let Inst{31-28} = V3{3-0};
1446:   let Inst{27-24} = M5;
1447:   let Inst{23}    = !if (!eq (m6or{3}, 1), 1, M6{3});
1448:   let Inst{22}    = !if (!eq (m6or{2}, 1), 1, M6{2});
1449:   let Inst{21}    = !if (!eq (m6or{1}, 1), 1, M6{1});
1450:   let Inst{20}    = !if (!eq (m6or{0}, 1), 1, M6{0});
1451:   let Inst{19-16} = 0;
1452:   let Inst{15-12} = V4{3-0};
1453:   let Inst{11}    = V1{4};
1454:   let Inst{10}    = V2{4};
1455:   let Inst{9}     = V3{4};
1456:   let Inst{8}     = V4{4};
1457:   let Inst{7-0}   = op{7-0};
1458: }
1459: 
1460: class InstVRRe<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1461:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1462:   field bits<48> Inst;
1463: 
1464:   bits<5> V1;
1465:   bits<5> V2;
1466:   bits<5> V3;
1467:   bits<5> V4;
1468:   bits<4> M5;
1469:   bits<4> M6;
1470: 
1471:   let Inst{47-40} = op{15-8};
1472:   let Inst{39-36} = V1{3-0};
1473:   let Inst{35-32} = V2{3-0};
1474:   let Inst{31-28} = V3{3-0};
1475:   let Inst{27-24} = M6;
1476:   let Inst{23-20} = 0;
1477:   let Inst{19-16} = M5;
1478:   let Inst{15-12} = V4{3-0};
1479:   let Inst{11}    = V1{4};
1480:   let Inst{10}    = V2{4};
1481:   let Inst{9}     = V3{4};
1482:   let Inst{8}     = V4{4};
1483:   let Inst{7-0}   = op{7-0};
1484: }
1485: 
```
- **EN**: This block declares or refines TableGen records such as `InstVRRe`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `InstVRRe` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1486-1540 / 第 1486-1540 行
```tablegen
1486: class InstVRRf<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1487:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1488:   field bits<48> Inst;
1489: 
1490:   bits<5> V1;
1491:   bits<4> R2;
1492:   bits<4> R3;
1493: 
1494:   let Inst{47-40} = op{15-8};
1495:   let Inst{39-36} = V1{3-0};
1496:   let Inst{35-32} = R2;
1497:   let Inst{31-28} = R3;
1498:   let Inst{27-12} = 0;
1499:   let Inst{11}    = V1{4};
1500:   let Inst{10-8}  = 0;
1501:   let Inst{7-0}   = op{7-0};
1502: }
1503: 
1504: class InstVRRg<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1505:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1506:   field bits<48> Inst;
1507: 
1508:   bits<5> V1;
1509:   bits<16> I2;
1510: 
1511:   let Inst{47-40} = op{15-8};
1512:   let Inst{39-36} = 0;
1513:   let Inst{35-32} = V1{3-0};
1514:   let Inst{31-28} = 0;
1515:   let Inst{27-12} = I2;
1516:   let Inst{11}    = 0;
1517:   let Inst{10}    = V1{4};
1518:   let Inst{9-8}   = 0;
1519:   let Inst{7-0}   = op{7-0};
1520: }
1521: 
1522: class InstVRRh<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1523:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1524:   field bits<48> Inst;
1525: 
1526:   bits<5> V1;
1527:   bits<5> V2;
1528:   bits<4> M3;
1529: 
1530:   let Inst{47-40} = op{15-8};
1531:   let Inst{39-36} = 0;
1532:   let Inst{35-32} = V1{3-0};
1533:   let Inst{31-28} = V2{3-0};
1534:   let Inst{27-24} = 0;
1535:   let Inst{23-20} = M3;
1536:   let Inst{19-12} = 0;
1537:   let Inst{11}    = 0;
1538:   let Inst{10}    = V1{4};
1539:   let Inst{9}     = V2{4};
1540:   let Inst{8}     = 0;
```
- **EN**: This block declares or refines TableGen records such as `InstVRRf`, `InstVRRg`, `InstVRRh`.
- **CN**: 该代码块声明或细化了 `InstVRRf`, `InstVRRg`, `InstVRRh` 等 TableGen 记录。

### Lines 1541-1595 / 第 1541-1595 行
```tablegen
1541:   let Inst{7-0}   = op{7-0};
1542: }
1543: 
1544: class InstVRRi<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1545:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1546:   field bits<48> Inst;
1547: 
1548:   bits<4> R1;
1549:   bits<5> V2;
1550:   bits<4> M3;
1551:   bits<4> M4;
1552: 
1553:   let Inst{47-40} = op{15-8};
1554:   let Inst{39-36} = R1;
1555:   let Inst{35-32} = V2{3-0};
1556:   let Inst{31-24} = 0;
1557:   let Inst{23-20} = M3;
1558:   let Inst{19-16} = M4;
1559:   let Inst{15-12} = 0;
1560:   let Inst{11}    = 0;
1561:   let Inst{10}    = V2{4};
1562:   let Inst{9-8}   = 0;
1563:   let Inst{7-0}   = op{7-0};
1564: }
1565: 
1566: class InstVRRj<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1567:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1568:   field bits<48> Inst;
1569: 
1570:   bits<5> V1;
1571:   bits<5> V2;
1572:   bits<5> V3;
1573:   bits<4> M4;
1574: 
1575:   let Inst{47-40} = op{15-8};
1576:   let Inst{39-36} = V1{3-0};
1577:   let Inst{35-32} = V2{3-0};
1578:   let Inst{31-28} = V3{3-0};
1579:   let Inst{27-24} = 0;
1580:   let Inst{23-20} = M4;
1581:   let Inst{19-16} = 0;
1582:   let Inst{15-12} = 0;
1583:   let Inst{11}    = V1{4};
1584:   let Inst{10}    = V2{4};
1585:   let Inst{9}     = V3{4};
1586:   let Inst{8}     = 0;
1587:   let Inst{7-0}   = op{7-0};
1588: }
1589: 
1590: class InstVRRk<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1591:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1592:   field bits<48> Inst;
1593: 
1594:   bits<5> V1;
1595:   bits<5> V2;
```
- **EN**: This block declares or refines TableGen records such as `InstVRRi`, `InstVRRj`, `InstVRRk`.
- **CN**: 该代码块声明或细化了 `InstVRRi`, `InstVRRj`, `InstVRRk` 等 TableGen 记录。

### Lines 1596-1650 / 第 1596-1650 行
```tablegen
1596:   bits<4> M3;
1597: 
1598:   let Inst{47-40} = op{15-8};
1599:   let Inst{39-36} = V1{3-0};
1600:   let Inst{35-32} = V2{3-0};
1601:   let Inst{31-28} = 0;
1602:   let Inst{27-24} = 0;
1603:   let Inst{23-20} = M3;
1604:   let Inst{19-16} = 0;
1605:   let Inst{15-12} = 0;
1606:   let Inst{11}    = V1{4};
1607:   let Inst{10}    = V2{4};
1608:   let Inst{9}     = 0;
1609:   let Inst{8}     = 0;
1610:   let Inst{7-0}   = op{7-0};
1611: }
1612: 
1613: class InstVRSa<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1614:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1615:   field bits<48> Inst;
1616: 
1617:   bits<5> V1;
1618:   bits<4> B2;
1619:   bits<12> D2;
1620:   bits<5> V3;
1621:   bits<4> M4;
1622: 
1623:   let Inst{47-40} = op{15-8};
1624:   let Inst{39-36} = V1{3-0};
1625:   let Inst{35-32} = V3{3-0};
1626:   let Inst{31-28} = B2;
1627:   let Inst{27-16} = D2;
1628:   let Inst{15-12} = M4;
1629:   let Inst{11}    = V1{4};
1630:   let Inst{10}    = V3{4};
1631:   let Inst{9-8}   = 0;
1632:   let Inst{7-0}   = op{7-0};
1633: }
1634: 
1635: class InstVRSb<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1636:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1637:   field bits<48> Inst;
1638: 
1639:   bits<5> V1;
1640:   bits<4> B2;
1641:   bits<12> D2;
1642:   bits<4> R3;
1643:   bits<4> M4;
1644: 
1645:   let Inst{47-40} = op{15-8};
1646:   let Inst{39-36} = V1{3-0};
1647:   let Inst{35-32} = R3;
1648:   let Inst{31-28} = B2;
1649:   let Inst{27-16} = D2;
1650:   let Inst{15-12} = M4;
```
- **EN**: This block declares or refines TableGen records such as `InstVRSa`, `InstVRSb`.
- **CN**: 该代码块声明或细化了 `InstVRSa`, `InstVRSb` 等 TableGen 记录。

### Lines 1651-1705 / 第 1651-1705 行
```tablegen
1651:   let Inst{11}    = V1{4};
1652:   let Inst{10-8}  = 0;
1653:   let Inst{7-0}   = op{7-0};
1654: }
1655: 
1656: class InstVRSc<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1657:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1658:   field bits<48> Inst;
1659: 
1660:   bits<4> R1;
1661:   bits<4> B2;
1662:   bits<12> D2;
1663:   bits<5> V3;
1664:   bits<4> M4;
1665: 
1666:   let Inst{47-40} = op{15-8};
1667:   let Inst{39-36} = R1;
1668:   let Inst{35-32} = V3{3-0};
1669:   let Inst{31-28} = B2;
1670:   let Inst{27-16} = D2;
1671:   let Inst{15-12} = M4;
1672:   let Inst{11}    = 0;
1673:   let Inst{10}    = V3{4};
1674:   let Inst{9-8}   = 0;
1675:   let Inst{7-0}   = op{7-0};
1676: }
1677: 
1678: class InstVRSd<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1679:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1680:   field bits<48> Inst;
1681: 
1682:   bits<5> V1;
1683:   bits<4> B2;
1684:   bits<12> D2;
1685:   bits<4> R3;
1686: 
1687:   let Inst{47-40} = op{15-8};
1688:   let Inst{39-36} = 0;
1689:   let Inst{35-32} = R3;
1690:   let Inst{31-28} = B2;
1691:   let Inst{27-16} = D2;
1692:   let Inst{15-12} = V1{3-0};
1693:   let Inst{11-9}  = 0;
1694:   let Inst{8}     = V1{4};
1695:   let Inst{7-0}   = op{7-0};
1696: }
1697: 
1698: class InstVRV<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1699:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1700:   field bits<48> Inst;
1701: 
1702:   bits<5> V1;
1703:   bits<5> V2;
1704:   bits<4> B2;
1705:   bits<12> D2;
```
- **EN**: This block declares or refines TableGen records such as `InstVRSc`, `InstVRSd`, `InstVRV`.
- **CN**: 该代码块声明或细化了 `InstVRSc`, `InstVRSd`, `InstVRV` 等 TableGen 记录。

### Lines 1706-1760 / 第 1706-1760 行
```tablegen
1706:   bits<4> M3;
1707: 
1708:   let Inst{47-40} = op{15-8};
1709:   let Inst{39-36} = V1{3-0};
1710:   let Inst{35-32} = V2{3-0};
1711:   let Inst{31-28} = B2;
1712:   let Inst{27-16} = D2;
1713:   let Inst{15-12} = M3;
1714:   let Inst{11}    = V1{4};
1715:   let Inst{10}    = V2{4};
1716:   let Inst{9-8}   = 0;
1717:   let Inst{7-0}   = op{7-0};
1718: }
1719: 
1720: class InstVRX<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1721:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1722:   field bits<48> Inst;
1723: 
1724:   bits<5> V1;
1725:   bits<4> X2;
1726:   bits<4> B2;
1727:   bits<12> D2;
1728:   bits<4> M3;
1729: 
1730:   let Inst{47-40} = op{15-8};
1731:   let Inst{39-36} = V1{3-0};
1732:   let Inst{35-32} = X2;
1733:   let Inst{31-28} = B2;
1734:   let Inst{27-16} = D2;
1735:   let Inst{15-12} = M3;
1736:   let Inst{11}    = V1{4};
1737:   let Inst{10-8}  = 0;
1738:   let Inst{7-0}   = op{7-0};
1739: }
1740: 
1741: class InstVSI<bits<16> op, dag outs, dag ins, string asmstr, list<dag> pattern>
1742:   : InstSystemZ<6, outs, ins, asmstr, pattern> {
1743:   field bits<48> Inst;
1744: 
1745:   bits<5> V1;
1746:   bits<4> B2;
1747:   bits<12> D2;
1748:   bits<8> I3;
1749: 
1750:   let Inst{47-40} = op{15-8};
1751:   let Inst{39-32} = I3;
1752:   let Inst{31-28} = B2;
1753:   let Inst{27-16} = D2;
1754:   let Inst{15-12} = V1{3-0};
1755:   let Inst{11-9}  = 0;
1756:   let Inst{8}     = V1{4};
1757:   let Inst{7-0}   = op{7-0};
1758: }
1759: 
1760: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `InstVRX`, `InstVSI`.
- **CN**: 该代码块声明或细化了 `InstVRX`, `InstVSI` 等 TableGen 记录。

### Lines 1761-1815 / 第 1761-1815 行
```tablegen
1761: // Instruction classes for .insn directives
1762: //===----------------------------------------------------------------------===//
1763: 
1764: class DirectiveInsnE<dag outs, dag ins, string asmstr, list<dag> pattern>
1765:   : InstE<0, outs, ins, asmstr, pattern> {
1766:   bits<16> enc;
1767: 
1768:   let Inst = enc;
1769: }
1770: 
1771: class DirectiveInsnRI<dag outs, dag ins, string asmstr, list<dag> pattern>
1772:   : InstRIa<0, outs, ins, asmstr, pattern> {
1773:   bits<32> enc;
1774: 
1775:   let Inst{31-24} = enc{31-24};
1776:   let Inst{19-16} = enc{19-16};
1777: }
1778: 
1779: class DirectiveInsnRIE<dag outs, dag ins, string asmstr, list<dag> pattern>
1780:   : InstRIEd<0, outs, ins, asmstr, pattern> {
1781:   bits<48> enc;
1782: 
1783:   let Inst{47-40} = enc{47-40};
1784:   let Inst{7-0}   = enc{7-0};
1785: }
1786: 
1787: class DirectiveInsnRIL<dag outs, dag ins, string asmstr, list<dag> pattern>
1788:   : InstRILa<0, outs, ins, asmstr, pattern> {
1789:   bits<48> enc;
1790:   string type;
1791: 
1792:   let Inst{47-40} = enc{47-40};
1793:   let Inst{35-32} = enc{35-32};
1794: }
1795: 
1796: class DirectiveInsnRIS<dag outs, dag ins, string asmstr, list<dag> pattern>
1797:   : InstRIS<0, outs, ins, asmstr, pattern> {
1798:   bits<48> enc;
1799: 
1800:   let Inst{47-40} = enc{47-40};
1801:   let Inst{7-0}   = enc{7-0};
1802: }
1803: 
1804: class DirectiveInsnRR<dag outs, dag ins, string asmstr, list<dag> pattern>
1805:   : InstRR<0, outs, ins, asmstr, pattern> {
1806:   bits<16> enc;
1807: 
1808:   let Inst{15-8} = enc{15-8};
1809: }
1810: 
1811: class DirectiveInsnRRE<dag outs, dag ins, string asmstr, list<dag> pattern>
1812:   : InstRRE<0, outs, ins, asmstr, pattern> {
1813:   bits<32> enc;
1814: 
1815:   let Inst{31-16} = enc{31-16};
```
- **EN**: This block declares or refines TableGen records such as `DirectiveInsnE`, `DirectiveInsnRI`, `DirectiveInsnRIE`, `DirectiveInsnRIL`, `DirectiveInsnRIS`, `DirectiveInsnRR`.
- **CN**: 该代码块声明或细化了 `DirectiveInsnE`, `DirectiveInsnRI`, `DirectiveInsnRIE`, `DirectiveInsnRIL`, `DirectiveInsnRIS`, `DirectiveInsnRR` 等 TableGen 记录。

### Lines 1816-1870 / 第 1816-1870 行
```tablegen
1816: }
1817: 
1818: class DirectiveInsnRRF<dag outs, dag ins, string asmstr, list<dag> pattern>
1819:   : InstRRFa<0, outs, ins, asmstr, pattern> {
1820:   bits<32> enc;
1821: 
1822:   let Inst{31-16} = enc{31-16};
1823: }
1824: 
1825: class DirectiveInsnRRS<dag outs, dag ins, string asmstr, list<dag> pattern>
1826:   : InstRRS<0, outs, ins, asmstr, pattern> {
1827:   bits<48> enc;
1828: 
1829:   let Inst{47-40} = enc{47-40};
1830:   let Inst{7-0}   = enc{7-0};
1831: }
1832: 
1833: class DirectiveInsnRS<dag outs, dag ins, string asmstr, list<dag> pattern>
1834:   : InstRSa<0, outs, ins, asmstr, pattern> {
1835:   bits<32> enc;
1836: 
1837:   let Inst{31-24} = enc{31-24};
1838: }
1839: 
1840: class DirectiveInsnRSE<dag outs, dag ins, string asmstr, list<dag> pattern>
1841:   : InstRSEa<6, outs, ins, asmstr, pattern> {
1842:   bits <48> enc;
1843: 
1844:   let Inst{47-40} = enc{47-40};
1845:   let Inst{7-0}   = enc{7-0};
1846: }
1847: 
1848: class DirectiveInsnRSI<dag outs, dag ins, string asmstr, list<dag> pattern>
1849:   : InstRSI<0, outs, ins, asmstr, pattern> {
1850:   bits<32> enc;
1851: 
1852:   let Inst{31-24} = enc{31-24};
1853: }
1854: 
1855: class DirectiveInsnRSY<dag outs, dag ins, string asmstr, list<dag> pattern>
1856:   : InstRSYa<0, outs, ins, asmstr, pattern> {
1857:   bits<48> enc;
1858: 
1859:   let Inst{47-40} = enc{47-40};
1860:   let Inst{7-0}   = enc{7-0};
1861: }
1862: 
1863: class DirectiveInsnRX<dag outs, dag ins, string asmstr, list<dag> pattern>
1864:   : InstRXa<0, outs, ins, asmstr, pattern> {
1865:   bits<32> enc;
1866: 
1867:   let Inst{31-24} = enc{31-24};
1868: }
1869: 
1870: class DirectiveInsnRXE<dag outs, dag ins, string asmstr, list<dag> pattern>
```
- **EN**: This block declares or refines TableGen records such as `DirectiveInsnRRF`, `DirectiveInsnRRS`, `DirectiveInsnRS`, `DirectiveInsnRSE`, `DirectiveInsnRSI`, `DirectiveInsnRSY`.
- **CN**: 该代码块声明或细化了 `DirectiveInsnRRF`, `DirectiveInsnRRS`, `DirectiveInsnRS`, `DirectiveInsnRSE`, `DirectiveInsnRSI`, `DirectiveInsnRSY` 等 TableGen 记录。

### Lines 1871-1925 / 第 1871-1925 行
```tablegen
1871:   : InstRXE<0, outs, ins, asmstr, pattern> {
1872:   bits<48> enc;
1873: 
1874:   let M3 = 0;
1875: 
1876:   let Inst{47-40} = enc{47-40};
1877:   let Inst{7-0}   = enc{7-0};
1878: }
1879: 
1880: class DirectiveInsnRXF<dag outs, dag ins, string asmstr, list<dag> pattern>
1881:   : InstRXF<0, outs, ins, asmstr, pattern> {
1882:   bits<48> enc;
1883: 
1884:   let Inst{47-40} = enc{47-40};
1885:   let Inst{7-0}   = enc{7-0};
1886: }
1887: 
1888: class DirectiveInsnRXY<dag outs, dag ins, string asmstr, list<dag> pattern>
1889:   : InstRXYa<0, outs, ins, asmstr, pattern> {
1890:   bits<48> enc;
1891: 
1892:   let Inst{47-40} = enc{47-40};
1893:   let Inst{7-0}   = enc{7-0};
1894: }
1895: 
1896: class DirectiveInsnS<dag outs, dag ins, string asmstr, list<dag> pattern>
1897:   : InstS<0, outs, ins, asmstr, pattern> {
1898:   bits<32> enc;
1899: 
1900:   let Inst{31-16} = enc{31-16};
1901: }
1902: 
1903: class DirectiveInsnSI<dag outs, dag ins, string asmstr, list<dag> pattern>
1904:   : InstSI<0, outs, ins, asmstr, pattern> {
1905:   bits<32> enc;
1906: 
1907:   let Inst{31-24} = enc{31-24};
1908: }
1909: 
1910: class DirectiveInsnSIY<dag outs, dag ins, string asmstr, list<dag> pattern>
1911:   : InstSIY<0, outs, ins, asmstr, pattern> {
1912:   bits<48> enc;
1913: 
1914:   let Inst{47-40} = enc{47-40};
1915:   let Inst{7-0}   = enc{7-0};
1916: }
1917: 
1918: class DirectiveInsnSIL<dag outs, dag ins, string asmstr, list<dag> pattern>
1919:   : InstSIL<0, outs, ins, asmstr, pattern> {
1920:   bits<48> enc;
1921: 
1922:   let Inst{47-32} = enc{47-32};
1923: }
1924: 
1925: class DirectiveInsnSS<dag outs, dag ins, string asmstr, list<dag> pattern>
```
- **EN**: This block declares or refines TableGen records such as `DirectiveInsnRXF`, `DirectiveInsnRXY`, `DirectiveInsnS`, `DirectiveInsnSI`, `DirectiveInsnSIY`, `DirectiveInsnSIL`.
- **CN**: 该代码块声明或细化了 `DirectiveInsnRXF`, `DirectiveInsnRXY`, `DirectiveInsnS`, `DirectiveInsnSI`, `DirectiveInsnSIY`, `DirectiveInsnSIL` 等 TableGen 记录。

### Lines 1926-1980 / 第 1926-1980 行
```tablegen
1926:   : InstSSd<0, outs, ins, asmstr, pattern> {
1927:   bits<48> enc;
1928: 
1929:   let Inst{47-40} = enc{47-40};
1930: }
1931: 
1932: class DirectiveInsnSSE<dag outs, dag ins, string asmstr, list<dag> pattern>
1933:   : InstSSE<0, outs, ins, asmstr, pattern> {
1934:   bits<48> enc;
1935: 
1936:   let Inst{47-32} = enc{47-32};
1937: }
1938: 
1939: class DirectiveInsnSSF<dag outs, dag ins, string asmstr, list<dag> pattern>
1940:   : InstSSF<0, outs, ins, asmstr, pattern> {
1941:   bits<48> enc;
1942: 
1943:   let Inst{47-40} = enc{47-40};
1944:   let Inst{35-32} = enc{35-32};
1945: }
1946: 
1947: class DirectiveInsnVRI<dag outs, dag ins, string asmstr, list<dag> pattern>
1948:   : InstVRIe<0, outs, ins, asmstr, pattern> {
1949:   bits<48> enc;
1950: 
1951:   let Inst{47-40} = enc{47-40};
1952:   let Inst{7-0}   = enc{7-0};
1953: }
1954: 
1955: class DirectiveInsnVRR<dag outs, dag ins, string asmstr, list<dag> pattern>
1956:   : InstVRRc<0, outs, ins, asmstr, pattern> {
1957:   bits<48> enc;
1958: 
1959:   let Inst{47-40} = enc{47-40};
1960:   let Inst{7-0}   = enc{7-0};
1961: }
1962: 
1963: class DirectiveInsnVRS<dag outs, dag ins, string asmstr, list<dag> pattern>
1964:   : InstVRSc<0, outs, ins, asmstr, pattern> {
1965:   bits<48> enc;
1966: 
1967:   let Inst{47-40} = enc{47-40};
1968:   let Inst{7-0}   = enc{7-0};
1969: }
1970: 
1971: class DirectiveInsnVRV<dag outs, dag ins, string asmstr, list<dag> pattern>
1972:   : InstVRV<0, outs, ins, asmstr, pattern> {
1973:   bits<48> enc;
1974: 
1975:   let Inst{47-40} = enc{47-40};
1976:   let Inst{7-0}   = enc{7-0};
1977: }
1978: 
1979: class DirectiveInsnVRX<dag outs, dag ins, string asmstr, list<dag> pattern>
1980:   : InstVRX<0, outs, ins, asmstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `DirectiveInsnSSE`, `DirectiveInsnSSF`, `DirectiveInsnVRI`, `DirectiveInsnVRR`, `DirectiveInsnVRS`, `DirectiveInsnVRV`.
- **CN**: 该代码块声明或细化了 `DirectiveInsnSSE`, `DirectiveInsnSSF`, `DirectiveInsnVRI`, `DirectiveInsnVRR`, `DirectiveInsnVRS`, `DirectiveInsnVRV` 等 TableGen 记录。

### Lines 1981-2035 / 第 1981-2035 行
```tablegen
1981:   bits<48> enc;
1982: 
1983:   let Inst{47-40} = enc{47-40};
1984:   let Inst{7-0}   = enc{7-0};
1985: }
1986: 
1987: class DirectiveInsnVSI<dag outs, dag ins, string asmstr, list<dag> pattern>
1988:   : InstVSI<0, outs, ins, asmstr, pattern> {
1989:   bits<48> enc;
1990: 
1991:   let Inst{47-40} = enc{47-40};
1992:   let Inst{7-0}   = enc{7-0};
1993: }
1994: 
1995: 
1996: //===----------------------------------------------------------------------===//
1997: // Variants of instructions with condition mask
1998: //===----------------------------------------------------------------------===//
1999: //
2000: // For instructions using a condition mask (e.g. conditional branches,
2001: // compare-and-branch instructions, or conditional move instructions),
2002: // we generally need to create multiple instruction patterns:
2003: //
2004: // - One used for code generation, which encodes the condition mask as an
2005: //   MI operand, but writes out an extended mnemonic for better readability.
2006: // - One pattern for the base form of the instruction with an explicit
2007: //   condition mask (encoded as a plain integer MI operand).
2008: // - Specific patterns for each extended mnemonic, where the condition mask
2009: //   is implied by the pattern name and not otherwise encoded at all.
2010: //
2011: // We need the latter primarily for the assembler and disassembler, since the
2012: // assembler parser is not able to decode part of an instruction mnemonic
2013: // into an operand.  Thus we provide separate patterns for each mnemonic.
2014: //
2015: // Note that in some cases there are two different mnemonics for the same
2016: // condition mask.  In this case we cannot have both instructions available
2017: // to the disassembler at the same time since the encodings are not distinct.
2018: // Therefore the alternate forms are marked isAsmParserOnly.
2019: //
2020: // We don't make one of the two names an alias of the other because
2021: // we need the custom parsing routines to select the correct register class.
2022: //
2023: // This section provides helpers for generating the specific forms.
2024: //
2025: //===----------------------------------------------------------------------===//
2026: 
2027: // A class to describe a variant of an instruction with condition mask.
2028: class CondVariant<bits<4> ccmaskin, string suffixin, bit alternatein,
2029:                   string asmvariantin = ""> {
2030:   // The fixed condition mask to use.
2031:   bits<4> ccmask = ccmaskin;
2032: 
2033:   // The suffix to use for the extended assembler mnemonic.
2034:   string suffix = suffixin;
2035: 
```
- **EN**: This block declares or refines TableGen records such as `DirectiveInsnVSI`, `CondVariant`.
- **CN**: 该代码块声明或细化了 `DirectiveInsnVSI`, `CondVariant` 等 TableGen 记录。

### Lines 2036-2090 / 第 2036-2090 行
```tablegen
2036:   // Whether this is an alternate that needs to be marked isAsmParserOnly.
2037:   bit alternate = alternatein;
2038: 
2039:   // Whether this needs be to restricted to a specific dialect.
2040:   // Valid values are "gnu" and "hlasm", which when passed in
2041:   // will set AsmVariantName.
2042:   string asmvariant = asmvariantin;
2043: }
2044: 
2045: // Condition mask 15 means "always true", which is used to define
2046: // unconditional branches as a variant of conditional branches.
2047: def CondAlways : CondVariant<15, "", 0>;
2048: 
2049: // Condition masks for general instructions that can set all 4 bits.
2050: def CondVariantO   : CondVariant<1,  "o",   0>;
2051: def CondVariantH   : CondVariant<2,  "h",   0>;
2052: def CondVariantP   : CondVariant<2,  "p",   1>;
2053: def CondVariantNLE : CondVariant<3,  "nle", 0, "gnu">;
2054: def CondVariantL   : CondVariant<4,  "l",   0>;
2055: def CondVariantM   : CondVariant<4,  "m",   1>;
2056: def CondVariantNHE : CondVariant<5,  "nhe", 0, "gnu">;
2057: def CondVariantLH  : CondVariant<6,  "lh",  0, "gnu">;
2058: def CondVariantNE  : CondVariant<7,  "ne",  0>;
2059: def CondVariantNZ  : CondVariant<7,  "nz",  1>;
2060: def CondVariantE   : CondVariant<8,  "e",   0>;
2061: def CondVariantZ   : CondVariant<8,  "z",   1>;
2062: def CondVariantNLH : CondVariant<9,  "nlh", 0, "gnu">;
2063: def CondVariantHE  : CondVariant<10, "he",  0, "gnu">;
2064: def CondVariantNL  : CondVariant<11, "nl",  0>;
2065: def CondVariantNM  : CondVariant<11, "nm",  1>;
2066: def CondVariantLE  : CondVariant<12, "le",  0, "gnu">;
2067: def CondVariantNH  : CondVariant<13, "nh",  0>;
2068: def CondVariantNP  : CondVariant<13, "np",  1>;
2069: def CondVariantNO  : CondVariant<14, "no",  0>;
2070: 
2071: // A helper class to look up one of the above by name.
2072: class CV<string name>
2073:   : CondVariant<!cast<CondVariant>("CondVariant"#name).ccmask,
2074:                 !cast<CondVariant>("CondVariant"#name).suffix,
2075:                 !cast<CondVariant>("CondVariant"#name).alternate,
2076:                 !cast<CondVariant>("CondVariant"#name).asmvariant>;
2077: 
2078: // Condition masks for integer instructions (e.g. compare-and-branch).
2079: // This is like the list above, except that condition 3 is not possible
2080: // and that the low bit of the mask is therefore always 0.  This means
2081: // that each condition has two names.  Conditions "o" and "no" are not used.
2082: def IntCondVariantH   : CondVariant<2,  "h",   0>;
2083: def IntCondVariantNLE : CondVariant<2,  "nle", 1, "gnu">;
2084: def IntCondVariantL   : CondVariant<4,  "l",   0>;
2085: def IntCondVariantNHE : CondVariant<4,  "nhe", 1, "gnu">;
2086: def IntCondVariantLH  : CondVariant<6,  "lh",  0, "gnu">;
2087: def IntCondVariantNE  : CondVariant<6,  "ne",  1>;
2088: def IntCondVariantE   : CondVariant<8,  "e",   0>;
2089: def IntCondVariantNLH : CondVariant<8,  "nlh", 1, "gnu">;
2090: def IntCondVariantHE  : CondVariant<10, "he",  0, "gnu">;
```
- **EN**: This block declares or refines TableGen records such as `CondAlways`, `CondVariantO`, `CondVariantH`, `CondVariantP`, `CondVariantNLE`, `CondVariantL`.
- **CN**: 该代码块声明或细化了 `CondAlways`, `CondVariantO`, `CondVariantH`, `CondVariantP`, `CondVariantNLE`, `CondVariantL` 等 TableGen 记录。

### Lines 2091-2145 / 第 2091-2145 行
```tablegen
2091: def IntCondVariantNL  : CondVariant<10, "nl",  1>;
2092: def IntCondVariantLE  : CondVariant<12, "le",  0, "gnu">;
2093: def IntCondVariantNH  : CondVariant<12, "nh",  1>;
2094: 
2095: // A helper class to look up one of the above by name.
2096: class ICV<string name>
2097:   : CondVariant<!cast<CondVariant>("IntCondVariant"#name).ccmask,
2098:                 !cast<CondVariant>("IntCondVariant"#name).suffix,
2099:                 !cast<CondVariant>("IntCondVariant"#name).alternate,
2100:                 !cast<CondVariant>("IntCondVariant"#name).asmvariant>;
2101: 
2102: // Defines a class that makes it easier to define
2103: // a MnemonicAlias when CondVariant's are involved.
2104: multiclass MnemonicCondBranchAlias<CondVariant V, string from, string to,
2105:                                    string asmvariant = V.asmvariant> {
2106:   if !or(!eq(V.asmvariant, ""), !eq(V.asmvariant, asmvariant)) then
2107:     def "" : MnemonicAlias<!subst("#", V.suffix, from),
2108:                            !subst("#", V.suffix, to),
2109:                            asmvariant>;
2110: }
2111: 
2112: //===----------------------------------------------------------------------===//
2113: // Instruction definitions with semantics
2114: //===----------------------------------------------------------------------===//
2115: //
2116: // These classes have the form [Cond]<Category><Format>, where <Format> is one
2117: // of the formats defined above and where <Category> describes the inputs
2118: // and outputs.  "Cond" is used if the instruction is conditional,
2119: // in which case the 4-bit condition-code mask is added as a final operand.
2120: // <Category> can be one of:
2121: //
2122: //   Inherent:
2123: //     One register output operand and no input operands.
2124: //
2125: //   InherentDual:
2126: //     Two register output operands and no input operands.
2127: //
2128: //   StoreInherent:
2129: //     One address operand.  The instruction stores to the address.
2130: //
2131: //   SideEffectInherent:
2132: //     No input or output operands, but causes some side effect.
2133: //
2134: //   Branch:
2135: //     One branch target.  The instruction branches to the target.
2136: //
2137: //   Call:
2138: //     One output operand and one branch target.  The instruction stores
2139: //     the return address to the output operand and branches to the target.
2140: //
2141: //   CmpBranch:
2142: //     Two input operands and one optional branch target.  The instruction
2143: //     compares the two input operands and branches or traps on the result.
2144: //
2145: //   BranchUnary:
```
- **EN**: This block declares or refines TableGen records such as `IntCondVariantNL`, `IntCondVariantLE`, `IntCondVariantNH`, `ICV`, `MnemonicCondBranchAlias`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `IntCondVariantNL`, `IntCondVariantLE`, `IntCondVariantNH`, `ICV`, `MnemonicCondBranchAlias` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2146-2200 / 第 2146-2200 行
```tablegen
2146: //     One register output operand, one register input operand and one branch
2147: //     target.  The instructions stores a modified form of the source register
2148: //     in the destination register and branches on the result.
2149: //
2150: //   BranchBinary:
2151: //     One register output operand, two register input operands and one branch
2152: //     target. The instructions stores a modified form of one of the source
2153: //     registers in the destination register and branches on the result.
2154: //
2155: //   LoadMultiple:
2156: //     One address input operand and two explicit output operands.
2157: //     The instruction loads a range of registers from the address,
2158: //     with the explicit operands giving the first and last register
2159: //     to load.  Other loaded registers are added as implicit definitions.
2160: //
2161: //   StoreMultiple:
2162: //     Two explicit input register operands and an address operand.
2163: //     The instruction stores a range of registers to the address,
2164: //     with the explicit operands giving the first and last register
2165: //     to store.  Other stored registers are added as implicit uses.
2166: //
2167: //   StoreLength:
2168: //     One value operand, one length operand and one address operand.
2169: //     The instruction stores the value operand to the address but
2170: //     doesn't write more than the number of bytes specified by the
2171: //     length operand.
2172: //
2173: //   LoadAddress:
2174: //     One register output operand and one address operand.
2175: //
2176: //   LoadIndexedAddress:
2177: //     One register output operand and one indexed address operand.
2178: //
2179: //   SideEffectAddress:
2180: //     One address operand.  No output operands, but causes some side effect.
2181: //
2182: //   Unary:
2183: //     One register output operand and one input operand.
2184: //
2185: //   Store:
2186: //     One address operand and one other input operand.  The instruction
2187: //     stores to the address.
2188: //
2189: //   SideEffectUnary:
2190: //     One input operand.  No output operands, but causes some side effect.
2191: //
2192: //   Binary:
2193: //     One register output operand and two input operands.
2194: //
2195: //   StoreBinary:
2196: //     One address operand and two other input operands.  The instruction
2197: //     stores to the address.
2198: //
2199: //   SideEffectBinary:
2200: //     Two input operands.  No output operands, but causes some side effect.
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 2201-2255 / 第 2201-2255 行
```tablegen
2201: //
2202: //   Compare:
2203: //     Two input operands and an implicit CC output operand.
2204: //
2205: //   Test:
2206: //     One or two input operands and an implicit CC output operand.  If
2207: //     present, the second input operand is an "address" operand used as
2208: //     a test class mask.
2209: //
2210: //   Ternary:
2211: //     One register output operand and three input operands.
2212: //
2213: //   SideEffectTernary:
2214: //     Three input operands.  No output operands, but causes some side effect.
2215: //
2216: //   Quaternary:
2217: //     One register output operand and four input operands.
2218: //
2219: //   LoadAndOp:
2220: //     One output operand and two input operands, one of which is an address.
2221: //     The instruction both reads from and writes to the address.
2222: //
2223: //   CmpSwap:
2224: //     One output operand and three input operands, one of which is an address.
2225: //     The instruction both reads from and writes to the address.
2226: //
2227: //   RotateSelect:
2228: //     One output operand and five input operands.  The first two operands
2229: //     are registers and the other three are immediates.
2230: //
2231: //   Prefetch:
2232: //     One 4-bit immediate operand and one address operand.  The immediate
2233: //     operand is 1 for a load prefetch and 2 for a store prefetch.
2234: //
2235: //   BranchPreload:
2236: //     One 4-bit immediate operand and two address operands.
2237: //
2238: // The format determines which input operands are tied to output operands,
2239: // and also determines the shape of any address operand.
2240: //
2241: // Multiclasses of the form <Category><Format>Pair define two instructions,
2242: // one with <Category><Format> and one with <Category><Format>Y.  The name
2243: // of the first instruction has no suffix, the name of the second has
2244: // an extra "y".
2245: //
2246: //===----------------------------------------------------------------------===//
2247: 
2248: class InherentRRE<string mnemonic, bits<16> opcode, RegisterOperand cls,
2249:                   SDPatternOperator operator>
2250:   : InstRRE<opcode, (outs cls:$R1), (ins),
2251:             mnemonic#"\t$R1",
2252:             [(set cls:$R1, (operator))]> {
2253:   let R2 = 0;
2254: }
2255: 
```
- **EN**: This block declares or refines TableGen records such as `InherentRRE`.
- **CN**: 该代码块声明或细化了 `InherentRRE` 等 TableGen 记录。

### Lines 2256-2310 / 第 2256-2310 行
```tablegen
2256: class InherentDualRRE<string mnemonic, bits<16> opcode, RegisterOperand cls>
2257:   : InstRRE<opcode, (outs cls:$R1, cls:$R2), (ins),
2258:             mnemonic#"\t$R1, $R2", []>;
2259: 
2260: class InherentVRIa<string mnemonic, bits<16> opcode, bits<16> value>
2261:   : InstVRIa<opcode, (outs VR128:$V1), (ins), mnemonic#"\t$V1", []> {
2262:   let I2 = value;
2263:   let M3 = 0;
2264: }
2265: 
2266: class StoreInherentS<string mnemonic, bits<16> opcode,
2267:                      SDPatternOperator operator, bits<5> bytes>
2268:   : InstS<opcode, (outs), (ins (bdaddr12only $B2, $D2):$BD2),
2269:           mnemonic#"\t$BD2", [(operator bdaddr12only:$BD2)]> {
2270:   let mayStore = 1;
2271:   let AccessBytes = bytes;
2272: }
2273: 
2274: class SideEffectInherentE<string mnemonic, bits<16>opcode>
2275:   : InstE<opcode, (outs), (ins), mnemonic, []>;
2276: 
2277: class SideEffectInherentS<string mnemonic, bits<16> opcode,
2278:                           SDPatternOperator operator>
2279:   : InstS<opcode, (outs), (ins), mnemonic, [(operator)]> {
2280:   let B2 = 0;
2281:   let D2 = 0;
2282: }
2283: 
2284: class SideEffectInherentRRE<string mnemonic, bits<16> opcode>
2285:   : InstRRE<opcode, (outs), (ins), mnemonic, []> {
2286:   let R1 = 0;
2287:   let R2 = 0;
2288: }
2289: 
2290: // Allow an optional TLS marker symbol to generate TLS call relocations.
2291: class CallRI<string mnemonic, bits<12> opcode>
2292:   : InstRIb<opcode, (outs), (ins GR64:$R1, brtarget16tls:$RI2),
2293:             mnemonic#"\t$R1, $RI2", []>;
2294: 
2295: // Allow an optional TLS marker symbol to generate TLS call relocations.
2296: class CallRIL<string mnemonic, bits<12> opcode>
2297:   : InstRILb<opcode, (outs), (ins GR64:$R1, brtarget32tls:$RI2),
2298:              mnemonic#"\t$R1, $RI2", []>;
2299: 
2300: class CallRR<string mnemonic, bits<8> opcode>
2301:   : InstRR<opcode, (outs), (ins GR64:$R1, ADDR64:$R2),
2302:            mnemonic#"\t$R1, $R2", []>;
2303: 
2304: class CallRX<string mnemonic, bits<8> opcode>
2305:   : InstRXa<opcode, (outs), (ins GR64:$R1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
2306:             mnemonic#"\t$R1, $XBD2", []>;
2307: 
2308: class CondBranchRI<string mnemonic, bits<12> opcode,
2309:                    SDPatternOperator operator = null_frag>
2310:   : InstRIc<opcode, (outs), (ins cond4:$valid, cond4:$M1, brtarget16:$RI2),
```
- **EN**: This block declares or refines TableGen records such as `InherentDualRRE`, `InherentVRIa`, `StoreInherentS`, `SideEffectInherentE`, `SideEffectInherentS`, `SideEffectInherentRRE`.
- **CN**: 该代码块声明或细化了 `InherentDualRRE`, `InherentVRIa`, `StoreInherentS`, `SideEffectInherentE`, `SideEffectInherentS`, `SideEffectInherentRRE` 等 TableGen 记录。

### Lines 2311-2365 / 第 2311-2365 行
```tablegen
2311:             !subst("#", "${M1}", mnemonic)#"\t$RI2",
2312:             [(operator cond4:$valid, cond4:$M1, bb:$RI2)]> {
2313:   let CCMaskFirst = 1;
2314: }
2315: 
2316: class AsmCondBranchRI<string mnemonic, bits<12> opcode>
2317:   : InstRIc<opcode, (outs), (ins imm32zx4:$M1, brtarget16:$RI2),
2318:             mnemonic#"\t$M1, $RI2", []>;
2319: 
2320: class NeverCondBranchRI<string mnemonic, bits<12> opcode>
2321:   : InstRIc<opcode, (outs), (ins brtarget16:$RI2),
2322:             mnemonic#"\t$RI2", []> {
2323:   let M1 = 0;
2324: }
2325: 
2326: class FixedCondBranchRI<CondVariant V, string mnemonic, bits<12> opcode,
2327:                         SDPatternOperator operator = null_frag>
2328:   : InstRIc<opcode, (outs), (ins brtarget16:$RI2),
2329:             !subst("#", V.suffix, mnemonic)#"\t$RI2", [(operator bb:$RI2)]> {
2330:   let isAsmParserOnly = V.alternate;
2331:   let AsmVariantName = V.asmvariant;
2332:   let M1 = V.ccmask;
2333: }
2334: 
2335: class CondBranchRIL<string mnemonic, bits<12> opcode>
2336:   : InstRILc<opcode, (outs), (ins cond4:$valid, cond4:$M1, brtarget32:$RI2),
2337:              !subst("#", "${M1}", mnemonic)#"\t$RI2", []> {
2338:   let CCMaskFirst = 1;
2339: }
2340: 
2341: class AsmCondBranchRIL<string mnemonic, bits<12> opcode>
2342:   : InstRILc<opcode, (outs), (ins imm32zx4:$M1, brtarget32:$RI2),
2343:              mnemonic#"\t$M1, $RI2", []>;
2344: 
2345: class NeverCondBranchRIL<string mnemonic, bits<12> opcode>
2346:   : InstRILc<opcode, (outs), (ins brtarget32:$RI2),
2347:              mnemonic#"\t$RI2", []> {
2348:   let M1 = 0;
2349: }
2350: 
2351: class FixedCondBranchRIL<CondVariant V, string mnemonic, bits<12> opcode>
2352:   : InstRILc<opcode, (outs), (ins brtarget32:$RI2),
2353:              !subst("#", V.suffix, mnemonic)#"\t$RI2", []> {
2354:   let isAsmParserOnly = V.alternate;
2355:   let AsmVariantName = V.asmvariant;
2356:   let M1 = V.ccmask;
2357: }
2358: 
2359: class CondBranchRR<string mnemonic, bits<8> opcode>
2360:   : InstRR<opcode, (outs), (ins cond4:$valid, cond4:$R1, GR64:$R2),
2361:            !subst("#", "${R1}", mnemonic)#"\t$R2", []> {
2362:   let CCMaskFirst = 1;
2363: }
2364: 
2365: class AsmCondBranchRR<string mnemonic, bits<8> opcode>
```
- **EN**: This block declares or refines TableGen records such as `AsmCondBranchRI`, `NeverCondBranchRI`, `FixedCondBranchRI`, `CondBranchRIL`, `AsmCondBranchRIL`, `NeverCondBranchRIL`.
- **CN**: 该代码块声明或细化了 `AsmCondBranchRI`, `NeverCondBranchRI`, `FixedCondBranchRI`, `CondBranchRIL`, `AsmCondBranchRIL`, `NeverCondBranchRIL` 等 TableGen 记录。

### Lines 2366-2420 / 第 2366-2420 行
```tablegen
2366:   : InstRR<opcode, (outs), (ins imm32zx4:$R1, GR64:$R2),
2367:            mnemonic#"\t$R1, $R2", []>;
2368: 
2369: multiclass NeverCondBranchRR<string mnemonic, bits<8> opcode> {
2370:   // For the no-op (always false) branch, the target is optional.
2371:   def "" : InstRR<opcode, (outs), (ins GR64:$R2),
2372:                   mnemonic#"\t$R2", []> {
2373:              let R1 = 0;
2374:            }
2375:   def Opt : InstRR<opcode, (outs), (ins), mnemonic, []> {
2376:               let R1 = 0;
2377:               let R2 = 0;
2378:             }
2379: }
2380: 
2381: class FixedCondBranchRR<CondVariant V, string mnemonic, bits<8> opcode,
2382:                       SDPatternOperator operator = null_frag>
2383:   : InstRR<opcode, (outs), (ins ADDR64:$R2),
2384:            !subst("#", V.suffix, mnemonic)#"\t$R2", [(operator ADDR64:$R2)]> {
2385:   let isAsmParserOnly = V.alternate;
2386:   let AsmVariantName = V.asmvariant;
2387:   let R1 = V.ccmask;
2388: }
2389: 
2390: class CondBranchRX<string mnemonic, bits<8> opcode>
2391:   : InstRXb<opcode, (outs),
2392:             (ins cond4:$valid, cond4:$M1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
2393:             !subst("#", "${M1}", mnemonic)#"\t$XBD2", []> {
2394:   let CCMaskFirst = 1;
2395: }
2396: 
2397: class AsmCondBranchRX<string mnemonic, bits<8> opcode>
2398:   : InstRXb<opcode, (outs),
2399:             (ins imm32zx4:$M1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
2400:             mnemonic#"\t$M1, $XBD2", []>;
2401: 
2402: multiclass NeverCondBranchRX<string mnemonic, bits<8> opcode> {
2403:   // For the no-op (always false) branch, the target is optional.
2404:   def "" : InstRXb<opcode, (outs),
2405:                   (ins (bdxaddr12only $B2, $D2, $X2):$XBD2),
2406:                   mnemonic#"\t$XBD2", []> {
2407:              let M1 = 0;
2408:            }
2409:   def Opt : InstRXb<opcode, (outs), (ins), mnemonic, []> {
2410:               let M1 = 0;
2411:               let B2 = 0;
2412:               let D2 = 0;
2413:               let X2 = 0;
2414:             }
2415: }
2416: 
2417: class FixedCondBranchRX<CondVariant V, string mnemonic, bits<8> opcode>
2418:   : InstRXb<opcode, (outs), (ins (bdxaddr12only $B2, $D2, $X2):$XBD2),
2419:             !subst("#", V.suffix, mnemonic)#"\t$XBD2", []> {
2420:   let isAsmParserOnly = V.alternate;
```
- **EN**: This block declares or refines TableGen records such as `NeverCondBranchRR`, `Opt`, `FixedCondBranchRR`, `CondBranchRX`, `AsmCondBranchRX`, `NeverCondBranchRX`.
- **CN**: 该代码块声明或细化了 `NeverCondBranchRR`, `Opt`, `FixedCondBranchRR`, `CondBranchRX`, `AsmCondBranchRX`, `NeverCondBranchRX` 等 TableGen 记录。

### Lines 2421-2475 / 第 2421-2475 行
```tablegen
2421:   let AsmVariantName = V.asmvariant;
2422:   let M1 = V.ccmask;
2423: }
2424: 
2425: class CondBranchRXY<string mnemonic, bits<16> opcode>
2426:   : InstRXYb<opcode, (outs), (ins cond4:$valid, cond4:$M1,
2427:              (bdxaddr20only $B2, $D2, $X2):$XBD2),
2428:              !subst("#", "${M1}", mnemonic)#"\t$XBD2", []> {
2429:   let CCMaskFirst = 1;
2430:   let mayLoad = 1;
2431: }
2432: 
2433: class AsmCondBranchRXY<string mnemonic, bits<16> opcode>
2434:   : InstRXYb<opcode, (outs),
2435:              (ins imm32zx4:$M1, (bdxaddr20only $B2, $D2, $X2):$XBD2),
2436:              mnemonic#"\t$M1, $XBD2", []> {
2437:   let mayLoad = 1;
2438: }
2439: 
2440: class FixedCondBranchRXY<CondVariant V, string mnemonic, bits<16> opcode,
2441:                          SDPatternOperator operator = null_frag>
2442:   : InstRXYb<opcode, (outs), (ins (bdxaddr20only $B2, $D2, $X2):$XBD2),
2443:              !subst("#", V.suffix, mnemonic)#"\t$XBD2",
2444:              [(operator (load bdxaddr20only:$XBD2))]> {
2445:   let isAsmParserOnly = V.alternate;
2446:   let AsmVariantName = V.asmvariant;
2447:   let M1 = V.ccmask;
2448:   let mayLoad = 1;
2449: }
2450: 
2451: class CmpBranchRIEa<string mnemonic, bits<16> opcode,
2452:                     RegisterOperand cls, ImmOpWithPattern imm>
2453:   : InstRIEa<opcode, (outs), (ins cls:$R1, imm:$I2, cond4:$M3),
2454:              mnemonic#"$M3\t$R1, $I2", []>;
2455: 
2456: class AsmCmpBranchRIEa<string mnemonic, bits<16> opcode,
2457:                        RegisterOperand cls, ImmOpWithPattern imm>
2458:   : InstRIEa<opcode, (outs), (ins cls:$R1, imm:$I2, imm32zx4:$M3),
2459:              mnemonic#"\t$R1, $I2, $M3", []>;
2460: 
2461: class FixedCmpBranchRIEa<CondVariant V, string mnemonic, bits<16> opcode,
2462:                           RegisterOperand cls, ImmOpWithPattern imm>
2463:   : InstRIEa<opcode, (outs), (ins cls:$R1, imm:$I2),
2464:              mnemonic#V.suffix#"\t$R1, $I2", []> {
2465:   let isAsmParserOnly = V.alternate;
2466:   let AsmVariantName = V.asmvariant;
2467:   let M3 = V.ccmask;
2468: }
2469: 
2470: multiclass CmpBranchRIEaPair<string mnemonic, bits<16> opcode,
2471:                              RegisterOperand cls, ImmOpWithPattern imm> {
2472:   let isCodeGenOnly = 1 in
2473:     def "" : CmpBranchRIEa<mnemonic, opcode, cls, imm>;
2474:   def Asm : AsmCmpBranchRIEa<mnemonic, opcode, cls, imm>;
2475: }
```
- **EN**: This block declares or refines TableGen records such as `CondBranchRXY`, `AsmCondBranchRXY`, `FixedCondBranchRXY`, `CmpBranchRIEa`, `AsmCmpBranchRIEa`, `FixedCmpBranchRIEa`.
- **CN**: 该代码块声明或细化了 `CondBranchRXY`, `AsmCondBranchRXY`, `FixedCondBranchRXY`, `CmpBranchRIEa`, `AsmCmpBranchRIEa`, `FixedCmpBranchRIEa` 等 TableGen 记录。

### Lines 2476-2530 / 第 2476-2530 行
```tablegen
2476: 
2477: class CmpBranchRIEb<string mnemonic, bits<16> opcode,
2478:                     RegisterOperand cls>
2479:   : InstRIEb<opcode, (outs),
2480:              (ins cls:$R1, cls:$R2, cond4:$M3, brtarget16:$RI4),
2481:              mnemonic#"$M3\t$R1, $R2, $RI4", []>;
2482: 
2483: class AsmCmpBranchRIEb<string mnemonic, bits<16> opcode,
2484:                        RegisterOperand cls>
2485:   : InstRIEb<opcode, (outs),
2486:              (ins cls:$R1, cls:$R2, imm32zx4:$M3, brtarget16:$RI4),
2487:              mnemonic#"\t$R1, $R2, $M3, $RI4", []>;
2488: 
2489: class FixedCmpBranchRIEb<CondVariant V, string mnemonic, bits<16> opcode,
2490:                          RegisterOperand cls>
2491:   : InstRIEb<opcode, (outs), (ins cls:$R1, cls:$R2, brtarget16:$RI4),
2492:              mnemonic#V.suffix#"\t$R1, $R2, $RI4", []> {
2493:   let isAsmParserOnly = V.alternate;
2494:   let AsmVariantName = V.asmvariant;
2495:   let M3 = V.ccmask;
2496: }
2497: 
2498: multiclass CmpBranchRIEbPair<string mnemonic, bits<16> opcode,
2499:                              RegisterOperand cls> {
2500:   let isCodeGenOnly = 1 in
2501:     def "" : CmpBranchRIEb<mnemonic, opcode, cls>;
2502:   def Asm : AsmCmpBranchRIEb<mnemonic, opcode, cls>;
2503: }
2504: 
2505: class CmpBranchRIEc<string mnemonic, bits<16> opcode,
2506:                     RegisterOperand cls, ImmOpWithPattern imm>
2507:   : InstRIEc<opcode, (outs),
2508:              (ins cls:$R1, imm:$I2, cond4:$M3, brtarget16:$RI4),
2509:              mnemonic#"$M3\t$R1, $I2, $RI4", []>;
2510: 
2511: class AsmCmpBranchRIEc<string mnemonic, bits<16> opcode,
2512:                        RegisterOperand cls, ImmOpWithPattern imm>
2513:   : InstRIEc<opcode, (outs),
2514:              (ins cls:$R1, imm:$I2, imm32zx4:$M3, brtarget16:$RI4),
2515:              mnemonic#"\t$R1, $I2, $M3, $RI4", []>;
2516: 
2517: class FixedCmpBranchRIEc<CondVariant V, string mnemonic, bits<16> opcode,
2518:                          RegisterOperand cls, ImmOpWithPattern imm>
2519:   : InstRIEc<opcode, (outs), (ins cls:$R1, imm:$I2, brtarget16:$RI4),
2520:              mnemonic#V.suffix#"\t$R1, $I2, $RI4", []> {
2521:   let isAsmParserOnly = V.alternate;
2522:   let AsmVariantName = V.asmvariant;
2523:   let M3 = V.ccmask;
2524: }
2525: 
2526: multiclass CmpBranchRIEcPair<string mnemonic, bits<16> opcode,
2527:                             RegisterOperand cls, ImmOpWithPattern imm> {
2528:   let isCodeGenOnly = 1 in
2529:     def "" : CmpBranchRIEc<mnemonic, opcode, cls, imm>;
2530:   def Asm : AsmCmpBranchRIEc<mnemonic, opcode, cls, imm>;
```
- **EN**: This block declares or refines TableGen records such as `CmpBranchRIEb`, `AsmCmpBranchRIEb`, `FixedCmpBranchRIEb`, `CmpBranchRIEbPair`, `Asm`, `CmpBranchRIEc`.
- **CN**: 该代码块声明或细化了 `CmpBranchRIEb`, `AsmCmpBranchRIEb`, `FixedCmpBranchRIEb`, `CmpBranchRIEbPair`, `Asm`, `CmpBranchRIEc` 等 TableGen 记录。

### Lines 2531-2585 / 第 2531-2585 行
```tablegen
2531: }
2532: 
2533: class CmpBranchRRFc<string mnemonic, bits<16> opcode,
2534:                     RegisterOperand cls>
2535:   : InstRRFc<opcode, (outs), (ins cls:$R1, cls:$R2, cond4:$M3),
2536:              mnemonic#"$M3\t$R1, $R2", []>;
2537: 
2538: class AsmCmpBranchRRFc<string mnemonic, bits<16> opcode,
2539:                        RegisterOperand cls>
2540:   : InstRRFc<opcode, (outs), (ins cls:$R1, cls:$R2, imm32zx4:$M3),
2541:              mnemonic#"\t$R1, $R2, $M3", []>;
2542: 
2543: multiclass CmpBranchRRFcPair<string mnemonic, bits<16> opcode,
2544:                              RegisterOperand cls> {
2545:   let isCodeGenOnly = 1 in
2546:     def "" : CmpBranchRRFc<mnemonic, opcode, cls>;
2547:   def Asm : AsmCmpBranchRRFc<mnemonic, opcode, cls>;
2548: }
2549: 
2550: class FixedCmpBranchRRFc<CondVariant V, string mnemonic, bits<16> opcode,
2551:                           RegisterOperand cls>
2552:   : InstRRFc<opcode, (outs), (ins cls:$R1, cls:$R2),
2553:              mnemonic#V.suffix#"\t$R1, $R2", []> {
2554:   let isAsmParserOnly = V.alternate;
2555:   let AsmVariantName = V.asmvariant;
2556:   let M3 = V.ccmask;
2557: }
2558: 
2559: class CmpBranchRRS<string mnemonic, bits<16> opcode,
2560:                    RegisterOperand cls>
2561:   : InstRRS<opcode, (outs),
2562:             (ins cls:$R1, cls:$R2, cond4:$M3, (bdaddr12only $B4, $D4):$BD4),
2563:             mnemonic#"$M3\t$R1, $R2, $BD4", []>;
2564: 
2565: class AsmCmpBranchRRS<string mnemonic, bits<16> opcode,
2566:                       RegisterOperand cls>
2567:   : InstRRS<opcode, (outs),
2568:             (ins cls:$R1, cls:$R2, imm32zx4:$M3, (bdaddr12only $B4, $D4):$BD4),
2569:             mnemonic#"\t$R1, $R2, $M3, $BD4", []>;
2570: 
2571: class FixedCmpBranchRRS<CondVariant V, string mnemonic, bits<16> opcode,
2572:                         RegisterOperand cls>
2573:   : InstRRS<opcode, (outs),
2574:             (ins cls:$R1, cls:$R2, (bdaddr12only $B4, $D4):$BD4),
2575:             mnemonic#V.suffix#"\t$R1, $R2, $BD4", []> {
2576:   let isAsmParserOnly = V.alternate;
2577:   let AsmVariantName = V.asmvariant;
2578:   let M3 = V.ccmask;
2579: }
2580: 
2581: multiclass CmpBranchRRSPair<string mnemonic, bits<16> opcode,
2582:                             RegisterOperand cls> {
2583:   let isCodeGenOnly = 1 in
2584:     def "" : CmpBranchRRS<mnemonic, opcode, cls>;
2585:   def Asm : AsmCmpBranchRRS<mnemonic, opcode, cls>;
```
- **EN**: This block declares or refines TableGen records such as `CmpBranchRRFc`, `AsmCmpBranchRRFc`, `CmpBranchRRFcPair`, `Asm`, `FixedCmpBranchRRFc`, `CmpBranchRRS`.
- **CN**: 该代码块声明或细化了 `CmpBranchRRFc`, `AsmCmpBranchRRFc`, `CmpBranchRRFcPair`, `Asm`, `FixedCmpBranchRRFc`, `CmpBranchRRS` 等 TableGen 记录。

### Lines 2586-2640 / 第 2586-2640 行
```tablegen
2586: }
2587: 
2588: class CmpBranchRIS<string mnemonic, bits<16> opcode,
2589:                    RegisterOperand cls, ImmOpWithPattern imm>
2590:   : InstRIS<opcode, (outs),
2591:             (ins cls:$R1, imm:$I2, cond4:$M3, (bdaddr12only $B4, $D4):$BD4),
2592:             mnemonic#"$M3\t$R1, $I2, $BD4", []>;
2593: 
2594: class AsmCmpBranchRIS<string mnemonic, bits<16> opcode,
2595:                       RegisterOperand cls, ImmOpWithPattern imm>
2596:   : InstRIS<opcode, (outs),
2597:             (ins cls:$R1, imm:$I2, imm32zx4:$M3, (bdaddr12only $B4, $D4):$BD4),
2598:             mnemonic#"\t$R1, $I2, $M3, $BD4", []>;
2599: 
2600: class FixedCmpBranchRIS<CondVariant V, string mnemonic, bits<16> opcode,
2601:                         RegisterOperand cls, ImmOpWithPattern imm>
2602:   : InstRIS<opcode, (outs),
2603:             (ins cls:$R1, imm:$I2, (bdaddr12only $B4, $D4):$BD4),
2604:             mnemonic#V.suffix#"\t$R1, $I2, $BD4", []> {
2605:   let isAsmParserOnly = V.alternate;
2606:   let AsmVariantName = V.asmvariant;
2607:   let M3 = V.ccmask;
2608: }
2609: 
2610: multiclass CmpBranchRISPair<string mnemonic, bits<16> opcode,
2611:                             RegisterOperand cls, ImmOpWithPattern imm> {
2612:   let isCodeGenOnly = 1 in
2613:     def "" : CmpBranchRIS<mnemonic, opcode, cls, imm>;
2614:   def Asm : AsmCmpBranchRIS<mnemonic, opcode, cls, imm>;
2615: }
2616: 
2617: class CmpBranchRSYb<string mnemonic, bits<16> opcode,
2618:                     RegisterOperand cls>
2619:   : InstRSYb<opcode, (outs),
2620:              (ins cls:$R1, (bdaddr20only $B2, $D2):$BD2, cond4:$M3),
2621:              mnemonic#"$M3\t$R1, $BD2", []>;
2622: 
2623: class AsmCmpBranchRSYb<string mnemonic, bits<16> opcode,
2624:                        RegisterOperand cls>
2625:   : InstRSYb<opcode, (outs),
2626:              (ins cls:$R1, (bdaddr20only $B2, $D2):$BD2, imm32zx4:$M3),
2627:              mnemonic#"\t$R1, $M3, $BD2", []>;
2628: 
2629: multiclass CmpBranchRSYbPair<string mnemonic, bits<16> opcode,
2630:                              RegisterOperand cls> {
2631:   let isCodeGenOnly = 1 in
2632:     def "" : CmpBranchRSYb<mnemonic, opcode, cls>;
2633:   def Asm : AsmCmpBranchRSYb<mnemonic, opcode, cls>;
2634: }
2635: 
2636: class FixedCmpBranchRSYb<CondVariant V, string mnemonic, bits<16> opcode,
2637:                           RegisterOperand cls>
2638:   : InstRSYb<opcode, (outs), (ins cls:$R1, (bdaddr20only $B2, $D2):$BD2),
2639:              mnemonic#V.suffix#"\t$R1, $BD2", []> {
2640:   let isAsmParserOnly = V.alternate;
```
- **EN**: This block declares or refines TableGen records such as `CmpBranchRIS`, `AsmCmpBranchRIS`, `FixedCmpBranchRIS`, `CmpBranchRISPair`, `Asm`, `CmpBranchRSYb`.
- **CN**: 该代码块声明或细化了 `CmpBranchRIS`, `AsmCmpBranchRIS`, `FixedCmpBranchRIS`, `CmpBranchRISPair`, `Asm`, `CmpBranchRSYb` 等 TableGen 记录。

### Lines 2641-2695 / 第 2641-2695 行
```tablegen
2641:   let AsmVariantName = V.asmvariant;
2642:   let M3 = V.ccmask;
2643: }
2644: 
2645: class BranchUnaryRI<string mnemonic, bits<12> opcode, RegisterOperand cls>
2646:   : InstRIb<opcode, (outs cls:$R1), (ins cls:$R1src, brtarget16:$RI2),
2647:             mnemonic#"\t$R1, $RI2", []> {
2648:   let Constraints = "$R1 = $R1src";
2649: }
2650: 
2651: class BranchUnaryRIL<string mnemonic, bits<12> opcode, RegisterOperand cls>
2652:   : InstRILb<opcode, (outs cls:$R1), (ins cls:$R1src, brtarget32:$RI2),
2653:              mnemonic#"\t$R1, $RI2", []> {
2654:   let Constraints = "$R1 = $R1src";
2655: }
2656: 
2657: class BranchUnaryRR<string mnemonic, bits<8> opcode, RegisterOperand cls>
2658:   : InstRR<opcode, (outs cls:$R1), (ins cls:$R1src, GR64:$R2),
2659:            mnemonic#"\t$R1, $R2", []> {
2660:   let Constraints = "$R1 = $R1src";
2661: }
2662: 
2663: class BranchUnaryRRE<string mnemonic, bits<16> opcode, RegisterOperand cls>
2664:   : InstRRE<opcode, (outs cls:$R1), (ins cls:$R1src, GR64:$R2),
2665:             mnemonic#"\t$R1, $R2", []> {
2666:   let Constraints = "$R1 = $R1src";
2667: }
2668: 
2669: class BranchUnaryRX<string mnemonic, bits<8> opcode, RegisterOperand cls>
2670:   : InstRXa<opcode, (outs cls:$R1),
2671:             (ins cls:$R1src, (bdxaddr12only $B2, $D2, $X2):$XBD2),
2672:             mnemonic#"\t$R1, $XBD2", []> {
2673:   let Constraints = "$R1 = $R1src";
2674: }
2675: 
2676: class BranchUnaryRXY<string mnemonic, bits<16> opcode, RegisterOperand cls>
2677:   : InstRXYa<opcode, (outs cls:$R1),
2678:              (ins cls:$R1src, (bdxaddr20only $B2, $D2, $X2):$XBD2),
2679:              mnemonic#"\t$R1, $XBD2", []> {
2680:   let Constraints = "$R1 = $R1src";
2681: }
2682: 
2683: class BranchBinaryRSI<string mnemonic, bits<8> opcode, RegisterOperand cls>
2684:   : InstRSI<opcode, (outs cls:$R1), (ins cls:$R1src, cls:$R3, brtarget16:$RI2),
2685:             mnemonic#"\t$R1, $R3, $RI2", []> {
2686:   let Constraints = "$R1 = $R1src";
2687: }
2688: 
2689: class BranchBinaryRIEe<string mnemonic, bits<16> opcode, RegisterOperand cls>
2690:   : InstRIEe<opcode, (outs cls:$R1),
2691:              (ins cls:$R1src, cls:$R3, brtarget16:$RI2),
2692:              mnemonic#"\t$R1, $R3, $RI2", []> {
2693:   let Constraints = "$R1 = $R1src";
2694: }
2695: 
```
- **EN**: This block declares or refines TableGen records such as `BranchUnaryRI`, `BranchUnaryRIL`, `BranchUnaryRR`, `BranchUnaryRRE`, `BranchUnaryRX`, `BranchUnaryRXY`.
- **CN**: 该代码块声明或细化了 `BranchUnaryRI`, `BranchUnaryRIL`, `BranchUnaryRR`, `BranchUnaryRRE`, `BranchUnaryRX`, `BranchUnaryRXY` 等 TableGen 记录。

### Lines 2696-2750 / 第 2696-2750 行
```tablegen
2696: class BranchBinaryRS<string mnemonic, bits<8> opcode, RegisterOperand cls>
2697:   : InstRSa<opcode, (outs cls:$R1),
2698:             (ins cls:$R1src, cls:$R3, (bdaddr12only $B2, $D2):$BD2),
2699:             mnemonic#"\t$R1, $R3, $BD2", []> {
2700:   let Constraints = "$R1 = $R1src";
2701: }
2702: 
2703: class BranchBinaryRSY<string mnemonic, bits<16> opcode, RegisterOperand cls>
2704:   : InstRSYa<opcode,
2705:              (outs cls:$R1),
2706:              (ins cls:$R1src, cls:$R3, (bdaddr20only $B2, $D2):$BD2),
2707:              mnemonic#"\t$R1, $R3, $BD2", []> {
2708:   let Constraints = "$R1 = $R1src";
2709: }
2710: 
2711: class LoadMultipleRS<string mnemonic, bits<8> opcode, RegisterOperand cls,
2712:                      AddressingMode mode = bdaddr12only>
2713:   : InstRSa<opcode, (outs cls:$R1, cls:$R3), (ins (mode $B2, $D2):$BD2),
2714:             mnemonic#"\t$R1, $R3, $BD2", []> {
2715:   let mayLoad = 1;
2716: }
2717: 
2718: class LoadMultipleRSY<string mnemonic, bits<16> opcode, RegisterOperand cls,
2719:                       AddressingMode mode = bdaddr20only>
2720:   : InstRSYa<opcode, (outs cls:$R1, cls:$R3), (ins (mode $B2, $D2):$BD2),
2721:              mnemonic#"\t$R1, $R3, $BD2", []> {
2722:   let mayLoad = 1;
2723: }
2724: 
2725: multiclass LoadMultipleRSPair<string mnemonic, bits<8> rsOpcode,
2726:                               bits<16> rsyOpcode, RegisterOperand cls> {
2727:   let DispKey = mnemonic # cls in {
2728:     let DispSize = "12" in
2729:       def "" : LoadMultipleRS<mnemonic, rsOpcode, cls, bdaddr12pair>;
2730:     let DispSize = "20" in
2731:       def Y  : LoadMultipleRSY<mnemonic#"y", rsyOpcode, cls, bdaddr20pair>;
2732:   }
2733: }
2734: 
2735: class LoadMultipleSSe<string mnemonic, bits<8> opcode, RegisterOperand cls>
2736:   : InstSSe<opcode, (outs cls:$R1, cls:$R3),
2737:             (ins (bdaddr12only $B2, $D2):$BD2, (bdaddr12only $B4, $D4):$BD4),
2738:             mnemonic#"\t$R1, $R3, $BD2, $BD4", []> {
2739:   let mayLoad = 1;
2740: }
2741: 
2742: multiclass LoadMultipleVRSaAlign<string mnemonic, bits<16> opcode> {
2743:   let mayLoad = 1 in {
2744:     def Align : InstVRSa<opcode, (outs VR128:$V1, VR128:$V3),
2745:                         (ins (bdaddr12only $B2, $D2):$BD2, imm32zx4:$M4),
2746:                         mnemonic#"\t$V1, $V3, $BD2, $M4", []>;
2747:     let M4 = 0 in
2748:       def "" : InstVRSa<opcode, (outs VR128:$V1, VR128:$V3),
2749:                         (ins (bdaddr12only $B2, $D2):$BD2),
2750:                         mnemonic#"\t$V1, $V3, $BD2", []>;
```
- **EN**: This block declares or refines TableGen records such as `BranchBinaryRS`, `BranchBinaryRSY`, `LoadMultipleRS`, `LoadMultipleRSY`, `LoadMultipleRSPair`, `Y`.
- **CN**: 该代码块声明或细化了 `BranchBinaryRS`, `BranchBinaryRSY`, `LoadMultipleRS`, `LoadMultipleRSY`, `LoadMultipleRSPair`, `Y` 等 TableGen 记录。

### Lines 2751-2805 / 第 2751-2805 行
```tablegen
2751:   }
2752: }
2753: 
2754: class StoreRILPC<string mnemonic, bits<12> opcode, SDPatternOperator operator,
2755:                  RegisterOperand cls>
2756:   : InstRILb<opcode, (outs), (ins cls:$R1, pcrel32:$RI2),
2757:              mnemonic#"\t$R1, $RI2",
2758:              [(operator cls:$R1, pcrel32:$RI2)]> {
2759:   let mayStore = 1;
2760:   // We want PC-relative addresses to be tried ahead of BD and BDX addresses.
2761:   // However, BDXs have two extra operands and are therefore 6 units more
2762:   // complex.
2763:   let AddedComplexity = 7;
2764: }
2765: 
2766: class StoreRX<string mnemonic, bits<8> opcode, SDPatternOperator operator,
2767:               RegisterOperand cls, bits<5> bytes,
2768:               AddressingMode mode = bdxaddr12only>
2769:   : InstRXa<opcode, (outs), (ins cls:$R1, (mode $B2, $D2, $X2):$XBD2),
2770:             mnemonic#"\t$R1, $XBD2",
2771:             [(operator cls:$R1, mode:$XBD2)]> {
2772:   let OpKey = mnemonic#"r"#cls;
2773:   let OpType = "mem";
2774:   let mayStore = 1;
2775:   let AccessBytes = bytes;
2776: }
2777: 
2778: class StoreRXY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
2779:                RegisterOperand cls, bits<5> bytes,
2780:                AddressingMode mode = bdxaddr20only>
2781:   : InstRXYa<opcode, (outs), (ins cls:$R1, (mode $B2, $D2, $X2):$XBD2),
2782:              mnemonic#"\t$R1, $XBD2",
2783:              [(operator cls:$R1, mode:$XBD2)]> {
2784:   let OpKey = mnemonic#"r"#cls;
2785:   let OpType = "mem";
2786:   let mayStore = 1;
2787:   let AccessBytes = bytes;
2788: }
2789: 
2790: multiclass StoreRXPair<string mnemonic, bits<8> rxOpcode, bits<16> rxyOpcode,
2791:                        SDPatternOperator operator, RegisterOperand cls,
2792:                        bits<5> bytes> {
2793:   let DispKey = mnemonic # cls in {
2794:     let DispSize = "12" in
2795:       def "" : StoreRX<mnemonic, rxOpcode, operator, cls, bytes, bdxaddr12pair>;
2796:     let DispSize = "20" in
2797:       def Y  : StoreRXY<mnemonic#"y", rxyOpcode, operator, cls, bytes,
2798:                         bdxaddr20pair>;
2799:   }
2800: }
2801: 
2802: class StoreVRX<string mnemonic, bits<16> opcode, SDPatternOperator operator,
2803:                TypedReg tr, bits<5> bytes, bits<4> type = 0>
2804:   : InstVRX<opcode, (outs),
2805:             (ins tr.op:$V1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
```
- **EN**: This block declares or refines TableGen records such as `StoreRILPC`, `StoreRX`, `StoreRXY`, `StoreRXPair`, `Y`, `StoreVRX`.
- **CN**: 该代码块声明或细化了 `StoreRILPC`, `StoreRX`, `StoreRXY`, `StoreRXPair`, `Y`, `StoreVRX` 等 TableGen 记录。

### Lines 2806-2860 / 第 2806-2860 行
```tablegen
2806:             mnemonic#"\t$V1, $XBD2",
2807:             [(operator (tr.vt tr.op:$V1), bdxaddr12only:$XBD2)]> {
2808:   let M3 = type;
2809:   let mayStore = 1;
2810:   let AccessBytes = bytes;
2811: }
2812: 
2813: class StoreVRXGeneric<string mnemonic, bits<16> opcode>
2814:   : InstVRX<opcode, (outs),
2815:             (ins VR128:$V1, (bdxaddr12only $B2, $D2, $X2):$XBD2, imm32zx4:$M3),
2816:             mnemonic#"\t$V1, $XBD2, $M3", []> {
2817:   let mayStore = 1;
2818: }
2819: 
2820: multiclass StoreVRXAlign<string mnemonic, bits<16> opcode> {
2821:   let mayStore = 1, AccessBytes = 16 in {
2822:     def Align : InstVRX<opcode, (outs),
2823:                         (ins VR128:$V1, (bdxaddr12only $B2, $D2, $X2):$XBD2,
2824:                              imm32zx4:$M3),
2825:                         mnemonic#"\t$V1, $XBD2, $M3", []>;
2826:     let M3 = 0 in
2827:       def "" : InstVRX<opcode, (outs),
2828:                        (ins VR128:$V1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
2829:                        mnemonic#"\t$V1, $XBD2", []>;
2830:   }
2831: }
2832: 
2833: class StoreLengthVRSb<string mnemonic, bits<16> opcode,
2834:                       SDPatternOperator operator, bits<5> bytes>
2835:   : InstVRSb<opcode, (outs),
2836:              (ins VR128:$V1, GR32:$R3, (bdaddr12only $B2, $D2):$BD2),
2837:              mnemonic#"\t$V1, $R3, $BD2",
2838:              [(operator VR128:$V1, GR32:$R3, bdaddr12only:$BD2)]> {
2839:   let M4 = 0;
2840:   let mayStore = 1;
2841:   let AccessBytes = bytes;
2842: }
2843: 
2844: class StoreLengthVRSd<string mnemonic, bits<16> opcode,
2845:                       SDPatternOperator operator, bits<5> bytes>
2846:   : InstVRSd<opcode, (outs),
2847:              (ins VR128:$V1, GR32:$R3, (bdaddr12only $B2, $D2):$BD2),
2848:              mnemonic#"\t$V1, $R3, $BD2",
2849:              [(operator VR128:$V1, GR32:$R3, bdaddr12only:$BD2)]> {
2850:   let mayStore = 1;
2851:   let AccessBytes = bytes;
2852: }
2853: 
2854: class StoreLengthVSI<string mnemonic, bits<16> opcode,
2855:                      SDPatternOperator operator, bits<5> bytes>
2856:   : InstVSI<opcode, (outs),
2857:             (ins VR128:$V1, (bdaddr12only $B2, $D2):$BD2, imm32zx8:$I3),
2858:             mnemonic#"\t$V1, $BD2, $I3",
2859:             [(operator VR128:$V1, imm32zx8:$I3, bdaddr12only:$BD2)]> {
2860:   let mayStore = 1;
```
- **EN**: This block declares or refines TableGen records such as `StoreVRXGeneric`, `StoreVRXAlign`, `Align`, `StoreLengthVRSb`, `StoreLengthVRSd`, `StoreLengthVSI`.
- **CN**: 该代码块声明或细化了 `StoreVRXGeneric`, `StoreVRXAlign`, `Align`, `StoreLengthVRSb`, `StoreLengthVRSd`, `StoreLengthVSI` 等 TableGen 记录。

### Lines 2861-2915 / 第 2861-2915 行
```tablegen
2861:   let AccessBytes = bytes;
2862: }
2863: 
2864: class StoreMultipleRS<string mnemonic, bits<8> opcode, RegisterOperand cls,
2865:                       AddressingMode mode = bdaddr12only>
2866:   : InstRSa<opcode, (outs), (ins cls:$R1, cls:$R3, (mode $B2, $D2):$BD2),
2867:             mnemonic#"\t$R1, $R3, $BD2", []> {
2868:   let mayStore = 1;
2869: }
2870: 
2871: class StoreMultipleRSY<string mnemonic, bits<16> opcode, RegisterOperand cls,
2872:                        AddressingMode mode = bdaddr20only>
2873:   : InstRSYa<opcode, (outs), (ins cls:$R1, cls:$R3, (mode $B2, $D2):$BD2),
2874:              mnemonic#"\t$R1, $R3, $BD2", []> {
2875:   let mayStore = 1;
2876: }
2877: 
2878: multiclass StoreMultipleRSPair<string mnemonic, bits<8> rsOpcode,
2879:                                bits<16> rsyOpcode, RegisterOperand cls> {
2880:   let DispKey = mnemonic # cls in {
2881:     let DispSize = "12" in
2882:       def "" : StoreMultipleRS<mnemonic, rsOpcode, cls, bdaddr12pair>;
2883:     let DispSize = "20" in
2884:       def Y  : StoreMultipleRSY<mnemonic#"y", rsyOpcode, cls, bdaddr20pair>;
2885:   }
2886: }
2887: 
2888: multiclass StoreMultipleVRSaAlign<string mnemonic, bits<16> opcode> {
2889:   let mayStore = 1 in {
2890:     def Align : InstVRSa<opcode, (outs), (ins VR128:$V1, VR128:$V3,
2891:                                               (bdaddr12only $B2, $D2):$BD2,
2892:                                               imm32zx4:$M4),
2893:                          mnemonic#"\t$V1, $V3, $BD2, $M4", []>;
2894:     let M4 = 0 in
2895:       def "" : InstVRSa<opcode, (outs), (ins VR128:$V1, VR128:$V3,
2896:                                              (bdaddr12only $B2, $D2):$BD2),
2897:                         mnemonic#"\t$V1, $V3, $BD2", []>;
2898:   }
2899: }
2900: 
2901: // StoreSI* instructions are used to store an integer to memory, but the
2902: // addresses are more restricted than for normal stores.  If we are in the
2903: // situation of having to force either the address into a register or the
2904: // constant into a register, it's usually better to do the latter.
2905: // We therefore match the address in the same way as a normal store and
2906: // only use the StoreSI* instruction if the matched address is suitable.
2907: class StoreSI<string mnemonic, bits<8> opcode, SDPatternOperator operator,
2908:               ImmOpWithPattern imm>
2909:   : InstSI<opcode, (outs), (ins (mviaddr12pair $B1, $D1):$BD1, imm:$I2),
2910:            mnemonic#"\t$BD1, $I2",
2911:            [(operator imm:$I2, mviaddr12pair:$BD1)]> {
2912:   let mayStore = 1;
2913: }
2914: 
2915: class StoreSIY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
```
- **EN**: This block declares or refines TableGen records such as `StoreMultipleRS`, `StoreMultipleRSY`, `StoreMultipleRSPair`, `Y`, `StoreMultipleVRSaAlign`, `Align`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `StoreMultipleRS`, `StoreMultipleRSY`, `StoreMultipleRSPair`, `Y`, `StoreMultipleVRSaAlign`, `Align` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2916-2970 / 第 2916-2970 行
```tablegen
2916:                ImmOpWithPattern imm>
2917:   : InstSIY<opcode, (outs), (ins (mviaddr20pair $B1, $D1):$BD1, imm:$I2),
2918:             mnemonic#"\t$BD1, $I2",
2919:             [(operator imm:$I2, mviaddr20pair:$BD1)]> {
2920:   let mayStore = 1;
2921: }
2922: 
2923: class StoreSIL<string mnemonic, bits<16> opcode, SDPatternOperator operator,
2924:                ImmOpWithPattern imm>
2925:   : InstSIL<opcode, (outs), (ins (mviaddr12pair $B1, $D1):$BD1, imm:$I2),
2926:             mnemonic#"\t$BD1, $I2",
2927:             [(operator imm:$I2, mviaddr12pair:$BD1)]> {
2928:   let mayStore = 1;
2929: }
2930: 
2931: multiclass StoreSIPair<string mnemonic, bits<8> siOpcode, bits<16> siyOpcode,
2932:                        SDPatternOperator operator, ImmOpWithPattern imm> {
2933:   let DispKey = mnemonic in {
2934:     let DispSize = "12" in
2935:       def "" : StoreSI<mnemonic, siOpcode, operator, imm>;
2936:     let DispSize = "20" in
2937:       def Y  : StoreSIY<mnemonic#"y", siyOpcode, operator, imm>;
2938:   }
2939: }
2940: 
2941: class StoreSSE<string mnemonic, bits<16> opcode>
2942:   : InstSSE<opcode, (outs),
2943:             (ins (bdaddr12only $B1, $D1):$BD1, (bdaddr12only $B2, $D2):$BD2),
2944:             mnemonic#"\t$BD1, $BD2", []> {
2945:   let mayStore = 1;
2946: }
2947: 
2948: class CondStoreRSY<string mnemonic, bits<16> opcode,
2949:                    RegisterOperand cls, bits<5> bytes,
2950:                    AddressingMode mode = bdaddr20only>
2951:   : InstRSYb<opcode, (outs),
2952:              (ins cls:$R1, (mode $B2, $D2):$BD2, cond4:$valid, cond4:$M3),
2953:              mnemonic#"$M3\t$R1, $BD2", []> {
2954:   let mayStore = 1;
2955:   let AccessBytes = bytes;
2956:   let CCMaskLast = 1;
2957: }
2958: 
2959: // Like CondStoreRSY, but used for the raw assembly form.  The condition-code
2960: // mask is the third operand rather than being part of the mnemonic.
2961: class AsmCondStoreRSY<string mnemonic, bits<16> opcode,
2962:                       RegisterOperand cls, bits<5> bytes,
2963:                       AddressingMode mode = bdaddr20only>
2964:   : InstRSYb<opcode, (outs), (ins cls:$R1, (mode $B2, $D2):$BD2, imm32zx4:$M3),
2965:              mnemonic#"\t$R1, $BD2, $M3", []> {
2966:   let mayStore = 1;
2967:   let AccessBytes = bytes;
2968: }
2969: 
2970: // Like CondStoreRSY, but with a fixed CC mask.
```
- **EN**: This block declares or refines TableGen records such as `StoreSIL`, `StoreSIPair`, `Y`, `StoreSSE`, `CondStoreRSY`, `AsmCondStoreRSY`.
- **CN**: 该代码块声明或细化了 `StoreSIL`, `StoreSIPair`, `Y`, `StoreSSE`, `CondStoreRSY`, `AsmCondStoreRSY` 等 TableGen 记录。

### Lines 2971-3025 / 第 2971-3025 行
```tablegen
2971: class FixedCondStoreRSY<CondVariant V, string mnemonic, bits<16> opcode,
2972:                         RegisterOperand cls, bits<5> bytes,
2973:                         AddressingMode mode = bdaddr20only>
2974:   : InstRSYb<opcode, (outs), (ins cls:$R1, (mode $B2, $D2):$BD2),
2975:              mnemonic#V.suffix#"\t$R1, $BD2", []> {
2976:   let mayStore = 1;
2977:   let AccessBytes = bytes;
2978:   let isAsmParserOnly = V.alternate;
2979:   let AsmVariantName = V.asmvariant;
2980:   let M3 = V.ccmask;
2981: }
2982: 
2983: multiclass CondStoreRSYPair<string mnemonic, bits<16> opcode,
2984:                             RegisterOperand cls, bits<5> bytes,
2985:                             AddressingMode mode = bdaddr20only> {
2986:   let isCodeGenOnly = 1 in
2987:     def "" : CondStoreRSY<mnemonic, opcode, cls, bytes, mode>;
2988:   def Asm : AsmCondStoreRSY<mnemonic, opcode, cls, bytes, mode>;
2989: }
2990: 
2991: class SideEffectUnaryI<string mnemonic, bits<8> opcode, ImmOpWithPattern imm>
2992:   : InstI<opcode, (outs), (ins imm:$I1),
2993:           mnemonic#"\t$I1", []>;
2994: 
2995: class SideEffectUnaryRR<string mnemonic, bits<8>opcode, RegisterOperand cls>
2996:   : InstRR<opcode, (outs), (ins cls:$R1),
2997:            mnemonic#"\t$R1", []> {
2998:   let R2 = 0;
2999: }
3000: 
3001: class SideEffectUnaryRRE<string mnemonic, bits<16> opcode, RegisterOperand cls,
3002:                          SDPatternOperator operator>
3003:   : InstRRE<opcode, (outs), (ins cls:$R1),
3004:             mnemonic#"\t$R1", [(operator cls:$R1)]> {
3005:   let R2 = 0;
3006: }
3007: 
3008: class SideEffectUnaryS<string mnemonic, bits<16> opcode,
3009:                        SDPatternOperator operator, bits<5> bytes,
3010:                        AddressingMode mode = bdaddr12only>
3011:   : InstS<opcode, (outs), (ins (mode $B2, $D2):$BD2),
3012:           mnemonic#"\t$BD2", [(operator mode:$BD2)]> {
3013:   let mayLoad = 1;
3014:   let AccessBytes = bytes;
3015: }
3016: 
3017: class SideEffectUnarySIY<string mnemonic, bits<16> opcode,
3018:                          bits<5> bytes,
3019:                          AddressingMode mode = bdaddr20only>
3020:   : InstSIY<opcode, (outs), (ins (mode $B1, $D1):$BD1),
3021:             mnemonic#"\t$BD1", []> {
3022:   let mayLoad = 1;
3023:   let AccessBytes = bytes;
3024:   let I2 = 0;
3025: }
```
- **EN**: This block declares or refines TableGen records such as `FixedCondStoreRSY`, `CondStoreRSYPair`, `Asm`, `SideEffectUnaryI`, `SideEffectUnaryRR`, `SideEffectUnaryRRE`.
- **CN**: 该代码块声明或细化了 `FixedCondStoreRSY`, `CondStoreRSYPair`, `Asm`, `SideEffectUnaryI`, `SideEffectUnaryRR`, `SideEffectUnaryRRE` 等 TableGen 记录。

### Lines 3026-3080 / 第 3026-3080 行
```tablegen
3026: 
3027: class SideEffectAddressS<string mnemonic, bits<16> opcode,
3028:                         SDPatternOperator operator,
3029:                         AddressingMode mode = bdaddr12only>
3030:   : InstS<opcode, (outs), (ins (mode $B2, $D2):$BD2),
3031:           mnemonic#"\t$BD2", [(operator mode:$BD2)]>;
3032: 
3033: class LoadAddressRX<string mnemonic, bits<8> opcode,
3034:                     SDPatternOperator operator, AddressingMode mode>
3035:   : InstRXa<opcode, (outs GR64:$R1), (ins (mode $B2, $D2, $X2):$XBD2),
3036:             mnemonic#"\t$R1, $XBD2",
3037:             [(set GR64:$R1, (operator mode:$XBD2))]>;
3038: 
3039: class LoadAddressRXY<string mnemonic, bits<16> opcode,
3040:                      SDPatternOperator operator, AddressingMode mode>
3041:   : InstRXYa<opcode, (outs GR64:$R1), (ins (mode $B2, $D2, $X2):$XBD2),
3042:              mnemonic#"\t$R1, $XBD2",
3043:              [(set GR64:$R1, (operator mode:$XBD2))]>;
3044: 
3045: multiclass LoadAddressRXPair<string mnemonic, bits<8> rxOpcode,
3046:                              bits<16> rxyOpcode, SDPatternOperator operator> {
3047:   let DispKey = mnemonic in {
3048:     let DispSize = "12" in
3049:       def "" : LoadAddressRX<mnemonic, rxOpcode, operator, laaddr12pair>;
3050:     let DispSize = "20" in
3051:       def Y  : LoadAddressRXY<mnemonic#"y", rxyOpcode, operator, laaddr20pair>;
3052:   }
3053: }
3054: 
3055: class LoadAddressRIL<string mnemonic, bits<12> opcode,
3056:                      SDPatternOperator operator>
3057:   : InstRILb<opcode, (outs GR64:$R1), (ins pcrel32:$RI2),
3058:              mnemonic#"\t$R1, $RI2",
3059:              [(set GR64:$R1, (operator pcrel32:$RI2))]>;
3060: 
3061: multiclass LoadIndexedAddressRXY<string mnemonic, bits<16> opcode,
3062:                                  SDPatternOperator ext,
3063:                                  SDPatternOperator shift = bitconvert> {
3064:   def "" : InstRXYa<opcode, (outs GR64:$R1),
3065:                             (ins (lxaaddr20only $B2, $D2, $X2):$XBD2),
3066:                     mnemonic#"\t$R1, $XBD2", []>;
3067: 
3068:   // Patterns matching LXA with displacement.
3069:   def : Pat<(add ADDR64:$base,
3070:                  (shift (i64 (ext (add ADDR32:$index, disp20imm32:$disp))))),
3071:             (!cast<Instruction>(NAME) ADDR64:$base, imm32:$disp, ADDR32:$index)>;
3072:   def : Pat<(shift (i64 (ext (add ADDR32:$index, disp20imm32:$disp)))),
3073:             (!cast<Instruction>(NAME) zero_reg, imm32:$disp, ADDR32:$index)>;
3074: 
3075:   // Patterns matching LXA without displacement.  These are only beneficial
3076:   // if we have a non-trivial shift.  Also, we need to add some complexity
3077:   // to account for the fact that the regular shift patterns have rather
3078:   // high complexity values due to allowing base + displacement.
3079:   if !ne(shift, bitconvert) then let AddedComplexity = 2 in {
3080:     def : Pat<(add ADDR64:$base, (shift (i64 (ext ADDR32:$index)))),
```
- **EN**: This block declares or refines TableGen records such as `SideEffectAddressS`, `LoadAddressRX`, `LoadAddressRXY`, `LoadAddressRXPair`, `Y`, `LoadAddressRIL`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `SideEffectAddressS`, `LoadAddressRX`, `LoadAddressRXY`, `LoadAddressRXPair`, `Y`, `LoadAddressRIL` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3081-3135 / 第 3081-3135 行
```tablegen
3081:               (!cast<Instruction>(NAME) ADDR64:$base, 0, ADDR32:$index)>;
3082:     def : Pat<(shift (i64 (ext ADDR32:$index))),
3083:               (!cast<Instruction>(NAME) zero_reg, 0, ADDR32:$index)>;
3084:   }
3085: }
3086: 
3087: class UnaryRR<string mnemonic, bits<8> opcode, SDPatternOperator operator,
3088:               RegisterOperand cls1, RegisterOperand cls2>
3089:   : InstRR<opcode, (outs cls1:$R1), (ins cls2:$R2),
3090:            mnemonic#"\t$R1, $R2",
3091:            [(set cls1:$R1, (operator cls2:$R2))]> {
3092:   let OpKey = mnemonic#cls1;
3093:   let OpType = "reg";
3094: }
3095: 
3096: class UnaryRRE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3097:                RegisterOperand cls1, RegisterOperand cls2>
3098:   : InstRRE<opcode, (outs cls1:$R1), (ins cls2:$R2),
3099:             mnemonic#"\t$R1, $R2",
3100:             [(set cls1:$R1, (operator cls2:$R2))]> {
3101:   let OpKey = mnemonic#cls1;
3102:   let OpType = "reg";
3103: }
3104: 
3105: class UnaryTiedRRE<string mnemonic, bits<16> opcode, RegisterOperand cls>
3106:   : InstRRE<opcode, (outs cls:$R1), (ins cls:$R1src),
3107:             mnemonic#"\t$R1", []> {
3108:   let Constraints = "$R1 = $R1src";
3109:   let R2 = 0;
3110: }
3111: 
3112: class UnaryMemRRFc<string mnemonic, bits<16> opcode,
3113:                    RegisterOperand cls1, RegisterOperand cls2>
3114:   : InstRRFc<opcode, (outs cls2:$R2, cls1:$R1), (ins cls1:$R1src),
3115:             mnemonic#"\t$R1, $R2", []> {
3116:   let Constraints = "$R1 = $R1src";
3117:   let M3 = 0;
3118: }
3119: 
3120: class UnaryRI<string mnemonic, bits<12> opcode, SDPatternOperator operator,
3121:               RegisterOperand cls, ImmOpWithPattern imm>
3122:   : InstRIa<opcode, (outs cls:$R1), (ins imm:$I2),
3123:             mnemonic#"\t$R1, $I2",
3124:             [(set cls:$R1, (operator imm:$I2))]>;
3125: 
3126: class UnaryRIL<string mnemonic, bits<12> opcode, SDPatternOperator operator,
3127:                RegisterOperand cls, ImmOpWithPattern imm>
3128:   : InstRILa<opcode, (outs cls:$R1), (ins imm:$I2),
3129:              mnemonic#"\t$R1, $I2",
3130:              [(set cls:$R1, (operator imm:$I2))]>;
3131: 
3132: class UnaryRILPC<string mnemonic, bits<12> opcode, SDPatternOperator operator,
3133:                  RegisterOperand cls>
3134:   : InstRILb<opcode, (outs cls:$R1), (ins pcrel32:$RI2),
3135:              mnemonic#"\t$R1, $RI2",
```
- **EN**: This block declares or refines TableGen records such as `UnaryRR`, `UnaryRRE`, `UnaryTiedRRE`, `UnaryMemRRFc`, `UnaryRI`, `UnaryRIL`.
- **CN**: 该代码块声明或细化了 `UnaryRR`, `UnaryRRE`, `UnaryTiedRRE`, `UnaryMemRRFc`, `UnaryRI`, `UnaryRIL` 等 TableGen 记录。

### Lines 3136-3190 / 第 3136-3190 行
```tablegen
3136:              [(set cls:$R1, (operator pcrel32:$RI2))]> {
3137:   let mayLoad = 1;
3138:   // We want PC-relative addresses to be tried ahead of BD and BDX addresses.
3139:   // However, BDXs have two extra operands and are therefore 6 units more
3140:   // complex.
3141:   let AddedComplexity = 7;
3142: }
3143: 
3144: class CondUnaryRSY<string mnemonic, bits<16> opcode,
3145:                    SDPatternOperator operator, RegisterOperand cls,
3146:                    bits<5> bytes, AddressingMode mode = bdaddr20only>
3147:   : InstRSYb<opcode, (outs cls:$R1),
3148:              (ins cls:$R1src, (mode $B2, $D2):$BD2, cond4:$valid, cond4:$M3),
3149:              mnemonic#"$M3\t$R1, $BD2",
3150:              [(set cls:$R1,
3151:                    (z_select_ccmask (operator bdaddr20only:$BD2), cls:$R1src,
3152:                                     cond4:$valid, cond4:$M3))]> {
3153:   let Constraints = "$R1 = $R1src";
3154:   let mayLoad = 1;
3155:   let AccessBytes = bytes;
3156:   let CCMaskLast = 1;
3157:   let OpKey = mnemonic#"r"#cls;
3158:   let OpType = "mem";
3159:   let MemKey = mnemonic#cls;
3160:   let MemType = "target";
3161: }
3162: 
3163: // Like CondUnaryRSY, but used for the raw assembly form.  The condition-code
3164: // mask is the third operand rather than being part of the mnemonic.
3165: class AsmCondUnaryRSY<string mnemonic, bits<16> opcode,
3166:                       RegisterOperand cls, bits<5> bytes,
3167:                       AddressingMode mode = bdaddr20only>
3168:   : InstRSYb<opcode, (outs cls:$R1),
3169:              (ins cls:$R1src, (mode $B2, $D2):$BD2, imm32zx4:$M3),
3170:              mnemonic#"\t$R1, $BD2, $M3", []> {
3171:   let mayLoad = 1;
3172:   let AccessBytes = bytes;
3173:   let Constraints = "$R1 = $R1src";
3174: }
3175: 
3176: // Like CondUnaryRSY, but with a fixed CC mask.
3177: class FixedCondUnaryRSY<CondVariant V, string mnemonic, bits<16> opcode,
3178:                         RegisterOperand cls, bits<5> bytes,
3179:                         AddressingMode mode = bdaddr20only>
3180:   : InstRSYb<opcode, (outs cls:$R1), (ins cls:$R1src, (mode $B2, $D2):$BD2),
3181:              mnemonic#V.suffix#"\t$R1, $BD2", []> {
3182:   let Constraints = "$R1 = $R1src";
3183:   let mayLoad = 1;
3184:   let AccessBytes = bytes;
3185:   let isAsmParserOnly = V.alternate;
3186:   let AsmVariantName = V.asmvariant;
3187:   let M3 = V.ccmask;
3188: }
3189: 
3190: multiclass CondUnaryRSYPair<string mnemonic, bits<16> opcode,
```
- **EN**: This block declares or refines TableGen records such as `CondUnaryRSY`, `AsmCondUnaryRSY`, `FixedCondUnaryRSY`, `CondUnaryRSYPair`.
- **CN**: 该代码块声明或细化了 `CondUnaryRSY`, `AsmCondUnaryRSY`, `FixedCondUnaryRSY`, `CondUnaryRSYPair` 等 TableGen 记录。

### Lines 3191-3245 / 第 3191-3245 行
```tablegen
3191:                             SDPatternOperator operator,
3192:                             RegisterOperand cls, bits<5> bytes,
3193:                             AddressingMode mode = bdaddr20only> {
3194:   let isCodeGenOnly = 1 in
3195:     def "" : CondUnaryRSY<mnemonic, opcode, operator, cls, bytes, mode>;
3196:   def Asm : AsmCondUnaryRSY<mnemonic, opcode, cls, bytes, mode>;
3197: }
3198: 
3199: class UnaryRX<string mnemonic, bits<8> opcode, SDPatternOperator operator,
3200:               RegisterOperand cls, bits<5> bytes,
3201:               AddressingMode mode = bdxaddr12only>
3202:   : InstRXa<opcode, (outs cls:$R1), (ins (mode $B2, $D2, $X2):$XBD2),
3203:             mnemonic#"\t$R1, $XBD2",
3204:             [(set cls:$R1, (operator mode:$XBD2))]> {
3205:   let OpKey = mnemonic#"r"#cls;
3206:   let OpType = "mem";
3207:   let mayLoad = 1;
3208:   let AccessBytes = bytes;
3209: }
3210: 
3211: class UnaryRXE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3212:                RegisterOperand cls, bits<5> bytes>
3213:   : InstRXE<opcode, (outs cls:$R1), (ins (bdxaddr12only $B2, $D2, $X2):$XBD2),
3214:             mnemonic#"\t$R1, $XBD2",
3215:             [(set cls:$R1, (operator bdxaddr12only:$XBD2))]> {
3216:   let OpKey = mnemonic#"r"#cls;
3217:   let OpType = "mem";
3218:   let mayLoad = 1;
3219:   let AccessBytes = bytes;
3220:   let M3 = 0;
3221: }
3222: 
3223: class UnaryRXY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3224:                RegisterOperand cls, bits<5> bytes,
3225:                AddressingMode mode = bdxaddr20only>
3226:   : InstRXYa<opcode, (outs cls:$R1), (ins (mode $B2, $D2, $X2):$XBD2),
3227:              mnemonic#"\t$R1, $XBD2",
3228:              [(set cls:$R1, (operator mode:$XBD2))]> {
3229:   let OpKey = mnemonic#"r"#cls;
3230:   let OpType = "mem";
3231:   let mayLoad = 1;
3232:   let AccessBytes = bytes;
3233: }
3234: 
3235: multiclass UnaryRXPair<string mnemonic, bits<8> rxOpcode, bits<16> rxyOpcode,
3236:                        SDPatternOperator operator, RegisterOperand cls,
3237:                        bits<5> bytes> {
3238:   let DispKey = mnemonic # cls in {
3239:     let DispSize = "12" in
3240:       def "" : UnaryRX<mnemonic, rxOpcode, operator, cls, bytes, bdxaddr12pair>;
3241:     let DispSize = "20" in
3242:       def Y  : UnaryRXY<mnemonic#"y", rxyOpcode, operator, cls, bytes,
3243:                         bdxaddr20pair>;
3244:   }
3245: }
```
- **EN**: This block declares or refines TableGen records such as `Asm`, `UnaryRX`, `UnaryRXE`, `UnaryRXY`, `UnaryRXPair`, `Y`.
- **CN**: 该代码块声明或细化了 `Asm`, `UnaryRX`, `UnaryRXE`, `UnaryRXY`, `UnaryRXPair`, `Y` 等 TableGen 记录。

### Lines 3246-3300 / 第 3246-3300 行
```tablegen
3246: 
3247: class UnaryVRIa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3248:                 TypedReg tr, ImmOpWithPattern imm, bits<4> type = 0>
3249:   : InstVRIa<opcode, (outs tr.op:$V1), (ins imm:$I2),
3250:              mnemonic#"\t$V1, $I2",
3251:              [(set (tr.vt tr.op:$V1), (operator (i32 timm:$I2)))]> {
3252:   let M3 = type;
3253: }
3254: 
3255: class UnaryVRIaGeneric<string mnemonic, bits<16> opcode, ImmOpWithPattern imm>
3256:   : InstVRIa<opcode, (outs VR128:$V1), (ins imm:$I2, imm32zx4:$M3),
3257:              mnemonic#"\t$V1, $I2, $M3", []>;
3258: 
3259: class UnaryVRRa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3260:                 TypedReg tr1, TypedReg tr2, bits<4> type = 0, bits<4> m4 = 0,
3261:                 bits<4> m5 = 0, string fp_mnemonic = "">
3262:   : InstVRRa<opcode, (outs tr1.op:$V1), (ins tr2.op:$V2),
3263:              mnemonic#"\t$V1, $V2",
3264:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2)))]> {
3265:   let M3 = type;
3266:   let M4 = m4;
3267:   let M5 = m5;
3268:   let OpKey = fp_mnemonic#!subst("VR", "FP", !cast<string>(tr1.op));
3269:   let OpType = "reg";
3270: }
3271: 
3272: class UnaryVRRaGeneric<string mnemonic, bits<16> opcode, bits<4> m4 = 0,
3273:                        bits<4> m5 = 0>
3274:   : InstVRRa<opcode, (outs VR128:$V1), (ins VR128:$V2, imm32zx4:$M3),
3275:              mnemonic#"\t$V1, $V2, $M3", []> {
3276:   let M4 = m4;
3277:   let M5 = m5;
3278: }
3279: 
3280: class UnaryVRRaFloatGeneric<string mnemonic, bits<16> opcode, bits<4> m5 = 0>
3281:   : InstVRRa<opcode, (outs VR128:$V1),
3282:              (ins VR128:$V2, imm32zx4:$M3, imm32zx4:$M4),
3283:              mnemonic#"\t$V1, $V2, $M3, $M4", []> {
3284:   let M5 = m5;
3285: }
3286: 
3287: // Declare a pair of instructions, one which sets CC and one which doesn't.
3288: // The CC-setting form ends with "S" and sets the low bit of M5.
3289: // The form that does not set CC has an extra operand to optionally allow
3290: // specifying arbitrary M5 values in assembler.
3291: multiclass UnaryExtraVRRaSPair<string mnemonic, bits<16> opcode,
3292:                                SDPatternOperator operator,
3293:                                SDPatternOperator operator_cc,
3294:                                TypedReg tr1, TypedReg tr2, bits<4> type> {
3295:   let M3 = type, M4 = 0 in
3296:     def "" : InstVRRa<opcode, (outs tr1.op:$V1),
3297:                       (ins tr2.op:$V2, imm32zx4:$M5),
3298:                       mnemonic#"\t$V1, $V2, $M5", []>;
3299:   def : Pat<(tr1.vt (operator (tr2.vt tr2.op:$V2))),
3300:             (!cast<Instruction>(NAME) tr2.op:$V2, 0)>;
```
- **EN**: This block declares or refines TableGen records such as `UnaryVRIa`, `UnaryVRIaGeneric`, `UnaryVRRa`, `UnaryVRRaGeneric`, `UnaryVRRaFloatGeneric`, `UnaryExtraVRRaSPair`.
- **CN**: 该代码块声明或细化了 `UnaryVRIa`, `UnaryVRIaGeneric`, `UnaryVRRa`, `UnaryVRRaGeneric`, `UnaryVRRaFloatGeneric`, `UnaryExtraVRRaSPair` 等 TableGen 记录。

### Lines 3301-3355 / 第 3301-3355 行
```tablegen
3301:   def : InstAlias<mnemonic#"\t$V1, $V2",
3302:                   (!cast<Instruction>(NAME) tr1.op:$V1, tr2.op:$V2, 0)>;
3303:   let Defs = [CC] in
3304:     def S : UnaryVRRa<mnemonic#"s", opcode, operator_cc, tr1, tr2,
3305:                       type, 0, 1>;
3306: }
3307: 
3308: multiclass UnaryExtraVRRaSPairGeneric<string mnemonic, bits<16> opcode> {
3309:   let M4 = 0, Defs = [CC] in
3310:     def "" : InstVRRa<opcode, (outs VR128:$V1),
3311:                      (ins VR128:$V2, imm32zx4:$M3, imm32zx4:$M5),
3312:                      mnemonic#"\t$V1, $V2, $M3, $M5", []>;
3313:   def : InstAlias<mnemonic#"\t$V1, $V2, $M3",
3314:                   (!cast<Instruction>(NAME) VR128:$V1, VR128:$V2,
3315:                                             imm32zx4:$M3, 0)>;
3316: }
3317: 
3318: class UnaryVRX<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3319:                TypedReg tr, bits<5> bytes, bits<4> type = 0>
3320:   : InstVRX<opcode, (outs tr.op:$V1), (ins (bdxaddr12only $B2, $D2, $X2):$XBD2),
3321:             mnemonic#"\t$V1, $XBD2",
3322:             [(set (tr.vt tr.op:$V1), (operator bdxaddr12only:$XBD2))]> {
3323:   let M3 = type;
3324:   let mayLoad = 1;
3325:   let AccessBytes = bytes;
3326: }
3327: 
3328: class UnaryVRXGeneric<string mnemonic, bits<16> opcode>
3329:   : InstVRX<opcode, (outs VR128:$V1),
3330:             (ins (bdxaddr12only $B2, $D2, $X2):$XBD2, imm32zx4:$M3),
3331:             mnemonic#"\t$V1, $XBD2, $M3", []> {
3332:   let mayLoad = 1;
3333: }
3334: 
3335: multiclass UnaryVRXAlign<string mnemonic, bits<16> opcode> {
3336:   let mayLoad = 1, AccessBytes = 16 in {
3337:     def Align : InstVRX<opcode, (outs VR128:$V1),
3338:                         (ins (bdxaddr12only $B2, $D2, $X2):$XBD2, imm32zx4:$M3),
3339:                         mnemonic#"\t$V1, $XBD2, $M3", []>;
3340:     let M3 = 0 in
3341:       def "" : InstVRX<opcode, (outs VR128:$V1),
3342:                        (ins (bdxaddr12only $B2, $D2, $X2):$XBD2),
3343:                        mnemonic#"\t$V1, $XBD2", []>;
3344:   }
3345: }
3346: 
3347: class SideEffectBinaryRX<string mnemonic, bits<8> opcode,
3348:                          RegisterOperand cls>
3349:   : InstRXa<opcode, (outs), (ins cls:$R1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
3350:             mnemonic#"\t$R1, $XBD2", []>;
3351: 
3352: class SideEffectBinaryRXY<string mnemonic, bits<16> opcode,
3353:                           RegisterOperand cls>
3354:   : InstRXYa<opcode, (outs), (ins cls:$R1, (bdxaddr20only $B2, $D2, $X2):$XBD2),
3355:              mnemonic#"\t$R1, $XBD2", []>;
```
- **EN**: This block declares or refines TableGen records such as `S`, `UnaryExtraVRRaSPairGeneric`, `UnaryVRX`, `UnaryVRXGeneric`, `UnaryVRXAlign`, `Align`.
- **CN**: 该代码块声明或细化了 `S`, `UnaryExtraVRRaSPairGeneric`, `UnaryVRX`, `UnaryVRXGeneric`, `UnaryVRXAlign`, `Align` 等 TableGen 记录。

### Lines 3356-3410 / 第 3356-3410 行
```tablegen
3356: 
3357: class SideEffectBinaryRILPC<string mnemonic, bits<12> opcode,
3358:                             RegisterOperand cls>
3359:   : InstRILb<opcode, (outs), (ins cls:$R1, pcrel32:$RI2),
3360:              mnemonic#"\t$R1, $RI2", []> {
3361:   // We want PC-relative addresses to be tried ahead of BD and BDX addresses.
3362:   // However, BDXs have two extra operands and are therefore 6 units more
3363:   // complex.
3364:   let AddedComplexity = 7;
3365: }
3366: 
3367: class SideEffectBinaryRRE<string mnemonic, bits<16> opcode,
3368:                           RegisterOperand cls1, RegisterOperand cls2>
3369:   : InstRRE<opcode, (outs), (ins cls1:$R1, cls2:$R2),
3370:             mnemonic#"\t$R1, $R2", []>;
3371: 
3372: class SideEffectBinaryRRFa<string mnemonic, bits<16> opcode,
3373:                            RegisterOperand cls1, RegisterOperand cls2>
3374:   : InstRRFa<opcode, (outs), (ins cls1:$R1, cls2:$R2),
3375:              mnemonic#"\t$R1, $R2", []> {
3376:   let R3 = 0;
3377:   let M4 = 0;
3378: }
3379: 
3380: class SideEffectBinaryRRFc<string mnemonic, bits<16> opcode,
3381:                            RegisterOperand cls1, RegisterOperand cls2>
3382:   : InstRRFc<opcode, (outs), (ins cls1:$R1, cls2:$R2),
3383:              mnemonic#"\t$R1, $R2", []> {
3384:   let M3 = 0;
3385: }
3386: 
3387: class SideEffectBinaryIE<string mnemonic, bits<16> opcode,
3388:                          ImmOpWithPattern imm1, ImmOpWithPattern imm2>
3389:   : InstIE<opcode, (outs), (ins imm1:$I1, imm2:$I2),
3390:            mnemonic#"\t$I1, $I2", []>;
3391: 
3392: class SideEffectBinarySI<string mnemonic, bits<8> opcode, Operand imm>
3393:   : InstSI<opcode, (outs), (ins (bdaddr12only $B1, $D1):$BD1, imm:$I2),
3394:            mnemonic#"\t$BD1, $I2", []>;
3395: 
3396: class SideEffectBinarySIL<string mnemonic, bits<16> opcode,
3397:                           SDPatternOperator operator, ImmOpWithPattern imm>
3398:   : InstSIL<opcode, (outs), (ins (bdaddr12only $B1, $D1):$BD1, imm:$I2),
3399:             mnemonic#"\t$BD1, $I2", [(operator bdaddr12only:$BD1, imm:$I2)]>;
3400: 
3401: class SideEffectBinarySSa<string mnemonic, bits<8> opcode>
3402:   : InstSSa<opcode, (outs), (ins (bdladdr12onlylen8 $B1, $D1, $L1):$BDL1,
3403:                                  (bdaddr12only $B2, $D2):$BD2),
3404:             mnemonic#"\t$BDL1, $BD2", []>;
3405: 
3406: class SideEffectBinarySSb<string mnemonic, bits<8> opcode>
3407:   : InstSSb<opcode,
3408:             (outs), (ins (bdladdr12onlylen4 $B1, $D1, $L1):$BDL1,
3409:                          (bdladdr12onlylen4 $B2, $D2, $L2):$BDL2),
3410:             mnemonic#"\t$BDL1, $BDL2", []>;
```
- **EN**: This block declares or refines TableGen records such as `SideEffectBinaryRILPC`, `SideEffectBinaryRRE`, `SideEffectBinaryRRFa`, `SideEffectBinaryRRFc`, `SideEffectBinaryIE`, `SideEffectBinarySI`.
- **CN**: 该代码块声明或细化了 `SideEffectBinaryRILPC`, `SideEffectBinaryRRE`, `SideEffectBinaryRRFa`, `SideEffectBinaryRRFc`, `SideEffectBinaryIE`, `SideEffectBinarySI` 等 TableGen 记录。

### Lines 3411-3465 / 第 3411-3465 行
```tablegen
3411: 
3412: class SideEffectBinarySSf<string mnemonic, bits<8> opcode>
3413:   : InstSSf<opcode, (outs), (ins (bdaddr12only $B1, $D1):$BD1,
3414:                                  (bdladdr12onlylen8 $B2, $D2, $L2):$BDL2),
3415:             mnemonic#"\t$BD1, $BDL2", []>;
3416: 
3417: class SideEffectBinarySSE<string mnemonic, bits<16> opcode>
3418:   : InstSSE<opcode, (outs),
3419:             (ins (bdaddr12only $B1, $D1):$BD1, (bdaddr12only $B2, $D2):$BD2),
3420:             mnemonic#"\t$BD1, $BD2", []>;
3421: 
3422: class SideEffectBinaryMemMemRR<string mnemonic, bits<8> opcode,
3423:                                RegisterOperand cls1, RegisterOperand cls2>
3424:   : InstRR<opcode, (outs cls1:$R1, cls2:$R2), (ins cls1:$R1src, cls2:$R2src),
3425:            mnemonic#"\t$R1, $R2", []> {
3426:     let Constraints = "$R1 = $R1src, $R2 = $R2src";
3427: }
3428: 
3429: class SideEffectBinaryMemRRE<string mnemonic, bits<16> opcode,
3430:                              RegisterOperand cls1, RegisterOperand cls2>
3431:   : InstRRE<opcode, (outs cls2:$R2), (ins cls1:$R1, cls2:$R2src),
3432:             mnemonic#"\t$R1, $R2", []> {
3433:   let Constraints = "$R2 = $R2src";
3434: }
3435: 
3436: class SideEffectBinaryMemMemRRE<string mnemonic, bits<16> opcode,
3437:                                 RegisterOperand cls1, RegisterOperand cls2>
3438:   : InstRRE<opcode, (outs cls1:$R1, cls2:$R2), (ins cls1:$R1src, cls2:$R2src),
3439:             mnemonic#"\t$R1, $R2", []> {
3440:     let Constraints = "$R1 = $R1src, $R2 = $R2src";
3441: }
3442: 
3443: class SideEffectBinaryMemMemRRFc<string mnemonic, bits<16> opcode,
3444:                                  RegisterOperand cls1, RegisterOperand cls2>
3445:   : InstRRFc<opcode, (outs cls1:$R1, cls2:$R2), (ins cls1:$R1src, cls2:$R2src),
3446:              mnemonic#"\t$R1, $R2", []> {
3447:   let Constraints = "$R1 = $R1src, $R2 = $R2src";
3448:   let M3 = 0;
3449: }
3450: 
3451: class BinaryRR<string mnemonic, bits<8> opcode, SDPatternOperator operator,
3452:                RegisterOperand cls1, RegisterOperand cls2>
3453:   : InstRR<opcode, (outs cls1:$R1), (ins cls1:$R1src, cls2:$R2),
3454:            mnemonic#"\t$R1, $R2",
3455:            [(set cls1:$R1, (operator cls1:$R1src, cls2:$R2))]> {
3456:   let OpKey = mnemonic#cls1;
3457:   let OpType = "reg";
3458:   let Constraints = "$R1 = $R1src";
3459: }
3460: 
3461: class BinaryRRE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3462:                 RegisterOperand cls1, RegisterOperand cls2>
3463:   : InstRRE<opcode, (outs cls1:$R1), (ins cls1:$R1src, cls2:$R2),
3464:             mnemonic#"\t$R1, $R2",
3465:             [(set cls1:$R1, (operator cls1:$R1src, cls2:$R2))]> {
```
- **EN**: This block declares or refines TableGen records such as `SideEffectBinarySSf`, `SideEffectBinarySSE`, `SideEffectBinaryMemMemRR`, `SideEffectBinaryMemRRE`, `SideEffectBinaryMemMemRRE`, `SideEffectBinaryMemMemRRFc`.
- **CN**: 该代码块声明或细化了 `SideEffectBinarySSf`, `SideEffectBinarySSE`, `SideEffectBinaryMemMemRR`, `SideEffectBinaryMemRRE`, `SideEffectBinaryMemMemRRE`, `SideEffectBinaryMemMemRRFc` 等 TableGen 记录。

### Lines 3466-3520 / 第 3466-3520 行
```tablegen
3466:   let OpKey = mnemonic#cls1;
3467:   let OpType = "reg";
3468:   let Constraints = "$R1 = $R1src";
3469: }
3470: 
3471: class BinaryRRD<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3472:                 RegisterOperand cls1, RegisterOperand cls2>
3473:   : InstRRD<opcode, (outs cls1:$R1), (ins cls2:$R3, cls2:$R2),
3474:             mnemonic#"\t$R1, $R3, $R2",
3475:             [(set cls1:$R1, (operator cls2:$R3, cls2:$R2))]> {
3476:   let OpKey = mnemonic#cls;
3477:   let OpType = "reg";
3478: }
3479: 
3480: class BinaryRRFa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3481:                  RegisterOperand cls1, RegisterOperand cls2,
3482:                  RegisterOperand cls3>
3483:   : InstRRFa<opcode, (outs cls1:$R1), (ins cls2:$R2, cls3:$R3),
3484:              mnemonic#"\t$R1, $R2, $R3",
3485:              [(set cls1:$R1, (operator cls2:$R2, cls3:$R3))]> {
3486:   let M4 = 0;
3487:   let OpKey = mnemonic#cls1;
3488:   let OpType = "reg";
3489: }
3490: 
3491: 
3492: class UnaryRRFa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3493:                  RegisterOperand cls1, RegisterOperand cls2>
3494:   : InstRRFa<opcode, (outs cls1:$R1), (ins cls2:$R2, cls2:$R3),
3495:              mnemonic#"\t$R1, $R2",
3496:              [(set cls1:$R1, (operator cls2:$R2, cls2:$R3))]> {
3497:   let R3 = R2;
3498:   let M4 = 0;
3499:   let OpKey = mnemonic#cls1;
3500:   let OpType = "reg";
3501: }
3502: 
3503: 
3504: multiclass BinaryRRAndK<string mnemonic, bits<8> opcode1, bits<16> opcode2,
3505:                         SDPatternOperator operator, RegisterOperand cls1,
3506:                         RegisterOperand cls2> {
3507:   let NumOpsKey = mnemonic in {
3508:     let NumOpsValue = "3" in
3509:       def K : BinaryRRFa<mnemonic#"k", opcode2, operator, cls1, cls1, cls2>,
3510:               Requires<[FeatureDistinctOps]>;
3511:     let NumOpsValue = "2" in
3512:       def "" : BinaryRR<mnemonic, opcode1, operator, cls1, cls2>;
3513:   }
3514: }
3515: 
3516: multiclass BinaryRREAndK<string mnemonic, bits<16> opcode1, bits<16> opcode2,
3517:                          SDPatternOperator operator, RegisterOperand cls1,
3518:                          RegisterOperand cls2> {
3519:   let NumOpsKey = mnemonic in {
3520:     let NumOpsValue = "3" in
```
- **EN**: This block declares or refines TableGen records such as `BinaryRRD`, `BinaryRRFa`, `UnaryRRFa`, `BinaryRRAndK`, `K`, `BinaryRREAndK`.
- **CN**: 该代码块声明或细化了 `BinaryRRD`, `BinaryRRFa`, `UnaryRRFa`, `BinaryRRAndK`, `K`, `BinaryRREAndK` 等 TableGen 记录。

### Lines 3521-3575 / 第 3521-3575 行
```tablegen
3521:       def K : BinaryRRFa<mnemonic#"k", opcode2, operator, cls1, cls1, cls2>,
3522:               Requires<[FeatureDistinctOps]>;
3523:     let NumOpsValue = "2" in
3524:       def "" : BinaryRRE<mnemonic, opcode1, operator, cls1, cls2>;
3525:   }
3526: }
3527: 
3528: class BinaryRRFb<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3529:                  RegisterOperand cls1, RegisterOperand cls2,
3530:                  RegisterOperand cls3>
3531:   : InstRRFb<opcode, (outs cls1:$R1), (ins cls2:$R2, cls3:$R3),
3532:              mnemonic#"\t$R1, $R3, $R2",
3533:              [(set cls1:$R1, (operator cls2:$R2, cls3:$R3))]> {
3534:   let M4 = 0;
3535: }
3536: 
3537: class BinaryRRFc<string mnemonic, bits<16> opcode,
3538:                  RegisterOperand cls1, RegisterOperand cls2>
3539:   : InstRRFc<opcode, (outs cls1:$R1), (ins cls2:$R2, imm32zx4:$M3),
3540:              mnemonic#"\t$R1, $R2, $M3", []>;
3541: 
3542: class BinaryMemRRFc<string mnemonic, bits<16> opcode,
3543:                     RegisterOperand cls1, RegisterOperand cls2, ImmOpWithPattern imm>
3544:   : InstRRFc<opcode, (outs cls2:$R2, cls1:$R1), (ins cls1:$R1src, imm:$M3),
3545:             mnemonic#"\t$R1, $R2, $M3", []> {
3546:   let Constraints = "$R1 = $R1src";
3547: }
3548: 
3549: multiclass BinaryMemRRFcOpt<string mnemonic, bits<16> opcode,
3550:                             RegisterOperand cls1, RegisterOperand cls2> {
3551:   def "" : BinaryMemRRFc<mnemonic, opcode, cls1, cls2, imm32zx4>;
3552:   def Opt : UnaryMemRRFc<mnemonic, opcode, cls1, cls2>;
3553: }
3554: 
3555: class BinaryRRFd<string mnemonic, bits<16> opcode, RegisterOperand cls1,
3556:                 RegisterOperand cls2>
3557:   : InstRRFd<opcode, (outs cls1:$R1), (ins cls2:$R2, imm32zx4:$M4),
3558:              mnemonic#"\t$R1, $R2, $M4", []>;
3559: 
3560: class BinaryRRFe<string mnemonic, bits<16> opcode, RegisterOperand cls1,
3561:                 RegisterOperand cls2>
3562:   : InstRRFe<opcode, (outs cls1:$R1), (ins imm32zx4:$M3, cls2:$R2),
3563:              mnemonic#"\t$R1, $M3, $R2", []> {
3564:   let M4 = 0;
3565: }
3566: 
3567: class CondBinaryRRF<string mnemonic, bits<16> opcode, RegisterOperand cls1,
3568:                    RegisterOperand cls2>
3569:   : InstRRFc<opcode, (outs cls1:$R1),
3570:              (ins cls1:$R1src, cls2:$R2, cond4:$valid, cond4:$M3),
3571:              mnemonic#"$M3\t$R1, $R2",
3572:              [(set cls1:$R1, (z_select_ccmask cls2:$R2, cls1:$R1src,
3573:                                               cond4:$valid, cond4:$M3))]> {
3574:   let Constraints = "$R1 = $R1src";
3575:   let CCMaskLast = 1;
```
- **EN**: This block declares or refines TableGen records such as `K`, `BinaryRRFb`, `BinaryRRFc`, `BinaryMemRRFc`, `BinaryMemRRFcOpt`, `Opt`.
- **CN**: 该代码块声明或细化了 `K`, `BinaryRRFb`, `BinaryRRFc`, `BinaryMemRRFc`, `BinaryMemRRFcOpt`, `Opt` 等 TableGen 记录。

### Lines 3576-3630 / 第 3576-3630 行
```tablegen
3576:   let NumOpsKey = !subst("loc", "sel", mnemonic);
3577:   let NumOpsValue = "2";
3578:   let OpKey = mnemonic#cls1;
3579:   let OpType = "reg";
3580: }
3581: 
3582: // Like CondBinaryRRF, but used for the raw assembly form.  The condition-code
3583: // mask is the third operand rather than being part of the mnemonic.
3584: class AsmCondBinaryRRF<string mnemonic, bits<16> opcode, RegisterOperand cls1,
3585:                        RegisterOperand cls2>
3586:   : InstRRFc<opcode, (outs cls1:$R1),
3587:              (ins cls1:$R1src, cls2:$R2, imm32zx4:$M3),
3588:              mnemonic#"\t$R1, $R2, $M3", []> {
3589:   let Constraints = "$R1 = $R1src";
3590: }
3591: 
3592: // Like CondBinaryRRF, but with a fixed CC mask.
3593: class FixedCondBinaryRRF<CondVariant V, string mnemonic, bits<16> opcode,
3594:                          RegisterOperand cls1, RegisterOperand cls2>
3595:   : InstRRFc<opcode, (outs cls1:$R1), (ins cls1:$R1src, cls2:$R2),
3596:              mnemonic#V.suffix#"\t$R1, $R2", []> {
3597:   let Constraints = "$R1 = $R1src";
3598:   let isAsmParserOnly = V.alternate;
3599:   let AsmVariantName = V.asmvariant;
3600:   let M3 = V.ccmask;
3601: }
3602: 
3603: multiclass CondBinaryRRFPair<string mnemonic, bits<16> opcode,
3604:                              RegisterOperand cls1, RegisterOperand cls2> {
3605:   let isCodeGenOnly = 1 in
3606:     def "" : CondBinaryRRF<mnemonic, opcode, cls1, cls2>;
3607:   def Asm : AsmCondBinaryRRF<mnemonic, opcode, cls1, cls2>;
3608: }
3609: 
3610: class CondBinaryRRFa<string mnemonic, bits<16> opcode, RegisterOperand cls1,
3611:                     RegisterOperand cls2, RegisterOperand cls3>
3612:   : InstRRFa<opcode, (outs cls1:$R1),
3613:              (ins cls3:$R3, cls2:$R2, cond4:$valid, cond4:$M4),
3614:              mnemonic#"$M4\t$R1, $R2, $R3",
3615:              [(set cls1:$R1, (z_select_ccmask cls2:$R2, cls3:$R3,
3616:                                               cond4:$valid, cond4:$M4))]> {
3617:   let CCMaskLast = 1;
3618:   let NumOpsKey = mnemonic;
3619:   let NumOpsValue = "3";
3620:   let OpKey = mnemonic#cls1;
3621:   let OpType = "reg";
3622: }
3623: 
3624: // Like CondBinaryRRFa, but used for the raw assembly form.  The condition-code
3625: // mask is the third operand rather than being part of the mnemonic.
3626: class AsmCondBinaryRRFa<string mnemonic, bits<16> opcode, RegisterOperand cls1,
3627:                         RegisterOperand cls2, RegisterOperand cls3>
3628:   : InstRRFa<opcode, (outs cls1:$R1), (ins cls3:$R3, cls2:$R2, imm32zx4:$M4),
3629:              mnemonic#"\t$R1, $R2, $R3, $M4", []>;
3630: 
```
- **EN**: This block declares or refines TableGen records such as `AsmCondBinaryRRF`, `FixedCondBinaryRRF`, `CondBinaryRRFPair`, `Asm`, `CondBinaryRRFa`, `AsmCondBinaryRRFa`.
- **CN**: 该代码块声明或细化了 `AsmCondBinaryRRF`, `FixedCondBinaryRRF`, `CondBinaryRRFPair`, `Asm`, `CondBinaryRRFa`, `AsmCondBinaryRRFa` 等 TableGen 记录。

### Lines 3631-3685 / 第 3631-3685 行
```tablegen
3631: // Like CondBinaryRRFa, but with a fixed CC mask.
3632: class FixedCondBinaryRRFa<CondVariant V, string mnemonic, bits<16> opcode,
3633:                          RegisterOperand cls1, RegisterOperand cls2,
3634:                          RegisterOperand cls3>
3635:   : InstRRFa<opcode, (outs cls1:$R1), (ins cls3:$R3, cls2:$R2),
3636:              mnemonic#V.suffix#"\t$R1, $R2, $R3", []> {
3637:   let isAsmParserOnly = V.alternate;
3638:   let AsmVariantName = V.asmvariant;
3639:   let M4 = V.ccmask;
3640: }
3641: 
3642: multiclass CondBinaryRRFaPair<string mnemonic, bits<16> opcode,
3643:                              RegisterOperand cls1, RegisterOperand cls2,
3644:                              RegisterOperand cls3> {
3645:   let isCodeGenOnly = 1 in
3646:     def "" : CondBinaryRRFa<mnemonic, opcode, cls1, cls2, cls3>;
3647:   def Asm : AsmCondBinaryRRFa<mnemonic, opcode, cls1, cls2, cls3>;
3648: }
3649: 
3650: class BinaryRI<string mnemonic, bits<12> opcode, SDPatternOperator operator,
3651:                RegisterOperand cls, ImmOpWithPattern imm>
3652:   : InstRIa<opcode, (outs cls:$R1), (ins cls:$R1src, imm:$I2),
3653:             mnemonic#"\t$R1, $I2",
3654:             [(set cls:$R1, (operator cls:$R1src, imm:$I2))]> {
3655:   let Constraints = "$R1 = $R1src";
3656: }
3657: 
3658: class BinaryRIE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3659:                 RegisterOperand cls, ImmOpWithPattern imm>
3660:   : InstRIEd<opcode, (outs cls:$R1), (ins cls:$R3, imm:$I2),
3661:              mnemonic#"\t$R1, $R3, $I2",
3662:              [(set cls:$R1, (operator cls:$R3, imm:$I2))]>;
3663: 
3664: multiclass BinaryRIAndK<string mnemonic, bits<12> opcode1, bits<16> opcode2,
3665:                         SDPatternOperator operator, RegisterOperand cls,
3666:                         ImmOpWithPattern imm> {
3667:   let NumOpsKey = mnemonic in {
3668:     let NumOpsValue = "3" in
3669:       def K : BinaryRIE<mnemonic#"k", opcode2, operator, cls, imm>,
3670:               Requires<[FeatureDistinctOps]>;
3671:     let NumOpsValue = "2" in
3672:       def "" : BinaryRI<mnemonic, opcode1, operator, cls, imm>;
3673:   }
3674: }
3675: 
3676: class CondBinaryRIE<string mnemonic, bits<16> opcode, RegisterOperand cls,
3677:                     ImmOpWithPattern imm>
3678:   : InstRIEg<opcode, (outs cls:$R1),
3679:              (ins cls:$R1src, imm:$I2, cond4:$valid, cond4:$M3),
3680:              mnemonic#"$M3\t$R1, $I2",
3681:              [(set cls:$R1, (z_select_ccmask imm:$I2, cls:$R1src,
3682:                                              cond4:$valid, cond4:$M3))]> {
3683:   let Constraints = "$R1 = $R1src";
3684:   let CCMaskLast = 1;
3685: }
```
- **EN**: This block declares or refines TableGen records such as `FixedCondBinaryRRFa`, `CondBinaryRRFaPair`, `Asm`, `BinaryRI`, `BinaryRIE`, `BinaryRIAndK`.
- **CN**: 该代码块声明或细化了 `FixedCondBinaryRRFa`, `CondBinaryRRFaPair`, `Asm`, `BinaryRI`, `BinaryRIE`, `BinaryRIAndK` 等 TableGen 记录。

### Lines 3686-3740 / 第 3686-3740 行
```tablegen
3686: 
3687: // Like CondBinaryRIE, but used for the raw assembly form.  The condition-code
3688: // mask is the third operand rather than being part of the mnemonic.
3689: class AsmCondBinaryRIE<string mnemonic, bits<16> opcode, RegisterOperand cls,
3690:                        ImmOpWithPattern imm>
3691:   : InstRIEg<opcode, (outs cls:$R1),
3692:              (ins cls:$R1src, imm:$I2, imm32zx4:$M3),
3693:              mnemonic#"\t$R1, $I2, $M3", []> {
3694:   let Constraints = "$R1 = $R1src";
3695: }
3696: 
3697: // Like CondBinaryRIE, but with a fixed CC mask.
3698: class FixedCondBinaryRIE<CondVariant V, string mnemonic, bits<16> opcode,
3699:                          RegisterOperand cls, ImmOpWithPattern imm>
3700:   : InstRIEg<opcode, (outs cls:$R1), (ins cls:$R1src, imm:$I2),
3701:              mnemonic#V.suffix#"\t$R1, $I2", []> {
3702:   let Constraints = "$R1 = $R1src";
3703:   let isAsmParserOnly = V.alternate;
3704:   let AsmVariantName = V.asmvariant;
3705:   let M3 = V.ccmask;
3706: }
3707: 
3708: multiclass CondBinaryRIEPair<string mnemonic, bits<16> opcode,
3709:                              RegisterOperand cls, ImmOpWithPattern imm> {
3710:   let isCodeGenOnly = 1 in
3711:     def "" : CondBinaryRIE<mnemonic, opcode, cls, imm>;
3712:   def Asm : AsmCondBinaryRIE<mnemonic, opcode, cls, imm>;
3713: }
3714: 
3715: class BinaryRIL<string mnemonic, bits<12> opcode, SDPatternOperator operator,
3716:                 RegisterOperand cls, ImmOpWithPattern imm>
3717:   : InstRILa<opcode, (outs cls:$R1), (ins cls:$R1src, imm:$I2),
3718:              mnemonic#"\t$R1, $I2",
3719:              [(set cls:$R1, (operator cls:$R1src, imm:$I2))]> {
3720:   let Constraints = "$R1 = $R1src";
3721: }
3722: 
3723: class BinaryRS<string mnemonic, bits<8> opcode, SDPatternOperator operator,
3724:                RegisterOperand cls>
3725:   : InstRSa<opcode, (outs cls:$R1),
3726:             (ins cls:$R1src, (shift12only $B2, $D2):$BD2),
3727:             mnemonic#"\t$R1, $BD2",
3728:             [(set cls:$R1, (operator cls:$R1src, shift12only:$BD2))]> {
3729:   let R3 = 0;
3730:   let Constraints = "$R1 = $R1src";
3731: }
3732: 
3733: class BinaryRSY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3734:                 RegisterOperand cls>
3735:   : InstRSYa<opcode, (outs cls:$R1), (ins cls:$R3, (shift20only $B2, $D2):$BD2),
3736:              mnemonic#"\t$R1, $R3, $BD2",
3737:              [(set cls:$R1, (operator cls:$R3, shift20only:$BD2))]>;
3738: 
3739: multiclass BinaryRSAndK<string mnemonic, bits<8> opcode1, bits<16> opcode2,
3740:                         SDPatternOperator operator, RegisterOperand cls> {
```
- **EN**: This block declares or refines TableGen records such as `AsmCondBinaryRIE`, `FixedCondBinaryRIE`, `CondBinaryRIEPair`, `Asm`, `BinaryRIL`, `BinaryRS`.
- **CN**: 该代码块声明或细化了 `AsmCondBinaryRIE`, `FixedCondBinaryRIE`, `CondBinaryRIEPair`, `Asm`, `BinaryRIL`, `BinaryRS` 等 TableGen 记录。

### Lines 3741-3795 / 第 3741-3795 行
```tablegen
3741:   let NumOpsKey = mnemonic in {
3742:     let NumOpsValue = "3" in
3743:       def K  : BinaryRSY<mnemonic#"k", opcode2, operator, cls>,
3744:                Requires<[FeatureDistinctOps]>;
3745:     let NumOpsValue = "2" in
3746:       def "" : BinaryRS<mnemonic, opcode1, operator, cls>;
3747:   }
3748: }
3749: 
3750: class BinaryRSL<string mnemonic, bits<16> opcode, RegisterOperand cls>
3751:   : InstRSLb<opcode, (outs cls:$R1),
3752:              (ins (bdladdr12onlylen8 $B2, $D2, $L2):$BDL2, imm32zx4:$M3),
3753:              mnemonic#"\t$R1, $BDL2, $M3", []> {
3754:   let mayLoad = 1;
3755: }
3756: 
3757: class BinaryRX<string mnemonic, bits<8> opcode, SDPatternOperator operator,
3758:                RegisterOperand cls, SDPatternOperator load, bits<5> bytes,
3759:                AddressingMode mode = bdxaddr12only>
3760:   : InstRXa<opcode, (outs cls:$R1), (ins cls:$R1src, (mode $B2, $D2, $X2):$XBD2),
3761:             mnemonic#"\t$R1, $XBD2",
3762:             [(set cls:$R1, (operator cls:$R1src, (load mode:$XBD2)))]> {
3763:   let OpKey = mnemonic#"r"#cls;
3764:   let OpType = "mem";
3765:   let Constraints = "$R1 = $R1src";
3766:   let mayLoad = 1;
3767:   let AccessBytes = bytes;
3768: }
3769: 
3770: class BinaryRXE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3771:                   RegisterOperand cls, SDPatternOperator load, bits<5> bytes>
3772:   : InstRXE<opcode, (outs cls:$R1),
3773:             (ins cls:$R1src, (bdxaddr12only $B2, $D2, $X2):$XBD2),
3774:             mnemonic#"\t$R1, $XBD2",
3775:             [(set cls:$R1, (operator cls:$R1src,
3776:                                      (load bdxaddr12only:$XBD2)))]> {
3777:   let OpKey = mnemonic#"r"#cls;
3778:   let OpType = "mem";
3779:   let Constraints = "$R1 = $R1src";
3780:   let mayLoad = 1;
3781:   let AccessBytes = bytes;
3782:   let M3 = 0;
3783: }
3784: 
3785: class BinaryRXF<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3786:                 RegisterOperand cls1, RegisterOperand cls2,
3787:                 SDPatternOperator load, bits<5> bytes>
3788:   : InstRXF<opcode, (outs cls1:$R1),
3789:             (ins cls2:$R3, (bdxaddr12only $B2, $D2, $X2):$XBD2),
3790:             mnemonic#"\t$R1, $R3, $XBD2",
3791:             [(set cls1:$R1, (operator cls2:$R3, (load bdxaddr12only:$XBD2)))]> {
3792:   let OpKey = mnemonic#"r"#cls;
3793:   let OpType = "mem";
3794:   let mayLoad = 1;
3795:   let AccessBytes = bytes;
```
- **EN**: This block declares or refines TableGen records such as `K`, `BinaryRSL`, `BinaryRX`, `BinaryRXE`, `BinaryRXF`.
- **CN**: 该代码块声明或细化了 `K`, `BinaryRSL`, `BinaryRX`, `BinaryRXE`, `BinaryRXF` 等 TableGen 记录。

### Lines 3796-3850 / 第 3796-3850 行
```tablegen
3796: }
3797: 
3798: class BinaryRXY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3799:                 RegisterOperand cls, SDPatternOperator load, bits<5> bytes,
3800:                 AddressingMode mode = bdxaddr20only>
3801:   : InstRXYa<opcode, (outs cls:$R1),
3802:              (ins cls:$R1src, (mode $B2, $D2, $X2):$XBD2),
3803:              mnemonic#"\t$R1, $XBD2",
3804:              [(set cls:$R1, (operator cls:$R1src, (load mode:$XBD2)))]> {
3805:   let OpKey = mnemonic#"r"#cls;
3806:   let OpType = "mem";
3807:   let Constraints = "$R1 = $R1src";
3808:   let mayLoad = 1;
3809:   let AccessBytes = bytes;
3810: }
3811: 
3812: multiclass BinaryRXPair<string mnemonic, bits<8> rxOpcode, bits<16> rxyOpcode,
3813:                         SDPatternOperator operator, RegisterOperand cls,
3814:                         SDPatternOperator load, bits<5> bytes> {
3815:   let DispKey = mnemonic # cls in {
3816:     let DispSize = "12" in
3817:       def "" : BinaryRX<mnemonic, rxOpcode, operator, cls, load, bytes,
3818:                         bdxaddr12pair>;
3819:     let DispSize = "20" in
3820:       def Y  : BinaryRXY<mnemonic#"y", rxyOpcode, operator, cls, load, bytes,
3821:                          bdxaddr20pair>;
3822:   }
3823: }
3824: 
3825: class BinarySI<string mnemonic, bits<8> opcode, SDPatternOperator operator,
3826:                Operand imm, AddressingMode mode = bdaddr12only>
3827:   : InstSI<opcode, (outs), (ins (mode $B1, $D1):$BD1, imm:$I2),
3828:            mnemonic#"\t$BD1, $I2",
3829:            [(store (operator (z_load mode:$BD1), imm:$I2), mode:$BD1)]> {
3830:   let mayLoad = 1;
3831:   let mayStore = 1;
3832: }
3833: 
3834: class BinarySIY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3835:                 Operand imm, AddressingMode mode = bdaddr20only>
3836:   : InstSIY<opcode, (outs), (ins (mode $B1, $D1):$BD1, imm:$I2),
3837:             mnemonic#"\t$BD1, $I2",
3838:             [(store (operator (z_load mode:$BD1), imm:$I2), mode:$BD1)]> {
3839:   let mayLoad = 1;
3840:   let mayStore = 1;
3841: }
3842: 
3843: multiclass BinarySIPair<string mnemonic, bits<8> siOpcode,
3844:                         bits<16> siyOpcode, SDPatternOperator operator,
3845:                         Operand imm> {
3846:   let DispKey = mnemonic # cls in {
3847:     let DispSize = "12" in
3848:       def "" : BinarySI<mnemonic, siOpcode, operator, imm, bdaddr12pair>;
3849:     let DispSize = "20" in
3850:       def Y  : BinarySIY<mnemonic#"y", siyOpcode, operator, imm, bdaddr20pair>;
```
- **EN**: This block declares or refines TableGen records such as `BinaryRXY`, `BinaryRXPair`, `Y`, `BinarySI`, `BinarySIY`, `BinarySIPair`.
- **CN**: 该代码块声明或细化了 `BinaryRXY`, `BinaryRXPair`, `Y`, `BinarySI`, `BinarySIY`, `BinarySIPair` 等 TableGen 记录。

### Lines 3851-3905 / 第 3851-3905 行
```tablegen
3851:   }
3852: }
3853: 
3854: class BinarySSF<string mnemonic, bits<12> opcode, RegisterOperand cls>
3855:   : InstSSF<opcode, (outs cls:$R3),
3856:             (ins (bdaddr12pair $B1, $D1):$BD1, (bdaddr12pair $B2, $D2):$BD2),
3857:             mnemonic#"\t$R3, $BD1, $BD2", []> {
3858:   let mayLoad = 1;
3859: }
3860: 
3861: class BinaryVRIb<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3862:                  TypedReg tr, bits<4> type>
3863:   : InstVRIb<opcode, (outs tr.op:$V1), (ins imm32zx8:$I2, imm32zx8:$I3),
3864:              mnemonic#"\t$V1, $I2, $I3",
3865:              [(set (tr.vt tr.op:$V1), (operator imm32zx8_timm:$I2, imm32zx8_timm:$I3))]> {
3866:   let M4 = type;
3867: }
3868: 
3869: class BinaryVRIbGeneric<string mnemonic, bits<16> opcode>
3870:   : InstVRIb<opcode, (outs VR128:$V1),
3871:              (ins imm32zx8:$I2, imm32zx8:$I3, imm32zx4:$M4),
3872:              mnemonic#"\t$V1, $I2, $I3, $M4", []>;
3873: 
3874: class BinaryVRIc<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3875:                  TypedReg tr1, TypedReg tr2, bits<4> type>
3876:   : InstVRIc<opcode, (outs tr1.op:$V1), (ins tr2.op:$V3, imm32zx16:$I2),
3877:              mnemonic#"\t$V1, $V3, $I2",
3878:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V3),
3879:                                                   imm32zx16_timm:$I2))]> {
3880:   let M4 = type;
3881: }
3882: 
3883: class BinaryVRIcGeneric<string mnemonic, bits<16> opcode>
3884:   : InstVRIc<opcode, (outs VR128:$V1),
3885:              (ins VR128:$V3, imm32zx16:$I2, imm32zx4:$M4),
3886:              mnemonic#"\t$V1, $V3, $I2, $M4", []>;
3887: 
3888: class BinaryVRIe<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3889:                  TypedReg tr1, TypedReg tr2, bits<4> type, bits<4> m5>
3890:   : InstVRIe<opcode, (outs tr1.op:$V1), (ins tr2.op:$V2, imm32zx12:$I3),
3891:              mnemonic#"\t$V1, $V2, $I3",
3892:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
3893:                                                   imm32zx12_timm:$I3))]> {
3894:   let M4 = type;
3895:   let M5 = m5;
3896: }
3897: 
3898: class BinaryVRIeFloatGeneric<string mnemonic, bits<16> opcode>
3899:   : InstVRIe<opcode, (outs VR128:$V1),
3900:              (ins VR128:$V2, imm32zx12:$I3, imm32zx4:$M4, imm32zx4:$M5),
3901:              mnemonic#"\t$V1, $V2, $I3, $M4, $M5", []>;
3902: 
3903: class BinaryVRIh<string mnemonic, bits<16> opcode>
3904:   : InstVRIh<opcode, (outs VR128:$V1),
3905:              (ins imm32zx16:$I2, imm32zx4:$I3),
```
- **EN**: This block declares or refines TableGen records such as `BinarySSF`, `BinaryVRIb`, `BinaryVRIbGeneric`, `BinaryVRIc`, `BinaryVRIcGeneric`, `BinaryVRIe`.
- **CN**: 该代码块声明或细化了 `BinarySSF`, `BinaryVRIb`, `BinaryVRIbGeneric`, `BinaryVRIc`, `BinaryVRIcGeneric`, `BinaryVRIe` 等 TableGen 记录。

### Lines 3906-3960 / 第 3906-3960 行
```tablegen
3906:              mnemonic#"\t$V1, $I2, $I3", []>;
3907: 
3908: class BinaryVRRa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3909:                  TypedReg tr1, TypedReg tr2, bits<4> type = 0, bits<4> m4 = 0>
3910:   : InstVRRa<opcode, (outs tr1.op:$V1), (ins tr2.op:$V2, imm32zx4:$M5),
3911:              mnemonic#"\t$V1, $V2, $M5",
3912:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
3913:                                                   imm32zx12:$M5))]> {
3914:   let M3 = type;
3915:   let M4 = m4;
3916: }
3917: 
3918: class BinaryVRRaFloatGeneric<string mnemonic, bits<16> opcode>
3919:   : InstVRRa<opcode, (outs VR128:$V1),
3920:              (ins VR128:$V2, imm32zx4:$M3, imm32zx4:$M4, imm32zx4:$M5),
3921:              mnemonic#"\t$V1, $V2, $M3, $M4, $M5", []>;
3922: 
3923: class BinaryVRRb<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3924:                  TypedReg tr1, TypedReg tr2, bits<4> type = 0,
3925:                  bits<4> modifier = 0>
3926:   : InstVRRb<opcode, (outs tr1.op:$V1), (ins tr2.op:$V2, tr2.op:$V3),
3927:              mnemonic#"\t$V1, $V2, $V3",
3928:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
3929:                                                   (tr2.vt tr2.op:$V3)))]> {
3930:   let M4 = type;
3931:   let M5 = modifier;
3932: }
3933: 
3934: class BinaryExtraVRRb<string mnemonic, bits<16> opcode, bits<4> type = 0>
3935:   : InstVRRb<opcode, (outs VR128:$V1), (ins VR128:$V2, VR128:$V3, imm32zx4:$M5),
3936:              mnemonic#"\t$V1, $V2, $V3, $M5", []> {
3937:   let M4 = type;
3938: }
3939: 
3940: class BinaryExtraVRRbGeneric<string mnemonic, bits<16> opcode>
3941:   : InstVRRb<opcode, (outs VR128:$V1),
3942:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5),
3943:              mnemonic#"\t$V1, $V2, $V3, $M4, $M5", []>;
3944: 
3945: // Declare a pair of instructions, one which sets CC and one which doesn't.
3946: // The CC-setting form ends with "S" and sets the low bit of M5.
3947: multiclass BinaryVRRbSPair<string mnemonic, bits<16> opcode,
3948:                            SDPatternOperator operator,
3949:                            SDPatternOperator operator_cc, TypedReg tr1,
3950:                            TypedReg tr2, bits<4> type, bits<4> modifier = 0> {
3951:   def "" : BinaryVRRb<mnemonic, opcode, operator, tr1, tr2, type,
3952:                       !and (modifier, 14)>;
3953:   let Defs = [CC] in
3954:     def S : BinaryVRRb<mnemonic#"s", opcode, operator_cc, tr1, tr2, type,
3955:                        !add (!and (modifier, 14), 1)>;
3956: }
3957: 
3958: class BinaryVRRbSPairGeneric<string mnemonic, bits<16> opcode>
3959:   : InstVRRb<opcode, (outs VR128:$V1),
3960:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5),
```
- **EN**: This block declares or refines TableGen records such as `BinaryVRRa`, `BinaryVRRaFloatGeneric`, `BinaryVRRb`, `BinaryExtraVRRb`, `BinaryExtraVRRbGeneric`, `BinaryVRRbSPair`.
- **CN**: 该代码块声明或细化了 `BinaryVRRa`, `BinaryVRRaFloatGeneric`, `BinaryVRRb`, `BinaryExtraVRRb`, `BinaryExtraVRRbGeneric`, `BinaryVRRbSPair` 等 TableGen 记录。

### Lines 3961-4015 / 第 3961-4015 行
```tablegen
3961:              mnemonic#"\t$V1, $V2, $V3, $M4, $M5", []> {
3962:   let Defs = [CC];
3963: }
3964: 
3965: // Declare a pair of instructions, one which sets CC and one which doesn't.
3966: // The CC-setting form ends with "S" and sets the low bit of M5.
3967: // The form that does not set CC has an extra operand to optionally allow
3968: // specifying arbitrary M5 values in assembler.
3969: multiclass BinaryExtraVRRbSPair<string mnemonic, bits<16> opcode,
3970:                                 SDPatternOperator operator,
3971:                                 SDPatternOperator operator_cc,
3972:                                 TypedReg tr1, TypedReg tr2, bits<4> type> {
3973:   let M4 = type in
3974:     def "" : InstVRRb<opcode, (outs tr1.op:$V1),
3975:                       (ins tr2.op:$V2, tr2.op:$V3, imm32zx4:$M5),
3976:                       mnemonic#"\t$V1, $V2, $V3, $M5", []>;
3977:   def : Pat<(tr1.vt (operator (tr2.vt tr2.op:$V2), (tr2.vt tr2.op:$V3))),
3978:             (!cast<Instruction>(NAME) tr2.op:$V2, tr2.op:$V3, 0)>;
3979:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3",
3980:                   (!cast<Instruction>(NAME) tr1.op:$V1, tr2.op:$V2,
3981:                                             tr2.op:$V3, 0)>;
3982:   let Defs = [CC] in
3983:     def S : BinaryVRRb<mnemonic#"s", opcode, operator_cc, tr1, tr2, type, 1>;
3984: }
3985: 
3986: multiclass BinaryExtraVRRbSPairGeneric<string mnemonic, bits<16> opcode> {
3987:   let Defs = [CC] in
3988:     def "" : InstVRRb<opcode, (outs VR128:$V1),
3989:                      (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5),
3990:                      mnemonic#"\t$V1, $V2, $V3, $M4, $M5", []>;
3991:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3, $M4",
3992:                   (!cast<Instruction>(NAME) VR128:$V1, VR128:$V2, VR128:$V3,
3993:                                             imm32zx4:$M4, 0)>;
3994: }
3995: 
3996: class BinaryVRRc<string mnemonic, bits<16> opcode, SDPatternOperator operator,
3997:                  TypedReg tr1, TypedReg tr2, bits<4> type = 0, bits<4> m5 = 0,
3998:                  bits<4> m6 = 0, string fp_mnemonic = "">
3999:   : InstVRRc<opcode, (outs tr1.op:$V1), (ins tr2.op:$V2, tr2.op:$V3),
4000:              mnemonic#"\t$V1, $V2, $V3",
4001:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4002:                                                   (tr2.vt tr2.op:$V3)))]> {
4003:   let M4 = type;
4004:   let M5 = m5;
4005:   let M6 = m6;
4006:   let OpKey = fp_mnemonic#"MemFold"#!subst("VR", "FP", !cast<string>(tr1.op));
4007:   let OpType = "reg";
4008: }
4009: 
4010: class BinaryVRRcGeneric<string mnemonic, bits<16> opcode, bits<4> m5 = 0,
4011:                         bits<4> m6 = 0>
4012:   : InstVRRc<opcode, (outs VR128:$V1),
4013:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4),
4014:              mnemonic#"\t$V1, $V2, $V3, $M4", []> {
4015:   let M5 = m5;
```
- **EN**: This block declares or refines TableGen records such as `BinaryExtraVRRbSPair`, `S`, `BinaryExtraVRRbSPairGeneric`, `BinaryVRRc`, `BinaryVRRcGeneric`.
- **CN**: 该代码块声明或细化了 `BinaryExtraVRRbSPair`, `S`, `BinaryExtraVRRbSPairGeneric`, `BinaryVRRc`, `BinaryVRRcGeneric` 等 TableGen 记录。

### Lines 4016-4070 / 第 4016-4070 行
```tablegen
4016:   let M6 = m6;
4017: }
4018: 
4019: class BinaryVRRcFloatGeneric<string mnemonic, bits<16> opcode, bits<4> m6 = 0>
4020:   : InstVRRc<opcode, (outs VR128:$V1),
4021:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5),
4022:              mnemonic#"\t$V1, $V2, $V3, $M4, $M5", []> {
4023:   let M6 = m6;
4024: }
4025: 
4026: // Declare a pair of instructions, one which sets CC and one which doesn't.
4027: // The CC-setting form ends with "S" and sets the low bit of M5.
4028: multiclass BinaryVRRcSPair<string mnemonic, bits<16> opcode,
4029:                            SDPatternOperator operator,
4030:                            SDPatternOperator operator_cc, TypedReg tr1,
4031:                            TypedReg tr2, bits<4> type, bits<4> m5,
4032:                            bits<4> modifier = 0> {
4033:   def "" : BinaryVRRc<mnemonic, opcode, operator, tr1, tr2, type,
4034:                       m5, !and (modifier, 14)>;
4035:   let Defs = [CC] in
4036:     def S : BinaryVRRc<mnemonic#"s", opcode, operator_cc, tr1, tr2, type,
4037:                        m5, !add (!and (modifier, 14), 1)>;
4038: }
4039: 
4040: class BinaryVRRcSPairFloatGeneric<string mnemonic, bits<16> opcode>
4041:   : InstVRRc<opcode, (outs VR128:$V1),
4042:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5,
4043:                   imm32zx4:$M6),
4044:              mnemonic#"\t$V1, $V2, $V3, $M4, $M5, $M6", []>;
4045: 
4046: class BinaryVRRf<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4047:                  TypedReg tr>
4048:   : InstVRRf<opcode, (outs tr.op:$V1), (ins GR64:$R2, GR64:$R3),
4049:              mnemonic#"\t$V1, $R2, $R3",
4050:              [(set (tr.vt tr.op:$V1), (operator GR64:$R2, GR64:$R3))]>;
4051: 
4052: class BinaryVRRi<string mnemonic, bits<16> opcode, RegisterOperand cls>
4053:   : InstVRRi<opcode, (outs cls:$R1), (ins VR128:$V2, imm32zx4:$M3),
4054:              mnemonic#"\t$R1, $V2, $M3", []> {
4055:   let M4 = 0;
4056: }
4057: 
4058: class BinaryVRRk<string mnemonic, bits<16> opcode>
4059:   : InstVRRk<opcode, (outs VR128:$V1), (ins VR128:$V2, imm32zx4:$M3),
4060:              mnemonic#"\t$V1, $V2, $M3", []>;
4061: 
4062: class BinaryVRSa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4063:                  TypedReg tr1, TypedReg tr2, bits<4> type>
4064:   : InstVRSa<opcode, (outs tr1.op:$V1),
4065:              (ins tr2.op:$V3, (shift12only $B2, $D2):$BD2),
4066:              mnemonic#"\t$V1, $V3, $BD2",
4067:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V3),
4068:                                                   shift12only:$BD2))]> {
4069:   let M4 = type;
4070: }
```
- **EN**: This block declares or refines TableGen records such as `BinaryVRRcFloatGeneric`, `BinaryVRRcSPair`, `S`, `BinaryVRRcSPairFloatGeneric`, `BinaryVRRf`, `BinaryVRRi`.
- **CN**: 该代码块声明或细化了 `BinaryVRRcFloatGeneric`, `BinaryVRRcSPair`, `S`, `BinaryVRRcSPairFloatGeneric`, `BinaryVRRf`, `BinaryVRRi` 等 TableGen 记录。

### Lines 4071-4125 / 第 4071-4125 行
```tablegen
4071: 
4072: class BinaryVRSaGeneric<string mnemonic, bits<16> opcode>
4073:   : InstVRSa<opcode, (outs VR128:$V1),
4074:              (ins VR128:$V3, (shift12only $B2, $D2):$BD2, imm32zx4:$M4),
4075:              mnemonic#"\t$V1, $V3, $BD2, $M4", []>;
4076: 
4077: class BinaryVRSb<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4078:                  bits<5> bytes>
4079:   : InstVRSb<opcode, (outs VR128:$V1),
4080:              (ins GR32:$R3, (bdaddr12only $B2, $D2):$BD2),
4081:              mnemonic#"\t$V1, $R3, $BD2",
4082:              [(set VR128:$V1, (operator GR32:$R3, bdaddr12only:$BD2))]> {
4083:   let M4 = 0;
4084:   let mayLoad = 1;
4085:   let AccessBytes = bytes;
4086: }
4087: 
4088: class BinaryVRSc<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4089:                  TypedReg tr, bits<4> type>
4090:   : InstVRSc<opcode, (outs GR64:$R1),
4091:              (ins tr.op:$V3, (shift12only $B2, $D2):$BD2),
4092:              mnemonic#"\t$R1, $V3, $BD2",
4093:              [(set GR64:$R1, (operator (tr.vt tr.op:$V3), shift12only:$BD2))]> {
4094:   let M4 = type;
4095: }
4096: 
4097: class BinaryVRScGeneric<string mnemonic, bits<16> opcode>
4098:   : InstVRSc<opcode, (outs GR64:$R1),
4099:              (ins VR128:$V3, (shift12only $B2, $D2):$BD2, imm32zx4: $M4),
4100:              mnemonic#"\t$R1, $V3, $BD2, $M4", []>;
4101: 
4102: class BinaryVRSd<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4103:                  bits<5> bytes>
4104:   : InstVRSd<opcode, (outs VR128:$V1),
4105:              (ins GR32:$R3, (bdaddr12only $B2, $D2):$BD2),
4106:              mnemonic#"\t$V1, $R3, $BD2",
4107:              [(set VR128:$V1, (operator GR32:$R3, bdaddr12only:$BD2))]> {
4108:   let mayLoad = 1;
4109:   let AccessBytes = bytes;
4110: }
4111: 
4112: class BinaryVRX<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4113:                 TypedReg tr, bits<5> bytes>
4114:   : InstVRX<opcode, (outs VR128:$V1),
4115:             (ins (bdxaddr12only $B2, $D2, $X2):$XBD2, imm32zx4:$M3),
4116:             mnemonic#"\t$V1, $XBD2, $M3",
4117:             [(set (tr.vt tr.op:$V1), (operator bdxaddr12only:$XBD2,
4118:                                                imm32zx4_timm:$M3))]> {
4119:   let mayLoad = 1;
4120:   let AccessBytes = bytes;
4121: }
4122: 
4123: class StoreBinaryRS<string mnemonic, bits<8> opcode, RegisterOperand cls,
4124:                     bits<5> bytes, AddressingMode mode = bdaddr12only>
4125:   : InstRSb<opcode, (outs), (ins cls:$R1, imm32zx4:$M3, (mode $B2, $D2):$BD2),
```
- **EN**: This block declares or refines TableGen records such as `BinaryVRSaGeneric`, `BinaryVRSb`, `BinaryVRSc`, `BinaryVRScGeneric`, `BinaryVRSd`, `BinaryVRX`.
- **CN**: 该代码块声明或细化了 `BinaryVRSaGeneric`, `BinaryVRSb`, `BinaryVRSc`, `BinaryVRScGeneric`, `BinaryVRSd`, `BinaryVRX` 等 TableGen 记录。

### Lines 4126-4180 / 第 4126-4180 行
```tablegen
4126:             mnemonic#"\t$R1, $M3, $BD2", []> {
4127:   let mayStore = 1;
4128:   let AccessBytes = bytes;
4129: }
4130: 
4131: class StoreBinaryRSY<string mnemonic, bits<16> opcode, RegisterOperand cls,
4132:                      bits<5> bytes, AddressingMode mode = bdaddr20only>
4133:   : InstRSYb<opcode, (outs), (ins cls:$R1, imm32zx4:$M3, (mode $B2, $D2):$BD2),
4134:              mnemonic#"\t$R1, $M3, $BD2", []> {
4135:   let mayStore = 1;
4136:   let AccessBytes = bytes;
4137: }
4138: 
4139: multiclass StoreBinaryRSPair<string mnemonic, bits<8> rsOpcode,
4140:                              bits<16> rsyOpcode, RegisterOperand cls,
4141:                              bits<5> bytes> {
4142:   let DispKey = mnemonic # cls in {
4143:     let DispSize = "12" in
4144:       def "" : StoreBinaryRS<mnemonic, rsOpcode, cls, bytes, bdaddr12pair>;
4145:     let DispSize = "20" in
4146:       def Y  : StoreBinaryRSY<mnemonic#"y", rsyOpcode, cls, bytes,
4147:                               bdaddr20pair>;
4148:   }
4149: }
4150: 
4151: class StoreBinaryRSL<string mnemonic, bits<16> opcode, RegisterOperand cls>
4152:   : InstRSLb<opcode, (outs),
4153:              (ins cls:$R1, (bdladdr12onlylen8 $B2, $D2, $L2):$BDL2,
4154:                   imm32zx4:$M3),
4155:              mnemonic#"\t$R1, $BDL2, $M3", []> {
4156:   let mayStore = 1;
4157: }
4158: 
4159: class BinaryVSI<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4160:                 bits<5> bytes>
4161:   : InstVSI<opcode, (outs VR128:$V1),
4162:             (ins (bdaddr12only $B2, $D2):$BD2, imm32zx8:$I3),
4163:             mnemonic#"\t$V1, $BD2, $I3",
4164:             [(set VR128:$V1, (operator imm32zx8:$I3, bdaddr12only:$BD2))]> {
4165:   let mayLoad = 1;
4166:   let AccessBytes = bytes;
4167: }
4168: 
4169: class StoreBinaryVRV<string mnemonic, bits<16> opcode, bits<5> bytes,
4170:                      ImmOpWithPattern index>
4171:   : InstVRV<opcode, (outs),
4172:             (ins VR128:$V1, (bdvaddr12only $B2, $D2, $V2):$VBD2, index:$M3),
4173:             mnemonic#"\t$V1, $VBD2, $M3", []> {
4174:   let mayStore = 1;
4175:   let AccessBytes = bytes;
4176: }
4177: 
4178: class StoreBinaryVRX<string mnemonic, bits<16> opcode,
4179:                      SDPatternOperator operator, TypedReg tr, bits<5> bytes,
4180:                      ImmOpWithPattern index>
```
- **EN**: This block declares or refines TableGen records such as `StoreBinaryRSY`, `StoreBinaryRSPair`, `Y`, `StoreBinaryRSL`, `BinaryVSI`, `StoreBinaryVRV`.
- **CN**: 该代码块声明或细化了 `StoreBinaryRSY`, `StoreBinaryRSPair`, `Y`, `StoreBinaryRSL`, `BinaryVSI`, `StoreBinaryVRV` 等 TableGen 记录。

### Lines 4181-4235 / 第 4181-4235 行
```tablegen
4181:   : InstVRX<opcode, (outs),
4182:             (ins tr.op:$V1, (bdxaddr12only $B2, $D2, $X2):$XBD2, index:$M3),
4183:             mnemonic#"\t$V1, $XBD2, $M3",
4184:             [(operator (tr.vt tr.op:$V1), bdxaddr12only:$XBD2, index:$M3)]> {
4185:   let mayStore = 1;
4186:   let AccessBytes = bytes;
4187: }
4188: 
4189: class MemoryBinarySSd<string mnemonic, bits<8> opcode,
4190:                       RegisterOperand cls>
4191:   : InstSSd<opcode, (outs),
4192:             (ins (bdraddr12only $B1, $D1, $R1):$RBD1,
4193:                  (bdaddr12only $B2, $D2):$BD2, cls:$R3),
4194:             mnemonic#"\t$RBD1, $BD2, $R3", []>;
4195: 
4196: class CompareRR<string mnemonic, bits<8> opcode, SDPatternOperator operator,
4197:                 RegisterOperand cls1, RegisterOperand cls2>
4198:   : InstRR<opcode, (outs), (ins cls1:$R1, cls2:$R2),
4199:            mnemonic#"\t$R1, $R2",
4200:            [(set CC, (operator cls1:$R1, cls2:$R2))]> {
4201:   let OpKey = mnemonic#cls1;
4202:   let OpType = "reg";
4203:   let isCompare = 1;
4204: }
4205: 
4206: class CompareRRE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4207:                  RegisterOperand cls1, RegisterOperand cls2>
4208:   : InstRRE<opcode, (outs), (ins cls1:$R1, cls2:$R2),
4209:             mnemonic#"\t$R1, $R2",
4210:             [(set CC, (operator cls1:$R1, cls2:$R2))]> {
4211:   let OpKey = mnemonic#cls1;
4212:   let OpType = "reg";
4213:   let isCompare = 1;
4214: }
4215: 
4216: class CompareRI<string mnemonic, bits<12> opcode, SDPatternOperator operator,
4217:                 RegisterOperand cls, ImmOpWithPattern imm>
4218:   : InstRIa<opcode, (outs), (ins cls:$R1, imm:$I2),
4219:             mnemonic#"\t$R1, $I2",
4220:             [(set CC, (operator cls:$R1, imm:$I2))]> {
4221:   let isCompare = 1;
4222: }
4223: 
4224: class CompareRIL<string mnemonic, bits<12> opcode, SDPatternOperator operator,
4225:                  RegisterOperand cls, ImmOpWithPattern imm>
4226:   : InstRILa<opcode, (outs), (ins cls:$R1, imm:$I2),
4227:              mnemonic#"\t$R1, $I2",
4228:              [(set CC, (operator cls:$R1, imm:$I2))]> {
4229:   let isCompare = 1;
4230: }
4231: 
4232: class CompareRILPC<string mnemonic, bits<12> opcode, SDPatternOperator operator,
4233:                    RegisterOperand cls, SDPatternOperator load>
4234:   : InstRILb<opcode, (outs), (ins cls:$R1, pcrel32:$RI2),
4235:              mnemonic#"\t$R1, $RI2",
```
- **EN**: This block declares or refines TableGen records such as `MemoryBinarySSd`, `CompareRR`, `CompareRRE`, `CompareRI`, `CompareRIL`, `CompareRILPC`.
- **CN**: 该代码块声明或细化了 `MemoryBinarySSd`, `CompareRR`, `CompareRRE`, `CompareRI`, `CompareRIL`, `CompareRILPC` 等 TableGen 记录。

### Lines 4236-4290 / 第 4236-4290 行
```tablegen
4236:              [(set CC, (operator cls:$R1, (load pcrel32:$RI2)))]> {
4237:   let isCompare = 1;
4238:   let mayLoad = 1;
4239:   // We want PC-relative addresses to be tried ahead of BD and BDX addresses.
4240:   // However, BDXs have two extra operands and are therefore 6 units more
4241:   // complex.
4242:   let AddedComplexity = 7;
4243: }
4244: 
4245: class CompareRX<string mnemonic, bits<8> opcode, SDPatternOperator operator,
4246:                 RegisterOperand cls, SDPatternOperator load, bits<5> bytes,
4247:                 AddressingMode mode = bdxaddr12only>
4248:   : InstRXa<opcode, (outs), (ins cls:$R1, (mode $B2, $D2, $X2):$XBD2),
4249:             mnemonic#"\t$R1, $XBD2",
4250:             [(set CC, (operator cls:$R1, (load mode:$XBD2)))]> {
4251:   let OpKey = mnemonic#"r"#cls;
4252:   let OpType = "mem";
4253:   let isCompare = 1;
4254:   let mayLoad = 1;
4255:   let AccessBytes = bytes;
4256: }
4257: 
4258: class CompareRXE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4259:                  RegisterOperand cls, SDPatternOperator load, bits<5> bytes>
4260:   : InstRXE<opcode, (outs), (ins cls:$R1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
4261:             mnemonic#"\t$R1, $XBD2",
4262:             [(set CC, (operator cls:$R1, (load bdxaddr12only:$XBD2)))]> {
4263:   let OpKey = mnemonic#"r"#cls;
4264:   let OpType = "mem";
4265:   let isCompare = 1;
4266:   let mayLoad = 1;
4267:   let AccessBytes = bytes;
4268:   let M3 = 0;
4269: }
4270: 
4271: class CompareRXY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4272:                  RegisterOperand cls, SDPatternOperator load, bits<5> bytes,
4273:                  AddressingMode mode = bdxaddr20only>
4274:   : InstRXYa<opcode, (outs), (ins cls:$R1, (mode $B2, $D2, $X2):$XBD2),
4275:              mnemonic#"\t$R1, $XBD2",
4276:              [(set CC, (operator cls:$R1, (load mode:$XBD2)))]> {
4277:   let OpKey = mnemonic#"r"#cls;
4278:   let OpType = "mem";
4279:   let isCompare = 1;
4280:   let mayLoad = 1;
4281:   let AccessBytes = bytes;
4282: }
4283: 
4284: multiclass CompareRXPair<string mnemonic, bits<8> rxOpcode, bits<16> rxyOpcode,
4285:                          SDPatternOperator operator, RegisterOperand cls,
4286:                          SDPatternOperator load, bits<5> bytes> {
4287:   let DispKey = mnemonic # cls in {
4288:     let DispSize = "12" in
4289:       def "" : CompareRX<mnemonic, rxOpcode, operator, cls,
4290:                          load, bytes, bdxaddr12pair>;
```
- **EN**: This block declares or refines TableGen records such as `CompareRX`, `CompareRXE`, `CompareRXY`, `CompareRXPair`.
- **CN**: 该代码块声明或细化了 `CompareRX`, `CompareRXE`, `CompareRXY`, `CompareRXPair` 等 TableGen 记录。

### Lines 4291-4345 / 第 4291-4345 行
```tablegen
4291:     let DispSize = "20" in
4292:       def Y  : CompareRXY<mnemonic#"y", rxyOpcode, operator, cls,
4293:                           load, bytes, bdxaddr20pair>;
4294:   }
4295: }
4296: 
4297: class CompareRS<string mnemonic, bits<8> opcode, RegisterOperand cls,
4298:                 bits<5> bytes, AddressingMode mode = bdaddr12only>
4299:   : InstRSb<opcode, (outs), (ins cls:$R1, imm32zx4:$M3, (mode $B2, $D2):$BD2),
4300:             mnemonic#"\t$R1, $M3, $BD2", []> {
4301:   let mayLoad = 1;
4302:   let AccessBytes = bytes;
4303: }
4304: 
4305: class CompareRSY<string mnemonic, bits<16> opcode, RegisterOperand cls,
4306:                  bits<5> bytes, AddressingMode mode = bdaddr20only>
4307:   : InstRSYb<opcode, (outs), (ins cls:$R1, imm32zx4:$M3, (mode $B2, $D2):$BD2),
4308:              mnemonic#"\t$R1, $M3, $BD2", []> {
4309:   let mayLoad = 1;
4310:   let AccessBytes = bytes;
4311: }
4312: 
4313: multiclass CompareRSPair<string mnemonic, bits<8> rsOpcode, bits<16> rsyOpcode,
4314:                          RegisterOperand cls, bits<5> bytes> {
4315:   let DispKey = mnemonic # cls in {
4316:     let DispSize = "12" in
4317:       def "" : CompareRS<mnemonic, rsOpcode, cls, bytes, bdaddr12pair>;
4318:     let DispSize = "20" in
4319:       def Y  : CompareRSY<mnemonic#"y", rsyOpcode, cls, bytes, bdaddr20pair>;
4320:   }
4321: }
4322: 
4323: class CompareSSb<string mnemonic, bits<8> opcode>
4324:   : InstSSb<opcode,
4325:             (outs), (ins (bdladdr12onlylen4 $B1, $D1, $L1):$BDL1,
4326:                          (bdladdr12onlylen4 $B2, $D2, $L2):$BDL2),
4327:             mnemonic#"\t$BDL1, $BDL2", []> {
4328:   let isCompare = 1;
4329:   let mayLoad = 1;
4330: }
4331: 
4332: class CompareSI<string mnemonic, bits<8> opcode, SDPatternOperator operator,
4333:                 SDPatternOperator load, ImmOpWithPattern imm,
4334:                 AddressingMode mode = bdaddr12only>
4335:   : InstSI<opcode, (outs), (ins (mode $B1, $D1):$BD1, imm:$I2),
4336:            mnemonic#"\t$BD1, $I2",
4337:            [(set CC, (operator (load mode:$BD1), imm:$I2))]> {
4338:   let isCompare = 1;
4339:   let mayLoad = 1;
4340: }
4341: 
4342: class CompareSIL<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4343:                  SDPatternOperator load, ImmOpWithPattern imm>
4344:   : InstSIL<opcode, (outs), (ins (bdaddr12only $B1, $D1):$BD1, imm:$I2),
4345:             mnemonic#"\t$BD1, $I2",
```
- **EN**: This block declares or refines TableGen records such as `Y`, `CompareRS`, `CompareRSY`, `CompareRSPair`, `Y`, `CompareSSb`.
- **CN**: 该代码块声明或细化了 `Y`, `CompareRS`, `CompareRSY`, `CompareRSPair`, `Y`, `CompareSSb` 等 TableGen 记录。

### Lines 4346-4400 / 第 4346-4400 行
```tablegen
4346:             [(set CC, (operator (load bdaddr12only:$BD1), imm:$I2))]> {
4347:   let isCompare = 1;
4348:   let mayLoad = 1;
4349: }
4350: 
4351: class CompareSIY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4352:                  SDPatternOperator load, ImmOpWithPattern imm,
4353:                  AddressingMode mode = bdaddr20only>
4354:   : InstSIY<opcode, (outs), (ins (mode $B1, $D1):$BD1, imm:$I2),
4355:             mnemonic#"\t$BD1, $I2",
4356:             [(set CC, (operator (load mode:$BD1), imm:$I2))]> {
4357:   let isCompare = 1;
4358:   let mayLoad = 1;
4359: }
4360: 
4361: multiclass CompareSIPair<string mnemonic, bits<8> siOpcode, bits<16> siyOpcode,
4362:                          SDPatternOperator operator, SDPatternOperator load,
4363:                          ImmOpWithPattern imm> {
4364:   let DispKey = mnemonic in {
4365:     let DispSize = "12" in
4366:       def "" : CompareSI<mnemonic, siOpcode, operator, load, imm, bdaddr12pair>;
4367:     let DispSize = "20" in
4368:       def Y  : CompareSIY<mnemonic#"y", siyOpcode, operator, load, imm,
4369:                           bdaddr20pair>;
4370:   }
4371: }
4372: 
4373: class CompareVRRa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4374:                   TypedReg tr, bits<4> type, string fp_mnemonic = "">
4375:   : InstVRRa<opcode, (outs), (ins tr.op:$V1, tr.op:$V2),
4376:              mnemonic#"\t$V1, $V2",
4377:              [(set CC, (operator (tr.vt tr.op:$V1), (tr.vt tr.op:$V2)))]> {
4378:   let isCompare = 1;
4379:   let M3 = type;
4380:   let M4 = 0;
4381:   let M5 = 0;
4382:   let OpKey = fp_mnemonic#!subst("VR", "FP", !cast<string>(tr.op));
4383:   let OpType = "reg";
4384: }
4385: 
4386: class CompareVRRaGeneric<string mnemonic, bits<16> opcode>
4387:   : InstVRRa<opcode, (outs), (ins VR128:$V1, VR128:$V2, imm32zx4:$M3),
4388:              mnemonic#"\t$V1, $V2, $M3", []> {
4389:   let isCompare = 1;
4390:   let M4 = 0;
4391:   let M5 = 0;
4392: }
4393: 
4394: class CompareVRRaFloatGeneric<string mnemonic, bits<16> opcode>
4395:   : InstVRRa<opcode, (outs),
4396:              (ins VR64:$V1, VR64:$V2, imm32zx4:$M3, imm32zx4:$M4),
4397:              mnemonic#"\t$V1, $V2, $M3, $M4", []> {
4398:   let isCompare = 1;
4399:   let M5 = 0;
4400: }
```
- **EN**: This block declares or refines TableGen records such as `CompareSIY`, `CompareSIPair`, `Y`, `CompareVRRa`, `CompareVRRaGeneric`, `CompareVRRaFloatGeneric`.
- **CN**: 该代码块声明或细化了 `CompareSIY`, `CompareSIPair`, `Y`, `CompareVRRa`, `CompareVRRaGeneric`, `CompareVRRaFloatGeneric` 等 TableGen 记录。

### Lines 4401-4455 / 第 4401-4455 行
```tablegen
4401: 
4402: class CompareVRRh<string mnemonic, bits<16> opcode>
4403:   : InstVRRh<opcode, (outs), (ins VR128:$V1, VR128:$V2, imm32zx4:$M3),
4404:              mnemonic#"\t$V1, $V2, $M3", []> {
4405:   let isCompare = 1;
4406: }
4407: 
4408: class TestInherentS<string mnemonic, bits<16> opcode,
4409:                     SDPatternOperator operator>
4410:   : InstS<opcode, (outs), (ins), mnemonic, [(set CC, (operator))]> {
4411:   let B2 = 0;
4412:   let D2 = 0;
4413: }
4414: 
4415: class TestRXE<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4416:               RegisterOperand cls>
4417:   : InstRXE<opcode, (outs), (ins cls:$R1, (bdxaddr12only $B2, $D2, $X2):$XBD2),
4418:             mnemonic#"\t$R1, $XBD2",
4419:             [(set CC, (operator cls:$R1, bdxaddr12only:$XBD2))]> {
4420:   let M3 = 0;
4421: }
4422: 
4423: class TestBinarySIL<string mnemonic, bits<16> opcode,
4424:                     SDPatternOperator operator, ImmOpWithPattern imm>
4425:   : InstSIL<opcode, (outs), (ins (bdaddr12only $B1, $D1):$BD1, imm:$I2),
4426:             mnemonic#"\t$BD1, $I2",
4427:             [(set CC, (operator bdaddr12only:$BD1, imm:$I2))]>;
4428: 
4429: class TestRSL<string mnemonic, bits<16> opcode>
4430:   : InstRSLa<opcode, (outs), (ins (bdladdr12onlylen4 $B1, $D1, $L1):$BDL1),
4431:              mnemonic#"\t$BDL1", []> {
4432:   let mayLoad = 1;
4433: }
4434: 
4435: class TestVRRg<string mnemonic, bits<16> opcode>
4436:   : InstVRRg<opcode, (outs), (ins VR128:$V1),
4437:              mnemonic#"\t$V1", []> {
4438:   let I2 = 0;
4439: }
4440: 
4441: class TestExtraVRRg<string mnemonic, bits<16> opcode>
4442:   : InstVRRg<opcode, (outs), (ins VR128:$V1, imm32zx16:$I2),
4443:              mnemonic#"\t$V1, $I2", []>;
4444: 
4445: class TestExtraVRIl<string mnemonic, bits<16> opcode>
4446:   : InstVRIl<opcode, (outs), (ins VR128:$V1, VR128:$V2, imm32zx16:$I3),
4447:              mnemonic#"\t$V1, $V2, $I3", []>;
4448: 
4449: class SideEffectTernarySSc<string mnemonic, bits<8> opcode>
4450:   : InstSSc<opcode, (outs), (ins (bdladdr12onlylen4 $B1, $D1, $L1):$BDL1,
4451:                                  (shift12only $B2, $D2):$BD2, imm32zx4:$I3),
4452:             mnemonic#"\t$BDL1, $BD2, $I3", []>;
4453: 
4454: class SideEffectTernaryRRFa<string mnemonic, bits<16> opcode,
4455:                             RegisterOperand cls1, RegisterOperand cls2,
```
- **EN**: This block declares or refines TableGen records such as `CompareVRRh`, `TestInherentS`, `TestRXE`, `TestBinarySIL`, `TestRSL`, `TestVRRg`.
- **CN**: 该代码块声明或细化了 `CompareVRRh`, `TestInherentS`, `TestRXE`, `TestBinarySIL`, `TestRSL`, `TestVRRg` 等 TableGen 记录。

### Lines 4456-4510 / 第 4456-4510 行
```tablegen
4456:                             RegisterOperand cls3>
4457:   : InstRRFa<opcode, (outs), (ins cls1:$R1, cls2:$R2, cls3:$R3),
4458:              mnemonic#"\t$R1, $R2, $R3", []> {
4459:   let M4 = 0;
4460: }
4461: 
4462: class SideEffectTernaryMemMemRRFa<string mnemonic, bits<16> opcode,
4463:                                   RegisterOperand cls1, RegisterOperand cls2,
4464:                                   RegisterOperand cls3>
4465:   : InstRRFa<opcode, (outs cls1:$R1, cls2:$R2),
4466:              (ins cls1:$R1src, cls2:$R2src, cls3:$R3),
4467:              mnemonic#"\t$R1, $R2, $R3", []> {
4468:   let Constraints = "$R1 = $R1src, $R2 = $R2src";
4469:   let M4 = 0;
4470: }
4471: 
4472: class SideEffectTernaryRRFb<string mnemonic, bits<16> opcode,
4473:                             RegisterOperand cls1, RegisterOperand cls2,
4474:                             RegisterOperand cls3>
4475:   : InstRRFb<opcode, (outs), (ins cls1:$R1, cls2:$R2, cls3:$R3),
4476:              mnemonic#"\t$R1, $R3, $R2", []> {
4477:   let M4 = 0;
4478: }
4479: 
4480: class SideEffectTernaryMemMemMemRRFb<string mnemonic, bits<16> opcode,
4481:                                      RegisterOperand cls1,
4482:                                      RegisterOperand cls2,
4483:                                      RegisterOperand cls3>
4484:   : InstRRFb<opcode, (outs cls1:$R1, cls2:$R2, cls3:$R3),
4485:              (ins cls1:$R1src, cls2:$R2src, cls3:$R3src),
4486:              mnemonic#"\t$R1, $R3, $R2", []> {
4487:   let Constraints = "$R1 = $R1src, $R2 = $R2src, $R3 = $R3src";
4488:   let M4 = 0;
4489: }
4490: 
4491: class SideEffectTernaryRRFc<string mnemonic, bits<16> opcode,
4492:                             RegisterOperand cls1, RegisterOperand cls2,
4493:                             ImmOpWithPattern imm>
4494:   : InstRRFc<opcode, (outs), (ins cls1:$R1, cls2:$R2, imm:$M3),
4495:              mnemonic#"\t$R1, $R2, $M3", []>;
4496: 
4497: multiclass SideEffectTernaryRRFcOpt<string mnemonic, bits<16> opcode,
4498:                                     RegisterOperand cls1,
4499:                                     RegisterOperand cls2> {
4500:   def "" : SideEffectTernaryRRFc<mnemonic, opcode, cls1, cls2, imm32zx4>;
4501:   def Opt : SideEffectBinaryRRFc<mnemonic, opcode, cls1, cls2>;
4502: }
4503: 
4504: class SideEffectTernaryMemMemRRFc<string mnemonic, bits<16> opcode,
4505:                                   RegisterOperand cls1, RegisterOperand cls2,
4506:                                   ImmOpWithPattern imm>
4507:   : InstRRFc<opcode, (outs cls1:$R1, cls2:$R2),
4508:              (ins cls1:$R1src, cls2:$R2src, imm:$M3),
4509:              mnemonic#"\t$R1, $R2, $M3", []> {
4510:   let Constraints = "$R1 = $R1src, $R2 = $R2src";
```
- **EN**: This block declares or refines TableGen records such as `SideEffectTernaryMemMemRRFa`, `SideEffectTernaryRRFb`, `SideEffectTernaryMemMemMemRRFb`, `SideEffectTernaryRRFc`, `SideEffectTernaryRRFcOpt`, `Opt`.
- **CN**: 该代码块声明或细化了 `SideEffectTernaryMemMemRRFa`, `SideEffectTernaryRRFb`, `SideEffectTernaryMemMemMemRRFb`, `SideEffectTernaryRRFc`, `SideEffectTernaryRRFcOpt`, `Opt` 等 TableGen 记录。

### Lines 4511-4565 / 第 4511-4565 行
```tablegen
4511: }
4512: 
4513: multiclass SideEffectTernaryMemMemRRFcOpt<string mnemonic, bits<16> opcode,
4514:                                           RegisterOperand cls1,
4515:                                           RegisterOperand cls2> {
4516:   def "" : SideEffectTernaryMemMemRRFc<mnemonic, opcode, cls1, cls2, imm32zx4>;
4517:   def Opt : SideEffectBinaryMemMemRRFc<mnemonic, opcode, cls1, cls2>;
4518: }
4519: 
4520: class SideEffectTernarySSF<string mnemonic, bits<12> opcode,
4521:                            RegisterOperand cls>
4522:   : InstSSF<opcode, (outs),
4523:             (ins (bdaddr12only $B1, $D1):$BD1,
4524:                  (bdaddr12only $B2, $D2):$BD2, cls:$R3),
4525:             mnemonic#"\t$BD1, $BD2, $R3", []>;
4526: 
4527: class TernaryRRFa<string mnemonic, bits<16> opcode,
4528:                  RegisterOperand cls1, RegisterOperand cls2,
4529:                  RegisterOperand cls3>
4530:   : InstRRFa<opcode, (outs cls1:$R1), (ins cls2:$R2, cls3:$R3, imm32zx4:$M4),
4531:              mnemonic#"\t$R1, $R2, $R3, $M4", []>;
4532: 
4533: class TernaryRRFb<string mnemonic, bits<16> opcode,
4534:                   RegisterOperand cls1, RegisterOperand cls2,
4535:                   RegisterOperand cls3>
4536:   : InstRRFb<opcode, (outs cls1:$R1, cls3:$R3),
4537:              (ins cls1:$R1src, cls2:$R2, imm32zx4:$M4),
4538:              mnemonic#"\t$R1, $R3, $R2, $M4", []> {
4539:   let Constraints = "$R1 = $R1src";
4540: }
4541: 
4542: class TernaryRRFe<string mnemonic, bits<16> opcode, RegisterOperand cls1,
4543:                   RegisterOperand cls2>
4544:   : InstRRFe<opcode, (outs cls1:$R1),
4545:              (ins imm32zx4:$M3, cls2:$R2, imm32zx4:$M4),
4546:              mnemonic#"\t$R1, $M3, $R2, $M4", []>;
4547: 
4548: class TernaryRRD<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4549:                  RegisterOperand cls1, RegisterOperand cls2>
4550:   : InstRRD<opcode, (outs cls1:$R1), (ins cls2:$R1src, cls2:$R3, cls2:$R2),
4551:             mnemonic#"\t$R1, $R3, $R2",
4552:             [(set cls1:$R1, (operator cls2:$R1src, cls2:$R3, cls2:$R2))]> {
4553:   let OpKey = mnemonic#cls;
4554:   let OpType = "reg";
4555:   let Constraints = "$R1 = $R1src";
4556: }
4557: 
4558: class TernaryRS<string mnemonic, bits<8> opcode, RegisterOperand cls,
4559:                 bits<5> bytes, AddressingMode mode = bdaddr12only>
4560:   : InstRSb<opcode, (outs cls:$R1),
4561:             (ins cls:$R1src, imm32zx4:$M3, (mode $B2, $D2):$BD2),
4562:             mnemonic#"\t$R1, $M3, $BD2", []> {
4563: 
4564:   let Constraints = "$R1 = $R1src";
4565:   let mayLoad = 1;
```
- **EN**: This block declares or refines TableGen records such as `SideEffectTernaryMemMemRRFcOpt`, `Opt`, `SideEffectTernarySSF`, `TernaryRRFa`, `TernaryRRFb`, `TernaryRRFe`.
- **CN**: 该代码块声明或细化了 `SideEffectTernaryMemMemRRFcOpt`, `Opt`, `SideEffectTernarySSF`, `TernaryRRFa`, `TernaryRRFb`, `TernaryRRFe` 等 TableGen 记录。

### Lines 4566-4620 / 第 4566-4620 行
```tablegen
4566:   let AccessBytes = bytes;
4567: }
4568: 
4569: class TernaryRSY<string mnemonic, bits<16> opcode, RegisterOperand cls,
4570:                 bits<5> bytes, AddressingMode mode = bdaddr20only>
4571:   : InstRSYb<opcode, (outs cls:$R1),
4572:              (ins cls:$R1src, imm32zx4:$M3, (mode $B2, $D2):$BD2),
4573:              mnemonic#"\t$R1, $M3, $BD2", []> {
4574: 
4575:   let Constraints = "$R1 = $R1src";
4576:   let mayLoad = 1;
4577:   let AccessBytes = bytes;
4578: }
4579: 
4580: multiclass TernaryRSPair<string mnemonic, bits<8> rsOpcode, bits<16> rsyOpcode,
4581:                          RegisterOperand cls, bits<5> bytes> {
4582:   let DispKey = mnemonic # cls in {
4583:     let DispSize = "12" in
4584:       def "" : TernaryRS<mnemonic, rsOpcode, cls, bytes, bdaddr12pair>;
4585:     let DispSize = "20" in
4586:       def Y  : TernaryRSY<mnemonic#"y", rsyOpcode, cls, bytes, bdaddr20pair>;
4587:   }
4588: }
4589: 
4590: class SideEffectTernaryRS<string mnemonic, bits<8> opcode,
4591:                           RegisterOperand cls1, RegisterOperand cls2>
4592:   : InstRSa<opcode, (outs),
4593:             (ins cls1:$R1, cls2:$R3, (bdaddr12only $B2, $D2):$BD2),
4594:             mnemonic#"\t$R1, $R3, $BD2", []>;
4595: 
4596: class SideEffectTernaryRSY<string mnemonic, bits<16> opcode,
4597:                            RegisterOperand cls1, RegisterOperand cls2>
4598:   : InstRSYa<opcode, (outs),
4599:              (ins cls1:$R1, cls2:$R3, (bdaddr20only $B2, $D2):$BD2),
4600:              mnemonic#"\t$R1, $R3, $BD2", []>;
4601: 
4602: class SideEffectTernaryMemMemRS<string mnemonic, bits<8> opcode,
4603:                                 RegisterOperand cls1, RegisterOperand cls2>
4604:   : InstRSa<opcode, (outs cls1:$R1, cls2:$R3),
4605:             (ins cls1:$R1src, cls2:$R3src, (shift12only $B2, $D2):$BD2),
4606:             mnemonic#"\t$R1, $R3, $BD2", []> {
4607:     let Constraints = "$R1 = $R1src, $R3 = $R3src";
4608: }
4609: 
4610: class SideEffectTernaryMemMemRSY<string mnemonic, bits<16> opcode,
4611:                                  RegisterOperand cls1, RegisterOperand cls2>
4612:   : InstRSYa<opcode, (outs cls1:$R1, cls2:$R3),
4613:              (ins cls1:$R1src, cls2:$R3src, (shift20only $B2, $D2):$BD2),
4614:              mnemonic#"\t$R1, $R3, $BD2", []> {
4615:     let Constraints = "$R1 = $R1src, $R3 = $R3src";
4616: }
4617: 
4618: class TernaryRXF<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4619:                  RegisterOperand cls1, RegisterOperand cls2,
4620:                  SDPatternOperator load, bits<5> bytes>
```
- **EN**: This block declares or refines TableGen records such as `TernaryRSY`, `TernaryRSPair`, `Y`, `SideEffectTernaryRS`, `SideEffectTernaryRSY`, `SideEffectTernaryMemMemRS`.
- **CN**: 该代码块声明或细化了 `TernaryRSY`, `TernaryRSPair`, `Y`, `SideEffectTernaryRS`, `SideEffectTernaryRSY`, `SideEffectTernaryMemMemRS` 等 TableGen 记录。

### Lines 4621-4675 / 第 4621-4675 行
```tablegen
4621:   : InstRXF<opcode, (outs cls1:$R1),
4622:             (ins cls2:$R1src, cls2:$R3, (bdxaddr12only $B2, $D2, $X2):$XBD2),
4623:             mnemonic#"\t$R1, $R3, $XBD2",
4624:             [(set cls1:$R1, (operator cls2:$R1src, cls2:$R3,
4625:                                       (load bdxaddr12only:$XBD2)))]> {
4626:   let OpKey = mnemonic#"r"#cls;
4627:   let OpType = "mem";
4628:   let Constraints = "$R1 = $R1src";
4629:   let mayLoad = 1;
4630:   let AccessBytes = bytes;
4631: }
4632: 
4633: class TernaryVRIa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4634:                   TypedReg tr1, TypedReg tr2, ImmOpWithPattern imm, ImmOpWithPattern index>
4635:   : InstVRIa<opcode, (outs tr1.op:$V1), (ins tr2.op:$V1src, imm:$I2, index:$M3),
4636:              mnemonic#"\t$V1, $I2, $M3",
4637:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V1src),
4638:                                                   imm:$I2, index:$M3))]> {
4639:   let Constraints = "$V1 = $V1src";
4640: }
4641: 
4642: class TernaryVRId<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4643:                   TypedReg tr1, TypedReg tr2, bits<4> type>
4644:   : InstVRId<opcode, (outs tr1.op:$V1),
4645:              (ins tr2.op:$V2, tr2.op:$V3, imm32zx8:$I4),
4646:              mnemonic#"\t$V1, $V2, $V3, $I4",
4647:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4648:                                                   (tr2.vt tr2.op:$V3),
4649:                                                   imm32zx8_timm:$I4))]> {
4650:   let M5 = type;
4651: }
4652: 
4653: class TernaryVRIi<string mnemonic, bits<16> opcode, RegisterOperand cls>
4654:   : InstVRIi<opcode, (outs VR128:$V1),
4655:              (ins cls:$R2, imm32zx8:$I3, imm32zx4:$M4),
4656:              mnemonic#"\t$V1, $R2, $I3, $M4", []>;
4657: 
4658: class TernaryVRIj<string mnemonic, bits<16> opcode>
4659:   : InstVRIj<opcode, (outs VR128:$V1),
4660:              (ins VR128:$V2, imm32zx8:$I3, imm32zx4:$M4),
4661:              mnemonic#"\t$V1, $V2, $I3, $M4", []>;
4662: 
4663: class TernaryVRRa<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4664:                   TypedReg tr1, TypedReg tr2, bits<4> type, bits<4> m4or>
4665:   : InstVRRa<opcode, (outs tr1.op:$V1),
4666:              (ins tr2.op:$V2, imm32zx4:$M4, imm32zx4:$M5),
4667:              mnemonic#"\t$V1, $V2, $M4, $M5",
4668:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4669:                                                   imm32zx4_timm:$M4,
4670:                                                   imm32zx4_timm:$M5))],
4671:              m4or> {
4672:   let M3 = type;
4673: }
4674: 
4675: class TernaryVRRaFloatGeneric<string mnemonic, bits<16> opcode>
```
- **EN**: This block declares or refines TableGen records such as `TernaryVRIa`, `TernaryVRId`, `TernaryVRIi`, `TernaryVRIj`, `TernaryVRRa`, `TernaryVRRaFloatGeneric`.
- **CN**: 该代码块声明或细化了 `TernaryVRIa`, `TernaryVRId`, `TernaryVRIi`, `TernaryVRIj`, `TernaryVRRa`, `TernaryVRRaFloatGeneric` 等 TableGen 记录。

### Lines 4676-4730 / 第 4676-4730 行
```tablegen
4676:   : InstVRRa<opcode, (outs VR128:$V1),
4677:              (ins VR128:$V2, imm32zx4:$M3, imm32zx4:$M4, imm32zx4:$M5),
4678:              mnemonic#"\t$V1, $V2, $M3, $M4, $M5", []>;
4679: 
4680: class TernaryVRRb<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4681:                   TypedReg tr1, TypedReg tr2, bits<4> type,
4682:                   SDPatternOperator m5mask, bits<4> m5or>
4683:   : InstVRRb<opcode, (outs tr1.op:$V1),
4684:              (ins tr2.op:$V2, tr2.op:$V3, m5mask:$M5),
4685:              mnemonic#"\t$V1, $V2, $V3, $M5",
4686:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4687:                                                   (tr2.vt tr2.op:$V3),
4688:                                                   m5mask:$M5))],
4689:              m5or> {
4690:   let M4 = type;
4691: }
4692: 
4693: // Declare a pair of instructions, one which sets CC and one which doesn't.
4694: // The CC-setting form ends with "S" and sets the low bit of M5.
4695: // Also create aliases to make use of M5 operand optional in assembler.
4696: multiclass TernaryOptVRRbSPair<string mnemonic, bits<16> opcode,
4697:                                SDPatternOperator operator,
4698:                                SDPatternOperator operator_cc,
4699:                                TypedReg tr1, TypedReg tr2, bits<4> type,
4700:                                bits<4> modifier = 0> {
4701:   def "" : TernaryVRRb<mnemonic, opcode, operator, tr1, tr2, type,
4702:                        imm32zx4even_timm, !and (modifier, 14)>;
4703:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3",
4704:                   (!cast<Instruction>(NAME) tr1.op:$V1, tr2.op:$V2,
4705:                                             tr2.op:$V3, 0)>;
4706:   let Defs = [CC] in
4707:     def S : TernaryVRRb<mnemonic#"s", opcode, operator_cc, tr1, tr2, type,
4708:                         imm32zx4even_timm, !add(!and (modifier, 14), 1)>;
4709:   def : InstAlias<mnemonic#"s\t$V1, $V2, $V3",
4710:                   (!cast<Instruction>(NAME#"S") tr1.op:$V1, tr2.op:$V2,
4711:                                                 tr2.op:$V3, 0)>;
4712: }
4713: 
4714: multiclass TernaryOptVRRbSPairGeneric<string mnemonic, bits<16> opcode> {
4715:   let Defs = [CC] in
4716:     def "" : InstVRRb<opcode, (outs VR128:$V1),
4717:                      (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5),
4718:                      mnemonic#"\t$V1, $V2, $V3, $M4, $M5", []>;
4719:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3, $M4",
4720:                   (!cast<Instruction>(NAME) VR128:$V1, VR128:$V2, VR128:$V3,
4721:                                             imm32zx4:$M4, 0)>;
4722: }
4723: 
4724: class TernaryVRRc<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4725:                   TypedReg tr1, TypedReg tr2>
4726:   : InstVRRc<opcode, (outs tr1.op:$V1),
4727:              (ins tr2.op:$V2, tr2.op:$V3, imm32zx4:$M4),
4728:              mnemonic#"\t$V1, $V2, $V3, $M4",
4729:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4730:                                                   (tr2.vt tr2.op:$V3),
```
- **EN**: This block declares or refines TableGen records such as `TernaryVRRb`, `TernaryOptVRRbSPair`, `S`, `TernaryOptVRRbSPairGeneric`, `TernaryVRRc`.
- **CN**: 该代码块声明或细化了 `TernaryVRRb`, `TernaryOptVRRbSPair`, `S`, `TernaryOptVRRbSPairGeneric`, `TernaryVRRc` 等 TableGen 记录。

### Lines 4731-4785 / 第 4731-4785 行
```tablegen
4731:                                                   imm32zx4_timm:$M4))]> {
4732:   let M5 = 0;
4733:   let M6 = 0;
4734: }
4735: 
4736: class TernaryVRRcInt<string mnemonic, bits<16> opcode,
4737:                      SDPatternOperator operator, TypedReg tr1, TypedReg tr2,
4738:                      bits<4> type = 0>
4739:   : InstVRRc<opcode, (outs tr1.op:$V1),
4740:              (ins tr2.op:$V2, tr2.op:$V3, imm32zx4:$M5),
4741:              mnemonic#"\t$V1, $V2, $V3, $M5",
4742:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4743:                                                   (tr2.vt tr2.op:$V3),
4744:                                                   imm32zx4_timm:$M5))]> {
4745:   let M4 = type;
4746:   let M6 = 0;
4747: }
4748: 
4749: class TernaryVRRcIntGeneric<string mnemonic, bits<16> opcode>
4750:   : InstVRRc<opcode, (outs VR128:$V1),
4751:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5),
4752:              mnemonic#"\t$V1, $V2, $V3, $M4, $M5", []> {
4753:   let M6 = 0;
4754: }
4755: 
4756: class TernaryVRRcFloat<string mnemonic, bits<16> opcode,
4757:                        SDPatternOperator operator, TypedReg tr1, TypedReg tr2,
4758:                        bits<4> type = 0, bits<4> m5 = 0>
4759:   : InstVRRc<opcode, (outs tr1.op:$V1),
4760:              (ins tr2.op:$V2, tr2.op:$V3, imm32zx4:$M6),
4761:              mnemonic#"\t$V1, $V2, $V3, $M6",
4762:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4763:                                                   (tr2.vt tr2.op:$V3),
4764:                                                   imm32zx4_timm:$M6))]> {
4765:   let M4 = type;
4766:   let M5 = m5;
4767: }
4768: 
4769: class TernaryVRRcFloatGeneric<string mnemonic, bits<16> opcode>
4770:   : InstVRRc<opcode, (outs VR128:$V1),
4771:              (ins VR128:$V2, VR128:$V3, imm32zx4:$M4, imm32zx4:$M5,
4772:                   imm32zx4:$M6),
4773:              mnemonic#"\t$V1, $V2, $V3, $M4, $M5, $M6", []>;
4774: 
4775: class TernaryVRRd<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4776:                   TypedReg tr1, TypedReg tr2, bits<4> type = 0, bits<4> m6 = 0>
4777:   : InstVRRd<opcode, (outs tr1.op:$V1),
4778:              (ins tr2.op:$V2, tr2.op:$V3, tr1.op:$V4),
4779:              mnemonic#"\t$V1, $V2, $V3, $V4",
4780:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4781:                                                   (tr2.vt tr2.op:$V3),
4782:                                                   (tr1.vt tr1.op:$V4)))]> {
4783:   let M5 = type;
4784:   let M6 = m6;
4785: }
```
- **EN**: This block declares or refines TableGen records such as `TernaryVRRcInt`, `TernaryVRRcIntGeneric`, `TernaryVRRcFloat`, `TernaryVRRcFloatGeneric`, `TernaryVRRd`.
- **CN**: 该代码块声明或细化了 `TernaryVRRcInt`, `TernaryVRRcIntGeneric`, `TernaryVRRcFloat`, `TernaryVRRcFloatGeneric`, `TernaryVRRd` 等 TableGen 记录。

### Lines 4786-4840 / 第 4786-4840 行
```tablegen
4786: 
4787: class TernaryVRRdGeneric<string mnemonic, bits<16> opcode>
4788:   : InstVRRd<opcode, (outs VR128:$V1),
4789:              (ins VR128:$V2, VR128:$V3, VR128:$V4, imm32zx4:$M5),
4790:              mnemonic#"\t$V1, $V2, $V3, $V4, $M5", []> {
4791:   let M6 = 0;
4792: }
4793: 
4794: // Ternary operation where the assembler mnemonic has an extra operand to
4795: // optionally allow specifying arbitrary M6 values.
4796: multiclass TernaryExtraVRRd<string mnemonic, bits<16> opcode,
4797:                              SDPatternOperator operator,
4798:                              TypedReg tr1, TypedReg tr2, bits<4> type> {
4799:   let M5 = type, Defs = [CC] in
4800:     def "" : InstVRRd<opcode, (outs tr1.op:$V1),
4801:                       (ins tr2.op:$V2, tr2.op:$V3, tr1.op:$V4, imm32zx4:$M6),
4802:                       mnemonic#"\t$V1, $V2, $V3, $V4, $M6", []>;
4803:   def : Pat<(operator (tr2.vt tr2.op:$V2), (tr2.vt tr2.op:$V3),
4804:                       (tr1.vt tr1.op:$V4)),
4805:             (!cast<Instruction>(NAME) tr2.op:$V2, tr2.op:$V3, tr1.op:$V4, 0)>;
4806:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3, $V4",
4807:                   (!cast<Instruction>(NAME) tr1.op:$V1, tr2.op:$V2,
4808:                                             tr2.op:$V3, tr1.op:$V4, 0)>;
4809: }
4810: 
4811: multiclass TernaryExtraVRRdGeneric<string mnemonic, bits<16> opcode> {
4812:   let Defs = [CC] in
4813:     def "" : InstVRRd<opcode, (outs VR128:$V1),
4814:                       (ins VR128:$V2, VR128:$V3, VR128:$V4,
4815:                        imm32zx4:$M5, imm32zx4:$M6),
4816:                       mnemonic#"\t$V1, $V2, $V3, $V4, $M5, $M6", []>;
4817:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3, $V4, $M5",
4818:                   (!cast<Instruction>(NAME) VR128:$V1, VR128:$V2, VR128:$V3,
4819:                                             VR128:$V4, imm32zx4:$M5, 0)>;
4820: }
4821: 
4822: class TernaryVRRe<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4823:                   TypedReg tr1, TypedReg tr2, bits<4> m5 = 0, bits<4> type = 0,
4824:                   string fp_mnemonic = "">
4825:   : InstVRRe<opcode, (outs tr1.op:$V1),
4826:              (ins tr2.op:$V2, tr2.op:$V3, tr1.op:$V4),
4827:              mnemonic#"\t$V1, $V2, $V3, $V4",
4828:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4829:                                                   (tr2.vt tr2.op:$V3),
4830:                                                   (tr1.vt tr1.op:$V4)))]> {
4831:   let M5 = m5;
4832:   let M6 = type;
4833:   let OpKey = fp_mnemonic#"MemFold"#!subst("VR", "FP", !cast<string>(tr1.op));
4834:   let OpType = "reg";
4835: }
4836: 
4837: class TernaryVRReFloatGeneric<string mnemonic, bits<16> opcode>
4838:   : InstVRRe<opcode, (outs VR128:$V1),
4839:              (ins VR128:$V2, VR128:$V3, VR128:$V4, imm32zx4:$M5, imm32zx4:$M6),
4840:              mnemonic#"\t$V1, $V2, $V3, $V4, $M5, $M6", []>;
```
- **EN**: This block declares or refines TableGen records such as `TernaryVRRdGeneric`, `TernaryExtraVRRd`, `TernaryExtraVRRdGeneric`, `TernaryVRRe`, `TernaryVRReFloatGeneric`.
- **CN**: 该代码块声明或细化了 `TernaryVRRdGeneric`, `TernaryExtraVRRd`, `TernaryExtraVRRdGeneric`, `TernaryVRRe`, `TernaryVRReFloatGeneric` 等 TableGen 记录。

### Lines 4841-4895 / 第 4841-4895 行
```tablegen
4841: 
4842: class TernaryVRSb<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4843:                   TypedReg tr1, TypedReg tr2, RegisterOperand cls, bits<4> type>
4844:   : InstVRSb<opcode, (outs tr1.op:$V1),
4845:              (ins tr2.op:$V1src, cls:$R3, (shift12only $B2, $D2):$BD2),
4846:              mnemonic#"\t$V1, $R3, $BD2",
4847:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V1src),
4848:                                                   cls:$R3,
4849:                                                   shift12only:$BD2))]> {
4850:   let Constraints = "$V1 = $V1src";
4851:   let M4 = type;
4852: }
4853: 
4854: class TernaryVRRi<string mnemonic, bits<16> opcode, RegisterOperand cls>
4855:   : InstVRRi<opcode, (outs cls:$R1), (ins VR128:$V2,
4856:                                       imm32zx4:$M3, imm32zx4:$M4),
4857:              mnemonic#"\t$R1, $V2, $M3, $M4", []>;
4858: 
4859: class TernaryVRRj<string mnemonic, bits<16> opcode>
4860:   : InstVRRj<opcode, (outs VR128:$V1), (ins VR128:$V2,
4861:                                         VR128:$V3, imm32zx4:$M4),
4862:              mnemonic#"\t$V1, $V2, $V3, $M4", []>;
4863: 
4864: class TernaryVRSbGeneric<string mnemonic, bits<16> opcode>
4865:   : InstVRSb<opcode, (outs VR128:$V1),
4866:              (ins VR128:$V1src, GR64:$R3, (shift12only $B2, $D2):$BD2,
4867:                   imm32zx4:$M4),
4868:              mnemonic#"\t$V1, $R3, $BD2, $M4", []> {
4869:   let Constraints = "$V1 = $V1src";
4870: }
4871: 
4872: class TernaryVRV<string mnemonic, bits<16> opcode, bits<5> bytes,
4873:                  ImmOpWithPattern index>
4874:   : InstVRV<opcode, (outs VR128:$V1),
4875:            (ins VR128:$V1src, (bdvaddr12only $B2, $D2, $V2):$VBD2, index:$M3),
4876:            mnemonic#"\t$V1, $VBD2, $M3", []> {
4877:   let Constraints = "$V1 = $V1src";
4878:   let mayLoad = 1;
4879:   let AccessBytes = bytes;
4880: }
4881: 
4882: class TernaryVRX<string mnemonic, bits<16> opcode, SDPatternOperator operator,
4883:                  TypedReg tr1, TypedReg tr2, bits<5> bytes, ImmOpWithPattern index>
4884:   : InstVRX<opcode, (outs tr1.op:$V1),
4885:            (ins tr2.op:$V1src, (bdxaddr12only $B2, $D2, $X2):$XBD2, index:$M3),
4886:            mnemonic#"\t$V1, $XBD2, $M3",
4887:            [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V1src),
4888:                                                 bdxaddr12only:$XBD2,
4889:                                                 index:$M3))]> {
4890:   let Constraints = "$V1 = $V1src";
4891:   let mayLoad = 1;
4892:   let AccessBytes = bytes;
4893: }
4894: 
4895: class QuaternaryVRId<string mnemonic, bits<16> opcode, SDPatternOperator operator,
```
- **EN**: This block declares or refines TableGen records such as `TernaryVRSb`, `TernaryVRRi`, `TernaryVRRj`, `TernaryVRSbGeneric`, `TernaryVRV`, `TernaryVRX`.
- **CN**: 该代码块声明或细化了 `TernaryVRSb`, `TernaryVRRi`, `TernaryVRRj`, `TernaryVRSbGeneric`, `TernaryVRV`, `TernaryVRX` 等 TableGen 记录。

### Lines 4896-4950 / 第 4896-4950 行
```tablegen
4896:                      TypedReg tr1, TypedReg tr2, bits<4> type>
4897:   : InstVRId<opcode, (outs tr1.op:$V1),
4898:              (ins tr2.op:$V1src, tr2.op:$V2, tr2.op:$V3, imm32zx8:$I4),
4899:              mnemonic#"\t$V1, $V2, $V3, $I4",
4900:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V1src),
4901:                                                   (tr2.vt tr2.op:$V2),
4902:                                                   (tr2.vt tr2.op:$V3),
4903:                                                   imm32zx8_timm:$I4))]> {
4904:   let Constraints = "$V1 = $V1src";
4905:   let M5 = type;
4906: }
4907: 
4908: class QuaternaryVRIdGeneric<string mnemonic, bits<16> opcode>
4909:   : InstVRId<opcode, (outs VR128:$V1),
4910:              (ins VR128:$V1src, VR128:$V2, VR128:$V3,
4911:                   imm32zx8:$I4, imm32zx4:$M5),
4912:              mnemonic#"\t$V1, $V2, $V3, $I4, $M5", []> {
4913:   let Constraints = "$V1 = $V1src";
4914: }
4915: 
4916: class QuaternaryVRIf<string mnemonic, bits<16> opcode>
4917:   : InstVRIf<opcode, (outs VR128:$V1),
4918:              (ins VR128:$V2, VR128:$V3,
4919:                   imm32zx8:$I4, imm32zx4:$M5),
4920:             mnemonic#"\t$V1, $V2, $V3, $I4, $M5", []>;
4921: 
4922: class QuaternaryVRIg<string mnemonic, bits<16> opcode>
4923:   : InstVRIg<opcode, (outs VR128:$V1),
4924:              (ins VR128:$V2, imm32zx8:$I3,
4925:                   imm32zx8:$I4, imm32zx4:$M5),
4926:              mnemonic#"\t$V1, $V2, $I3, $I4, $M5", []>;
4927: 
4928: class QuaternaryVRIk<string mnemonic, bits<16> opcode,
4929:                      SDPatternOperator operator, TypedReg tr>
4930:   : InstVRIk<opcode, (outs VR128:$V1),
4931:              (ins VR128:$V2, VR128:$V3, VR128:$V4, imm32zx8:$I5),
4932:              mnemonic#"\t$V1, $V2, $V3, $V4, $I5",
4933:              [(set (tr.vt tr.op:$V1), (operator (tr.vt tr.op:$V2),
4934:                                                 (tr.vt tr.op:$V3),
4935:                                                 (tr.vt tr.op:$V4),
4936:                                                 imm32zx8_timm:$I5))]>;
4937: 
4938: class QuaternaryVRRd<string mnemonic, bits<16> opcode,
4939:                      SDPatternOperator operator, TypedReg tr1, TypedReg tr2,
4940:                      TypedReg tr3, TypedReg tr4, bits<4> type,
4941:                      SDPatternOperator m6mask = imm32zx4_timm, bits<4> m6or = 0>
4942:   : InstVRRd<opcode, (outs tr1.op:$V1),
4943:              (ins tr2.op:$V2, tr3.op:$V3, tr4.op:$V4, m6mask:$M6),
4944:              mnemonic#"\t$V1, $V2, $V3, $V4, $M6",
4945:              [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2),
4946:                                                   (tr3.vt tr3.op:$V3),
4947:                                                   (tr4.vt tr4.op:$V4),
4948:                                                   m6mask:$M6))],
4949:              m6or> {
4950:   let M5 = type;
```
- **EN**: This block declares or refines TableGen records such as `QuaternaryVRIdGeneric`, `QuaternaryVRIf`, `QuaternaryVRIg`, `QuaternaryVRIk`, `QuaternaryVRRd`.
- **CN**: 该代码块声明或细化了 `QuaternaryVRIdGeneric`, `QuaternaryVRIf`, `QuaternaryVRIg`, `QuaternaryVRIk`, `QuaternaryVRRd` 等 TableGen 记录。

### Lines 4951-5005 / 第 4951-5005 行
```tablegen
4951: }
4952: 
4953: class QuaternaryVRRdGeneric<string mnemonic, bits<16> opcode>
4954:   : InstVRRd<opcode, (outs VR128:$V1),
4955:              (ins VR128:$V2, VR128:$V3, VR128:$V4, imm32zx4:$M5, imm32zx4:$M6),
4956:              mnemonic#"\t$V1, $V2, $V3, $V4, $M5, $M6", []>;
4957: 
4958: // Declare a pair of instructions, one which sets CC and one which doesn't.
4959: // The CC-setting form ends with "S" and sets the low bit of M6.
4960: // Also create aliases to make use of M6 operand optional in assembler.
4961: multiclass QuaternaryOptVRRdSPair<string mnemonic, bits<16> opcode,
4962:                                   SDPatternOperator operator,
4963:                                 SDPatternOperator operator_cc,
4964:                                 TypedReg tr1, TypedReg tr2, bits<4> type,
4965:                                 bits<4> modifier = 0> {
4966:   def "" : QuaternaryVRRd<mnemonic, opcode, operator,
4967:                           tr1, tr2, tr2, tr2, type,
4968:                           imm32zx4even_timm, !and (modifier, 14)>;
4969:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3, $V4",
4970:                   (!cast<Instruction>(NAME) tr1.op:$V1, tr2.op:$V2,
4971:                                             tr2.op:$V3, tr2.op:$V4, 0)>;
4972:   let Defs = [CC] in
4973:     def S : QuaternaryVRRd<mnemonic#"s", opcode, operator_cc,
4974:                            tr1, tr2, tr2, tr2, type,
4975:                            imm32zx4even_timm, !add (!and (modifier, 14), 1)>;
4976:   def : InstAlias<mnemonic#"s\t$V1, $V2, $V3, $V4",
4977:                   (!cast<Instruction>(NAME#"S") tr1.op:$V1, tr2.op:$V2,
4978:                                                 tr2.op:$V3, tr2.op:$V4, 0)>;
4979: }
4980: 
4981: multiclass QuaternaryOptVRRdSPairGeneric<string mnemonic, bits<16> opcode> {
4982:   let Defs = [CC] in
4983:     def "" : QuaternaryVRRdGeneric<mnemonic, opcode>;
4984:   def : InstAlias<mnemonic#"\t$V1, $V2, $V3, $V4, $M5",
4985:                   (!cast<Instruction>(NAME) VR128:$V1, VR128:$V2, VR128:$V3,
4986:                                             VR128:$V4, imm32zx4_timm:$M5, 0)>;
4987: }
4988: 
4989: class SideEffectQuaternaryRRFa<string mnemonic, bits<16> opcode,
4990:                                RegisterOperand cls1, RegisterOperand cls2,
4991:                                RegisterOperand cls3>
4992:   : InstRRFa<opcode, (outs), (ins cls1:$R1, cls2:$R2, cls3:$R3, imm32zx4:$M4),
4993:              mnemonic#"\t$R1, $R2, $R3, $M4", []>;
4994: 
4995: multiclass SideEffectQuaternaryRRFaOptOpt<string mnemonic, bits<16> opcode,
4996:                                           RegisterOperand cls1,
4997:                                           RegisterOperand cls2,
4998:                                           RegisterOperand cls3> {
4999:   def "" : SideEffectQuaternaryRRFa<mnemonic, opcode, cls1, cls2, cls3>;
5000:   def Opt : SideEffectTernaryRRFa<mnemonic, opcode, cls1, cls2, cls3>;
5001:   def OptOpt : SideEffectBinaryRRFa<mnemonic, opcode, cls1, cls2>;
5002: }
5003: 
5004: class SideEffectQuaternaryRRFb<string mnemonic, bits<16> opcode,
5005:                                RegisterOperand cls1, RegisterOperand cls2,
```
- **EN**: This block declares or refines TableGen records such as `QuaternaryVRRdGeneric`, `QuaternaryOptVRRdSPair`, `S`, `QuaternaryOptVRRdSPairGeneric`, `SideEffectQuaternaryRRFa`, `SideEffectQuaternaryRRFaOptOpt`.
- **CN**: 该代码块声明或细化了 `QuaternaryVRRdGeneric`, `QuaternaryOptVRRdSPair`, `S`, `QuaternaryOptVRRdSPairGeneric`, `SideEffectQuaternaryRRFa`, `SideEffectQuaternaryRRFaOptOpt` 等 TableGen 记录。

### Lines 5006-5060 / 第 5006-5060 行
```tablegen
5006:                                RegisterOperand cls3>
5007:   : InstRRFb<opcode, (outs), (ins cls1:$R1, cls2:$R2, cls3:$R3, imm32zx4:$M4),
5008:              mnemonic#"\t$R1, $R3, $R2, $M4", []>;
5009: 
5010: multiclass SideEffectQuaternaryRRFbOpt<string mnemonic, bits<16> opcode,
5011:                                        RegisterOperand cls1,
5012:                                        RegisterOperand cls2,
5013:                                        RegisterOperand cls3> {
5014:   def "" : SideEffectQuaternaryRRFb<mnemonic, opcode, cls1, cls2, cls3>;
5015:   def Opt : SideEffectTernaryRRFb<mnemonic, opcode, cls1, cls2, cls3>;
5016: }
5017: 
5018: class SideEffectQuaternarySSe<string mnemonic, bits<8> opcode,
5019:                               RegisterOperand cls>
5020:   : InstSSe<opcode, (outs),
5021:             (ins cls:$R1, (bdaddr12only $B2, $D2):$BD2, cls:$R3,
5022:                  (bdaddr12only $B4, $D4):$BD4),
5023:             mnemonic#"\t$R1, $BD2, $R3, $BD4", []>;
5024: 
5025: class LoadAndOpRSY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
5026:                   RegisterOperand cls, AddressingMode mode = bdaddr20only>
5027:   : InstRSYa<opcode, (outs cls:$R1), (ins cls:$R3, (mode $B2, $D2):$BD2),
5028:              mnemonic#"\t$R1, $R3, $BD2",
5029:              [(set cls:$R1, (operator mode:$BD2, cls:$R3))]> {
5030:   let mayLoad = 1;
5031:   let mayStore = 1;
5032: }
5033: 
5034: class CmpSwapRRE<string mnemonic, bits<16> opcode,
5035:                  RegisterOperand cls1, RegisterOperand cls2>
5036:   : InstRRE<opcode, (outs cls1:$R1), (ins cls1:$R1src, cls2:$R2),
5037:             mnemonic#"\t$R1, $R2", []> {
5038:   let Constraints = "$R1 = $R1src";
5039:   let mayLoad = 1;
5040:   let mayStore = 1;
5041: }
5042: 
5043: class CmpSwapRS<string mnemonic, bits<8> opcode, SDPatternOperator operator,
5044:                 RegisterOperand cls, AddressingMode mode = bdaddr12only>
5045:   : InstRSa<opcode, (outs cls:$R1),
5046:             (ins cls:$R1src, cls:$R3, (mode $B2, $D2):$BD2),
5047:             mnemonic#"\t$R1, $R3, $BD2",
5048:             [(set cls:$R1, (operator mode:$BD2, cls:$R1src, cls:$R3))]> {
5049:   let Constraints = "$R1 = $R1src";
5050:   let mayLoad = 1;
5051:   let mayStore = 1;
5052: }
5053: 
5054: class CmpSwapRSY<string mnemonic, bits<16> opcode, SDPatternOperator operator,
5055:                  RegisterOperand cls, AddressingMode mode = bdaddr20only>
5056:   : InstRSYa<opcode, (outs cls:$R1),
5057:              (ins cls:$R1src, cls:$R3, (mode $B2, $D2):$BD2),
5058:              mnemonic#"\t$R1, $R3, $BD2",
5059:              [(set cls:$R1, (operator mode:$BD2, cls:$R1src, cls:$R3))]> {
5060:   let Constraints = "$R1 = $R1src";
```
- **EN**: This block declares or refines TableGen records such as `SideEffectQuaternaryRRFbOpt`, `Opt`, `SideEffectQuaternarySSe`, `LoadAndOpRSY`, `CmpSwapRRE`, `CmpSwapRS`.
- **CN**: 该代码块声明或细化了 `SideEffectQuaternaryRRFbOpt`, `Opt`, `SideEffectQuaternarySSe`, `LoadAndOpRSY`, `CmpSwapRRE`, `CmpSwapRS` 等 TableGen 记录。

### Lines 5061-5115 / 第 5061-5115 行
```tablegen
5061:   let mayLoad = 1;
5062:   let mayStore = 1;
5063: }
5064: 
5065: multiclass CmpSwapRSPair<string mnemonic, bits<8> rsOpcode, bits<16> rsyOpcode,
5066:                          SDPatternOperator operator, RegisterOperand cls> {
5067:   let DispKey = mnemonic # cls in {
5068:     let DispSize = "12" in
5069:       def "" : CmpSwapRS<mnemonic, rsOpcode, operator, cls, bdaddr12pair>;
5070:     let DispSize = "20" in
5071:       def Y  : CmpSwapRSY<mnemonic#"y", rsyOpcode, operator, cls, bdaddr20pair>;
5072:   }
5073: }
5074: 
5075: multiclass RotateSelectRIEf<string mnemonic, bits<16> opcode, RegisterOperand cls1,
5076:                        RegisterOperand cls2, bits<8> I3Or = 0, bits<8> I4Or = 0> {
5077:   let Constraints = "$R1 = $R1src" in {
5078:   def "" : InstRIEf<opcode, (outs cls1:$R1),
5079:              (ins cls1:$R1src, cls2:$R2, imm32zx8:$I3, imm32zx8:$I4,
5080:                   imm32zx8:$I5),
5081:              mnemonic#"\t$R1, $R2, $I3, $I4, $I5", [], I3Or, I4Or>;
5082:   let I5 = 0 in
5083:     def Opt : InstRIEf<opcode, (outs cls1:$R1),
5084:                (ins cls1:$R1src, cls2:$R2, imm32zx8:$I3, imm32zx8:$I4),
5085:                mnemonic#"\t$R1, $R2, $I3, $I4", [], I3Or, I4Or>;
5086:   }
5087: }
5088: 
5089: class PrefetchRXY<string mnemonic, bits<16> opcode, SDPatternOperator operator>
5090:   : InstRXYb<opcode, (outs),
5091:              (ins imm32zx4:$M1, (bdxaddr20only $B2, $D2, $X2):$XBD2),
5092:              mnemonic#"\t$M1, $XBD2",
5093:              [(operator imm32zx4_timm:$M1, bdxaddr20only:$XBD2)]>;
5094: 
5095: class PrefetchRILPC<string mnemonic, bits<12> opcode,
5096:                     SDPatternOperator operator>
5097:   : InstRILc<opcode, (outs), (ins imm32zx4_timm:$M1, pcrel32:$RI2),
5098:              mnemonic#"\t$M1, $RI2",
5099:              [(operator imm32zx4_timm:$M1, pcrel32:$RI2)]> {
5100:   // We want PC-relative addresses to be tried ahead of BD and BDX addresses.
5101:   // However, BDXs have two extra operands and are therefore 6 units more
5102:   // complex.
5103:   let AddedComplexity = 7;
5104: }
5105: 
5106: class BranchPreloadSMI<string mnemonic, bits<8> opcode>
5107:   : InstSMI<opcode, (outs),
5108:             (ins imm32zx4:$M1, brtarget16bpp:$RI2,
5109:                  (bdaddr12only $B3, $D3):$BD3),
5110:             mnemonic#"\t$M1, $RI2, $BD3", []>;
5111: 
5112: class BranchPreloadMII<string mnemonic, bits<8> opcode>
5113:   : InstMII<opcode, (outs),
5114:             (ins imm32zx4:$M1, brtarget12bpp:$RI2, brtarget24bpp:$RI3),
5115:             mnemonic#"\t$M1, $RI2, $RI3", []>;
```
- **EN**: This block declares or refines TableGen records such as `CmpSwapRSPair`, `Y`, `RotateSelectRIEf`, `Opt`, `PrefetchRXY`, `PrefetchRILPC`.
- **CN**: 该代码块声明或细化了 `CmpSwapRSPair`, `Y`, `RotateSelectRIEf`, `Opt`, `PrefetchRXY`, `PrefetchRILPC` 等 TableGen 记录。

### Lines 5116-5170 / 第 5116-5170 行
```tablegen
5116: 
5117: //===----------------------------------------------------------------------===//
5118: // Pseudo instructions
5119: //===----------------------------------------------------------------------===//
5120: //
5121: // Convenience instructions that get lowered to real instructions
5122: // by either SystemZTargetLowering::EmitInstrWithCustomInserter()
5123: // or SystemZInstrInfo::expandPostRAPseudo().
5124: //
5125: //===----------------------------------------------------------------------===//
5126: 
5127: class Pseudo<dag outs, dag ins, list<dag> pattern>
5128:   : InstSystemZ<0, outs, ins, "", pattern> {
5129:   let isPseudo = 1;
5130:   let isCodeGenOnly = 1;
5131: }
5132: 
5133: // Like UnaryRI, but expanded after RA depending on the choice of register.
5134: class UnaryRIPseudo<SDPatternOperator operator, RegisterOperand cls,
5135:                     ImmOpWithPattern imm>
5136:   : Pseudo<(outs cls:$R1), (ins imm:$I2),
5137:            [(set cls:$R1, (operator imm:$I2))]>;
5138: 
5139: // Like UnaryRXY, but expanded after RA depending on the choice of register.
5140: class UnaryRXYPseudo<string key, SDPatternOperator operator,
5141:                      RegisterOperand cls, bits<5> bytes,
5142:                      AddressingMode mode = bdxaddr20only>
5143:   : Pseudo<(outs cls:$R1), (ins (mode $B2, $D2, $X2):$XBD2),
5144:            [(set cls:$R1, (operator mode:$XBD2))]> {
5145:   let OpKey = key#"r"#cls;
5146:   let OpType = "mem";
5147:   let mayLoad = 1;
5148:   let Has20BitOffset = 1;
5149:   let HasIndex = 1;
5150:   let AccessBytes = bytes;
5151: }
5152: 
5153: // Like UnaryRR, but expanded after RA depending on the choice of registers.
5154: class UnaryRRPseudo<string key, SDPatternOperator operator,
5155:                     RegisterOperand cls1, RegisterOperand cls2>
5156:   : Pseudo<(outs cls1:$R1), (ins cls2:$R2),
5157:            [(set cls1:$R1, (operator cls2:$R2))]> {
5158:   let OpKey = key#cls1;
5159:   let OpType = "reg";
5160: }
5161: 
5162: // Like BinaryRI, but expanded after RA depending on the choice of register.
5163: class BinaryRIPseudo<SDPatternOperator operator, RegisterOperand cls,
5164:                      ImmOpWithPattern imm>
5165:   : Pseudo<(outs cls:$R1), (ins cls:$R1src, imm:$I2),
5166:            [(set cls:$R1, (operator cls:$R1src, imm:$I2))]> {
5167:   let Constraints = "$R1 = $R1src";
5168: }
5169: 
5170: // Like BinaryRIE, but expanded after RA depending on the choice of register.
```
- **EN**: This block declares or refines TableGen records such as `Pseudo`, `UnaryRIPseudo`, `UnaryRXYPseudo`, `UnaryRRPseudo`, `BinaryRIPseudo`.
- **CN**: 该代码块声明或细化了 `Pseudo`, `UnaryRIPseudo`, `UnaryRXYPseudo`, `UnaryRRPseudo`, `BinaryRIPseudo` 等 TableGen 记录。

### Lines 5171-5225 / 第 5171-5225 行
```tablegen
5171: class BinaryRIEPseudo<SDPatternOperator operator, RegisterOperand cls,
5172:                       ImmOpWithPattern imm>
5173:   : Pseudo<(outs cls:$R1), (ins cls:$R3, imm:$I2),
5174:            [(set cls:$R1, (operator cls:$R3, imm:$I2))]>;
5175: 
5176: // Like BinaryRIAndK, but expanded after RA depending on the choice of register.
5177: multiclass BinaryRIAndKPseudo<string key, SDPatternOperator operator,
5178:                               RegisterOperand cls, ImmOpWithPattern imm> {
5179:   let NumOpsKey = key in {
5180:     let NumOpsValue = "3" in
5181:       def K : BinaryRIEPseudo<operator, cls, imm>,
5182:               Requires<[FeatureHighWord, FeatureDistinctOps]>;
5183:     let NumOpsValue = "2" in
5184:       def "" : BinaryRIPseudo<operator, cls, imm>,
5185:                Requires<[FeatureHighWord]>;
5186:   }
5187: }
5188: 
5189: // A pseudo that is used during register allocation when folding a memory
5190: // operand. The 3-address register instruction with a spilled source cannot
5191: // be converted directly to a target 2-address reg/mem instruction.
5192: // Mapping:  <INSN>R  ->  MemFoldPseudo  ->  <INSN>
5193: class MemFoldPseudo<string mnemonic, RegisterOperand cls, bits<5> bytes,
5194:                     AddressingMode mode>
5195:   : Pseudo<(outs cls:$R1), (ins cls:$R2, (mode $B2, $D2, $X2):$XBD2), []> {
5196:     let OpKey = !subst("mscrk", "msrkc",
5197:                 !subst("msgcrk", "msgrkc",
5198:                 mnemonic#"rk"#cls));
5199:     let OpType = "mem";
5200:     let MemKey = mnemonic#cls;
5201:     let MemType = "pseudo";
5202:     let mayLoad = 1;
5203:     let AccessBytes = bytes;
5204:     let HasIndex = 1;
5205:     let hasNoSchedulingInfo = 1;
5206: }
5207: 
5208: // Same as MemFoldPseudo but for mapping a W... vector instruction
5209: class MemFoldPseudo_FP<string mnemonic, RegisterOperand cls, bits<5> bytes,
5210:                     AddressingMode mode>
5211:   : MemFoldPseudo<mnemonic, cls, bytes, mode> {
5212:     let OpKey = mnemonic#"r"#"MemFold"#cls;
5213: }
5214: 
5215: class MemFoldPseudo_FPTern<string mnemonic, RegisterOperand cls, bits<5> bytes,
5216:                            AddressingMode mode>
5217:   : Pseudo<(outs cls:$R1),
5218:            (ins cls:$R2, cls:$R3, (mode $B2, $D2, $X2):$XBD2), []> {
5219:     let OpKey = mnemonic#"r"#"MemFold"#cls;
5220:     let OpType = "mem";
5221:     let MemKey = mnemonic#cls;
5222:     let MemType = "pseudo";
5223:     let mayLoad = 1;
5224:     let AccessBytes = bytes;
5225:     let HasIndex = 1;
```
- **EN**: This block declares or refines TableGen records such as `BinaryRIEPseudo`, `BinaryRIAndKPseudo`, `K`, `MemFoldPseudo`, `MemFoldPseudo_FP`, `MemFoldPseudo_FPTern`.
- **CN**: 该代码块声明或细化了 `BinaryRIEPseudo`, `BinaryRIAndKPseudo`, `K`, `MemFoldPseudo`, `MemFoldPseudo_FP`, `MemFoldPseudo_FPTern` 等 TableGen 记录。

### Lines 5226-5280 / 第 5226-5280 行
```tablegen
5226:     let hasNoSchedulingInfo = 1;
5227: }
5228: 
5229: // Same as MemFoldPseudo but for Load On Condition with CC operands.
5230: class MemFoldPseudo_CondMove<string mnemonic, RegisterOperand cls, bits<5> bytes,
5231:                              AddressingMode mode>
5232:   : Pseudo<(outs cls:$R1),
5233:            (ins cls:$R2, (mode $B2, $D2):$BD2, cond4:$valid, cond4:$M3), []> {
5234:     let OpKey = !subst("loc", "sel", mnemonic)#"r"#cls;
5235:     let OpType = "mem";
5236:     let MemKey = mnemonic#cls;
5237:     let MemType = "pseudo";
5238:     let mayLoad = 1;
5239:     let AccessBytes = bytes;
5240:     let hasNoSchedulingInfo = 1;
5241: }
5242: 
5243: // Like CompareRI, but expanded after RA depending on the choice of register.
5244: class CompareRIPseudo<SDPatternOperator operator, RegisterOperand cls,
5245:                       ImmOpWithPattern imm>
5246:   : Pseudo<(outs), (ins cls:$R1, imm:$I2),
5247:            [(set CC, (operator cls:$R1, imm:$I2))]> {
5248:   let isCompare = 1;
5249: }
5250: 
5251: // Like CompareRXY, but expanded after RA depending on the choice of register.
5252: class CompareRXYPseudo<SDPatternOperator operator, RegisterOperand cls,
5253:                        SDPatternOperator load, bits<5> bytes,
5254:                        AddressingMode mode = bdxaddr20only>
5255:   : Pseudo<(outs), (ins cls:$R1, (mode $B2, $D2, $X2):$XBD2),
5256:            [(set CC, (operator cls:$R1, (load mode:$XBD2)))]> {
5257:   let mayLoad = 1;
5258:   let Has20BitOffset = 1;
5259:   let HasIndex = 1;
5260:   let AccessBytes = bytes;
5261: }
5262: 
5263: // Like TestBinarySIL, but expanded later.
5264: class TestBinarySILPseudo<SDPatternOperator operator, ImmOpWithPattern imm>
5265:   : Pseudo<(outs), (ins (bdaddr12only $B1, $D1):$BD1, imm:$I2),
5266:            [(set CC, (operator bdaddr12only:$BD1, imm:$I2))]>;
5267: 
5268: // Like CondBinaryRRF, but expanded after RA depending on the choice of
5269: // register.
5270: class CondBinaryRRFPseudo<string mnemonic, RegisterOperand cls1,
5271:                           RegisterOperand cls2>
5272:   : Pseudo<(outs cls1:$R1),
5273:            (ins cls1:$R1src, cls2:$R2, cond4:$valid, cond4:$M3),
5274:            [(set cls1:$R1, (z_select_ccmask cls2:$R2, cls1:$R1src,
5275:                                             cond4:$valid, cond4:$M3))]> {
5276:   let Constraints = "$R1 = $R1src";
5277:   let CCMaskLast = 1;
5278:   let NumOpsKey = !subst("loc", "sel", mnemonic);
5279:   let NumOpsValue = "2";
5280:   let OpKey = mnemonic#cls1;
```
- **EN**: This block declares or refines TableGen records such as `MemFoldPseudo_CondMove`, `CompareRIPseudo`, `CompareRXYPseudo`, `TestBinarySILPseudo`, `CondBinaryRRFPseudo`.
- **CN**: 该代码块声明或细化了 `MemFoldPseudo_CondMove`, `CompareRIPseudo`, `CompareRXYPseudo`, `TestBinarySILPseudo`, `CondBinaryRRFPseudo` 等 TableGen 记录。

### Lines 5281-5335 / 第 5281-5335 行
```tablegen
5281:   let OpType = "reg";
5282: }
5283: 
5284: // Like CondBinaryRRFa, but expanded after RA depending on the choice of
5285: // register.
5286: class CondBinaryRRFaPseudo<string mnemonic, RegisterOperand cls1,
5287:                            RegisterOperand cls2, RegisterOperand cls3>
5288:   : Pseudo<(outs cls1:$R1),
5289:            (ins cls3:$R3, cls2:$R2, cond4:$valid, cond4:$M4),
5290:            [(set cls1:$R1, (z_select_ccmask cls2:$R2, cls3:$R3,
5291:                                             cond4:$valid, cond4:$M4))]> {
5292:   let CCMaskLast = 1;
5293:   let NumOpsKey = mnemonic;
5294:   let NumOpsValue = "3";
5295:   let OpKey = mnemonic#cls1;
5296:   let OpType = "reg";
5297: }
5298: 
5299: // Like CondBinaryRIE, but expanded after RA depending on the choice of
5300: // register.
5301: class CondBinaryRIEPseudo<RegisterOperand cls, ImmOpWithPattern imm>
5302:   : Pseudo<(outs cls:$R1),
5303:            (ins cls:$R1src, imm:$I2, cond4:$valid, cond4:$M3),
5304:            [(set cls:$R1, (z_select_ccmask imm:$I2, cls:$R1src,
5305:                                            cond4:$valid, cond4:$M3))]> {
5306:   let Constraints = "$R1 = $R1src";
5307:   let CCMaskLast = 1;
5308: }
5309: 
5310: // Like CondUnaryRSY, but expanded after RA depending on the choice of
5311: // register.
5312: class CondUnaryRSYPseudo<string mnemonic, SDPatternOperator operator,
5313:                          RegisterOperand cls, bits<5> bytes,
5314:                          AddressingMode mode = bdaddr20only>
5315:   : Pseudo<(outs cls:$R1),
5316:            (ins cls:$R1src, (mode $B2, $D2):$BD2, cond4:$valid, cond4:$R3),
5317:            [(set cls:$R1,
5318:                  (z_select_ccmask (operator mode:$BD2), cls:$R1src,
5319:                                   cond4:$valid, cond4:$R3))]> {
5320:   let Constraints = "$R1 = $R1src";
5321:   let mayLoad = 1;
5322:   let AccessBytes = bytes;
5323:   let CCMaskLast = 1;
5324:   let OpKey = mnemonic#"r"#cls;
5325:   let OpType = "mem";
5326:   let MemKey = mnemonic#cls;
5327:   let MemType = "target";
5328: }
5329: 
5330: // Like CondStoreRSY, but expanded after RA depending on the choice of
5331: // register.
5332: class CondStoreRSYPseudo<RegisterOperand cls, bits<5> bytes,
5333:                          AddressingMode mode = bdaddr20only>
5334:   : Pseudo<(outs),
5335:            (ins cls:$R1, (mode $B2, $D2):$BD2, cond4:$valid, cond4:$R3), []> {
```
- **EN**: This block declares or refines TableGen records such as `CondBinaryRRFaPseudo`, `CondBinaryRIEPseudo`, `CondUnaryRSYPseudo`, `CondStoreRSYPseudo`.
- **CN**: 该代码块声明或细化了 `CondBinaryRRFaPseudo`, `CondBinaryRIEPseudo`, `CondUnaryRSYPseudo`, `CondStoreRSYPseudo` 等 TableGen 记录。

### Lines 5336-5390 / 第 5336-5390 行
```tablegen
5336:   let mayStore = 1;
5337:   let AccessBytes = bytes;
5338:   let CCMaskLast = 1;
5339: }
5340: 
5341: // Like StoreRXY, but expanded after RA depending on the choice of register.
5342: class StoreRXYPseudo<SDPatternOperator operator, RegisterOperand cls,
5343:                      bits<5> bytes, AddressingMode mode = bdxaddr20only>
5344:   : Pseudo<(outs), (ins cls:$R1, (mode $B2, $D2, $X2):$XBD2),
5345:            [(operator cls:$R1, mode:$XBD2)]> {
5346:   let mayStore = 1;
5347:   let Has20BitOffset = 1;
5348:   let HasIndex = 1;
5349:   let AccessBytes = bytes;
5350: }
5351: 
5352: // Like RotateSelectRIEf, but expanded after RA depending on the choice
5353: // of registers.
5354: class RotateSelectRIEfPseudo<RegisterOperand cls1, RegisterOperand cls2>
5355:   : Pseudo<(outs cls1:$R1),
5356:            (ins cls1:$R1src, cls2:$R2, imm32zx8:$I3, imm32zx8:$I4,
5357:                 imm32zx8:$I5),
5358:            []> {
5359:   let Constraints = "$R1 = $R1src";
5360: }
5361: 
5362: // Implements "$dst = $cc & (8 >> CC) ? $src1 : $src2", where CC is
5363: // the value of the PSW's 2-bit condition code field.
5364: class SelectWrapper<ValueType vt, RegisterOperand cls>
5365:   : Pseudo<(outs cls:$dst),
5366:            (ins cls:$src1, cls:$src2, imm32zx4:$valid, imm32zx4:$cc),
5367:            [(set (vt cls:$dst), (z_select_ccmask cls:$src1, cls:$src2,
5368:                                             imm32zx4_timm:$valid, imm32zx4_timm:$cc))]> {
5369:   let usesCustomInserter = 1;
5370:   let hasNoSchedulingInfo = 1;
5371:   let Uses = [CC];
5372: }
5373: 
5374: // Stores $new to $addr if $cc is true ("" case) or false (Inv case).
5375: multiclass CondStores<RegisterOperand cls, SDPatternOperator store,
5376:                       SDPatternOperator load, AddressingMode mode> {
5377:   let Uses = [CC], usesCustomInserter = 1, hasNoSchedulingInfo = 1,
5378:       mayLoad = 1, mayStore = 1 in {
5379:     def "" : Pseudo<(outs),
5380:                     (ins cls:$new, mode:$addr, imm32zx4:$valid, imm32zx4:$cc),
5381:                     [(store (z_select_ccmask cls:$new, (load mode:$addr),
5382:                                              imm32zx4_timm:$valid, imm32zx4_timm:$cc),
5383:                             mode:$addr)]>;
5384:     def Inv : Pseudo<(outs),
5385:                      (ins cls:$new, mode:$addr, imm32zx4:$valid, imm32zx4:$cc),
5386:                      [(store (z_select_ccmask (load mode:$addr), cls:$new,
5387:                                               imm32zx4_timm:$valid, imm32zx4_timm:$cc),
5388:                               mode:$addr)]>;
5389:   }
5390: }
```
- **EN**: This block declares or refines TableGen records such as `StoreRXYPseudo`, `RotateSelectRIEfPseudo`, `SelectWrapper`, `CondStores`, `Inv`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `StoreRXYPseudo`, `RotateSelectRIEfPseudo`, `SelectWrapper`, `CondStores`, `Inv` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5391-5445 / 第 5391-5445 行
```tablegen
5391: 
5392: // OPERATOR is ATOMIC_SWAPW or an ATOMIC_LOADW_* operation.  PAT and OPERAND
5393: // describe the second (non-memory) operand.
5394: class AtomicLoadWBinary<SDPatternOperator operator, dag pat,
5395:                         DAGOperand operand>
5396:   : Pseudo<(outs GR32:$dst),
5397:            (ins bdaddr20only:$ptr, operand:$src2, ADDR32:$bitshift,
5398:                 ADDR32:$negbitshift, uimm32:$bitsize),
5399:            [(set GR32:$dst, (operator bdaddr20only:$ptr, pat, ADDR32:$bitshift,
5400:                                       ADDR32:$negbitshift, uimm32:$bitsize))]> {
5401:   let Defs = [CC];
5402:   let Has20BitOffset = 1;
5403:   let mayLoad = 1;
5404:   let mayStore = 1;
5405:   let usesCustomInserter = 1;
5406:   let hasNoSchedulingInfo = 1;
5407: }
5408: 
5409: // Specializations of AtomicLoadWBinary.
5410: class AtomicLoadWBinaryReg<SDPatternOperator operator>
5411:   : AtomicLoadWBinary<operator, (i32 GR32:$src2), GR32>;
5412: class AtomicLoadWBinaryImm<SDPatternOperator operator, ImmOpWithPattern imm>
5413:   : AtomicLoadWBinary<operator, (i32 imm:$src2), imm>;
5414: 
5415: // A pseudo instruction that is a direct alias of a real instruction.
5416: // These aliases are used in cases where a particular register operand is
5417: // fixed or where the same instruction is used with different register sizes.
5418: // The size parameter is the size in bytes of the associated real instruction.
5419: class Alias<int size, dag outs, dag ins, list<dag> pattern>
5420:   : InstSystemZ<size, outs, ins, "", pattern> {
5421:   let isPseudo = 1;
5422:   let isCodeGenOnly = 1;
5423: }
5424: 
5425: class UnaryAliasVRS<RegisterOperand cls1, RegisterOperand cls2>
5426:  : Alias<6, (outs cls1:$src1), (ins cls2:$src2), []>;
5427: 
5428: // An alias of a UnaryVRR*, but with different register sizes.
5429: class UnaryAliasVRR<SDPatternOperator operator, TypedReg tr1, TypedReg tr2>
5430:   : Alias<6, (outs tr1.op:$V1), (ins tr2.op:$V2),
5431:           [(set (tr1.vt tr1.op:$V1), (operator (tr2.vt tr2.op:$V2)))]>;
5432: 
5433: // An alias of a UnaryVRX, but with different register sizes.
5434: class UnaryAliasVRX<SDPatternOperator operator, TypedReg tr,
5435:                     AddressingMode mode = bdxaddr12only>
5436:   : Alias<6, (outs tr.op:$V1), (ins (mode $B2, $D2, $X2):$XBD2),
5437:           [(set (tr.vt tr.op:$V1), (operator mode:$XBD2))]>;
5438: 
5439: // An alias of a StoreVRX, but with different register sizes.
5440: class StoreAliasVRX<SDPatternOperator operator, TypedReg tr,
5441:                     AddressingMode mode = bdxaddr12only>
5442:   : Alias<6, (outs), (ins tr.op:$V1, (mode $B2, $D2, $X2):$XBD2),
5443:           [(operator (tr.vt tr.op:$V1), mode:$XBD2)]>;
5444: 
5445: // An alias of a BinaryRI, but with different register sizes.
```
- **EN**: This block declares or refines TableGen records such as `AtomicLoadWBinary`, `AtomicLoadWBinaryReg`, `AtomicLoadWBinaryImm`, `Alias`, `UnaryAliasVRS`, `UnaryAliasVRR`.
- **CN**: 该代码块声明或细化了 `AtomicLoadWBinary`, `AtomicLoadWBinaryReg`, `AtomicLoadWBinaryImm`, `Alias`, `UnaryAliasVRS`, `UnaryAliasVRR` 等 TableGen 记录。

### Lines 5446-5500 / 第 5446-5500 行
```tablegen
5446: class BinaryAliasRI<SDPatternOperator operator, RegisterOperand cls,
5447:                     ImmOpWithPattern imm>
5448:   : Alias<4, (outs cls:$R1), (ins cls:$R1src, imm:$I2),
5449:           [(set cls:$R1, (operator cls:$R1src, imm:$I2))]> {
5450:   let Constraints = "$R1 = $R1src";
5451: }
5452: 
5453: // An alias of a BinaryRIL, but with different register sizes.
5454: class BinaryAliasRIL<SDPatternOperator operator, RegisterOperand cls,
5455:                      ImmOpWithPattern imm>
5456:   : Alias<6, (outs cls:$R1), (ins cls:$R1src, imm:$I2),
5457:           [(set cls:$R1, (operator cls:$R1src, imm:$I2))]> {
5458:   let Constraints = "$R1 = $R1src";
5459: }
5460: 
5461: // An alias of a BinaryVRRf, but with different register sizes.
5462: class BinaryAliasVRRf<RegisterOperand cls>
5463:   : Alias<6, (outs VR128:$V1), (ins cls:$R2, cls:$R3), []>;
5464: 
5465: // An alias of a CompareRI, but with different register sizes.
5466: class CompareAliasRI<SDPatternOperator operator, RegisterOperand cls,
5467:                      ImmOpWithPattern imm>
5468:   : Alias<4, (outs), (ins cls:$R1, imm:$I2),
5469:           [(set CC, (operator cls:$R1, imm:$I2))]> {
5470:   let isCompare = 1;
5471: }
5472: 
5473: // An alias of a RotateSelectRIEf, but with different register sizes.
5474: multiclass RotateSelectAliasRIEf<RegisterOperand cls1, RegisterOperand cls2> {
5475:   let Constraints = "$R1 = $R1src" in {
5476:     def "" : Alias<6, (outs cls1:$R1),
5477:             (ins cls1:$R1src, cls2:$R2, imm32zx8:$I3, imm32zx8:$I4,
5478:                 imm32zx8:$I5), []>;
5479:     def Opt : Alias<6, (outs cls1:$R1),
5480:             (ins cls1:$R1src, cls2:$R2, imm32zx8:$I3, imm32zx8:$I4), []>;
5481:   }
5482: }
5483: 
5484: class MemsetPseudo<DAGOperand lenop, DAGOperand byteop>
5485:   : Pseudo<(outs), (ins bdaddr12only:$dest, lenop:$length, byteop:$B),
5486:            [(z_memset_mvc bdaddr12only:$dest, lenop:$length, byteop:$B)]> {
5487:   let Defs = [CC];
5488:   let mayLoad = 1;
5489:   let mayStore = 1;
5490:   let usesCustomInserter = 1;
5491:   let hasNoSchedulingInfo = 1;
5492: }
5493: 
5494: //===----------------------------------------------------------------------===//
5495: // Multiclasses that emit both real and pseudo instructions
5496: //===----------------------------------------------------------------------===//
5497: 
5498: multiclass BinaryRXYAndPseudo<string mnemonic, bits<16> opcode,
5499:                               SDPatternOperator operator, RegisterOperand cls,
5500:                               SDPatternOperator load, bits<5> bytes,
```
- **EN**: This block declares or refines TableGen records such as `BinaryAliasRI`, `BinaryAliasRIL`, `BinaryAliasVRRf`, `CompareAliasRI`, `RotateSelectAliasRIEf`, `Opt`.
- **CN**: 该代码块声明或细化了 `BinaryAliasRI`, `BinaryAliasRIL`, `BinaryAliasVRRf`, `CompareAliasRI`, `RotateSelectAliasRIEf`, `Opt` 等 TableGen 记录。

### Lines 5501-5555 / 第 5501-5555 行
```tablegen
5501:                               AddressingMode mode = bdxaddr20only> {
5502:   def "" : BinaryRXY<mnemonic, opcode, operator, cls, load, bytes, mode> {
5503:     let MemKey = mnemonic#cls;
5504:     let MemType = "target";
5505:   }
5506:   let Has20BitOffset = 1 in
5507:     def _MemFoldPseudo : MemFoldPseudo<mnemonic, cls, bytes, mode>;
5508: }
5509: 
5510: multiclass BinaryRXPairAndPseudo<string mnemonic, bits<8> rxOpcode,
5511:                                  bits<16> rxyOpcode, SDPatternOperator operator,
5512:                                  RegisterOperand cls,
5513:                                  SDPatternOperator load, bits<5> bytes> {
5514:   let DispKey = mnemonic # cls in {
5515:     def "" : BinaryRX<mnemonic, rxOpcode, operator, cls, load, bytes,
5516:                       bdxaddr12pair> {
5517:       let DispSize = "12";
5518:       let MemKey = mnemonic#cls;
5519:       let MemType = "target";
5520:     }
5521:     let DispSize = "20" in
5522:       def Y  : BinaryRXY<mnemonic#"y", rxyOpcode, operator, cls, load,
5523:                          bytes, bdxaddr20pair>;
5524:   }
5525:   def _MemFoldPseudo : MemFoldPseudo<mnemonic, cls, bytes, bdxaddr12pair>;
5526: }
5527: 
5528: multiclass BinaryRXEAndPseudo<string mnemonic, bits<16> opcode,
5529:                               SDPatternOperator operator, RegisterOperand cls,
5530:                               SDPatternOperator load, bits<5> bytes> {
5531:   def "" : BinaryRXE<mnemonic, opcode, operator, cls, load, bytes> {
5532:     let MemKey = mnemonic#cls;
5533:     let MemType = "target";
5534:   }
5535:   def _MemFoldPseudo : MemFoldPseudo_FP<mnemonic, cls, bytes, bdxaddr12pair>;
5536: }
5537: 
5538: multiclass TernaryRXFAndPseudo<string mnemonic, bits<16> opcode,
5539:                                SDPatternOperator operator, RegisterOperand cls1,
5540:                                RegisterOperand cls2, SDPatternOperator load,
5541:                                bits<5> bytes> {
5542:   def "" : TernaryRXF<mnemonic, opcode, operator, cls1, cls2, load, bytes> {
5543:     let MemKey = mnemonic#cls1;
5544:     let MemType = "target";
5545:   }
5546:   def _MemFoldPseudo : MemFoldPseudo_FPTern<mnemonic, cls1, bytes, bdxaddr12pair>;
5547: }
5548: 
5549: multiclass CondUnaryRSYPairAndMemFold<string mnemonic, bits<16> opcode,
5550:                                       SDPatternOperator operator,
5551:                                       RegisterOperand cls, bits<5> bytes,
5552:                                       AddressingMode mode = bdaddr20only> {
5553:   defm "" : CondUnaryRSYPair<mnemonic, opcode, operator, cls, bytes, mode>;
5554:   def _MemFoldPseudo : MemFoldPseudo_CondMove<mnemonic, cls, bytes, mode>;
5555: }
```
- **EN**: This block declares or refines TableGen records such as `_MemFoldPseudo`, `BinaryRXPairAndPseudo`, `Y`, `_MemFoldPseudo`, `BinaryRXEAndPseudo`, `_MemFoldPseudo`.
- **CN**: 该代码块声明或细化了 `_MemFoldPseudo`, `BinaryRXPairAndPseudo`, `Y`, `_MemFoldPseudo`, `BinaryRXEAndPseudo`, `_MemFoldPseudo` 等 TableGen 记录。

### Lines 5556-5610 / 第 5556-5610 行
```tablegen
5556: 
5557: multiclass CondUnaryRSYPseudoAndMemFold<string mnemonic,
5558:                                         SDPatternOperator operator,
5559:                                         RegisterOperand cls, bits<5> bytes,
5560:                                         AddressingMode mode = bdaddr20only> {
5561:   def "" : CondUnaryRSYPseudo<mnemonic, operator, cls, bytes, mode>;
5562:   def _MemFoldPseudo : MemFoldPseudo_CondMove<mnemonic, cls, bytes, mode>;
5563: }
5564: 
5565: // Define an instruction that operates on two fixed-length blocks of memory,
5566: // and associated pseudo instructions for operating on blocks of any size.
5567: // There are two pseudos for the different cases of when the length is
5568: // constant or variable. The length operand of a pseudo is actually one less
5569: // than the intended number of bytes, since the register case needs to use an
5570: // EXRL with a target instruction that adds one to the length always.
5571: multiclass MemorySS<string mnemonic, bits<8> opcode, SDPatternOperator memop> {
5572:   def "" : SideEffectBinarySSa<mnemonic, opcode>;
5573:   let usesCustomInserter = 1, hasNoSchedulingInfo = 1, Defs = [CC] in {
5574:     def Imm : Pseudo<(outs), (ins bdaddr12only:$dest, bdaddr12only:$src,
5575:                                   imm64:$length),
5576:                              [(memop bdaddr12only:$dest, bdaddr12only:$src,
5577:                                      imm64:$length)]>;
5578:     def Reg : Pseudo<(outs), (ins bdaddr12only:$dest, bdaddr12only:$src,
5579:                                   ADDR64:$length),
5580:                              [(memop bdaddr12only:$dest, bdaddr12only:$src,
5581:                                      ADDR64:$length)]>;
5582:   }
5583: }
5584: 
5585: // The same, but setting a CC result as comparison operator.
5586: multiclass CompareMemorySS<string mnemonic, bits<8> opcode,
5587:                            SDPatternOperator memop> {
5588:   def "" : SideEffectBinarySSa<mnemonic, opcode>;
5589:   let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in {
5590:     def Imm : Pseudo<(outs), (ins bdaddr12only:$dest, bdaddr12only:$src,
5591:                                   imm64:$length),
5592:                           [(set CC, (memop bdaddr12only:$dest, bdaddr12only:$src,
5593:                                            imm64:$length))]>;
5594:     def Reg : Pseudo<(outs), (ins bdaddr12only:$dest, bdaddr12only:$src,
5595:                                   ADDR64:$length),
5596:                           [(set CC, (memop bdaddr12only:$dest, bdaddr12only:$src,
5597:                                            ADDR64:$length))]>;
5598:   }
5599: }
5600: 
5601: // Define an instruction that operates on two strings, both terminated
5602: // by the character in R0.  The instruction processes a CPU-determinated
5603: // number of bytes at a time and sets CC to 3 if the instruction needs
5604: // to be repeated.  Also define a pseudo instruction that represents
5605: // the full loop (the main instruction plus the branch on CC==3).
5606: multiclass StringRRE<string mnemonic, bits<16> opcode,
5607:                      SDPatternOperator operator> {
5608:   let Uses = [R0L] in
5609:     def "" : SideEffectBinaryMemMemRRE<mnemonic, opcode, GR64, GR64>;
5610:   let usesCustomInserter = 1, hasNoSchedulingInfo = 1 in
```
- **EN**: This block declares or refines TableGen records such as `CondUnaryRSYPseudoAndMemFold`, `_MemFoldPseudo`, `MemorySS`, `Imm`, `Reg`, `CompareMemorySS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `CondUnaryRSYPseudoAndMemFold`, `_MemFoldPseudo`, `MemorySS`, `Imm`, `Reg`, `CompareMemorySS` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5611-5615 / 第 5611-5615 行
```tablegen
5611:     def Loop : Pseudo<(outs GR64:$end),
5612:                       (ins GR64:$start1, GR64:$start2, GR32:$char),
5613:                       [(set GR64:$end, (operator GR64:$start1, GR64:$start2,
5614:                                                  GR32:$char))]>;
5615: }
```
- **EN**: This block declares or refines TableGen records such as `Loop`.
- **CN**: 该代码块声明或细化了 `Loop` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
