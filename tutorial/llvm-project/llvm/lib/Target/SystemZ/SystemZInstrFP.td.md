# SystemZInstrFP.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrFP.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```tablegen
   1: //==- SystemZInstrFP.td - Floating-point SystemZ instructions --*- tblgen-*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: // TODO: Most floating-point instructions (except for simple moves and the
  10: // like) can raise exceptions -- should they have hasSideEffects=1 ?
  11: 
  12: //===----------------------------------------------------------------------===//
  13: // Select instructions
  14: //===----------------------------------------------------------------------===//
  15: 
  16: // C's ?: operator for floating-point operands.
  17: let Predicates = [FeatureVector] in {
  18:   def SelectVR32 : SelectWrapper<f32, VR32>;
  19:   def SelectVR64 : SelectWrapper<f64, VR64>;
  20: }
  21: def SelectF32  : SelectWrapper<f32, FP32>;
  22: def SelectF64  : SelectWrapper<f64, FP64>;
  23: let Predicates = [FeatureNoVectorEnhancements1] in
  24:   def SelectF128 : SelectWrapper<f128, FP128>;
  25: let Predicates = [FeatureVectorEnhancements1] in
  26:   def SelectVR128 : SelectWrapper<f128, VR128>;
  27: 
  28: defm CondStoreF32 : CondStores<FP32, simple_store,
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SelectVR32`, `SelectVR64`, `SelectF32`, `SelectF64`, `SelectF128`, `SelectVR128`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SelectVR32`, `SelectVR64`, `SelectF32`, `SelectF64`, `SelectF128`, `SelectVR128` 等 TableGen 记录。

### Lines 29-56 / 第 29-56 行
```tablegen
  29:                                simple_load, bdxaddr20only>;
  30: defm CondStoreF64 : CondStores<FP64, simple_store,
  31:                                simple_load, bdxaddr20only>;
  32: 
  33: //===----------------------------------------------------------------------===//
  34: // Move instructions
  35: //===----------------------------------------------------------------------===//
  36: 
  37: // Load zero.
  38: let isAsCheapAsAMove = 1, isMoveImm = 1 in {
  39:   let isCodeGenOnly = 1 in
  40:     def LZER_16 : InherentRRE<"lzer", 0xB374, FP16,  fpimm0>;
  41:   def LZER : InherentRRE<"lzer", 0xB374, FP32,  fpimm0>;
  42:   def LZDR : InherentRRE<"lzdr", 0xB375, FP64,  fpimm0>;
  43:   def LZXR : InherentRRE<"lzxr", 0xB376, FP128, fpimm0>;
  44: }
  45: 
  46: // Moves between two floating-point registers.
  47: let isMoveReg = 1 in  {
  48:   def LER : UnaryRR <"ler", 0x38,   null_frag, FP32,  FP32>;
  49:   def LDR : UnaryRR <"ldr", 0x28,   null_frag, FP64,  FP64>;
  50:   def LXR : UnaryRRE<"lxr", 0xB365, null_frag, FP128, FP128>;
  51:   // For z13 we prefer LDR over LER to avoid partial register dependencies.
  52:   let isCodeGenOnly = 1 in {
  53:     def LER16 : UnaryRR<"ler", 0x38, null_frag, FP16, FP16>;
  54:     def LDR16 : UnaryRR<"ldr", 0x28, null_frag, FP16, FP16>;
  55:     def LDR32 : UnaryRR<"ldr", 0x28, null_frag, FP32, FP32>;
  56:   }
```
- **EN**: This block declares or refines TableGen records such as `CondStoreF64`, `LZER_16`, `LZER`, `LZDR`, `LZXR`, `LER`.
- **CN**: 该代码块声明或细化了 `CondStoreF64`, `LZER_16`, `LZER`, `LZDR`, `LZXR`, `LER` 等 TableGen 记录。

### Lines 57-84 / 第 57-84 行
```tablegen
  57: }
  58: 
  59: 
  60: // Moves between two floating-point registers that also set the condition
  61: // codes. Note that these instructions will turn SNaNs into QNaNs and should
  62: // not be used for comparison if the result will be used afterwards.
  63: let Uses = [FPC], mayRaiseFPException = 1,
  64:     Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0xF in {
  65:   def LTEBR : UnaryRRE<"ltebr", 0xB302, null_frag, FP32, FP32>;
  66:   def LTDBR : UnaryRRE<"ltdbr", 0xB312, null_frag, FP64, FP64>;
  67:   def LTXBR : UnaryRRE<"ltxbr", 0xB342, null_frag, FP128, FP128>;
  68: }
  69: 
  70: // Use a load-and-test for compare against zero (via a pseudo to simplify
  71: // instruction selection).
  72: let Uses = [FPC], mayRaiseFPException = 1,
  73:     Defs = [CC], usesCustomInserter = 1, hasNoSchedulingInfo = 1 in {
  74:   def LTEBRCompare_Pseudo : Pseudo<(outs), (ins FP32:$R1), []>;
  75:   def LTDBRCompare_Pseudo : Pseudo<(outs), (ins FP64:$R1), []>;
  76:   def LTXBRCompare_Pseudo : Pseudo<(outs), (ins FP128:$R1), []>;
  77: }
  78: defm : CompareZeroFP<LTEBRCompare_Pseudo, FP32>;
  79: defm : CompareZeroFP<LTDBRCompare_Pseudo, FP64>;
  80: let Predicates = [FeatureNoVectorEnhancements1] in
  81:   defm : CompareZeroFP<LTXBRCompare_Pseudo, FP128>;
  82: 
  83: // Moves between 64-bit integer and floating-point registers.
  84: def LGDR : UnaryRRE<"lgdr", 0xB3CD, bitconvert, GR64, FP64>;
```
- **EN**: This block declares or refines TableGen records such as `LTEBR`, `LTDBR`, `LTXBR`, `LTEBRCompare_Pseudo`, `LTDBRCompare_Pseudo`, `LTXBRCompare_Pseudo`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `LTEBR`, `LTDBR`, `LTXBR`, `LTEBRCompare_Pseudo`, `LTDBRCompare_Pseudo`, `LTXBRCompare_Pseudo` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 85-112 / 第 85-112 行
```tablegen
  85: def LDGR : UnaryRRE<"ldgr", 0xB3C1, bitconvert, FP64, GR64>;
  86: 
  87: // fcopysign with an FP16 result.
  88: let isCodeGenOnly = 1 in {
  89:   def CPSDRhh : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP16, FP16, FP16>;
  90:   def CPSDRhs : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP16, FP16, FP32>;
  91:   def CPSDRhd : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP16, FP16, FP64>;
  92: }
  93: 
  94: // The sign of an FP128 is in the high register.
  95: let Predicates = [FeatureNoVectorEnhancements1] in
  96:   def : Pat<(fcopysign FP16:$src1, (f128 FP128:$src2)),
  97:             (CPSDRhd FP16:$src1, (EXTRACT_SUBREG FP128:$src2, subreg_h64))>;
  98: let Predicates = [FeatureVectorEnhancements1] in
  99:   def : Pat<(fcopysign FP16:$src1, (f128 VR128:$src2)),
 100:             (CPSDRhd FP16:$src1, (EXTRACT_SUBREG VR128:$src2, subreg_h64))>;
 101: 
 102: 
 103: // fcopysign with an FP32 result.
 104: let isCodeGenOnly = 1 in {
 105:   def CPSDRsh : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP32, FP32, FP16>;
 106:   def CPSDRss : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP32, FP32, FP32>;
 107:   def CPSDRsd : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP32, FP32, FP64>;
 108: }
 109: 
 110: // The sign of an FP128 is in the high register.
 111: let Predicates = [FeatureNoVectorEnhancements1] in
 112:   def : Pat<(fcopysign FP32:$src1, (f128 FP128:$src2)),
```
- **EN**: This block declares or refines TableGen records such as `LDGR`, `CPSDRhh`, `CPSDRhs`, `CPSDRhd`, `CPSDRsh`, `CPSDRss`.
- **CN**: 该代码块声明或细化了 `LDGR`, `CPSDRhh`, `CPSDRhs`, `CPSDRhd`, `CPSDRsh`, `CPSDRss` 等 TableGen 记录。

