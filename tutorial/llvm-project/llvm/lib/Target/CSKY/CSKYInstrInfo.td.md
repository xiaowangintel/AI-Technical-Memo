# CSKYInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYInstrInfo.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file describes the CSKY instructions in TableGen format.
- 目的（中文）: 使用 TableGen 定义目标指令，描述操作码、操作数、匹配模式与调度信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //===-- CSKYInstrInfo.td - Target Description for CSKY -----*- tablegen -*-===//
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
  13: 
  14: //===----------------------------------------------------------------------===//
  15: // CSKY specific DAG Nodes.
  16: //===----------------------------------------------------------------------===//
  17: 
  18: // Target-independent type requirements, but with target-specific formats.
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。

### Lines 19-36

```tablegen
  19: def SDT_CallSeqStart : SDCallSeqStart<[SDTCisVT<0, i32>,
  20:                                        SDTCisVT<1, i32>]>;
  21: 
  22: def SDT_CallSeqEnd   : SDCallSeqEnd<[SDTCisVT<0, i32>,
  23:                                      SDTCisVT<1, i32>]>;
  24: 
  25: def SDT_CSKYCall : SDTypeProfile<0, 2, [SDTCisVT<0, iPTR>, SDTCisVT<1, iPTR>]>;
  26: 
  27: def SDT_CSKYCallReg : SDTypeProfile<0, 1, [SDTCisVT<0, iPTR>]>;
  28: 
  29: def SDT_CSKY_LOADADDR : SDTypeProfile<1, 2, [SDTCisVT<0, i32>,
  30:                         SDTCisVT<1, iPTR>, SDTCisVT<2, iPTR>]>;
  31: 
  32: def callseq_start : SDNode<"ISD::CALLSEQ_START", SDT_CallSeqStart,
  33:                            [SDNPHasChain, SDNPOutGlue]>;
  34: def callseq_end   : SDNode<"ISD::CALLSEQ_END", SDT_CallSeqEnd,
  35:                            [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
  36: 
```

- EN: This range defines declarative TableGen records such as SDT_CallSeqStart, SDT_CallSeqEnd, SDT_CSKYCall, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 SDT_CallSeqStart, SDT_CallSeqEnd, SDT_CSKYCall 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 37-54

```tablegen
  37: def CSKY_RET : SDNode<"CSKYISD::RET", SDTNone,
  38:     [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
  39: 
  40: def CSKY_CALL : SDNode<"CSKYISD::CALL", SDT_CSKYCall,
  41:   [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;
  42: 
  43: def CSKY_CALLReg : SDNode<"CSKYISD::CALLReg", SDT_CSKYCallReg,
  44:   [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;
  45: 
  46: def CSKY_TAIL : SDNode<"CSKYISD::TAIL", SDT_CSKYCall,
  47:   [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;
  48: 
  49: def CSKY_TAILReg : SDNode<"CSKYISD::TAILReg", SDT_CSKYCallReg,
  50:   [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue, SDNPVariadic]>;
  51: 
  52: def CSKY_LOAD_ADDR : SDNode<"CSKYISD::LOAD_ADDR", SDT_CSKY_LOADADDR>;
  53: 
  54: //===----------------------------------------------------------------------===//
```

- EN: This range defines declarative TableGen records such as CSKY_RET, CSKY_CALL, CSKY_CALLReg, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 CSKY_RET, CSKY_CALL, CSKY_CALLReg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 55-72

```tablegen
  55: // Operand and SDNode transformation definitions.
  56: //===----------------------------------------------------------------------===//
  57: class ImmAsmOperand<string prefix, int width, string suffix> : AsmOperandClass {
  58:   let Name = prefix # "Imm" # width # suffix;
  59:   let RenderMethod = "addImmOperands";
  60:   let DiagnosticType = !strconcat("Invalid", Name);
  61: }
  62: 
  63: class SImmAsmOperand<int width, string suffix = "">
  64:     : ImmAsmOperand<"S", width, suffix> {
  65: }
  66: 
  67: class UImmAsmOperand<int width, string suffix = "">
  68:     : ImmAsmOperand<"U", width, suffix> {
  69: }
  70: 
  71: class OImmAsmOperand<int width, string suffix = "">
  72:     : ImmAsmOperand<"O", width, suffix> {
```

- EN: This range defines declarative TableGen records such as ImmAsmOperand, SImmAsmOperand, UImmAsmOperand, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ImmAsmOperand, SImmAsmOperand, UImmAsmOperand 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 73-90

```tablegen
  73: }
  74: 
  75: def to_tframeindex : SDNodeXForm<frameindex, [{
  76:   auto FI = cast<FrameIndexSDNode>(N);
  77:   return CurDAG->getTargetFrameIndex(FI->getIndex(), TLI->getPointerTy(CurDAG->getDataLayout()));
  78: }]>;
  79: 
  80: def to_tconstpool : SDNodeXForm<constpool, [{
  81:   auto CP = cast<ConstantPoolSDNode>(N);
  82:   return CurDAG->getTargetConstantPool(CP->getConstVal(), TLI->getPointerTy(CurDAG->getDataLayout()),
  83:                     CP->getAlign(), CP->getOffset(), CSKYII::MO_None);
  84: }]>;
  85: 
  86: def to_tconstpool_hi16 : SDNodeXForm<constpool, [{
  87:   auto CP = cast<ConstantPoolSDNode>(N);
  88:   return CurDAG->getTargetConstantPool(CP->getConstVal(), TLI->getPointerTy(CurDAG->getDataLayout()),
  89:                     CP->getAlign(), CP->getOffset(), CSKYII::MO_ADDR_HI16);
  90: }]>;
```

- EN: This range defines declarative TableGen records such as to_tframeindex, to_tconstpool, to_tconstpool_hi16, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 to_tframeindex, to_tconstpool, to_tconstpool_hi16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 91-108

```tablegen
  91: 
  92: def to_tconstpool_lo16 : SDNodeXForm<constpool, [{
  93:   auto CP = cast<ConstantPoolSDNode>(N);
  94:   return CurDAG->getTargetConstantPool(CP->getConstVal(), TLI->getPointerTy(CurDAG->getDataLayout()),
  95:                     CP->getAlign(), CP->getOffset(), CSKYII::MO_ADDR_LO16);
  96: }]>;
  97: 
  98: class oimm<int num> : Operand<i32>,
  99:   ImmLeaf<i32, "return isUInt<"#num#">(Imm - 1);"> {
 100:   let EncoderMethod = "getOImmOpValue";
 101:   let ParserMatchClass = OImmAsmOperand<num>;
 102:   let DecoderMethod = "decodeOImmOperand<"#num#">";
 103: }
 104: 
 105: def imm_neg_XFORM : SDNodeXForm<imm, [{
 106:   return CurDAG->getTargetConstant(-N->getSExtValue(), SDLoc(N), MVT::i32);
 107: }]>;
 108: 
```

- EN: This range defines declarative TableGen records such as to_tconstpool_lo16, oimm, imm_neg_XFORM, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 to_tconstpool_lo16, oimm, imm_neg_XFORM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 109-126

```tablegen
 109: class oimm_neg<int num> : Operand<i32>,
 110:   ImmLeaf<i32, "return isUInt<"#num#">(-Imm - 1);"> {
 111: }
 112: 
 113: class uimm<int num, int shift = 0> : Operand<i32>,
 114:   ImmLeaf<i32, "return isShiftedUInt<"#num#", "#shift#">(Imm);"> {
 115:   let EncoderMethod = "getImmOpValue<"#shift#">";
 116:   let ParserMatchClass =
 117:     !if(!ne(shift, 0),
 118:         UImmAsmOperand<num, "Shift"#shift>,
 119:         UImmAsmOperand<num>);
 120:   let DecoderMethod = "decodeUImmOperand<"#num#", "#shift#">";
 121: }
 122: 
 123: class uimm_neg<int num, int shift = 0> : Operand<i32>,
 124:   ImmLeaf<i32, "return isShiftedUInt<"#num#", "#shift#">(-Imm);"> {
 125: }
 126: 
```

- EN: This range defines declarative TableGen records such as oimm_neg, uimm, uimm_neg, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 oimm_neg, uimm, uimm_neg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 127-144

```tablegen
 127: class simm<int num, int shift = 0> : Operand<i32>,
 128:   ImmLeaf<i32, "return isShiftedInt<"#num#", "#shift#">(Imm);"> {
 129:   let EncoderMethod = "getImmOpValue<"#shift#">";
 130:   let ParserMatchClass = SImmAsmOperand<num>;
 131:   let DecoderMethod = "decodeSImmOperand<"#num#", "#shift#">";
 132: }
 133: 
 134: def nimm_XFORM : SDNodeXForm<imm, [{
 135:   return CurDAG->getTargetConstant(~N->getSExtValue(), SDLoc(N), MVT::i32);
 136: }]>;
 137: class nimm<int num> : Operand<i32>,
 138:   ImmLeaf<i32, "return isUInt<"#num#">(~Imm);", nimm_XFORM> {
 139:   let ParserMatchClass = UImmAsmOperand<num>;
 140: }
 141: 
 142: def uimm32_hi16 : SDNodeXForm<imm, [{
 143:   return CurDAG->getTargetConstant((N->getZExtValue() >> 16) & 0xFFFF,
 144:     SDLoc(N), MVT::i32);
```

- EN: This range defines declarative TableGen records such as simm, nimm_XFORM, nimm, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 simm, nimm_XFORM, nimm 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 145-162

```tablegen
 145: }]>;
 146: def uimm32_lo16 : SDNodeXForm<imm, [{
 147:   return CurDAG->getTargetConstant(N->getZExtValue()& 0xFFFF, SDLoc(N), MVT::i32);
 148: }]>;
 149: def uimm16_16_xform : Operand<i32>,
 150:   ImmLeaf<i32, "return isShiftedUInt<16, 16>(Imm);", uimm32_hi16> {
 151:   let ParserMatchClass = UImmAsmOperand<16>;
 152:   let EncoderMethod = "getImmOpValue";
 153: }
 154: 
 155: def uimm_shift : Operand<i32>, ImmLeaf<i32, "return isUInt<2>(Imm);"> {
 156:   let EncoderMethod = "getImmShiftOpValue";
 157:   let ParserMatchClass = UImmAsmOperand<2>;
 158:   let DecoderMethod = "decodeImmShiftOpValue";
 159: }
 160: 
 161: // Optimize (or x, imm) to (BSETI x, log2(imm)). We should exclude the
 162: // case can be opimized to (ORI32/ORI16 x, imm).
```

- EN: This range defines declarative TableGen records such as uimm32_lo16, uimm16_16_xform, uimm_shift, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm32_lo16, uimm16_16_xform, uimm_shift 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 163-180

```tablegen
 163: def imm32_1_pop_bit_XFORM : SDNodeXForm<imm, [{
 164:   uint32_t I = N->getZExtValue();
 165:   return CurDAG->getTargetConstant(llvm::Log2_32(I), SDLoc(N),
 166:                                    N->getValueType(0));
 167: }]>;
 168: def imm32_1_pop_bit : PatLeaf<(imm), [{
 169:   uint32_t I = N->getZExtValue();
 170:   return llvm::popcount(I) == 1 && I > 0xfff;
 171: }]>;
 172: 
 173: // Optimize (and x, imm) to (BCLRI x, log2(~imm)). We should exclude the
 174: // case can be opimized to (ANDNI x, ~imm).
 175: def imm32_1_zero_bit_XFORM : SDNodeXForm<imm, [{
 176:   uint32_t I = ~N->getZExtValue();
 177:   return CurDAG->getTargetConstant(llvm::Log2_32(I), SDLoc(N),
 178:                                    N->getValueType(0));
 179: }]>;
 180: def imm32_1_zero_bit : PatLeaf<(imm), [{
```

- EN: This range defines declarative TableGen records such as imm32_1_pop_bit_XFORM, imm32_1_pop_bit, imm32_1_zero_bit_XFORM, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 imm32_1_pop_bit_XFORM, imm32_1_pop_bit, imm32_1_zero_bit_XFORM 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 181-198

```tablegen
 181:   uint32_t I = ~N->getZExtValue();
 182:   return llvm::popcount(I) == 1 && I > 0xfff;
 183: }]>;
 184: 
 185: def CSKYSymbol : AsmOperandClass {
 186:   let Name = "CSKYSymbol";
 187:   let RenderMethod = "addImmOperands";
 188:   let DiagnosticType = "InvalidCSKYSymbol";
 189:   let ParserMethod = "parseCSKYSymbol";
 190: }
 191: 
 192: def br_symbol : Operand<OtherVT> {
 193:   let EncoderMethod =
 194:     "getBranchSymbolOpValue<CSKY::fixup_csky_pcrel_imm16_scale2>";
 195:   let ParserMatchClass = CSKYSymbol;
 196:   let DecoderMethod = "decodeSImmOperand<16, 1>";
 197:   let PrintMethod = "printCSKYSymbolOperand";
 198:   let OperandType = "OPERAND_PCREL";
```

- EN: This range defines declarative TableGen records such as CSKYSymbol, br_symbol, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CSKYSymbol, br_symbol 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 199-216

```tablegen
 199: }
 200: 
 201: def call_symbol : Operand<iPTR> {
 202:   let ParserMatchClass = CSKYSymbol;
 203:   let EncoderMethod = "getCallSymbolOpValue";
 204:   let DecoderMethod = "decodeSImmOperand<26, 1>";
 205:   let PrintMethod = "printCSKYSymbolOperand";
 206:   let OperandType = "OPERAND_PCREL";
 207: }
 208: 
 209: def Constpool : AsmOperandClass {
 210:   let Name = "Constpool";
 211:   let RenderMethod = "addConstpoolOperands";
 212:   let DiagnosticType = "InvalidConstpool";
 213:   let ParserMethod = "parseConstpoolSymbol";
 214: }
 215: 
 216: def constpool_symbol : Operand<iPTR> {
```

- EN: This range defines declarative TableGen records such as call_symbol, Constpool, constpool_symbol, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 call_symbol, Constpool, constpool_symbol 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 217-234

```tablegen
 217:   let ParserMatchClass = Constpool;
 218:   let EncoderMethod =
 219:     "getConstpoolSymbolOpValue<CSKY::fixup_csky_pcrel_uimm16_scale4>";
 220:   let DecoderMethod = "decodeUImmOperand<16, 2>";
 221:   let PrintMethod = "printConstpool";
 222:   let OperandType = "OPERAND_PCREL";
 223: }
 224: 
 225: def DataAsmClass : AsmOperandClass {
 226:   let Name = "DataSymbol";
 227:   let RenderMethod = "addConstpoolOperands";
 228:   let DiagnosticType = "InvalidConstpool";
 229:   let ParserMethod = "parseDataSymbol";
 230: }
 231: 
 232: class data_symbol<string reloc, int shift> : Operand<iPTR> {
 233:   let ParserMatchClass = Constpool;
 234:   let EncoderMethod =
```

