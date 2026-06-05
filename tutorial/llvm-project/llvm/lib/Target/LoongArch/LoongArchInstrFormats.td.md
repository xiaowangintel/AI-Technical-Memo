# LoongArchInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 LoongArch 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //===- LoongArchInstrFormats.td - LoongArch Instr. Formats -*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Describe LoongArch instructions format
  11: //
  12: //  opcode       - operation code.
  13: //  rd           - destination register operand.
  14: //  r{j/k}       - source register operand.
  15: //  immN         - immediate data operand.
  16: //
  17: //===----------------------------------------------------------------------===//
  18: 
  19: class LAInst<dag outs, dag ins, string opcstr, string opnstr,
  20:              list<dag> pattern = []>
  21:     : Instruction {
  22:   field bits<32> Inst;
  23: 
  24:   let Namespace = "LoongArch";
  25:   let Size = 4;
  26:   let OutOperandList = outs;
  27:   let InOperandList = ins;
  28:   let AsmString = opcstr # "\t" # opnstr;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `LAInst`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `LAInst` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29:   let Pattern = pattern;
  30: 
  31:   // Target-specific instruction info and defaults
  32: 
  33:   bit IsSubjectToAMORdConstraint = 0;
  34:   let TSFlags{0} = IsSubjectToAMORdConstraint;
  35: 
  36:   bit IsAMCAS = 0;
  37:   let TSFlags{1} = IsAMCAS;
  38: }
  39: 
  40: // Pseudo instructions
  41: class Pseudo<dag outs, dag ins, list<dag> pattern = [], string opcstr = "",
  42:              string opnstr = "">
  43:     : LAInst<outs, ins, opcstr, opnstr, pattern> {
  44:   let isPseudo = 1;
  45:   let isCodeGenOnly = 1;
  46: }
  47: 
  48: class deriveInsnMnemonic<string name> {
  49:   string ret = !tolower(!subst("@", "_", !subst("_", ".", !subst("__", "@", name))));
  50: }
  51: 
  52: // 2R-type
  53: // <opcode | rj | rd>
  54: class Fmt2R<bits<32> op, dag outs, dag ins, string opnstr,
  55:             list<dag> pattern = []>
  56:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `Pseudo`, `deriveInsnMnemonic`, `Fmt2R`.
- **CN**: 该代码块声明或细化了 `Pseudo`, `deriveInsnMnemonic`, `Fmt2R` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57:   bits<5> rj;
  58:   bits<5> rd;
  59: 
  60:   let Inst{31-0} = op;
  61:   let Inst{9-5} = rj;
  62:   let Inst{4-0} = rd;
  63: }
  64: 
  65: // 3R-type
  66: // <opcode | rk | rj | rd>
  67: class Fmt3R<bits<32> op, dag outs, dag ins, string opnstr,
  68:             list<dag> pattern = []>
  69:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  70:   bits<5> rk;
  71:   bits<5> rj;
  72:   bits<5> rd;
  73: 
  74:   let Inst{31-0} = op;
  75:   let Inst{14-10} = rk;
  76:   let Inst{9-5} = rj;
  77:   let Inst{4-0} = rd;
  78: }
  79: 
  80: // 3RI2-type
  81: // <opcode | I2 | rk | rj | rd>
  82: class Fmt3RI2<bits<32> op, dag outs, dag ins, string opnstr,
  83:               list<dag> pattern = []>
  84:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `Fmt3R`, `Fmt3RI2`.
- **CN**: 该代码块声明或细化了 `Fmt3R`, `Fmt3RI2` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85:   bits<2> imm2;
  86:   bits<5> rk;
  87:   bits<5> rj;
  88:   bits<5> rd;
  89: 
  90:   let Inst{31-0} = op;
  91:   let Inst{16-15} = imm2;
  92:   let Inst{14-10} = rk;
  93:   let Inst{9-5} = rj;
  94:   let Inst{4-0} = rd;
  95: }
  96: 
  97: // 3RI3-type
  98: // <opcode | I3 | rk | rj | rd>
  99: class Fmt3RI3<bits<32> op, dag outs, dag ins, string opnstr,
 100:               list<dag> pattern = []>
 101:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 102:   bits<3> imm3;
 103:   bits<5> rk;
 104:   bits<5> rj;
 105:   bits<5> rd;
 106: 
 107:   let Inst{31-0} = op;
 108:   let Inst{17-15} = imm3;
 109:   let Inst{14-10} = rk;
 110:   let Inst{9-5} = rj;
 111:   let Inst{4-0} = rd;
 112: }