### Lines 113-140 / 第 113-140 行
```tablegen
 113:             (CPSDRsd FP32:$src1, (EXTRACT_SUBREG FP128:$src2, subreg_h64))>;
 114: let Predicates = [FeatureVectorEnhancements1] in
 115:   def : Pat<(fcopysign FP32:$src1, (f128 VR128:$src2)),
 116:             (CPSDRsd FP32:$src1, (EXTRACT_SUBREG VR128:$src2, subreg_h64))>;
 117: 
 118: // fcopysign with an FP64 result.
 119: let isCodeGenOnly = 1 in {
 120:   def CPSDRdh : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP64, FP64, FP16>;
 121:   def CPSDRds : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP64, FP64, FP32>;
 122: }
 123: def CPSDRdd : BinaryRRFb<"cpsdr", 0xB372, fcopysign, FP64, FP64, FP64>;
 124: 
 125: // The sign of an FP128 is in the high register.
 126: let Predicates = [FeatureNoVectorEnhancements1] in
 127:   def : Pat<(fcopysign FP64:$src1, (f128 FP128:$src2)),
 128:             (CPSDRdd FP64:$src1, (EXTRACT_SUBREG FP128:$src2, subreg_h64))>;
 129: let Predicates = [FeatureVectorEnhancements1] in
 130:   def : Pat<(fcopysign FP64:$src1, (f128 VR128:$src2)),
 131:             (CPSDRdd FP64:$src1, (EXTRACT_SUBREG VR128:$src2, subreg_h64))>;
 132: 
 133: // fcopysign with an FP128 result.  Use "upper" as the high half and leave
 134: // the low half as-is.
 135: class CopySign128<RegisterOperand cls, dag upper>
 136:   : Pat<(fcopysign FP128:$src1, cls:$src2),
 137:         (INSERT_SUBREG FP128:$src1, upper, subreg_h64)>;
 138: 
 139: let Predicates = [FeatureNoVectorEnhancements1] in {
 140:   def : CopySign128<FP16,  (CPSDRdh (EXTRACT_SUBREG FP128:$src1, subreg_h64),
```
- **EN**: This block declares or refines TableGen records such as `CPSDRdh`, `CPSDRds`, `CPSDRdd`, `CopySign128`.
- **CN**: 该代码块声明或细化了 `CPSDRdh`, `CPSDRds`, `CPSDRdd`, `CopySign128` 等 TableGen 记录。

### Lines 141-168 / 第 141-168 行
```tablegen
 141:                                     FP16:$src2)>;
 142:   def : CopySign128<FP32,  (CPSDRds (EXTRACT_SUBREG FP128:$src1, subreg_h64),
 143:                                     FP32:$src2)>;
 144:   def : CopySign128<FP64,  (CPSDRdd (EXTRACT_SUBREG FP128:$src1, subreg_h64),
 145:                                     FP64:$src2)>;
 146:   def : CopySign128<FP128, (CPSDRdd (EXTRACT_SUBREG FP128:$src1, subreg_h64),
 147:                                     (EXTRACT_SUBREG FP128:$src2, subreg_h64))>;
 148: }
 149: 
 150: // The length is given as one less for MVCImm.
 151: defm LoadStoreF32  : MVCLoadStore<load, f32,  MVCImm, 3>;
 152: defm LoadStoreF64  : MVCLoadStore<load, f64,  MVCImm, 7>;
 153: defm LoadStoreF128 : MVCLoadStore<load, f128, MVCImm, 15>;
 154: 
 155: //===----------------------------------------------------------------------===//
 156: // Load instructions
 157: //===----------------------------------------------------------------------===//
 158: 
 159: let canFoldAsLoad = 1, SimpleBDXLoad = 1, mayLoad = 1,
 160:     isReMaterializable = 1  in {
 161:   let isCodeGenOnly = 1 in
 162:     // Reload f16 from 4-byte spill slot.
 163:     defm LE16 : UnaryRXPair<"le", 0x78, 0xED64, z_load, FP16, 4>;
 164:   defm LE : UnaryRXPair<"le", 0x78, 0xED64, z_load, FP32, 4>;
 165:   defm LD : UnaryRXPair<"ld", 0x68, 0xED65, z_load, FP64, 8>;
 166: 
 167:   // For z13 we prefer LDE over LE to avoid partial register dependencies.
 168:   let isCodeGenOnly = 1 in
```
- **EN**: This block declares or refines TableGen records such as `LoadStoreF32`, `LoadStoreF64`, `LoadStoreF128`, `LE16`, `LE`, `LD`.
- **CN**: 该代码块声明或细化了 `LoadStoreF32`, `LoadStoreF64`, `LoadStoreF128`, `LE16`, `LE`, `LD` 等 TableGen 记录。

