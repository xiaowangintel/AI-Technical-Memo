# CSKYInstrInfoF2.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrInfoF2.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the CSKY instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===- CSKYInstrInfoF2.td - CSKY Instruction Float2.0 ------*- tablegen -*-===//
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
  13: def regseq_f2 : Operand<i32> {
  14:   let EncoderMethod = "getRegisterSeqOpValue";
  15:   let ParserMatchClass = RegSeqAsmOperand<"V2">;
  16:   let PrintMethod = "printRegisterSeq";
  17:   let DecoderMethod = "DecodeRegSeqOperandF2";
  18:   let MIOperandInfo = (ops FPR32, uimm5);
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as regseq_f2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 regseq_f2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 19-36

```tablegen
  19: }
  20: 
  21: def regseq_d2 : Operand<i32> {
  22:   let EncoderMethod = "getRegisterSeqOpValue";
  23:   let ParserMatchClass = RegSeqAsmOperand<"V2">;
  24:   let PrintMethod = "printRegisterSeq";
  25:   let DecoderMethod = "DecodeRegSeqOperandD2";
  26:   let MIOperandInfo = (ops FPR64, uimm5);
  27: }
  28: 
  29: def FPR32Op : RegisterOperand<FPR32, "printFPR">;
  30: def FPR64Op : RegisterOperand<FPR64, "printFPR">;
  31: 
  32: include "CSKYInstrFormatsF2.td"
  33: 
  34: // Predicates
  35: def IsOrAdd: PatFrag<(ops node:$A, node:$B), (or node:$A, node:$B), [{
  36:   return isOrEquivalentToAdd(N);
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as regseq_d2, FPR32Op, FPR64Op, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 regseq_d2, FPR32Op, FPR64Op 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: }]>;
  38: 
  39: //===----------------------------------------------------------------------===//
  40: // Instructions
  41: //===----------------------------------------------------------------------===//
  42: 
  43: defm f2FADD   : F2_XYZ_T<0b000000, "fadd", BinOpFrag<(fadd node:$LHS, node:$RHS)>>;
  44: defm f2FSUB   : F2_XYZ_T<0b000001, "fsub", BinOpFrag<(fsub node:$LHS, node:$RHS)>>;
  45: defm f2FDIV   : F2_XYZ_T<0b011000, "fdiv", BinOpFrag<(fdiv node:$LHS, node:$RHS)>>;
  46: defm f2FMUL   : F2_XYZ_T<0b010000, "fmul", BinOpFrag<(fmul node:$LHS, node:$RHS)>>;
  47: 
  48: defm f2FMAXNM : F2_XYZ_T<0b101000, "fmaxnm", BinOpFrag<(fmaxnum node:$LHS, node:$RHS)>>;
  49: defm f2FMINNM : F2_XYZ_T<0b101001, "fminnm", BinOpFrag<(fminnum node:$LHS, node:$RHS)>>;
  50: 
  51: defm f2FABS   : F2_XZ_T<0b000110, "fabs", fabs>;
  52: defm f2FNEG   : F2_XZ_T<0b000111, "fneg", fneg>;
  53: defm f2FSQRT  : F2_XZ_T<0b011010, "fsqrt", fsqrt>;
  54: defm f2FMOV   : F2_XZ_SET_T<0b000100, "fmov">;
```

- EN: This range defines declarative TableGen records such as f2FADD, f2FSUB, f2FDIV, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 f2FADD, f2FSUB, f2FDIV 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: def f2FMOVX   : F2_XZ_SET<0b00001, FPR32Op, 0b000101, "fmovx.32">;
  56: 
  57: defm f2RECIP   : F2_XZ_SET_T<0b011001, "frecip">;
  58: 
  59: // fld/fst
  60: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
  61:   def f2FLD_S : F2_LDST_S<0b0, "fld", (outs FPR32Op:$vrz), (ins GPR:$rx, uimm8_2:$imm8)>;
  62:   let Predicates = [HasFPUv3_DF] in
  63:   def f2FLD_D : F2_LDST_D<0b0, "fld", (outs FPR64Op:$vrz), (ins GPR:$rx, uimm8_2:$imm8)>;
  64: }
  65: let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in {
  66:   def f2FST_S : F2_LDST_S<0b1, "fst", (outs), (ins FPR32Op:$vrz, GPR:$rx, uimm8_2:$imm8)>;
  67:   let Predicates = [HasFPUv3_DF] in
  68:   def f2FST_D : F2_LDST_D<0b1, "fst", (outs), (ins FPR64Op:$vrz, GPR:$rx, uimm8_2:$imm8)>;
  69: }
  70: 
  71: let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in {
  72:   def f2FSTM_S : F2_LDSTM_S<0b1, 0, "fstm", (outs), (ins GPR:$rx, regseq_f2:$regs, variable_ops)>;
```

- EN: This range defines declarative TableGen records such as f2FMOVX, f2RECIP, f2FLD_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 f2FMOVX, f2RECIP, f2FLD_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73:   let Predicates = [HasFPUv3_DF] in
  74:   def f2FSTM_D : F2_LDSTM_D<0b1, 0, "fstm", (outs), (ins GPR:$rx, regseq_d2:$regs, variable_ops)>;
  75: 
  76:   def f2FSTMU_S : F2_LDSTM_S<0b1, 0b100, "fstmu", (outs), (ins GPR:$rx, regseq_f2:$regs, variable_ops)>;
  77:   let Predicates = [HasFPUv3_DF] in
  78:   def f2FSTMU_D : F2_LDSTM_D<0b1, 0b100, "fstmu", (outs), (ins GPR:$rx, regseq_d2:$regs, variable_ops)>;
  79: }
  80: 
  81: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
  82:   def f2FLDM_S : F2_LDSTM_S<0b0, 0, "fldm", (outs), (ins GPR:$rx, regseq_f2:$regs, variable_ops)>;
  83:   let Predicates = [HasFPUv3_DF] in
  84:   def f2FLDM_D : F2_LDSTM_D<0b0, 0, "fldm", (outs), (ins GPR:$rx,  regseq_d2:$regs, variable_ops)>;
  85: 
  86:   def f2FLDMU_S : F2_LDSTM_S<0b0, 0b100, "fldmu", (outs), (ins GPR:$rx, regseq_f2:$regs, variable_ops)>;
  87:   let Predicates = [HasFPUv3_DF] in
  88:   def f2FLDMU_D : F2_LDSTM_D<0b0, 0b100, "fldmu", (outs), (ins GPR:$rx, regseq_d2:$regs, variable_ops)>;
  89: }
  90: 
```

- EN: This range defines declarative TableGen records such as f2FSTM_D, f2FSTMU_S, f2FSTMU_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 f2FSTM_D, f2FSTMU_S, f2FSTMU_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 91-108

```tablegen
  91: multiclass FLSR {
  92:   let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in {
  93:     def FLDR_S : F2_LDSTR_S<0b0, "fldr", (outs FPR32Op:$rz), (ins GPR:$rx, GPR:$ry, uimm2:$imm)>;
  94:     let Predicates = [HasFPUv3_DF] in
  95:     def FLDR_D : F2_LDSTR_D<0b0, "fldr", (outs FPR64Op:$rz), (ins GPR:$rx, GPR:$ry, uimm2:$imm)>;
  96:   }
  97:   let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in {
  98:     def FSTR_S : F2_LDSTR_S<0b1, "fstr", (outs), (ins FPR32Op:$rz, GPR:$rx, GPR:$ry, uimm2:$imm)>;
  99:     let Predicates = [HasFPUv3_DF] in
 100:     def FSTR_D : F2_LDSTR_D<0b1, "fstr", (outs), (ins FPR64Op:$rz, GPR:$rx, GPR:$ry, uimm2:$imm)>;
 101:   }
 102: }
 103: 
 104: defm f2: FLSR;
 105: 
 106: def f2FLRW_S : F2_LRW<0b00, 0b0, "flrw.32", (outs FPR32Op:$vrz), (ins fconstpool_symbol:$imm8)>;
 107: def f2FLRW_D : F2_LRW<0b01, 0b0, "flrw.64", (outs FPR64Op:$vrz), (ins fconstpool_symbol:$imm8)>;
 108: 
```

- EN: This range defines declarative TableGen records such as FLSR, FLDR_S, FLDR_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 FLSR, FLDR_S, FLDR_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109: def : Pat<(f32 (load constpool:$src)), (f2FLRW_S (to_tconstpool tconstpool:$src))>, Requires<[HasFPUv3_SF]>;
 110: def : Pat<(f64 (load constpool:$src)), (f2FLRW_D (to_tconstpool tconstpool:$src))>, Requires<[HasFPUv3_DF]>;
 111: 
 112: defm : LdPat<load, uimm8_2, f2FLD_S, f32>, Requires<[HasFPUv3_SF]>;
 113: defm : LdPat<load, uimm8_2, f2FLD_D, f64>, Requires<[HasFPUv3_DF]>;
 114: defm : LdrPat<load, f2FLDR_S, f32>, Requires<[HasFPUv3_SF]>;
 115: defm : LdrPat<load, f2FLDR_D, f64>, Requires<[HasFPUv3_DF]>;
 116: 
 117: defm : StPat<store, f32, uimm8_2, f2FST_S>, Requires<[HasFPUv3_SF]>;
 118: defm : StPat<store, f64, uimm8_2, f2FST_D>, Requires<[HasFPUv3_DF]>;
 119: defm : StrPat<store, f32, f2FSTR_S>, Requires<[HasFPUv3_SF]>;
 120: defm : StrPat<store, f64, f2FSTR_D>, Requires<[HasFPUv3_DF]>;
 121: 
 122: // fmfvr
 123: let vry = 0 in
 124: def f2FMFVRL   : F2_XYZ<0b00011, 0b011001, "fmfvr.32.1\t$vrz, $vrx",
 125:                         (outs GPR:$vrz), (ins FPR32Op:$vrx),
 126:                         [(set GPR:$vrz, (bitconvert FPR32Op:$vrx))]>;
```

- EN: This range defines declarative TableGen records such as f2FMFVRL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 f2FMFVRL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 127-144

```tablegen
 127: // TODO: vrz and vrz+1
 128: def f2FMFVRL_2 : F2_XYZ<0b00011, 0b111010, "fmfvr.32.2\t$vrz, $vry, $vrx",
 129:                         (outs GPR:$vrz, GPR:$vry), (ins FPR64Op:$vrx),
 130:                         []>;
 131: 
 132: let Predicates = [HasFPUv3_DF] in {
 133: let vry = 0 in {
 134: let isCodeGenOnly = 1 in
 135: def f2FMFVRL_D : F2_XYZ<0b00011, 0b011001, "fmfvr.32.1\t$vrz, $vrx",
 136:                         (outs GPR:$vrz), (ins FPR64Op:$vrx),
 137:                         []>;
 138: def f2FMFVRH_D : F2_XYZ<0b00011, 0b011000, "fmfvrh\t$vrz, $vrx",
 139:                         (outs GPR:$vrz), (ins FPR64Op:$vrx),
 140:                         []>;
 141: }
 142: def f2FMFVR_D  : F2_XYZ<0b00011, 0b111000, "fmfvr.64\t$vrz, $vry, $vrx",
 143:                         (outs GPR:$vrz, GPR:$vry), (ins FPR64Op:$vrx),
 144:                         [(set GPR:$vrz, GPR:$vry, (CSKY_BITCAST_TO_LOHI FPR64Op:$vrx))]>;
```

- EN: This range defines declarative TableGen records such as f2FMFVRL_2, f2FMFVRL_D, f2FMFVRH_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 f2FMFVRL_2, f2FMFVRL_D, f2FMFVRH_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145: }
 146: 
 147: // fmtvr
 148: def f2FMTVRL   : F2_XZ_P<0b00011, 0b011011, "fmtvr.32.1",
 149:                          [(set FPR32Op:$vrz, (bitconvert GPR:$vrx))],
 150:                          (outs FPR32Op:$vrz), (ins GPR:$vrx)>;
 151: // TODO: vrz and vrz+1
 152: def f2FMTVRL_2 : F2_XYZ<0b00011, 0b111110, "fmtvr.32.2\t$vrz, $vrx, $vry",
 153:                         (outs FPR32Op:$vrz), (ins GPR:$vrx, GPR:$vry),
 154:                         []>;
 155: 
 156: let Predicates = [HasFPUv3_DF] in {
 157: let isCodeGenOnly = 1 in
 158: def f2FMTVRL_D : F2_XZ_P<0b00011, 0b011011, "fmtvr.32.1",
 159:                          [],
 160:                          (outs FPR64Op:$vrz), (ins GPR:$vrx)>;
 161: let Constraints = "$vrZ = $vrz" in
 162: def f2FMTVRH_D : F2_XZ_P<0b00011, 0b011010, "fmtvrh",
```

- EN: This range defines declarative TableGen records such as f2FMTVRL, f2FMTVRL_2, f2FMTVRL_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 f2FMTVRL, f2FMTVRL_2, f2FMTVRL_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163:                          [],
 164:                          (outs FPR64Op:$vrz), (ins FPR64Op:$vrZ, GPR:$vrx)>;
 165: def f2FMTVR_D  : F2_XYZ<0b00011, 0b111100, "fmtvr.64\t$vrz, $vrx, $vry",
 166:                         (outs FPR64Op:$vrz), (ins GPR:$vrx, GPR:$vry),
 167:                         [(set FPR64Op:$vrz, (CSKY_BITCAST_FROM_LOHI GPR:$vrx, GPR:$vry))]>;
 168: }
 169: 
 170: // fcmp
 171: 
 172: defm f2FCMPHS: F2_CXY_T<0b001100, "fcmphs">;
 173: defm f2FCMPLT: F2_CXY_T<0b001101, "fcmplt">;
 174: defm f2FCMPNE: F2_CXY_T<0b001110, "fcmpne">;
 175: defm f2FCMPUO: F2_CXY_T<0b001111, "fcmpuo">;
 176: 
 177: defm f2FCMPHSZ: F2_CX_T<0b001000, "fcmphsz">;
 178: defm f2FCMPHZ : F2_CX_T<0b101010, "fcmphz">;
 179: defm f2FCMPLSZ: F2_CX_T<0b101011, "fcmplsz">;
 180: defm f2FCMPLTZ: F2_CX_T<0b001001, "fcmpltz">;
```

- EN: This range defines declarative TableGen records such as f2FMTVR_D, f2FCMPHS, f2FCMPLT, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 f2FMTVR_D, f2FCMPHS, f2FCMPLT 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181: defm f2FCMPNEZ: F2_CX_T<0b001010, "fcmpnez">;
 182: defm f2FCMPUOZ: F2_CX_T<0b001011, "fcmpuoz">;
 183: 
 184: defm f2FMULA : F2_XYZZ_T<0b010100, "fmula",
 185:   TriOpFrag<(fadd (fmul node:$LHS, node:$MHS), node:$RHS)>>;
 186: 
 187: defm f2FMULS : F2_XYZZ_T<0b010110, "fmuls",
 188:   TriOpFrag<(fsub node:$RHS, (fmul node:$LHS, node:$MHS))>>;
 189: 
 190: defm f2FFMULA : F2_XYZZ_T<0b110000, "ffmula",
 191:   TriOpFrag<(fma node:$LHS, node:$MHS, node:$RHS)>>;
 192: 
 193: defm f2FFMULS : F2_XYZZ_T<0b110001, "ffmuls",
 194:   TriOpFrag<(fma (fneg node:$LHS), node:$MHS, node:$RHS)>>;
 195: 
 196: defm f2FFNMULA : F2_XYZZ_T<0b110010, "ffnmula",
 197:   TriOpFrag<(fneg (fma node:$LHS, node:$MHS, node:$RHS))>>;
 198: 
```

- EN: This range defines declarative TableGen records such as f2FCMPNEZ, f2FCMPUOZ, f2FMULA, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 f2FCMPNEZ, f2FCMPUOZ, f2FMULA 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 199-216

```tablegen
 199: defm f2FFNMULS : F2_XYZZ_T<0b110011, "ffnmuls",
 200:   TriOpFrag<(fma node:$LHS, node:$MHS, (fneg node:$RHS))>>;
 201: 
 202: defm f2FNMULA : F2_XYZZ_T<0b010111, "fnmula",
 203:   TriOpFrag<(fneg (fadd (fmul node:$LHS, node:$MHS), node:$RHS))>>;
 204: 
 205: defm f2FNMULS : F2_XYZZ_T<0b010101, "fnmuls",
 206:   TriOpFrag<(fneg (fsub node:$RHS, (fmul node:$LHS, node:$MHS)))>>;
 207: 
 208: defm f2FNMUL : F2_XYZ_T<0b010001, "fnmul",
 209:   BinOpFrag<(fneg (fmul node:$LHS, node:$RHS))>>;
 210: 
 211: // multiplication
 212: let Predicates = [HasFPUv3_SF] in {
 213:   def : Pat<(f32 (fmul (fneg FPR32Op:$vrx), FPR32Op:$vry)),
 214:             (f2FNMUL_S FPR32Op:$vrx, FPR32Op:$vry)>;
 215: }
 216: let Predicates = [HasFPUv3_DF] in {
```

- EN: This range defines declarative TableGen records such as f2FFNMULS, f2FNMULA, f2FNMULS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 f2FFNMULS, f2FNMULA, f2FNMULS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 217-234

```tablegen
 217:   def : Pat<(f64 (fmul (fneg FPR64Op:$vrx), FPR64Op:$vry)),
 218:             (f2FNMUL_D FPR64Op:$vrx, FPR64Op:$vry)>;
 219: }
 220: 
 221: // fcvt
 222: def f2FFTOS32_S  : F2_XZ_P<0b01000, 0b011011, "fftoi.f32.s32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 223: def f2FFTOU32_S  : F2_XZ_P<0b01000, 0b011010, "fftoi.f32.u32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 224: def f2FS32TOF_S  : F2_XZ_P<0b01001, 0b011011, "fitof.s32.f32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 225: def f2FU32TOF_S  : F2_XZ_P<0b01001, 0b011010, "fitof.u32.f32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 226: def f2FFTOXU32_S  : F2_XZ_P<0b01000, 0b001010, "fftox.f32.u32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 227: def f2FFTOXS32_S  : F2_XZ_P<0b01000, 0b001011, "fftox.f32.s32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 228: def f2FXTOFU32_S  : F2_XZ_P<0b01001, 0b001010, "fxtof.u32.f32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 229: def f2FXTOFS32_S  : F2_XZ_P<0b01001, 0b001011, "fxtof.s32.f32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 230: let Predicates = [HasFPUv3_DF] in {
 231: def f2FFTOS32_D  : F2_XZ_P<0b01000, 0b011101, "fftoi.f64.s32", [], (outs FPR32Op:$vrz), (ins FPR64Op:$vrx)>;
 232: def f2FFTOU32_D  : F2_XZ_P<0b01000, 0b011100, "fftoi.f64.u32", [], (outs FPR32Op:$vrz), (ins FPR64Op:$vrx)>;
 233: def f2FS32TOF_D  : F2_XZ_P<0b01001, 0b011101, "fitof.s32.f64", [], (outs FPR64Op:$vrz), (ins FPR32Op:$vrx)>;
 234: def f2FU32TOF_D  : F2_XZ_P<0b01001, 0b011100, "fitof.u32.f64", [], (outs FPR64Op:$vrz), (ins FPR32Op:$vrx)>;
```

- EN: This range defines declarative TableGen records such as f2FFTOS32_S, f2FFTOU32_S, f2FS32TOF_S, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 f2FFTOS32_S, f2FFTOU32_S, f2FS32TOF_S 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 235-252

```tablegen
 235: def f2FFTOXU32_D  : F2_XZ_P<0b01000, 0b001100, "fftox.f64.u32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 236: def f2FFTOXS32_D  : F2_XZ_P<0b01000, 0b001101, "fftox.f64.s32", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 237: def f2FXTOFU32_D  : F2_XZ_P<0b01001, 0b001100, "fxtof.u32.f64", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 238: def f2FXTOFS32_D  : F2_XZ_P<0b01001, 0b001101, "fxtof.s32.f64", [], (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 239: }
 240: 
 241: defm f2FF32TOSI32 : F2_XZ_RM<0b00011, 0b0000, "fftoi.f32.s32", (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 242: defm f2FF32TOUI32 : F2_XZ_RM<0b00011, 0b0001, "fftoi.f32.u32", (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 243: defm f2FF32TOFI32 : F2_XZ_RM<0b01000, 0b1001, "fftofi.f32", (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 244: let Predicates = [HasFPUv3_DF] in {
 245: defm f2FF64TOSI32 : F2_XZ_RM<0b00011, 0b0010, "fftoi.f64.s32", (outs FPR32Op:$vrz), (ins FPR64Op:$vrx)>;
 246: defm f2FF64TOUI32 : F2_XZ_RM<0b00011, 0b0011, "fftoi.f64.u32", (outs FPR32Op:$vrz), (ins FPR64Op:$vrx)>;
 247: defm f2FF64TOFI32 : F2_XZ_RM<0b01000, 0b1010, "fftofi.f64", (outs FPR32Op:$vrz), (ins FPR32Op:$vrx)>;
 248: }
 249: 
 250: def : Pat<(i32 (fp_to_sint (fround FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOSI32_RN  $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 251: def : Pat<(i32 (fp_to_uint (fround FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOUI32_RN  $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 252: def : Pat<(i32 (fp_to_sint (fceil  FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOSI32_RPI $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
```

- EN: This range defines declarative TableGen records such as f2FFTOXU32_D, f2FFTOXS32_D, f2FXTOFU32_D, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 f2FFTOXU32_D, f2FFTOXS32_D, f2FXTOFU32_D 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 253-270

```tablegen
 253: def : Pat<(i32 (fp_to_uint (fceil  FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOUI32_RPI $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 254: def : Pat<(i32 (fp_to_sint (ffloor FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOSI32_RNI $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 255: def : Pat<(i32 (fp_to_uint (ffloor FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOUI32_RNI $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 256: def : Pat<(i32 (fp_to_sint (ftrunc FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOSI32_RZ  $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 257: def : Pat<(i32 (fp_to_uint (ftrunc FPR32Op:$vrx))), (COPY_TO_REGCLASS (f2FF32TOUI32_RZ  $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 258: def : Pat<(i32 (fp_to_sint FPR32Op:$vrx)), (COPY_TO_REGCLASS (f2FF32TOSI32_RZ $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 259: def : Pat<(i32 (fp_to_uint FPR32Op:$vrx)), (COPY_TO_REGCLASS (f2FF32TOUI32_RZ $vrx), GPR)>, Requires<[HasFPUv3_SF]>;
 260: 
 261: def : Pat<(i32 (fp_to_sint (fround FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOSI32_RN  $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 262: def : Pat<(i32 (fp_to_uint (fround FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOUI32_RN  $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 263: def : Pat<(i32 (fp_to_sint (fceil  FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOSI32_RPI $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 264: def : Pat<(i32 (fp_to_uint (fceil  FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOUI32_RPI $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 265: def : Pat<(i32 (fp_to_sint (ffloor FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOSI32_RNI $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 266: def : Pat<(i32 (fp_to_uint (ffloor FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOUI32_RNI $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 267: def : Pat<(i32 (fp_to_sint (ftrunc FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOSI32_RZ  $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 268: def : Pat<(i32 (fp_to_uint (ftrunc FPR64Op:$vrx))), (COPY_TO_REGCLASS (f2FF64TOUI32_RZ  $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 269: def : Pat<(i32 (fp_to_sint FPR64Op:$vrx)), (COPY_TO_REGCLASS (f2FF64TOSI32_RZ $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
 270: def : Pat<(i32 (fp_to_uint FPR64Op:$vrx)), (COPY_TO_REGCLASS (f2FF64TOUI32_RZ $vrx), GPR)>, Requires<[HasFPUv3_DF]>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 271-288

```tablegen
 271: 
 272: def : Pat<(sint_to_fp GPR:$vrx), (f2FS32TOF_S (COPY_TO_REGCLASS $vrx, FPR32))>, Requires<[HasFPUv3_SF]>;
 273: def : Pat<(uint_to_fp GPR:$vrx), (f2FU32TOF_S (COPY_TO_REGCLASS $vrx, FPR32))>, Requires<[HasFPUv3_SF]>;
 274: def : Pat<(sint_to_fp GPR:$vrx), (f2FS32TOF_D (COPY_TO_REGCLASS $vrx, FPR32))>, Requires<[HasFPUv3_DF]>;
 275: def : Pat<(uint_to_fp GPR:$vrx), (f2FU32TOF_D (COPY_TO_REGCLASS $vrx, FPR32))>, Requires<[HasFPUv3_DF]>;
 276: 
 277: let Predicates = [HasFPUv3_DF] in {
 278: def f2FDTOS   : F2_XZ_P<0b00011, 0b010110, "fdtos", [(set FPR32Op:$vrz, (fpround FPR64Op:$vrx))], (outs FPR32Op:$vrz),
 279:                         (ins FPR64Op:$vrx)>;
 280: def f2FSTOD   : F2_XZ_P<0b00011, 0b010111, "fstod", [(set FPR64Op:$vrz, (fpextend FPR32Op:$vrx))], (outs FPR64Op:$vrz),
 281:                         (ins FPR32Op:$vrx)>;
 282: }
 283: 
 284: // fsel
 285: defm f2FSEL: F2_CXYZ_T<0b111001, "fsel">;
 286: 
 287: def f2FINS: F2_XZ_SET<0b00000, FPR32Op, 0b011011, "fins.32">;
 288: 
```

- EN: This range defines declarative TableGen records such as f2FDTOS, f2FSTOD, f2FSEL, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 f2FDTOS, f2FSTOD, f2FSEL 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 289-306

```tablegen
 289: def : Pat<(f32 fpimm16:$imm),(COPY_TO_REGCLASS (MOVI32 (fpimm32_lo16 fpimm16:$imm)), FPR32)>,
 290:         Requires<[HasFPUv3_SF]>;
 291: def : Pat<(f32 fpimm16_16:$imm), (COPY_TO_REGCLASS (MOVIH32 (fpimm32_hi16 fpimm16_16:$imm)), FPR32)>,
 292:         Requires<[HasFPUv3_SF]>;
 293: def : Pat<(f32 fpimm:$imm),(COPY_TO_REGCLASS (ORI32 (MOVIH32 (fpimm32_hi16 fpimm:$imm)), (fpimm32_lo16 fpimm:$imm)), FPR32)>,
 294:         Requires<[HasFPUv3_SF]>;
 295: 
 296: 
 297: multiclass BRCond_Bin_F2<CondCode CC, string Instr, Instruction Br0, Instruction Br1, Instruction MV, bit IsSelectSwap = 0> {
 298:   let Predicates = [HasFPUv3_SF] in
 299:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)), bb:$imm16),
 300:             (Br0 (!cast<Instruction>(Instr#_S) FPR32Op:$rs1, FPR32Op:$rs2), bb:$imm16)>;
 301:   let Predicates = [HasFPUv3_SF] in
 302:   def : Pat<(brcond (xor (i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)), 1), bb:$imm16),
 303:             (Br1 (!cast<Instruction>(Instr#_S) FPR32Op:$rs1, FPR32Op:$rs2), bb:$imm16)>;
 304:   let Predicates = [HasFPUv3_DF] in
 305:   def : Pat<(brcond (i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)), bb:$imm16),
 306:             (Br0 (!cast<Instruction>(Instr#_D) FPR64Op:$rs1, FPR64Op:$rs2), bb:$imm16)>;
```

- EN: This range defines declarative TableGen records such as BRCond_Bin_F2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BRCond_Bin_F2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 307-324

```tablegen
 307:   let Predicates = [HasFPUv3_DF] in
 308:   def : Pat<(brcond (xor (i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)), 1), bb:$imm16),
 309:             (Br1 (!cast<Instruction>(Instr#_D) FPR64Op:$rs1, FPR64Op:$rs2), bb:$imm16)>;
 310: 
 311:   let Predicates = [HasFPUv3_SF] in
 312:   def : Pat<(i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)),
 313:             (MV (!cast<Instruction>(Instr#_S) FPR32Op:$rs1, FPR32Op:$rs2))>;
 314:   let Predicates = [HasFPUv3_DF] in
 315:   def : Pat<(i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)),
 316:             (MV (!cast<Instruction>(Instr#_D) FPR64Op:$rs1, FPR64Op:$rs2))>;
 317: 
 318:   let Predicates = [HasFPUv3_SF] in {
 319:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)), FPR32Op:$rx, FPR32Op:$false),
 320:             !if(
 321:                 !eq(IsSelectSwap, 0),
 322:                 (f2FSEL_S (!cast<Instruction>(Instr#_S) FPR32Op:$rs1, FPR32Op:$rs2), FPR32Op:$rx, FPR32Op:$false),
 323:                 (f2FSEL_S (!cast<Instruction>(Instr#_S) FPR32Op:$rs1, FPR32Op:$rs2), FPR32Op:$false, FPR32Op:$rx)
 324:                )>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 325-342

```tablegen
 325:   }
 326:   let Predicates = [HasFPUv3_DF] in {
 327:   def : Pat<(select (i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)), FPR64Op:$rx, FPR64Op:$false),
 328:             !if(
 329:                 !eq(IsSelectSwap, 0),
 330:                 (f2FSEL_D (!cast<Instruction>(Instr#_D) FPR64Op:$rs1, FPR64Op:$rs2), FPR64Op:$rx, FPR64Op:$false),
 331:                 (f2FSEL_D (!cast<Instruction>(Instr#_D) FPR64Op:$rs1, FPR64Op:$rs2), FPR64Op:$false, FPR64Op:$rx)
 332:                )>;
 333:   }
 334: }
 335: 
 336: multiclass BRCond_Bin_SWAP_F2<CondCode CC, string Instr, Instruction Br0, Instruction Br1, Instruction MV, bit IsSelectSwap = 0> {
 337:   let Predicates = [HasFPUv3_SF] in
 338:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)), bb:$imm16),
 339:             (Br0 (!cast<Instruction>(Instr#_S) FPR32Op:$rs2, FPR32Op:$rs1), bb:$imm16)>;
 340:   let Predicates = [HasFPUv3_SF] in
 341:   def : Pat<(brcond (xor (i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)), 1), bb:$imm16),
 342:             (Br1 (!cast<Instruction>(Instr#_S) FPR32Op:$rs2, FPR32Op:$rs1), bb:$imm16)>;
```

- EN: This range defines declarative TableGen records such as BRCond_Bin_SWAP_F2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BRCond_Bin_SWAP_F2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 343-360

```tablegen
 343:   let Predicates = [HasFPUv3_DF] in
 344:   def : Pat<(brcond (i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)), bb:$imm16),
 345:             (Br0 (!cast<Instruction>(Instr#_D) FPR64Op:$rs2, FPR64Op:$rs1), bb:$imm16)>;
 346:   let Predicates = [HasFPUv3_DF] in
 347:   def : Pat<(brcond (xor (i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)), 1), bb:$imm16),
 348:             (Br1 (!cast<Instruction>(Instr#_D) FPR64Op:$rs2, FPR64Op:$rs1), bb:$imm16)>;
 349: 
 350:   let Predicates = [HasFPUv3_SF] in
 351:   def : Pat<(i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)),
 352:             (MV (!cast<Instruction>(Instr#_S) FPR32Op:$rs2, FPR32Op:$rs1))>;
 353:   let Predicates = [HasFPUv3_DF] in
 354:   def : Pat<(i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)),
 355:             (MV (!cast<Instruction>(Instr#_D) FPR64Op:$rs2, FPR64Op:$rs1))>;
 356: 
 357:   let Predicates = [HasFPUv3_SF] in {
 358:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, FPR32Op:$rs2, CC)), FPR32Op:$rx, FPR32Op:$false),
 359:             !if(
 360:                 !eq(IsSelectSwap, 0),
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 361-378

```tablegen
 361:                 (f2FSEL_S (!cast<Instruction>(Instr#_S) FPR32Op:$rs2, FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false),
 362:                 (f2FSEL_S (!cast<Instruction>(Instr#_S) FPR32Op:$rs2, FPR32Op:$rs1), FPR32Op:$false, FPR32Op:$rx)
 363:                )>;
 364:   }
 365:   let Predicates = [HasFPUv3_DF] in {
 366:   def : Pat<(select (i32 (setcc FPR64Op:$rs1, FPR64Op:$rs2, CC)), FPR64Op:$rx, FPR64Op:$false),
 367:             !if(
 368:                 !eq(IsSelectSwap, 0),
 369:                 (f2FSEL_D (!cast<Instruction>(Instr#_D) FPR64Op:$rs2, FPR64Op:$rs1), FPR64Op:$rx, FPR64Op:$false),
 370:                 (f2FSEL_D (!cast<Instruction>(Instr#_D) FPR64Op:$rs2, FPR64Op:$rs1), FPR64Op:$false, FPR64Op:$rx)
 371:                )>;
 372:   }
 373: }
 374: 
 375: // inverse (order && compare) to (unorder || inverse(compare))
 376: 
 377: defm : BRCond_Bin_F2<SETUNE, "f2FCMPNE", BT32, BF32, MVC32>;
 378: defm : BRCond_Bin_F2<SETOEQ, "f2FCMPNE", BF32, BT32, MVCV32, 1>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 379-396

```tablegen
 379: defm : BRCond_Bin_F2<SETOGE, "f2FCMPHS", BT32, BF32, MVC32>;
 380: defm : BRCond_Bin_F2<SETOLT, "f2FCMPLT", BT32, BF32, MVC32>;
 381: defm : BRCond_Bin_F2<SETUO, "f2FCMPUO", BT32, BF32, MVC32>;
 382: defm : BRCond_Bin_F2<SETO, "f2FCMPUO", BF32, BT32, MVCV32, 1>;
 383: defm : BRCond_Bin_SWAP_F2<SETOGT, "f2FCMPLT", BT32, BF32, MVC32>;
 384: defm : BRCond_Bin_SWAP_F2<SETOLE, "f2FCMPHS", BT32, BF32, MVC32>;
 385: 
 386: defm : BRCond_Bin_F2<SETNE, "f2FCMPNE", BT32, BF32, MVC32>;
 387: defm : BRCond_Bin_F2<SETEQ, "f2FCMPNE", BF32, BT32, MVCV32, 1>;
 388: defm : BRCond_Bin_F2<SETGE, "f2FCMPHS", BT32, BF32, MVC32>;
 389: defm : BRCond_Bin_F2<SETLT, "f2FCMPLT", BT32, BF32, MVC32>;
 390: defm : BRCond_Bin_SWAP_F2<SETGT, "f2FCMPLT", BT32, BF32, MVC32>;
 391: defm : BRCond_Bin_SWAP_F2<SETLE, "f2FCMPHS", BT32, BF32, MVC32>;
 392: 
 393: // ------
 394: 
 395: let Predicates = [HasFPUv3_SF] in {
 396:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETOGE)), bb:$imm16),
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 397-414

```tablegen
 397:             (BT32 (f2FCMPHSZ_S FPR32Op:$rs1), bb:$imm16)>;
 398:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETOGE)),
 399:             (MVC32 (f2FCMPHSZ_S FPR32Op:$rs1))>;
 400:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETOGE)), FPR32Op:$rx, FPR32Op:$false),
 401:             (f2FSEL_S (f2FCMPHSZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 402:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETOLT)), bb:$imm16),
 403:             (BT32 (f2FCMPLTZ_S FPR32Op:$rs1), bb:$imm16)>;
 404:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETOLT)),
 405:             (MVC32 (f2FCMPLTZ_S FPR32Op:$rs1))>;
 406:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETOLT)), FPR32Op:$rx, FPR32Op:$false),
 407:             (f2FSEL_S (f2FCMPLTZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 408:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETOLE)), bb:$imm16),
 409:             (BT32 (f2FCMPLSZ_S FPR32Op:$rs1), bb:$imm16)>;
 410:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETOLE)),
 411:             (MVC32 (f2FCMPLSZ_S FPR32Op:$rs1))>;
 412:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETOLE)), FPR32Op:$rx, FPR32Op:$false),
 413:             (f2FSEL_S (f2FCMPLSZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 414:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETOGT)), bb:$imm16),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 415-432

```tablegen
 415:             (BT32 (f2FCMPHZ_S FPR32Op:$rs1), bb:$imm16)>;
 416:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETOGT)),
 417:             (MVC32 (f2FCMPHZ_S FPR32Op:$rs1))>;
 418:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETOGT)), FPR32Op:$rx, FPR32Op:$false),
 419:             (f2FSEL_S (f2FCMPHZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 420:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETUNE)), bb:$imm16),
 421:             (BT32 (f2FCMPNEZ_S FPR32Op:$rs1), bb:$imm16)>;
 422:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETUNE)),
 423:             (MVC32 (f2FCMPNEZ_S FPR32Op:$rs1))>;
 424:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETUNE)), FPR32Op:$rx, FPR32Op:$false),
 425:             (f2FSEL_S (f2FCMPNEZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 426:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm, SETUO)), bb:$imm16),
 427:             (BT32 (f2FCMPUOZ_S FPR32Op:$rs1), bb:$imm16)>;
 428:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm, SETUO)),
 429:             (MVC32 (f2FCMPUOZ_S FPR32Op:$rs1))>;
 430:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm, SETUO)), FPR32Op:$rx, FPR32Op:$false),
 431:             (f2FSEL_S (f2FCMPUOZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 432:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETGE)), bb:$imm16),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 433-450

```tablegen
 433:             (BT32 (f2FCMPHSZ_S FPR32Op:$rs1), bb:$imm16)>;
 434:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETGE)),
 435:             (MVC32 (f2FCMPHSZ_S FPR32Op:$rs1))>;
 436:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETGE)), FPR32Op:$rx, FPR32Op:$false),
 437:             (f2FSEL_S (f2FCMPHSZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 438:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETLT)), bb:$imm16),
 439:             (BT32 (f2FCMPLTZ_S FPR32Op:$rs1), bb:$imm16)>;
 440:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETLT)),
 441:             (MVC32 (f2FCMPLTZ_S FPR32Op:$rs1))>;
 442:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETLT)), FPR32Op:$rx, FPR32Op:$false),
 443:             (f2FSEL_S (f2FCMPLTZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 444:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETLE)), bb:$imm16),
 445:             (BT32 (f2FCMPLSZ_S FPR32Op:$rs1), bb:$imm16)>;
 446:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETLE)),
 447:             (MVC32 (f2FCMPLSZ_S FPR32Op:$rs1))>;
 448:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETLE)), FPR32Op:$rx, FPR32Op:$false),
 449:             (f2FSEL_S (f2FCMPLSZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 450:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETGT)), bb:$imm16),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 451-468

