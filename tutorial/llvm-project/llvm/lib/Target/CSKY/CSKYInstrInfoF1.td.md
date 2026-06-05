# CSKYInstrInfoF1.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrInfoF1.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the CSKY instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- CSKYInstrInfoF1.td - CSKY Instruction Float1.0 ------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the CSKY instructions in TableGen format.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: def regseq_f1 : Operand<iPTR> {
  14:   let EncoderMethod = "getRegisterSeqOpValue";
  15:   let ParserMatchClass = RegSeqAsmOperand<"V1">;
  16:   let PrintMethod = "printRegisterSeq";
  17:   let DecoderMethod = "DecodeRegSeqOperandF1";
  18:   let MIOperandInfo = (ops sFPR32, uimm5);
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as regseq_f1, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 regseq_f1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19: }
  20: 
  21: def regseq_d1 : Operand<iPTR> {
  22:   let EncoderMethod = "getRegisterSeqOpValue";
  23:   let ParserMatchClass = RegSeqAsmOperand<"V1">;
  24:   let PrintMethod = "printRegisterSeq";
  25:   let DecoderMethod = "DecodeRegSeqOperandD1";
  26:   let MIOperandInfo = (ops sFPR64, uimm5);
  27: }
  28: 
  29: def sFPR32Op : RegisterOperand<sFPR32, "printFPR">;
  30: def sFPR64Op : RegisterOperand<sFPR64, "printFPR">;
  31: def sFPR64_V_OP : RegisterOperand<sFPR64_V, "printFPR">;
  32: 
  33: include "CSKYInstrFormatsF1.td"
  34: 
  35: //===----------------------------------------------------------------------===//
  36: // CSKY specific DAG Nodes.
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as regseq_d1, sFPR32Op, sFPR64Op, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 regseq_d1, sFPR32Op, sFPR64Op 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: //===----------------------------------------------------------------------===//
  38: 
  39: def SDT_BITCAST_TO_LOHI : SDTypeProfile<2, 1, [SDTCisSameAs<0, 1>]>;
  40: def CSKY_BITCAST_TO_LOHI : SDNode<"CSKYISD::BITCAST_TO_LOHI", SDT_BITCAST_TO_LOHI>;
  41: def SDT_BITCAST_FROM_LOHI : SDTypeProfile<1, 2, [SDTCisSameAs<1, 2>]>;
  42: def CSKY_BITCAST_FROM_LOHI : SDNode<"CSKYISD::BITCAST_FROM_LOHI", SDT_BITCAST_FROM_LOHI>;
  43: //===----------------------------------------------------------------------===//
  44: // Operand and SDNode transformation definitions.
  45: //===----------------------------------------------------------------------===//
  46: 
  47: def fpimm0 : PatLeaf<(fpimm), [{ return N->isExactlyValue(+0.0); }]>;
  48: 
  49: def fpimm32_hi16 : SDNodeXForm<fpimm, [{
  50:   return CurDAG->getTargetConstant(
  51:     (N->getValueAPF().bitcastToAPInt().getZExtValue() >> 16) & 0xFFFF,
  52:     SDLoc(N), MVT::i32);
  53: }]>;
  54: 
```

- EN: This range defines declarative TableGen records such as SDT_BITCAST_TO_LOHI, CSKY_BITCAST_TO_LOHI, SDT_BITCAST_FROM_LOHI, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SDT_BITCAST_TO_LOHI, CSKY_BITCAST_TO_LOHI, SDT_BITCAST_FROM_LOHI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: def fpimm32_lo16 : SDNodeXForm<fpimm, [{
  56:   return CurDAG->getTargetConstant(
  57:     N->getValueAPF().bitcastToAPInt().getZExtValue() & 0xFFFF,
  58:     SDLoc(N), MVT::i32);
  59: }]>;
  60: 
  61: class fpimm_xform<int width, int shift = 0> : SDNodeXForm<fpimm,
  62:   "return CurDAG->getTargetConstant(N->getValueAPF().bitcastToAPInt().lshr("#shift#").getLoBits("#width#"), SDLoc(N), MVT::i32);">;
  63: 
  64: class fpimm_xform_i16<int width, int shift = 0> : SDNodeXForm<fpimm,
  65:   "return CurDAG->getTargetConstant(N->getValueAPF().bitcastToAPInt().lshr("#shift#").getLoBits("#width#"), SDLoc(N), MVT::i16);">;
  66: 
  67: class fpimm_t<int width, int shift = 0> : PatLeaf<(fpimm),
  68:    "return isShiftedUInt<"#width#", "#shift#">(N->getValueAPF().bitcastToAPInt().getZExtValue());">;
  69: 
  70: def fpimm8 : fpimm_t<8>;
  71: def fpimm8_8 : fpimm_t<8, 8>;
  72: def fpimm8_16 : fpimm_t<8, 16>;
```