### Lines 169-196 / 第 169-196 行
```tablegen
 169:     def LDE32 : UnaryRXE<"lde", 0xED24, null_frag, FP32, 4>;
 170: 
 171:   // These instructions are split after register allocation, so we don't
 172:   // want a custom inserter.
 173:   let Has20BitOffset = 1, HasIndex = 1, Is128Bit = 1 in {
 174:     def LX : Pseudo<(outs FP128:$dst), (ins bdxaddr20only128:$src),
 175:                      [(set FP128:$dst, (load bdxaddr20only128:$src))]>;
 176:   }
 177: }
 178: 
 179: //===----------------------------------------------------------------------===//
 180: // Store instructions
 181: //===----------------------------------------------------------------------===//
 182: 
 183: let SimpleBDXStore = 1, mayStore = 1 in {
 184:   let isCodeGenOnly = 1 in
 185:     // Spill f16 to 4-byte spill slot.
 186:     defm STE16 : StoreRXPair<"ste", 0x70, 0xED66, store, FP16, 4>;
 187:   defm STE : StoreRXPair<"ste", 0x70, 0xED66, store, FP32, 4>;
 188:   defm STD : StoreRXPair<"std", 0x60, 0xED67, store, FP64, 8>;
 189: 
 190:   // These instructions are split after register allocation, so we don't
 191:   // want a custom inserter.
 192:   let Has20BitOffset = 1, HasIndex = 1, Is128Bit = 1 in {
 193:     def STX : Pseudo<(outs), (ins FP128:$src, bdxaddr20only128:$dst),
 194:                      [(store FP128:$src, bdxaddr20only128:$dst)]>;
 195:   }
 196: }
```
- **EN**: This block declares or refines TableGen records such as `LDE32`, `LX`, `STE16`, `STE`, `STD`, `STX`.
- **CN**: 该代码块声明或细化了 `LDE32`, `LX`, `STE16`, `STE`, `STD`, `STX` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```tablegen
 197: 
 198: //===----------------------------------------------------------------------===//
 199: // Conversion instructions
 200: //===----------------------------------------------------------------------===//
 201: 
 202: // Convert floating-point values to narrower representations, rounding
 203: // according to the current mode.  The destination of LEXBR and LDXBR
 204: // is a 128-bit value, but only the first register of the pair is used.
 205: let Uses = [FPC], mayRaiseFPException = 1 in {
 206:   def LEDBR : UnaryRRE<"ledbr", 0xB344, any_fpround, FP32, FP64>;
 207:   def LEXBR : UnaryRRE<"lexbr", 0xB346, null_frag, FP128, FP128>;
 208:   def LDXBR : UnaryRRE<"ldxbr", 0xB345, null_frag, FP128, FP128>;
 209: 
 210:   def LEDBRA : TernaryRRFe<"ledbra", 0xB344, FP32,  FP64>,
 211:                Requires<[FeatureFPExtension]>;
 212:   def LEXBRA : TernaryRRFe<"lexbra", 0xB346, FP128, FP128>,
 213:                Requires<[FeatureFPExtension]>;
 214:   def LDXBRA : TernaryRRFe<"ldxbra", 0xB345, FP128, FP128>,
 215:                Requires<[FeatureFPExtension]>;
 216: }
 217: 
 218: let Predicates = [FeatureNoVectorEnhancements1] in {
 219:   def : Pat<(f32 (any_fpround FP128:$src)),
 220:             (EXTRACT_SUBREG (LEXBR FP128:$src), subreg_h32)>;
 221:   def : Pat<(f64 (any_fpround FP128:$src)),
 222:             (EXTRACT_SUBREG (LDXBR FP128:$src), subreg_h64)>;
 223: }
 224: 
```
- **EN**: This block declares or refines TableGen records such as `LEDBR`, `LEXBR`, `LDXBR`, `LEDBRA`, `LEXBRA`, `LDXBRA`.
- **CN**: 该代码块声明或细化了 `LEDBR`, `LEXBR`, `LDXBR`, `LEDBRA`, `LEXBRA`, `LDXBRA` 等 TableGen 记录。

### Lines 225-252 / 第 225-252 行
```tablegen
 225: // Extend register floating-point values to wider representations.
 226: let Uses = [FPC], mayRaiseFPException = 1 in {
 227:   def LDEBR : UnaryRRE<"ldebr", 0xB304, any_fpextend, FP64, FP32>;
 228:   def LXEBR : UnaryRRE<"lxebr", 0xB306, null_frag, FP128, FP32>;
 229:   def LXDBR : UnaryRRE<"lxdbr", 0xB305, null_frag, FP128, FP64>;
 230: }
 231: let Predicates = [FeatureNoVectorEnhancements1] in {
 232:   def : Pat<(f128 (any_fpextend (f32 FP32:$src))), (LXEBR FP32:$src)>;
 233:   def : Pat<(f128 (any_fpextend (f64 FP64:$src))), (LXDBR FP64:$src)>;
 234: }
 235: 
 236: // Extend memory floating-point values to wider representations.
 237: let Uses = [FPC], mayRaiseFPException = 1 in {
 238:   def LDEB : UnaryRXE<"ldeb", 0xED04, z_any_extloadf32, FP64, 4>;
 239:   def LXEB : UnaryRXE<"lxeb", 0xED06, null_frag, FP128, 4>;
 240:   def LXDB : UnaryRXE<"lxdb", 0xED05, null_frag, FP128, 8>;
 241: }
 242: let Predicates = [FeatureNoVectorEnhancements1] in {
 243:   def : Pat<(f128 (z_any_extloadf32 bdxaddr12only:$src)),
 244:             (LXEB bdxaddr12only:$src)>;
 245:   def : Pat<(f128 (z_any_extloadf64 bdxaddr12only:$src)),
 246:             (LXDB bdxaddr12only:$src)>;
 247: }
 248: 
 249: // Convert a signed integer register value to a floating-point one.
 250: let Uses = [FPC], mayRaiseFPException = 1 in {
 251:   def CEFBR : UnaryRRE<"cefbr", 0xB394, any_sint_to_fp, FP32,  GR32>;
 252:   def CDFBR : UnaryRRE<"cdfbr", 0xB395, any_sint_to_fp, FP64,  GR32>;
```
- **EN**: This block declares or refines TableGen records such as `LDEBR`, `LXEBR`, `LXDBR`, `LDEB`, `LXEB`, `LXDB`.
- **CN**: 该代码块声明或细化了 `LDEBR`, `LXEBR`, `LXDBR`, `LDEB`, `LXEB`, `LXDB` 等 TableGen 记录。

