# LoongArchFloat64InstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchFloat64InstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: // LoongArchFloat64InstrInfo.td - Double-Precision Float instr --*- tablegen -*-
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the basic double-precision floating-point instructions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: //===----------------------------------------------------------------------===//
  14: // LoongArch specific DAG Nodes.
  15: //===----------------------------------------------------------------------===//
  16: 
  17: def SDT_LoongArchMOVGR2FR_D
  18:     : SDTypeProfile<1, 1, [SDTCisVT<0, f64>, SDTCisVT<1, i64>]>;
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SDT_LoongArchMOVGR2FR_D`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SDT_LoongArchMOVGR2FR_D` 等 TableGen 记录。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: def SDT_LoongArchMOVGR2FR_D_LO_HI
  20:     : SDTypeProfile<1, 2, [SDTCisVT<0, f64>, SDTCisVT<1, i32>,
  21:                            SDTCisSameAs<1, 2>]>;
  22: 
  23: // FPR<->GPR transfer operations
  24: def loongarch_movgr2fr_d
  25:     : SDNode<"LoongArchISD::MOVGR2FR_D", SDT_LoongArchMOVGR2FR_D>;
  26: def loongarch_movgr2fr_d_lo_hi
  27:     : SDNode<"LoongArchISD::MOVGR2FR_D_LO_HI", SDT_LoongArchMOVGR2FR_D_LO_HI>;
  28: 
  29: //===----------------------------------------------------------------------===//
  30: // Instructions
  31: //===----------------------------------------------------------------------===//
  32: 
  33: let Predicates = [HasBasicD] in {
  34: 
  35: // Arithmetic Operation Instructions
  36: def FADD_D : FP_ALU_3R<0x01010000, FPR64>;
```
- **EN**: This block declares or refines TableGen records such as `SDT_LoongArchMOVGR2FR_D_LO_HI`, `loongarch_movgr2fr_d`, `loongarch_movgr2fr_d_lo_hi`, `FADD_D`.
- **CN**: 该代码块声明或细化了 `SDT_LoongArchMOVGR2FR_D_LO_HI`, `loongarch_movgr2fr_d`, `loongarch_movgr2fr_d_lo_hi`, `FADD_D` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: def FSUB_D : FP_ALU_3R<0x01030000, FPR64>;
  38: def FMUL_D : FP_ALU_3R<0x01050000, FPR64>;
  39: def FDIV_D : FP_ALU_3R<0x01070000, FPR64>;
  40: def FMADD_D  : FP_ALU_4R<0x08200000, FPR64>;
  41: def FMSUB_D  : FP_ALU_4R<0x08600000, FPR64>;
  42: def FNMADD_D : FP_ALU_4R<0x08a00000, FPR64>;
  43: def FNMSUB_D : FP_ALU_4R<0x08e00000, FPR64>;
  44: def FMAX_D  : FP_ALU_3R<0x01090000, FPR64>;
  45: def FMIN_D  : FP_ALU_3R<0x010b0000, FPR64>;
  46: def FMAXA_D : FP_ALU_3R<0x010d0000, FPR64>;
  47: def FMINA_D : FP_ALU_3R<0x010f0000, FPR64>;
  48: def FABS_D   : FP_ALU_2R<0x01140800, FPR64>;
  49: def FNEG_D   : FP_ALU_2R<0x01141800, FPR64>;
  50: def FSQRT_D  : FP_ALU_2R<0x01144800, FPR64>;
  51: def FRECIP_D : FP_ALU_2R<0x01145800, FPR64>;
  52: def FRSQRT_D : FP_ALU_2R<0x01146800, FPR64>;
  53: def FRECIPE_D : FP_ALU_2R<0x01147800, FPR64>;
  54: def FRSQRTE_D : FP_ALU_2R<0x01148800, FPR64>;
```
- **EN**: This block declares or refines TableGen records such as `FSUB_D`, `FMUL_D`, `FDIV_D`, `FMADD_D`, `FMSUB_D`, `FNMADD_D`.
- **CN**: 该代码块声明或细化了 `FSUB_D`, `FMUL_D`, `FDIV_D`, `FMADD_D`, `FMSUB_D`, `FNMADD_D` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: def FSCALEB_D : FP_ALU_3R<0x01110000, FPR64>;
  56: def FLOGB_D   : FP_ALU_2R<0x01142800, FPR64>;
  57: def FCOPYSIGN_D : FP_ALU_3R<0x01130000, FPR64>;
  58: def FCLASS_D  : FP_ALU_2R<0x01143800, FPR64>;
  59: 
  60: // Comparison Instructions
  61: def FCMP_CAF_D  : FP_CMP<0x0c200000, FPR64>;
  62: def FCMP_CUN_D  : FP_CMP<0x0c240000, FPR64>;
  63: def FCMP_CEQ_D  : FP_CMP<0x0c220000, FPR64>;
  64: def FCMP_CUEQ_D : FP_CMP<0x0c260000, FPR64>;
  65: def FCMP_CLT_D  : FP_CMP<0x0c210000, FPR64>;
  66: def FCMP_CULT_D : FP_CMP<0x0c250000, FPR64>;
  67: def FCMP_CLE_D  : FP_CMP<0x0c230000, FPR64>;
  68: def FCMP_CULE_D : FP_CMP<0x0c270000, FPR64>;
  69: def FCMP_CNE_D  : FP_CMP<0x0c280000, FPR64>;
  70: def FCMP_COR_D  : FP_CMP<0x0c2a0000, FPR64>;
  71: def FCMP_CUNE_D : FP_CMP<0x0c2c0000, FPR64>;
  72: def FCMP_SAF_D  : FP_CMP<0x0c208000, FPR64>;
```
- **EN**: This block declares or refines TableGen records such as `FSCALEB_D`, `FLOGB_D`, `FCOPYSIGN_D`, `FCLASS_D`, `FCMP_CAF_D`, `FCMP_CUN_D`.
- **CN**: 该代码块声明或细化了 `FSCALEB_D`, `FLOGB_D`, `FCOPYSIGN_D`, `FCLASS_D`, `FCMP_CAF_D`, `FCMP_CUN_D` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73: def FCMP_SUN_D  : FP_CMP<0x0c248000, FPR64>;
  74: def FCMP_SEQ_D  : FP_CMP<0x0c228000, FPR64>;
  75: def FCMP_SUEQ_D : FP_CMP<0x0c268000, FPR64>;
  76: def FCMP_SLT_D  : FP_CMP<0x0c218000, FPR64>;
  77: def FCMP_SULT_D : FP_CMP<0x0c258000, FPR64>;
  78: def FCMP_SLE_D  : FP_CMP<0x0c238000, FPR64>;
  79: def FCMP_SULE_D : FP_CMP<0x0c278000, FPR64>;
  80: def FCMP_SNE_D  : FP_CMP<0x0c288000, FPR64>;
  81: def FCMP_SOR_D  : FP_CMP<0x0c2a8000, FPR64>;
  82: def FCMP_SUNE_D : FP_CMP<0x0c2c8000, FPR64>;
  83: 
  84: // Conversion Instructions
  85: def FFINT_S_L : FP_CONV<0x011d1800, FPR32, FPR64>;
  86: def FTINT_L_S : FP_CONV<0x011b2400, FPR64, FPR32>;
  87: def FTINTRM_L_S : FP_CONV<0x011a2400, FPR64, FPR32>;
  88: def FTINTRP_L_S : FP_CONV<0x011a6400, FPR64, FPR32>;
  89: def FTINTRZ_L_S : FP_CONV<0x011aa400, FPR64, FPR32>;
  90: def FTINTRNE_L_S : FP_CONV<0x011ae400, FPR64, FPR32>;
```
- **EN**: This block declares or refines TableGen records such as `FCMP_SUN_D`, `FCMP_SEQ_D`, `FCMP_SUEQ_D`, `FCMP_SLT_D`, `FCMP_SULT_D`, `FCMP_SLE_D`.
- **CN**: 该代码块声明或细化了 `FCMP_SUN_D`, `FCMP_SEQ_D`, `FCMP_SUEQ_D`, `FCMP_SLT_D`, `FCMP_SULT_D`, `FCMP_SLE_D` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: def FCVT_S_D : FP_CONV<0x01191800, FPR32, FPR64>;
  92: def FCVT_D_S : FP_CONV<0x01192400, FPR64, FPR32>;
  93: def FFINT_D_W : FP_CONV<0x011d2000, FPR64, FPR32>;
  94: def FFINT_D_L : FP_CONV<0x011d2800, FPR64, FPR64>;
  95: def FTINT_W_D : FP_CONV<0x011b0800, FPR32, FPR64>;
  96: def FTINT_L_D : FP_CONV<0x011b2800, FPR64, FPR64>;
  97: def FTINTRM_W_D : FP_CONV<0x011a0800, FPR32, FPR64>;
  98: def FTINTRM_L_D : FP_CONV<0x011a2800, FPR64, FPR64>;
  99: def FTINTRP_W_D : FP_CONV<0x011a4800, FPR32, FPR64>;
 100: def FTINTRP_L_D : FP_CONV<0x011a6800, FPR64, FPR64>;
 101: def FTINTRZ_W_D : FP_CONV<0x011a8800, FPR32, FPR64>;
 102: def FTINTRZ_L_D : FP_CONV<0x011aa800, FPR64, FPR64>;
 103: def FTINTRNE_W_D : FP_CONV<0x011ac800, FPR32, FPR64>;
 104: def FTINTRNE_L_D : FP_CONV<0x011ae800, FPR64, FPR64>;
 105: def FRINT_D : FP_CONV<0x011e4800, FPR64, FPR64>;
 106: 
 107: // Move Instructions
 108: def FMOV_D        : FP_MOV<0x01149800, FPR64, FPR64>;
```
- **EN**: This block declares or refines TableGen records such as `FCVT_S_D`, `FCVT_D_S`, `FFINT_D_W`, `FFINT_D_L`, `FTINT_W_D`, `FTINT_L_D`.
- **CN**: 该代码块声明或细化了 `FCVT_S_D`, `FCVT_D_S`, `FFINT_D_W`, `FFINT_D_L`, `FTINT_W_D`, `FTINT_L_D` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: def MOVFRH2GR_S   : FP_MOV<0x0114bc00, GPR, FPR64>;
 110: let isCodeGenOnly = 1 in {
 111: def MOVFR2GR_S_64 : FP_MOV<0x0114b400, GPR, FPR64>;
 112: def FSEL_xD : FP_SEL<0x0d000000, FPR64>;
 113: } // isCodeGenOnly = 1
 114: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, Constraints = "$dst = $out" in {
 115: def MOVGR2FRH_W : FPFmtMOV<0x0114ac00, (outs FPR64:$out),
 116:                            (ins FPR64:$dst, GPR:$src),
 117:                            "$dst, $src">;
 118: } // hasSideEffects = 0, mayLoad = 0, mayStore = 0, Constraints = "$dst = $out"
 119: 
 120: // Common Memory Access Instructions
 121: def FLD_D : FP_LOAD_2RI12<0x2b800000, FPR64>;
 122: def FST_D : FP_STORE_2RI12<0x2bc00000, FPR64>;
 123: def FLDX_D : FP_LOAD_3R<0x38340000, FPR64>;
 124: def FSTX_D : FP_STORE_3R<0x383c0000, FPR64>;
 125: 
 126: // Bound Check Memory Access Instructions
```
- **EN**: This block declares or refines TableGen records such as `MOVFRH2GR_S`, `MOVFR2GR_S_64`, `FSEL_xD`, `MOVGR2FRH_W`, `FLD_D`, `FST_D`.
- **CN**: 该代码块声明或细化了 `MOVFRH2GR_S`, `MOVFR2GR_S_64`, `FSEL_xD`, `MOVGR2FRH_W`, `FLD_D`, `FST_D` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127: def FLDGT_D : FP_LOAD_3R<0x38748000, FPR64>;
 128: def FLDLE_D : FP_LOAD_3R<0x38758000, FPR64>;
 129: def FSTGT_D : FP_STORE_3R<0x38768000, FPR64>;
 130: def FSTLE_D : FP_STORE_3R<0x38778000, FPR64>;
 131: 
 132: } // Predicates = [HasBasicD]
 133: 
 134: // Instructions only available on LA64
 135: let Predicates = [HasBasicD, IsLA64] in {
 136: def MOVGR2FR_D  : FP_MOV<0x0114a800, FPR64, GPR>;
 137: def MOVFR2GR_D  : FP_MOV<0x0114b800, GPR, FPR64>;
 138: } // Predicates = [HasBasicD, IsLA64]
 139: 
 140: // Instructions only available on LA32
 141: let Predicates = [HasBasicD, IsLA32], isCodeGenOnly = 1 in {
 142: def MOVGR2FR_W_64 : FP_MOV<0x0114a400, FPR64, GPR>;
 143: } // Predicates = [HasBasicD, IsLA32], isCodeGenOnly = 1
 144: 
