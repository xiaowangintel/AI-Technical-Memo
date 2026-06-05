# LoongArchLASXInstrFormats.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchLASXInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 LoongArch 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: // LoongArchLASXInstrFormats.td - LoongArch LASX Instr Formats - tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: //  Describe LoongArch LASX instructions format
  11: //
  12: //  opcode       - operation code.
  13: //  xd/rd/cd     - destination register operand.
  14: //  {r/x}{j/k}   - source register operand.
  15: //  immN         - immediate data operand.
  16: //
  17: //===----------------------------------------------------------------------===//
  18: 
  19: // 1RI13-type
  20: // <opcode | I13 | xd>
  21: class Fmt1RI13_XI<bits<32> op, dag outs, dag ins, string opnstr,
  22:                   list<dag> pattern = []>
  23:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  24:   bits<13> imm13;
  25:   bits<5> xd;
  26: 
  27:   let Inst{31-0} = op;
  28:   let Inst{17-5} = imm13;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `Fmt1RI13_XI`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `Fmt1RI13_XI` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29:   let Inst{4-0} = xd;
  30: }
  31: 
  32: // 2R-type
  33: // <opcode | xj | xd>
  34: class Fmt2R_XX<bits<32> op, dag outs, dag ins, string opnstr,
  35:                list<dag> pattern = []>
  36:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  37:   bits<5> xj;
  38:   bits<5> xd;
  39: 
  40:   let Inst{31-0} = op;
  41:   let Inst{9-5} = xj;
  42:   let Inst{4-0} = xd;
  43: }
  44: 
  45: // <opcode | rj | xd>
  46: class Fmt2R_XR<bits<32> op, dag outs, dag ins, string opnstr,
  47:                list<dag> pattern = []>
  48:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  49:   bits<5> rj;
  50:   bits<5> xd;
  51: 
  52:   let Inst{31-0} = op;
  53:   let Inst{9-5} = rj;
  54:   let Inst{4-0} = xd;
  55: }
  56: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt2R_XX`, `Fmt2R_XR`.
- **CN**: 该代码块声明或细化了 `Fmt2R_XX`, `Fmt2R_XR` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57: // <opcode | xj | cd>
  58: class Fmt2R_CX<bits<32> op, dag outs, dag ins, string opnstr,
  59:                list<dag> pattern = []>
  60:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  61:   bits<5> xj;
  62:   bits<3> cd;
  63: 
  64:   let Inst{31-0} = op;
  65:   let Inst{9-5} = xj;
  66:   let Inst{2-0} = cd;
  67: }
  68: 
  69: // 2RI1-type
  70: // <opcode | I1 | xj | xd>
  71: class Fmt2RI1_XXI<bits<32> op, dag outs, dag ins, string opnstr,
  72:                   list<dag> pattern = []>
  73:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  74:   bits<1> imm1;
  75:   bits<5> xj;
  76:   bits<5> xd;
  77: 
  78:   let Inst{31-0} = op;
  79:   let Inst{10} = imm1;
  80:   let Inst{9-5} = xj;
  81:   let Inst{4-0} = xd;
  82: }
  83: 
  84: // 2RI2-type
```
- **EN**: This block declares or refines TableGen records such as `Fmt2R_CX`, `Fmt2RI1_XXI`.
- **CN**: 该代码块声明或细化了 `Fmt2R_CX`, `Fmt2RI1_XXI` 等 TableGen 记录。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: // <opcode | I2 | xj | xd>
  86: class Fmt2RI2_XXI<bits<32> op, dag outs, dag ins, string opnstr,
  87:                   list<dag> pattern = []>
  88:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
  89:   bits<2> imm2;
  90:   bits<5> xj;
  91:   bits<5> xd;
  92: 
  93:   let Inst{31-0} = op;
  94:   let Inst{11-10} = imm2;
  95:   let Inst{9-5} = xj;
  96:   let Inst{4-0} = xd;
  97: }
  98: 
  99: // <opcode | I2 | rj | xd>
 100: class Fmt2RI2_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 101:                   list<dag> pattern = []>
 102:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 103:   bits<2> imm2;
 104:   bits<5> rj;
 105:   bits<5> xd;
 106: 
 107:   let Inst{31-0} = op;
 108:   let Inst{11-10} = imm2;
 109:   let Inst{9-5} = rj;
 110:   let Inst{4-0} = xd;
 111: }
 112: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI2_XXI`, `Fmt2RI2_XRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI2_XXI`, `Fmt2RI2_XRI` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113: // <opcode | I2 | xj | rd>
 114: class Fmt2RI2_RXI<bits<32> op, dag outs, dag ins, string opnstr,
 115:                   list<dag> pattern = []>
 116:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 117:   bits<2> imm2;
 118:   bits<5> xj;
 119:   bits<5> rd;
 120: 
 121:   let Inst{31-0} = op;
 122:   let Inst{11-10} = imm2;
 123:   let Inst{9-5} = xj;
 124:   let Inst{4-0} = rd;
 125: }
 126: 
 127: // 2RI3-type
 128: // <opcode | I3 | xj | xd>
 129: class Fmt2RI3_XXI<bits<32> op, dag outs, dag ins, string opnstr,
 130:                   list<dag> pattern = []>
 131:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 132:   bits<3> imm3;
 133:   bits<5> xj;
 134:   bits<5> xd;
 135: 
 136:   let Inst{31-0} = op;
 137:   let Inst{12-10} = imm3;
 138:   let Inst{9-5} = xj;
 139:   let Inst{4-0} = xd;
 140: }
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI2_RXI`, `Fmt2RI3_XXI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI2_RXI`, `Fmt2RI3_XXI` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141: 
 142: // <opcode | I3 | rj | xd>
 143: class Fmt2RI3_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 144:                   list<dag> pattern = []>
 145:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 146:   bits<3> imm3;
 147:   bits<5> rj;
 148:   bits<5> xd;
 149: 
 150:   let Inst{31-0} = op;
 151:   let Inst{12-10} = imm3;
 152:   let Inst{9-5} = rj;
 153:   let Inst{4-0} = xd;
 154: }
 155: 
 156: // <opcode | I3 | xj | rd>
 157: class Fmt2RI3_RXI<bits<32> op, dag outs, dag ins, string opnstr,
 158:                   list<dag> pattern = []>
 159:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 160:   bits<3> imm3;
 161:   bits<5> xj;
 162:   bits<5> rd;
 163: 
 164:   let Inst{31-0} = op;
 165:   let Inst{12-10} = imm3;
 166:   let Inst{9-5} = xj;
 167:   let Inst{4-0} = rd;
 168: }
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI3_XRI`, `Fmt2RI3_RXI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI3_XRI`, `Fmt2RI3_RXI` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169: 
 170: // 2RI4-type
 171: // <opcode | I4 | xj | xd>
 172: class Fmt2RI4_XXI<bits<32> op, dag outs, dag ins, string opnstr,
 173:                   list<dag> pattern = []>
 174:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 175:   bits<4> imm4;
 176:   bits<5> xj;
 177:   bits<5> xd;
 178: 
 179:   let Inst{31-0} = op;
 180:   let Inst{13-10} = imm4;
 181:   let Inst{9-5} = xj;
 182:   let Inst{4-0} = xd;
 183: }
 184: 
 185: // <opcode | I4 | rj | xd>
 186: class Fmt2RI4_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 187:                   list<dag> pattern = []>
 188:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 189:   bits<4> imm4;
 190:   bits<5> rj;
 191:   bits<5> xd;
 192: 
 193:   let Inst{31-0} = op;
 194:   let Inst{13-10} = imm4;
 195:   let Inst{9-5} = rj;
 196:   let Inst{4-0} = xd;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI4_XXI`, `Fmt2RI4_XRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI4_XXI`, `Fmt2RI4_XRI` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: }
 198: 
 199: // <opcode | I4 | xj | rd>
 200: class Fmt2RI4_RXI<bits<32> op, dag outs, dag ins, string opnstr,
 201:                   list<dag> pattern = []>
 202:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 203:   bits<4> imm4;
 204:   bits<5> xj;
 205:   bits<5> rd;
 206: 
 207:   let Inst{31-0} = op;
 208:   let Inst{13-10} = imm4;
 209:   let Inst{9-5} = xj;
 210:   let Inst{4-0} = rd;
 211: }
 212: 
 213: // 2RI5-type
 214: // <opcode | I5 | xj | xd>
 215: class Fmt2RI5_XXI<bits<32> op, dag outs, dag ins, string opnstr,
 216:                   list<dag> pattern = []>
 217:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 218:   bits<5> imm5;
 219:   bits<5> xj;
 220:   bits<5> xd;
 221: 
 222:   let Inst{31-0} = op;
 223:   let Inst{14-10} = imm5;
 224:   let Inst{9-5} = xj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI4_RXI`, `Fmt2RI5_XXI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI4_RXI`, `Fmt2RI5_XXI` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225:   let Inst{4-0} = xd;
 226: }
 227: 
 228: // 2RI6-type
 229: // <opcode | I6 | xj | xd>
 230: class Fmt2RI6_XXI<bits<32> op, dag outs, dag ins, string opnstr,
 231:                   list<dag> pattern = []>
 232:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 233:   bits<6> imm6;
 234:   bits<5> xj;
 235:   bits<5> xd;
 236: 
 237:   let Inst{31-0} = op;
 238:   let Inst{15-10} = imm6;
 239:   let Inst{9-5} = xj;
 240:   let Inst{4-0} = xd;
 241: }
 242: 
 243: // 2RI7-type
 244: // <opcode | I7 | xj | xd>
 245: class Fmt2RI7_XXI<bits<32> op, dag outs, dag ins, string opnstr,
 246:                   list<dag> pattern = []>
 247:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 248:   bits<7> imm7;
 249:   bits<5> xj;
 250:   bits<5> xd;
 251: 
 252:   let Inst{31-0} = op;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI6_XXI`, `Fmt2RI7_XXI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI6_XXI`, `Fmt2RI7_XXI` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253:   let Inst{16-10} = imm7;
 254:   let Inst{9-5} = xj;
 255:   let Inst{4-0} = xd;
 256: }
 257: 
 258: // 2RI8-type
 259: // <opcode | I8 | xj | xd>
 260: class Fmt2RI8_XXI<bits<32> op, dag outs, dag ins, string opnstr,
 261:                   list<dag> pattern = []>
 262:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 263:   bits<8> imm8;
 264:   bits<5> xj;
 265:   bits<5> xd;
 266: 
 267:   let Inst{31-0} = op;
 268:   let Inst{17-10} = imm8;
 269:   let Inst{9-5} = xj;
 270:   let Inst{4-0} = xd;
 271: }
 272: 
 273: // 2RI8I2-type
 274: // <opcode | I2 | I8 | xj | xd>
 275: class Fmt2RI8I2_XRII<bits<32> op, dag outs, dag ins, string opnstr,
 276:                      list<dag> pattern = []>
 277:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 278:   bits<2> imm2;
 279:   bits<8> imm8;
 280:   bits<5> rj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8_XXI`, `Fmt2RI8I2_XRII`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8_XXI`, `Fmt2RI8I2_XRII` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281:   bits<5> xd;
 282: 
 283:   let Inst{31-0} = op;
 284:   let Inst{19-18} = imm2;
 285:   let Inst{17-10} = imm8;
 286:   let Inst{9-5} = rj;
 287:   let Inst{4-0} = xd;
 288: }
 289: 
 290: // 2RI8I3-type
 291: // <opcode | I3 | I8 | xj | xd>
 292: class Fmt2RI8I3_XRII<bits<32> op, dag outs, dag ins, string opnstr,
 293:                      list<dag> pattern = []>
 294:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 295:   bits<3> imm3;
 296:   bits<8> imm8;
 297:   bits<5> rj;
 298:   bits<5> xd;
 299: 
 300:   let Inst{31-0} = op;
 301:   let Inst{20-18} = imm3;
 302:   let Inst{17-10} = imm8;
 303:   let Inst{9-5} = rj;
 304:   let Inst{4-0} = xd;
 305: }
 306: 
 307: // 2RI8I4-type
 308: // <opcode | I4 | I8 | xj | xd>
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8I3_XRII`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8I3_XRII` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309: class Fmt2RI8I4_XRII<bits<32> op, dag outs, dag ins, string opnstr,
 310:                      list<dag> pattern = []>
 311:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 312:   bits<4> imm4;
 313:   bits<8> imm8;
 314:   bits<5> rj;
 315:   bits<5> xd;
 316: 
 317:   let Inst{31-0} = op;
 318:   let Inst{21-18} = imm4;
 319:   let Inst{17-10} = imm8;
 320:   let Inst{9-5} = rj;
 321:   let Inst{4-0} = xd;
 322: }
 323: 
 324: // 2RI8I5-type
 325: // <opcode | I5 | I8 | xj | xd>
 326: class Fmt2RI8I5_XRII<bits<32> op, dag outs, dag ins, string opnstr,
 327:                      list<dag> pattern = []>
 328:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 329:   bits<5> imm5;
 330:   bits<8> imm8;
 331:   bits<5> rj;
 332:   bits<5> xd;
 333: 
 334:   let Inst{31-0} = op;
 335:   let Inst{22-18} = imm5;
 336:   let Inst{17-10} = imm8;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI8I4_XRII`, `Fmt2RI8I5_XRII`.
- **CN**: 该代码块声明或细化了 `Fmt2RI8I4_XRII`, `Fmt2RI8I5_XRII` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337:   let Inst{9-5} = rj;
 338:   let Inst{4-0} = xd;
 339: }
 340: 
 341: // 2RI9-type
 342: // <opcode | I9 | rj | xd>
 343: class Fmt2RI9_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 344:                   list<dag> pattern = []>
 345:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 346:   bits<9> imm9;
 347:   bits<5> rj;
 348:   bits<5> xd;
 349: 
 350:   let Inst{31-0} = op;
 351:   let Inst{18-10} = imm9;
 352:   let Inst{9-5} = rj;
 353:   let Inst{4-0} = xd;
 354: }
 355: 
 356: // 2RI10-type
 357: // <opcode | I10 | rj | xd>
 358: class Fmt2RI10_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 359:                   list<dag> pattern = []>
 360:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 361:   bits<10> imm10;
 362:   bits<5> rj;
 363:   bits<5> xd;
 364: 
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI9_XRI`, `Fmt2RI10_XRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI9_XRI`, `Fmt2RI10_XRI` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365:   let Inst{31-0} = op;
 366:   let Inst{19-10} = imm10;
 367:   let Inst{9-5} = rj;
 368:   let Inst{4-0} = xd;
 369: }
 370: 
 371: // 2RI11-type
 372: // <opcode | I11 | rj | xd>
 373: class Fmt2RI11_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 374:                   list<dag> pattern = []>
 375:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 376:   bits<11> imm11;
 377:   bits<5> rj;
 378:   bits<5> xd;
 379: 
 380:   let Inst{31-0} = op;
 381:   let Inst{20-10} = imm11;
 382:   let Inst{9-5} = rj;
 383:   let Inst{4-0} = xd;
 384: }
 385: 
 386: // 2RI12-type
 387: // <opcode | I12 | rj | xd>
 388: class Fmt2RI12_XRI<bits<32> op, dag outs, dag ins, string opnstr,
 389:                   list<dag> pattern = []>
 390:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 391:   bits<12> imm12;
 392:   bits<5> rj;
