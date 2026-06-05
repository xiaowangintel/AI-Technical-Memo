# LoongArchLBTInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchLBTInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines LoongArch backend metadata using LLVM TableGen DSL; specifically it defines or implements target instruction information.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义 LoongArch 后端元数据；具体而言，它定义或实现目标指令信息。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //===- LoongArchLBTInstrInfo.td - LoongArch LBT instructions -*- tablegen -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file describes the LBT extension instructions.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: //===----------------------------------------------------------------------===//
  14: // Instructions
  15: //===----------------------------------------------------------------------===//
  16: 
  17: let hasSideEffects = 0, mayLoad = 0, mayStore = 0, Predicates = [HasExtLBT] in {
  18: 
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: def MOVGR2SCR : FmtGR2SCR<0x00000800>;
  20: def MOVSCR2GR : FmtSCR2GR<0x00000c00>;
  21: 
  22: def JISCR0 : FmtJISCR<0x48000200>;
  23: def JISCR1 : FmtJISCR<0x48000300>;
  24: 
  25: def ADDU12I_W : ALU_2RI5<0x00290000, simm5>;
  26: 
  27: def ADC_B : ALU_3R<0x00300000>;
  28: def ADC_H : ALU_3R<0x00308000>;
  29: def ADC_W : ALU_3R<0x00310000>;
  30: 
  31: def SBC_B : ALU_3R<0x00320000>;
  32: def SBC_H : ALU_3R<0x00328000>;
  33: def SBC_W : ALU_3R<0x00330000>;
  34: 
  35: def ROTR_B : ALU_3R<0x001a0000>;
  36: def ROTR_H : ALU_3R<0x001a8000>;
```
- **EN**: This block declares or refines TableGen records such as `MOVGR2SCR`, `MOVSCR2GR`, `JISCR0`, `JISCR1`, `ADDU12I_W`, `ADC_B`.
- **CN**: 该代码块声明或细化了 `MOVGR2SCR`, `MOVSCR2GR`, `JISCR0`, `JISCR1`, `ADDU12I_W`, `ADC_B` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: 
  38: def ROTRI_B : Fmt2RI3<0x004c2000>;
  39: def ROTRI_H : Fmt2RI4<0x004c4000>;
  40: 
  41: def RCR_B : ALU_3R<0x00340000>;
  42: def RCR_H : ALU_3R<0x00348000>;
  43: def RCR_W : ALU_3R<0x00350000>;
  44: 
  45: def RCRI_B : Fmt2RI3<0x00502000>;
  46: def RCRI_H : Fmt2RI4<0x00504000>;
  47: def RCRI_W : ALU_2RI5<0x00508000, uimm5>;
  48: 
  49: def FCVT_UD_D : FP_CONV<0x0114e400>;
  50: def FCVT_LD_D : FP_CONV<0x0114e000>;
  51: def FCVT_D_LD : FP_ALU_3R<0x01150000>;
  52: 
  53: let mayLoad = 1 in {
  54: def LDL_W : LOAD_2RI12<0x2e000000>;
```
- **EN**: This block declares or refines TableGen records such as `ROTRI_B`, `ROTRI_H`, `RCR_B`, `RCR_H`, `RCR_W`, `RCRI_B`.
- **CN**: 该代码块声明或细化了 `ROTRI_B`, `ROTRI_H`, `RCR_B`, `RCR_H`, `RCR_W`, `RCRI_B` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: def LDR_W : LOAD_2RI12<0x2e400000>;
  56: } // mayLoad = 1
  57: 
  58: let mayStore = 1 in {
  59: def STL_W : STORE_2RI12<0x2f000000>;
  60: def STR_W : STORE_2RI12<0x2f400000>;
  61: } // mayStore = 1
  62: 
  63: def X86ADC_B : NoDstFmt2R<0x003f000c>;
  64: def X86ADC_H : NoDstFmt2R<0x003f000d>;
  65: def X86ADC_W : NoDstFmt2R<0x003f000e>;
  66: def X86ADD_B : NoDstFmt2R<0x003f0004>;
  67: def X86ADD_H : NoDstFmt2R<0x003f0005>;
  68: def X86ADD_W : NoDstFmt2R<0x003f0006>;
  69: 
  70: def X86INC_B : NoDstFmt1R<0x00008000>;
  71: def X86INC_H : NoDstFmt1R<0x00008001>;
  72: def X86INC_W : NoDstFmt1R<0x00008002>;