```
- **EN**: This block declares or refines TableGen records such as `FLDGT_D`, `FLDLE_D`, `FSTGT_D`, `FSTLE_D`, `MOVGR2FR_D`, `MOVFR2GR_D`.
- **CN**: 该代码块声明或细化了 `FLDGT_D`, `FLDLE_D`, `FSTGT_D`, `FSTLE_D`, `MOVGR2FR_D`, `MOVFR2GR_D` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: //===----------------------------------------------------------------------===//
 146: // Pseudo-instructions and codegen patterns
 147: //===----------------------------------------------------------------------===//
 148: 
 149: let Predicates = [HasBasicD] in {
 150: 
 151: /// Float arithmetic operations
 152: 
 153: def : PatFprFpr<fadd, FADD_D, FPR64>;
 154: def : PatFprFpr<fsub, FSUB_D, FPR64>;
 155: def : PatFprFpr<fmul, FMUL_D, FPR64>;
 156: def : PatFprFpr<fdiv, FDIV_D, FPR64>;
 157: def : PatFprFpr<fcopysign, FCOPYSIGN_D, FPR64>;
 158: def : PatFprFpr<fmaxnum_ieee, FMAX_D, FPR64>;
 159: def : PatFprFpr<fmaxnum, FMAX_D, FPR64>;
 160: def : PatFprFpr<fminnum_ieee, FMIN_D, FPR64>;
 161: def : PatFprFpr<fminnum, FMIN_D, FPR64>;
 162: def : PatFpr<fneg, FNEG_D, FPR64>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 163-180 / 第 163-180 行
```tablegen
 163: def : PatFpr<fabs, FABS_D, FPR64>;
 164: def : PatFpr<fsqrt, FSQRT_D, FPR64>;
 165: def : Pat<(fdiv fpimm1, (fsqrt FPR64:$fj)), (FRSQRT_D FPR64:$fj)>;
 166: let Predicates = [IsLA32] in {
 167: def : Pat<(fdiv (loongarch_movgr2fr_d_lo_hi (i32 0), (i32 1072693248)),
 168:                 (fsqrt FPR64:$fj)),
 169:           (FRSQRT_D FPR64:$fj)>;
 170: } // Predicates = [IsLA32]
 171: def : Pat<(fcopysign FPR64:$fj, FPR32:$fk),
 172:           (FCOPYSIGN_D FPR64:$fj, (FCVT_D_S FPR32:$fk))>;
 173: def : Pat<(fcopysign FPR32:$fj, FPR64:$fk),
 174:           (FCOPYSIGN_S FPR32:$fj, (FCVT_S_D FPR64:$fk))>;
 175: def : Pat<(fcanonicalize FPR64:$fj), (FMAX_D $fj, $fj)>;
 176: let Predicates = [IsLA32] in {
 177: def : Pat<(is_fpclass FPR64:$fj, (i32 timm:$mask)),
 178:           (SLTU R0, (ANDI (MOVFR2GR_S_64 (FCLASS_D FPR64:$fj)),
 179:                           (to_fclass_mask timm:$mask)))>;
 180: } // Predicates = [IsLA32]
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: let Predicates = [IsLA64] in {
 182: def : Pat<(is_fpclass FPR64:$fj, (i32 timm:$mask)),
 183:           (SLTU R0, (ANDI (MOVFR2GR_D (FCLASS_D FPR64:$fj)),
 184:                           (to_fclass_mask timm:$mask)))>;
 185: } // Predicates = [IsLA64]
 186: 
 187: /// Setcc
 188: 
 189: // Match non-signaling comparison
 190: 
 191: // SETOGT/SETOGE/SETUGT/SETUGE/SETGE/SETNE/SETGT will expand into
 192: // SETOLT/SETOLE/SETULT/SETULE/SETLE/SETEQ/SETLT.
 193: def : PatFPSetcc<SETOEQ, FCMP_CEQ_D,  FPR64>;
 194: def : PatFPSetcc<SETEQ,  FCMP_CEQ_D,  FPR64>;
 195: def : PatFPSetcc<SETOLT, FCMP_CLT_D,  FPR64>;
 196: def : PatFPSetcc<SETOLE, FCMP_CLE_D,  FPR64>;
 197: def : PatFPSetcc<SETLE,  FCMP_CLE_D,  FPR64>;
 198: def : PatFPSetcc<SETONE, FCMP_CNE_D,  FPR64>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: def : PatFPSetcc<SETO,   FCMP_COR_D,  FPR64>;
 200: def : PatFPSetcc<SETUEQ, FCMP_CUEQ_D, FPR64>;
 201: def : PatFPSetcc<SETULT, FCMP_CULT_D, FPR64>;
 202: def : PatFPSetcc<SETULE, FCMP_CULE_D, FPR64>;
 203: def : PatFPSetcc<SETUNE, FCMP_CUNE_D, FPR64>;
 204: def : PatFPSetcc<SETUO,  FCMP_CUN_D,  FPR64>;
 205: def : PatFPSetcc<SETLT,  FCMP_CLT_D,  FPR64>;
 206: 
 207: defm : PatFPBrcond<SETOEQ, FCMP_CEQ_D, FPR64>;
 208: defm : PatFPBrcond<SETEQ,  FCMP_CEQ_D, FPR64>;
 209: defm : PatFPBrcond<SETOLT, FCMP_CLT_D, FPR64>;
 210: defm : PatFPBrcond<SETOLE, FCMP_CLE_D, FPR64>;
 211: defm : PatFPBrcond<SETLE,  FCMP_CLE_D, FPR64>;
 212: defm : PatFPBrcond<SETONE, FCMP_CNE_D, FPR64>;
 213: defm : PatFPBrcond<SETO,   FCMP_COR_D, FPR64>;
 214: defm : PatFPBrcond<SETUEQ, FCMP_CUEQ_D, FPR64>;
 215: defm : PatFPBrcond<SETULT, FCMP_CULT_D, FPR64>;
 216: defm : PatFPBrcond<SETULE, FCMP_CULE_D, FPR64>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-234 / 第 217-234 行
