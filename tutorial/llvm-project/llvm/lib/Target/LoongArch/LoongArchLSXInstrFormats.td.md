# LoongArchLSXInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchLSXInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 LoongArch 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: // LoongArchLSXInstrFormats.td - LoongArch LSX Instr Formats -*- tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Describe LoongArch LSX instructions format
  11: //
  12: //  opcode       - operation code.
  13: //  vd/rd/cd     - destination register operand.
  14: //  {r/v}{j/k}   - source register operand.
  15: //  immN         - immediate data operand.
  16: //
  17: //===----------------------------------------------------------------------===//
  18: 
  19: // 1RI13-type
  20: // <opcode | I13 | vd>
  21: class Fmt1RI13_VI<bits<32> op, dag outs, dag ins, string opnstr,
  22:                   list<dag> pattern = []>
  23:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  24:   bits<13> imm13;
  25:   bits<5> vd;
  26: 
  27:   let Inst{31-0} = op;
  28:   let Inst{17-5} = imm13;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Fmt1RI13_VI`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Fmt1RI13_VI` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29:   let Inst{4-0} = vd;
  30: }
  31: 
  32: // 2R-type
  33: // <opcode | vj | vd>
  34: class Fmt2R_VV<bits<32> op, dag outs, dag ins, string opnstr,
  35:                list<dag> pattern = []>
  36:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  37:   bits<5> vj;
  38:   bits<5> vd;
  39: 
  40:   let Inst{31-0} = op;
  41:   let Inst{9-5} = vj;
  42:   let Inst{4-0} = vd;
  43: }
  44: 
  45: // <opcode | rj | vd>
  46: class Fmt2R_VR<bits<32> op, dag outs, dag ins, string opnstr,
  47:                list<dag> pattern = []>
  48:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  49:   bits<5> rj;
  50:   bits<5> vd;
  51: 
  52:   let Inst{31-0} = op;
  53:   let Inst{9-5} = rj;
  54:   let Inst{4-0} = vd;
  55: }
  56: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt2R_VV`, `Fmt2R_VR`.
- **CN**: 该代码块声明或细化了 `Fmt2R_VV`, `Fmt2R_VR` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57: // <opcode | vj | cd>
  58: class Fmt2R_CV<bits<32> op, dag outs, dag ins, string opnstr,
  59:                list<dag> pattern = []>
  60:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  61:   bits<5> vj;
  62:   bits<3> cd;
  63: 
  64:   let Inst{31-0} = op;
  65:   let Inst{9-5} = vj;
  66:   let Inst{2-0} = cd;
  67: }
  68: 
  69: // 2RI1-type
  70: // <opcode | I1 | vj | vd>
  71: class Fmt2RI1_VVI<bits<32> op, dag outs, dag ins, string opnstr,
  72:                   list<dag> pattern = []>
  73:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  74:   bits<1> imm1;
  75:   bits<5> vj;
  76:   bits<5> vd;
  77: 
  78:   let Inst{31-0} = op;
  79:   let Inst{10} = imm1;
  80:   let Inst{9-5} = vj;
  81:   let Inst{4-0} = vd;
  82: }
  83: 
  84: // <opcode | I1 | rj | vd>
```
- **EN**: This block declares or refines TableGen records such as `Fmt2R_CV`, `Fmt2RI1_VVI`.
- **CN**: 该代码块声明或细化了 `Fmt2R_CV`, `Fmt2RI1_VVI` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: class Fmt2RI1_VRI<bits<32> op, dag outs, dag ins, string opnstr,
  86:                   list<dag> pattern = []>
  87:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  88:   bits<1> imm1;
  89:   bits<5> rj;
  90:   bits<5> vd;
  91: 
  92:   let Inst{31-0} = op;
  93:   let Inst{10} = imm1;
  94:   let Inst{9-5} = rj;
  95:   let Inst{4-0} = vd;
  96: }
  97: 
  98: // <opcode | I1 | vj | rd>
  99: class Fmt2RI1_RVI<bits<32> op, dag outs, dag ins, string opnstr,
 100:                   list<dag> pattern = []>
 101:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 102:   bits<1> imm1;
 103:   bits<5> vj;
 104:   bits<5> rd;
 105: 
 106:   let Inst{31-0} = op;
 107:   let Inst{10} = imm1;
 108:   let Inst{9-5} = vj;
 109:   let Inst{4-0} = rd;
 110: }
 111: 
 112: // 2RI2-type
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI1_VRI`, `Fmt2RI1_RVI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI1_VRI`, `Fmt2RI1_RVI` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113: // <opcode | I2 | vj | vd>
 114: class Fmt2RI2_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 115:                   list<dag> pattern = []>
 116:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 117:   bits<2> imm2;
 118:   bits<5> vj;
 119:   bits<5> vd;
 120: 
 121:   let Inst{31-0} = op;
 122:   let Inst{11-10} = imm2;
 123:   let Inst{9-5} = vj;
 124:   let Inst{4-0} = vd;
 125: }
 126: 
 127: // <opcode | I2 | rj | vd>
 128: class Fmt2RI2_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 129:                   list<dag> pattern = []>
 130:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 131:   bits<2> imm2;
 132:   bits<5> rj;
 133:   bits<5> vd;
 134: 
 135:   let Inst{31-0} = op;
 136:   let Inst{11-10} = imm2;
 137:   let Inst{9-5} = rj;
 138:   let Inst{4-0} = vd;
 139: }
 140: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI2_VVI`, `Fmt2RI2_VRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI2_VVI`, `Fmt2RI2_VRI` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: // <opcode | I2 | vj | rd>
 142: class Fmt2RI2_RVI<bits<32> op, dag outs, dag ins, string opnstr,
 143:                   list<dag> pattern = []>
 144:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 145:   bits<2> imm2;
 146:   bits<5> vj;
 147:   bits<5> rd;
 148: 
 149:   let Inst{31-0} = op;
 150:   let Inst{11-10} = imm2;
 151:   let Inst{9-5} = vj;
 152:   let Inst{4-0} = rd;
 153: }
 154: 
 155: // 2RI3-type
 156: // <opcode | I3 | vj | vd>
 157: class Fmt2RI3_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 158:                   list<dag> pattern = []>
 159:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 160:   bits<3> imm3;
 161:   bits<5> vj;
 162:   bits<5> vd;
 163: 
 164:   let Inst{31-0} = op;
 165:   let Inst{12-10} = imm3;
 166:   let Inst{9-5} = vj;
 167:   let Inst{4-0} = vd;
 168: }
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI2_RVI`, `Fmt2RI3_VVI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI2_RVI`, `Fmt2RI3_VVI` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: 
 170: // <opcode | I3 | rj | vd>
 171: class Fmt2RI3_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 172:                   list<dag> pattern = []>
 173:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 174:   bits<3> imm3;
 175:   bits<5> rj;
 176:   bits<5> vd;
 177: 
 178:   let Inst{31-0} = op;
 179:   let Inst{12-10} = imm3;
 180:   let Inst{9-5} = rj;
 181:   let Inst{4-0} = vd;
 182: }
 183: 
 184: // <opcode | I3 | vj | rd>
 185: class Fmt2RI3_RVI<bits<32> op, dag outs, dag ins, string opnstr,
 186:                   list<dag> pattern = []>
 187:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 188:   bits<3> imm3;
 189:   bits<5> vj;
 190:   bits<5> rd;
 191: 
 192:   let Inst{31-0} = op;
 193:   let Inst{12-10} = imm3;
 194:   let Inst{9-5} = vj;
 195:   let Inst{4-0} = rd;
 196: }
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI3_VRI`, `Fmt2RI3_RVI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI3_VRI`, `Fmt2RI3_RVI` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: 
 198: // 2RI4-type
 199: // <opcode | I4 | vj | vd>
 200: class Fmt2RI4_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 201:                   list<dag> pattern = []>
 202:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 203:   bits<4> imm4;
 204:   bits<5> vj;
 205:   bits<5> vd;
 206: 
 207:   let Inst{31-0} = op;
 208:   let Inst{13-10} = imm4;
 209:   let Inst{9-5} = vj;
 210:   let Inst{4-0} = vd;
 211: }
 212: 
 213: // <opcode | I4 | rj | vd>
 214: class Fmt2RI4_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 215:                   list<dag> pattern = []>
 216:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 217:   bits<4> imm4;
 218:   bits<5> rj;
 219:   bits<5> vd;
 220: 
 221:   let Inst{31-0} = op;
 222:   let Inst{13-10} = imm4;
 223:   let Inst{9-5} = rj;
 224:   let Inst{4-0} = vd;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI4_VVI`, `Fmt2RI4_VRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI4_VVI`, `Fmt2RI4_VRI` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225: }
 226: 
 227: // <opcode | I4 | vj | rd>
 228: class Fmt2RI4_RVI<bits<32> op, dag outs, dag ins, string opnstr,
 229:                   list<dag> pattern = []>
 230:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 231:   bits<4> imm4;
 232:   bits<5> vj;
 233:   bits<5> rd;
 234: 
 235:   let Inst{31-0} = op;
 236:   let Inst{13-10} = imm4;
 237:   let Inst{9-5} = vj;
 238:   let Inst{4-0} = rd;
 239: }
 240: 
 241: // 2RI5-type
 242: // <opcode | I5 | vj | vd>
 243: class Fmt2RI5_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 244:                   list<dag> pattern = []>
 245:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 246:   bits<5> imm5;
 247:   bits<5> vj;
 248:   bits<5> vd;
 249: 
 250:   let Inst{31-0} = op;
 251:   let Inst{14-10} = imm5;
 252:   let Inst{9-5} = vj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI4_RVI`, `Fmt2RI5_VVI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI4_RVI`, `Fmt2RI5_VVI` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253:   let Inst{4-0} = vd;
 254: }
 255: 
 256: // 2RI6-type
 257: // <opcode | I6 | vj | vd>
 258: class Fmt2RI6_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 259:                   list<dag> pattern = []>
 260:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 261:   bits<6> imm6;
 262:   bits<5> vj;
 263:   bits<5> vd;
 264: 
 265:   let Inst{31-0} = op;
 266:   let Inst{15-10} = imm6;
 267:   let Inst{9-5} = vj;
 268:   let Inst{4-0} = vd;
 269: }
 270: 
 271: // 2RI7-type
 272: // <opcode | I7 | vj | vd>
 273: class Fmt2RI7_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 274:                   list<dag> pattern = []>
 275:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 276:   bits<7> imm7;
 277:   bits<5> vj;
 278:   bits<5> vd;
 279: 
 280:   let Inst{31-0} = op;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI6_VVI`, `Fmt2RI7_VVI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI6_VVI`, `Fmt2RI7_VVI` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281:   let Inst{16-10} = imm7;
 282:   let Inst{9-5} = vj;
 283:   let Inst{4-0} = vd;
 284: }
 285: 
 286: // 2RI8-type
 287: // <opcode | I8 | vj | vd>
 288: class Fmt2RI8_VVI<bits<32> op, dag outs, dag ins, string opnstr,
 289:                   list<dag> pattern = []>
 290:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 291:   bits<8> imm8;
 292:   bits<5> vj;
 293:   bits<5> vd;
 294: 
 295:   let Inst{31-0} = op;
 296:   let Inst{17-10} = imm8;
 297:   let Inst{9-5} = vj;
 298:   let Inst{4-0} = vd;
 299: }
 300: 
 301: // 2RI8I1-type
 302: // <opcode | I1 | I8 | vj | vd>
 303: class Fmt2RI8I1_VRII<bits<32> op, dag outs, dag ins, string opnstr,
 304:                      list<dag> pattern = []>
 305:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 306:   bits<1> imm1;
 307:   bits<8> imm8;
 308:   bits<5> rj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8_VVI`, `Fmt2RI8I1_VRII`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8_VVI`, `Fmt2RI8I1_VRII` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309:   bits<5> vd;
 310: 
 311:   let Inst{31-0} = op;
 312:   let Inst{18} = imm1;
 313:   let Inst{17-10} = imm8;
 314:   let Inst{9-5} = rj;
 315:   let Inst{4-0} = vd;
 316: }
 317: 
 318: // 2RI8I2-type
 319: // <opcode | I2 | I8 | vj | vd>
 320: class Fmt2RI8I2_VRII<bits<32> op, dag outs, dag ins, string opnstr,
 321:                      list<dag> pattern = []>
 322:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 323:   bits<2> imm2;
 324:   bits<8> imm8;
 325:   bits<5> rj;
 326:   bits<5> vd;
 327: 
 328:   let Inst{31-0} = op;
 329:   let Inst{19-18} = imm2;
 330:   let Inst{17-10} = imm8;
 331:   let Inst{9-5} = rj;
 332:   let Inst{4-0} = vd;
 333: }
 334: 
 335: // 2RI8I3-type
 336: // <opcode | I3 | I8 | vj | vd>
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8I2_VRII`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8I2_VRII` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337: class Fmt2RI8I3_VRII<bits<32> op, dag outs, dag ins, string opnstr,
 338:                      list<dag> pattern = []>
 339:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 340:   bits<3> imm3;
 341:   bits<8> imm8;
 342:   bits<5> rj;
 343:   bits<5> vd;
 344: 
 345:   let Inst{31-0} = op;
 346:   let Inst{20-18} = imm3;
 347:   let Inst{17-10} = imm8;
 348:   let Inst{9-5} = rj;
 349:   let Inst{4-0} = vd;
 350: }
 351: 
 352: // 2RI8I4-type
 353: // <opcode | I4 | I8 | vj | vd>
 354: class Fmt2RI8I4_VRII<bits<32> op, dag outs, dag ins, string opnstr,
 355:                      list<dag> pattern = []>
 356:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 357:   bits<4> imm4;
 358:   bits<8> imm8;
 359:   bits<5> rj;
 360:   bits<5> vd;
 361: 
 362:   let Inst{31-0} = op;
 363:   let Inst{21-18} = imm4;
 364:   let Inst{17-10} = imm8;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8I3_VRII`, `Fmt2RI8I4_VRII`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8I3_VRII`, `Fmt2RI8I4_VRII` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365:   let Inst{9-5} = rj;
 366:   let Inst{4-0} = vd;
 367: }
 368: // 2RI9-type
 369: // <opcode | I9 | rj | vd>
 370: class Fmt2RI9_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 371:                   list<dag> pattern = []>
 372:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 373:   bits<9> imm9;
 374:   bits<5> rj;
 375:   bits<5> vd;
 376: 
 377:   let Inst{31-0} = op;
 378:   let Inst{18-10} = imm9;
 379:   let Inst{9-5} = rj;
 380:   let Inst{4-0} = vd;
 381: }
 382: 
 383: // 2RI10-type
 384: // <opcode | I10 | rj | vd>
 385: class Fmt2RI10_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 386:                   list<dag> pattern = []>
 387:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 388:   bits<10> imm10;
 389:   bits<5> rj;
 390:   bits<5> vd;
 391: 
 392:   let Inst{31-0} = op;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI9_VRI`, `Fmt2RI10_VRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI9_VRI`, `Fmt2RI10_VRI` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393:   let Inst{19-10} = imm10;
 394:   let Inst{9-5} = rj;
 395:   let Inst{4-0} = vd;
 396: }
 397: 
 398: // 2RI11-type
 399: // <opcode | I11 | rj | vd>
 400: class Fmt2RI11_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 401:                   list<dag> pattern = []>
 402:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 403:   bits<11> imm11;
 404:   bits<5> rj;
 405:   bits<5> vd;
 406: 
 407:   let Inst{31-0} = op;
 408:   let Inst{20-10} = imm11;
 409:   let Inst{9-5} = rj;
 410:   let Inst{4-0} = vd;
 411: }
 412: 
 413: // 2RI12-type
 414: // <opcode | I12 | rj | vd>
 415: class Fmt2RI12_VRI<bits<32> op, dag outs, dag ins, string opnstr,
 416:                   list<dag> pattern = []>
 417:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 418:   bits<12> imm12;
 419:   bits<5> rj;
 420:   bits<5> vd;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI11_VRI`, `Fmt2RI12_VRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI11_VRI`, `Fmt2RI12_VRI` 等 TableGen 记录。

### Lines 421-448 / 第 421-448 行
```tablegen
 421: 
 422:   let Inst{31-0} = op;
 423:   let Inst{21-10} = imm12;
 424:   let Inst{9-5} = rj;
 425:   let Inst{4-0} = vd;
 426: }
 427: 
 428: // 3R-type
 429: // <opcode | vk | vj | vd>
 430: class Fmt3R_VVV<bits<32> op, dag outs, dag ins, string opnstr,
 431:                 list<dag> pattern = []>
 432:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 433:   bits<5> vk;
 434:   bits<5> vj;
 435:   bits<5> vd;
 436: 
 437:   let Inst{31-0} = op;
 438:   let Inst{14-10} = vk;
 439:   let Inst{9-5} = vj;
 440:   let Inst{4-0} = vd;
 441: }
 442: 
 443: // <opcode | rk | vj | vd>
 444: class Fmt3R_VVR<bits<32> op, dag outs, dag ins, string opnstr,
 445:                 list<dag> pattern = []>
 446:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 447:   bits<5> rk;
 448:   bits<5> vj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt3R_VVV`, `Fmt3R_VVR`.
- **CN**: 该代码块声明或细化了 `Fmt3R_VVV`, `Fmt3R_VVR` 等 TableGen 记录。

### Lines 449-476 / 第 449-476 行
```tablegen
 449:   bits<5> vd;
 450: 
 451:   let Inst{31-0} = op;
 452:   let Inst{14-10} = rk;
 453:   let Inst{9-5} = vj;
 454:   let Inst{4-0} = vd;
 455: }
 456: 
 457: // <opcode | rk | rj | vd>
 458: class Fmt3R_VRR<bits<32> op, dag outs, dag ins, string opnstr,
 459:                 list<dag> pattern = []>
 460:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 461:   bits<5> rk;
 462:   bits<5> rj;
 463:   bits<5> vd;
 464: 
 465:   let Inst{31-0} = op;
 466:   let Inst{14-10} = rk;
 467:   let Inst{9-5} = rj;
 468:   let Inst{4-0} = vd;
 469: }
 470: 
 471: // 4R-type
 472: // <opcode | va | vk | vj | vd>
 473: class Fmt4R_VVVV<bits<32> op, dag outs, dag ins, string opnstr,
 474:                  list<dag> pattern = []>
 475:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 476:   bits<5> va;
```
- **EN**: This block declares or refines TableGen records such as `Fmt3R_VRR`, `Fmt4R_VVVV`.
- **CN**: 该代码块声明或细化了 `Fmt3R_VRR`, `Fmt4R_VVVV` 等 TableGen 记录。

### Lines 477-486 / 第 477-486 行
```tablegen
 477:   bits<5> vk;
 478:   bits<5> vj;
 479:   bits<5> vd;
 480: 
 481:   let Inst{31-0} = op;
 482:   let Inst{19-15} = va;
 483:   let Inst{14-10} = vk;
 484:   let Inst{9-5} = vj;
 485:   let Inst{4-0} = vd;
 486: }
```
- **EN**: This span continues the file's main responsibility: defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
