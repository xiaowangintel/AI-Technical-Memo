# ARCInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCInstrFormats.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- ARCInstrFormats.td - ARC Instruction Formats --------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Instruction format superclass
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class Encoding64 {
  14:   field bits<64> Inst;
  15: }
  16: 
  17: // Address operands
  18: 
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as Encoding64, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 Encoding64 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19: class immU<int BSz> : Operand<i32>, PatLeaf<(imm),
  20:     "\n    return isUInt<"#BSz#">(N->getSExtValue());"> {
  21: }
  22: 
  23: def immU6 : immU<6>;
  24: 
  25: class immS<int BSz> : Operand<i32>, PatLeaf<(imm),
  26:     "\n    return isInt<"#BSz#">(N->getSExtValue());"> {
  27:   let DecoderMethod = "DecodeSignedOperand<"#BSz#">";
  28: }
  29: 
  30: // e.g. s3 field may encode the signed integers values -1 .. 6
  31: // using binary codes 111, 000, 001, 010, 011, 100, 101, and 110, respectively
  32: class immC<int BSz> : Operand<i32>, PatLeaf<(imm),
  33:     "\n    return isInt<"#BSz#">(N->getSExtValue());"> {
  34:   let DecoderMethod = "DecodeFromCyclicRange<"#BSz#">";
  35: }
  36: 
```

- EN: This range defines declarative TableGen records such as immU, immU6, immS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 immU, immU6, immS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: def MEMii : Operand<i32> {
  38:   let MIOperandInfo = (ops i32imm, i32imm);
  39: }
  40: 
  41: def MEMrs9 : Operand<iAny> {
  42:   let MIOperandInfo = (ops GPR32:$B, immS<9>:$S9);
  43:   let PrintMethod = "printMemOperandRI";
  44:   let DecoderMethod = "DecodeMEMrs9";
  45: }
  46: 
  47: def MEMrlimm : Operand<iAny> {
  48:   let MIOperandInfo = (ops GPR32:$B, i32imm:$LImm);
  49:   let PrintMethod = "printMemOperandRI";
  50:   let DecoderMethod = "DecodeMEMrlimm";
  51: }
  52: 
  53: def GPR32Reduced : Operand<iAny> {
  54:   let DecoderMethod = "DecodeGBR32ShortRegister";
```

- EN: This range defines declarative TableGen records such as MEMii, MEMrs9, MEMrlimm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MEMii, MEMrs9, MEMrlimm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55: }
  56: 
  57: // Helper classes for load/store instructions
  58: class DataSizeMode<bits<2> mode, string instSfx, string asmSfx> {
  59:   bits<2> Value = mode;
  60:   string  InstSuffix = instSfx;
  61:   string  AsmSuffix  = asmSfx;
  62: }
  63: class ExtMode<bit mode, string instSfx, string asmSfx> {
  64:   bit     Value = mode;
  65:   string  InstSuffix = instSfx;
  66:   string  AsmSuffix  = asmSfx;
  67: }
  68: 
  69: class AddrMode<bits<2> mode, string instSfx, string asmSfx> {
  70:   bits<2> Value = mode;
  71:   string  InstSuffix = instSfx;
  72:   string  AsmSuffix  = asmSfx;
```

- EN: This range defines declarative TableGen records such as DataSizeMode, ExtMode, AddrMode, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 DataSizeMode, ExtMode, AddrMode 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73: }
  74: 
  75: class CacheMode<bit mode, string instSfx, string asmSfx> {
  76:   bit     Value = mode;
  77:   string  InstSuffix = instSfx;
  78:   string  AsmSuffix  = asmSfx;
  79: }
  80: 
  81: def ByteSM : DataSizeMode<0b01, "B", "b">;
  82: def HalfSM : DataSizeMode<0b10, "H", "h">;
  83: def WordSM : DataSizeMode<0b00,  "",  "">;
  84: 
  85: def NoEM      : ExtMode<0,   "",   "">;
  86: def SignedEM  : ExtMode<1, "_X", ".x">;
  87: 
  88: def NoAM      : AddrMode<0b00, "", "">;
  89: def PreIncAM  : AddrMode<0b01, "_AW", ".aw">;
  90: def PostIncAM : AddrMode<0b10, "_AB", ".ab">;
```

- EN: This range defines declarative TableGen records such as CacheMode, ByteSM, HalfSM, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CacheMode, ByteSM, HalfSM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 91-108

```tablegen
  91: 
  92: def NoCC       : CacheMode<0b0,    "",    "">;
  93: def UncachedCC : CacheMode<0b1, "_DI", ".di">;
  94: 
  95: class InstARC<int sz, dag outs, dag ins, string asmstr, list<dag> pattern>
  96:     : Instruction, Encoding64 {
  97: 
  98:   let Namespace = "ARC";
  99:   dag OutOperandList = outs;
 100:   dag InOperandList = ins;
 101:   let AsmString = asmstr;
 102:   let Pattern = pattern;
 103:   let Size = sz;
 104: 
 105:   // Load/Store instruction properties
 106:   DataSizeMode ZZ = WordSM;
 107:   ExtMode X = NoEM;
 108:   AddrMode AA = NoAM;
```

- EN: This range defines declarative TableGen records such as NoCC, UncachedCC, InstARC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 NoCC, UncachedCC, InstARC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 109-126

```tablegen
 109:   CacheMode DI = NoCC;
 110: 
 111:   // Field used for relation models
 112:   string BaseOpcode = "";
 113: 
 114:   //TSFlags
 115:   let TSFlags{1-0} = AA.Value;
 116: }
 117: 
 118: // ARC pseudo instructions format
 119: class PseudoInstARC<dag outs, dag ins, string asmstr, list<dag> pattern>
 120:    : InstARC<0, outs, ins, asmstr, pattern> {
 121:   let isPseudo = 1;
 122: }
 123: 
 124: //===----------------------------------------------------------------------===//
 125: // Instruction formats
 126: //===----------------------------------------------------------------------===//
```

- EN: This range defines declarative TableGen records such as PseudoInstARC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 PseudoInstARC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127: 
 128: // All 32-bit ARC instructions have a 5-bit "major" opcode class designator
 129: // in bits 27-31.
 130: //
 131: // Some general naming conventions:
 132: // N  - Delay Slot bit.  ARC v2 branch instructions have an optional delay slot
 133: //      which is encoded with this bit.  When set, a delay slot exists.
 134: // cc - Condition code.
 135: // SX - Signed X-bit immediate.
 136: // UX - Unsigned X-bit immediate.
 137: //
 138: // [ABC] - 32-bit register operand.  These are 6-bit fields.  This encodes the
 139: //         standard 32 general purpose registers, and allows use of additional
 140: //         (extension) registers.  This also encodes an instruction that uses
 141: //         a 32-bit Long Immediate (LImm), using 0x3e==62 as the field value.
 142: //         This makes 32-bit format instructions with Long Immediates
 143: //         64-bit instructions, with the Long Immediate in bits 32-63.
 144: // A - Inst[5-0] = A[5-0], when the format has A.  A is always a register.
```

- EN: This range continues the implementation of the backend component described by ARCInstrFormats.td, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 145-162

```tablegen
 145: // B - Inst[14-12] = B[5-3], Inst[26-24] = B[2-0], when the format has B.
 146: //     B is always a register.
 147: // C - Inst[11-6] = C[5-0], when the format has C.  C can either be a register,
 148: //     or a 6-bit unsigned immediate (immU6), depending on the format.
 149: // F - Many instructions specify a flag bit. When set, the result of these
 150: //     instructions will set the ZNCV flags of the STATUS32 register
 151: //     (Zero/Negative/Carry/oVerflow).
 152: 
 153: // Branch Instructions.
 154: class F32_BR<bits<5> major, dag outs, dag ins, bit b16, string asmstr,
 155:              list<dag> pattern> :
 156:   InstARC<4, outs, ins, asmstr, pattern> {
 157:   bit N;
 158: 
 159:   let Inst{31-27} = major;
 160:   let Inst{16} = b16;
 161:   let Inst{5} = N;
 162: }
```

- EN: This range defines declarative TableGen records such as F32_BR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_BR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163: 
 164: class F32_BR_COND<bits<5> major, dag outs, dag ins, bit b16, string asmstr,
 165:                   list<dag> pattern> :
 166:   F32_BR<major, outs, ins, b16, asmstr, pattern> {
 167:   bits<21> S21; // 2-byte aligned 21-bit byte-offset.
 168:   bits<5> cc;
 169:   let Inst{26-18} = S21{10-2};
 170:   let Inst{15-6} = S21{20-11};
 171:   let Inst{4-0} = cc;
 172: }
 173: 
 174: class F32_BR_UCOND_FAR<bits<5> major, dag outs, dag ins, bit b16, string asmstr,
 175:                        list<dag> pattern> :
 176:   F32_BR<major, outs, ins, b16, asmstr, pattern> {
 177:   bits<25> S25; // 2-byte aligned 25-bit byte-offset.
 178:   let Inst{26-18} = S25{10-2};
 179:   let Inst{15-6} = S25{20-11};
 180:   let Inst{4} = 0;
```

- EN: This range defines declarative TableGen records such as F32_BR_COND, F32_BR_UCOND_FAR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_BR_COND, F32_BR_UCOND_FAR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181:   let Inst{3-0} = S25{24-21};
 182: }
 183: 
 184: class F32_BR0_COND<dag outs, dag ins, string asmstr, list<dag> pat> :
 185:   F32_BR_COND<0b00000, outs, ins, 0, asmstr, pat> {
 186:   let Inst{17} = S21{1};
 187: }
 188: 
 189: // Branch targets are 2-byte aligned, so S25[0] is implied 0.
 190: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0   |
 191: // |S25[10-1]                    | 1|S25[20-11]               |N|0|S25[24-21]|
 192: class F32_BR0_UCOND_FAR<dag outs, dag ins, string asmstr, list<dag> pat> :
 193:   F32_BR_UCOND_FAR<0b00000, outs, ins, 1, asmstr, pat> {
 194:   let Inst{17} = S25{1};
 195: }
 196: 
 197: // BL targets (functions) are 4-byte aligned, so S25[1-0] = 0b00
 198: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0   |
