# CSKYInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrFormats.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- CSKYInstrFormats.td - CSKY Instruction Formats -----*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: class AddrMode<bits<5> val> {
  10:   bits<5> Value = val;
  11: }
  12: 
  13: def AddrModeNone : AddrMode<0>;
  14: def AddrMode32B : AddrMode<1>;   // ld32.b, ld32.bs, st32.b, st32.bs, +4kb
  15: def AddrMode32H : AddrMode<2>;   // ld32.h, ld32.hs, st32.h, st32.hs, +8kb
  16: def AddrMode32WD : AddrMode<3>;  // ld32.w, st32.w, ld32.d, st32.d, +16kb
  17: def AddrMode16B : AddrMode<4>;   // ld16.b, +32b
  18: def AddrMode16H : AddrMode<5>;   // ld16.h, +64b
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as AddrMode, AddrModeNone, AddrMode32B, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 AddrMode, AddrModeNone, AddrMode32B 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19: def AddrMode16W : AddrMode<6>;   // ld16.w, +128b or +1kb
  20: def AddrMode32SDF : AddrMode<7>; // flds, fldd, +1kb
  21: 
  22: class CSKYInst<AddrMode am, int sz, dag outs, dag ins, string asmstr,
  23:                list<dag> pattern> : Instruction {
  24:   let Namespace = "CSKY";
  25:   int Size = sz;
  26:   AddrMode AM = am;
  27:   let OutOperandList = outs;
  28:   let InOperandList = ins;
  29:   let AsmString = asmstr;
  30:   let Pattern = pattern;
  31:   let Itinerary = NoItinerary;
  32:   let TSFlags{4 - 0} = AM.Value;
  33: }
  34: 
  35: class CSKYPseudo<dag outs, dag ins, string asmstr, list<dag> pattern>
  36:     : CSKYInst<AddrModeNone, 0, outs, ins, asmstr, pattern> {
```

- EN: This range defines declarative TableGen records such as AddrMode16W, AddrMode32SDF, CSKYInst, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 AddrMode16W, AddrMode32SDF, CSKYInst 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 37-54

```tablegen
  37:   let isCodeGenOnly = 1;
  38:   let isPseudo = 1;
  39: }
  40: 
  41: class CSKY32Inst<AddrMode am, bits<6> opcode, dag outs, dag ins, string asmstr,
  42:                  list<dag> pattern>
  43:     : CSKYInst<am, 4, outs, ins, asmstr, pattern> {
  44:   field bits<32> Inst;
  45:   let Inst{31 - 26} = opcode;
  46: }
  47: 
  48: class CSKY16Inst<AddrMode am, dag outs, dag ins, string asmstr, list<dag> pattern>
  49:   : CSKYInst<am, 2, outs, ins, asmstr, pattern> {
  50:   field bits<16> Inst;
  51: }
  52: 
  53: // CSKY 32-bit instruction
  54: // Format< OP[6] | Offset[26] >
```

- EN: This range defines declarative TableGen records such as CSKY32Inst, CSKY16Inst, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKY32Inst, CSKY16Inst 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55: // Instruction(1): bsr32
  56: class J<bits<6> opcode, dag outs, dag ins, string op, list<dag> pattern>
  57:     : CSKY32Inst<AddrModeNone, opcode, outs, ins, !strconcat(op, "\t$offset"),
  58:                  pattern> {
  59:   bits<26> offset;
  60:   let Inst{25 - 0} = offset;
  61:   let isCall = 1;
  62:   let Defs = [ R15 ];
  63: }
  64: 
  65: // Format< OP[6] | RZ[5] | SOP[3] | OFFSET[18] >
  66: // Instructions(7): grs, lrs32.b, lrs32.h, lrs32.w, srs32.b, srs32.h, srs32.w
  67: class I_18_Z_L<bits<3> sop, string asm, dag outs, dag ins, list<dag> pattern>
  68:     : CSKY32Inst<AddrModeNone, 0x33, outs, ins, asm, pattern> {
  69:   bits<5> rz;
  70:   bits<18> offset;
  71:   let Inst{25 - 21} = rz;
  72:   let Inst{20 - 18} = sop;
```

- EN: This range defines declarative TableGen records such as J, I_18_Z_L, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 J, I_18_Z_L 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   let Inst{17 - 0} = offset;
  74: }
  75: 
  76: // Format< OP[6] | RZ[5] | RX[5] | IMM[16] >
  77: // Instructions(1): ori32
  78: class I_16_ZX<string op, ImmLeaf ImmType, list<dag> pattern>
  79:     : CSKY32Inst<AddrModeNone, 0x3b,
  80:                  (outs GPR:$rz), (ins GPR:$rx,ImmType:$imm16),
  81:                  !strconcat(op, "\t$rz, $rx, $imm16"), pattern> {
  82:   bits<5> rz;
  83:   bits<5> rx;
  84:   bits<16> imm16;
  85:   let Inst{25 - 21} = rz;
  86:   let Inst{20 - 16} = rx;
  87:   let Inst{15 - 0} = imm16;
  88: }
  89: 
  90: // Format< OP[6] | SOP[5] | RZ[5] | IMM[16] >