```
- **EN**: This block declares or refines TableGen records such as `LDR_W`, `STL_W`, `STR_W`, `X86ADC_B`, `X86ADC_H`, `X86ADC_W`.
- **CN**: 该代码块声明或细化了 `LDR_W`, `STL_W`, `STR_W`, `X86ADC_B`, `X86ADC_H`, `X86ADC_W` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73: 
  74: def X86SBC_B : NoDstFmt2R<0x003f0010>;
  75: def X86SBC_H : NoDstFmt2R<0x003f0011>;
  76: def X86SBC_W : NoDstFmt2R<0x003f0012>;
  77: def X86SUB_B : NoDstFmt2R<0x003f0008>;
  78: def X86SUB_H : NoDstFmt2R<0x003f0009>;
  79: def X86SUB_W : NoDstFmt2R<0x003f000a>;
  80: 
  81: def X86DEC_B : NoDstFmt1R<0x00008004>;
  82: def X86DEC_H : NoDstFmt1R<0x00008005>;
  83: def X86DEC_W : NoDstFmt1R<0x00008006>;
  84: 
  85: def X86AND_B : NoDstFmt2R<0x003f8010>;
  86: def X86AND_H : NoDstFmt2R<0x003f8011>;
  87: def X86AND_W : NoDstFmt2R<0x003f8012>;
  88: 
  89: def X86OR_B : NoDstFmt2R<0x003f8014>;
  90: def X86OR_H : NoDstFmt2R<0x003f8015>;
```
- **EN**: This block declares or refines TableGen records such as `X86SBC_B`, `X86SBC_H`, `X86SBC_W`, `X86SUB_B`, `X86SUB_H`, `X86SUB_W`.
- **CN**: 该代码块声明或细化了 `X86SBC_B`, `X86SBC_H`, `X86SBC_W`, `X86SUB_B`, `X86SUB_H`, `X86SUB_W` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: def X86OR_W : NoDstFmt2R<0x003f8016>;
  92: 
  93: def X86XOR_B : NoDstFmt2R<0x003f8018>;
  94: def X86XOR_H : NoDstFmt2R<0x003f8019>;
  95: def X86XOR_W : NoDstFmt2R<0x003f801a>;
  96: 
  97: def X86MUL_B : NoDstFmt2R<0x003e8000>;
  98: def X86MUL_H : NoDstFmt2R<0x003e8001>;
  99: def X86MUL_W : NoDstFmt2R<0x003e8002>;
 100: def X86MUL_BU : NoDstFmt2R<0x003e8004>;
 101: def X86MUL_HU : NoDstFmt2R<0x003e8005>;
 102: 
 103: def X86RCL_B : NoDstFmt2R<0x003f800c>;
 104: def X86RCL_H : NoDstFmt2R<0x003f800d>;
 105: def X86RCL_W : NoDstFmt2R<0x003f800e>;
 106: def X86RCLI_B : NoDstFmt1RI3<0x00542018>;
 107: def X86RCLI_H : NoDstFmt1RI4<0x00544019>;
 108: def X86RCLI_W : NoDstFmt1RI5<0x0054801a>;