```

- EN: This range defines declarative TableGen records such as F32_BR0_COND, F32_BR0_UCOND_FAR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_BR0_COND, F32_BR0_UCOND_FAR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199: // |S25[10-2]                 | 1| 0|S25[20-11]               |N|0|S25[24-21]|
 200: class F32_BR1_BL_UCOND_FAR<dag outs, dag ins, string asmstr, list<dag> pat> :
 201:   F32_BR_UCOND_FAR<0b00001, outs, ins, 0, asmstr, pat> {
 202:   let Inst{17} = 1;
 203: }
 204: 
 205: // BLcc targets have 21 bit range, and are 4-byte aligned.
 206: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 207: // |S25[10-2]                 | 0| 0|S25[20-11]               |N|0|cc     |
 208: class F32_BR1_BL_COND<dag outs, dag ins, string asmstr, list<dag> pat> :
 209:   F32_BR_COND<0b00001, outs, ins, 0, asmstr, pat> {
 210:   let Inst{17} = 0;
 211: }
 212: 
 213: // BRcc targets have limited 9-bit range.  These are for compare and branch
 214: // in single instruction.  Their targets are 2-byte aligned.  They also use
 215: // a different (3-bit) set of condition codes.
 216: // |26|25|24|23|22|21|20|19|18|17|16|15   |14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
```

