# LoongArchFloat32InstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchFloat32InstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: // LoongArchFloat32InstrInfo.td - Single-Precision Float instr --*- tablegen -*-
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the basic single-precision floating-point instructions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: def NotBoolXor : PatFrags<(ops node:$val),
  14:                           [(xor node:$val, -1), (xor node:$val, 1)]>;
  15: 
  16: //===----------------------------------------------------------------------===//
  17: // LoongArch specific DAG Nodes.
  18: //===----------------------------------------------------------------------===//
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `NotBoolXor`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `NotBoolXor` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: 
  20: def SDT_LoongArchMOVGR2FR_W
  21:     : SDTypeProfile<1, 1, [SDTCisVT<0, f32>, SDTCisVT<1, i32>]>;
  22: def SDT_LoongArchMOVGR2FR_W_LA64
  23:     : SDTypeProfile<1, 1, [SDTCisVT<0, f32>, SDTCisVT<1, i64>]>;
  24: def SDT_LoongArchMOVFR2GR_S_LA64
  25:     : SDTypeProfile<1, 1, [SDTCisVT<0, i64>, SDTCisVT<1, f32>]>;
  26: def SDT_LoongArchFTINT : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisFP<1>]>;
  27: def SDT_LoongArchFRECIPE : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisFP<1>]>;
  28: def SDT_LoongArchFRSQRTE : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisFP<1>]>;
  29: def SDT_LoongArchITOF : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisSameAs<0, 1>]>;
  30: 
  31: // ISD::BRCOND is custom-lowered to LoongArchISD::BRCOND for floating-point
  32: // comparisons to prevent recursive lowering.
  33: def loongarch_brcond : SDNode<"LoongArchISD::BRCOND", SDTBrcond, [SDNPHasChain]>;
  34: 
  35: // FPR<->GPR transfer operations
  36: def loongarch_movgr2fr_w
```
- **EN**: This block declares or refines TableGen records such as `SDT_LoongArchMOVGR2FR_W`, `SDT_LoongArchMOVGR2FR_W_LA64`, `SDT_LoongArchMOVFR2GR_S_LA64`, `SDT_LoongArchFTINT`, `SDT_LoongArchFRECIPE`, `SDT_LoongArchFRSQRTE`.
- **CN**: 该代码块声明或细化了 `SDT_LoongArchMOVGR2FR_W`, `SDT_LoongArchMOVGR2FR_W_LA64`, `SDT_LoongArchMOVFR2GR_S_LA64`, `SDT_LoongArchFTINT`, `SDT_LoongArchFRECIPE`, `SDT_LoongArchFRSQRTE` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:     : SDNode<"LoongArchISD::MOVGR2FR_W", SDT_LoongArchMOVGR2FR_W>;
  38: def loongarch_movgr2fr_w_la64
  39:     : SDNode<"LoongArchISD::MOVGR2FR_W_LA64", SDT_LoongArchMOVGR2FR_W_LA64>;
  40: def loongarch_movfr2gr_s_la64
  41:     : SDNode<"LoongArchISD::MOVFR2GR_S_LA64", SDT_LoongArchMOVFR2GR_S_LA64>;
  42: 
  43: def loongarch_ftint : SDNode<"LoongArchISD::FTINT", SDT_LoongArchFTINT>;
  44: 
  45: // Floating point approximate reciprocal operation
  46: def loongarch_frecipe : SDNode<"LoongArchISD::FRECIPE", SDT_LoongArchFRECIPE>;
  47: def loongarch_frsqrte : SDNode<"LoongArchISD::FRSQRTE", SDT_LoongArchFRSQRTE>;
  48: def loongarch_sitof : SDNode<"LoongArchISD::SITOF", SDT_LoongArchITOF>;
  49: 
  50: //===----------------------------------------------------------------------===//
  51: // Instructions
  52: //===----------------------------------------------------------------------===//
  53: 
  54: let Predicates = [HasBasicF] in {
```
- **EN**: This block declares or refines TableGen records such as `loongarch_movgr2fr_w_la64`, `loongarch_movfr2gr_s_la64`, `loongarch_ftint`, `loongarch_frecipe`, `loongarch_frsqrte`, `loongarch_sitof`.
- **CN**: 该代码块声明或细化了 `loongarch_movgr2fr_w_la64`, `loongarch_movfr2gr_s_la64`, `loongarch_ftint`, `loongarch_frecipe`, `loongarch_frsqrte`, `loongarch_sitof` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: 
  56: // Arithmetic Operation Instructions
  57: def FADD_S : FP_ALU_3R<0x01008000>;
  58: def FSUB_S : FP_ALU_3R<0x01028000>;
  59: def FMUL_S : FP_ALU_3R<0x01048000>;
  60: def FDIV_S : FP_ALU_3R<0x01068000>;
  61: def FMADD_S  : FP_ALU_4R<0x08100000>;
  62: def FMSUB_S  : FP_ALU_4R<0x08500000>;
  63: def FNMADD_S : FP_ALU_4R<0x08900000>;
  64: def FNMSUB_S : FP_ALU_4R<0x08d00000>;
  65: def FMAX_S  : FP_ALU_3R<0x01088000>;
  66: def FMIN_S  : FP_ALU_3R<0x010a8000>;
  67: def FMAXA_S : FP_ALU_3R<0x010c8000>;
  68: def FMINA_S : FP_ALU_3R<0x010e8000>;
  69: def FABS_S   : FP_ALU_2R<0x01140400>;
  70: def FNEG_S   : FP_ALU_2R<0x01141400>;
  71: def FSQRT_S  : FP_ALU_2R<0x01144400>;
  72: def FRECIP_S : FP_ALU_2R<0x01145400>;
