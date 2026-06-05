# CSKYInstrFormatsF1.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrFormatsF1.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines reusable TableGen instruction format classes and encoding fields.
- 目的（中文）: 定义可复用的 TableGen 指令格式类与编码字段。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- CSKYInstrFormatsF1.td - CSKY Float1.0 Instr Format --*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // CSKY Instruction Format Float1.0 Definitions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: class CSKYFP1Inst<dag outs, dag ins, string asmstr, list<dag> pattern>
  14:   : CSKY32Inst<AddrModeNone, 0x3d, outs, ins, asmstr, pattern>, Requires<[HasFPUv2_SF]> {
  15: }
  16: 
  17: class F_XYZ_BASE<bits<5> datatype, bits<6> sop, dag outs, dag ins, string opcodestr, list<dag> pattern>
  18:   : CSKYFP1Inst<outs, ins, opcodestr, pattern> {
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as CSKYFP1Inst, F_XYZ_BASE, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 CSKYFP1Inst, F_XYZ_BASE 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:   bits<4> vrx;
  20:   bits<4> vry;
  21:   bits<4> vrz;
  22:   let Inst{25 - 21} = {0, vry};
  23:   let Inst{20 - 16} = {0, vrx};
  24:   let Inst{15 - 11} = datatype;
  25:   let Inst{10 - 5} = sop;
  26:   let Inst{4 - 0} = {0, vrz};
  27: }
  28: 
  29: class F_XZ_GF<bits<5> datatype, bits<6> sop, dag outs, dag ins, string opcodestr, list<dag> pattern>
  30:   : CSKYFP1Inst<outs, ins, opcodestr, pattern> {
  31:   bits<4> vrx;
  32:   bits<5> rz;
  33:   let Inst{25 - 21} = 0;
  34:   let Inst{20 - 16} = {0, vrx};
  35:   let Inst{15 - 11} = datatype;
  36:   let Inst{10 - 5} = sop;
```

- EN: This range defines declarative TableGen records such as F_XZ_GF, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XZ_GF 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37:   let Inst{4 - 0} = {rz};
  38: }
  39: 
  40: class F_XZ_FG<bits<5> datatype, bits<6> sop, dag outs, dag ins, string opcodestr, list<dag> pattern>
  41:   : CSKYFP1Inst<outs, ins, opcodestr, pattern> {
  42:   bits<5> rx;
  43:   bits<4> vrz;
  44:   let Inst{25 - 21} = 0;
  45:   let Inst{20 - 16} = {rx};
  46:   let Inst{15 - 11} = datatype;
  47:   let Inst{10 - 5} = sop;
  48:   let Inst{4 - 0} = {0, vrz};
  49: }
  50: 
  51: class F_XZ_TRANS_FROM<bits<6> sop, string op, RegisterOperand regtype1, RegisterOperand regtype2>
  52:   : F_XZ_GF<3, sop, (outs regtype1:$rz), (ins regtype2:$vrx), !strconcat(op, "\t$rz, $vrx"),
  53:   []>;
  54: 
```

- EN: This range defines declarative TableGen records such as F_XZ_FG, F_XZ_TRANS_FROM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XZ_FG, F_XZ_TRANS_FROM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-72

```tablegen
  55: class F_XZ_TRANS_TO<bits<6> sop, string op, RegisterOperand regtype1, RegisterOperand regtype2>
  56:   : F_XZ_FG<3, sop, (outs regtype1:$vrz), (ins regtype2:$rx), !strconcat(op, "\t$vrz, $rx"),
  57:   []>;
  58: 
  59: let vry = 0 in {
  60: class F_XZ<bits<5> datatype, bits<6> sop, string op, string op_su, PatFrag opnode, RegisterOperand regtype>
  61:   : F_XYZ_BASE<datatype, sop, (outs regtype:$vrz), (ins regtype:$vrx), !strconcat(op#op_su, "\t$vrz, $vrx"),
  62:   [(set regtype:$vrz, (opnode regtype:$vrx))]>;
  63: 
  64: class F_MOV<bits<5> datatype, bits<6> sop, string op, string op_su, RegisterOperand regtype>
  65:   : F_XYZ_BASE<datatype, sop, (outs regtype:$vrz), (ins regtype:$vrx), !strconcat(op#op_su, "\t$vrz, $vrx"),
  66:   []>;
  67: 
  68: class F_XZ_TRANS<bits<6> sop, string op, RegisterOperand regtype1, RegisterOperand regtype2>
  69:   : F_XYZ_BASE<3, sop, (outs regtype1:$vrz), (ins regtype2:$vrx), !strconcat(op, "\t$vrz, $vrx"),
  70:   []>;
  71: 
  72: class F_XZ_TRANS_DS<bits<6> sop, string op, PatFrag opnode>
```

- EN: This range defines declarative TableGen records such as F_XZ_TRANS_TO, F_XZ, F_MOV, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XZ_TRANS_TO, F_XZ, F_MOV 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   : F_XYZ_BASE<3, sop, (outs sFPR32Op:$vrz), (ins sFPR64Op:$vrx), !strconcat(op, "\t$vrz, $vrx"),
  74:   [(set sFPR32Op:$vrz, (opnode sFPR64Op:$vrx))]>;
  75: 
  76: class F_XZ_TRANS_SD<bits<6> sop, string op, PatFrag opnode>
  77:   : F_XYZ_BASE<3, sop, (outs sFPR64Op:$vrz), (ins sFPR32Op:$vrx), !strconcat(op, "\t$vrz, $vrx"),
  78:   [(set sFPR64Op:$vrz, (opnode sFPR32Op:$vrx))]>;
  79: }
  80: 
  81: multiclass FT_MOV<bits<6> sop, string op> {
  82:   def _S :  F_MOV<0, sop, op, "s", sFPR32Op>;
  83:   let Predicates = [HasFPUv2_DF] in
  84:   def _D :  F_MOV<1, sop, op, "d", sFPR64Op>;
  85: }
  86: 
  87: multiclass FT_XZ<bits<6> sop, string op, PatFrag opnode> {
  88:   def _S :  F_XZ<0, sop, op, "s", opnode, sFPR32Op>;
  89:   let Predicates = [HasFPUv2_DF] in
  90:   def _D :  F_XZ<1, sop, op, "d", opnode, sFPR64Op>;
```

- EN: This range defines declarative TableGen records such as F_XZ_TRANS_SD, FT_MOV, _S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XZ_TRANS_SD, FT_MOV, _S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91: }
  92: 
  93: let vrz = 0, isCompare = 1 in {
  94:   class F_CMPXY<bits<5> datatype, bits<6> sop, string op, string op_su,
  95:                 RegisterOperand regtype>
  96:       : F_XYZ_BASE<datatype, sop, (outs CARRY:$ca),
  97:                    (ins regtype:$vrx, regtype:$vry),
  98:                    !strconcat(op#op_su, "\t$vrx, $vry"), []> {
  99:     bits<0> ca;
 100:   }
 101: 
 102:   let vry = 0 in
 103:   class F_CMPZX<bits<5> datatype, bits<6> sop, string op, string op_su,
 104:                 RegisterOperand regtype>
 105:       : F_XYZ_BASE<datatype, sop, (outs CARRY:$ca), (ins regtype:$vrx),
 106:                    !strconcat(op#op_su, "\t$vrx"), []> {
 107:     bits<0> ca;
 108:   }
```

- EN: This range defines declarative TableGen records such as F_CMPXY, F_CMPZX, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_CMPXY, F_CMPZX 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109: }
 110: 
 111: class F_XYZ<bits<5> datatype, bits<6> sop, string op, string op_su, PatFrag opnode, RegisterOperand regtype>
 112:   : F_XYZ_BASE<datatype, sop, (outs regtype:$vrz), (ins regtype:$vrx, regtype:$vry),
 113:     !strconcat(op#op_su, "\t$vrz, $vrx, $vry"),
 114:   [(set regtype:$vrz, (opnode regtype:$vrx, regtype:$vry))]>;
 115: 
 116: multiclass FT_XYZ<bits<6> sop, string op, PatFrag opnode> {
 117:   def _S :  F_XYZ<0, sop, op, "s", opnode, sFPR32Op>;
 118:   let Predicates = [HasFPUv2_DF] in
 119:   def _D :  F_XYZ<1, sop, op, "d", opnode, sFPR64Op>;
 120: }
 121: 
 122: let Constraints = "$vrt = $vrz" in {
 123: class F_ACCUM_XYZ<bits<5> datatype, bits<6> sop, string op, string op_su, PatFrag opnode, RegisterOperand regtype>
 124:   : F_XYZ_BASE<datatype, sop, (outs regtype:$vrz), (ins regtype:$vrt, regtype:$vrx, regtype:$vry),
 125:     !strconcat(op#op_su, "\t$vrz, $vrx, $vry"),
 126:   [(set regtype:$vrz, (opnode regtype:$vrt, regtype:$vrx, regtype:$vry))]>;
```

- EN: This range defines declarative TableGen records such as F_XYZ, FT_XYZ, _S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XYZ, FT_XYZ, _S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127: }
 128: 
 129: multiclass FT_ACCUM_XYZ<bits<6> sop, string op, PatFrag opnode> {
 130:   def _S :  F_ACCUM_XYZ<0, sop, op, "s", opnode, sFPR32Op>;
 131:   let Predicates = [HasFPUv2_DF] in
 132:   def _D :  F_ACCUM_XYZ<1, sop, op, "d", opnode, sFPR64Op>;
 133: }
 134: 
 135: multiclass FT_CMPXY<bits<6> sop, string op> {
 136:   def _S :  F_CMPXY<0, sop, op, "s", sFPR32Op>;
 137:   let Predicates = [HasFPUv2_DF] in
 138:   def _D :  F_CMPXY<1, sop, op, "d", sFPR64Op>;
 139: }
 140: 
 141: 
 142: multiclass FT_CMPZX<bits<6> sop, string op> {
 143:   def _S :  F_CMPZX<0, sop, op, "s", sFPR32Op>;
 144:   let Predicates = [HasFPUv2_DF] in
```

- EN: This range defines declarative TableGen records such as FT_ACCUM_XYZ, _S, _D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FT_ACCUM_XYZ, _S, _D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145:   def _D :  F_CMPZX<1, sop, op, "d", sFPR64Op>;
 146: }
 147: 
 148: class F_I8_XY_MEM<bits<7> sop, bits<1> sop_su, dag outs, dag ins, string opcodestr, list<dag> pattern>
 149:   : CSKY32Inst<AddrMode32SDF, 0x3d, outs, ins, opcodestr, pattern> {
 150:   bits<5> rx;
 151:   bits<4> vrz;
 152:   bits<8> imm8;
 153:   let Inst{25} = 0;
 154:   let Inst{24 - 21} = imm8{7 - 4};  //imm4h
 155:   let Inst{20 - 16} = rx;  //rx
 156:   let Inst{15 - 9} = sop;
 157:   let Inst{8} = sop_su;
 158:   let Inst{7 - 4} = imm8{3 - 0}; // imm4l
 159:   let Inst{3 - 0} = vrz;
 160: }
 161: 
 162: class F_I4_XY_MEM<bits<7> sop, bits<1> sop_su, dag outs, dag ins, string opcodestr, list<dag> pattern>
```

- EN: This range defines declarative TableGen records such as _D, F_I8_XY_MEM, F_I4_XY_MEM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 _D, F_I8_XY_MEM, F_I4_XY_MEM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:   : CSKY32Inst<AddrMode32SDF, 0x3d, outs, ins, opcodestr, pattern> {
 164:   bits<10> regs;
 165:   bits<5> rx;
 166: 
 167:   let Inst{25} = 0;
 168:   let Inst{24 - 21} = regs{3-0};  //imm4
 169:   let Inst{20 - 16} = rx;  //rx
 170:   let Inst{15 - 9} = sop;
 171:   let Inst{8} = sop_su;
 172:   let Inst{7 - 4} = 0;
 173:   let Inst{3 - 0} = regs{8-5};
 174: }
 175: 
 176: class F_I8_Z_MEM<bits<7> sop, bits<1> sop_su, dag outs, dag ins, string opcodestr, list<dag> pattern>
 177:   : CSKY32Inst<AddrModeNone, 0x3d, outs, ins, opcodestr, pattern> {
 178:   bits<4> vrz;
 179:   bits<8> imm8;
 180:   let Inst{25} = 0;
```

- EN: This range defines declarative TableGen records such as F_I8_Z_MEM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_I8_Z_MEM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 181-198

```tablegen
 181:   let Inst{24 - 21} = imm8{7 - 4};  //imm4h
 182:   let Inst{20 - 16} = 0;  //rx
 183:   let Inst{15 - 9} = sop;
 184:   let Inst{8} = sop_su;
 185:   let Inst{7 - 4} = imm8{3 - 0}; // imm4l
 186:   let Inst{3 - 0} = vrz;
 187: }
 188: 
 189: class F_XYZ_MEM<bits<7> sop, bits<1> sop_su, dag outs, dag ins, string opcodestr, list<dag> pattern>
 190:   : CSKY32Inst<AddrModeNone, 0x3d, outs, ins, opcodestr, pattern> {
 191:   bits<5> rx;
 192:   bits<5> ry;
 193:   bits<4> vrz;
 194:   bits<2> imm;
 195: 
 196:   let Inst{25 - 21} = ry;  // ry;
 197:   let Inst{20 - 16} = rx;  // rx;
 198:   let Inst{15 - 9} = sop;
```

- EN: This range defines declarative TableGen records such as F_XYZ_MEM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XYZ_MEM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199:   let Inst{8} = sop_su;
 200:   let Inst{7} = 0;
 201:   let Inst{6,5} = imm;  // shift;
 202:   let Inst{4} = 0;
 203:   let Inst{3 - 0} = vrz;
 204: }
 205: 
 206: class F_XYAI_LD<bits<7> sop, bits<1> sop_su, string op, string op_su,
 207:                  RegisterOperand regtype, Operand operand>
 208:   : F_I8_XY_MEM<sop, sop_su, (outs regtype:$vrz), (ins GPR:$rx, operand:$imm8),
 209:     !strconcat(op#op_su, "\t$vrz, ($rx, ${imm8})"), []>;
 210: 
 211: class F_XYAR_LD<bits<7> sop, bits<1> sop_su, string op, string op_su,
 212:                  RegisterOperand regtype>
 213:   : F_XYZ_MEM<sop, sop_su, (outs regtype:$vrz), (ins GPR:$rx, GPR:$ry, uimm2:$imm),
 214:     op#op_su#"\t$vrz, ($rx, $ry << ${imm})", []>;
 215: 
 216: class F_XYAI_ST<bits<7> sop, bits<1> sop_su, string op, string op_su,
```

- EN: This range defines declarative TableGen records such as F_XYAI_LD, F_XYAR_LD, F_XYAI_ST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 F_XYAI_LD, F_XYAR_LD, F_XYAI_ST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217:                  RegisterOperand regtype, Operand operand>
 218:   : F_I8_XY_MEM<sop, sop_su, (outs), (ins regtype:$vrz, GPR:$rx, operand:$imm8),
 219:     !strconcat(op#op_su, "\t$vrz, ($rx, ${imm8})"), []>;
 220: 
 221: class F_XYAR_ST<bits<7> sop, bits<1> sop_su, string op, string op_su,
 222:                  RegisterOperand regtype>
 223:   : F_XYZ_MEM<sop, sop_su, (outs), (ins regtype:$vrz, GPR:$rx, GPR:$ry, uimm2:$imm),
 224:     op#op_su#"\t$vrz, ($rx, $ry << ${imm})", []>;
 225: 
 226: def Mem8SL2 : Operand<iPTR>, ComplexPattern<iPTR, 2, "SelectAddrRegImm8", []> {
 227:   let MIOperandInfo = (ops GPR, i32imm);
 228:   let PrintMethod = "printAddrModeRegImmOperand";
 229:   let EncoderMethod = "getAddrModeFloatImm8_sl2OpValue";
 230: }
 231: 
 232: def FRRS : Operand<iPTR>, ComplexPattern<iPTR, 3, "SelectAddrRegReg", []> {
 233:   let MIOperandInfo = (ops GPR, GPR, i32imm);
 234:   let PrintMethod = "printAddrModeRegRegSLOperand";
```

- EN: This range defines declarative TableGen records such as F_XYAR_ST, Mem8SL2, FRRS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 F_XYAR_ST, Mem8SL2, FRRS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 235-252

```tablegen
 235:   let EncoderMethod = "getAddrModeFloatRegRegSLOpValue";
 236: }
 237: 
 238: multiclass FT_XYAI_LD<bits<7> sop, string op> {
 239:   def _S :  F_XYAI_LD<sop, 0, op, "s", sFPR32Op, uimm8_2>;
 240:   let Predicates = [HasFPUv2_DF] in
 241:   def _D :  F_XYAI_LD<sop, 1, op, "d", sFPR64Op, uimm8_2>;
 242: }
 243: 
 244: multiclass FT_XYAR_LD<bits<7> sop, string op> {
 245:   def _S :  F_XYAR_LD<sop, 0, op, "s", sFPR32Op>;
 246:   let Predicates = [HasFPUv2_DF] in
 247:   def _D :  F_XYAR_LD<sop, 1, op, "d", sFPR64Op>;
 248: }
 249: 
 250: multiclass FT_XYAI_ST<bits<7> sop, string op> {
 251:   def _S :  F_XYAI_ST<sop, 0, op, "s", sFPR32Op, uimm8_2>;
 252:   let Predicates = [HasFPUv2_DF] in
```

- EN: This range defines declarative TableGen records such as FT_XYAI_LD, _S, _D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FT_XYAI_LD, _S, _D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253:   def _D :  F_XYAI_ST<sop, 1, op, "d", sFPR64Op, uimm8_2>;
 254: }
 255: 
 256: multiclass FT_XYAR_ST<bits<7> sop, string op> {
 257:   def _S :  F_XYAR_ST<sop, 0, op, "s", sFPR32Op>;
 258:   let Predicates = [HasFPUv2_DF] in
 259:   def _D :  F_XYAR_ST<sop, 1, op, "d", sFPR64Op>;
 260: }
 261: 
 262: multiclass FT_XYAR_STM<bits<7> sop, string op> {
 263:   def _S :  F_I4_XY_MEM<sop, 0, (outs),
 264:     (ins GPR:$rx, regseq_f1:$regs, variable_ops),
 265:       !strconcat(op#"s", "\t$regs, (${rx})"), []>;
 266:   let Predicates = [HasFPUv2_DF] in
 267:   def _D :  F_I4_XY_MEM<sop, 1, (outs),
 268:     (ins GPR:$rx, regseq_d1:$regs, variable_ops),
 269:       !strconcat(op#"d", "\t$regs, (${rx})"), []>;
 270: }
```

- EN: This range defines declarative TableGen records such as _D, FT_XYAR_ST, _S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 _D, FT_XYAR_ST, _S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-280

```tablegen
 271: 
 272: multiclass FT_XYAR_LDM<bits<7> sop, string op> {
 273:   def _S :  F_I4_XY_MEM<sop, 0, (outs),
 274:     (ins GPR:$rx, regseq_f1:$regs, variable_ops),
 275:       !strconcat(op#"s", "\t$regs, (${rx})"), []>;
 276:   let Predicates = [HasFPUv2_DF] in
 277:   def _D :  F_I4_XY_MEM<sop, 1, (outs),
 278:     (ins GPR:$rx, regseq_d1:$regs, variable_ops),
 279:       !strconcat(op#"d", "\t$regs, (${rx})"), []>;
 280: }
```

- EN: This range defines declarative TableGen records such as FT_XYAR_LDM, _S, _D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FT_XYAR_LDM, _S, _D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Instruction format fields / 指令格式字段
- Reusable TableGen classes / 可复用 TableGen 类

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