```

- EN: This range defines declarative TableGen records such as I_16_ZX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_ZX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91: // Instructions(3): movi32, movih32, (bgeni32)
  92: class I_16_MOV<bits<5> sop, string op, ImmLeaf ImmType>
  93:     : CSKY32Inst<AddrModeNone, 0x3a, (outs GPR:$rz), (ins ImmType:$imm16),
  94:                  !strconcat(op, "\t$rz, $imm16"),
  95:                  [(set GPR:$rz, ImmType:$imm16)]> {
  96:   bits<5> rz;
  97:   bits<16> imm16;
  98:   let Inst{25 - 21} = sop;
  99:   let Inst{20 - 16} = rz;
 100:   let Inst{15 - 0} = imm16;
 101:   let isReMaterializable = 1;
 102:   let isAsCheapAsAMove = 1;
 103:   let isMoveImm = 1;
 104: }
 105: 
 106: // Format< OP[6] | SOP[5] | RZ[5] | OFFSET[16] >
 107: // Instructions(1): lrw32
 108: class I_16_Z_L<bits<5> sop, string op, dag ins, list<dag> pattern>
```

- EN: This range defines declarative TableGen records such as I_16_MOV, I_16_Z_L, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_MOV, I_16_Z_L 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:   : CSKY32Inst<AddrModeNone, 0x3a, (outs GPR:$rz), ins,
 110:   !strconcat(op, "\t$rz, $imm16"), pattern> {
 111:   bits<5> rz;
 112:   bits<16> imm16;
 113:   let Inst{25 - 21} = sop;
 114:   let Inst{20 - 16} = rz;
 115:   let Inst{15 - 0} = imm16;
 116: }
 117: 
 118: // Format< OP[6] | SOP[5] | 00000[5] | OFFSET[16] >
 119: // Instructions(5): bt32, bf32, br32, jmpi32, jsri32
 120: class I_16_L<bits<5> sop, dag outs, dag ins, string asm, list<dag> pattern>
 121:     : CSKY32Inst<AddrModeNone, 0x3a, outs, ins, asm, pattern> {
 122:   bits<16> imm16;
 123:   let Inst{25 - 21} = sop;
 124:   let Inst{20 - 16} = 0;
 125:   let Inst{15 - 0} = imm16;
 126: }
```

- EN: This range defines declarative TableGen records such as I_16_L, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_L 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127: 
 128: // Format< OP[6] | SOP[5] | RX[5] | 0000000000000000[16] >
 129: // Instructions(2): jmp32, jsr32
 130: class I_16_JX<bits<5> sop, string op, list<dag> pattern>
 131:     : CSKY32Inst<AddrModeNone, 0x3a, (outs), (ins GPR:$rx),
 132:                  !strconcat(op, "\t$rx"), pattern> {
 133:   bits<5> rx;
 134:   bits<16> imm16;
 135:   let Inst{25 - 21} = sop;
 136:   let Inst{20 - 16} = rx;
 137:   let Inst{15 - 0} = 0;
 138: }
 139: 
 140: // Format< OP[6] | SOP[5] | RX[5] | 00000000000000[14] | IMM[2] >
 141: // Instructions(1): jmpix32
 142: class I_16_J_XI<bits<5> sop, string op, Operand operand, list<dag> pattern>
 143:     : CSKY32Inst<AddrModeNone, 0x3a, (outs),
 144:                  (ins GPR:$rx, operand:$imm2),
```

