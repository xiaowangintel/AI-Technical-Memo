# AVRInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRInstrFormats.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- AVRInstrInfo.td - AVR Instruction Formats ----------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // AVR Instruction Format Definitions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: // A generic AVR instruction.
  14: class AVRInst<dag outs, dag ins, string asmstr, list<dag> pattern>
  15:     : Instruction {
  16:   let Namespace = "AVR";
  17: 
  18:   dag OutOperandList = outs;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as AVRInst, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 AVRInst 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19:   dag InOperandList = ins;
  20:   let AsmString = asmstr;
  21:   let Pattern = pattern;
  22: }
  23: 
  24: /// A 16-bit AVR instruction.
  25: class AVRInst16<dag outs, dag ins, string asmstr, list<dag> pattern>
  26:     : AVRInst<outs, ins, asmstr, pattern> {
  27:   field bits<16> Inst;
  28: 
  29:   let Size = 2;
  30: }
  31: 
  32: /// a 32-bit AVR instruction.
  33: class AVRInst32<dag outs, dag ins, string asmstr, list<dag> pattern>
  34:     : AVRInst<outs, ins, asmstr, pattern> {
  35:   field bits<32> Inst;
  36: 
```

- EN: This range defines declarative TableGen records such as AVRInst16, AVRInst32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 AVRInst16, AVRInst32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 37-54

```tablegen
  37:   let Size = 4;
  38: }
  39: 
  40: // A class for pseudo instructions.
  41: // Pseudo instructions are not real AVR instructions. The DAG stores
  42: // pseudo instructions which are replaced by real AVR instructions by
  43: // AVRExpandPseudoInsts.cpp.
  44: //
  45: // For example, the ADDW (add wide, as in add 16 bit values) instruction
  46: // is defined as a pseudo instruction. In AVRExpandPseudoInsts.cpp,
  47: // the instruction is then replaced by two add instructions - one for each byte.
  48: class Pseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
  49:     : AVRInst16<outs, ins, asmstr, pattern> {
  50:   let Pattern = pattern;
  51: 
  52:   let isPseudo = 1;
  53:   let isCodeGenOnly = 1;
  54: }