```tablegen
 217: defm : PatFPBrcond<SETUNE, FCMP_CUNE_D, FPR64>;
 218: defm : PatFPBrcond<SETUO,  FCMP_CUN_D, FPR64>;
 219: defm : PatFPBrcond<SETLT,  FCMP_CLT_D, FPR64>;
 220: 
 221: // Match signaling comparison
 222: 
 223: def : PatStrictFsetccs<SETOEQ, FCMP_SEQ_D,  FPR64>;
 224: def : PatStrictFsetccs<SETOLT, FCMP_SLT_D,  FPR64>;
 225: def : PatStrictFsetccs<SETOLE, FCMP_SLE_D,  FPR64>;
 226: def : PatStrictFsetccs<SETONE, FCMP_SNE_D,  FPR64>;
 227: def : PatStrictFsetccs<SETO,   FCMP_SOR_D,  FPR64>;
 228: def : PatStrictFsetccs<SETUEQ, FCMP_SUEQ_D, FPR64>;
 229: def : PatStrictFsetccs<SETULT, FCMP_SULT_D, FPR64>;
 230: def : PatStrictFsetccs<SETULE, FCMP_SULE_D, FPR64>;
 231: def : PatStrictFsetccs<SETUNE, FCMP_SUNE_D, FPR64>;
 232: def : PatStrictFsetccs<SETUO,  FCMP_SUN_D,  FPR64>;
 233: def : PatStrictFsetccs<SETLT,  FCMP_SLT_D,  FPR64>;
 234: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```tablegen
 235: /// Select
 236: 
 237: def : Pat<(select CFR:$cc, FPR64:$fk, FPR64:$fj),
 238:           (FSEL_xD FPR64:$fj, FPR64:$fk, CFR:$cc)>;
 239: 
 240: /// Selectcc
 241: 
 242: def : PatFPSelectcc<SETOEQ, FCMP_CEQ_D,  FSEL_xD, FPR64>;
 243: def : PatFPSelectcc<SETOLT, FCMP_CLT_D,  FSEL_xD, FPR64>;
 244: def : PatFPSelectcc<SETOLE, FCMP_CLE_D,  FSEL_xD, FPR64>;
 245: def : PatFPSelectcc<SETONE, FCMP_CNE_D,  FSEL_xD, FPR64>;
 246: def : PatFPSelectcc<SETO,   FCMP_COR_D,  FSEL_xD, FPR64>;
 247: def : PatFPSelectcc<SETUEQ, FCMP_CUEQ_D, FSEL_xD, FPR64>;
 248: def : PatFPSelectcc<SETULT, FCMP_CULT_D, FSEL_xD, FPR64>;
 249: def : PatFPSelectcc<SETULE, FCMP_CULE_D, FSEL_xD, FPR64>;
 250: def : PatFPSelectcc<SETUNE, FCMP_CUNE_D, FSEL_xD, FPR64>;
 251: def : PatFPSelectcc<SETUO,  FCMP_CUN_D,  FSEL_xD, FPR64>;
 252: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-270 / 第 253-270 行