- EN: This range defines declarative TableGen records such as DataAsmClass, data_symbol, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 DataAsmClass, data_symbol 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 235-252

```tablegen
 235:     "getDataSymbolOpValue<"#reloc#">";
 236:   let DecoderMethod = "decodeUImmOperand<18, "#shift#">";
 237:   let PrintMethod = "printDataSymbol";
 238: }
 239: 
 240: def bare_symbol : Operand<iPTR> {
 241:   let ParserMatchClass = CSKYSymbol;
 242:   let EncoderMethod = "getBareSymbolOpValue";
 243:   let PrintMethod = "printCSKYSymbolOperand";
 244:   let DecoderMethod = "decodeSImmOperand<18, 1>";
 245:   let OperandType = "OPERAND_PCREL";
 246: }
 247: 
 248: def oimm3 : oimm<3> {
 249:   let MCOperandPredicate = [{
 250:     int64_t Imm;
 251:     if (MCOp.evaluateAsConstantImm(Imm))
 252:       return isUInt<3>(Imm - 1);
```

- EN: This range defines declarative TableGen records such as bare_symbol, oimm3, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 bare_symbol, oimm3 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 253-270

```tablegen
 253:     return MCOp.isBareSymbolRef();
 254:   }];
 255: }
 256: def oimm4 : oimm<4>;
 257: def oimm5 : oimm<5> {
 258:   let MCOperandPredicate = [{
 259:     int64_t Imm;
 260:     if (MCOp.evaluateAsConstantImm(Imm))
 261:       return isUInt<5>(Imm - 1);
 262:     return MCOp.isBareSymbolRef();
 263:   }];
 264: }
 265: def oimm6 : oimm<6>;
 266: 
 267: def imm5_idly : Operand<i32>, ImmLeaf<i32,
 268:   "return Imm <= 32 && Imm >= 0;"> {
 269:   let EncoderMethod = "getImmOpValueIDLY";
 270:   let DecoderMethod = "decodeOImmOperand<5>";
```

- EN: This range defines declarative TableGen records such as oimm4, oimm5, oimm6, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 oimm4, oimm5, oimm6 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 271-288

```tablegen
 271: }
 272: 
 273: def oimm8 : oimm<8> {
 274:   let MCOperandPredicate = [{
 275:     int64_t Imm;
 276:     if (MCOp.evaluateAsConstantImm(Imm))
 277:       return isUInt<8>(Imm - 1);
 278:     return MCOp.isBareSymbolRef();
 279:   }];
 280: }
 281: def oimm12 : oimm<12> {
 282:   let MCOperandPredicate = [{
 283:     int64_t Imm;
 284:     if (MCOp.evaluateAsConstantImm(Imm))
 285:       return isUInt<12>(Imm - 1);
 286:     return MCOp.isBareSymbolRef();
 287:   }];
 288: }
```

- EN: This range defines declarative TableGen records such as oimm8, oimm12, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 oimm8, oimm12 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 289-306

```tablegen
 289: def oimm16 : oimm<16> {
 290:   let MCOperandPredicate = [{
 291:     int64_t Imm;
 292:     if (MCOp.evaluateAsConstantImm(Imm))
 293:       return isUInt<16>(Imm - 1);
 294:     return MCOp.isBareSymbolRef();
 295:   }];
 296: }
 297: 
 298: def oimm8_neg : oimm_neg<8> {
 299:   let MCOperandPredicate = [{
 300:     int64_t Imm;
 301:     if (MCOp.evaluateAsConstantImm(Imm))
 302:       return isUInt<8>(-Imm - 1);
 303:     return MCOp.isBareSymbolRef();
 304:   }];
 305: }
 306: def oimm12_neg : oimm_neg<12> {
```

- EN: This range defines declarative TableGen records such as oimm16, oimm8_neg, oimm12_neg, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 oimm16, oimm8_neg, oimm12_neg 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 307-324

```tablegen
 307:   let MCOperandPredicate = [{
 308:     int64_t Imm;
 309:     if (MCOp.evaluateAsConstantImm(Imm))
 310:       return isUInt<12>(-Imm - 1);
 311:     return MCOp.isBareSymbolRef();
 312:   }];
 313: }
 314: 
 315: def nimm12 : nimm<12>;
 316: 
 317: def uimm1 : uimm<1>;
 318: def uimm2 : uimm<2>;
 319: 
 320: 
 321: def uimm2_jmpix : Operand<i32>,
 322:   ImmLeaf<i32, "return Imm == 16 || Imm == 24 || Imm == 32 || Imm == 40;"> {
 323:   let EncoderMethod = "getImmJMPIX";
 324:   let DecoderMethod = "decodeJMPIXImmOperand";
```

- EN: This range defines declarative TableGen records such as nimm12, uimm1, uimm2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 nimm12, uimm1, uimm2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 325-342

```tablegen
 325: }
 326: 
 327: def uimm3 : uimm<3>;
 328: def uimm4 : uimm<4>;
 329: def uimm5 : uimm<5> {
 330:   let MCOperandPredicate = [{
 331:     int64_t Imm;
 332:     if (MCOp.evaluateAsConstantImm(Imm))
 333:       return isShiftedUInt<5, 0>(Imm);
 334:     return MCOp.isBareSymbolRef();
 335:   }];
 336: }
 337: def uimm5_msb_size : uimm<5> {
 338:   let EncoderMethod = "getImmOpValueMSBSize";
 339: }
 340: 
 341: def uimm5_1 : uimm<5, 1> {
 342:   let MCOperandPredicate = [{
```

- EN: This range defines declarative TableGen records such as uimm3, uimm4, uimm5, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm3, uimm4, uimm5 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 343-360

```tablegen
 343:     int64_t Imm;
 344:     if (MCOp.evaluateAsConstantImm(Imm))
 345:       return isShiftedUInt<5, 1>(Imm);
 346:     return MCOp.isBareSymbolRef();
 347:   }];
 348: }
 349: def uimm5_2 : uimm<5, 2> {
 350:   let MCOperandPredicate = [{
 351:     int64_t Imm;
 352:     if (MCOp.evaluateAsConstantImm(Imm))
 353:       return isShiftedUInt<5, 2>(Imm);
 354:     return MCOp.isBareSymbolRef();
 355:   }];
 356: }
 357: def uimm6 : uimm<6>;
 358: def uimm7 : uimm<7>;
 359: def uimm7_1 : uimm<7, 1>;
 360: def uimm7_2 : uimm<7, 2>{
```

- EN: This range defines declarative TableGen records such as uimm5_2, uimm6, uimm7, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm5_2, uimm6, uimm7 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 361-378

```tablegen
 361:   let MCOperandPredicate = [{
 362:     int64_t Imm;
 363:     if (MCOp.evaluateAsConstantImm(Imm))
 364:       return isShiftedUInt<7, 2>(Imm);
 365:     return MCOp.isBareSymbolRef();
 366:   }];
 367: }
 368: def uimm7_3 : uimm<7, 3>;
 369: def uimm8 : uimm<8> {
 370:   let MCOperandPredicate = [{
 371:     int64_t Imm;
 372:     if (MCOp.evaluateAsConstantImm(Imm))
 373:       return isShiftedUInt<8, 0>(Imm);
 374:     return MCOp.isBareSymbolRef();
 375:   }];
 376: }
 377: def uimm8_2 : uimm<8, 2> {
 378:   let MCOperandPredicate = [{
```

- EN: This range defines declarative TableGen records such as uimm7_3, uimm8, uimm8_2, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm7_3, uimm8, uimm8_2 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 379-396

```tablegen
 379:     int64_t Imm;
 380:     if (MCOp.evaluateAsConstantImm(Imm))
 381:       return isShiftedUInt<8, 2>(Imm);
 382:     return MCOp.isBareSymbolRef();
 383:   }];
 384: }
 385: def uimm8_3 : uimm<8, 3>;
 386: def uimm8_8 : uimm<8, 8>;
 387: def uimm8_16 : uimm<8, 16>;
 388: def uimm8_24 : uimm<8, 24>;
 389: def uimm12 : uimm<12>  {
 390:   let MCOperandPredicate = [{
 391:     int64_t Imm;
 392:     if (MCOp.evaluateAsConstantImm(Imm))
 393:       return isShiftedUInt<12, 0>(Imm);
 394:     return MCOp.isBareSymbolRef();
 395:   }];
 396: }
```

- EN: This range defines declarative TableGen records such as uimm8_3, uimm8_8, uimm8_16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm8_3, uimm8_8, uimm8_16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 397-414

```tablegen
 397: def uimm12_1 : uimm<12, 1> {
 398:   let MCOperandPredicate = [{
 399:     int64_t Imm;
 400:     if (MCOp.evaluateAsConstantImm(Imm))
 401:       return isShiftedUInt<12, 1>(Imm);
 402:     return MCOp.isBareSymbolRef();
 403:   }];
 404: }
 405: def uimm12_2 : uimm<12, 2> {
 406:   let MCOperandPredicate = [{
 407:     int64_t Imm;
 408:     if (MCOp.evaluateAsConstantImm(Imm))
 409:       return isShiftedUInt<12, 2>(Imm);
 410:     return MCOp.isBareSymbolRef();
 411:   }];
 412: }
 413: def uimm16 : uimm<16> {
 414:   let MCOperandPredicate = [{
```

- EN: This range defines declarative TableGen records such as uimm12_1, uimm12_2, uimm16, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm12_1, uimm12_2, uimm16 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 415-432

```tablegen
 415:     int64_t Imm;
 416:     if (MCOp.evaluateAsConstantImm(Imm))
 417:       return isShiftedUInt<16, 0>(Imm);
 418:     return MCOp.isBareSymbolRef();
 419:   }];
 420: }
 421: def uimm16_8 : uimm<16, 8>;
 422: def uimm16_16 : uimm<16, 16>;
 423: def uimm20 : uimm<20>;
 424: def uimm24 : uimm<24>;
 425: def uimm24_8 : uimm<24, 8>;
 426: 
 427: def uimm5_neg : uimm_neg<5>;
 428: 
 429: def simm8_2 : simm<8, 2>;
 430: 
 431: class RegSeqAsmOperand<string Suffix = ""> : AsmOperandClass {
 432:   let Name = "RegSeq"#Suffix;
```

- EN: This range defines declarative TableGen records such as uimm16_8, uimm16_16, uimm20, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 uimm16_8, uimm16_16, uimm20 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 433-450

```tablegen
 433:   let RenderMethod = "addRegSeqOperands";
 434:   let DiagnosticType = "InvalidRegSeq";
 435:   let ParserMethod = "parseRegSeq";
 436: }
 437: 
 438: def regseq : Operand<iPTR> {
 439:   let EncoderMethod = "getRegisterSeqOpValue";
 440:   let ParserMatchClass = RegSeqAsmOperand<"">;
 441:   let PrintMethod = "printRegisterSeq";
 442:   let DecoderMethod = "DecodeRegSeqOperand";
 443:   let MIOperandInfo = (ops GPR, uimm5);
 444: }
 445: 
 446: def RegListAsmOperand : AsmOperandClass {
 447:   let Name = "RegList";
 448:   let RenderMethod = "addRegListOperands";
 449:   let DiagnosticType = "InvalidRegList";
 450:   let ParserMethod = "parseRegList";
```

- EN: This range defines declarative TableGen records such as regseq, RegListAsmOperand, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 regseq, RegListAsmOperand 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 451-468

```tablegen
 451: }
 452: 
 453: def reglist : Operand<iPTR> {
 454:   let ParserMatchClass = RegListAsmOperand;
 455:   let PrintMethod = "printRegisterList";
 456: }
 457: 
 458: def PSRFlag : AsmOperandClass {
 459:   let Name = "PSRFlag";
 460:   let RenderMethod = "addImmOperands";
 461:   let DiagnosticType = "InvalidPSRFlag";
 462:   let ParserMethod = "parsePSRFlag";
 463: }
 464: 
 465: def psrflag : Operand<i32>, ImmLeaf<i32, "return isShiftedUInt<5, 0>(Imm);"> {
 466:   let EncoderMethod = "getImmOpValue";
 467:   let ParserMatchClass = PSRFlag;
 468:   let PrintMethod = "printPSRFlag";
```

- EN: This range defines declarative TableGen records such as reglist, PSRFlag, psrflag, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 reglist, PSRFlag, psrflag 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 469-486

```tablegen
 469: }
 470: 
 471: multiclass uimm8SRLXForm<SDNode opc> {
 472:   def _0: SDNodeXForm<opc,
 473:     [{return CurDAG->getTargetConstant((N->getZExtValue() >> 0) & 0xFF, SDLoc(N), MVT::i32);}]>;
 474:   def _8: SDNodeXForm<opc,
 475:     [{return CurDAG->getTargetConstant((N->getZExtValue() >> 8) & 0xFF, SDLoc(N), MVT::i32);}]>;
 476:   def _16: SDNodeXForm<opc,
 477:     [{return CurDAG->getTargetConstant((N->getZExtValue() >> 16) & 0xFF, SDLoc(N), MVT::i32);}]>;
 478:   def _24: SDNodeXForm<opc,
 479:     [{return CurDAG->getTargetConstant((N->getZExtValue() >> 24) & 0xFF, SDLoc(N), MVT::i32);}]>;
 480: }
 481: 
 482: defm uimm8SRL : uimm8SRLXForm<imm>;
 483: 
 484: //===----------------------------------------------------------------------===//
 485: // Instruction Formats
 486: //===----------------------------------------------------------------------===//
```

- EN: This range defines declarative TableGen records such as uimm8SRLXForm, _0, _8, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 uimm8SRLXForm, _0, _8 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 487-504