```
- **EN**: This block declares or refines TableGen records such as `X86OR_W`, `X86XOR_B`, `X86XOR_H`, `X86XOR_W`, `X86MUL_B`, `X86MUL_H`.
- **CN**: 该代码块声明或细化了 `X86OR_W`, `X86XOR_B`, `X86XOR_H`, `X86XOR_W`, `X86MUL_B`, `X86MUL_H` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: 
 110: def X86RCR_B : NoDstFmt2R<0x003f8008>;
 111: def X86RCR_H : NoDstFmt2R<0x003f8009>;
 112: def X86RCR_W : NoDstFmt2R<0x003f800a>;
 113: def X86RCRI_B : NoDstFmt1RI3<0x00542010>;
 114: def X86RCRI_H : NoDstFmt1RI4<0x00544011>;
 115: def X86RCRI_W : NoDstFmt1RI5<0x00548012>;
 116: 
 117: def X86ROTL_B : NoDstFmt2R<0x003f8004>;
 118: def X86ROTL_H : NoDstFmt2R<0x003f8005>;
 119: def X86ROTL_W : NoDstFmt2R<0x003f8006>;
 120: def X86ROTLI_B : NoDstFmt1RI3<0x00542014>;
 121: def X86ROTLI_H : NoDstFmt1RI4<0x00544015>;
 122: def X86ROTLI_W : NoDstFmt1RI5<0x00548016>;
 123: 
 124: def X86ROTR_B : NoDstFmt2R<0x003f8000>;
 125: def X86ROTR_H : NoDstFmt2R<0x003f8001>;
 126: def X86ROTR_W : NoDstFmt2R<0x003f8003>;
```
- **EN**: This block declares or refines TableGen records such as `X86RCR_B`, `X86RCR_H`, `X86RCR_W`, `X86RCRI_B`, `X86RCRI_H`, `X86RCRI_W`.
- **CN**: 该代码块声明或细化了 `X86RCR_B`, `X86RCR_H`, `X86RCR_W`, `X86RCRI_B`, `X86RCRI_H`, `X86RCRI_W` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127: def X86ROTRI_B : NoDstFmt1RI3<0x0054200c>;
 128: def X86ROTRI_H : NoDstFmt1RI4<0x0054400d>;
 129: def X86ROTRI_W : NoDstFmt1RI5<0x0054800e>;
 130: 
 131: def X86SLL_B : NoDstFmt2R<0x003f0014>;
 132: def X86SLL_H : NoDstFmt2R<0x003f0015>;
 133: def X86SLL_W : NoDstFmt2R<0x003f0016>;
 134: def X86SLLI_B : NoDstFmt1RI3<0x00542000>;
 135: def X86SLLI_H : NoDstFmt1RI4<0x00544001>;
 136: def X86SLLI_W : NoDstFmt1RI5<0x00548002>;
 137: 
 138: def X86SRL_B : NoDstFmt2R<0x003f0018>;
 139: def X86SRL_H : NoDstFmt2R<0x003f0019>;
 140: def X86SRL_W : NoDstFmt2R<0x003f001a>;
 141: def X86SRLI_B : NoDstFmt1RI3<0x00542004>;
 142: def X86SRLI_H : NoDstFmt1RI4<0x00544005>;
 143: def X86SRLI_W : NoDstFmt1RI5<0x00548006>;
 144: 