```
- **EN**: This block declares or refines TableGen records such as `Fmt3RI3`.
- **CN**: 该代码块声明或细化了 `Fmt3RI3` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113: 
 114: // 2RI5-type
 115: // <opcode | I5 | rj | rd>
 116: class Fmt2RI5<bits<32> op, dag outs, dag ins, string opnstr,
 117:               list<dag> pattern = []>
 118:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 119:   bits<5> imm5;
 120:   bits<5> rj;
 121:   bits<5> rd;
 122: 
 123:   let Inst{31-0} = op;
 124:   let Inst{14-10} = imm5;
 125:   let Inst{9-5} = rj;
 126:   let Inst{4-0} = rd;
 127: }
 128: 
 129: // 2RI6-type
 130: // <opcode | I6 | rj | rd>
 131: class Fmt2RI6<bits<32> op, dag outs, dag ins, string opnstr,
 132:               list<dag> pattern = []>
 133:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 134:   bits<6> imm6;
 135:   bits<5> rj;
 136:   bits<5> rd;
 137: 
 138:   let Inst{31-0} = op;
 139:   let Inst{15-10} = imm6;
 140:   let Inst{9-5} = rj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI5`, `Fmt2RI6`.
- **CN**: 该代码块声明或细化了 `Fmt2RI5`, `Fmt2RI6` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141:   let Inst{4-0} = rd;
 142: }
 143: 
 144: // 2RI8-type
 145: // <opcode | I8 | rj | rd>
 146: class Fmt2RI8<bits<32> op, dag outs, dag ins, string opnstr,
 147:               list<dag> pattern = []>
 148:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 149:   bits<8> imm8;
 150:   bits<5> rj;
 151:   bits<5> rd;
 152: 
 153:   let Inst{31-0} = op;
 154:   let Inst{17-10} = imm8;
 155:   let Inst{9-5} = rj;
 156:   let Inst{4-0} = rd;
 157: }
 158: 
 159: // 2RI12-type
 160: // <opcode | I12 | rj | rd>
 161: class Fmt2RI12<bits<32> op, dag outs, dag ins, string opnstr,
 162:                list<dag> pattern = []>
 163:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 164:   bits<12> imm12;
 165:   bits<5> rj;
 166:   bits<5> rd;
 167: 
 168:   let Inst{31-0} = op;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8`, `Fmt2RI12`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8`, `Fmt2RI12` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169:   let Inst{21-10} = imm12;
 170:   let Inst{9-5} = rj;
 171:   let Inst{4-0} = rd;
 172: }
 173: 
 174: // 2RI14-type
 175: // <opcode | I14 | rj | rd>
 176: class Fmt2RI14<bits<32> op, dag outs, dag ins, string opnstr,
 177:                list<dag> pattern = []>
 178:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 179:   bits<14> imm14;
 180:   bits<5> rj;
 181:   bits<5> rd;
 182: 
 183:   let Inst{31-0} = op;
 184:   let Inst{23-10} = imm14;
 185:   let Inst{9-5} = rj;
 186:   let Inst{4-0} = rd;
 187: }
 188: 
 189: // 2RI16-type
 190: // <opcode | I16 | rj | rd>
 191: class Fmt2RI16<bits<32> op, dag outs, dag ins, string opnstr,
 192:                list<dag> pattern = []>
 193:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 194:   bits<16> imm16;
 195:   bits<5> rj;
 196:   bits<5> rd;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI14`, `Fmt2RI16`.
- **CN**: 该代码块声明或细化了 `Fmt2RI14`, `Fmt2RI16` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: 
 198:   let Inst{31-0} = op;
 199:   let Inst{25-10} = imm16;
 200:   let Inst{9-5} = rj;
 201:   let Inst{4-0} = rd;
 202: }
 203: 
 204: // 1RI20-type
 205: // <opcode | I20 | rd>
 206: class Fmt1RI20<bits<32> op, dag outs, dag ins, string opnstr,
 207:                list<dag> pattern = []>
 208:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 209:   bits<20> imm20;
 210:   bits<5> rd;
 211: 
 212:   let Inst{31-0} = op;
 213:   let Inst{24-5} = imm20;
 214:   let Inst{4-0} = rd;
 215: }
 216: 
 217: // 1RI21-type
 218: // <opcode | I21[15:0] | rj | I21[20:16]>
 219: class Fmt1RI21<bits<32> op, dag outs, dag ins, string opnstr,
 220:                list<dag> pattern = []>
 221:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 222:   bits<21> imm21;
 223:   bits<5> rj;
 224: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt1RI20`, `Fmt1RI21`.