### Lines 253-280 / 第 253-280 行
```tablegen
 253:   def CXFBR : UnaryRRE<"cxfbr", 0xB396, any_sint_to_fp, FP128, GR32>;
 254: 
 255:   def CEGBR : UnaryRRE<"cegbr", 0xB3A4, any_sint_to_fp, FP32,  GR64>;
 256:   def CDGBR : UnaryRRE<"cdgbr", 0xB3A5, any_sint_to_fp, FP64,  GR64>;
 257:   def CXGBR : UnaryRRE<"cxgbr", 0xB3A6, any_sint_to_fp, FP128, GR64>;
 258: }
 259: 
 260: // The FP extension feature provides versions of the above that allow
 261: // specifying rounding mode and inexact-exception suppression flags.
 262: let Uses = [FPC], mayRaiseFPException = 1, Predicates = [FeatureFPExtension] in {
 263:   def CEFBRA : TernaryRRFe<"cefbra", 0xB394, FP32,  GR32>;
 264:   def CDFBRA : TernaryRRFe<"cdfbra", 0xB395, FP64,  GR32>;
 265:   def CXFBRA : TernaryRRFe<"cxfbra", 0xB396, FP128, GR32>;
 266: 
 267:   def CEGBRA : TernaryRRFe<"cegbra", 0xB3A4, FP32,  GR64>;
 268:   def CDGBRA : TernaryRRFe<"cdgbra", 0xB3A5, FP64,  GR64>;
 269:   def CXGBRA : TernaryRRFe<"cxgbra", 0xB3A6, FP128, GR64>;
 270: }
 271: 
 272: // Convert an unsigned integer register value to a floating-point one.
 273: let Predicates = [FeatureFPExtension] in {
 274:   let Uses = [FPC], mayRaiseFPException = 1 in {
 275:     def CELFBR : TernaryRRFe<"celfbr", 0xB390, FP32,  GR32>;
 276:     def CDLFBR : TernaryRRFe<"cdlfbr", 0xB391, FP64,  GR32>;
 277:     def CXLFBR : TernaryRRFe<"cxlfbr", 0xB392, FP128, GR32>;
 278: 
 279:     def CELGBR : TernaryRRFe<"celgbr", 0xB3A0, FP32,  GR64>;
 280:     def CDLGBR : TernaryRRFe<"cdlgbr", 0xB3A1, FP64,  GR64>;
```
- **EN**: This block declares or refines TableGen records such as `CXFBR`, `CEGBR`, `CDGBR`, `CXGBR`, `CEFBRA`, `CDFBRA`.
- **CN**: 该代码块声明或细化了 `CXFBR`, `CEGBR`, `CDGBR`, `CXGBR`, `CEFBRA`, `CDFBRA` 等 TableGen 记录。

### Lines 281-308 / 第 281-308 行
```tablegen
 281:     def CXLGBR : TernaryRRFe<"cxlgbr", 0xB3A2, FP128, GR64>;
 282:   }
 283: 
 284:   def : Pat<(f32  (any_uint_to_fp GR32:$src)), (CELFBR 0, GR32:$src, 0)>;
 285:   def : Pat<(f64  (any_uint_to_fp GR32:$src)), (CDLFBR 0, GR32:$src, 0)>;
 286:   def : Pat<(f128 (any_uint_to_fp GR32:$src)), (CXLFBR 0, GR32:$src, 0)>;
 287: 
 288:   def : Pat<(f32  (any_uint_to_fp GR64:$src)), (CELGBR 0, GR64:$src, 0)>;
 289:   def : Pat<(f64  (any_uint_to_fp GR64:$src)), (CDLGBR 0, GR64:$src, 0)>;
 290:   def : Pat<(f128 (any_uint_to_fp GR64:$src)), (CXLGBR 0, GR64:$src, 0)>;
 291: }
 292: 
 293: // Convert a floating-point register value to a signed integer value,
 294: // with the second operand (modifier M3) specifying the rounding mode.
 295: let Uses = [FPC], mayRaiseFPException = 1, Defs = [CC] in {
 296:   def CFEBR : BinaryRRFe<"cfebr", 0xB398, GR32, FP32>;
 297:   def CFDBR : BinaryRRFe<"cfdbr", 0xB399, GR32, FP64>;
 298:   def CFXBR : BinaryRRFe<"cfxbr", 0xB39A, GR32, FP128>;
 299: 
 300:   def CGEBR : BinaryRRFe<"cgebr", 0xB3A8, GR64, FP32>;
 301:   def CGDBR : BinaryRRFe<"cgdbr", 0xB3A9, GR64, FP64>;
 302:   def CGXBR : BinaryRRFe<"cgxbr", 0xB3AA, GR64, FP128>;
 303: }
 304: 
 305: // fp_to_sint always rounds towards zero, which is modifier value 5.
 306: def : Pat<(i32 (any_fp_to_sint FP32:$src)),  (CFEBR 5, FP32:$src)>;
 307: def : Pat<(i32 (any_fp_to_sint FP64:$src)),  (CFDBR 5, FP64:$src)>;
 308: def : Pat<(i32 (any_fp_to_sint FP128:$src)), (CFXBR 5, FP128:$src)>;
```
- **EN**: This block declares or refines TableGen records such as `CXLGBR`, `CFEBR`, `CFDBR`, `CFXBR`, `CGEBR`, `CGDBR`.
- **CN**: 该代码块声明或细化了 `CXLGBR`, `CFEBR`, `CFDBR`, `CFXBR`, `CGEBR`, `CGDBR` 等 TableGen 记录。

### Lines 309-336 / 第 309-336 行
```tablegen
 309: 
 310: def : Pat<(i64 (any_fp_to_sint FP32:$src)),  (CGEBR 5, FP32:$src)>;
 311: def : Pat<(i64 (any_fp_to_sint FP64:$src)),  (CGDBR 5, FP64:$src)>;
 312: def : Pat<(i64 (any_fp_to_sint FP128:$src)), (CGXBR 5, FP128:$src)>;
 313: 
 314: // The FP extension feature provides versions of the above that allow
 315: // also specifying the inexact-exception suppression flag.
 316: let Uses = [FPC], mayRaiseFPException = 1,
 317:     Predicates = [FeatureFPExtension], Defs = [CC] in {
 318:   def CFEBRA : TernaryRRFe<"cfebra", 0xB398, GR32, FP32>;
 319:   def CFDBRA : TernaryRRFe<"cfdbra", 0xB399, GR32, FP64>;
 320:   def CFXBRA : TernaryRRFe<"cfxbra", 0xB39A, GR32, FP128>;
 321: 
 322:   def CGEBRA : TernaryRRFe<"cgebra", 0xB3A8, GR64, FP32>;
 323:   def CGDBRA : TernaryRRFe<"cgdbra", 0xB3A9, GR64, FP64>;
 324:   def CGXBRA : TernaryRRFe<"cgxbra", 0xB3AA, GR64, FP128>;
 325: }
 326: 
 327: // Convert a floating-point register value to an unsigned integer value.
 328: let Predicates = [FeatureFPExtension] in {
 329:   let Uses = [FPC], mayRaiseFPException = 1, Defs = [CC] in {
 330:     def CLFEBR : TernaryRRFe<"clfebr", 0xB39C, GR32, FP32>;
 331:     def CLFDBR : TernaryRRFe<"clfdbr", 0xB39D, GR32, FP64>;
 332:     def CLFXBR : TernaryRRFe<"clfxbr", 0xB39E, GR32, FP128>;
 333: 
 334:     def CLGEBR : TernaryRRFe<"clgebr", 0xB3AC, GR64, FP32>;
 335:     def CLGDBR : TernaryRRFe<"clgdbr", 0xB3AD, GR64, FP64>;
 336:     def CLGXBR : TernaryRRFe<"clgxbr", 0xB3AE, GR64, FP128>;
```
- **EN**: This block declares or refines TableGen records such as `CFEBRA`, `CFDBRA`, `CFXBRA`, `CGEBRA`, `CGDBRA`, `CGXBRA`.
- **CN**: 该代码块声明或细化了 `CFEBRA`, `CFDBRA`, `CFXBRA`, `CGEBRA`, `CGDBRA`, `CGXBRA` 等 TableGen 记录。