- EN: This range defines declarative TableGen records such as fpimm32_lo16, fpimm_xform, fpimm_xform_i16, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 fpimm32_lo16, fpimm_xform, fpimm_xform_i16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 73-90

```tablegen
  73: def fpimm8_24 : fpimm_t<8, 24>;
  74: def fpimm16 : fpimm_t<16>;
  75: def fpimm16_8 : fpimm_t<16, 8>;
  76: def fpimm16_16 : fpimm_t<16, 16>;
  77: def fpimm24 : fpimm_t<24>;
  78: def fpimm24_8 : fpimm_t<24, 8>;
  79: def fpimm32 : fpimm_t<32>;
  80: 
  81: def fpimm8_sr0_XFORM : fpimm_xform<8>;
  82: def fpimm8_sr8_XFORM : fpimm_xform<8, 8>;
  83: def fpimm8_sr16_XFORM : fpimm_xform<8, 16>;
  84: def fpimm8_sr24_XFORM : fpimm_xform<8, 24>;
  85: 
  86: def fpimm8_sr0_i16_XFORM : fpimm_xform_i16<8>;
  87: def fpimm8_sr8_i16_XFORM : fpimm_xform_i16<8, 8>;
  88: 
  89: def fconstpool_symbol : Operand<iPTR> {
  90:   let ParserMatchClass = Constpool;
```

- EN: This range defines declarative TableGen records such as fpimm8_24, fpimm16, fpimm16_8, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 fpimm8_24, fpimm16, fpimm16_8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91:   let EncoderMethod =
  92:     "getConstpoolSymbolOpValue<CSKY::fixup_csky_pcrel_uimm8_scale4>";
  93:   let DecoderMethod = "decodeUImmOperand<8, 2>";
  94:   let PrintMethod = "printConstpool";
  95:   let OperandType = "OPERAND_PCREL";
  96: }
  97: 
  98: 
  99: 
 100: //===----------------------------------------------------------------------===//
 101: // Instructions
 102: //===----------------------------------------------------------------------===//
 103: 
 104: //arithmetic
 105: 
 106: def FABSM : F_XZ<0x2, 0b000110, "fabsm", "", UnOpFrag<(fabs node:$Src)>, sFPR64_V_OP>;
 107: def FNEGM : F_XZ<0x2, 0b000111, "fnegm", "", UnOpFrag<(fneg node:$Src)>, sFPR64_V_OP>;
 108: def FADDM : F_XYZ<0x2, 0b000000, "faddm", "", BinOpFrag<(fadd node:$LHS, node:$RHS)>, sFPR64_V_OP>;
```

- EN: This range defines declarative TableGen records such as FABSM, FNEGM, FADDM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FABSM, FNEGM, FADDM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109: def FSUBM : F_XYZ<0x2, 0b000001, "fsubm", "", BinOpFrag<(fsub node:$LHS, node:$RHS)>, sFPR64_V_OP>;
 110: def FMULM : F_XYZ<0x2, 0b010000, "fmulm", "", BinOpFrag<(fmul node:$LHS, node:$RHS)>, sFPR64_V_OP>;
 111: def FNMULM : F_XYZ<0x2, 0b010001, "fnmulm", "", BinOpFrag<(fneg (fmul node:$LHS, node:$RHS))>, sFPR64_V_OP>;
 112: def FMACM : F_ACCUM_XYZ<0x2, 0b010100, "fmacm", "", TriOpFrag<(fadd node:$LHS, (fmul node:$MHS, node:$RHS))>, sFPR64_V_OP>;
 113: def FMSCM : F_ACCUM_XYZ<0x2, 0b010101, "fmscm", "", TriOpFrag<(fsub (fmul node:$MHS, node:$RHS), node:$LHS)>, sFPR64_V_OP>;
 114: def FNMACM : F_ACCUM_XYZ<0x2, 0b010110, "fnmacm", "", TriOpFrag<(fsub node:$LHS, (fmul node:$MHS, node:$RHS))>, sFPR64_V_OP>;
 115: def FNMSCM : F_ACCUM_XYZ<0x2, 0b010111, "fnmscm", "", TriOpFrag<(fneg (fadd node:$LHS, (fmul node:$MHS, node:$RHS)))>, sFPR64_V_OP>;
 116: 
 117: def FMOVM :  F_MOV<0x2, 0b000100, "fmovm", "", sFPR64_V_OP>;
 118: 
 119: defm FABS   : FT_XZ<0b000110, "fabs", UnOpFrag<(fabs node:$Src)>>;
 120: defm FNEG   : FT_XZ<0b000111, "fneg", UnOpFrag<(fneg node:$Src)>>;
 121: defm FSQRT  : FT_XZ<0b011010, "fsqrt", UnOpFrag<(fsqrt node:$Src)>>;
 122: 
 123: defm FADD   : FT_XYZ<0b000000, "fadd", BinOpFrag<(fadd node:$LHS, node:$RHS)>>;
 124: defm FSUB   : FT_XYZ<0b000001, "fsub", BinOpFrag<(fsub node:$LHS, node:$RHS)>>;
 125: defm FDIV   : FT_XYZ<0b011000, "fdiv", BinOpFrag<(fdiv node:$LHS, node:$RHS)>>;
 126: defm FMUL   : FT_XYZ<0b010000, "fmul", BinOpFrag<(fmul node:$LHS, node:$RHS)>>;
```