- EN: This range defines declarative TableGen records such as I_16_JX, I_16_J_XI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_JX, I_16_J_XI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:                  !strconcat(op, "\t$rx, $imm2"), pattern> {
 146:   bits<5> rx;
 147:   bits<2> imm2;
 148:   let Inst{25 - 21} = sop;
 149:   let Inst{20 - 16} = rx;
 150:   let Inst{15 - 2} = 0;
 151:   let Inst{1 - 0} = imm2;
 152: }
 153: 
 154: // Format< OP[6] | SOP[5] | PCODE[5] | 0000000000000000[16] >
 155: // Instructions(1): rts32
 156: class I_16_RET<bits<5> sop, bits<5> pcode, string op, list<dag> pattern>
 157:     : CSKY32Inst<AddrModeNone, 0x3a, (outs), (ins), op, pattern> {
 158:   let Inst{25 - 21} = sop;
 159:   let Inst{20 - 16} = pcode;
 160:   let Inst{15 - 0} = 0;
 161:   let isTerminator = 1;
 162:   let isReturn = 1;
```

- EN: This range defines declarative TableGen records such as I_16_RET, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_RET 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:   let isBarrier = 1;
 164:   let Uses = [ R15 ];
 165: }
 166: 
 167: // Format< OP[6] | SOP[5] | RX[5] | IMM16[16] >
 168: // Instructions(3): cmpnei32, cmphsi32, cmplti32
 169: class I_16_X<bits<5> sop, string op, Operand operand>
 170:     : CSKY32Inst<AddrModeNone, 0x3a, (outs CARRY:$ca),
 171:                  (ins GPR:$rx, operand:$imm16),
 172:                  !strconcat(op, "\t$rx, $imm16"), []> {
 173:   bits<0> ca;
 174:   bits<16> imm16;
 175:   bits<5> rx;
 176:   let Inst{25 - 21} = sop;
 177:   let Inst{20 - 16} = rx;
 178:   let Inst{15 - 0} = imm16;
 179:   let isCompare = 1;
 180: }
```

- EN: This range defines declarative TableGen records such as I_16_X, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_X 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181: 
 182: // Format< OP[6] | SOP[5] | RX[5] | OFFSET[16] >
 183: // Instructions(7): bez32, bnez32, bnezad32, bhz32, blsz32, blz32, bhsz32
 184: class I_16_X_L<bits<5> sop, string op, Operand operand>
 185:     : CSKY32Inst<AddrModeNone, 0x3a, (outs), (ins GPR:$rx, operand:$imm16),
 186:                  !strconcat(op, "\t$rx, $imm16"), []> {
 187:   bits<5> rx;
 188:   bits<16> imm16;
 189:   let Inst{25 - 21} = sop;
 190:   let Inst{20 - 16} = rx;
 191:   let Inst{15 - 0} = imm16;
 192:   let isBranch = 1;
 193:   let isTerminator = 1;
 194: }
 195: 
 196: // Format< OP[6] | RZ[5] | RX[5] | SOP[4] | IMM[12] >
 197: // Instructions(5): addi32, subi32, andi32, andni32, xori32
 198: class I_12<bits<4> sop, string op, SDNode node, ImmLeaf ImmType>
```

- EN: This range defines declarative TableGen records such as I_16_X_L, I_12, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_16_X_L, I_12 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199:     : CSKY32Inst<AddrModeNone, 0x39, (outs GPR:$rz),
 200:     (ins GPR:$rx, ImmType:$imm12), !strconcat(op, "\t$rz, $rx, $imm12"),
 201:     [(set GPR:$rz, (node GPR:$rx, ImmType:$imm12))]> {
 202:   bits<5> rz;
 203:   bits<5> rx;
 204:   bits<12> imm12;
 205:   let Inst{25 - 21} = rz;
 206:   let Inst{20 - 16} = rx;
 207:   let Inst{15 - 12} = sop;
 208:   let Inst{11 - 0} = imm12;
 209: }
 210: 
 211: class I_LDST<AddrMode am, bits<6> opcode, bits<4> sop, dag outs, dag ins,
 212:              string op, list<dag> pattern>
 213:     : CSKY32Inst<am, opcode, outs, ins, !strconcat(op, "\t$rz, ($rx, ${imm12})"),
 214:                  pattern> {
 215:   bits<5> rx;
 216:   bits<5> rz;
```

- EN: This range defines declarative TableGen records such as I_LDST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_LDST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217:   bits<12> imm12;
 218:   let Inst{25 - 21} = rz;
 219:   let Inst{20 - 16} = rx;
 220:   let Inst{15 - 12} = sop;
 221:   let Inst{11 - 0} = imm12;
 222: }
 223: 
 224: class I_PLDR<AddrMode am, bits<6> opcode, bits<4> sop, dag outs, dag ins,
 225:              string op, list<dag> pattern>
 226:     : CSKY32Inst<am, opcode, outs, ins, !strconcat(op, "\t($rx, ${imm12})"),
 227:                  pattern> {
 228:   bits<5> rx;
 229:   bits<12> imm12;
 230:   let Inst{25 - 21} = 0;
 231:   let Inst{20 - 16} = rx;
 232:   let Inst{15 - 12} = sop;
 233:   let Inst{11 - 0} = imm12;
 234: }
```

- EN: This range defines declarative TableGen records such as I_PLDR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_PLDR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-252

```tablegen
 235: 
 236: 
 237: // Format< OP[6] | RZ[5] | RX[5] | SOP[4] | OFFSET[12] >
 238: // Instructions(6): ld32.b, ld32.bs, ld32.h, ld32.hs, ld32.w
 239: class I_LD<AddrMode am, bits<4> sop, string op, Operand operand>
 240:     : I_LDST<am, 0x36, sop,
 241:     (outs GPR:$rz), (ins GPR:$rx, operand:$imm12), op, []>;
 242: 
 243: // Format< OP[6] | RZ[5] | RX[5] | SOP[4] | OFFSET[12] >
 244: // Instructions(4): st32.b, st32.h, st32.w
 245: class I_ST<AddrMode am, bits<4> sop, string op, Operand operand>
 246:     : I_LDST<am, 0x37, sop, (outs),
 247:     (ins GPR:$rz, GPR:$rx, operand:$imm12), op, []>;
 248: 
 249: // Format< OP[6] | SOP[5] | PCODE[5] | 0000[4] | 000 | R28 | LIST2[3] | R15 |
 250: // LIST1[4] >
 251: // Instructions(2): push32, pop32
 252: class I_12_PP<bits<5> sop, bits<5> pcode, dag outs, dag ins, string op>