### Lines 337-364 / 第 337-364 行
```tablegen
 337:   }
 338: 
 339:   def : Pat<(i32 (any_fp_to_uint FP32:$src)),  (CLFEBR 5, FP32:$src,  0)>;
 340:   def : Pat<(i32 (any_fp_to_uint FP64:$src)),  (CLFDBR 5, FP64:$src,  0)>;
 341:   def : Pat<(i32 (any_fp_to_uint FP128:$src)), (CLFXBR 5, FP128:$src, 0)>;
 342: 
 343:   def : Pat<(i64 (any_fp_to_uint FP32:$src)),  (CLGEBR 5, FP32:$src,  0)>;
 344:   def : Pat<(i64 (any_fp_to_uint FP64:$src)),  (CLGDBR 5, FP64:$src,  0)>;
 345:   def : Pat<(i64 (any_fp_to_uint FP128:$src)), (CLGXBR 5, FP128:$src, 0)>;
 346: }
 347: 
 348: 
 349: //===----------------------------------------------------------------------===//
 350: // Unary arithmetic
 351: //===----------------------------------------------------------------------===//
 352: 
 353: // We prefer generic instructions during isel, because they do not
 354: // clobber CC and therefore give the scheduler more freedom. In cases
 355: // the CC is actually useful, the SystemZElimCompare pass will try to
 356: // convert generic instructions into opcodes that also set CC. Note
 357: // that lcdf / lpdf / lndf only affect the sign bit, and can therefore
 358: // be used with fp32 as well. This could be done for fp128, in which
 359: // case the operands would have to be tied.
 360: 
 361: // Negation (Load Complement).
 362: let Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0xF in {
 363:   def LCEBR : UnaryRRE<"lcebr", 0xB303, null_frag, FP32,  FP32>;
 364:   def LCDBR : UnaryRRE<"lcdbr", 0xB313, null_frag, FP64,  FP64>;
```
- **EN**: This block declares or refines TableGen records such as `LCEBR`, `LCDBR`.
- **CN**: 该代码块声明或细化了 `LCEBR`, `LCDBR` 等 TableGen 记录。

### Lines 365-392 / 第 365-392 行
```tablegen
 365:   def LCXBR : UnaryRRE<"lcxbr", 0xB343, fneg, FP128, FP128>;
 366: }
 367: // Generic form, which does not set CC.
 368: def LCDFR : UnaryRRE<"lcdfr", 0xB373, fneg, FP64,  FP64>;
 369: let isCodeGenOnly = 1 in {
 370:   def LCDFR_16 : UnaryRRE<"lcdfr", 0xB373, fneg, FP16,  FP16>;
 371:   def LCDFR_32 : UnaryRRE<"lcdfr", 0xB373, fneg, FP32,  FP32>;
 372: }
 373: 
 374: // Absolute value (Load Positive).
 375: let Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0xF in {
 376:   def LPEBR : UnaryRRE<"lpebr", 0xB300, null_frag, FP32,  FP32>;
 377:   def LPDBR : UnaryRRE<"lpdbr", 0xB310, null_frag, FP64,  FP64>;
 378:   def LPXBR : UnaryRRE<"lpxbr", 0xB340, fabs, FP128, FP128>;
 379: }
 380: // Generic form, which does not set CC.
 381: def LPDFR : UnaryRRE<"lpdfr", 0xB370, fabs, FP64,  FP64>;
 382: let isCodeGenOnly = 1 in {
 383:   def LPDFR_16 : UnaryRRE<"lpdfr", 0xB370, fabs, FP16,  FP16>;
 384:   def LPDFR_32 : UnaryRRE<"lpdfr", 0xB370, fabs, FP32,  FP32>;
 385: }
 386: 
 387: // Negative absolute value (Load Negative).
 388: let Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0xF in {
 389:   def LNEBR : UnaryRRE<"lnebr", 0xB301, null_frag, FP32,  FP32>;
 390:   def LNDBR : UnaryRRE<"lndbr", 0xB311, null_frag, FP64,  FP64>;
 391:   def LNXBR : UnaryRRE<"lnxbr", 0xB341, fnabs, FP128, FP128>;
 392: }
```
- **EN**: This block declares or refines TableGen records such as `LCXBR`, `LCDFR`, `LCDFR_16`, `LCDFR_32`, `LPEBR`, `LPDBR`.
- **CN**: 该代码块声明或细化了 `LCXBR`, `LCDFR`, `LCDFR_16`, `LCDFR_32`, `LPEBR`, `LPDBR` 等 TableGen 记录。

### Lines 393-420 / 第 393-420 行
```tablegen
 393: // Generic form, which does not set CC.
 394: def LNDFR : UnaryRRE<"lndfr", 0xB371, fnabs, FP64,  FP64>;
 395: let isCodeGenOnly = 1 in {
 396:   def LNDFR_16 : UnaryRRE<"lndfr", 0xB371, fnabs, FP16,  FP16>;
 397:   def LNDFR_32 : UnaryRRE<"lndfr", 0xB371, fnabs, FP32,  FP32>;
 398: }
 399: 
 400: // Square root.
 401: let Uses = [FPC], mayRaiseFPException = 1 in {
 402:   def SQEBR : UnaryRRE<"sqebr", 0xB314, any_fsqrt, FP32,  FP32>;
 403:   def SQDBR : UnaryRRE<"sqdbr", 0xB315, any_fsqrt, FP64,  FP64>;
 404:   def SQXBR : UnaryRRE<"sqxbr", 0xB316, any_fsqrt, FP128, FP128>;
 405: 
 406:   def SQEB : UnaryRXE<"sqeb", 0xED14, loadu<any_fsqrt>, FP32, 4>;
 407:   def SQDB : UnaryRXE<"sqdb", 0xED15, loadu<any_fsqrt>, FP64, 8>;
 408: }
 409: 
 410: // Round to an integer, with the second operand (modifier M3) specifying
 411: // the rounding mode.  These forms always check for inexact conditions.
 412: let Uses = [FPC], mayRaiseFPException = 1 in {
 413:   def FIEBR : BinaryRRFe<"fiebr", 0xB357, FP32,  FP32>;
 414:   def FIDBR : BinaryRRFe<"fidbr", 0xB35F, FP64,  FP64>;
 415:   def FIXBR : BinaryRRFe<"fixbr", 0xB347, FP128, FP128>;
 416: }
 417: 
 418: // frint rounds according to the current mode (modifier 0) and detects
 419: // inexact conditions.
 420: def : Pat<(any_frint FP32:$src),  (FIEBR 0, FP32:$src)>;
```
- **EN**: This block declares or refines TableGen records such as `LNDFR`, `LNDFR_16`, `LNDFR_32`, `SQEBR`, `SQDBR`, `SQXBR`.
- **CN**: 该代码块声明或细化了 `LNDFR`, `LNDFR_16`, `LNDFR_32`, `SQEBR`, `SQDBR`, `SQXBR` 等 TableGen 记录。