- EN: This range defines declarative TableGen records such as FSUBM, FMULM, FNMULM, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FSUBM, FMULM, FNMULM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 127-144

```tablegen
 127: defm FNMUL  : FT_XYZ<0b010001, "fnmul", BinOpFrag<(fneg (fmul node:$LHS, node:$RHS))>>;
 128: defm FMAC   : FT_ACCUM_XYZ<0b010100, "fmac", TriOpFrag<(fadd node:$LHS, (fmul node:$MHS, node:$RHS))>>;
 129: defm FMSC   : FT_ACCUM_XYZ<0b010101, "fmsc", TriOpFrag<(fsub (fmul node:$MHS, node:$RHS), node:$LHS)>>;
 130: defm FNMAC  : FT_ACCUM_XYZ<0b010110, "fnmac", TriOpFrag<(fsub node:$LHS, (fmul node:$MHS, node:$RHS))>>;
 131: defm FNMSC  : FT_ACCUM_XYZ<0b010111, "fnmsc", TriOpFrag<(fneg (fadd node:$LHS, (fmul node:$MHS, node:$RHS)))>>;
 132: 
 133: defm FCMPHS : FT_CMPXY<0b001100, "fcmphs">;
 134: defm FCMPLT : FT_CMPXY<0b001101, "fcmplt">;
 135: defm FCMPNE : FT_CMPXY<0b001110, "fcmpne">;
 136: defm FCMPUO : FT_CMPXY<0b001111, "fcmpuo">;
 137: defm FCMPZHS : FT_CMPZX<0b001000, "fcmpzhs">;
 138: defm FCMPZLS : FT_CMPZX<0b001001, "fcmpzls">;
 139: defm FCMPZNE : FT_CMPZX<0b001010, "fcmpzne">;
 140: defm FCMPZUO : FT_CMPZX<0b001011, "fcmpzuo">;
 141: 
 142: defm FRECIP   : FT_MOV<0b011001, "frecip">;
 143: 
 144: // multiplication
```

- EN: This range defines declarative TableGen records such as FNMUL, FMAC, FMSC, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 FNMUL, FMAC, FMSC 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 145-162

```tablegen
 145: let Predicates = [HasFPUv2_SF] in {
 146:   def : Pat<(f32 (fmul (fneg sFPR32Op:$vrx), sFPR32Op:$vry)),
 147:             (FNMUL_S sFPR32Op:$vrx, sFPR32Op:$vry)>;
 148: }
 149: let Predicates = [HasFPUv2_DF] in {
 150:   def : Pat<(f64 (fmul (fneg sFPR64Op:$vrx), sFPR64Op:$vry)),
 151:             (FNMUL_D sFPR64Op:$vrx, sFPR64Op:$vry)>;
 152: }
 153: 
 154: //fmov, fmtvr, fmfvr
 155: defm FMOV : FT_MOV<0b000100, "fmov">;
 156: def FMFVRL : F_XZ_GF<3, 0b011001, (outs GPR:$rz), (ins sFPR32Op:$vrx),
 157:                      "fmfvrl\t$rz, $vrx", [(set GPR:$rz, (bitconvert sFPR32Op:$vrx))]>;
 158: def FMTVRL : F_XZ_FG<3, 0b011011, (outs sFPR32Op:$vrz), (ins GPR:$rx),
 159:                      "fmtvrl\t$vrz, $rx", [(set sFPR32Op:$vrz, (bitconvert GPR:$rx))]>;
 160: 
 161: let Predicates = [HasFPUv2_DF] in {
 162:   let isCodeGenOnly = 1 in
```

- EN: This range defines declarative TableGen records such as FMOV, FMFVRL, FMTVRL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 FMOV, FMFVRL, FMTVRL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 163-180