```

- EN: This range defines declarative TableGen records such as I_LD, I_ST, I_12_PP, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 I_LD, I_ST, I_12_PP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 253-270

```tablegen
 253:     : CSKY32Inst<AddrModeNone, 0x3a, outs, ins, !strconcat(op, "\t$regs"), []> {
 254:   bits<12> regs;
 255:   let Inst{25 - 21} = sop;
 256:   let Inst{20 - 16} = pcode;
 257:   let Inst{15 - 12} = 0;
 258:   let Inst{11 - 0} = regs;
 259:   let Uses = [R14];
 260:   let Defs = [R14];
 261: }
 262: 
 263: // Format< OP[6] | RZ[5] | RX[5] | SOP[6] | PCODE[5] | IMM[5]>
 264: // Instructions(4): incf32, inct32, decf32, dect32
 265: class I_5_ZX<bits<6> sop, bits<5> pcode, string op, ImmLeaf ImmType,
 266:              list<dag> pattern>
 267:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz),
 268:                  (ins CARRY:$cond, GPR:$false, GPR:$rx, ImmType:$imm5),
 269:                  !strconcat(op, "\t$rz, $rx, $imm5"), pattern> {
 270:   bits<0> cond;
```

- EN: This range defines declarative TableGen records such as I_5_ZX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_ZX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271:   bits<5> rz;
 272:   bits<5> rx;
 273:   bits<5> imm5;
 274:   let Inst{25 - 21} = rz;
 275:   let Inst{20 - 16} = rx;
 276:   let Inst{15 - 10} = sop;
 277:   let Inst{9 - 5} = pcode;
 278:   let Inst{4 - 0} = imm5;
 279:   let Constraints = "$rz = $false";
 280: }
 281: 
 282: // Format< OP[6] | IMM[5] | RX[5] | SOP[6] | PCODE[5] | RZ[5]>
 283: // Instructions(13): decgt32, declt32, decne32, lsli32, lslc32, lsri32
 284: //                   lsrc32, asri32, asrc32, rotli32, xsr32, bclri32, bseti32
 285: class I_5_XZ<bits<6> sop, bits<5> pcode, string op, dag outs, dag ins,
 286:              list<dag> pattern>
 287:     : CSKY32Inst<AddrModeNone, 0x31, outs, ins,
 288:                  !strconcat(op, "\t$rz, $rx, $imm5"), pattern> {
```

- EN: This range defines declarative TableGen records such as I_5_XZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289:   bits<5> imm5;
 290:   bits<5> rx;
 291:   bits<5> rz;
 292:   let Inst{25 - 21} = imm5;
 293:   let Inst{20 - 16} = rx;
 294:   let Inst{15 - 10} = sop;
 295:   let Inst{9 - 5} = pcode;
 296:   let Inst{4 - 0} = rz;
 297: }
 298: 
 299: // mtcr32, mfcr32
 300: class I_5_XZ_CR<bits<6> sop, bits<5> pcode, string opStr, dag outs, dag ins,
 301:              list<dag> pattern>
 302:     : CSKY32Inst<AddrModeNone, 0x30, outs, ins, opStr, pattern> {
 303:   bits<5> sel;
 304:   bits<5> rx;
 305:   bits<5> cr;
 306:   let Inst{25 - 21} = sel;
```

- EN: This range defines declarative TableGen records such as I_5_XZ_CR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_CR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307:   let Inst{20 - 16} = rx;
 308:   let Inst{15 - 10} = sop;
 309:   let Inst{9 - 5} = pcode;
 310:   let Inst{4 - 0} = cr;
 311: }
 312: 
 313: // sync
 314: class I_5_XZ_SYNC<bits<6> sop, bits<5> pcode, string opStr, bits<1> S, bits<1> I>
 315:     : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins), opStr, []> {
 316:   let Inst{25 - 21} = 0;
 317:   let Inst{20 - 16} = 0;
 318:   let Inst{15 - 10} = sop;
 319:   let Inst{9 - 5} = pcode;
 320:   let Inst{4 - 0} = 0;
 321:   let Inst{25} = S;
 322:   let Inst{21} = I;
 323: 
 324: }
```

- EN: This range defines declarative TableGen records such as I_5_XZ_SYNC, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_SYNC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 325-342

```tablegen
 325: 
 326: // Priviledged Instructions
 327: class I_5_XZ_PRIVI<bits<6> sop, bits<5> pcode, string opStr>
 328:     : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins), opStr, []> {
 329:   let Inst{25 - 21} = 0;
 330:   let Inst{20 - 16} = 0;
 331:   let Inst{15 - 10} = sop;
 332:   let Inst{9 - 5} = pcode;
 333:   let Inst{4 - 0} = 0;
 334: }
 335: 
 336: class I_CP<bits<4> sop, dag outs, dag ins, string opStr>
 337:     : CSKY32Inst<AddrModeNone, 0x3f, outs, ins, opStr, []> {
 338:   bits<5> cpid;
 339:   bits<12> usdef;
 340:   let Inst{25 - 21} = cpid;
 341:   let Inst{20 - 16} = 0;
 342:   let Inst{15 - 12} = sop;
```

- EN: This range defines declarative TableGen records such as I_5_XZ_PRIVI, I_CP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_PRIVI, I_CP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343:   let Inst{11 - 0} = usdef;
 344: }
 345: 
 346: class I_CPOP<dag outs, dag ins, string opStr>
 347:     : CSKY32Inst<AddrModeNone, 0x3f, outs, ins, opStr, []> {
 348:   bits<5> cpid;
 349:   bits<20> usdef;
 350:   let Inst{25 - 21} = cpid;
 351:   let Inst{20 - 16} = usdef{19-15};
 352:   let Inst{15} = 1;
 353:   let Inst{14 - 0} = usdef{14-0};
 354: }
 355: 
 356: class I_CP_Z<bits<4> sop, dag outs, dag ins, string opStr>
 357:     : CSKY32Inst<AddrModeNone, 0x3f, outs, ins, opStr, []> {
 358:   bits<5> cpid;
 359:   bits<12> usdef;
 360:   bits<5> rz;
```

- EN: This range defines declarative TableGen records such as I_CPOP, I_CP_Z, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_CPOP, I_CP_Z 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361: 
 362:   let Inst{25 - 21} = cpid;
 363:   let Inst{20 - 16} = rz;
 364:   let Inst{15 - 12} = sop;
 365:   let Inst{11 - 0} = usdef;
 366: }
 367: 
 368: class I_5_CACHE<bits<6> sop, bits<5> pcode, string opStr>
 369:     : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins), opStr, []> {
 370:   let Inst{25 - 21} = pcode;
 371:   let Inst{20 - 16} = 0;
 372:   let Inst{15 - 10} = sop;
 373:   let Inst{9 - 5} = 0b00001;
 374:   let Inst{4 - 0} = 0;
 375: }
 376: 
 377: class I_5_X_CACHE<bits<6> sop, bits<5> pcode, string opStr>
 378:     : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins GPR:$rx), opStr #"\t$rx", []> {
```

- EN: This range defines declarative TableGen records such as I_5_CACHE, I_5_X_CACHE, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_CACHE, I_5_X_CACHE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379:   bits<5> rx;
 380: 
 381:   let Inst{25 - 21} = pcode;
 382:   let Inst{20 - 16} = rx;
 383:   let Inst{15 - 10} = sop;
 384:   let Inst{9 - 5} = 0b00001;
 385:   let Inst{4 - 0} = 0;
 386: }
 387: 
 388: // Format< OP[6] | RY[5] | RX[5] | SOP[6] | PCODE[5] | IMM[5]>
 389: // Instructions(2): ldm32, (ldq32), stm32, (stq32)
 390: class I_5_YX<bits<6> opcode, bits<6> sop, dag outs, dag ins, string opStr, list<dag> pattern>
 391:     : CSKY32Inst<AddrModeNone, opcode, outs, ins, opStr, pattern> {
 392:   bits<10> regs;
 393:   bits<5> rx;
 394: 
 395:   let Inst{25 - 21} = regs{9 - 5}; // ry
 396:   let Inst{20 - 16} = rx;
```

- EN: This range defines declarative TableGen records such as I_5_YX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_YX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397:   let Inst{15 - 10} = sop;
 398:   let Inst{9 - 5} = 0b00001;
 399:   let Inst{4 - 0} = regs{4 - 0}; // imm5
 400: }
 401: 
 402: // Format< OP[6] | LSB[5] | RX[5] | SOP[6] | MSB[5] | RZ[5]>
 403: // Instructions(6): zext32, zextb32, zexth32, sext32, sextb32, sexth32
 404: class I_5_XZ_U<bits<6> sop, dag outs, dag ins, string op, list<dag> pattern>
 405:     : CSKY32Inst<AddrModeNone, 0x31, outs, ins, op #"\t$rz, $rx, $msb, $lsb",
 406:                  pattern> {
 407:   bits<5> rx;
 408:   bits<5> rz;
 409:   bits<5> msb;
 410:   bits<5> lsb;
 411:   let Inst{25 - 21} = lsb; // lsb
 412:   let Inst{20 - 16} = rx;
 413:   let Inst{15 - 10} = sop;
 414:   let Inst{9 - 5} = msb; // msb
```

- EN: This range defines declarative TableGen records such as I_5_XZ_U, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_U 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415:   let Inst{4 - 0} = rz;
 416: }
 417: 
 418: class I_5_XZ_INS<bits<6> sop, dag outs, dag ins, string op, list<dag> pattern>
 419:     : CSKY32Inst<AddrModeNone, 0x31, outs, ins, op #"\t$rz, $rx, $msb, $lsb",
 420:                  pattern> {
 421:   bits<5> rx;
 422:   bits<5> rz;
 423:   bits<5> msb;
 424:   bits<5> lsb;
 425:   let Inst{25 - 21} = rz;
 426:   let Inst{20 - 16} = rx;
 427:   let Inst{15 - 10} = sop;
 428:   let Inst{9 - 5} = msb;
 429:   let Inst{4 - 0} = lsb;
 430: }
 431: 
 432: // Format< OP[6] | LSB[5] | RX[5] | SOP[6] | MSB[5] | RZ[5]>