```tablegen
 487: 
 488: include "CSKYInstrFormats.td"
 489: 
 490: //===----------------------------------------------------------------------===//
 491: // Instruction definitions.
 492: //===----------------------------------------------------------------------===//
 493: 
 494: class TriOpFrag<dag res> : PatFrag<(ops node: $LHS, node:$MHS, node:$RHS), res>;
 495: class BinOpFrag<dag res> : PatFrag<(ops node:$LHS, node:$RHS), res>;
 496: class UnOpFrag<dag res> : PatFrag<(ops node:$Src), res>;
 497: 
 498: def eqToAdd : PatFrag<(ops node:$lhs, node:$rhs), (or node:$lhs, node:$rhs), [{
 499:   return isOrEquivalentToAdd(N);
 500: }]>;
 501: 
 502: def BaseAddr : ComplexPattern<iPTR, 1, "SelectBaseAddr">;
 503: 
 504: 
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as TriOpFrag, BinOpFrag, UnOpFrag, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 TriOpFrag, BinOpFrag, UnOpFrag 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 505-522

```tablegen
 505: //===----------------------------------------------------------------------===//
 506: // CSKYPseudo
 507: //===----------------------------------------------------------------------===//
 508: 
 509: // Pessimistically assume the stack pointer will be clobbered
 510: let Defs = [R14], Uses = [R14] in {
 511: def ADJCALLSTACKDOWN : CSKYPseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
 512:   "!ADJCALLSTACKDOWN $amt1, $amt2", [(callseq_start timm:$amt1, timm:$amt2)]>;
 513: def ADJCALLSTACKUP   : CSKYPseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
 514:   "!ADJCALLSTACKUP $amt1, $amt2", [(callseq_end timm:$amt1, timm:$amt2)]>;
 515: } // Defs = [R14], Uses = [R14]
 516: 
 517: 
 518: //===----------------------------------------------------------------------===//
 519: // Basic ALU instructions.
 520: //===----------------------------------------------------------------------===//
 521: 
 522: let Predicates = [iHasE2] in {
```

- EN: This range defines declarative TableGen records such as ADJCALLSTACKDOWN, ADJCALLSTACKUP, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADJCALLSTACKDOWN, ADJCALLSTACKUP 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 523-540

```tablegen
 523:   let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
 524:   let isAdd = 1 in
 525:   def ADDI32 : I_12<0x0, "addi32", add, oimm12>;
 526:   def SUBI32 : I_12<0x1, "subi32", sub, oimm12>;
 527:   def ORI32 : I_16_ZX<"ori32", uimm16,
 528:   [(set GPR:$rz, (or GPR:$rx, uimm16:$imm16))]>;
 529:   def XORI32 : I_12<0x4, "xori32", xor, uimm12>;
 530:   def ANDI32 : I_12<0x2, "andi32", and, uimm12>;
 531:   def ANDNI32 : I_12<0x3, "andni32", and, nimm12>;
 532:   def LSLI32 : I_5_XZ<0x12, 0x1, "lsli32",
 533:     (outs GPR:$rz), (ins GPR:$rx, uimm5:$imm5),
 534:     [(set GPR:$rz, (shl GPR:$rx, uimm5:$imm5))]>;
 535:   def LSRI32 : I_5_XZ<0x12, 0x2, "lsri32",
 536:     (outs GPR:$rz), (ins GPR:$rx, uimm5:$imm5),
 537:     [(set GPR:$rz, (srl GPR:$rx, uimm5:$imm5))]>;
 538:   def ASRI32 : I_5_XZ<0x12, 0x4, "asri32",
 539:     (outs GPR:$rz), (ins GPR:$rx, uimm5:$imm5),
 540:     [(set GPR:$rz, (sra GPR:$rx, uimm5:$imm5))]>;
```

- EN: This range defines declarative TableGen records such as ADDI32, SUBI32, ORI32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADDI32, SUBI32, ORI32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 541-558

```tablegen
 541:   def ROTLI32 : I_5_XZ<0x12, 0x8, "rotli32",
 542:     (outs GPR:$rz), (ins GPR:$rx, uimm5:$imm5),
 543:     [(set GPR:$rz, (rotl GPR:$rx, uimm5:$imm5))]>;
 544: 
 545:   def ROTRI32 : CSKYPseudo<(outs GPR:$rz), (ins GPR:$rx, oimm5:$imm5),
 546:                             "rotri32 $rz, $rx, $imm5", []>;
 547:   }
 548:   let isAdd = 1 in
 549:   def ADDU32 : R_YXZ_SP_F1<0x0, 0x1,
 550:     BinOpFrag<(add node:$LHS, node:$RHS)>, "addu32", 1>;
 551:   def SUBU32 : R_YXZ_SP_F1<0x0, 0x4,
 552:     BinOpFrag<(sub node:$LHS, node:$RHS)>, "subu32">;
 553: 
 554:   def MULT32 : R_YXZ_SP_F1<0x21, 0x1,
 555:     BinOpFrag<(mul node:$LHS, node:$RHS)>, "mult32", 1>;
 556:   def AND32 : R_YXZ_SP_F1<0x8, 0x1,
 557:     BinOpFrag<(and node:$LHS, node:$RHS)>, "and32", 1>;
 558:   def ANDN32 : R_YXZ_SP_F1<0x8, 0x2,
```

- EN: This range defines declarative TableGen records such as ROTLI32, ROTRI32, ADDU32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ROTLI32, ROTRI32, ADDU32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 559-576

```tablegen
 559:     BinOpFrag<(and node:$LHS, (not node:$RHS))>, "andn32">;
 560:   def OR32: R_YXZ_SP_F1<0x9, 0x1,
 561:     BinOpFrag<(or node:$LHS, node:$RHS)>, "or32", 1>;
 562:   def XOR32 : R_YXZ_SP_F1<0x9, 0x2,
 563:     BinOpFrag<(xor node:$LHS, node:$RHS)>, "xor32", 1>;
 564:   def NOR32 : R_YXZ_SP_F1<0x9, 0x4,
 565:     BinOpFrag<(not (or node:$LHS, node:$RHS))>, "nor32", 1>;
 566:   let isCodeGenOnly = 1 in
 567:   def NOT32 : R_XXZ<0b001001, 0b00100, (outs GPR:$rz), (ins GPR:$rx),
 568:     "not32", [(set GPR:$rz, (not GPR:$rx))]>;
 569: 
 570:   let Size = 8 in
 571:   def NEG32 : CSKYPseudo<(outs GPR:$rd), (ins GPR:$rx), "neg32 $rd, $rx", []>;
 572: 
 573:   let Size = 8 in
 574:   def RSUBI32 : CSKYPseudo<(outs GPR:$rd), (ins GPR:$rx, uimm12:$imm12), "rsubi32 $rd, $rx, $imm12", []>;
 575: 
 576:   def : Pat<(add GPR:$rs1, (oimm12_neg:$im)),
```

- EN: This range defines declarative TableGen records such as OR32, XOR32, NOR32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 OR32, XOR32, NOR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 577-594

```tablegen
 577:             (SUBI32 GPR:$rs1, (imm_neg_XFORM oimm12_neg:$im))>;
 578: 
 579:   def LSL32 : R_YXZ_SP_F1<0x10, 0x1,
 580:     BinOpFrag<(shl node:$LHS, node:$RHS)>, "lsl32">;
 581:   def LSR32 : R_YXZ_SP_F1<0x10, 0x2,
 582:     BinOpFrag<(srl node:$LHS, node:$RHS)>, "lsr32">;
 583:   def ASR32 : R_YXZ_SP_F1<0x10, 0x4,
 584:     BinOpFrag<(sra node:$LHS, node:$RHS)>, "asr32">;
 585:   def ROTL32 : R_YXZ_SP_F1<0x10, 0x8,
 586:     BinOpFrag<(rotl node:$LHS, (and node:$RHS, 0x1f))>, "rotl32">;
 587: 
 588:   def BMASKI32 : I_5_Z<0b010100, 0x1, "bmaski32", oimm5, []>;
 589:   def LSLC32 : I_5_XZ<0x13, 0x1, "lslc32", (outs GPR:$rz, CARRY:$cout),
 590:                       (ins GPR:$rx, oimm5:$imm5), []> {
 591:     bits<0> cout;
 592:   }
 593:   def LSRC32 : I_5_XZ<0x13, 0x2, "lsrc32", (outs GPR:$rz, CARRY:$cout),
 594:                       (ins GPR:$rx, oimm5:$imm5), []> {
```

- EN: This range defines declarative TableGen records such as LSL32, LSR32, ASR32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 LSL32, LSR32, ASR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 595-612

```tablegen
 595:     bits<0> cout;
 596:   }
 597:   def ASRC32 : I_5_XZ<0x13, 0x4, "asrc32", (outs GPR:$rz, CARRY:$cout),
 598:                       (ins GPR:$rx, oimm5:$imm5), []> {
 599:     bits<0> cout;
 600:   }
 601:   def XSR32 : I_5_XZ<0x13, 0x8, "xsr32", (outs GPR:$rz, CARRY:$cout),
 602:                      (ins GPR:$rx, oimm5:$imm5, CARRY:$cin), []> {
 603:     bits<0> cout;
 604:     bits<0> cin;
 605:   }
 606: 
 607:   def IXH32 : R_YXZ_SP_F1<0x2, 0x1,
 608:     BinOpFrag<(add node:$LHS, (shl node:$RHS, (i32 1)))>, "ixh32">;
 609:   def IXW32 : R_YXZ_SP_F1<0x2, 0x2,
 610:     BinOpFrag<(add node:$LHS, (shl node:$RHS, (i32 2)))>, "ixw32">;
 611:   let Predicates = [iHas2E3] in
 612:   def IXD32 : R_YXZ_SP_F1<0x2, 0x4,
```

- EN: This range defines declarative TableGen records such as ASRC32, XSR32, IXH32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ASRC32, XSR32, IXH32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 613-630

```tablegen
 613:     BinOpFrag<(add node:$LHS, (shl node:$RHS, (i32 3)))>, "ixd32">;
 614: 
 615:   let isCommutable = 1, isAdd = 1 in
 616:   def ADDC32 : R_YXZ<0x31, 0x0, 0x2, (outs GPR:$rz, CARRY:$cout),
 617:                      (ins GPR:$rx, GPR:$ry, CARRY:$cin), "addc32", []> {
 618:     bits<0> cout;
 619:     bits<0> cin;
 620:   }
 621:   def SUBC32 : R_YXZ<0x31, 0x0, 0x8, (outs GPR:$rz, CARRY:$cout),
 622:                      (ins GPR:$rx, GPR:$ry, CARRY:$cin), "subc32", []> {
 623:     bits<0> cout;
 624:     bits<0> cin;
 625:   }
 626: 
 627:   def INCF32 : I_5_ZX<0x3, 0x1, "incf32", uimm5, []>;
 628:   def INCT32 : I_5_ZX<0x3, 0x2, "inct32", uimm5, []>;
 629:   def DECF32 : I_5_ZX<0x3, 0x4, "decf32", uimm5, []>;
 630:   def DECT32 : I_5_ZX<0x3, 0x8, "dect32", uimm5, []>;
```

- EN: This range defines declarative TableGen records such as ADDC32, SUBC32, INCF32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ADDC32, SUBC32, INCF32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 631-648

```tablegen
 631: }
 632: 
 633: let Predicates = [iHas2E3] in {
 634:   def DIVS32 : R_YXZ_SP_F1<0x20, 0x2,
 635:     BinOpFrag<(sdiv node:$LHS, node:$RHS)>, "divs32">;
 636:   def DIVU32 : R_YXZ_SP_F1<0x20, 0x1,
 637:     BinOpFrag<(udiv node:$LHS, node:$RHS)>, "divu32">;
 638: 
 639:   def DECGT32 : I_5_XZ<0x4, 0x1, "decgt32", (outs GPR:$rz, CARRY:$cout),
 640:                        (ins GPR:$rx, uimm5:$imm5), []> {
 641:     bits<0> cout;
 642:   }
 643:   def DECLT32 : I_5_XZ<0x4, 0x2, "declt32", (outs GPR:$rz, CARRY:$cout),
 644:                        (ins GPR:$rx, uimm5:$imm5), []> {
 645:     bits<0> cout;
 646:   }
 647:   def DECNE32 : I_5_XZ<0x4, 0x4, "decne32", (outs GPR:$rz, CARRY:$cout),
 648:                        (ins GPR:$rx, uimm5:$imm5), []> {
```

- EN: This range defines declarative TableGen records such as DIVS32, DIVU32, DECGT32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 DIVS32, DIVU32, DECGT32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 649-666

```tablegen
 649:     bits<0> cout;
 650:   }
 651: 
 652:   def SEXT32 : I_5_XZ_U<0x16, (outs GPR:$rz), (ins GPR:$rx, uimm5:$msb, uimm5:$lsb), "sext32", []>;
 653:   let isCodeGenOnly = 1 in {
 654:   def SEXTB32 : I_5_XZ_US<0x16, 0, 7, "sextb32", sext_inreg, i8>;
 655:   def SEXTH32 : I_5_XZ_US<0x16, 0, 15, "sexth32", sext_inreg, i16>;
 656:   def ZEXTB32 : I_5_XZ_UZ<0x15, 0, 7, "zextb32", 255>;
 657:   def ZEXTH32 : I_5_XZ_UZ<0x15, 0, 15, "zexth32", 65535>;
 658:   }
 659:   def ZEXT32 : I_5_XZ_U<0x15, (outs GPR:$rz), (ins GPR:$rx, uimm5:$msb, uimm5:$lsb), "zext32",[]>;
 660: 
 661:   let Constraints = "$rZ = $rz" in
 662:   def INS32 : I_5_XZ_INS<0b010111, (outs GPR:$rz), (ins GPR:$rZ, GPR:$rx, uimm5_msb_size:$msb, uimm5:$lsb), "ins32", []>;
 663: }
 664: 
 665: let Predicates = [iHas3E3r1] in {
 666: def MULTS32 : R_YXZ<0x3e, 0x20, 0x10, (outs GPRPair:$rz),
```

- EN: This range defines declarative TableGen records such as SEXT32, SEXTB32, SEXTH32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SEXT32, SEXTB32, SEXTH32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 667-684

```tablegen
 667:     (ins GPR:$rx, GPR:$ry), "mul.s32", []>;
 668: def MULTU32 : R_YXZ<0x3e, 0x20, 0x00, (outs GPRPair:$rz),
 669:     (ins GPR:$rx, GPR:$ry), "mul.u32", []>;
 670: 
 671: let Constraints = "$rZ = $rz" in {
 672: def MULATS32 : R_YXZ<0x3e, 0x20, 0x14, (outs GPRPair:$rZ),
 673:     (ins GPRPair:$rz, GPR:$rx, GPR:$ry), "mula.s32", []>;
 674: def MULATU32 : R_YXZ<0x3e, 0x20, 0x04, (outs GPRPair:$rZ),
 675:     (ins GPRPair:$rz, GPR:$rx, GPR:$ry), "mula.u32", []>;
 676: }
 677: }
 678: 
 679: def MULSH32 : R_YXZ<0x31, 0b100100, 0b00001, (outs GPR:$rz),
 680:     (ins GPR:$rx, GPR:$ry), "mulsh32", []>;
 681: 
 682: //===----------------------------------------------------------------------===//
 683: // Load & Store instructions.
 684: //===----------------------------------------------------------------------===//