```tablegen
 163:   def FMFVRL_D : F_XZ_GF<3, 0b011001, (outs GPR:$rz), (ins sFPR64Op:$vrx),
 164:                          "fmfvrl\t$rz, $vrx", []>;
 165:   def FMFVRH_D : F_XZ_GF<3, 0b011000, (outs GPR:$rz), (ins sFPR64Op:$vrx),
 166:                          "fmfvrh\t$rz, $vrx", []>;
 167:   let isCodeGenOnly = 1 in
 168:   def FMTVRL_D : F_XZ_FG<3, 0b011011, (outs sFPR64Op:$vrz), (ins GPR:$rx),
 169:                          "fmtvrl\t$vrz, $rx", []>;
 170: let Constraints = "$vrZ = $vrz" in
 171:   def FMTVRH_D : F_XZ_FG<3, 0b011010, (outs sFPR64Op:$vrz), (ins sFPR64Op:$vrZ, GPR:$rx),
 172:                          "fmtvrh\t$vrz, $rx", []>;
 173: }
 174: 
 175: //fcvt
 176: 
 177: def FSITOS  : F_XZ_TRANS<0b010000, "fsitos", sFPR32Op, sFPR32Op>;
 178: def : Pat<(f32 (sint_to_fp GPR:$a)),
 179:           (FSITOS (COPY_TO_REGCLASS GPR:$a, sFPR32))>,
 180:           Requires<[HasFPUv2_SF]>;
```

- EN: This range defines declarative TableGen records such as FMFVRL_D, FMFVRH_D, FMTVRL_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 FMFVRL_D, FMFVRH_D, FMTVRL_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 181-198

```tablegen
 181: 
 182: def FUITOS  : F_XZ_TRANS<0b010001, "fuitos", sFPR32Op, sFPR32Op>;
 183: def : Pat<(f32 (uint_to_fp GPR:$a)),
 184:           (FUITOS (COPY_TO_REGCLASS GPR:$a, sFPR32))>,
 185:           Requires<[HasFPUv2_SF]>;
 186: 
 187: def FSITOD  : F_XZ_TRANS<0b010100, "fsitod", sFPR64Op, sFPR64Op>;
 188: def : Pat<(f64 (sint_to_fp GPR:$a)),
 189:             (FSITOD (COPY_TO_REGCLASS GPR:$a, sFPR64))>,
 190:            Requires<[HasFPUv2_DF]>;
 191: 
 192: def FUITOD  : F_XZ_TRANS<0b010101, "fuitod", sFPR64Op, sFPR64Op>;
 193: def : Pat<(f64 (uint_to_fp GPR:$a)),
 194:             (FUITOD (COPY_TO_REGCLASS GPR:$a, sFPR64))>,
 195:            Requires<[HasFPUv2_DF]>;
 196: 
 197: let Predicates = [HasFPUv2_DF] in {
 198: def FDTOS   : F_XZ_TRANS_DS<0b010110,"fdtos", UnOpFrag<(fpround node:$Src)>>;
```

- EN: This range defines declarative TableGen records such as FUITOS, FSITOD, FUITOD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 FUITOS, FSITOD, FUITOD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 199-216

```tablegen
 199: def FSTOD   : F_XZ_TRANS_SD<0b010111,"fstod", UnOpFrag<(fpextend node:$Src)>>;
 200: }
 201: 
 202: def rpiFSTOSI : F_XZ_TRANS<0b000010, "fstosi.rpi", sFPR32Op, sFPR32Op>;
 203: def rpiFSTOUI : F_XZ_TRANS<0b000110, "fstoui.rpi", sFPR32Op, sFPR32Op>;
 204: def rzFSTOSI : F_XZ_TRANS<0b000001, "fstosi.rz", sFPR32Op, sFPR32Op>;
 205: def rzFSTOUI : F_XZ_TRANS<0b000101, "fstoui.rz", sFPR32Op, sFPR32Op>;
 206: def rnFSTOSI : F_XZ_TRANS<0b000000, "fstosi.rn", sFPR32Op, sFPR32Op>;
 207: def rnFSTOUI : F_XZ_TRANS<0b000100, "fstoui.rn", sFPR32Op, sFPR32Op>;
 208: def rniFSTOSI : F_XZ_TRANS<0b000011, "fstosi.rni", sFPR32Op, sFPR32Op>;
 209: def rniFSTOUI : F_XZ_TRANS<0b000111, "fstoui.rni", sFPR32Op, sFPR32Op>;
 210: 
 211: let Predicates = [HasFPUv2_DF] in {
 212: def rpiFDTOSI : F_XZ_TRANS<0b001010, "fdtosi.rpi", sFPR64Op, sFPR64Op>;
 213: def rpiFDTOUI : F_XZ_TRANS<0b001110, "fdtoui.rpi", sFPR64Op, sFPR64Op>;
 214: def rzFDTOSI : F_XZ_TRANS<0b001001, "fdtosi.rz", sFPR64Op, sFPR64Op>;
 215: def rzFDTOUI : F_XZ_TRANS<0b001101, "fdtoui.rz", sFPR64Op, sFPR64Op>;
 216: def rnFDTOSI : F_XZ_TRANS<0b001000, "fdtosi.rn", sFPR64Op, sFPR64Op>;
```