- EN: This range defines declarative TableGen records such as F32_BR1_BL_UCOND_FAR, F32_BR1_BL_COND, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_BR1_BL_UCOND_FAR, F32_BR1_BL_COND 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: // |B[2-0]  |S9[7-1]             | 1|S9[8]|B[5-3]  |C            |N|u|0|cc   |
 218: class F32_BR1_BCC<dag outs, dag ins, string asmstr, bit IsU6,
 219:                   list<dag> pattern> :
 220:   InstARC<4, outs, ins, asmstr, pattern> {
 221: 
 222:   bits<3> cc;
 223:   bits<6> B;
 224:   bits<6> C;
 225:   bit N;
 226:   bits<9> S9; // 2-byte aligned 9-bit byte-offset.
 227: 
 228:   let Inst{31-27} = 0b00001;
 229:   let Inst{26-24} = B{2-0};
 230:   let Inst{23-17} = S9{7-1};
 231:   let Inst{16} = 1;
 232:   let Inst{15} = S9{8};
 233:   let Inst{14-12} = B{5-3};
 234:   let Inst{11-6} = C;
```

- EN: This range defines declarative TableGen records such as F32_BR1_BCC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_BR1_BCC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-252

```tablegen
 235:   let Inst{5} = N;
 236:   let Inst{4} = IsU6;
 237:   let Inst{3} = 0;
 238:   let Inst{2-0} = cc;
 239: }
 240: 
 241: // General operations instructions.
 242: // Single Operand Instructions.  Inst[5-0] specifies the specific operation
 243: // for this format.
 244: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 245: // |B[2-0]  | 0| 0| 1| 0| 1| 1| 1| 1| F|B[5-3]  |C            |subop      |
 246: class F32_SOP_RR<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 247:                  string asmstr, list<dag> pattern> :
 248:   InstARC<4, outs, ins, asmstr, pattern> {
 249: 
 250:   bits<6> C;
 251:   bits<6> B;
 252: 
```

- EN: This range defines declarative TableGen records such as F32_SOP_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_SOP_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253:   let Inst{31-27} = major;
 254:   let Inst{26-24} = B{2-0};
 255:   let Inst{23-22} = 0b00;
 256:   let Inst{21-16} = 0b101111;
 257:   let Inst{15} = F;
 258:   let Inst{14-12} = B{5-3};
 259:   let Inst{11-6} = C;
 260:   let Inst{5-0} = subop;
 261: }
 262: 
 263: // Dual Operand Instructions.  Inst[21-16] specifies the specific operation
 264: // for this format.
 265: 
 266: // 3-register Dual Operand instruction.
 267: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 268: // |B[2-0]  | 0| 0|            subop| F|B[5-3]  |C            |A          |
 269: class F32_DOP_RR<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 270:                  string asmstr, list<dag> pattern> :
```

- EN: This range defines declarative TableGen records such as F32_DOP_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271:   InstARC<4, outs, ins, asmstr, pattern> {
 272:   bits<6> C;
 273:   bits<6> B;
 274:   bits<6> A;
 275: 
 276:   let Inst{31-27} = major;
 277:   let Inst{26-24} = B{2-0};
 278:   let Inst{23-22} = 0b00;
 279:   let Inst{21-16} = subop;
 280:   let Inst{15} = F;
 281:   let Inst{14-12} = B{5-3};
 282:   let Inst{11-6} = C;
 283:   let Inst{5-0} = A;
 284: }
 285: 
 286: // Conditional Dual Operand instruction.  This instruction uses B as the
 287: // first 2 operands (i.e, add.cc B, B, C).
 288: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: // |B[2-0]  | 1| 1|            subop| F|B[5-3]  |C            |A          |
 290: class F32_DOP_CC_RR<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 291:                     string asmstr, list<dag> pattern> :
 292:   InstARC<4, outs, ins, asmstr, pattern> {
 293:   bits<5> cc;
 294:   bits<6> C;
 295:   bits<6> B;
 296: 
 297:   let Inst{31-27} = major;
 298:   let Inst{26-24} = B{2-0};
 299:   let Inst{23-22} = 0b11;
 300:   let Inst{21-16} = subop;
 301:   let Inst{15} = F;
 302:   let Inst{14-12} = B{5-3};
 303:   let Inst{11-6} = C;
 304:   let Inst{5} = 0;
 305:   let Inst{4-0} = cc;
 306: }
```

- EN: This range defines declarative TableGen records such as F32_DOP_CC_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_CC_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307: 
 308: 
 309: // 2-register, unsigned 6-bit immediate Dual Operand instruction.
 310: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 311: // |B[2-0]  | 0| 1|            subop| F|B[5-3]  |U6           |A          |
 312: class F32_DOP_RU6<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 313:                   string asmstr, list<dag> pattern> :
 314:   InstARC<4, outs, ins, asmstr, pattern> {
 315:   bits<6> U6;
 316:   bits<6> B;
 317:   bits<6> A;
 318: 
 319:   let Inst{31-27} = major;
 320:   let Inst{26-24} = B{2-0};
 321:   let Inst{23-22} = 0b01;
 322:   let Inst{21-16} = subop;
 323:   let Inst{15} = F;
 324:   let Inst{14-12} = B{5-3};
```

- EN: This range defines declarative TableGen records such as F32_DOP_RU6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_RU6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 325-342

```tablegen
 325:   let Inst{11-6} = U6;
 326:   let Inst{5-0} = A;
 327: }
 328: 
 329: // 1-register, unsigned 6-bit, immediate Dual Operand instruction with
 330: // condition code.
 331: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 332: // |B[2-0]  | 1| 1|            subop| F|B[5-3]  |U6           |1|cc       |
 333: class F32_DOP_CC_RU6<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 334:                      string asmstr, list<dag> pattern> :
 335:   InstARC<4, outs, ins, asmstr, pattern> {
 336: 
 337:   bits<5> cc;
 338:   bits<6> U6;
 339:   bits<6> B;
 340: 
 341:   let Inst{31-27} = major;
 342:   let Inst{26-24} = B{2-0};
```

- EN: This range defines declarative TableGen records such as F32_DOP_CC_RU6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_CC_RU6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343:   let Inst{23-22} = 0b11;
 344:   let Inst{21-16} = subop;
 345:   let Inst{15} = F;
 346:   let Inst{14-12} = B{5-3};
 347:   let Inst{11-6} = U6;
 348:   let Inst{5} = 1;
 349:   let Inst{4-0} = cc;
 350: 
 351:   let DecoderMethod = "DecodeCCRU6Instruction";
 352: }
 353: 
 354: // 2-register, unsigned 6-bit immediate Dual Operand instruction with
 355: // condition code. This instruction uses B as the first 2 operands
 356: // (i.e, add.cc B, B, u6).
 357: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 358: // |B[2-0]  | 1| 1|            subop| F|B[5-3]  |U6           |1|cc       |
 359: class F32_DOP_CC_RRU6<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 360:                       string asmstr, list<dag> pattern> :
```

- EN: This range defines declarative TableGen records such as F32_DOP_CC_RRU6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_CC_RRU6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361:   InstARC<4, outs, ins, asmstr, pattern> {
 362:   bits<5> cc;
 363:   bits<6> U6;
 364:   bits<6> B;
 365: 
 366:   let Inst{31-27} = major;
 367:   let Inst{26-24} = B{2-0};
 368:   let Inst{23-22} = 0b11;
 369:   let Inst{21-16} = subop;
 370:   let Inst{15} = F;
 371:   let Inst{14-12} = B{5-3};
 372:   let Inst{11-6} = U6;
 373:   let Inst{5} = 1;
 374:   let Inst{4-0} = cc;
 375: }
 376: 
 377: // 2-register, signed 12-bit immediate Dual Operand instruction.
 378: // This instruction uses B as the first 2 operands (i.e., add B, B, -128).
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 380: // |B[2-0]  | 1| 0|            subop| F|B[5-3]  |S12[5-0]     |S12[11-6]  |
 381: class F32_DOP_RS12<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 382:                    string asmstr, list<dag> pattern> :
 383:   InstARC<4, outs, ins, asmstr, pattern> {
 384:   bits<6> B;
 385:   bits<12> S12;
 386: 
 387:   let Inst{31-27} = major;
 388:   let Inst{26-24} = B{2-0};
 389:   let Inst{23-22} = 0b10;
 390:   let Inst{21-16} = subop;
 391:   let Inst{15} = F;
 392:   let Inst{14-12} = B{5-3};
 393:   let Inst{11-6} = S12{5-0};
 394:   let Inst{5-0} = S12{11-6};
 395: }
 396: 
```