```

- EN: This range defines declarative TableGen records such as I_5_XZ_INS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_INS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433: // Instructions(6): zext32, zextb32, zexth32, sext32, sextb32, sexth32
 434: class I_5_XZ_U2<bits<6> sop, bits<5> lsb, bits<5> msb, dag outs, dag ins,
 435:   string op, list<dag> pattern>
 436:   : CSKY32Inst<AddrModeNone, 0x31, outs, ins, !strconcat(op, "\t$rz, $rx"), pattern> {
 437:   bits<5> rx;
 438:   bits<5> rz;
 439:   let Inst{25 - 21} = lsb;     // lsb
 440:   let Inst{20 - 16} = rx;
 441:   let Inst{15 - 10} = sop;
 442:   let Inst{9 - 5} = msb;       // msb
 443:   let Inst{4 - 0} = rz;
 444: }
 445: 
 446: // Format< OP[6] | RZ[5] | RX[5] | SOP[6] | SIZE[5] | LSB[5]>
 447: // Instructions(1): ins32
 448: class I_5_ZX_U<bits<6> sop, string op, Operand operand, list<dag> pattern>
 449:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz), (ins operand:$size_lsb),
 450:                  !strconcat(op, "\t$rz, operand:$size_lsb"), pattern> {
```

- EN: This range defines declarative TableGen records such as I_5_XZ_U2, I_5_ZX_U, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_U2, I_5_ZX_U 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451:   bits<10> size_lsb;
 452:   bits<5> rz;
 453:   bits<5> rx;
 454:   let Inst{25 - 21} = rz;
 455:   let Inst{20 - 16} = rx;
 456:   let Inst{15 - 10} = sop;
 457:   let Inst{9 - 5} = size_lsb{9 - 5}; // size
 458:   let Inst{4 - 0} = size_lsb{4 - 0}; // lsb
 459: }
 460: 
 461: // sextb, sexth
 462: class I_5_XZ_US<bits<6> sop, bits<5> lsb, bits<5> msb, string op,
 463:   SDNode opnode, ValueType type>
 464:   : I_5_XZ_U2<sop, lsb, msb, (outs GPR:$rz), (ins GPR:$rx), op,
 465:   [(set GPR:$rz, (opnode GPR:$rx, type))]>;
 466: 
 467: class I_5_XZ_UZ<bits<6> sop, bits<5> lsb, bits<5> msb, string op, int v>
 468:   : I_5_XZ_U2<sop, lsb, msb, (outs GPR:$rz), (ins GPR:$rx), op,