### Lines 421-448 / 第 421-448 行
```tablegen
 421: def : Pat<(any_frint FP64:$src),  (FIDBR 0, FP64:$src)>;
 422: def : Pat<(any_frint FP128:$src), (FIXBR 0, FP128:$src)>;
 423: 
 424: let Predicates = [FeatureFPExtension] in {
 425:   // Extended forms of the FIxBR instructions.  M4 can be set to 4
 426:   // to suppress detection of inexact conditions.
 427:   let Uses = [FPC], mayRaiseFPException = 1 in {
 428:     def FIEBRA : TernaryRRFe<"fiebra", 0xB357, FP32,  FP32>;
 429:     def FIDBRA : TernaryRRFe<"fidbra", 0xB35F, FP64,  FP64>;
 430:     def FIXBRA : TernaryRRFe<"fixbra", 0xB347, FP128, FP128>;
 431:   }
 432: 
 433:   // fnearbyint is like frint but does not detect inexact conditions.
 434:   def : Pat<(any_fnearbyint FP32:$src),  (FIEBRA 0, FP32:$src,  4)>;
 435:   def : Pat<(any_fnearbyint FP64:$src),  (FIDBRA 0, FP64:$src,  4)>;
 436:   def : Pat<(any_fnearbyint FP128:$src), (FIXBRA 0, FP128:$src, 4)>;
 437: 
 438:   // floor is no longer allowed to raise an inexact condition,
 439:   // so restrict it to the cases where the condition can be suppressed.
 440:   // Mode 7 is round towards -inf.
 441:   def : Pat<(any_ffloor FP32:$src),  (FIEBRA 7, FP32:$src,  4)>;
 442:   def : Pat<(any_ffloor FP64:$src),  (FIDBRA 7, FP64:$src,  4)>;
 443:   def : Pat<(any_ffloor FP128:$src), (FIXBRA 7, FP128:$src, 4)>;
 444: 
 445:   // Same idea for ceil, where mode 6 is round towards +inf.
 446:   def : Pat<(any_fceil FP32:$src),  (FIEBRA 6, FP32:$src,  4)>;
 447:   def : Pat<(any_fceil FP64:$src),  (FIDBRA 6, FP64:$src,  4)>;
 448:   def : Pat<(any_fceil FP128:$src), (FIXBRA 6, FP128:$src, 4)>;
```
- **EN**: This block declares or refines TableGen records such as `FIEBRA`, `FIDBRA`, `FIXBRA`.
- **CN**: 该代码块声明或细化了 `FIEBRA`, `FIDBRA`, `FIXBRA` 等 TableGen 记录。

### Lines 449-476 / 第 449-476 行
```tablegen
 449: 
 450:   // Same idea for trunc, where mode 5 is round towards zero.
 451:   def : Pat<(any_ftrunc FP32:$src),  (FIEBRA 5, FP32:$src,  4)>;
 452:   def : Pat<(any_ftrunc FP64:$src),  (FIDBRA 5, FP64:$src,  4)>;
 453:   def : Pat<(any_ftrunc FP128:$src), (FIXBRA 5, FP128:$src, 4)>;
 454: 
 455:   // Same idea for roundeven, where mode 4 is round towards nearest
 456:   // with ties to even.
 457:   def : Pat<(any_froundeven FP32:$src),  (FIEBRA 4, FP32:$src,  4)>;
 458:   def : Pat<(any_froundeven FP64:$src),  (FIDBRA 4, FP64:$src,  4)>;
 459:   def : Pat<(any_froundeven FP128:$src), (FIXBRA 4, FP128:$src, 4)>;
 460: 
 461:   // Same idea for round, where mode 1 is round towards nearest with
 462:   // ties away from zero.
 463:   def : Pat<(any_fround FP32:$src),  (FIEBRA 1, FP32:$src,  4)>;
 464:   def : Pat<(any_fround FP64:$src),  (FIDBRA 1, FP64:$src,  4)>;
 465:   def : Pat<(any_fround FP128:$src), (FIXBRA 1, FP128:$src, 4)>;
 466: }
 467: 
 468: //===----------------------------------------------------------------------===//
 469: // Binary arithmetic
 470: //===----------------------------------------------------------------------===//
 471: 
 472: // Addition.
 473: let Uses = [FPC], mayRaiseFPException = 1,
 474:     Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0xF in {
 475:   let isCommutable = 1 in {
 476:     def AEBR : BinaryRRE<"aebr", 0xB30A, any_fadd, FP32,  FP32>;
```
- **EN**: This block declares or refines TableGen records such as `AEBR`.
- **CN**: 该代码块声明或细化了 `AEBR` 等 TableGen 记录。