```

- EN: This range defines declarative TableGen records such as MULTU32, MULATS32, MULATU32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MULTU32, MULATS32, MULATU32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 685-702

```tablegen
 685: 
 686: def LD32B : I_LD<AddrMode32B, 0x0, "ld32.b", uimm12>;
 687: def LD32H : I_LD<AddrMode32H, 0x1, "ld32.h", uimm12_1>;
 688: def LD32W : I_LD<AddrMode32WD, 0x2, "ld32.w", uimm12_2>;
 689: 
 690: let OutOperandList = (outs GPRPair:$rz) in
 691: def LD32D : I_LD<AddrMode32WD, 0x3, "ld32.d", uimm12_2>;
 692: 
 693: let Predicates = [iHasE2] in {
 694:   def LD32BS : I_LD<AddrMode32B, 0x4, "ld32.bs", uimm12>;
 695:   def LD32HS : I_LD<AddrMode32H, 0x5, "ld32.hs", uimm12_1>;
 696: 
 697:   def LDM32 : I_5_YX<0b110100, 0b000111,
 698:     (outs), (ins GPR:$rx, regseq:$regs, variable_ops), "ldm32\t$regs, (${rx})", []>;
 699:   def STM32 : I_5_YX<0b110101, 0b000111,
 700:     (outs), (ins GPR:$rx, regseq:$regs, variable_ops), "stm32\t$regs, (${rx})", []>;
 701: 
 702:   let Size = 4, isCodeGenOnly = 0 in {
```

- EN: This range defines declarative TableGen records such as LD32B, LD32H, LD32W, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LD32B, LD32H, LD32W 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 703-720

```tablegen
 703:   def LDQ32 : CSKYPseudo<(outs), (ins GPR:$rx, regseq:$regs, variable_ops),
 704:                              "ldq32\t$regs, (${rx})", []>;
 705:   def STQ32 : CSKYPseudo<(outs), (ins GPR:$rx, regseq:$regs, variable_ops),
 706:                              "stq32\t$regs, (${rx})", []>;
 707:   }
 708: 
 709: }
 710: 
 711: def ST32B : I_ST<AddrMode32B, 0x0, "st32.b", uimm12>;
 712: def ST32H : I_ST<AddrMode32H, 0x1, "st32.h", uimm12_1>;
 713: def ST32W : I_ST<AddrMode32WD, 0x2, "st32.w", uimm12_2>;
 714: 
 715: let InOperandList = (ins GPRPair:$rz, GPR:$rx, uimm12_2:$imm12 ) in
 716: def ST32D : I_ST<AddrMode32WD, 0x3, "st32.d", uimm12_2>;
 717: 
 718: let Predicates = [iHas2E3] in {
 719:   def LDR32B :  I_LDR<0x0, "ldr32.b">;
 720:   def LDR32BS :  I_LDR<0x4, "ldr32.bs">;
```

- EN: This range defines declarative TableGen records such as LDQ32, STQ32, ST32B, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDQ32, STQ32, ST32B 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 721-738

```tablegen
 721:   def LDR32H :  I_LDR<0x1, "ldr32.h">;
 722:   def LDR32HS :  I_LDR<0x5, "ldr32.hs">;
 723:   def LDR32W :  I_LDR<0x2, "ldr32.w">;
 724:   def STR32B :  I_STR<0x0, "str32.b">;
 725:   def STR32H :  I_STR<0x1, "str32.h">;
 726:   def STR32W :  I_STR<0x2, "str32.w">;
 727: }
 728: 
 729: // Indicate that we're dumping the CR register, so we'll need to
 730: // scavenge a register for it.
 731: let mayStore = 1 in {
 732: def SPILL_CARRY : CSKYPseudo<(outs), (ins CARRY:$cond, GPR:$rx, uimm12_2:$imm),
 733:                              "!SPILL_CARRY $cond, $rx, $imm", []>;
 734: }
 735: 
 736: // Indicate that we're restoring the CR register (previously
 737: // spilled), so we'll need to scavenge a register for it.
 738: let mayLoad = 1 in {
```

- EN: This range defines declarative TableGen records such as LDR32H, LDR32HS, LDR32W, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LDR32H, LDR32HS, LDR32W 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 739-756

```tablegen
 739: def RESTORE_CARRY : CSKYPseudo<(outs CARRY:$cond), (ins GPR:$rx, uimm12_2:$imm),
 740:                                 "!RESTORE_CARRY $cond, $rx, $imm", []>;
 741: }
 742: 
 743: let mayLoad = 1 in {
 744: def STORE_PAIR : CSKYPseudo<(outs), (ins GPRPair:$rz, GPR:$rx, uimm12_2:$imm),
 745:                             "!STORE_PAIR $rz, $rx, $imm", []>;
 746: }
 747: 
 748: let mayLoad = 1 in {
 749: def LOAD_PAIR : CSKYPseudo<(outs GPRPair:$rz), (ins GPR:$rx, uimm12_2:$imm),
 750:                             "!LOAD_PAIR $rz, $rx, $imm", []>;
 751: }
 752: 
 753: //===----------------------------------------------------------------------===//
 754: // Compare instructions.
 755: //===----------------------------------------------------------------------===//
 756: let Predicates = [iHasE2] in {
```

- EN: This range defines declarative TableGen records such as RESTORE_CARRY, STORE_PAIR, LOAD_PAIR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 RESTORE_CARRY, STORE_PAIR, LOAD_PAIR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 757-774

```tablegen
 757:   def CMPNEI32 : I_16_X<0x1A, "cmpnei32", uimm16>;
 758:   def CMPHSI32 : I_16_X<0x18, "cmphsi32", oimm16>;
 759:   def CMPLTI32 : I_16_X<0x19, "cmplti32", oimm16>;
 760:   def CMPLEI32 : CSKYPseudo<(outs CARRY:$ca), (ins GPR:$rx, uimm16:$imm16),
 761:     "cmplei32\t$rx, $imm16", []>;
 762: }
 763: let Predicates = [iHas2E3] in {
 764:   def CMPNE32 : R_YX<0x1, 0x4, "cmpne32">;
 765:   def CMPHS32 : R_YX<0x1, 0x1, "cmphs32">;
 766:   def CMPLT32 : R_YX<0x1, 0x2, "cmplt32">;
 767: 
 768:   def SETC32 : CSKY32Inst<AddrModeNone, 0x31, (outs CARRY:$ca), (ins), "setc32",
 769:                           []> {
 770:     bits<0> ca;
 771:     let Inst{25 - 21} = 0; //rx
 772:     let Inst{20 - 16} = 0; //ry
 773:     let Inst{15 - 10} = 0x1;
 774:     let Inst{9 - 5} = 0x1;
```

- EN: This range defines declarative TableGen records such as CMPNEI32, CMPHSI32, CMPLTI32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CMPNEI32, CMPHSI32, CMPLTI32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 775-792

```tablegen
 775:     let Inst{4 - 0} = 0;
 776:     let isCompare = 1;
 777:   }
 778:   def CLRC32 : CSKY32Inst<AddrModeNone, 0x31, (outs CARRY:$ca), (ins), "clrc32",
 779:                           []> {
 780:     bits<0> ca;
 781:     let Inst{25 - 21} = 0; //rx
 782:     let Inst{20 - 16} = 0; //ry
 783:     let Inst{15 - 10} = 0x1;
 784:     let Inst{9 - 5} = 0x4;
 785:     let Inst{4 - 0} = 0;
 786:     let isCompare = 1;
 787:   }
 788: 
 789:   def TST32 : R_YX<0x8, 0x4, "tst32">;
 790:   def TSTNBZ32 : R_X<0x8, 0x8, (outs CARRY:$ca), (ins GPR:$rx), "tstnbz32",
 791:                      []> {
 792:     bits<0> ca;
```

- EN: This range defines declarative TableGen records such as CLRC32, TST32, TSTNBZ32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CLRC32, TST32, TSTNBZ32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 793-810

```tablegen
 793:   }
 794: }
 795: 
 796: //===----------------------------------------------------------------------===//
 797: // Data move instructions.
 798: //===----------------------------------------------------------------------===//
 799: 
 800: let Predicates= [iHasE2] in {
 801:   let isCodeGenOnly = 1 in {
 802:   def MOVT32 : R_ZX<0x3, 0x2, "movt32", []>;
 803:   def MOVF32 : R_ZX<0x3, 0x1, "movf32", []>;
 804:   }
 805:   def MOVI32 : I_16_MOV<0x10, "movi32", uimm16>;
 806:   let Size = 4, isCodeGenOnly = 0 in
 807:   def BGENI : CSKYPseudo<(outs GPR:$dst), (ins uimm5:$imm), "bgeni\t$dst, $imm", []>;
 808:   def MOVIH32 : I_16_MOV<0x11, "movih32", uimm16_16_xform>;
 809:   def MVC32 : R_Z_1<0x1, 0x8, "mvc32">;
 810:   let isCodeGenOnly = 1 in
```

- EN: This range defines declarative TableGen records such as MOVT32, MOVF32, MOVI32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MOVT32, MOVF32, MOVI32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 811-828

```tablegen
 811:   def MOV32 : R_XZ<0x12, 0x1, "mov32">;
 812: 
 813:   let usesCustomInserter = 1 in
 814:   def ISEL32 : CSKYPseudo<(outs GPR:$dst), (ins CARRY:$cond, GPR:$src1, GPR:$src2),
 815:     "!isel32\t$dst, $src1, src2", [(set GPR:$dst, (select CARRY:$cond, GPR:$src1, GPR:$src2))]>;
 816: }
 817: 
 818: let Predicates = [iHas2E3] in {
 819:   def MVCV32 : R_Z_1<0x1, 0x10, "mvcv32">;
 820:   def CLRF32 : R_Z_2<0xB, 0x1, "clrf32">;
 821:   def CLRT32 : R_Z_2<0xB, 0x2, "clrt32">;
 822: }
 823: 
 824: //===----------------------------------------------------------------------===//
 825: // Branch and call instructions.
 826: //===----------------------------------------------------------------------===//
 827: 
 828: let isBranch = 1, isTerminator = 1 in {
```

- EN: This range defines declarative TableGen records such as MOV32, ISEL32, MVCV32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MOV32, ISEL32, MVCV32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 829-846

```tablegen
 829:   let isBarrier = 1, isPredicable = 1 in
 830:     def BR32 : I_16_L<0x0, (outs), (ins br_symbol:$imm16), "br32\t$imm16",
 831:                      [(br bb:$imm16)]>;
 832: 
 833:   def BT32 : I_16_L<0x3, (outs), (ins CARRY:$ca, br_symbol:$imm16),
 834:                     "bt32\t$imm16", [(brcond CARRY:$ca, bb:$imm16)]>,
 835:              Requires<[iHasE2]> {
 836:     bits<0> ca;
 837:   }
 838:   def BF32 : I_16_L<0x2, (outs), (ins CARRY:$ca, br_symbol:$imm16),
 839:                     "bf32\t$imm16", []>, Requires<[iHasE2]> {
 840:     bits<0> ca;
 841:   }
 842: }
 843: 
 844: let Predicates = [iHas2E3] in {
 845:   def BEZ32 : I_16_X_L<0x8, "bez32", br_symbol>;
 846:   def BNEZ32 : I_16_X_L<0x9, "bnez32", br_symbol>;
```

- EN: This range defines declarative TableGen records such as BR32, BT32, BF32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BR32, BT32, BF32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 847-864

```tablegen
 847:   def BHZ32 : I_16_X_L<0xA, "bhz32", br_symbol>;
 848:   def BLSZ32 : I_16_X_L<0xB, "blsz32", br_symbol>;
 849:   def BLZ32 : I_16_X_L<0xC, "blz32", br_symbol>;
 850:   def BHSZ32 : I_16_X_L<0xD, "bhsz32", br_symbol>;
 851: 
 852:   let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in {
 853:     def JMP32 : I_16_JX<0x6, "jmp32", [(brind GPR:$rx)]>; // jmp to register
 854:     def JMPI32 : I_16_L<0x16, (outs), (ins constpool_symbol:$imm16),
 855:                    "jmpi32\t$imm16", []>;
 856:   }
 857: 
 858:   let isCall = 1, Defs = [ R15 ] in
 859:     def JSR32 : I_16_JX<0x7, "jsr32", []>;
 860: 
 861:   let isCall = 1, Defs = [ R15 ] , mayLoad = 1 in
 862:     def JSRI32: I_16_L<0x17, (outs),
 863:       (ins constpool_symbol:$imm16), "jsri32\t$imm16", []>;
 864: }
