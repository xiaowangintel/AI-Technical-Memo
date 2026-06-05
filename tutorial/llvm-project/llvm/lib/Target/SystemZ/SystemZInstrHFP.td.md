# SystemZInstrHFP.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrHFP.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //==- SystemZInstrHFP.td - Floating-point SystemZ instructions -*- tblgen-*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // The instructions in this file implement SystemZ hexadecimal floating-point
  10: // arithmetic.  Since this format is not mapped to any source-language data
  11: // type, these instructions are not used for code generation, but are provided
  12: // for use with the assembler and disassembler only.
  13: //
  14: //===----------------------------------------------------------------------===//
  15: 
  16: //===----------------------------------------------------------------------===//
  17: // Move instructions
  18: //===----------------------------------------------------------------------===//
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: 
  20: // Load and test.
  21: let Defs = [CC] in {
  22:   def LTER : UnaryRR <"lter", 0x32,   null_frag, FP32,  FP32>;
  23:   def LTDR : UnaryRR <"ltdr", 0x22,   null_frag, FP64,  FP64>;
  24:   def LTXR : UnaryRRE<"ltxr", 0xB362, null_frag, FP128, FP128>;
  25: }
  26: 
  27: //===----------------------------------------------------------------------===//
  28: // Conversion instructions
  29: //===----------------------------------------------------------------------===//
  30: 
  31: // Convert floating-point values to narrower representations.
  32: def LEDR : UnaryRR <"ledr", 0x35,   null_frag, FP32, FP64>;
  33: def LEXR : UnaryRRE<"lexr", 0xB366, null_frag, FP32, FP128>;
  34: def LDXR : UnaryRR <"ldxr", 0x25,   null_frag, FP64, FP128>;
  35: let isAsmParserOnly = 1 in {
  36:   def LRER : UnaryRR <"lrer", 0x35, null_frag, FP32, FP64>;
```
- **EN**: This block declares or refines TableGen records such as `LTER`, `LTDR`, `LTXR`, `LEDR`, `LEXR`, `LDXR`.
- **CN**: 该代码块声明或细化了 `LTER`, `LTDR`, `LTXR`, `LEDR`, `LEXR`, `LDXR` 等 TableGen 记录。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:   def LRDR : UnaryRR <"lrdr", 0x25, null_frag, FP64, FP128>;
  38: }
  39: 
  40: // Extend floating-point values to wider representations.
  41: def LDER : UnaryRRE<"lder", 0xB324, null_frag, FP64,  FP32>;
  42: def LXER : UnaryRRE<"lxer", 0xB326, null_frag, FP128, FP32>;
  43: def LXDR : UnaryRRE<"lxdr", 0xB325, null_frag, FP128, FP64>;
  44: 
  45: def LDE : UnaryRXE<"lde", 0xED24, null_frag, FP64,  4>;
  46: def LXE : UnaryRXE<"lxe", 0xED26, null_frag, FP128, 4>;
  47: def LXD : UnaryRXE<"lxd", 0xED25, null_frag, FP128, 8>;
  48: 
  49: // Convert a signed integer register value to a floating-point one.
  50: def CEFR : UnaryRRE<"cefr", 0xB3B4, null_frag, FP32,  GR32>;
  51: def CDFR : UnaryRRE<"cdfr", 0xB3B5, null_frag, FP64,  GR32>;
  52: def CXFR : UnaryRRE<"cxfr", 0xB3B6, null_frag, FP128, GR32>;
  53: 
  54: def CEGR : UnaryRRE<"cegr", 0xB3C4, null_frag, FP32,  GR64>;
```
- **EN**: This block declares or refines TableGen records such as `LRDR`, `LDER`, `LXER`, `LXDR`, `LDE`, `LXE`.
- **CN**: 该代码块声明或细化了 `LRDR`, `LDER`, `LXER`, `LXDR`, `LDE`, `LXE` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: def CDGR : UnaryRRE<"cdgr", 0xB3C5, null_frag, FP64,  GR64>;
  56: def CXGR : UnaryRRE<"cxgr", 0xB3C6, null_frag, FP128, GR64>;
  57: 
  58: // Convert a floating-point register value to a signed integer value,
  59: // with the second operand (modifier M3) specifying the rounding mode.
  60: let Defs = [CC] in {
  61:   def CFER : BinaryRRFe<"cfer", 0xB3B8, GR32, FP32>;
  62:   def CFDR : BinaryRRFe<"cfdr", 0xB3B9, GR32, FP64>;
  63:   def CFXR : BinaryRRFe<"cfxr", 0xB3BA, GR32, FP128>;
  64: 
  65:   def CGER : BinaryRRFe<"cger", 0xB3C8, GR64, FP32>;
  66:   def CGDR : BinaryRRFe<"cgdr", 0xB3C9, GR64, FP64>;
  67:   def CGXR : BinaryRRFe<"cgxr", 0xB3CA, GR64, FP128>;
  68: }
  69: 
  70: // Convert BFP to HFP.
  71: let Defs = [CC] in {
  72:   def THDER : UnaryRRE<"thder", 0xB358, null_frag, FP64, FP32>;
```
- **EN**: This block declares or refines TableGen records such as `CDGR`, `CXGR`, `CFER`, `CFDR`, `CFXR`, `CGER`.
- **CN**: 该代码块声明或细化了 `CDGR`, `CXGR`, `CFER`, `CFDR`, `CFXR`, `CGER` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73:   def THDR  : UnaryRRE<"thdr",  0xB359, null_frag, FP64, FP64>;
  74: }
  75: 
  76: // Convert HFP to BFP.
  77: let Defs = [CC] in {
  78:   def TBEDR : BinaryRRFe<"tbedr", 0xB350, FP32, FP64>;
  79:   def TBDR  : BinaryRRFe<"tbdr",  0xB351, FP64, FP64>;
  80: }
  81: 
  82: 
  83: //===----------------------------------------------------------------------===//
  84: // Unary arithmetic
  85: //===----------------------------------------------------------------------===//
  86: 
  87: // Negation (Load Complement).
  88: let Defs = [CC] in {
  89:   def LCER : UnaryRR <"lcer", 0x33,   null_frag, FP32,  FP32>;
  90:   def LCDR : UnaryRR <"lcdr", 0x23,   null_frag, FP64,  FP64>;
```
- **EN**: This block declares or refines TableGen records such as `THDR`, `TBEDR`, `TBDR`, `LCER`, `LCDR`.
- **CN**: 该代码块声明或细化了 `THDR`, `TBEDR`, `TBDR`, `LCER`, `LCDR` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91:   def LCXR : UnaryRRE<"lcxr", 0xB363, null_frag, FP128, FP128>;
  92: }
  93: 
  94: // Absolute value (Load Positive).
  95: let Defs = [CC] in {
  96:   def LPER : UnaryRR <"lper", 0x30,   null_frag, FP32,  FP32>;
  97:   def LPDR : UnaryRR <"lpdr", 0x20,   null_frag, FP64,  FP64>;
  98:   def LPXR : UnaryRRE<"lpxr", 0xB360, null_frag, FP128, FP128>;
  99: }
 100: 
 101: // Negative absolute value (Load Negative).
 102: let Defs = [CC] in {
 103:   def LNER : UnaryRR <"lner", 0x31,   null_frag, FP32,  FP32>;
 104:   def LNDR : UnaryRR <"lndr", 0x21,   null_frag, FP64,  FP64>;
 105:   def LNXR : UnaryRRE<"lnxr", 0xB361, null_frag, FP128, FP128>;
 106: }
 107: 
 108: // Halve.
```
- **EN**: This block declares or refines TableGen records such as `LCXR`, `LPER`, `LPDR`, `LPXR`, `LNER`, `LNDR`.
- **CN**: 该代码块声明或细化了 `LCXR`, `LPER`, `LPDR`, `LPXR`, `LNER`, `LNDR` 等 TableGen 记录。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: def HER : UnaryRR <"her", 0x34, null_frag, FP32, FP32>;
 110: def HDR : UnaryRR <"hdr", 0x24, null_frag, FP64, FP64>;
 111: 
 112: // Square root.
 113: def SQER : UnaryRRE<"sqer", 0xB245, null_frag, FP32,  FP32>;
 114: def SQDR : UnaryRRE<"sqdr", 0xB244, null_frag, FP64,  FP64>;
 115: def SQXR : UnaryRRE<"sqxr", 0xB336, null_frag, FP128, FP128>;
 116: 
 117: def SQE : UnaryRXE<"sqe", 0xED34, null_frag, FP32, 4>;
 118: def SQD : UnaryRXE<"sqd", 0xED35, null_frag, FP64, 8>;
 119: 
 120: // Round to an integer (rounding towards zero).
 121: def FIER : UnaryRRE<"fier", 0xB377, null_frag, FP32,  FP32>;
 122: def FIDR : UnaryRRE<"fidr", 0xB37F, null_frag, FP64,  FP64>;
 123: def FIXR : UnaryRRE<"fixr", 0xB367, null_frag, FP128, FP128>;
 124: 
 125: 
 126: //===----------------------------------------------------------------------===//
```
- **EN**: This block declares or refines TableGen records such as `HER`, `HDR`, `SQER`, `SQDR`, `SQXR`, `SQE`.
- **CN**: 该代码块声明或细化了 `HER`, `HDR`, `SQER`, `SQDR`, `SQXR`, `SQE` 等 TableGen 记录。

### Lines 127-144 / 第 127-144 行
```tablegen
 127: // Binary arithmetic
 128: //===----------------------------------------------------------------------===//
 129: 
 130: // Addition.
 131: let Defs = [CC] in {
 132:   let isCommutable = 1 in {
 133:     def AER : BinaryRR<"aer", 0x3A, null_frag, FP32,  FP32>;
 134:     def ADR : BinaryRR<"adr", 0x2A, null_frag, FP64,  FP64>;
 135:     def AXR : BinaryRR<"axr", 0x36, null_frag, FP128, FP128>;
 136:   }
 137:   def AE : BinaryRX<"ae", 0x7A, null_frag, FP32, z_load, 4>;
 138:   def AD : BinaryRX<"ad", 0x6A, null_frag, FP64, z_load, 8>;
 139: }
 140: 
 141: // Addition (unnormalized).
 142: let Defs = [CC] in {
 143:   let isCommutable = 1 in {
 144:     def AUR : BinaryRR<"aur", 0x3E, null_frag, FP32, FP32>;
```
- **EN**: This block declares or refines TableGen records such as `AER`, `ADR`, `AXR`, `AE`, `AD`, `AUR`.
- **CN**: 该代码块声明或细化了 `AER`, `ADR`, `AXR`, `AE`, `AD`, `AUR` 等 TableGen 记录。

### Lines 145-162 / 第 145-162 行
```tablegen
 145:     def AWR : BinaryRR<"awr", 0x2E, null_frag, FP64, FP64>;
 146:   }
 147:   def AU : BinaryRX<"au", 0x7E, null_frag, FP32, z_load, 4>;
 148:   def AW : BinaryRX<"aw", 0x6E, null_frag, FP64, z_load, 8>;
 149: }
 150: 
 151: // Subtraction.
 152: let Defs = [CC] in {
 153:   def SER : BinaryRR<"ser", 0x3B, null_frag, FP32,  FP32>;
 154:   def SDR : BinaryRR<"sdr", 0x2B, null_frag, FP64,  FP64>;
 155:   def SXR : BinaryRR<"sxr", 0x37, null_frag, FP128, FP128>;
 156: 
 157:   def SE : BinaryRX<"se", 0x7B, null_frag, FP32, z_load, 4>;
 158:   def SD : BinaryRX<"sd", 0x6B, null_frag, FP64, z_load, 8>;
 159: }
 160: 
 161: // Subtraction (unnormalized).
 162: let Defs = [CC] in {
```
- **EN**: This block declares or refines TableGen records such as `AWR`, `AU`, `AW`, `SER`, `SDR`, `SXR`.
- **CN**: 该代码块声明或细化了 `AWR`, `AU`, `AW`, `SER`, `SDR`, `SXR` 等 TableGen 记录。

### Lines 163-180 / 第 163-180 行
```tablegen
 163:   def SUR : BinaryRR<"sur", 0x3F, null_frag, FP32, FP32>;
 164:   def SWR : BinaryRR<"swr", 0x2F, null_frag, FP64, FP64>;
 165: 
 166:   def SU : BinaryRX<"su", 0x7F, null_frag, FP32, z_load, 4>;
 167:   def SW : BinaryRX<"sw", 0x6F, null_frag, FP64, z_load, 8>;
 168: }
 169: 
 170: // Multiplication.
 171: let isCommutable = 1 in {
 172:   def MEER : BinaryRRE<"meer", 0xB337, null_frag, FP32,  FP32>;
 173:   def MDR  : BinaryRR <"mdr",  0x2C,   null_frag, FP64,  FP64>;
 174:   def MXR  : BinaryRR <"mxr",  0x26,   null_frag, FP128, FP128>;
 175: }
 176: def MEE : BinaryRXE<"mee", 0xED37, null_frag, FP32, z_load, 4>;
 177: def MD  : BinaryRX <"md",  0x6C,   null_frag, FP64, z_load, 8>;
 178: 
 179: // Extending multiplication (f32 x f32 -> f64).
 180: def MDER : BinaryRR<"mder", 0x3C, null_frag, FP64, FP32>;
```
- **EN**: This block declares or refines TableGen records such as `SUR`, `SWR`, `SU`, `SW`, `MEER`, `MDR`.
- **CN**: 该代码块声明或细化了 `SUR`, `SWR`, `SU`, `SW`, `MEER`, `MDR` 等 TableGen 记录。

### Lines 181-198 / 第 181-198 行
```tablegen
 181: def MDE  : BinaryRX<"mde",  0x7C, null_frag, FP64, z_load, 4>;
 182: let isAsmParserOnly = 1 in {
 183:   def MER : BinaryRR<"mer", 0x3C, null_frag, FP64, FP32>;
 184:   def ME  : BinaryRX<"me",  0x7C, null_frag, FP64, z_load, 4>;
 185: }
 186: 
 187: // Extending multiplication (f64 x f64 -> f128).
 188: def MXDR : BinaryRR<"mxdr", 0x27, null_frag, FP128, FP64>;
 189: def MXD  : BinaryRX<"mxd",  0x67, null_frag, FP128, z_load, 8>;
 190: 
 191: // Fused multiply-add.
 192: def MAER : TernaryRRD<"maer", 0xB32E, null_frag, FP32, FP32>;
 193: def MADR : TernaryRRD<"madr", 0xB33E, null_frag, FP64, FP64>;
 194: def MAE  : TernaryRXF<"mae",  0xED2E, null_frag, FP32, FP32, z_load, 4>;
 195: def MAD  : TernaryRXF<"mad",  0xED3E, null_frag, FP64, FP64, z_load, 8>;
 196: 
 197: // Fused multiply-subtract.
 198: def MSER : TernaryRRD<"mser", 0xB32F, null_frag, FP32, FP32>;
```
- **EN**: This block declares or refines TableGen records such as `MDE`, `MER`, `ME`, `MXDR`, `MXD`, `MAER`.
- **CN**: 该代码块声明或细化了 `MDE`, `MER`, `ME`, `MXDR`, `MXD`, `MAER` 等 TableGen 记录。

### Lines 199-216 / 第 199-216 行
```tablegen
 199: def MSDR : TernaryRRD<"msdr", 0xB33F, null_frag, FP64, FP64>;
 200: def MSE  : TernaryRXF<"mse",  0xED2F, null_frag, FP32, FP32, z_load, 4>;
 201: def MSD  : TernaryRXF<"msd",  0xED3F, null_frag, FP64, FP64, z_load, 8>;
 202: 
 203: // Multiplication (unnormalized).
 204: def MYR  : BinaryRRD<"myr",  0xB33B, null_frag, FP128, FP64>;
 205: def MYHR : BinaryRRD<"myhr", 0xB33D, null_frag, FP64,  FP64>;
 206: def MYLR : BinaryRRD<"mylr", 0xB339, null_frag, FP64,  FP64>;
 207: def MY   : BinaryRXF<"my",   0xED3B, null_frag, FP128, FP64, z_load, 8>;
 208: def MYH  : BinaryRXF<"myh",  0xED3D, null_frag, FP64,  FP64, z_load, 8>;
 209: def MYL  : BinaryRXF<"myl",  0xED39, null_frag, FP64,  FP64, z_load, 8>;
 210: 
 211: // Fused multiply-add (unnormalized).
 212: def MAYHR : TernaryRRD<"mayhr", 0xB33C, null_frag, FP64,  FP64>;
 213: def MAYLR : TernaryRRD<"maylr", 0xB338, null_frag, FP64,  FP64>;
 214: def MAYH  : TernaryRXF<"mayh",  0xED3C, null_frag, FP64,  FP64, z_load, 8>;
 215: def MAYL  : TernaryRXF<"mayl",  0xED38, null_frag, FP64,  FP64, z_load, 8>;
 216: 
```
- **EN**: This block declares or refines TableGen records such as `MSDR`, `MSE`, `MSD`, `MYR`, `MYHR`, `MYLR`.
- **CN**: 该代码块声明或细化了 `MSDR`, `MSE`, `MSD`, `MYR`, `MYHR`, `MYLR` 等 TableGen 记录。

### Lines 217-234 / 第 217-234 行
```tablegen
 217: // MAY and MAYR allow the user to specify the floating point register pair
 218: // making up the FP128 register by either the lower-numbered register or the
 219: // higher-numbered register, in contrast to all other floating point
 220: // instructions.
 221: // For this reason, the defs below accept `FP64,FP64` instead of `FP128,FP64`.
 222: // This is ok since these instructions are not used in code generation.
 223: // If and when code generation is enabled, the code gen variants should be
 224: // split out from this and use the proper register classes, while these should
 225: // remain for the Assembler and Disassembler to remain compliant with the POP.
 226: def MAY   : TernaryRXF<"may",   0xED3A, null_frag, FP64, FP64, z_load, 8>;
 227: def MAYR  : TernaryRRD<"mayr",  0xB33A, null_frag, FP64, FP64>;
 228: 
 229: // Division.
 230: def DER : BinaryRR <"der", 0x3D,   null_frag, FP32,  FP32>;
 231: def DDR : BinaryRR <"ddr", 0x2D,   null_frag, FP64,  FP64>;
 232: def DXR : BinaryRRE<"dxr", 0xB22D, null_frag, FP128, FP128>;
 233: def DE  : BinaryRX <"de",  0x7D,   null_frag, FP32, z_load, 4>;
 234: def DD  : BinaryRX <"dd",  0x6D,   null_frag, FP64, z_load, 8>;
```
- **EN**: This block declares or refines TableGen records such as `MAY`, `MAYR`, `DER`, `DDR`, `DXR`, `DE`.
- **CN**: 该代码块声明或细化了 `MAY`, `MAYR`, `DER`, `DDR`, `DXR`, `DE` 等 TableGen 记录。

### Lines 235-249 / 第 235-249 行
```tablegen
 235: 
 236: 
 237: //===----------------------------------------------------------------------===//
 238: // Comparisons
 239: //===----------------------------------------------------------------------===//
 240: 
 241: let Defs = [CC] in {
 242:   def CER : CompareRR <"cer", 0x39,   null_frag, FP32,  FP32>;
 243:   def CDR : CompareRR <"cdr", 0x29,   null_frag, FP64,  FP64>;
 244:   def CXR : CompareRRE<"cxr", 0xB369, null_frag, FP128, FP128>;
 245: 
 246:   def CE : CompareRX<"ce", 0x79, null_frag, FP32, z_load, 4>;
 247:   def CD : CompareRX<"cd", 0x69, null_frag, FP64, z_load, 8>;
 248: }
 249: 
```
- **EN**: This block declares or refines TableGen records such as `CER`, `CDR`, `CXR`, `CE`, `CD`.
- **CN**: 该代码块声明或细化了 `CER`, `CDR`, `CXR`, `CE`, `CD` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Disassembly**: Decodes binary encodings back into symbolic instructions. / 将二进制编码解码回符号化指令。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