- EN: This range defines declarative TableGen records such as FSTOD, rpiFSTOSI, rpiFSTOUI, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FSTOD, rpiFSTOSI, rpiFSTOUI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217: def rnFDTOUI : F_XZ_TRANS<0b001100, "fdtoui.rn", sFPR64Op, sFPR64Op>;
 218: def rniFDTOSI : F_XZ_TRANS<0b001011, "fdtosi.rni", sFPR64Op, sFPR64Op>;
 219: def rniFDTOUI : F_XZ_TRANS<0b001111, "fdtoui.rni", sFPR64Op, sFPR64Op>;
 220: }
 221: 
 222: multiclass FPToIntegerPats<SDNode round, string SUFFIX> {
 223:   def : Pat<(i32 (fp_to_sint (round sFPR32Op:$Rn))),
 224:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FSTOSI) sFPR32Op:$Rn), GPR)>,
 225:             Requires<[HasFPUv2_SF]>;
 226:   def : Pat<(i32 (fp_to_uint (round sFPR32Op:$Rn))),
 227:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FSTOUI) sFPR32Op:$Rn), GPR)>,
 228:             Requires<[HasFPUv2_SF]>;
 229:   def : Pat<(i32 (fp_to_sint (round sFPR64Op:$Rn))),
 230:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FDTOSI) sFPR64Op:$Rn), GPR)>,
 231:             Requires<[HasFPUv2_DF]>;
 232:   def : Pat<(i32 (fp_to_uint (round sFPR64Op:$Rn))),
 233:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FDTOUI) sFPR64Op:$Rn), GPR)>,
 234:             Requires<[HasFPUv2_DF]>;