```

- EN: This range defines declarative TableGen records such as BHZ32, BLSZ32, BLZ32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BHZ32, BLSZ32, BLZ32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 865-882

```tablegen
 865: 
 866: def BNEZAD32 : CSKY32Inst<AddrModeNone, 0x3a,
 867:   (outs GPR:$rx_u), (ins GPR:$rx, br_symbol:$imm16), "bnezad32\t$rx, $imm16", []> {
 868:   bits<5> rx;
 869:   bits<16> imm16;
 870:   let Inst{25 - 21} = 0x1;
 871:   let Inst{20 - 16} = rx;
 872:   let Inst{15 - 0} = imm16;
 873:   let isBranch = 1;
 874:   let isTerminator = 1;
 875:   let Constraints = "$rx_u = $rx";
 876:   let Predicates = [iHas2E3, iHas10E60];
 877: }
 878: 
 879: def BSR32 : J<0x38, (outs), (ins call_symbol:$offset), "bsr32", []>;
 880: 
 881: def BSR32_BR : J<0x38, (outs), (ins call_symbol:$offset), "bsr32", []>{
 882:   let isCodeGenOnly = 1;
```

- EN: This range defines declarative TableGen records such as BNEZAD32, BSR32, BSR32_BR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BNEZAD32, BSR32, BSR32_BR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 883-900

```tablegen
 883:   let isBranch = 1;
 884:   let isTerminator = 1;
 885:   let isBarrier = 1;
 886:   let isPredicable = 1;
 887:   let Defs = [ R15 ];
 888: }
 889: 
 890: //===----------------------------------------------------------------------===//
 891: // Symbol address instructions.
 892: //===----------------------------------------------------------------------===//
 893: 
 894: def data_symbol_b : data_symbol<"CSKY::fixup_csky_doffset_imm18", 0>;
 895: def data_symbol_h : data_symbol<"CSKY::fixup_csky_doffset_imm18_scale2", 1>;
 896: def data_symbol_w : data_symbol<"CSKY::fixup_csky_doffset_imm18_scale4", 2> {
 897:   let ParserMatchClass = DataAsmClass;
 898: }
 899: 
 900: let Predicates = [iHas2E3] in {
```

- EN: This range defines declarative TableGen records such as data_symbol_b, data_symbol_h, data_symbol_w, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 data_symbol_b, data_symbol_h, data_symbol_w 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 901-918

```tablegen
 901: 
 902: def GRS32 : I_18_Z_L<0x3, "grs32\t$rz, $offset",
 903:                     (outs GPR:$rz), (ins bare_symbol:$offset), []>;
 904: 
 905: let Uses = [R28] in {
 906: def LRS32B : I_18_Z_L<0x0, "lrs32.b\t$rz, $offset",
 907:                     (outs GPR:$rz), (ins data_symbol_b:$offset), []>;
 908: def LRS32H : I_18_Z_L<0x1, "lrs32.h\t$rz, $offset",
 909:                     (outs GPR:$rz), (ins data_symbol_h:$offset), []>;
 910: def LRS32W : I_18_Z_L<0x2, "lrs32.w\t$rz, $offset",
 911:                     (outs GPR:$rz), (ins data_symbol_w:$offset), []>;
 912: def SRS32B : I_18_Z_L<0x4, "srs32.b\t$rz, $offset",
 913:                     (outs), (ins GPR:$rz, data_symbol_b:$offset), []>;
 914: def SRS32H : I_18_Z_L<0x5, "srs32.h\t$rz, $offset",
 915:                     (outs), (ins GPR:$rz, data_symbol_h:$offset), []>;
 916: def SRS32W : I_18_Z_L<0x6, "srs32.w\t$rz, $offset",
 917:                     (outs), (ins GPR:$rz, data_symbol_w:$offset), []>;
 918: }
```

- EN: This range defines declarative TableGen records such as GRS32, LRS32B, LRS32H, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 GRS32, LRS32B, LRS32H 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 919-936

```tablegen
 919: 
 920: def PUSH32 : I_12_PP<0b11111, 0b00000, (outs), (ins reglist:$regs, variable_ops), "push32 $regs">;
 921: 
 922: let Uses = [R14, R15], isReturn = 1, isTerminator = 1, isBarrier = 1 in
 923: def POP32 : I_12_PP<0b11110, 0b00000, (outs), (ins reglist:$regs, variable_ops), "pop32 $regs">;
 924: 
 925: }
 926: 
 927: let mayLoad = 1, mayStore = 0 in {
 928: def LRW32 : I_16_Z_L<0x14, "lrw32", (ins constpool_symbol:$imm16), []>;
 929: let isCodeGenOnly = 1 in
 930: def LRW32_Gen : I_16_Z_L<0x14, "lrw32", (ins bare_symbol:$src1, constpool_symbol:$imm16), []>;
 931: }
 932: 
 933: //===----------------------------------------------------------------------===//
 934: // Atomic and fence instructions.
 935: //===----------------------------------------------------------------------===//
 936: 
```

- EN: This range defines declarative TableGen records such as PUSH32, POP32, LRW32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 PUSH32, POP32, LRW32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 937-954

```tablegen
 937: let Predicates = [iHasMP1E2] in {
 938:   def BRWARW : BAR<0b01111, "bar.brwarw", 0>;
 939:   def BRWARWS : BAR<0b01111, "bar.brwarws", 1>;
 940:   def BRARW : BAR<0b00111, "bar.brarw", 0>;
 941:   def BRARWS : BAR<0b00111, "bar.brarws", 1>;
 942:   def BRWAW : BAR<0b01110, "bar.brwaw", 0>;
 943:   def BRWAWS : BAR<0b01110, "bar.brwaws", 1>;
 944:   def BRAR : BAR<0b00101, "bar.brar", 0>;
 945:   def BRARS : BAR<0b00101, "bar.brars", 1>;
 946:   def BWAW : BAR<0b01010, "bar.bwaw", 0>;
 947:   def BWAWS : BAR<0b01010, "bar.bwaws", 1>;
 948: 
 949:   def LDEX32W : I_LD<AddrMode32WD, 0x7, "ldex32.w", uimm12_2>;
 950:   let Constraints = "$rd = $rz" in
 951:     def STEX32W : I_LDST<AddrMode32WD, 0x37, 7,
 952:       (outs GPR:$rd), (ins GPR:$rz, GPR:$rx, uimm12_2:$imm12), "stex32.w", []>;
 953: }
 954: 