```tablegen
 253: /// Loads
 254: 
 255: defm : LdPat<load, FLD_D, f64>;
 256: def : RegRegLdPat<load, FLDX_D, f64>;
 257: 
 258: /// Stores
 259: 
 260: defm : StPat<store, FST_D, FPR64, f64>;
 261: def : RegRegStPat<store, FSTX_D, FPR64, f64>;
 262: 
 263: /// FP conversion operations
 264: 
 265: def : Pat<(loongarch_ftint FPR64:$src), (FTINTRZ_W_D FPR64:$src)>;
 266: def : Pat<(f64 (loongarch_ftint FPR64:$src)), (FTINTRZ_L_D FPR64:$src)>;
 267: def : Pat<(loongarch_ftint FPR32:$src), (FTINTRZ_L_S FPR32:$src)>;
 268: 
 269: // f64 -> f32
 270: def : Pat<(f32 (fpround FPR64:$src)), (FCVT_S_D FPR64:$src)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 271-288 / 第 271-288 行
```tablegen
 271: // f32 -> f64
 272: def : Pat<(f64 (fpextend FPR32:$src)), (FCVT_D_S FPR32:$src)>;
 273: 
 274: // FP reciprocal operation
 275: def : Pat<(fdiv fpimm1, FPR64:$src), (FRECIP_D $src)>;
 276: let Predicates = [IsLA32] in {
 277: def : Pat<(fdiv (loongarch_movgr2fr_d_lo_hi (i32 0), (i32 1072693248)), FPR64:$src),
 278:           (FRECIP_D FPR64:$src)>;
 279: } // Predicates = [IsLA32]
 280: 
 281: let Predicates = [HasFrecipe] in {
 282: // FP approximate reciprocal operation
 283: def : Pat<(int_loongarch_frecipe_d FPR64:$src), (FRECIPE_D FPR64:$src)>;
 284: def : Pat<(int_loongarch_frsqrte_d FPR64:$src), (FRSQRTE_D FPR64:$src)>;
 285: def : Pat<(loongarch_frecipe FPR64:$src), (FRECIPE_D FPR64:$src)>;
 286: def : Pat<(loongarch_frsqrte FPR64:$src), (FRSQRTE_D FPR64:$src)>;
 287: }
 288: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 289-306 / 第 289-306 行