```

- EN: This range defines declarative TableGen records such as rnFDTOUI, rniFDTOSI, rniFDTOUI, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 rnFDTOUI, rniFDTOSI, rniFDTOUI 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 235-252

```tablegen
 235: }
 236: 
 237: defm: FPToIntegerPats<fceil, "rpi">;
 238: defm: FPToIntegerPats<fround, "rn">;
 239: defm: FPToIntegerPats<ffloor, "rni">;
 240: 
 241: multiclass FPToIntegerTowardszeroPats<string SUFFIX> {
 242:   def : Pat<(i32 (fp_to_sint sFPR32Op:$Rn)),
 243:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FSTOSI) sFPR32Op:$Rn), GPR)>,
 244:             Requires<[HasFPUv2_SF]>;
 245:   def : Pat<(i32 (fp_to_uint sFPR32Op:$Rn)),
 246:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FSTOUI) sFPR32Op:$Rn), GPR)>,
 247:             Requires<[HasFPUv2_SF]>;
 248:   def : Pat<(i32 (fp_to_sint sFPR64Op:$Rn)),
 249:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FDTOSI) sFPR64Op:$Rn), GPR)>,
 250:             Requires<[HasFPUv2_DF]>;
 251:   def : Pat<(i32 (fp_to_uint sFPR64Op:$Rn)),
 252:             (COPY_TO_REGCLASS (!cast<Instruction>(SUFFIX # FDTOUI) sFPR64Op:$Rn), GPR)>,
```

- EN: This range defines declarative TableGen records such as FPToIntegerTowardszeroPats, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 FPToIntegerTowardszeroPats 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 253-270

```tablegen
 253:             Requires<[HasFPUv2_DF]>;
 254: }
 255: 
 256: defm: FPToIntegerTowardszeroPats<"rz">;
 257: 
 258: 
 259: //fld, fst
 260: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
 261:   defm FLD : FT_XYAI_LD<0b0010000, "fld">;
 262:   defm FLDR : FT_XYAR_LD<0b0010100, "fldr">;
 263:   defm FLDM : FT_XYAR_LDM<0b0011000, "fldm">;
 264: 
 265:   let Predicates = [HasFPUv2_DF] in
 266:   def FLDRM : F_XYAR_LD<0b0010101, 0, "fldrm", "", sFPR64Op>;
 267:   let Predicates = [HasFPUv2_DF] in
 268:   def FLDMM : F_I4_XY_MEM<0b0011001, 0,
 269:     (outs), (ins GPR:$rx, regseq_d1:$regs, variable_ops), "fldmm\t$regs, (${rx})", []>;
 270:   let Predicates = [HasFPUv2_DF] in
```

- EN: This range defines declarative TableGen records such as FLD, FLDR, FLDM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FLD, FLDR, FLDM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271:   def FLDM : F_XYAI_LD<0b0010001, 0, "fldm", "", sFPR64Op, uimm8_3>;
 272: }
 273: 
 274: 
 275: 
 276: let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in {
 277:   defm FST : FT_XYAI_ST<0b0010010, "fst">;
 278:   defm FSTR : FT_XYAR_ST<0b0010110, "fstr">;
 279:   defm FSTM : FT_XYAR_STM<0b0011010, "fstm">;
 280: 
 281:   let Predicates = [HasFPUv2_DF] in
 282:   def FSTRM : F_XYAR_ST<0b0010111, 0, "fstrm", "", sFPR64Op>;
 283:   let Predicates = [HasFPUv2_DF] in
 284:   def FSTMM :  F_I4_XY_MEM<0b0011011, 0,
 285:     (outs), (ins GPR:$rx, regseq_d1:$regs, variable_ops), "fstmm\t$regs, (${rx})", []>;
 286:   let Predicates = [HasFPUv2_DF] in
 287:   def FSTM : F_XYAI_ST<0b0010011, 0, "fstm", "", sFPR64Op, uimm8_3>;
 288: }
```

- EN: This range defines declarative TableGen records such as FLDM, FST, FSTR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FLDM, FST, FSTR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: 
 290: defm : LdPat<load, uimm8_2, FLD_S, f32>, Requires<[HasFPUv2_SF]>;
 291: defm : LdPat<load, uimm8_2, FLD_D, f64>, Requires<[HasFPUv2_DF]>;
 292: defm : LdrPat<load, FLDR_S, f32>, Requires<[HasFPUv2_SF]>;
 293: defm : LdrPat<load, FLDR_D, f64>, Requires<[HasFPUv2_DF]>;
 294: 
 295: defm : StPat<store, f32, uimm8_2, FST_S>, Requires<[HasFPUv2_SF]>;
 296: defm : StPat<store, f64, uimm8_2, FST_D>, Requires<[HasFPUv2_DF]>;
 297: defm : StrPat<store, f32, FSTR_S>, Requires<[HasFPUv2_SF]>;
 298: defm : StrPat<store, f64, FSTR_D>, Requires<[HasFPUv2_DF]>;
 299: 
 300: 
 301: def : Pat<(f32 fpimm16:$imm), (COPY_TO_REGCLASS (MOVI32 (fpimm32_lo16 fpimm16:$imm)), sFPR32)>,
 302:         Requires<[HasFPUv2_SF]>;
 303: def : Pat<(f32 fpimm16_16:$imm), (f32 (COPY_TO_REGCLASS (MOVIH32 (fpimm32_hi16 fpimm16_16:$imm)), sFPR32))>,
 304:         Requires<[HasFPUv2_SF]>;
 305: def : Pat<(f32 fpimm:$imm), (COPY_TO_REGCLASS (ORI32 (MOVIH32 (fpimm32_hi16 fpimm:$imm)), (fpimm32_lo16 fpimm:$imm)), sFPR32)>,
 306:         Requires<[HasFPUv2_SF]>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 307-324

```tablegen
 307: 
 308: def : Pat<(f64(CSKY_BITCAST_FROM_LOHI GPR:$rs1, GPR:$rs2)), (FMTVRH_D(FMTVRL_D GPR:$rs1), GPR:$rs2)>,
 309:         Requires<[HasFPUv2_DF]>;
 310: 
 311: multiclass BRCond_Bin<CondCode CC, string Instr, Instruction Br0, Instruction Br1, Instruction MV> {
 312:   let Predicates = [HasFPUv2_SF] in
 313:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, sFPR32Op:$rs2, CC)), bb:$imm16),
 314:             (Br0 (!cast<Instruction>(Instr#_S) sFPR32Op:$rs1, sFPR32Op:$rs2), bb:$imm16)>;
 315:   let Predicates = [HasFPUv2_SF] in
 316:   def : Pat<(brcond (xor (i32 (setcc sFPR32Op:$rs1, sFPR32Op:$rs2, CC)), 1), bb:$imm16),
 317:             (Br1 (!cast<Instruction>(Instr#_S) sFPR32Op:$rs1, sFPR32Op:$rs2), bb:$imm16)>;
 318:   let Predicates = [HasFPUv2_DF] in
 319:   def : Pat<(brcond (i32 (setcc sFPR64Op:$rs1, sFPR64Op:$rs2, CC)), bb:$imm16),
 320:             (Br0 (!cast<Instruction>(Instr#_D) sFPR64Op:$rs1, sFPR64Op:$rs2), bb:$imm16)>;
 321:   let Predicates = [HasFPUv2_DF] in
 322:   def : Pat<(brcond (xor (i32 (setcc sFPR64Op:$rs1, sFPR64Op:$rs2, CC)), 1), bb:$imm16),
 323:             (Br1 (!cast<Instruction>(Instr#_D) sFPR64Op:$rs1, sFPR64Op:$rs2), bb:$imm16)>;
 324: 
```

- EN: This range defines declarative TableGen records such as BRCond_Bin, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BRCond_Bin 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 325-342

```tablegen
 325:   let Predicates = [HasFPUv2_SF] in
 326:   def : Pat<(i32 (setcc sFPR32Op:$rs1, sFPR32Op:$rs2, CC)),
 327:             (MV (!cast<Instruction>(Instr#_S) sFPR32Op:$rs1, sFPR32Op:$rs2))>;
 328:   let Predicates = [HasFPUv2_DF] in
 329:   def : Pat<(i32 (setcc sFPR64Op:$rs1, sFPR64Op:$rs2, CC)),
 330:             (MV (!cast<Instruction>(Instr#_D) sFPR64Op:$rs1, sFPR64Op:$rs2))>;
 331: }
 332: 
 333: multiclass BRCond_Bin_SWAP<CondCode CC, string Instr, Instruction Br0, Instruction Br1, Instruction MV> {
 334:   let Predicates = [HasFPUv2_SF] in
 335:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, sFPR32Op:$rs2, CC)), bb:$imm16),
 336:             (Br0 (!cast<Instruction>(Instr#_S) sFPR32Op:$rs2, sFPR32Op:$rs1), bb:$imm16)>;
 337:   let Predicates = [HasFPUv2_SF] in
 338:   def : Pat<(brcond (xor (i32 (setcc sFPR32Op:$rs1, sFPR32Op:$rs2, CC)), 1), bb:$imm16),
 339:             (Br1 (!cast<Instruction>(Instr#_S) sFPR32Op:$rs2, sFPR32Op:$rs1), bb:$imm16)>;
 340:   let Predicates = [HasFPUv2_DF] in
 341:   def : Pat<(brcond (i32 (setcc sFPR64Op:$rs1, sFPR64Op:$rs2, CC)), bb:$imm16),
 342:             (Br0 (!cast<Instruction>(Instr#_D) sFPR64Op:$rs2, sFPR64Op:$rs1), bb:$imm16)>;
```

- EN: This range defines declarative TableGen records such as BRCond_Bin_SWAP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BRCond_Bin_SWAP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 343-360

```tablegen
 343:   let Predicates = [HasFPUv2_DF] in
 344:   def : Pat<(brcond (xor (i32 (setcc sFPR64Op:$rs1, sFPR64Op:$rs2, CC)), 1), bb:$imm16),
 345:             (Br1 (!cast<Instruction>(Instr#_D) sFPR64Op:$rs2, sFPR64Op:$rs1), bb:$imm16)>;
 346: 
 347:   let Predicates = [HasFPUv2_SF] in
 348:   def : Pat<(i32 (setcc sFPR32Op:$rs1, sFPR32Op:$rs2, CC)),
 349:             (MV (!cast<Instruction>(Instr#_S) sFPR32Op:$rs2, sFPR32Op:$rs1))>;
 350:   let Predicates = [HasFPUv2_DF] in
 351:   def : Pat<(i32 (setcc sFPR64Op:$rs1, sFPR64Op:$rs2, CC)),
 352:             (MV (!cast<Instruction>(Instr#_D) sFPR64Op:$rs2, sFPR64Op:$rs1))>;
 353: }
 354: 
 355: // inverse (order && compare) to (unorder || inverse(compare))
 356: 
 357: defm : BRCond_Bin<SETUNE, "FCMPNE", BT32, BF32, MVC32>;
 358: defm : BRCond_Bin<SETOEQ, "FCMPNE", BF32, BT32, MVCV32>;
 359: defm : BRCond_Bin<SETOGE, "FCMPHS", BT32, BF32, MVC32>;
 360: defm : BRCond_Bin<SETOLT, "FCMPLT", BT32, BF32, MVC32>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 361-378

```tablegen
 361: defm : BRCond_Bin<SETUO, "FCMPUO", BT32, BF32, MVC32>;
 362: defm : BRCond_Bin<SETO, "FCMPUO", BF32, BT32, MVCV32>;
 363: defm : BRCond_Bin_SWAP<SETOGT, "FCMPLT", BT32, BF32, MVC32>;
 364: defm : BRCond_Bin_SWAP<SETOLE, "FCMPHS", BT32, BF32, MVC32>;
 365: 
 366: defm : BRCond_Bin<SETNE, "FCMPNE", BT32, BF32, MVC32>;
 367: defm : BRCond_Bin<SETEQ, "FCMPNE", BF32, BT32, MVCV32>;
 368: defm : BRCond_Bin<SETGE, "FCMPHS", BT32, BF32, MVC32>;
 369: defm : BRCond_Bin<SETLT, "FCMPLT", BT32, BF32, MVC32>;
 370: defm : BRCond_Bin_SWAP<SETGT, "FCMPLT", BT32, BF32, MVC32>;
 371: defm : BRCond_Bin_SWAP<SETLE, "FCMPHS", BT32, BF32, MVC32>;
 372: 
 373: // -----------
 374: 
 375: let Predicates = [HasFPUv2_SF] in {
 376:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETOGE)), bb:$imm16),
 377:             (BT32 (FCMPZHS_S sFPR32Op:$rs1), bb:$imm16)>;
 378:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETOGE)),
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 379-396