```

- EN: This range defines declarative TableGen records such as BRWARW, BRWARWS, BRARW, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BRWARW, BRWARWS, BRARW 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 955-972

```tablegen
 955: //===----------------------------------------------------------------------===//
 956: // Other operation instructions.
 957: //===----------------------------------------------------------------------===//
 958: 
 959: let Predicates = [iHas2E3] in {
 960:   def BREV32 : R_XZ<0x18, 0x10, "brev32">;
 961:   def ABS32 : R_XZ<0x0, 0x10, "abs32">;
 962:   def BGENR32 : R_XZ<0x14, 0x2, "bgenr32">;
 963:   def REVB32 : R_XZ<0x18, 0x4, "revb32">;
 964:   def REVH32 : R_XZ<0x18, 0x8, "revh32">;
 965: }
 966: 
 967: let Predicates = [iHasE2] in {
 968:   def FF0 : R_XZ<0x1F, 0x1, "ff0.32">;
 969:   def FF1 : R_XZ<0x1F, 0x2, "ff1.32">;
 970:   def XTRB0 : R_XZ<0x1C, 0x1, "xtrb0.32">;
 971:   def XTRB1 : R_XZ<0x1C, 0x2, "xtrb1.32">;
 972:   def XTRB2 : R_XZ<0x1C, 0x4, "xtrb2.32">;
```

- EN: This range defines declarative TableGen records such as BREV32, ABS32, BGENR32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 BREV32, ABS32, BGENR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 973-990

```tablegen
 973:   def XTRB3 : R_XZ<0x1C, 0x8, "xtrb3.32">;
 974:   def BTSTI32 : I_5_X<0x0A, 0x4, "btsti32", uimm5, []>;
 975:   def BCLRI32 : I_5_XZ<0xA, 0x1, "bclri32",
 976:     (outs GPR:$rz), (ins GPR:$rx, uimm5:$imm5), []>;
 977:   def BSETI32 : I_5_XZ<0xA, 0x2, "bseti32",
 978:     (outs GPR:$rz), (ins GPR:$rx, uimm5:$imm5), []>;
 979: }
 980: 
 981: //===----------------------------------------------------------------------===//
 982: // Special instructions.
 983: //===----------------------------------------------------------------------===//
 984: 
 985: def MFFCR : CSKY32Inst<AddrModeNone, 0x30,
 986:   (outs GPR:$rx), (ins), "mfcr\t$rx, fcr", []> {
 987:   bits<5> rx;
 988: 
 989:   let Inst{25 - 21} = 0b00010;
 990:   let Inst{20 - 16} = 0b00001;
```

- EN: This range defines declarative TableGen records such as XTRB3, BTSTI32, BCLRI32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 XTRB3, BTSTI32, BCLRI32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 991-1008

```tablegen
 991:   let Inst{15 - 10} = 0b011000;
 992:   let Inst{9 - 5} = 0b00001;
 993:   let Inst{4 - 0} = rx;
 994:   let hasSideEffects = 1;
 995:   let isCodeGenOnly = 1;
 996: }
 997: 
 998: def MTFCR : CSKY32Inst<AddrModeNone, 0x30,
 999:   (outs), (ins GPR:$rx), "mtcr\t$rx, fcr", []> {
1000:   bits<5> rx;
1001: 
1002:   let Inst{25 - 21} = 0b00010;
1003:   let Inst{20 - 16} = rx;
1004:   let Inst{15 - 10} = 0b011001;
1005:   let Inst{9 - 5} = 0b00001;
1006:   let Inst{4 - 0} = 0b00001;
1007:   let hasSideEffects = 1;
1008:   let isCodeGenOnly = 1;
```

- EN: This range defines declarative TableGen records such as MTFCR, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 MTFCR 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1009-1026

```tablegen
1009: }
1010: 
1011: def SYNC32 : I_5_IMM5<0x30, 0b000001, 0b00001, "sync32", uimm5, []>;
1012: 
1013: def SYNC0_32 : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins),
1014:                  "sync32", []> {
1015:   let Inst{25 - 21} = 0;
1016:   let Inst{20 - 16} = 0;
1017:   let Inst{15 - 10} = 0b000001;
1018:   let Inst{9 - 5} = 0b00001;
1019:   let Inst{4 - 0} = 0;
1020: }
1021: 
1022: def SYNC_32_I : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins),
1023:                  "sync32.i", []> {
1024:   let Inst{25 - 21} = 1;
1025:   let Inst{20 - 16} = 0;
1026:   let Inst{15 - 10} = 0b000001;
```

- EN: This range defines declarative TableGen records such as SYNC32, SYNC0_32, SYNC_32_I, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SYNC32, SYNC0_32, SYNC_32_I 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1027-1044

```tablegen
1027:   let Inst{9 - 5} = 0b00001;
1028:   let Inst{4 - 0} = 0;
1029: }
1030: 
1031: def SYNC_32_S : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins),
1032:                  "sync32.s", []> {
1033:   let Inst{25 - 21} = 0b10000;
1034:   let Inst{20 - 16} = 0;
1035:   let Inst{15 - 10} = 0b000001;
1036:   let Inst{9 - 5} = 0b00001;
1037:   let Inst{4 - 0} = 0;
1038: }
1039: 
1040: def SYNC_32_IS : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins),
1041:                  "sync32.is", []> {
1042:   let Inst{25 - 21} = 0b10001;
1043:   let Inst{20 - 16} = 0;
1044:   let Inst{15 - 10} = 0b000001;
```

- EN: This range defines declarative TableGen records such as SYNC_32_S, SYNC_32_IS, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 SYNC_32_S, SYNC_32_IS 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1045-1062

```tablegen
1045:   let Inst{9 - 5} = 0b00001;
1046:   let Inst{4 - 0} = 0;
1047: }
1048: 
1049: let Predicates = [iHas2E3] in {
1050:   def RFI32 : I_5_XZ_PRIVI<0x11, 0x1, "rfi32">;
1051:   def SCE32 : I_5_IMM5<0x30, 0b000110, 0b00001, "sce32", uimm4, []>;
1052: }
1053: let Predicates = [HasExtendLrw] in
1054: def IDLY32 : I_5_IMM5<0x30, 0b000111, 0b00001, "idly32", imm5_idly, []>;
1055: def STOP32 : I_5_XZ_PRIVI<0x12, 0x1, "stop32">;
1056: def WAIT32 : I_5_XZ_PRIVI<0x13, 0x1, "wait32">;
1057: def DOZE32 : I_5_XZ_PRIVI<0x14, 0x1, "doze32">;
1058: def WE32 : I_5_XZ_PRIVI<0b010101, 0x1, "we32">;
1059: def SE32 : I_5_XZ_PRIVI<0b010110, 0x1, "se32">;
1060: def WSC32 : I_5_XZ_PRIVI<0b001111, 0x1, "wsc32">;
1061: 
1062: def CPOP32 : I_CPOP<(outs), (ins uimm5:$cpid, uimm20:$usdef), "cpop32 <$cpid, ${usdef}>">;
```

- EN: This range defines declarative TableGen records such as RFI32, SCE32, IDLY32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 RFI32, SCE32, IDLY32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1063-1080

```tablegen
1063: def CPRC32 : I_CP<0b0100, (outs CARRY:$ca), (ins uimm5:$cpid, uimm12:$usdef),
1064:                   "cprc32 <$cpid, ${usdef}>"> {
1065:   bits<0> ca;
1066: }
1067: def CPRCR32 : I_CP_Z<0b0010, (outs GPR:$rz), (ins uimm5:$cpid, uimm12:$usdef), "cprcr32 $rz, <$cpid, ${usdef}>">;
1068: def CPRGR32 : I_CP_Z<0b0000, (outs GPR:$rz), (ins uimm5:$cpid, uimm12:$usdef), "cprgr32 $rz, <$cpid, ${usdef}>">;
1069: def CPWCR32 : I_CP_Z<0b0011, (outs), (ins GPR:$rz, uimm5:$cpid, uimm12:$usdef), "cpwcr32 $rz, <$cpid, ${usdef}>">;
1070: def CPWGR32 : I_CP_Z<0b0001, (outs), (ins GPR:$rz, uimm5:$cpid, uimm12:$usdef), "cpwgr32 $rz, <$cpid, ${usdef}>">;
1071: 
1072: let Predicates = [iHas3r2E3r3] in {
1073: def DCACHE_IALL32 : I_5_CACHE<0b100101, 0b01000, "dcache32.iall">;
1074: def DCACHE_CALL32 : I_5_CACHE<0b100101, 0b00100, "dcache32.call">;
1075: def DCACHE_CIALL32 : I_5_CACHE<0b100101, 0b01100, "dcache32.ciall">;
1076: def DCACHE_IVA32 : I_5_X_CACHE<0b100101, 0b01011, "dcache32.iva">;
1077: def DCACHE_ISW32: I_5_X_CACHE<0b100101, 0b01010, "dcache32.isw">;
1078: def DCACHE_CVA32 : I_5_X_CACHE<0b100101, 0b00111, "dcache32.cva">;
1079: def DCACHE_CVAL32 : I_5_X_CACHE<0b100101, 0b10111, "dcache32.cval1">;
1080: def DCACHE_CSW32 : I_5_X_CACHE<0b100101, 0b00110, "dcache32.csw">;
```

- EN: This range defines declarative TableGen records such as CPRC32, CPRCR32, CPRGR32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 CPRC32, CPRCR32, CPRGR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1081-1098

```tablegen
1081: def DCACHE_CIVA32 : I_5_X_CACHE<0b100101, 0b01111, "dcache32.civa">;
1082: def DCACHE_CISW32 : I_5_X_CACHE<0b100101, 0b01110, "dcache32.cisw">;
1083: 
1084: def ICACHE_IALL32 : I_5_CACHE<0b100100, 0b01000, "icache32.iall">;
1085: def ICACHE_IALLS32 : I_5_CACHE<0b100100, 0b11000, "icache32.ialls">;
1086: def ICACHE_IVA32 : I_5_X_CACHE<0b100100, 0b01011, "icache32.iva">;
1087: 
1088: def TLBI_VAA32 : I_5_X_CACHE<0b100010, 0b00010, "tlbi32.vaa">;
1089: def TLBI_VAAS32 : I_5_X_CACHE<0b100010, 0b10010, "tlbi32.vaas">;
1090: def TLBI_ASID32 : I_5_X_CACHE<0b100010, 0b00001, "tlbi32.asid">;
1091: def TLBI_ASIDS32 : I_5_X_CACHE<0b100010, 0b10001, "tlbi32.asids">;
1092: def TLBI_VA32 : I_5_X_CACHE<0b100010, 0b00011, "tlbi32.va">;
1093: def TLBI_VAS32 : I_5_X_CACHE<0b100010, 0b10011, "tlbi32.vas">;
1094: def TLBI_ALL32 : I_5_CACHE<0b100010, 0b00000, "tlbi32.all">;
1095: def TLBI_ALLS32 : I_5_CACHE<0b100010, 0b10000, "tlbi32.alls">;
1096: 
1097: def L2CACHE_IALL : I_5_CACHE<0b100110, 0b01000, "l2cache.iall">;
1098: def L2CACHE_CALL : I_5_CACHE<0b100110, 0b00100, "l2cache.call">;
```

- EN: This range defines declarative TableGen records such as DCACHE_CIVA32, DCACHE_CISW32, ICACHE_IALL32, capturing target metadata instead of imperative runtime logic.
- 中文: 这一段定义了 DCACHE_CIVA32, DCACHE_CISW32, ICACHE_IALL32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 1099-1116

```tablegen
1099: def L2CACHE_CIALL : I_5_CACHE<0b100110, 0b01100, "l2cache.ciall">;
1100: }
1101: 
1102: def PLDR32 :I_PLDR<AddrMode32WD, 0x36, 0b0110, (outs), (ins GPR:$rx, uimm12_2:$imm12), "pldr32", []>;
1103: def PLDW32 :I_PLDR<AddrMode32WD, 0x37, 0b0110, (outs), (ins GPR:$rx, uimm12_2:$imm12), "pldw32", []>;
1104: 
1105: def TRAP32 : CSKY32Inst<AddrModeNone, 0x30, (outs), (ins uimm2:$imm2), "trap32 ${imm2}", []> {
1106:   bits<2> imm2;
1107: 
1108:   let Inst{25 - 21} = 0;
1109:   let Inst{20 - 16} = 0;
1110:   let Inst{15 - 12} = 0b0010;
1111:   let Inst{11 - 10} = imm2;
1112:   let Inst{9 - 5} = 0b00001;
1113:   let Inst{4 - 0} = 0;
1114: 
1115: }
1116: 
```

- EN: This range defines declarative TableGen records such as L2CACHE_CIALL, PLDR32, PLDW32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 L2CACHE_CIALL, PLDR32, PLDW32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1117-1134

```tablegen
1117: //===----------------------------------------------------------------------===//
1118: // Instruction Patterns.
1119: //===----------------------------------------------------------------------===//
1120: 
1121: // Load & Store Patterns
1122: multiclass LdPat<PatFrag LoadOp, ImmLeaf imm_type, Instruction Inst, ValueType Type> {
1123:   def : Pat<(Type (LoadOp GPR:$rs1)), (Inst GPR:$rs1, 0)>;
1124:   def : Pat<(Type (LoadOp (i32 frameindex:$rs1))), (Inst (i32 (to_tframeindex tframeindex:$rs1)), 0)>;
1125:   def : Pat<(Type (LoadOp (add GPR:$rs1, imm_type:$uimm))),
1126:             (Inst GPR:$rs1, imm_type:$uimm)>;
1127:   def : Pat<(Type (LoadOp (add frameindex:$rs1, imm_type:$uimm))),
1128:             (Inst (i32 (to_tframeindex tframeindex:$rs1)), imm_type:$uimm)>;
1129:   def : Pat<(Type (LoadOp (eqToAdd frameindex:$rs1, imm_type:$uimm))),
1130:             (Inst (i32 (to_tframeindex tframeindex:$rs1)), imm_type:$uimm)>;
1131:   def : Pat<(Type (LoadOp (add GPR:$rs1, tglobaladdr:$gd))),
1132:             (Inst GPR:$rs1, tglobaladdr:$gd)>;
1133: }
1134: 
```

- EN: This range defines declarative TableGen records such as LdPat, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LdPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1135-1152

```tablegen
1135: defm : LdPat<extloadi8, uimm12, LD32B, i32>;
1136: defm : LdPat<zextloadi8, uimm12, LD32B, i32>;
1137: let Predicates = [iHasE2] in {
1138:   defm : LdPat<sextloadi8, uimm12, LD32BS, i32>;
1139: }
1140: defm : LdPat<extloadi16, uimm12_1, LD32H, i32>;
1141: defm : LdPat<zextloadi16, uimm12_1, LD32H, i32>;
1142: let Predicates = [iHasE2] in {
1143: defm : LdPat<sextloadi16, uimm12_1, LD32HS, i32>;
1144: }
1145: defm : LdPat<load, uimm12_2, LD32W, i32>;
1146: 
1147: multiclass LdrPat<PatFrag LoadOp, Instruction Inst, ValueType Type> {
1148:   def : Pat<(Type (LoadOp (add GPR:$rs1, GPR:$rs2))), (Inst GPR:$rs1, GPR:$rs2, 0)>;
1149:   def : Pat<(Type (LoadOp (add GPR:$rs1, (shl GPR:$rs2, (i32 1))))), (Inst GPR:$rs1, GPR:$rs2, 1)>;
1150:   def : Pat<(Type (LoadOp (add GPR:$rs1, (shl GPR:$rs2, (i32 2))))), (Inst GPR:$rs1, GPR:$rs2, 2)>;
1151:   def : Pat<(Type (LoadOp (add GPR:$rs1, (shl GPR:$rs2, (i32 3))))), (Inst GPR:$rs1, GPR:$rs2, 3)>;
1152: }
```

- EN: This range defines declarative TableGen records such as LdrPat, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 LdrPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1153-1170

```tablegen
1153: 
1154: let Predicates = [iHas2E3] in {
1155:   defm : LdrPat<zextloadi8, LDR32B, i32>;
1156:   defm : LdrPat<sextloadi8, LDR32BS, i32>;
1157:   defm : LdrPat<extloadi8, LDR32BS, i32>;
1158:   defm : LdrPat<zextloadi16, LDR32H, i32>;
1159:   defm : LdrPat<sextloadi16, LDR32HS, i32>;
1160:   defm : LdrPat<extloadi16, LDR32HS, i32>;
1161:   defm : LdrPat<load, LDR32W, i32>;
1162: }
1163: 
1164: multiclass StPat<PatFrag StoreOp, ValueType Type, ImmLeaf imm_type, Instruction Inst> {
1165:   def : Pat<(StoreOp Type:$rs2, GPR:$rs1), (Inst Type:$rs2, GPR:$rs1, 0)>;
1166:   def : Pat<(StoreOp Type:$rs2, frameindex:$rs1), (Inst Type:$rs2, (i32 (to_tframeindex tframeindex:$rs1)), 0)>;
1167:   def : Pat<(StoreOp Type:$rs2, (add GPR:$rs1, imm_type:$uimm12)),
1168:             (Inst Type:$rs2, GPR:$rs1, imm_type:$uimm12)>;
1169:   def : Pat<(StoreOp Type:$rs2, (add frameindex:$rs1, imm_type:$uimm12)),
1170:             (Inst Type:$rs2, (i32 (to_tframeindex tframeindex:$rs1)), imm_type:$uimm12)>;
```

- EN: This range defines declarative TableGen records such as StPat, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 StPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1171-1188

```tablegen
1171:   def : Pat<(StoreOp Type:$rs2, (eqToAdd frameindex:$rs1, imm_type:$uimm12)),
1172:             (Inst Type:$rs2, (i32 (to_tframeindex tframeindex:$rs1)), imm_type:$uimm12)>;
1173: }
1174: 
1175: defm : StPat<truncstorei8, i32, uimm12, ST32B>;
1176: defm : StPat<truncstorei16, i32, uimm12_1, ST32H>;
1177: defm : StPat<store, i32, uimm12_2, ST32W>;
1178: 
1179: multiclass StrPat<PatFrag StoreOp, ValueType Type, Instruction Inst> {
1180:   def : Pat<(StoreOp Type:$rz, (add GPR:$rs1, GPR:$rs2)), (Inst Type:$rz, GPR:$rs1, GPR:$rs2, 0)>;
1181:   def : Pat<(StoreOp Type:$rz, (add GPR:$rs1, (shl GPR:$rs2, (i32 1)))), (Inst Type:$rz, GPR:$rs1, GPR:$rs2, 1)>;
1182:   def : Pat<(StoreOp Type:$rz, (add GPR:$rs1, (shl GPR:$rs2, (i32 2)))), (Inst Type:$rz, GPR:$rs1, GPR:$rs2, 2)>;
1183:   def : Pat<(StoreOp Type:$rz, (add GPR:$rs1, (shl GPR:$rs2, (i32 3)))), (Inst Type:$rz, GPR:$rs1, GPR:$rs2, 3)>;
1184: }
1185: 
1186: let Predicates = [iHas2E3] in {
1187:   defm : StrPat<truncstorei8, i32, STR32B>;
1188:   defm : StrPat<truncstorei16, i32, STR32H>;
```

- EN: This range defines declarative TableGen records such as StrPat, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 StrPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1189-1206

```tablegen
1189:   defm : StrPat<store, i32, STR32W>;
1190: 
1191:   // Sext & Zext Patterns
1192:   def : Pat<(sext_inreg GPR:$src, i1), (SEXT32 GPR:$src, 0, 0)>;
1193:   def : Pat<(and GPR:$src, 255), (ZEXT32 GPR:$src, 7, 0)>;
1194:   def : Pat<(and GPR:$src, 65535), (ZEXT32 GPR:$src, 15, 0)>;
1195: 
1196:    // Call Patterns
1197:   def : Pat<(CSKY_CALL tglobaladdr, tconstpool:$src2), (JSRI32 tconstpool:$src2)>;
1198:   def : Pat<(CSKY_CALL texternalsym, tconstpool:$src2), (JSRI32 tconstpool:$src2)>;
1199:   def : Pat<(CSKY_TAIL tglobaladdr, tconstpool:$src2), (JMPI32 tconstpool:$src2)>;
1200:   def : Pat<(CSKY_TAIL texternalsym, tconstpool:$src2), (JMPI32 tconstpool:$src2)>;
1201: 
1202:   def : Pat<(CSKY_CALLReg GPR:$src), (JSR32 GPR:$src)>;
1203:   def : Pat<(CSKY_TAILReg GPR:$src), (JMP32 GPR:$src)>;
1204: }
1205: 
1206: // Symbol address Patterns
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1207-1224