```

- EN: This range defines declarative TableGen records such as Pseudo, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 Pseudo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 55-72

```tablegen
  55: 
  56: //===----------------------------------------------------------------------===//
  57: // Register / register instruction: <|opcode|ffrd|dddd|rrrr|>
  58: // opcode = 4 bits.
  59: // f = secondary opcode = 2 bits
  60: // d = destination = 5 bits
  61: // r = source = 5 bits
  62: // (Accepts all registers)
  63: //===----------------------------------------------------------------------===//
  64: class FRdRr<bits<4> opcode, bits<2> f, dag outs, dag ins, string asmstr,
  65:             list<dag> pattern> : AVRInst16<outs, ins, asmstr, pattern> {
  66:   bits<5> rd;
  67:   bits<5> rr;
  68: 
  69:   let Inst{15 - 12} = opcode;
  70:   let Inst{11 - 10} = f;
  71:   let Inst{9} = rr{4};
  72:   let Inst{8 - 4} = rd;
```

- EN: This range defines declarative TableGen records such as FRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   let Inst{3 - 0} = rr{3 - 0};
  74: }
  75: 
  76: //===----------------------------------------------------------------------===//
  77: // Instruction of the format `<mnemonic> Z, Rd`
  78: // <|1001|001r|rrrr|0ttt>
  79: //===----------------------------------------------------------------------===//
  80: class FZRd<bits<3> t, dag outs, dag ins, string asmstr, list<dag> pattern>
  81:     : AVRInst16<outs, ins, asmstr, pattern> {
  82:   bits<0> z;
  83:   bits<5> rd;
  84: 
  85:   let Inst{15 - 12} = 0b1001;
  86: 
  87:   let Inst{11 - 9} = 0b001;
  88:   let Inst{8} = rd{4};
  89: 
  90:   let Inst{7 - 4} = rd{3 - 0};
```

- EN: This range defines declarative TableGen records such as FZRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FZRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91: 
  92:   let Inst{3} = 0;
  93:   let Inst{2 - 0} = t;
  94: }
  95: 
  96: //===----------------------------------------------------------------------===//
  97: // Register / immediate8 instruction: <|opcode|KKKK|dddd|KKKK|>
  98: // opcode = 4 bits.
  99: // K = constant data = 8 bits
 100: // d = destination = 4 bits
 101: // (Only accepts r16-r31)
 102: //===----------------------------------------------------------------------===//
 103: class FRdK<bits<4> opcode, dag outs, dag ins, string asmstr, list<dag> pattern>
 104:     : AVRInst16<outs, ins, asmstr, pattern> {
 105:   bits<4> rd;
 106:   bits<8> k;
 107: 
 108:   let Inst{15 - 12} = opcode;
```

- EN: This range defines declarative TableGen records such as FRdK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:   let Inst{11 - 8} = k{7 - 4};
 110:   let Inst{7 - 4} = rd{3 - 0};
 111:   let Inst{3 - 0} = k{3 - 0};
 112: 
 113:   let isAsCheapAsAMove = 1;
 114: }
 115: 
 116: //===----------------------------------------------------------------------===//
 117: // Register instruction: <|opcode|fffd|dddd|ffff|>
 118: // opcode = 4 bits.
 119: // f = secondary opcode = 7 bits
 120: // d = destination = 5 bits
 121: // (Accepts all registers)
 122: //===----------------------------------------------------------------------===//
 123: class FRd<bits<4> opcode, bits<7> f, dag outs, dag ins, string asmstr,
 124:           list<dag> pattern> : AVRInst16<outs, ins, asmstr, pattern> {
 125:   bits<5> rd;
 126: 
```

- EN: This range defines declarative TableGen records such as FRd, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FRd 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127:   let Inst{15 - 12} = opcode;
 128:   let Inst{11 - 9} = f{6 - 4};
 129:   let Inst{8 - 4} = rd;
 130:   let Inst{3 - 0} = f{3 - 0};
 131: }
 132: 
 133: //===----------------------------------------------------------------------===//
 134: // [STD/LDD] P+q, Rr special encoding: <|10q0|qqtr|rrrr|pqqq>
 135: // t = type (1 for STD, 0 for LDD)
 136: // q = displacement (6 bits)
 137: // r = register (5 bits)
 138: // p = pointer register (1 bit) [1 for Y, 0 for Z]
 139: //===----------------------------------------------------------------------===//
 140: class FSTDLDD<bit type, dag outs, dag ins, string asmstr, list<dag> pattern>
 141:     : AVRInst16<outs, ins, asmstr, pattern> {
 142:   bits<7> memri;
 143:   bits<5> reg; // the GP register
 144: 
```

- EN: This range defines declarative TableGen records such as FSTDLDD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FSTDLDD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:   let Inst{15 - 14} = 0b10;
 146:   let Inst{13} = memri{5};
 147:   let Inst{12} = 0;
 148: 
 149:   let Inst{11 - 10} = memri{4 - 3};
 150:   let Inst{9} = type;
 151:   let Inst{8} = reg{4};
 152: 
 153:   let Inst{7 - 4} = reg{3 - 0};
 154: 
 155:   let Inst{3} = memri{6};
 156:   let Inst{2 - 0} = memri{2 - 0};
 157: }
 158: 
 159: //===---------------------------------------------------------------------===//
 160: // An ST/LD instruction.
 161: // <|100i|00tr|rrrr|ppaa|>
 162: // t = type (1 for store, 0 for load)
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163: // a = regular/postinc/predec (reg = 0b00, postinc = 0b01, predec = 0b10)
 164: // p = pointer register
 165: // r = src/dst register
 166: //
 167: // Note that the bit labelled 'i' above does not follow a simple pattern,
 168: // so there exists a post encoder method to set it manually. Also a specified
 169: // decoder method is needed.
 170: //===---------------------------------------------------------------------===//
 171: class FSTLD<bit type, bits<2> mode, dag outs, dag ins, string asmstr,
 172:             list<dag> pattern> : AVRInst16<outs, ins, asmstr, pattern> {
 173:   bits<5> reg;
 174: 
 175:   let Inst{15 - 13} = 0b100;
 176:   // This bit varies depending on the arguments and the mode.
 177:   // We have a post encoder method to set this bit manually.
 178:   let Inst{12} = 0;
 179: 
 180:   let Inst{11 - 10} = 0b00;
```

- EN: This range defines declarative TableGen records such as FSTLD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FSTLD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181:   let Inst{9} = type;
 182:   let Inst{8} = reg{4};
 183: 
 184:   let Inst{7 - 4} = reg{3 - 0};
 185: 
 186:   let Inst{1 - 0} = mode{1 - 0};
 187: 
 188:   let DecoderMethod = "decodeLoadStore";
 189:   let PostEncoderMethod = "loadStorePostEncoder";
 190: }
 191: 
 192: //===---------------------------------------------------------------------===//
 193: // Special format for the LPM/ELPM instructions
 194: // [E]LPM Rd, Z[+]
 195: // <|1001|000d|dddd|01ep>
 196: // d = destination register
 197: // e = is elpm
 198: // p = is postincrement
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199: //===---------------------------------------------------------------------===//
 200: class FLPMX<bit e, bit p, dag outs, dag ins, string asmstr, list<dag> pattern>
 201:     : AVRInst16<outs, ins, asmstr, pattern> {
 202:   bits<0> z;
 203:   bits<5> rd;
 204: 
 205:   let Inst{15 - 9} = 0b1001000;
 206:   let Inst{8 - 4} = rd;
 207:   let Inst{3 - 2} = 0b01;
 208:   let Inst{1} = e;
 209:   let Inst{0} = p;
 210: }
 211: 
 212: //===----------------------------------------------------------------------===//
 213: // MOVWRdRr special encoding: <|0000|0001|dddd|rrrr|>
 214: // d = destination = 4 bits
 215: // r = source = 4 bits
 216: // (Only accepts register pairs)