```tablegen
 289: // fmadd.d: fj * fk + fa
 290: def : Pat<(fma FPR64:$fj, FPR64:$fk, FPR64:$fa), (FMADD_D $fj, $fk, $fa)>;
 291: 
 292: // fmsub.d: fj * fk - fa
 293: def : Pat<(fma FPR64:$fj, FPR64:$fk, (fneg FPR64:$fa)),
 294:           (FMSUB_D FPR64:$fj, FPR64:$fk, FPR64:$fa)>;
 295: 
 296: // fnmadd.d: -(fj * fk + fa)
 297: def : Pat<(fneg (fma FPR64:$fj, FPR64:$fk, FPR64:$fa)),
 298:           (FNMADD_D FPR64:$fj, FPR64:$fk, FPR64:$fa)>;
 299: 
 300: // fnmadd.d: -fj * fk - fa (the nsz flag on the FMA)
 301: def : Pat<(fma_nsz (fneg FPR64:$fj), FPR64:$fk, (fneg FPR64:$fa)),
 302:           (FNMADD_D FPR64:$fj, FPR64:$fk, FPR64:$fa)>;
 303: 
 304: // fnmsub.d: -(fj * fk - fa)
 305: def : Pat<(fneg (fma FPR64:$fj, FPR64:$fk, (fneg FPR64:$fa))),
 306:           (FNMSUB_D FPR64:$fj, FPR64:$fk, FPR64:$fa)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 307-324 / 第 307-324 行
```tablegen
 307: 
 308: // fnmsub.d: -fj * fk + fa (the nsz flag on the FMA)
 309: def : Pat<(fma_nsz (fneg FPR64:$fj), FPR64:$fk, FPR64:$fa),
 310:           (FNMSUB_D FPR64:$fj, FPR64:$fk, FPR64:$fa)>;
 311: 
 312: // ffint.d.l
 313: def : Pat<(loongarch_sitof FPR64:$fj), (FFINT_D_L FPR64:$fj)>;
 314: } // Predicates = [HasBasicD]
 315: 
 316: /// Floating point constants
 317: 
 318: let Predicates = [HasBasicD, IsLA64] in {
 319: def : Pat<(f64 fpimm0), (MOVGR2FR_D R0)>;
 320: def : Pat<(f64 fpimm0neg), (FNEG_D (MOVGR2FR_D R0))>;
 321: def : Pat<(f64 fpimm1), (FFINT_D_L (MOVGR2FR_D (ADDI_D R0, 1)))>;
 322: } // Predicates = [HasBasicD, IsLA64]
 323: let Predicates = [HasBasicD, IsLA32] in {
 324: def : Pat<(f64 fpimm0), (MOVGR2FRH_W (MOVGR2FR_W_64 R0), R0)>;
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 325-342 / 第 325-342 行
```tablegen
 325: def : Pat<(f64 fpimm0neg), (FNEG_D (MOVGR2FRH_W (MOVGR2FR_W_64 R0), R0))>;
 326: def : Pat<(f64 fpimm1), (FCVT_D_S (FFINT_S_W (MOVGR2FR_W (ADDI_W R0, 1))))>;
 327: } // Predicates = [HasBasicD, IsLA32]
 328: 
 329: /// Convert int to FP
 330: 
 331: let Predicates = [HasBasicD, IsLA64] in {
 332: def : Pat<(f32 (sint_to_fp GPR:$src)), (FFINT_S_L (MOVGR2FR_D GPR:$src))>;
 333: def : Pat<(f64 (sint_to_fp (i64 (sexti32 (i64 GPR:$src))))),
 334:           (FFINT_D_W (MOVGR2FR_W GPR:$src))>;
 335: def : Pat<(f64 (sint_to_fp GPR:$src)), (FFINT_D_L (MOVGR2FR_D GPR:$src))>;
 336: 
 337: def : Pat<(bitconvert GPR:$src), (MOVGR2FR_D GPR:$src)>;
 338: def : Pat<(loongarch_movgr2fr_d GPR:$src), (MOVGR2FR_D GPR:$src)>;
 339: } // Predicates = [HasBasicD, IsLA64]
 340: let Predicates = [HasBasicD, IsLA32] in {
 341: def : Pat<(f64 (sint_to_fp (i32 GPR:$src))), (FFINT_D_W (MOVGR2FR_W GPR:$src))>;
 342: 
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 343-360 / 第 343-360 行
```tablegen
 343: def : Pat<(f64 (loongarch_movgr2fr_d_lo_hi (i32 GPR:$lo), (i32 GPR:$hi))),
 344:           (MOVGR2FRH_W (MOVGR2FR_W_64 GPR:$lo), GPR:$hi)>;
 345: } // Predicates = [HasBasicD, IsLA32]
 346: 
 347: // Convert FP to int
 348: let Predicates = [HasBasicD, IsLA64] in {
 349: def : Pat<(bitconvert FPR64:$src), (MOVFR2GR_D FPR64:$src)>;
 350: } // Predicates = [HasBasicD, IsLA64]
 351: 
 352: // FP Rounding
 353: let Predicates = [HasBasicD, IsLA64] in {
 354: def : PatFpr<frint, FRINT_D, FPR64>;
 355: } // Predicates = [HasBasicD, IsLA64]
 356: 
 357: /// Pseudo-instructions needed for the soft-float ABI with LA32D
 358: 
 359: let Predicates = [HasBasicD, IsLA32] in {
 360: // Moves two GPRs to an FPR.
```
- **EN**: This span continues the file's main responsibility: defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 361-371 / 第 361-371 行
```tablegen
 361: let usesCustomInserter = 1 in
 362: def BuildPairF64Pseudo
 363:     : Pseudo<(outs FPR64:$dst), (ins GPR:$src1, GPR:$src2),
 364:              [(set FPR64:$dst, (loongarch_build_pair_f64 GPR:$src1, GPR:$src2))]>;
 365: 
 366: // Moves an FPR to two GPRs.
 367: let usesCustomInserter = 1 in
 368: def SplitPairF64Pseudo
 369:     : Pseudo<(outs GPR:$dst1, GPR:$dst2), (ins FPR64:$src),
 370:              [(set GPR:$dst1, GPR:$dst2, (loongarch_split_pair_f64 FPR64:$src))]>;
 371: } // Predicates = [HasBasicD, IsLA32]
```
- **EN**: This block declares or refines TableGen records such as `BuildPairF64Pseudo`, `SplitPairF64Pseudo`.
- **CN**: 该代码块声明或细化了 `BuildPairF64Pseudo`, `SplitPairF64Pseudo` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