```
- **EN**: This block declares or refines TableGen records such as `X86ROTRI_B`, `X86ROTRI_H`, `X86ROTRI_W`, `X86SLL_B`, `X86SLL_H`, `X86SLL_W`.
- **CN**: 该代码块声明或细化了 `X86ROTRI_B`, `X86ROTRI_H`, `X86ROTRI_W`, `X86SLL_B`, `X86SLL_H`, `X86SLL_W` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: def X86SRA_B : NoDstFmt2R<0x003f001c>;
 146: def X86SRA_H : NoDstFmt2R<0x003f001d>;
 147: def X86SRA_W : NoDstFmt2R<0x003f001e>;
 148: def X86SRAI_B : NoDstFmt1RI3<0x00542008>;
 149: def X86SRAI_H : NoDstFmt1RI4<0x00544009>;
 150: def X86SRAI_W : NoDstFmt1RI5<0x0054800a>;
 151: 
 152: def SETX86J : Fmt1RI4<0x00368000>;
 153: def SETX86LOOPE : ALU_2R<0x00007800>;
 154: def SETX86LOOPNE : ALU_2R<0x00007c00>;
 155: def X86MFFLAG : Fmt1RI8<0x005c0000>;
 156: def X86MTFLAG : Fmt1RI8<0x005c0020>;
 157: def X86MFTOP : FmtMFTOP<0x00007400>;
 158: def X86MTTOP : FmtMTTOP<0x00007000>;
 159: 
 160: def X86INCTOP : FmtI32<0x00008009>;
 161: def X86DECTOP : FmtI32<0x00008029>;
 162: def X86SETTM : FmtI32<0x00008008>;
```
- **EN**: This block declares or refines TableGen records such as `X86SRA_B`, `X86SRA_H`, `X86SRA_W`, `X86SRAI_B`, `X86SRAI_H`, `X86SRAI_W`.
- **CN**: 该代码块声明或细化了 `X86SRA_B`, `X86SRA_H`, `X86SRA_W`, `X86SRAI_B`, `X86SRAI_H`, `X86SRAI_W` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163: def X86CLRTM : FmtI32<0x00008028>;
 164: def X86SETTAG : Fmt1RI5I8<0x00580000>;
 165: 
 166: def ARMADD_W : NoDstFmt2RI4<0x00370010>;
 167: def ARMSUB_W : NoDstFmt2RI4<0x00378010>;
 168: def ARMADC_W : NoDstFmt2RI4<0x00380010>;
 169: def ARMSBC_W : NoDstFmt2RI4<0x00388010>;
 170: def ARMAND_W : NoDstFmt2RI4<0x00390010>;
 171: def ARMOR_W : NoDstFmt2RI4<0x00398010>;
 172: def ARMXOR_W : NoDstFmt2RI4<0x003a0010>;
 173: def ARMNOT_W : NoDstFmt1RI4<0x003fc01c>;
 174: def ARMSLL_W : NoDstFmt2RI4<0x003a8010>;
 175: def ARMSRL_W : NoDstFmt2RI4<0x003b0010>;
 176: def ARMSRA_W : NoDstFmt2RI4<0x003b8010>;
 177: def ARMROTR_W : NoDstFmt2RI4<0x003c0010>;
 178: def ARMSLLI_W : NoDstFmt1RI5I4<0x003c8010>;
 179: def ARMSRLI_W : NoDstFmt1RI5I4<0x003d0010>;
 180: def ARMSRAI_W : NoDstFmt1RI5I4<0x003d8010>;
```
- **EN**: This block declares or refines TableGen records such as `X86CLRTM`, `X86SETTAG`, `ARMADD_W`, `ARMSUB_W`, `ARMADC_W`, `ARMSBC_W`.
- **CN**: 该代码块声明或细化了 `X86CLRTM`, `X86SETTAG`, `ARMADD_W`, `ARMSUB_W`, `ARMADC_W`, `ARMSBC_W` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: def ARMROTRI_W : NoDstFmt1RI5I4<0x003e0010>;
 182: def ARMRRX_W : NoDstFmt1RI4<0x003fc01f>;
 183: def ARMMOVE : Fmt2RI4<0x00364000>;
 184: def ARMMOV_W : NoDstFmt1RI4<0x003fc01d>;
 185: 
 186: def ARMMFFLAG : Fmt1RI8<0x005c0040>;
 187: def ARMMTFLAG : Fmt1RI8<0x005c0060>;
 188: def SETARMJ : Fmt1RI4<0x0036c000>;
 189: 
 190: let Predicates = [IsLA64] in {
 191: def ADDU12I_D : ALU_2RI5<0x00298000, simm5>;
 192: def ADC_D : ALU_3R<0x00318000>;
 193: def SBC_D : ALU_3R<0x00338000>;
 194: def RCR_D : ALU_3R<0x00358000>;
 195: def RCRI_D : ALU_2RI6<0x00510000, uimm6>;
 196: 
 197: // mayLoad = 1
 198: let mayLoad = 1 in {
```
- **EN**: This block declares or refines TableGen records such as `ARMROTRI_W`, `ARMRRX_W`, `ARMMOVE`, `ARMMOV_W`, `ARMMFFLAG`, `ARMMTFLAG`.
- **CN**: 该代码块声明或细化了 `ARMROTRI_W`, `ARMRRX_W`, `ARMMOVE`, `ARMMOV_W`, `ARMMFFLAG`, `ARMMTFLAG` 等 TableGen 记录。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: def LDL_D : LOAD_2RI12<0x2e800000>;
 200: def LDR_D : LOAD_2RI12<0x2ec00000>;
 201: } // mayLoad = 1
 202: 
 203: let mayStore = 1 in {
 204: def STL_D : STORE_2RI12<0x2f800000>;
 205: def STR_D : STORE_2RI12<0x2fc00000>;
 206: } // mayStore = 1
 207: 
 208: def X86ADC_D : NoDstFmt2R<0x003f000f>;
 209: def X86ADD_D : NoDstFmt2R<0x003f0007>;
 210: def X86ADD_WU : NoDstFmt2R<0x003f0000>;
 211: def X86ADD_DU : NoDstFmt2R<0x003f0001>;
 212: def X86INC_D : NoDstFmt1R<0x00008003>;
 213: def X86SBC_D : NoDstFmt2R<0x003f0013>;
 214: def X86SUB_WU : NoDstFmt2R<0x003f0002>;
 215: def X86SUB_D : NoDstFmt2R<0x003f000b>;
 216: def X86SUB_DU : NoDstFmt2R<0x003f0003>;