```

- EN: This range defines declarative TableGen records such as FLPMX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FLPMX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: //===----------------------------------------------------------------------===//
 218: class FMOVWRdRr<dag outs, dag ins, string asmstr, list<dag> pattern>
 219:     : AVRInst16<outs, ins, asmstr, pattern> {
 220:   bits<4> rd;
 221:   bits<4> rr;
 222: 
 223:   let Inst{15 - 8} = 0b00000001;
 224:   let Inst{7 - 4} = rd;
 225:   let Inst{3 - 0} = rr;
 226: }
 227: 
 228: //===----------------------------------------------------------------------===//
 229: // MULS special encoding: <|0000|0010|dddd|rrrr|>
 230: // d = multiplicand = 4 bits
 231: // r = multiplier = 4 bits
 232: // (Only accepts r16-r31)
 233: //===----------------------------------------------------------------------===//
 234: class FMULSRdRr<dag outs, dag ins, string asmstr, list<dag> pattern>
```

- EN: This range defines declarative TableGen records such as FMOVWRdRr, FMULSRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FMOVWRdRr, FMULSRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-252

```tablegen
 235:     : AVRInst16<outs, ins, asmstr, pattern> {
 236:   bits<4> rd;
 237:   bits<4> rr;
 238: 
 239:   let Inst{15 - 8} = 0b00000010;
 240:   let Inst{7 - 4} = rd;
 241:   let Inst{3 - 0} = rr;
 242: }
 243: 
 244: //===----------------------------------------------------------------------===//
 245: // MULSU special encoding: <|0000|0011|0ddd|0rrr|>
 246: // d = multiplicand = 3 bits
 247: // r = multiplier = 3 bits
 248: // (Only accepts r16-r23)
 249: //===----------------------------------------------------------------------===//
 250: class FMULSURdRr<dag outs, dag ins, string asmstr, list<dag> pattern>
 251:     : AVRInst16<outs, ins, asmstr, pattern> {
 252:   bits<3> rd;
```

- EN: This range defines declarative TableGen records such as FMULSURdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FMULSURdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253:   bits<3> rr;
 254: 
 255:   let Inst{15 - 8} = 0b00000011;
 256:   let Inst{7} = 0;
 257:   let Inst{6 - 4} = rd;
 258:   let Inst{3} = 0;
 259:   let Inst{2 - 0} = rr;
 260: }
 261: 
 262: // Special encoding for the FMUL family of instructions.
 263: //
 264: // <0000|0011|fddd|frrr|>
 265: //
 266: // ff = 0b01 for FMUL
 267: //      0b10 for FMULS
 268: //      0b11 for FMULSU
 269: //
 270: // ddd = destination register
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271: // rrr = source register
 272: class FFMULRdRr<bits<2> f, dag outs, dag ins, string asmstr, list<dag> pattern>
 273:     : AVRInst16<outs, ins, asmstr, pattern> {
 274:   bits<3> rd;
 275:   bits<3> rr;
 276: 
 277:   let Inst{15 - 8} = 0b00000011;
 278:   let Inst{7} = f{1};
 279:   let Inst{6 - 4} = rd;
 280:   let Inst{3} = f{0};
 281:   let Inst{2 - 0} = rr;
 282: }
 283: 
 284: //===----------------------------------------------------------------------===//
 285: // Arithmetic word instructions (ADIW / SBIW): <|1001|011f|kkdd|kkkk|>
 286: // f = secondary opcode = 1 bit
 287: // k = constant data = 6 bits
 288: // d = destination = 2 bits
