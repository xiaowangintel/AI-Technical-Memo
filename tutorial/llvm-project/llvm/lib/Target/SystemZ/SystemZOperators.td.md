# SystemZOperators.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZOperators.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```tablegen
   1: //===-- SystemZOperators.td - SystemZ-specific operators ------*- tblgen-*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: //===----------------------------------------------------------------------===//
  10: // Type profiles
  11: //===----------------------------------------------------------------------===//
  12: def SDT_CallSeqStart        : SDCallSeqStart<[SDTCisVT<0, i64>,
  13:                                               SDTCisVT<1, i64>]>;
  14: def SDT_CallSeqEnd          : SDCallSeqEnd<[SDTCisVT<0, i64>,
  15:                                             SDTCisVT<1, i64>]>;
  16: def SDT_ZCall               : SDTypeProfile<0, -1, [SDTCisPtrTy<0>]>;
  17: def SDT_ZCmp                : SDTypeProfile<1, 2,
  18:                                             [SDTCisVT<0, i32>,
  19:                                              SDTCisSameAs<1, 2>]>;
  20: def SDT_ZICmp               : SDTypeProfile<1, 3,
  21:                                             [SDTCisVT<0, i32>,
  22:                                              SDTCisSameAs<1, 2>,
  23:                                              SDTCisVT<3, i32>]>;
  24: def SDT_ZBRCCMask           : SDTypeProfile<0, 4,
  25:                                             [SDTCisVT<0, i32>,
  26:                                              SDTCisVT<1, i32>,
  27:                                              SDTCisVT<2, OtherVT>,
  28:                                              SDTCisVT<3, i32>]>;
  29: def SDT_ZSelectCCMask       : SDTypeProfile<1, 5,
  30:                                             [SDTCisSameAs<0, 1>,
  31:                                              SDTCisSameAs<1, 2>,
  32:                                              SDTCisVT<3, i32>,
  33:                                              SDTCisVT<4, i32>,
  34:                                              SDTCisVT<5, i32>]>;
  35: def SDT_ZWrapPtr            : SDTypeProfile<1, 1,
  36:                                             [SDTCisSameAs<0, 1>,
  37:                                              SDTCisPtrTy<0>]>;
  38: def SDT_ZWrapOffset         : SDTypeProfile<1, 2,
  39:                                             [SDTCisSameAs<0, 1>,
  40:                                              SDTCisSameAs<0, 2>,
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. This block declares or refines TableGen records such as `SDT_CallSeqStart`, `SDT_CallSeqEnd`, `SDT_ZCall`, `SDT_ZCmp`, `SDT_ZICmp`, `SDT_ZBRCCMask`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 该代码块声明或细化了 `SDT_CallSeqStart`, `SDT_CallSeqEnd`, `SDT_ZCall`, `SDT_ZCmp`, `SDT_ZICmp`, `SDT_ZBRCCMask` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```tablegen
  41:                                              SDTCisPtrTy<0>]>;
  42: def SDT_ZAdjDynAlloc        : SDTypeProfile<1, 0, [SDTCisVT<0, i64>]>;
  43: def SDT_ZProbedAlloca       : SDTypeProfile<1, 2,
  44:                                             [SDTCisSameAs<0, 1>,
  45:                                              SDTCisSameAs<0, 2>,
  46:                                              SDTCisPtrTy<0>]>;
  47: def SDT_ZGR128Binary        : SDTypeProfile<1, 2,
  48:                                             [SDTCisVT<0, untyped>,
  49:                                              SDTCisInt<1>,
  50:                                              SDTCisInt<2>]>;
  51: def SDT_ZBinaryWithFlags    : SDTypeProfile<2, 2,
  52:                                             [SDTCisInt<0>,
  53:                                              SDTCisVT<1, i32>,
  54:                                              SDTCisSameAs<0, 2>,
  55:                                              SDTCisSameAs<0, 3>]>;
  56: def SDT_ZBinaryWithCarry    : SDTypeProfile<2, 3,
  57:                                             [SDTCisInt<0>,
  58:                                              SDTCisVT<1, i32>,
  59:                                              SDTCisSameAs<0, 2>,
  60:                                              SDTCisSameAs<0, 3>,
  61:                                              SDTCisVT<1, i32>]>;
  62: def SDT_ZBinaryConv         : SDTypeProfile<1, 2,
  63:                                             [SDTCisInt<0>,
  64:                                              SDTCisInt<1>,
  65:                                              SDTCisSameAs<1, 2>]>;
  66: def SDT_ZTernary            : SDTypeProfile<1, 3,
  67:                                             [SDTCisInt<0>,
  68:                                              SDTCisSameAs<0, 1>,
  69:                                              SDTCisSameAs<0, 2>,
  70:                                              SDTCisSameAs<0, 3>]>;
  71: def SDT_ZAtomicLoadBinaryW  : SDTypeProfile<1, 5,
  72:                                             [SDTCisVT<0, i32>,
  73:                                              SDTCisPtrTy<1>,
  74:                                              SDTCisVT<2, i32>,
  75:                                              SDTCisVT<3, i32>,
  76:                                              SDTCisVT<4, i32>,
  77:                                              SDTCisVT<5, i32>]>;
  78: def SDT_ZAtomicCmpSwapW     : SDTypeProfile<2, 6,
  79:                                             [SDTCisVT<0, i32>,
  80:                                              SDTCisVT<1, i32>,
```
- **EN**: This block declares or refines TableGen records such as `SDT_ZAdjDynAlloc`, `SDT_ZProbedAlloca`, `SDT_ZGR128Binary`, `SDT_ZBinaryWithFlags`, `SDT_ZBinaryWithCarry`, `SDT_ZBinaryConv`.
- **CN**: 该代码块声明或细化了 `SDT_ZAdjDynAlloc`, `SDT_ZProbedAlloca`, `SDT_ZGR128Binary`, `SDT_ZBinaryWithFlags`, `SDT_ZBinaryWithCarry`, `SDT_ZBinaryConv` 等 TableGen 记录。

### Lines 81-120 / 第 81-120 行
```tablegen
  81:                                              SDTCisPtrTy<2>,
  82:                                              SDTCisVT<3, i32>,
  83:                                              SDTCisVT<4, i32>,
  84:                                              SDTCisVT<5, i32>,
  85:                                              SDTCisVT<6, i32>,
  86:                                              SDTCisVT<7, i32>]>;
  87: def SDT_ZAtomicCmpSwap      : SDTypeProfile<2, 3,
  88:                                             [SDTCisInt<0>,
  89:                                              SDTCisVT<1, i32>,
  90:                                              SDTCisPtrTy<2>,
  91:                                              SDTCisSameAs<0, 3>,
  92:                                              SDTCisSameAs<0, 4>]>;
  93: def SDT_ZAtomicLoad128      : SDTypeProfile<1, 1,
  94:                                             [SDTCisVT<0, untyped>,
  95:                                              SDTCisPtrTy<1>]>;
  96: def SDT_ZAtomicStore128     : SDTypeProfile<0, 2,
  97:                                             [SDTCisVT<0, untyped>,
  98:                                              SDTCisPtrTy<1>]>;
  99: def SDT_ZAtomicCmpSwap128   : SDTypeProfile<2, 3,
 100:                                             [SDTCisVT<0, untyped>,
 101:                                              SDTCisVT<1, i32>,
 102:                                              SDTCisPtrTy<2>,
 103:                                              SDTCisVT<3, untyped>,
 104:                                              SDTCisVT<4, untyped>]>;
 105: def SDT_ZMemMemLength       : SDTypeProfile<0, 3,
 106:                                             [SDTCisPtrTy<0>,
 107:                                              SDTCisPtrTy<1>,
 108:                                              SDTCisVT<2, i64>]>;
 109: def SDT_ZMemMemLengthCC     : SDTypeProfile<1, 3,
 110:                                             [SDTCisVT<0, i32>,
 111:                                              SDTCisPtrTy<1>,
 112:                                              SDTCisPtrTy<2>,
 113:                                              SDTCisVT<3, i64>]>;
 114: def SDT_ZMemsetMVC          : SDTypeProfile<0, 3,
 115:                                             [SDTCisPtrTy<0>,
 116:                                              SDTCisVT<1, i64>,
 117:                                              SDTCisVT<2, i32>]>;
 118: def SDT_ZString             : SDTypeProfile<1, 3,
 119:                                             [SDTCisPtrTy<0>,
 120:                                              SDTCisPtrTy<1>,
```
- **EN**: This block declares or refines TableGen records such as `SDT_ZAtomicCmpSwap`, `SDT_ZAtomicLoad128`, `SDT_ZAtomicStore128`, `SDT_ZAtomicCmpSwap128`, `SDT_ZMemMemLength`, `SDT_ZMemMemLengthCC`.
- **CN**: 该代码块声明或细化了 `SDT_ZAtomicCmpSwap`, `SDT_ZAtomicLoad128`, `SDT_ZAtomicStore128`, `SDT_ZAtomicCmpSwap128`, `SDT_ZMemMemLength`, `SDT_ZMemMemLengthCC` 等 TableGen 记录。

### Lines 121-160 / 第 121-160 行
```tablegen
 121:                                              SDTCisPtrTy<2>,
 122:                                              SDTCisVT<3, i32>]>;
 123: def SDT_ZStringCC           : SDTypeProfile<2, 3,
 124:                                             [SDTCisPtrTy<0>,
 125:                                              SDTCisVT<1, i32>,
 126:                                              SDTCisPtrTy<2>,
 127:                                              SDTCisPtrTy<3>,
 128:                                              SDTCisVT<4, i32>]>;
 129: def SDT_ZIPM                : SDTypeProfile<1, 1,
 130:                                             [SDTCisVT<0, i32>,
 131:                                              SDTCisVT<1, i32>]>;
 132: def SDT_ZPrefetch           : SDTypeProfile<0, 2,
 133:                                             [SDTCisVT<0, i32>,
 134:                                              SDTCisPtrTy<1>]>;
 135: def SDT_ZStoreInherent      : SDTypeProfile<0, 1,
 136:                                             [SDTCisPtrTy<0>]>;
 137: def SDT_ZTBegin             : SDTypeProfile<1, 2,
 138:                                             [SDTCisVT<0, i32>,
 139:                                              SDTCisPtrTy<1>,
 140:                                              SDTCisVT<2, i32>]>;
 141: def SDT_ZADAENTRY           : SDTypeProfile<1, 3,
 142:                                             [SDTCisPtrTy<0>,
 143:                                              SDTCisPtrTy<1>,
 144:                                              SDTCisPtrTy<2>,
 145:                                              SDTCisVT<3, i64>]>;
 146: def SDT_ZTEnd               : SDTypeProfile<1, 0,
 147:                                             [SDTCisVT<0, i32>]>;
 148: def SDT_ZInsertVectorElt    : SDTypeProfile<1, 3,
 149:                                             [SDTCisVec<0>,
 150:                                              SDTCisSameAs<0, 1>,
 151:                                              SDTCisVT<3, i32>]>;
 152: def SDT_ZExtractVectorElt   : SDTypeProfile<1, 2,
 153:                                             [SDTCisVec<1>,
 154:                                              SDTCisVT<2, i32>]>;
 155: def SDT_ZReplicate          : SDTypeProfile<1, 1,
 156:                                             [SDTCisVec<0>]>;
 157: def SDT_ZVecUnpack          : SDTypeProfile<1, 1,
 158:                                             [SDTCisVec<1>]>;
 159: def SDT_ZVecUnaryConv       : SDTypeProfile<1, 1,
 160:                                             [SDTCisVec<0>,
```
- **EN**: This block declares or refines TableGen records such as `SDT_ZStringCC`, `SDT_ZIPM`, `SDT_ZPrefetch`, `SDT_ZStoreInherent`, `SDT_ZTBegin`, `SDT_ZADAENTRY`.
- **CN**: 该代码块声明或细化了 `SDT_ZStringCC`, `SDT_ZIPM`, `SDT_ZPrefetch`, `SDT_ZStoreInherent`, `SDT_ZTBegin`, `SDT_ZADAENTRY` 等 TableGen 记录。

### Lines 161-200 / 第 161-200 行
```tablegen
 161:                                              SDTCisVec<1>]>;
 162: def SDT_ZVecUnary           : SDTypeProfile<1, 1,
 163:                                             [SDTCisVec<0>,
 164:                                              SDTCisSameAs<0, 1>]>;
 165: def SDT_ZVecUnaryCC         : SDTypeProfile<2, 1,
 166:                                             [SDTCisVec<0>,
 167:                                              SDTCisVT<1, i32>,
 168:                                              SDTCisSameAs<0, 2>]>;
 169: def SDT_ZVecCompare         : SDTypeProfile<1, 2,
 170:                                             [SDTCisSameAs<0, 1>,
 171:                                              SDTCisSameAs<0, 2>]>;
 172: def SDT_ZVecCompareCC       : SDTypeProfile<2, 2,
 173:                                             [SDTCisVT<1, i32>,
 174:                                              SDTCisSameAs<0, 2>,
 175:                                              SDTCisSameAs<0, 2>]>;
 176: def SDT_ZVecBinary          : SDTypeProfile<1, 2,
 177:                                             [SDTCisVec<0>,
 178:                                              SDTCisSameAs<0, 1>,
 179:                                              SDTCisSameAs<0, 2>]>;
 180: def SDT_ZVecBinaryCC        : SDTypeProfile<2, 2,
 181:                                             [SDTCisVec<0>,
 182:                                              SDTCisVT<1, i32>,
 183:                                              SDTCisSameAs<0, 2>,
 184:                                              SDTCisSameAs<0, 2>]>;
 185: def SDT_ZVecBinaryInt       : SDTypeProfile<1, 2,
 186:                                             [SDTCisVec<0>,
 187:                                              SDTCisSameAs<0, 1>,
 188:                                              SDTCisVT<2, i32>]>;
 189: def SDT_ZVecBinaryConv      : SDTypeProfile<1, 2,
 190:                                             [SDTCisVec<0>,
 191:                                              SDTCisVec<1>,
 192:                                              SDTCisSameAs<1, 2>]>;
 193: def SDT_ZVecBinaryConvCC    : SDTypeProfile<2, 2,
 194:                                             [SDTCisVec<0>,
 195:                                              SDTCisVT<1, i32>,
 196:                                              SDTCisVec<2>,
 197:                                              SDTCisSameAs<2, 3>]>;
 198: def SDT_ZVecBinaryConvIntCC : SDTypeProfile<2, 2,
 199:                                             [SDTCisVec<0>,
 200:                                              SDTCisVT<1, i32>,
```
- **EN**: This block declares or refines TableGen records such as `SDT_ZVecUnary`, `SDT_ZVecUnaryCC`, `SDT_ZVecCompare`, `SDT_ZVecCompareCC`, `SDT_ZVecBinary`, `SDT_ZVecBinaryCC`.
- **CN**: 该代码块声明或细化了 `SDT_ZVecUnary`, `SDT_ZVecUnaryCC`, `SDT_ZVecCompare`, `SDT_ZVecCompareCC`, `SDT_ZVecBinary`, `SDT_ZVecBinaryCC` 等 TableGen 记录。

### Lines 201-240 / 第 201-240 行
```tablegen
 201:                                              SDTCisVec<2>,
 202:                                              SDTCisVT<3, i32>]>;
 203: def SDT_ZRotateMask         : SDTypeProfile<1, 2,
 204:                                             [SDTCisVec<0>,
 205:                                              SDTCisVT<1, i32>,
 206:                                              SDTCisVT<2, i32>]>;
 207: def SDT_ZJoinDwords         : SDTypeProfile<1, 2,
 208:                                             [SDTCisVT<0, v2i64>,
 209:                                              SDTCisVT<1, i64>,
 210:                                              SDTCisVT<2, i64>]>;
 211: def SDT_ZVecTernary         : SDTypeProfile<1, 3,
 212:                                             [SDTCisVec<0>,
 213:                                              SDTCisSameAs<0, 1>,
 214:                                              SDTCisSameAs<0, 2>,
 215:                                              SDTCisSameAs<0, 3>]>;
 216: def SDT_ZVecTernaryConvCC   : SDTypeProfile<2, 3,
 217:                                             [SDTCisVec<0>,
 218:                                              SDTCisVT<1, i32>,
 219:                                              SDTCisVec<2>,
 220:                                              SDTCisSameAs<2, 3>,
 221:                                              SDTCisSameAs<0, 4>]>;
 222: def SDT_ZVecTernaryInt      : SDTypeProfile<1, 3,
 223:                                             [SDTCisVec<0>,
 224:                                              SDTCisSameAs<0, 1>,
 225:                                              SDTCisSameAs<0, 2>,
 226:                                              SDTCisVT<3, i32>]>;
 227: def SDT_ZVecTernaryIntCC    : SDTypeProfile<2, 3,
 228:                                             [SDTCisVec<0>,
 229:                                              SDTCisVT<1, i32>,
 230:                                              SDTCisSameAs<0, 2>,
 231:                                              SDTCisSameAs<0, 3>,
 232:                                              SDTCisVT<4, i32>]>;
 233: def SDT_ZVecQuaternaryInt   : SDTypeProfile<1, 4,
 234:                                             [SDTCisVec<0>,
 235:                                              SDTCisSameAs<0, 1>,
 236:                                              SDTCisSameAs<0, 2>,
 237:                                              SDTCisSameAs<0, 3>,
 238:                                              SDTCisVT<4, i32>]>;
 239: def SDT_ZVecQuaternaryIntCC : SDTypeProfile<2, 4,
 240:                                             [SDTCisVec<0>,
```
- **EN**: This block declares or refines TableGen records such as `SDT_ZRotateMask`, `SDT_ZJoinDwords`, `SDT_ZVecTernary`, `SDT_ZVecTernaryConvCC`, `SDT_ZVecTernaryInt`, `SDT_ZVecTernaryIntCC`.
- **CN**: 该代码块声明或细化了 `SDT_ZRotateMask`, `SDT_ZJoinDwords`, `SDT_ZVecTernary`, `SDT_ZVecTernaryConvCC`, `SDT_ZVecTernaryInt`, `SDT_ZVecTernaryIntCC` 等 TableGen 记录。

### Lines 241-280 / 第 241-280 行
```tablegen
 241:                                              SDTCisVT<1, i32>,
 242:                                              SDTCisSameAs<0, 2>,
 243:                                              SDTCisSameAs<0, 3>,
 244:                                              SDTCisSameAs<0, 4>,
 245:                                              SDTCisVT<5, i32>]>;
 246: def SDT_ZTest               : SDTypeProfile<1, 2,
 247:                                             [SDTCisVT<0, i32>,
 248:                                              SDTCisVT<2, i64>]>;
 249: 
 250: def SDT_ZSetJmp             : SDTypeProfile<1, 1,
 251:                                             [SDTCisInt<0>,
 252:                                              SDTCisPtrTy<1>]>;
 253: def SDT_ZLongJmp            : SDTypeProfile<0, 1, [SDTCisPtrTy<0>]>;
 254: 
 255: 
 256: //===----------------------------------------------------------------------===//
 257: // Node definitions
 258: //===----------------------------------------------------------------------===//
 259: 
 260: // These are target-independent nodes, but have target-specific formats.
 261: def callseq_start       : SDNode<"ISD::CALLSEQ_START", SDT_CallSeqStart,
 262:                                  [SDNPHasChain, SDNPSideEffect, SDNPOutGlue]>;
 263: def callseq_end         : SDNode<"ISD::CALLSEQ_END",   SDT_CallSeqEnd,
 264:                                  [SDNPHasChain, SDNPSideEffect, SDNPOptInGlue,
 265:                                   SDNPOutGlue]>;
 266: def global_offset_table : SDNode<"ISD::GLOBAL_OFFSET_TABLE", SDTPtrLeaf>;
 267: 
 268: // Return with a glue operand.  Operand 0 is the chain operand.
 269: def z_retglue           : SDNode<"SystemZISD::RET_GLUE", SDTNone,
 270:                                  [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
 271: 
 272: // Calls a function.  Operand 0 is the chain operand and operand 1
 273: // is the target address.  The arguments start at operand 2.
 274: // There is an optional glue operand at the end.
 275: def z_call              : SDNode<"SystemZISD::CALL", SDT_ZCall,
 276:                                  [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue,
 277:                                   SDNPVariadic]>;
 278: def z_sibcall           : SDNode<"SystemZISD::SIBCALL", SDT_ZCall,
 279:                                  [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue,
 280:                                   SDNPVariadic]>;
```
- **EN**: This block declares or refines TableGen records such as `SDT_ZTest`, `SDT_ZSetJmp`, `SDT_ZLongJmp`, `callseq_start`, `callseq_end`, `global_offset_table`.
- **CN**: 该代码块声明或细化了 `SDT_ZTest`, `SDT_ZSetJmp`, `SDT_ZLongJmp`, `callseq_start`, `callseq_end`, `global_offset_table` 等 TableGen 记录。

### Lines 281-320 / 第 281-320 行
```tablegen
 281: // TLS calls.  Like regular calls, except operand 1 is the TLS symbol.
 282: // (The call target is implicitly __tls_get_offset.)
 283: def z_tls_gdcall        : SDNode<"SystemZISD::TLS_GDCALL", SDT_ZCall,
 284:                                  [SDNPHasChain, SDNPInGlue, SDNPOutGlue,
 285:                                   SDNPVariadic]>;
 286: def z_tls_ldcall        : SDNode<"SystemZISD::TLS_LDCALL", SDT_ZCall,
 287:                                  [SDNPHasChain, SDNPInGlue, SDNPOutGlue,
 288:                                   SDNPVariadic]>;
 289: 
 290: // Wraps a TargetGlobalAddress that should be loaded using PC-relative
 291: // accesses (LARL).  Operand 0 is the address.
 292: def z_pcrel_wrapper     : SDNode<"SystemZISD::PCREL_WRAPPER", SDT_ZWrapPtr, []>;
 293: 
 294: // Used in cases where an offset is applied to a TargetGlobalAddress.
 295: // Operand 0 is the full TargetGlobalAddress and operand 1 is a
 296: // PCREL_WRAPPER for an anchor point.  This is used so that we can
 297: // cheaply refer to either the full address or the anchor point
 298: // as a register base.
 299: def z_pcrel_offset      : SDNode<"SystemZISD::PCREL_OFFSET",
 300:                                  SDT_ZWrapOffset, []>;
 301: 
 302: // Integer comparisons.  There are three operands: the two values
 303: // to compare, and an integer of type SystemZICMP.
 304: def z_icmp              : SDNode<"SystemZISD::ICMP", SDT_ZICmp>;
 305: 
 306: // Floating-point comparisons.  The two operands are the values to compare.
 307: def z_fcmp              : SDNode<"SystemZISD::FCMP", SDT_ZCmp>;
 308: 
 309: let IsStrictFP = true in {
 310:   // Strict variants of scalar floating-point comparisons.
 311:   // Quiet and signaling versions.
 312:   def z_strict_fcmp : SDNode<"SystemZISD::STRICT_FCMP", SDT_ZCmp,
 313:                              [SDNPHasChain]>;
 314:   def z_strict_fcmps : SDNode<"SystemZISD::STRICT_FCMPS", SDT_ZCmp,
 315:                               [SDNPHasChain]>;
 316: }
 317: 
 318: // Test under mask.  The first operand is ANDed with the second operand
 319: // and the condition codes are set on the result.  The third operand is
 320: // a boolean that is true if the condition codes need to distinguish
```
- **EN**: This block declares or refines TableGen records such as `z_tls_gdcall`, `z_tls_ldcall`, `z_pcrel_wrapper`, `z_pcrel_offset`, `z_icmp`, `z_fcmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `z_tls_gdcall`, `z_tls_ldcall`, `z_pcrel_wrapper`, `z_pcrel_offset`, `z_icmp`, `z_fcmp` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 321-360 / 第 321-360 行
```tablegen
 321: // between CCMASK_TM_MIXED_MSB_0 and CCMASK_TM_MIXED_MSB_1 (which the
 322: // register forms do but the memory forms don't).
 323: def z_tm                : SDNode<"SystemZISD::TM", SDT_ZICmp>;
 324: 
 325: // Branches if a condition is true.  Operand 0 is the chain operand;
 326: // operand 1 is the 4-bit condition-code mask, with bit N in
 327: // big-endian order meaning "branch if CC=N"; operand 2 is the
 328: // target block and operand 3 is the flag operand.
 329: def z_br_ccmask_1       : SDNode<"SystemZISD::BR_CCMASK", SDT_ZBRCCMask,
 330:                                  [SDNPHasChain]>;
 331: 
 332: // Selects between operand 0 and operand 1.  Operand 2 is the
 333: // mask of condition-code values for which operand 0 should be
 334: // chosen over operand 1; it has the same form as BR_CCMASK.
 335: // Operand 3 is the flag operand.
 336: def z_select_ccmask_1   : SDNode<"SystemZISD::SELECT_CCMASK",
 337:                                  SDT_ZSelectCCMask>;
 338: 
 339: // Store the CC value in bits 29 and 28 of an integer.
 340: def z_ipm_1             : SDNode<"SystemZISD::IPM", SDT_ZIPM>;
 341: 
 342: // Evaluates to the gap between the stack pointer and the
 343: // base of the dynamically-allocatable area.
 344: def z_adjdynalloc       : SDNode<"SystemZISD::ADJDYNALLOC", SDT_ZAdjDynAlloc>;
 345: 
 346: // For allocating stack space when using stack clash protector.
 347: // Allocation is performed by block, and each block is probed.
 348: def z_probed_alloca     : SDNode<"SystemZISD::PROBED_ALLOCA", SDT_ZProbedAlloca,
 349:                                  [SDNPHasChain]>;
 350: 
 351: // Count number of bits set in operand 0 per byte.
 352: def z_popcnt            : SDNode<"SystemZISD::POPCNT", SDTIntUnaryOp>;
 353: 
 354: // Wrappers around the ISD opcodes of the same name.  The output is GR128.
 355: // Input operands may be GR64 or GR32, depending on the instruction.
 356: def z_smul_lohi         : SDNode<"SystemZISD::SMUL_LOHI", SDT_ZGR128Binary>;
 357: def z_umul_lohi         : SDNode<"SystemZISD::UMUL_LOHI", SDT_ZGR128Binary>;
 358: def z_sdivrem           : SDNode<"SystemZISD::SDIVREM", SDT_ZGR128Binary>;
 359: def z_udivrem           : SDNode<"SystemZISD::UDIVREM", SDT_ZGR128Binary>;
 360: 
```
- **EN**: This block declares or refines TableGen records such as `z_tm`, `z_br_ccmask_1`, `z_select_ccmask_1`, `z_ipm_1`, `z_adjdynalloc`, `z_probed_alloca`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `z_tm`, `z_br_ccmask_1`, `z_select_ccmask_1`, `z_ipm_1`, `z_adjdynalloc`, `z_probed_alloca` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 361-400 / 第 361-400 行
```tablegen
 361: // Add/subtract with overflow/carry.  These have the same operands as
 362: // the corresponding standard operations, except with the carry flag
 363: // replaced by a condition code value.
 364: def z_saddo             : SDNode<"SystemZISD::SADDO", SDT_ZBinaryWithFlags>;
 365: def z_ssubo             : SDNode<"SystemZISD::SSUBO", SDT_ZBinaryWithFlags>;
 366: def z_uaddo             : SDNode<"SystemZISD::UADDO", SDT_ZBinaryWithFlags>;
 367: def z_usubo             : SDNode<"SystemZISD::USUBO", SDT_ZBinaryWithFlags>;
 368: def z_addcarry_1        : SDNode<"SystemZISD::ADDCARRY", SDT_ZBinaryWithCarry>;
 369: def z_subcarry_1        : SDNode<"SystemZISD::SUBCARRY", SDT_ZBinaryWithCarry>;
 370: 
 371: // Compute carry/borrow indication for add/subtract.
 372: def z_vacc              : SDNode<"SystemZISD::VACC", SDTIntBinOp>;
 373: def z_vscbi             : SDNode<"SystemZISD::VSCBI", SDTIntBinOp>;
 374: 
 375: // Add/subtract with carry/borrow.
 376: def z_vac               : SDNode<"SystemZISD::VAC", SDT_ZTernary>;
 377: def z_vsbi              : SDNode<"SystemZISD::VSBI", SDT_ZTernary>;
 378: 
 379: // Compute carry/borrow indication for add/subtract with carry/borrow.
 380: def z_vaccc             : SDNode<"SystemZISD::VACCC", SDT_ZTernary>;
 381: def z_vsbcbi            : SDNode<"SystemZISD::VSBCBI", SDT_ZTernary>;
 382: 
 383: // High-word multiply-and-add.
 384: def z_vmah              : SDNode<"SystemZISD::VMAH", SDT_ZTernary>;
 385: def z_vmalh             : SDNode<"SystemZISD::VMALH", SDT_ZTernary>;
 386: 
 387: // Widen and multiply even/odd vector elements.
 388: def z_vme               : SDNode<"SystemZISD::VME", SDT_ZBinaryConv>;
 389: def z_vmle              : SDNode<"SystemZISD::VMLE", SDT_ZBinaryConv>;
 390: def z_vmo               : SDNode<"SystemZISD::VMO", SDT_ZBinaryConv>;
 391: def z_vmlo              : SDNode<"SystemZISD::VMLO", SDT_ZBinaryConv>;
 392: 
 393: // Byte swapping load/store.  Same operands as regular load/store.
 394: def z_loadbswap        : SDNode<"SystemZISD::LRV", SDTLoad,
 395:                                  [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
 396: def z_storebswap       : SDNode<"SystemZISD::STRV", SDTStore,
 397:                                  [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
 398: 
 399: // Element swapping load/store.  Same operands as regular load/store.
 400: def z_loadeswap        : SDNode<"SystemZISD::VLER", SDTLoad,
```
- **EN**: This block declares or refines TableGen records such as `z_saddo`, `z_ssubo`, `z_uaddo`, `z_usubo`, `z_addcarry_1`, `z_subcarry_1`.
- **CN**: 该代码块声明或细化了 `z_saddo`, `z_ssubo`, `z_uaddo`, `z_usubo`, `z_addcarry_1`, `z_subcarry_1` 等 TableGen 记录。

### Lines 401-440 / 第 401-440 行
```tablegen
 401:                                  [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
 402: def z_storeeswap       : SDNode<"SystemZISD::VSTER", SDTStore,
 403:                                  [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
 404: 
 405: // Use STORE CLOCK FAST to store current TOD clock value.
 406: def z_stckf            : SDNode<"SystemZISD::STCKF", SDT_ZStoreInherent,
 407:                                 [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
 408: 
 409: // Test Data Class.
 410: //
 411: // Operand 0: the value to test
 412: // Operand 1: the bit mask
 413: def z_tdc               : SDNode<"SystemZISD::TDC", SDT_ZTest>;
 414: 
 415: def z_eh_sjlj_setjmp    : SDNode<"ISD::EH_SJLJ_SETJMP", SDT_ZSetJmp,
 416:                                  [SDNPHasChain, SDNPSideEffect]>;
 417: def z_eh_sjlj_longjmp   : SDNode<"ISD::EH_SJLJ_LONGJMP", SDT_ZLongJmp,
 418:                                  [SDNPHasChain, SDNPSideEffect]>;
 419: 
 420: 
 421: // Defined because the index is an i32 rather than a pointer.
 422: def z_vector_insert     : SDNode<"ISD::INSERT_VECTOR_ELT",
 423:                                  SDT_ZInsertVectorElt>;
 424: def z_vector_extract    : SDNode<"ISD::EXTRACT_VECTOR_ELT",
 425:                                  SDT_ZExtractVectorElt>;
 426: 
 427: // Create a vector constant by filling byte N of the result with bit
 428: // 15-N of the single operand.
 429: def z_byte_mask         : SDNode<"SystemZISD::BYTE_MASK", SDT_ZReplicate>;
 430: 
 431: // Create a vector constant by replicating an element-sized RISBG-style mask.
 432: // The first operand specifies the starting set bit and the second operand
 433: // specifies the ending set bit.  Both operands count from the MSB of the
 434: // element.
 435: def z_rotate_mask       : SDNode<"SystemZISD::ROTATE_MASK", SDT_ZRotateMask>;
 436: 
 437: // Replicate a GPR scalar value into all elements of a vector.
 438: def z_replicate         : SDNode<"SystemZISD::REPLICATE", SDT_ZReplicate>;
 439: 
 440: // Create a vector from two i64 GPRs.
```
- **EN**: This block declares or refines TableGen records such as `z_storeeswap`, `z_stckf`, `z_tdc`, `z_eh_sjlj_setjmp`, `z_eh_sjlj_longjmp`, `z_vector_insert`.
- **CN**: 该代码块声明或细化了 `z_storeeswap`, `z_stckf`, `z_tdc`, `z_eh_sjlj_setjmp`, `z_eh_sjlj_longjmp`, `z_vector_insert` 等 TableGen 记录。

### Lines 441-480 / 第 441-480 行
```tablegen
 441: def z_join_dwords       : SDNode<"SystemZISD::JOIN_DWORDS", SDT_ZJoinDwords>;
 442: 
 443: // Replicate one element of a vector into all elements.  The first operand
 444: // is the vector and the second is the index of the element to replicate.
 445: def z_splat             : SDNode<"SystemZISD::SPLAT", SDT_ZVecBinaryInt>;
 446: 
 447: // Interleave elements from the high half of operand 0 and the high half
 448: // of operand 1.
 449: def z_merge_high        : SDNode<"SystemZISD::MERGE_HIGH", SDT_ZVecBinary>;
 450: 
 451: // Likewise for the low halves.
 452: def z_merge_low         : SDNode<"SystemZISD::MERGE_LOW", SDT_ZVecBinary>;
 453: 
 454: // Concatenate the vectors in the first two operands, shift them left
 455: // by the third operand, and take the first half of the result.
 456: def z_shl_double        : SDNode<"SystemZISD::SHL_DOUBLE", SDT_ZVecTernaryInt>;
 457: 
 458: // Concatenate the vectors in the first two operands, shift them left/right
 459: // bitwise by the third operand, and take the first/last half of the result.
 460: def z_shl_double_bit    : SDNode<"SystemZISD::SHL_DOUBLE_BIT", SDT_ZVecTernaryInt>;
 461: def z_shr_double_bit    : SDNode<"SystemZISD::SHR_DOUBLE_BIT", SDT_ZVecTernaryInt>;
 462: 
 463: // Take one element of the first v2i64 operand and the one element of
 464: // the second v2i64 operand and concatenate them to form a v2i64 result.
 465: // The third operand is a 4-bit value of the form 0A0B, where A and B
 466: // are the element selectors for the first operand and second operands
 467: // respectively.
 468: def z_permute_dwords    : SDNode<"SystemZISD::PERMUTE_DWORDS",
 469:                                  SDT_ZVecTernaryInt>;
 470: 
 471: // Perform a general vector permute on vector operands 0 and 1.
 472: // Each byte of operand 2 controls the corresponding byte of the result,
 473: // in the same way as a byte-level VECTOR_SHUFFLE mask.
 474: def z_permute           : SDNode<"SystemZISD::PERMUTE", SDT_ZVecTernary>;
 475: 
 476: // Pack vector operands 0 and 1 into a single vector with half-sized elements.
 477: def z_pack              : SDNode<"SystemZISD::PACK", SDT_ZVecBinaryConv>;
 478: 
 479: // Likewise, but saturate the result and set CC.  PACKS_CC does signed
 480: // saturation and PACKLS_CC does unsigned saturation.
```
- **EN**: This block declares or refines TableGen records such as `z_join_dwords`, `z_splat`, `z_merge_high`, `z_merge_low`, `z_shl_double`, `z_shl_double_bit`.
- **CN**: 该代码块声明或细化了 `z_join_dwords`, `z_splat`, `z_merge_high`, `z_merge_low`, `z_shl_double`, `z_shl_double_bit` 等 TableGen 记录。

### Lines 481-520 / 第 481-520 行
```tablegen
 481: def z_packs_cc          : SDNode<"SystemZISD::PACKS_CC", SDT_ZVecBinaryConvCC>;
 482: def z_packls_cc         : SDNode<"SystemZISD::PACKLS_CC", SDT_ZVecBinaryConvCC>;
 483: 
 484: // Unpack the first half of vector operand 0 into double-sized elements.
 485: // UNPACK_HIGH sign-extends and UNPACKL_HIGH zero-extends.
 486: def z_unpack_high       : SDNode<"SystemZISD::UNPACK_HIGH", SDT_ZVecUnpack>;
 487: def z_unpackl_high      : SDNode<"SystemZISD::UNPACKL_HIGH", SDT_ZVecUnpack>;
 488: 
 489: // Likewise for the second half.
 490: def z_unpack_low        : SDNode<"SystemZISD::UNPACK_LOW", SDT_ZVecUnpack>;
 491: def z_unpackl_low       : SDNode<"SystemZISD::UNPACKL_LOW", SDT_ZVecUnpack>;
 492: 
 493: // Shift/rotate each element of vector operand 0 by the number of bits
 494: // specified by scalar operand 1.
 495: def z_vshl_by_scalar    : SDNode<"SystemZISD::VSHL_BY_SCALAR",
 496:                                  SDT_ZVecBinaryInt>;
 497: def z_vsrl_by_scalar    : SDNode<"SystemZISD::VSRL_BY_SCALAR",
 498:                                  SDT_ZVecBinaryInt>;
 499: def z_vsra_by_scalar    : SDNode<"SystemZISD::VSRA_BY_SCALAR",
 500:                                  SDT_ZVecBinaryInt>;
 501: def z_vrotl_by_scalar   : SDNode<"SystemZISD::VROTL_BY_SCALAR",
 502:                                  SDT_ZVecBinaryInt>;
 503: 
 504: // For each element of the output type, sum across all sub-elements of
 505: // operand 0 belonging to the corresponding element, and add in the
 506: // rightmost sub-element of the corresponding element of operand 1.
 507: def z_vsum              : SDNode<"SystemZISD::VSUM", SDT_ZBinaryConv>;
 508: 
 509: // Compare integer vector operands 0 and 1 to produce the usual 0/-1
 510: // vector result.  VICMPE is for equality, VICMPH for "signed greater than"
 511: // and VICMPHL for "unsigned greater than".
 512: def z_vicmpe            : SDNode<"SystemZISD::VICMPE", SDT_ZVecCompare>;
 513: def z_vicmph            : SDNode<"SystemZISD::VICMPH", SDT_ZVecCompare>;
 514: def z_vicmphl           : SDNode<"SystemZISD::VICMPHL", SDT_ZVecCompare>;
 515: 
 516: // Likewise, but also set the condition codes on the result.
 517: def z_vicmpes           : SDNode<"SystemZISD::VICMPES", SDT_ZVecCompareCC>;
 518: def z_vicmphs           : SDNode<"SystemZISD::VICMPHS", SDT_ZVecCompareCC>;
 519: def z_vicmphls          : SDNode<"SystemZISD::VICMPHLS", SDT_ZVecCompareCC>;
 520: 
```
- **EN**: This block declares or refines TableGen records such as `z_packs_cc`, `z_packls_cc`, `z_unpack_high`, `z_unpackl_high`, `z_unpack_low`, `z_unpackl_low`.
- **CN**: 该代码块声明或细化了 `z_packs_cc`, `z_packls_cc`, `z_unpack_high`, `z_unpackl_high`, `z_unpack_low`, `z_unpackl_low` 等 TableGen 记录。

### Lines 521-560 / 第 521-560 行
```tablegen
 521: // Compare floating-point vector operands 0 and 1 to produce the usual 0/-1
 522: // vector result.  VFCMPE is for "ordered and equal", VFCMPH for "ordered and
 523: // greater than" and VFCMPHE for "ordered and greater than or equal to".
 524: def z_vfcmpe            : SDNode<"SystemZISD::VFCMPE", SDT_ZVecBinaryConv>;
 525: def z_vfcmph            : SDNode<"SystemZISD::VFCMPH", SDT_ZVecBinaryConv>;
 526: def z_vfcmphe           : SDNode<"SystemZISD::VFCMPHE", SDT_ZVecBinaryConv>;
 527: 
 528: // Likewise, but also set the condition codes on the result.
 529: def z_vfcmpes           : SDNode<"SystemZISD::VFCMPES", SDT_ZVecBinaryConvCC>;
 530: def z_vfcmphs           : SDNode<"SystemZISD::VFCMPHS", SDT_ZVecBinaryConvCC>;
 531: def z_vfcmphes          : SDNode<"SystemZISD::VFCMPHES", SDT_ZVecBinaryConvCC>;
 532: 
 533: // Extend the even f32 elements of vector operand 0 to produce a vector
 534: // of f64 elements.
 535: def z_vextend           : SDNode<"SystemZISD::VEXTEND", SDT_ZVecUnaryConv>;
 536: 
 537: // Round the f64 elements of vector operand 0 to f32s and store them in the
 538: // even elements of the result.
 539: def z_vround            : SDNode<"SystemZISD::VROUND", SDT_ZVecUnaryConv>;
 540: 
 541: let IsStrictFP = true in {
 542:   // Strict variants of vector floating-point comparisons.
 543:   // Quiet and signaling versions.
 544:   def z_strict_vfcmpe   : SDNode<"SystemZISD::STRICT_VFCMPE",
 545:                                  SDT_ZVecBinaryConv, [SDNPHasChain]>;
 546:   def z_strict_vfcmph   : SDNode<"SystemZISD::STRICT_VFCMPH",
 547:                                  SDT_ZVecBinaryConv, [SDNPHasChain]>;
 548:   def z_strict_vfcmphe  : SDNode<"SystemZISD::STRICT_VFCMPHE",
 549:                                  SDT_ZVecBinaryConv, [SDNPHasChain]>;
 550:   def z_strict_vfcmpes  : SDNode<"SystemZISD::STRICT_VFCMPES",
 551:                                  SDT_ZVecBinaryConv, [SDNPHasChain]>;
 552:   def z_strict_vfcmphs  : SDNode<"SystemZISD::STRICT_VFCMPHS",
 553:                                  SDT_ZVecBinaryConv, [SDNPHasChain]>;
 554:   def z_strict_vfcmphes : SDNode<"SystemZISD::STRICT_VFCMPHES",
 555:                                  SDT_ZVecBinaryConv, [SDNPHasChain]>;
 556: 
 557:   // Strict variants of VEXTEND and VROUND.
 558:   def z_strict_vextend  : SDNode<"SystemZISD::STRICT_VEXTEND",
 559:                                  SDT_ZVecUnaryConv, [SDNPHasChain]>;
 560:   def z_strict_vround   : SDNode<"SystemZISD::STRICT_VROUND",
```
- **EN**: This block declares or refines TableGen records such as `z_vfcmpe`, `z_vfcmph`, `z_vfcmphe`, `z_vfcmpes`, `z_vfcmphs`, `z_vfcmphes`.
- **CN**: 该代码块声明或细化了 `z_vfcmpe`, `z_vfcmph`, `z_vfcmphe`, `z_vfcmpes`, `z_vfcmphs`, `z_vfcmphes` 等 TableGen 记录。

### Lines 561-600 / 第 561-600 行
```tablegen
 561:                                  SDT_ZVecUnaryConv, [SDNPHasChain]>;
 562: }
 563: 
 564: // AND the two vector operands together and set CC based on the result.
 565: def z_vtm               : SDNode<"SystemZISD::VTM", SDT_ZCmp>;
 566: 
 567: // i128 high integer comparisons.
 568: def z_scmp128hi         : SDNode<"SystemZISD::SCMP128HI", SDT_ZCmp>;
 569: def z_ucmp128hi         : SDNode<"SystemZISD::UCMP128HI", SDT_ZCmp>;
 570: 
 571: // String operations that set CC as a side-effect.
 572: def z_vfae_cc           : SDNode<"SystemZISD::VFAE_CC", SDT_ZVecTernaryIntCC>;
 573: def z_vfaez_cc          : SDNode<"SystemZISD::VFAEZ_CC", SDT_ZVecTernaryIntCC>;
 574: def z_vfee_cc           : SDNode<"SystemZISD::VFEE_CC", SDT_ZVecBinaryCC>;
 575: def z_vfeez_cc          : SDNode<"SystemZISD::VFEEZ_CC", SDT_ZVecBinaryCC>;
 576: def z_vfene_cc          : SDNode<"SystemZISD::VFENE_CC", SDT_ZVecBinaryCC>;
 577: def z_vfenez_cc         : SDNode<"SystemZISD::VFENEZ_CC", SDT_ZVecBinaryCC>;
 578: def z_vistr_cc          : SDNode<"SystemZISD::VISTR_CC", SDT_ZVecUnaryCC>;
 579: def z_vstrc_cc          : SDNode<"SystemZISD::VSTRC_CC",
 580:                                  SDT_ZVecQuaternaryIntCC>;
 581: def z_vstrcz_cc         : SDNode<"SystemZISD::VSTRCZ_CC",
 582:                                  SDT_ZVecQuaternaryIntCC>;
 583: def z_vstrs_cc          : SDNode<"SystemZISD::VSTRS_CC",
 584:                                  SDT_ZVecTernaryConvCC>;
 585: def z_vstrsz_cc         : SDNode<"SystemZISD::VSTRSZ_CC",
 586:                                  SDT_ZVecTernaryConvCC>;
 587: 
 588: // Test floating-point data class for vectors.
 589: def z_vftci             : SDNode<"SystemZISD::VFTCI", SDT_ZVecBinaryConvIntCC>;
 590: 
 591: class AtomicWOp<string name, SDTypeProfile profile = SDT_ZAtomicLoadBinaryW>
 592:   : SDNode<"SystemZISD::"#name, profile,
 593:            [SDNPHasChain, SDNPMayStore, SDNPMayLoad, SDNPMemOperand]>;
 594: 
 595: // Wrappers around the inner loop of an 8- or 16-bit ATOMIC_SWAP or
 596: // ATOMIC_LOAD_<op>.
 597: //
 598: // Operand 0: the address of the containing 32-bit-aligned field
 599: // Operand 1: the second operand of <op>, in the high bits of an i32
 600: //            for everything except ATOMIC_SWAPW
```
- **EN**: This block declares or refines TableGen records such as `z_vtm`, `z_scmp128hi`, `z_ucmp128hi`, `z_vfae_cc`, `z_vfaez_cc`, `z_vfee_cc`.
- **CN**: 该代码块声明或细化了 `z_vtm`, `z_scmp128hi`, `z_ucmp128hi`, `z_vfae_cc`, `z_vfaez_cc`, `z_vfee_cc` 等 TableGen 记录。

### Lines 601-640 / 第 601-640 行
```tablegen
 601: // Operand 2: how many bits to rotate the i32 left to bring the first
 602: //            operand into the high bits
 603: // Operand 3: the negative of operand 2, for rotating the other way
 604: // Operand 4: the width of the field in bits (8 or 16)
 605: def z_atomic_swapw      : AtomicWOp<"ATOMIC_SWAPW">;
 606: def z_atomic_loadw_add  : AtomicWOp<"ATOMIC_LOADW_ADD">;
 607: def z_atomic_loadw_sub  : AtomicWOp<"ATOMIC_LOADW_SUB">;
 608: def z_atomic_loadw_and  : AtomicWOp<"ATOMIC_LOADW_AND">;
 609: def z_atomic_loadw_or   : AtomicWOp<"ATOMIC_LOADW_OR">;
 610: def z_atomic_loadw_xor  : AtomicWOp<"ATOMIC_LOADW_XOR">;
 611: def z_atomic_loadw_nand : AtomicWOp<"ATOMIC_LOADW_NAND">;
 612: def z_atomic_loadw_min  : AtomicWOp<"ATOMIC_LOADW_MIN">;
 613: def z_atomic_loadw_max  : AtomicWOp<"ATOMIC_LOADW_MAX">;
 614: def z_atomic_loadw_umin : AtomicWOp<"ATOMIC_LOADW_UMIN">;
 615: def z_atomic_loadw_umax : AtomicWOp<"ATOMIC_LOADW_UMAX">;
 616: 
 617: // Atomic compare-and-swap returning CC value.
 618: // Val, CC, OUTCHAIN = ATOMIC_CMP_SWAP(INCHAIN, ptr, cmp, swap)
 619: def z_atomic_cmp_swap   : SDNode<"SystemZISD::ATOMIC_CMP_SWAP",
 620:                                  SDT_ZAtomicCmpSwap,
 621:                                  [SDNPHasChain, SDNPMayStore, SDNPMayLoad,
 622:                                   SDNPMemOperand]>;
 623: 
 624: // A wrapper around the inner loop of an ATOMIC_CMP_SWAP.
 625: //
 626: // Operand 0: the address of the containing 32-bit-aligned field
 627: // Operand 1: the compare value, in the low bits of an i32
 628: // Operand 2: the swap value, in the low bits of an i32
 629: // Operand 3: how many bits to rotate the i32 left to bring the first
 630: //            operand into the high bits
 631: // Operand 4: the negative of operand 2, for rotating the other way
 632: // Operand 5: the width of the field in bits (8 or 16)
 633: def z_atomic_cmp_swapw  : SDNode<"SystemZISD::ATOMIC_CMP_SWAPW",
 634:                                  SDT_ZAtomicCmpSwapW,
 635:                                  [SDNPHasChain, SDNPMayStore, SDNPMayLoad,
 636:                                   SDNPMemOperand]>;
 637: 
 638: // 128-bit atomic load.
 639: // Val, OUTCHAIN = ATOMIC_LOAD_128(INCHAIN, ptr)
 640: def z_atomic_load_128   : SDNode<"SystemZISD::ATOMIC_LOAD_128",
```
- **EN**: This block declares or refines TableGen records such as `z_atomic_swapw`, `z_atomic_loadw_add`, `z_atomic_loadw_sub`, `z_atomic_loadw_and`, `z_atomic_loadw_or`, `z_atomic_loadw_xor`.
- **CN**: 该代码块声明或细化了 `z_atomic_swapw`, `z_atomic_loadw_add`, `z_atomic_loadw_sub`, `z_atomic_loadw_and`, `z_atomic_loadw_or`, `z_atomic_loadw_xor` 等 TableGen 记录。

### Lines 641-680 / 第 641-680 行
```tablegen
 641:                                  SDT_ZAtomicLoad128,
 642:                                  [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
 643: 
 644: // 128-bit atomic store.
 645: // OUTCHAIN = ATOMIC_STORE_128(INCHAIN, val, ptr)
 646: def z_atomic_store_128  : SDNode<"SystemZISD::ATOMIC_STORE_128",
 647:                                  SDT_ZAtomicStore128,
 648:                                  [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
 649: 
 650: // 128-bit atomic compare-and-swap.
 651: // Val, CC, OUTCHAIN = ATOMIC_CMP_SWAP(INCHAIN, ptr, cmp, swap)
 652: def z_atomic_cmp_swap_128 : SDNode<"SystemZISD::ATOMIC_CMP_SWAP_128",
 653:                                    SDT_ZAtomicCmpSwap128,
 654:                                    [SDNPHasChain, SDNPMayStore, SDNPMayLoad,
 655:                                     SDNPMemOperand]>;
 656: 
 657: // Use a series of MVCs to copy bytes from one memory location to another.
 658: // The operands are:
 659: // - the target address
 660: // - the source address
 661: // - the constant length
 662: //
 663: // This isn't a memory opcode because we'd need to attach two
 664: // MachineMemOperands rather than one.
 665: def z_mvc               : SDNode<"SystemZISD::MVC", SDT_ZMemMemLength,
 666:                                  [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
 667: 
 668: // Similar to MVC, but for logic operations (AND, OR, XOR).
 669: def z_nc                : SDNode<"SystemZISD::NC", SDT_ZMemMemLength,
 670:                                   [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
 671: def z_oc                : SDNode<"SystemZISD::OC", SDT_ZMemMemLength,
 672:                                   [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
 673: def z_xc                : SDNode<"SystemZISD::XC", SDT_ZMemMemLength,
 674:                                   [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
 675: 
 676: // Use CLC to compare two blocks of memory, with the same comments
 677: // as for MVC.
 678: def z_clc               : SDNode<"SystemZISD::CLC", SDT_ZMemMemLengthCC,
 679:                                  [SDNPHasChain, SDNPMayLoad]>;
 680: 
```
- **EN**: This block declares or refines TableGen records such as `z_atomic_store_128`, `z_atomic_cmp_swap_128`, `z_mvc`, `z_nc`, `z_oc`, `z_xc`.
- **CN**: 该代码块声明或细化了 `z_atomic_store_128`, `z_atomic_cmp_swap_128`, `z_mvc`, `z_nc`, `z_oc`, `z_xc` 等 TableGen 记录。

### Lines 681-720 / 第 681-720 行
```tablegen
 681: // Use MVC to set a block of memory after storing the first byte.
 682: def z_memset_mvc        : SDNode<"SystemZISD::MEMSET_MVC", SDT_ZMemsetMVC,
 683:                                  [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
 684: 
 685: // Use a CLST-based sequence to implement strcmp().  The two input operands
 686: // are the addresses of the strings to compare.
 687: def z_strcmp            : SDNode<"SystemZISD::STRCMP", SDT_ZStringCC,
 688:                                  [SDNPHasChain, SDNPMayLoad]>;
 689: 
 690: // Use an MVST-based sequence to implement stpcpy().
 691: def z_stpcpy            : SDNode<"SystemZISD::STPCPY", SDT_ZString,
 692:                                  [SDNPHasChain, SDNPMayStore, SDNPMayLoad]>;
 693: 
 694: // Use an SRST-based sequence to search a block of memory.  The first
 695: // operand is the end address, the second is the start, and the third
 696: // is the character to search for.  CC is set to 1 on success and 2
 697: // on failure.
 698: def z_search_string     : SDNode<"SystemZISD::SEARCH_STRING", SDT_ZStringCC,
 699:                                  [SDNPHasChain, SDNPMayLoad]>;
 700: 
 701: // Prefetch from the second operand using the 4-bit control code in
 702: // the first operand.  The code is 1 for a load prefetch and 2 for
 703: // a store prefetch.
 704: def z_prefetch          : SDNode<"SystemZISD::PREFETCH", SDT_ZPrefetch,
 705:                                  [SDNPHasChain, SDNPMayLoad, SDNPMayStore,
 706:                                   SDNPMemOperand]>;
 707: 
 708: // Transaction begin.  The first operand is the chain, the second
 709: // the TDB pointer, and the third the immediate control field.
 710: // Returns CC value and chain.
 711: def z_tbegin            : SDNode<"SystemZISD::TBEGIN", SDT_ZTBegin,
 712:                                  [SDNPHasChain, SDNPMayStore, SDNPSideEffect]>;
 713: def z_tbegin_nofloat    : SDNode<"SystemZISD::TBEGIN_NOFLOAT", SDT_ZTBegin,
 714:                                  [SDNPHasChain, SDNPMayStore, SDNPSideEffect]>;
 715: 
 716: // Transaction end.  Just the chain operand.  Returns CC value and chain.
 717: def z_tend              : SDNode<"SystemZISD::TEND", SDT_ZTEnd,
 718:                                  [SDNPHasChain, SDNPSideEffect]>;
 719: 
 720: // z/OS XPLINK ADA Entry
```
- **EN**: This block declares or refines TableGen records such as `z_memset_mvc`, `z_strcmp`, `z_stpcpy`, `z_search_string`, `z_prefetch`, `z_tbegin`.
- **CN**: 该代码块声明或细化了 `z_memset_mvc`, `z_strcmp`, `z_stpcpy`, `z_search_string`, `z_prefetch`, `z_tbegin` 等 TableGen 记录。

### Lines 721-760 / 第 721-760 行
```tablegen
 721: // Wraps a TargetGlobalAddress that should be loaded from a function's
 722: // AssociatedData Area (ADA). Tha ADA is passed to the function by the
 723: // caller in the XPLink ABI defined register R5.
 724: // Operand 0: the GlobalValue/External Symbol
 725: // Operand 1: the ADA register
 726: // Operand 2: the offset (0 for the first and 8 for the second element in the
 727: // function descriptor)
 728: def z_ada_entry         : SDNode<"SystemZISD::ADA_ENTRY",
 729:                                   SDT_ZADAENTRY>;
 730: 
 731: def z_vshl              : SDNode<"ISD::SHL", SDT_ZVecBinary>;
 732: def z_vsra              : SDNode<"ISD::SRA", SDT_ZVecBinary>;
 733: def z_vsrl              : SDNode<"ISD::SRL", SDT_ZVecBinary>;
 734: 
 735: //===----------------------------------------------------------------------===//
 736: // Pattern fragments
 737: //===----------------------------------------------------------------------===//
 738: 
 739: def z_loadbswap16 : PatFrag<(ops node:$addr), (z_loadbswap node:$addr), [{
 740:   return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i16;
 741: }]>;
 742: def z_loadbswap32 : PatFrag<(ops node:$addr), (z_loadbswap node:$addr), [{
 743:   return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i32;
 744: }]>;
 745: def z_loadbswap64 : PatFrag<(ops node:$addr), (z_loadbswap node:$addr), [{
 746:   return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i64;
 747: }]>;
 748: 
 749: def z_storebswap16 : PatFrag<(ops node:$src, node:$addr),
 750:                              (z_storebswap node:$src, node:$addr), [{
 751:   return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i16;
 752: }]>;
 753: def z_storebswap32 : PatFrag<(ops node:$src, node:$addr),
 754:                              (z_storebswap node:$src, node:$addr), [{
 755:   return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i32;
 756: }]>;
 757: def z_storebswap64 : PatFrag<(ops node:$src, node:$addr),
 758:                              (z_storebswap node:$src, node:$addr), [{
 759:   return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i64;
 760: }]>;
```
- **EN**: This block declares or refines TableGen records such as `z_ada_entry`, `z_vshl`, `z_vsra`, `z_vsrl`, `z_loadbswap16`, `z_loadbswap32`.
- **CN**: 该代码块声明或细化了 `z_ada_entry`, `z_vshl`, `z_vsra`, `z_vsrl`, `z_loadbswap16`, `z_loadbswap32` 等 TableGen 记录。

### Lines 761-800 / 第 761-800 行
```tablegen
 761: 
 762: // Fragments including CC as an implicit source.
 763: def z_br_ccmask
 764:   : PatFrag<(ops node:$valid, node:$mask, node:$bb),
 765:             (z_br_ccmask_1 node:$valid, node:$mask, node:$bb, CC)>;
 766: def z_select_ccmask
 767:   : PatFrag<(ops node:$true, node:$false, node:$valid, node:$mask),
 768:             (z_select_ccmask_1 node:$true, node:$false,
 769:                                node:$valid, node:$mask, CC)>;
 770: def z_ipm : PatFrag<(ops), (z_ipm_1 CC)>;
 771: def z_addcarry : PatFrag<(ops node:$lhs, node:$rhs),
 772:                               (z_addcarry_1 node:$lhs, node:$rhs, CC)>;
 773: def z_subcarry : PatFrag<(ops node:$lhs, node:$rhs),
 774:                               (z_subcarry_1 node:$lhs, node:$rhs, CC)>;
 775: 
 776: // Signed and unsigned comparisons.
 777: def z_scmp : PatFrag<(ops node:$a, node:$b), (z_icmp node:$a, node:$b, timm), [{
 778:   unsigned Type = N->getConstantOperandVal(2);
 779:   return Type != SystemZICMP::UnsignedOnly;
 780: }]>;
 781: def z_ucmp : PatFrag<(ops node:$a, node:$b), (z_icmp node:$a, node:$b, timm), [{
 782:   unsigned Type = N->getConstantOperandVal(2);
 783:   return Type != SystemZICMP::SignedOnly;
 784: }]>;
 785: 
 786: // Register- and memory-based TEST UNDER MASK.
 787: def z_tm_reg : PatFrag<(ops node:$a, node:$b), (z_tm node:$a, node:$b, timm)>;
 788: def z_tm_mem : PatFrag<(ops node:$a, node:$b), (z_tm node:$a, node:$b, 0)>;
 789: 
 790: // Shifts by small immediate amounts.
 791: def shl1 : PatFrag<(ops node:$src), (shl node:$src, (i32 1))>;
 792: def shl2 : PatFrag<(ops node:$src), (shl node:$src, (i32 2))>;
 793: def shl3 : PatFrag<(ops node:$src), (shl node:$src, (i32 3))>;
 794: def shl4 : PatFrag<(ops node:$src), (shl node:$src, (i32 4))>;
 795: 
 796: // Register sign-extend operations.  Sub-32-bit values are represented as i32s.
 797: def sext8  : PatFrag<(ops node:$src), (sext_inreg node:$src, i8)>;
 798: def sext16 : PatFrag<(ops node:$src), (sext_inreg node:$src, i16)>;
 799: def sext32 : PatFrag<(ops node:$src), (sext (i32 node:$src))>;
 800: 
```
- **EN**: This block declares or refines TableGen records such as `z_br_ccmask`, `z_select_ccmask`, `z_ipm`, `z_addcarry`, `z_subcarry`, `z_scmp`.
- **CN**: 该代码块声明或细化了 `z_br_ccmask`, `z_select_ccmask`, `z_ipm`, `z_addcarry`, `z_subcarry`, `z_scmp` 等 TableGen 记录。

### Lines 801-840 / 第 801-840 行
```tablegen
 801: // Match extensions of an i32 to an i64, followed by an in-register sign
 802: // extension from a sub-i32 value.
 803: def sext8dbl : PatFrag<(ops node:$src), (sext8 (anyext node:$src))>;
 804: def sext16dbl : PatFrag<(ops node:$src), (sext16 (anyext node:$src))>;
 805: 
 806: // Register zero-extend operations.  Sub-32-bit values are represented as i32s.
 807: def zext8  : PatFrag<(ops node:$src), (and node:$src, 0xff)>;
 808: def zext16 : PatFrag<(ops node:$src), (and node:$src, 0xffff)>;
 809: def zext32 : PatFrag<(ops node:$src), (zext (i32 node:$src))>;
 810: 
 811: // Match a 64-bit value that is guaranteed to have been sign-
 812: // or zero-extended from a 32-bit value.
 813: def assertsext32 : PatFrag<(ops node:$src), (assertsext node:$src), [{
 814:   return cast<VTSDNode>(N->getOperand(1))->getVT() == MVT::i32;
 815: }]>;
 816: def assertzext32 : PatFrag<(ops node:$src), (assertzext node:$src), [{
 817:   return cast<VTSDNode>(N->getOperand(1))->getVT() == MVT::i32;
 818: }]>;
 819: 
 820: // Match a load or a non-extending atomic load.
 821: def z_load : PatFrags<(ops node:$ptr),
 822:                       [(load node:$ptr),
 823:                        (atomic_load node:$ptr)], [{
 824:   if (auto *AL = dyn_cast<AtomicSDNode>(N))
 825:     if (AL->getExtensionType() != ISD::NON_EXTLOAD)
 826:       return false;
 827:   return true;
 828: }]>;
 829: 
 830: // Sign extending (atomic) loads.
 831: def z_sextload : PatFrags<(ops node:$ptr),
 832:                           [(unindexedload node:$ptr),
 833:                            (atomic_load node:$ptr)], [{
 834:   return getLoadExtType(N) == ISD::SEXTLOAD;
 835: }]>;
 836: def z_sextloadi8 : PatFrag<(ops node:$ptr), (z_sextload node:$ptr), [{
 837:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i8;
 838: }]>;
 839: def z_sextloadi16 : PatFrag<(ops node:$ptr), (z_sextload node:$ptr), [{
 840:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i16;
```
- **EN**: This block declares or refines TableGen records such as `sext8dbl`, `sext16dbl`, `zext8`, `zext16`, `zext32`, `assertsext32`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `sext8dbl`, `sext16dbl`, `zext8`, `zext16`, `zext32`, `assertsext32` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-880 / 第 841-880 行
```tablegen
 841: }]>;
 842: def z_sextloadi32 : PatFrag<(ops node:$ptr), (z_sextload node:$ptr), [{
 843:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i32;
 844: }]>;
 845: def z_sextloadi64 : PatFrag<(ops node:$ptr), (z_sextload node:$ptr), [{
 846:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i64;
 847: }]>;
 848: 
 849: // Zero extending (atomic) loads.
 850: def z_zextload : PatFrags<(ops node:$ptr),
 851:                           [(unindexedload node:$ptr),
 852:                            (atomic_load node:$ptr)], [{
 853:   return getLoadExtType(N) == ISD::ZEXTLOAD;
 854: }]>;
 855: def z_zextloadi8 : PatFrag<(ops node:$ptr), (z_zextload node:$ptr), [{
 856:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i8;
 857: }]>;
 858: def z_zextloadi16 : PatFrag<(ops node:$ptr), (z_zextload node:$ptr), [{
 859:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i16;
 860: }]>;
 861: def z_zextloadi32 : PatFrag<(ops node:$ptr), (z_zextload node:$ptr), [{
 862:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i32;
 863: }]>;
 864: def z_zextloadi64 : PatFrag<(ops node:$ptr), (z_zextload node:$ptr), [{
 865:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i64;
 866: }]>;
 867: 
 868: // Extending (atomic) loads in which the extension type can be signed.
 869: def z_asextload : PatFrags<(ops node:$ptr),
 870:                            [(unindexedload node:$ptr),
 871:                             (atomic_load node:$ptr)], [{
 872:   ISD::LoadExtType ETy = getLoadExtType(N);
 873:   return ETy == ISD::EXTLOAD || ETy == ISD::SEXTLOAD;
 874: }]>;
 875: def z_asextloadi8 : PatFrag<(ops node:$ptr), (z_asextload node:$ptr), [{
 876:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i8;
 877: }]>;
 878: def z_asextloadi16 : PatFrag<(ops node:$ptr), (z_asextload node:$ptr), [{
 879:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i16;
 880: }]>;
```
- **EN**: This block declares or refines TableGen records such as `z_sextloadi32`, `z_sextloadi64`, `z_zextload`, `z_zextloadi8`, `z_zextloadi16`, `z_zextloadi32`.
- **CN**: 该代码块声明或细化了 `z_sextloadi32`, `z_sextloadi64`, `z_zextload`, `z_zextloadi8`, `z_zextloadi16`, `z_zextloadi32` 等 TableGen 记录。

### Lines 881-920 / 第 881-920 行
```tablegen
 881: def z_asextloadi32 : PatFrag<(ops node:$ptr), (z_asextload node:$ptr), [{
 882:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i32;
 883: }]>;
 884: 
 885: // Extending (atomic) loads in which the extension type can be unsigned.
 886: def z_azextload : PatFrags<(ops node:$ptr),
 887:                            [(unindexedload node:$ptr),
 888:                            (atomic_load node:$ptr)], [{
 889:   ISD::LoadExtType ETy = getLoadExtType(N);
 890:   return ETy == ISD::EXTLOAD || ETy == ISD::ZEXTLOAD;
 891: }]>;
 892: def z_azextloadi8 : PatFrag<(ops node:$ptr), (z_azextload node:$ptr), [{
 893:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i8;
 894: }]>;
 895: def z_azextloadi16 : PatFrag<(ops node:$ptr), (z_azextload node:$ptr), [{
 896:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i16;
 897: }]>;
 898: def z_azextloadi32 : PatFrag<(ops node:$ptr), (z_azextload node:$ptr), [{
 899:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i32;
 900: }]>;
 901: 
 902: // Extending (atomic) loads in which the extension type doesn't matter.
 903: def z_anyextload : PatFrags<(ops node:$ptr),
 904:                             [(unindexedload node:$ptr),
 905:                              (atomic_load node:$ptr)], [{
 906:   return getLoadExtType(N) != ISD::NON_EXTLOAD;
 907: }]>;
 908: def z_anyextloadi8 : PatFrag<(ops node:$ptr), (z_anyextload node:$ptr), [{
 909:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i8;
 910: }]>;
 911: def z_anyextloadi16 : PatFrag<(ops node:$ptr), (z_anyextload node:$ptr), [{
 912:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i16;
 913: }]>;
 914: def z_anyextloadi32 : PatFrag<(ops node:$ptr), (z_anyextload node:$ptr), [{
 915:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i32;
 916: }]>;
 917: def z_anyextloadi64 : PatFrag<(ops node:$ptr), (z_anyextload node:$ptr), [{
 918:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i64;
 919: }]>;
 920: 
```
- **EN**: This block declares or refines TableGen records such as `z_asextloadi32`, `z_azextload`, `z_azextloadi8`, `z_azextloadi16`, `z_azextloadi32`, `z_anyextload`.
- **CN**: 该代码块声明或细化了 `z_asextloadi32`, `z_azextload`, `z_azextloadi8`, `z_azextloadi16`, `z_azextloadi32`, `z_anyextload` 等 TableGen 记录。

### Lines 921-960 / 第 921-960 行
```tablegen
 921: // Extending non-atomic loads in which the extension type doesn't matter.
 922: def anyextload : PatFrag<(ops node:$ptr), (unindexedload node:$ptr), [{
 923:   return cast<LoadSDNode>(N)->getExtensionType() != ISD::NON_EXTLOAD;
 924: }]>;
 925: def anyextloadi8 : PatFrag<(ops node:$ptr), (anyextload node:$ptr), [{
 926:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::i8;
 927: }]>;
 928: def anyextloadi16 : PatFrag<(ops node:$ptr), (anyextload node:$ptr), [{
 929:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::i16;
 930: }]>;
 931: def anyextloadi32 : PatFrag<(ops node:$ptr), (anyextload node:$ptr), [{
 932:   return cast<LoadSDNode>(N)->getMemoryVT() == MVT::i32;
 933: }]>;
 934: 
 935: // Extending (atomic) loads that are not sign/zero extending.
 936: def z_extload : PatFrags<(ops node:$ptr),
 937:                          [(extload node:$ptr),
 938:                           (atomic_load node:$ptr)], [{
 939:   return getLoadExtType(N) == ISD::EXTLOAD;
 940: }]>;
 941: def z_extloadi8 : PatFrag<(ops node:$ptr), (z_extload node:$ptr), [{
 942:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i8;
 943: }]>;
 944: def z_extloadi16 : PatFrag<(ops node:$ptr), (z_extload node:$ptr), [{
 945:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i16;
 946: }]>;
 947: def z_extloadi32 : PatFrag<(ops node:$ptr), (z_extload node:$ptr), [{
 948:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i32;
 949: }]>;
 950: def z_extloadi64 : PatFrag<(ops node:$ptr), (z_extload node:$ptr), [{
 951:   return cast<MemSDNode>(N)->getMemoryVT() == MVT::i64;
 952: }]>;
 953: 
 954: // Extending atomic FP loads.
 955: def z_any_extloadf32 : PatFrags<(ops node:$ptr),
 956:                                 [(any_extloadf32 node:$ptr),
 957:                                  (any_fpextend (f32 (atomic_load node:$ptr)))]>;
 958: def z_any_extloadf64 : PatFrags<(ops node:$ptr),
 959:                                 [(any_extloadf64 node:$ptr),
 960:                                  (any_fpextend (f64 (atomic_load node:$ptr)))]>;
```
- **EN**: This block declares or refines TableGen records such as `anyextload`, `anyextloadi8`, `anyextloadi16`, `anyextloadi32`, `z_extload`, `z_extloadi8`.
- **CN**: 该代码块声明或细化了 `anyextload`, `anyextloadi8`, `anyextloadi16`, `anyextloadi32`, `z_extload`, `z_extloadi8` 等 TableGen 记录。

### Lines 961-1000 / 第 961-1000 行
```tablegen
 961: 
 962: // Aligned loads.
 963: class AlignedLoad<SDPatternOperator load>
 964:   : PatFrag<(ops node:$addr), (load node:$addr),
 965:   [{ return storeLoadIsAligned(N); }]>;
 966: def aligned_z_load         : AlignedLoad<z_load>;
 967: def aligned_z_asextloadi16 : AlignedLoad<z_asextloadi16>;
 968: def aligned_z_asextloadi32 : AlignedLoad<z_asextloadi32>;
 969: def aligned_z_azextloadi16 : AlignedLoad<z_azextloadi16>;
 970: def aligned_z_azextloadi32 : AlignedLoad<z_azextloadi32>;
 971: 
 972: // Aligned stores.
 973: class AlignedStore<SDPatternOperator store>
 974:   : PatFrag<(ops node:$src, node:$addr), (store node:$src, node:$addr),
 975:   [{ return storeLoadIsAligned(N); }]>;
 976: def aligned_store         : AlignedStore<store>;
 977: def aligned_truncstorei16 : AlignedStore<truncstorei16>;
 978: def aligned_truncstorei32 : AlignedStore<truncstorei32>;
 979: 
 980: // Non-volatile loads.  Used for instructions that might access the storage
 981: // location multiple times.
 982: class NonvolatileLoad<SDPatternOperator load>
 983:   : PatFrag<(ops node:$addr), (load node:$addr), [{
 984:   auto *Load = cast<LoadSDNode>(N);
 985:   return !Load->isVolatile();
 986: }]>;
 987: def nonvolatile_anyextloadi8  : NonvolatileLoad<anyextloadi8>;
 988: def nonvolatile_anyextloadi16 : NonvolatileLoad<anyextloadi16>;
 989: def nonvolatile_anyextloadi32 : NonvolatileLoad<anyextloadi32>;
 990: 
 991: // Non-volatile stores.
 992: class NonvolatileStore<SDPatternOperator store>
 993:   : PatFrag<(ops node:$src, node:$addr), (store node:$src, node:$addr), [{
 994:   auto *Store = cast<StoreSDNode>(N);
 995:   return !Store->isVolatile();
 996: }]>;
 997: def nonvolatile_truncstorei8  : NonvolatileStore<truncstorei8>;
 998: def nonvolatile_truncstorei16 : NonvolatileStore<truncstorei16>;
 999: def nonvolatile_truncstorei32 : NonvolatileStore<truncstorei32>;
1000: 
```
- **EN**: This block declares or refines TableGen records such as `AlignedLoad`, `aligned_z_load`, `aligned_z_asextloadi16`, `aligned_z_asextloadi32`, `aligned_z_azextloadi16`, `aligned_z_azextloadi32`.
- **CN**: 该代码块声明或细化了 `AlignedLoad`, `aligned_z_load`, `aligned_z_asextloadi16`, `aligned_z_asextloadi32`, `aligned_z_azextloadi16`, `aligned_z_azextloadi32` 等 TableGen 记录。

### Lines 1001-1040 / 第 1001-1040 行
```tablegen
1001: // A store of a load that can be implemented using MVC.
1002: def mvc_store : PatFrag<(ops node:$value, node:$addr),
1003:                         (unindexedstore node:$value, node:$addr),
1004:                         [{ return storeLoadCanUseMVC(N); }]>;
1005: 
1006: // Binary read-modify-write operations on memory in which the other
1007: // operand is also memory and for which block operations like NC can
1008: // be used.  There are two patterns for each operator, depending on
1009: // which operand contains the "other" load.
1010: multiclass block_op<SDPatternOperator operator> {
1011:   def "1" : PatFrag<(ops node:$value, node:$addr),
1012:                     (unindexedstore (operator node:$value,
1013:                                               (unindexedload node:$addr)),
1014:                                     node:$addr),
1015:                     [{ return storeLoadCanUseBlockBinary(N, 0); }]>;
1016:   def "2" : PatFrag<(ops node:$value, node:$addr),
1017:                     (unindexedstore (operator (unindexedload node:$addr),
1018:                                               node:$value),
1019:                                     node:$addr),
1020:                     [{ return storeLoadCanUseBlockBinary(N, 1); }]>;
1021: }
1022: defm block_and : block_op<and>;
1023: defm block_or  : block_op<or>;
1024: defm block_xor : block_op<xor>;
1025: 
1026: // Insertions.
1027: class insert_imm<int mask> : PatFrag<(ops node:$src1, node:$src2),
1028:                                      (or (and node:$src1, mask), node:$src2)>;
1029: 
1030: def inserti8   : insert_imm<-256>;
1031: def insertll   : insert_imm<0xffff0000>;
1032: def insertlh   : insert_imm<0x0000ffff>;
1033: def insertll64 : insert_imm<0xffffffffffff0000>;
1034: def insertlh64 : insert_imm<0xffffffff0000ffff>;
1035: def inserthl64 : insert_imm<0xffff0000ffffffff>;
1036: def inserthh64 : insert_imm<0x0000ffffffffffff>;
1037: def insertlf   : insert_imm<0xffffffff00000000>;
1038: def inserthf   : insert_imm<0x00000000ffffffff>;
1039: 
1040: // ORs that can be treated as insertions.
```
- **EN**: This block declares or refines TableGen records such as `mvc_store`, `block_op`, `block_and`, `block_or`, `block_xor`, `insert_imm`.
- **CN**: 该代码块声明或细化了 `mvc_store`, `block_op`, `block_and`, `block_or`, `block_xor`, `insert_imm` 等 TableGen 记录。

### Lines 1041-1080 / 第 1041-1080 行
```tablegen
1041: def or_as_inserti8 : PatFrag<(ops node:$src1, node:$src2),
1042:                              (or node:$src1, node:$src2), [{
1043:   unsigned BitWidth = N->getValueType(0).getScalarSizeInBits();
1044:   return CurDAG->MaskedValueIsZero(N->getOperand(0),
1045:                                    APInt::getLowBitsSet(BitWidth, 8));
1046: }]>;
1047: 
1048: // ORs that can be treated as reversed insertions.
1049: def or_as_revinserti8 : PatFrag<(ops node:$src1, node:$src2),
1050:                                 (or node:$src1, node:$src2), [{
1051:   unsigned BitWidth = N->getValueType(0).getScalarSizeInBits();
1052:   return CurDAG->MaskedValueIsZero(N->getOperand(1),
1053:                                    APInt::getLowBitsSet(BitWidth, 8));
1054: }]>;
1055: 
1056: // Negative integer absolute.
1057: def z_inegabs : PatFrag<(ops node:$src), (ineg (abs node:$src))>;
1058: 
1059: // Integer multiply-and-add
1060: class z_muladd<SDPatternOperator mulop>
1061:   : PatFrag<(ops node:$src1, node:$src2, node:$src3),
1062:             (add (mulop node:$src1, node:$src2), node:$src3)>;
1063: 
1064: // Alternatives to match operations with or without an overflow CC result.
1065: def z_sadd : PatFrags<(ops node:$src1, node:$src2),
1066:                       [(z_saddo node:$src1, node:$src2),
1067:                        (add node:$src1, node:$src2)]>;
1068: def z_uadd : PatFrags<(ops node:$src1, node:$src2),
1069:                       [(z_uaddo node:$src1, node:$src2),
1070:                        (add node:$src1, node:$src2)]>;
1071: def z_ssub : PatFrags<(ops node:$src1, node:$src2),
1072:                       [(z_ssubo node:$src1, node:$src2),
1073:                        (sub node:$src1, node:$src2)]>;
1074: def z_usub : PatFrags<(ops node:$src1, node:$src2),
1075:                       [(z_usubo node:$src1, node:$src2),
1076:                        (sub node:$src1, node:$src2)]>;
1077: 
1078: // Combined logical operations.
1079: def andc : PatFrag<(ops node:$src1, node:$src2),
1080:                    (and node:$src1, (not node:$src2))>;
```
- **EN**: This block declares or refines TableGen records such as `or_as_inserti8`, `or_as_revinserti8`, `z_inegabs`, `z_muladd`, `z_sadd`, `z_uadd`.
- **CN**: 该代码块声明或细化了 `or_as_inserti8`, `or_as_revinserti8`, `z_inegabs`, `z_muladd`, `z_sadd`, `z_uadd` 等 TableGen 记录。

### Lines 1081-1120 / 第 1081-1120 行
```tablegen
1081: def orc  : PatFrag<(ops node:$src1, node:$src2),
1082:                    (or node:$src1, (not node:$src2))>;
1083: def nand : PatFrag<(ops node:$src1, node:$src2),
1084:                    (not (and node:$src1, node:$src2))>;
1085: def nor  : PatFrag<(ops node:$src1, node:$src2),
1086:                    (not (or node:$src1, node:$src2))>;
1087: def nxor : PatFrag<(ops node:$src1, node:$src2),
1088:                    (not (xor node:$src1, node:$src2))>;
1089: 
1090: // Fused multiply-subtract, using the natural operand order.
1091: def any_fms : PatFrag<(ops node:$src1, node:$src2, node:$src3),
1092:                       (any_fma node:$src1, node:$src2, (fneg node:$src3))>;
1093: 
1094: // Fused multiply-add and multiply-subtract, but with the order of the
1095: // operands matching SystemZ's MA and MS instructions.
1096: def z_any_fma : PatFrag<(ops node:$src1, node:$src2, node:$src3),
1097:                         (any_fma node:$src2, node:$src3, node:$src1)>;
1098: def z_any_fms : PatFrag<(ops node:$src1, node:$src2, node:$src3),
1099:                         (any_fma node:$src2, node:$src3, (fneg node:$src1))>;
1100: 
1101: // Negative fused multiply-add and multiply-subtract.
1102: def any_fnma : PatFrag<(ops node:$src1, node:$src2, node:$src3),
1103:                        (fneg (any_fma node:$src1, node:$src2, node:$src3))>;
1104: def any_fnms : PatFrag<(ops node:$src1, node:$src2, node:$src3),
1105:                        (fneg (any_fms node:$src1, node:$src2, node:$src3))>;
1106: 
1107: // Floating-point negative absolute.
1108: def fnabs : PatFrag<(ops node:$ptr), (fneg (fabs node:$ptr))>;
1109: 
1110: // Floating-point operations which will not participate in reassociation, and
1111: // therefore are candidates for reg/mem folding during isel.
1112: def z_any_fadd_noreassoc : PatFrag<(ops node:$src1, node:$src2),
1113:                                    (any_fadd node:$src1, node:$src2),
1114:                                    [{ return !shouldSelectForReassoc(N); }]>;
1115: def z_any_fsub_noreassoc : PatFrag<(ops node:$src1, node:$src2),
1116:                                    (any_fsub node:$src1, node:$src2),
1117:                                    [{ return !shouldSelectForReassoc(N); }]>;
1118: def z_any_fmul_noreassoc : PatFrag<(ops node:$src1, node:$src2),
1119:                                    (any_fmul node:$src1, node:$src2),
1120:                                    [{ return !shouldSelectForReassoc(N); }]>;
```
- **EN**: This block declares or refines TableGen records such as `orc`, `nand`, `nor`, `nxor`, `any_fms`, `z_any_fma`.
- **CN**: 该代码块声明或细化了 `orc`, `nand`, `nor`, `nxor`, `any_fms`, `z_any_fma` 等 TableGen 记录。

### Lines 1121-1160 / 第 1121-1160 行
```tablegen
1121: 
1122: // Strict floating-point fragments.
1123: def z_any_fcmp    : PatFrags<(ops node:$lhs, node:$rhs),
1124:                              [(z_strict_fcmp node:$lhs, node:$rhs),
1125:                               (z_fcmp node:$lhs, node:$rhs)]>;
1126: def z_any_vfcmpe  : PatFrags<(ops node:$lhs, node:$rhs),
1127:                              [(z_strict_vfcmpe node:$lhs, node:$rhs),
1128:                               (z_vfcmpe node:$lhs, node:$rhs)]>;
1129: def z_any_vfcmph  : PatFrags<(ops node:$lhs, node:$rhs),
1130:                              [(z_strict_vfcmph node:$lhs, node:$rhs),
1131:                               (z_vfcmph node:$lhs, node:$rhs)]>;
1132: def z_any_vfcmphe : PatFrags<(ops node:$lhs, node:$rhs),
1133:                              [(z_strict_vfcmphe node:$lhs, node:$rhs),
1134:                               (z_vfcmphe node:$lhs, node:$rhs)]>;
1135: def z_any_vextend : PatFrags<(ops node:$src),
1136:                              [(z_strict_vextend node:$src),
1137:                               (z_vextend node:$src)]>;
1138: def z_any_vround  : PatFrags<(ops node:$src),
1139:                              [(z_strict_vround node:$src),
1140:                               (z_vround node:$src)]>;
1141: 
1142: // Create a unary operator that loads from memory and then performs
1143: // the given operation on it.
1144: class loadu<SDPatternOperator operator, SDPatternOperator load = z_load>
1145:   : PatFrag<(ops node:$addr), (operator (load node:$addr))>;
1146: 
1147: // Create a store operator that performs the given unary operation
1148: // on the value before storing it.
1149: class storeu<SDPatternOperator operator, SDPatternOperator store = store>
1150:   : PatFrag<(ops node:$value, node:$addr),
1151:             (store (operator node:$value), node:$addr)>;
1152: 
1153: // Create a store operator that performs the given inherent operation
1154: // and stores the resulting value.
1155: class storei<SDPatternOperator operator, SDPatternOperator store = store>
1156:   : PatFrag<(ops node:$addr),
1157:             (store (operator), node:$addr)>;
1158: 
1159: // Create a shift operator that optionally ignores an AND of the
1160: // shift count with an immediate if the bottom 6 bits are all set.
```
- **EN**: This block declares or refines TableGen records such as `z_any_fcmp`, `z_any_vfcmpe`, `z_any_vfcmph`, `z_any_vfcmphe`, `z_any_vextend`, `z_any_vround`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `z_any_fcmp`, `z_any_vfcmpe`, `z_any_vfcmph`, `z_any_vfcmphe`, `z_any_vextend`, `z_any_vround` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1161-1200 / 第 1161-1200 行
```tablegen
1161: def imm32bottom6set : IntImmLeaf<i32, [{
1162:   return (Imm.getZExtValue() & 0x3f) == 0x3f;
1163: }]>;
1164: class shiftop<SDPatternOperator operator>
1165:   : PatFrags<(ops node:$val, node:$count),
1166:              [(operator node:$val, node:$count),
1167:               (operator node:$val, (and node:$count, imm32bottom6set))]>;
1168: 
1169: // Create a shift operator that optionally ignores an AND of the
1170: // shift count with an immediate if the bottom 7 bits are all set.
1171: def imm32bottom7set : IntImmLeaf<i32, [{
1172:   return (Imm.getZExtValue() & 0x7f) == 0x7f;
1173: }]>;
1174: class vshiftop<SDPatternOperator operator>
1175:   : PatFrags<(ops node:$val, node:$count),
1176:              [(operator node:$val, node:$count),
1177:               (operator node:$val, (and node:$count, imm32bottom7set))]>;
1178: 
1179: def imm32mod64  : IntImmLeaf<i32, [{
1180:   return Imm.getZExtValue() % 64 == 0;
1181: }]>;
1182: 
1183: def imm32nobits : IntImmLeaf<i32, [{
1184:   return (Imm.getZExtValue() & 0x07) == 0;
1185: }]>;
1186: def imm32nobytes : IntImmLeaf<i32, [{
1187:   return (Imm.getZExtValue() & 0x78) == 0;
1188: }]>;
1189: 
1190: // Load a scalar and replicate it in all elements of a vector.
1191: class z_replicate_load<ValueType scalartype, SDPatternOperator load>
1192:   : PatFrag<(ops node:$addr),
1193:             (z_replicate (scalartype (load node:$addr)))>;
1194: def z_replicate_loadi8  : z_replicate_load<i32, z_anyextloadi8>;
1195: def z_replicate_loadi16 : z_replicate_load<i32, z_anyextloadi16>;
1196: def z_replicate_loadi32 : z_replicate_load<i32, z_load>;
1197: def z_replicate_loadi64 : z_replicate_load<i64, z_load>;
1198: def z_replicate_loadf16 : z_replicate_load<f16, z_load>;
1199: def z_replicate_loadf32 : z_replicate_load<f32, z_load>;
1200: def z_replicate_loadf64 : z_replicate_load<f64, z_load>;
```
- **EN**: This block declares or refines TableGen records such as `imm32bottom6set`, `shiftop`, `imm32bottom7set`, `vshiftop`, `imm32mod64`, `imm32nobits`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `imm32bottom6set`, `shiftop`, `imm32bottom7set`, `vshiftop`, `imm32mod64`, `imm32nobits` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1201-1240 / 第 1201-1240 行
```tablegen
1201: // Byte-swapped replicated vector element loads.
1202: def z_replicate_loadbswapi16 : z_replicate_load<i32, z_loadbswap16>;
1203: def z_replicate_loadbswapi32 : z_replicate_load<i32, z_loadbswap32>;
1204: def z_replicate_loadbswapi64 : z_replicate_load<i64, z_loadbswap64>;
1205: 
1206: // Load a scalar and insert it into a single element of a vector.
1207: class z_vle<ValueType scalartype, SDPatternOperator load>
1208:   : PatFrag<(ops node:$vec, node:$addr, node:$index),
1209:             (z_vector_insert node:$vec, (scalartype (load node:$addr)),
1210:                              node:$index)>;
1211: def z_vlei8  : z_vle<i32, z_anyextloadi8>;
1212: def z_vlei16 : z_vle<i32, z_anyextloadi16>;
1213: def z_vlei32 : z_vle<i32, z_load>;
1214: def z_vlei64 : z_vle<i64, z_load>;
1215: def z_vlef16 : z_vle<f16, z_load>;
1216: def z_vlef32 : z_vle<f32, z_load>;
1217: def z_vlef64 : z_vle<f64, z_load>;
1218: // Byte-swapped vector element loads.
1219: def z_vlebri16 : z_vle<i32, z_loadbswap16>;
1220: def z_vlebri32 : z_vle<i32, z_loadbswap32>;
1221: def z_vlebri64 : z_vle<i64, z_loadbswap64>;
1222: 
1223: // Load a scalar and insert it into the low element of the high i64 of a
1224: // zeroed vector.
1225: class z_vllez<ValueType scalartype, SDPatternOperator load, int index>
1226:   : PatFrag<(ops node:$addr),
1227:             (z_vector_insert immAllZerosV,
1228:                              (scalartype (load node:$addr)), (i32 index))>;
1229: def z_vllezi8  : z_vllez<i32, z_anyextloadi8, 7>;
1230: def z_vllezi16 : z_vllez<i32, z_anyextloadi16, 3>;
1231: def z_vllezi32 : z_vllez<i32, z_load, 1>;
1232: def z_vllezi64 : PatFrags<(ops node:$addr),
1233:                           [(z_vector_insert immAllZerosV,
1234:                                             (i64 (z_load node:$addr)), (i32 0)),
1235:                            (z_join_dwords (i64 (z_load node:$addr)), (i64 0))]>;
1236: // We use high merges to form a v4f32 from four f32s.  Propagating zero
1237: // into all elements but index 1 gives this expression.
1238: def z_vllezf32 : PatFrag<(ops node:$addr),
1239:                          (z_merge_high
1240:                           (v2i64
```
- **EN**: This block declares or refines TableGen records such as `z_replicate_loadbswapi16`, `z_replicate_loadbswapi32`, `z_replicate_loadbswapi64`, `z_vle`, `z_vlei8`, `z_vlei16`.
- **CN**: 该代码块声明或细化了 `z_replicate_loadbswapi16`, `z_replicate_loadbswapi32`, `z_replicate_loadbswapi64`, `z_vle`, `z_vlei8`, `z_vlei16` 等 TableGen 记录。

### Lines 1241-1280 / 第 1241-1280 行
```tablegen
1241:                            (z_unpackl_high
1242:                             (v4i32
1243:                              (bitconvert
1244:                               (v4f32 (scalar_to_vector
1245:                                       (f32 (z_load node:$addr)))))))),
1246:                           (v2i64
1247:                            (bitconvert (v4f32 immAllZerosV))))>;
1248: def z_vllezf64 : PatFrag<(ops node:$addr),
1249:                          (z_merge_high
1250:                           (v2f64 (scalar_to_vector (f64 (z_load node:$addr)))),
1251:                           immAllZerosV)>;
1252: 
1253: // Similarly for the high element of a zeroed vector.
1254: def z_vllezli32 : z_vllez<i32, z_load, 0>;
1255: def z_vllezlf32 : PatFrag<(ops node:$addr),
1256:                           (z_merge_high
1257:                            (v2i64
1258:                             (bitconvert
1259:                              (z_merge_high
1260:                               (v4f32 (scalar_to_vector
1261:                                       (f32 (z_load node:$addr)))),
1262:                               (v4f32 immAllZerosV)))),
1263:                            (v2i64
1264:                             (bitconvert (v4f32 immAllZerosV))))>;
1265: 
1266: // Byte-swapped variants.
1267: def z_vllebrzi16  : z_vllez<i32, z_loadbswap16, 3>;
1268: def z_vllebrzi32  : z_vllez<i32, z_loadbswap32, 1>;
1269: def z_vllebrzli32 : z_vllez<i32, z_loadbswap32, 0>;
1270: def z_vllebrzi64  : PatFrags<(ops node:$addr),
1271:                              [(z_vector_insert immAllZerosV,
1272:                                                (i64 (z_loadbswap64 node:$addr)),
1273:                                                (i32 0)),
1274:                               (z_join_dwords (i64 (z_loadbswap64 node:$addr)),
1275:                                              (i64 0))]>;
1276: 
1277: 
1278: // Store one element of a vector.
1279: class z_vste<ValueType scalartype, SDPatternOperator store>
1280:   : PatFrag<(ops node:$vec, node:$addr, node:$index),
```
- **EN**: This block declares or refines TableGen records such as `z_vllezf64`, `z_vllezli32`, `z_vllezlf32`, `z_vllebrzi16`, `z_vllebrzi32`, `z_vllebrzli32`.
- **CN**: 该代码块声明或细化了 `z_vllezf64`, `z_vllezli32`, `z_vllezlf32`, `z_vllebrzi16`, `z_vllebrzi32`, `z_vllebrzli32` 等 TableGen 记录。

### Lines 1281-1320 / 第 1281-1320 行
```tablegen
1281:             (store (scalartype (z_vector_extract node:$vec, node:$index)),
1282:                    node:$addr)>;
1283: def z_vstei8  : z_vste<i32, truncstorei8>;
1284: def z_vstei16 : z_vste<i32, truncstorei16>;
1285: def z_vstei32 : z_vste<i32, store>;
1286: def z_vstei64 : z_vste<i64, store>;
1287: def z_vstef16 : z_vste<f16, store>;
1288: def z_vstef32 : z_vste<f32, store>;
1289: def z_vstef64 : z_vste<f64, store>;
1290: // Byte-swapped vector element stores.
1291: def z_vstebri16 : z_vste<i32, z_storebswap16>;
1292: def z_vstebri32 : z_vste<i32, z_storebswap32>;
1293: def z_vstebri64 : z_vste<i64, z_storebswap64>;
1294: 
1295: // Arithmetic negation on vectors.
1296: def z_vneg : PatFrag<(ops node:$x), (sub immAllZerosV, node:$x)>;
1297: 
1298: // Bitwise negation on vectors.
1299: def z_vnot : PatFrag<(ops node:$x), (xor node:$x, immAllOnesV)>;
1300: 
1301: // In-register element-wise zero extension from i1 on vectors.
1302: def vsplat_imm_eq_1 : PatFrag<(ops), (build_vector), [{
1303:   APInt Imm;
1304:   return ISD::isConstantSplatVector(N, Imm) && Imm == 1;
1305: }]>;
1306: def z_vzext1 : PatFrag<(ops node:$x), (and node:$x, vsplat_imm_eq_1)>;
1307: 
1308: // Vector constants for saturating truncation, containing the minimum and
1309: // maximum value for the integer type that is half of the element width.
1310: def ssat_trunc_min_vec: PatFrag<(ops), (build_vector), [{
1311:   APInt Imm;
1312:   EVT EltTy = N->getValueType(0).getVectorElementType();
1313:   unsigned SizeInBits = EltTy.getSizeInBits();
1314:   APInt min = APInt::getSignedMinValue(SizeInBits / 2).sext(SizeInBits);
1315:   return ISD::isConstantSplatVector(N, Imm) && APInt::isSameValue(Imm, min);
1316: }]>;
1317: def ssat_trunc_max_vec: PatFrag<(ops), (build_vector), [{
1318:   APInt Imm;
1319:   EVT EltTy = N->getValueType(0).getVectorElementType();
1320:   unsigned SizeInBits = EltTy.getSizeInBits();
```
- **EN**: This block declares or refines TableGen records such as `z_vstei8`, `z_vstei16`, `z_vstei32`, `z_vstei64`, `z_vstef16`, `z_vstef32`.
- **CN**: 该代码块声明或细化了 `z_vstei8`, `z_vstei16`, `z_vstei32`, `z_vstei64`, `z_vstef16`, `z_vstef32` 等 TableGen 记录。

### Lines 1321-1355 / 第 1321-1355 行
```tablegen
1321:   APInt max = APInt::getSignedMaxValue(SizeInBits / 2).sext(SizeInBits);
1322:   return ISD::isConstantSplatVector(N, Imm) && APInt::isSameValue(Imm, max);
1323: }]>;
1324: 
1325: def usat_trunc_max_vec: PatFrag<(ops), (build_vector), [{
1326:   APInt Imm;
1327:   EVT EltTy = N->getValueType(0).getVectorElementType();
1328:   unsigned SizeInBits = EltTy.getSizeInBits();
1329:   APInt max = APInt::getMaxValue(SizeInBits / 2).zext(SizeInBits);
1330:   return ISD::isConstantSplatVector(N, Imm) && APInt::isSameValue(Imm, max);
1331: }]>;
1332: 
1333: // Signed "integer greater than zero" on vectors.
1334: def z_vicmph_zero : PatFrag<(ops node:$x), (z_vicmph node:$x, immAllZerosV)>;
1335: 
1336: // Signed "integer less than zero" on vectors.
1337: def z_vicmpl_zero : PatFrag<(ops node:$x), (z_vicmph immAllZerosV, node:$x)>;
1338: 
1339: // Sign-extend the i64 elements of a vector.
1340: class z_vse<int shift>
1341:   : PatFrag<(ops node:$src),
1342:             (z_vsra_by_scalar (z_vshl_by_scalar node:$src, shift), shift)>;
1343: def z_vsei8  : z_vse<56>;
1344: def z_vsei16 : z_vse<48>;
1345: def z_vsei32 : z_vse<32>;
1346: 
1347: // ...and again with the extensions being done on individual i64 scalars.
1348: class z_vse_by_parts<SDPatternOperator operator, int index1, int index2>
1349:   : PatFrag<(ops node:$src),
1350:             (z_join_dwords
1351:              (operator (z_vector_extract node:$src, index1)),
1352:              (operator (z_vector_extract node:$src, index2)))>;
1353: def z_vsei8_by_parts  : z_vse_by_parts<sext8dbl, 7, 15>;
1354: def z_vsei16_by_parts : z_vse_by_parts<sext16dbl, 3, 7>;
1355: def z_vsei32_by_parts : z_vse_by_parts<sext32, 1, 3>;
```
- **EN**: This block declares or refines TableGen records such as `usat_trunc_max_vec:`, `z_vicmph_zero`, `z_vicmpl_zero`, `z_vse`, `z_vsei8`, `z_vsei16`.
- **CN**: 该代码块声明或细化了 `usat_trunc_max_vec:`, `z_vicmph_zero`, `z_vicmpl_zero`, `z_vse`, `z_vsei8`, `z_vsei16` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