```tablegen
1207: def : Pat<(CSKY_LOAD_ADDR tglobaladdr, tconstpool:$src2), (LRW32 tconstpool:$src2)>;
1208: def : Pat<(CSKY_LOAD_ADDR tblockaddress, tconstpool:$src2), (LRW32 tconstpool:$src2)>;
1209: def : Pat<(CSKY_LOAD_ADDR tjumptable:$src1, tconstpool:$src2), (LRW32_Gen tjumptable:$src1, tconstpool:$src2)>;
1210: def : Pat<(CSKY_LOAD_ADDR texternalsym, tconstpool:$src2), (LRW32 tconstpool:$src2)>;
1211: def : Pat<(CSKY_LOAD_ADDR tconstpool:$src1, tconstpool:$src2), (LRW32_Gen tconstpool:$src1, tconstpool:$src2)>;
1212: 
1213: let Predicates = [iHas2E3] in
1214:   def : Pat<(i32 constpool:$src), (GRS32 (to_tconstpool tconstpool:$src))>;
1215: 
1216: let Predicates = [iHasE2] in
1217:   def : Pat<(i32 constpool:$src),
1218:     (ORI32 (MOVIH32 (to_tconstpool_hi16 tconstpool:$src)),
1219:            (to_tconstpool_lo16 tconstpool:$src))>;
1220: 
1221: def : Pat<(i32 (load constpool:$src)), (LRW32 (to_tconstpool tconstpool:$src))>;
1222: 
1223: // Branch Patterns.
1224: let Predicates = [iHasE2] in {
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1225-1242

```tablegen
1225: def : Pat<(brcond CARRY:$ca, bb:$imm16),
1226:           (BT32 CARRY:$ca, bb:$imm16)>;
1227: 
1228: multiclass BTF32Pat0<PatFrag cond0, PatFrag cond1, ImmLeaf imm_ty, Instruction inst> {
1229:   def : Pat<(brcond (i32 (cond0 GPR:$rs1, uimm16:$rs2)), bb:$imm16),
1230:           (BT32 (inst GPR:$rs1, imm_ty:$rs2), bb:$imm16)>;
1231:   def : Pat<(brcond (i32 (cond1 GPR:$rs1, uimm16:$rs2)), bb:$imm16),
1232:           (BF32 (inst GPR:$rs1, imm_ty:$rs2), bb:$imm16)>;
1233: }
1234: 
1235: defm : BTF32Pat0<setne, seteq, uimm16, CMPNEI32>;
1236: defm : BTF32Pat0<setuge, setult, oimm16, CMPHSI32>;
1237: defm : BTF32Pat0<setlt, setge, oimm16, CMPLTI32>;
1238: 
1239: def : Pat<(brcond (i32 (setne (and GPR:$rs, imm32_1_pop_bit:$im), 0)), bb:$imm16),
1240:           (BT32 (BTSTI32 GPR:$rs, (imm32_1_pop_bit_XFORM imm32_1_pop_bit:$im)),
1241:                 bb:$imm16)>;
1242: def : Pat<(brcond (i32 (seteq (and GPR:$rs, imm32_1_pop_bit:$im), 0)), bb:$imm16),
```

- EN: This range defines declarative TableGen records such as BTF32Pat0, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BTF32Pat0 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1243-1260

```tablegen
1243:           (BF32 (BTSTI32 GPR:$rs, (imm32_1_pop_bit_XFORM imm32_1_pop_bit:$im)),
1244:                 bb:$imm16)>;
1245: }
1246: 
1247: let Predicates = [iHas2E3] in {
1248: 
1249: def : Pat<(brcond (i32 (setne GPR:$rs1, GPR:$rs2)), bb:$imm16),
1250:           (BT32 (CMPNE32 GPR:$rs1, GPR:$rs2), bb:$imm16)>;
1251: def : Pat<(brcond (i32 (seteq GPR:$rs1, GPR:$rs2)), bb:$imm16),
1252:           (BF32 (CMPNE32 GPR:$rs1, GPR:$rs2), bb:$imm16)>;
1253: 
1254: multiclass BTF32Pat1<PatFrag cond0, PatFrag cond1, Instruction cmp,
1255:                      Instruction br> {
1256:   def : Pat<(brcond (i32 (cond0 GPR:$rs1, GPR:$rs2)), bb:$imm16),
1257:             (br (cmp GPR:$rs1, GPR:$rs2), bb:$imm16)>;
1258:   def : Pat<(brcond (i32 (cond1 GPR:$rs1, GPR:$rs2)), bb:$imm16),
1259:             (br (cmp GPR:$rs2, GPR:$rs1), bb:$imm16)>;
1260: }
```

- EN: This range defines declarative TableGen records such as BTF32Pat1, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 BTF32Pat1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1261-1278

```tablegen
1261: 
1262: defm : BTF32Pat1<setuge, setule, CMPHS32, BT32>;
1263: defm : BTF32Pat1<setult, setugt, CMPHS32, BF32>;
1264: defm : BTF32Pat1<setlt, setgt, CMPLT32, BT32>;
1265: defm : BTF32Pat1<setge, setle, CMPLT32, BF32>;
1266: 
1267: def : Pat<(brcond (i32 (seteq GPR:$rs1, (i32 0))), bb:$imm16),
1268:           (BEZ32 GPR:$rs1, bb:$imm16)>;
1269: def : Pat<(brcond (i32 (setne GPR:$rs1, (i32 0))), bb:$imm16),
1270:           (BNEZ32 GPR:$rs1, bb:$imm16)>;
1271: def : Pat<(brcond (i32 (setlt GPR:$rs1, (i32 0))), bb:$imm16),
1272:           (BLZ32 GPR:$rs1, bb:$imm16)>;
1273: def : Pat<(brcond (i32 (setlt GPR:$rs1, (i32 1))), bb:$imm16),
1274:           (BLSZ32 GPR:$rs1, bb:$imm16)>;
1275: def : Pat<(brcond (i32 (setge GPR:$rs1, (i32 0))), bb:$imm16),
1276:           (BHSZ32 GPR:$rs1, bb:$imm16)>;
1277: def : Pat<(brcond (i32 (setge GPR:$rs1, (i32 1))), bb:$imm16),
1278:           (BHZ32 GPR:$rs1, bb:$imm16)>;
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1279-1296

```tablegen
1279: def : Pat<(brcond (i32 (setgt GPR:$rs1, (i32 0))), bb:$imm16),
1280:           (BHZ32 GPR:$rs1, bb:$imm16)>;
1281: def : Pat<(brcond (i32 (setgt GPR:$rs1, (i32 -1))), bb:$imm16),
1282:           (BHSZ32 GPR:$rs1, bb:$imm16)>;
1283: def : Pat<(brcond (i32 (setle GPR:$rs1, (i32 0))), bb:$imm16),
1284:           (BLSZ32 GPR:$rs1, bb:$imm16)>;
1285: def : Pat<(brcond (i32 (setle GPR:$rs1, (i32 -1))), bb:$imm16),
1286:           (BLZ32 GPR:$rs1, bb:$imm16)>;
1287: }
1288: 
1289: // Compare Patterns.
1290: let Predicates = [iHas2E3] in {
1291:   def : Pat<(setne GPR:$rs1, GPR:$rs2),
1292:             (CMPNE32 GPR:$rs1, GPR:$rs2)>;
1293:   def : Pat<(setne (and GPR:$rs, imm32_1_pop_bit:$im), 0),
1294:             (BTSTI32 GPR:$rs, (imm32_1_pop_bit_XFORM imm32_1_pop_bit:$im))>;
1295:   def : Pat<(i32 (seteq GPR:$rs1, GPR:$rs2)),
1296:             (MVCV32 (CMPNE32 GPR:$rs1, GPR:$rs2))>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1297-1314

```tablegen
1297:   def : Pat<(setuge GPR:$rs1, GPR:$rs2),
1298:             (CMPHS32 GPR:$rs1, GPR:$rs2)>;
1299:   def : Pat<(setule GPR:$rs1, GPR:$rs2),
1300:             (CMPHS32 GPR:$rs2, GPR:$rs1)>;
1301:   def : Pat<(i32 (setult GPR:$rs1, GPR:$rs2)),
1302:             (MVCV32 (CMPHS32 GPR:$rs1, GPR:$rs2))>;
1303:   def : Pat<(i32 (setugt GPR:$rs1, GPR:$rs2)),
1304:             (MVCV32 (CMPHS32 GPR:$rs2, GPR:$rs1))>;
1305:   def : Pat<(setlt GPR:$rs1, GPR:$rs2),
1306:             (CMPLT32 GPR:$rs1, GPR:$rs2)>;
1307:   def : Pat<(setgt GPR:$rs1, GPR:$rs2),
1308:             (CMPLT32 GPR:$rs2, GPR:$rs1)>;
1309:   def : Pat<(i32 (setge GPR:$rs1, GPR:$rs2)),
1310:             (MVCV32 (CMPLT32 GPR:$rs1, GPR:$rs2))>;
1311:   def : Pat<(i32 (setle GPR:$rs1, GPR:$rs2)),
1312:             (MVCV32 (CMPLT32 GPR:$rs2, GPR:$rs1))>;
1313: }
1314: 
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1315-1332