```
- **EN**: This block declares or refines TableGen records such as `Fmt2RI11_XRI`, `Fmt2RI12_XRI`.
- **CN**: 该代码块声明或细化了 `Fmt2RI11_XRI`, `Fmt2RI12_XRI` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393:   bits<5> xd;
 394: 
 395:   let Inst{31-0} = op;
 396:   let Inst{21-10} = imm12;
 397:   let Inst{9-5} = rj;
 398:   let Inst{4-0} = xd;
 399: }
 400: 
 401: // 3R-type
 402: // <opcode | xk | xj | xd>
 403: class Fmt3R_XXX<bits<32> op, dag outs, dag ins, string opnstr,
 404:                 list<dag> pattern = []>
 405:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 406:   bits<5> xk;
 407:   bits<5> xj;
 408:   bits<5> xd;
 409: 
 410:   let Inst{31-0} = op;
 411:   let Inst{14-10} = xk;
 412:   let Inst{9-5} = xj;
 413:   let Inst{4-0} = xd;
 414: }
 415: 
 416: // <opcode | rk | xj | xd>
 417: class Fmt3R_XXR<bits<32> op, dag outs, dag ins, string opnstr,
 418:                 list<dag> pattern = []>
 419:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 420:   bits<5> rk;
```
- **EN**: This block declares or refines TableGen records such as `Fmt3R_XXX`, `Fmt3R_XXR`.
- **CN**: 该代码块声明或细化了 `Fmt3R_XXX`, `Fmt3R_XXR` 等 TableGen 记录。