```
- **EN**: This block declares or refines TableGen records such as `FADD_S`, `FSUB_S`, `FMUL_S`, `FDIV_S`, `FMADD_S`, `FMSUB_S`.
- **CN**: 该代码块声明或细化了 `FADD_S`, `FSUB_S`, `FMUL_S`, `FDIV_S`, `FMADD_S`, `FMSUB_S` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73: def FRSQRT_S : FP_ALU_2R<0x01146400>;
  74: def FRECIPE_S : FP_ALU_2R<0x01147400>;
  75: def FRSQRTE_S : FP_ALU_2R<0x01148400>;
  76: def FSCALEB_S : FP_ALU_3R<0x01108000>;
  77: def FLOGB_S   : FP_ALU_2R<0x01142400>;
  78: def FCOPYSIGN_S : FP_ALU_3R<0x01128000>;
  79: def FCLASS_S  : FP_ALU_2R<0x01143400>;
  80: 
  81: 
  82: // Comparison Instructions
  83: def FCMP_CAF_S  : FP_CMP<0x0c100000>;
  84: def FCMP_CUN_S  : FP_CMP<0x0c140000>;
  85: def FCMP_CEQ_S  : FP_CMP<0x0c120000>;
  86: def FCMP_CUEQ_S : FP_CMP<0x0c160000>;
  87: def FCMP_CLT_S  : FP_CMP<0x0c110000>;
  88: def FCMP_CULT_S : FP_CMP<0x0c150000>;
  89: def FCMP_CLE_S  : FP_CMP<0x0c130000>;
  90: def FCMP_CULE_S : FP_CMP<0x0c170000>;
```
- **EN**: This block declares or refines TableGen records such as `FRSQRT_S`, `FRECIPE_S`, `FRSQRTE_S`, `FSCALEB_S`, `FLOGB_S`, `FCOPYSIGN_S`.
- **CN**: 该代码块声明或细化了 `FRSQRT_S`, `FRECIPE_S`, `FRSQRTE_S`, `FSCALEB_S`, `FLOGB_S`, `FCOPYSIGN_S` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: def FCMP_CNE_S  : FP_CMP<0x0c180000>;
  92: def FCMP_COR_S  : FP_CMP<0x0c1a0000>;
  93: def FCMP_CUNE_S : FP_CMP<0x0c1c0000>;
  94: def FCMP_SAF_S  : FP_CMP<0x0c108000>;
  95: def FCMP_SUN_S  : FP_CMP<0x0c148000>;
  96: def FCMP_SEQ_S  : FP_CMP<0x0c128000>;
  97: def FCMP_SUEQ_S : FP_CMP<0x0c168000>;
  98: def FCMP_SLT_S  : FP_CMP<0x0c118000>;
  99: def FCMP_SULT_S : FP_CMP<0x0c158000>;
 100: def FCMP_SLE_S  : FP_CMP<0x0c138000>;
 101: def FCMP_SULE_S : FP_CMP<0x0c178000>;
 102: def FCMP_SNE_S  : FP_CMP<0x0c188000>;
 103: def FCMP_SOR_S  : FP_CMP<0x0c1a8000>;
 104: def FCMP_SUNE_S : FP_CMP<0x0c1c8000>;
 105: 
 106: // Conversion Instructions
 107: def FFINT_S_W    : FP_CONV<0x011d1000>;
 108: def FTINT_W_S    : FP_CONV<0x011b0400>;