```tablegen
 379:             (MVC32 (FCMPZHS_S sFPR32Op:$rs1))>;
 380:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETOLT)), bb:$imm16),
 381:             (BF32 (FCMPZHS_S sFPR32Op:$rs1), bb:$imm16)>;
 382:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETOLT)),
 383:             (MVCV32 (FCMPZHS_S sFPR32Op:$rs1))>;
 384:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETOLE)), bb:$imm16),
 385:             (BT32 (FCMPZLS_S sFPR32Op:$rs1), bb:$imm16)>;
 386:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETOLE)),
 387:             (MVC32 (FCMPZLS_S sFPR32Op:$rs1))>;
 388:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETOGT)), bb:$imm16),
 389:             (BF32 (FCMPZLS_S sFPR32Op:$rs1), bb:$imm16)>;
 390:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETOGT)),
 391:             (MVCV32 (FCMPZLS_S sFPR32Op:$rs1))>;
 392:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETUNE)), bb:$imm16),
 393:             (BT32 (FCMPZNE_S sFPR32Op:$rs1), bb:$imm16)>;
 394:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETUNE)),
 395:             (MVC32 (FCMPZNE_S sFPR32Op:$rs1))>;
 396:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETOEQ)), bb:$imm16),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 397-414

```tablegen
 397:             (BF32 (FCMPZNE_S sFPR32Op:$rs1), bb:$imm16)>;
 398:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETOEQ)),
 399:             (MVCV32 (FCMPZNE_S sFPR32Op:$rs1))>;
 400:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm, SETUO)), bb:$imm16),
 401:             (BT32 (FCMPZUO_S sFPR32Op:$rs1), bb:$imm16)>;
 402:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm, SETUO)),
 403:             (MVC32 (FCMPZUO_S sFPR32Op:$rs1))>;
 404:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm, SETO)), bb:$imm16),
 405:             (BF32 (FCMPZUO_S sFPR32Op:$rs1), bb:$imm16)>;
 406:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm, SETO)),
 407:             (MVCV32 (FCMPZUO_S sFPR32Op:$rs1))>;
 408:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETGE)), bb:$imm16),
 409:             (BT32 (FCMPZHS_S sFPR32Op:$rs1), bb:$imm16)>;
 410:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETGE)),
 411:             (MVC32 (FCMPZHS_S sFPR32Op:$rs1))>;
 412:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETLT)), bb:$imm16),
 413:             (BF32 (FCMPZHS_S sFPR32Op:$rs1), bb:$imm16)>;
 414:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETLT)),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 415-432