### Lines 421-448 / 第 421-448 行
```tablegen
 421:   bits<5> xj;
 422:   bits<5> xd;
 423: 
 424:   let Inst{31-0} = op;
 425:   let Inst{14-10} = rk;
 426:   let Inst{9-5} = xj;
 427:   let Inst{4-0} = xd;
 428: }
 429: 
 430: // <opcode | rk | rj | xd>
 431: class Fmt3R_XRR<bits<32> op, dag outs, dag ins, string opnstr,
 432:                 list<dag> pattern = []>
 433:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
 434:   bits<5> rk;
 435:   bits<5> rj;
 436:   bits<5> xd;
 437: 
 438:   let Inst{31-0} = op;
 439:   let Inst{14-10} = rk;
 440:   let Inst{9-5} = rj;
 441:   let Inst{4-0} = xd;
 442: }
 443: 
 444: // 4R-type
 445: // <opcode | xa | xk | xj | xd>
 446: class Fmt4R_XXXX<bits<32> op, dag outs, dag ins, string opnstr,
 447:                  list<dag> pattern = []>
 448:     : LAInst<outs, ins, deriveInsnMnemonic<NAME>.ret, opnstr, pattern> {
```
- **EN**: This block declares or refines TableGen records such as `Fmt3R_XRR`, `Fmt4R_XXXX`.
- **CN**: 该代码块声明或细化了 `Fmt3R_XRR`, `Fmt4R_XXXX` 等 TableGen 记录。

### Lines 449-459 / 第 449-459 行
```tablegen
 449:   bits<5> xa;
 450:   bits<5> xk;
 451:   bits<5> xj;
 452:   bits<5> xd;
 453: 
 454:   let Inst{31-0} = op;
 455:   let Inst{19-15} = xa;
 456:   let Inst{14-10} = xk;
 457:   let Inst{9-5} = xj;
 458:   let Inst{4-0} = xd;
 459: }
```
- **EN**: This span continues the file's main responsibility: defines declarative LoongArch backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