```

- EN: This range defines declarative TableGen records such as FFMULRdRr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FFMULRdRr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: // (Only accepts r25:24 r27:26 r29:28 r31:30)
 290: //===----------------------------------------------------------------------===//
 291: class FWRdK<bit f, dag outs, dag ins, string asmstr, list<dag> pattern>
 292:     : AVRInst16<outs, ins, asmstr, pattern> {
 293:   bits<2> rd;
 294:   bits<6> k;
 295: 
 296:   let Inst{15 - 9} = 0b1001011;
 297:   let Inst{8} = f;
 298:   let Inst{7 - 6} = k{5 - 4};
 299:   let Inst{5 - 4} = rd;
 300:   let Inst{3 - 0} = k{3 - 0};
 301: }
 302: 
 303: //===----------------------------------------------------------------------===//
 304: // In I/O instruction: <|1011|0AAd|dddd|AAAA|>
 305: // A = I/O location address = 6 bits
 306: // d = destination = 5 bits
```

- EN: This range defines declarative TableGen records such as FWRdK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FWRdK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307: // (Accepts all registers)
 308: //===----------------------------------------------------------------------===//
 309: class FIORdA<dag outs, dag ins, string asmstr, list<dag> pattern>
 310:     : AVRInst16<outs, ins, asmstr, pattern> {
 311:   bits<5> rd;
 312:   bits<6> A;
 313: 
 314:   let Inst{15 - 11} = 0b10110;
 315:   let Inst{10 - 9} = A{5 - 4};
 316:   let Inst{8 - 4} = rd;
 317:   let Inst{3 - 0} = A{3 - 0};
 318: }
 319: 
 320: //===----------------------------------------------------------------------===//
 321: // Out I/O instruction: <|1011|1AAr|rrrr|AAAA|>
 322: // A = I/O location address = 6 bits
 323: // d = destination = 5 bits
 324: // (Accepts all registers)
```

- EN: This range defines declarative TableGen records such as FIORdA, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FIORdA 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 325-342

```tablegen
 325: //===----------------------------------------------------------------------===//
 326: class FIOARr<dag outs, dag ins, string asmstr, list<dag> pattern>
 327:     : AVRInst16<outs, ins, asmstr, pattern> {
 328:   bits<6> A;
 329:   bits<5> rr;
 330: 
 331:   let Inst{15 - 11} = 0b10111;
 332:   let Inst{10 - 9} = A{5 - 4};
 333:   let Inst{8 - 4} = rr;
 334:   let Inst{3 - 0} = A{3 - 0};
 335: }
 336: 
 337: //===----------------------------------------------------------------------===//
 338: // I/O bit instruction.
 339: // <|1001|10tt|AAAA|Abbb>
 340: // t = type (1 for SBI, 0 for CBI)
 341: // A = I/O location address (5 bits)
 342: // b = bit number
```

- EN: This range defines declarative TableGen records such as FIOARr, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FIOARr 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343: //===----------------------------------------------------------------------===//
 344: class FIOBIT<bits<2> t, dag outs, dag ins, string asmstr, list<dag> pattern>
 345:     : AVRInst16<outs, ins, asmstr, pattern> {
 346:   bits<5> addr;
 347:   bits<3> b;
 348: 
 349:   let Inst{15 - 10} = 0b100110;
 350:   let Inst{9 - 8} = t;
 351:   let Inst{7 - 3} = addr;
 352:   let Inst{2 - 0} = b{2 - 0};
 353: }
 354: 
 355: //===----------------------------------------------------------------------===//
 356: // BST/BLD instruction.
 357: // <|1111|1ttd|dddd|0bbb>
 358: // t = type (1 for BST, 0 for BLD)
 359: // d = destination register
 360: // b = bit
```

- EN: This range defines declarative TableGen records such as FIOBIT, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FIOBIT 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361: //===----------------------------------------------------------------------===//
 362: class FRdB<bits<2> t, dag outs, dag ins, string asmstr, list<dag> pattern>
 363:     : AVRInst16<outs, ins, asmstr, pattern> {
 364:   bits<5> rd;
 365:   bits<3> b;
 366: 
 367:   let Inst{15 - 12} = 0b1111;
 368: 
 369:   let Inst{11} = 0b1;
 370:   let Inst{10 - 9} = t;
 371:   let Inst{8} = rd{4};
 372: 
 373:   let Inst{7 - 4} = rd{3 - 0};
 374: 
 375:   let Inst{3} = 0;
 376:   let Inst{2 - 0} = b;
 377: }
 378: 