```
- **EN**: This block declares or refines TableGen records such as `LDL_D`, `LDR_D`, `STL_D`, `STR_D`, `X86ADC_D`, `X86ADD_D`.
- **CN**: 该代码块声明或细化了 `LDL_D`, `LDR_D`, `STL_D`, `STR_D`, `X86ADC_D`, `X86ADD_D` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217: def X86DEC_D : NoDstFmt1R<0x00008007>;
 218: def X86AND_D : NoDstFmt2R<0x003f8013>;
 219: def X86OR_D : NoDstFmt2R<0x003f8017>;
 220: def X86XOR_D : NoDstFmt2R<0x003f801b>;
 221: def X86MUL_D : NoDstFmt2R<0x003e8003>;
 222: def X86MUL_WU : NoDstFmt2R<0x003e8006>;
 223: def X86MUL_DU : NoDstFmt2R<0x003e8007>;
 224: def X86RCL_D : NoDstFmt2R<0x003f800f>;
 225: def X86RCLI_D : NoDstFmt1RI6<0x0055001b>;
 226: def X86RCR_D : NoDstFmt2R<0x003f800b>;
 227: def X86RCRI_D : NoDstFmt1RI6<0x00550013>;
 228: def X86ROTL_D : NoDstFmt2R<0x003f8007>;
 229: def X86ROTLI_D : NoDstFmt1RI6<0x00550017>;
 230: def X86ROTR_D : NoDstFmt2R<0x003f8002>;
 231: def X86ROTRI_D : NoDstFmt1RI6<0x0055000f>;
 232: def X86SLL_D : NoDstFmt2R<0x003f0017>;
 233: def X86SLLI_D : NoDstFmt1RI6<0x00550003>;
 234: def X86SRL_D : NoDstFmt2R<0x003f001b>;
```
- **EN**: This block declares or refines TableGen records such as `X86DEC_D`, `X86AND_D`, `X86OR_D`, `X86XOR_D`, `X86MUL_D`, `X86MUL_WU`.
- **CN**: 该代码块声明或细化了 `X86DEC_D`, `X86AND_D`, `X86OR_D`, `X86XOR_D`, `X86MUL_D`, `X86MUL_WU` 等 TableGen 记录。

### Lines 235-241 / 第 235-241 行
```tablegen
 235: def X86SRLI_D : NoDstFmt1RI6<0x00550007>;
 236: def X86SRA_D : NoDstFmt2R<0x003f001f>;
 237: def X86SRAI_D : NoDstFmt1RI6<0x0055000b>;
 238: def ARMMOV_D : NoDstFmt1RI4<0x003fc01e>;
 239: 
 240: } // Predicates = [IsLA64]
 241: } // hasSideEffects = 0, mayLoad = 0, mayStore = 0, Predicates = [HasExtLBT]
```
- **EN**: This block declares or refines TableGen records such as `X86SRLI_D`, `X86SRA_D`, `X86SRAI_D`, `ARMMOV_D`.
- **CN**: 该代码块声明或细化了 `X86SRLI_D`, `X86SRA_D`, `X86SRAI_D`, `ARMMOV_D` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