- EN: This range defines declarative TableGen records such as F32_DOP_RS12, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_RS12 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397: // 1-register, signed 12-bit immediate Dual Operand instruction.
 398: // This instruction uses B as the first operand (i.e., lr B, [%count0]).
 399: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 400: // |B[2-0]  | 1| 0|            subop| F|B[5-3]  |S12[5-0]     |S12[11-6]  |
 401: class F32_SOP_RS12<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 402:                    string asmstr, list<dag> pattern> :
 403:   InstARC<4, outs, ins, asmstr, pattern> {
 404:   bits<6> B;
 405:   bits<12> S12;
 406: 
 407:   let Inst{31-27} = major;
 408:   let Inst{26-24} = B{2-0};
 409:   let Inst{23-22} = 0b10;
 410:   let Inst{21-16} = subop;
 411:   let Inst{15} = F;
 412:   let Inst{14-12} = B{5-3};
 413:   let Inst{11-6} = S12{5-0};
 414:   let Inst{5-0} = S12{11-6};
```

- EN: This range defines declarative TableGen records such as F32_SOP_RS12, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_SOP_RS12 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415: 
 416:   let DecoderMethod = "DecodeSOPwithRS12";
 417: }
 418: 
 419: // 1-register, unsigned 6-bit immediate Dual Operand instruction.
 420: // This instruction uses B as the first operand.
 421: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 422: // |B[2-0]  | 0| 1|            subop| F|B[5-3]  |U6           |0|0|0|0|0|0|
 423: class F32_SOP_RU6<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 424:                   string asmstr, list<dag> pattern> :
 425:   InstARC<4, outs, ins, asmstr, pattern> {
 426:   bits<6> B;
 427:   bits<6> U6;
 428: 
 429:   let Inst{31-27} = major;
 430:   let Inst{26-24} = B{2-0};
 431:   let Inst{23-22} = 0b01;
 432:   let Inst{21-16} = subop;
```

- EN: This range defines declarative TableGen records such as F32_SOP_RU6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_SOP_RU6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433:   let Inst{15} = F;
 434:   let Inst{14-12} = B{5-3};
 435:   let Inst{11-6} = U6;
 436:   let Inst{5-0} = 0;
 437: 
 438:   let DecoderMethod = "DecodeSOPwithRU6";
 439: }
 440: 
 441: // 2-register, 32-bit immediate (LImm) Dual Operand instruction.
 442: // This instruction has the 32-bit immediate in bits 32-63, and
 443: // 62 in the C register operand slot, but is otherwise F32_DOP_RR.
 444: class F32_DOP_RLIMM<bits<5> major, bits<6> subop, bit F, dag outs, dag ins,
 445:                     string asmstr, list<dag> pattern> :
 446:   InstARC<8, outs, ins, asmstr, pattern> {
 447:   bits<6> B;
 448:   bits<6> A;
 449:   bits<32> LImm;
 450: 
```

- EN: This range defines declarative TableGen records such as F32_DOP_RLIMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_DOP_RLIMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451:   let Inst{63-32} = LImm;
 452:   let Inst{31-27} = major;
 453:   let Inst{26-24} = B{2-0};
 454:   let Inst{23-22} = 0b00;
 455:   let Inst{21-16} = subop;
 456:   let Inst{15} = F;
 457:   let Inst{14-12} = B{5-3};
 458:   let Inst{11-6} = 0b111110;
 459:   let Inst{5-0} = A;
 460: }
 461: 
 462: 
 463: // Load and store instructions.
 464: // In addition to the previous naming conventions, load and store instructions
 465: // have:
 466: // di - Uncached bit.  When set, loads/stores bypass the cache and access
 467: //      memory directly.
 468: // aa - Incrementing mode.  Loads and stores can write-back address pre- or
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469: //      post- memory operation.
 470: // zz - Memory size (can be 8/16/32 bit load/store).
 471: //  x - Sign-extending.  When set, short loads can be sign-extended to 32-bits.
 472: // Loads and Stores support different memory addressing modes:
 473: // Base Register + Signed 9-bit Immediate: Both Load/Store.
 474: // LImm: Both Load/Store (Load/Store from a fixed 32-bit address).
 475: // Register + Register: Load Only.
 476: // Register + LImm: Load Only.
 477: 
 478: // Register + S9 Load. (B + S9)
 479: // |26|25|24|23|22|21|20|19|18|17|16|15   |14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 480: // |B[2-0]  |S9[7-0]                |S9[8]|B[5-3]  |di|aa  |zz |x|A          |
 481: class F32_LD_RS9<bit x, bits<2> aa, bit di, bits<2> zz, dag outs, dag ins,
 482:                  string asmstr, list<dag> pattern> :
 483:   InstARC<4, outs, ins, asmstr, pattern> {
 484:   bits<6> B;
 485:   bits<6> A;
 486:   bits<9> S9;
```

- EN: This range defines declarative TableGen records such as F32_LD_RS9, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 F32_LD_RS9 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 487-504

```tablegen
 487: 
 488:   let Inst{31-27} = 0b00010;
 489:   let Inst{26-24} = B{2-0};
 490:   let Inst{23-16} = S9{7-0};
 491:   let Inst{15} = S9{8};
 492:   let Inst{14-12} = B{5-3};
 493:   let Inst{11} = di;
 494:   let Inst{10-9} = aa;
 495:   let Inst{8-7} = zz;
 496:   let Inst{6} = x;
 497:   let Inst{5-0} = A;
 498: 
 499:   let BaseOpcode = "ld_rs9";
 500: }
 501: 
 502: class F32_LD_ADDR<bit x, bits<2> aa, bit di, bits<2> zz, dag outs, dag ins,
 503:                   string asmstr, list<dag> pattern> :
 504:   F32_LD_RS9<x, aa, di, zz, outs, ins, asmstr, pattern> {
```

- EN: This range defines declarative TableGen records such as F32_LD_ADDR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_LD_ADDR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 505-522

```tablegen
 505:   bits<15> addr;
 506: 
 507:   let B = addr{14-9};
 508:   let S9 = addr{8-0};
 509: 
 510:   let BaseOpcode = "ld_rs9";
 511: }
 512: 
 513: 
 514: // LImm Load.  The 32-bit immediate address is in Inst[63-32].
 515: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
 516: // | 1| 1| 0| 0                        | 1| 1| 1|di| 0|0|zz |x|A          |
 517: class F32_LD_LIMM<bit x, bit di, bits<2> zz, dag outs, dag ins,
 518:                   string asmstr, list<dag> pattern> :
 519:   InstARC<8, outs, ins, asmstr, pattern> {
 520:   bits<6> LImmReg = 0b111110;
 521:   bits<6> A;
 522:   bits<32> LImm;
```

- EN: This range defines declarative TableGen records such as F32_LD_LIMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_LD_LIMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523: 
 524:   let Inst{63-32} = LImm;
 525:   let Inst{31-27} = 0b00010;
 526:   let Inst{26-24} = LImmReg{2-0};
 527:   let Inst{23-15} = 0;
 528:   let Inst{14-12} = LImmReg{5-3};
 529:   let Inst{11} = di;
 530:   let Inst{10-9} = 0;
 531:   let Inst{8-7} = zz;
 532:   let Inst{6} = x;
 533:   let Inst{5-0} = A;
 534:   let DecoderMethod = "DecodeLdLImmInstruction";
 535: 
 536:   let BaseOpcode = "ld_limm";
 537: }
 538: 
 539: // Register + LImm load.  The 32-bit immediate address is in Inst[63-32].
 540: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5|4|3|2|1|0|
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541: // |B[2-0]  |aa   | 1| 1| 0|zz   | x|di|B[5-3]  | 1| 1|1|1|1|0|A          |
 542: class F32_LD_RLIMM<bit x, bits<2> aa, bit di, bits<2> zz, dag outs, dag ins,
 543:                    string asmstr, list<dag> pattern> :
 544:   InstARC<8, outs, ins, asmstr, pattern> {
 545:   bits<6> LImmReg = 0b111110;
 546:   bits<32> LImm;
 547:   bits<6> B;
 548:   bits<6> A;
 549:   bits<38> addr;
 550:   let B = addr{37-32};
 551:   let LImm = addr{31-0};
 552: 
 553:   let Inst{63-32} = LImm;
 554:   let Inst{31-27} = 0b00100;
 555:   let Inst{26-24} = B{2-0};
 556:   let Inst{23-22} = aa;
 557:   let Inst{21-19} = 0b110;
 558:   let Inst{18-17} = zz;
```

- EN: This range defines declarative TableGen records such as F32_LD_RLIMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_LD_RLIMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 559-576

```tablegen
 559:   let Inst{16} = x;
 560:   let Inst{15} = di;
 561:   let Inst{14-12} = B{5-3};
 562:   let Inst{11-6} = LImmReg;
 563:   let Inst{5-0} = A;
 564:   let DecoderMethod = "DecodeLdRLImmInstruction";
 565: 
 566:   let BaseOpcode = "ld_rlimm";
 567: }
 568: 
 569: // Register + S9 Store. (B + S9)
 570: // |26|25|24|23|22|21|20|19|18|17|16|15   |14|13|12|11|10|9|8|7|6|5 |4|3|2|1|0|
 571: // |B[2-0]  |S9[7-0]                |S9[8]|B[5-3]  |C            |di|aa |zz |0|
 572: class F32_ST_RS9<bits<2> aa, bit di, bits<2> zz, dag outs, dag ins,
 573:                  string asmstr, list<dag> pattern> :
 574:   InstARC<4, outs, ins, asmstr, pattern> {
 575:   bits<6> B;
 576:   bits<6> C;
```

- EN: This range defines declarative TableGen records such as F32_ST_RS9, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_ST_RS9 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 577-594

```tablegen
 577:   bits<9> S9;
 578: 
 579:   let Inst{31-27} = 0b00011;
 580:   let Inst{26-24} = B{2-0};
 581:   let Inst{23-16} = S9{7-0};
 582:   let Inst{15} = S9{8};
 583:   let Inst{14-12} = B{5-3};
 584:   let Inst{11-6} = C;
 585:   let Inst{5} = di;
 586:   let Inst{4-3} = aa;
 587:   let Inst{2-1} = zz;
 588:   let Inst{0} = 0;
 589: 
 590:   let BaseOpcode = "st_rs9";
 591: }
 592: 
 593: class F32_ST_ADDR<bits<2> aa, bit di, bits<2> zz, dag outs, dag ins,
 594:                   string asmstr, list<dag> pattern> :