- **CN**: 该代码块声明或细化了 `Fmt1RI20`, `Fmt1RI21` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225:   let Inst{31-0} = op;
 226:   let Inst{25-10} = imm21{15-0};
 227:   let Inst{9-5} = rj;
 228:   let Inst{4-0} = imm21{20-16};
 229: }
 230: 
 231: // I15-type
 232: // <opcode | I15>
 233: class FmtI15<bits<32> op, dag outs, dag ins, string opnstr,
 234:              list<dag> pattern = []>
 235:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 236:   bits<15> imm15;
 237: 
 238:   let Inst{31-0} = op;
 239:   let Inst{14-0} = imm15;
 240: }
 241: 
 242: // I26-type
 243: // <opcode | I26[15:0] | I26[25:16]>
 244: class FmtI26<bits<32> op, dag outs, dag ins, string opnstr,
 245:              list<dag> pattern = []>
 246:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 247:   bits<26> imm26;
 248: 
 249:   let Inst{31-0} = op;
 250:   let Inst{25-10} = imm26{15-0};
 251:   let Inst{9-0} = imm26{25-16};
 252: }
```
- **EN**: This block declares or refines TableGen records such as `FmtI15`, `FmtI26`.
- **CN**: 该代码块声明或细化了 `FmtI15`, `FmtI26` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253: 
 254: // FmtBSTR_W
 255: // <opcode | msbw | lsbw | rj | rd>
 256: class FmtBSTR_W<bits<32> op, dag outs, dag ins, string opnstr,
 257:                 list<dag> pattern = []>
 258:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 259:   bits<5> msbw;
 260:   bits<5> lsbw;
 261:   bits<5> rj;
 262:   bits<5> rd;
 263: 
 264:   let Inst{31-0} = op;
 265:   let Inst{20-16} = msbw;
 266:   let Inst{14-10} = lsbw;
 267:   let Inst{9-5} = rj;
 268:   let Inst{4-0} = rd;
 269: }
 270: 
 271: // FmtBSTR_D
 272: // <opcode | msbd | lsbd | rj | rd>
 273: class FmtBSTR_D<bits<32> op, dag outs, dag ins, string opnstr,
 274:                 list<dag> pattern = []>
 275:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 276:   bits<6> msbd;
 277:   bits<6> lsbd;
 278:   bits<5> rj;
 279:   bits<5> rd;
 280: 
```
- **EN**: This block declares or refines TableGen records such as `FmtBSTR_W`, `FmtBSTR_D`.
- **CN**: 该代码块声明或细化了 `FmtBSTR_W`, `FmtBSTR_D` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281:   let Inst{31-0} = op;
 282:   let Inst{21-16} = msbd;
 283:   let Inst{15-10} = lsbd;
 284:   let Inst{9-5} = rj;
 285:   let Inst{4-0} = rd;
 286: }
 287: 
 288: // FmtASRT
 289: // <opcode | rk | rj>
 290: class FmtASRT<bits<32> op, dag outs, dag ins, string opnstr,
 291:               list<dag> pattern = []>
 292:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 293:   bits<5> rk;
 294:   bits<5> rj;
 295: 
 296:   let Inst{31-0} = op;
 297:   let Inst{14-10} = rk;
 298:   let Inst{9-5} = rj;
 299: }
 300: 
 301: // FmtPRELD
 302: // < 0b0010101011 | I12 | rj | I5>
 303: class FmtPRELD<dag outs, dag ins, string opnstr, list<dag> pattern = []>
 304:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 305:   bits<12> imm12;
 306:   bits<5> rj;
 307:   bits<5> imm5;
 308: 