### Lines 477-504 / 第 477-504 行
```tablegen
 477:     def ADBR : BinaryRRE<"adbr", 0xB31A, any_fadd, FP64,  FP64>;
 478:     def AXBR : BinaryRRE<"axbr", 0xB34A, any_fadd, FP128, FP128>;
 479:   }
 480:   defm AEB : BinaryRXEAndPseudo<"aeb", 0xED0A, z_any_fadd_noreassoc, FP32,
 481:                                 z_load, 4>;
 482:   defm ADB : BinaryRXEAndPseudo<"adb", 0xED1A, z_any_fadd_noreassoc, FP64,
 483:                                 z_load, 8>;
 484: }
 485: 
 486: // Subtraction.
 487: let Uses = [FPC], mayRaiseFPException = 1,
 488:     Defs = [CC], CCValues = 0xF, CompareZeroCCMask = 0xF in {
 489:   def SEBR : BinaryRRE<"sebr", 0xB30B, any_fsub, FP32,  FP32>;
 490:   def SDBR : BinaryRRE<"sdbr", 0xB31B, any_fsub, FP64,  FP64>;
 491:   def SXBR : BinaryRRE<"sxbr", 0xB34B, any_fsub, FP128, FP128>;
 492: 
 493:   defm SEB : BinaryRXEAndPseudo<"seb",  0xED0B, z_any_fsub_noreassoc, FP32,
 494:                                 z_load, 4>;
 495:   defm SDB : BinaryRXEAndPseudo<"sdb",  0xED1B, z_any_fsub_noreassoc, FP64,
 496:                                 z_load, 8>;
 497: }
 498: 
 499: // Multiplication.
 500: let Uses = [FPC], mayRaiseFPException = 1 in {
 501:   let isCommutable = 1 in {
 502:     def MEEBR : BinaryRRE<"meebr", 0xB317, any_fmul, FP32,  FP32>;
 503:     def MDBR  : BinaryRRE<"mdbr",  0xB31C, any_fmul, FP64,  FP64>;
 504:     def MXBR  : BinaryRRE<"mxbr",  0xB34C, any_fmul, FP128, FP128>;
```
- **EN**: This block declares or refines TableGen records such as `ADBR`, `AXBR`, `AEB`, `ADB`, `SEBR`, `SDBR`.
- **CN**: 该代码块声明或细化了 `ADBR`, `AXBR`, `AEB`, `ADB`, `SEBR`, `SDBR` 等 TableGen 记录。

### Lines 505-532 / 第 505-532 行
```tablegen
 505:   }
 506:   defm MEEB : BinaryRXEAndPseudo<"meeb", 0xED17, z_any_fmul_noreassoc, FP32,
 507:                                   z_load, 4>;
 508:   defm MDB  : BinaryRXEAndPseudo<"mdb",  0xED1C, z_any_fmul_noreassoc, FP64,
 509:                                   z_load, 8>;
 510: }
 511: 
 512: // f64 multiplication of two FP32 registers.
 513: let Uses = [FPC], mayRaiseFPException = 1 in
 514:   def MDEBR : BinaryRRE<"mdebr", 0xB30C, null_frag, FP64, FP32>;
 515: def : Pat<(any_fmul (f64 (any_fpextend FP32:$src1)),
 516:                     (f64 (any_fpextend FP32:$src2))),
 517:           (MDEBR (INSERT_SUBREG (f64 (IMPLICIT_DEF)),
 518:                                 FP32:$src1, subreg_h32), FP32:$src2)>;
 519: 
 520: // f64 multiplication of an FP32 register and an f32 memory.
 521: let Uses = [FPC], mayRaiseFPException = 1 in
 522:   def MDEB : BinaryRXE<"mdeb", 0xED0C, null_frag, FP64, z_load, 4>;
 523: def : Pat<(any_fmul (f64 (any_fpextend FP32:$src1)),
 524:                     (f64 (any_extloadf32 bdxaddr12only:$addr))),
 525:           (MDEB (INSERT_SUBREG (f64 (IMPLICIT_DEF)), FP32:$src1, subreg_h32),
 526:                 bdxaddr12only:$addr)>;
 527: 
 528: // f128 multiplication of two FP64 registers.
 529: let Uses = [FPC], mayRaiseFPException = 1 in
 530:   def MXDBR : BinaryRRE<"mxdbr", 0xB307, null_frag, FP128, FP64>;
 531: let Predicates = [FeatureNoVectorEnhancements1] in
 532:   def : Pat<(any_fmul (f128 (any_fpextend FP64:$src1)),
```
- **EN**: This block declares or refines TableGen records such as `MEEB`, `MDB`, `MDEBR`, `MDEB`, `MXDBR`.
- **CN**: 该代码块声明或细化了 `MEEB`, `MDB`, `MDEBR`, `MDEB`, `MXDBR` 等 TableGen 记录。

### Lines 533-560 / 第 533-560 行
```tablegen
 533:                       (f128 (any_fpextend FP64:$src2))),
 534:             (MXDBR (INSERT_SUBREG (f128 (IMPLICIT_DEF)),
 535:                                   FP64:$src1, subreg_h64), FP64:$src2)>;
 536: 
 537: // f128 multiplication of an FP64 register and an f64 memory.
 538: let Uses = [FPC], mayRaiseFPException = 1 in
 539:   def MXDB : BinaryRXE<"mxdb", 0xED07, null_frag, FP128, z_load, 8>;
 540: let Predicates = [FeatureNoVectorEnhancements1] in
 541:   def : Pat<(any_fmul (f128 (any_fpextend FP64:$src1)),
 542:                       (f128 (any_extloadf64 bdxaddr12only:$addr))),
 543:             (MXDB (INSERT_SUBREG (f128 (IMPLICIT_DEF)), FP64:$src1, subreg_h64),
 544:                   bdxaddr12only:$addr)>;
 545: 
 546: // Fused multiply-add.
 547: let Uses = [FPC], mayRaiseFPException = 1 in {
 548:   def MAEBR : TernaryRRD<"maebr", 0xB30E, z_any_fma, FP32, FP32>;
 549:   def MADBR : TernaryRRD<"madbr", 0xB31E, z_any_fma, FP64, FP64>;
 550: 
 551:   defm MAEB : TernaryRXFAndPseudo<"maeb", 0xED0E, z_any_fma, FP32, FP32, z_load, 4>;
 552:   defm MADB : TernaryRXFAndPseudo<"madb", 0xED1E, z_any_fma, FP64, FP64, z_load, 8>;
 553: }
 554: 
 555: // Fused multiply-subtract.
 556: let Uses = [FPC], mayRaiseFPException = 1 in {
 557:   def MSEBR : TernaryRRD<"msebr", 0xB30F, z_any_fms, FP32, FP32>;
 558:   def MSDBR : TernaryRRD<"msdbr", 0xB31F, z_any_fms, FP64, FP64>;
 559: 
 560:   defm MSEB : TernaryRXFAndPseudo<"mseb", 0xED0F, z_any_fms, FP32, FP32, z_load, 4>;
```
- **EN**: This block declares or refines TableGen records such as `MXDB`, `MAEBR`, `MADBR`, `MAEB`, `MADB`, `MSEBR`.
- **CN**: 该代码块声明或细化了 `MXDB`, `MAEBR`, `MADBR`, `MAEB`, `MADB`, `MSEBR` 等 TableGen 记录。