```

- EN: This range defines declarative TableGen records such as I_5_XZ_US, I_5_XZ_UZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_XZ_US, I_5_XZ_UZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469:   [(set GPR:$rz, (and GPR:$rx, (i32 v)))]>;
 470: 
 471: // Format< OP[6] | IMM[5] | RX[5] | SOP[6] | PCODE[5] | 00000 >
 472: // Instructions(1): btsti32
 473: class I_5_X<bits<6> sop, bits<5> pcode, string op, ImmLeaf ImmType,
 474:             list<dag> pattern>
 475:     : CSKY32Inst<AddrModeNone, 0x31, (outs CARRY:$ca),
 476:                  (ins GPR:$rx, ImmType:$imm5),
 477:                  !strconcat(op, "\t$rx, $imm5"), pattern> {
 478:   bits<0> ca;
 479:   bits<5> imm5;
 480:   bits<5> rx;
 481:   let Inst{25 - 21} = imm5;
 482:   let Inst{20 - 16} = rx;
 483:   let Inst{15 - 10} = sop;
 484:   let Inst{9 - 5} = pcode;
 485:   let Inst{4 - 0} = 0;
 486:   let isCompare = 1;
```

- EN: This range defines declarative TableGen records such as I_5_X, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_X 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 487-504

```tablegen
 487: }
 488: 
 489: // Format< OP[6] | IMM[5] | 00000[5] | SOP[6] | PCODE[5] | RZ[5]>
 490: // Instructions(1): bmaski32
 491: class I_5_Z<bits<6> sop, bits<5> pcode, string op, ImmLeaf ImmType,
 492:             list<dag> pattern>
 493:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz), (ins ImmType:$imm5),
 494:                  !strconcat(op, "\t$rz, $imm5"), pattern> {
 495:   bits<5> imm5;
 496:   bits<5> rz;
 497:   let Inst{25 - 21} = imm5;
 498:   let Inst{20 - 16} = 0;
 499:   let Inst{15 - 10} = sop;
 500:   let Inst{9 - 5} = pcode;
 501:   let Inst{4 - 0} = rz;
 502: }
 503: 
 504: class I_5_IMM5<bits<6> opcode, bits<6> sop, bits<5> pcode, string op, ImmLeaf ImmType,
```

- EN: This range defines declarative TableGen records such as I_5_Z, I_5_IMM5, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_5_Z, I_5_IMM5 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 505-522

```tablegen
 505:             list<dag> pattern>
 506:     : CSKY32Inst<AddrModeNone, opcode, (outs), (ins ImmType:$imm5),
 507:                  !strconcat(op, "\t$imm5"), pattern> {
 508:   bits<5> imm5;
 509:   let Inst{25 - 21} = imm5;
 510:   let Inst{20 - 16} = 0;
 511:   let Inst{15 - 10} = sop;
 512:   let Inst{9 - 5} = pcode;
 513:   let Inst{4 - 0} = 0;
 514: }
 515: 
 516: // Format< OP[6] | RY[5] | RX[5] | SOP[6] | PCODE[5] | RZ[5] >
 517: // Instructions(24): addu32, addc32, subu32, subc32, (rsub32), ixh32, ixw32,
 518: // ixd32, and32, andn32, or32, xor32, nor32, lsl32, lsr32, asr32, rotl32
 519: // mult32, divu32, divs32, mul.(u/s)32, mula.32.l, mula.u32, mulall.s16.s
 520: class R_YXZ<bits<6> opcode, bits<6> sop, bits<5> pcode, dag outs, dag ins,
 521:             string op, list<dag> pattern>
 522:     : CSKY32Inst<AddrModeNone, opcode, outs, ins,
```