```

- EN: This range defines declarative TableGen records such as FRdB, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FRdB 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379: // Special encoding for the `DES K` instruction.
 380: //
 381: // <|1001|0100|KKKK|1011>
 382: //
 383: // KKKK = 4 bit immediate
 384: class FDES<dag outs, dag ins, string asmstr, list<dag> pattern>
 385:     : AVRInst16<outs, ins, asmstr, pattern> {
 386:   bits<4> k;
 387: 
 388:   let Inst{15 - 12} = 0b1001;
 389: 
 390:   let Inst{11 - 8} = 0b0100;
 391: 
 392:   let Inst{7 - 4} = k;
 393: 
 394:   let Inst{3 - 0} = 0b1011;
 395: }
 396: 
```

- EN: This range defines declarative TableGen records such as FDES, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FDES 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397: //===----------------------------------------------------------------------===//
 398: // Conditional Branching instructions: <|1111|0fkk|kkkk|ksss|>
 399: // f = secondary opcode = 1 bit
 400: // k = constant address = 7 bits
 401: // s = bit in status register = 3 bits
 402: //===----------------------------------------------------------------------===//
 403: class FBRsk<bit f, bits<3> s, dag outs, dag ins, string asmstr,
 404:             list<dag> pattern> : AVRInst16<outs, ins, asmstr, pattern> {
 405:   bits<7> k;
 406: 
 407:   let Inst{15 - 11} = 0b11110;
 408:   let Inst{10} = f;
 409:   let Inst{9 - 3} = k;
 410:   let Inst{2 - 0} = s;
 411: }
 412: 
 413: //===----------------------------------------------------------------------===//
 414: // Special, opcode only instructions: <|opcode|>
```

- EN: This range defines declarative TableGen records such as FBRsk, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FBRsk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415: //===----------------------------------------------------------------------===//
 416: 
 417: class F16<bits<16> opcode, dag outs, dag ins, string asmstr, list<dag> pattern>
 418:     : AVRInst16<outs, ins, asmstr, pattern> {
 419:   let Inst = opcode;
 420: }
 421: 
 422: //===----------------------------------------------------------------------===//
 423: // Branching instructions with immediate12: <|110f|kkkk|kkkk|kkkk|>
 424: // f = secondary opcode = 1 bit
 425: // k = constant address = 12 bits
 426: //===----------------------------------------------------------------------===//
 427: class FBRk<bit f, dag outs, dag ins, string asmstr, list<dag> pattern>
 428:     : AVRInst16<outs, ins, asmstr, pattern> {
 429:   bits<12> k;
 430: 
 431:   let Inst{15 - 13} = 0b110;
 432:   let Inst{12} = f;
```

- EN: This range defines declarative TableGen records such as F16, FBRk, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16, FBRk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433:   let Inst{11 - 0} = k;
 434: }
 435: 
 436: //===----------------------------------------------------------------------===//
 437: // 32 bits branching instructions: <|1001|010k|kkkk|fffk|kkkk|kkkk|kkkk|kkkk|>
 438: // f = secondary opcode = 3 bits
 439: // k = constant address = 22 bits
 440: //===----------------------------------------------------------------------===//
 441: class F32BRk<bits<3> f, dag outs, dag ins, string asmstr, list<dag> pattern>
 442:     : AVRInst32<outs, ins, asmstr, pattern> {
 443:   bits<22> k;
 444: 
 445:   let Inst{31 - 25} = 0b1001010;
 446:   let Inst{24 - 20} = k{21 - 17};
 447:   let Inst{19 - 17} = f;
 448:   let Inst{16 - 0} = k{16 - 0};
 449: }
 450: 
```

- EN: This range defines declarative TableGen records such as F32BRk, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32BRk 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451: //===----------------------------------------------------------------------===//
 452: // 32 bits direct mem instructions: <|1001|00fd|dddd|0000|kkkk|kkkk|kkkk|kkkk|>
 453: // f = secondary opcode = 1 bit
 454: // d = destination = 5 bits
 455: // k = constant address = 16 bits
 456: // (Accepts all registers)
 457: //===----------------------------------------------------------------------===//
 458: class F32DM<bit f, dag outs, dag ins, string asmstr, list<dag> pattern>
 459:     : AVRInst32<outs, ins, asmstr, pattern> {
 460:   bits<5> rd;
 461:   bits<16> k;
 462: 
 463:   let Inst{31 - 28} = 0b1001;
 464: 
 465:   let Inst{27 - 26} = 0b00;
 466:   let Inst{25} = f;
 467:   let Inst{24} = rd{4};
 468: 
```