```

- EN: This range defines declarative TableGen records such as F32_ST_ADDR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_ST_ADDR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 595-612

```tablegen
 595:   F32_ST_RS9<aa, di, zz, outs, ins, asmstr, pattern> {
 596:   bits<15> addr;
 597: 
 598:   let B = addr{14-9};
 599:   let S9 = addr{8-0};
 600: 
 601:   let BaseOpcode = "st_rs9";
 602: }
 603: 
 604: // LImm Store.
 605: // |26|25|24|23|22|21|20|19|18|17|16|15|14|13|12|11|10|9|8|7|6|5 |4|3|2|1|0|
 606: // | 1| 1| 0| 0                        | 1| 1| 1|C            |di|0|0|zz |0|
 607: class F32_ST_LIMM<bit di, bits<2> zz, dag outs, dag ins,
 608:                   string asmstr, list<dag> pattern> :
 609:   InstARC<8, outs, ins, asmstr, pattern> {
 610:   bits<6> LImmReg = 0b111110;
 611:   bits<6> C;
 612:   bits<32> LImm;
```

- EN: This range defines declarative TableGen records such as F32_ST_LIMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F32_ST_LIMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 613-630

```tablegen
 613: 
 614:   let Inst{63-32} = LImm;
 615:   let Inst{31-27} = 0b00011;
 616:   let Inst{26-24} = LImmReg{2-0};
 617:   let Inst{23-15} = 0;
 618:   let Inst{14-12} = LImmReg{5-3};
 619:   let Inst{11-6} = C;
 620:   let Inst{5} = di;
 621:   let Inst{4-3} = 0;
 622:   let Inst{2-1} = zz;
 623:   let Inst{0} = 0;
 624:   let DecoderMethod = "DecodeStLImmInstruction";
 625: 
 626:   let BaseOpcode = "st_limm";
 627: }
 628: 
 629: // Compact Move/Load.
 630: // |10|9|8|7|6|5|4|3|2|1|0|
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 631-648

```tablegen
 631: // |      |h    |   |i|H  |
 632: class F16_COMPACT<bits<1> i, dag outs, dag ins,
 633:                  string asmstr> :
 634:   InstARC<2, outs, ins, asmstr, []> {
 635: 
 636:   bits<5> h;
 637: 
 638:   let Inst{15-11} = 0b01000;
 639:   let Inst{7-5} = h{2-0};
 640:   let Inst{2} = i;
 641:   let Inst{1-0} = h{4-3};
 642: }
 643: 
 644: // Compact Load/Add/Sub.
 645: class F16_LD_ADD_SUB<dag outs, dag ins, string asmstr> :
 646:   InstARC<2, outs, ins, asmstr, []> {
 647: 
 648:   bits<3> b;
```

- EN: This range defines declarative TableGen records such as F16_COMPACT, F16_LD_ADD_SUB, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_COMPACT, F16_LD_ADD_SUB 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649:   let Inst{15-11} = 0b01001;
 650:   let Inst{10-8} = b;
 651: }
 652: 
 653: class F16_LD_SUB<bit i, string asmstr> :
 654:   F16_LD_ADD_SUB<(outs GPR32:$a), (ins GPR32:$b, GPR32:$c),
 655:   asmstr> {
 656: 
 657:   bits<3> a;
 658:   bits<3> c;
 659: 
 660:   let Inst{7-5} = c;
 661:   let Inst{4} = i;
 662:   let Inst{3} = 0;
 663:   let Inst{2-0} = a;
 664: }
 665: 
 666: class F16_ADD :