```tablegen
 451:             (BT32 (f2FCMPHZ_S FPR32Op:$rs1), bb:$imm16)>;
 452:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETGT)),
 453:             (MVC32 (f2FCMPHZ_S FPR32Op:$rs1))>;
 454:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETGT)), FPR32Op:$rx, FPR32Op:$false),
 455:             (f2FSEL_S (f2FCMPHZ_S FPR32Op:$rs1), FPR32Op:$rx, FPR32Op:$false)>;
 456: 
 457: 
 458:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm, SETO)), bb:$imm16),
 459:             (BF32 (f2FCMPUOZ_S FPR32Op:$rs1), bb:$imm16)>;
 460:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm, SETO)),
 461:             (MVCV32 (f2FCMPUOZ_S FPR32Op:$rs1))>;
 462:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm, SETO)), FPR32Op:$rx, FPR32Op:$false),
 463:             (f2FSEL_S (f2FCMPUOZ_S FPR32Op:$rs1), FPR32Op:$false, FPR32Op:$rx)>;
 464:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETOEQ)), bb:$imm16),
 465:             (BF32 (f2FCMPNEZ_S FPR32Op:$rs1), bb:$imm16)>;
 466:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETOEQ)),
 467:             (MVCV32 (f2FCMPNEZ_S FPR32Op:$rs1))>;
 468:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETOEQ)), FPR32Op:$rx, FPR32Op:$false),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 469-484

```tablegen
 469:             (f2FSEL_S (f2FCMPNEZ_S FPR32Op:$rs1), FPR32Op:$false, FPR32Op:$rx)>;
 470:   def : Pat<(brcond (i32 (setcc FPR32Op:$rs1, fpimm0, SETEQ)), bb:$imm16),
 471:             (BF32 (f2FCMPNEZ_S FPR32Op:$rs1), bb:$imm16)>;
 472:   def : Pat<(i32 (setcc FPR32Op:$rs1, fpimm0, SETEQ)),
 473:             (MVCV32 (f2FCMPNEZ_S FPR32Op:$rs1))>;
 474:   def : Pat<(select (i32 (setcc FPR32Op:$rs1, fpimm0, SETEQ)), FPR32Op:$rx, FPR32Op:$false),
 475:             (f2FSEL_S (f2FCMPNEZ_S FPR32Op:$rs1), FPR32Op:$false, FPR32Op:$rx)>;
 476: }
 477: 
 478: 
 479: let Predicates = [HasFPUv3_SF] in
 480: def : Pat<(select CARRY:$ca, FPR32Op:$rx, FPR32Op:$false),
 481:           (f2FSEL_S CARRY:$ca, FPR32Op:$rx, FPR32Op:$false)>;
 482: let Predicates = [HasFPUv3_DF] in
 483: def : Pat<(select CARRY:$ca, FPR64Op:$rx, FPR64Op:$false),
 484:           (f2FSEL_D CARRY:$ca, FPR64Op:$rx, FPR64Op:$false)>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYInstrFormatsF2.td`
- LLVM subsystems / LLVM 子系统: TableGen