```
- **EN**: This block declares or refines TableGen records such as `FCMP_CNE_S`, `FCMP_COR_S`, `FCMP_CUNE_S`, `FCMP_SAF_S`, `FCMP_SUN_S`, `FCMP_SEQ_S`.
- **CN**: 该代码块声明或细化了 `FCMP_CNE_S`, `FCMP_COR_S`, `FCMP_CUNE_S`, `FCMP_SAF_S`, `FCMP_SUN_S`, `FCMP_SEQ_S` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: def FTINTRM_W_S  : FP_CONV<0x011a0400>;
 110: def FTINTRP_W_S  : FP_CONV<0x011a4400>;
 111: def FTINTRZ_W_S  : FP_CONV<0x011a8400>;
 112: def FTINTRNE_W_S : FP_CONV<0x011ac400>;
 113: def FRINT_S      : FP_CONV<0x011e4400>;
 114: 
 115: // Move Instructions
 116: def FSEL_xS    : FP_SEL<0x0d000000>;
 117: def FMOV_S     : FP_MOV<0x01149400>;
 118: def MOVGR2FR_W : FP_MOV<0x0114a400, FPR32, GPR>;
 119: def MOVFR2GR_S : FP_MOV<0x0114b400, GPR, FPR32>;
 120: let hasSideEffects = 1 in {
 121: def MOVGR2FCSR : FP_MOV<0x0114c000, FCSR, GPR>;
 122: def MOVFCSR2GR : FP_MOV<0x0114c800, GPR, FCSR>;
 123: } // hasSideEffects = 1
 124: def MOVFR2CF_xS : FP_MOV<0x0114d000, CFR, FPR32>;
 125: def MOVCF2FR_xS : FP_MOV<0x0114d400, FPR32, CFR>;
 126: def MOVGR2CF    : FP_MOV<0x0114d800, CFR, GPR>;
```
- **EN**: This block declares or refines TableGen records such as `FTINTRM_W_S`, `FTINTRP_W_S`, `FTINTRZ_W_S`, `FTINTRNE_W_S`, `FRINT_S`, `FSEL_xS`.
- **CN**: 该代码块声明或细化了 `FTINTRM_W_S`, `FTINTRP_W_S`, `FTINTRZ_W_S`, `FTINTRNE_W_S`, `FRINT_S`, `FSEL_xS` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127: def MOVCF2GR    : FP_MOV<0x0114dc00, GPR, CFR>;
 128: 
 129: // Branch Instructions
 130: def BCEQZ : FP_BRANCH<0x48000000>;
 131: def BCNEZ : FP_BRANCH<0x48000100>;
 132: 
 133: // Common Memory Access Instructions
 134: def FLD_S : FP_LOAD_2RI12<0x2b000000>;
 135: def FST_S : FP_STORE_2RI12<0x2b400000>;
 136: def FLDX_S : FP_LOAD_3R<0x38300000>;
 137: def FSTX_S : FP_STORE_3R<0x38380000>;
 138: 
 139: // Bound Check Memory Access Instructions
 140: def FLDGT_S : FP_LOAD_3R<0x38740000>;
 141: def FLDLE_S : FP_LOAD_3R<0x38750000>;
 142: def FSTGT_S : FP_STORE_3R<0x38760000>;
 143: def FSTLE_S : FP_STORE_3R<0x38770000>;
 144: 
