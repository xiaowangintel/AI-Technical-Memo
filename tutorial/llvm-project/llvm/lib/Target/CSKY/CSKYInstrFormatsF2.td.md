# CSKYInstrFormatsF2.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrFormatsF2.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- CSKYInstrFormatsF2.td - CSKY Float2.0 Instr Format --*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // CSKY Instruction Format Float2.0 Definitions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class CSKYInstF2<AddrMode am, dag outs, dag ins, string opcodestr,
  14:                  list<dag> pattern>
  15:     : CSKY32Inst<am, 0x3d, outs, ins, opcodestr, pattern> {
  16:   let Predicates = [HasFPUv3_SF];
  17:   let DecoderNamespace = "FPUV3";
  18: }
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as CSKYInstF2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 CSKYInstF2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19: 
  20: class F2_XYZ<bits<5> datatype, bits<6> sop, string opcodestr, dag outs, dag ins,
  21:              list<dag> pattern>
  22:     : CSKYInstF2<AddrModeNone, outs, ins, opcodestr, pattern> {
  23:   bits<5> vry;
  24:   bits<5> vrx;
  25:   bits<5> vrz;
  26: 
  27:   let Inst{25-21} = vry;
  28:   let Inst{20-16} = vrx;
  29:   let Inst{15-11} = datatype;
  30:   let Inst{10-5} = sop;
  31:   let Inst{4-0} = vrz;
  32: }
  33: 
  34: multiclass F2_XYZ_T<bits<6> sop, string op, PatFrag opnode> {
  35:   def _S : F2_XYZ<0b00000, sop, op#".32"#"\t$vrz, $vrx, $vry",
  36:              (outs FPR32Op:$vrz), (ins FPR32Op:$vrx, FPR32Op:$vry),
```

- EN: This range defines declarative TableGen records such as F2_XYZ, F2_XYZ_T, _S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_XYZ, F2_XYZ_T, _S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37:              [(set FPR32Op:$vrz, (opnode FPR32Op:$vrx, FPR32Op:$vry))]>;
  38:   let Predicates = [HasFPUv3_DF] in
  39:   def _D : F2_XYZ<0b00001, sop, op#".64"#"\t$vrz, $vrx, $vry",
  40:              (outs FPR64Op:$vrz), (ins FPR64Op:$vrx, FPR64Op:$vry),
  41:              [(set FPR64Op:$vrz, (opnode FPR64Op:$vrx, FPR64Op:$vry))]>;
  42: }
  43: 
  44: let Constraints = "$vrZ = $vrz" in
  45: multiclass F2_XYZZ_T<bits<6> sop, string op, PatFrag opnode> {
  46:   def _S : F2_XYZ<0b00000, sop, op#".32"#"\t$vrz, $vrx, $vry",
  47:                   (outs FPR32Op:$vrz), (ins FPR32Op:$vrZ, FPR32Op:$vrx, FPR32Op:$vry),
  48:                   [(set FPR32Op:$vrz, (opnode FPR32Op:$vrx, FPR32Op:$vry, FPR32Op:$vrZ))]>;
  49:   let Predicates = [HasFPUv3_DF] in
  50:   def _D : F2_XYZ<0b00001, sop, op#".64"#"\t$vrz, $vrx, $vry",
  51:                   (outs FPR64Op:$vrz), (ins FPR64Op:$vrZ, FPR64Op:$vrx, FPR64Op:$vry),
  52:                   [(set FPR64Op:$vrz, (opnode FPR64Op:$vrx, FPR64Op:$vry, FPR64Op:$vrZ))]>;
  53: }
  54: 
```

- EN: This range defines declarative TableGen records such as _D, F2_XYZZ_T, _S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 _D, F2_XYZZ_T, _S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55: let vry = 0 in {
  56: class F2_XZ<bits<5> datatype, RegisterOperand regtype, bits<6> sop, string op, SDNode opnode>
  57:     : F2_XYZ<datatype, sop, !strconcat(op, "\t$vrz, $vrx"),
  58:              (outs regtype:$vrz), (ins regtype:$vrx),
  59:              [(set regtype:$vrz, (opnode regtype:$vrx))]>;
  60: 
  61: class F2_XZ_SET<bits<5> datatype, RegisterOperand regtype, bits<6> sop, string op>
  62:     : F2_XYZ<datatype, sop, !strconcat(op, "\t$vrz, $vrx"),
  63:              (outs regtype:$vrz), (ins regtype:$vrx),
  64:              []>;
  65: 
  66: class F2_XZ_P<bits<5> datatype, bits<6> sop, string op, list<dag> pattern = [],
  67:               dag outs, dag ins>
  68:     : F2_XYZ<datatype, sop, op#"\t$vrz, $vrx", outs, ins, pattern>;
  69: }
  70: 
  71: multiclass F2_XZ_RM<bits<5> datatype, bits<4> sop, string op, dag outs, dag ins> {
  72:   def _RN  : F2_XZ_P<datatype, {sop, 0b00}, op#".rn", [], outs, ins>;
```

- EN: This range defines declarative TableGen records such as F2_XZ, F2_XZ_SET, F2_XZ_P, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_XZ, F2_XZ_SET, F2_XZ_P 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   def _RZ  : F2_XZ_P<datatype, {sop, 0b01}, op#".rz", [], outs, ins>;
  74:   def _RPI : F2_XZ_P<datatype, {sop, 0b10}, op#".rpi", [], outs, ins>;
  75:   def _RNI : F2_XZ_P<datatype, {sop, 0b11}, op#".rni", [], outs, ins>;
  76: }
  77: 
  78: multiclass F2_XZ_T<bits<6> sop, string op, SDNode opnode> {
  79:   def _S : F2_XZ<0b00000, FPR32Op, sop, op#".32", opnode>;
  80:   let Predicates = [HasFPUv3_DF] in
  81:   def _D : F2_XZ<0b00001, FPR64Op, sop, op#".64", opnode>;
  82: }
  83: 
  84: multiclass F2_XZ_SET_T<bits<6> sop, string op, string suffix = ""> {
  85:   def _S : F2_XZ_SET<0b00000, FPR32Op, sop, op#".32"#suffix>;
  86:   let Predicates = [HasFPUv3_DF] in
  87:   def _D : F2_XZ_SET<0b00001, FPR64Op, sop, op#".64"#suffix>;
  88: }
  89: 
  90: 
```

- EN: This range defines declarative TableGen records such as _RZ, _RPI, _RNI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 _RZ, _RPI, _RNI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91: let vrz = 0, isCompare = 1 in
  92: class F2_CXY<bits<5> datatype, RegisterOperand regtype, bits<6> sop, string op>
  93:     : F2_XYZ<datatype, sop, !strconcat(op, "\t$vrx, $vry"),
  94:              (outs CARRY:$ca), (ins regtype:$vrx, regtype:$vry), []> {
  95:   bits<0> ca;
  96: }
  97: 
  98: multiclass F2_CXY_T<bits<6> sop, string op> {
  99:   def _S : F2_CXY<0b00000, FPR32Op, sop, op#".32">;
 100:   let Predicates = [HasFPUv3_DF] in
 101:   def _D : F2_CXY<0b00001, FPR64Op, sop, op#".64">;
 102: }
 103: 
 104: 
 105: let vrz = 0, vry = 0, isCompare = 1 in
 106: class F2_CX<bits<5> datatype, RegisterOperand regtype, bits<6> sop, string op>
 107:     : F2_XYZ<datatype, sop, !strconcat(op, "\t$vrx"), (outs CARRY:$ca),
 108:              (ins regtype:$vrx), []> {
```

- EN: This range defines declarative TableGen records such as F2_CXY, F2_CXY_T, _S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_CXY, F2_CXY_T, _S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109:   bits<0> ca;
 110: }
 111: 
 112: multiclass F2_CX_T<bits<6> sop, string op> {
 113:   def _S : F2_CX<0b00000, FPR32Op, sop, op#".32">;
 114:   let Predicates = [HasFPUv3_DF] in
 115:   def _D : F2_CX<0b00001, FPR64Op, sop, op#".64">;
 116: }
 117: 
 118: 
 119: class F2_LDST<bits<2> datatype, bits<1> sop, string op, dag outs, dag ins>
 120:     : CSKYInstF2<AddrMode32SDF, outs, ins,
 121:                  !strconcat(op, "\t$vrz, ($rx, ${imm8})"), []> {
 122:   bits<10> imm8;
 123:   bits<5> rx;
 124:   bits<5> vrz;
 125: 
 126:   let Inst{25} = vrz{4};
```

- EN: This range defines declarative TableGen records such as F2_CX_T, _S, _D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_CX_T, _S, _D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127:   let Inst{24-21} = imm8{7-4};
 128:   let Inst{20-16} = rx;
 129:   let Inst{15-11} = 0b00100;
 130:   let Inst{10} = sop;
 131:   let Inst{9-8} = datatype;
 132:   let Inst{7-4} = imm8{3-0};
 133:   let Inst{3-0} = vrz{3-0};
 134: }
 135: 
 136: class F2_LDST_S<bits<1> sop, string op, dag outs, dag ins>
 137:     : F2_LDST<0b00, sop, op#".32", outs, ins>;
 138: class F2_LDST_D<bits<1> sop, string op, dag outs, dag ins>
 139:     : F2_LDST<0b01, sop, op#".64", outs, ins>;
 140: 
 141: class F2_LDSTM<bits<2> datatype, bits<1> sop, bits<3> sop2, string op, dag outs, dag ins>
 142:     : CSKYInstF2<AddrMode32SDF, outs, ins,
 143:                  !strconcat(op, "\t$regs, (${rx})"), []> {
 144:   bits<10> regs;
```

- EN: This range defines declarative TableGen records such as F2_LDST_S, F2_LDST_D, F2_LDSTM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_LDST_S, F2_LDST_D, F2_LDSTM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:   bits<5> rx;
 146: 
 147:   let Inst{25-21} = regs{4-0};
 148:   let Inst{20-16} = rx;
 149:   let Inst{15-11} = 0b00110;
 150:   let Inst{10} = sop;
 151:   let Inst{9-8} = datatype;
 152:   let Inst{7-5} = sop2;
 153:   let Inst{4-0} = regs{9-5};
 154: }
 155: 
 156: class F2_LDSTM_S<bits<1> sop, bits<3> sop2, string op, dag outs, dag ins>
 157:     : F2_LDSTM<0b00, sop, sop2, op#".32", outs, ins>;
 158: class F2_LDSTM_D<bits<1> sop, bits<3> sop2, string op, dag outs, dag ins>
 159:     : F2_LDSTM<0b01, sop, sop2, op#".64", outs, ins>;
 160: 
 161: 
 162: class F2_LDSTR<bits<2> datatype, bits<1> sop, string op, dag outs, dag ins>
```

- EN: This range defines declarative TableGen records such as F2_LDSTM_S, F2_LDSTM_D, F2_LDSTR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_LDSTM_S, F2_LDSTM_D, F2_LDSTR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:     : CSKYInstF2<AddrModeNone, outs, ins,
 164:                  op#"\t$rz, ($rx, $ry << ${imm})", []> {
 165:   bits<5> rx;
 166:   bits<5> ry;
 167:   bits<5> rz;
 168:   bits<2> imm;
 169: 
 170:   let Inst{25-21} = ry;
 171:   let Inst{20-16} = rx;
 172:   let Inst{15-11} = 0b00101;
 173:   let Inst{10} = sop;
 174:   let Inst{9-8} = datatype;
 175:   let Inst{7} = 0;
 176:   let Inst{6-5} = imm;
 177:   let Inst{4-0} = rz;
 178: }
 179: 
 180: class F2_LDSTR_S<bits<1> sop, string op, dag outs, dag ins>
```

- EN: This range defines declarative TableGen records such as F2_LDSTR_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_LDSTR_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181:     : F2_LDSTR<0b00, sop, op#".32", outs, ins>;
 182: class F2_LDSTR_D<bits<1> sop, string op, dag outs, dag ins>
 183:     : F2_LDSTR<0b01, sop, op#".64", outs, ins>;
 184: 
 185: class F2_CXYZ<bits<5> datatype, RegisterOperand regtype, bits<6> sop, string op>
 186:     : F2_XYZ<datatype, sop, !strconcat(op, "\t$vrz, $vrx, $vry"),
 187:              (outs regtype:$vrz), (ins CARRY:$ca, regtype:$vrx, regtype:$vry),
 188:              []> {
 189:   bits<0> ca;
 190: }
 191: 
 192: multiclass F2_CXYZ_T<bits<6> sop, string op> {
 193:   def _S : F2_CXYZ<0b00000, FPR32Op, sop, op#".32">;
 194:   let Predicates = [HasFPUv3_DF] in
 195:   def _D : F2_CXYZ<0b00001, FPR64Op, sop, op#".64">;
 196: }
 197: 
 198: class F2_LRW<bits<2> datatype, bits<1> sop, string op, dag outs, dag ins>
```

- EN: This range defines declarative TableGen records such as F2_LDSTR_D, F2_CXYZ, F2_CXYZ_T, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F2_LDSTR_D, F2_CXYZ, F2_CXYZ_T 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-213

```tablegen
 199:     : CSKYInstF2<AddrModeNone, outs, ins,
 200:                  !strconcat(op, "\t$vrz, ${imm8}"), []> {
 201:   bits<10> imm8;
 202:   bits<5> rx;
 203:   bits<5> vrz;
 204: 
 205:   let Inst{25} = vrz{4};
 206:   let Inst{24-21} = imm8{7-4};
 207:   let Inst{20-16} = 0;
 208:   let Inst{15-11} = 0b00111;
 209:   let Inst{10} = sop;
 210:   let Inst{9-8} = datatype;
 211:   let Inst{7-4} = imm8{3-0};
 212:   let Inst{3-0} = vrz{3-0};
 213: }
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