```

- EN: This range defines declarative TableGen records such as F16_LD_SUB, F16_ADD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_LD_SUB, F16_ADD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 667-684

```tablegen
 667:   F16_LD_ADD_SUB<(outs GPR32:$r), (ins GPR32:$b, immU<6>:$u6),
 668:   "add_s\t$r, $b, $u6"> {
 669: 
 670:   bit r;
 671:   bits<6> u6;
 672: 
 673:   let Inst{7} = r;
 674:   let Inst{6-4} = u6{5-3};
 675:   let Inst{3} = 1;
 676:   let Inst{2-0} = u6{2-0};
 677: }
 678: 
 679: // Compact Load/Store.
 680: class F16_LD_ST_1<dag outs, dag ins, string asmstr> :
 681:   InstARC<2, outs, ins, asmstr, []> {
 682: 
 683:   let Inst{15-11} = 0b01010;
 684: }
```

- EN: This range defines declarative TableGen records such as F16_LD_ST_1, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_LD_ST_1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-702

```tablegen
 685: 
 686: class F16_LD_ST_s11<bit i, string asmstr> :
 687:   F16_LD_ST_1<(outs), (ins immS<11>:$s11), asmstr> {
 688: 
 689:   bits<11> s11;
 690: 
 691:   let Inst{10-5} = s11{10-5};
 692:   let Inst{4} = i;
 693:   let Inst{3} = 0;
 694:   let Inst{2-0} = s11{4-2};
 695:   let s11{1-0} = 0b00;
 696: }
 697: 
 698: class F16_LDI_u7 :
 699:   F16_LD_ST_1<(outs GPR32:$b), (ins immU<7>:$u7),
 700:   "ldi_s\t$b, [$u7]"> {
 701: 
 702:   bits<3> b;
```

- EN: This range defines declarative TableGen records such as F16_LD_ST_s11, F16_LDI_u7, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_LD_ST_s11, F16_LDI_u7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 703-720

```tablegen
 703:   bits<7> u7;
 704: 
 705:   let Inst{10-8} = b;
 706:   let Inst{7-4} = u7{6-3};
 707:   let Inst{3} = 1;
 708:   let Inst{2-0} = u7{2-0};
 709: }
 710: 
 711: // Indexed Jump or Execute.
 712: class F16_JLI_EI<bit i, string asmstr> :
 713:   InstARC<2, (outs), (ins immU<10>:$u10),
 714:   !strconcat(asmstr, "\t$u10"), []> {
 715: 
 716:   bits<10> u10;
 717: 
 718:   let Inst{15-11} = 0b01011;
 719:   let Inst{10} = i;
 720:   let Inst{9-0} = u10;
```

- EN: This range defines declarative TableGen records such as F16_JLI_EI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_JLI_EI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 721-738

```tablegen
 721: }
 722: 
 723: // Load/Add Register-Register.
 724: class F16_LD_ADD_RR<bits<2> i, string asmstr> :
 725:   InstARC<2, (outs GPR32:$a), (ins GPR32:$b, GPR32:$c),
 726:   asmstr, []> {
 727: 
 728:   bits<3> a;
 729:   bits<3> b;
 730:   bits<3> c;
 731: 
 732:   let Inst{15-11} = 0b01100;
 733:   let Inst{10-8} = b;
 734:   let Inst{7-5} = c;
 735:   let Inst{4-3} = i;
 736:   let Inst{2-0} = a;
 737: }
 738: 
```

- EN: This range defines declarative TableGen records such as F16_LD_ADD_RR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_LD_ADD_RR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 739-756

```tablegen
 739: // Load/Add GP-Relative.
 740: class F16_GP_LD_ADD<bits<2> i, dag ins, string asmstr> :
 741:   InstARC<2, (outs), ins, asmstr, []> {
 742: 
 743:   let Inst{15-11} = 0b11001;
 744:   let Inst{10-9} = i;
 745: }
 746: 
 747: // Add/Sub/Shift Register-Immediate.
 748: // |10|9|8|7|6|5|4|3|2|1|0|
 749: // |b     |c    |i  |u    |
 750: class F16_ADD_IMM<bits<2> i, string asmstr> :
 751:   InstARC<2, (outs GPR32:$c), (ins GPR32:$b, immU<3>:$u3),
 752:   !strconcat(asmstr, "\t$c, $b, $u3"), []> {
 753: 
 754:   bits<3> b;
 755:   bits<3> c;
 756:   bits<3> u3;
```

- EN: This range defines declarative TableGen records such as F16_GP_LD_ADD, F16_ADD_IMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_GP_LD_ADD, F16_ADD_IMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 757-774

```tablegen
 757: 
 758:   let Inst{15-11} = 0b01101;
 759:   let Inst{10-8} = b;
 760:   let Inst{7-5} = c;
 761:   let Inst{4-3} = i;
 762:   let Inst{2-0} = u3;
 763: }
 764: 
 765: // Dual Register Operations.
 766: // |10|9|8|7|6|5|4|3|2|1|0|
 767: // |b/s   |h    |i    |H  |
 768: class F16_OP_HREG<bits<3> i, dag outs, dag ins, string asmstr> :
 769:   InstARC<2, outs, ins, asmstr, []> {
 770: 
 771:   bits<3> b_s3;
 772:   bits<5> h;
 773: 
 774:   let Inst{15-11} = 0b01110;
```

- EN: This range defines declarative TableGen records such as F16_OP_HREG, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_OP_HREG 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 775-792

```tablegen
 775:   let Inst{10-8} = b_s3;
 776:   let Inst{7-5} = h{2-0};
 777:   let Inst{4-2} = i;
 778:   let Inst{1-0} = h{4-3};
 779: }
 780: 
 781: class F16_OP_HREG30<bits<3> i, dag outs, dag ins, string asmstr> :
 782:   F16_OP_HREG<i, outs, ins, asmstr> {
 783: 
 784:   bits<5> LImmReg = 0b11110;
 785:   let Inst{7-5} = LImmReg{2-0};
 786:   let Inst{1-0} = LImmReg{4-3};
 787: }
 788: 
 789: class F16_OP_HREG_LIMM<bits<3> i, dag outs, dag ins, string asmstr> :
 790:   F16_OP_HREG30<i, outs, ins, asmstr> {
 791: 
 792:   bits<32> LImm;
```

- EN: This range defines declarative TableGen records such as F16_OP_HREG30, F16_OP_HREG_LIMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_OP_HREG30, F16_OP_HREG_LIMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 793-810

```tablegen
 793:   let Inst{47-16} = LImm;
 794:   let Size = 6;
 795: }
 796: 
 797: // General compact DOP format.
 798: class F16_GEN_DOP_BASE<bits<5> i, dag outs, dag ins, string asmstr> :
 799:   InstARC<2, outs, ins, asmstr, []> {
 800: 
 801:   bits<3> b;
 802:   bits<3> c;
 803:   let Inst{15-11} = 0b01111;
 804:   let Inst{10-8} = b;
 805:   let Inst{7-5} = c;
 806:   let Inst{4-0} = i;
 807: }
 808: 
 809: class F16_GEN_DOP<bits<5> i, string asmstr> :
 810:   F16_GEN_DOP_BASE<i, (outs GPR32:$b), (ins GPR32:$c),