- EN: This range defines declarative TableGen records such as R_YXZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_YXZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523:                  !strconcat(op, "\t$rz, $rx, $ry"), pattern> {
 524:   bits<5> ry;
 525:   bits<5> rx;
 526:   bits<5> rz;
 527:   let Inst{25 - 21} = ry;
 528:   let Inst{20 - 16} = rx;
 529:   let Inst{15 - 10} = sop;
 530:   let Inst{9 - 5} = pcode;
 531:   let Inst{4 - 0} = rz;
 532: }
 533: 
 534: // R_YXZ instructions with simple pattern
 535: // Output: GPR:rz
 536: // Input: GPR:rx, GPR:ry
 537: // Asm string: op rz, rx, ry
 538: // Instructions: addu32, subu32, ixh32, ixw32, ixd32, and32, andn32, or32,
 539: // xor32, nor32, lsl32, lsr32, asr32, mult32, divu32, divs32
 540: class R_YXZ_SP_F1<bits<6> sop, bits<5> pcode, PatFrag opnode, string op,
```

- EN: This range defines declarative TableGen records such as R_YXZ_SP_F1, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_YXZ_SP_F1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541:   bit Commutable = 0> : R_YXZ<0x31, sop, pcode, (outs GPR:$rz),
 542:   (ins GPR:$rx, GPR:$ry), op, [(set GPR:$rz, (opnode GPR:$rx, GPR:$ry))]> {
 543:   let isCommutable = Commutable;
 544: }
 545: 
 546: // Format< OP[6] | RY[5] | RX[5] | SOP[6] | PCODE[5] | RZ[5] >
 547: // Instructions:(8) ldr32.b, ldr32.h, ldr32.bs, ldr32.hs, ldr32.w,
 548: //                  str32.b, str32.h, str32.w
 549: class R_YXZ_LDST<bits<6> opcode, bits<6> sop, dag outs,
 550:                  dag ins, string op, list<dag> pattern>
 551:     : CSKY32Inst<AddrModeNone, opcode, outs, ins,
 552:                  op # "\t$rz, ($rx, $ry << ${imm})", pattern> {
 553:   bits<5> rx;
 554:   bits<5> ry;
 555:   bits<5> rz;
 556:   bits<5> imm;
 557:   let Inst{25 - 21} = ry; // ry;
 558:   let Inst{20 - 16} = rx; // rx;
```

- EN: This range defines declarative TableGen records such as R_YXZ_LDST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_YXZ_LDST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 559-576

```tablegen
 559:   let Inst{15 - 10} = sop;
 560:   let Inst{9 - 5} = imm;  // pcode;
 561:   let Inst{4 - 0} = rz;
 562: }
 563: 
 564: class I_LDR<bits<6> sop, string op> : R_YXZ_LDST<0x34, sop,
 565:   (outs GPR:$rz), (ins GPR:$rx, GPR:$ry, uimm_shift:$imm), op, []>;
 566: 
 567: class I_STR<bits<6> sop, string op> : R_YXZ_LDST<0x35, sop,
 568:   (outs), (ins GPR:$rz, GPR:$rx, GPR:$ry, uimm_shift:$imm), op, []>;
 569: 
 570: // Format< OP[6] | RX[5] | RX[5] | SOP[6] | PCODE[5] | RZ[5] >
 571: // Instructions:(1) not32
 572: class R_XXZ<bits<6> sop, bits<5> pcode, dag outs, dag ins, string op,
 573:             list<dag> pattern>
 574:     : CSKY32Inst<AddrModeNone, 0x31, outs, ins, !strconcat(op, "\t$rz, $rx"),
 575:                  pattern> {
 576:   bits<5> rx;
```

- EN: This range defines declarative TableGen records such as I_LDR, I_STR, R_XXZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I_LDR, I_STR, R_XXZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 577-594

```tablegen
 577:   bits<5> rz;
 578:   let Inst{25 - 21} = rx;
 579:   let Inst{20 - 16} = rx;
 580:   let Inst{15 - 10} = sop;
 581:   let Inst{9 - 5} = pcode;
 582:   let Inst{4 - 0} = rz;
 583: }
 584: 
 585: // Format< OP[6] | RY[5] | RX[5] | SOP[6] | PCODE[5] | 00000[5] >
 586: // Instructions:(4) cmpne32, cmphs32, cmplt32, tst32
 587: class R_YX<bits<6> sop, bits<5> pcode, string op>
 588:     : CSKY32Inst<AddrModeNone, 0x31, (outs CARRY:$ca), (ins GPR:$rx, GPR:$ry),
 589:                  !strconcat(op, "\t$rx, $ry"), []> {
 590:   bits<0> ca;
 591:   bits<5> ry;
 592:   bits<5> rx;
 593:   let Inst{25 - 21} = ry;
 594:   let Inst{20 - 16} = rx;
```

- EN: This range defines declarative TableGen records such as R_YX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_YX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 595-612

