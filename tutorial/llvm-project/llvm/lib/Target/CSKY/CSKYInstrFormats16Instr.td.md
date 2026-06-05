# CSKYInstrFormats16Instr.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrFormats16Instr.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- CSKYInstrFormats16Instr.td - 16-bit Instr. Formats -*- tablegen --*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: class J16<bits<5> sop, string opstr, dag ins>
  10:   : CSKY16Inst<AddrModeNone, (outs), ins,
  11:     !strconcat(opstr, "\t$offset"), []> {
  12:   bits<10> offset;
  13:   let Inst{15} = 0;
  14:   let Inst{14 - 10} = sop;
  15:   let Inst{9 - 0} = offset;
  16: }
  17: 
  18: class J16_B<bits<5> sop, string opstr>
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as J16, J16_B, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 J16, J16_B 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19:     : CSKY16Inst<AddrModeNone, (outs), (ins CARRY:$ca, br_symbol_16bit:$offset),
  20:                  !strconcat(opstr, "\t$offset"), []> {
  21:   bits<0> ca;
  22:   bits<10> offset;
  23:   let Inst{15} = 0;
  24:   let Inst{14 - 10} = sop;
  25:   let Inst{9 - 0} = offset;
  26: }
  27: 
  28: class R16_XYZ<bits<2> sop, string opstr, SDNode opnode> : CSKY16Inst<AddrModeNone,
  29:   (outs mGPR:$rz), (ins mGPR:$rx, mGPR:$ry), !strconcat(opstr, "\t$rz, $rx, $ry"),
  30:   [(set mGPR:$rz, (opnode mGPR:$rx, mGPR:$ry)) ]> {
  31:   bits<3> rz;
  32:   bits<3> rx;
  33:   bits<3> ry;
  34:   let Inst{15 - 11} = 0b01011;
  35:   let Inst{10 - 8} = rx;
  36:   let Inst{7 - 5} = rz;
```

- EN: This range defines declarative TableGen records such as R16_XYZ, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R16_XYZ 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37:   let Inst{4 - 2} = ry;
  38:   let Inst{1, 0} = sop;
  39: }
  40: 
  41: class R16_XZ_BINOP<bits<4> op, bits<2> sop, string opstr, PatFrag opnode> : CSKY16Inst<
  42:   AddrModeNone, (outs sGPR:$rz), (ins sGPR:$rZ, sGPR:$rx), !strconcat(opstr, "\t$rz, $rx"),
  43:   [(set sGPR:$rz, (opnode sGPR:$rZ, sGPR:$rx))]> {
  44:   bits<4> rz;
  45:   bits<4> rx;
  46:   let Inst{15, 14} = 0b01;
  47:   let Inst{13 - 10} = op;
  48:   let Inst{9 - 6} = rz;
  49:   let Inst{5 - 2} = rx;
  50:   let Inst{1, 0} = sop;
  51:   let Constraints = "$rz = $rZ";
  52: }
  53: 
  54: class R16_XZ_BINOP_NOPat<bits<4> op, bits<2> sop, string opstr> : CSKY16Inst<
```

- EN: This range defines declarative TableGen records such as R16_XZ_BINOP, R16_XZ_BINOP_NOPat, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 R16_XZ_BINOP, R16_XZ_BINOP_NOPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 55-72

```tablegen
  55:   AddrModeNone, (outs sGPR:$rz), (ins sGPR:$rZ, sGPR:$rx), !strconcat(opstr, "\t$rz, $rx"),
  56:   []> {
  57:   bits<4> rz;
  58:   bits<4> rx;
  59:   let Inst{15, 14} = 0b01;
  60:   let Inst{13 - 10} = op;
  61:   let Inst{9 - 6} = rz;
  62:   let Inst{5 - 2} = rx;
  63:   let Inst{1, 0} = sop;
  64:   let Constraints = "$rz = $rZ";
  65: }
  66: 
  67: class R16_XZ_BINOP_C<bits<4> op, bits<2> sop, string opstr> : CSKY16Inst<
  68:   AddrModeNone, (outs sGPR:$rz, CARRY:$cout),
  69:   (ins sGPR:$rZ, sGPR:$rx, CARRY:$cin), !strconcat(opstr, "\t$rz, $rx"), []> {
  70:   bits<0> cout;
  71:   bits<0> cin;
  72:   bits<4> rz;
```

- EN: This range defines declarative TableGen records such as R16_XZ_BINOP_C, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R16_XZ_BINOP_C 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   bits<4> rx;
  74:   let Inst{15, 14} = 0b01;
  75:   let Inst{13 - 10} = op;
  76:   let Inst{9 - 6} = rz;
  77:   let Inst{5 - 2} = rx;
  78:   let Inst{1, 0} = sop;
  79:   let Constraints = "$rz = $rZ";
  80: }
  81: 
  82: class R16_XZ_UNOP<bits<4> op, bits<2> sop, string opstr> : CSKY16Inst<
  83:   AddrModeNone, (outs sGPR:$rz), (ins sGPR:$rx), !strconcat(opstr, "\t$rz, $rx"),
  84:   []> {
  85:   bits<4> rz;
  86:   bits<4> rx;
  87:   let Inst{15, 14} = 0b01;
  88:   let Inst{13 - 10} = op;
  89:   let Inst{9 - 6} = rz;
  90:   let Inst{5 - 2} = rx;
```

- EN: This range defines declarative TableGen records such as R16_XZ_UNOP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R16_XZ_UNOP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91:   let Inst{1, 0} = sop;
  92: }
  93: 
  94: class R16_Z_UNOP<bits<4> op, bits<2> sop, string opstr> : CSKY16Inst<
  95:   AddrModeNone, (outs sGPR:$rz), (ins sGPR:$rx), !strconcat(opstr, "\t$rz"),
  96:   []> {
  97:   bits<4> rz;
  98:   bits<4> rx;
  99:   let Inst{15, 14} = 0b01;
 100:   let Inst{13 - 10} = op;
 101:   let Inst{9 - 6} = rz;
 102:   let Inst{5 - 2} = rx;
 103:   let Inst{1, 0} = sop;
 104:   let Constraints = "$rz = $rx";
 105: }
 106: 
 107: class R16_XY_CMP<bits<2> sop, string opstr>
 108:     : CSKY16Inst<AddrModeNone, (outs CARRY:$ca), (ins sGPR:$rx, sGPR:$ry),
```

- EN: This range defines declarative TableGen records such as R16_Z_UNOP, R16_XY_CMP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R16_Z_UNOP, R16_XY_CMP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:                  !strconcat(opstr, "\t$rx, $ry"), []> {
 110:   bits<0> ca;
 111:   bits<4> ry;
 112:   bits<4> rx;
 113:   let Inst{15, 14} = 0b01;
 114:   let Inst{13 - 10} = 0b1001;
 115:   let Inst{9 - 6} = ry;
 116:   let Inst{5 - 2} = rx;
 117:   let Inst{1, 0} = sop;
 118:   let isCompare = 1;
 119: }
 120: 
 121: class R16_X_J<bits<8> op_rz, bits<2> sop, string opstr> : CSKY16Inst<
 122:   AddrModeNone, (outs), (ins sGPR:$rx), !strconcat(opstr, "\t$rx"), []> {
 123:   bits<4> rx;
 124:   let Inst{15, 14} = 0b01;
 125:   let Inst{13 - 6} = op_rz;
 126:   let Inst{5 - 2} = rx;
```

- EN: This range defines declarative TableGen records such as R16_X_J, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 R16_X_J 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127:   let Inst{1, 0} = sop;
 128: }
 129: 
 130: class I16_Z_8<bits<3> op, dag ins, string asmstr>
 131:   : CSKY16Inst<AddrModeNone, (outs mGPR:$rz), ins, asmstr, []> {
 132:   bits<3> rz;
 133:   bits<8> imm8;
 134:   let Inst{15, 14} = 0b00;
 135:   let Inst{13 - 11} = op;
 136:   let Inst{10 - 8} = rz;
 137:   let Inst{7 - 0} = imm8;
 138: }
 139: 
 140: class I16_Z_5<bits<3> sop, dag outs, dag ins,string opstr>
 141:   : CSKY16Inst<AddrModeNone, outs, ins,
 142:   !strconcat(opstr, "\t$rz, $imm5"), []> {
 143:   bits<3> rz;
 144:   bits<5> imm5;
```

- EN: This range defines declarative TableGen records such as I16_Z_8, I16_Z_5, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I16_Z_8, I16_Z_5 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:   let Inst{15, 14} = 0b00;
 146:   let Inst{13 - 11} = 0b111;
 147:   let Inst{10 - 8} = rz;
 148:   let Inst{7 - 5} = sop;
 149:   let Inst{4 - 0} = imm5;
 150: }
 151: 
 152: class I16_X_CMP<bits<3> sop, string opstr, Operand Immoperand>
 153:     : CSKY16Inst<AddrModeNone, (outs CARRY:$ca),
 154:                  (ins mGPR:$rx, Immoperand:$imm5),
 155:                  !strconcat(opstr, "\t$rx, $imm5"), []> {
 156:   bits<0> ca;
 157:   bits<3> rx;
 158:   bits<5> imm5;
 159:   let Inst{15, 14} = 0b00;
 160:   let Inst{13 - 11} = 0b111;
 161:   let Inst{10 - 8} = rx;
 162:   let Inst{7 - 5} = sop;
```

- EN: This range defines declarative TableGen records such as I16_X_CMP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I16_X_CMP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:   let Inst{4 - 0} = imm5;
 164:   let isCompare = 1;
 165: }
 166: 
 167: class I16_SP_IMM7<bits<3> sop, string opstr>
 168:     : CSKY16Inst<AddrModeNone, (outs GPRSP:$sp2),
 169:                  (ins GPRSP:$sp1, uimm7_2:$imm7),
 170:                  !strconcat(opstr, "\t$sp2, $sp1, $imm7"), []> {
 171:   bits<0> sp2;
 172:   bits<0> sp1;
 173:   bits<7> imm7;
 174:   let Inst{15, 14} = 0b00;
 175:   let Inst{13 - 10} = 0b0101;
 176:   let Inst{9, 8} = imm7{6,5};
 177:   let Inst{7 - 5} = sop;
 178:   let Inst{4 - 0} = imm7{4 - 0};
 179: }
 180: 
```

- EN: This range defines declarative TableGen records such as I16_SP_IMM7, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I16_SP_IMM7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181: class I16_XZ_IMM5<bits<3> sop, string opstr, SDNode opnode> : CSKY16Inst<
 182:   AddrModeNone, (outs mGPR:$rz), (ins mGPR:$rx, uimm5:$imm5),
 183:   !strconcat(opstr, "\t$rz, $rx, $imm5"), [(set mGPR:$rz, (opnode mGPR:$rx, uimm5:$imm5))]> {
 184:   bits<3> rx;
 185:   bits<3> rz;
 186:   bits<5> imm5;
 187:   let Inst{15, 14} = 0b01;
 188:   let Inst{13 - 11} = sop;
 189:   let Inst{10 - 8} = rx;
 190:   let Inst{7 - 5} = rz;
 191:   let Inst{4 - 0} = imm5;
 192: }
 193: 
 194: class I16_XZ_LDST<AddrMode am, bits<3> sop, string opstr, dag outs, dag ins>
 195:   : CSKY16Inst<am, outs, ins, !strconcat(opstr, "\t$rz, ($rx, ${imm})"),
 196:   []> {
 197:   bits<3> rx;
 198:   bits<3> rz;
```

- EN: This range defines declarative TableGen records such as I16_XZ_IMM5, I16_XZ_LDST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I16_XZ_IMM5, I16_XZ_LDST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199:   bits<5> imm;
 200:   let Inst{15, 14} = 0b10;
 201:   let Inst{13 - 11} = sop;
 202:   let Inst{10 - 8} = rx;
 203:   let Inst{7 - 5} = rz;
 204:   let Inst{4 - 0} = imm;
 205: }
 206: 
 207: class I16_ZSP_LDST<AddrMode am, bits<3> sop, string opstr, dag outs, dag ins> : CSKY16Inst<
 208:   am,  outs, ins, !strconcat(opstr, "\t$rz, ($sp, ${addr})"),
 209:   []> {
 210:   bits<3> rz;
 211:   bits<8> addr;
 212:   let Inst{15, 14} = 0b10;
 213:   let Inst{13 - 11} = sop;
 214:   let Inst{10 - 8} = addr{7 - 5};
 215:   let Inst{7 - 5} = rz;
 216:   let Inst{4 - 0} = addr{4 - 0};
```

- EN: This range defines declarative TableGen records such as I16_ZSP_LDST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I16_ZSP_LDST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: }
 218: 
 219: class I16_XZ_IMM3<bits<2> sop, string opstr, SDNode opnode> : CSKY16Inst<
 220:   AddrModeNone, (outs mGPR:$rz), (ins mGPR:$rx, oimm3:$oimm3),
 221:   !strconcat(opstr, "\t$rz, $rx, $oimm3"), [(set mGPR:$rz, (opnode mGPR:$rx, oimm3:$oimm3))]> {
 222:   bits<3> rx;
 223:   bits<3> rz;
 224:   bits<3> oimm3;
 225:   let Inst{15, 14} = 0b01;
 226:   let Inst{13 - 11} = 0b011;
 227:   let Inst{10 - 8} = rx;
 228:   let Inst{7 - 5} = rz;
 229:   let Inst{4 - 2} = oimm3;
 230:   let Inst{1, 0} = sop;
 231: }
 232: 
 233: class I16_BPushPop<bits<11> op, bits<2> uop, dag out, dag ins, string opstr> :
 234:   CSKY16Inst<AddrModeNone, out, ins, opstr, []>{
```

- EN: This range defines declarative TableGen records such as I16_XZ_IMM3, I16_BPushPop, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 I16_XZ_IMM3, I16_BPushPop 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-241

```tablegen
 235:   bits<3> rz;
 236:   let Inst{15- 5} = op;
 237:   let Inst{4 -2} = rz;
 238:   let Inst{1,0} = uop;
 239:   let Predicates = [HasJAVA];
 240:   let hasSideEffects = 1;
 241: }
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