```tablegen
 415:             (MVCV32 (FCMPZHS_S sFPR32Op:$rs1))>;
 416:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETLE)), bb:$imm16),
 417:             (BT32 (FCMPZLS_S sFPR32Op:$rs1), bb:$imm16)>;
 418:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETLE)),
 419:             (MVC32 (FCMPZLS_S sFPR32Op:$rs1))>;
 420:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETGT)), bb:$imm16),
 421:             (BF32 (FCMPZLS_S sFPR32Op:$rs1), bb:$imm16)>;
 422:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETGT)),
 423:             (MVCV32 (FCMPZLS_S sFPR32Op:$rs1))>;
 424:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETNE)), bb:$imm16),
 425:             (BT32 (FCMPZNE_S sFPR32Op:$rs1), bb:$imm16)>;
 426:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETNE)),
 427:             (MVC32 (FCMPZNE_S sFPR32Op:$rs1))>;
 428:   def : Pat<(brcond (i32 (setcc sFPR32Op:$rs1, fpimm0, SETEQ)), bb:$imm16),
 429:             (BF32 (FCMPZNE_S sFPR32Op:$rs1), bb:$imm16)>;
 430:   def : Pat<(i32 (setcc sFPR32Op:$rs1, fpimm0, SETEQ)),
 431:             (MVCV32 (FCMPZNE_S sFPR32Op:$rs1))>;
 432: }
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 433-442

```tablegen
 433: 
 434: let usesCustomInserter = 1 in  {
 435:   let Predicates = [HasFPUv2_SF] in
 436:   def FSELS : CSKYPseudo<(outs sFPR32Op:$dst), (ins CARRY:$cond, sFPR32Op:$src1, sFPR32Op:$src2),
 437:     "!fsels\t$dst, $src1, src2", [(set sFPR32Op:$dst, (select CARRY:$cond, sFPR32Op:$src1, sFPR32Op:$src2))]>;
 438: 
 439:   let Predicates = [HasFPUv2_DF] in
 440:   def FSELD : CSKYPseudo<(outs sFPR64Op:$dst), (ins CARRY:$cond, sFPR64Op:$src1, sFPR64Op:$src2),
 441:     "!fseld\t$dst, $src1, src2", [(set sFPR64Op:$dst, (select CARRY:$cond, sFPR64Op:$src1, sFPR64Op:$src2))]>;
 442: }
```

- EN: This range defines declarative TableGen records such as FSELS, FSELD, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FSELS, FSELD 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYInstrFormatsF1.td`
- LLVM subsystems / LLVM 子系统: TableGen