```tablegen
 595:   let Inst{15 - 10} = sop;
 596:   let Inst{9 - 5} = pcode;
 597:   let Inst{4 - 0} = 0;
 598:   let isCompare = 1;
 599: }
 600: 
 601: // Format< OP[6] | 00000[5] | RX[5] | SOP[6] | PCODE[5] | RZ[5] >
 602: // Instructions:(12)
 603: //   mov32, xtrb0.32, xtrb1.32, xtrb2.32, xtrb3.32, brev32, revb32
 604: //   revh32, abs32, ff0.32, ff1.32, bgenr32
 605: class R_XZ<bits<6> sop, bits<5> pcode, string op>
 606:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz), (ins GPR:$rx),
 607:                  !strconcat(op, "\t$rz, $rx"), []> {
 608:   bits<5> rx;
 609:   bits<5> rz;
 610:   let Inst{25 - 21} = 0;
 611:   let Inst{20 - 16} = rx;
 612:   let Inst{15 - 10} = sop;
```

- EN: This range defines declarative TableGen records such as R_XZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_XZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 613-630

```tablegen
 613:   let Inst{9 - 5} = pcode;
 614:   let Inst{4 - 0} = rz;
 615: }
 616: 
 617: // Format< OP[6] | RZ[5] | RX[5] | SOP[6] | PCODE[5] | 00000[5] >
 618: // Instructions:(2) movf32, movt32
 619: class R_ZX<bits<6> sop, bits<5> pcode, string op, list<dag> pattern>
 620:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz),
 621:                  (ins CARRY:$ca, GPR:$rx, GPR:$false),
 622:                  !strconcat(op, "\t$rz, $rx"), pattern> {
 623:   bits<5> rz;
 624:   bits<5> rx;
 625:   let Inst{25 - 21} = rz;
 626:   let Inst{20 - 16} = rx;
 627:   let Inst{15 - 10} = sop;
 628:   let Inst{9 - 5} = pcode;
 629:   let Inst{4 - 0} = 0;
 630:   let Constraints = "$rz = $false";
```

- EN: This range defines declarative TableGen records such as R_ZX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_ZX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 631-648

```tablegen
 631:   let isSelect = 1;
 632: }
 633: 
 634: // Format< OP[6] | 00000[5] | RX[5] | SOP[6] | PCODE[5] | 00000[5] >
 635: // Instructions:(1) tstnbz32
 636: class R_X<bits<6> sop, bits<5> pcode, dag outs, dag ins, string op, list<dag> pattern>
 637:     : CSKY32Inst<AddrModeNone, 0x31, outs, ins, !strconcat(op, "\t$rx"), pattern> {
 638:   bits<5> rx;
 639:   let Inst{25 - 21} = 0;
 640:   let Inst{20 - 16} = rx;
 641:   let Inst{15 - 10} = sop;
 642:   let Inst{9 - 5} = pcode;
 643:   let Inst{4 - 0} = 0;
 644: }
 645: 
 646: // Format< OP[6] | 00000[5] | 00000[5] | SOP[6] | PCODE[5] | RZ[5] >
 647: // Instructions:(2) mvc32, mvcv32
 648: class R_Z_1<bits<6> sop, bits<5> pcode, string op>
```

- EN: This range defines declarative TableGen records such as R_X, R_Z_1, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_X, R_Z_1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz), (ins CARRY:$ca),
 650:                  !strconcat(op, "\t$rz"), []> {
 651:   bits<0> ca;
 652:   bits<5> rz;
 653:   let Inst{25 - 21} = 0;
 654:   let Inst{20 - 16} = 0;
 655:   let Inst{15 - 10} = sop;
 656:   let Inst{9 - 5} = pcode;
 657:   let Inst{4 - 0} = rz;
 658: }
 659: 
 660: // Format< OP[6] | RZ[5] | 00000[5] | SOP[6] | PCODE[5] | 00000[5] >
 661: // Instructions:(2) clrf32, clrt32
 662: class R_Z_2<bits<6> sop, bits<5> pcode, string op>
 663:     : CSKY32Inst<AddrModeNone, 0x31, (outs GPR:$rz),
 664:                  (ins CARRY:$ca, GPR:$false), !strconcat(op, "\t$rz"), []> {
 665:   bits<0> ca;
 666:   bits<5> rz;
```

- EN: This range defines declarative TableGen records such as R_Z_2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R_Z_2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 667-684

```tablegen
 667:   let Inst{25 - 21} = rz;
 668:   let Inst{20 - 16} = 0;
 669:   let Inst{15 - 10} = sop;
 670:   let Inst{9 - 5} = pcode;
 671:   let Inst{4 - 0} = 0;
 672:   let Constraints = "$rz = $false";
 673: }
 674: 
 675: class BAR<bits<5> sop, string op, bits<1> signed>
 676:   : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins), op, []> {
 677:   let Inst{25} = signed;
 678:   let Inst{24 - 16} = 0;
 679:   let Inst{15 - 5} = 0x421;
 680:   let Inst{4 - 0} = sop;
 681:   let hasSideEffects = 1;
 682:   let mayLoad = 0;
 683:   let mayStore = 0;
 684: }
```

- EN: This range defines declarative TableGen records such as BAR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BAR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