```
- **EN**: This block declares or refines TableGen records such as `MOVCF2GR`, `BCEQZ`, `BCNEZ`, `FLD_S`, `FST_S`, `FLDX_S`.
- **CN**: 该代码块声明或细化了 `MOVCF2GR`, `BCEQZ`, `BCNEZ`, `FLD_S`, `FST_S`, `FLDX_S` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: // Pseudo instructions for spill/reload CFRs.
 146: let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
 147: def PseudoST_CFR : Pseudo<(outs),
 148:                           (ins CFR:$ccd, GPR:$rj, grlenimm:$imm)>;
 149: let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
 150: def PseudoLD_CFR : Pseudo<(outs CFR:$ccd),
 151:                           (ins GPR:$rj, grlenimm:$imm)>;
 152: 
 153: // SET_CFR_{FALSE,TRUE}
 154: // These instructions are defined in order to avoid expensive check error if
 155: // regular instruction patterns are used.
 156: // fcmp.caf.s $dst, $fa0, $fa0
 157: def SET_CFR_FALSE : SET_CFR<0x0c100000, "fcmp.caf.s">;
 158: // fcmp.cueq.s $dst, $fa0, $fa0
 159: def SET_CFR_TRUE  : SET_CFR<0x0c160000, "fcmp.cueq.s">;
 160: 
 161: // Pseudo instruction for copying CFRs.
 162: def PseudoCopyCFR : Pseudo<(outs CFR:$dst), (ins CFR:$src)> {
```
- **EN**: This block declares or refines TableGen records such as `PseudoST_CFR`, `PseudoLD_CFR`, `SET_CFR_FALSE`, `SET_CFR_TRUE`, `PseudoCopyCFR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `PseudoST_CFR`, `PseudoLD_CFR`, `SET_CFR_FALSE`, `SET_CFR_TRUE`, `PseudoCopyCFR` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:   let mayLoad = 0;
 164:   let mayStore = 0;
 165:   let hasSideEffects = 0;
 166:   let Size = 12;
 167: }
 168: 
 169: } // Predicates = [HasBasicF]
 170: 
 171: //===----------------------------------------------------------------------===//
 172: // Pseudo-instructions and codegen patterns
 173: //===----------------------------------------------------------------------===//
 174: 
 175: /// Generic pattern classes
 176: 
 177: class PatFpr<SDPatternOperator OpNode, LAInst Inst, RegisterClass RegTy>
 178:     : Pat<(OpNode RegTy:$fj), (Inst $fj)>;
 179: class PatFprFpr<SDPatternOperator OpNode, LAInst Inst, RegisterClass RegTy>
 180:     : Pat<(OpNode RegTy:$fj, RegTy:$fk), (Inst $fj, $fk)>;
```
- **EN**: This block declares or refines TableGen records such as `PatFpr`, `PatFprFpr`.
- **CN**: 该代码块声明或细化了 `PatFpr`, `PatFprFpr` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: 
 182: let Predicates = [HasBasicF] in {
 183: 
 184: /// Float arithmetic operations
 185: 
 186: def : PatFprFpr<fadd, FADD_S, FPR32>;
 187: def : PatFprFpr<fsub, FSUB_S, FPR32>;
 188: def : PatFprFpr<fmul, FMUL_S, FPR32>;
 189: def : PatFprFpr<fdiv, FDIV_S, FPR32>;
 190: def : PatFprFpr<fcopysign, FCOPYSIGN_S, FPR32>;
 191: def : PatFprFpr<fmaxnum_ieee, FMAX_S, FPR32>;
 192: def : PatFprFpr<fmaxnum, FMAX_S, FPR32>;
 193: def : PatFprFpr<fminnum_ieee, FMIN_S, FPR32>;
 194: def : PatFprFpr<fminnum, FMIN_S, FPR32>;
 195: def : PatFpr<fneg, FNEG_S, FPR32>;
 196: def : PatFpr<fabs, FABS_S, FPR32>;
 197: def : PatFpr<fsqrt, FSQRT_S, FPR32>;
 198: def : Pat<(fdiv fpimm1, (fsqrt FPR32:$fj)), (FRSQRT_S FPR32:$fj)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: let Predicates = [HasBasicF, IsLA64] in {
 200: def : Pat<(fdiv (loongarch_movgr2fr_w_la64 (i64 1065353216)), (fsqrt FPR32:$fj)),
 201:           (FRSQRT_S FPR32:$fj)>;
 202: } // Predicates = [HasBasicF, IsLA64]
 203: let Predicates = [HasBasicF, IsLA32] in {
 204: def : Pat<(fdiv (loongarch_movgr2fr_w (i32 1065353216)), (fsqrt FPR32:$fj)),
 205:           (FRSQRT_S FPR32:$fj)>;
 206: } // Predicates = [HasBasicF, IsLA32]
 207: def : Pat<(fcanonicalize FPR32:$fj), (FMAX_S $fj, $fj)>;
 208: def : Pat<(is_fpclass FPR32:$fj, (i32 timm:$mask)),
 209:           (SLTU R0, (ANDI (MOVFR2GR_S (FCLASS_S FPR32:$fj)),
 210:                           (to_fclass_mask timm:$mask)))>;
 211: 
 212: /// Setcc
 213: 
 214: // Match non-signaling comparison
 215: 
 216: class PatFPSetcc<CondCode cc, LAInst CmpInst, RegisterClass RegTy>
```
- **EN**: This block declares or refines TableGen records such as `PatFPSetcc`.
- **CN**: 该代码块声明或细化了 `PatFPSetcc` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217:     : Pat<(any_fsetcc RegTy:$fj, RegTy:$fk, cc),
 218:           (CmpInst RegTy:$fj, RegTy:$fk)>;
 219: // SETOGT/SETOGE/SETUGT/SETUGE/SETGE/SETNE/SETGT will expand into
 220: // SETOLT/SETOLE/SETULT/SETULE/SETLE/SETEQ/SETLT.
 221: def : PatFPSetcc<SETOEQ, FCMP_CEQ_S,  FPR32>;
 222: def : PatFPSetcc<SETEQ,  FCMP_CEQ_S,  FPR32>;
 223: def : PatFPSetcc<SETOLT, FCMP_CLT_S,  FPR32>;
 224: def : PatFPSetcc<SETOLE, FCMP_CLE_S,  FPR32>;
 225: def : PatFPSetcc<SETLE,  FCMP_CLE_S,  FPR32>;
 226: def : PatFPSetcc<SETONE, FCMP_CNE_S,  FPR32>;
 227: def : PatFPSetcc<SETO,   FCMP_COR_S,  FPR32>;
 228: def : PatFPSetcc<SETUEQ, FCMP_CUEQ_S, FPR32>;
 229: def : PatFPSetcc<SETULT, FCMP_CULT_S, FPR32>;
 230: def : PatFPSetcc<SETULE, FCMP_CULE_S, FPR32>;
 231: def : PatFPSetcc<SETUNE, FCMP_CUNE_S, FPR32>;
 232: def : PatFPSetcc<SETUO,  FCMP_CUN_S,  FPR32>;
 233: def : PatFPSetcc<SETLT,  FCMP_CLT_S,  FPR32>;
 234: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```tablegen
 235: multiclass PatFPBrcond<CondCode cc, LAInst CmpInst, RegisterClass RegTy> {
 236:   def : Pat<(loongarch_brcond (NotBoolXor (GRLenVT (setcc RegTy:$fj, RegTy:$fk, cc))),
 237:                               bb:$imm21),
 238:             (BCEQZ (CmpInst RegTy:$fj, RegTy:$fk), bb:$imm21)>;
 239:   def : Pat<(loongarch_brcond (GRLenVT (setcc RegTy:$fj, RegTy:$fk, cc)), bb:$imm21),
 240:             (BCNEZ (CmpInst RegTy:$fj, RegTy:$fk), bb:$imm21)>;
 241: }
 242: 
 243: defm : PatFPBrcond<SETOEQ, FCMP_CEQ_S, FPR32>;
 244: defm : PatFPBrcond<SETEQ , FCMP_CEQ_S, FPR32>;
 245: defm : PatFPBrcond<SETOLT, FCMP_CLT_S, FPR32>;
 246: defm : PatFPBrcond<SETOLE, FCMP_CLE_S, FPR32>;
 247: defm : PatFPBrcond<SETLE,  FCMP_CLE_S, FPR32>;
 248: defm : PatFPBrcond<SETONE, FCMP_CNE_S, FPR32>;
 249: defm : PatFPBrcond<SETO,   FCMP_COR_S, FPR32>;
 250: defm : PatFPBrcond<SETUEQ, FCMP_CUEQ_S, FPR32>;
 251: defm : PatFPBrcond<SETULT, FCMP_CULT_S, FPR32>;
 252: defm : PatFPBrcond<SETULE, FCMP_CULE_S, FPR32>;
```
- **EN**: This block declares or refines TableGen records such as `PatFPBrcond`.
- **CN**: 该代码块声明或细化了 `PatFPBrcond` 等 TableGen 记录。

### Lines 253-270 / 第 253-270 行
```tablegen
 253: defm : PatFPBrcond<SETUNE, FCMP_CUNE_S, FPR32>;
 254: defm : PatFPBrcond<SETUO,  FCMP_CUN_S, FPR32>;
 255: defm : PatFPBrcond<SETLT,  FCMP_CLT_S, FPR32>;
 256: 
 257: // Match signaling comparison
 258: 
 259: class PatStrictFsetccs<CondCode cc, LAInst CmpInst, RegisterClass RegTy>
 260:     : Pat<(strict_fsetccs RegTy:$fj, RegTy:$fk, cc),
 261:           (CmpInst RegTy:$fj, RegTy:$fk)>;
 262: def : PatStrictFsetccs<SETOEQ, FCMP_SEQ_S,  FPR32>;
 263: def : PatStrictFsetccs<SETOLT, FCMP_SLT_S,  FPR32>;
 264: def : PatStrictFsetccs<SETOLE, FCMP_SLE_S,  FPR32>;
 265: def : PatStrictFsetccs<SETONE, FCMP_SNE_S,  FPR32>;
 266: def : PatStrictFsetccs<SETO,   FCMP_SOR_S,  FPR32>;
 267: def : PatStrictFsetccs<SETUEQ, FCMP_SUEQ_S, FPR32>;
 268: def : PatStrictFsetccs<SETULT, FCMP_SULT_S, FPR32>;
 269: def : PatStrictFsetccs<SETULE, FCMP_SULE_S, FPR32>;
 270: def : PatStrictFsetccs<SETUNE, FCMP_SUNE_S, FPR32>;
```
- **EN**: This block declares or refines TableGen records such as `PatStrictFsetccs`.
- **CN**: 该代码块声明或细化了 `PatStrictFsetccs` 等 TableGen 记录。

### Lines 271-288 / 第 271-288 行
```tablegen
 271: def : PatStrictFsetccs<SETUO,  FCMP_SUN_S,  FPR32>;
 272: def : PatStrictFsetccs<SETLT,  FCMP_SLT_S,  FPR32>;
 273: 
 274: /// Select
 275: 
 276: def : Pat<(select CFR:$cc, FPR32:$fk, FPR32:$fj),
 277:           (FSEL_xS FPR32:$fj, FPR32:$fk, CFR:$cc)>;
 278: 
 279: /// Selectcc
 280: 
 281: class PatFPSelectcc<CondCode cc, LAInst CmpInst, LAInst SelInst,
 282:                     RegisterClass RegTy>
 283:     : Pat<(select (GRLenVT (setcc RegTy:$a, RegTy:$b, cc)), RegTy:$t, RegTy:$f),
 284:           (SelInst RegTy:$f, RegTy:$t, (CmpInst RegTy:$a, RegTy:$b))>;
 285: def : PatFPSelectcc<SETOEQ, FCMP_CEQ_S,  FSEL_xS, FPR32>;
 286: def : PatFPSelectcc<SETOLT, FCMP_CLT_S,  FSEL_xS, FPR32>;
 287: def : PatFPSelectcc<SETOLE, FCMP_CLE_S,  FSEL_xS, FPR32>;
 288: def : PatFPSelectcc<SETONE, FCMP_CNE_S,  FSEL_xS, FPR32>;
```
- **EN**: This block declares or refines TableGen records such as `PatFPSelectcc`.
- **CN**: 该代码块声明或细化了 `PatFPSelectcc` 等 TableGen 记录。

### Lines 289-306 / 第 289-306 行
```tablegen
 289: def : PatFPSelectcc<SETO,   FCMP_COR_S,  FSEL_xS, FPR32>;
 290: def : PatFPSelectcc<SETUEQ, FCMP_CUEQ_S, FSEL_xS, FPR32>;
 291: def : PatFPSelectcc<SETULT, FCMP_CULT_S, FSEL_xS, FPR32>;
 292: def : PatFPSelectcc<SETULE, FCMP_CULE_S, FSEL_xS, FPR32>;
 293: def : PatFPSelectcc<SETUNE, FCMP_CUNE_S, FSEL_xS, FPR32>;
 294: def : PatFPSelectcc<SETUO,  FCMP_CUN_S,  FSEL_xS, FPR32>;
 295: 
 296: /// Loads
 297: 
 298: defm : LdPat<load, FLD_S, f32>;
 299: def : RegRegLdPat<load, FLDX_S, f32>;
 300: 
 301: /// Stores
 302: 
 303: defm : StPat<store, FST_S, FPR32, f32>;
 304: def : RegRegStPat<store, FSTX_S, FPR32, f32>;
 305: 
 306: /// Floating point constants
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 307-324 / 第 307-324 行
```tablegen
 307: 
 308: def : Pat<(f32 fpimm0), (MOVGR2FR_W R0)>;
 309: def : Pat<(f32 fpimm0neg), (FNEG_S (MOVGR2FR_W R0))>;
 310: def : Pat<(f32 fpimm1), (FFINT_S_W (MOVGR2FR_W (ADDI_W R0, 1)))>;
 311: 
 312: // FP Conversion
 313: def : Pat<(loongarch_ftint FPR32:$src), (FTINTRZ_W_S FPR32:$src)>;
 314: 
 315: // FP reciprocal operation
 316: def : Pat<(fdiv fpimm1, FPR32:$src), (FRECIP_S $src)>;
 317: let Predicates = [HasBasicF, IsLA64] in {
 318: def : Pat<(fdiv (loongarch_movgr2fr_w_la64 (i64 1065353216)), FPR32:$src),
 319:           (FRECIP_S $src)>;
 320: } // Predicates = [HasBasicF, IsLA64]
 321: let Predicates = [HasBasicF, IsLA32] in {
 322: def : Pat<(fdiv (loongarch_movgr2fr_w (i32 1065353216)), FPR32:$src),
 323:           (FRECIP_S $src)>;
 324: } // Predicates = [HasBasicF, IsLA32]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 325-342 / 第 325-342 行
```tablegen
 325: 
 326: let Predicates = [HasFrecipe] in {
 327: // FP approximate reciprocal operation
 328: def : Pat<(int_loongarch_frecipe_s FPR32:$src), (FRECIPE_S FPR32:$src)>;
 329: def : Pat<(int_loongarch_frsqrte_s FPR32:$src), (FRSQRTE_S FPR32:$src)>;
 330: def : Pat<(loongarch_frecipe FPR32:$src), (FRECIPE_S FPR32:$src)>;
 331: def : Pat<(loongarch_frsqrte FPR32:$src), (FRSQRTE_S FPR32:$src)>;
 332: }
 333: 
 334: // fmadd.s: fj * fk + fa
 335: def : Pat<(fma FPR32:$fj, FPR32:$fk, FPR32:$fa), (FMADD_S $fj, $fk, $fa)>;
 336: 
 337: // fmsub.s: fj * fk - fa
 338: def : Pat<(fma FPR32:$fj, FPR32:$fk, (fneg FPR32:$fa)),
 339:           (FMSUB_S FPR32:$fj, FPR32:$fk, FPR32:$fa)>;
 340: 
 341: // fnmadd.s: -(fj * fk + fa)
 342: def : Pat<(fneg (fma FPR32:$fj, FPR32:$fk, FPR32:$fa)),
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 343-360 / 第 343-360 行
```tablegen
 343:           (FNMADD_S FPR32:$fj, FPR32:$fk, FPR32:$fa)>;
 344: 
 345: // fnmadd.s: -fj * fk - fa (the nsz flag on the FMA)
 346: def : Pat<(fma_nsz (fneg FPR32:$fj), FPR32:$fk, (fneg FPR32:$fa)),
 347:           (FNMADD_S FPR32:$fj, FPR32:$fk, FPR32:$fa)>;
 348: 
 349: // fnmsub.s: -(fj * fk - fa)
 350: def : Pat<(fneg (fma FPR32:$fj, FPR32:$fk, (fneg FPR32:$fa))),
 351:           (FNMSUB_S FPR32:$fj, FPR32:$fk, FPR32:$fa)>;
 352: 
 353: // fnmsub.s: -fj * fk + fa (the nsz flag on the FMA)
 354: def : Pat<(fma_nsz (fneg FPR32:$fj), FPR32:$fk, FPR32:$fa),
 355:           (FNMSUB_S FPR32:$fj, FPR32:$fk, FPR32:$fa)>;
 356: 
 357: // ffint.s.w
 358: def : Pat<(loongarch_sitof FPR32:$fj), (FFINT_S_W FPR32:$fj)>;
 359: } // Predicates = [HasBasicF]
 360: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-378 / 第 361-378 行
```tablegen
 361: let Predicates = [HasBasicF, IsLA64] in {
 362: // GPR -> FPR
 363: def : Pat<(loongarch_movgr2fr_w_la64 GPR:$src), (MOVGR2FR_W GPR:$src)>;
 364: // FPR -> GPR
 365: def : Pat<(loongarch_movfr2gr_s_la64 FPR32:$src),
 366:           (MOVFR2GR_S FPR32:$src)>;
 367: // int -> f32
 368: def : Pat<(f32 (sint_to_fp (i64 (sexti32 (i64 GPR:$src))))),
 369:           (FFINT_S_W (MOVGR2FR_W GPR:$src))>;
 370: // uint -> f32
 371: def : Pat<(f32 (uint_to_fp (i64 (sexti32 (i64 GPR:$src))))),
 372:           (FFINT_S_W (MOVGR2FR_W GPR:$src))>;
 373: } // Predicates = [HasBasicF, IsLA64]
 374: 
 375: // FP Rounding
 376: let Predicates = [HasBasicF, IsLA64] in {
 377: def : PatFpr<frint, FRINT_S, FPR32>;
 378: } // Predicates = [HasBasicF, IsLA64]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 379-388 / 第 379-388 行
```tablegen
 379: 
 380: let Predicates = [HasBasicF, IsLA32] in {
 381: // GPR -> FPR
 382: def : Pat<(bitconvert (i32 GPR:$src)), (MOVGR2FR_W GPR:$src)>;
 383: def : Pat<(loongarch_movgr2fr_w (i32 GPR:$src)), (MOVGR2FR_W GPR:$src)>;
 384: // FPR -> GPR
 385: def : Pat<(i32 (bitconvert FPR32:$src)), (MOVFR2GR_S FPR32:$src)>;
 386: // int -> f32
 387: def : Pat<(f32 (sint_to_fp (i32 GPR:$src))), (FFINT_S_W (MOVGR2FR_W GPR:$src))>;
 388: } // Predicates = [HasBasicF, IsLA32]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