```
- **EN**: This block declares or refines TableGen records such as `FmtASRT`, `FmtPRELD`.
- **CN**: 该代码块声明或细化了 `FmtASRT`, `FmtPRELD` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309:   let Inst{31-22} = 0b0010101011;
 310:   let Inst{21-10} = imm12;
 311:   let Inst{9-5} = rj;
 312:   let Inst{4-0} = imm5;
 313: }
 314: 
 315: // FmtPRELDX
 316: // < 0b00111000001011000 | rk | rj | I5>
 317: class FmtPRELDX<dag outs, dag ins, string opnstr, list<dag> pattern = []>
 318:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 319:   bits<5> rk;
 320:   bits<5> rj;
 321:   bits<5> imm5;
 322: 
 323:   let Inst{31-15} = 0b00111000001011000;
 324:   let Inst{14-10} = rk;
 325:   let Inst{9-5} = rj;
 326:   let Inst{4-0} = imm5;
 327: }
 328: 
 329: // FmtCSR
 330: // <opcode | csr_num | rd>
 331: class FmtCSR<bits<32> op, dag outs, dag ins, string opnstr,
 332:              list<dag> pattern = []>
 333:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 334:   bits<14> csr_num;
 335:   bits<5> rd;
 336: 
```
- **EN**: This block declares or refines TableGen records such as `FmtPRELDX`, `FmtCSR`.
- **CN**: 该代码块声明或细化了 `FmtPRELDX`, `FmtCSR` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337:   let Inst{31-0} = op;
 338:   let Inst{23-10} = csr_num;
 339:   let Inst{4-0} = rd;
 340: }
 341: 
 342: // FmtCSRXCHG
 343: // <opcode | csr_num | rj | rd>
 344: class FmtCSRXCHG<bits<32> op, dag outs, dag ins, string opnstr,
 345:                  list<dag> pattern = []>
 346:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 347:   bits<14> csr_num;
 348:   bits<5> rj;
 349:   bits<5> rd;
 350: 
 351:   let Inst{31-0} = op;
 352:   let Inst{23-10} = csr_num;
 353:   let Inst{9-5} = rj;
 354:   let Inst{4-0} = rd;
 355: }
 356: 
 357: // FmtCACOP
 358: // <0b0000011000 | I12 | rj | I5>
 359: class FmtCACOP<dag outs, dag ins, string opnstr, list<dag> pattern = []>
 360:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 361:   bits<12> imm12;
 362:   bits<5> rj;
 363:   bits<5> op;
 364: 
```
- **EN**: This block declares or refines TableGen records such as `FmtCSRXCHG`, `FmtCACOP`.
- **CN**: 该代码块声明或细化了 `FmtCSRXCHG`, `FmtCACOP` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365:   let Inst{31-22} = 0b0000011000;
 366:   let Inst{21-10} = imm12;
 367:   let Inst{9-5} = rj;
 368:   let Inst{4-0} = op;
 369: }
 370: 
 371: // FmtIMM32
 372: // <I32>
 373: class FmtI32<bits<32> op, list<dag> pattern = []>
 374:     : LAInst<(outs), (ins), deriveInsnMnemonic<NAME>.ret, "", pattern> {
 375:   let Inst{31-0} = op;
 376: }
 377: 
 378: // FmtINVTLB
 379: // <0b00000110010010011 | rk | rj | I5>
 380: class FmtINVTLB<dag outs, dag ins, string opnstr, list<dag> pattern = []>
 381:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 382:   bits<5> rk;
 383:   bits<5> rj;
 384:   bits<5> op;
 385: 
 386:   let Inst{31-15} = 0b00000110010010011;
 387:   let Inst{14-10} = rk;
 388:   let Inst{9-5} = rj;
 389:   let Inst{4-0} = op;
 390: }
 391: 
 392: // FmtLDPTE
```
- **EN**: This block declares or refines TableGen records such as `FmtI32`, `FmtINVTLB`.
- **CN**: 该代码块声明或细化了 `FmtI32`, `FmtINVTLB` 等 TableGen 记录。

### Lines 393-416 / 第 393-416 行
```tablegen
 393: // <0b00000110010001 | seq | rj | 00000>
 394: class FmtLDPTE<dag outs, dag ins, string opnstr, list<dag> pattern = []>
 395:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 396:   bits<8> seq;
 397:   bits<5> rj;
 398: 
 399:   let Inst{31-18} = 0b00000110010001;
 400:   let Inst{17-10} = seq;
 401:   let Inst{9-5} = rj;
 402:   let Inst{4-0} = 0b00000;
 403: }
 404: 
 405: // FmtUD
 406: // <0b0011100001100000000001 | I5 | I5>
 407: class FmtUD<dag outs, dag ins, string opnstr, list<dag> pattern = []>
 408:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 409:   bits<5> imm5;
 410: 
 411:   let Inst{31-10} = 0b0011100001100000000001;
 412:   let Inst{9-5} = imm5;
 413:   let Inst{4-0} = imm5;
 414: 
 415:   let DecoderMethod = "DecodeAMOrUDInstruction";
 416: }
```
- **EN**: This block declares or refines TableGen records such as `FmtLDPTE`, `FmtUD`.
- **CN**: 该代码块声明或细化了 `FmtLDPTE`, `FmtUD` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