```

- EN: This range defines declarative TableGen records such as F16_GEN_DOP_BASE, F16_GEN_DOP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_GEN_DOP_BASE, F16_GEN_DOP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 811-828

```tablegen
 811:   !strconcat(asmstr, "\t$b, $b, $c")>;
 812: 
 813: class F16_GEN_DOP_NODST<bits<5> i, string asmstr> :
 814:   F16_GEN_DOP_BASE<i, (outs), (ins GPR32:$b, GPR32:$c),
 815:   !strconcat(asmstr, "\t$b, $c")>;
 816: 
 817: class F16_GEN_DOP_SINGLESRC<bits<5> i, string asmstr> :
 818:   F16_GEN_DOP_BASE<i, (outs GPR32:$b), (ins GPR32:$c),
 819:   !strconcat(asmstr, "\t$b, $c")>;
 820: 
 821: class F16_GEN_SOP_BASE<bits<3> i, dag outs, dag ins, string asmstr> :
 822:   F16_GEN_DOP_BASE<0b00000, outs, ins, asmstr> {
 823: 
 824:   let c = i;
 825: }
 826: 
 827: class F16_GEN_SOP<bits<3> i, string asmstr> :
 828:   F16_GEN_SOP_BASE<i, (outs), (ins GPR32:$b), asmstr>;
```

- EN: This range defines declarative TableGen records such as F16_GEN_DOP_NODST, F16_GEN_DOP_SINGLESRC, F16_GEN_SOP_BASE, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_GEN_DOP_NODST, F16_GEN_DOP_SINGLESRC, F16_GEN_SOP_BASE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 829-846

```tablegen
 829: 
 830: class F16_GEN_ZOP<bits<3> i, string asmstr> :
 831:   F16_GEN_SOP_BASE<0b111, (outs), (ins), asmstr> {
 832: 
 833:   let b = i;
 834: }
 835: 
 836: // Compact Load/Store with Offset Format.
 837: class F16_LD_ST_OFF<bits<5> opc, dag outs, dag ins, string asmstr> :
 838:   InstARC<2, outs, ins, !strconcat(asmstr, "\t$c, [$b, $off]"), []> {
 839: 
 840:   bits<3> b;
 841:   bits<3> c;
 842:   let Inst{15-11} = opc;
 843:   let Inst{10-8} = b;
 844:   let Inst{7-5} = c;
 845: }
 846: 
```

- EN: This range defines declarative TableGen records such as F16_GEN_ZOP, F16_LD_ST_OFF, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_GEN_ZOP, F16_LD_ST_OFF 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 847-864

```tablegen
 847: class F16_LD_ST_WORD_OFF<bits<5> opc, dag outs, dag ins, string asmstr> :
 848:   F16_LD_ST_OFF<opc, outs, ins, asmstr> {
 849: 
 850:   bits<7> off;
 851:   let Inst{4-0} = off{6-2};
 852:   let off{1-0} = 0b00;
 853: }
 854: 
 855: class F16_LD_ST_HALF_OFF<bits<5> opc, dag outs, dag ins, string asmstr> :
 856:   F16_LD_ST_OFF<opc, outs, ins, asmstr> {
 857: 
 858:   bits<6> off;
 859:   let Inst{4-0} = off{5-1};
 860:   let off{0} = 0b0;
 861: }
 862: 
 863: class F16_LD_ST_BYTE_OFF<bits<5> opc, dag outs, dag ins, string asmstr> :
 864:   F16_LD_ST_OFF<opc, outs, ins, asmstr> {
```

- EN: This range defines declarative TableGen records such as F16_LD_ST_WORD_OFF, F16_LD_ST_HALF_OFF, F16_LD_ST_BYTE_OFF, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_LD_ST_WORD_OFF, F16_LD_ST_HALF_OFF, F16_LD_ST_BYTE_OFF 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 865-882

```tablegen
 865: 
 866:   bits<5> off;
 867:   let Inst{4-0} = off;
 868: }
 869: 
 870: // Shift/Subtract/Bit Immediate.
 871: // |10|9|8|7|6|5|4|3|2|1|0|
 872: // |b     |i    |u        |
 873: class F16_SH_SUB_BIT<bits<3> i, string asmstr> :
 874:   InstARC<2, (outs), (ins GPR32:$b, immU<5>:$u5), asmstr, []> {
 875: 
 876:   bits<3> b;
 877:   bits<5> u5;
 878: 
 879:   let Inst{15-11} = 0b10111;
 880:   let Inst{10-8} = b;
 881:   let Inst{7-5} = i;
 882:   let Inst{4-0} = u5;
```

- EN: This range defines declarative TableGen records such as F16_SH_SUB_BIT, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_SH_SUB_BIT 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 883-900

```tablegen
 883: }
 884: 
 885: class F16_SH_SUB_BIT_DST<bits<3> i, string asmstr> :
 886:   F16_SH_SUB_BIT<i, !strconcat(asmstr, "\t$b, $b, $u5")>;
 887: 
 888: // 16-bit stack-based operations.
 889: // |10|9|8|7|6|5|4|3|2|1|0|
 890: // |b     |i    |u        |
 891: class F16_SP_OPS<bits<3> i,
 892:   dag outs, dag ins, string asmstr> :
 893:   InstARC<2, outs, ins, asmstr, []> {
 894: 
 895:   bits<3> fieldB;
 896:   bits<5> fieldU;
 897: 
 898:   let Inst{15-11} = 0b11000;
 899:   let Inst{10-8} = fieldB;
 900:   let Inst{7-5} = i;
```

- EN: This range defines declarative TableGen records such as F16_SH_SUB_BIT_DST, F16_SP_OPS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_SH_SUB_BIT_DST, F16_SP_OPS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 901-918

```tablegen
 901:   let Inst{4-0} = fieldU;
 902: }
 903: 
 904: class F16_SP_OPS_u7_aligned<bits<3> i,
 905:   dag outs, dag ins, string asmstr> :
 906:   F16_SP_OPS<i, outs, ins, asmstr> {
 907: 
 908:   bits<3> b3;
 909:   bits<7> u7;
 910: 
 911:   let fieldB = b3;
 912:   let fieldU = u7{6-2};
 913:   let u7{1-0} = 0b00;
 914: }
 915: 
 916: class F16_SP_OPS_bconst<bits<3> b, string asmop> :
 917:   F16_SP_OPS_u7_aligned<0b101,
 918:   (outs), (ins immU<7>:$u7),