```tablegen
1315: let Predicates = [iHasE2] in {
1316:   def : Pat<(setne GPR:$rs1, uimm16:$rs2),
1317:             (CMPNEI32 GPR:$rs1, uimm16:$rs2)>;
1318:   let Predicates = [iHas2E3] in
1319:   def : Pat<(i32 (seteq GPR:$rs1, uimm16:$rs2)),
1320:             (MVCV32 (CMPNEI32 GPR:$rs1, uimm16:$rs2))>;
1321:   def : Pat<(setuge GPR:$rs1, oimm16:$rs2),
1322:             (CMPHSI32 GPR:$rs1, oimm16:$rs2)>;
1323:   let Predicates = [iHas2E3] in
1324:   def : Pat<(i32 (setult GPR:$rs1, oimm16:$rs2)),
1325:             (MVCV32 (CMPHSI32 GPR:$rs1, oimm16:$rs2))>;
1326:   def : Pat<(setlt GPR:$rs1, oimm16:$rs2),
1327:             (CMPLTI32 GPR:$rs1, oimm16:$rs2)>;
1328:   let Predicates = [iHas2E3] in
1329:   def : Pat<(i32 (setge GPR:$rs1, oimm16:$rs2)),
1330:             (MVCV32 (CMPLTI32 GPR:$rs1, oimm16:$rs2))>;
1331: }
1332: 
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1333-1350

```tablegen
1333: // Select Patterns.
1334: let Predicates = [iHasE2] in {
1335: 
1336: def : Pat<(select (i32 (setne GPR:$rs1, uimm16:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$false),
1337:           (INCT32 (CMPNEI32 GPR:$rs1, uimm16:$rs2), GPR:$false, GPR:$rx, uimm5:$imm)>;
1338: def : Pat<(select (i32 (seteq GPR:$rs1, uimm16:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$false),
1339:           (INCF32 (CMPNEI32 GPR:$rs1, uimm16:$rs2), GPR:$false, GPR:$rx, uimm5:$imm)>;
1340: def : Pat<(select (i32 (setne GPR:$rs1, uimm16:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$false),
1341:           (DECT32 (CMPNEI32 GPR:$rs1, uimm16:$rs2), GPR:$false, GPR:$rx,
1342:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1343: def : Pat<(select (i32 (seteq GPR:$rs1, uimm16:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$false),
1344:           (DECF32 (CMPNEI32 GPR:$rs1, uimm16:$rs2), GPR:$false, GPR:$rx,
1345:                   (imm_neg_XFORM uimm5:$imm))>;
1346: 
1347: multiclass INCDECPat<PatFrag cond0, PatFrag cond1, Instruction cmp> {
1348:   def : Pat<(select (i32 (cond0 GPR:$rs1, oimm16:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$other),
1349:             (INCT32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx, uimm5:$imm)>;
1350:   def : Pat<(select (i32 (cond1 GPR:$rs1, oimm16:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$other),
```

- EN: This range defines declarative TableGen records such as INCDECPat, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 INCDECPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1351-1368

```tablegen
1351:             (INCF32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx, uimm5:$imm)>;
1352:   def : Pat<(select (i32 (cond0 GPR:$rs1, oimm16:$rs2)), GPR:$other, (add GPR:$rx, uimm5:$imm)),
1353:             (INCF32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx, uimm5:$imm)>;
1354:   def : Pat<(select (i32 (cond1 GPR:$rs1, oimm16:$rs2)), GPR:$other, (add GPR:$rx, uimm5:$imm)),
1355:             (INCT32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx, uimm5:$imm)>;
1356:   def : Pat<(select (i32 (cond0 GPR:$rs1, oimm16:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$other),
1357:             (DECT32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx,
1358:                     (imm_neg_XFORM uimm5_neg:$imm))>;
1359:   def : Pat<(select (i32 (cond1 GPR:$rs1, oimm16:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$other),
1360:             (DECF32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx,
1361:                     (imm_neg_XFORM uimm5_neg:$imm))>;
1362:   def : Pat<(select (i32 (cond0 GPR:$rs1, oimm16:$rs2)), GPR:$other, (add GPR:$rx, uimm5_neg:$imm)),
1363:             (DECF32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx,
1364:                     (imm_neg_XFORM uimm5_neg:$imm))>;
1365:   def : Pat<(select (i32 (cond1 GPR:$rs1, oimm16:$rs2)), GPR:$other, (add GPR:$rx, uimm5_neg:$imm)),
1366:             (DECT32 (cmp GPR:$rs1, oimm16:$rs2), GPR:$other, GPR:$rx,
1367:                     (imm_neg_XFORM uimm5_neg:$imm))>;
1368: }
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1369-1386

```tablegen
1369: 
1370: defm : INCDECPat<setuge, setult, CMPHSI32>;
1371: defm : INCDECPat<setlt, setge, CMPLTI32>;
1372: 
1373: def : Pat<(select CARRY:$ca, (add GPR:$rx, uimm5:$imm), GPR:$other),
1374:           (INCT32 CARRY:$ca, GPR:$other, GPR:$rx, uimm5:$imm)>;
1375: def : Pat<(select CARRY:$ca, GPR:$other, (add GPR:$rx, uimm5:$imm)),
1376:           (INCF32 CARRY:$ca, GPR:$other, GPR:$rx, uimm5:$imm)>;
1377: def : Pat<(select (and CARRY:$ca, 1), (add GPR:$rx, uimm5:$imm), GPR:$other),
1378:           (INCT32 CARRY:$ca, GPR:$other, GPR:$rx, uimm5:$imm)>;
1379: def : Pat<(select (and CARRY:$ca, 1), GPR:$other, (add GPR:$rx, uimm5:$imm)),
1380:           (INCF32 CARRY:$ca, GPR:$other, GPR:$rx, uimm5:$imm)>;
1381: 
1382: def : Pat<(select CARRY:$ca, (add GPR:$rx, uimm5_neg:$imm), GPR:$other),
1383:           (DECT32 CARRY:$ca, GPR:$other, GPR:$rx, (imm_neg_XFORM uimm5_neg:$imm))>;
1384: def : Pat<(select CARRY:$ca, GPR:$other, (add GPR:$rx, uimm5_neg:$imm)),
1385:           (DECF32 CARRY:$ca, GPR:$other, GPR:$rx, (imm_neg_XFORM uimm5_neg:$imm))>;
1386: def : Pat<(select (and CARRY:$ca, 1), (add GPR:$rx, uimm5_neg:$imm), GPR:$other),
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1387-1404

```tablegen
1387:           (DECT32 CARRY:$ca, GPR:$other, GPR:$rx, (imm_neg_XFORM uimm5_neg:$imm))>;
1388: def : Pat<(select (and CARRY:$ca, 1), GPR:$other, (add GPR:$rx, uimm5_neg:$imm)),
1389:           (DECF32 CARRY:$ca, GPR:$other, GPR:$rx, (imm_neg_XFORM uimm5_neg:$imm))>;
1390: 
1391: def : Pat<(select CARRY:$ca, GPR:$rx, GPR:$false),
1392:           (MOVT32 CARRY:$ca, GPR:$rx, GPR:$false)>;
1393: def : Pat<(select (and CARRY:$ca, 1), GPR:$rx, GPR:$false),
1394:           (MOVT32 CARRY:$ca, GPR:$rx, GPR:$false)>;
1395: 
1396: multiclass MOVTF32Pat0<PatFrag cond0, PatFrag cond1, ImmLeaf imm_ty, Instruction inst> {
1397:   def : Pat<(select (i32 (cond0 GPR:$rs1, imm_ty:$rs2)), GPR:$rx, GPR:$false),
1398:             (MOVT32 (inst GPR:$rs1, imm_ty:$rs2), GPR:$rx, GPR:$false)>;
1399:   def : Pat<(select (i32 (cond1 GPR:$rs1, imm_ty:$rs2)), GPR:$rx, GPR:$false),
1400:             (MOVF32 (inst GPR:$rs1, imm_ty:$rs2), GPR:$rx, GPR:$false)>;
1401: }
1402: 
1403: defm : MOVTF32Pat0<setne, seteq, uimm16, CMPNEI32>;
1404: defm : MOVTF32Pat0<setuge, setult, oimm16, CMPHSI32>;
```

- EN: This range defines declarative TableGen records such as MOVTF32Pat0, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MOVTF32Pat0 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1405-1422

```tablegen
1405: defm : MOVTF32Pat0<setlt, setge, oimm16, CMPLTI32>;
1406: 
1407: def : Pat<(select CARRY:$ca, GPR:$rx, GPR:$false),
1408:           (ISEL32 CARRY:$ca, GPR:$rx, GPR:$false)>;
1409: def : Pat<(select (and CARRY:$ca, 1), GPR:$rx, GPR:$false),
1410:           (ISEL32 CARRY:$ca, GPR:$rx, GPR:$false)>;
1411: 
1412: def : Pat<(select (i32 (setne (and GPR:$rs, imm32_1_pop_bit:$im), 0)),
1413:                   GPR:$true, GPR:$false),
1414:           (MOVT32 (BTSTI32 GPR:$rs, (imm32_1_pop_bit_XFORM imm32_1_pop_bit:$im)),
1415:                   GPR:$true, GPR:$false)>;
1416: def : Pat<(select (i32 (seteq (and GPR:$rs, imm32_1_pop_bit:$im), 0)),
1417:                   GPR:$true, GPR:$false),
1418:           (MOVF32 (BTSTI32 GPR:$rs, (imm32_1_pop_bit_XFORM imm32_1_pop_bit:$im)),
1419:                   GPR:$true, GPR:$false)>;
1420: }
1421: 
1422: let Predicates = [iHas2E3] in {
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1423-1440

```tablegen
1423: def : Pat<(select (i32 (setne GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$false),
1424:           (INCT32 (CMPNE32 GPR:$rs1, GPR:$rs2), GPR:$false, GPR:$rx, uimm5:$imm)>;
1425: def : Pat<(select (i32 (seteq GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$false),
1426:           (INCF32 (CMPNE32 GPR:$rs1, GPR:$rs2), GPR:$false, GPR:$rx, uimm5:$imm)>;
1427: def : Pat<(select (i32 (setne GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$false),
1428:           (DECT32 (CMPNE32 GPR:$rs1, GPR:$rs2), GPR:$false, GPR:$rx,
1429:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1430: def : Pat<(select (i32 (seteq GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$false),
1431:           (DECF32 (CMPNE32 GPR:$rs1, GPR:$rs2), GPR:$false, GPR:$rx,
1432:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1433: 
1434: multiclass INCPat<PatFrag cond0, PatFrag cond1, Instruction cmp, Instruction inc0, Instruction inc1> {
1435:   def : Pat<(select (i32 (cond0 GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$other),
1436:             (inc0 (cmp GPR:$rs1, GPR:$rs2), GPR:$other, GPR:$rx, uimm5:$imm)>;
1437:   def : Pat<(select (i32 (cond0 GPR:$rs1, GPR:$rs2)), GPR:$other, (add GPR:$rx, uimm5:$imm)),
1438:             (inc1 (cmp GPR:$rs1, GPR:$rs2), GPR:$other, GPR:$rx, uimm5:$imm)>;
1439:   def : Pat<(select (i32 (cond1 GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5:$imm), GPR:$other),
1440:             (inc0 (cmp GPR:$rs2, GPR:$rs1), GPR:$other, GPR:$rx, uimm5:$imm)>;
```

- EN: This range defines declarative TableGen records such as INCPat, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 INCPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1441-1458

```tablegen
1441:   def : Pat<(select (i32 (cond1 GPR:$rs1, GPR:$rs2)), GPR:$other, (add GPR:$rx, uimm5:$imm)),
1442:             (inc1 (cmp GPR:$rs2, GPR:$rs1), GPR:$other, GPR:$rx, uimm5:$imm)>;
1443: }
1444: 
1445: defm : INCPat<setuge, setule, CMPHS32, INCT32, INCF32>;
1446: defm : INCPat<setult, setugt, CMPHS32, INCF32, INCT32>;
1447: defm : INCPat<setlt, setgt, CMPLT32, INCT32, INCF32>;
1448: defm : INCPat<setge, setle, CMPLT32, INCF32, INCT32>;
1449: 
1450: multiclass DECPat<PatFrag cond0, PatFrag cond1, Instruction cmp, Instruction dec0, Instruction dec1> {
1451:   def : Pat<(select (i32 (cond0 GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$other),
1452:             (dec0 (cmp GPR:$rs1, GPR:$rs2), GPR:$other, GPR:$rx,
1453:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1454:   def : Pat<(select (i32 (cond0 GPR:$rs1, GPR:$rs2)), GPR:$other, (add GPR:$rx, uimm5_neg:$imm)),
1455:             (dec1 (cmp GPR:$rs1, GPR:$rs2), GPR:$other, GPR:$rx,
1456:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1457:   def : Pat<(select (i32 (cond1 GPR:$rs1, GPR:$rs2)), (add GPR:$rx, uimm5_neg:$imm), GPR:$other),
1458:             (dec0 (cmp GPR:$rs2, GPR:$rs1), GPR:$other, GPR:$rx,
```

- EN: This range defines declarative TableGen records such as DECPat, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 DECPat 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1459-1476

```tablegen
1459:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1460:   def : Pat<(select (i32 (cond1 GPR:$rs1, GPR:$rs2)), GPR:$other, (add GPR:$rx, uimm5_neg:$imm)),
1461:             (dec1 (cmp GPR:$rs2, GPR:$rs1), GPR:$other, GPR:$rx,
1462:                   (imm_neg_XFORM uimm5_neg:$imm))>;
1463: }
1464: 
1465: defm : DECPat<setuge, setule, CMPHS32, DECT32, DECF32>;
1466: defm : DECPat<setult, setugt, CMPHS32, DECF32, DECT32>;
1467: defm : DECPat<setlt, setgt, CMPLT32, DECT32, DECF32>;
1468: defm : DECPat<setge, setle, CMPLT32, DECF32, DECT32>;
1469: 
1470: def : Pat<(select (i32 (setne GPR:$rs1, GPR:$rs2)), GPR:$rx, GPR:$false),
1471:           (MOVT32 (CMPNE32 GPR:$rs1, GPR:$rs2), GPR:$rx, GPR:$false)>;
1472: def : Pat<(select (i32 (seteq GPR:$rs1, GPR:$rs2)), GPR:$rx, GPR:$false),
1473:           (MOVF32 (CMPNE32 GPR:$rs1, GPR:$rs2), GPR:$rx, GPR:$false)>;
1474: 
1475: multiclass MOVTF32Pat1<PatFrag cond0, PatFrag cond1, Instruction cmp_inst,
1476:                        Instruction mov_inst> {
```

- EN: This range defines declarative TableGen records such as MOVTF32Pat1, capturing target metadata instead of imperative runtime logic. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 MOVTF32Pat1 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1477-1494

```tablegen
1477:   def : Pat<(select (i32 (cond0 GPR:$rs1, GPR:$rs2)), GPR:$rx, GPR:$false),
1478:             (mov_inst (cmp_inst GPR:$rs1, GPR:$rs2), GPR:$rx, GPR:$false)>;
1479:   def : Pat<(select (i32 (cond1 GPR:$rs1, GPR:$rs2)), GPR:$rx, GPR:$false),
1480:             (mov_inst (cmp_inst GPR:$rs2, GPR:$rs1), GPR:$rx, GPR:$false)>;
1481: }
1482: 
1483: defm : MOVTF32Pat1<setuge, setule, CMPHS32, MOVT32>;
1484: defm : MOVTF32Pat1<setult, setugt, CMPHS32, MOVF32>;
1485: defm : MOVTF32Pat1<setlt, setgt, CMPLT32, MOVT32>;
1486: defm : MOVTF32Pat1<setge, setle, CMPLT32, MOVF32>;
1487: 
1488: def : Pat<(select CARRY:$ca, (i32 0), GPR:$other),
1489:           (CLRT32 CARRY:$ca, GPR:$other)>;
1490: def : Pat<(select CARRY:$ca, GPR:$other, (i32 0)),
1491:           (CLRF32 CARRY:$ca, GPR:$other)>;
1492: }
1493: 
1494: // Constant materialize patterns.
```

- EN: Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1495-1512

```tablegen
1495: let Predicates = [iHasE2] in
1496:   def : Pat<(i32 imm:$imm),
1497:             (ORI32 (MOVIH32 (uimm32_hi16 imm:$imm)), (uimm32_lo16 imm:$imm))>;
1498: 
1499: // Bit operations.
1500: let Predicates = [iHasE2] in {
1501:   def : Pat<(or GPR:$rs, imm32_1_pop_bit:$imm),
1502:             (BSETI32 GPR:$rs, (imm32_1_pop_bit_XFORM imm32_1_pop_bit:$imm))>;
1503:   def : Pat<(and GPR:$rs, imm32_1_zero_bit:$imm),
1504:             (BCLRI32 GPR:$rs, (imm32_1_zero_bit_XFORM imm32_1_zero_bit:$imm))>;
1505: }
1506: 
1507: // Other operations.
1508: let Predicates = [iHasE2] in {
1509:   def : Pat<(rotl GPR:$rs1, GPR:$rs2),
1510:             (ROTL32 GPR:$rs1, (ANDI32 GPR:$rs2, 0x1f))>;
1511:   let Predicates = [iHas2E3] in {
1512:     def : Pat<(bitreverse GPR:$rx), (BREV32 GPR:$rx)>;
```

- EN: `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1513-1530

```tablegen
1513:     def : Pat<(bswap GPR:$rx), (REVB32 GPR:$rx)>;
1514:     def : Pat<(i32 (cttz GPR:$rx)), (FF1 (BREV32 GPR:$rx))>;
1515:   }
1516:   def : Pat<(i32 (ctlz GPR:$rx)), (FF1 GPR:$rx)>;
1517: }
1518: 
1519: //===----------------------------------------------------------------------===//
1520: // Pseudo for assembly
1521: //===----------------------------------------------------------------------===//
1522: 
1523: let isCall = 1, Defs = [ R15 ], mayLoad = 1, Size = 4, isCodeGenOnly = 0 in
1524: def JBSR32 : CSKYPseudo<(outs), (ins call_symbol:$src1), "jbsr32\t$src1", []>;
1525: 
1526: def JBR32 : CSKYPseudo<(outs), (ins br_symbol:$src1), "jbr32\t$src1", []> {
1527:   let isBranch = 1;
1528:   let isTerminator = 1;
1529:   let isBarrier = 1;
1530:   let isIndirectBranch = 1;
```

- EN: This range defines declarative TableGen records such as JBSR32, JBR32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records. Pattern records connect generic LLVM operations to concrete target instructions used during instruction selection.
- 中文: 这一段定义了 JBSR32, JBR32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。 模式记录把通用 LLVM 操作连接到具体目标指令，供指令选择阶段使用。

### Lines 1531-1548

```tablegen
1531:   let mayLoad = 1;
1532:   let Size = 4;
1533: }
1534: 
1535: def JBT32 : CSKYPseudo<(outs), (ins CARRY:$ca, br_symbol:$src1), "jbt32\t$src1", []> {
1536:   let isBranch = 1;
1537:   let isTerminator = 1;
1538:   let isIndirectBranch = 1;
1539:   let mayLoad = 1;
1540:   let Size = 4;
1541: }
1542: 
1543: def JBF32 : CSKYPseudo<(outs), (ins CARRY:$ca, br_symbol:$src1), "jbf32\t$src1", []> {
1544:   let isBranch = 1;
1545:   let isTerminator = 1;
1546:   let isIndirectBranch = 1;
1547:   let mayLoad = 1;
1548:   let Size = 4;
```

- EN: This range defines declarative TableGen records such as JBT32, JBF32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JBT32, JBF32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1549-1566

```tablegen
1549: }
1550: 
1551: def JBT_E : CSKYPseudo<(outs), (ins CARRY:$ca, br_symbol:$src1), "!jbt_e\t$src1", []> {
1552:   let isBranch = 1;
1553:   let isTerminator = 1;
1554:   let isIndirectBranch = 1;
1555:   let mayLoad = 1;
1556:   let Size = 6;
1557: }
1558: 
1559: def JBF_E : CSKYPseudo<(outs), (ins CARRY:$ca, br_symbol:$src1), "!jbf_e\t$src1", []> {
1560:   let isBranch = 1;
1561:   let isTerminator = 1;
1562:   let isIndirectBranch = 1;
1563:   let mayLoad = 1;
1564:   let Size = 6;
1565: }
1566: 
```

- EN: This range defines declarative TableGen records such as JBT_E, JBF_E, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 JBT_E, JBF_E 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1567-1584

```tablegen
1567: let mayLoad = 1, Size = 2, isCodeGenOnly = 0 in
1568: def PseudoLRW32 : CSKYPseudo<(outs GPR:$rz), (ins bare_symbol:$src), "lrw32 $rz, $src", []>;
1569: 
1570: let mayLoad = 1, Size = 4, isCodeGenOnly = 0 in
1571: def PseudoJSRI32 : CSKYPseudo<(outs), (ins call_symbol:$src), "jsri32 $src", []>;
1572: 
1573: let mayLoad = 1, Size = 4, isCodeGenOnly = 0 in
1574: def PseudoJMPI32 : CSKYPseudo<(outs), (ins br_symbol:$src), "jmpi32 $src", []>;
1575: 
1576: let isNotDuplicable = 1, mayLoad = 1, mayStore = 0, Size = 8 in
1577: def PseudoTLSLA32 : CSKYPseudo<(outs GPR:$dst1, GPR:$dst2),
1578:   (ins constpool_symbol:$src, i32imm:$label), "!tlslrw32\t$dst1, $dst2, $src, $label", []>;
1579: 
1580: let hasSideEffects = 0, isNotDuplicable = 1 in
1581: def CONSTPOOL_ENTRY : CSKYPseudo<(outs),
1582:   (ins i32imm:$instid, i32imm:$cpidx, i32imm:$size), "", []>;
1583: 
1584: include "CSKYInstrInfo16Instr.td"
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions. This range defines declarative TableGen records such as PseudoLRW32, PseudoJSRI32, PseudoJMPI32, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。 这一段定义了 PseudoLRW32, PseudoJSRI32, PseudoJMPI32 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 1585-1587

```tablegen
1585: include "CSKYInstrInfoF1.td"
1586: include "CSKYInstrInfoF2.td"
1587: include "CSKYInstrAlias.td"
```

- EN: It imports other TableGen fragments so the current description can reuse shared records, register sets, or instruction definitions.
- 中文: 它引入其他 TableGen 片段，以便当前描述复用共享记录、寄存器集合或指令定义。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- TableGen instruction records / TableGen 指令记录
- Encoding and patterns / 编码与匹配模式
- Pseudo-instruction handling / 伪指令处理
- Pattern matching / 模式匹配

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYInstrFormats.td`, `CSKYInstrInfo16Instr.td`, `CSKYInstrInfoF1.td`, `CSKYInstrInfoF2.td`, `CSKYInstrAlias.td`
- LLVM subsystems / LLVM 子系统: TableGen
- Local companions / 本地配套文件: `CSKYInstrInfo.h`, `CSKYInstrInfo.cpp`