- EN: This range defines declarative TableGen records such as F32DM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32DM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469:   let Inst{23 - 20} = rd{3 - 0};
 470: 
 471:   let Inst{19 - 16} = 0b0000;
 472: 
 473:   let Inst{15 - 0} = k;
 474: }
 475: 
 476: //===---------------------------------------------------------------------===//
 477: // Special format for the LDS/STS instructions on AVRTiny.
 478: // <|1010|ikkk|dddd|kkkk>
 479: // d = R16 ~ R31
 480: // i = 0 - lds, 1 - sts
 481: // k = 7-bit data space address
 482: //===---------------------------------------------------------------------===//
 483: class FLDSSTSTINY<bit i, dag outs, dag ins, string asmstr, list<dag> pattern>
 484:     : AVRInst16<outs, ins, asmstr, pattern> {
 485:   bits<5> rd;
 486:   bits<7> k;
```

- EN: This range defines declarative TableGen records such as FLDSSTSTINY, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FLDSSTSTINY 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 487-504

```tablegen
 487: 
 488:   let Inst{15 - 12} = 0b1010;
 489: 
 490:   let Inst{11} = i;
 491: 
 492:   let Inst{10 - 8} = k{6 - 4};
 493:   let Inst{7 - 4} = rd{3 - 0};
 494:   let Inst{3 - 0} = k{3 - 0};
 495: 
 496:   let DecoderNamespace = "AVRTiny";
 497: }
 498: 
 499: // <|1001|0100|bfff|1000>
 500: class FS<bit b, dag outs, dag ins, string asmstr, list<dag> pattern>
 501:     : AVRInst16<outs, ins, asmstr, pattern> {
 502:   bits<3> s;
 503: 
 504:   let Inst{15 - 12} = 0b1001;
```

- EN: This range defines declarative TableGen records such as FS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 505-522

```tablegen
 505: 
 506:   let Inst{11 - 8} = 0b0100;
 507: 
 508:   let Inst{7} = b;
 509:   let Inst{6 - 4} = s;
 510: 
 511:   let Inst{3 - 0} = 0b1000;
 512: }
 513: 
 514: // Set/clr bit in status flag instructions/
 515: // <BRBS|BRBC> s, k
 516: // ---------------------
 517: // <|1111|0fkk|kkkk|ksss>
 518: class FSK<bit f, dag outs, dag ins, string asmstr, list<dag> pattern>
 519:     : AVRInst16<outs, ins, asmstr, pattern> {
 520:   bits<7> k;
 521:   bits<3> s;
 522: 
```

- EN: This range defines declarative TableGen records such as FSK, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FSK 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523:   let Inst{15 - 12} = 0b1111;
 524: 
 525:   let Inst{11} = 0;
 526:   let Inst{10} = f;
 527:   let Inst{9 - 3} = k;
 528:   let Inst{2 - 0} = s;
 529: }
 530: 
 531: class ExtensionPseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
 532:     : Pseudo<outs, ins, asmstr, pattern> {
 533:   let Defs = [SREG];
 534:   let hasSideEffects = 0;
 535: }
 536: 
 537: class StorePseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
 538:     : Pseudo<outs, ins, asmstr, pattern> {
 539:   let Defs = [SP];
 540: }
```

- EN: This range defines declarative TableGen records such as ExtensionPseudo, StorePseudo, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ExtensionPseudo, StorePseudo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-554

```tablegen
 541: 
 542: class SelectPseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
 543:     : Pseudo<outs, ins, asmstr, pattern> {
 544:   let usesCustomInserter = 1;
 545: 
 546:   let Uses = [SREG];
 547: }
 548: 
 549: class ShiftPseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
 550:     : Pseudo<outs, ins, asmstr, pattern> {
 551:   let usesCustomInserter = 1;
 552:   let hasSideEffects = 0;
 553:   let Defs = [SREG];
 554: }
```

- EN: This range defines declarative TableGen records such as SelectPseudo, ShiftPseudo, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SelectPseudo, ShiftPseudo 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