```

- EN: This range defines declarative TableGen records such as F16_SP_OPS_u7_aligned, F16_SP_OPS_bconst, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_SP_OPS_u7_aligned, F16_SP_OPS_bconst 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 919-936

```tablegen
 919:   !strconcat(asmop, "\t%sp, %sp, $u7")> {
 920: 
 921:   let fieldB = b;
 922: }
 923: 
 924: class F16_SP_OPS_uconst<bits<3> i,
 925:   dag outs, dag ins, string asmop> :
 926:   F16_SP_OPS_u7_aligned<i, outs, ins,
 927:   !strconcat(asmop, "\t$b3")> {
 928: 
 929:   let fieldU = 0b00001;
 930: }
 931: 
 932: class F16_SP_OPS_buconst<bits<3> i, string asmop> :
 933:   F16_SP_OPS_u7_aligned<i, (outs), (ins),
 934:     !strconcat(asmop, "\t%blink")> {
 935: 
 936:   let fieldB = 0x000;
```

- EN: This range defines declarative TableGen records such as F16_SP_OPS_uconst, F16_SP_OPS_buconst, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_SP_OPS_uconst, F16_SP_OPS_buconst 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 937-954

```tablegen
 937:   let fieldU = 0b10001;
 938: }
 939: 
 940: class F16_SP_LD<bits<3> i, string asmop> : F16_SP_OPS_u7_aligned<i,
 941:                          (outs GPR32Reduced:$b3), (ins immU<7>:$u7),
 942:                          !strconcat(asmop, "\t$b3, [%sp, $u7]")>;
 943: 
 944: class F16_SP_ST<bits<3> i, string asmop> : F16_SP_OPS_u7_aligned<i,
 945:                          (outs), (ins GPR32Reduced:$b3, immU<7>:$u7),
 946:                          !strconcat(asmop, "\t$b3, [%sp, $u7]")>;
 947: 
 948: // Compact MOV/ADD/CMP Immediate Format.
 949: class F16_OP_IMM<bits<5> opc, dag outs, dag ins, string asmstr> :
 950:   InstARC<2, outs, ins, asmstr, []> {
 951: 
 952:   bits<3> b;
 953:   let Inst{15-11} = opc;
 954:   let Inst{10-8} = b;
```

- EN: This range defines declarative TableGen records such as F16_SP_LD, F16_SP_ST, F16_OP_IMM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_SP_LD, F16_SP_ST, F16_OP_IMM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 955-972

```tablegen
 955: }
 956: 
 957: class F16_OP_U7<bit i, string asmstr> :
 958:   F16_OP_IMM<0b11100, (outs GPR32:$b), (ins immU<7>:$u7), asmstr> {
 959: 
 960:   bits<7> u7;
 961:   let Inst{7} = i;
 962:   let Inst{6-0} = u7;
 963: }
 964: 
 965: // Special types for different instruction operands.
 966: def ccond : Operand<i32> {
 967:   let PrintMethod = "printPredicateOperand";
 968: }
 969: 
 970: def brccond : Operand<i32> {
 971:   let PrintMethod = "printBRCCPredicateOperand";
 972: }
```

- EN: This range defines declarative TableGen records such as F16_OP_U7, ccond, brccond, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_OP_U7, ccond, brccond 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 973-990

```tablegen
 973: 
 974: // Branch/call targets of different offset sizes.
 975: class BCTarget<ValueType vt> : Operand<vt> {
 976:   let OperandType = "OPERAND_PCREL";
 977: }
 978: 
 979: def btarget : BCTarget<OtherVT>;
 980: 
 981: class BCTargetSigned<ValueType vt, int BSz> : BCTarget<vt> {
 982:   let DecoderMethod = "DecodeBranchTargetS<"#BSz#">";
 983: }
 984: 
 985: class BranchTargetS<int BSz> : BCTargetSigned<OtherVT, BSz>;
 986: def btargetS7 : BranchTargetS<7>;
 987: def btargetS8 : BranchTargetS<8>;
 988: def btargetS9 : BranchTargetS<9>;
 989: def btargetS10 : BranchTargetS<10>;
 990: def btargetS13 : BranchTargetS<13>;
```

- EN: This range defines declarative TableGen records such as BCTarget, btarget, BCTargetSigned, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BCTarget, btarget, BCTargetSigned 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 991-1008

```tablegen
 991: def btargetS21 : BranchTargetS<21>;
 992: def btargetS25 : BranchTargetS<25>;
 993: 
 994: class CallTargetS<int BSz> : BCTargetSigned<i32, BSz>;
 995: def calltargetS25: CallTargetS<25>;
 996: 
 997: // Compact Branch on Compare Register with Zero.
 998: class F16_BCC_REG<bit i, string asmstr> :
 999:   InstARC<2, (outs), (ins GPR32:$b, btargetS8:$s8),
1000:   !strconcat(asmstr, "\t$b, 0, $s8"), []> {
1001: 
1002:   bits<3> b;
1003:   bits<8> s8;
1004: 
1005:   let Inst{15-11} = 0b11101;
1006:   let Inst{10-8} = b;
1007:   let Inst{7} = i;
1008:   let Inst{6-0} = s8{7-1};
```

- EN: This range defines declarative TableGen records such as btargetS21, btargetS25, CallTargetS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 btargetS21, btargetS25, CallTargetS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1009-1026

```tablegen
1009:   let s8{0} = 0b0;
1010: }
1011: 
1012: // Compact Branch Conditionally Format.
1013: class F16_BCC<bits<2> i, dag ins, string asmstr> :
1014:   InstARC<2, (outs), ins, asmstr, []> {
1015: 
1016:   let Inst{15-11} = 0b11110;
1017:   let Inst{10-9} = i;
1018: }
1019: 
1020: class F16_BCC_s10<bits<2> i, string asmstr> :
1021:   F16_BCC<i, (ins btargetS10:$s),
1022:   !strconcat(asmstr, "\t$s")> {
1023: 
1024:   bits<10> s;
1025:   let Inst{8-0} = s{9-1};
1026:   let s{0} = 0b0;
```

- EN: This range defines declarative TableGen records such as F16_BCC, F16_BCC_s10, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_BCC, F16_BCC_s10 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1027-1037

```tablegen
1027: }
1028: 
1029: class F16_BCC_s7<bits<3> i, string asmstr> :
1030:   F16_BCC<0b11, (ins btargetS7:$s),
1031:   !strconcat(asmstr, "\t$s")> {
1032: 
1033:   bits<7> s;
1034:   let Inst{8-6} = i;
1035:   let Inst{5-0} = s{6-1};
1036:   let s{0} = 0b0;
1037: }
```

- EN: This range defines declarative TableGen records such as F16_BCC_s7, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F16_BCC_s7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