### Lines 561-588 / 第 561-588 行
```tablegen
 561:   defm MSDB : TernaryRXFAndPseudo<"msdb", 0xED1F, z_any_fms, FP64, FP64, z_load, 8>;
 562: }
 563: 
 564: // Division.
 565: let Uses = [FPC], mayRaiseFPException = 1 in {
 566:   def DEBR : BinaryRRE<"debr", 0xB30D, any_fdiv, FP32,  FP32>;
 567:   def DDBR : BinaryRRE<"ddbr", 0xB31D, any_fdiv, FP64,  FP64>;
 568:   def DXBR : BinaryRRE<"dxbr", 0xB34D, any_fdiv, FP128, FP128>;
 569: 
 570:   defm DEB : BinaryRXEAndPseudo<"deb", 0xED0D, any_fdiv, FP32, z_load, 4>;
 571:   defm DDB : BinaryRXEAndPseudo<"ddb", 0xED1D, any_fdiv, FP64, z_load, 8>;
 572: }
 573: 
 574: // Divide to integer.
 575: let Uses = [FPC], mayRaiseFPException = 1, Defs = [CC] in {
 576:   def DIEBR : TernaryRRFb<"diebr", 0xB353, FP32, FP32, FP32>;
 577:   def DIDBR : TernaryRRFb<"didbr", 0xB35B, FP64, FP64, FP64>;
 578: }
 579: 
 580: //===----------------------------------------------------------------------===//
 581: // Comparisons
 582: //===----------------------------------------------------------------------===//
 583: 
 584: let Uses = [FPC], mayRaiseFPException = 1, Defs = [CC], CCValues = 0xF in {
 585:   def CEBR : CompareRRE<"cebr", 0xB309, z_any_fcmp, FP32,  FP32>;
 586:   def CDBR : CompareRRE<"cdbr", 0xB319, z_any_fcmp, FP64,  FP64>;
 587:   def CXBR : CompareRRE<"cxbr", 0xB349, z_any_fcmp, FP128, FP128>;
 588: 
```
- **EN**: This block declares or refines TableGen records such as `MSDB`, `DEBR`, `DDBR`, `DXBR`, `DEB`, `DDB`.
- **CN**: 该代码块声明或细化了 `MSDB`, `DEBR`, `DDBR`, `DXBR`, `DEB`, `DDB` 等 TableGen 记录。

### Lines 589-616 / 第 589-616 行
```tablegen
 589:   def CEB : CompareRXE<"ceb", 0xED09, z_any_fcmp, FP32, z_load, 4>;
 590:   def CDB : CompareRXE<"cdb", 0xED19, z_any_fcmp, FP64, z_load, 8>;
 591: 
 592:   def KEBR : CompareRRE<"kebr", 0xB308, z_strict_fcmps, FP32,  FP32>;
 593:   def KDBR : CompareRRE<"kdbr", 0xB318, z_strict_fcmps, FP64,  FP64>;
 594:   def KXBR : CompareRRE<"kxbr", 0xB348, z_strict_fcmps, FP128, FP128>;
 595: 
 596:   def KEB : CompareRXE<"keb", 0xED08, z_strict_fcmps, FP32, z_load, 4>;
 597:   def KDB : CompareRXE<"kdb", 0xED18, z_strict_fcmps, FP64, z_load, 8>;
 598: }
 599: 
 600: // Test Data Class.
 601: let Defs = [CC], CCValues = 0xC in {
 602:   def TCEB : TestRXE<"tceb", 0xED10, z_tdc, FP32>;
 603:   def TCDB : TestRXE<"tcdb", 0xED11, z_tdc, FP64>;
 604:   def TCXB : TestRXE<"tcxb", 0xED12, z_tdc, FP128>;
 605: }
 606: 
 607: //===----------------------------------------------------------------------===//
 608: // Floating-point control register instructions
 609: //===----------------------------------------------------------------------===//
 610: 
 611: let hasSideEffects = 1 in {
 612:   let mayLoad = 1, mayStore = 1 in {
 613:     // TODO: EFPC and SFPC do not touch memory at all
 614:     let Uses = [FPC] in {
 615:       def EFPC  : InherentRRE<"efpc", 0xB38C, GR32, int_s390_efpc>;
 616:       def STFPC : StoreInherentS<"stfpc", 0xB29C, storei<int_s390_efpc>, 4>;
```
- **EN**: This block declares or refines TableGen records such as `CEB`, `CDB`, `KEBR`, `KDBR`, `KXBR`, `KEB`.
- **CN**: 该代码块声明或细化了 `CEB`, `CDB`, `KEBR`, `KDBR`, `KXBR`, `KEB` 等 TableGen 记录。

### Lines 617-644 / 第 617-644 行
```tablegen
 617:     }
 618: 
 619:     let Defs = [FPC] in {
 620:       def SFPC : SideEffectUnaryRRE<"sfpc", 0xB384, GR32, int_s390_sfpc>;
 621:       def LFPC : SideEffectUnaryS<"lfpc", 0xB29D, loadu<int_s390_sfpc>, 4>;
 622:     }
 623:   }
 624: 
 625:   let Defs = [FPC], mayRaiseFPException = 1 in {
 626:     def SFASR : SideEffectUnaryRRE<"sfasr", 0xB385, GR32, null_frag>;
 627:     def LFAS  : SideEffectUnaryS<"lfas", 0xB2BD, null_frag, 4>;
 628:   }
 629: 
 630:   let Uses = [FPC], Defs = [FPC] in {
 631:     def SRNMB : SideEffectAddressS<"srnmb", 0xB2B8, null_frag, shift12only>,
 632:                 Requires<[FeatureFPExtension]>;
 633:     def SRNM  : SideEffectAddressS<"srnm", 0xB299, null_frag, shift12only>;
 634:     def SRNMT : SideEffectAddressS<"srnmt", 0xB2B9, null_frag, shift12only>;
 635:   }
 636: }
 637: 
 638: //===----------------------------------------------------------------------===//
 639: // Peepholes
 640: //===----------------------------------------------------------------------===//
 641: 
 642: def : Pat<(f16  fpimmneg0), (LCDFR_16 (LZER_16))>;
 643: def : Pat<(f32  fpimmneg0), (LCDFR_32 (LZER))>;
 644: def : Pat<(f64  fpimmneg0), (LCDFR (LZDR))>;
```
- **EN**: This block declares or refines TableGen records such as `SFPC`, `LFPC`, `SFASR`, `LFAS`, `SRNMB`, `SRNM`.
- **CN**: 该代码块声明或细化了 `SFPC`, `LFPC`, `SFASR`, `LFAS`, `SRNMB`, `SRNM` 等 TableGen 记录。

### Lines 645-645 / 第 645-645 行
```tablegen
 645: def : Pat<(f128 fpimmneg0), (LCXBR (LZXR))>;
```
- **EN**: This span continues the file's main responsibility: defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
