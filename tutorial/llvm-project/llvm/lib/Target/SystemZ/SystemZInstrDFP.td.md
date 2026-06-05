# SystemZInstrDFP.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrDFP.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //==- SystemZInstrDFP.td - Floating-point SystemZ instructions -*- tblgen-*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // The instructions in this file implement SystemZ decimal floating-point
  10: // arithmetic.  These instructions are inot currently used for code generation,
  11: // are provided for use with the assembler and disassembler only.  If LLVM
  12: // ever supports decimal floating-point types (_Decimal64 etc.), they can
  13: // also be used for code generation for those types.
  14: //
  15: //===----------------------------------------------------------------------===//
  16: 
  17: //===----------------------------------------------------------------------===//
  18: // Move instructions
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: //===----------------------------------------------------------------------===//
  20: 
  21: // Load and test.
  22: let Uses = [FPC], Defs = [CC] in {
  23:   def LTDTR : UnaryRRE<"ltdtr", 0xB3D6, null_frag, FP64,  FP64>;
  24:   def LTXTR : UnaryRRE<"ltxtr", 0xB3DE, null_frag, FP128, FP128>;
  25: }
  26: 
  27: 
  28: //===----------------------------------------------------------------------===//
  29: // Conversion instructions
  30: //===----------------------------------------------------------------------===//
  31: 
  32: // Convert floating-point values to narrower representations.  The destination
  33: // of LDXTR is a 128-bit value, but only the first register of the pair is used.
  34: let Uses = [FPC] in {
  35:   def LEDTR : TernaryRRFe<"ledtr", 0xB3D5, FP32,  FP64>;
  36:   def LDXTR : TernaryRRFe<"ldxtr", 0xB3DD, FP128, FP128>;
```
- **EN**: This block declares or refines TableGen records such as `LTDTR`, `LTXTR`, `LEDTR`, `LDXTR`.
- **CN**: 该代码块声明或细化了 `LTDTR`, `LTXTR`, `LEDTR`, `LDXTR` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37: }
  38: 
  39: // Extend floating-point values to wider representations.
  40: let Uses = [FPC] in {
  41:   def LDETR : BinaryRRFd<"ldetr", 0xB3D4, FP64,  FP32>;
  42:   def LXDTR : BinaryRRFd<"lxdtr", 0xB3DC, FP128, FP64>;
  43: }
  44: 
  45: // Convert a signed integer value to a floating-point one.
  46: let Uses = [FPC] in {
  47:   def CDGTR : UnaryRRE<"cdgtr", 0xB3F1, null_frag, FP64,  GR64>;
  48:   def CXGTR : UnaryRRE<"cxgtr", 0xB3F9, null_frag, FP128, GR64>;
  49:   let Predicates = [FeatureFPExtension] in {
  50:     def CDGTRA : TernaryRRFe<"cdgtra", 0xB3F1, FP64,  GR64>;
  51:     def CXGTRA : TernaryRRFe<"cxgtra", 0xB3F9, FP128, GR64>;
  52:     def CDFTR : TernaryRRFe<"cdftr", 0xB951, FP64,  GR32>;
  53:     def CXFTR : TernaryRRFe<"cxftr", 0xB959, FP128, GR32>;
  54:   }
```
- **EN**: This block declares or refines TableGen records such as `LDETR`, `LXDTR`, `CDGTR`, `CXGTR`, `CDGTRA`, `CXGTRA`.
- **CN**: 该代码块声明或细化了 `LDETR`, `LXDTR`, `CDGTR`, `CXGTR`, `CDGTRA`, `CXGTRA` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: }
  56: 
  57: // Convert an unsigned integer value to a floating-point one.
  58: let Uses = [FPC], Predicates = [FeatureFPExtension] in {
  59:   def CDLGTR : TernaryRRFe<"cdlgtr", 0xB952, FP64,  GR64>;
  60:   def CXLGTR : TernaryRRFe<"cxlgtr", 0xB95A, FP128, GR64>;
  61:   def CDLFTR : TernaryRRFe<"cdlftr", 0xB953, FP64,  GR32>;
  62:   def CXLFTR : TernaryRRFe<"cxlftr", 0xB95B, FP128, GR32>;
  63: }
  64: 
  65: // Convert a floating-point value to a signed integer value.
  66: let Uses = [FPC], Defs = [CC] in {
  67:   def CGDTR : BinaryRRFe<"cgdtr", 0xB3E1, GR64, FP64>;
  68:   def CGXTR : BinaryRRFe<"cgxtr", 0xB3E9, GR64, FP128>;
  69:   let Predicates = [FeatureFPExtension] in {
  70:     def CGDTRA : TernaryRRFe<"cgdtra", 0xB3E1, GR64, FP64>;
  71:     def CGXTRA : TernaryRRFe<"cgxtra", 0xB3E9, GR64, FP128>;
  72:     def CFDTR : TernaryRRFe<"cfdtr", 0xB941, GR32, FP64>;
```
- **EN**: This block declares or refines TableGen records such as `CDLGTR`, `CXLGTR`, `CDLFTR`, `CXLFTR`, `CGDTR`, `CGXTR`.
- **CN**: 该代码块声明或细化了 `CDLGTR`, `CXLGTR`, `CDLFTR`, `CXLFTR`, `CGDTR`, `CGXTR` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:     def CFXTR : TernaryRRFe<"cfxtr", 0xB949, GR32, FP128>;
  74:   }
  75: }
  76: 
  77: // Convert a floating-point value to an unsigned integer value.
  78: let Uses = [FPC], Defs = [CC] in {
  79:   let Predicates = [FeatureFPExtension] in {
  80:     def CLGDTR : TernaryRRFe<"clgdtr", 0xB942, GR64, FP64>;
  81:     def CLGXTR : TernaryRRFe<"clgxtr", 0xB94A, GR64, FP128>;
  82:     def CLFDTR : TernaryRRFe<"clfdtr", 0xB943, GR32, FP64>;
  83:     def CLFXTR : TernaryRRFe<"clfxtr", 0xB94B, GR32, FP128>;
  84:   }
  85: }
  86: 
  87: // Convert a packed value to a floating-point one.
  88: def CDSTR : UnaryRRE<"cdstr", 0xB3F3, null_frag, FP64,  GR64>;
  89: def CXSTR : UnaryRRE<"cxstr", 0xB3FB, null_frag, FP128, GR128>;
  90: def CDUTR : UnaryRRE<"cdutr", 0xB3F2, null_frag, FP64,  GR64>;
```
- **EN**: This block declares or refines TableGen records such as `CFXTR`, `CLGDTR`, `CLGXTR`, `CLFDTR`, `CLFXTR`, `CDSTR`.
- **CN**: 该代码块声明或细化了 `CFXTR`, `CLGDTR`, `CLGXTR`, `CLFDTR`, `CLFXTR`, `CDSTR` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: def CXUTR : UnaryRRE<"cxutr", 0xB3FA, null_frag, FP128, GR128>;
  92: 
  93: // Convert a floating-point value to a packed value.
  94: def CSDTR : BinaryRRFd<"csdtr", 0xB3E3, GR64,  FP64>;
  95: def CSXTR : BinaryRRFd<"csxtr", 0xB3EB, GR128, FP128>;
  96: def CUDTR : UnaryRRE<"cudtr", 0xB3E2, null_frag, GR64,  FP64>;
  97: def CUXTR : UnaryRRE<"cuxtr", 0xB3EA, null_frag, GR128, FP128>;
  98: 
  99: // Convert from/to memory values in the zoned format.
 100: let Predicates = [FeatureDFPZonedConversion] in {
 101:   def CDZT : BinaryRSL<"cdzt", 0xEDAA, FP64>;
 102:   def CXZT : BinaryRSL<"cxzt", 0xEDAB, FP128>;
 103:   def CZDT : StoreBinaryRSL<"czdt", 0xEDA8, FP64>;
 104:   def CZXT : StoreBinaryRSL<"czxt", 0xEDA9, FP128>;
 105: }
 106: 
 107: // Convert from/to memory values in the packed format.
 108: let Predicates = [FeatureDFPPackedConversion] in {
```
- **EN**: This block declares or refines TableGen records such as `CXUTR`, `CSDTR`, `CSXTR`, `CUDTR`, `CUXTR`, `CDZT`.
- **CN**: 该代码块声明或细化了 `CXUTR`, `CSDTR`, `CSXTR`, `CUDTR`, `CUXTR`, `CDZT` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109:   def CDPT : BinaryRSL<"cdpt", 0xEDAE, FP64>;
 110:   def CXPT : BinaryRSL<"cxpt", 0xEDAF, FP128>;
 111:   def CPDT : StoreBinaryRSL<"cpdt", 0xEDAC, FP64>;
 112:   def CPXT : StoreBinaryRSL<"cpxt", 0xEDAD, FP128>;
 113: }
 114: 
 115: // Perform floating-point operation.
 116: let Defs = [CC, R1L, F0Q], Uses = [FPC, R0L, F4Q] in
 117:   def PFPO : SideEffectInherentE<"pfpo", 0x010A>;
 118: 
 119: 
 120: //===----------------------------------------------------------------------===//
 121: // Unary arithmetic
 122: //===----------------------------------------------------------------------===//
 123: 
 124: // Round to an integer, with the second operand (M3) specifying the rounding
 125: // mode.  M4 can be set to 4 to suppress detection of inexact conditions.
 126: let Uses = [FPC] in {
```
- **EN**: This block declares or refines TableGen records such as `CDPT`, `CXPT`, `CPDT`, `CPXT`, `PFPO`.
- **CN**: 该代码块声明或细化了 `CDPT`, `CXPT`, `CPDT`, `CPXT`, `PFPO` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127:   def FIDTR : TernaryRRFe<"fidtr", 0xB3D7, FP64,  FP64>;
 128:   def FIXTR : TernaryRRFe<"fixtr", 0xB3DF, FP128, FP128>;
 129: }
 130: 
 131: // Extract biased exponent.
 132: def EEDTR : UnaryRRE<"eedtr", 0xB3E5, null_frag, GR64, FP64>;
 133: def EEXTR : UnaryRRE<"eextr", 0xB3ED, null_frag, GR64, FP128>;
 134: 
 135: // Extract significance.
 136: def ESDTR : UnaryRRE<"esdtr", 0xB3E7, null_frag, GR64, FP64>;
 137: def ESXTR : UnaryRRE<"esxtr", 0xB3EF, null_frag, GR64, FP128>;
 138: 
 139: 
 140: //===----------------------------------------------------------------------===//
 141: // Binary arithmetic
 142: //===----------------------------------------------------------------------===//
 143: 
 144: // Addition.
```
- **EN**: This block declares or refines TableGen records such as `FIDTR`, `FIXTR`, `EEDTR`, `EEXTR`, `ESDTR`, `ESXTR`.
- **CN**: 该代码块声明或细化了 `FIDTR`, `FIXTR`, `EEDTR`, `EEXTR`, `ESDTR`, `ESXTR` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145: let Uses = [FPC], Defs = [CC] in {
 146:   let isCommutable = 1 in {
 147:     def ADTR : BinaryRRFa<"adtr", 0xB3D2, null_frag, FP64,  FP64,  FP64>;
 148:     def AXTR : BinaryRRFa<"axtr", 0xB3DA, null_frag, FP128, FP128, FP128>;
 149:   }
 150:   let Predicates = [FeatureFPExtension] in {
 151:     def ADTRA : TernaryRRFa<"adtra", 0xB3D2, FP64,  FP64,  FP64>;
 152:     def AXTRA : TernaryRRFa<"axtra", 0xB3DA, FP128, FP128, FP128>;
 153:   }
 154: }
 155: 
 156: // Subtraction.
 157: let Uses = [FPC], Defs = [CC] in {
 158:   def SDTR : BinaryRRFa<"sdtr", 0xB3D3, null_frag, FP64,  FP64,  FP64>;
 159:   def SXTR : BinaryRRFa<"sxtr", 0xB3DB, null_frag, FP128, FP128, FP128>;
 160:   let Predicates = [FeatureFPExtension] in {
 161:     def SDTRA : TernaryRRFa<"sdtra", 0xB3D3, FP64,  FP64,  FP64>;
 162:     def SXTRA : TernaryRRFa<"sxtra", 0xB3DB, FP128, FP128, FP128>;
```
- **EN**: This block declares or refines TableGen records such as `ADTR`, `AXTR`, `ADTRA`, `AXTRA`, `SDTR`, `SXTR`.
- **CN**: 该代码块声明或细化了 `ADTR`, `AXTR`, `ADTRA`, `AXTRA`, `SDTR`, `SXTR` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:   }
 164: }
 165: 
 166: // Multiplication.
 167: let Uses = [FPC] in {
 168:   let isCommutable = 1 in {
 169:     def MDTR : BinaryRRFa<"mdtr", 0xB3D0, null_frag, FP64,  FP64,  FP64>;
 170:     def MXTR : BinaryRRFa<"mxtr", 0xB3D8, null_frag, FP128, FP128, FP128>;
 171:   }
 172:   let Predicates = [FeatureFPExtension] in {
 173:     def MDTRA : TernaryRRFa<"mdtra", 0xB3D0, FP64,  FP64,  FP64>;
 174:     def MXTRA : TernaryRRFa<"mxtra", 0xB3D8, FP128, FP128, FP128>;
 175:   }
 176: }
 177: 
 178: // Division.
 179: let Uses = [FPC] in {
 180:   def DDTR : BinaryRRFa<"ddtr", 0xB3D1, null_frag, FP64,  FP64,  FP64>;
```
- **EN**: This block declares or refines TableGen records such as `MDTR`, `MXTR`, `MDTRA`, `MXTRA`, `DDTR`.
- **CN**: 该代码块声明或细化了 `MDTR`, `MXTR`, `MDTRA`, `MXTRA`, `DDTR` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181:   def DXTR : BinaryRRFa<"dxtr", 0xB3D9, null_frag, FP128, FP128, FP128>;
 182:   let Predicates = [FeatureFPExtension] in {
 183:     def DDTRA : TernaryRRFa<"ddtra", 0xB3D1, FP64,  FP64,  FP64>;
 184:     def DXTRA : TernaryRRFa<"dxtra", 0xB3D9, FP128, FP128, FP128>;
 185:   }
 186: }
 187: 
 188: // Quantize.
 189: let Uses = [FPC] in {
 190:   def QADTR : TernaryRRFb<"qadtr", 0xB3F5, FP64,  FP64,  FP64>;
 191:   def QAXTR : TernaryRRFb<"qaxtr", 0xB3FD, FP128, FP128, FP128>;
 192: }
 193: 
 194: // Reround.
 195: let Uses = [FPC] in {
 196:   def RRDTR : TernaryRRFb<"rrdtr", 0xB3F7, FP64,  GR32, FP64>;
 197:   def RRXTR : TernaryRRFb<"rrxtr", 0xB3FF, FP128, GR32, FP128>;
 198: }
```
- **EN**: This block declares or refines TableGen records such as `DXTR`, `DDTRA`, `DXTRA`, `QADTR`, `QAXTR`, `RRDTR`.
- **CN**: 该代码块声明或细化了 `DXTR`, `DDTRA`, `DXTRA`, `QADTR`, `QAXTR`, `RRDTR` 等 TableGen 记录。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: 
 200: // Shift significand left/right.
 201: def SLDT : BinaryRXF<"sldt", 0xED40, null_frag, FP64,  FP64,  null_frag, 0>;
 202: def SLXT : BinaryRXF<"slxt", 0xED48, null_frag, FP128, FP128, null_frag, 0>;
 203: def SRDT : BinaryRXF<"srdt", 0xED41, null_frag, FP64,  FP64,  null_frag, 0>;
 204: def SRXT : BinaryRXF<"srxt", 0xED49, null_frag, FP128, FP128, null_frag, 0>;
 205: 
 206: // Insert biased exponent.
 207: def IEDTR : BinaryRRFb<"iedtr", 0xB3F6, null_frag, FP64,  GR64, FP64>;
 208: def IEXTR : BinaryRRFb<"iextr", 0xB3FE, null_frag, FP128, GR64, FP128>;
 209: 
 210: 
 211: //===----------------------------------------------------------------------===//
 212: // Comparisons
 213: //===----------------------------------------------------------------------===//
 214: 
 215: // Compare.
 216: let Uses = [FPC], Defs = [CC] in {
```
- **EN**: This block declares or refines TableGen records such as `SLDT`, `SLXT`, `SRDT`, `SRXT`, `IEDTR`, `IEXTR`.
- **CN**: 该代码块声明或细化了 `SLDT`, `SLXT`, `SRDT`, `SRXT`, `IEDTR`, `IEXTR` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217:   def CDTR : CompareRRE<"cdtr", 0xB3E4, null_frag, FP64,  FP64>;
 218:   def CXTR : CompareRRE<"cxtr", 0xB3EC, null_frag, FP128, FP128>;
 219: }
 220: 
 221: // Compare and signal.
 222: let Uses = [FPC], Defs = [CC] in {
 223:   def KDTR : CompareRRE<"kdtr", 0xB3E0, null_frag, FP64,  FP64>;
 224:   def KXTR : CompareRRE<"kxtr", 0xB3E8, null_frag, FP128, FP128>;
 225: }
 226: 
 227: // Compare biased exponent.
 228: let Defs = [CC] in {
 229:   def CEDTR : CompareRRE<"cedtr", 0xB3F4, null_frag, FP64,  FP64>;
 230:   def CEXTR : CompareRRE<"cextr", 0xB3FC, null_frag, FP128, FP128>;
 231: }
 232: 
 233: // Test Data Class.
 234: let Defs = [CC] in {
```
- **EN**: This block declares or refines TableGen records such as `CDTR`, `CXTR`, `KDTR`, `KXTR`, `CEDTR`, `CEXTR`.
- **CN**: 该代码块声明或细化了 `CDTR`, `CXTR`, `KDTR`, `KXTR`, `CEDTR`, `CEXTR` 等 TableGen 记录。

### Lines 235-246 / 第 235-246 行
```tablegen
 235:   def TDCET : TestRXE<"tdcet", 0xED50, null_frag, FP32>;
 236:   def TDCDT : TestRXE<"tdcdt", 0xED54, null_frag, FP64>;
 237:   def TDCXT : TestRXE<"tdcxt", 0xED58, null_frag, FP128>;
 238: }
 239: 
 240: // Test Data Group.
 241: let Defs = [CC] in {
 242:   def TDGET : TestRXE<"tdget", 0xED51, null_frag, FP32>;
 243:   def TDGDT : TestRXE<"tdgdt", 0xED55, null_frag, FP64>;
 244:   def TDGXT : TestRXE<"tdgxt", 0xED59, null_frag, FP128>;
 245: }
 246: 
```
- **EN**: This block declares or refines TableGen records such as `TDCET`, `TDCDT`, `TDCXT`, `TDGET`, `TDGDT`, `TDGXT`.
- **CN**: 该代码块声明或细化了 `TDCET`, `TDCDT`, `TDCXT`, `TDGET`, `TDGDT`, `TDGXT` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
